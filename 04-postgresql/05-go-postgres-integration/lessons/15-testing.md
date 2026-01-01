# Testing Database Code - اختبار كود الـ Database 🧪

<div dir="rtl">

## مقدمة

اختبار كود الـ database مهم لضمان صحة العمليات. هنتعلم طرق مختلفة للاختبار: unit tests مع mocks، integration tests مع database حقيقي، و testcontainers.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Testing Strategies

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Database Testing Strategies                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Unit Tests with Mocks                                           │
│     ├── Fast (milliseconds)                                         │
│     ├── No database required                                        │
│     ├── Test business logic                                         │
│     └── Mock repository interface                                   │
│                                                                      │
│  2. Integration Tests with Test Database                             │
│     ├── Slower (seconds)                                            │
│     ├── Real database required                                      │
│     ├── Test actual SQL queries                                     │
│     └── Catch database-specific issues                              │
│                                                                      │
│  3. Testcontainers                                                  │
│     ├── Real database in Docker                                     │
│     ├── Isolated per test                                           │
│     ├── CI/CD friendly                                              │
│     └── Best of both worlds                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎭 Unit Tests with Mocks

<div dir="rtl">

### Mock Repository

</div>

```go
// internal/repository/mock/user.go
package mock

import (
    "context"
    "sync"
    "time"

    "myproject/internal/domain"
)

type UserRepository struct {
    mu       sync.RWMutex
    users    map[uint]*domain.User
    nextID   uint

    // For custom behavior in tests
    CreateFunc func(ctx context.Context, user *domain.User) error
    GetByIDFunc func(ctx context.Context, id uint) (*domain.User, error)
}

func NewUserRepository() *UserRepository {
    return &UserRepository{
        users:  make(map[uint]*domain.User),
        nextID: 1,
    }
}

func (r *UserRepository) Create(ctx context.Context, user *domain.User) error {
    if r.CreateFunc != nil {
        return r.CreateFunc(ctx, user)
    }

    r.mu.Lock()
    defer r.mu.Unlock()

    // Check for duplicates
    for _, u := range r.users {
        if u.Email == user.Email {
            return domain.ErrEmailExists
        }
        if u.Username == user.Username {
            return domain.ErrUsernameExists
        }
    }

    user.ID = r.nextID
    user.CreatedAt = time.Now()
    user.UpdatedAt = time.Now()
    r.nextID++

    r.users[user.ID] = user
    return nil
}

func (r *UserRepository) GetByID(ctx context.Context, id uint) (*domain.User, error) {
    if r.GetByIDFunc != nil {
        return r.GetByIDFunc(ctx, id)
    }

    r.mu.RLock()
    defer r.mu.RUnlock()

    user, ok := r.users[id]
    if !ok {
        return nil, nil
    }
    return user, nil
}

func (r *UserRepository) GetByEmail(ctx context.Context, email string) (*domain.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    for _, user := range r.users {
        if user.Email == email {
            return user, nil
        }
    }
    return nil, nil
}

func (r *UserRepository) GetByUsername(ctx context.Context, username string) (*domain.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    for _, user := range r.users {
        if user.Username == username {
            return user, nil
        }
    }
    return nil, nil
}

func (r *UserRepository) Update(ctx context.Context, user *domain.User) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    if _, ok := r.users[user.ID]; !ok {
        return domain.ErrUserNotFound
    }

    user.UpdatedAt = time.Now()
    r.users[user.ID] = user
    return nil
}

func (r *UserRepository) Delete(ctx context.Context, id uint) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    if _, ok := r.users[id]; !ok {
        return domain.ErrUserNotFound
    }

    delete(r.users, id)
    return nil
}

func (r *UserRepository) List(ctx context.Context, filter domain.UserFilter, limit, offset int) ([]domain.User, int64, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    var result []domain.User
    for _, user := range r.users {
        if filter.IsActive != nil && user.IsActive != *filter.IsActive {
            continue
        }
        result = append(result, *user)
    }

    total := int64(len(result))

    // Apply pagination
    if offset >= len(result) {
        return []domain.User{}, total, nil
    }
    end := offset + limit
    if end > len(result) {
        end = len(result)
    }

    return result[offset:end], total, nil
}

// Helper method for tests
func (r *UserRepository) Reset() {
    r.mu.Lock()
    defer r.mu.Unlock()
    r.users = make(map[uint]*domain.User)
    r.nextID = 1
}
```

<div dir="rtl">

### Service Tests

</div>

```go
// internal/service/user_test.go
package service_test

import (
    "context"
    "errors"
    "testing"

    "myproject/internal/domain"
    "myproject/internal/repository/mock"
    "myproject/internal/service"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestUserService_Create(t *testing.T) {
    tests := []struct {
        name    string
        input   domain.CreateUserInput
        wantErr error
    }{
        {
            name: "success",
            input: domain.CreateUserInput{
                Username: "testuser",
                Email:    "test@example.com",
                Password: "password123",
            },
            wantErr: nil,
        },
        {
            name: "short password",
            input: domain.CreateUserInput{
                Username: "testuser",
                Email:    "test@example.com",
                Password: "short",
            },
            wantErr: domain.ErrInvalidInput,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            repo := mock.NewUserRepository()
            svc := service.NewUserService(repo)
            ctx := context.Background()

            user, err := svc.Create(ctx, tt.input)

            if tt.wantErr != nil {
                assert.Error(t, err)
                assert.True(t, errors.Is(err, tt.wantErr))
                return
            }

            require.NoError(t, err)
            assert.NotZero(t, user.ID)
            assert.Equal(t, tt.input.Username, user.Username)
            assert.Equal(t, tt.input.Email, user.Email)
            assert.NotEmpty(t, user.PasswordHash)
        })
    }
}

func TestUserService_Create_DuplicateEmail(t *testing.T) {
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)
    ctx := context.Background()

    // Create first user
    _, err := svc.Create(ctx, domain.CreateUserInput{
        Username: "user1",
        Email:    "test@example.com",
        Password: "password123",
    })
    require.NoError(t, err)

    // Try duplicate email
    _, err = svc.Create(ctx, domain.CreateUserInput{
        Username: "user2",
        Email:    "test@example.com",
        Password: "password123",
    })

    assert.ErrorIs(t, err, domain.ErrEmailExists)
}

func TestUserService_GetByID(t *testing.T) {
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)
    ctx := context.Background()

    // Create user
    created, err := svc.Create(ctx, domain.CreateUserInput{
        Username: "testuser",
        Email:    "test@example.com",
        Password: "password123",
    })
    require.NoError(t, err)

    // Get user
    user, err := svc.GetByID(ctx, created.ID)
    require.NoError(t, err)
    assert.Equal(t, created.ID, user.ID)
    assert.Equal(t, "testuser", user.Username)
}

func TestUserService_GetByID_NotFound(t *testing.T) {
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)
    ctx := context.Background()

    user, err := svc.GetByID(ctx, 999)

    assert.Nil(t, user)
    assert.ErrorIs(t, err, domain.ErrUserNotFound)
}

func TestUserService_Authenticate(t *testing.T) {
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)
    ctx := context.Background()

    // Create user
    _, err := svc.Create(ctx, domain.CreateUserInput{
        Username: "testuser",
        Email:    "test@example.com",
        Password: "password123",
    })
    require.NoError(t, err)

    // Authenticate
    user, err := svc.Authenticate(ctx, "test@example.com", "password123")
    require.NoError(t, err)
    assert.Equal(t, "testuser", user.Username)

    // Wrong password
    _, err = svc.Authenticate(ctx, "test@example.com", "wrongpassword")
    assert.ErrorIs(t, err, domain.ErrInvalidPassword)
}
```

---

## 🔌 Integration Tests

```go
// internal/repository/postgres/user_test.go
package postgres_test

import (
    "context"
    "os"
    "testing"

    "myproject/internal/domain"
    "myproject/internal/repository/postgres"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

var testDB *gorm.DB

func TestMain(m *testing.M) {
    // Setup
    dsn := os.Getenv("TEST_DATABASE_URL")
    if dsn == "" {
        dsn = "postgres://postgres:password@localhost:5432/myapp_test?sslmode=disable"
    }

    var err error
    testDB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        panic("failed to connect to test database: " + err.Error())
    }

    // Migrate
    testDB.AutoMigrate(&domain.User{})

    // Run tests
    code := m.Run()

    // Cleanup
    sqlDB, _ := testDB.DB()
    sqlDB.Close()

    os.Exit(code)
}

func setupTest(t *testing.T) *gorm.DB {
    // Start transaction
    tx := testDB.Begin()
    t.Cleanup(func() {
        tx.Rollback()
    })
    return tx
}

func TestUserRepository_Create(t *testing.T) {
    db := setupTest(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    user := &domain.User{
        Username:     "testuser",
        Email:        "test@example.com",
        PasswordHash: "hash",
        IsActive:     true,
    }

    err := repo.Create(ctx, user)

    require.NoError(t, err)
    assert.NotZero(t, user.ID)
    assert.NotZero(t, user.CreatedAt)
}

func TestUserRepository_Create_DuplicateEmail(t *testing.T) {
    db := setupTest(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    user1 := &domain.User{
        Username:     "user1",
        Email:        "test@example.com",
        PasswordHash: "hash",
    }
    err := repo.Create(ctx, user1)
    require.NoError(t, err)

    user2 := &domain.User{
        Username:     "user2",
        Email:        "test@example.com",
        PasswordHash: "hash",
    }
    err = repo.Create(ctx, user2)

    assert.ErrorIs(t, err, domain.ErrEmailExists)
}

func TestUserRepository_GetByID(t *testing.T) {
    db := setupTest(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    // Create
    user := &domain.User{
        Username:     "testuser",
        Email:        "test@example.com",
        PasswordHash: "hash",
    }
    repo.Create(ctx, user)

    // Get
    found, err := repo.GetByID(ctx, user.ID)

    require.NoError(t, err)
    assert.Equal(t, user.ID, found.ID)
    assert.Equal(t, "testuser", found.Username)
}

func TestUserRepository_List_WithFilter(t *testing.T) {
    db := setupTest(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    // Create test data
    users := []*domain.User{
        {Username: "active1", Email: "a1@test.com", PasswordHash: "h", IsActive: true},
        {Username: "active2", Email: "a2@test.com", PasswordHash: "h", IsActive: true},
        {Username: "inactive", Email: "i@test.com", PasswordHash: "h", IsActive: false},
    }
    for _, u := range users {
        repo.Create(ctx, u)
    }

    // Test filter
    active := true
    result, total, err := repo.List(ctx, domain.UserFilter{IsActive: &active}, 10, 0)

    require.NoError(t, err)
    assert.Equal(t, int64(2), total)
    assert.Len(t, result, 2)
}
```

---

## 🐳 Testcontainers

```go
// internal/repository/postgres/user_container_test.go
package postgres_test

import (
    "context"
    "testing"
    "time"

    "myproject/internal/domain"
    "myproject/internal/repository/postgres"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/testcontainers/testcontainers-go/wait"
    gormpostgres "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func setupContainer(t *testing.T) *gorm.DB {
    ctx := context.Background()

    // Start PostgreSQL container
    container, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:15-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("test"),
        postgres.WithPassword("test"),
        testcontainers.WithWaitStrategy(
            wait.ForLog("database system is ready to accept connections").
                WithOccurrence(2).
                WithStartupTimeout(5*time.Second),
        ),
    )
    require.NoError(t, err)

    t.Cleanup(func() {
        container.Terminate(ctx)
    })

    // Get connection string
    connStr, err := container.ConnectionString(ctx, "sslmode=disable")
    require.NoError(t, err)

    // Connect with GORM
    db, err := gorm.Open(gormpostgres.Open(connStr), &gorm.Config{})
    require.NoError(t, err)

    // Migrate
    db.AutoMigrate(&domain.User{})

    return db
}

func TestUserRepository_WithContainer(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping container test in short mode")
    }

    db := setupContainer(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    // Create
    user := &domain.User{
        Username:     "containertest",
        Email:        "container@test.com",
        PasswordHash: "hash",
        IsActive:     true,
    }
    err := repo.Create(ctx, user)
    require.NoError(t, err)

    // Get
    found, err := repo.GetByID(ctx, user.ID)
    require.NoError(t, err)
    assert.Equal(t, "containertest", found.Username)

    // Update
    found.IsActive = false
    err = repo.Update(ctx, found)
    require.NoError(t, err)

    // Verify
    updated, _ := repo.GetByID(ctx, user.ID)
    assert.False(t, updated.IsActive)

    // Delete
    err = repo.Delete(ctx, user.ID)
    require.NoError(t, err)

    deleted, _ := repo.GetByID(ctx, user.ID)
    assert.Nil(t, deleted)
}
```

---

## 🎭 sqlmock for GORM

```go
// internal/repository/postgres/user_mock_test.go
package postgres_test

import (
    "context"
    "regexp"
    "testing"
    "time"

    "myproject/internal/domain"
    "myproject/internal/repository/postgres"
    "github.com/DATA-DOG/go-sqlmock"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    gormpostgres "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func setupMock(t *testing.T) (*gorm.DB, sqlmock.Sqlmock) {
    mockDB, mock, err := sqlmock.New()
    require.NoError(t, err)

    dialector := gormpostgres.New(gormpostgres.Config{
        Conn:       mockDB,
        DriverName: "postgres",
    })

    db, err := gorm.Open(dialector, &gorm.Config{})
    require.NoError(t, err)

    t.Cleanup(func() {
        mockDB.Close()
    })

    return db, mock
}

func TestUserRepository_GetByID_Mock(t *testing.T) {
    db, mock := setupMock(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    // Setup expectations
    rows := sqlmock.NewRows([]string{
        "id", "username", "email", "password_hash", "is_active", "created_at", "updated_at",
    }).AddRow(1, "testuser", "test@example.com", "hash", true, time.Now(), time.Now())

    mock.ExpectQuery(regexp.QuoteMeta(
        `SELECT * FROM "users" WHERE "users"."id" = $1`,
    )).WithArgs(1).WillReturnRows(rows)

    // Execute
    user, err := repo.GetByID(ctx, 1)

    // Assert
    require.NoError(t, err)
    assert.Equal(t, uint(1), user.ID)
    assert.Equal(t, "testuser", user.Username)
    assert.NoError(t, mock.ExpectationsWereMet())
}
```

---

## 📋 Test Fixtures

```go
// internal/testutil/fixtures.go
package testutil

import (
    "myproject/internal/domain"
    "time"
)

func NewTestUser(overrides ...func(*domain.User)) *domain.User {
    user := &domain.User{
        Username:     "testuser",
        Email:        "test@example.com",
        PasswordHash: "$2a$10$...",
        FullName:     "Test User",
        IsActive:     true,
        Role:         "user",
        CreatedAt:    time.Now(),
        UpdatedAt:    time.Now(),
    }

    for _, override := range overrides {
        override(user)
    }

    return user
}

// Usage
func TestSomething(t *testing.T) {
    user := testutil.NewTestUser(func(u *domain.User) {
        u.Username = "custom"
        u.IsActive = false
    })
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Testing Best Practices                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use table-driven tests                                           │
│     tests := []struct{ name string; input; want }{}                 │
│                                                                      │
│  2. Use transactions for cleanup                                     │
│     tx := db.Begin(); t.Cleanup(func() { tx.Rollback() })          │
│                                                                      │
│  3. Use testcontainers for CI/CD                                     │
│     Real database, isolated per test                                │
│                                                                      │
│  4. Mock at the right level                                          │
│     Repository interface, not database directly                     │
│                                                                      │
│  5. Test edge cases                                                  │
│     Duplicates, not found, invalid input                            │
│                                                                      │
│  6. Use test fixtures                                                │
│     NewTestUser(), NewTestOrder()                                   │
│                                                                      │
│  7. Separate unit and integration tests                              │
│     go test -short skips slow tests                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Mock Repository** لـ unit tests سريعة
2. **Integration tests** للتأكد من SQL الفعلي
3. **Testcontainers** لـ CI/CD مع database حقيقي
4. استخدم **transactions** للـ cleanup
5. **Table-driven tests** لتغطية حالات متعددة
6. **Test fixtures** لإنشاء بيانات اختبار

</div>

---

## 🎉 انتهى Module 05!

<div dir="rtl">

**تهانينا!** أنهيت Module 05: Go + PostgreSQL Integration 🚀

**➡️ الـ Module التالي: [Advanced Topics](../../06-advanced-topics/README.md)**

</div>

---

<div align="center">

[⬅️ السابق](./14-error-handling.md) | [🏠 العودة للـ Module](../README.md) | [Module التالي ➡️](../../06-advanced-topics/README.md)

</div>
