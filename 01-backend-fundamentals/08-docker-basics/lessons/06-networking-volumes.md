# Lesson 6: Docker Networking & Volumes - الشبكات والـ Volumes 🔌

<div dir="rtl">

## المقدمة

في هذا الدرس هنفهم إزاي الـ containers بتتواصل مع بعض وإزاي نحفظ الـ data.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Docker Networking Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Docker Network Types                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. bridge (Default)                                                │
│     └─ Containers on same host can communicate                     │
│     └─ Isolated from host network                                  │
│                                                                      │
│  2. host                                                            │
│     └─ Container shares host's network                             │
│     └─ No port mapping needed                                      │
│     └─ Less isolation                                              │
│                                                                      │
│  3. none                                                            │
│     └─ No network access                                           │
│     └─ Complete isolation                                          │
│                                                                      │
│  4. overlay                                                         │
│     └─ Multi-host networking (Swarm)                               │
│     └─ For distributed systems                                     │
│                                                                      │
│  5. macvlan                                                         │
│     └─ Container gets its own MAC address                          │
│     └─ Appears as physical device on network                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Bridge Network (Default)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Bridge Network                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Host Machine                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                               │    │
│  │  ┌───────────────── docker0 bridge ─────────────────────┐    │    │
│  │  │                  (172.17.0.1)                         │    │    │
│  │  │                                                       │    │    │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │    │    │
│  │  │  │ Container 1 │  │ Container 2 │  │ Container 3 │   │    │    │
│  │  │  │ 172.17.0.2  │  │ 172.17.0.3  │  │ 172.17.0.4  │   │    │    │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘   │    │    │
│  │  │                                                       │    │    │
│  │  └───────────────────────────────────────────────────────┘    │    │
│  │                                                               │    │
│  │  Containers can talk to each other via IP                    │    │
│  │                                                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Default Bridge vs Custom Bridge

```bash
# Default bridge (docker0)
docker run -d --name app1 nginx
docker run -d --name app2 nginx

# Containers can't reach each other by name!
docker exec app1 ping app2  # ❌ Fails

# Custom bridge network
docker network create mynet
docker run -d --name app1 --network mynet nginx
docker run -d --name app2 --network mynet nginx

# Now they can!
docker exec app1 ping app2  # ✅ Works
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                Default Bridge vs Custom Bridge                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Default Bridge (docker0):                                          │
│  ─────────────────────────                                           │
│  • Automatic for all containers                                     │
│  • NO DNS resolution between containers                             │
│  • Must use IP addresses                                            │
│  • All containers exposed to each other                             │
│                                                                      │
│  Custom Bridge:                                                     │
│  ──────────────                                                      │
│  • Created explicitly                                               │
│  • DNS resolution by container name ✅                              │
│  • Better isolation (only joined containers)                        │
│  • Can connect/disconnect live                                      │
│                                                                      │
│  Always use custom bridge networks!                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ Network Commands

```bash
# List networks
docker network ls
# NETWORK ID     NAME      DRIVER    SCOPE
# abc123         bridge    bridge    local
# def456         host      host      local
# ghi789         none      null      local

# Create network
docker network create myapp-network
docker network create --driver bridge myapp-network

# Inspect network
docker network inspect myapp-network

# Connect container to network
docker network connect myapp-network mycontainer

# Disconnect from network
docker network disconnect myapp-network mycontainer

# Remove network
docker network rm myapp-network

# Remove unused networks
docker network prune
```

### Run with Network

```bash
# Run container on specific network
docker run -d --name db \
  --network myapp-network \
  postgres:15

docker run -d --name app \
  --network myapp-network \
  -e DB_HOST=db \
  myapp:latest

# App can now connect to: db:5432
```

---

## 3️⃣ Host Network

```bash
# Use host network (Linux only)
docker run -d --network host nginx

# Container uses host's network directly
# No port mapping needed (-p flag ignored)
# Port 80 on container = Port 80 on host
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Host Network                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Use when:                                                          │
│  ──────────                                                          │
│  • Need maximum network performance                                 │
│  • Container must bind to host ports directly                       │
│  • Need access to host's network interfaces                         │
│                                                                      │
│  Avoid when:                                                        │
│  ────────────                                                        │
│  • Need port mapping                                                │
│  • Need network isolation                                           │
│  • Running multiple instances on same port                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ Docker Volumes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Why Volumes?                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Problem: Containers are ephemeral                                  │
│  ────────────────────────────────────                                │
│                                                                      │
│  docker run postgres...                                             │
│  # Create database, add data                                        │
│  docker rm postgres                                                 │
│  # All data is GONE! 💥                                             │
│                                                                      │
│  Solution: Volumes                                                  │
│  ─────────────────────                                               │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Volume (on host)                                             │    │
│  │  /var/lib/docker/volumes/pgdata/_data                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              │ Mounted                              │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Container                                                    │    │
│  │  /var/lib/postgresql/data                                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Container removed? Data persists in volume! ✅                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Volume Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Volume Types                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Named Volumes (Recommended)                                     │
│     ──────────────────────────────                                   │
│     docker volume create mydata                                     │
│     docker run -v mydata:/app/data myapp                            │
│                                                                      │
│     • Managed by Docker                                             │
│     • Portable across hosts                                         │
│     • Best for production                                           │
│                                                                      │
│  2. Bind Mounts                                                     │
│     ─────────────                                                    │
│     docker run -v /host/path:/container/path myapp                  │
│     docker run -v $(pwd):/app myapp                                 │
│                                                                      │
│     • Direct host directory mapping                                 │
│     • Good for development (live code reload)                       │
│     • Host-dependent                                                │
│                                                                      │
│  3. tmpfs Mounts                                                    │
│     ─────────────                                                    │
│     docker run --tmpfs /app/tmp myapp                               │
│                                                                      │
│     • Stored in memory only                                         │
│     • Lost when container stops                                     │
│     • Good for sensitive data, caches                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ Volume Commands

```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls
# DRIVER    VOLUME NAME
# local     mydata
# local     pgdata

# Inspect volume
docker volume inspect mydata
# [
#   {
#     "Name": "mydata",
#     "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
#     ...
#   }
# ]

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune

# Remove ALL volumes (dangerous!)
docker volume prune -a
```

### Using Volumes

```bash
# Named volume
docker run -d \
  --name db \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# Bind mount (development)
docker run -d \
  --name app \
  -v $(pwd):/app \
  -v /app/node_modules \     # Exclude node_modules
  myapp:dev

# Read-only mount
docker run -d \
  -v ./config:/app/config:ro \
  myapp

# tmpfs mount
docker run -d \
  --tmpfs /app/tmp:size=100m \
  myapp
```

---

## 6️⃣ Docker Compose Networking & Volumes

```yaml
version: "3.8"

services:
  app:
    build: .
    networks:
      - frontend
      - backend
    volumes:
      - app-data:/app/data
      - ./logs:/app/logs     # Bind mount
    depends_on:
      - db

  nginx:
    image: nginx:alpine
    networks:
      - frontend            # Only frontend
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

  db:
    image: postgres:15
    networks:
      - backend             # Only backend
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7
    networks:
      - backend
    volumes:
      - redis-data:/data

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true          # No external access

volumes:
  app-data:
  postgres-data:
  redis-data:
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Network Isolation                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────── frontend network ─────────────────────┐     │
│  │                                                             │     │
│  │    ┌─────────┐              ┌─────────┐                    │     │
│  │    │  nginx  │◄────────────▶│   app   │                    │     │
│  │    └─────────┘              └─────────┘                    │     │
│  │         ▲                        │                          │     │
│  └─────────│────────────────────────│──────────────────────────┘     │
│            │                        │                                │
│      Internet                       │                                │
│                                     ▼                                │
│  ┌───────────────────── backend network ──────────────────────┐     │
│  │                      (internal: true)                       │     │
│  │                                                             │     │
│  │    ┌─────────┐              ┌─────────┐    ┌─────────┐     │     │
│  │    │   app   │◄────────────▶│   db    │    │  redis  │     │     │
│  │    └─────────┘              └─────────┘    └─────────┘     │     │
│  │                                                             │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  ❌ nginx can't reach db directly                                   │
│  ❌ Internet can't reach backend (internal network)                 │
│  ✅ app is in both networks (bridge between them)                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Backup & Restore Volumes

### Backup

```bash
# Backup volume to tar file
docker run --rm \
  -v mydata:/source:ro \
  -v $(pwd):/backup \
  alpine tar cvf /backup/mydata-backup.tar /source

# Backup database
docker exec db pg_dump -U postgres mydb > backup.sql
```

### Restore

```bash
# Restore from tar
docker run --rm \
  -v mydata:/target \
  -v $(pwd):/backup \
  alpine tar xvf /backup/mydata-backup.tar -C /target --strip 1

# Restore database
docker exec -i db psql -U postgres mydb < backup.sql
```

---

## 8️⃣ Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Networking Best Practices                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Always use custom bridge networks                               │
│  ✅ Use network aliases for service discovery                       │
│  ✅ Isolate frontend and backend networks                           │
│  ✅ Use internal networks for sensitive services                    │
│  ❌ Don't use default bridge (docker0)                              │
│  ❌ Don't expose database ports to host                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    Volume Best Practices                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Use named volumes for production data                           │
│  ✅ Use bind mounts for development (live reload)                   │
│  ✅ Use :ro for read-only mounts                                    │
│  ✅ Backup volumes regularly                                        │
│  ✅ Use volume drivers for cloud storage                            │
│  ❌ Don't store sensitive data without encryption                   │
│  ❌ Don't use anonymous volumes in production                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Custom bridge** = أفضل من default bridge (DNS resolution)
- ✅ **Named volumes** = للـ production data
- ✅ **Bind mounts** = للـ development (live reload)
- ✅ **Internal networks** = للـ services الحساسة
- ✅ **:ro** = للـ read-only mounts
- ✅ Volume data persists حتى بعد حذف الـ container

</div>

---

## 🎉 Module Complete!

<div dir="rtl">

مبروك! أنت خلصت **Module 1.8: Docker Basics** 🎉

راجعنا:
- ما هو Docker وكيف يختلف عن VMs
- Docker Commands الأساسية
- كتابة Dockerfile
- Docker لتطبيقات Go
- Docker Compose
- Networking & Volumes

**➡️ [Next Module: Architecture Patterns](../../09-architecture-patterns/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Docker Compose](./05-docker-compose.md) | [📚 Module Home](../README.md) | [🏠 Track 1](../../README.md)

</div>
