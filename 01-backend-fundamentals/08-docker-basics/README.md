# Module 1.8: Docker Basics - أساسيات Docker 🐳

<div dir="rtl">

## نظرة عامة

**Docker** هو أداة containerization تخليك تشغل التطبيقات في بيئة معزولة ومتسقة. في هذا الـ Module هنتعلم الأساسيات.

**المدة المتوقعة:** 3-4 ساعات

</div>

---

## 🎯 Learning Objectives

<div dir="rtl">

بعد إكمال هذا الـ Module، ستتمكن من:

</div>

- ✅ Understand containers vs VMs
- ✅ Build Docker images
- ✅ Run and manage containers
- ✅ Write Dockerfiles for Go apps
- ✅ Use Docker Compose
- ✅ Understand Docker networking

---

## 📊 Why Docker?

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Why Docker?                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Problem: "It works on my machine!" 🤷                              │
│  ──────────────────────────────────────                              │
│                                                                      │
│  Developer Machine:          Production Server:                     │
│  ┌─────────────────┐         ┌─────────────────┐                    │
│  │ Go 1.21         │         │ Go 1.19         │ ← Different!       │
│  │ Ubuntu 22.04    │         │ CentOS 7        │ ← Different!       │
│  │ PostgreSQL 15   │         │ PostgreSQL 12   │ ← Different!       │
│  │ My config files │         │ Other configs   │ ← Different!       │
│  └─────────────────┘         └─────────────────┘                    │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  Solution: Docker Container 📦                                      │
│  ─────────────────────────────                                       │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Same container everywhere!                       │    │
│  │  ┌─────────────────────────────────────────────────────┐     │    │
│  │  │ Go 1.21 + Ubuntu 22.04 + PostgreSQL 15 + Configs    │     │    │
│  │  └─────────────────────────────────────────────────────┘     │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Works on: Dev Machine ✅ | Staging ✅ | Production ✅              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Module Contents

### Lesson 1: What is Docker?
<div dir="rtl">

- Containers vs Virtual Machines
- Docker Architecture
- Images vs Containers
- Docker Hub

</div>

**➡️ [Start Lesson 1](./lessons/01-what-is-docker.md)**

---

### Lesson 2: Docker Commands
<div dir="rtl">

- docker run, stop, start
- docker ps, logs, exec
- docker images, pull, push
- docker rm, rmi

</div>

**➡️ [Start Lesson 2](./lessons/02-docker-commands.md)**

---

### Lesson 3: Dockerfile
<div dir="rtl">

- كتابة Dockerfile
- FROM, COPY, RUN, CMD
- Multi-stage builds
- Best practices

</div>

**➡️ [Start Lesson 3](./lessons/03-dockerfile.md)**

---

### Lesson 4: Docker for Go
<div dir="rtl">

- Dockerfile لتطبيقات Go
- Multi-stage build للـ small images
- Development vs Production
- Hot reload في Development

</div>

**➡️ [Start Lesson 4](./lessons/04-docker-for-go.md)**

---

### Lesson 5: Docker Compose
<div dir="rtl">

- ما هو Docker Compose
- docker-compose.yml
- تشغيل multiple services
- Networking بين Containers

</div>

**➡️ [Start Lesson 5](./lessons/05-docker-compose.md)**

---

### Lesson 6: Docker Networking & Volumes
<div dir="rtl">

- Docker Networks
- Bridge, Host, None
- Volumes للـ persistent data
- Bind mounts

</div>

**➡️ [Start Lesson 6](./lessons/06-networking-volumes.md)**

---

## 🛠️ Prerequisites

<div dir="rtl">

قبل البدء، تأكد من:

</div>

- ✅ Docker installed ([Get Docker](https://docs.docker.com/get-docker/))
- ✅ Basic command line knowledge
- ✅ Completed Go basics modules

### Verify Installation

```bash
# Check Docker version
docker --version
# Docker version 24.0.0, build ...

# Verify Docker is running
docker run hello-world
# Hello from Docker! ...
```

---

## 📊 Docker Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Docker Architecture                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     Docker Client (CLI)                       │    │
│  │                  docker build, run, pull...                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     Docker Daemon                             │    │
│  │                   (dockerd process)                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│           ┌──────────────────┼──────────────────┐                    │
│           ▼                  ▼                  ▼                    │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐              │
│  │   Images    │    │ Containers  │    │  Networks   │              │
│  │             │    │             │    │  & Volumes  │              │
│  └─────────────┘    └─────────────┘    └─────────────┘              │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Docker Hub / Registry                      │    │
│  │                   (Store/Share Images)                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Quick Reference

### Common Commands

```bash
# Images
docker pull nginx                  # Download image
docker images                      # List images
docker rmi nginx                   # Remove image

# Containers
docker run -d -p 80:80 nginx      # Run container
docker ps                          # List running
docker ps -a                       # List all
docker stop <id>                   # Stop container
docker rm <id>                     # Remove container

# Build
docker build -t myapp .            # Build image
docker push myapp                  # Push to registry

# Compose
docker-compose up -d               # Start services
docker-compose down                # Stop services
docker-compose logs -f             # View logs
```

### Dockerfile Cheat Sheet

```dockerfile
FROM golang:1.21-alpine           # Base image
WORKDIR /app                       # Set working dir
COPY go.mod go.sum ./              # Copy files
RUN go mod download                # Run command
COPY . .                           # Copy rest
RUN go build -o main .             # Build app
EXPOSE 8080                        # Document port
CMD ["./main"]                     # Default command
```

---

## ⏭️ Next Module

<div dir="rtl">

بعد إكمال هذا الـ Module، انتقل إلى:

**➡️ [Module 1.9: Architecture Patterns](../09-architecture-patterns/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Caching Basics](../07-caching-basics/README.md) | [🏠 Track 1 Home](../README.md) | [➡️ Next: Architecture Patterns](../09-architecture-patterns/README.md)

</div>
