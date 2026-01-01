# Database Migrations - إدارة الـ Schema 🔄

<div dir="rtl">

## مقدمة

الـ Migrations هي طريقة لإدارة تغييرات الـ database schema بشكل منظم وقابل للتتبع. بدل ما تغير الـ database manually، بتكتب migration files.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 لماذا الـ Migrations؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Without Migrations                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Developer A: "أضفت عمود جديد للـ users"                           │
│   Developer B: "إيه اسمه؟ إيه الـ type؟"                            │
│   Developer A: "مش فاكر بالظبط..."                                  │
│                                                                      │
│   Production: "الـ app crashed! missing column!"                    │
│                                                                      │
│   ❌ No version control                                              │
│   ❌ No history                                                      │
│   ❌ No rollback                                                     │
│   ❌ Team sync issues                                                │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                     With Migrations                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   migrations/                                                        │
│   ├── 001_create_users.sql                                          │
│   ├── 002_add_email_to_users.sql                                    │
│   ├── 003_create_orders.sql                                         │
│   └── 004_add_status_to_orders.sql                                  │
│                                                                      │
│   ✅ Version controlled                                              │
│   ✅ Full history                                                    │
│   ✅ Can rollback                                                    │
│   ✅ Team stays in sync                                              │
│   ✅ Repeatable deployments                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 طرق الـ Migrations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Migration Approaches                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. GORM AutoMigrate                                                │
│     ├── سهل للـ development                                         │
│     ├── لا يحذف أعمدة                                               │
│     └── ❌ مش مناسب للـ production                                  │
│                                                                      │
│  2. golang-migrate                                                  │
│     ├── SQL files                                                   │
│     ├── Up/Down migrations                                          │
│     └── ✅ Production ready                                          │
│                                                                      │
│  3. goose                                                           │
│     ├── SQL or Go migrations                                        │
│     ├── Up/Down migrations                                          │
│     └── ✅ Production ready                                          │
│                                                                      │
│  4. Atlas                                                           │
│     ├── Declarative schema                                          │
│     ├── Auto-generate migrations                                    │
│     └── ✅ Modern approach                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🏃 GORM AutoMigrate

<div dir="rtl">

للـ development فقط:

</div>

```go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

type User struct {
    ID       uint   `gorm:"primaryKey"`
    Username string `gorm:"uniqueIndex;size:50"`
    Email    string `gorm:"uniqueIndex;size:255"`
    Age      int    `gorm:"default:0"`
}

type Order struct {
    ID     uint   `gorm:"primaryKey"`
    UserID uint   `gorm:"index"`
    Total  float64
    Status string `gorm:"default:'pending'"`
}

func main() {
    db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})

    // AutoMigrate
    err := db.AutoMigrate(
        &User{},
        &Order{},
    )
    if err != nil {
        log.Fatal(err)
    }
}
```

<div dir="rtl">

### ⚠️ قيود AutoMigrate

</div>

```go
// ❌ لا يحذف أعمدة
// لو شلت عمود من الـ struct، الـ database مش هيتأثر

// ❌ لا يغير types بشكل آمن
// varchar(50) → varchar(100) ممكن يفشل

// ❌ لا يعمل down migration
// مفيش rollback

// ❌ لا يتعامل مع data migration
// نقل البيانات بين الأعمدة
```

---

## 📝 Manual SQL Migrations

<div dir="rtl">

إنشاء migrations يدوياً:

</div>

```
migrations/
├── 000001_create_users.up.sql
├── 000001_create_users.down.sql
├── 000002_create_orders.up.sql
├── 000002_create_orders.down.sql
├── 000003_add_phone_to_users.up.sql
└── 000003_add_phone_to_users.down.sql
```

**000001_create_users.up.sql**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
```

**000001_create_users.down.sql**
```sql
DROP TABLE IF EXISTS users;
```

**000002_create_orders.up.sql**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    status VARCHAR(20) DEFAULT 'pending',
    total_amount DECIMAL(10, 2) DEFAULT 0,
    shipping_address TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
```

**000002_create_orders.down.sql**
```sql
DROP TABLE IF EXISTS orders;
```

**000003_add_phone_to_users.up.sql**
```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
CREATE INDEX idx_users_phone ON users(phone);
```

**000003_add_phone_to_users.down.sql**
```sql
DROP INDEX IF EXISTS idx_users_phone;
ALTER TABLE users DROP COLUMN phone;
```

---

## 🔄 Data Migrations

<div dir="rtl">

نقل البيانات أثناء الـ migration:

</div>

```sql
-- 000004_split_full_name.up.sql
-- Step 1: Add new columns
ALTER TABLE users ADD COLUMN first_name VARCHAR(50);
ALTER TABLE users ADD COLUMN last_name VARCHAR(50);

-- Step 2: Migrate data
UPDATE users SET
    first_name = split_part(full_name, ' ', 1),
    last_name = CASE
        WHEN array_length(string_to_array(full_name, ' '), 1) > 1
        THEN split_part(full_name, ' ', 2)
        ELSE ''
    END
WHERE full_name IS NOT NULL;

-- Step 3: Drop old column (optional, could be in next migration)
-- ALTER TABLE users DROP COLUMN full_name;
```

```sql
-- 000004_split_full_name.down.sql
-- Restore full_name
ALTER TABLE users ADD COLUMN full_name VARCHAR(100);

UPDATE users SET
    full_name = CONCAT(first_name, ' ', last_name)
WHERE first_name IS NOT NULL;

ALTER TABLE users DROP COLUMN first_name;
ALTER TABLE users DROP COLUMN last_name;
```

---

## 🏭 Migration Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Migration Best Practices                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. One change per migration                                         │
│     ├── 001_create_users.sql                                        │
│     └── 002_add_email_index.sql (مش في نفس الملف)                   │
│                                                                      │
│  2. Always write DOWN migration                                      │
│     لازم تقدر ترجع لأي نقطة                                         │
│                                                                      │
│  3. Never edit applied migrations                                    │
│     Migration اتنفذ؟ اكتب واحد جديد!                                 │
│                                                                      │
│  4. Test migrations locally first                                    │
│     migrate up → test → migrate down → migrate up                   │
│                                                                      │
│  5. Use transactions when possible                                   │
│     BEGIN; ... COMMIT;                                               │
│                                                                      │
│  6. Consider backwards compatibility                                 │
│     Add column → deploy code → remove old column                    │
│                                                                      │
│  7. Keep migrations fast                                             │
│     Large data migrations في off-peak hours                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Migration States

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Migration States                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Database State               Available Migrations                  │
│   ─────────────────────        ──────────────────────────           │
│                                                                      │
│   schema_migrations:           migrations/                           │
│   ┌─────────┐                  ┌─────────────────────────┐          │
│   │ version │                  │ 001_create_users      ✓ │          │
│   ├─────────┤                  │ 002_create_orders     ✓ │          │
│   │   1     │                  │ 003_add_phone         ✓ │          │
│   │   2     │                  │ 004_add_avatar        ✗ │ ← Pending │
│   │   3     │                  │ 005_add_settings      ✗ │          │
│   └─────────┘                  └─────────────────────────┘          │
│                                                                      │
│   Current version: 3                                                 │
│   Latest version: 5                                                  │
│   Pending: 2 migrations                                              │
│                                                                      │
│   Commands:                                                          │
│   migrate up     → Apply all pending (4, 5)                         │
│   migrate up 1   → Apply one (4)                                    │
│   migrate down 1 → Rollback one (3)                                 │
│   migrate goto 2 → Go to version 2                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Simple Migration Runner

```go
package migrations

import (
    "database/sql"
    "embed"
    "fmt"
    "sort"
    "strings"
)

//go:embed *.sql
var migrationsFS embed.FS

type Migration struct {
    Version int
    Name    string
    Up      string
    Down    string
}

func loadMigrations() ([]Migration, error) {
    entries, err := migrationsFS.ReadDir(".")
    if err != nil {
        return nil, err
    }

    migrationsMap := make(map[int]*Migration)

    for _, entry := range entries {
        if entry.IsDir() {
            continue
        }

        name := entry.Name()
        if !strings.HasSuffix(name, ".sql") {
            continue
        }

        var version int
        var direction string
        fmt.Sscanf(name, "%d_%s", &version, &direction)

        if _, ok := migrationsMap[version]; !ok {
            migrationsMap[version] = &Migration{Version: version}
        }

        content, _ := migrationsFS.ReadFile(name)

        if strings.Contains(name, ".up.") {
            migrationsMap[version].Up = string(content)
            migrationsMap[version].Name = strings.TrimSuffix(name, ".up.sql")
        } else if strings.Contains(name, ".down.") {
            migrationsMap[version].Down = string(content)
        }
    }

    migrations := make([]Migration, 0, len(migrationsMap))
    for _, m := range migrationsMap {
        migrations = append(migrations, *m)
    }
    sort.Slice(migrations, func(i, j int) bool {
        return migrations[i].Version < migrations[j].Version
    })

    return migrations, nil
}

func MigrateUp(db *sql.DB) error {
    // Create migrations table
    _, err := db.Exec(`
        CREATE TABLE IF NOT EXISTS schema_migrations (
            version INT PRIMARY KEY,
            applied_at TIMESTAMPTZ DEFAULT NOW()
        )
    `)
    if err != nil {
        return err
    }

    migrations, err := loadMigrations()
    if err != nil {
        return err
    }

    for _, m := range migrations {
        // Check if already applied
        var exists bool
        db.QueryRow("SELECT EXISTS(SELECT 1 FROM schema_migrations WHERE version = $1)",
            m.Version).Scan(&exists)

        if exists {
            continue
        }

        // Apply migration
        _, err := db.Exec(m.Up)
        if err != nil {
            return fmt.Errorf("migration %d failed: %w", m.Version, err)
        }

        // Record migration
        _, err = db.Exec("INSERT INTO schema_migrations (version) VALUES ($1)", m.Version)
        if err != nil {
            return err
        }

        fmt.Printf("Applied migration %d: %s\n", m.Version, m.Name)
    }

    return nil
}
```

---

## 🔄 Zero-Downtime Migrations

```
┌─────────────────────────────────────────────────────────────────────┐
│                 Zero-Downtime Migration Strategy                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Scenario: Rename column 'name' to 'full_name'                       │
│                                                                      │
│  ❌ Wrong way (downtime):                                            │
│  ALTER TABLE users RENAME COLUMN name TO full_name;                 │
│  -- Old code breaks immediately!                                     │
│                                                                      │
│  ✅ Right way (zero-downtime):                                       │
│                                                                      │
│  Step 1: Add new column                                              │
│  ALTER TABLE users ADD COLUMN full_name VARCHAR(100);               │
│                                                                      │
│  Step 2: Copy data                                                   │
│  UPDATE users SET full_name = name WHERE full_name IS NULL;         │
│                                                                      │
│  Step 3: Add trigger for sync                                        │
│  CREATE TRIGGER sync_name ...                                       │
│                                                                      │
│  Step 4: Deploy new code (reads full_name, writes both)             │
│                                                                      │
│  Step 5: Remove old column                                           │
│  ALTER TABLE users DROP COLUMN name;                                │
│                                                                      │
│  Step 6: Remove trigger                                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Key Takeaways

<div dir="rtl">

1. **AutoMigrate** للـ development فقط
2. استخدم **migration tool** للـ production
3. دايماً اكتب **DOWN migration**
4. **لا تعدل** migration بعد ما اتنفذ
5. **اختبر** locally قبل production
6. فكر في **zero-downtime** للـ large tables

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [golang-migrate](./12-golang-migrate.md)**

</div>

---

<div align="center">

[⬅️ السابق](./10-gorm-advanced.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./12-golang-migrate.md)

</div>
