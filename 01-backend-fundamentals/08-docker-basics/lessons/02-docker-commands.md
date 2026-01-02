# Lesson 2: Docker Commands - أوامر Docker 📝

<div dir="rtl">

## المقدمة

في هذا الدرس هنتعلم أهم Docker commands اللي هتستخدمها يومياً.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Command Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Docker Command Categories                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Images:          docker pull, build, push, images, rmi             │
│  Containers:      docker run, start, stop, rm, ps, logs             │
│  Inspection:      docker inspect, exec, top                         │
│  Networks:        docker network create, ls, connect                │
│  Volumes:         docker volume create, ls, rm                      │
│  System:          docker system df, prune                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Image Commands

### Pull Images

```bash
# Pull from Docker Hub
docker pull nginx                  # Latest tag
docker pull nginx:1.25             # Specific version
docker pull nginx:1.25-alpine      # Alpine variant

# Pull from other registries
docker pull ghcr.io/user/app:1.0
docker pull gcr.io/project/image:tag
```

### List Images

```bash
# List all images
docker images

# Output:
# REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
# nginx        1.25      a6bd71f48f68   2 days ago    187MB
# golang       1.21      8a69a7da5d7e   1 week ago    814MB
# alpine       latest    05455a08881e   2 weeks ago   7.38MB

# Filter images
docker images nginx                     # Only nginx
docker images --filter "dangling=true"  # Untagged images

# Show image IDs only
docker images -q
```

### Build Images

```bash
# Build from Dockerfile in current directory
docker build -t myapp:1.0 .

# Build with specific Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod .

# Build with build arguments
docker build -t myapp:1.0 --build-arg VERSION=1.0 .

# Build without cache
docker build -t myapp:1.0 --no-cache .

# Show build progress
docker build -t myapp:1.0 --progress=plain .
```

### Push Images

```bash
# Login first
docker login
docker login ghcr.io

# Tag for registry
docker tag myapp:1.0 myuser/myapp:1.0

# Push
docker push myuser/myapp:1.0

# Push all tags
docker push myuser/myapp --all-tags
```

### Remove Images

```bash
# Remove specific image
docker rmi nginx:1.25

# Remove by ID
docker rmi a6bd71f48f68

# Force remove (even if container using it)
docker rmi -f nginx:1.25

# Remove all unused images
docker image prune

# Remove ALL images
docker rmi $(docker images -q)
```

---

## 2️⃣ Container Commands

### Run Containers

```bash
# Basic run
docker run nginx

# Run with options
docker run -d \                   # Detached (background)
           -p 8080:80 \           # Port mapping host:container
           --name web \           # Container name
           --restart unless-stopped \  # Restart policy
           nginx:1.25-alpine

# Run interactively
docker run -it ubuntu bash        # Interactive terminal

# Run and remove when done
docker run --rm nginx             # Auto-remove on exit

# Run with environment variables
docker run -e MYSQL_ROOT_PASSWORD=secret mysql

# Run with volume
docker run -v /host/path:/container/path nginx

# Run with resource limits
docker run --memory=512m --cpus=1 nginx
```

### Port Mapping

```bash
# Map specific port
docker run -p 8080:80 nginx       # host:container

# Map multiple ports
docker run -p 8080:80 -p 8443:443 nginx

# Map to specific interface
docker run -p 127.0.0.1:8080:80 nginx  # Only localhost

# Random port mapping
docker run -P nginx               # Map all exposed ports
```

### List Containers

```bash
# Running containers
docker ps

# Output:
# CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS                  NAMES
# abc123         nginx   "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp   web

# All containers (including stopped)
docker ps -a

# Container IDs only
docker ps -q

# Filter
docker ps --filter "status=running"
docker ps --filter "name=web"

# Format output
docker ps --format "{{.Names}}: {{.Status}}"
```

### Start/Stop/Restart

```bash
# Stop running container
docker stop web                   # Graceful stop (SIGTERM)
docker stop web -t 5              # Wait 5 seconds before SIGKILL

# Kill container (force)
docker kill web                   # Immediate SIGKILL

# Start stopped container
docker start web

# Restart container
docker restart web

# Pause/Unpause
docker pause web
docker unpause web
```

### Remove Containers

```bash
# Remove stopped container
docker rm web

# Force remove running container
docker rm -f web

# Remove all stopped containers
docker container prune

# Remove all containers
docker rm -f $(docker ps -aq)
```

---

## 3️⃣ Inspection Commands

### Logs

```bash
# View logs
docker logs web

# Follow logs (tail -f)
docker logs -f web

# Last N lines
docker logs --tail 100 web

# With timestamps
docker logs -t web

# Since specific time
docker logs --since 2024-01-01T00:00:00 web
docker logs --since 1h web         # Last hour
```

### Exec (Run Commands Inside)

```bash
# Run command in container
docker exec web ls /var/www

# Interactive shell
docker exec -it web bash
docker exec -it web sh             # For Alpine

# Run as different user
docker exec -u root web whoami

# With environment variable
docker exec -e MY_VAR=value web env
```

### Inspect

```bash
# Full container info (JSON)
docker inspect web

# Specific info
docker inspect --format='{{.NetworkSettings.IPAddress}}' web
docker inspect --format='{{.State.Status}}' web
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web

# Image info
docker inspect nginx:1.25
```

### Stats and Top

```bash
# Resource usage (live)
docker stats

# Specific container
docker stats web

# One-time snapshot
docker stats --no-stream

# Processes inside container
docker top web
```

---

## 4️⃣ Copy Files

```bash
# Copy from host to container
docker cp ./local/file.txt web:/app/file.txt

# Copy from container to host
docker cp web:/app/logs ./local/logs

# Copy directory
docker cp ./local/dir web:/app/
```

---

## 5️⃣ System Commands

### Disk Usage

```bash
# Show disk usage
docker system df

# Output:
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          10        5         5.2GB     2.1GB (40%)
# Containers      15        3         500MB     400MB (80%)
# Local Volumes   5         3         1GB       200MB (20%)
# Build Cache     50        0         2GB       2GB (100%)

# Detailed
docker system df -v
```

### Cleanup

```bash
# Remove unused data
docker system prune

# Remove EVERYTHING unused
docker system prune -a

# Include volumes
docker system prune -a --volumes

# Remove specific
docker container prune     # Stopped containers
docker image prune         # Dangling images
docker image prune -a      # All unused images
docker volume prune        # Unused volumes
docker network prune       # Unused networks
docker builder prune       # Build cache
```

---

## 6️⃣ Common Workflows

### Development Workflow

```bash
# 1. Build image
docker build -t myapp:dev .

# 2. Run for testing
docker run --rm -p 8080:8080 myapp:dev

# 3. Make changes, rebuild
docker build -t myapp:dev .

# 4. Run again
docker run --rm -p 8080:8080 myapp:dev
```

### Debug Workflow

```bash
# 1. Check container status
docker ps -a

# 2. View logs
docker logs mycontainer

# 3. Get inside container
docker exec -it mycontainer sh

# 4. Inspect container
docker inspect mycontainer

# 5. Check resource usage
docker stats mycontainer
```

### Cleanup Workflow

```bash
# Stop all containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# Remove unused images
docker image prune -a

# Full cleanup
docker system prune -a --volumes
```

---

## 7️⃣ Command Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Docker Commands Cheat Sheet                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  IMAGES                                                             │
│  ───────                                                             │
│  docker pull <image>          Download image                        │
│  docker images                List images                           │
│  docker build -t <name> .     Build image                           │
│  docker push <image>          Push to registry                      │
│  docker rmi <image>           Remove image                          │
│                                                                      │
│  CONTAINERS                                                         │
│  ───────────                                                         │
│  docker run <image>           Run container                         │
│  docker ps                    List running containers               │
│  docker ps -a                 List all containers                   │
│  docker stop <container>      Stop container                        │
│  docker start <container>     Start container                       │
│  docker rm <container>        Remove container                      │
│                                                                      │
│  INSPECTION                                                         │
│  ───────────                                                         │
│  docker logs <container>      View logs                             │
│  docker exec -it <c> sh       Shell into container                  │
│  docker inspect <container>   Detailed info                         │
│  docker stats                 Resource usage                        │
│                                                                      │
│  CLEANUP                                                            │
│  ───────                                                             │
│  docker system prune          Remove unused data                    │
│  docker system prune -a       Remove ALL unused                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **docker run -d** = Run في الـ background
- ✅ **docker run -p** = Map الـ ports
- ✅ **docker run --rm** = Auto-remove عند الإغلاق
- ✅ **docker logs -f** = Follow الـ logs
- ✅ **docker exec -it** = Interactive shell
- ✅ **docker system prune** = Clean up

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم كتابة Dockerfile:

**➡️ [Lesson 3: Dockerfile](./03-dockerfile.md)**

</div>

---

<div align="center">

[⬅️ Previous: What is Docker](./01-what-is-docker.md) | [📚 Module Home](../README.md) | [➡️ Next: Dockerfile](./03-dockerfile.md)

</div>
