# Lesson 2: Layered Architecture - العمارة الطبقية 📚

<div dir="rtl">

## المقدمة

**Layered Architecture** من أشهر الـ patterns! الفكرة بسيطة: قسّم الكود إلى طبقات، كل طبقة لها مسؤولية واضحة.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 The Layers

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Layered Architecture                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Presentation Layer                         │    │
│  │              (HTTP Handlers, Controllers)                     │    │
│  │           Receives requests, returns responses                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     Business Layer                            │    │
│  │                  (Services, Use Cases)                        │    │
│  │            Contains business logic and rules                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Persistence Layer                          │    │
│  │                (Repositories, DAOs)                           │    │
│  │              Handles data storage/retrieval                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     Database Layer                            │    │
│  │              (PostgreSQL, MongoDB, etc.)                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Rule: Each layer only talks to the layer directly below it!        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Layer Responsibilities

### Presentation Layer (Handler)

```go
// internal/handler/user_handler.go
package handler

type UserHandler struct {
    userService service.UserService
}

func NewUserHandler(userService service.UserService) *UserHandler {
    return &UserHandler{userService: userService}
}

// Responsibilities:
// ✓ Parse HTTP request
// ✓ Validate input format
// ✓ Call service layer
// ✓ Format response
// ✗ NO business logic!
// ✗ NO database queries!

func (h *UserHandler) GetUser(c *gin.Context) {
    // 1. Parse request
    id, err := strconv.ParseInt(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(400, gin.H{"error": "Invalid ID"})
        return
    }

    // 2. Call service (business layer)
    user, err := h.userService.GetUser(c.Request.Context(), id)
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }

    // 3. Format response
    c.JSON(200, UserResponse{
        ID:    user.ID,
        Name:  user.Name,
        Email: user.Email,
    })
}

func (h *UserHandler) CreateUser(c *gin.Context) {
    // 1. Parse & validate request
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }

    // 2. Call service
    user, err := h.userService.CreateUser(c.Request.Context(), service.CreateUserInput{
        Name:     req.Name,
        Email:    req.Email,
        Password: req.Password,
    })
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }

    // 3. Response
    c.JSON(201, UserResponse{
        ID:    user.ID,
        Name:  user.Name,
        Email: user.Email,
    })
}

// Request/Response DTOs (Data Transfer Objects)
type CreateUserRequest struct {
    Name     string `json:"name" binding:"required"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
}

type UserResponse struct {
    ID    int64  `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
    // Note: NO password in response!
}
```

### Business Layer (Service)

```go
// internal/service/user_service.go
package service

type UserService interface {
    GetUser(ctx context.Context, id int64) (*domain.User, error)
    CreateUser(ctx context.Context, input CreateUserInput) (*domain.User, error)
    UpdateUser(ctx context.Context, id int64, input UpdateUserInput) (*domain.User, error)
}

type userService struct {
    repo         repository.UserRepository
    passwordHash PasswordHasher
    emailSender  EmailSender
}

func NewUserService(
    repo repository.UserRepository,
    passwordHash PasswordHasher,
    emailSender EmailSender,
) UserService {
    return &userService{
        repo:         repo,
        passwordHash: passwordHash,
        emailSender:  emailSender,
    }
}

// Responsibilities:
// ✓ Business logic
// ✓ Validation of business rules
// ✓ Coordinate between repositories
// ✗ NO HTTP handling!
// ✗ NO direct SQL queries!

func (s *userService) CreateUser(ctx context.Context, input CreateUserInput) (*domain.User, error) {
    // 1. Business validation
    if err := s.validateEmail(input.Email); err != nil {
        return nil, err
    }

    // 2. Check if email already exists (business rule)
    existing, _ := s.repo.FindByEmail(ctx, input.Email)
    if existing != nil {
        return nil, ErrEmailAlreadyExists
    }

    // 3. Hash password (business logic)
    hashedPassword, err := s.passwordHash.Hash(input.Password)
    if err != nil {
        return nil, err
    }

    // 4. Create domain entity
    user := &domain.User{
        Name:         input.Name,
        Email:        input.Email,
        PasswordHash: hashedPassword,
        CreatedAt:    time.Now(),
    }

    // 5. Persist via repository
    if err := s.repo.Create(ctx, user); err != nil {
        return nil, err
    }

    // 6. Side effects (send welcome email)
    go s.emailSender.SendWelcome(user.Email, user.Name)

    return user, nil
}

func (s *userService) GetUser(ctx context.Context, id int64) (*domain.User, error) {
    user, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, err
    }
    if user == nil {
        return nil, ErrUserNotFound
    }
    return user, nil
}

type CreateUserInput struct {
    Name     string
    Email    string
    Password string
}
```

### Persistence Layer (Repository)

```go
// internal/repository/user_repository.go
package repository

type UserRepository interface {
    Create(ctx context.Context, user *domain.User) error
    FindByID(ctx context.Context, id int64) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    Update(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id int64) error
}

type postgresUserRepository struct {
    db *sql.DB
}

func NewPostgresUserRepository(db *sql.DB) UserRepository {
    return &postgresUserRepository{db: db}
}

// Responsibilities:
// ✓ Database queries
// ✓ Map between domain entities and DB rows
// ✗ NO business logic!
// ✗ NO HTTP handling!

func (r *postgresUserRepository) Create(ctx context.Context, user *domain.User) error {
    query := `
        INSERT INTO users (name, email, password_hash, created_at)
        VALUES ($1, $2, $3, $4)
        RETURNING id
    `
    return r.db.QueryRowContext(ctx, query,
        user.Name,
        user.Email,
        user.PasswordHash,
        user.CreatedAt,
    ).Scan(&user.ID)
}

func (r *postgresUserRepository) FindByID(ctx context.Context, id int64) (*domain.User, error) {
    query := `
        SELECT id, name, email, password_hash, created_at
        FROM users
        WHERE id = $1
    `
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Name,
        &user.Email,
        &user.PasswordHash,
        &user.CreatedAt,
    )
    if err == sql.ErrNoRows {
        return nil, nil
    }
    if err != nil {
        return nil, err
    }
    return user, nil
}

func (r *postgresUserRepository) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    query := `SELECT id, name, email, password_hash, created_at FROM users WHERE email = $1`
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &user.ID, &user.Name, &user.Email, &user.PasswordHash, &user.CreatedAt,
    )
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return user, err
}
```

### Domain Layer (Entities)

```go
// internal/domain/user.go
package domain

// Domain entity - pure business object
// No database tags, no JSON tags (those belong to other layers)
type User struct {
    ID           int64
    Name         string
    Email        string
    PasswordHash string
    CreatedAt    time.Time
    UpdatedAt    time.Time
}

// Business methods on domain entity
func (u *User) IsEmailVerified() bool {
    // Business logic
    return u.EmailVerifiedAt != nil
}

func (u *User) CanCreateOrder() bool {
    // Business rule
    return u.IsEmailVerified() && u.Status == "active"
}
```

---

## 2️⃣ Project Structure

```
myapp/
├── cmd/
│   └── server/
│       └── main.go           # Entry point, wiring
├── internal/
│   ├── domain/               # Domain entities
│   │   ├── user.go
│   │   └── order.go
│   ├── repository/           # Persistence layer
│   │   ├── interface.go      # Repository interfaces
│   │   ├── user_repository.go
│   │   └── order_repository.go
│   ├── service/              # Business layer
│   │   ├── user_service.go
│   │   └── order_service.go
│   ├── handler/              # Presentation layer
│   │   ├── user_handler.go
│   │   ├── order_handler.go
│   │   └── dto/              # Request/Response types
│   │       ├── user_dto.go
│   │       └── order_dto.go
│   └── config/
│       └── config.go
├── pkg/                      # Reusable packages
├── go.mod
└── go.sum
```

---

## 3️⃣ Wiring Everything Together

```go
// cmd/server/main.go
package main

func main() {
    // 1. Load config
    cfg := config.Load()

    // 2. Setup database
    db, err := sql.Open("postgres", cfg.DatabaseURL)
    if err != nil {
        log.Fatal(err)
    }

    // 3. Create repositories (persistence layer)
    userRepo := repository.NewPostgresUserRepository(db)
    orderRepo := repository.NewPostgresOrderRepository(db)

    // 4. Create services (business layer)
    passwordHasher := service.NewBcryptHasher()
    emailSender := service.NewSMTPSender(cfg.SMTPConfig)

    userService := service.NewUserService(userRepo, passwordHasher, emailSender)
    orderService := service.NewOrderService(orderRepo, userRepo)

    // 5. Create handlers (presentation layer)
    userHandler := handler.NewUserHandler(userService)
    orderHandler := handler.NewOrderHandler(orderService)

    // 6. Setup router
    r := gin.Default()

    r.GET("/users/:id", userHandler.GetUser)
    r.POST("/users", userHandler.CreateUser)
    r.GET("/orders/:id", orderHandler.GetOrder)
    r.POST("/orders", orderHandler.CreateOrder)

    // 7. Start server
    r.Run(":8080")
}
```

---

## 4️⃣ Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Request Flow Example                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  POST /users                                                        │
│  {"name": "Ahmed", "email": "ahmed@test.com", "password": "123456"} │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Handler Layer                              │    │
│  │  1. Parse JSON into CreateUserRequest                        │    │
│  │  2. Validate request format                                  │    │
│  │  3. Call userService.CreateUser()                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Service Layer                              │    │
│  │  1. Validate email format (business rule)                    │    │
│  │  2. Check email uniqueness via repo                          │    │
│  │  3. Hash password                                            │    │
│  │  4. Create User entity                                       │    │
│  │  5. Call repo.Create()                                       │    │
│  │  6. Send welcome email                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Repository Layer                           │    │
│  │  1. Execute INSERT query                                     │    │
│  │  2. Return created user with ID                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Database                                   │    │
│  │  INSERT INTO users (...) VALUES (...) RETURNING id           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ Benefits & Trade-offs

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Benefits                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Separation of Concerns                                          │
│     Each layer has one responsibility                               │
│                                                                      │
│  ✅ Testability                                                     │
│     Easy to mock layers for unit testing                            │
│                                                                      │
│  ✅ Maintainability                                                 │
│     Changes isolated to specific layer                              │
│                                                                      │
│  ✅ Understandability                                               │
│     Clear structure, easy to navigate                               │
│                                                                      │
│  ✅ Flexibility                                                     │
│     Easy to swap implementations (e.g., change database)            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    Trade-offs                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ⚠️ More Code                                                       │
│     Interfaces, DTOs, mapping between layers                        │
│                                                                      │
│  ⚠️ Layer Leakage                                                   │
│     Easy to accidentally skip layers                                │
│                                                                      │
│  ⚠️ Performance                                                     │
│     Data transformation between layers                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Handler** = HTTP فقط (parse, validate format, respond)
- ✅ **Service** = Business logic (rules, validation, coordination)
- ✅ **Repository** = Database queries فقط
- ✅ **Domain** = Business entities (pure objects)
- ✅ كل طبقة تتكلم مع اللي تحتها بس!
- ✅ استخدم Interfaces للـ loose coupling

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم Clean Architecture:

**➡️ [Lesson 3: Clean Architecture](./03-clean-architecture.md)**

</div>

---

<div align="center">

[⬅️ Previous: Monolith vs Microservices](./01-monolith-vs-microservices.md) | [📚 Module Home](../README.md) | [➡️ Next: Clean Architecture](./03-clean-architecture.md)

</div>
