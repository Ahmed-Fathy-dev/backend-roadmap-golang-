# Error Handling - التعامل مع الأخطاء ⚠️

<div dir="rtl">

## مقدمة

التعامل الصحيح مع أخطاء الـ database مهم جداً لبناء applications قوية. هنتعلم إزاي نتعرف على أنواع الأخطاء ونتعامل معاها صح.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 أنواع أخطاء PostgreSQL

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Error Categories                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Class 23 — Integrity Constraint Violation                          │
│  ├── 23000: integrity_constraint_violation                          │
│  ├── 23001: restrict_violation                                      │
│  ├── 23502: not_null_violation                                      │
│  ├── 23503: foreign_key_violation                                   │
│  ├── 23505: unique_violation                                        │
│  └── 23514: check_violation                                         │
│                                                                      │
│  Class 40 — Transaction Rollback                                     │
│  ├── 40001: serialization_failure                                   │
│  ├── 40002: transaction_integrity_constraint_violation              │
│  ├── 40003: statement_completion_unknown                            │
│  └── 40P01: deadlock_detected                                       │
│                                                                      │
│  Class 42 — Syntax Error or Access Rule Violation                    │
│  ├── 42000: syntax_error_or_access_rule_violation                   │
│  ├── 42501: insufficient_privilege                                  │
│  ├── 42601: syntax_error                                            │
│  └── 42P01: undefined_table                                         │
│                                                                      │
│  Class 53 — Insufficient Resources                                   │
│  ├── 53000: insufficient_resources                                  │
│  ├── 53100: disk_full                                               │
│  └── 53300: too_many_connections                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 pgx Error Types

```go
package db

import (
    "errors"

    "github.com/jackc/pgx/v5/pgconn"
)

// PostgreSQL error codes
const (
    UniqueViolation     = "23505"
    ForeignKeyViolation = "23503"
    NotNullViolation    = "23502"
    CheckViolation      = "23514"
    SerializationFail   = "40001"
    DeadlockDetected    = "40P01"
)

// IsPgError checks if error is a PostgreSQL error with specific code
func IsPgError(err error, code string) bool {
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        return pgErr.Code == code
    }
    return false
}

// GetPgError extracts PostgreSQL error details
func GetPgError(err error) *pgconn.PgError {
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        return pgErr
    }
    return nil
}

// Error check functions
func IsUniqueViolation(err error) bool {
    return IsPgError(err, UniqueViolation)
}

func IsForeignKeyViolation(err error) bool {
    return IsPgError(err, ForeignKeyViolation)
}

func IsNotNullViolation(err error) bool {
    return IsPgError(err, NotNullViolation)
}

func IsCheckViolation(err error) bool {
    return IsPgError(err, CheckViolation)
}

func IsSerializationFailure(err error) bool {
    return IsPgError(err, SerializationFail)
}

func IsDeadlock(err error) bool {
    return IsPgError(err, DeadlockDetected)
}
```

---

## 🏗️ Domain Errors

```go
// internal/domain/errors.go
package domain

import "errors"

// Base errors
var (
    ErrNotFound          = errors.New("not found")
    ErrAlreadyExists     = errors.New("already exists")
    ErrInvalidInput      = errors.New("invalid input")
    ErrUnauthorized      = errors.New("unauthorized")
    ErrForbidden         = errors.New("forbidden")
    ErrConflict          = errors.New("conflict")
    ErrInternal          = errors.New("internal error")
)

// User-specific errors
var (
    ErrUserNotFound      = errors.New("user not found")
    ErrEmailExists       = errors.New("email already exists")
    ErrUsernameExists    = errors.New("username already exists")
    ErrInvalidPassword   = errors.New("invalid password")
    ErrUserInactive      = errors.New("user is inactive")
)

// Order-specific errors
var (
    ErrOrderNotFound     = errors.New("order not found")
    ErrInsufficientStock = errors.New("insufficient stock")
    ErrOrderCancelled    = errors.New("order already cancelled")
)

// Custom error type with details
type AppError struct {
    Code    string `json:"code"`
    Message string `json:"message"`
    Details any    `json:"details,omitempty"`
    Err     error  `json:"-"`
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return e.Err.Error()
    }
    return e.Message
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// Constructors
func NewNotFoundError(resource string) *AppError {
    return &AppError{
        Code:    "NOT_FOUND",
        Message: resource + " not found",
        Err:     ErrNotFound,
    }
}

func NewValidationError(details any) *AppError {
    return &AppError{
        Code:    "VALIDATION_ERROR",
        Message: "validation failed",
        Details: details,
        Err:     ErrInvalidInput,
    }
}

func NewConflictError(message string) *AppError {
    return &AppError{
        Code:    "CONFLICT",
        Message: message,
        Err:     ErrConflict,
    }
}
```

---

## 📦 Repository Error Handling

```go
// internal/repository/postgres/user.go
package postgres

import (
    "context"
    "errors"
    "fmt"

    "myproject/internal/db"
    "myproject/internal/domain"
    "github.com/jackc/pgx/v5/pgconn"
    "gorm.io/gorm"
)

type userRepository struct {
    db *gorm.DB
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    err := r.db.WithContext(ctx).Create(user).Error
    if err != nil {
        return r.translateError(err)
    }
    return nil
}

func (r *userRepository) GetByID(ctx context.Context, id uint) (*domain.User, error) {
    var user domain.User
    err := r.db.WithContext(ctx).First(&user, id).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, domain.ErrUserNotFound
        }
        return nil, fmt.Errorf("get user by id: %w", err)
    }
    return &user, nil
}

func (r *userRepository) translateError(err error) error {
    if err == nil {
        return nil
    }

    // Check for PostgreSQL-specific errors
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        switch pgErr.Code {
        case db.UniqueViolation:
            return r.translateUniqueViolation(pgErr)
        case db.ForeignKeyViolation:
            return fmt.Errorf("referenced record not found: %w", domain.ErrNotFound)
        case db.NotNullViolation:
            return fmt.Errorf("required field missing: %s: %w", pgErr.ColumnName, domain.ErrInvalidInput)
        case db.CheckViolation:
            return fmt.Errorf("constraint violation: %s: %w", pgErr.ConstraintName, domain.ErrInvalidInput)
        }
    }

    // GORM-specific errors
    if errors.Is(err, gorm.ErrRecordNotFound) {
        return domain.ErrNotFound
    }

    return fmt.Errorf("database error: %w", err)
}

func (r *userRepository) translateUniqueViolation(pgErr *pgconn.PgError) error {
    // Parse constraint name to determine which field
    switch pgErr.ConstraintName {
    case "users_email_key", "idx_users_email":
        return domain.ErrEmailExists
    case "users_username_key", "idx_users_username":
        return domain.ErrUsernameExists
    default:
        return fmt.Errorf("duplicate value: %w", domain.ErrAlreadyExists)
    }
}
```

---

## 🌐 HTTP Error Handling

```go
// internal/handler/errors.go
package handler

import (
    "errors"
    "net/http"

    "myproject/internal/domain"
    "github.com/gin-gonic/gin"
)

type ErrorResponse struct {
    Code    string `json:"code"`
    Message string `json:"message"`
    Details any    `json:"details,omitempty"`
}

func HandleError(c *gin.Context, err error) {
    // Check for AppError
    var appErr *domain.AppError
    if errors.As(err, &appErr) {
        status := mapAppErrorToStatus(appErr)
        c.JSON(status, ErrorResponse{
            Code:    appErr.Code,
            Message: appErr.Message,
            Details: appErr.Details,
        })
        return
    }

    // Check for domain errors
    status, response := mapDomainError(err)
    c.JSON(status, response)
}

func mapAppErrorToStatus(err *domain.AppError) int {
    switch {
    case errors.Is(err.Err, domain.ErrNotFound):
        return http.StatusNotFound
    case errors.Is(err.Err, domain.ErrAlreadyExists), errors.Is(err.Err, domain.ErrConflict):
        return http.StatusConflict
    case errors.Is(err.Err, domain.ErrInvalidInput):
        return http.StatusBadRequest
    case errors.Is(err.Err, domain.ErrUnauthorized):
        return http.StatusUnauthorized
    case errors.Is(err.Err, domain.ErrForbidden):
        return http.StatusForbidden
    default:
        return http.StatusInternalServerError
    }
}

func mapDomainError(err error) (int, ErrorResponse) {
    switch {
    case errors.Is(err, domain.ErrUserNotFound),
         errors.Is(err, domain.ErrOrderNotFound),
         errors.Is(err, domain.ErrNotFound):
        return http.StatusNotFound, ErrorResponse{
            Code:    "NOT_FOUND",
            Message: err.Error(),
        }

    case errors.Is(err, domain.ErrEmailExists),
         errors.Is(err, domain.ErrUsernameExists),
         errors.Is(err, domain.ErrAlreadyExists):
        return http.StatusConflict, ErrorResponse{
            Code:    "CONFLICT",
            Message: err.Error(),
        }

    case errors.Is(err, domain.ErrInvalidPassword):
        return http.StatusUnauthorized, ErrorResponse{
            Code:    "UNAUTHORIZED",
            Message: "invalid credentials",
        }

    case errors.Is(err, domain.ErrInvalidInput):
        return http.StatusBadRequest, ErrorResponse{
            Code:    "BAD_REQUEST",
            Message: err.Error(),
        }

    case errors.Is(err, domain.ErrInsufficientStock):
        return http.StatusConflict, ErrorResponse{
            Code:    "INSUFFICIENT_STOCK",
            Message: err.Error(),
        }

    default:
        // Log the actual error
        // logger.Error("internal error", "error", err)
        return http.StatusInternalServerError, ErrorResponse{
            Code:    "INTERNAL_ERROR",
            Message: "an internal error occurred",
        }
    }
}
```

---

## 🔄 Retry Logic

```go
package db

import (
    "context"
    "fmt"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

type RetryConfig struct {
    MaxRetries int
    BaseDelay  time.Duration
    MaxDelay   time.Duration
}

var DefaultRetryConfig = RetryConfig{
    MaxRetries: 3,
    BaseDelay:  100 * time.Millisecond,
    MaxDelay:   2 * time.Second,
}

func WithRetry[T any](ctx context.Context, cfg RetryConfig, fn func() (T, error)) (T, error) {
    var result T
    var lastErr error

    for attempt := 0; attempt <= cfg.MaxRetries; attempt++ {
        result, lastErr = fn()
        if lastErr == nil {
            return result, nil
        }

        // Check if retryable
        if !isRetryable(lastErr) {
            return result, lastErr
        }

        // Don't wait on last attempt
        if attempt < cfg.MaxRetries {
            delay := calculateDelay(attempt, cfg)
            select {
            case <-ctx.Done():
                return result, ctx.Err()
            case <-time.After(delay):
            }
        }
    }

    return result, fmt.Errorf("max retries exceeded: %w", lastErr)
}

func isRetryable(err error) bool {
    // Serialization and deadlock errors are retryable
    if IsSerializationFailure(err) || IsDeadlock(err) {
        return true
    }

    // Connection errors might be retryable
    // Add more conditions as needed

    return false
}

func calculateDelay(attempt int, cfg RetryConfig) time.Duration {
    // Exponential backoff with jitter
    delay := cfg.BaseDelay * time.Duration(1<<uint(attempt))
    if delay > cfg.MaxDelay {
        delay = cfg.MaxDelay
    }
    return delay
}

// Usage
func (r *orderRepository) CreateWithRetry(ctx context.Context, order *Order) error {
    _, err := WithRetry(ctx, DefaultRetryConfig, func() (any, error) {
        return nil, r.Create(ctx, order)
    })
    return err
}
```

---

## 📝 Logging Errors

```go
package handler

import (
    "log/slog"
    "net/http"

    "github.com/gin-gonic/gin"
)

func ErrorMiddleware(logger *slog.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()

        // Check if there were any errors
        if len(c.Errors) > 0 {
            for _, e := range c.Errors {
                // Log the error
                logger.Error("request error",
                    "error", e.Error(),
                    "path", c.Request.URL.Path,
                    "method", c.Request.Method,
                    "status", c.Writer.Status(),
                    "client_ip", c.ClientIP(),
                    "request_id", c.GetString("request_id"),
                )
            }
        }
    }
}

// Usage in handler
func (h *UserHandler) Create(c *gin.Context) {
    var input CreateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.Error(err) // Add to errors for middleware
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid input"})
        return
    }

    user, err := h.service.Create(c.Request.Context(), input)
    if err != nil {
        c.Error(err) // Add to errors for middleware
        HandleError(c, err)
        return
    }

    c.JSON(http.StatusCreated, user)
}
```

---

## 🧪 Testing Errors

```go
package service_test

import (
    "context"
    "testing"

    "myproject/internal/domain"
    "myproject/internal/repository/mock"
    "myproject/internal/service"
    "github.com/stretchr/testify/assert"
)

func TestUserService_Create_EmailExists(t *testing.T) {
    // Setup
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)
    ctx := context.Background()

    // Create first user
    _, err := svc.Create(ctx, domain.CreateUserInput{
        Username: "user1",
        Email:    "test@example.com",
        Password: "password123",
    })
    assert.NoError(t, err)

    // Try to create with same email
    _, err = svc.Create(ctx, domain.CreateUserInput{
        Username: "user2",
        Email:    "test@example.com",
        Password: "password123",
    })

    // Assert
    assert.Error(t, err)
    assert.ErrorIs(t, err, domain.ErrEmailExists)
}

func TestUserService_GetByID_NotFound(t *testing.T) {
    repo := mock.NewUserRepository()
    svc := service.NewUserService(repo)
    ctx := context.Background()

    user, err := svc.GetByID(ctx, 999)

    assert.Nil(t, user)
    assert.ErrorIs(t, err, domain.ErrUserNotFound)
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Error Handling Best Practices                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use domain-specific errors                                       │
│     ErrUserNotFound vs generic "not found"                          │
│                                                                      │
│  2. Wrap errors with context                                         │
│     fmt.Errorf("get user: %w", err)                                 │
│                                                                      │
│  3. Use errors.Is for comparison                                     │
│     errors.Is(err, domain.ErrNotFound)                              │
│                                                                      │
│  4. Translate DB errors at repository layer                          │
│     Convert PostgreSQL codes to domain errors                       │
│                                                                      │
│  5. Don't expose internal details to clients                         │
│     "internal error" instead of stack trace                         │
│                                                                      │
│  6. Log detailed errors server-side                                  │
│     Include request ID, path, user ID                               │
│                                                                      │
│  7. Implement retry for transient errors                             │
│     Serialization failures, deadlocks                               │
│                                                                      │
│  8. Return appropriate HTTP status codes                             │
│     404 for NotFound, 409 for Conflict                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. استخدم **domain-specific errors**
2. **Wrap errors** لإضافة context
3. استخدم **errors.Is** للمقارنة
4. **ترجم** أخطاء PostgreSQL في الـ repository
5. **لا تكشف** تفاصيل داخلية للـ client
6. **سجل** الأخطاء بالتفصيل server-side
7. **Retry** للأخطاء المؤقتة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Testing Database Code](./15-testing.md)**

</div>

---

<div align="center">

[⬅️ السابق](./13-repository-pattern.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./15-testing.md)

</div>
