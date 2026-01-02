# Lesson 2: Caching Strategies - استراتيجيات التخزين المؤقت 📋

<div dir="rtl">

## المقدمة

في عدة طرق للتعامل مع الـ Cache. اختيار الاستراتيجية الصح يعتمد على طبيعة التطبيق وحجم البيانات.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Strategies Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Caching Strategies                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Cache-Aside (Lazy Loading)                                      │
│     └─ الأكثر شيوعاً - التطبيق يدير الـ Cache                       │
│                                                                      │
│  2. Read-Through                                                    │
│     └─ الـ Cache يجلب البيانات تلقائياً                             │
│                                                                      │
│  3. Write-Through                                                   │
│     └─ كل كتابة تروح للـ Cache والـ DB معاً                         │
│                                                                      │
│  4. Write-Behind (Write-Back)                                       │
│     └─ الكتابة للـ Cache أولاً، ثم للـ DB لاحقاً                    │
│                                                                      │
│  5. Refresh-Ahead                                                   │
│     └─ تحديث الـ Cache قبل انتهاء الصلاحية                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Cache-Aside (Lazy Loading)

<div dir="rtl">

**الاستراتيجية الأكثر شيوعاً!** التطبيق هو اللي يدير الـ Cache.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Cache-Aside Pattern                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  READ Flow:                                                         │
│  ──────────                                                          │
│                                                                      │
│  ┌────────┐  1.Request  ┌────────┐                                  │
│  │ Client │────────────▶│  App   │                                  │
│  └────────┘             └────────┘                                  │
│                              │                                       │
│                    2. Check  │                                       │
│                       Cache  ▼                                       │
│                         ┌────────┐                                   │
│                         │ Cache  │                                   │
│                         └────────┘                                   │
│                              │                                       │
│              ┌───────────────┴───────────────┐                       │
│              │                               │                       │
│         Cache Hit                       Cache Miss                   │
│              │                               │                       │
│              ▼                               ▼                       │
│        Return Data              3. Query Database                   │
│                                        ┌────────┐                    │
│                                        │   DB   │                    │
│                                        └────────┘                    │
│                                              │                       │
│                               4. Store in   │                       │
│                                  Cache      ▼                       │
│                                        ┌────────┐                    │
│                                        │ Cache  │                    │
│                                        └────────┘                    │
│                                              │                       │
│                               5. Return     │                       │
│                                  Data       ▼                       │
│                                        ┌────────┐                    │
│                                        │ Client │                    │
│                                        └────────┘                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```go
type UserService struct {
    db    *sql.DB
    cache *redis.Client
}

// Cache-Aside Read
func (s *UserService) GetUser(ctx context.Context, id int64) (*User, error) {
    cacheKey := fmt.Sprintf("user:%d", id)

    // Step 1: Check cache
    cached, err := s.cache.Get(ctx, cacheKey).Result()
    if err == nil {
        // Cache hit!
        var user User
        json.Unmarshal([]byte(cached), &user)
        return &user, nil
    }

    // Step 2: Cache miss - query database
    user, err := s.queryUserFromDB(ctx, id)
    if err != nil {
        return nil, err
    }

    // Step 3: Store in cache
    data, _ := json.Marshal(user)
    s.cache.Set(ctx, cacheKey, data, 30*time.Minute)

    return user, nil
}

// Cache-Aside Write (Invalidation)
func (s *UserService) UpdateUser(ctx context.Context, user *User) error {
    // Step 1: Update database
    err := s.updateUserInDB(ctx, user)
    if err != nil {
        return err
    }

    // Step 2: Invalidate cache (don't update!)
    cacheKey := fmt.Sprintf("user:%d", user.ID)
    s.cache.Del(ctx, cacheKey)

    return nil
}
```

### Pros & Cons

```
✅ Pros:
─────────
• Simple to implement
• Only caches data that's actually requested
• Cache failure doesn't break the app
• Most control over caching logic

❌ Cons:
─────────
• Cache miss = slow first request
• Potential for stale data
• Application manages cache explicitly
• Cache stampede possible
```

---

## 2️⃣ Read-Through

<div dir="rtl">

الـ Cache library هو اللي يجلب البيانات من الـ DB تلقائياً.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Read-Through Pattern                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────┐  1.Request  ┌─────────────────────┐                     │
│  │ Client │────────────▶│                     │                     │
│  └────────┘             │   Cache Library     │                     │
│       ▲                 │   (with loader)     │                     │
│       │                 │                     │                     │
│       │                 └─────────────────────┘                     │
│       │                          │                                   │
│       │              ┌───────────┴───────────┐                       │
│       │              │                       │                       │
│       │         Cache Hit               Cache Miss                   │
│       │              │                       │                       │
│       │              │              2. Auto-load                    │
│       │              │                 from DB                      │
│       │              │                       ▼                       │
│       │              │                  ┌────────┐                   │
│       │              │                  │   DB   │                   │
│       │              │                  └────────┘                   │
│       │              │                       │                       │
│       │              │              3. Store & Return               │
│       └──────────────┴───────────────────────┘                       │
│                     4. Return Data                                   │
│                                                                      │
│  الفرق: التطبيق لا يتعامل مع DB مباشرة للقراءة                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```go
type ReadThroughCache struct {
    cache  *redis.Client
    loader func(ctx context.Context, key string) (interface{}, error)
    ttl    time.Duration
}

func NewReadThroughCache(
    cache *redis.Client,
    loader func(ctx context.Context, key string) (interface{}, error),
    ttl time.Duration,
) *ReadThroughCache {
    return &ReadThroughCache{
        cache:  cache,
        loader: loader,
        ttl:    ttl,
    }
}

func (c *ReadThroughCache) Get(ctx context.Context, key string) (interface{}, error) {
    // Try cache first
    cached, err := c.cache.Get(ctx, key).Result()
    if err == nil {
        var result interface{}
        json.Unmarshal([]byte(cached), &result)
        return result, nil
    }

    // Cache miss: auto-load from source
    result, err := c.loader(ctx, key)
    if err != nil {
        return nil, err
    }

    // Store in cache
    data, _ := json.Marshal(result)
    c.cache.Set(ctx, key, data, c.ttl)

    return result, nil
}

// Usage
func main() {
    cache := NewReadThroughCache(
        redisClient,
        func(ctx context.Context, key string) (interface{}, error) {
            // Extract ID from key like "user:123"
            id := extractID(key)
            return db.GetUser(ctx, id)
        },
        30*time.Minute,
    )

    // Just call Get - cache handles everything!
    user, err := cache.Get(ctx, "user:123")
}
```

---

## 3️⃣ Write-Through

<div dir="rtl">

كل عملية كتابة تروح للـ Cache والـ DB **معاً** في نفس الوقت.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Write-Through Pattern                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────┐  1.Write    ┌────────┐                                  │
│  │ Client │────────────▶│  App   │                                  │
│  └────────┘             └────────┘                                  │
│                              │                                       │
│                              │ 2. Write to both                     │
│                              │    (synchronously)                   │
│                    ┌─────────┴─────────┐                             │
│                    │                   │                             │
│                    ▼                   ▼                             │
│               ┌────────┐         ┌────────┐                         │
│               │ Cache  │         │   DB   │                         │
│               └────────┘         └────────┘                         │
│                    │                   │                             │
│                    └─────────┬─────────┘                             │
│                              │                                       │
│                    3. Both succeed?                                 │
│                              │                                       │
│                              ▼                                       │
│                    4. Return Success                                │
│                                                                      │
│  ⚠️ Both writes must succeed for operation to complete              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```go
type WriteThroughService struct {
    db    *sql.DB
    cache *redis.Client
}

func (s *WriteThroughService) CreateUser(ctx context.Context, user *User) error {
    // Start transaction for consistency
    tx, err := s.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    // Step 1: Write to database
    err = s.insertUserToDB(ctx, tx, user)
    if err != nil {
        return err
    }

    // Step 2: Write to cache
    cacheKey := fmt.Sprintf("user:%d", user.ID)
    data, _ := json.Marshal(user)
    err = s.cache.Set(ctx, cacheKey, data, 30*time.Minute).Err()
    if err != nil {
        // Cache write failed - rollback DB
        return err
    }

    // Step 3: Commit transaction
    return tx.Commit()
}

func (s *WriteThroughService) UpdateUser(ctx context.Context, user *User) error {
    // Update both DB and Cache synchronously
    err := s.updateUserInDB(ctx, user)
    if err != nil {
        return err
    }

    cacheKey := fmt.Sprintf("user:%d", user.ID)
    data, _ := json.Marshal(user)
    return s.cache.Set(ctx, cacheKey, data, 30*time.Minute).Err()
}
```

### Pros & Cons

```
✅ Pros:
─────────
• Cache always consistent with DB
• Read operations always fast (data in cache)
• No stale data issue

❌ Cons:
─────────
• Write operations slower (two writes)
• Cache may store data that's never read
• Higher complexity
• Cache failure = write failure
```

---

## 4️⃣ Write-Behind (Write-Back)

<div dir="rtl">

الكتابة للـ Cache أولاً، ثم للـ DB **بشكل asynchronous** لاحقاً.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Write-Behind Pattern                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────┐  1.Write    ┌────────┐                                  │
│  │ Client │────────────▶│  App   │                                  │
│  └────────┘             └────────┘                                  │
│       ▲                      │                                       │
│       │                      │ 2. Write to Cache                    │
│       │                      ▼                                       │
│       │                 ┌────────┐                                   │
│       │                 │ Cache  │                                   │
│       │                 └────────┘                                   │
│       │                      │                                       │
│   3. Return                  │ 4. Queue for                         │
│      immediately             │    async write                       │
│                              ▼                                       │
│                         ┌────────┐                                   │
│                         │ Queue  │                                   │
│                         └────────┘                                   │
│                              │                                       │
│                              │ 5. Background                        │
│                              │    worker writes                     │
│                              ▼                                       │
│                         ┌────────┐                                   │
│                         │   DB   │  (Eventually)                    │
│                         └────────┘                                   │
│                                                                      │
│  ⚠️ Risk: Data loss if cache fails before DB write!                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```go
type WriteBehindService struct {
    cache    *redis.Client
    writeQueue chan WriteOperation
}

type WriteOperation struct {
    Key   string
    Value interface{}
    Op    string // "create", "update", "delete"
}

func NewWriteBehindService(cache *redis.Client, db *sql.DB) *WriteBehindService {
    s := &WriteBehindService{
        cache:      cache,
        writeQueue: make(chan WriteOperation, 1000),
    }

    // Start background writer
    go s.backgroundWriter(db)

    return s
}

// Fast write - only to cache
func (s *WriteBehindService) UpdateUser(ctx context.Context, user *User) error {
    cacheKey := fmt.Sprintf("user:%d", user.ID)
    data, _ := json.Marshal(user)

    // Step 1: Write to cache (fast!)
    err := s.cache.Set(ctx, cacheKey, data, 1*time.Hour).Err()
    if err != nil {
        return err
    }

    // Step 2: Queue for async DB write
    s.writeQueue <- WriteOperation{
        Key:   cacheKey,
        Value: user,
        Op:    "update",
    }

    return nil // Return immediately!
}

// Background worker
func (s *WriteBehindService) backgroundWriter(db *sql.DB) {
    batch := make([]WriteOperation, 0, 100)
    ticker := time.NewTicker(100 * time.Millisecond)

    for {
        select {
        case op := <-s.writeQueue:
            batch = append(batch, op)

            // Batch full? Write now
            if len(batch) >= 100 {
                s.writeBatchToDB(db, batch)
                batch = batch[:0]
            }

        case <-ticker.C:
            // Time-based flush
            if len(batch) > 0 {
                s.writeBatchToDB(db, batch)
                batch = batch[:0]
            }
        }
    }
}

func (s *WriteBehindService) writeBatchToDB(db *sql.DB, batch []WriteOperation) {
    tx, _ := db.Begin()
    for _, op := range batch {
        switch op.Op {
        case "update":
            user := op.Value.(*User)
            // UPDATE users SET ... WHERE id = ?
            tx.Exec("UPDATE users SET name=?, email=? WHERE id=?",
                user.Name, user.Email, user.ID)
        }
    }
    tx.Commit()
}
```

### Pros & Cons

```
✅ Pros:
─────────
• Very fast writes
• Reduced DB load (batch writes)
• Good for write-heavy workloads

❌ Cons:
─────────
• DATA LOSS RISK if cache fails!
• Complex to implement correctly
• Eventual consistency only
• Debugging is harder
```

---

## 5️⃣ Refresh-Ahead

<div dir="rtl">

تحديث الـ Cache تلقائياً **قبل** انتهاء صلاحيته.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Refresh-Ahead Pattern                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TTL = 10 minutes                                                   │
│  Refresh threshold = 80% (8 minutes)                                │
│                                                                      │
│  Timeline:                                                          │
│  ────────────────────────────────────────────────────────────────   │
│  0min        4min        8min         10min                         │
│    │           │           │             │                          │
│    ├───────────┼───────────┼─────────────┤                          │
│    │   Normal  │   Normal  │ Refresh     │ Expired                  │
│    │   reads   │   reads   │ Zone        │ (if not refreshed)       │
│    │           │           │             │                          │
│                            ▼                                        │
│                     If request comes here:                          │
│                     1. Return cached data                           │
│                     2. Async refresh from DB                        │
│                     3. Update cache with new TTL                    │
│                                                                      │
│  Benefit: No user ever sees cache miss!                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```go
type RefreshAheadCache struct {
    cache           *redis.Client
    loader          func(ctx context.Context, key string) (interface{}, error)
    ttl             time.Duration
    refreshThreshold float64 // e.g., 0.8 = refresh at 80% of TTL
    refreshing      sync.Map // Track keys being refreshed
}

func (c *RefreshAheadCache) Get(ctx context.Context, key string) (interface{}, error) {
    // Get value and TTL
    pipe := c.cache.Pipeline()
    getCmd := pipe.Get(ctx, key)
    ttlCmd := pipe.TTL(ctx, key)
    pipe.Exec(ctx)

    cached, err := getCmd.Result()
    if err == redis.Nil {
        // Cache miss - load synchronously
        return c.loadAndCache(ctx, key)
    }

    // Check if we should refresh
    remainingTTL := ttlCmd.Val()
    threshold := time.Duration(float64(c.ttl) * c.refreshThreshold)

    if remainingTTL < threshold {
        // Trigger async refresh
        c.asyncRefresh(ctx, key)
    }

    var result interface{}
    json.Unmarshal([]byte(cached), &result)
    return result, nil
}

func (c *RefreshAheadCache) asyncRefresh(ctx context.Context, key string) {
    // Prevent multiple refreshes for same key
    if _, loaded := c.refreshing.LoadOrStore(key, true); loaded {
        return // Already refreshing
    }

    go func() {
        defer c.refreshing.Delete(key)

        result, err := c.loader(ctx, key)
        if err != nil {
            return // Keep existing cached value
        }

        data, _ := json.Marshal(result)
        c.cache.Set(ctx, key, data, c.ttl)
    }()
}

func (c *RefreshAheadCache) loadAndCache(ctx context.Context, key string) (interface{}, error) {
    result, err := c.loader(ctx, key)
    if err != nil {
        return nil, err
    }

    data, _ := json.Marshal(result)
    c.cache.Set(ctx, key, data, c.ttl)

    return result, nil
}
```

---

## 📊 Strategy Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Strategy Comparison                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Strategy       │ Consistency │ Performance │ Complexity │ Use Case │
│  ───────────────┼─────────────┼─────────────┼────────────┼──────────│
│  Cache-Aside    │ Medium      │ Good        │ Simple     │ General  │
│  Read-Through   │ Medium      │ Good        │ Medium     │ Library  │
│  Write-Through  │ High        │ Medium      │ Medium     │ Banking  │
│  Write-Behind   │ Low         │ Excellent   │ Complex    │ Analytics│
│  Refresh-Ahead  │ Medium      │ Excellent   │ Medium     │ Hot data │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Decision Guide

```
Choose Cache-Aside when:
✓ You want simple implementation
✓ You need full control
✓ Data isn't accessed predictably

Choose Write-Through when:
✓ Data consistency is critical
✓ Read performance is important
✓ You can tolerate slower writes

Choose Write-Behind when:
✓ Write performance is critical
✓ You can tolerate eventual consistency
✓ Data loss is acceptable (analytics, logs)

Choose Refresh-Ahead when:
✓ You have predictably hot data
✓ Cache misses are very expensive
✓ You want to eliminate miss latency
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Cache-Aside** = الأكثر شيوعاً، التطبيق يدير الـ Cache
- ✅ **Read-Through** = الـ Cache يجلب البيانات تلقائياً
- ✅ **Write-Through** = كتابة متزامنة للـ Cache والـ DB
- ✅ **Write-Behind** = كتابة سريعة للـ Cache، ثم async للـ DB
- ✅ **Refresh-Ahead** = تحديث قبل انتهاء الصلاحية
- ✅ اختر الاستراتيجية حسب **consistency** vs **performance**

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد ما فهمت الاستراتيجيات، خلينا نتعلم عن Redis:

**➡️ [Lesson 3: Redis Fundamentals](./03-redis-fundamentals.md)**

</div>

---

<div align="center">

[⬅️ Previous: What is Caching](./01-what-is-caching.md) | [📚 Module Home](../README.md) | [➡️ Next: Redis Fundamentals](./03-redis-fundamentals.md)

</div>
