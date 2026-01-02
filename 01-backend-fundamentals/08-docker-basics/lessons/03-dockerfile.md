# Lesson 3: Dockerfile - كتابة Dockerfile 📄

<div dir="rtl">

## المقدمة

**Dockerfile** هو ملف نصي يحتوي على التعليمات لبناء Docker image. كل سطر = instruction = layer.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Dockerfile Structure

```dockerfile
# Comment
INSTRUCTION arguments

# Example
FROM golang:1.21-alpine
WORKDIR /app
COPY . .
RUN go build -o main .
EXPOSE 8080
CMD ["./main"]
```

---

## 1️⃣ Essential Instructions

### FROM - Base Image

```dockerfile
# Official image
FROM golang:1.21

# With tag
FROM golang:1.21-alpine

# From scratch (empty)
FROM scratch

# Multiple FROM (multi-stage)
FROM golang:1.21 AS builder
FROM alpine:latest
```

### WORKDIR - Working Directory

```dockerfile
# Set working directory
WORKDIR /app

# All subsequent commands run here
COPY . .           # Copies to /app
RUN go build       # Runs in /app

# Can be set multiple times
WORKDIR /app
WORKDIR src        # Now /app/src
WORKDIR ../config  # Now /app/config
```

### COPY vs ADD

```dockerfile
# COPY - Simple copy (Recommended)
COPY file.txt /app/
COPY *.go /app/
COPY src/ /app/src/

# COPY with --chown
COPY --chown=user:group files/ /app/

# ADD - Extra features (avoid unless needed)
ADD file.txt /app/                  # Same as COPY
ADD https://example.com/file /app/  # Download from URL
ADD archive.tar.gz /app/            # Auto-extract archives
```

### RUN - Execute Commands

```dockerfile
# Shell form
RUN apt-get update
RUN apt-get install -y curl

# Combine commands (fewer layers!)
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Exec form
RUN ["go", "build", "-o", "main", "."]
```

### CMD vs ENTRYPOINT

```dockerfile
# CMD - Default command (can be overridden)
CMD ["./main"]
CMD ["npm", "start"]

# Override with: docker run myapp other-command

# ENTRYPOINT - Always runs (arguments appended)
ENTRYPOINT ["./main"]
CMD ["--port", "8080"]

# docker run myapp              → ./main --port 8080
# docker run myapp --port 9000  → ./main --port 9000
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CMD vs ENTRYPOINT                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Scenario              │ CMD Only        │ ENTRYPOINT + CMD         │
│  ──────────────────────┼─────────────────┼──────────────────────────│
│  docker run img        │ Runs CMD        │ ENTRYPOINT + CMD         │
│  docker run img foo    │ Runs "foo"      │ ENTRYPOINT + "foo"       │
│                                                                      │
│  Use CMD when:                                                      │
│  • Default command, easily overridable                              │
│  • Example: web server, but allow shell access                      │
│                                                                      │
│  Use ENTRYPOINT when:                                               │
│  • Container should always run specific command                     │
│  • Example: CLI tool that takes arguments                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### EXPOSE

```dockerfile
# Document which ports are used (informational)
EXPOSE 8080
EXPOSE 8080/tcp
EXPOSE 8080/udp

# Note: Doesn't actually publish the port!
# Still need: docker run -p 8080:8080
```

### ENV - Environment Variables

```dockerfile
# Set environment variable
ENV APP_ENV=production
ENV PORT=8080

# Multiple
ENV APP_ENV=production \
    PORT=8080 \
    LOG_LEVEL=info

# Use in subsequent instructions
RUN echo $APP_ENV
```

### ARG - Build Arguments

```dockerfile
# Define build argument
ARG VERSION=1.0.0
ARG GO_VERSION=1.21

# Use in FROM
ARG GO_VERSION=1.21
FROM golang:${GO_VERSION}

# Use in other instructions
ARG VERSION
RUN echo "Building version ${VERSION}"
LABEL version="${VERSION}"

# Build with: docker build --build-arg VERSION=2.0.0 .
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                      ARG vs ENV                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                │ ARG                    │ ENV                       │
│  ──────────────┼────────────────────────┼───────────────────────────│
│  Available     │ Build time only        │ Build + Runtime           │
│  Set via       │ --build-arg            │ -e or Dockerfile          │
│  Use case      │ Versions, flags        │ Config, paths             │
│                                                                      │
│  # ARG: Only during build                                           │
│  ARG VERSION=1.0.0                                                  │
│  RUN echo ${VERSION}     # Works                                    │
│  CMD echo ${VERSION}     # Empty at runtime!                        │
│                                                                      │
│  # ENV: Available at runtime too                                    │
│  ENV VERSION=1.0.0                                                  │
│  CMD echo ${VERSION}     # Works!                                   │
│                                                                      │
│  # Pass ARG to ENV                                                  │
│  ARG VERSION                                                        │
│  ENV VERSION=${VERSION}                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### USER

```dockerfile
# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to user
USER appuser

# Subsequent commands run as appuser
COPY --chown=appuser:appgroup . .
CMD ["./main"]
```

### LABEL

```dockerfile
# Add metadata
LABEL maintainer="developer@example.com"
LABEL version="1.0"
LABEL description="My awesome app"

# View labels
# docker inspect --format='{{.Config.Labels}}' myimage
```

---

## 2️⃣ Multi-Stage Builds

<div dir="rtl">

لتقليل حجم الـ final image بشكل كبير!

</div>

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o main .

# Stage 2: Run (final image)
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Multi-Stage Benefits                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Single Stage:                                                      │
│  ┌─────────────────────────────────────────────────┐                │
│  │  golang:1.21 base                    (800MB)    │                │
│  │  + source code                       (10MB)     │                │
│  │  + dependencies                      (100MB)    │                │
│  │  + compiled binary                   (20MB)     │                │
│  │  ═══════════════════════════════════════════    │                │
│  │  Total: ~930MB 😱                              │                │
│  └─────────────────────────────────────────────────┘                │
│                                                                      │
│  Multi-Stage:                                                       │
│  ┌─────────────────────────────────────────────────┐                │
│  │  alpine base                         (7MB)      │                │
│  │  + compiled binary only              (20MB)     │                │
│  │  ═══════════════════════════════════════════    │                │
│  │  Total: ~27MB 🎉                               │                │
│  └─────────────────────────────────────────────────┘                │
│                                                                      │
│  97% smaller!                                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Advanced Multi-Stage

```dockerfile
# Stage 1: Dependencies
FROM golang:1.21 AS deps
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

# Stage 2: Build
FROM deps AS builder
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o main .

# Stage 3: Test
FROM builder AS tester
RUN go test ./...

# Stage 4: Final
FROM scratch
COPY --from=builder /app/main /main
ENTRYPOINT ["/main"]
```

---

## 3️⃣ Best Practices

### Layer Caching

```dockerfile
# ❌ Bad: Cache invalidated on any code change
FROM golang:1.21
COPY . .
RUN go mod download
RUN go build -o main .

# ✅ Good: Dependencies cached separately
FROM golang:1.21
WORKDIR /app
COPY go.mod go.sum ./          # Change rarely
RUN go mod download            # Cached if go.mod unchanged
COPY . .                       # Source changes often
RUN go build -o main .         # Rebuilt on source change
```

### Minimize Layers

```dockerfile
# ❌ Bad: Many layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*

# ✅ Good: Single layer
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*
```

### Non-Root User

```dockerfile
# ✅ Security: Don't run as root
FROM alpine:latest
RUN addgroup -S app && adduser -S app -G app
USER app
WORKDIR /home/app
COPY --chown=app:app ./main .
CMD ["./main"]
```

### .dockerignore

```dockerignore
# .dockerignore file
.git
.gitignore
*.md
Dockerfile
docker-compose.yml
.env
.env.*
node_modules
vendor
tmp
*.log
*.test
__pycache__
.pytest_cache
```

### Order Instructions Wisely

```dockerfile
# Less frequently changed → Top
# More frequently changed → Bottom

FROM golang:1.21-alpine
WORKDIR /app

# 1. System dependencies (rarely change)
RUN apk add --no-cache ca-certificates

# 2. Go dependencies (change sometimes)
COPY go.mod go.sum ./
RUN go mod download

# 3. Source code (changes often)
COPY . .
RUN go build -o main .

# 4. Runtime config
EXPOSE 8080
CMD ["./main"]
```

---

## 4️⃣ Complete Example

### Simple Go App Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

# Build stage
FROM golang:1.21-alpine AS builder

# Install dependencies
RUN apk add --no-cache git ca-certificates

WORKDIR /app

# Download dependencies first (cache layer)
COPY go.mod go.sum ./
RUN go mod download

# Copy source and build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-s -w" -o /app/server ./cmd/server

# Final stage
FROM alpine:3.19

# Security: non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Install runtime dependencies
RUN apk add --no-cache ca-certificates tzdata

WORKDIR /app

# Copy binary from builder
COPY --from=builder /app/server .

# Copy config if needed
COPY --from=builder /app/config ./config

# Set ownership
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

ENTRYPOINT ["./server"]
```

---

## 5️⃣ Debugging Dockerfiles

```bash
# Build with progress output
docker build --progress=plain -t myapp .

# Build up to specific stage
docker build --target builder -t myapp:builder .

# Run intermediate stage for debugging
docker run -it myapp:builder sh

# Check image layers
docker history myapp

# Inspect image
docker inspect myapp
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **FROM** = Base image (ابدأ بـ alpine للـ smaller size)
- ✅ **COPY** > ADD (أبسط وأوضح)
- ✅ **Multi-stage** = Images أصغر بكثير
- ✅ **Layer caching** = ترتيب الـ instructions مهم
- ✅ **Non-root user** = أفضل للـ security
- ✅ **.dockerignore** = لا تنسها!

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم Dockerfile خصوصاً لـ Go:

**➡️ [Lesson 4: Docker for Go](./04-docker-for-go.md)**

</div>

---

<div align="center">

[⬅️ Previous: Docker Commands](./02-docker-commands.md) | [📚 Module Home](../README.md) | [➡️ Next: Docker for Go](./04-docker-for-go.md)

</div>
