# Connection Strings 🔗

<div dir="rtl">

## مقدمة

**Connection String** هو النص اللي بيحتوي كل المعلومات اللازمة للاتصال بالـ Database. فهمه مهم جداً لأنك هتستخدمه في كل تطبيق بيتصل بـ PostgreSQL.

**المدة المتوقعة:** 10-15 دقيقة

</div>

---

## 📝 الصيغة الأساسية

<div dir="rtl">

### URI Format (الأكثر شيوعاً)

</div>

```
postgresql://[user[:password]@][host][:port][/database][?parameters]
```

<div dir="rtl">

### تفصيل المكونات

</div>

```
postgresql://postgres:mypassword@localhost:5432/myapp?sslmode=disable
    │           │        │          │      │    │        │
    │           │        │          │      │    │        └─ Parameters
    │           │        │          │      │    └────────── Database
    │           │        │          │      └─────────────── Port
    │           │        │          └────────────────────── Host
    │           │        └───────────────────────────────── Password
    │           └────────────────────────────────────────── User
    └────────────────────────────────────────────────────── Protocol
```

---

## 🔧 المكونات بالتفصيل

<div dir="rtl">

### 1. Protocol

</div>

```
postgresql://    ← الصيغة الكاملة
postgres://      ← اختصار (نفس الشيء)
```

<div dir="rtl">

### 2. User (اسم المستخدم)

</div>

```
postgresql://postgres@...        ← اسم فقط
postgresql://my_user@...         ← اسم مخصص
```

<div dir="rtl">

### 3. Password (كلمة السر)

</div>

```
postgresql://postgres:password@...      ← عادي
postgresql://postgres:p%40ss@...        ← URL Encoded (@ = %40)
postgresql://postgres:pass%23word@...   ← URL Encoded (# = %23)
```

<div dir="rtl">

**ملاحظة مهمة:** لو الـ Password فيها أحرف خاصة، لازم تعمل URL Encoding:

| الرمز | Encoded |
|-------|---------|
| `@` | `%40` |
| `#` | `%23` |
| `/` | `%2F` |
| `:` | `%3A` |
| `%` | `%25` |
| ` ` (مسافة) | `%20` |

### 4. Host

</div>

```
postgresql://...@localhost/...          ← Local machine
postgresql://...@127.0.0.1/...          ← Same (IP)
postgresql://...@db.example.com/...     ← Remote server
postgresql://...@192.168.1.100/...      ← Remote IP
```

<div dir="rtl">

### 5. Port

</div>

```
postgresql://...@localhost:5432/...     ← Default port
postgresql://...@localhost:5433/...     ← Custom port
postgresql://...@localhost/...          ← Default (5432) لو مش موجود
```

<div dir="rtl">

### 6. Database

</div>

```
postgresql://...@localhost/myapp        ← اسم الـ Database
postgresql://...@localhost/postgres     ← Database الافتراضية
```

<div dir="rtl">

### 7. Parameters (اختيارية)

</div>

```
?sslmode=disable                        ← Parameter واحد
?sslmode=require&connect_timeout=10     ← Parameters متعددة
```

---

## 🔐 SSL/TLS Parameters

<div dir="rtl">

### sslmode Options

| القيمة | الوصف | الاستخدام |
|--------|-------|----------|
| `disable` | بدون SSL | Development محلي |
| `allow` | يحاول SSL، لو فشل يكمل بدونه | |
| `prefer` | يفضل SSL، لو فشل يكمل بدونه | Default |
| `require` | يتطلب SSL، بدون تحقق من الشهادة | Production |
| `verify-ca` | يتطلب SSL مع تحقق من CA | Secure |
| `verify-full` | يتطلب SSL مع تحقق كامل | Most Secure |

</div>

```
# Development
?sslmode=disable

# Production
?sslmode=require

# Highly Secure
?sslmode=verify-full&sslrootcert=/path/to/ca.crt
```

---

## ⏱️ Parameters مهمة أخرى

<div dir="rtl">

### Connection Timeout

</div>

```
?connect_timeout=10     ← 10 ثواني للاتصال
```

<div dir="rtl">

### Application Name

</div>

```
?application_name=myapp     ← مفيد للـ Monitoring
```

<div dir="rtl">

### Search Path (Schema)

</div>

```
?search_path=myschema      ← تحديد الـ Schema الافتراضي
```

<div dir="rtl">

### Timezone

</div>

```
?timezone=UTC              ← أو Africa/Cairo
```

---

## 📋 أمثلة كاملة

<div dir="rtl">

### 1. Development (محلي)

</div>

```
postgresql://postgres:postgres123@localhost:5432/myapp?sslmode=disable
```

<div dir="rtl">

### 2. Production (سيرفر)

</div>

```
postgresql://app_user:SecureP%40ss@db.example.com:5432/production_db?sslmode=require&connect_timeout=10
```

<div dir="rtl">

### 3. Docker

</div>

```
postgresql://postgres:postgres@postgres:5432/myapp?sslmode=disable
                                  │
                                  └── اسم الـ container
```

<div dir="rtl">

### 4. Cloud Providers

</div>

```
# AWS RDS
postgresql://admin:password@mydb.abc123.us-east-1.rds.amazonaws.com:5432/myapp?sslmode=require

# Google Cloud SQL
postgresql://user:password@/mydb?host=/cloudsql/project:region:instance

# Heroku
postgresql://user:pass@ec2-12-345-67-890.compute-1.amazonaws.com:5432/dbname

# Supabase
postgresql://postgres:password@db.xxxx.supabase.co:5432/postgres

# Neon
postgresql://user:password@ep-cool-name-123456.us-east-2.aws.neon.tech/mydb?sslmode=require
```

---

## 🔒 الأمان وأفضل الممارسات

<div dir="rtl">

### ❌ لا تفعل هذا

</div>

```go
// ❌ وحش - Connection String في الكود
db, _ := sql.Open("postgres",
    "postgresql://postgres:MySecretPass@localhost/myapp")

// ❌ وحش - في Commit
const dbURL = "postgresql://admin:SuperSecret123@prod.example.com/app"
```

<div dir="rtl">

### ✅ افعل هذا

</div>

```go
// ✅ كويس - من Environment Variable
dbURL := os.Getenv("DATABASE_URL")
db, err := sql.Open("postgres", dbURL)

// ✅ كويس - بناء الـ URL من متغيرات منفصلة
host := os.Getenv("DB_HOST")
port := os.Getenv("DB_PORT")
user := os.Getenv("DB_USER")
pass := os.Getenv("DB_PASSWORD")
dbname := os.Getenv("DB_NAME")

dbURL := fmt.Sprintf("postgresql://%s:%s@%s:%s/%s?sslmode=disable",
    user, url.QueryEscape(pass), host, port, dbname)
```

<div dir="rtl">

### ملف .env

</div>

```bash
# .env (لا تعمل commit لهذا الملف!)
DATABASE_URL=postgresql://postgres:password@localhost:5432/myapp?sslmode=disable

# أو منفصلين
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=password
DB_NAME=myapp
DB_SSLMODE=disable
```

<div dir="rtl">

### .gitignore

</div>

```gitignore
# Never commit these!
.env
.env.local
.env.production
*.pem
```

---

## 🔄 الصيغة البديلة (Key-Value)

<div dir="rtl">

بعض الأدوات تستخدم صيغة key=value بدل URI:

</div>

```
host=localhost port=5432 user=postgres password=secret dbname=myapp sslmode=disable
```

<div dir="rtl">

### مقارنة

</div>

```
# URI Format
postgresql://postgres:secret@localhost:5432/myapp?sslmode=disable

# Key-Value Format
host=localhost port=5432 user=postgres password=secret dbname=myapp sslmode=disable
```

<div dir="rtl">

### في psql

</div>

```bash
# URI
psql "postgresql://postgres:password@localhost/myapp"

# Key-Value
psql "host=localhost user=postgres password=password dbname=myapp"
```

---

## 💻 استخدام Connection String في Go

<div dir="rtl">

### مع database/sql

</div>

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "os"

    _ "github.com/lib/pq"
)

func main() {
    // من Environment Variable
    dbURL := os.Getenv("DATABASE_URL")
    if dbURL == "" {
        // Fallback للـ Development
        dbURL = "postgresql://postgres:postgres@localhost:5432/myapp?sslmode=disable"
    }

    db, err := sql.Open("postgres", dbURL)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Test Connection
    err = db.Ping()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("Connected successfully!")
}
```

<div dir="rtl">

### مع pgx (أسرع)

</div>

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"

    "github.com/jackc/pgx/v5"
)

func main() {
    dbURL := os.Getenv("DATABASE_URL")

    conn, err := pgx.Connect(context.Background(), dbURL)
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close(context.Background())

    var greeting string
    err = conn.QueryRow(context.Background(), "SELECT 'Hello, World!'").Scan(&greeting)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(greeting)
}
```

<div dir="rtl">

### مع GORM

</div>

```go
package main

import (
    "log"
    "os"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func main() {
    dsn := os.Getenv("DATABASE_URL")

    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }

    // Use db...
    sqlDB, _ := db.DB()
    defer sqlDB.Close()
}
```

---

## 🧪 اختبار الاتصال

<div dir="rtl">

### من Command Line

</div>

```bash
# اختبار بسيط
psql "postgresql://postgres:password@localhost:5432/myapp" -c "SELECT 1"

# مع timeout
psql "postgresql://postgres:password@localhost:5432/myapp?connect_timeout=5" -c "SELECT 1"
```

<div dir="rtl">

### من Go

</div>

```go
func testConnection(dbURL string) error {
    db, err := sql.Open("postgres", dbURL)
    if err != nil {
        return fmt.Errorf("failed to open: %w", err)
    }
    defer db.Close()

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    if err := db.PingContext(ctx); err != nil {
        return fmt.Errorf("failed to ping: %w", err)
    }

    return nil
}
```

---

## 📋 Quick Reference

<div dir="rtl">

### الصيغة الكاملة

</div>

```
postgresql://user:password@host:port/database?param1=value1&param2=value2
```

<div dir="rtl">

### Parameters الشائعة

</div>

| Parameter | القيم | الافتراضي |
|-----------|-------|-----------|
| `sslmode` | disable, require, verify-full | prefer |
| `connect_timeout` | ثواني | 0 (لا نهائي) |
| `application_name` | string | - |
| `search_path` | schema names | public |
| `timezone` | timezone name | server default |

<div dir="rtl">

### أمثلة سريعة

</div>

```bash
# Development
postgresql://postgres:postgres@localhost/myapp?sslmode=disable

# Docker
postgresql://postgres:postgres@db/myapp

# Production
postgresql://app:SecurePass@prod-db.com/myapp?sslmode=require

# With timeout
postgresql://postgres:pass@localhost/myapp?sslmode=disable&connect_timeout=10
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ فهمت صيغة Connection String
- [ ] ✅ عرفت كل المكونات
- [ ] ✅ عرفت الـ SSL modes
- [ ] ✅ عرفت تستخدم Environment Variables
- [ ] ✅ عرفت تكتب Connection String صحيح

</div>

---

## ⏭️ Module التالي

<div dir="rtl">

مبروك! أنهيت Module 4.1: Installation & Setup!

**➡️ [Module 4.2: SQL Basics](../02-sql-basics/README.md)**

</div>

---

<div align="center">

[⬅️ السابق: إنشاء أول Database](./07-first-database.md) | [🏠 العودة للـ Module](../README.md)

</div>
