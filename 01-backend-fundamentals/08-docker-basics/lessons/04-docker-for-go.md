# Lesson 4: Docker for Go - Docker لتطبيقات Go 🐹

<div dir="rtl">

## المقدمة

Go و Docker مثاليين مع بعض! Go ينتج binary واحد statically linked، وده بيخلي الـ Docker images صغيرة جداً.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 Why Go + Docker is Great

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Go + Docker Advantages                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Go produces:                                                       │
│  ─────────────                                                       │
│  • Single binary (no runtime needed)                                │
│  • Statically linked (no dependencies)                              │
│  • Cross-compilation built-in                                       │
│  • Small binaries (10-30MB typically)                               │
│                                                                      │
│  Result:                                                            │
│  ─────────                                                           │
│  • Final Docker image: 10-30MB (vs 800MB+ with runtime)             │
│  • Can run from `scratch` (empty) image                             │
│  • Fast container startup                                           │
│  • Minimal attack surface                                           │
│                                                                      │
│  Comparison:                                                        │
│  ────────────                                                        │
│  │ Language │ Base Image      │ Final Size │                       │
│  ├──────────┼─────────────────┼────────────┤                       │
│  │ Go       │ scratch         │ 15MB       │                       │
│  │ Go       │ alpine          │ 20MB       │                       │
│  │ Node.js  │ node:alpine     │ 150MB+     │                       │
│  │ Python   │ python:slim     │ 200MB+     │                       │
│  │ Java     │ eclipse-temurin │ 300MB+     │                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Production Dockerfile

### Minimal Production Build

```dockerfile
# syntax=docker/dockerfile:1

# ==================== Build Stage ====================
FROM golang:1.21-alpine AS builder

# Install git for private dependencies (if needed)
RUN apk add --no-cache git ca-certificates tzdata

# Create non-root user for final image
RUN adduser -D -g '' appuser

WORKDIR /app

# Download dependencies (cached layer)
COPY go.mod go.sum ./
RUN go mod download && go mod verify

# Copy source
COPY . .

# Build with optimizations
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build \
    -ldflags='-w -s -extldflags "-static"' \
    -o /app/server \
    ./cmd/server

# ==================== Final Stage ====================
FROM scratch

# Import from builder
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /etc/passwd /etc/passwd

# Copy binary
COPY --from=builder /app/server /server

# Use non-root user
USER appuser

# Expose port
EXPOSE 8080

# Run
ENTRYPOINT ["/server"]
```

### Build Flags Explained

```bash
CGO_ENABLED=0     # Disable CGO for static binary
GOOS=linux        # Target OS
GOARCH=amd64      # Target architecture (or arm64 for M1/M2)

-ldflags='-w -s'  # Strip debug info (smaller binary)
# -w: Omit DWARF symbol table
# -s: Omit symbol table and debug info
# Result: ~30% smaller binary

-extldflags "-static"  # Fully static linking
```

---

## 2️⃣ Alpine vs Scratch

### Using Alpine (Recommended for most cases)

```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -ldflags='-s -w' -o server .

FROM alpine:3.19
RUN apk --no-cache add ca-certificates tzdata
COPY --from=builder /app/server /server
EXPOSE 8080
CMD ["/server"]
```

### Using Scratch (Smallest possible)

```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags='-s -w' -o server .

FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

### Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Alpine vs Scratch                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                 │ Alpine                │ Scratch                   │
│  ───────────────┼───────────────────────┼───────────────────────────│
│  Base size      │ ~7MB                  │ 0MB                       │
│  Shell access   │ ✅ Yes (ash)          │ ❌ No                     │
│  Package mgr    │ ✅ apk                │ ❌ None                   │
│  Debugging      │ ✅ Easy               │ ❌ Hard                   │
│  Security       │ Good                  │ Best (no shell!)          │
│  Use when       │ Need debugging,       │ Maximum security,         │
│                 │ extra tools           │ smallest size             │
│                                                                      │
│  Recommendation:                                                    │
│  • Development: Alpine (easier debugging)                           │
│  • Production: Scratch or Distroless (security)                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ Development Dockerfile

### With Hot Reload (Air)

```dockerfile
# Dockerfile.dev
FROM golang:1.21-alpine

# Install Air for hot reload
RUN go install github.com/cosmtrek/air@latest

WORKDIR /app

# Copy go.mod first for caching
COPY go.mod go.sum ./
RUN go mod download

# Source will be mounted as volume
# COPY . .  # Not needed - mounted instead

EXPOSE 8080

CMD ["air", "-c", ".air.toml"]
```

### Air Configuration (.air.toml)

```toml
# .air.toml
root = "."
tmp_dir = "tmp"

[build]
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ./cmd/server"
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor"]
  exclude_regex = ["_test.go"]
  include_ext = ["go", "tpl", "tmpl", "html"]
  kill_delay = "0s"

[log]
  time = false

[color]
  main = "magenta"
  watcher = "cyan"
  build = "yellow"
  runner = "green"

[misc]
  clean_on_exit = true
```

### Docker Compose for Development

```yaml
# docker-compose.dev.yml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8080:8080"
    volumes:
      - .:/app                    # Mount source code
      - go-modules:/go/pkg/mod    # Cache modules
    environment:
      - APP_ENV=development
      - DB_HOST=db
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  go-modules:
  postgres-data:
```

---

## 4️⃣ Multi-Architecture Build

```dockerfile
# Build for multiple architectures
FROM --platform=$BUILDPLATFORM golang:1.21-alpine AS builder

ARG TARGETPLATFORM
ARG TARGETOS
ARG TARGETARCH

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .

# Build for target platform
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -ldflags='-s -w' -o server .

FROM alpine:3.19
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

```bash
# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:latest \
  --push \
  .
```

---

## 5️⃣ Testing in Docker

```dockerfile
# Dockerfile.test
FROM golang:1.21-alpine

RUN apk add --no-cache gcc musl-dev

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Run tests
CMD ["go", "test", "-v", "-race", "-cover", "./..."]
```

```bash
# Run tests in Docker
docker build -f Dockerfile.test -t myapp:test .
docker run --rm myapp:test
```

### With Coverage Report

```dockerfile
FROM golang:1.21-alpine

WORKDIR /app
COPY . .
RUN go mod download

# Run tests with coverage
RUN go test -coverprofile=coverage.out ./...
RUN go tool cover -html=coverage.out -o coverage.html

# Copy coverage report out
CMD ["cat", "coverage.out"]
```

---

## 6️⃣ Complete Project Structure

```
myapp/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handler/
│   ├── service/
│   └── repository/
├── pkg/
├── config/
│   └── config.yaml
├── migrations/
├── scripts/
├── .air.toml
├── .dockerignore
├── Dockerfile
├── Dockerfile.dev
├── docker-compose.yml
├── docker-compose.dev.yml
├── go.mod
├── go.sum
└── Makefile
```

### .dockerignore

```dockerignore
# Git
.git
.gitignore

# IDE
.idea
.vscode
*.swp

# Build artifacts
bin/
tmp/
*.exe

# Test files
*_test.go
coverage.*

# Docs
*.md
docs/

# Docker
Dockerfile*
docker-compose*
.docker

# CI
.github
.gitlab-ci.yml

# Dev files
.env.local
.air.toml

# Vendor (if not vendoring)
# vendor/
```

### Makefile

```makefile
.PHONY: build run test docker-build docker-run docker-dev

# Go commands
build:
	go build -o bin/server ./cmd/server

run:
	go run ./cmd/server

test:
	go test -v ./...

# Docker commands
docker-build:
	docker build -t myapp:latest .

docker-run:
	docker run -p 8080:8080 myapp:latest

docker-dev:
	docker-compose -f docker-compose.dev.yml up --build

docker-down:
	docker-compose down

# Multi-platform
docker-buildx:
	docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

---

## 7️⃣ Security Best Practices

```dockerfile
# 1. Use specific version tags
FROM golang:1.21.5-alpine3.19 AS builder  # Not :latest

# 2. Run as non-root
FROM alpine:3.19
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser
USER appuser

# 3. Read-only filesystem
# docker run --read-only myapp

# 4. Drop capabilities
# docker run --cap-drop=ALL myapp

# 5. Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
    CMD wget --spider http://localhost:8080/health || exit 1

# 6. Don't store secrets in image
# Use docker secrets or environment variables

# 7. Scan for vulnerabilities
# docker scout cves myapp:latest
# or: trivy image myapp:latest
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Multi-stage build** = images أصغر 10x
- ✅ **CGO_ENABLED=0** = static binary
- ✅ **-ldflags='-s -w'** = smaller binary
- ✅ **scratch** للـ production، **alpine** للـ debugging
- ✅ **Air** للـ hot reload في development
- ✅ **Non-root user** = أفضل للـ security

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم Docker Compose:

**➡️ [Lesson 5: Docker Compose](./05-docker-compose.md)**

</div>

---

<div align="center">

[⬅️ Previous: Dockerfile](./03-dockerfile.md) | [📚 Module Home](../README.md) | [➡️ Next: Docker Compose](./05-docker-compose.md)

</div>
