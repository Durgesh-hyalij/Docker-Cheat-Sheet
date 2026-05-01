# 🐳 Docker — Complete Guide for Beginners to Intermediate

> **What is Docker?**
> Docker is a platform that packages your application and all its dependencies into a **container** — a lightweight, portable box that runs the same way on any machine. No more "it works on my machine" problems.

---

## 📦 Core Concepts (Understand These First)

| Term | What It Means |
|------|--------------|
| **Image** | A blueprint/template for a container (like a class in OOP) |
| **Container** | A running instance of an image (like an object) |
| **Dockerfile** | A recipe file that tells Docker how to build an image |
| **Docker Hub** | Cloud registry to store and share images (like GitHub for code) |
| **Volume** | Persistent storage that survives container restarts |
| **Network** | How containers talk to each other |
| **Port Mapping** | Bridge between your machine's port and the container's port |

> 💡 **Simple Analogy:** Image = Recipe, Container = Cooked Dish, Docker Hub = Recipe Book Website

---

## 🏗️ Project Setup — Step by Step

### Step 1 — Add `requirements.txt` to Your Project

List all Python dependencies your project needs:

```
flask==3.0.0
requests==2.31.0
# add all your packages here
```

> ⚠️ Always pin versions (e.g. `flask==3.0.0`) for reproducible builds.

---

### Step 2 — Create Your `Dockerfile`

```dockerfile
# ─────────────────────────────────────────
# Base Image — Python 3.11 official image
# ─────────────────────────────────────────
FROM python:3.11

# Set Working Directory inside the container
WORKDIR /app

# Copy all project files from your machine into /app in the container
COPY . .

# Install all Python dependencies (no cache = smaller image size)
RUN pip install --no-cache-dir -r requirements.txt

# Tell Docker this container will use port 5000
EXPOSE 5000

# Command to run when container starts
CMD ["python", "app.py"]
```

> ✅ **Pro Tip:** The `EXPOSE` instruction is documentation — it doesn't actually open the port. The real port binding happens with `-p` flag when running the container.

---

### Step 3 — Configure `app.py` for Docker

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Docker!"

if __name__ == "__main__":
    # host="0.0.0.0" → makes Flask accessible outside the container
    # Without this, Flask only listens inside the container and you can't reach it
    app.run(host="0.0.0.0", port=5000, debug=True)
```

> ⚠️ **Critical:** `host="0.0.0.0"` is mandatory. Without it, your app won't be accessible from outside the container, even with port mapping.

---

### Step 4 — Recommended Project Structure

```
my-project/
├── app.py               ← Your Flask application
├── requirements.txt     ← Python dependencies
├── Dockerfile           ← Docker build instructions
├── .dockerignore        ← Files to exclude from image (see below)
├── static/              ← CSS, JS, images
└── templates/           ← HTML templates
```

---

### Step 5 — Add a `.dockerignore` File *(Often Forgotten!)*

This file tells Docker what NOT to copy into the image — keeps your image lean and secure:

```
# .dockerignore
__pycache__/
*.pyc
*.pyo
.env
.git
.gitignore
*.log
venv/
env/
node_modules/
.DS_Store
```

> 💡 **Why this matters:** Without `.dockerignore`, your `.env` secrets and unnecessary files get baked into the image. Always add this.

---

## 🔨 Building Your Image

```bash
# Syntax: docker build -t username/image_name .
# The dot (.) at the end = "build from current directory"

docker build -t durgeshcoder/personal_project_R .
```

**Breaking it down:**
- `docker build` → command to build an image
- `-t` → tag (give your image a name)
- `durgeshcoder/personal_project_R` → format: `username/project_name`
- `.` → location of Dockerfile (current directory) — **never forget this dot!**

---

## 🚀 Running Your Container

```bash
# Syntax: docker run -p host_port:container_port image_name
docker run -p 8888:5000 durgeshcoder/personal_project_R
```

**Breaking it down:**
- `-p 8888:5000` → map port 8888 on YOUR machine to port 5000 INSIDE the container
- Visit `http://localhost:8888` to see your app

### Useful Run Flags

```bash
# Run in background (detached mode) — container keeps running after you close terminal
docker run -d -p 8888:5000 durgeshcoder/personal_project_R

# Give your container a custom name
docker run -d -p 8888:5000 --name my_app durgeshcoder/personal_project_R

# Pass environment variables
docker run -d -p 8888:5000 -e SECRET_KEY=mysecret durgeshcoder/personal_project_R

# Mount a volume (local folder → container folder)
docker run -d -p 8888:5000 -v $(pwd):/app durgeshcoder/personal_project_R

# Auto-remove container when it stops
docker run --rm -p 8888:5000 durgeshcoder/personal_project_R
```

---

## 📤 Pushing to Docker Hub

```bash
# Step 1 — Login to Docker Hub
docker login
# Enter your Docker Hub username and password when prompted

# Step 2 — Tag your image (if not already tagged with your username)
docker tag <image_name_or_id> yourusername/project_name

# Step 3 — Push to Docker Hub
docker push yourusername/project_name

# Example:
docker push durgeshcoder/personal_project_R
```

> 💡 Anyone with internet can now pull and run your project!

---

## 📥 Pulling from Docker Hub

```bash
# Pull an image from Docker Hub
docker pull durgeshcoder/personal_project_R

# Pull official images (no username needed for official ones)
docker pull python:3.11
docker pull nginx
docker pull postgres
```

---

## 🔍 Viewing & Managing Containers

```bash
# Show only RUNNING containers
docker ps

# Show ALL containers (running + stopped)
docker ps -a

# Show only container IDs
docker ps -q
```

---

## 🖼️ Viewing & Managing Images

```bash
# List all images
docker images

# Count total images
docker images -q | wc -l

# Show image details (layers, history)
docker inspect image_name
docker history image_name
```

---

## ⏹️ Stopping Containers

```bash
# Stop a specific container (graceful — gives it time to clean up)
docker stop <container_id_or_name>

# Kill a container immediately (force stop)
docker kill <container_id_or_name>

# Stop ALL running containers
docker stop $(docker ps -q)
```

---

## 🗑️ Deleting Containers & Images

```bash
# Remove a specific container (must be stopped first)
docker rm <container_id_or_name>

# Force remove a running container
docker rm -f <container_id_or_name>

# Remove ALL stopped containers
docker rm $(docker ps -aq)

# Remove a specific image
docker rmi <image_name_or_id>

# Remove ALL images
docker rmi $(docker images -q)

# Force remove an image (even if containers are using it)
docker rmi -f <image_name_or_id>
```

---

## 🧹 Nuclear Option — Clean Everything

```bash
# Remove ALL containers, images, networks, and build cache
docker system prune -a

# Just remove unused resources (keeps used ones)
docker system prune

# See how much disk space Docker is using
docker system df
```

---

## 🐚 Getting Inside a Running Container

```bash
# Open a bash shell inside a running container
docker exec -it <container_name> bash

# Or sh (for Alpine-based images that don't have bash)
docker exec -it <container_name> sh

# Run a one-off command inside the container
docker exec <container_name> python --version
```

> 💡 **Use case:** Debug issues, inspect files, or run database migrations inside the container.

---

## 📋 Viewing Container Logs

```bash
# See container logs
docker logs <container_name>

# Follow logs in real-time (like tail -f)
docker logs -f <container_name>

# Last 50 lines only
docker logs --tail 50 <container_name>
```

---

## 🔄 Using Docker Compose *(Bonus — For Multi-Container Apps)*

When you have multiple services (Flask app + PostgreSQL + Redis), use `docker-compose.yml`:

```yaml
# docker-compose.yml
version: "3.9"

services:
  web:
    build: .
    ports:
      - "8888:5000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pg_data:/var/lib/postgresql/data

volumes:
  pg_data:
```

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Stop all services
docker-compose down

# Stop and delete volumes too
docker-compose down -v
```

---

## ⚡ Quick Reference Cheat Sheet

| Action | Command |
|--------|---------|
| Build image | `docker build -t name .` |
| Run container | `docker run -p 8888:5000 name` |
| Run in background | `docker run -d -p 8888:5000 name` |
| List running containers | `docker ps` |
| List all containers | `docker ps -a` |
| List images | `docker images` |
| Stop container | `docker stop <name>` |
| Remove container | `docker rm <name>` |
| Remove image | `docker rmi <name>` |
| Shell into container | `docker exec -it <name> bash` |
| View logs | `docker logs -f <name>` |
| Push to hub | `docker push user/name` |
| Pull from hub | `docker pull user/name` |
| Clean everything | `docker system prune -a` |

---

## 🛑 Common Mistakes & Fixes

| Mistake | Fix |
|---------|-----|
| Forgot the `.` in `docker build` | Always end the command with a space and `.` |
| App not accessible after `docker run` | Make sure `host="0.0.0.0"` in Flask |
| Port already in use | Use a different host port: `-p 9999:5000` |
| Changes not reflecting | Rebuild the image: `docker build -t name .` |
| Image too large | Add `.dockerignore` and use `--no-cache-dir` in pip |
| Can't push to Docker Hub | Run `docker login` first |
| Container exits immediately | Check logs: `docker logs <name>` |

---

## 🔗 Useful Links

- [Docker Hub](https://hub.docker.com) — Find and publish images
- [Docker Docs](https://docs.docker.com) — Official documentation
- [Play with Docker](https://labs.play-with-docker.com) — Free browser-based Docker playground
- [Dockerfile Best Practices](https://docs.docker.com/develop/dev-best-practices/)

---

*Made with Durgesh Hyalij❤️ — From zero to Docker hero.*
