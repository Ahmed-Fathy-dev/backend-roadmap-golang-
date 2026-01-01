# GORM Basics - مقدمة في GORM 🏗️

<div dir="rtl">

## مقدمة

GORM هو الـ ORM (Object-Relational Mapping) الأشهر في Go. بيسمح لك تتعامل مع الـ database باستخدام Go structs بدل SQL مباشرة.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 ما هو ORM؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Without ORM (Raw SQL)                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   db.Query("SELECT id, username, email FROM users WHERE id = $1", 1)│
│   rows.Scan(&user.ID, &user.Username, &user.Email)                  │
│                                                                      │
│   db.Exec("INSERT INTO users (username, email) VALUES ($1, $2)",    │
│           "ahmed", "ahmed@example.com")                              │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                      With ORM (GORM)                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   db.First(&user, 1)                                                │
│                                                                      │
│   db.Create(&User{Username: "ahmed", Email: "ahmed@example.com"})   │
│                                                                      │
│   ✅ أبسط وأسهل للقراءة                                             │
│   ✅ Type-safe                                                       │
│   ✅ Auto-migrations                                                 │
│   ✅ Relationships management                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 التثبيت

```bash
# GORM core
go get -u gorm.io/gorm

# PostgreSQL driver
go get -u gorm.io/driver/postgres
```

---

## 🔌 الاتصال

```go
package main

import (
    "fmt"
    "log"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
)

func main() {
    // Connection string
    dsn := "host=localhost user=postgres password=secret dbname=mydb port=5432 sslmode=disable"

    // الاتصال مع إعدادات
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info), // Log all queries
    })
    if err != nil {
        log.Fatal("Failed to connect:", err)
    }

    fmt.Println("Connected to PostgreSQL via GORM!")

    // الحصول على underlying *sql.DB
    sqlDB, err := db.DB()
    if err != nil {
        log.Fatal(err)
    }

    // Connection pool settings
    sqlDB.SetMaxOpenConns(25)
    sqlDB.SetMaxIdleConns(5)
}
```

---

## 📦 تعريف Model

```go
package models

import (
    "time"
    "gorm.io/gorm"
)

// User model
type User struct {
    // gorm.Model بيضيف: ID, CreatedAt, UpdatedAt, DeletedAt
    gorm.Model

    Username  string `gorm:"size:50;uniqueIndex;not null"`
    Email     string `gorm:"size:255;uniqueIndex;not null"`
    Password  string `gorm:"size:255;not null"`
    FullName  string `gorm:"size:100"`
    IsActive  bool   `gorm:"default:true"`
    Role      string `gorm:"size:20;default:'user'"`
}

// أو بدون gorm.Model للتحكم الكامل
type Product struct {
    ID          uint      `gorm:"primaryKey"`
    SKU         string    `gorm:"size:50;uniqueIndex;not null"`
    Name        string    `gorm:"size:200;not null"`
    Description string    `gorm:"type:text"`
    Price       float64   `gorm:"type:decimal(10,2);not null"`
    Stock       int       `gorm:"default:0"`
    CategoryID  *uint     `gorm:"index"` // nullable foreign key
    IsAvailable bool      `gorm:"default:true"`
    CreatedAt   time.Time
    UpdatedAt   time.Time
}

// Custom table name
func (Product) TableName() string {
    return "products"
}
```

---

## 🔧 GORM Tags

```go
type Example struct {
    // Primary Key
    ID uint `gorm:"primaryKey"`

    // Column name
    UserName string `gorm:"column:user_name"`

    // Size/Type
    Name        string  `gorm:"size:100"`
    Description string  `gorm:"type:text"`
    Price       float64 `gorm:"type:decimal(10,2)"`

    // Constraints
    Email    string `gorm:"uniqueIndex;not null"`
    Username string `gorm:"unique"`
    Age      int    `gorm:"check:age >= 0"`

    // Default value
    IsActive bool   `gorm:"default:true"`
    Role     string `gorm:"default:'user'"`

    // Index
    Status string `gorm:"index"`
    Code   string `gorm:"uniqueIndex"`

    // Composite index
    FirstName string `gorm:"index:idx_name"`
    LastName  string `gorm:"index:idx_name"`

    // Ignore field
    TempField string `gorm:"-"`

    // Auto timestamps
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"` // Soft delete
}
```

---

## 🏃 Auto-Migration

<div dir="rtl">

GORM بيقدر يعمل migrate للـ schema تلقائياً:

</div>

```go
func main() {
    db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})

    // Auto-migrate - creates/updates tables
    err := db.AutoMigrate(
        &User{},
        &Product{},
        &Order{},
        &OrderItem{},
    )
    if err != nil {
        log.Fatal("Migration failed:", err)
    }

    fmt.Println("Migration completed!")
}
```

<div dir="rtl">

### ⚠️ ملاحظات على AutoMigrate

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AutoMigrate Behavior                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ DOES:                                                            │
│  ├── Create tables                                                  │
│  ├── Add missing columns                                            │
│  ├── Create indexes                                                 │
│  └── Add foreign keys                                               │
│                                                                      │
│  ❌ DOES NOT:                                                        │
│  ├── Delete unused columns                                          │
│  ├── Delete unused tables                                           │
│  ├── Change column types (might fail)                               │
│  └── Rename columns/tables                                          │
│                                                                      │
│  📝 Recommendation:                                                  │
│  Use AutoMigrate for development only!                              │
│  Use proper migration tools (golang-migrate) for production.        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ➕ Create (INSERT)

```go
func createExamples(db *gorm.DB) {
    // Create single record
    user := User{
        Username: "ahmed",
        Email:    "ahmed@example.com",
        Password: "hashed_password",
        FullName: "Ahmed Ali",
    }
    result := db.Create(&user)

    if result.Error != nil {
        log.Fatal(result.Error)
    }
    fmt.Printf("Created user ID: %d\n", user.ID) // ID populated automatically!
    fmt.Printf("Rows affected: %d\n", result.RowsAffected)

    // Create with selected fields only
    db.Select("Username", "Email", "Password").Create(&User{
        Username: "sara",
        Email:    "sara@example.com",
        Password: "hash",
        FullName: "Ignored!", // لن يُحفظ
    })

    // Create multiple records
    users := []User{
        {Username: "user1", Email: "user1@example.com", Password: "hash1"},
        {Username: "user2", Email: "user2@example.com", Password: "hash2"},
        {Username: "user3", Email: "user3@example.com", Password: "hash3"},
    }
    db.Create(&users)

    // Create in batches
    db.CreateInBatches(&users, 100) // 100 per batch
}
```

---

## 📖 Read (SELECT)

```go
func readExamples(db *gorm.DB) {
    var user User
    var users []User

    // Get first record (by primary key order)
    db.First(&user)
    // SELECT * FROM users ORDER BY id LIMIT 1

    // Get by primary key
    db.First(&user, 10)
    // SELECT * FROM users WHERE id = 10

    // Get by primary key (multiple)
    db.Find(&users, []int{1, 2, 3})
    // SELECT * FROM users WHERE id IN (1, 2, 3)

    // Get last record
    db.Last(&user)
    // SELECT * FROM users ORDER BY id DESC LIMIT 1

    // Get one record (no order)
    db.Take(&user)
    // SELECT * FROM users LIMIT 1

    // Get all records
    db.Find(&users)
    // SELECT * FROM users

    // Conditions
    db.Where("email = ?", "ahmed@example.com").First(&user)
    db.Where("age > ?", 18).Find(&users)
    db.Where("username LIKE ?", "%ahmed%").Find(&users)
    db.Where("created_at > ?", time.Now().AddDate(0, -1, 0)).Find(&users)

    // Struct condition
    db.Where(&User{Username: "ahmed", IsActive: true}).First(&user)

    // Map condition
    db.Where(map[string]interface{}{"username": "ahmed", "is_active": true}).First(&user)

    // Multiple conditions
    db.Where("username = ?", "ahmed").Where("is_active = ?", true).First(&user)

    // Or condition
    db.Where("username = ?", "ahmed").Or("email = ?", "ahmed@example.com").First(&user)

    // Not condition
    db.Not("role = ?", "admin").Find(&users)

    // Select specific columns
    db.Select("id", "username", "email").Find(&users)

    // Order
    db.Order("created_at DESC").Find(&users)
    db.Order("username ASC, created_at DESC").Find(&users)

    // Limit & Offset
    db.Limit(10).Offset(20).Find(&users)

    // Count
    var count int64
    db.Model(&User{}).Where("is_active = ?", true).Count(&count)

    // Distinct
    var usernames []string
    db.Model(&User{}).Distinct("username").Pluck("username", &usernames)

    // Group By
    type Result struct {
        Role  string
        Count int64
    }
    var results []Result
    db.Model(&User{}).Select("role, count(*) as count").Group("role").Scan(&results)
}
```

---

## ✏️ Update (UPDATE)

```go
func updateExamples(db *gorm.DB) {
    var user User
    db.First(&user, 1)

    // Update single field
    db.Model(&user).Update("email", "newemail@example.com")
    // UPDATE users SET email='newemail@example.com', updated_at=NOW() WHERE id=1

    // Update multiple fields using struct
    db.Model(&user).Updates(User{Email: "new@example.com", FullName: "New Name"})
    // ⚠️ Zero values will be ignored!

    // Update multiple fields using map (includes zero values)
    db.Model(&user).Updates(map[string]interface{}{
        "email":     "new@example.com",
        "is_active": false, // سيتم تحديثه حتى لو false
    })

    // Update selected fields only
    db.Model(&user).Select("email", "full_name").Updates(User{
        Email:    "selected@example.com",
        FullName: "Selected Update",
        IsActive: false, // سيتم تجاهله
    })

    // Update without loading the record
    db.Model(&User{}).Where("id = ?", 1).Update("is_active", false)

    // Update all records (careful!)
    db.Model(&User{}).Where("is_active = ?", false).Update("role", "inactive")

    // Increment/Decrement
    db.Model(&Product{}).Where("id = ?", 1).Update("stock", gorm.Expr("stock - ?", 1))

    // Update with SQL expression
    db.Model(&user).Update("updated_at", gorm.Expr("NOW()"))

    // Save - updates all fields
    user.Email = "updated@example.com"
    user.FullName = "Updated Name"
    db.Save(&user)
    // UPDATE users SET username=..., email=..., full_name=..., ... WHERE id=1
}
```

---

## 🗑️ Delete (DELETE)

```go
func deleteExamples(db *gorm.DB) {
    var user User
    db.First(&user, 1)

    // Delete by model (needs primary key)
    db.Delete(&user)
    // DELETE FROM users WHERE id = 1

    // Delete by primary key
    db.Delete(&User{}, 10)
    // DELETE FROM users WHERE id = 10

    // Delete multiple by primary keys
    db.Delete(&User{}, []int{1, 2, 3})
    // DELETE FROM users WHERE id IN (1, 2, 3)

    // Delete with conditions
    db.Where("is_active = ?", false).Delete(&User{})
    // DELETE FROM users WHERE is_active = false

    // Delete all (careful!)
    db.Where("1 = 1").Delete(&User{})
    // أو
    db.Exec("DELETE FROM users")
}
```

---

## 🔄 Soft Delete

<div dir="rtl">

لو الـ Model فيه DeletedAt، GORM هيعمل soft delete تلقائياً:

</div>

```go
type User struct {
    gorm.Model // includes DeletedAt
    Username string
    Email    string
}

func softDeleteExample(db *gorm.DB) {
    var user User
    db.First(&user, 1)

    // Soft delete
    db.Delete(&user)
    // UPDATE users SET deleted_at = NOW() WHERE id = 1
    // الصف لسه موجود في الـ database!

    // Normal queries تتجاهل deleted records
    db.Find(&users)
    // SELECT * FROM users WHERE deleted_at IS NULL

    // Include soft deleted records
    db.Unscoped().Find(&users)
    // SELECT * FROM users

    // Find only soft deleted records
    db.Unscoped().Where("deleted_at IS NOT NULL").Find(&users)

    // Permanent delete (hard delete)
    db.Unscoped().Delete(&user)
    // DELETE FROM users WHERE id = 1
}
```

---

## 🔍 Advanced Queries

```go
func advancedQueries(db *gorm.DB) {
    // Raw SQL
    var users []User
    db.Raw("SELECT * FROM users WHERE age > ?", 18).Scan(&users)

    // Raw SQL for non-model results
    type Result struct {
        Username string
        Total    int
    }
    var results []Result
    db.Raw("SELECT username, COUNT(*) as total FROM orders GROUP BY username").Scan(&results)

    // Exec for non-SELECT queries
    db.Exec("UPDATE users SET is_active = ? WHERE last_login < ?", false, time.Now().AddDate(-1, 0, 0))

    // Subquery
    subQuery := db.Model(&Order{}).Select("user_id").Where("status = ?", "completed")
    db.Where("id IN (?)", subQuery).Find(&users)

    // FirstOrCreate
    var user User
    db.FirstOrCreate(&user, User{Username: "newuser"})
    // Creates if doesn't exist, otherwise returns existing

    // FirstOrInit (doesn't save)
    db.FirstOrInit(&user, User{Username: "newuser"})

    // Assign attributes if not found
    db.Where(User{Username: "admin"}).Attrs(User{Role: "admin"}).FirstOrCreate(&user)

    // Pluck - get single column
    var emails []string
    db.Model(&User{}).Pluck("email", &emails)

    // Scan to different struct
    type UserSummary struct {
        ID       uint
        Username string
        Email    string
    }
    var summaries []UserSummary
    db.Model(&User{}).Select("id", "username", "email").Scan(&summaries)
}
```

---

## ⚙️ GORM Configuration

```go
func configureGorm() (*gorm.DB, error) {
    dsn := "host=localhost user=postgres password=secret dbname=mydb port=5432"

    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
        // Logger configuration
        Logger: logger.Default.LogMode(logger.Info),

        // Disable default transaction
        SkipDefaultTransaction: true,

        // Naming strategy
        NamingStrategy: schema.NamingStrategy{
            TablePrefix:   "app_",    // table prefix
            SingularTable: true,      // use singular table name
        },

        // Disable foreign key constraint when migrating
        DisableForeignKeyConstraintWhenMigrating: true,

        // PrepareStmt - caches prepared statements
        PrepareStmt: true,
    })

    if err != nil {
        return nil, err
    }

    // Get underlying sql.DB
    sqlDB, err := db.DB()
    if err != nil {
        return nil, err
    }

    // Connection pool
    sqlDB.SetMaxOpenConns(25)
    sqlDB.SetMaxIdleConns(5)
    sqlDB.SetConnMaxLifetime(time.Hour)

    return db, nil
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                      GORM Best Practices                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use pointer for nullable fields                                  │
│     CategoryID *uint // nullable foreign key                        │
│                                                                      │
│  2. Use map for updates with zero values                             │
│     db.Model(&user).Updates(map[string]interface{}{"age": 0})       │
│                                                                      │
│  3. Check errors!                                                    │
│     if result := db.Create(&user); result.Error != nil { ... }      │
│                                                                      │
│  4. Use AutoMigrate for development only                             │
│     For production: use golang-migrate or similar                   │
│                                                                      │
│  5. Enable PrepareStmt for performance                               │
│     &gorm.Config{PrepareStmt: true}                                 │
│                                                                      │
│  6. Use Scopes for reusable query logic                              │
│     db.Scopes(ActiveUsers, Paginate(1, 10)).Find(&users)           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **GORM** بيبسط التعامل مع الـ database
2. استخدم **Tags** لتحديد الـ schema
3. **AutoMigrate** للـ development فقط
4. استخدم **map** للـ updates مع zero values
5. **Soft Delete** تلقائي لو فيه DeletedAt
6. دايماً **تحقق من الأخطاء**

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [GORM Models](./08-gorm-models.md)**

</div>

---

<div align="center">

[⬅️ السابق](./06-transactions.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./08-gorm-models.md)

</div>
