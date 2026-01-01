# database/sql Basics - مقدمة في database/sql 📦

<div dir="rtl">

## مقدمة

`database/sql` هي الـ package الأساسية في Go للتعامل مع قواعد البيانات. بتوفر interface موحد للتعامل مع أي database سواء PostgreSQL أو MySQL أو SQLite.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 ما هو database/sql؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                     database/sql Architecture                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    Your Go Application                       │   │
│   └───────────────────────────┬─────────────────────────────────┘   │
│                               │                                      │
│                               ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                   database/sql Package                       │   │
│   │  ┌─────────────────────────────────────────────────────┐    │   │
│   │  │  - sql.DB (connection pool)                          │    │   │
│   │  │  - sql.Tx (transaction)                              │    │   │
│   │  │  - sql.Rows (query results)                          │    │   │
│   │  │  - sql.Stmt (prepared statement)                     │    │   │
│   │  └─────────────────────────────────────────────────────┘    │   │
│   └───────────────────────────┬─────────────────────────────────┘   │
│                               │                                      │
│                        Driver Interface                              │
│                               │                                      │
│   ┌───────────┬───────────────┼───────────────┬───────────────┐     │
│   │           │               │               │               │     │
│   ▼           ▼               ▼               ▼               ▼     │
│ ┌─────┐   ┌───────┐      ┌─────────┐     ┌──────┐      ┌─────────┐ │
│ │ pgx │   │lib/pq │      │go-mysql │     │sqlite│      │ oracle  │ │
│ └──┬──┘   └───┬───┘      └────┬────┘     └──┬───┘      └────┬────┘ │
│    │          │               │             │               │       │
└────┼──────────┼───────────────┼─────────────┼───────────────┼───────┘
     │          │               │             │               │
     ▼          ▼               ▼             ▼               ▼
 PostgreSQL  PostgreSQL      MySQL        SQLite          Oracle
```

---

## 🔧 التثبيت

```bash
# إنشاء مشروع جديد
mkdir go-postgres-demo
cd go-postgres-demo
go mod init go-postgres-demo

# تثبيت pgx driver (الأفضل لـ PostgreSQL)
go get github.com/jackc/pgx/v5/stdlib

# أو lib/pq (القديم لكن مستقر)
go get github.com/lib/pq
```

---

## 📝 الاتصال الأساسي

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    // Import الـ driver - الـ underscore مهم!
    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    // Connection string (DSN)
    dsn := "postgres://postgres:password@localhost:5432/mydb?sslmode=disable"

    // فتح الاتصال
    // ملاحظة: sql.Open لا يتصل فعلياً بالـ database
    // هو بس بيجهز الـ connection pool
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        log.Fatal("Error opening database:", err)
    }
    defer db.Close()

    // اختبار الاتصال الفعلي
    if err := db.Ping(); err != nil {
        log.Fatal("Error connecting to database:", err)
    }

    fmt.Println("✅ Connected to PostgreSQL!")
}
```

---

## 🔐 Connection String Formats

```go
// Format 1: URL Style (الأكثر شيوعاً)
dsn := "postgres://user:password@host:port/dbname?sslmode=disable"

// Format 2: Key-Value Style
dsn := "host=localhost port=5432 user=postgres password=secret dbname=mydb sslmode=disable"

// Format 3: مع SSL
dsn := "postgres://user:password@host:port/dbname?sslmode=require"

// Format 4: من Environment Variables (الأفضل للـ production)
dsn := os.Getenv("DATABASE_URL")
```

<div dir="rtl">

### SSL Modes

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SSL Modes                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  disable     │ بدون SSL (للـ development فقط!)                      │
│  allow       │ يحاول SSL أولاً، لو فشل يستخدم بدون                  │
│  prefer      │ يحاول SSL أولاً (default)                            │
│  require     │ لازم SSL، بدون التحقق من الشهادة                     │
│  verify-ca   │ لازم SSL + التحقق من CA                              │
│  verify-full │ لازم SSL + التحقق الكامل (production)                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 الأنواع الأساسية في database/sql

```go
// sql.DB - الـ connection pool
// مش اتصال واحد، ده pool من الاتصالات
var db *sql.DB

// sql.Rows - نتائج query متعددة
// لازم تقفله بـ Close()
var rows *sql.Rows

// sql.Row - صف واحد
// مش محتاج Close()
var row *sql.Row

// sql.Tx - Transaction
var tx *sql.Tx

// sql.Stmt - Prepared Statement
var stmt *sql.Stmt

// sql.Result - نتيجة INSERT/UPDATE/DELETE
var result sql.Result
```

---

## 🔍 Query Methods

```go
// 1. Query - لإرجاع صفوف متعددة
rows, err := db.Query("SELECT id, name FROM users")

// 2. QueryRow - لإرجاع صف واحد
row := db.QueryRow("SELECT name FROM users WHERE id = $1", 1)

// 3. Exec - لـ INSERT/UPDATE/DELETE
result, err := db.Exec("INSERT INTO users (name) VALUES ($1)", "Ahmed")

// 4. QueryContext / ExecContext - مع Context (للـ timeout)
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
rows, err := db.QueryContext(ctx, "SELECT * FROM users")
```

---

## 📖 مثال كامل: Query

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/jackc/pgx/v5/stdlib"
)

type User struct {
    ID    int
    Name  string
    Email string
}

func main() {
    db, err := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Query متعدد الصفوف
    rows, err := db.Query("SELECT id, name, email FROM users WHERE id > $1", 0)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close() // مهم جداً!

    var users []User
    for rows.Next() {
        var u User
        // Scan بينقل القيم من الصف للـ struct
        if err := rows.Scan(&u.ID, &u.Name, &u.Email); err != nil {
            log.Fatal(err)
        }
        users = append(users, u)
    }

    // التحقق من أخطاء أثناء الـ iteration
    if err := rows.Err(); err != nil {
        log.Fatal(err)
    }

    for _, u := range users {
        fmt.Printf("User: %d - %s (%s)\n", u.ID, u.Name, u.Email)
    }
}
```

---

## 📖 مثال كامل: QueryRow

```go
func getUserByID(db *sql.DB, id int) (*User, error) {
    var u User

    err := db.QueryRow(
        "SELECT id, name, email FROM users WHERE id = $1",
        id,
    ).Scan(&u.ID, &u.Name, &u.Email)

    if err != nil {
        if err == sql.ErrNoRows {
            // مفيش نتيجة - مش error فعلي
            return nil, nil
        }
        return nil, err
    }

    return &u, nil
}

func main() {
    // ...

    user, err := getUserByID(db, 1)
    if err != nil {
        log.Fatal(err)
    }

    if user == nil {
        fmt.Println("User not found")
    } else {
        fmt.Printf("Found: %+v\n", user)
    }
}
```

---

## ✏️ مثال كامل: Exec

```go
func createUser(db *sql.DB, name, email string) (int64, error) {
    result, err := db.Exec(
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        name, email,
    )
    if err != nil {
        return 0, err
    }

    // LastInsertId مش مدعوم في PostgreSQL!
    // استخدم RETURNING بدلاً منه
    id, err := result.LastInsertId()
    if err != nil {
        return 0, err // هيرجع error في PostgreSQL
    }

    return id, nil
}

// الطريقة الصحيحة في PostgreSQL
func createUserReturning(db *sql.DB, name, email string) (int, error) {
    var id int
    err := db.QueryRow(
        "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id",
        name, email,
    ).Scan(&id)

    return id, err
}

func updateUser(db *sql.DB, id int, name string) error {
    result, err := db.Exec(
        "UPDATE users SET name = $1 WHERE id = $2",
        name, id,
    )
    if err != nil {
        return err
    }

    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("user %d not found", id)
    }

    return nil
}

func deleteUser(db *sql.DB, id int) error {
    result, err := db.Exec("DELETE FROM users WHERE id = $1", id)
    if err != nil {
        return err
    }

    rowsAffected, _ := result.RowsAffected()
    fmt.Printf("Deleted %d rows\n", rowsAffected)

    return nil
}
```

---

## ⚠️ الأخطاء الشائعة

<div dir="rtl">

### 1. نسيان Close()

</div>

```go
// ❌ غلط - memory leak!
rows, _ := db.Query("SELECT * FROM users")
for rows.Next() {
    // ...
}

// ✅ صح
rows, err := db.Query("SELECT * FROM users")
if err != nil {
    return err
}
defer rows.Close() // دايماً!

for rows.Next() {
    // ...
}
```

<div dir="rtl">

### 2. تجاهل rows.Err()

</div>

```go
// ❌ غلط - ممكن يكون فيه error
for rows.Next() {
    rows.Scan(&user)
}

// ✅ صح
for rows.Next() {
    if err := rows.Scan(&user); err != nil {
        return err
    }
}
if err := rows.Err(); err != nil {
    return err
}
```

<div dir="rtl">

### 3. استخدام Placeholders غلط

</div>

```go
// ❌ غلط - SQL Injection!
query := fmt.Sprintf("SELECT * FROM users WHERE name = '%s'", name)
db.Query(query)

// ✅ صح - Parameterized Query
db.Query("SELECT * FROM users WHERE name = $1", name)

// PostgreSQL: $1, $2, $3
// MySQL: ?, ?, ?
```

---

## 📊 Null Handling

```go
import "database/sql"

type User struct {
    ID       int
    Name     string
    Email    sql.NullString // ممكن يكون NULL
    Age      sql.NullInt64
    IsActive sql.NullBool
}

func getUser(db *sql.DB, id int) (*User, error) {
    var u User
    err := db.QueryRow(
        "SELECT id, name, email, age, is_active FROM users WHERE id = $1",
        id,
    ).Scan(&u.ID, &u.Name, &u.Email, &u.Age, &u.IsActive)

    if err != nil {
        return nil, err
    }

    return &u, nil
}

func main() {
    // ...
    user, _ := getUser(db, 1)

    // التحقق من NULL
    if user.Email.Valid {
        fmt.Println("Email:", user.Email.String)
    } else {
        fmt.Println("Email is NULL")
    }

    if user.Age.Valid {
        fmt.Println("Age:", user.Age.Int64)
    }
}
```

<div dir="rtl">

### بديل: استخدام Pointers

</div>

```go
type User struct {
    ID    int
    Name  string
    Email *string // nil لو NULL
    Age   *int
}

func getUser(db *sql.DB, id int) (*User, error) {
    var u User
    err := db.QueryRow(
        "SELECT id, name, email, age FROM users WHERE id = $1",
        id,
    ).Scan(&u.ID, &u.Name, &u.Email, &u.Age)

    return &u, err
}

func main() {
    user, _ := getUser(db, 1)

    if user.Email != nil {
        fmt.Println("Email:", *user.Email)
    }
}
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم Context

</div>

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

rows, err := db.QueryContext(ctx, "SELECT * FROM users")
```

<div dir="rtl">

### 2. sql.DB آمن للاستخدام المتزامن

</div>

```go
// ✅ صح - شارك sql.DB بين goroutines
var db *sql.DB // global أو في struct

func handler1() {
    db.Query("...")
}

func handler2() {
    db.Query("...")
}
```

<div dir="rtl">

### 3. استخدم Environment Variables

</div>

```go
func getDB() (*sql.DB, error) {
    dsn := os.Getenv("DATABASE_URL")
    if dsn == "" {
        return nil, fmt.Errorf("DATABASE_URL not set")
    }

    return sql.Open("pgx", dsn)
}
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **database/sql** هو الـ standard interface في Go
2. **sql.Open** لا يتصل فعلياً - استخدم **Ping**
3. دايماً **defer rows.Close()** بعد Query
4. استخدم **Parameterized Queries** - لا تبني SQL strings
5. **PostgreSQL placeholders**: $1, $2, $3
6. استخدم **Context** للـ timeout
7. تعامل مع **NULL** باستخدام sql.NullXxx أو pointers

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [pgx Driver](./02-pgx-driver.md)**

</div>

---

<div align="center">

[🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./02-pgx-driver.md)

</div>
