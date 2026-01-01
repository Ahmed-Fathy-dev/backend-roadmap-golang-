# Basic CRUD - عمليات CRUD الأساسية في Go 📝

<div dir="rtl">

## مقدمة

في هذا الدرس هنتعلم إزاي ننفذ عمليات CRUD (Create, Read, Update, Delete) في Go مع PostgreSQL باستخدام كل من database/sql و pgx.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 🗄️ إعداد الجدول

```sql
-- جدول Users للتجربة
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 📦 تعريف الـ Model

```go
package models

import "time"

type User struct {
    ID           int       `json:"id"`
    Username     string    `json:"username"`
    Email        string    `json:"email"`
    PasswordHash string    `json:"-"` // لا يظهر في JSON
    FullName     *string   `json:"full_name,omitempty"` // nullable
    IsActive     bool      `json:"is_active"`
    CreatedAt    time.Time `json:"created_at"`
    UpdatedAt    time.Time `json:"updated_at"`
}

// Input DTOs
type CreateUserInput struct {
    Username string `json:"username"`
    Email    string `json:"email"`
    Password string `json:"password"`
    FullName string `json:"full_name,omitempty"`
}

type UpdateUserInput struct {
    Email    *string `json:"email,omitempty"`
    FullName *string `json:"full_name,omitempty"`
    IsActive *bool   `json:"is_active,omitempty"`
}
```

---

## ➕ Create (INSERT)

<div dir="rtl">

### باستخدام database/sql

</div>

```go
package repository

import (
    "context"
    "database/sql"
    "time"
)

type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

// Create - إنشاء مستخدم جديد
func (r *UserRepository) Create(ctx context.Context, input CreateUserInput) (*User, error) {
    query := `
        INSERT INTO users (username, email, password_hash, full_name)
        VALUES ($1, $2, $3, $4)
        RETURNING id, username, email, full_name, is_active, created_at, updated_at
    `

    var user User
    var fullName sql.NullString

    err := r.db.QueryRowContext(ctx, query,
        input.Username,
        input.Email,
        hashPassword(input.Password), // hash the password!
        nullString(input.FullName),
    ).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &fullName,
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )

    if err != nil {
        return nil, err
    }

    if fullName.Valid {
        user.FullName = &fullName.String
    }

    return &user, nil
}

// Helper functions
func nullString(s string) sql.NullString {
    if s == "" {
        return sql.NullString{}
    }
    return sql.NullString{String: s, Valid: true}
}

func hashPassword(password string) string {
    // في الواقع استخدم bcrypt!
    return "hashed_" + password
}
```

<div dir="rtl">

### باستخدام pgx

</div>

```go
package repository

import (
    "context"

    "github.com/jackc/pgx/v5/pgxpool"
)

type PgxUserRepository struct {
    pool *pgxpool.Pool
}

func NewPgxUserRepository(pool *pgxpool.Pool) *PgxUserRepository {
    return &PgxUserRepository{pool: pool}
}

func (r *PgxUserRepository) Create(ctx context.Context, input CreateUserInput) (*User, error) {
    query := `
        INSERT INTO users (username, email, password_hash, full_name)
        VALUES ($1, $2, $3, $4)
        RETURNING id, username, email, full_name, is_active, created_at, updated_at
    `

    var user User
    err := r.pool.QueryRow(ctx, query,
        input.Username,
        input.Email,
        hashPassword(input.Password),
        nilIfEmpty(input.FullName),
    ).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.FullName, // pgx يتعامل مع pointers تلقائياً
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )

    return &user, err
}

func nilIfEmpty(s string) *string {
    if s == "" {
        return nil
    }
    return &s
}
```

---

## 📖 Read (SELECT)

<div dir="rtl">

### قراءة مستخدم واحد

</div>

```go
// GetByID - البحث بالـ ID
func (r *UserRepository) GetByID(ctx context.Context, id int) (*User, error) {
    query := `
        SELECT id, username, email, full_name, is_active, created_at, updated_at
        FROM users
        WHERE id = $1
    `

    var user User
    var fullName sql.NullString

    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &fullName,
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )

    if err != nil {
        if err == sql.ErrNoRows {
            return nil, nil // مش موجود
        }
        return nil, err
    }

    if fullName.Valid {
        user.FullName = &fullName.String
    }

    return &user, nil
}

// GetByEmail - البحث بالـ email
func (r *UserRepository) GetByEmail(ctx context.Context, email string) (*User, error) {
    query := `
        SELECT id, username, email, password_hash, full_name, is_active, created_at, updated_at
        FROM users
        WHERE email = $1
    `

    var user User
    var fullName sql.NullString

    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.PasswordHash,
        &fullName,
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )

    if err != nil {
        if err == sql.ErrNoRows {
            return nil, nil
        }
        return nil, err
    }

    if fullName.Valid {
        user.FullName = &fullName.String
    }

    return &user, nil
}
```

<div dir="rtl">

### قراءة مستخدمين متعددين

</div>

```go
// GetAll - كل المستخدمين
func (r *UserRepository) GetAll(ctx context.Context) ([]User, error) {
    query := `
        SELECT id, username, email, full_name, is_active, created_at, updated_at
        FROM users
        ORDER BY created_at DESC
    `

    rows, err := r.db.QueryContext(ctx, query)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var users []User
    for rows.Next() {
        var user User
        var fullName sql.NullString

        err := rows.Scan(
            &user.ID,
            &user.Username,
            &user.Email,
            &fullName,
            &user.IsActive,
            &user.CreatedAt,
            &user.UpdatedAt,
        )
        if err != nil {
            return nil, err
        }

        if fullName.Valid {
            user.FullName = &fullName.String
        }

        users = append(users, user)
    }

    if err := rows.Err(); err != nil {
        return nil, err
    }

    return users, nil
}

// GetAllActive - المستخدمين النشطين فقط
func (r *UserRepository) GetAllActive(ctx context.Context) ([]User, error) {
    query := `
        SELECT id, username, email, full_name, is_active, created_at, updated_at
        FROM users
        WHERE is_active = TRUE
        ORDER BY username
    `
    // نفس الـ pattern...
}
```

<div dir="rtl">

### Pagination

</div>

```go
type PaginationParams struct {
    Page     int // starts from 1
    PageSize int
}

type PaginatedResult struct {
    Users      []User `json:"users"`
    TotalCount int    `json:"total_count"`
    Page       int    `json:"page"`
    PageSize   int    `json:"page_size"`
    TotalPages int    `json:"total_pages"`
}

func (r *UserRepository) GetPaginated(ctx context.Context, params PaginationParams) (*PaginatedResult, error) {
    // Validate
    if params.Page < 1 {
        params.Page = 1
    }
    if params.PageSize < 1 || params.PageSize > 100 {
        params.PageSize = 20
    }

    offset := (params.Page - 1) * params.PageSize

    // Count total
    var totalCount int
    err := r.db.QueryRowContext(ctx, "SELECT COUNT(*) FROM users").Scan(&totalCount)
    if err != nil {
        return nil, err
    }

    // Get page
    query := `
        SELECT id, username, email, full_name, is_active, created_at, updated_at
        FROM users
        ORDER BY created_at DESC
        LIMIT $1 OFFSET $2
    `

    rows, err := r.db.QueryContext(ctx, query, params.PageSize, offset)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var users []User
    for rows.Next() {
        var user User
        var fullName sql.NullString

        err := rows.Scan(
            &user.ID,
            &user.Username,
            &user.Email,
            &fullName,
            &user.IsActive,
            &user.CreatedAt,
            &user.UpdatedAt,
        )
        if err != nil {
            return nil, err
        }

        if fullName.Valid {
            user.FullName = &fullName.String
        }
        users = append(users, user)
    }

    totalPages := (totalCount + params.PageSize - 1) / params.PageSize

    return &PaginatedResult{
        Users:      users,
        TotalCount: totalCount,
        Page:       params.Page,
        PageSize:   params.PageSize,
        TotalPages: totalPages,
    }, nil
}
```

---

## ✏️ Update (UPDATE)

```go
// Update - تحديث مستخدم
func (r *UserRepository) Update(ctx context.Context, id int, input UpdateUserInput) (*User, error) {
    // Build dynamic query
    query := "UPDATE users SET updated_at = NOW()"
    args := []interface{}{}
    argNum := 1

    if input.Email != nil {
        query += fmt.Sprintf(", email = $%d", argNum)
        args = append(args, *input.Email)
        argNum++
    }

    if input.FullName != nil {
        query += fmt.Sprintf(", full_name = $%d", argNum)
        args = append(args, *input.FullName)
        argNum++
    }

    if input.IsActive != nil {
        query += fmt.Sprintf(", is_active = $%d", argNum)
        args = append(args, *input.IsActive)
        argNum++
    }

    query += fmt.Sprintf(" WHERE id = $%d", argNum)
    args = append(args, id)

    query += " RETURNING id, username, email, full_name, is_active, created_at, updated_at"

    var user User
    var fullName sql.NullString

    err := r.db.QueryRowContext(ctx, query, args...).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &fullName,
        &user.IsActive,
        &user.CreatedAt,
        &user.UpdatedAt,
    )

    if err != nil {
        if err == sql.ErrNoRows {
            return nil, fmt.Errorf("user %d not found", id)
        }
        return nil, err
    }

    if fullName.Valid {
        user.FullName = &fullName.String
    }

    return &user, nil
}

// UpdateEmail - تحديث الـ email فقط
func (r *UserRepository) UpdateEmail(ctx context.Context, id int, email string) error {
    query := `
        UPDATE users
        SET email = $1, updated_at = NOW()
        WHERE id = $2
    `

    result, err := r.db.ExecContext(ctx, query, email, id)
    if err != nil {
        return err
    }

    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("user %d not found", id)
    }

    return nil
}

// Deactivate - تعطيل المستخدم
func (r *UserRepository) Deactivate(ctx context.Context, id int) error {
    query := `
        UPDATE users
        SET is_active = FALSE, updated_at = NOW()
        WHERE id = $1
    `

    result, err := r.db.ExecContext(ctx, query, id)
    if err != nil {
        return err
    }

    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("user %d not found", id)
    }

    return nil
}
```

---

## 🗑️ Delete (DELETE)

```go
// Delete - حذف مستخدم (Hard Delete)
func (r *UserRepository) Delete(ctx context.Context, id int) error {
    query := "DELETE FROM users WHERE id = $1"

    result, err := r.db.ExecContext(ctx, query, id)
    if err != nil {
        return err
    }

    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("user %d not found", id)
    }

    return nil
}

// SoftDelete - حذف ناعم (تعطيل + تاريخ الحذف)
// يحتاج إضافة عمود deleted_at للجدول
func (r *UserRepository) SoftDelete(ctx context.Context, id int) error {
    query := `
        UPDATE users
        SET is_active = FALSE,
            deleted_at = NOW(),
            updated_at = NOW()
        WHERE id = $1 AND deleted_at IS NULL
    `

    result, err := r.db.ExecContext(ctx, query, id)
    if err != nil {
        return err
    }

    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("user %d not found or already deleted", id)
    }

    return nil
}

// DeleteMany - حذف عدة مستخدمين
func (r *UserRepository) DeleteMany(ctx context.Context, ids []int) (int64, error) {
    if len(ids) == 0 {
        return 0, nil
    }

    // Build placeholders: $1, $2, $3, ...
    placeholders := make([]string, len(ids))
    args := make([]interface{}, len(ids))
    for i, id := range ids {
        placeholders[i] = fmt.Sprintf("$%d", i+1)
        args[i] = id
    }

    query := fmt.Sprintf(
        "DELETE FROM users WHERE id IN (%s)",
        strings.Join(placeholders, ", "),
    )

    result, err := r.db.ExecContext(ctx, query, args...)
    if err != nil {
        return 0, err
    }

    return result.RowsAffected()
}
```

---

## 🔍 Search & Filter

```go
type UserFilter struct {
    Username   *string
    Email      *string
    IsActive   *bool
    CreatedFrom *time.Time
    CreatedTo   *time.Time
}

func (r *UserRepository) Search(ctx context.Context, filter UserFilter) ([]User, error) {
    query := `
        SELECT id, username, email, full_name, is_active, created_at, updated_at
        FROM users
        WHERE 1=1
    `
    args := []interface{}{}
    argNum := 1

    if filter.Username != nil {
        query += fmt.Sprintf(" AND username ILIKE $%d", argNum)
        args = append(args, "%"+*filter.Username+"%")
        argNum++
    }

    if filter.Email != nil {
        query += fmt.Sprintf(" AND email ILIKE $%d", argNum)
        args = append(args, "%"+*filter.Email+"%")
        argNum++
    }

    if filter.IsActive != nil {
        query += fmt.Sprintf(" AND is_active = $%d", argNum)
        args = append(args, *filter.IsActive)
        argNum++
    }

    if filter.CreatedFrom != nil {
        query += fmt.Sprintf(" AND created_at >= $%d", argNum)
        args = append(args, *filter.CreatedFrom)
        argNum++
    }

    if filter.CreatedTo != nil {
        query += fmt.Sprintf(" AND created_at <= $%d", argNum)
        args = append(args, *filter.CreatedTo)
        argNum++
    }

    query += " ORDER BY created_at DESC"

    rows, err := r.db.QueryContext(ctx, query, args...)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    // ... scan rows
}
```

---

## 🏭 مثال كامل

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "log"
    "time"

    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    // الاتصال
    db, err := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    repo := NewUserRepository(db)
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    // Create
    user, err := repo.Create(ctx, CreateUserInput{
        Username: "ahmed",
        Email:    "ahmed@example.com",
        Password: "secret123",
        FullName: "Ahmed Ali",
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Created: %+v\n", user)

    // Read
    found, err := repo.GetByID(ctx, user.ID)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Found: %+v\n", found)

    // Update
    newEmail := "ahmed.ali@example.com"
    updated, err := repo.Update(ctx, user.ID, UpdateUserInput{
        Email: &newEmail,
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Updated: %+v\n", updated)

    // List
    users, err := repo.GetAll(ctx)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Total users: %d\n", len(users))

    // Delete
    err = repo.Delete(ctx, user.ID)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Deleted!")
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CRUD Best Practices                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Always use Context with timeout                                  │
│     ctx, cancel := context.WithTimeout(ctx, 5*time.Second)          │
│                                                                      │
│  2. Use RETURNING for INSERT/UPDATE                                  │
│     INSERT INTO... RETURNING id, created_at                         │
│                                                                      │
│  3. Check RowsAffected for UPDATE/DELETE                            │
│     if rows == 0 { return ErrNotFound }                             │
│                                                                      │
│  4. Handle NULL values properly                                      │
│     sql.NullString or *string                                       │
│                                                                      │
│  5. Never expose password_hash in responses                          │
│     json:"-" tag                                                     │
│                                                                      │
│  6. Use transactions for related operations                          │
│     Create user + profile together                                   │
│                                                                      │
│  7. Validate input before database operations                        │
│     Check email format, username length                              │
│                                                                      │
│  8. Log errors with context                                          │
│     log.Error("failed to create user", "error", err, "input", inp)  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. استخدم **RETURNING** لإرجاع البيانات بعد INSERT/UPDATE
2. تحقق من **RowsAffected** للتأكد من نجاح UPDATE/DELETE
3. تعامل مع **NULL** باستخدام sql.NullXxx أو pointers
4. استخدم **Context** مع timeout
5. **لا تعرض** password_hash في الـ responses
6. **Validate** المدخلات قبل الحفظ

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Prepared Statements](./05-prepared-statements.md)**

</div>

---

<div align="center">

[⬅️ السابق](./03-connection-pooling.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./05-prepared-statements.md)

</div>
