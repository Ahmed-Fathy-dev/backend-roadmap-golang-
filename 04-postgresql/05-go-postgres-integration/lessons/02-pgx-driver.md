# pgx Driver - استخدام pgx للـ PostgreSQL 🚀

<div dir="rtl">

## مقدمة

`pgx` هو الـ driver الأفضل والأسرع لـ PostgreSQL في Go. بيوفر ميزات PostgreSQL-specific زي COPY، Batch، والـ native types.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 pgx vs lib/pq

```
┌─────────────────────────────────────────────────────────────────────┐
│                      pgx vs lib/pq                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Feature                │    pgx        │    lib/pq                 │
│  ───────────────────────┼───────────────┼─────────────────────────  │
│  Performance            │    ⭐⭐⭐⭐⭐    │    ⭐⭐⭐                   │
│  PostgreSQL Features    │    ⭐⭐⭐⭐⭐    │    ⭐⭐⭐                   │
│  COPY Support           │    ✅ Native   │    ❌ Limited             │
│  Batch Queries          │    ✅ Yes      │    ❌ No                  │
│  Listen/Notify          │    ✅ Easy     │    ✅ Supported           │
│  Binary Protocol        │    ✅ Yes      │    ❌ Text only           │
│  Connection Pooling     │    ✅ pgxpool  │    ❌ External            │
│  Active Development     │    ✅ Active   │    ⚠️ Maintenance         │
│                                                                      │
│  Recommendation: استخدم pgx للمشاريع الجديدة                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 التثبيت

```bash
# pgx v5 (الأحدث)
go get github.com/jackc/pgx/v5

# للاستخدام مع database/sql
go get github.com/jackc/pgx/v5/stdlib

# pgxpool للـ connection pooling
go get github.com/jackc/pgx/v5/pgxpool
```

---

## 🔌 طرق الاتصال

<div dir="rtl">

### 1. مع database/sql (التوافقية)

</div>

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    // مثل أي driver تاني
    db, err := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // استخدم db.Query, db.Exec, etc.
    var version string
    db.QueryRow("SELECT version()").Scan(&version)
    fmt.Println("PostgreSQL:", version)
}
```

<div dir="rtl">

### 2. pgx Native (الأفضل للأداء)

</div>

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5"
)

func main() {
    ctx := context.Background()

    // اتصال مباشر بـ pgx
    conn, err := pgx.Connect(ctx, "postgres://postgres:password@localhost:5432/mydb")
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close(ctx)

    var greeting string
    err = conn.QueryRow(ctx, "SELECT 'Hello, pgx!'").Scan(&greeting)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(greeting)
}
```

<div dir="rtl">

### 3. pgxpool (للـ Production)

</div>

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()

    // Pool بيدير الـ connections تلقائياً
    pool, err := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    if err != nil {
        log.Fatal(err)
    }
    defer pool.Close()

    // استخدم الـ pool
    var count int
    err = pool.QueryRow(ctx, "SELECT COUNT(*) FROM users").Scan(&count)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Users count: %d\n", count)
}
```

---

## ⚙️ إعدادات pgxpool

```go
package main

import (
    "context"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

func createPool() (*pgxpool.Pool, error) {
    ctx := context.Background()

    // إعدادات متقدمة
    config, err := pgxpool.ParseConfig("postgres://postgres:password@localhost:5432/mydb")
    if err != nil {
        return nil, err
    }

    // Pool Configuration
    config.MaxConns = 25                          // أقصى عدد connections
    config.MinConns = 5                           // أقل عدد connections
    config.MaxConnLifetime = time.Hour            // عمر الـ connection
    config.MaxConnIdleTime = 30 * time.Minute     // وقت الـ idle
    config.HealthCheckPeriod = time.Minute        // فحص صحة الـ connections

    // Connection Configuration
    config.ConnConfig.ConnectTimeout = 5 * time.Second

    // إنشاء الـ pool
    return pgxpool.NewWithConfig(ctx, config)
}

func main() {
    pool, err := createPool()
    if err != nil {
        panic(err)
    }
    defer pool.Close()

    // استخدم الـ pool
    // pool.Query, pool.QueryRow, pool.Exec
}
```

---

## 📝 CRUD مع pgx

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

type User struct {
    ID        int
    Username  string
    Email     string
    CreatedAt time.Time
}

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // CREATE
    var id int
    err := pool.QueryRow(ctx,
        "INSERT INTO users (username, email) VALUES ($1, $2) RETURNING id",
        "ahmed", "ahmed@example.com",
    ).Scan(&id)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Created user ID: %d\n", id)

    // READ - صف واحد
    var user User
    err = pool.QueryRow(ctx,
        "SELECT id, username, email, created_at FROM users WHERE id = $1",
        id,
    ).Scan(&user.ID, &user.Username, &user.Email, &user.CreatedAt)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("User: %+v\n", user)

    // READ - صفوف متعددة
    rows, err := pool.Query(ctx, "SELECT id, username, email, created_at FROM users")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()

    var users []User
    for rows.Next() {
        var u User
        err := rows.Scan(&u.ID, &u.Username, &u.Email, &u.CreatedAt)
        if err != nil {
            log.Fatal(err)
        }
        users = append(users, u)
    }
    fmt.Printf("Total users: %d\n", len(users))

    // UPDATE
    tag, err := pool.Exec(ctx,
        "UPDATE users SET email = $1 WHERE id = $2",
        "new@example.com", id,
    )
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Updated %d rows\n", tag.RowsAffected())

    // DELETE
    tag, err = pool.Exec(ctx, "DELETE FROM users WHERE id = $1", id)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Deleted %d rows\n", tag.RowsAffected())
}
```

---

## 🚀 Batch Queries

<div dir="rtl">

الـ Batch بيسمح لك تنفذ queries متعددة في request واحد - أسرع بكتير!

</div>

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5"
    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // إنشاء Batch
    batch := &pgx.Batch{}

    // إضافة queries للـ batch
    batch.Queue("INSERT INTO users (username, email) VALUES ($1, $2) RETURNING id",
        "user1", "user1@example.com")
    batch.Queue("INSERT INTO users (username, email) VALUES ($1, $2) RETURNING id",
        "user2", "user2@example.com")
    batch.Queue("INSERT INTO users (username, email) VALUES ($1, $2) RETURNING id",
        "user3", "user3@example.com")
    batch.Queue("SELECT COUNT(*) FROM users")

    // تنفيذ الـ batch
    results := pool.SendBatch(ctx, batch)
    defer results.Close()

    // قراءة نتائج الـ inserts
    for i := 0; i < 3; i++ {
        var id int
        err := results.QueryRow().Scan(&id)
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("Created user %d\n", id)
    }

    // قراءة نتيجة الـ count
    var count int
    err := results.QueryRow().Scan(&count)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Total users: %d\n", count)
}
```

---

## 📦 COPY Protocol

<div dir="rtl">

الـ COPY Protocol هو أسرع طريقة لإدخال بيانات كثيرة - أسرع 100x من INSERT!

</div>

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5"
    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // البيانات اللي عايزين ندخلها
    users := [][]interface{}{
        {"user1", "user1@example.com"},
        {"user2", "user2@example.com"},
        {"user3", "user3@example.com"},
        {"user4", "user4@example.com"},
        {"user5", "user5@example.com"},
    }

    // COPY FROM
    copyCount, err := pool.CopyFrom(
        ctx,
        pgx.Identifier{"users"},       // اسم الجدول
        []string{"username", "email"}, // الأعمدة
        pgx.CopyFromRows(users),       // البيانات
    )
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Inserted %d rows using COPY\n", copyCount)
}
```

<div dir="rtl">

### مقارنة الأداء

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│              Performance: INSERT vs COPY (10,000 rows)               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Method                      │   Time        │   Relative           │
│  ────────────────────────────┼───────────────┼────────────────────  │
│  Single INSERTs              │   ~10 sec     │   100x slower        │
│  Batch INSERTs (100/batch)   │   ~1 sec      │   10x slower         │
│  COPY                        │   ~100 ms     │   Fastest! ⚡        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔔 Listen/Notify

<div dir="rtl">

PostgreSQL بيدعم Pub/Sub - تقدر تستقبل notifications في الـ Go app.

</div>

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // الحصول على connection للـ Listen
    conn, err := pool.Acquire(ctx)
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Release()

    // الاشتراك في channel
    _, err = conn.Exec(ctx, "LISTEN user_events")
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("Listening for notifications on 'user_events'...")

    // انتظار الـ notifications
    for {
        notification, err := conn.Conn().WaitForNotification(ctx)
        if err != nil {
            log.Fatal(err)
        }

        fmt.Printf("Received: Channel=%s, Payload=%s\n",
            notification.Channel, notification.Payload)
    }
}

// في terminal تاني:
// psql> NOTIFY user_events, 'New user created: ahmed';
```

---

## 📊 PostgreSQL Types

<div dir="rtl">

pgx بيدعم PostgreSQL types مباشرة:

</div>

```go
package main

import (
    "context"
    "fmt"
    "net"
    "time"

    "github.com/jackc/pgx/v5/pgtype"
    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // Arrays
    var tags []string
    pool.QueryRow(ctx, "SELECT ARRAY['go', 'postgres', 'api']").Scan(&tags)
    fmt.Println("Tags:", tags)

    // JSON/JSONB
    var metadata map[string]interface{}
    pool.QueryRow(ctx, `SELECT '{"key": "value"}'::jsonb`).Scan(&metadata)
    fmt.Println("Metadata:", metadata)

    // UUID
    var id pgtype.UUID
    pool.QueryRow(ctx, "SELECT gen_random_uuid()").Scan(&id)
    fmt.Printf("UUID: %x\n", id.Bytes)

    // INET/CIDR
    var ip net.IP
    pool.QueryRow(ctx, "SELECT '192.168.1.1'::inet").Scan(&ip)
    fmt.Println("IP:", ip)

    // Timestamp with timezone
    var ts time.Time
    pool.QueryRow(ctx, "SELECT NOW()").Scan(&ts)
    fmt.Println("Time:", ts)

    // Range types
    var dateRange pgtype.Range[pgtype.Date]
    pool.QueryRow(ctx, "SELECT '[2024-01-01, 2024-12-31)'::daterange").Scan(&dateRange)
    fmt.Printf("Date Range: %+v\n", dateRange)
}
```

---

## 🔒 Prepared Statements

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // pgx بيعمل prepare تلقائي!
    // لكن ممكن تعمله manually للـ performance

    conn, _ := pool.Acquire(ctx)
    defer conn.Release()

    // Prepare
    _, err := conn.Conn().Prepare(ctx, "get_user", "SELECT id, username FROM users WHERE id = $1")
    if err != nil {
        log.Fatal(err)
    }

    // استخدام الـ prepared statement
    var id int
    var username string
    err = conn.QueryRow(ctx, "get_user", 1).Scan(&id, &username)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("User: %d - %s\n", id, username)
}
```

---

## 📊 Row Helpers

<div dir="rtl">

pgx v5 بيوفر helpers مفيدة لتحويل الصفوف:

</div>

```go
package main

import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
    "github.com/jackc/pgx/v5/pgxpool"
)

type User struct {
    ID       int    `db:"id"`
    Username string `db:"username"`
    Email    string `db:"email"`
}

func main() {
    ctx := context.Background()
    pool, _ := pgxpool.New(ctx, "postgres://postgres:password@localhost:5432/mydb")
    defer pool.Close()

    // CollectRows - جمع كل الصفوف في slice
    rows, _ := pool.Query(ctx, "SELECT id, username, email FROM users")
    users, err := pgx.CollectRows(rows, pgx.RowToStructByName[User])
    if err != nil {
        panic(err)
    }
    fmt.Printf("Users: %+v\n", users)

    // CollectOneRow - صف واحد
    rows2, _ := pool.Query(ctx, "SELECT id, username, email FROM users WHERE id = $1", 1)
    user, err := pgx.CollectOneRow(rows2, pgx.RowToStructByName[User])
    if err != nil {
        panic(err)
    }
    fmt.Printf("User: %+v\n", user)
}
```

---

## 💡 Best Practices

```go
// 1. استخدم pgxpool في Production
pool, _ := pgxpool.New(ctx, connString)
defer pool.Close()

// 2. استخدم Context دايماً
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
pool.QueryRow(ctx, query)

// 3. Handle errors properly
rows, err := pool.Query(ctx, query)
if err != nil {
    return fmt.Errorf("query failed: %w", err)
}
defer rows.Close()

// 4. استخدم COPY للـ bulk inserts
pool.CopyFrom(ctx, pgx.Identifier{"table"}, columns, pgx.CopyFromRows(data))

// 5. استخدم Batch للـ multiple queries
batch := &pgx.Batch{}
batch.Queue(query1, args1...)
batch.Queue(query2, args2...)
results := pool.SendBatch(ctx, batch)
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **pgx** هو الأفضل لـ PostgreSQL في Go
2. استخدم **pgxpool** للـ connection pooling
3. **Batch** لتنفيذ queries متعددة في request واحد
4. **COPY** أسرع 100x من INSERT للـ bulk data
5. **Listen/Notify** للـ real-time notifications
6. pgx بيدعم **PostgreSQL types** مباشرة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Connection Pooling](./03-connection-pooling.md)**

</div>

---

<div align="center">

[⬅️ السابق](./01-database-sql.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./03-connection-pooling.md)

</div>
