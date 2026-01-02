# Lesson 3: Rate Limiting - تحديد معدل الطلبات ⏱️

<div dir="rtl">

## المقدمة

**Rate Limiting** بيحمي الـ API من الـ abuse والـ DoS attacks عن طريق تحديد عدد الـ requests المسموحة.

بدونه، مستخدم واحد ممكن يضرب السيرفر بآلاف الـ requests!

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 Why Rate Limiting?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Why Rate Limiting?                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Without Rate Limiting:                                              │
│  ──────────────────────                                              │
│  Attacker → 1000 requests/second → Server crashes! 💥              │
│  Bot      → Scrapes all your data                                   │
│  User     → Forgot loop, floods API                                 │
│                                                                      │
│  With Rate Limiting:                                                 │
│  ────────────────────                                                │
│  Attacker → 100 requests... → 429 Too Many Requests ⛔             │
│  Bot      → Blocked or slowed down                                  │
│  User     → Gets warning, fixes code                                │
│                                                                      │
│  Benefits:                                                           │
│  ─────────                                                           │
│  ✅ Prevents DoS/DDoS attacks                                       │
│  ✅ Fair usage for all users                                        │
│  ✅ Protects expensive operations                                   │
│  ✅ Controls API costs                                               │
│  ✅ Prevents brute force attacks                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Rate Limiting Strategies

### By IP Address

```go
// Simple: Limit per IP
// 100 requests per minute per IP

Client IP: 192.168.1.1 → 50/100 used
Client IP: 192.168.1.2 → 80/100 used
Client IP: 192.168.1.3 → 100/100 used → BLOCKED

// Problem: Multiple users behind same IP (NAT)
// Solution: Combine with user ID when authenticated
```

### By User ID

```go
// Better for authenticated APIs
// 1000 requests per hour per user

User ID: 123 → 500/1000 used
User ID: 456 → 999/1000 used
User ID: 789 → 1000/1000 → BLOCKED

// Even if same IP, different limits per user
```

### By API Key

```go
// Common for third-party APIs
// Different tiers = different limits

Free Tier:     100 requests/day
Pro Tier:      10,000 requests/day
Enterprise:    Unlimited

api_key: "sk_free_xxx"  → 90/100 used
api_key: "sk_pro_yyy"   → 5000/10000 used
```

---

## 2️⃣ Algorithms

### Fixed Window

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Fixed Window Counter                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Window: 1 minute                                                    │
│  Limit: 100 requests                                                 │
│                                                                      │
│  12:00:00 ─────────────────────────────────── 12:01:00              │
│     [Request 1] [Request 2] ... [Request 100]                       │
│     Counter: 100 ✅                                                 │
│                                                                      │
│     [Request 101] → BLOCKED! ⛔                                     │
│                                                                      │
│  12:01:00 ─────────────────────────────────── 12:02:00              │
│     Counter resets to 0                                              │
│     [Request 1] ✅                                                  │
│                                                                      │
│  ⚠️ Problem: Burst at window boundary                               │
│     12:00:59 → 100 requests ✅                                      │
│     12:01:00 → 100 requests ✅                                      │
│     = 200 requests in 2 seconds!                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Sliding Window Log

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Sliding Window Log                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Keep timestamp of each request                                      │
│  Window slides with current time                                     │
│                                                                      │
│  Request log for user 123:                                           │
│  [12:00:15, 12:00:30, 12:00:45, 12:01:00, 12:01:10, ...]           │
│                                                                      │
│  At 12:01:30:                                                        │
│  - Look back 1 minute (12:00:30 to 12:01:30)                       │
│  - Count: 4 requests                                                 │
│  - If < 100 → Allow                                                 │
│                                                                      │
│  ✅ More accurate than fixed window                                 │
│  ❌ More memory (stores all timestamps)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Token Bucket (Most Common)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Token Bucket Algorithm                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Bucket: [●●●●●●●●●●] (10 tokens max)                               │
│                                                                      │
│  Refill Rate: 1 token per second                                    │
│  Each request costs 1 token                                         │
│                                                                      │
│  Time 0:  [●●●●●●●●●●] 10 tokens                                    │
│  Request: [●●●●●●●●●○]  9 tokens                                    │
│  Request: [●●●●●●●●○○]  8 tokens                                    │
│  ...                                                                 │
│  Request: [○○○○○○○○○○]  0 tokens                                    │
│  Request: BLOCKED! ⛔ (no tokens)                                   │
│                                                                      │
│  After 5 seconds:                                                    │
│  Refill:  [●●●●●○○○○○]  5 tokens                                    │
│  Request: [●●●●○○○○○○]  4 tokens ✅                                 │
│                                                                      │
│  ✅ Allows bursts (up to bucket size)                               │
│  ✅ Smooth rate limiting                                             │
│  ✅ Memory efficient                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ Implementation in Go

### Simple In-Memory Rate Limiter

```go
import (
    "sync"
    "time"
)

type RateLimiter struct {
    requests map[string][]time.Time
    mu       sync.RWMutex
    limit    int
    window   time.Duration
}

func NewRateLimiter(limit int, window time.Duration) *RateLimiter {
    return &RateLimiter{
        requests: make(map[string][]time.Time),
        limit:    limit,
        window:   window,
    }
}

func (rl *RateLimiter) Allow(key string) bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()

    now := time.Now()
    windowStart := now.Add(-rl.window)

    // Get existing requests for this key
    timestamps, exists := rl.requests[key]

    // Filter out old requests
    var recent []time.Time
    if exists {
        for _, t := range timestamps {
            if t.After(windowStart) {
                recent = append(recent, t)
            }
        }
    }

    // Check if under limit
    if len(recent) >= rl.limit {
        return false
    }

    // Add current request
    rl.requests[key] = append(recent, now)
    return true
}

// Middleware
func RateLimitMiddleware(limiter *RateLimiter) gin.HandlerFunc {
    return func(c *gin.Context) {
        // Use IP as key (or user ID for authenticated routes)
        key := c.ClientIP()

        if !limiter.Allow(key) {
            c.JSON(429, gin.H{
                "error":       "Too many requests",
                "retry_after": "60 seconds",
            })
            c.Abort()
            return
        }

        c.Next()
    }
}

// Usage
func main() {
    r := gin.Default()

    // 100 requests per minute
    limiter := NewRateLimiter(100, time.Minute)
    r.Use(RateLimitMiddleware(limiter))

    r.GET("/api/data", getData)
    r.Run()
}
```

### Using Redis (Production)

```go
import (
    "context"
    "time"
    "github.com/go-redis/redis/v8"
)

type RedisRateLimiter struct {
    client *redis.Client
    limit  int
    window time.Duration
}

func NewRedisRateLimiter(client *redis.Client, limit int, window time.Duration) *RedisRateLimiter {
    return &RedisRateLimiter{
        client: client,
        limit:  limit,
        window: window,
    }
}

func (rl *RedisRateLimiter) Allow(ctx context.Context, key string) (bool, int, error) {
    now := time.Now().UnixNano()
    windowStart := now - rl.window.Nanoseconds()

    pipe := rl.client.Pipeline()

    // Remove old entries
    pipe.ZRemRangeByScore(ctx, key, "0", fmt.Sprintf("%d", windowStart))

    // Add current request
    pipe.ZAdd(ctx, key, &redis.Z{Score: float64(now), Member: now})

    // Count requests in window
    countCmd := pipe.ZCard(ctx, key)

    // Set expiry on the key
    pipe.Expire(ctx, key, rl.window)

    _, err := pipe.Exec(ctx)
    if err != nil {
        return false, 0, err
    }

    count := int(countCmd.Val())
    remaining := rl.limit - count

    return count <= rl.limit, remaining, nil
}

// Middleware with headers
func RedisRateLimitMiddleware(limiter *RedisRateLimiter) gin.HandlerFunc {
    return func(c *gin.Context) {
        ctx := c.Request.Context()
        key := "ratelimit:" + c.ClientIP()

        allowed, remaining, err := limiter.Allow(ctx, key)
        if err != nil {
            c.JSON(500, gin.H{"error": "Rate limit error"})
            c.Abort()
            return
        }

        // Set rate limit headers
        c.Header("X-RateLimit-Limit", fmt.Sprintf("%d", limiter.limit))
        c.Header("X-RateLimit-Remaining", fmt.Sprintf("%d", remaining))

        if !allowed {
            c.Header("Retry-After", "60")
            c.JSON(429, gin.H{
                "error":       "Rate limit exceeded",
                "retry_after": 60,
            })
            c.Abort()
            return
        }

        c.Next()
    }
}
```

---

## 4️⃣ Response Headers

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1640000000

---

HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000060

{
  "error": "Rate limit exceeded",
  "message": "Too many requests. Please try again later.",
  "retry_after": 60
}
```

---

## 5️⃣ Different Limits for Different Routes

```go
func main() {
    r := gin.Default()

    // Different limiters for different routes
    publicLimiter := NewRedisRateLimiter(rdb, 100, time.Minute)      // 100/min
    authLimiter := NewRedisRateLimiter(rdb, 5, time.Minute)          // 5/min (login)
    heavyLimiter := NewRedisRateLimiter(rdb, 10, time.Hour)          // 10/hour (reports)

    // Public API
    r.GET("/api/products", RateLimit(publicLimiter), getProducts)

    // Auth routes (stricter to prevent brute force)
    auth := r.Group("/auth")
    auth.Use(RateLimit(authLimiter))
    {
        auth.POST("/login", login)
        auth.POST("/register", register)
    }

    // Heavy operations
    r.GET("/api/reports", RateLimit(heavyLimiter), generateReport)
}
```

---

## 6️⃣ Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                Rate Limiting Best Practices                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use Redis for distributed systems                                │
│     └─ In-memory won't work with multiple servers                  │
│                                                                      │
│  2. Return proper status code (429)                                  │
│     └─ Not 403 (forbidden) or 500 (error)                          │
│                                                                      │
│  3. Include Retry-After header                                       │
│     └─ Tells client when to retry                                  │
│                                                                      │
│  4. Include rate limit headers                                       │
│     └─ X-RateLimit-Limit, Remaining, Reset                         │
│                                                                      │
│  5. Different limits per tier                                        │
│     └─ Free vs Pro vs Enterprise                                   │
│                                                                      │
│  6. Stricter limits on sensitive endpoints                          │
│     └─ Login, password reset, payment                              │
│                                                                      │
│  7. Consider authenticated vs anonymous                              │
│     └─ Logged-in users may get higher limits                       │
│                                                                      │
│  8. Log rate limit hits                                              │
│     └─ Monitor for attacks or abuse                                │
│                                                                      │
│  9. Graceful degradation                                             │
│     └─ If Redis fails, allow requests (fail open)                  │
│     └─ Or use local fallback (fail closed)                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Rate Limiting** = حماية من الـ abuse والـ attacks
- ✅ **Token Bucket** = أشهر algorithm
- ✅ **Redis** = للـ production multi-server
- ✅ **429 Status** = الـ response الصحيح
- ✅ **Headers** = X-RateLimit-* و Retry-After
- ✅ **Different limits** حسب الـ endpoint والـ user tier

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 4: Input Validation](./04-input-validation.md)**

</div>

---

<div align="center">

[⬅️ Previous: CORS](./02-cors.md) | [📚 Module Home](../README.md) | [➡️ Next: Input Validation](./04-input-validation.md)

</div>
