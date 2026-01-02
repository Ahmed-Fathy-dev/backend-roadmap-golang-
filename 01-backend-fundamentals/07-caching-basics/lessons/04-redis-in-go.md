# Lesson 4: Redis in Go - استخدام Redis في Go 🚀

<div dir="rtl">

## المقدمة

في هذا الدرس هنتعلم إزاي نستخدم Redis في تطبيقات Go باستخدام مكتبة **go-redis**.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 1️⃣ Installation

```bash
# Install go-redis v9
go get github.com/redis/go-redis/v9
```

---

## 2️⃣ Connection Setup

```go
package main

import (
    "context"
    "fmt"
    "time"

    "github.com/redis/go-redis/v9"
)

func main() {
    // Create Redis client
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379", // Redis server address
        Password: "",               // No password
        DB:       0,                // Default DB

        // Connection pool settings
        PoolSize:     10,               // Max connections
        MinIdleConns: 5,                // Min idle connections
        PoolTimeout:  30 * time.Second, // Wait time for connection

        // Timeouts
        DialTimeout:  5 * time.Second,
        ReadTimeout:  3 * time.Second,
        WriteTimeout: 3 * time.Second,
    })

    // Test connection
    ctx := context.Background()
    _, err := rdb.Ping(ctx).Result()
    if err != nil {
        panic(fmt.Sprintf("Failed to connect to Redis: %v", err))
    }

    fmt.Println("Connected to Redis!")

    // Don't forget to close
    defer rdb.Close()
}
```

### Connection with URL

```go
// Using URL (good for production with environment variables)
opt, err := redis.ParseURL("redis://user:password@localhost:6379/0")
if err != nil {
    panic(err)
}

rdb := redis.NewClient(opt)

// Or with TLS (rediss://)
opt, _ := redis.ParseURL("rediss://user:password@your-redis.cloud:6379/0")
```

### Singleton Pattern

```go
package cache

import (
    "context"
    "os"
    "sync"
    "time"

    "github.com/redis/go-redis/v9"
)

var (
    client *redis.Client
    once   sync.Once
)

// GetClient returns singleton Redis client
func GetClient() *redis.Client {
    once.Do(func() {
        client = redis.NewClient(&redis.Options{
            Addr:         os.Getenv("REDIS_URL"),
            Password:     os.Getenv("REDIS_PASSWORD"),
            DB:           0,
            PoolSize:     10,
            MinIdleConns: 5,
        })

        // Verify connection
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()

        if err := client.Ping(ctx).Err(); err != nil {
            panic("Redis connection failed: " + err.Error())
        }
    })

    return client
}
```

---

## 3️⃣ Basic Operations

### Strings

```go
func stringOperations(ctx context.Context, rdb *redis.Client) {
    // SET
    err := rdb.Set(ctx, "name", "Ahmed", 0).Err() // 0 = no expiration
    if err != nil {
        log.Fatal(err)
    }

    // SET with expiration
    err = rdb.Set(ctx, "session:123", "user_data", 30*time.Minute).Err()
    if err != nil {
        log.Fatal(err)
    }

    // Alternative: SetEx
    err = rdb.SetEx(ctx, "session:456", "data", 1*time.Hour).Err()

    // GET
    val, err := rdb.Get(ctx, "name").Result()
    if err == redis.Nil {
        fmt.Println("Key does not exist")
    } else if err != nil {
        log.Fatal(err)
    } else {
        fmt.Println("name:", val) // "Ahmed"
    }

    // GET with default value
    val = rdb.Get(ctx, "nonexistent").Val() // Returns "" if not found

    // SetNX (Set if Not eXists)
    wasSet, err := rdb.SetNX(ctx, "lock:resource", "owner", 10*time.Second).Result()
    if wasSet {
        fmt.Println("Lock acquired!")
    } else {
        fmt.Println("Lock already held")
    }

    // DELETE
    deleted, err := rdb.Del(ctx, "name", "session:123").Result()
    fmt.Printf("Deleted %d keys\n", deleted)

    // INCREMENT
    rdb.Set(ctx, "counter", 0, 0)
    newVal, _ := rdb.Incr(ctx, "counter").Result()      // 1
    newVal, _ = rdb.IncrBy(ctx, "counter", 5).Result()  // 6
    newVal, _ = rdb.Decr(ctx, "counter").Result()       // 5

    // EXISTS
    exists, _ := rdb.Exists(ctx, "counter").Result()
    fmt.Printf("Exists: %d\n", exists) // 1 if exists

    // TTL
    ttl, _ := rdb.TTL(ctx, "session:456").Result()
    fmt.Printf("TTL: %v\n", ttl)
}
```

### Store Structs as JSON

```go
type User struct {
    ID    int64  `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func storeUser(ctx context.Context, rdb *redis.Client, user *User) error {
    data, err := json.Marshal(user)
    if err != nil {
        return err
    }

    key := fmt.Sprintf("user:%d", user.ID)
    return rdb.Set(ctx, key, data, 30*time.Minute).Err()
}

func getUser(ctx context.Context, rdb *redis.Client, id int64) (*User, error) {
    key := fmt.Sprintf("user:%d", id)

    data, err := rdb.Get(ctx, key).Result()
    if err == redis.Nil {
        return nil, nil // Not found
    }
    if err != nil {
        return nil, err
    }

    var user User
    err = json.Unmarshal([]byte(data), &user)
    return &user, err
}
```

---

## 4️⃣ Hashes

```go
func hashOperations(ctx context.Context, rdb *redis.Client) {
    // HSET - single field
    err := rdb.HSet(ctx, "user:1", "name", "Ahmed").Err()

    // HSET - multiple fields
    err = rdb.HSet(ctx, "user:1",
        "name", "Ahmed",
        "email", "ahmed@test.com",
        "age", 25,
    ).Err()

    // Or use map
    fields := map[string]interface{}{
        "name":  "Ahmed",
        "email": "ahmed@test.com",
        "age":   25,
    }
    err = rdb.HSet(ctx, "user:1", fields).Err()

    // HGET - single field
    name, err := rdb.HGet(ctx, "user:1", "name").Result()
    fmt.Println("Name:", name)

    // HGETALL - all fields
    result, err := rdb.HGetAll(ctx, "user:1").Result()
    for field, value := range result {
        fmt.Printf("%s: %s\n", field, value)
    }

    // Scan into struct
    var user struct {
        Name  string `redis:"name"`
        Email string `redis:"email"`
        Age   int    `redis:"age"`
    }
    err = rdb.HGetAll(ctx, "user:1").Scan(&user)
    fmt.Printf("User: %+v\n", user)

    // HMGET - multiple fields
    values, err := rdb.HMGet(ctx, "user:1", "name", "email").Result()

    // HINCRBY - increment numeric field
    newAge, err := rdb.HIncrBy(ctx, "user:1", "age", 1).Result()

    // HDEL - delete field
    rdb.HDel(ctx, "user:1", "age")

    // HEXISTS - check field exists
    exists, _ := rdb.HExists(ctx, "user:1", "name").Result()
}
```

### User Cache with Hash

```go
type UserCache struct {
    rdb *redis.Client
}

func (c *UserCache) SetUser(ctx context.Context, user *User) error {
    key := fmt.Sprintf("user:%d", user.ID)
    return c.rdb.HSet(ctx, key,
        "id", user.ID,
        "name", user.Name,
        "email", user.Email,
    ).Err()
}

func (c *UserCache) GetUser(ctx context.Context, id int64) (*User, error) {
    key := fmt.Sprintf("user:%d", id)

    exists, _ := c.rdb.Exists(ctx, key).Result()
    if exists == 0 {
        return nil, nil
    }

    var user User
    err := c.rdb.HGetAll(ctx, key).Scan(&user)
    return &user, err
}

func (c *UserCache) UpdateField(ctx context.Context, id int64, field string, value interface{}) error {
    key := fmt.Sprintf("user:%d", id)
    return c.rdb.HSet(ctx, key, field, value).Err()
}
```

---

## 5️⃣ Lists

```go
func listOperations(ctx context.Context, rdb *redis.Client) {
    // LPUSH - push to left (beginning)
    err := rdb.LPush(ctx, "queue", "task1", "task2").Err()

    // RPUSH - push to right (end)
    err = rdb.RPush(ctx, "queue", "task3").Err()

    // LPOP - pop from left
    val, err := rdb.LPop(ctx, "queue").Result()
    if err == redis.Nil {
        fmt.Println("Queue is empty")
    }

    // RPOP - pop from right
    val, err = rdb.RPop(ctx, "queue").Result()

    // BLPOP - blocking pop (wait for item)
    result, err := rdb.BLPop(ctx, 30*time.Second, "queue").Result()
    if err == redis.Nil {
        fmt.Println("Timeout - no items")
    } else {
        fmt.Printf("Queue: %s, Value: %s\n", result[0], result[1])
    }

    // LRANGE - get range (without removing)
    items, err := rdb.LRange(ctx, "queue", 0, -1).Result() // All items
    items, err = rdb.LRange(ctx, "queue", 0, 9).Result()   // First 10

    // LLEN - get length
    length, err := rdb.LLen(ctx, "queue").Result()

    // LTRIM - keep only specified range
    err = rdb.LTrim(ctx, "recent", 0, 99).Err() // Keep only first 100
}
```

### Simple Job Queue

```go
type JobQueue struct {
    rdb      *redis.Client
    queueKey string
}

type Job struct {
    ID      string          `json:"id"`
    Type    string          `json:"type"`
    Payload json.RawMessage `json:"payload"`
}

func (q *JobQueue) Enqueue(ctx context.Context, job *Job) error {
    data, err := json.Marshal(job)
    if err != nil {
        return err
    }
    return q.rdb.RPush(ctx, q.queueKey, data).Err()
}

func (q *JobQueue) Dequeue(ctx context.Context, timeout time.Duration) (*Job, error) {
    result, err := q.rdb.BLPop(ctx, timeout, q.queueKey).Result()
    if err == redis.Nil {
        return nil, nil // Timeout
    }
    if err != nil {
        return nil, err
    }

    var job Job
    err = json.Unmarshal([]byte(result[1]), &job)
    return &job, err
}
```

---

## 6️⃣ Sets

```go
func setOperations(ctx context.Context, rdb *redis.Client) {
    // SADD - add members
    added, err := rdb.SAdd(ctx, "tags:post:1", "go", "redis", "cache").Result()
    fmt.Printf("Added %d new members\n", added)

    // SMEMBERS - get all members
    members, err := rdb.SMembers(ctx, "tags:post:1").Result()
    fmt.Println("Tags:", members)

    // SISMEMBER - check membership
    isMember, err := rdb.SIsMember(ctx, "tags:post:1", "go").Result()
    fmt.Println("Is member:", isMember)

    // SCARD - count members
    count, err := rdb.SCard(ctx, "tags:post:1").Result()

    // SREM - remove member
    removed, err := rdb.SRem(ctx, "tags:post:1", "cache").Result()

    // Set operations
    rdb.SAdd(ctx, "user:1:friends", "alice", "bob", "charlie")
    rdb.SAdd(ctx, "user:2:friends", "alice", "dave", "eve")

    // SINTER - intersection (mutual friends)
    mutual, _ := rdb.SInter(ctx, "user:1:friends", "user:2:friends").Result()
    fmt.Println("Mutual friends:", mutual)

    // SUNION - union (all friends)
    all, _ := rdb.SUnion(ctx, "user:1:friends", "user:2:friends").Result()

    // SDIFF - difference (friends of 1 not in 2)
    diff, _ := rdb.SDiff(ctx, "user:1:friends", "user:2:friends").Result()

    // SRANDMEMBER - random member
    random, _ := rdb.SRandMember(ctx, "tags:post:1").Result()
}
```

---

## 7️⃣ Sorted Sets

```go
func sortedSetOperations(ctx context.Context, rdb *redis.Client) {
    // ZADD - add with scores
    err := rdb.ZAdd(ctx, "leaderboard",
        redis.Z{Score: 100, Member: "player:1"},
        redis.Z{Score: 250, Member: "player:2"},
        redis.Z{Score: 150, Member: "player:3"},
    ).Err()

    // ZSCORE - get score
    score, err := rdb.ZScore(ctx, "leaderboard", "player:2").Result()
    fmt.Printf("Player 2 score: %.0f\n", score)

    // ZRANK - get rank (0-indexed, low to high)
    rank, err := rdb.ZRank(ctx, "leaderboard", "player:1").Result()

    // ZREVRANK - rank from high to low
    rank, err = rdb.ZRevRank(ctx, "leaderboard", "player:2").Result()
    fmt.Printf("Player 2 rank: %d (0 = top)\n", rank)

    // ZRANGE - get by rank (low to high)
    players, err := rdb.ZRange(ctx, "leaderboard", 0, -1).Result()

    // ZREVRANGE - get by rank (high to low)
    topPlayers, err := rdb.ZRevRange(ctx, "leaderboard", 0, 9).Result() // Top 10

    // With scores
    results, err := rdb.ZRevRangeWithScores(ctx, "leaderboard", 0, 9).Result()
    for _, z := range results {
        fmt.Printf("%s: %.0f\n", z.Member, z.Score)
    }

    // ZINCRBY - increment score
    newScore, err := rdb.ZIncrBy(ctx, "leaderboard", 50, "player:1").Result()

    // ZRANGEBYSCORE - get by score range
    inRange, err := rdb.ZRangeByScore(ctx, "leaderboard", &redis.ZRangeBy{
        Min: "100",
        Max: "200",
    }).Result()

    // ZCOUNT - count in score range
    count, err := rdb.ZCount(ctx, "leaderboard", "100", "200").Result()

    // ZREM - remove member
    rdb.ZRem(ctx, "leaderboard", "player:3")
}
```

### Leaderboard Implementation

```go
type Leaderboard struct {
    rdb *redis.Client
    key string
}

func NewLeaderboard(rdb *redis.Client, name string) *Leaderboard {
    return &Leaderboard{
        rdb: rdb,
        key: fmt.Sprintf("leaderboard:%s", name),
    }
}

func (l *Leaderboard) AddScore(ctx context.Context, playerID string, score float64) error {
    return l.rdb.ZIncrBy(ctx, l.key, score, playerID).Err()
}

func (l *Leaderboard) GetRank(ctx context.Context, playerID string) (int64, error) {
    rank, err := l.rdb.ZRevRank(ctx, l.key, playerID).Result()
    if err == redis.Nil {
        return -1, nil
    }
    return rank, err
}

func (l *Leaderboard) GetTopN(ctx context.Context, n int64) ([]PlayerScore, error) {
    results, err := l.rdb.ZRevRangeWithScores(ctx, l.key, 0, n-1).Result()
    if err != nil {
        return nil, err
    }

    players := make([]PlayerScore, len(results))
    for i, z := range results {
        players[i] = PlayerScore{
            PlayerID: z.Member.(string),
            Score:    z.Score,
            Rank:     int64(i + 1),
        }
    }
    return players, nil
}

type PlayerScore struct {
    PlayerID string
    Score    float64
    Rank     int64
}
```

---

## 8️⃣ Pipelining

<div dir="rtl">

لتنفيذ عدة commands في request واحد - أسرع بكثير!

</div>

```go
func pipelineExample(ctx context.Context, rdb *redis.Client) {
    // Without pipeline: Each command = 1 round trip
    // With pipeline: All commands = 1 round trip

    pipe := rdb.Pipeline()

    // Queue commands (not executed yet)
    incr := pipe.Incr(ctx, "counter")
    pipe.Expire(ctx, "counter", time.Hour)
    get := pipe.Get(ctx, "name")

    // Execute all at once
    _, err := pipe.Exec(ctx)
    if err != nil && err != redis.Nil {
        log.Fatal(err)
    }

    // Get results
    fmt.Println("Counter:", incr.Val())
    fmt.Println("Name:", get.Val())
}

// Batch operations
func batchGet(ctx context.Context, rdb *redis.Client, keys []string) (map[string]string, error) {
    pipe := rdb.Pipeline()

    cmds := make(map[string]*redis.StringCmd)
    for _, key := range keys {
        cmds[key] = pipe.Get(ctx, key)
    }

    _, err := pipe.Exec(ctx)
    if err != nil && err != redis.Nil {
        return nil, err
    }

    results := make(map[string]string)
    for key, cmd := range cmds {
        val, err := cmd.Result()
        if err == nil {
            results[key] = val
        }
    }

    return results, nil
}

func batchSet(ctx context.Context, rdb *redis.Client, items map[string]interface{}, ttl time.Duration) error {
    pipe := rdb.Pipeline()

    for key, value := range items {
        pipe.Set(ctx, key, value, ttl)
    }

    _, err := pipe.Exec(ctx)
    return err
}
```

---

## 9️⃣ Complete Cache Layer

```go
package cache

import (
    "context"
    "encoding/json"
    "fmt"
    "time"

    "github.com/redis/go-redis/v9"
)

type Cache struct {
    rdb *redis.Client
}

func NewCache(rdb *redis.Client) *Cache {
    return &Cache{rdb: rdb}
}

// Generic Get with type parameter
func Get[T any](c *Cache, ctx context.Context, key string) (*T, error) {
    data, err := c.rdb.Get(ctx, key).Result()
    if err == redis.Nil {
        return nil, nil
    }
    if err != nil {
        return nil, err
    }

    var result T
    err = json.Unmarshal([]byte(data), &result)
    return &result, err
}

// Generic Set
func Set[T any](c *Cache, ctx context.Context, key string, value T, ttl time.Duration) error {
    data, err := json.Marshal(value)
    if err != nil {
        return err
    }
    return c.rdb.Set(ctx, key, data, ttl).Err()
}

// GetOrSet - Cache-Aside pattern
func GetOrSet[T any](
    c *Cache,
    ctx context.Context,
    key string,
    ttl time.Duration,
    loader func() (*T, error),
) (*T, error) {
    // Try cache first
    result, err := Get[T](c, ctx, key)
    if err != nil {
        return nil, err
    }
    if result != nil {
        return result, nil // Cache hit
    }

    // Cache miss - load from source
    result, err = loader()
    if err != nil {
        return nil, err
    }

    // Store in cache (async to not block)
    go Set(c, context.Background(), key, result, ttl)

    return result, nil
}

// Delete
func (c *Cache) Delete(ctx context.Context, keys ...string) error {
    return c.rdb.Del(ctx, keys...).Err()
}

// DeletePattern - delete all keys matching pattern
func (c *Cache) DeletePattern(ctx context.Context, pattern string) error {
    var cursor uint64
    for {
        keys, nextCursor, err := c.rdb.Scan(ctx, cursor, pattern, 100).Result()
        if err != nil {
            return err
        }

        if len(keys) > 0 {
            c.rdb.Del(ctx, keys...)
        }

        cursor = nextCursor
        if cursor == 0 {
            break
        }
    }
    return nil
}

// Usage Example
func main() {
    rdb := redis.NewClient(&redis.Options{Addr: "localhost:6379"})
    cache := NewCache(rdb)
    ctx := context.Background()

    // Cache user
    user := &User{ID: 1, Name: "Ahmed"}
    Set(cache, ctx, "user:1", user, 30*time.Minute)

    // Get user
    cached, _ := Get[User](cache, ctx, "user:1")
    fmt.Printf("Cached: %+v\n", cached)

    // GetOrSet pattern
    user, _ = GetOrSet(cache, ctx, "user:2", 30*time.Minute, func() (*User, error) {
        // This runs only on cache miss
        return db.GetUser(ctx, 2)
    })
}
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ استخدم **go-redis/v9** - المكتبة الأكثر شيوعاً
- ✅ **Connection pooling** مدمج تلقائياً
- ✅ استخدم **context** لكل operation
- ✅ **Pipelining** لتحسين الأداء مع عدة commands
- ✅ **JSON** لتخزين الـ structs
- ✅ تحقق من **redis.Nil** للـ key not found

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد ما تعلمت استخدام Redis، خلينا نتعلم عن الـ Cache Invalidation:

**➡️ [Lesson 5: Cache Invalidation](./05-cache-invalidation.md)**

</div>

---

<div align="center">

[⬅️ Previous: Redis Fundamentals](./03-redis-fundamentals.md) | [📚 Module Home](../README.md) | [➡️ Next: Cache Invalidation](./05-cache-invalidation.md)

</div>
