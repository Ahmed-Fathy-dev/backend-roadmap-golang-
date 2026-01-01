# GORM Advanced - ميزات متقدمة 🚀

<div dir="rtl">

## مقدمة

في هذا الدرس هنتعلم الميزات المتقدمة في GORM زي الـ transactions، raw SQL، plugins، و performance optimization.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 🔄 Transactions

```go
package main

import (
    "gorm.io/gorm"
)

// Manual Transaction
func transferMoney(db *gorm.DB, fromID, toID uint, amount float64) error {
    tx := db.Begin()
    if tx.Error != nil {
        return tx.Error
    }

    // Defer rollback in case of error
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
        }
    }()

    // Deduct from source
    if err := tx.Model(&Account{}).Where("id = ? AND balance >= ?", fromID, amount).
        Update("balance", gorm.Expr("balance - ?", amount)).Error; err != nil {
        tx.Rollback()
        return err
    }

    // Add to destination
    if err := tx.Model(&Account{}).Where("id = ?", toID).
        Update("balance", gorm.Expr("balance + ?", amount)).Error; err != nil {
        tx.Rollback()
        return err
    }

    return tx.Commit().Error
}

// Transaction with callback (recommended)
func transferMoneyV2(db *gorm.DB, fromID, toID uint, amount float64) error {
    return db.Transaction(func(tx *gorm.DB) error {
        // Deduct from source
        result := tx.Model(&Account{}).Where("id = ? AND balance >= ?", fromID, amount).
            Update("balance", gorm.Expr("balance - ?", amount))
        if result.Error != nil {
            return result.Error
        }
        if result.RowsAffected == 0 {
            return errors.New("insufficient balance")
        }

        // Add to destination
        if err := tx.Model(&Account{}).Where("id = ?", toID).
            Update("balance", gorm.Expr("balance + ?", amount)).Error; err != nil {
            return err
        }

        // Return nil to commit
        return nil
    })
}

// Nested Transactions (Savepoints)
func complexOperation(db *gorm.DB) error {
    return db.Transaction(func(tx *gorm.DB) error {
        // Main operation
        if err := tx.Create(&User{Username: "main"}).Error; err != nil {
            return err
        }

        // Nested transaction
        err := tx.Transaction(func(tx2 *gorm.DB) error {
            // Risky operation
            return tx2.Create(&User{Username: "nested"}).Error
        })

        if err != nil {
            // Nested transaction rolled back, but main continues
            log.Println("Nested failed:", err)
        }

        return nil
    })
}
```

---

## 📝 Raw SQL

```go
// Raw SELECT
func rawQueries(db *gorm.DB) {
    var users []User

    // Simple raw query
    db.Raw("SELECT * FROM users WHERE is_active = ?", true).Scan(&users)

    // With named parameters
    db.Raw("SELECT * FROM users WHERE username = @username AND is_active = @active",
        sql.Named("username", "ahmed"),
        sql.Named("active", true),
    ).Scan(&users)

    // Custom result struct
    type UserSummary struct {
        Username    string
        OrderCount  int
        TotalSpent  float64
    }
    var summaries []UserSummary
    db.Raw(`
        SELECT u.username,
               COUNT(o.id) as order_count,
               COALESCE(SUM(o.total_amount), 0) as total_spent
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        GROUP BY u.id
    `).Scan(&summaries)

    // Exec for non-SELECT
    db.Exec("UPDATE users SET is_active = ? WHERE last_login < ?", false, time.Now().AddDate(-1, 0, 0))

    // Exec with RETURNING (PostgreSQL)
    var updatedIDs []uint
    db.Raw("UPDATE users SET is_active = false WHERE last_login < ? RETURNING id",
        time.Now().AddDate(-1, 0, 0)).Scan(&updatedIDs)
}

// Row & Rows for streaming
func streamingQueries(db *gorm.DB) {
    // Single row
    row := db.Raw("SELECT COUNT(*) FROM users").Row()
    var count int
    row.Scan(&count)

    // Multiple rows
    rows, _ := db.Raw("SELECT id, username FROM users").Rows()
    defer rows.Close()

    for rows.Next() {
        var id uint
        var username string
        rows.Scan(&id, &username)
        fmt.Printf("%d: %s\n", id, username)
    }
}
```

---

## 🔒 Locking

```go
// FOR UPDATE
func lockForUpdate(db *gorm.DB, productID uint) (*Product, error) {
    var product Product

    err := db.Transaction(func(tx *gorm.DB) error {
        // Lock the row
        if err := tx.Clauses(clause.Locking{Strength: "UPDATE"}).
            First(&product, productID).Error; err != nil {
            return err
        }

        // Modify
        product.Stock -= 1
        return tx.Save(&product).Error
    })

    return &product, err
}

// FOR SHARE (read lock)
func lockForShare(db *gorm.DB, userID uint) (*User, error) {
    var user User
    err := db.Clauses(clause.Locking{Strength: "SHARE"}).First(&user, userID).Error
    return &user, err
}

// SKIP LOCKED (for queue processing)
func getNextJob(db *gorm.DB) (*Job, error) {
    var job Job

    err := db.Transaction(func(tx *gorm.DB) error {
        if err := tx.Clauses(clause.Locking{
            Strength: "UPDATE",
            Options:  "SKIP LOCKED",
        }).Where("status = ?", "pending").
            Order("created_at").
            First(&job).Error; err != nil {
            return err
        }

        job.Status = "processing"
        return tx.Save(&job).Error
    })

    return &job, err
}
```

---

## 📊 Batch Operations

```go
// Batch Create
func batchCreate(db *gorm.DB, users []User) error {
    // Creates in batches of 100
    return db.CreateInBatches(&users, 100).Error
}

// Batch Update with different values
func batchUpdate(db *gorm.DB) error {
    // Using case statement
    return db.Exec(`
        UPDATE products SET price = CASE id
            WHEN 1 THEN 99.99
            WHEN 2 THEN 149.99
            WHEN 3 THEN 199.99
        END
        WHERE id IN (1, 2, 3)
    `).Error
}

// FindInBatches
func processUsersInBatches(db *gorm.DB) error {
    var users []User

    result := db.Where("is_active = ?", true).FindInBatches(&users, 100, func(tx *gorm.DB, batch int) error {
        for _, user := range users {
            // Process each user
            fmt.Printf("Processing batch %d, user %d\n", batch, user.ID)
        }

        // Modify and save if needed
        // tx.Save(&users)

        return nil
    })

    return result.Error
}
```

---

## 🎯 Clauses

```go
import "gorm.io/gorm/clause"

func clauseExamples(db *gorm.DB) {
    // UPSERT (ON CONFLICT)
    user := User{Username: "ahmed", Email: "ahmed@example.com"}
    db.Clauses(clause.OnConflict{
        Columns:   []clause.Column{{Name: "username"}},
        DoUpdates: clause.AssignmentColumns([]string{"email", "updated_at"}),
    }).Create(&user)
    // INSERT INTO users ... ON CONFLICT (username)
    // DO UPDATE SET email = excluded.email, updated_at = excluded.updated_at

    // UPSERT - Do Nothing
    db.Clauses(clause.OnConflict{DoNothing: true}).Create(&user)

    // RETURNING
    db.Clauses(clause.Returning{Columns: []clause.Column{{Name: "id"}}}).Create(&user)

    // INSERT IGNORE
    db.Clauses(clause.Insert{Modifier: "IGNORE"}).Create(&user)

    // Hints
    db.Clauses(hints.New("MAX_EXECUTION_TIME(1000)")).Find(&users)
}
```

---

## 🔌 Plugins & Callbacks

```go
// Custom Plugin
type MyPlugin struct{}

func (p *MyPlugin) Name() string {
    return "my_plugin"
}

func (p *MyPlugin) Initialize(db *gorm.DB) error {
    // Register callbacks
    db.Callback().Create().Before("gorm:create").Register("my_plugin:before_create", beforeCreate)
    db.Callback().Query().After("gorm:query").Register("my_plugin:after_query", afterQuery)
    return nil
}

func beforeCreate(db *gorm.DB) {
    log.Println("Before create:", db.Statement.Table)
}

func afterQuery(db *gorm.DB) {
    log.Printf("Query took: %v, rows: %d\n", db.Statement.Context, db.RowsAffected)
}

// Use plugin
func usePlugin() {
    db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    db.Use(&MyPlugin{})
}

// Prometheus metrics plugin example
type PrometheusPlugin struct {
    queryCounter   *prometheus.CounterVec
    queryDuration  *prometheus.HistogramVec
}

func (p *PrometheusPlugin) Initialize(db *gorm.DB) error {
    db.Callback().Query().After("gorm:query").Register("prometheus:query", func(db *gorm.DB) {
        p.queryCounter.WithLabelValues(db.Statement.Table, "query").Inc()
    })
    return nil
}
```

---

## 📊 Performance

```go
// 1. Use PrepareStmt
db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{
    PrepareStmt: true,
})

// 2. Disable default transaction for single operations
db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{
    SkipDefaultTransaction: true,
})

// 3. Select only needed columns
db.Select("id", "username").Find(&users)

// 4. Use Joins instead of Preload when filtering
db.Joins("Profile").Where("profiles.is_verified = ?", true).Find(&users)

// 5. Use indexes
type User struct {
    ID       uint   `gorm:"primaryKey"`
    Email    string `gorm:"uniqueIndex"`
    Username string `gorm:"index"`
    Status   string `gorm:"index:idx_status_created"`
    CreatedAt time.Time `gorm:"index:idx_status_created"`
}

// 6. Batch operations
db.CreateInBatches(&users, 100)
db.Where("status = ?", "old").Delete(&User{}) // Instead of loop

// 7. Use raw SQL for complex queries
db.Raw("SELECT ... complex query ...").Scan(&results)
```

---

## 🔍 Debugging

```go
// Enable logger
db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{
    Logger: logger.Default.LogMode(logger.Info),
})

// Debug mode for single query
db.Debug().Where("name = ?", "ahmed").First(&user)

// Custom logger
newLogger := logger.New(
    log.New(os.Stdout, "\r\n", log.LstdFlags),
    logger.Config{
        SlowThreshold:             200 * time.Millisecond,
        LogLevel:                  logger.Warn,
        IgnoreRecordNotFoundError: true,
        Colorful:                  true,
    },
)
db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{
    Logger: newLogger,
})

// DryRun - generate SQL without executing
stmt := db.Session(&gorm.Session{DryRun: true}).First(&user, 1).Statement
fmt.Println(stmt.SQL.String()) // SELECT * FROM users WHERE id = $1
fmt.Println(stmt.Vars)         // [1]

// ToSQL - get SQL string
sql := db.ToSQL(func(tx *gorm.DB) *gorm.DB {
    return tx.Where("id = ?", 1).First(&User{})
})
fmt.Println(sql)
```

---

## 🎨 Context & Timeout

```go
// With context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

db.WithContext(ctx).Find(&users)

// Session with context
db.Session(&gorm.Session{Context: ctx}).Find(&users)

// Global context (not recommended)
db = db.WithContext(ctx)
```

---

## 📦 Sharding

```go
import "gorm.io/sharding"

// Setup sharding
db.Use(sharding.Register(sharding.Config{
    ShardingKey:         "user_id",
    NumberOfShards:      4,
    PrimaryKeyGenerator: sharding.PKSnowflake,
}, "orders"))

// Now queries will be sharded automatically
db.Create(&Order{UserID: 123}) // Goes to shard based on user_id
db.Where("user_id = ?", 123).Find(&orders) // Queries correct shard
```

---

## 🧪 Testing with GORM

```go
import (
    "github.com/DATA-DOG/go-sqlmock"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func TestUserRepository(t *testing.T) {
    // Create mock database
    mockDB, mock, err := sqlmock.New()
    if err != nil {
        t.Fatal(err)
    }
    defer mockDB.Close()

    // Initialize GORM with mock
    dialector := postgres.New(postgres.Config{
        Conn:       mockDB,
        DriverName: "postgres",
    })
    db, _ := gorm.Open(dialector, &gorm.Config{})

    // Set expectations
    mock.ExpectQuery(`SELECT \* FROM "users" WHERE id = \$1`).
        WithArgs(1).
        WillReturnRows(sqlmock.NewRows([]string{"id", "username", "email"}).
            AddRow(1, "ahmed", "ahmed@example.com"))

    // Test
    var user User
    err = db.First(&user, 1).Error

    assert.NoError(t, err)
    assert.Equal(t, "ahmed", user.Username)
    assert.NoError(t, mock.ExpectationsWereMet())
}
```

---

## 💡 Best Practices Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                  GORM Advanced Best Practices                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use db.Transaction() for transactions                            │
│     Automatic rollback on error                                     │
│                                                                      │
│  2. Enable PrepareStmt for performance                               │
│     &gorm.Config{PrepareStmt: true}                                 │
│                                                                      │
│  3. Use raw SQL for complex queries                                  │
│     db.Raw("SELECT...").Scan(&results)                              │
│                                                                      │
│  4. Use batch operations                                             │
│     CreateInBatches, FindInBatches                                  │
│                                                                      │
│  5. FOR UPDATE for critical sections                                 │
│     clause.Locking{Strength: "UPDATE"}                              │
│                                                                      │
│  6. Use Context with timeout                                         │
│     db.WithContext(ctx)                                             │
│                                                                      │
│  7. Debug with DryRun                                                │
│     Session(&gorm.Session{DryRun: true})                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **db.Transaction()** أفضل من Begin/Commit manual
2. استخدم **raw SQL** للـ queries المعقدة
3. **FOR UPDATE** للـ critical sections
4. **PrepareStmt** لتحسين الأداء
5. **Context** للـ timeout والـ cancellation
6. **Debug()** و **DryRun** للتطوير

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Database Migrations](./11-migrations.md)**

</div>

---

<div align="center">

[⬅️ السابق](./09-gorm-relations.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./11-migrations.md)

</div>
