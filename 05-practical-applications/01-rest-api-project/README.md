# Project 1: REST API from Scratch 🔌

<div dir="rtl">

## نظرة عامة

سنبني **User Management API** كاملة من الصفر!

</div>

---

## 🎯 What We'll Build

```
API Endpoints:
POST   /api/users        - Create user
GET    /api/users        - List all users
GET    /api/users/:id    - Get user by ID
PUT    /api/users/:id    - Update user
DELETE /api/users/:id    - Delete user
```

---

## 📁 Project Structure

```
user-api/
├── main.go
├── models/
│   └── user.go
├── handlers/
│   └── user_handler.go
├── database/
│   └── db.go
├── go.mod
└── go.sum
```

---

## 🔧 Step 1: Initialize Project

```bash
mkdir user-api
cd user-api
go mod init user-api

# Install dependencies
go get github.com/gin-gonic/gin
go get gorm.io/gorm
go get gorm.io/driver/postgres
```

---

## 📝 Step 2: Create Models

**`models/user.go`:**

```go
package models

import "time"

type User struct {
    ID        uint      `json:"id" gorm:"primaryKey"`
    Name      string    `json:"name" binding:"required"`
    Email     string    `json:"email" binding:"required,email" gorm:"uniqueIndex"`
    Age       int       `json:"age" binding:"required,min=1,max=150"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}
```

---

## 💾 Step 3: Database Connection

**`database/db.go`:**

```go
package database

import (
    "log"
    "user-api/models"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

var DB *gorm.DB

func Connect() {
    dsn := "host=localhost user=postgres password=password dbname=userapi port=5432 sslmode=disable"

    var err error
    DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // Auto-migrate
    DB.AutoMigrate(&models.User{})

    log.Println("✓ Database connected")
}
```

---

## 🎮 Step 4: Handlers

**`handlers/user_handler.go`:**

```go
package handlers

import (
    "net/http"
    "user-api/database"
    "user-api/models"

    "github.com/gin-gonic/gin"
)

// Create User
func CreateUser(c *gin.Context) {
    var user models.User

    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    if err := database.DB.Create(&user).Error; err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusCreated, user)
}

// Get All Users
func GetAllUsers(c *gin.Context) {
    var users []models.User
    database.DB.Find(&users)
    c.JSON(http.StatusOK, users)
}

// Get User by ID
func GetUser(c *gin.Context) {
    var user models.User
    id := c.Param("id")

    if err := database.DB.First(&user, id).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    c.JSON(http.StatusOK, user)
}

// Update User
func UpdateUser(c *gin.Context) {
    var user models.User
    id := c.Param("id")

    if err := database.DB.First(&user, id).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    var updateData models.User
    if err := c.ShouldBindJSON(&updateData); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    database.DB.Model(&user).Updates(updateData)
    c.JSON(http.StatusOK, user)
}

// Delete User
func DeleteUser(c *gin.Context) {
    id := c.Param("id")

    if err := database.DB.Delete(&models.User{}, id).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    c.JSON(http.StatusOK, gin.H{"message": "User deleted successfully"})
}
```

---

## 🚀 Step 5: Main Application

**`main.go`:**

```go
package main

import (
    "user-api/database"
    "user-api/handlers"

    "github.com/gin-gonic/gin"
)

func main() {
    // Connect to database
    database.Connect()

    // Create router
    router := gin.Default()

    // Routes
    api := router.Group("/api")
    {
        api.POST("/users", handlers.CreateUser)
        api.GET("/users", handlers.GetAllUsers)
        api.GET("/users/:id", handlers.GetUser)
        api.PUT("/users/:id", handlers.UpdateUser)
        api.DELETE("/users/:id", handlers.DeleteUser)
    }

    // Start server
    router.Run(":8080")
}
```

---

## ▶️ Run the Project

```bash
# Make sure PostgreSQL is running
# Create database
createdb userapi

# Run application
go run main.go

# Server running on http://localhost:8080
```

---

## 🧪 Test with Postman/cURL

```bash
# Create User
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Ahmed","email":"ahmed@test.com","age":25}'

# Get All Users
curl http://localhost:8080/api/users

# Get Specific User
curl http://localhost:8080/api/users/1

# Update User
curl -X PUT http://localhost:8080/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Ahmed Ali","age":26}'

# Delete User
curl -X DELETE http://localhost:8080/api/users/1
```

---

## ✅ What You Learned

- ✅ Gin framework setup
- ✅ GORM ORM integration
- ✅ REST API structure
- ✅ Input validation
- ✅ Error handling
- ✅ CRUD operations

---

## ⏭️ Next Project

<div dir="rtl">

**➡️ [Project 2: Authentication System](../02-authentication-system/README.md)**

</div>

---

<div align="center">

[🏠 Track 5](../README.md)

</div>
