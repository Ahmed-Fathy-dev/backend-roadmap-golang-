# Lesson 3: Clean Architecture - العمارة النظيفة 🎯

<div dir="rtl">

## المقدمة

**Clean Architecture** (أو Hexagonal/Onion Architecture) هي pattern متقدم يضع الـ Business Logic في المركز ويجعل الـ framework والـ database تفاصيل خارجية.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 The Dependency Rule

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Clean Architecture                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│          ┌───────────────────────────────────────────┐              │
│          │           External Layer                   │              │
│          │    (Frameworks, DB, UI, Web, Devices)     │              │
│          │  ┌───────────────────────────────────┐    │              │
│          │  │        Interface Adapters          │    │              │
│          │  │   (Controllers, Gateways, Repos)   │    │              │
│          │  │  ┌───────────────────────────┐    │    │              │
│          │  │  │      Application Layer     │    │    │              │
│          │  │  │       (Use Cases)          │    │    │              │
│          │  │  │  ┌───────────────────┐    │    │    │              │
│          │  │  │  │                   │    │    │    │              │
│          │  │  │  │     Entities      │    │    │    │              │
│          │  │  │  │  (Domain Layer)   │    │    │    │              │
│          │  │  │  │                   │    │    │    │              │
│          │  │  │  └───────────────────┘    │    │    │              │
│          │  │  └───────────────────────────┘    │    │              │
│          │  └───────────────────────────────────┘    │              │
│          └───────────────────────────────────────────┘              │
│                                                                      │
│  🎯 THE DEPENDENCY RULE:                                            │
│     Dependencies point INWARD only!                                 │
│     Inner layers know NOTHING about outer layers.                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ The Layers Explained

### Entities (Domain Layer) - المركز

```go
// internal/domain/user.go
package domain

// Entity - Pure business object
// NO dependencies on anything external!
type User struct {
    ID        int64
    Email     string
    Name      string
    Password  string
    Status    UserStatus
    CreatedAt time.Time
}

type UserStatus string

const (
    UserStatusActive   UserStatus = "active"
    UserStatusInactive UserStatus = "inactive"
    UserStatusBanned   UserStatus = "banned"
)

// Business rules as methods
func (u *User) CanPlaceOrder() bool {
    return u.Status == UserStatusActive
}

func (u *User) IsValidEmail() bool {
    return strings.Contains(u.Email, "@")
}
```

### Use Cases (Application Layer)

```go
// internal/usecase/user_usecase.go
package usecase

// Use Case Input/Output - defines what the use case needs and returns
type CreateUserInput struct {
    Email    string
    Name     string
    Password string
}

type CreateUserOutput struct {
    ID    int64
    Email string
    Name  string
}

// Use Case Interface
type UserUseCase interface {
    CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserOutput, error)
    GetUser(ctx context.Context, id int64) (*CreateUserOutput, error)
    DeleteUser(ctx context.Context, id int64) error
}

// Repository Interface - DEFINED in use case layer!
// Implementation will be in outer layer
type UserRepository interface {
    Create(ctx context.Context, user *domain.User) error
    FindByID(ctx context.Context, id int64) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    Delete(ctx context.Context, id int64) error
}

// Password Hasher Interface
type PasswordHasher interface {
    Hash(password string) (string, error)
    Compare(hashed, password string) bool
}

// Use Case Implementation
type userUseCase struct {
    repo     UserRepository
    hasher   PasswordHasher
}

func NewUserUseCase(repo UserRepository, hasher PasswordHasher) UserUseCase {
    return &userUseCase{
        repo:   repo,
        hasher: hasher,
    }
}

func (uc *userUseCase) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserOutput, error) {
    // 1. Validate input
    if input.Email == "" || input.Name == "" {
        return nil, ErrInvalidInput
    }

    // 2. Check if email exists
    existing, _ := uc.repo.FindByEmail(ctx, input.Email)
    if existing != nil {
        return nil, ErrEmailExists
    }

    // 3. Hash password
    hashedPassword, err := uc.hasher.Hash(input.Password)
    if err != nil {
        return nil, err
    }

    // 4. Create domain entity
    user := &domain.User{
        Email:     input.Email,
        Name:      input.Name,
        Password:  hashedPassword,
        Status:    domain.UserStatusActive,
        CreatedAt: time.Now(),
    }

    // 5. Validate business rules
    if !user.IsValidEmail() {
        return nil, ErrInvalidEmail
    }

    // 6. Persist
    if err := uc.repo.Create(ctx, user); err != nil {
        return nil, err
    }

    // 7. Return output
    return &CreateUserOutput{
        ID:    user.ID,
        Email: user.Email,
        Name:  user.Name,
    }, nil
}
```

### Interface Adapters (Controllers, Repositories)

```go
// internal/adapter/http/user_handler.go
package http

type UserHandler struct {
    useCase usecase.UserUseCase
}

func NewUserHandler(uc usecase.UserUseCase) *UserHandler {
    return &UserHandler{useCase: uc}
}

// Adapter: HTTP -> Use Case
func (h *UserHandler) CreateUser(c *gin.Context) {
    // 1. Parse HTTP request
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }

    // 2. Convert to use case input
    input := usecase.CreateUserInput{
        Email:    req.Email,
        Name:     req.Name,
        Password: req.Password,
    }

    // 3. Call use case
    output, err := h.useCase.CreateUser(c.Request.Context(), input)
    if err != nil {
        h.handleError(c, err)
        return
    }

    // 4. Convert to HTTP response
    c.JSON(201, CreateUserResponse{
        ID:    output.ID,
        Email: output.Email,
        Name:  output.Name,
    })
}

// Request/Response are HTTP-specific
type CreateUserRequest struct {
    Email    string `json:"email" binding:"required,email"`
    Name     string `json:"name" binding:"required"`
    Password string `json:"password" binding:"required,min=8"`
}

type CreateUserResponse struct {
    ID    int64  `json:"id"`
    Email string `json:"email"`
    Name  string `json:"name"`
}
```

```go
// internal/adapter/repository/postgres_user_repository.go
package repository

// Adapter: Use Case -> PostgreSQL
type PostgresUserRepository struct {
    db *sql.DB
}

func NewPostgresUserRepository(db *sql.DB) usecase.UserRepository {
    return &PostgresUserRepository{db: db}
}

// Implements usecase.UserRepository interface
func (r *PostgresUserRepository) Create(ctx context.Context, user *domain.User) error {
    query := `
        INSERT INTO users (email, name, password, status, created_at)
        VALUES ($1, $2, $3, $4, $5)
        RETURNING id
    `
    return r.db.QueryRowContext(ctx, query,
        user.Email,
        user.Name,
        user.Password,
        user.Status,
        user.CreatedAt,
    ).Scan(&user.ID)
}

func (r *PostgresUserRepository) FindByID(ctx context.Context, id int64) (*domain.User, error) {
    query := `SELECT id, email, name, password, status, created_at FROM users WHERE id = $1`
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID, &user.Email, &user.Name, &user.Password, &user.Status, &user.CreatedAt,
    )
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return user, err
}
```

### External Layer (Frameworks, Drivers)

```go
// internal/infrastructure/bcrypt_hasher.go
package infrastructure

type BcryptHasher struct{}

func NewBcryptHasher() usecase.PasswordHasher {
    return &BcryptHasher{}
}

func (h *BcryptHasher) Hash(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(bytes), err
}

func (h *BcryptHasher) Compare(hashed, password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hashed), []byte(password))
    return err == nil
}
```

---

## 2️⃣ Project Structure

```
myapp/
├── cmd/
│   └── server/
│       └── main.go                 # Wiring, DI
├── internal/
│   ├── domain/                     # Entities (innermost)
│   │   ├── user.go
│   │   ├── order.go
│   │   └── errors.go
│   ├── usecase/                    # Application layer
│   │   ├── user_usecase.go
│   │   ├── order_usecase.go
│   │   └── interfaces.go           # Repository interfaces
│   ├── adapter/                    # Interface adapters
│   │   ├── http/                   # HTTP handlers
│   │   │   ├── user_handler.go
│   │   │   ├── order_handler.go
│   │   │   └── router.go
│   │   ├── repository/             # Repository implementations
│   │   │   ├── postgres_user.go
│   │   │   └── postgres_order.go
│   │   └── grpc/                   # gRPC handlers (optional)
│   └── infrastructure/             # External frameworks
│       ├── database/
│       │   └── postgres.go
│       ├── hasher/
│       │   └── bcrypt.go
│       └── email/
│           └── smtp.go
├── pkg/
├── go.mod
└── go.sum
```

---

## 3️⃣ Dependency Inversion in Action

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Dependency Inversion                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ Traditional (Dependency follows control flow):                  │
│                                                                      │
│     Handler ──────────▶ Service ──────────▶ Repository              │
│                                                 │                    │
│                                                 ▼                    │
│                                            PostgreSQL                │
│                                                                      │
│     (Everything depends on PostgreSQL!)                             │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  ✅ Clean Architecture (Dependency inversion):                      │
│                                                                      │
│     Handler ──────────▶ UseCase ◀────────── Repository              │
│                            │                    │                    │
│                            │    Interface       │                    │
│                            ▼                    │                    │
│                    «interface»                  │                    │
│                   UserRepository ◀──────────────┘                    │
│                                                                      │
│                                        PostgresUserRepo              │
│                                        MongoUserRepo                 │
│                                        MockUserRepo                  │
│                                                                      │
│     (UseCase defines interface, Repository implements it!)          │
│     (Can swap PostgreSQL for MongoDB without touching UseCase!)     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ Testing Benefits

```go
// Easy to test use cases with mocks!
// internal/usecase/user_usecase_test.go

type mockUserRepository struct {
    users map[int64]*domain.User
}

func (m *mockUserRepository) Create(ctx context.Context, user *domain.User) error {
    user.ID = int64(len(m.users) + 1)
    m.users[user.ID] = user
    return nil
}

func (m *mockUserRepository) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    for _, u := range m.users {
        if u.Email == email {
            return u, nil
        }
    }
    return nil, nil
}

type mockHasher struct{}

func (m *mockHasher) Hash(password string) (string, error) {
    return "hashed_" + password, nil
}

func TestCreateUser_Success(t *testing.T) {
    // Arrange
    repo := &mockUserRepository{users: make(map[int64]*domain.User)}
    hasher := &mockHasher{}
    uc := usecase.NewUserUseCase(repo, hasher)

    input := usecase.CreateUserInput{
        Email:    "test@example.com",
        Name:     "Test User",
        Password: "password123",
    }

    // Act
    output, err := uc.CreateUser(context.Background(), input)

    // Assert
    assert.NoError(t, err)
    assert.Equal(t, "test@example.com", output.Email)
    assert.Equal(t, int64(1), output.ID)
}

func TestCreateUser_EmailExists(t *testing.T) {
    // Arrange
    repo := &mockUserRepository{
        users: map[int64]*domain.User{
            1: {ID: 1, Email: "existing@example.com"},
        },
    }
    hasher := &mockHasher{}
    uc := usecase.NewUserUseCase(repo, hasher)

    input := usecase.CreateUserInput{
        Email:    "existing@example.com",
        Name:     "Test",
        Password: "password123",
    }

    // Act
    _, err := uc.CreateUser(context.Background(), input)

    // Assert
    assert.Equal(t, usecase.ErrEmailExists, err)
}
```

---

## 5️⃣ Wiring (main.go)

```go
// cmd/server/main.go
package main

func main() {
    // 1. Infrastructure
    db := infrastructure.NewPostgresDB(os.Getenv("DATABASE_URL"))
    hasher := infrastructure.NewBcryptHasher()
    emailSender := infrastructure.NewSMTPSender()

    // 2. Repositories (adapters)
    userRepo := repository.NewPostgresUserRepository(db)
    orderRepo := repository.NewPostgresOrderRepository(db)

    // 3. Use Cases
    userUC := usecase.NewUserUseCase(userRepo, hasher)
    orderUC := usecase.NewOrderUseCase(orderRepo, userRepo)

    // 4. Handlers (adapters)
    userHandler := http.NewUserHandler(userUC)
    orderHandler := http.NewOrderHandler(orderUC)

    // 5. Router
    router := http.NewRouter(userHandler, orderHandler)

    // 6. Start
    router.Run(":8080")
}
```

---

## 6️⃣ When to Use Clean Architecture?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    When to Use?                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Use Clean Architecture when:                                    │
│  ─────────────────────────────────                                   │
│  • Complex business logic                                           │
│  • Long-lived project                                               │
│  • Multiple delivery mechanisms (HTTP, gRPC, CLI)                   │
│  • Need to swap infrastructure (change DB, etc.)                    │
│  • Team values testability                                          │
│                                                                      │
│  ❌ May be overkill when:                                           │
│  ─────────────────────────                                           │
│  • Simple CRUD app                                                  │
│  • MVP/prototype                                                    │
│  • Very small team                                                  │
│  • Throwaway project                                                │
│                                                                      │
│  💡 Start simple, evolve when needed                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Dependency Rule** = الـ dependencies تتجه للداخل فقط
- ✅ **Domain** = لا يعتمد على أي شيء خارجي
- ✅ **Use Cases** = يعرّف الـ interfaces، لا ينفذها
- ✅ **Adapters** = تحول بين الـ external والـ internal
- ✅ **Infrastructure** = التفاصيل التقنية (DB, frameworks)
- ✅ سهل الاختبار والتغيير!

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعمق في Repository Pattern:

**➡️ [Lesson 4: Repository Pattern](./04-repository-pattern.md)**

</div>

---

<div align="center">

[⬅️ Previous: Layered Architecture](./02-layered-architecture.md) | [📚 Module Home](../README.md) | [➡️ Next: Repository Pattern](./04-repository-pattern.md)

</div>
