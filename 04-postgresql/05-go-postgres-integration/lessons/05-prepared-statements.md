# Prepared Statements - الـ Statements المجهزة 🎯

<div dir="rtl">

## مقدمة

الـ Prepared Statements بتسمح للـ database بتحليل وتخطيط الـ query مرة واحدة وتنفيذه عدة مرات. ده بيحسن الأداء والأمان.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 كيف يعمل Prepared Statement؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Without Prepared Statement                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Query 1: SELECT * FROM users WHERE id = 1                         │
│   ┌────────┐   ┌────────┐   ┌─────────┐   ┌──────────┐             │
│   │ Parse  │ → │ Plan   │ → │ Execute │ → │ Results  │             │
│   └────────┘   └────────┘   └─────────┘   └──────────┘             │
│                                                                      │
│   Query 2: SELECT * FROM users WHERE id = 2                         │
│   ┌────────┐   ┌────────┐   ┌─────────┐   ┌──────────┐             │
│   │ Parse  │ → │ Plan   │ → │ Execute │ → │ Results  │             │
│   └────────┘   └────────┘   └─────────┘   └──────────┘             │
│                                                                      │
│   ⏱️ Parse + Plan يتكرر كل مرة!                                     │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                    With Prepared Statement                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Prepare: SELECT * FROM users WHERE id = $1                        │
│   ┌────────┐   ┌────────┐                                           │
│   │ Parse  │ → │ Plan   │ → Stored!                                 │
│   └────────┘   └────────┘                                           │
│                                                                      │
│   Execute (id=1): ─────────→ ┌─────────┐ → Results                  │
│   Execute (id=2): ─────────→ │ Execute │ → Results                  │
│   Execute (id=3): ─────────→ └─────────┘ → Results                  │
│                                                                      │
│   ⚡ Parse + Plan مرة واحدة فقط!                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 استخدام database/sql

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "log"

    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    db, _ := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb")
    defer db.Close()

    // إعداد الـ Prepared Statement
    stmt, err := db.Prepare("SELECT id, username, email FROM users WHERE id = $1")
    if err != nil {
        log.Fatal(err)
    }
    defer stmt.Close() // مهم جداً!

    // استخدام الـ statement عدة مرات
    ids := []int{1, 2, 3, 4, 5}
    for _, id := range ids {
        var user struct {
            ID       int
            Username string
            Email    string
        }

        err := stmt.QueryRow(id).Scan(&user.ID, &user.Username, &user.Email)
        if err != nil {
            if err == sql.ErrNoRows {
                fmt.Printf("User %d not found\n", id)
                continue
            }
            log.Fatal(err)
        }

        fmt.Printf("User: %d - %s (%s)\n", user.ID, user.Username, user.Email)
    }
}
```

---

## 📝 Prepared Statement مع Context

```go
func getUsersWithPrepared(ctx context.Context, db *sql.DB, ids []int) ([]User, error) {
    // Prepare مع Context
    stmt, err := db.PrepareContext(ctx, `
        SELECT id, username, email, is_active
        FROM users
        WHERE id = $1
    `)
    if err != nil {
        return nil, fmt.Errorf("prepare failed: %w", err)
    }
    defer stmt.Close()

    users := make([]User, 0, len(ids))

    for _, id := range ids {
        var user User

        // Query مع Context
        err := stmt.QueryRowContext(ctx, id).Scan(
            &user.ID,
            &user.Username,
            &user.Email,
            &user.IsActive,
        )

        if err != nil {
            if err == sql.ErrNoRows {
                continue
            }
            return nil, fmt.Errorf("query failed for id %d: %w", id, err)
        }

        users = append(users, user)
    }

    return users, nil
}
```

---

## 🚀 pgx Prepared Statements

<div dir="rtl">

pgx بيعمل prepare تلقائي! لكن ممكن تعمله manually للتحكم:

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

    // الحصول على connection
    conn, err := pool.Acquire(ctx)
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Release()

    // Prepare manually
    _, err = conn.Conn().Prepare(ctx, "get_user_by_id",
        "SELECT id, username, email FROM users WHERE id = $1")
    if err != nil {
        log.Fatal(err)
    }

    // استخدام الـ prepared statement باسمه
    ids := []int{1, 2, 3}
    for _, id := range ids {
        var user struct {
            ID       int
            Username string
            Email    string
        }

        err := conn.QueryRow(ctx, "get_user_by_id", id).Scan(
            &user.ID,
            &user.Username,
            &user.Email,
        )

        if err != nil {
            if err == pgx.ErrNoRows {
                continue
            }
            log.Fatal(err)
        }

        fmt.Printf("User: %+v\n", user)
    }
}
```

---

## 📦 Multiple Prepared Statements

```go
type Queries struct {
    db           *sql.DB
    getUserByID  *sql.Stmt
    getUserByEmail *sql.Stmt
    createUser   *sql.Stmt
    updateUser   *sql.Stmt
    deleteUser   *sql.Stmt
}

func NewQueries(db *sql.DB) (*Queries, error) {
    q := &Queries{db: db}

    var err error

    // Prepare all statements
    q.getUserByID, err = db.Prepare(`
        SELECT id, username, email, is_active
        FROM users WHERE id = $1
    `)
    if err != nil {
        return nil, fmt.Errorf("prepare getUserByID: %w", err)
    }

    q.getUserByEmail, err = db.Prepare(`
        SELECT id, username, email, is_active
        FROM users WHERE email = $1
    `)
    if err != nil {
        return nil, fmt.Errorf("prepare getUserByEmail: %w", err)
    }

    q.createUser, err = db.Prepare(`
        INSERT INTO users (username, email, password_hash)
        VALUES ($1, $2, $3)
        RETURNING id
    `)
    if err != nil {
        return nil, fmt.Errorf("prepare createUser: %w", err)
    }

    q.updateUser, err = db.Prepare(`
        UPDATE users SET email = $1, updated_at = NOW()
        WHERE id = $2
    `)
    if err != nil {
        return nil, fmt.Errorf("prepare updateUser: %w", err)
    }

    q.deleteUser, err = db.Prepare(`
        DELETE FROM users WHERE id = $1
    `)
    if err != nil {
        return nil, fmt.Errorf("prepare deleteUser: %w", err)
    }

    return q, nil
}

// Close all statements
func (q *Queries) Close() error {
    stmts := []*sql.Stmt{
        q.getUserByID,
        q.getUserByEmail,
        q.createUser,
        q.updateUser,
        q.deleteUser,
    }

    var errs []error
    for _, stmt := range stmts {
        if stmt != nil {
            if err := stmt.Close(); err != nil {
                errs = append(errs, err)
            }
        }
    }

    if len(errs) > 0 {
        return fmt.Errorf("close errors: %v", errs)
    }
    return nil
}

// Usage methods
func (q *Queries) GetUserByID(ctx context.Context, id int) (*User, error) {
    var user User
    err := q.getUserByID.QueryRowContext(ctx, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.IsActive,
    )
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return &user, err
}

func (q *Queries) CreateUser(ctx context.Context, username, email, password string) (int, error) {
    var id int
    err := q.createUser.QueryRowContext(ctx, username, email, password).Scan(&id)
    return id, err
}
```

---

## 🔄 Prepared Statement في Transaction

```go
func transferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    ctx := context.Background()

    // بدء Transaction
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    // Prepare statements في الـ transaction
    deductStmt, err := tx.PrepareContext(ctx, `
        UPDATE accounts SET balance = balance - $1
        WHERE id = $2 AND balance >= $1
    `)
    if err != nil {
        return err
    }
    defer deductStmt.Close()

    addStmt, err := tx.PrepareContext(ctx, `
        UPDATE accounts SET balance = balance + $1
        WHERE id = $2
    `)
    if err != nil {
        return err
    }
    defer addStmt.Close()

    // خصم من المصدر
    result, err := deductStmt.ExecContext(ctx, amount, fromID)
    if err != nil {
        return err
    }

    rows, _ := result.RowsAffected()
    if rows == 0 {
        return fmt.Errorf("insufficient balance")
    }

    // إضافة للمستلم
    _, err = addStmt.ExecContext(ctx, amount, toID)
    if err != nil {
        return err
    }

    return tx.Commit()
}
```

---

## ⚠️ متى تستخدم Prepared Statements

```
┌─────────────────────────────────────────────────────────────────────┐
│                When to Use Prepared Statements                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ USE when:                                                        │
│  ─────────────                                                       │
│  • Same query executed many times with different values             │
│  • Batch processing (processing list of IDs)                        │
│  • Hot paths with high query frequency                              │
│  • Security is critical (SQL injection prevention)                  │
│                                                                      │
│  ❌ DON'T USE when:                                                  │
│  ─────────────────                                                   │
│  • Query executed only once                                          │
│  • Dynamic queries with varying structure                           │
│  • Using connection pooling with short-lived connections            │
│    (statement prepared on one connection may not work on another)   │
│                                                                      │
│  📝 Note on pgx:                                                     │
│  ─────────────────                                                   │
│  pgx automatically prepares and caches statements!                  │
│  Manual prepare is usually not needed with pgx.                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 مقارنة الأداء

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "time"

    _ "github.com/jackc/pgx/v5/stdlib"
)

func benchmarkWithoutPrepare(db *sql.DB, iterations int) time.Duration {
    ctx := context.Background()
    start := time.Now()

    for i := 0; i < iterations; i++ {
        var count int
        db.QueryRowContext(ctx, "SELECT COUNT(*) FROM users WHERE id > $1", i).Scan(&count)
    }

    return time.Since(start)
}

func benchmarkWithPrepare(db *sql.DB, iterations int) time.Duration {
    ctx := context.Background()

    stmt, _ := db.PrepareContext(ctx, "SELECT COUNT(*) FROM users WHERE id > $1")
    defer stmt.Close()

    start := time.Now()

    for i := 0; i < iterations; i++ {
        var count int
        stmt.QueryRowContext(ctx, i).Scan(&count)
    }

    return time.Since(start)
}

func main() {
    db, _ := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb")
    defer db.Close()

    iterations := 1000

    withoutPrepare := benchmarkWithoutPrepare(db, iterations)
    withPrepare := benchmarkWithPrepare(db, iterations)

    fmt.Printf("Without Prepare: %v\n", withoutPrepare)
    fmt.Printf("With Prepare:    %v\n", withPrepare)
    fmt.Printf("Improvement:     %.2fx faster\n", float64(withoutPrepare)/float64(withPrepare))
}

// Output (typical):
// Without Prepare: 2.5s
// With Prepare:    1.8s
// Improvement:     1.39x faster
```

---

## 🔒 Security Benefits

<div dir="rtl">

الـ Prepared Statements بتمنع SQL Injection تلقائياً:

</div>

```go
// ❌ خطر! SQL Injection ممكن
func unsafeSearch(db *sql.DB, username string) {
    // لو username = "'; DROP TABLE users; --"
    query := fmt.Sprintf("SELECT * FROM users WHERE username = '%s'", username)
    db.Query(query) // كارثة!
}

// ✅ آمن - Prepared Statement
func safeSearch(db *sql.DB, username string) {
    // القيمة بتتعامل كـ data مش SQL code
    stmt, _ := db.Prepare("SELECT * FROM users WHERE username = $1")
    defer stmt.Close()

    stmt.Query(username) // آمن!
}

// ✅ آمن - Parameterized Query (يعمل prepare داخلياً)
func alsoSafe(db *sql.DB, username string) {
    db.Query("SELECT * FROM users WHERE username = $1", username) // آمن!
}
```

---

## 💡 Best Practices

```go
// 1. دايماً Close الـ Prepared Statement
stmt, err := db.Prepare(query)
if err != nil {
    return err
}
defer stmt.Close() // ✅

// 2. تحقق من الأخطاء
stmt, err := db.Prepare(query)
if err != nil {
    return fmt.Errorf("prepare failed: %w", err)
}

// 3. استخدم Context
stmt, err := db.PrepareContext(ctx, query)

// 4. لا تستخدم Prepare لـ one-time queries
// ❌
stmt, _ := db.Prepare("SELECT * FROM users WHERE id = $1")
stmt.QueryRow(1)
stmt.Close()

// ✅
db.QueryRow("SELECT * FROM users WHERE id = $1", 1)

// 5. في pgx، اعتمد على الـ automatic caching
pool.Query(ctx, "SELECT * FROM users WHERE id = $1", id)
// pgx يعمل prepare وcache تلقائي!
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Prepared Statements** بتحسن الأداء عند تنفيذ نفس الـ query مرات كثيرة
2. بتوفر **حماية** من SQL Injection
3. دايماً **Close** الـ statement بعد الانتهاء
4. **pgx** بيعمل prepare تلقائي - ممكن ما تحتاج manual prepare
5. لا تستخدمها لـ **one-time queries**
6. استخدم **Context** للتحكم في الـ timeout

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Transactions in Go](./06-transactions.md)**

</div>

---

<div align="center">

[⬅️ السابق](./04-basic-crud.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./06-transactions.md)

</div>
