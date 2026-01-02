# Module 1.7: Caching Basics - أساسيات التخزين المؤقت ⚡

<div dir="rtl">

## نظرة عامة

**Caching** من أهم التقنيات لتحسين أداء التطبيقات! في هذا الـ Module هنتعلم إزاي نستخدم الـ Cache بشكل صحيح.

**المدة المتوقعة:** 3-4 ساعات

</div>

---

## 🎯 Learning Objectives

<div dir="rtl">

بعد إكمال هذا الـ Module، ستتمكن من:

</div>

- ✅ Understand what caching is and why it matters
- ✅ Know different caching strategies
- ✅ Implement caching with Redis
- ✅ Handle cache invalidation properly
- ✅ Design effective caching layers
- ✅ Understand CDN basics

---

## 📊 Why Caching?

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Performance Impact                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Without Cache:                                                      │
│  ┌────────┐    100ms    ┌────────┐    200ms    ┌────────┐          │
│  │ Client │────────────▶│ Server │────────────▶│   DB   │          │
│  └────────┘             └────────┘             └────────┘          │
│                                                                      │
│  Total: ~300ms per request                                          │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  With Cache:                                                         │
│  ┌────────┐    100ms    ┌────────┐    1ms     ┌────────┐           │
│  │ Client │────────────▶│ Server │───────────▶│ Cache  │           │
│  └────────┘             └────────┘             └────────┘           │
│                                                 (Redis)             │
│  Total: ~101ms per request (3x faster!)                             │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  With CDN:                                                           │
│  ┌────────┐    20ms     ┌────────┐                                  │
│  │ Client │────────────▶│  CDN   │  (No server hit!)                │
│  └────────┘             └────────┘                                  │
│                                                                      │
│  Total: ~20ms (15x faster!)                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Module Contents

### Lesson 1: What is Caching?
<div dir="rtl">

- تعريف الـ Caching
- أنواع الـ Cache
- متى نستخدم Cache؟
- Cache Hit vs Cache Miss

</div>

**➡️ [Start Lesson 1](./lessons/01-what-is-caching.md)**

---

### Lesson 2: Caching Strategies
<div dir="rtl">

- Cache-Aside Pattern
- Read-Through / Write-Through
- Write-Behind (Write-Back)
- Refresh-Ahead

</div>

**➡️ [Start Lesson 2](./lessons/02-caching-strategies.md)**

---

### Lesson 3: Redis Fundamentals
<div dir="rtl">

- ما هو Redis؟
- Data Types في Redis
- أوامر Redis الأساسية
- التثبيت والاتصال

</div>

**➡️ [Start Lesson 3](./lessons/03-redis-fundamentals.md)**

---

### Lesson 4: Redis in Go
<div dir="rtl">

- استخدام go-redis
- CRUD operations
- Expiration و TTL
- Pipelining

</div>

**➡️ [Start Lesson 4](./lessons/04-redis-in-go.md)**

---

### Lesson 5: Cache Invalidation
<div dir="rtl">

- لماذا Invalidation صعب؟
- TTL-Based Invalidation
- Event-Based Invalidation
- Cache Stampede Prevention

</div>

**➡️ [Start Lesson 5](./lessons/05-cache-invalidation.md)**

---

### Lesson 6: CDN Basics
<div dir="rtl">

- ما هو CDN؟
- كيف يعمل؟
- متى نستخدمه؟
- Popular CDN Providers

</div>

**➡️ [Start Lesson 6](./lessons/06-cdn-basics.md)**

---

## 🛠️ Prerequisites

<div dir="rtl">

قبل البدء، تأكد من:

</div>

- ✅ Completed Module 1.3 (REST API)
- ✅ Completed Module 1.5 (Database Concepts)
- ✅ Basic Go knowledge
- ✅ Redis installed (or Docker)

---

## 📊 Caching Layers

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Caching Architecture                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐                                                         │
│  │ Browser │──┬── Browser Cache (localStorage, Memory)              │
│  └─────────┘  │                                                      │
│               ▼                                                      │
│  ┌─────────┐                                                         │
│  │   CDN   │──┬── Edge Cache (Static assets, API responses)         │
│  └─────────┘  │                                                      │
│               ▼                                                      │
│  ┌─────────┐                                                         │
│  │  Nginx  │──┬── Reverse Proxy Cache                               │
│  └─────────┘  │                                                      │
│               ▼                                                      │
│  ┌─────────┐                                                         │
│  │   App   │──┬── Application Cache (Redis, Memcached)              │
│  └─────────┘  │                                                      │
│               ▼                                                      │
│  ┌─────────┐                                                         │
│  │   DB    │──┬── Query Cache, Buffer Pool                          │
│  └─────────┘                                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Quick Reference

### Common Cache Patterns

| Pattern | Use Case | Pros | Cons |
|---------|----------|------|------|
| Cache-Aside | General purpose | Simple, flexible | Manual management |
| Read-Through | Read-heavy | Automatic loading | Library support needed |
| Write-Through | Data consistency | Always in sync | Write latency |
| Write-Behind | Write-heavy | Fast writes | Data loss risk |

### Redis Commands Cheat Sheet

```bash
# String
SET key value EX 3600     # Set with 1 hour expiry
GET key                    # Get value
DEL key                    # Delete

# Hash
HSET user:1 name "Ahmed"   # Set field
HGET user:1 name           # Get field
HGETALL user:1             # Get all fields

# List
LPUSH queue item           # Push to left
RPOP queue                 # Pop from right

# Set
SADD tags "go" "redis"     # Add to set
SMEMBERS tags              # Get all members

# Sorted Set
ZADD leaderboard 100 "p1"  # Add with score
ZRANGE leaderboard 0 9     # Top 10
```

---

## ⏭️ Next Module

<div dir="rtl">

بعد إكمال هذا الـ Module، انتقل إلى:

**➡️ [Module 1.8: Docker Basics](../08-docker-basics/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: API Security](../06-api-security/README.md) | [🏠 Track 1 Home](../README.md) | [➡️ Next: Docker Basics](../08-docker-basics/README.md)

</div>
