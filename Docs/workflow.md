# Full Workflow — Step by Step

This document walks through the entire project from start to finish.

---

## Step 1 — Clone the Repo and Prepare Files

The project started with cloning a GitHub repository containing the Karikku website HTML files.

```bash
# Rename the website file
mv "Coco Bliss - Kerala Edition.html" index.html

# Create the website folder
mkdir karriku_website

# Move the file into the folder
mv index.html karriku_website/

# Verify
ls karriku_website/
# index.html
```

---

## Step 2 — Extract Apache Config from Base Image

Instead of writing an `httpd.conf` from scratch, it was pulled directly from the official Apache container:

```bash
# Run a temporary Apache container
docker container run --name web1 --rm -d httpd:alpine

# Copy the config file to local machine
docker cp web1:/usr/local/apache2/conf/httpd.conf .

# Verify
ls
# httpd.conf  karriku_website
```

---

## Step 3 — Write the Dockerfile

```dockerfile
FROM httpd:alpine
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
RUN rm -rf /usr/local/apache2/htdocs/*
COPY ./karriku_website/ /usr/local/apache2/htdocs/
CMD ["httpd-foreground"]
```

---

## Step 4 — Build the Docker Image

```bash
docker image build -t karrikuweb:v2 .
```

Successful output ends with:
```
=> naming to docker.io/library/karrikuweb:v2
```

---

## Step 5 — Run the Container

```bash
docker run -d -p 8082:80 karrikuweb:v2
```

- `-d` → Run in detached (background) mode
- `-p 8082:80` → Map host port 8082 to container port 80

---

## Step 6 — Verify the Website

**Browser:**
```
http://localhost:8082
```

**Terminal:**
```bash
curl -I http://localhost:8082
# HTTP/1.1 200 OK  ✅
```

---

## Step 7 — Login to Docker Hub

```bash
docker login
```

---

## Step 8 — Tag and Push to Docker Hub

```bash
# Tag with Docker Hub username
docker tag karrikuweb:v2 haseebspaniard/karrikuweb:v2
docker tag karrikuweb:v2 haseebspaniard/karrikuweb:latest

# Push both tags
docker push haseebspaniard/karrikuweb:v2
docker push haseebspaniard/karrikuweb:latest
```

---

## Step 9 — Test the Pull from Docker Hub

This step simulates what any user would experience pulling the image fresh:

```bash
# Remove local image
docker rmi haseebspaniard/karrikuweb:v2

# Pull from Docker Hub and run
docker run -d -p 8083:80 haseebspaniard/karrikuweb:v2
```

Visit `http://localhost:8083` — the Karikku website loads successfully. ✅

---

## Summary

| Step | Action | Command |
|------|--------|---------|
| 1 | Prepare files | `mv`, `mkdir` |
| 2 | Get Apache config | `docker cp` |
| 3 | Write Dockerfile | `nano Dockerfile` |
| 4 | Build image | `docker image build` |
| 5 | Run container | `docker run` |
| 6 | Verify | `curl -I` |
| 7 | Login | `docker login` |
| 8 | Push to Hub | `docker push` |
| 9 | Pull and test | `docker run` |
