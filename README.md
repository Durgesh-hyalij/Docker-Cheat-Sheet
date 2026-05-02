<div align="center">

# 🐳 Docker Complete Reference

[![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

[![Maintained](https://img.shields.io/badge/Maintained-yes-green?style=flat-square)](https://github.com/durgeshcoder)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)
[![Made with ❤️](https://img.shields.io/badge/Made%20with-%E2%9D%A4%EF%B8%8F-red?style=flat-square)](https://github.com/durgesh-hyalij)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-durgeshcoder-2CA5E0?style=flat-square&logo=docker)](https://hub.docker.com/u/durgeshcoder)

<br/>

> **The only Docker reference you'll ever need.**
> From zero to production — containers, images, multi-stage builds, volumes, and everything in between.

<br/>

```
docker pull durgeshcoder/knowledge  🚀
```

</div>

---

## 📖 Table of Contents

- [🐳 What is Docker?](#-what-is-docker)
- [⚡ Quick Start](#-quick-start)
- [📦 Dockerizing a Project](#-dockerizing-a-project)
  - [requirements.txt](#1️⃣-requirementstxt)
  - [Dockerfile](#2️⃣-dockerfile)
  - [Flask app.py](#3️⃣-flask-apppy)
- [🔨 Build, Run & Push](#-build-run--push)
- [🏗️ Multi-Stage Builds](#️-multi-stage-builds)
  - [The Problem](#the-problem)
  - [The Solution](#the-solution)
  - [Python Example](#python--flask-multi-stage)
  - [Node.js Example](#nodejs-multi-stage)
  - [Advanced Features](#advanced-features)
- [💾 Docker Volumes](#-docker-volumes)
  - [Why Volumes?](#why-volumes)
  - [3 Types of Storage](#3-types-of-storage)
  - [All Volume Commands](#all-volume-commands)
  - [Volumes in Docker Compose](#volumes-in-docker-compose)
- [📋 Commands Cheatsheet](#-commands-cheatsheet)
- [🐛 Common Errors & Fixes](#-common-errors--fixes)
- [⭐ Best Practices](#-best-practices)

---

## 🐳 What is Docker?

Docker is a platform to **package, ship, and run applications** inside lightweight isolated environments called **containers**.

```
Your App + Dependencies + Config  →  📦 Container  →  Runs anywhere
```

| | Docker Container | Virtual Machine |
|---|---|---|
| **Boot time** | Seconds ⚡ | Minutes 🐌 |
| **Size** | MBs | GBs |
| **OS** | Shares host kernel | Full OS inside |
| **Performance** | Near-native | Overhead |
| **Best for** | Microservices, apps | Full OS simulation |

### 🧱 The Docker Trinity

```
🖼️  Image      →  Read-only snapshot of your app  (like a class)
📦  Container  →  Running instance of an image     (like an object)
🌐  Registry   →  Remote storage for images        (like GitHub)
```

---

## ⚡ Quick Start

```bash
# 1. Check Docker is running
docker --version

# 2. Pull and run your first container
docker run hello-world

# 3. Run nginx web server
docker run -d -p 8080:80 --name myserver nginx

# 4. Visit http://localhost:8080 🎉
```

---

## 📦 Dockerizing a Project

### 1️⃣ requirements.txt

```txt
flask==3.0.0
requests==2.31.0
gunicorn==21.2.0
```

> 💡 **Pro tip:** Auto-generate with `pip freeze > requirements.txt`  
> Or use `pipreqs . --force` for only the packages you actually import.

---

### 2️⃣ Dockerfile

```dockerfile
# Base Image
FROM python:3.11

# Set Working Directory
WORKDIR /app

# Copy requirements FIRST (Docker layer cache optimization)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy rest of the app
COPY . .

# Expose port (documentation only — actual mapping in docker run)
EXPOSE 5000

# Run application
CMD ["python", "app.py"]
```

---

### 3️⃣ Flask app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return 'Hello from Docker! 🐳'

if __name__ == '__main__':
    # host='0.0.0.0' is CRITICAL
    # Without it, Flask only accepts connections from inside the container
    app.run(host="0.0.0.0", port=5000, debug=True)
```

> ⚠️ `host="0.0.0.0"` means "accept connections from any IP" — required for Docker port mapping to work.

---

### 🚫 .dockerignore

Always add a `.dockerignore` — keeps your image clean and your secrets safe:

```
__pycache__/
*.pyc
.env
.env.*
.git/
venv/
.venv/
tests/
*.log
node_modules/
```

---

## 🔨 Build, Run & Push

### Build an Image
```bash
# Syntax
docker build -t username/image_name:tag .
#                                         ^ Don't forget the dot!

# Example
docker build -t durgeshcoder/personal_project_R .

# With version tag
docker build -t durgeshcoder/personal_project_R:v1.0 .
```

### Run a Container
```bash
# Basic run
docker run -p 8888:5000 durgeshcoder/personal_project_R

# Recommended: detached + named
docker run -d -p 8888:5000 --name myapp durgeshcoder/personal_project_R

# With environment variables
docker run -d -p 8888:5000 -e SECRET_KEY=abc123 durgeshcoder/personal_project_R

# With volume mount (live code reload for dev)
docker run -d -p 8888:5000 -v $(pwd):/app durgeshcoder/personal_project_R
```

> 🌐 Access your app at `http://localhost:8888`

### Push to Docker Hub
```bash
# Step 1: Login
docker login

# Step 2: Tag (if needed)
docker tag my_image durgeshcoder/my_image:latest

# Step 3: Push
docker push durgeshcoder/personal_project_R
```

### Pull from Docker Hub
```bash
docker pull durgeshcoder/personal_project_R

# Anyone in the world can run your app with ONE command:
docker run -d -p 8888:5000 durgeshcoder/personal_project_R
```

---

## 🏗️ Multi-Stage Builds

### The Problem

A normal single-stage Dockerfile ships **everything** — build tools, compilers, source code — into your final image:

```dockerfile
# ❌ Single-stage — BAD for production
FROM python:3.11          # ~900MB base
RUN apt-get install -y gcc build-essential  # +300MB
COPY . .
RUN pip install -r requirements.txt
# Final image: ~1.2GB 😱
```

**Problems:**
- 🐘 Huge image (~1.2GB) — slow to push, pull, and deploy
- 🔓 Build tools available to attackers if container is compromised
- 📁 Source code, test files, secrets baked into the image

---

### The Solution

Multi-stage builds separate **build time** from **runtime**:

```
Stage 1 (Builder)  🔨  —  Full toolchain to compile/install your app
         ↓
         COPY --from=builder  (only the compiled output)
         ↓
Stage 2 (Runtime)  🚀  —  Tiny image with only what's needed to RUN
```

**The final image = Stage 2 only. Stage 1 is discarded. Never ships.**

---

### Python / Flask Multi-Stage

```dockerfile
# ════════════════════════════════════════════
# STAGE 1 — BUILDER
# Full Python image with build tools (gcc etc.)
# ════════════════════════════════════════════
FROM python:3.11 AS builder

WORKDIR /app

COPY requirements.txt .

# Install to /install (not system Python)
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt


# ════════════════════════════════════════════
# STAGE 2 — RUNTIME  ← This is your final image
# slim = Python + minimal OS (~150MB vs ~900MB)
# ════════════════════════════════════════════
FROM python:3.11-slim AS runtime

# Security: create non-root user
RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /app

# Copy ONLY installed packages from builder
COPY --from=builder /install /usr/local

# Copy app source (with correct ownership)
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

EXPOSE 5000

LABEL maintainer="durgeshcoder" version="1.0"

# Production server (not Flask dev server)
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
```

📊 **Size comparison:**

| Dockerfile | Base Image | Final Size |
|---|---|---|
| Single-stage | `python:3.11` | ~1.1 GB |
| Multi-stage | `python:3.11-slim` | ~180 MB |
| **Reduction** | | **~85% smaller 🚀** |

---

### Node.js Multi-Stage

```dockerfile
# ── STAGE 1: Production dependencies only
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production


# ── STAGE 2: Build (needs dev deps like TypeScript, webpack)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build          # Outputs to /app/dist


# ── STAGE 3: Runtime (Final image)
FROM node:20-alpine AS runtime

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=deps    /app/node_modules ./node_modules
COPY --from=builder /app/dist         ./dist
COPY --from=builder /app/package.json ./

USER appuser
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

### Advanced Features

#### 🎯 --target: Build Only a Specific Stage

```bash
# Build only up to 'builder' (useful for running tests in CI)
docker build --target builder -t myapp:test .

# Run tests inside builder stage
docker run myapp:test pytest tests/

# Build full production image
docker build --target runtime -t myapp:prod .
```

#### 🔑 Build ARGs: Pass Variables at Build Time

```dockerfile
FROM python:3.11 AS builder
ARG APP_VERSION=1.0.0
ARG PIP_EXTRA_INDEX_URL

RUN pip install --extra-index-url ${PIP_EXTRA_INDEX_URL} -r requirements.txt
# ARG is NOT available in later stages — by design (security)
```

```bash
docker build --build-arg APP_VERSION=2.1.0 -t myapp .
```

#### ⚡ Cache Mounts: Superfast Rebuilds (BuildKit)

```dockerfile
# Packages cached between builds — not re-downloaded every time!
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --prefix=/install -r requirements.txt
```

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1
docker build -t myapp .
```

---

## 💾 Docker Volumes

### Why Volumes?

> 🔴 **The Problem:** Container data is ephemeral. When a container is deleted, ALL its data is gone forever.

```
Without volumes:
  Run PostgreSQL → Add data → Delete container → 💀 All data gone

With volumes:
  Run PostgreSQL → Add data → Delete container → ✅ Data still exists
  Recreate container → Mount same volume → Data is back
```

---

### 3 Types of Storage

| Type | Storage Location | Best For |
|---|---|---|
| 📦 **Named Volume** | Docker managed (`/var/lib/docker/volumes/`) | Databases, persistent app data |
| 🔗 **Bind Mount** | Specific path on your host machine | Dev (live reload), config files |
| ⚡ **tmpfs** | RAM only (never touches disk) | Sensitive temp data, performance |

---

#### 📦 Named Volumes — Recommended for Production

```bash
# Create
docker volume create pgdata

# Use with -v flag
docker run -d \
  -v pgdata:/var/lib/postgresql/data \
  --name mydb \
  postgres:15

# Use with --mount (more explicit, preferred in scripts)
docker run -d \
  --mount type=volume,source=pgdata,target=/var/lib/postgresql/data \
  postgres:15
```

✅ Docker manages the path — works on all OS  
✅ Data survives container deletion  
✅ Easy to backup and migrate  

---

#### 🔗 Bind Mounts — Perfect for Development

```bash
# Mount current directory (live code reload!)
docker run -d \
  -p 5000:5000 \
  -v $(pwd):/app \
  --name flask_dev \
  myapp
# Edit app.py on your laptop → change is instantly inside the container!

# Mount specific file as read-only
docker run -d \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx
```

⚠️ Not recommended for production — use named volumes for databases  

---

#### ⚡ tmpfs — RAM-only Storage

```bash
# Stored in RAM, never written to disk
docker run -d \
  --tmpfs /tmp \
  --name myapp \
  myapp
```

✅ Perfect for session tokens, auth keys  
✅ High-performance scratch space  
✅ Data auto-deleted when container stops  

---

### When to Use Which?

| Use Case | Use This | Why |
|---|---|---|
| PostgreSQL / MySQL data | Named Volume | Must survive restarts |
| Dev: live code changes | Bind Mount | Host edits → instant in container |
| nginx config file | Bind Mount `:ro` | Read-only, specific file |
| Session tokens / secrets | tmpfs | Never touch disk |
| Uploaded user files | Named Volume | Persist across deploys |
| Sharing data between containers | Named Volume | Multiple containers, one source |
| Production DB data | Named Volume | **Never** bind mount in production |

---

### All Volume Commands

```bash
# ── Create & List ──────────────────────────────────────────
docker volume create myvolume          # Create named volume
docker volume ls                       # List all volumes
docker volume ls -f dangling=true      # List unused volumes only

# ── Inspect ────────────────────────────────────────────────
docker volume inspect myvolume         # See storage path, driver, etc.
docker ps --filter volume=myvolume     # Which containers use this volume?

# ── Remove ─────────────────────────────────────────────────
docker volume rm myvolume              # Remove specific volume
docker volume prune                    # Remove all unused volumes
docker volume prune -f                 # Remove without confirmation

# ── Backup & Restore ───────────────────────────────────────
# Backup
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/pgdata_backup.tar.gz -C /data .

# Restore
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/pgdata_backup.tar.gz -C /data
```

---

### Volumes in Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8888:5000"
    volumes:
      - app_data:/app/uploads        # Named volume
      - .:/app                       # Bind mount (dev only)
      - ./config.py:/app/config.py:ro  # Read-only config

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data  # ALWAYS use named volume for DB!
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

# Declare all named volumes here
volumes:
  pgdata:
  app_data:
  redis_data:
```

```bash
docker compose up -d          # Start — volumes auto-created
docker compose down           # Stop — volumes PRESERVED ✅
docker compose down -v        # Stop + DELETE volumes ⚠️ data gone!
```

---

## 📋 Commands Cheatsheet

### 🖼️ Images
```bash
docker build -t name:tag .        # Build image
docker images                     # List all images
docker images -q | wc -l          # Count images
docker rmi image_name             # Remove image
docker rmi $(docker images -q)    # Remove ALL images
docker history image_name         # Show build layers
docker image prune -a             # Remove all unused images
```

### 📦 Containers
```bash
docker run -d -p 8888:5000 --name myapp image   # Run (background)
docker run -it image bash                        # Run with shell
docker ps                                        # Running containers
docker ps -a                                     # All containers
docker stop myapp                                # Graceful stop
docker kill myapp                                # Force stop
docker rm myapp                                  # Remove container
docker rm -f myapp                               # Force remove
docker rm $(docker ps -aq)                       # Remove ALL containers
docker stop $(docker ps -aq)                     # Stop ALL containers
```

### 🔍 Inspect & Debug
```bash
docker logs myapp                 # View logs
docker logs -f myapp              # Follow live logs
docker exec -it myapp bash        # Shell into container
docker exec -it myapp sh          # Shell (alpine/slim images)
docker stats                      # Live CPU/RAM usage
docker inspect myapp              # Full container details
docker top myapp                  # Processes inside container
docker port myapp                 # Port mappings
docker cp myapp:/app/file.txt .   # Copy file from container
```

### 🐳 Docker Hub
```bash
docker login                            # Login
docker tag image username/repo:tag      # Tag image
docker push username/repo               # Upload to Hub
docker pull username/repo               # Download from Hub
```

### 🧹 Cleanup
```bash
docker system prune -a             # Remove everything (nuclear)
docker system df                   # Show disk usage
docker container prune             # Remove stopped containers
docker image prune -a              # Remove unused images
docker volume prune                # Remove unused volumes
docker network prune               # Remove unused networks
```

---

## 🐛 Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `Port already in use` | Another process on that port | Change host port: `-p 9000:5000` |
| `Cannot connect to daemon` | Docker Desktop not running | Start Docker Desktop first |
| `Image not found` | Wrong name or not pulled | Check spelling or `docker pull` first |
| `Permission denied` (Linux) | User not in docker group | `sudo usermod -aG docker $USER` |
| `No space left on device` | Docker using too much disk | `docker system prune -a` |
| Container exits immediately | Error in app or missing CMD | `docker logs <container>` to debug |
| App unreachable in browser | Flask not on `0.0.0.0` | Add `host="0.0.0.0"` in `app.run()` |
| DB data gone after restart | Volume not mounted | Add `-v pgdata:/var/lib/postgresql/data` |
| Can't remove volume | Container still using it | `docker rm container` first |

---

## ⭐ Best Practices

```
✅  Use specific version tags: FROM python:3.11  (not python:latest)
✅  Copy requirements.txt BEFORE your code  →  faster rebuilds via cache
✅  Use slim/alpine base images in production  →  85% smaller images
✅  Always add .dockerignore  →  faster builds, no secrets leaking
✅  Run as non-root user  →  USER appuser (security hardening)
✅  Never hardcode secrets  →  use -e flags or .env files
✅  Always use named volumes for databases  →  data survives restarts
✅  Tag images with versions  →  v1.0, v1.1  (easy rollback)
✅  Use docker compose for multi-service apps
✅  Use -d flag in production  →  always run detached
✅  Use COPY --chown=user:group  →  correct file ownership in one step
✅  Name all stages  →  COPY --from=builder  (not COPY --from=0)
```

---

## 📁 Repository Structure

```
📦 docker-cheatsheet
 ┣ 📄 README.md                        ← You are here
 ┣ 📄 Dockerfile                       ← Basic single-stage example
 ┣ 📄 Dockerfile.multistage            ← Multi-stage build example
 ┣ 📄 docker-compose.yml               ← Multi-service setup
 ┣ 📄 .dockerignore                    ← Files to exclude from build
 ┣ 📄 requirements.txt                 ← Python dependencies
 ┣ 📄 app.py                           ← Flask app
 ┗ 📂 docs
    ┣ 📄 Docker_Complete_Reference.docx
    ┗ 📄 Docker_MultiStage_Volumes.docx
```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the repository
2. Create your branch: `git checkout -b feature/add-kubernetes-notes`
3. Commit your changes: `git commit -m 'Add Kubernetes basics'`
4. Push to the branch: `git push origin feature/add-kubernetes-notes`
5. Open a Pull Request

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.

---

<div align="center">

**Made with ❤️ by [durgeshcoder](https://github.com/durgesh-hyalij)**

[![GitHub followers](https://img.shields.io/github/followers/durgeshcoder?style=social)](https://github.com/durgesh-hyalij)
[![Docker Hub Pulls](https://img.shields.io/badge/Docker%20Hub-durgeshcoder-2CA5E0?style=flat&logo=docker)](https://hub.docker.com/u/durgeshcoder)

⭐ **Star this repo if it helped you!** ⭐

```
docker run --rm durgeshcoder/thank-you  💙
```

</div>
