# Lesson 5: Docker Compose - تشغيل عدة Services معاً 🎼

<div dir="rtl">

## المقدمة

**Docker Compose** أداة لتعريف وتشغيل multi-container Docker applications. بدلاً من تشغيل كل container يدوياً، ملف واحد يعرف كل الـ services.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 Why Docker Compose?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Without Docker Compose                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  # Create network                                                   │
│  docker network create myapp-network                                │
│                                                                      │
│  # Run database                                                     │
│  docker run -d --name db \                                          │
│    --network myapp-network \                                        │
│    -e POSTGRES_PASSWORD=secret \                                    │
│    -v pgdata:/var/lib/postgresql/data \                             │
│    postgres:15                                                      │
│                                                                      │
│  # Run Redis                                                        │
│  docker run -d --name redis \                                       │
│    --network myapp-network \                                        │
│    redis:7                                                          │
│                                                                      │
│  # Run app                                                          │
│  docker run -d --name app \                                         │
│    --network myapp-network \                                        │
│    -p 8080:8080 \                                                   │
│    -e DB_HOST=db \                                                  │
│    myapp:latest                                                     │
│                                                                      │
│  😩 الكثير من الـ commands!                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    With Docker Compose                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  docker-compose up -d                                               │
│                                                                      │
│  🎉 Done!                                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ docker-compose.yml Basics

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  postgres-data:
```

### Key Concepts

```yaml
version: "3.8"      # Compose file version

services:           # Containers to run
  service-name:     # Container name
    image:          # Use existing image
    build:          # Or build from Dockerfile
    ports:          # Port mapping
    environment:    # Environment variables
    volumes:        # Data persistence
    depends_on:     # Start order
    networks:       # Custom networks
    restart:        # Restart policy

volumes:            # Named volumes
networks:           # Custom networks
```

---

## 2️⃣ Service Configuration

### Build Options

```yaml
services:
  app:
    # Simple build
    build: .

    # Or with options
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        VERSION: 1.0.0
      target: production
```

### Ports

```yaml
services:
  app:
    ports:
      - "8080:8080"           # host:container
      - "8443:443"            # Multiple ports
      - "127.0.0.1:9000:9000" # Bind to localhost only
      - "3000"                # Random host port
```

### Environment Variables

```yaml
services:
  app:
    # List format
    environment:
      - DB_HOST=localhost
      - DB_PORT=5432
      - DEBUG=true

    # Map format
    environment:
      DB_HOST: localhost
      DB_PORT: 5432
      DEBUG: "true"

    # From file
    env_file:
      - .env
      - .env.local
```

### Volumes

```yaml
services:
  db:
    volumes:
      # Named volume
      - postgres-data:/var/lib/postgresql/data

      # Bind mount (host path)
      - ./data:/data

      # Read-only
      - ./config:/app/config:ro

volumes:
  postgres-data:     # Declare named volume
```

### Dependencies

```yaml
services:
  app:
    depends_on:
      - db
      - redis
    # Simple: just start order

  app:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    # With conditions: wait for health
```

### Health Checks

```yaml
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  app:
    depends_on:
      db:
        condition: service_healthy
```

### Restart Policies

```yaml
services:
  app:
    restart: "no"              # Never restart
    restart: always            # Always restart
    restart: on-failure        # Only on failure
    restart: unless-stopped    # Unless manually stopped
```

---

## 3️⃣ Complete Example

```yaml
# docker-compose.yml
version: "3.8"

services:
  # ==================== Application ====================
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - APP_ENV=production
      - DB_HOST=db
      - DB_PORT=5432
      - DB_USER=myuser
      - DB_PASSWORD=mypassword
      - DB_NAME=mydb
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ==================== Database ====================
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ==================== Redis ====================
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"

  # ==================== Nginx (Optional) ====================
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      - app

volumes:
  postgres-data:
  redis-data:

networks:
  default:
    name: myapp-network
```

---

## 4️⃣ Docker Compose Commands

```bash
# Start services
docker-compose up                 # Foreground
docker-compose up -d              # Background (detached)
docker-compose up --build         # Rebuild images first

# Stop services
docker-compose stop               # Stop (keep containers)
docker-compose down               # Stop and remove containers
docker-compose down -v            # Also remove volumes
docker-compose down --rmi all     # Also remove images

# View status
docker-compose ps                 # List containers
docker-compose logs               # View logs
docker-compose logs -f            # Follow logs
docker-compose logs app           # Specific service

# Execute commands
docker-compose exec app sh        # Shell into container
docker-compose exec db psql -U myuser mydb

# Scale services
docker-compose up -d --scale app=3

# Restart service
docker-compose restart app

# Pull latest images
docker-compose pull

# Build images
docker-compose build
docker-compose build --no-cache
```

---

## 5️⃣ Development vs Production

### Development Compose

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
      - .:/app                    # Live code reload
      - /app/tmp                  # Exclude tmp
    environment:
      - APP_ENV=development
      - DEBUG=true
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres-dev:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # Development tools
  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - db

volumes:
  postgres-dev:
```

### Production Compose

```yaml
# docker-compose.prod.yml
version: "3.8"

services:
  app:
    image: myregistry/myapp:${VERSION:-latest}
    ports:
      - "8080:8080"
    environment:
      - APP_ENV=production
    env_file:
      - .env.prod
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
    restart: always
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15-alpine
    env_file:
      - .env.prod
    volumes:
      - postgres-prod:/var/lib/postgresql/data
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres-prod:
```

### Using Multiple Files

```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Or use COMPOSE_FILE env
export COMPOSE_FILE=docker-compose.yml:docker-compose.dev.yml
docker-compose up
```

---

## 6️⃣ Networking

```yaml
version: "3.8"

services:
  app:
    networks:
      - frontend
      - backend

  nginx:
    networks:
      - frontend     # Can reach app

  db:
    networks:
      - backend      # Only app can reach

networks:
  frontend:
  backend:
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Docker Compose Networking                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  By default:                                                        │
│  ────────────                                                        │
│  • All services in same compose file share a network                │
│  • Services can reach each other by service name                    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    default network                            │    │
│  │                                                               │    │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐                  │    │
│  │  │   app   │◄──▶│   db    │◄──▶│  redis  │                  │    │
│  │  └─────────┘    └─────────┘    └─────────┘                  │    │
│  │                                                               │    │
│  │  app can call: http://db:5432                                │    │
│  │  app can call: http://redis:6379                             │    │
│  │                                                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Environment File (.env)

```bash
# .env file
POSTGRES_USER=myuser
POSTGRES_PASSWORD=supersecret
POSTGRES_DB=mydb
APP_VERSION=1.0.0
```

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

  app:
    image: myapp:${APP_VERSION}
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **docker-compose.yml** = تعريف كل الـ services في ملف واحد
- ✅ **docker-compose up -d** = تشغيل كل الـ services
- ✅ **depends_on** = ترتيب بدء التشغيل
- ✅ **volumes** = للـ data persistence
- ✅ **networks** = للتواصل بين containers
- ✅ استخدم ملفات مختلفة للـ dev و prod

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم Networking و Volumes بالتفصيل:

**➡️ [Lesson 6: Networking & Volumes](./06-networking-volumes.md)**

</div>

---

<div align="center">

[⬅️ Previous: Docker for Go](./04-docker-for-go.md) | [📚 Module Home](../README.md) | [➡️ Next: Networking & Volumes](./06-networking-volumes.md)

</div>
