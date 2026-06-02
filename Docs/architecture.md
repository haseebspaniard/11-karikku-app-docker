# 🏗️ Architecture

## Overview

This project follows a simple but complete DevOps containerization workflow — from source files on a local machine to a running container accessible via browser.

---

## System Diagram

```
┌─────────────────────────────────────┐
│         Local Machine               │
│  Ubuntu Linux (VirtualBox)          │
│                                     │
│  ┌──────────────────────────────┐   │
│  │  Project Folder              │   │
│  │  ├── karriku_website/        │   │
│  │  │   └── index.html          │   │
│  │  ├── httpd.conf              │   │
│  │  └── Dockerfile              │   │
│  └──────────────────────────────┘   │
│              │                      │
│        docker build                 │
│              │                      │
│  ┌───────────▼──────────────────┐   │
│  │  Docker Image                │   │
│  │  karrikuweb:v2               │   │
│  │  (httpd:alpine base)         │   │
│  └───────────┬──────────────────┘   │
│              │                      │
│        docker run -p 8082:80        │
│              │                      │
│  ┌───────────▼──────────────────┐   │
│  │  Running Container           │   │
│  │  Apache serving index.html   │   │
│  │  localhost:8082              │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
              │
        docker push
              │
┌─────────────▼───────────────────────┐
│         Docker Hub                  │
│  haseebspaniard/karrikuweb:v2       │
│  haseebspaniard/karrikuweb:latest   │
└─────────────────────────────────────┘
              │
        docker pull (from anywhere)
              │
┌─────────────▼───────────────────────┐
│    Any Machine with Docker          │
│    docker run -p 8080:80            │
│    haseebspaniard/karrikuweb:v2     │
└─────────────────────────────────────┘
```

---

## Components

### Base Image — `httpd:alpine`
- Minimal Alpine Linux-based Apache HTTP Server
- Lightweight (~22MB content size)
- Production-ready Apache 2.4

### Custom `httpd.conf`
- Extracted directly from the base container using `docker cp`
- Ensures Apache is configured correctly for serving static files

### Website Files — `karriku_website/`
- Single-page static HTML website
- Kerala-themed (Karikku brand)
- Placed in Apache's document root `/usr/local/apache2/htdocs/`

### Docker Hub Registry
- Image versioned as `v1`, `v2`, and `latest`
- Publicly accessible — anyone can pull and run it

---

## Port Mapping

| Host Port | Container Port | Service |
|-----------|---------------|---------|
| 8082 | 80 | Apache HTTP Server |

---

## Image Versioning

| Tag | Status | Notes |
|-----|--------|-------|
| `v1` | ❌ Deleted | Broken — index.html was a directory |
| `v2` | ✅ Live | Fixed and working |
| `latest` | ✅ Live | Points to v2 |
