# Lesson 5: Cache Invalidation - إبطال الـ Cache ⚠️

<div dir="rtl">

## المقدمة

> "There are only two hard things in Computer Science: cache invalidation and naming things."
> — Phil Karlton

**Cache Invalidation** من أصعب المشاكل! في هذا الدرس هنتعلم إزاي نتعامل معاها.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 The Problem

```
┌─────────────────────────────────────────────────────────────────────┐
│                  The Cache Invalidation Problem                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Scenario:                                                          │
│  ──────────                                                          │
│  1. User "Ahmed" cached in Redis                                    │
│  2. Ahmed updates his name to "Ahmed Mohamed"                       │
│  3. What happens?                                                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                               │    │
│  │  Database:  name = "Ahmed Mohamed"  ✅ (Updated)             │    │
│  │  Cache:     name = "Ahmed"          ❌ (Stale!)              │    │
│  │                                                               │    │
│  │  Users see old data! 😱                                      │    │
│  │                                                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Solutions:                                                         │
│  ──────────                                                          │
│  1. TTL (Time-To-Live) - البيانات تنتهي صلاحيتها تلقائياً          │
│  2. Delete on Update - احذف من الـ Cache عند التحديث               │
│  3. Update Cache - حدث الـ Cache مع الـ DB                         │
│  4. Event-Based - استخدم events للتحديث                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ TTL-Based Invalidation

<div dir="rtl">

أبسط طريقة - البيانات تنتهي صلاحيتها بعد وقت محدد.

</div>

```go
func (s *UserService) GetUser(ctx context.Context, id int64) (*User, error) {
    key := fmt.Sprintf("user:%d", id)

    // Try cache
    cached, err := s.cache.Get(ctx, key).Result()
    if err == nil {
        var user User
        json.Unmarshal([]byte(cached), &user)
        return &user, nil
    }

    // Cache miss - get from DB
    user, err := s.db.GetUser(ctx, id)
    if err != nil {
        return nil, err
    }

    // Cache with TTL
    data, _ := json.Marshal(user)
    s.cache.Set(ctx, key, data, 5*time.Minute)  // Expires in 5 mins

    return user, nil
}
```

### TTL Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│                     TTL Strategy Guide                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Data Type              │ Recommended TTL │ Reason                  │
│  ───────────────────────┼─────────────────┼─────────────────────────│
│  User sessions          │ 30 min - 24h    │ Security                │
│  User profiles          │ 5-15 min        │ May change              │
│  Product listings       │ 1-5 min         │ Stock changes           │
│  Static config          │ 1-24 hours      │ Rarely changes          │
│  API rate limits        │ 1 min - 1 hour  │ Reset periodically      │
│  Search results         │ 5-30 min        │ Freshness matters       │
│  News/Feed              │ 1-5 min         │ Time-sensitive          │
│                                                                      │
│  ⚠️ Shorter TTL = More DB hits, fresher data                       │
│  ⚠️ Longer TTL  = Fewer DB hits, staler data                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Pros & Cons

```
✅ Pros:
• Very simple to implement
• No coordination needed
• Automatic cleanup

❌ Cons:
• Data can be stale until TTL expires
• Hard to choose "right" TTL
• Wasted cache if data changes frequently
```

---

## 2️⃣ Delete on Update (Invalidate)

<div dir="rtl">

عند تحديث البيانات، احذف من الـ Cache.

</div>

```go
func (s *UserService) UpdateUser(ctx context.Context, user *User) error {
    // Step 1: Update database
    err := s.db.UpdateUser(ctx, user)
    if err != nil {
        return err
    }

    // Step 2: Invalidate cache (DELETE, don't update!)
    key := fmt.Sprintf("user:%d", user.ID)
    s.cache.Del(ctx, key)

    return nil
}
```

### Why Delete instead of Update?

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Delete vs Update Cache                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Option 1: Update Cache ❌                                          │
│  ─────────────────────────                                           │
│  1. Update DB                                                       │
│  2. Update Cache                                                    │
│                                                                      │
│  Problem - Race Condition:                                          │
│  Request A: Update DB (name = "Ahmed")                              │
│  Request B: Update DB (name = "Mohamed")                            │
│  Request A: Update Cache (name = "Ahmed")  ← Old value!            │
│  Request B: Update Cache (name = "Mohamed")                         │
│                                                                      │
│  DB has "Mohamed", Cache has... random! 😱                          │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  Option 2: Delete Cache ✅                                          │
│  ─────────────────────────                                           │
│  1. Update DB                                                       │
│  2. Delete from Cache                                               │
│                                                                      │
│  Request A: Update DB → Delete Cache                                │
│  Request B: Update DB → Delete Cache                                │
│  Next Read: Cache miss → Get fresh from DB ✅                       │
│                                                                      │
│  Always consistent!                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Complete CRUD with Invalidation

```go
type UserCache struct {
    db    *sql.DB
    cache *redis.Client
}

func (c *UserCache) GetUser(ctx context.Context, id int64) (*User, error) {
    key := fmt.Sprintf("user:%d", id)

    // Check cache
    data, err := c.cache.Get(ctx, key).Result()
    if err == nil {
        var user User
        json.Unmarshal([]byte(data), &user)
        return &user, nil
    }

    // Cache miss
    user, err := c.queryUserFromDB(ctx, id)
    if err != nil {
        return nil, err
    }

    // Store in cache
    userData, _ := json.Marshal(user)
    c.cache.Set(ctx, key, userData, 15*time.Minute)

    return user, nil
}

func (c *UserCache) CreateUser(ctx context.Context, user *User) error {
    err := c.insertUserToDB(ctx, user)
    if err != nil {
        return err
    }
    // No cache to invalidate for new user
    // (optional: warm cache)
    return nil
}

func (c *UserCache) UpdateUser(ctx context.Context, user *User) error {
    err := c.updateUserInDB(ctx, user)
    if err != nil {
        return err
    }

    // Invalidate cache
    key := fmt.Sprintf("user:%d", user.ID)
    c.cache.Del(ctx, key)

    // Also invalidate related caches
    c.cache.Del(ctx, fmt.Sprintf("user:email:%s", user.Email))

    return nil
}

func (c *UserCache) DeleteUser(ctx context.Context, id int64) error {
    // Get user first (for email key)
    user, _ := c.GetUser(ctx, id)

    err := c.deleteUserFromDB(ctx, id)
    if err != nil {
        return err
    }

    // Invalidate all related caches
    c.cache.Del(ctx,
        fmt.Sprintf("user:%d", id),
        fmt.Sprintf("user:email:%s", user.Email),
    )

    return nil
}
```

---

## 3️⃣ Pattern-Based Invalidation

<div dir="rtl">

حذف كل الـ keys اللي تتبع pattern معين.

</div>

```go
// Delete all user-related cache
func (c *UserCache) InvalidateUserCaches(ctx context.Context, userID int64) error {
    patterns := []string{
        fmt.Sprintf("user:%d", userID),
        fmt.Sprintf("user:%d:*", userID),     // user:1:posts, user:1:friends
        fmt.Sprintf("*:user:%d", userID),     // posts:user:1
    }

    for _, pattern := range patterns {
        err := c.deleteByPattern(ctx, pattern)
        if err != nil {
            return err
        }
    }

    return nil
}

func (c *UserCache) deleteByPattern(ctx context.Context, pattern string) error {
    var cursor uint64
    var keysToDelete []string

    for {
        keys, nextCursor, err := c.cache.Scan(ctx, cursor, pattern, 100).Result()
        if err != nil {
            return err
        }

        keysToDelete = append(keysToDelete, keys...)
        cursor = nextCursor

        if cursor == 0 {
            break
        }
    }

    if len(keysToDelete) > 0 {
        return c.cache.Del(ctx, keysToDelete...).Err()
    }

    return nil
}
```

### Cache Key Design for Easy Invalidation

```go
// Good key design - easy to invalidate
// Prefix with entity type and ID

// User keys
"user:{id}"                  // user:123
"user:{id}:profile"          // user:123:profile
"user:{id}:posts"            // user:123:posts
"user:{id}:followers"        // user:123:followers

// Post keys
"post:{id}"                  // post:456
"post:{id}:comments"         // post:456:comments
"user:{id}:posts"            // user:123:posts (list of user's posts)

// Invalidate all user 123 data:
SCAN 0 MATCH "user:123:*"
DEL user:123 user:123:profile user:123:posts user:123:followers
```

---

## 4️⃣ Event-Based Invalidation

<div dir="rtl">

استخدم events (Pub/Sub أو Message Queue) لإعلام الـ services بالتحديثات.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Event-Based Invalidation                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐  Update   ┌─────────┐  Publish  ┌─────────────────┐   │
│  │   App   │──────────▶│   DB    │──────────▶│  Message Queue  │   │
│  └─────────┘           └─────────┘           └─────────────────┘   │
│                                                      │              │
│                                                      │ Event        │
│                                                      ▼              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     Subscribers                               │   │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐      │   │
│  │  │ Cache 1 │   │ Cache 2 │   │ Search  │   │ Analytics│     │   │
│  │  │ Server  │   │ Server  │   │ Index   │   │ Service  │     │   │
│  │  └─────────┘   └─────────┘   └─────────┘   └─────────┘      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  All services invalidate their caches when they receive event       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Using Redis Pub/Sub

```go
// Publisher (on data change)
func (s *UserService) UpdateUser(ctx context.Context, user *User) error {
    err := s.db.UpdateUser(ctx, user)
    if err != nil {
        return err
    }

    // Publish event
    event := UserUpdatedEvent{
        UserID:    user.ID,
        Timestamp: time.Now(),
    }
    eventData, _ := json.Marshal(event)
    s.redis.Publish(ctx, "user:updated", eventData)

    return nil
}

// Subscriber (cache invalidator)
func StartCacheInvalidator(ctx context.Context, rdb *redis.Client, cache *Cache) {
    pubsub := rdb.Subscribe(ctx, "user:updated", "user:deleted", "product:updated")

    go func() {
        for msg := range pubsub.Channel() {
            switch msg.Channel {
            case "user:updated", "user:deleted":
                var event UserUpdatedEvent
                json.Unmarshal([]byte(msg.Payload), &event)
                cache.DeletePattern(ctx, fmt.Sprintf("user:%d:*", event.UserID))

            case "product:updated":
                var event ProductUpdatedEvent
                json.Unmarshal([]byte(msg.Payload), &event)
                cache.Del(ctx, fmt.Sprintf("product:%d", event.ProductID))
            }
        }
    }()
}

type UserUpdatedEvent struct {
    UserID    int64     `json:"user_id"`
    Timestamp time.Time `json:"timestamp"`
}
```

---

## 5️⃣ Cache Stampede Prevention

<div dir="rtl">

عندما TTL ينتهي، ممكن كل الـ requests تروح للـ DB في نفس الوقت!

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cache Stampede Problem                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Time: 12:00:00 - Popular data cached                               │
│  TTL: 5 minutes                                                     │
│  Time: 12:05:00 - Cache expires!                                    │
│                                                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐               │
│  │ Request 1│ │ Request 2│ │ Request 3│ │ Request N│               │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘               │
│       │            │            │            │                      │
│       ▼            ▼            ▼            ▼                      │
│  ┌─────────────────────────────────────────────────┐               │
│  │              Cache: MISS! MISS! MISS!            │               │
│  └─────────────────────────────────────────────────┘               │
│       │            │            │            │                      │
│       ▼            ▼            ▼            ▼                      │
│  ┌─────────────────────────────────────────────────┐               │
│  │              Database: 💥 OVERLOADED!            │               │
│  └─────────────────────────────────────────────────┘               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Solution 1: Mutex Lock

```go
type CacheWithLock struct {
    cache *redis.Client
    locks sync.Map
}

func (c *CacheWithLock) GetWithLock(
    ctx context.Context,
    key string,
    ttl time.Duration,
    loader func() (interface{}, error),
) (interface{}, error) {
    // Try cache first
    data, err := c.cache.Get(ctx, key).Result()
    if err == nil {
        return data, nil
    }

    // Get or create lock for this key
    lockKey := key + ":lock"
    mu, _ := c.locks.LoadOrStore(lockKey, &sync.Mutex{})
    mutex := mu.(*sync.Mutex)

    mutex.Lock()
    defer mutex.Unlock()

    // Check cache again (maybe another goroutine filled it)
    data, err = c.cache.Get(ctx, key).Result()
    if err == nil {
        return data, nil
    }

    // We're the first one - load from source
    result, err := loader()
    if err != nil {
        return nil, err
    }

    // Store in cache
    c.cache.Set(ctx, key, result, ttl)

    return result, nil
}
```

### Solution 2: Distributed Lock (Redis)

```go
func (c *Cache) GetWithDistributedLock(
    ctx context.Context,
    key string,
    ttl time.Duration,
    loader func() (interface{}, error),
) (interface{}, error) {
    // Try cache first
    data, err := c.rdb.Get(ctx, key).Result()
    if err == nil {
        return data, nil
    }

    // Try to acquire lock
    lockKey := key + ":lock"
    lockID := uuid.New().String()
    acquired, err := c.rdb.SetNX(ctx, lockKey, lockID, 10*time.Second).Result()

    if !acquired {
        // Someone else is loading - wait and retry
        time.Sleep(100 * time.Millisecond)
        return c.GetWithDistributedLock(ctx, key, ttl, loader)
    }

    // We have the lock - make sure to release it
    defer c.releaseLock(ctx, lockKey, lockID)

    // Load from source
    result, err := loader()
    if err != nil {
        return nil, err
    }

    // Store in cache
    c.rdb.Set(ctx, key, result, ttl)

    return result, nil
}

func (c *Cache) releaseLock(ctx context.Context, key, lockID string) {
    // Only release if we own the lock
    script := `
        if redis.call("GET", KEYS[1]) == ARGV[1] then
            return redis.call("DEL", KEYS[1])
        else
            return 0
        end
    `
    c.rdb.Eval(ctx, script, []string{key}, lockID)
}
```

### Solution 3: Probabilistic Early Expiration

```go
func (c *Cache) GetWithEarlyExpiration(
    ctx context.Context,
    key string,
    baseTTL time.Duration,
    loader func() (interface{}, error),
) (interface{}, error) {
    // Get value and TTL
    pipe := c.rdb.Pipeline()
    getCmd := pipe.Get(ctx, key)
    ttlCmd := pipe.TTL(ctx, key)
    pipe.Exec(ctx)

    data, err := getCmd.Result()
    if err == redis.Nil {
        // Cache miss - load synchronously
        return c.loadAndCache(ctx, key, baseTTL, loader)
    }

    remainingTTL := ttlCmd.Val()

    // Probabilistic early refresh
    // As TTL decreases, probability of refresh increases
    threshold := float64(remainingTTL) / float64(baseTTL)
    random := rand.Float64()

    if random > threshold {
        // Refresh in background (don't block)
        go c.loadAndCache(context.Background(), key, baseTTL, loader)
    }

    return data, nil
}
```

---

## 6️⃣ Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Cache Invalidation Best Practices                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Delete, Don't Update                                            │
│     └─ Safer, avoids race conditions                                │
│                                                                      │
│  2. Use TTL as Safety Net                                           │
│     └─ Even with invalidation, set TTL                              │
│     └─ Protects against missed invalidations                        │
│                                                                      │
│  3. Invalidate After DB Commit                                      │
│     └─ Don't invalidate before DB write succeeds                    │
│                                                                      │
│  4. Design Keys for Easy Invalidation                               │
│     └─ user:{id}:* makes pattern matching easy                      │
│                                                                      │
│  5. Log Cache Operations                                            │
│     └─ Helps debug cache issues                                     │
│                                                                      │
│  6. Monitor Cache Hit Rate                                          │
│     └─ Alert if hit rate drops                                      │
│                                                                      │
│  7. Handle Invalidation Failures                                    │
│     └─ Log and alert, don't fail silently                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **TTL** = أبسط طريقة، لكن البيانات ممكن تكون قديمة
- ✅ **Delete on Update** = أفضل من Update (تجنب race conditions)
- ✅ **Event-Based** = للـ distributed systems
- ✅ **Cache Stampede** = مشكلة مهمة، استخدم locks أو early expiration
- ✅ **Pattern Keys** = صمم الـ keys لسهولة الحذف
- ✅ **TTL كـ Safety Net** = حتى مع invalidation، ضع TTL

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم عن CDN:

**➡️ [Lesson 6: CDN Basics](./06-cdn-basics.md)**

</div>

---

<div align="center">

[⬅️ Previous: Redis in Go](./04-redis-in-go.md) | [📚 Module Home](../README.md) | [➡️ Next: CDN Basics](./06-cdn-basics.md)

</div>
