# Project 2: Authentication System 🔐

<div dir="rtl">

## نظرة عامة

سنبني نظام **Authentication كامل** مع JWT!

</div>

---

## 🎯 Features

```
✅ User Registration
✅ User Login
✅ JWT Tokens
✅ Password Hashing (bcrypt)
✅ Protected Routes
✅ Refresh Tokens
```

---

## 🔧 Dependencies

```bash
go get github.com/gin-gonic/gin
go get gorm.io/gorm
go get gorm.io/driver/postgres
go get github.com/golang-jwt/jwt/v5
go get golang.org/x/crypto/bcrypt
```

---

## 📝 User Model

```go
package models

import (
    "time"
    "golang.org/x/crypto/bcrypt"
)

type User struct {
    ID        uint      `json:"id" gorm:"primaryKey"`
    Email     string    `json:"email" gorm:"uniqueIndex;not null"`
    Password  string    `json:"-"` // Never return password!
    Name      string    `json:"name"`
    CreatedAt time.Time `json:"created_at"`
}

// Hash password before saving
func (u *User) HashPassword(password string) error {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
    if err != nil {
        return err
    }
    u.Password = string(bytes)
    return nil
}

// Check password
func (u *User) CheckPassword(password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(u.Password), []byte(password))
    return err == nil
}
```

---

## 🎫 JWT Helper

```go
package auth

import (
    "errors"
    "time"

    "github.com/golang-jwt/jwt/v5"
)

var jwtKey = []byte("your-secret-key-change-in-production")

type Claims struct {
    UserID uint   `json:"user_id"`
    Email  string `json:"email"`
    jwt.RegisteredClaims
}

// Generate JWT
func GenerateToken(userID uint, email string) (string, error) {
    claims := &Claims{
        UserID: userID,
        Email:  email,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(jwtKey)
}

// Verify JWT
func VerifyToken(tokenString string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        return jwtKey, nil
    })

    if err != nil {
        return nil, err
    }

    if claims, ok := token.Claims.(*Claims); ok && token.Valid {
        return claims, nil
    }

    return nil, errors.New("invalid token")
}
```

---

## 🎮 Auth Handlers

```go
package handlers

import (
    "net/http"
    "auth-api/models"
    "auth-api/database"
    "auth-api/auth"

    "github.com/gin-gonic/gin"
)

// Register
func Register(c *gin.Context) {
    var input struct {
        Email    string `json:"email" binding:"required,email"`
        Password string `json:"password" binding:"required,min=6"`
        Name     string `json:"name" binding:"required"`
    }

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // Check if user exists
    var existingUser models.User
    if database.DB.Where("email = ?", input.Email).First(&existingUser).Error == nil {
        c.JSON(http.StatusConflict, gin.H{"error": "Email already registered"})
        return
    }

    // Create user
    user := models.User{
        Email: input.Email,
        Name:  input.Name,
    }

    if err := user.HashPassword(input.Password); err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to hash password"})
        return
    }

    if err := database.DB.Create(&user).Error; err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusCreated, gin.H{
        "message": "User registered successfully",
        "user":    user,
    })
}

// Login
func Login(c *gin.Context) {
    var input struct {
        Email    string `json:"email" binding:"required,email"`
        Password string `json:"password" binding:"required"`
    }

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // Find user
    var user models.User
    if err := database.DB.Where("email = ?", input.Email).First(&user).Error; err != nil {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
        return
    }

    // Check password
    if !user.CheckPassword(input.Password) {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
        return
    }

    // Generate token
    token, err := auth.GenerateToken(user.ID, user.Email)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to generate token"})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "token": token,
        "user":  user,
    })
}

// Get Profile (Protected)
func GetProfile(c *gin.Context) {
    userID, _ := c.Get("user_id")

    var user models.User
    if err := database.DB.First(&user, userID).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    c.JSON(http.StatusOK, user)
}
```

---

## 🔒 Auth Middleware

```go
package middleware

import (
    "net/http"
    "strings"
    "auth-api/auth"

    "github.com/gin-gonic/gin"
)

func AuthRequired() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Get token from header
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "Authorization header required",
            })
            return
        }

        // Check Bearer format
        parts := strings.Split(authHeader, " ")
        if len(parts) != 2 || parts[0] != "Bearer" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "Invalid authorization header format",
            })
            return
        }

        // Verify token
        claims, err := auth.VerifyToken(parts[1])
        if err != nil {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "Invalid or expired token",
            })
            return
        }

        // Set user info in context
        c.Set("user_id", claims.UserID)
        c.Set("user_email", claims.Email)

        c.Next()
    }
}
```

---

## 🚀 Main Application

```go
package main

import (
    "auth-api/database"
    "auth-api/handlers"
    "auth-api/middleware"

    "github.com/gin-gonic/gin"
)

func main() {
    database.Connect()

    router := gin.Default()

    // Public routes
    router.POST("/register", handlers.Register)
    router.POST("/login", handlers.Login)

    // Protected routes
    protected := router.Group("/api")
    protected.Use(middleware.AuthRequired())
    {
        protected.GET("/profile", handlers.GetProfile)
    }

    router.Run(":8080")
}
```

---

## 🧪 Testing

```bash
# Register
curl -X POST http://localhost:8080/register \
  -H "Content-Type: application/json" \
  -d '{"email":"ahmed@test.com","password":"123456","name":"Ahmed"}'

# Login
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ahmed@test.com","password":"123456"}'
# Output: {"token":"eyJhbGc...","user":{...}}

# Get Profile (with token)
curl http://localhost:8080/api/profile \
  -H "Authorization: Bearer eyJhbGc..."
```

---

## ✅ What You Learned

- ✅ User registration & login
- ✅ Password hashing (bcrypt)
- ✅ JWT generation & verification
- ✅ Protected routes
- ✅ Middleware
- ✅ Security best practices

---

## ⏭️ Next Project

**➡️ [Project 3: Full CRUD with Database](../03-crud-with-db/README.md)**

---

<div align="center">

[⬅️ Previous: REST API](../01-rest-api-project/README.md) | [🏠 Track 5](../README.md)

</div>
