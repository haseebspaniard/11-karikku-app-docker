# 🔧 Troubleshooting — Errors & Fixes

This document covers all the real errors encountered during this project and exactly how they were resolved. This is meant to help others who face the same issues.

---

## Error 1 — `cannot copy to non-directory`

### The Error
```
ERROR: failed to build: failed to solve: cannot copy to non-directory:
/var/lib/docker/buildkit/.../usr/local/apache2/htdocs/index.html
```

### Why It Happened
The `httpd:alpine` base image already contains a default `index.html` file at `/usr/local/apache2/htdocs/`. Docker couldn't copy the website folder's contents over this existing file.

### The Fix
Added a `RUN` step to clear the existing default files before copying:

```dockerfile
RUN rm -rf /usr/local/apache2/htdocs/*
```

Updated Dockerfile:
```dockerfile
FROM httpd:alpine
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
RUN rm -rf /usr/local/apache2/htdocs/*
COPY ./karriku_website/ /usr/local/apache2/htdocs/
CMD ["httpd-foreground"]
```

---

## Error 2 — `docker buildx build requires 1 argument`

### The Error
```
ERROR: docker: 'docker buildx build' requires 1 argument
```

### Why It Happened
The build command was run without specifying the build context (`.`):
```bash
docker image build -t karrikuweb:v1   # ← missing the dot
```

### The Fix
Always include `.` at the end to tell Docker to use the current directory as the build context:
```bash
docker image build -t karrikuweb:v1 .
```

---

## Error 3 — 301 Redirect to WordPress

### The Error
Opening `http://localhost:8081` was redirecting to `http://localhost/index.html/` — which was opening a WordPress site running on port 80.

### Why It Happened
The `index.html` inside `karriku_website/` was actually a **directory**, not a file. The folder structure was:
```
karriku_website/
└── index.html/        ← This is a FOLDER, not a file!
    └── index.html     ← Actual HTML file was inside here
```

Apache saw `index.html` as a directory and issued a 301 redirect with a trailing slash, which landed on the WordPress instance at `localhost:80`.

### Diagnosed With
```bash
docker exec -it cool_proskuriakova cat /usr/local/apache2/htdocs/index.html
# cat: read error: Is a directory
```

### The Fix
Manually restructured the files on the host:
```bash
# Move the actual file out with a temp name
mv karriku_website/index.html/index.html karriku_website/index_temp.html

# Remove the now-empty directory
rm -d karriku_website/index.html

# Rename back to index.html
mv karriku_website/index_temp.html karriku_website/index.html

# Verify — should show file, not directory
ls -la karriku_website/
```

Then rebuilt as `v2`:
```bash
docker image build -t karrikuweb:v2 .
```

---

## Error 4 — Push Access Denied

### The Error
```
push access denied, repository does not exist or may require authorization:
server message: insufficient_scope: authorization failed
```

### Why It Happened
Tried to push the image using just the image name without the Docker Hub username prefix:
```bash
docker image push karrikuweb:v1   # ← missing username prefix
```

Docker tried to push to `docker.io/library/karrikuweb` which is the official Docker library — access denied.

### The Fix
Tag the image with your Docker Hub username first, then push that tag:
```bash
docker tag karrikuweb:v1 haseebspaniard/karrikuweb:v1
docker push haseebspaniard/karrikuweb:v1
```

---

## Error 5 — `cannot remove: image is referenced in multiple repositories`

### The Error
```
Error response from daemon: conflict: unable to delete 5ce7f84a8f49
(must be forced) - image is referenced in multiple repositories
```

### Why It Happened
The same image ID was tagged with multiple names (`karrikuweb:v1`, `haseebspaniard/karrikuweb:v1`, `haseebspaniard/karrikuweb:latest`). Docker won't delete an image ID when multiple tags still reference it.

### The Fix
Remove each tag individually instead of trying to delete by image ID:
```bash
docker rmi karrikuweb:v1
docker rmi haseebspaniard/karrikuweb:v1
# Now the image ID has no more references and is gone
```

---

## Error 6 — Wrong Image Pulled After Deletion

### What Happened
After deleting the local `v1` image and running the container again, Docker pulled the **broken v1** from Docker Hub (the one with `index.html` as a directory) instead of using a fixed local image.

### Why It Happened
The broken image had already been pushed to Docker Hub as `v1`. When it was deleted locally, Docker simply pulled it back from the registry.

### The Fix
- Delete `v1` from Docker Hub via the website
- Build the fixed image as `v2`
- Push `v2` to Docker Hub

**Lesson learned:** Always verify the image works locally before pushing to Docker Hub.

---

## Key Lessons

| # | Lesson |
|---|--------|
| 1 | Always add `.` at the end of `docker build` commands |
| 2 | Clear default image files before copying your own |
| 3 | Verify file types with `ls -la` before building |
| 4 | Always prefix image names with your Docker Hub username before pushing |
| 5 | Test locally thoroughly before pushing to a registry |
| 6 | Pushing a broken image to Docker Hub doesn't disappear — clean it up |
