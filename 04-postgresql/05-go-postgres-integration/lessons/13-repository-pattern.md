# Repository Pattern - نمط الـ Repository 🏗️

<div dir="rtl">

## مقدمة

الـ Repository Pattern بيفصل بين منطق التطبيق وطبقة البيانات. ده بيسهل الاختبار والصيانة وتغيير الـ database لو احتجت.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 لماذا Repository Pattern؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Without Repository                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Handler                                                            │
│   ┌────────────────────────────────────────────────────────────┐    │
│   │  func GetUser(c *gin.Context) {                            │    │
│   │      id := c.Param("id")                                   │    │
│   │      var user User                                          │    │
│   │      db.First(&user, id)  // ← Direct DB access            │    │
│   │      c.JSON(200, user)                                      │    │
│   │  }                                                          │    │
│   └────────────────────────────────────────────────────────────┘    │
│                                                                      │
│   ❌ Hard to test (needs real database)                             │
│   ❌ Business logic mixed with data access                          │
│   ❌ Hard to change database                                        │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                     With Repository                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Handler → Service → Repository → Database                         │
│                                                                      │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│   │   Handler   │───▶│   Service   │───▶│ Repository  │──▶ DB      │
│   │ (HTTP/gRPC) │    │  (Business) │    │   (Data)    │            │
│   └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                      │
│   ✅ Easy to test (mock repository)                                 │
│   ✅ Clean separation                                                │
│   ✅ Can swap database easily                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📁 هيكل المشروع

```
myproject/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── domain/
│   │   ├── user.go           # Models & Interfaces
│   │   └── order.go
│   ├── repository/
│   │   ├── repository.go     # Repository interfaces
│   │   ├── postgres/
│   │   │   ├── user.go       # PostgreSQL implementation
│   │   │   └── order.go
│   │   └── memory/
│   │       └── user.go       # In-memory for testing
│   ├── service/
│   │   ├── user.go           # Business logic
│   │   └── order.go
│   └── handler/
│       ├── user.go           # HTTP handlers
│       └── order.go
├── pkg/
│   └── database/
│       └── postgres.go
└── go.mod
```

---

## 📦 Domain Models

```go
// internal/domain/user.go
package domain

import (
    "context"
    "time"
)

// Entity
type User struct {
    ID           uint      `json:"id"`
    Username     string    `json:"username"`
    Email        string    `json:"email"`
    PasswordHash string    `json:"-"`
    FullName     string    `json:"full_name,omitempty"`
    IsActive     bool      `json:"is_active"`
    Role         string    `json:"role"`
    CreatedAt    time.Time `json:"created_at"`
    UpdatedAt    time.Time `json:"updated_at"`
}

// DTOs
type CreateUserInput struct {
    Username string `json:"username" validate:"required,min=3,max=50"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
    FullName string `json:"full_name,omitempty"`
}

type UpdateUserInput struct {
    Email    *string `json:"email,omitempty" validate:"omitempty,email"`
    FullName *string `json:"full_name,omitempty"`
    IsActive *bool   `json:"is_active,omitempty"`
}

type UserFilter struct {
    Username    *string
    Email       *string
    IsActive    *bool
    Role        *string
    CreatedFrom *time.Time
    CreatedTo   *time.Time
}

// Repository Interface
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    GetByID(ctx context.Context, id uint) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
    GetByUsername(ctx context.Context, username string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id uint) error
    List(ctx context.Context, filter UserFilter, limit, offset int) ([]User, int64, error)
}
```

---

## 🏗️ PostgreSQL Implementation

```go
// internal/repository/postgres/user.go
package postgres

import (
    "context"
    "errors"
    "fmt"

    "myproject/internal/domain"
    "gorm.io/gorm"
)

type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) domain.UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    return r.db.WithContext(ctx).Create(user).Error
}

func (r *userRepository) GetByID(ctx context.Context, id uint) (*domain.User, error) {
    var user domain.User
    err := r.db.WithContext(ctx).First(&user, id).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, nil
        }
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) GetByEmail(ctx context.Context, email string) (*domain.User, error) {
    var user domain.User
    err := r.db.WithContext(ctx).Where("email = ?", email).First(&user).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, nil
        }
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) GetByUsername(ctx context.Context, username string) (*domain.User, error) {
    var user domain.User
    err := r.db.WithContext(ctx).Where("username = ?", username).First(&user).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, nil
        }
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) Update(ctx context.Context, user *domain.User) error {
    return r.db.WithContext(ctx).Save(user).Error
}

func (r *userRepository) Delete(ctx context.Context, id uint) error {
    result := r.db.WithContext(ctx).Delete(&domain.User{}, id)
    if result.Error != nil {
        return result.Error
    }
    if result.RowsAffected == 0 {
        return fmt.Errorf("user %d not found", id)
    }
    return nil
}

func (r *userRepository) List(ctx context.Context, filter domain.UserFilter, limit, offset int) ([]domain.User, int64, error) {
    var users []domain.User
    var total int64

    query := r.db.WithContext(ctx).Model(&domain.User{})

    // Apply filters
    if filter.Username != nil {
        query = query.Where("username ILIKE ?", "%"+*filter.Username+"%")
    }
    if filter.Email != nil {
        query = query.Where("email ILIKE ?", "%"+*filter.Email+"%")
    }
    if filter.IsActive != nil {
        query = query.Where("is_active = ?", *filter.IsActive)
    }
    if filter.Role != nil {
        query = query.Where("role = ?", *filter.Role)
    }
    if filter.CreatedFrom != nil {
        query = query.Where("created_at >= ?", *filter.CreatedFrom)
    }
    if filter.CreatedTo != nil {
        query = query.Where("created_at <= ?", *filter.CreatedTo)
    }

    // Count total
    if err := query.Count(&total).Error; err != nil {
        return nil, 0, err
    }

    // Get page
    if err := query.Order("created_at DESC").Offset(offset).Limit(limit).Find(&users).Error; err != nil {
        return nil, 0, err
    }

    return users, total, nil
}
```

---

## 🏢 Service Layer

```go
// internal/service/user.go
package service

import (
    "context"
    "errors"
    "fmt"

    "myproject/internal/domain"
    "golang.org/x/crypto/bcrypt"
)

var (
    ErrUserNotFound     = errors.New("user not found")
    ErrEmailExists      = errors.New("email already exists")
    ErrUsernameExists   = errors.New("username already exists")
    ErrInvalidPassword  = errors.New("invalid password")
)

type UserService struct {
    repo domain.UserRepository
}

func NewUserService(repo domain.UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) Create(ctx context.Context, input domain.CreateUserInput) (*domain.User, error) {
    // Check if email exists
    existing, err := s.repo.GetByEmail(ctx, input.Email)
    if err != nil {
        return nil, fmt.Errorf("check email: %w", err)
    }
    if existing != nil {
        return nil, ErrEmailExists
    }

    // Check if username exists
    existing, err = s.repo.GetByUsername(ctx, input.Username)
    if err != nil {
        return nil, fmt.Errorf("check username: %w", err)
    }
    if existing != nil {
        return nil, ErrUsernameExists
    }

    // Hash password
    hash, err := bcrypt.GenerateFromPassword([]byte(input.Password), bcrypt.DefaultCost)
    if err != nil {
        return nil, fmt.Errorf("hash password: %w", err)
    }

    user := &domain.User{
        Username:     input.Username,
        Email:        input.Email,
        PasswordHash: string(hash),
        FullName:     input.FullName,
        IsActive:     true,
        Role:         "user",
    }

    if err := s.repo.Create(ctx, user); err != nil {
        return nil, fmt.Errorf("create user: %w", err)
    }

    return user, nil
}

func (s *UserService) GetByID(ctx context.Context, id uint) (*domain.User, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("get user: %w", err)
    }
    if user == nil {
        return nil, ErrUserNotFound
    }
    return user, nil
}

func (s *UserService) Update(ctx context.Context, id uint, input domain.UpdateUserInput) (*domain.User, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        return nil, err
    }
    if user == nil {
        return nil, ErrUserNotFound
    }

    // Check email uniqueness if changing
    if input.Email != nil && *input.Email != user.Email {
        existing, err := s.repo.GetByEmail(ctx, *input.Email)
        if err != nil {
            return nil, err
        }
        if existing != nil {
            return nil, ErrEmailExists
        }
        user.Email = *input.Email
    }

    if input.FullName != nil {
        user.FullName = *input.FullName
    }
    if input.IsActive != nil {
        user.IsActive = *input.IsActive
    }

    if err := s.repo.Update(ctx, user); err != nil {
        return nil, fmt.Errorf("update user: %w", err)
    }

    return user, nil
}

func (s *UserService) Delete(ctx context.Context, id uint) error {
    return s.repo.Delete(ctx, id)
}

func (s *UserService) List(ctx context.Context, filter domain.UserFilter, page, pageSize int) ([]domain.User, int64, error) {
    if page < 1 {
        page = 1
    }
    if pageSize < 1 || pageSize > 100 {
        pageSize = 20
    }
    offset := (page - 1) * pageSize

    return s.repo.List(ctx, filter, pageSize, offset)
}

func (s *UserService) Authenticate(ctx context.Context, email, password string) (*domain.User, error) {
    user, err := s.repo.GetByEmail(ctx, email)
    if err != nil {
        return nil, err
    }
    if user == nil {
        return nil, ErrUserNotFound
    }

    if err := bcrypt.CompareHashAndPassword([]byte(user.PasswordHash), []byte(password)); err != nil {
        return nil, ErrInvalidPassword
    }

    if !user.IsActive {
        return nil, errors.New("user is not active")
    }

    return user, nil
}
```

---

## 🌐 HTTP Handler

```go
// internal/handler/user.go
package handler

import (
    "errors"
    "net/http"
    "strconv"

    "myproject/internal/domain"
    "myproject/internal/service"
    "github.com/gin-gonic/gin"
)

type UserHandler struct {
    service *service.UserService
}

func NewUserHandler(service *service.UserService) *UserHandler {
    return &UserHandler{service: service}
}

func (h *UserHandler) RegisterRoutes(r *gin.RouterGroup) {
    users := r.Group("/users")
    {
        users.POST("", h.Create)
        users.GET("", h.List)
        users.GET("/:id", h.GetByID)
        users.PUT("/:id", h.Update)
        users.DELETE("/:id", h.Delete)
    }
}

func (h *UserHandler) Create(c *gin.Context) {
    var input domain.CreateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.service.Create(c.Request.Context(), input)
    if err != nil {
        if errors.Is(err, service.ErrEmailExists) || errors.Is(err, service.ErrUsernameExists) {
            c.JSON(http.StatusConflict, gin.H{"error": err.Error()})
            return
        }
        c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
        return
    }

    c.JSON(http.StatusCreated, user)
}

func (h *UserHandler) GetByID(c *gin.Context) {
    id, err := strconv.ParseUint(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid id"})
        return
    }

    user, err := h.service.GetByID(c.Request.Context(), uint(id))
    if err != nil {
        if errors.Is(err, service.ErrUserNotFound) {
            c.JSON(http.StatusNotFound, gin.H{"error": "user not found"})
            return
        }
        c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
        return
    }

    c.JSON(http.StatusOK, user)
}

func (h *UserHandler) Update(c *gin.Context) {
    id, err := strconv.ParseUint(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid id"})
        return
    }

    var input domain.UpdateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.service.Update(c.Request.Context(), uint(id), input)
    if err != nil {
        if errors.Is(err, service.ErrUserNotFound) {
            c.JSON(http.StatusNotFound, gin.H{"error": "user not found"})
            return
        }
        if errors.Is(err, service.ErrEmailExists) {
            c.JSON(http.StatusConflict, gin.H{"error": err.Error()})
            return
        }
        c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
        return
    }

    c.JSON(http.StatusOK, user)
}

func (h *UserHandler) Delete(c *gin.Context) {
    id, err := strconv.ParseUint(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid id"})
        return
    }

    if err := h.service.Delete(c.Request.Context(), uint(id)); err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
        return
    }

    c.JSON(http.StatusOK, gin.H{"message": "deleted"})
}

func (h *UserHandler) List(c *gin.Context) {
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    pageSize, _ := strconv.Atoi(c.DefaultQuery("page_size", "20"))

    filter := domain.UserFilter{}
    if username := c.Query("username"); username != "" {
        filter.Username = &username
    }
    if email := c.Query("email"); email != "" {
        filter.Email = &email
    }

    users, total, err := h.service.List(c.Request.Context(), filter, page, pageSize)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "users":       users,
        "total":       total,
        "page":        page,
        "page_size":   pageSize,
        "total_pages": (total + int64(pageSize) - 1) / int64(pageSize),
    })
}
```

---

## 🔌 Wiring (main.go)

```go
// cmd/api/main.go
package main

import (
    "log"
    "os"

    "myproject/internal/handler"
    "myproject/internal/repository/postgres"
    "myproject/internal/service"

    "github.com/gin-gonic/gin"
    pgdriver "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func main() {
    // Database
    dsn := os.Getenv("DATABASE_URL")
    db, err := gorm.Open(pgdriver.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // Repositories
    userRepo := postgres.NewUserRepository(db)

    // Services
    userService := service.NewUserService(userRepo)

    // Handlers
    userHandler := handler.NewUserHandler(userService)

    // Router
    r := gin.Default()
    api := r.Group("/api/v1")

    userHandler.RegisterRoutes(api)

    // Start
    if err := r.Run(":8080"); err != nil {
        log.Fatal(err)
    }
}
```

---

## 🧪 Testing with Mock

```go
// internal/repository/mock/user.go
package mock

import (
    "context"
    "sync"

    "myproject/internal/domain"
)

type userRepository struct {
    mu    sync.RWMutex
    users map[uint]*domain.User
    id    uint
}

func NewUserRepository() domain.UserRepository {
    return &userRepository{
        users: make(map[uint]*domain.User),
    }
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    r.id++
    user.ID = r.id
    r.users[user.ID] = user
    return nil
}

func (r *userRepository) GetByID(ctx context.Context, id uint) (*domain.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    user, ok := r.users[id]
    if !ok {
        return nil, nil
    }
    return user, nil
}

// ... implement other methods

// Test
func TestUserService_Create(t *testing.T) {
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)

    user, err := svc.Create(context.Background(), domain.CreateUserInput{
        Username: "test",
        Email:    "test@example.com",
        Password: "password123",
    })

    assert.NoError(t, err)
    assert.NotNil(t, user)
    assert.Equal(t, "test", user.Username)
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Repository Pattern Best Practices                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Interface in domain package                                      │
│     Repository interface defines the contract                       │
│                                                                      │
│  2. Implementation in separate package                               │
│     postgres/, memory/, redis/                                      │
│                                                                      │
│  3. Service layer for business logic                                 │
│     Validation, password hashing, etc.                              │
│                                                                      │
│  4. Handler only handles HTTP                                        │
│     Parse request, call service, return response                    │
│                                                                      │
│  5. Use Context everywhere                                           │
│     For cancellation and timeouts                                   │
│                                                                      │
│  6. Return domain errors from service                                │
│     ErrUserNotFound, ErrEmailExists                                 │
│                                                                      │
│  7. Use mock repository for testing                                  │
│     Fast, no database needed                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Repository** يفصل الـ data access عن الـ business logic
2. استخدم **interfaces** في الـ domain package
3. **Service layer** للـ business logic
4. **Handler** للـ HTTP فقط
5. **Mock repository** لاختبار سريع
6. استخدم **Context** في كل العمليات

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Error Handling](./14-error-handling.md)**

</div>

---

<div align="center">

[⬅️ السابق](./12-golang-migrate.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./14-error-handling.md)

</div>
