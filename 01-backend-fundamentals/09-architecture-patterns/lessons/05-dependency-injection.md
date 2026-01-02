# Lesson 5: Dependency Injection - حقن التبعيات 💉

<div dir="rtl">

## المقدمة

**Dependency Injection (DI)** هو pattern حيث الـ object يستلم dependencies من الخارج بدلاً من إنشائها بنفسه. ده بيخلي الكود أسهل في الاختبار والتعديل.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 What is Dependency Injection?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Without vs With DI                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ Without DI (tight coupling):                                    │
│  ─────────────────────────────────                                   │
│                                                                      │
│  type UserService struct {                                          │
│      repo *PostgresUserRepository  // Creates its own dependency!  │
│  }                                                                  │
│                                                                      │
│  func NewUserService() *UserService {                               │
│      return &UserService{                                           │
│          repo: &PostgresUserRepository{                             │
│              db: sql.Open("postgres", "...")  // Hardcoded!        │
│          },                                                         │
│      }                                                              │
│  }                                                                  │
│                                                                      │
│  Problems:                                                          │
│  • Can't test without real database                                 │
│  • Can't switch to different database                               │
│  • Hidden dependencies                                              │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  ✅ With DI (loose coupling):                                       │
│  ───────────────────────────────                                     │
│                                                                      │
│  type UserService struct {                                          │
│      repo UserRepository  // Interface, not concrete!              │
│  }                                                                  │
│                                                                      │
│  func NewUserService(repo UserRepository) *UserService {            │
│      return &UserService{repo: repo}  // Injected!                 │
│  }                                                                  │
│                                                                      │
│  Benefits:                                                          │
│  • Easy to test with mock                                           │
│  • Can switch implementations                                       │
│  • Clear dependencies                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Constructor Injection (Recommended)

```go
// The most common and recommended approach in Go

// Define interfaces
type UserRepository interface {
    FindByID(ctx context.Context, id int64) (*User, error)
    Create(ctx context.Context, user *User) error
}

type PasswordHasher interface {
    Hash(password string) (string, error)
    Compare(hash, password string) bool
}

type EmailSender interface {
    Send(to, subject, body string) error
}

// Service with dependencies
type UserService struct {
    repo     UserRepository
    hasher   PasswordHasher
    emailer  EmailSender
}

// Constructor Injection - dependencies passed via constructor
func NewUserService(
    repo UserRepository,
    hasher PasswordHasher,
    emailer EmailSender,
) *UserService {
    return &UserService{
        repo:    repo,
        hasher:  hasher,
        emailer: emailer,
    }
}

func (s *UserService) CreateUser(ctx context.Context, input CreateUserInput) (*User, error) {
    // Use injected dependencies
    hashedPassword, err := s.hasher.Hash(input.Password)
    if err != nil {
        return nil, err
    }

    user := &User{
        Email:    input.Email,
        Password: hashedPassword,
    }

    if err := s.repo.Create(ctx, user); err != nil {
        return nil, err
    }

    // Send welcome email
    s.emailer.Send(user.Email, "Welcome!", "Thanks for signing up!")

    return user, nil
}
```

---

## 2️⃣ Manual Wiring (main.go)

```go
// cmd/server/main.go
package main

func main() {
    // 1. Create infrastructure dependencies
    db, _ := sql.Open("postgres", os.Getenv("DATABASE_URL"))
    redisClient := redis.NewClient(&redis.Options{Addr: "localhost:6379"})

    // 2. Create implementations
    userRepo := postgres.NewUserRepository(db)
    orderRepo := postgres.NewOrderRepository(db)
    hasher := bcrypt.NewHasher()
    emailer := smtp.NewEmailSender(smtpConfig)
    cache := rediscache.NewCache(redisClient)

    // 3. Create services (inject dependencies)
    userService := service.NewUserService(userRepo, hasher, emailer)
    orderService := service.NewOrderService(orderRepo, userRepo, cache)
    authService := service.NewAuthService(userRepo, hasher)

    // 4. Create handlers (inject services)
    userHandler := handler.NewUserHandler(userService)
    orderHandler := handler.NewOrderHandler(orderService)
    authHandler := handler.NewAuthHandler(authService)

    // 5. Setup router
    router := gin.Default()
    router.POST("/users", userHandler.Create)
    router.GET("/users/:id", userHandler.GetByID)
    router.POST("/orders", orderHandler.Create)
    router.POST("/auth/login", authHandler.Login)

    router.Run(":8080")
}
```

---

## 3️⃣ Using Wire (Google's DI Tool)

```go
// Install: go install github.com/google/wire/cmd/wire@latest

// internal/di/wire.go
//go:build wireinject
// +build wireinject

package di

import (
    "github.com/google/wire"
)

// Provider Sets - group related providers
var DatabaseSet = wire.NewSet(
    NewPostgresDB,
    postgres.NewUserRepository,
    postgres.NewOrderRepository,
)

var ServiceSet = wire.NewSet(
    service.NewUserService,
    service.NewOrderService,
    service.NewAuthService,
)

var HandlerSet = wire.NewSet(
    handler.NewUserHandler,
    handler.NewOrderHandler,
    handler.NewAuthHandler,
)

var InfrastructureSet = wire.NewSet(
    bcrypt.NewHasher,
    smtp.NewEmailSender,
    rediscache.NewCache,
)

// Injector function - Wire will generate implementation
func InitializeApp(cfg *config.Config) (*App, error) {
    wire.Build(
        DatabaseSet,
        InfrastructureSet,
        ServiceSet,
        HandlerSet,
        NewApp,
    )
    return nil, nil
}
```

```go
// internal/di/providers.go
package di

import (
    "database/sql"
)

func NewPostgresDB(cfg *config.Config) (*sql.DB, error) {
    return sql.Open("postgres", cfg.DatabaseURL)
}

type App struct {
    Router *gin.Engine
}

func NewApp(
    userHandler *handler.UserHandler,
    orderHandler *handler.OrderHandler,
    authHandler *handler.AuthHandler,
) *App {
    router := gin.Default()

    router.POST("/users", userHandler.Create)
    router.GET("/users/:id", userHandler.GetByID)
    router.POST("/orders", orderHandler.Create)
    router.POST("/auth/login", authHandler.Login)

    return &App{Router: router}
}
```

```bash
# Generate wire_gen.go
cd internal/di
wire
```

```go
// Generated: internal/di/wire_gen.go
func InitializeApp(cfg *config.Config) (*App, error) {
    db, err := NewPostgresDB(cfg)
    if err != nil {
        return nil, err
    }
    userRepository := postgres.NewUserRepository(db)
    orderRepository := postgres.NewOrderRepository(db)
    hasher := bcrypt.NewHasher()
    emailSender := smtp.NewEmailSender()
    userService := service.NewUserService(userRepository, hasher, emailSender)
    orderService := service.NewOrderService(orderRepository, userRepository)
    authService := service.NewAuthService(userRepository, hasher)
    userHandler := handler.NewUserHandler(userService)
    orderHandler := handler.NewOrderHandler(orderService)
    authHandler := handler.NewAuthHandler(authService)
    app := NewApp(userHandler, orderHandler, authHandler)
    return app, nil
}
```

```go
// cmd/server/main.go - Much cleaner!
func main() {
    cfg := config.Load()

    app, err := di.InitializeApp(cfg)
    if err != nil {
        log.Fatal(err)
    }

    app.Router.Run(":8080")
}
```

---

## 4️⃣ Using fx (Uber's DI Framework)

```go
// go get go.uber.org/fx

package main

import (
    "go.uber.org/fx"
)

func main() {
    fx.New(
        // Provide dependencies
        fx.Provide(
            config.Load,
            NewPostgresDB,
            postgres.NewUserRepository,
            postgres.NewOrderRepository,
            bcrypt.NewHasher,
            smtp.NewEmailSender,
            service.NewUserService,
            service.NewOrderService,
            handler.NewUserHandler,
            handler.NewOrderHandler,
            NewRouter,
        ),
        // Invoke to start
        fx.Invoke(StartServer),
    ).Run()
}

func NewRouter(
    userHandler *handler.UserHandler,
    orderHandler *handler.OrderHandler,
) *gin.Engine {
    r := gin.Default()
    r.POST("/users", userHandler.Create)
    r.GET("/users/:id", userHandler.GetByID)
    r.POST("/orders", orderHandler.Create)
    return r
}

func StartServer(lc fx.Lifecycle, router *gin.Engine) {
    lc.Append(fx.Hook{
        OnStart: func(ctx context.Context) error {
            go router.Run(":8080")
            return nil
        },
        OnStop: func(ctx context.Context) error {
            // Graceful shutdown
            return nil
        },
    })
}
```

---

## 5️⃣ Testing with DI

```go
// Easy to test because dependencies are injected!

func TestUserService_CreateUser(t *testing.T) {
    // Create mocks
    mockRepo := &MockUserRepository{}
    mockHasher := &MockHasher{}
    mockEmailer := &MockEmailSender{}

    // Inject mocks
    service := NewUserService(mockRepo, mockHasher, mockEmailer)

    // Setup expectations
    mockHasher.On("Hash", "password").Return("hashed", nil)
    mockRepo.On("Create", mock.Anything, mock.Anything).Return(nil)
    mockEmailer.On("Send", mock.Anything, mock.Anything, mock.Anything).Return(nil)

    // Test
    user, err := service.CreateUser(context.Background(), CreateUserInput{
        Email:    "test@example.com",
        Password: "password",
    })

    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, user)
    mockRepo.AssertExpectations(t)
    mockEmailer.AssertExpectations(t)
}

// Mock implementations
type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) Create(ctx context.Context, user *User) error {
    args := m.Called(ctx, user)
    return args.Error(0)
}

type MockHasher struct {
    mock.Mock
}

func (m *MockHasher) Hash(password string) (string, error) {
    args := m.Called(password)
    return args.String(0), args.Error(1)
}
```

---

## 6️⃣ DI Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DI Best Practices                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ DO:                                                             │
│  ─────                                                               │
│  • Depend on interfaces, not concrete types                         │
│  • Use constructor injection                                        │
│  • Keep constructors simple (just assign)                           │
│  • Wire dependencies at composition root (main.go)                  │
│  • Make dependencies explicit                                       │
│                                                                      │
│  ❌ DON'T:                                                          │
│  ─────────                                                           │
│  • Create dependencies inside methods                               │
│  • Use global variables for dependencies                            │
│  • Hide dependencies in struct initialization                       │
│  • Over-inject (pass only what's needed)                            │
│                                                                      │
│  💡 TIPS:                                                           │
│  ─────────                                                           │
│  • Interface should be defined where it's used                      │
│  • Small interfaces > large interfaces                              │
│  • Start manual, use Wire/fx when it gets complex                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DI Approaches Comparison                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Approach       │ Pros                    │ Cons                    │
│  ───────────────┼─────────────────────────┼─────────────────────────│
│  Manual         │ Simple, no magic        │ Verbose for large apps  │
│                 │ Clear dependencies      │ Manual updates needed   │
│                 │                         │                         │
│  Wire           │ Compile-time safety     │ Learning curve          │
│                 │ Generated code          │ Extra build step        │
│                 │ No runtime reflection   │                         │
│                 │                         │                         │
│  fx             │ Runtime flexibility     │ Runtime errors          │
│                 │ Lifecycle management    │ Magic, harder to debug  │
│                 │ Hot reloading           │                         │
│                                                                      │
│  Recommendation:                                                    │
│  • Small-medium app: Manual wiring                                  │
│  • Large app with complex dependencies: Wire                        │
│  • Need lifecycle management: fx                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **DI** = الـ object يستلم dependencies من الخارج
- ✅ **Constructor Injection** = الأفضل في Go
- ✅ **Interfaces** = للـ loose coupling
- ✅ **Wire** = للـ compile-time DI
- ✅ **fx** = للـ runtime DI مع lifecycle
- ✅ ابدأ manual، استخدم tools عند الحاجة

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم API Design Best Practices:

**➡️ [Lesson 6: API Design Best Practices](./06-api-design.md)**

</div>

---

<div align="center">

[⬅️ Previous: Repository Pattern](./04-repository-pattern.md) | [📚 Module Home](../README.md) | [➡️ Next: API Design](./06-api-design.md)

</div>
