# golang-migrate - أداة golang-migrate 🛠️

<div dir="rtl">

## مقدمة

golang-migrate هي أداة قوية لإدارة database migrations. بتدعم SQL files وكمان embedded migrations في الـ binary.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 🔧 التثبيت

```bash
# CLI tool
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest

# أو باستخدام brew (macOS)
brew install golang-migrate

# أو scoop (Windows)
scoop install migrate

# كـ library في المشروع
go get -u github.com/golang-migrate/migrate/v4
go get -u github.com/golang-migrate/migrate/v4/database/postgres
go get -u github.com/golang-migrate/migrate/v4/source/file
```

---

## 📁 هيكل المشروع

```
myproject/
├── cmd/
│   └── main.go
├── internal/
│   └── database/
│       └── database.go
├── migrations/
│   ├── 000001_create_users.up.sql
│   ├── 000001_create_users.down.sql
│   ├── 000002_create_orders.up.sql
│   ├── 000002_create_orders.down.sql
│   ├── 000003_add_phone_to_users.up.sql
│   └── 000003_add_phone_to_users.down.sql
├── go.mod
└── Makefile
```

---

## 📝 إنشاء Migrations

```bash
# إنشاء migration جديد
migrate create -ext sql -dir migrations -seq create_users

# ده هينشئ:
# migrations/000001_create_users.up.sql
# migrations/000001_create_users.down.sql
```

**migrations/000001_create_users.up.sql**
```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    role VARCHAR(20) DEFAULT 'user',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
CREATE INDEX IF NOT EXISTS idx_users_username ON users(username);
```

**migrations/000001_create_users.down.sql**
```sql
DROP INDEX IF EXISTS idx_users_username;
DROP INDEX IF EXISTS idx_users_email;
DROP TABLE IF EXISTS users;
```

**migrations/000002_create_orders.up.sql**
```sql
CREATE TABLE IF NOT EXISTS orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    total_amount DECIMAL(10, 2) DEFAULT 0,
    shipping_address TEXT,
    notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT fk_orders_user FOREIGN KEY (user_id)
        REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_orders_user_id ON orders(user_id);
CREATE INDEX IF NOT EXISTS idx_orders_status ON orders(status);
CREATE INDEX IF NOT EXISTS idx_orders_created_at ON orders(created_at);
```

**migrations/000002_create_orders.down.sql**
```sql
DROP INDEX IF EXISTS idx_orders_created_at;
DROP INDEX IF EXISTS idx_orders_status;
DROP INDEX IF EXISTS idx_orders_user_id;
DROP TABLE IF EXISTS orders;
```

---

## 🖥️ استخدام CLI

```bash
# Connection string
export DATABASE_URL="postgres://postgres:password@localhost:5432/mydb?sslmode=disable"

# تطبيق كل الـ migrations
migrate -database "${DATABASE_URL}" -path migrations up

# تطبيق n migrations
migrate -database "${DATABASE_URL}" -path migrations up 2

# التراجع عن آخر migration
migrate -database "${DATABASE_URL}" -path migrations down 1

# التراجع عن كل الـ migrations
migrate -database "${DATABASE_URL}" -path migrations down

# الذهاب لـ version معين
migrate -database "${DATABASE_URL}" -path migrations goto 3

# عرض الـ version الحالي
migrate -database "${DATABASE_URL}" -path migrations version

# Force version (للإصلاح)
migrate -database "${DATABASE_URL}" -path migrations force 3

# التحقق من الـ migrations
migrate -database "${DATABASE_URL}" -path migrations validate
```

---

## 📦 استخدام كـ Library

```go
package database

import (
    "database/sql"
    "embed"
    "fmt"

    "github.com/golang-migrate/migrate/v4"
    "github.com/golang-migrate/migrate/v4/database/postgres"
    "github.com/golang-migrate/migrate/v4/source/iofs"
    _ "github.com/jackc/pgx/v5/stdlib"
)

//go:embed migrations/*.sql
var migrationsFS embed.FS

type DB struct {
    *sql.DB
}

func New(dsn string) (*DB, error) {
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        return nil, err
    }

    if err := db.Ping(); err != nil {
        return nil, err
    }

    return &DB{db}, nil
}

func (db *DB) MigrateUp() error {
    driver, err := postgres.WithInstance(db.DB, &postgres.Config{})
    if err != nil {
        return fmt.Errorf("create driver: %w", err)
    }

    source, err := iofs.New(migrationsFS, "migrations")
    if err != nil {
        return fmt.Errorf("create source: %w", err)
    }

    m, err := migrate.NewWithInstance("iofs", source, "postgres", driver)
    if err != nil {
        return fmt.Errorf("create migrate: %w", err)
    }

    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        return fmt.Errorf("migrate up: %w", err)
    }

    return nil
}

func (db *DB) MigrateDown() error {
    driver, err := postgres.WithInstance(db.DB, &postgres.Config{})
    if err != nil {
        return err
    }

    source, err := iofs.New(migrationsFS, "migrations")
    if err != nil {
        return err
    }

    m, err := migrate.NewWithInstance("iofs", source, "postgres", driver)
    if err != nil {
        return err
    }

    if err := m.Down(); err != nil && err != migrate.ErrNoChange {
        return err
    }

    return nil
}

func (db *DB) MigrateVersion() (uint, bool, error) {
    driver, err := postgres.WithInstance(db.DB, &postgres.Config{})
    if err != nil {
        return 0, false, err
    }

    source, err := iofs.New(migrationsFS, "migrations")
    if err != nil {
        return 0, false, err
    }

    m, err := migrate.NewWithInstance("iofs", source, "postgres", driver)
    if err != nil {
        return 0, false, err
    }

    return m.Version()
}

func (db *DB) MigrateSteps(n int) error {
    driver, err := postgres.WithInstance(db.DB, &postgres.Config{})
    if err != nil {
        return err
    }

    source, err := iofs.New(migrationsFS, "migrations")
    if err != nil {
        return err
    }

    m, err := migrate.NewWithInstance("iofs", source, "postgres", driver)
    if err != nil {
        return err
    }

    return m.Steps(n)
}
```

---

## 🏭 استخدام في main.go

```go
package main

import (
    "flag"
    "log"
    "os"

    "myproject/internal/database"
)

func main() {
    // Flags
    migrateUp := flag.Bool("migrate-up", false, "Run migrations up")
    migrateDown := flag.Bool("migrate-down", false, "Run migrations down")
    flag.Parse()

    // Database connection
    dsn := os.Getenv("DATABASE_URL")
    if dsn == "" {
        log.Fatal("DATABASE_URL not set")
    }

    db, err := database.New(dsn)
    if err != nil {
        log.Fatal("Failed to connect:", err)
    }
    defer db.Close()

    // Handle migrations
    if *migrateUp {
        log.Println("Running migrations up...")
        if err := db.MigrateUp(); err != nil {
            log.Fatal("Migration up failed:", err)
        }
        log.Println("Migrations completed!")
        return
    }

    if *migrateDown {
        log.Println("Running migrations down...")
        if err := db.MigrateDown(); err != nil {
            log.Fatal("Migration down failed:", err)
        }
        log.Println("Migrations rolled back!")
        return
    }

    // Auto-migrate on start (development only!)
    if os.Getenv("AUTO_MIGRATE") == "true" {
        if err := db.MigrateUp(); err != nil {
            log.Fatal("Auto-migration failed:", err)
        }
    }

    // Start application...
    log.Println("Starting application...")
}
```

---

## 📋 Makefile

```makefile
# Database settings
DB_URL ?= postgres://postgres:password@localhost:5432/mydb?sslmode=disable

# Migration commands
.PHONY: migrate-create migrate-up migrate-down migrate-version

migrate-create:
	@read -p "Migration name: " name; \
	migrate create -ext sql -dir migrations -seq $$name

migrate-up:
	migrate -database "$(DB_URL)" -path migrations up

migrate-up-one:
	migrate -database "$(DB_URL)" -path migrations up 1

migrate-down:
	migrate -database "$(DB_URL)" -path migrations down 1

migrate-down-all:
	migrate -database "$(DB_URL)" -path migrations down

migrate-version:
	migrate -database "$(DB_URL)" -path migrations version

migrate-force:
	@read -p "Version: " version; \
	migrate -database "$(DB_URL)" -path migrations force $$version

migrate-validate:
	migrate -database "$(DB_URL)" -path migrations validate

# Development
.PHONY: dev db-reset

db-reset: migrate-down-all migrate-up
	@echo "Database reset complete!"

dev:
	AUTO_MIGRATE=true go run cmd/main.go
```

---

## ⚠️ التعامل مع الأخطاء

```go
import (
    "errors"
    "github.com/golang-migrate/migrate/v4"
)

func handleMigrationError(err error) {
    if err == nil {
        return
    }

    // No change needed
    if errors.Is(err, migrate.ErrNoChange) {
        log.Println("No migrations to apply")
        return
    }

    // Dirty database state
    if errors.Is(err, migrate.ErrDirty) {
        log.Println("Database is in dirty state. Run 'migrate force VERSION'")
        return
    }

    // Locked
    if errors.Is(err, migrate.ErrLocked) {
        log.Println("Database is locked by another migration")
        return
    }

    log.Fatal("Migration error:", err)
}
```

---

## 🔄 CI/CD Integration

```yaml
# .github/workflows/migrate.yml
name: Database Migrations

on:
  push:
    branches: [main]
    paths:
      - 'migrations/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install migrate
        run: |
          curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz
          sudo mv migrate /usr/local/bin/

      - name: Run migrations
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          migrate -database "${DATABASE_URL}" -path migrations up
```

```dockerfile
# Dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY . .
RUN go build -o main ./cmd/main.go

FROM alpine:latest

# Install migrate
RUN apk add --no-cache curl
RUN curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz
RUN mv migrate /usr/local/bin/

WORKDIR /app
COPY --from=builder /app/main .
COPY migrations ./migrations

# Entrypoint script
COPY docker-entrypoint.sh .
RUN chmod +x docker-entrypoint.sh

ENTRYPOINT ["./docker-entrypoint.sh"]
```

```bash
#!/bin/bash
# docker-entrypoint.sh

# Run migrations
echo "Running database migrations..."
migrate -database "${DATABASE_URL}" -path migrations up

# Start application
echo "Starting application..."
exec ./main
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                golang-migrate Best Practices                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use sequential numbering                                         │
│     migrate create -seq create_users                                │
│                                                                      │
│  2. Always write DOWN migrations                                     │
│     Test: up → down → up                                            │
│                                                                      │
│  3. Use transactions                                                 │
│     BEGIN; ... COMMIT;                                               │
│                                                                      │
│  4. Keep migrations small                                            │
│     One logical change per migration                                │
│                                                                      │
│  5. Use IF EXISTS / IF NOT EXISTS                                    │
│     DROP TABLE IF EXISTS users;                                     │
│                                                                      │
│  6. Test locally before production                                   │
│     make migrate-up migrate-down migrate-up                         │
│                                                                      │
│  7. Use embedded migrations in binary                                │
│     //go:embed migrations/*.sql                                     │
│                                                                      │
│  8. Run migrations at startup (dev only)                             │
│     Production: run separately before deployment                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **golang-migrate** أداة production-ready
2. استخدم **sequential numbering** للـ migrations
3. دايماً اكتب **UP و DOWN** migrations
4. استخدم **embed** للـ migrations في الـ binary
5. **اختبر** locally قبل production
6. في CI/CD، شغل migrations **قبل** الـ deployment

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Repository Pattern](./13-repository-pattern.md)**

</div>

---

<div align="center">

[⬅️ السابق](./11-migrations.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./13-repository-pattern.md)

</div>
