# Module 4.5: Go + PostgreSQL Integration 🔌

<div dir="rtl">

## نظرة عامة

ربط Go مع PostgreSQL - من Raw SQL لـ GORM ORM.

</div>

---

## 📚 Option 1: database/sql + pgx

### Installation:

```bash
go get github.com/jackc/pgx/v5
go get github.com/jackc/pgx/v5/stdlib
```

### Connection:

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    
    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    // Connection string
    dsn := "postgres://postgres:password@localhost:5432/myapp?sslmode=disable"
    
    // Open connection
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Test connection
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("Connected to PostgreSQL!")
}
```

### CRUD Example:

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
    Age   int
}

func main() {
    dsn := "postgres://postgres:password@localhost:5432/myapp?sslmode=disable"
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // CREATE
    _, err = db.Exec(`
        INSERT INTO users (name, email, age) 
        VALUES ($1, $2, $3)
    `, "Ahmed", "ahmed@test.com", 25)
    if err != nil {
        log.Fatal(err)
    }
    
    // READ
    var user User
    err = db.QueryRow("SELECT id, name, email, age FROM users WHERE email = $1", 
        "ahmed@test.com").Scan(&user.ID, &user.Name, &user.Email, &user.Age)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("User: %+v\n", user)
    
    //UPDATE
    _, err = db.Exec("UPDATE users SET age = $1 WHERE email = $2", 26, "ahmed@test.com")
    
    // DELETE
    _, err = db.Exec("DELETE FROM users WHERE email = $1", "ahmed@test.com")
}
```

---

## 🎯 Option 2: GORM (Recommended)

### Installation:

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
```

### Connection:

```go
package main

import (
    "fmt"
    "log"
    
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func main() {
    dsn := "host=localhost user=postgres password=password dbname=myapp port=5432 sslmode=disable"
    
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("Connected via GORM!")
}
```

### Models & Auto-Migration:

```go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "time"
)

type User struct {
    ID        uint      `gorm:"primaryKey"`
    Name      string    `gorm:"size:100;not null"`
    Email     string    `gorm:"size:100;uniqueIndex;not null"`
    Age       int
    CreatedAt time.Time
    UpdatedAt time.Time
}

func main() {
    dsn := "host=localhost user=postgres password=password dbname=myapp port=5432"
    db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    
    // Auto-migrate (creates table if doesn't exist)
    db.AutoMigrate(&User{})
}
```

### CRUD with GORM:

```go
package main

import (
    "fmt"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

type User struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string
    Email string `gorm:"uniqueIndex"`
    Age   int
}

func main() {
    dsn := "host=localhost user=postgres password=password dbname=myapp port=5432"
    db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    
    // CREATE
    user := User{Name: "Ahmed", Email: "ahmed@test.com", Age: 25}
    db.Create(&user)
    fmt.Println("Created user ID:", user.ID)
    
    // READ
    var foundUser User
    db.First(&foundUser, user.ID)  // Find by ID
    db.Where("email = ?", "ahmed@test.com").First(&foundUser)  // Find by email
    
    // READ ALL
    var users []User
    db.Find(&users)
    
    // UPDATE
    db.Model(&user).Update("age", 26)
    db.Model(&user).Updates(User{Name: "Ahmed Ali", Age: 26})
    
    // DELETE
    db.Delete(&user)
}
```

---

## 🔄 Complete API Example

```go
package main

import (
    "github.com/gin-gonic/gin"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

type User struct {
    ID    uint   `gorm:"primaryKey" json:"id"`
    Name  string `json:"name"`
    Email string `gorm:"uniqueIndex" json:"email"`
    Age   int    `json:"age"`
}

var db *gorm.DB

func initDB() {
    dsn := "host=localhost user=postgres password=password dbname=myapp port=5432"
    var err error
    db, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        panic(err)
    }
    db.AutoMigrate(&User{})
}

func main() {
    initDB()
    
    router := gin.Default()
    
    // GET all users
    router.GET("/users", func(c *gin.Context) {
        var users []User
        db.Find(&users)
        c.JSON(200, users)
    })
    
    // GET user by ID
    router.GET("/users/:id", func(c *gin.Context) {
        var user User
        if err := db.First(&user, c.Param("id")).Error; err != nil {
            c.JSON(404, gin.H{"error": "User not found"})
            return
        }
        c.JSON(200, user)
    })
    
    // CREATE user
    router.POST("/users", func(c *gin.Context) {
        var user User
        if err := c.ShouldBindJSON(&user); err != nil {
            c.JSON(400, gin.H{"error": err.Error()})
            return
        }
        
        if err := db.Create(&user).Error; err != nil {
            c.JSON(500, gin.H{"error": err.Error()})
            return
        }
        
        c.JSON(201, user)
    })
    
    // UPDATE user
    router.PUT("/users/:id", func(c *gin.Context) {
        var user User
        if err := db.First(&user, c.Param("id")).Error; err != nil {
            c.JSON(404, gin.H{"error": "User not found"})
            return
        }
        
        var updateData User
        if err := c.ShouldBindJSON(&updateData); err != nil {
            c.JSON(400, gin.H{"error": err.Error()})
            return
        }
        
        db.Model(&user).Updates(updateData)
        c.JSON(200, user)
    })
    
    // DELETE user
    router.DELETE("/users/:id", func(c *gin.Context) {
        if err := db.Delete(&User{}, c.Param("id")).Error; err != nil {
            c.JSON(404, gin.H{"error": "User not found"})
            return
        }
        c.JSON(200, gin.H{"message": "User deleted"})
    })
    
    router.Run(":8080")
}
```

---

## ✅ Best Practices

```go
// ✅ Use connection pooling
sqlDB, _ := db.DB()
sqlDB.SetMaxOpenConns(25)
sqlDB.SetMaxIdleConns(5)

// ✅ Handle errors properly
if err := db.Create(&user).Error; err != nil {
    // Handle error
}

// ✅ Use transactions
db.Transaction(func(tx *gorm.DB) error {
    tx.Create(&user)
    tx.Create(&profile)
    return nil
})

// ✅ Use environment variables
dsn := os.Getenv("DATABASE_URL")
```

---

## 🎉 Track 4 Complete!

<div dir="rtl">

**تهانينا!** أنهيت Track 4 🚀

**الآن جاهز لـ:**
**➡️ Track 5: Practical Applications** - بناء Full-Stack Apps!

</div>

---

<div align="center">

[⬅️ Previous: Joins](../04-joins-relations/README.md) | [🏠 Track 4](../README.md)

</div>
