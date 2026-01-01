# Connection Pooling - إدارة الـ Connections 🏊

<div dir="rtl">

## مقدمة

الـ Connection Pooling هو إدارة مجموعة من الاتصالات المفتوحة مسبقاً لإعادة استخدامها بدلاً من فتح اتصال جديد لكل request. ده مهم جداً للـ performance.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 لماذا Connection Pooling؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Without Connection Pool                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Request 1 ──► Open Connection ──► Query ──► Close Connection      │
│   Request 2 ──► Open Connection ──► Query ──► Close Connection      │
│   Request 3 ──► Open Connection ──► Query ──► Close Connection      │
│                                                                      │
│   ⏱️ كل request: ~50-100ms لفتح الاتصال                             │
│   😰 Load على الـ database server                                   │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                    With Connection Pool                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────┐       │
│   │              Connection Pool                             │       │
│   │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                    │       │
│   │  │Conn│ │Conn│ │Conn│ │Conn│ │Conn│  Pre-opened!       │       │
│   │  └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘                    │       │
│   └─────┼──────┼──────┼──────┼──────┼───────────────────────┘       │
│         │      │      │      │      │                                │
│   Request 1    │      │      │      │                                │
│         ▼      │      │      │      │                                │
│   Get Conn ◄───┘      │      │      │                                │
│         │             │      │      │                                │
│       Query           │      │      │                                │
│         │             │      │      │                                │
│   Return Conn ───────►│      │      │                                │
│                       │      │      │                                │
│   ⚡ كل request: ~0-1ms للحصول على connection                       │
│   ✅ Connections معاد استخدامها                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 database/sql Pool

<div dir="rtl">

`database/sql` عنده built-in connection pool:

</div>

```go
package main

import (
    "database/sql"
    "time"

    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    db, _ := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb")
    defer db.Close()

    // إعدادات الـ Pool
    db.SetMaxOpenConns(25)           // أقصى عدد connections مفتوحة
    db.SetMaxIdleConns(5)            // أقصى عدد connections idle
    db.SetConnMaxLifetime(time.Hour) // أقصى عمر للـ connection
    db.SetConnMaxIdleTime(30 * time.Minute) // أقصى وقت idle

    // استخدم db بشكل عادي
    // الـ pool بيدير الـ connections تلقائياً
}
```

---

## 📊 pgxpool Configuration

```go
package main

import (
    "context"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

func createPool() (*pgxpool.Pool, error) {
    config, err := pgxpool.ParseConfig("postgres://postgres:password@localhost:5432/mydb")
    if err != nil {
        return nil, err
    }

    // Pool Settings
    config.MaxConns = 25                      // أقصى عدد connections
    config.MinConns = 5                       // أقل عدد connections (محجوزة دايماً)
    config.MaxConnLifetime = time.Hour        // أقصى عمر الـ connection
    config.MaxConnIdleTime = 30 * time.Minute // أقصى وقت idle قبل الإغلاق
    config.HealthCheckPeriod = time.Minute    // فحص صحة الـ connections

    // Connection Settings
    config.ConnConfig.ConnectTimeout = 5 * time.Second

    return pgxpool.NewWithConfig(context.Background(), config)
}
```

---

## 📈 اختيار الأرقام الصحيحة

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Pool Sizing Guidelines                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MaxConns Formula:                                                   │
│  ─────────────────                                                   │
│  MaxConns = (CPU cores × 2) + Effective spindle count               │
│                                                                      │
│  For SSD (no spindle): MaxConns ≈ CPU cores × 2 + 1                 │
│                                                                      │
│  Examples:                                                           │
│  ├── 4 CPU cores + SSD  → MaxConns = 9-10                           │
│  ├── 8 CPU cores + SSD  → MaxConns = 17-20                          │
│  └── 16 CPU cores + SSD → MaxConns = 33-40                          │
│                                                                      │
│  MinConns:                                                           │
│  ─────────                                                           │
│  MinConns = Expected baseline concurrent queries                     │
│  Usually: 10-20% of MaxConns                                        │
│                                                                      │
│  للـ Web Application (typical):                                      │
│  ├── MaxConns: 20-50                                                 │
│  ├── MinConns: 5-10                                                  │
│  └── MaxIdleTime: 5-30 minutes                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 مراقبة الـ Pool

```go
package main

import (
    "context"
    "fmt"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

func monitorPool(pool *pgxpool.Pool) {
    ticker := time.NewTicker(10 * time.Second)
    for range ticker.C {
        stat := pool.Stat()

        fmt.Printf(`
Pool Stats:
  Total Connections:    %d
  Acquired Connections: %d
  Idle Connections:     %d
  Max Connections:      %d
  Empty Acquires:       %d
  Canceled Acquires:    %d
`,
            stat.TotalConns(),
            stat.AcquiredConns(),
            stat.IdleConns(),
            stat.MaxConns(),
            stat.EmptyAcquireCount(),
            stat.CanceledAcquireCount(),
        )
    }
}

func main() {
    pool, _ := pgxpool.New(context.Background(), "postgres://...")
    defer pool.Close()

    // مراقبة في goroutine منفصل
    go monitorPool(pool)

    // استخدم الـ pool
    // ...
}
```

---

## ⚠️ مشاكل شائعة

<div dir="rtl">

### 1. Connection Leak

</div>

```go
// ❌ غلط - Connection leak!
func getUsers(pool *pgxpool.Pool) ([]User, error) {
    rows, err := pool.Query(context.Background(), "SELECT * FROM users")
    if err != nil {
        return nil, err
    }
    // نسيت rows.Close()!

    var users []User
    for rows.Next() {
        // ...
    }
    return users, nil
}

// ✅ صح
func getUsers(pool *pgxpool.Pool) ([]User, error) {
    rows, err := pool.Query(context.Background(), "SELECT * FROM users")
    if err != nil {
        return nil, err
    }
    defer rows.Close() // دايماً!

    var users []User
    for rows.Next() {
        // ...
    }
    return users, rows.Err()
}
```

<div dir="rtl">

### 2. Connection Exhaustion

</div>

```go
// ❌ غلط - ممكن تخلص الـ connections!
func processAll(pool *pgxpool.Pool, items []int) {
    for _, item := range items {
        go func(i int) {
            // كل goroutine بتاخد connection
            pool.QueryRow(context.Background(), "SELECT * FROM items WHERE id = $1", i)
        }(item)
    }
}

// ✅ صح - استخدم semaphore أو worker pool
func processAll(pool *pgxpool.Pool, items []int) {
    // حدد عدد الـ concurrent operations
    sem := make(chan struct{}, 10)

    for _, item := range items {
        sem <- struct{}{} // acquire
        go func(i int) {
            defer func() { <-sem }() // release
            pool.QueryRow(context.Background(), "SELECT * FROM items WHERE id = $1", i)
        }(item)
    }
}
```

<div dir="rtl">

### 3. Long-Running Transactions

</div>

```go
// ❌ غلط - Transaction طويلة بتحجز connection
func processLargeData(pool *pgxpool.Pool) error {
    tx, _ := pool.Begin(context.Background())
    defer tx.Rollback(context.Background())

    // عملية طويلة جداً
    time.Sleep(5 * time.Minute)

    return tx.Commit(context.Background())
}

// ✅ صح - قسم العملية لـ batches
func processLargeData(pool *pgxpool.Pool) error {
    batchSize := 1000
    offset := 0

    for {
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)

        tx, _ := pool.Begin(ctx)
        // عالج batch صغير
        result, _ := tx.Exec(ctx,
            "UPDATE items SET processed = true LIMIT $1 OFFSET $2",
            batchSize, offset)
        tx.Commit(ctx)
        cancel()

        if result.RowsAffected() == 0 {
            break
        }
        offset += batchSize
    }
    return nil
}
```

---

## 🔧 Connection String Best Practices

```go
package main

import (
    "fmt"
    "os"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

// Config من Environment Variables
type DBConfig struct {
    Host         string
    Port         string
    User         string
    Password     string
    Database     string
    SSLMode      string
    MaxConns     int
    MinConns     int
    MaxLifetime  time.Duration
}

func loadConfig() DBConfig {
    return DBConfig{
        Host:        getEnv("DB_HOST", "localhost"),
        Port:        getEnv("DB_PORT", "5432"),
        User:        getEnv("DB_USER", "postgres"),
        Password:    os.Getenv("DB_PASSWORD"), // لا default للـ password!
        Database:    getEnv("DB_NAME", "myapp"),
        SSLMode:     getEnv("DB_SSLMODE", "disable"),
        MaxConns:    getEnvInt("DB_MAX_CONNS", 25),
        MinConns:    getEnvInt("DB_MIN_CONNS", 5),
        MaxLifetime: getEnvDuration("DB_MAX_LIFETIME", time.Hour),
    }
}

func (c DBConfig) ConnectionString() string {
    return fmt.Sprintf(
        "postgres://%s:%s@%s:%s/%s?sslmode=%s",
        c.User, c.Password, c.Host, c.Port, c.Database, c.SSLMode,
    )
}

func getEnv(key, defaultVal string) string {
    if val := os.Getenv(key); val != "" {
        return val
    }
    return defaultVal
}
```

---

## 📊 Healthcheck

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

func healthCheck(pool *pgxpool.Pool) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
        defer cancel()

        // Check 1: Can we ping?
        if err := pool.Ping(ctx); err != nil {
            w.WriteHeader(http.StatusServiceUnavailable)
            fmt.Fprintf(w, "Database ping failed: %v", err)
            return
        }

        // Check 2: Pool stats
        stat := pool.Stat()
        if stat.AcquiredConns() >= stat.MaxConns() {
            w.WriteHeader(http.StatusServiceUnavailable)
            fmt.Fprintf(w, "Connection pool exhausted")
            return
        }

        // Check 3: Simple query
        var result int
        err := pool.QueryRow(ctx, "SELECT 1").Scan(&result)
        if err != nil {
            w.WriteHeader(http.StatusServiceUnavailable)
            fmt.Fprintf(w, "Database query failed: %v", err)
            return
        }

        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, `{"status": "healthy", "connections": {"total": %d, "idle": %d}}`,
            stat.TotalConns(), stat.IdleConns())
    }
}
```

---

## 🏭 Production Configuration

```go
package main

import (
    "context"
    "log"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

func NewProductionPool(connString string) (*pgxpool.Pool, error) {
    config, err := pgxpool.ParseConfig(connString)
    if err != nil {
        return nil, err
    }

    // Pool Configuration
    config.MaxConns = 25
    config.MinConns = 5
    config.MaxConnLifetime = time.Hour
    config.MaxConnIdleTime = 30 * time.Minute
    config.HealthCheckPeriod = time.Minute

    // Connection Configuration
    config.ConnConfig.ConnectTimeout = 5 * time.Second

    // Tracing/Logging (optional)
    config.BeforeAcquire = func(ctx context.Context, conn *pgx.Conn) bool {
        log.Println("Acquiring connection from pool")
        return true // return false to reject
    }

    config.AfterRelease = func(conn *pgx.Conn) bool {
        log.Println("Releasing connection to pool")
        return true // return false to close instead
    }

    config.BeforeClose = func(conn *pgx.Conn) {
        log.Println("Closing connection")
    }

    return pgxpool.NewWithConfig(context.Background(), config)
}
```

---

## 💡 Best Practices Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Connection Pooling Best Practices                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Size the pool correctly                                          │
│     └── MaxConns ≈ (CPU × 2) + 1 for SSD                            │
│                                                                      │
│  2. Always close/release connections                                 │
│     └── defer rows.Close()                                          │
│     └── defer tx.Rollback()                                         │
│                                                                      │
│  3. Use Context with timeout                                         │
│     └── context.WithTimeout(ctx, 5*time.Second)                     │
│                                                                      │
│  4. Monitor pool stats                                               │
│     └── pool.Stat() في metrics/healthcheck                          │
│                                                                      │
│  5. Keep transactions short                                          │
│     └── تجنب long-running transactions                              │
│                                                                      │
│  6. Handle connection errors gracefully                              │
│     └── Retry logic مع backoff                                      │
│                                                                      │
│  7. Use environment variables for config                             │
│     └── DB_HOST, DB_PASSWORD, etc.                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Connection Pooling** ضروري للـ production
2. **MaxConns** ≈ (CPU × 2) + 1
3. دايماً **Close** الـ rows والـ transactions
4. استخدم **Context** مع timeout
5. راقب **Pool Stats** في الـ healthcheck
6. خلي الـ **Transactions قصيرة**

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Basic CRUD](./04-basic-crud.md)**

</div>

---

<div align="center">

[⬅️ السابق](./02-pgx-driver.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./04-basic-crud.md)

</div>
