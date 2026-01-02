# Lesson 3: Redis Fundamentals - أساسيات Redis 🔴

<div dir="rtl">

## المقدمة

**Redis** (Remote Dictionary Server) هو أشهر in-memory data store في العالم! سريع جداً وبيدعم data structures متقدمة.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 What is Redis?

```
┌─────────────────────────────────────────────────────────────────────┐
│                        What is Redis?                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Redis = Remote Dictionary Server                                   │
│                                                                      │
│  Key Features:                                                      │
│  ─────────────                                                       │
│  • In-Memory: البيانات في RAM (سريع جداً!)                          │
│  • Data Structures: مش بس key-value                                 │
│  • Persistence: يقدر يحفظ على Disk                                  │
│  • Pub/Sub: للـ Real-time messaging                                 │
│  • Atomic Operations: Thread-safe                                   │
│  • Replication: Master-Slave support                                │
│  • Clustering: للـ High Availability                                │
│                                                                      │
│  Performance:                                                       │
│  ────────────                                                        │
│  • 100,000+ operations/second (single thread!)                      │
│  • < 1ms latency                                                    │
│  • Compare: PostgreSQL ~10ms per query                              │
│                                                                      │
│  Use Cases:                                                         │
│  ──────────                                                          │
│  • Caching (الأكثر شيوعاً)                                          │
│  • Session storage                                                  │
│  • Rate limiting                                                    │
│  • Leaderboards                                                     │
│  • Real-time analytics                                              │
│  • Message queues                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Installation

### Using Docker (Recommended)

```bash
# Run Redis
docker run -d --name redis -p 6379:6379 redis:alpine

# With persistence
docker run -d --name redis \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:alpine redis-server --appendonly yes

# Connect with CLI
docker exec -it redis redis-cli
```

### Direct Installation

```bash
# macOS
brew install redis
brew services start redis

# Ubuntu
sudo apt update
sudo apt install redis-server
sudo systemctl start redis

# Windows (WSL recommended)
# Or download from: https://github.com/microsoftarchive/redis/releases
```

### Verify Installation

```bash
# Connect to Redis
redis-cli

# Test connection
127.0.0.1:6379> PING
PONG

# Simple test
127.0.0.1:6379> SET hello "world"
OK
127.0.0.1:6379> GET hello
"world"
```

---

## 2️⃣ Redis Data Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Redis Data Types                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Strings                                                         │
│     └─ Simple key-value (text, numbers, JSON)                       │
│     └─ Max size: 512MB                                              │
│                                                                      │
│  2. Hashes                                                          │
│     └─ Like objects/maps (field-value pairs)                        │
│     └─ Perfect for: User profiles, settings                         │
│                                                                      │
│  3. Lists                                                           │
│     └─ Ordered collection (linked list)                             │
│     └─ Perfect for: Queues, recent items                            │
│                                                                      │
│  4. Sets                                                            │
│     └─ Unordered unique values                                      │
│     └─ Perfect for: Tags, followers, unique visitors                │
│                                                                      │
│  5. Sorted Sets (ZSets)                                             │
│     └─ Sets with scores                                             │
│     └─ Perfect for: Leaderboards, rankings                          │
│                                                                      │
│  6. Streams                                                         │
│     └─ Append-only log                                              │
│     └─ Perfect for: Event sourcing, message queues                  │
│                                                                      │
│  7. HyperLogLog                                                     │
│     └─ Probabilistic unique count                                   │
│     └─ Perfect for: Counting unique visitors                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ Strings

<div dir="rtl">

أبسط نوع - مجرد key وقيمة.

</div>

```bash
# Basic operations
SET name "Ahmed"                    # Set value
GET name                            # Get value → "Ahmed"
DEL name                            # Delete
EXISTS name                         # Check exists → 0 or 1

# With expiration (TTL)
SET session:123 "user_data" EX 3600   # Expires in 1 hour
SETEX session:123 3600 "user_data"    # Same as above
TTL session:123                        # Check remaining time

# Conditional set
SETNX lock:resource "owner"    # Set if Not eXists
SET key value XX               # Only if exists
SET key value NX               # Only if NOT exists

# Increment/Decrement (atomic!)
SET counter 10
INCR counter                   # → 11
INCRBY counter 5               # → 16
DECR counter                   # → 15
DECRBY counter 3               # → 12

# Use case: Rate limiting
SET rate:user:123 1 EX 60      # First request
INCR rate:user:123             # Increment on each request
GET rate:user:123              # Check count
```

### String Patterns

```bash
# Store JSON
SET user:1 '{"name":"Ahmed","email":"ahmed@test.com"}'
GET user:1

# Store number as string
SET views:article:42 "1000"
INCR views:article:42

# Multiple get/set
MSET key1 "val1" key2 "val2" key3 "val3"
MGET key1 key2 key3
```

---

## 4️⃣ Hashes

<div dir="rtl">

مثل Objects في JavaScript أو structs في Go.

</div>

```bash
# Set fields
HSET user:1 name "Ahmed"
HSET user:1 email "ahmed@test.com"
HSET user:1 age 25

# Or set multiple at once
HSET user:1 name "Ahmed" email "ahmed@test.com" age 25

# Get field
HGET user:1 name                    # → "Ahmed"
HGET user:1 email                   # → "ahmed@test.com"

# Get all
HGETALL user:1
# 1) "name"
# 2) "Ahmed"
# 3) "email"
# 4) "ahmed@test.com"
# 5) "age"
# 6) "25"

# Get multiple fields
HMGET user:1 name email

# Check field exists
HEXISTS user:1 phone               # → 0

# Delete field
HDEL user:1 age

# Get all keys or values
HKEYS user:1                       # → ["name", "email"]
HVALS user:1                       # → ["Ahmed", "ahmed@test.com"]

# Increment numeric field
HINCRBY user:1 login_count 1
```

### Hash vs String (JSON)

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Hash vs String for Objects                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  String (JSON):                                                     │
│  ──────────────                                                      │
│  SET user:1 '{"name":"Ahmed","age":25,"email":"..."}'               │
│                                                                      │
│  ✅ Single operation to get all                                     │
│  ❌ Must read/write entire object                                   │
│  ❌ Need to parse JSON                                              │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  Hash:                                                              │
│  ─────                                                               │
│  HSET user:1 name "Ahmed" age 25 email "..."                        │
│                                                                      │
│  ✅ Update single field without reading entire object               │
│  ✅ Memory efficient for small hashes                               │
│  ✅ Atomic increment on numeric fields                              │
│  ❌ Can't set expiration on individual fields                       │
│                                                                      │
│  Use Hash when: Need to update individual fields often              │
│  Use String when: Always read/write entire object                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ Lists

<div dir="rtl">

Linked list - ممتاز للـ queues وآخر الإضافات.

</div>

```bash
# Push to list
LPUSH queue "task1"          # Push to left (head)
LPUSH queue "task2"          # queue = [task2, task1]
RPUSH queue "task3"          # Push to right (tail)
                             # queue = [task2, task1, task3]

# Pop from list
LPOP queue                   # → "task2" (removes from left)
RPOP queue                   # → "task3" (removes from right)

# Range (get without removing)
LRANGE queue 0 -1            # Get all
LRANGE queue 0 2             # Get first 3 elements

# Length
LLEN queue

# Block pop (wait for item)
BLPOP queue 30               # Wait up to 30 seconds for item
BRPOP queue 30

# Trim (keep only specified range)
LTRIM recent_searches 0 9    # Keep only last 10 items
```

### List Use Cases

```bash
# Recent Activity Feed
LPUSH activity:user:1 "liked post 42"
LPUSH activity:user:1 "commented on post 39"
LTRIM activity:user:1 0 49        # Keep last 50 activities
LRANGE activity:user:1 0 9        # Get last 10

# Message Queue (Simple)
RPUSH jobs '{"type":"email","to":"user@test.com"}'
BLPOP jobs 0                       # Worker waits for job

# Latest Articles
LPUSH latest_articles "article:100"
LPUSH latest_articles "article:101"
LTRIM latest_articles 0 9          # Keep latest 10
```

---

## 6️⃣ Sets

<div dir="rtl">

مجموعة من القيم الفريدة - بدون ترتيب.

</div>

```bash
# Add members
SADD tags:article:1 "go" "redis" "tutorial"
SADD tags:article:1 "go"         # Ignored (already exists)

# Get all members
SMEMBERS tags:article:1          # → ["go", "redis", "tutorial"]

# Check membership
SISMEMBER tags:article:1 "go"    # → 1 (true)
SISMEMBER tags:article:1 "java"  # → 0 (false)

# Count
SCARD tags:article:1             # → 3

# Remove
SREM tags:article:1 "tutorial"

# Set operations
SADD user:1:followers "user:2" "user:3" "user:4"
SADD user:2:followers "user:1" "user:3" "user:5"

# Intersection (mutual followers)
SINTER user:1:followers user:2:followers  # → ["user:3"]

# Union (all followers)
SUNION user:1:followers user:2:followers

# Difference
SDIFF user:1:followers user:2:followers   # In 1 but not in 2

# Random member
SRANDMEMBER tags:article:1 2     # Get 2 random tags
SPOP tags:article:1              # Remove and return random
```

### Set Use Cases

```bash
# Track Online Users
SADD online_users "user:1" "user:2" "user:3"
SREM online_users "user:1"           # User went offline
SMEMBERS online_users                # Who's online?
SCARD online_users                   # How many online?

# Mutual Friends
SADD friends:alice "bob" "charlie" "dave"
SADD friends:bob "alice" "charlie" "eve"
SINTER friends:alice friends:bob     # → ["charlie"]

# Tags System
SADD post:1:tags "go" "redis"
SADD tag:go:posts "1" "5" "10"
SMEMBERS tag:go:posts                # All posts with "go" tag
```

---

## 7️⃣ Sorted Sets (ZSets)

<div dir="rtl">

مثل Sets لكن كل عنصر له score - ممتاز للترتيب!

</div>

```bash
# Add with scores
ZADD leaderboard 100 "player:1"
ZADD leaderboard 250 "player:2"
ZADD leaderboard 150 "player:3"

# Get by rank (sorted by score, low to high)
ZRANGE leaderboard 0 -1              # All, lowest first
ZRANGE leaderboard 0 2 WITHSCORES    # Top 3 with scores

# Get by rank (high to low)
ZREVRANGE leaderboard 0 2            # Top 3, highest first

# Get score
ZSCORE leaderboard "player:2"        # → 250

# Get rank
ZRANK leaderboard "player:1"         # → 0 (lowest score)
ZREVRANK leaderboard "player:2"      # → 0 (highest score)

# Increment score
ZINCRBY leaderboard 50 "player:1"    # player:1 now 150

# Get by score range
ZRANGEBYSCORE leaderboard 100 200    # Scores between 100-200

# Count in range
ZCOUNT leaderboard 100 200           # → 2

# Remove
ZREM leaderboard "player:3"
```

### Sorted Set Use Cases

```bash
# Leaderboard
ZADD game:leaderboard 1500 "alice"
ZADD game:leaderboard 2000 "bob"
ZADD game:leaderboard 1800 "charlie"
ZREVRANGE game:leaderboard 0 9 WITHSCORES  # Top 10

# Time-based Expiration (Scheduled tasks)
# Score = Unix timestamp
ZADD scheduled_jobs 1704067200 "job:1"    # Execute at this time
ZADD scheduled_jobs 1704070800 "job:2"
# Get jobs due now:
ZRANGEBYSCORE scheduled_jobs 0 <current_timestamp>

# Priority Queue
ZADD priority_queue 1 "low_priority_task"
ZADD priority_queue 10 "high_priority_task"
ZPOPMAX priority_queue     # Get highest priority

# Trending Items (decay over time)
ZINCRBY trending:articles 1 "article:42"
ZINCRBY trending:articles 1 "article:42"
# Periodically: ZINTERSTORE to multiply scores by 0.9
```

---

## 8️⃣ Key Management

```bash
# Find keys
KEYS user:*                  # Find all user keys (⚠️ SLOW!)
SCAN 0 MATCH user:* COUNT 100  # Better: iterate in batches

# Key info
TYPE user:1                  # → "hash"
EXISTS user:1                # → 1
TTL user:1                   # → -1 (no expiration)

# Set expiration
EXPIRE user:1 3600           # Expire in 1 hour
EXPIREAT user:1 1704067200   # Expire at timestamp
PEXPIRE user:1 60000         # Expire in 60 seconds (ms)
PERSIST user:1               # Remove expiration

# Rename
RENAME old_key new_key
RENAMENX old_key new_key     # Only if new_key doesn't exist

# Delete
DEL key1 key2 key3           # Synchronous delete
UNLINK key1 key2 key3        # Async delete (faster for big keys)

# Dump/Restore (serialize)
DUMP user:1                  # → serialized data
RESTORE user:1 0 "<data>"    # Restore with 0 TTL
```

---

## 9️⃣ Transactions

```bash
# MULTI/EXEC - atomic transaction
MULTI
SET balance:1 100
SET balance:2 200
DECRBY balance:1 50
INCRBY balance:2 50
EXEC

# Watch (optimistic locking)
WATCH balance:1
GET balance:1                # → 100
MULTI
DECRBY balance:1 50
EXEC
# If balance:1 changed between WATCH and EXEC, EXEC returns nil
```

---

## 🔟 Pub/Sub

```bash
# Terminal 1: Subscribe
SUBSCRIBE news

# Terminal 2: Publish
PUBLISH news "Breaking: Redis is awesome!"

# Terminal 1 receives:
# 1) "message"
# 2) "news"
# 3) "Breaking: Redis is awesome!"

# Pattern subscribe
PSUBSCRIBE news:*
PUBLISH news:sports "Goal!"
PUBLISH news:tech "New release!"
```

---

## ✅ Quick Reference

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Redis Commands Cheat Sheet                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Strings:                                                           │
│  SET key val [EX sec] [NX|XX]  │  GET key  │  INCR/DECR key        │
│                                                                      │
│  Hashes:                                                            │
│  HSET key field val  │  HGET key field  │  HGETALL key             │
│                                                                      │
│  Lists:                                                             │
│  LPUSH/RPUSH key val  │  LPOP/RPOP key  │  LRANGE key 0 -1         │
│                                                                      │
│  Sets:                                                              │
│  SADD key val  │  SMEMBERS key  │  SINTER key1 key2                │
│                                                                      │
│  Sorted Sets:                                                       │
│  ZADD key score val  │  ZRANGE key 0 -1  │  ZREVRANGE key 0 9      │
│                                                                      │
│  Keys:                                                              │
│  KEYS pattern  │  EXISTS key  │  DEL key  │  EXPIRE key sec        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد ما فهمت Redis، خلينا نستخدمه في Go:

**➡️ [Lesson 4: Redis in Go](./04-redis-in-go.md)**

</div>

---

<div align="center">

[⬅️ Previous: Caching Strategies](./02-caching-strategies.md) | [📚 Module Home](../README.md) | [➡️ Next: Redis in Go](./04-redis-in-go.md)

</div>
