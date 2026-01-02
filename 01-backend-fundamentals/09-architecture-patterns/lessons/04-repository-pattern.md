# Lesson 4: Repository Pattern - نمط المستودع 🗄️

<div dir="rtl">

## المقدمة

**Repository Pattern** يفصل بين الـ business logic والـ data access. بدلاً من كتابة SQL في كل مكان، نستخدم interface موحد للتعامل مع البيانات.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 What is Repository Pattern?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Repository Pattern                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Without Repository:                                                │
│  ─────────────────────                                               │
│                                                                      │
│  ┌─────────┐     SQL queries everywhere!                           │
│  │ Service │─────▶ db.Query("SELECT * FROM users...")              │
│  └─────────┘─────▶ db.Query("INSERT INTO users...")                │
│         │────────▶ db.Query("UPDATE users SET...")                  │
│                                                                      │
│  Problems:                                                          │
│  • SQL scattered across codebase                                    │
│  • Hard to test (need real database)                                │
│  • Hard to switch databases                                         │
│  • Duplicate queries                                                │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  With Repository:                                                   │
│  ──────────────────                                                  │
│                                                                      │
│  ┌─────────┐     ┌──────────────┐     ┌──────────────┐             │
│  │ Service │────▶│  Repository  │────▶│   Database   │             │
│  └─────────┘     │  Interface   │     └──────────────┘             │
│                  └──────────────┘                                   │
│                         │                                            │
│                    Implementations:                                 │
│                    • PostgresRepo                                   │
│                    • MongoRepo                                      │
│                    • MockRepo (testing)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Basic Repository Interface

```go
// internal/repository/user_repository.go
package repository

import (
    "context"
    "myapp/internal/domain"
)

// Repository Interface - defines what operations are available
type UserRepository interface {
    // CRUD operations
    Create(ctx context.Context, user *domain.User) error
    FindByID(ctx context.Context, id int64) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    Update(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id int64) error

    // Query operations
    FindAll(ctx context.Context, filter UserFilter) ([]*domain.User, error)
    Count(ctx context.Context, filter UserFilter) (int64, error)

    // Batch operations
    CreateMany(ctx context.Context, users []*domain.User) error
    DeleteMany(ctx context.Context, ids []int64) error
}

// Filter for queries
type UserFilter struct {
    Status    *string
    CreatedAfter *time.Time
    Limit     int
    Offset    int
    OrderBy   string
}
```

---

## 2️⃣ PostgreSQL Implementation

```go
// internal/repository/postgres/user_repository.go
package postgres

type userRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) repository.UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    query := `
        INSERT INTO users (name, email, password_hash, status, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6)
        RETURNING id
    `
    return r.db.QueryRowContext(ctx, query,
        user.Name,
        user.Email,
        user.PasswordHash,
        user.Status,
        user.CreatedAt,
        user.UpdatedAt,
    ).Scan(&user.ID)
}

func (r *userRepository) FindByID(ctx context.Context, id int64) (*domain.User, error) {
    query := `
        SELECT id, name, email, password_hash, status, created_at, updated_at
        FROM users
        WHERE id = $1 AND deleted_at IS NULL
    `
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Name,
        &user.Email,
        &user.PasswordHash,
        &user.Status,
        &user.CreatedAt,
        &user.UpdatedAt,
    )
    if err == sql.ErrNoRows {
        return nil, nil // Not found
    }
    if err != nil {
        return nil, fmt.Errorf("find user by id: %w", err)
    }
    return user, nil
}

func (r *userRepository) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    query := `
        SELECT id, name, email, password_hash, status, created_at, updated_at
        FROM users
        WHERE email = $1 AND deleted_at IS NULL
    `
    user := &domain.User{}
    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &user.ID, &user.Name, &user.Email, &user.PasswordHash,
        &user.Status, &user.CreatedAt, &user.UpdatedAt,
    )
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return user, err
}

func (r *userRepository) Update(ctx context.Context, user *domain.User) error {
    query := `
        UPDATE users
        SET name = $1, email = $2, status = $3, updated_at = $4
        WHERE id = $5 AND deleted_at IS NULL
    `
    result, err := r.db.ExecContext(ctx, query,
        user.Name,
        user.Email,
        user.Status,
        time.Now(),
        user.ID,
    )
    if err != nil {
        return err
    }

    rows, _ := result.RowsAffected()
    if rows == 0 {
        return repository.ErrNotFound
    }
    return nil
}

func (r *userRepository) Delete(ctx context.Context, id int64) error {
    // Soft delete
    query := `UPDATE users SET deleted_at = $1 WHERE id = $2`
    _, err := r.db.ExecContext(ctx, query, time.Now(), id)
    return err
}

func (r *userRepository) FindAll(ctx context.Context, filter repository.UserFilter) ([]*domain.User, error) {
    query := `
        SELECT id, name, email, password_hash, status, created_at, updated_at
        FROM users
        WHERE deleted_at IS NULL
    `
    args := []interface{}{}
    argCount := 0

    // Dynamic filtering
    if filter.Status != nil {
        argCount++
        query += fmt.Sprintf(" AND status = $%d", argCount)
        args = append(args, *filter.Status)
    }
    if filter.CreatedAfter != nil {
        argCount++
        query += fmt.Sprintf(" AND created_at > $%d", argCount)
        args = append(args, *filter.CreatedAfter)
    }

    // Ordering
    if filter.OrderBy != "" {
        query += " ORDER BY " + filter.OrderBy
    } else {
        query += " ORDER BY created_at DESC"
    }

    // Pagination
    if filter.Limit > 0 {
        argCount++
        query += fmt.Sprintf(" LIMIT $%d", argCount)
        args = append(args, filter.Limit)
    }
    if filter.Offset > 0 {
        argCount++
        query += fmt.Sprintf(" OFFSET $%d", argCount)
        args = append(args, filter.Offset)
    }

    rows, err := r.db.QueryContext(ctx, query, args...)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var users []*domain.User
    for rows.Next() {
        user := &domain.User{}
        err := rows.Scan(
            &user.ID, &user.Name, &user.Email, &user.PasswordHash,
            &user.Status, &user.CreatedAt, &user.UpdatedAt,
        )
        if err != nil {
            return nil, err
        }
        users = append(users, user)
    }
    return users, rows.Err()
}
```

---

## 3️⃣ Generic Repository

```go
// internal/repository/generic.go
package repository

// Generic Repository for common operations
type Repository[T any] interface {
    Create(ctx context.Context, entity *T) error
    FindByID(ctx context.Context, id int64) (*T, error)
    Update(ctx context.Context, entity *T) error
    Delete(ctx context.Context, id int64) error
    FindAll(ctx context.Context, filter Filter) ([]*T, error)
}

type Filter struct {
    Conditions map[string]interface{}
    OrderBy    string
    Limit      int
    Offset     int
}

// Base repository with common logic
type BaseRepository[T any] struct {
    db        *sql.DB
    tableName string
    scanner   func(*sql.Row) (*T, error)
}

func (r *BaseRepository[T]) FindByID(ctx context.Context, id int64) (*T, error) {
    query := fmt.Sprintf("SELECT * FROM %s WHERE id = $1 AND deleted_at IS NULL", r.tableName)
    row := r.db.QueryRowContext(ctx, query, id)
    return r.scanner(row)
}
```

---

## 4️⃣ Unit of Work Pattern

<div dir="rtl">

للتعامل مع transactions عبر عدة repositories.

</div>

```go
// internal/repository/unit_of_work.go
package repository

type UnitOfWork interface {
    Users() UserRepository
    Orders() OrderRepository
    Begin(ctx context.Context) (UnitOfWork, error)
    Commit() error
    Rollback() error
}

type unitOfWork struct {
    db       *sql.DB
    tx       *sql.Tx
    userRepo UserRepository
    orderRepo OrderRepository
}

func NewUnitOfWork(db *sql.DB) UnitOfWork {
    return &unitOfWork{
        db:        db,
        userRepo:  postgres.NewUserRepository(db),
        orderRepo: postgres.NewOrderRepository(db),
    }
}

func (uow *unitOfWork) Begin(ctx context.Context) (UnitOfWork, error) {
    tx, err := uow.db.BeginTx(ctx, nil)
    if err != nil {
        return nil, err
    }

    return &unitOfWork{
        db:        uow.db,
        tx:        tx,
        userRepo:  postgres.NewUserRepositoryTx(tx),
        orderRepo: postgres.NewOrderRepositoryTx(tx),
    }, nil
}

func (uow *unitOfWork) Commit() error {
    if uow.tx == nil {
        return errors.New("no transaction to commit")
    }
    return uow.tx.Commit()
}

func (uow *unitOfWork) Rollback() error {
    if uow.tx == nil {
        return errors.New("no transaction to rollback")
    }
    return uow.tx.Rollback()
}

func (uow *unitOfWork) Users() UserRepository {
    return uow.userRepo
}

func (uow *unitOfWork) Orders() OrderRepository {
    return uow.orderRepo
}

// Usage in Service
func (s *OrderService) CreateOrder(ctx context.Context, input CreateOrderInput) error {
    // Begin transaction
    uow, err := s.uow.Begin(ctx)
    if err != nil {
        return err
    }
    defer uow.Rollback() // Rollback if not committed

    // Get user
    user, err := uow.Users().FindByID(ctx, input.UserID)
    if err != nil {
        return err
    }

    // Create order
    order := &domain.Order{UserID: user.ID, Items: input.Items}
    if err := uow.Orders().Create(ctx, order); err != nil {
        return err
    }

    // Update user's order count
    user.OrderCount++
    if err := uow.Users().Update(ctx, user); err != nil {
        return err
    }

    // Commit transaction
    return uow.Commit()
}
```

---

## 5️⃣ Mock Repository for Testing

```go
// internal/repository/mock/user_repository.go
package mock

type UserRepository struct {
    users  map[int64]*domain.User
    nextID int64
    mu     sync.RWMutex
}

func NewUserRepository() *UserRepository {
    return &UserRepository{
        users:  make(map[int64]*domain.User),
        nextID: 1,
    }
}

func (r *UserRepository) Create(ctx context.Context, user *domain.User) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    user.ID = r.nextID
    r.nextID++
    r.users[user.ID] = user
    return nil
}

func (r *UserRepository) FindByID(ctx context.Context, id int64) (*domain.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    user, ok := r.users[id]
    if !ok {
        return nil, nil
    }
    return user, nil
}

func (r *UserRepository) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    for _, user := range r.users {
        if user.Email == email {
            return user, nil
        }
    }
    return nil, nil
}

// Test helper methods
func (r *UserRepository) Reset() {
    r.mu.Lock()
    defer r.mu.Unlock()
    r.users = make(map[int64]*domain.User)
    r.nextID = 1
}

func (r *UserRepository) Seed(users []*domain.User) {
    for _, user := range users {
        r.Create(context.Background(), user)
    }
}
```

---

## 6️⃣ Testing with Mock Repository

```go
// internal/service/user_service_test.go
func TestUserService_CreateUser(t *testing.T) {
    // Arrange
    mockRepo := mock.NewUserRepository()
    mockHasher := &mockPasswordHasher{}
    service := NewUserService(mockRepo, mockHasher)

    // Act
    user, err := service.CreateUser(context.Background(), CreateUserInput{
        Name:     "Ahmed",
        Email:    "ahmed@test.com",
        Password: "password123",
    })

    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, user)
    assert.Equal(t, "Ahmed", user.Name)
    assert.Equal(t, int64(1), user.ID)
}

func TestUserService_CreateUser_DuplicateEmail(t *testing.T) {
    // Arrange
    mockRepo := mock.NewUserRepository()
    mockRepo.Seed([]*domain.User{
        {Email: "existing@test.com"},
    })
    service := NewUserService(mockRepo, &mockPasswordHasher{})

    // Act
    _, err := service.CreateUser(context.Background(), CreateUserInput{
        Email: "existing@test.com",
    })

    // Assert
    assert.Equal(t, ErrEmailExists, err)
}
```

---

## 7️⃣ Repository with GORM (ORM)

```go
// Alternative: Using GORM
type gormUserRepository struct {
    db *gorm.DB
}

func NewGormUserRepository(db *gorm.DB) repository.UserRepository {
    return &gormUserRepository{db: db}
}

func (r *gormUserRepository) Create(ctx context.Context, user *domain.User) error {
    return r.db.WithContext(ctx).Create(user).Error
}

func (r *gormUserRepository) FindByID(ctx context.Context, id int64) (*domain.User, error) {
    var user domain.User
    err := r.db.WithContext(ctx).First(&user, id).Error
    if errors.Is(err, gorm.ErrRecordNotFound) {
        return nil, nil
    }
    return &user, err
}

func (r *gormUserRepository) FindAll(ctx context.Context, filter repository.UserFilter) ([]*domain.User, error) {
    var users []*domain.User
    query := r.db.WithContext(ctx)

    if filter.Status != nil {
        query = query.Where("status = ?", *filter.Status)
    }
    if filter.Limit > 0 {
        query = query.Limit(filter.Limit)
    }
    if filter.Offset > 0 {
        query = query.Offset(filter.Offset)
    }

    err := query.Find(&users).Error
    return users, err
}
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Repository** = يفصل data access عن business logic
- ✅ **Interface** = يسمح بتبديل الـ implementation
- ✅ **Mock Repository** = للـ unit testing بدون database
- ✅ **Unit of Work** = للـ transactions عبر repositories
- ✅ لا SQL في الـ service layer!
- ✅ Repository يعيد domain entities فقط

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن خلينا نتعلم Dependency Injection:

**➡️ [Lesson 5: Dependency Injection](./05-dependency-injection.md)**

</div>

---

<div align="center">

[⬅️ Previous: Clean Architecture](./03-clean-architecture.md) | [📚 Module Home](../README.md) | [➡️ Next: Dependency Injection](./05-dependency-injection.md)

</div>
