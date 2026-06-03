# Dockerfile — Line by Line

## The Final Dockerfile

```dockerfile
FROM httpd:alpine
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
RUN rm -rf /usr/local/apache2/htdocs/*
COPY ./karriku_website/ /usr/local/apache2/htdocs/
CMD ["httpd-foreground"]
```

---

## Explanation

### `FROM httpd:alpine`
- Uses the official Apache HTTP Server image based on Alpine Linux
- Alpine is chosen for its minimal size — under 25MB
- This gives us a fully functional Apache server out of the box

### `COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf`
- Replaces the default Apache config inside the container with our custom one
- The `httpd.conf` was extracted from the base container using:
  ```bash
  docker cp web1:/usr/local/apache2/conf/httpd.conf .
  ```
- This ensures Apache behaves exactly as expected

### `RUN rm -rf /usr/local/apache2/htdocs/*`
- Clears the default Apache welcome page (`index.html`) that comes with the base image
- Without this step, Docker cannot overwrite the existing file when copying our website
- This was a key fix that resolved the `cannot copy to non-directory` build error

### `COPY ./karriku_website/ /usr/local/apache2/htdocs/`
- Copies our website files into Apache's document root
- The trailing slash on `./karriku_website/` ensures the **contents** of the folder are copied, not the folder itself

### `CMD ["httpd-foreground"]`
- Starts the Apache server in the foreground
- Required for Docker — containers need a foreground process to stay running
- This is the default entrypoint for the `httpd` image

---

## How the httpd.conf Was Obtained

Rather than writing an Apache config from scratch, it was extracted directly from a running container:

```bash
# Step 1: Run a temporary container
docker container run --name web1 --rm -d httpd:alpine

# Step 2: Copy the config out
docker cp web1:/usr/local/apache2/conf/httpd.conf .

# Step 3: Container auto-removes after stop (--rm flag)
```

This is a common and practical DevOps technique — use the image itself as a reference.
