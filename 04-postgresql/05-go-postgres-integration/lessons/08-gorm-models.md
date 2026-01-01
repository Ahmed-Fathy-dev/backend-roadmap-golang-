# GORM Models - تعريف الـ Models 📋

<div dir="rtl">

## مقدمة

في هذا الدرس هنتعمق في تعريف الـ GORM Models بشكل احترافي، بما فيها الـ tags المتقدمة، الـ hooks، والـ custom types.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📦 gorm.Model

```go
// gorm.Model المضمن
type Model struct {
    ID        uint           `gorm:"primaryKey"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
}

// استخدامه
type User struct {
    gorm.Model // embeds ID, CreatedAt, UpdatedAt, DeletedAt
    Username   string
    Email      string
}

// أو تعريف manual للتحكم الكامل
type Product struct {
    ID        uint      `gorm:"primaryKey;autoIncrement"`
    CreatedAt time.Time `gorm:"autoCreateTime"`
    UpdatedAt time.Time `gorm:"autoUpdateTime"`
    // بدون DeletedAt = لا soft delete
    Name      string
    Price     float64
}
```

---

## 🏷️ Field Tags Reference

```go
type CompleteExample struct {
    // Primary Key options
    ID        uint   `gorm:"primaryKey"`
    UUID      string `gorm:"primaryKey;type:uuid;default:gen_random_uuid()"`

    // Column settings
    UserName  string `gorm:"column:user_name;size:50"`
    Email     string `gorm:"type:varchar(255)"`

    // Constraints
    Code      string `gorm:"unique"`
    Name      string `gorm:"not null"`
    Age       int    `gorm:"check:age >= 0 AND age <= 150"`

    // Default values
    Status    string    `gorm:"default:'pending'"`
    IsActive  bool      `gorm:"default:true"`
    Balance   float64   `gorm:"default:0.00"`
    CreatedAt time.Time `gorm:"default:CURRENT_TIMESTAMP"`

    // Index
    SKU       string `gorm:"uniqueIndex"`
    Category  string `gorm:"index"`
    Type      string `gorm:"index:idx_type_status"`
    StatusCol string `gorm:"index:idx_type_status"`

    // Index with options
    Priority  int    `gorm:"index:idx_priority,sort:desc"`

    // Serialization
    Settings  JSON   `gorm:"type:jsonb"`

    // Precision for decimals
    Price     float64 `gorm:"type:decimal(10,2);precision:10;scale:2"`

    // Comment
    Notes     string `gorm:"comment:'User notes and comments'"`

    // Ignore field
    TempData  string `gorm:"-"`
    CacheKey  string `gorm:"-:all"` // ignore in all operations
    ReadOnly  string `gorm:"-:migration"` // ignore in migration only
}
```

---

## 📊 Custom Types

<div dir="rtl">

### JSON Type

</div>

```go
import (
    "database/sql/driver"
    "encoding/json"
    "errors"
)

// JSON custom type
type JSON map[string]interface{}

// Scan implements sql.Scanner
func (j *JSON) Scan(value interface{}) error {
    if value == nil {
        *j = nil
        return nil
    }

    bytes, ok := value.([]byte)
    if !ok {
        return errors.New("invalid type for JSON")
    }

    return json.Unmarshal(bytes, j)
}

// Value implements driver.Valuer
func (j JSON) Value() (driver.Value, error) {
    if j == nil {
        return nil, nil
    }
    return json.Marshal(j)
}

// استخدامه في Model
type Product struct {
    ID       uint   `gorm:"primaryKey"`
    Name     string
    Metadata JSON   `gorm:"type:jsonb"`
    Tags     JSON   `gorm:"type:jsonb;default:'{}'"`
}

// Usage
product := Product{
    Name: "Laptop",
    Metadata: JSON{
        "brand": "Apple",
        "specs": map[string]interface{}{
            "ram":     "16GB",
            "storage": "512GB",
        },
    },
}
db.Create(&product)
```

<div dir="rtl">

### StringArray Type

</div>

```go
import (
    "database/sql/driver"
    "strings"

    "github.com/lib/pq"
)

// StringArray for PostgreSQL arrays
type StringArray []string

func (a *StringArray) Scan(value interface{}) error {
    return pq.Array((*[]string)(a)).Scan(value)
}

func (a StringArray) Value() (driver.Value, error) {
    return pq.Array(a).Value()
}

// استخدامه
type Article struct {
    ID   uint        `gorm:"primaryKey"`
    Tags StringArray `gorm:"type:text[]"`
}

article := Article{
    Tags: StringArray{"go", "postgresql", "tutorial"},
}
```

<div dir="rtl">

### Enum Type

</div>

```go
type OrderStatus string

const (
    OrderPending   OrderStatus = "pending"
    OrderConfirmed OrderStatus = "confirmed"
    OrderShipped   OrderStatus = "shipped"
    OrderDelivered OrderStatus = "delivered"
    OrderCancelled OrderStatus = "cancelled"
)

func (s OrderStatus) IsValid() bool {
    switch s {
    case OrderPending, OrderConfirmed, OrderShipped, OrderDelivered, OrderCancelled:
        return true
    }
    return false
}

type Order struct {
    ID     uint        `gorm:"primaryKey"`
    Status OrderStatus `gorm:"type:varchar(20);default:'pending'"`
}

// Validation hook
func (o *Order) BeforeCreate(tx *gorm.DB) error {
    if !o.Status.IsValid() {
        return errors.New("invalid order status")
    }
    return nil
}
```

---

## 🪝 Model Hooks

<div dir="rtl">

GORM بيوفر hooks للتنفيذ قبل/بعد العمليات:

</div>

```go
type User struct {
    gorm.Model
    Username     string
    Email        string
    Password     string
    PasswordHash string
}

// BeforeCreate - قبل الإنشاء
func (u *User) BeforeCreate(tx *gorm.DB) error {
    // Hash password
    if u.Password != "" {
        hash, err := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)
        if err != nil {
            return err
        }
        u.PasswordHash = string(hash)
        u.Password = "" // Clear plain password
    }

    // Validate email
    if !strings.Contains(u.Email, "@") {
        return errors.New("invalid email format")
    }

    return nil
}

// AfterCreate - بعد الإنشاء
func (u *User) AfterCreate(tx *gorm.DB) error {
    // Send welcome email
    log.Printf("User %s created with ID %d", u.Username, u.ID)
    return nil
}

// BeforeUpdate - قبل التحديث
func (u *User) BeforeUpdate(tx *gorm.DB) error {
    // Re-hash password if changed
    if tx.Statement.Changed("Password") {
        hash, err := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)
        if err != nil {
            return err
        }
        tx.Statement.SetColumn("PasswordHash", string(hash))
        tx.Statement.SetColumn("Password", "")
    }
    return nil
}

// AfterUpdate
func (u *User) AfterUpdate(tx *gorm.DB) error {
    log.Printf("User %d updated", u.ID)
    return nil
}

// BeforeDelete
func (u *User) BeforeDelete(tx *gorm.DB) error {
    // Check if user can be deleted
    var orderCount int64
    tx.Model(&Order{}).Where("user_id = ? AND status != ?", u.ID, "completed").Count(&orderCount)
    if orderCount > 0 {
        return errors.New("cannot delete user with pending orders")
    }
    return nil
}

// AfterDelete
func (u *User) AfterDelete(tx *gorm.DB) error {
    log.Printf("User %d deleted", u.ID)
    return nil
}

// AfterFind - بعد القراءة
func (u *User) AfterFind(tx *gorm.DB) error {
    // Mask sensitive data
    u.Password = ""
    u.PasswordHash = "[HIDDEN]"
    return nil
}
```

---

## 🔧 Embedded Structs

```go
// Reusable embedded struct
type Timestamps struct {
    CreatedAt time.Time `gorm:"autoCreateTime"`
    UpdatedAt time.Time `gorm:"autoUpdateTime"`
}

type SoftDelete struct {
    DeletedAt gorm.DeletedAt `gorm:"index"`
}

type Auditable struct {
    CreatedBy uint
    UpdatedBy uint
}

// استخدام الـ embedded structs
type Article struct {
    ID      uint   `gorm:"primaryKey"`
    Title   string
    Content string
    Timestamps
    SoftDelete
    Auditable
}

// ينتج عنه جدول بالأعمدة:
// id, title, content, created_at, updated_at, deleted_at, created_by, updated_by

// Embedded struct with prefix
type Address struct {
    Street  string
    City    string
    Country string
}

type Company struct {
    ID              uint `gorm:"primaryKey"`
    Name            string
    BillingAddress  Address `gorm:"embedded;embeddedPrefix:billing_"`
    ShippingAddress Address `gorm:"embedded;embeddedPrefix:shipping_"`
}

// ينتج عنه:
// id, name, billing_street, billing_city, billing_country,
// shipping_street, shipping_city, shipping_country
```

---

## 📊 Full Model Example

```go
package models

import (
    "errors"
    "strings"
    "time"

    "golang.org/x/crypto/bcrypt"
    "gorm.io/gorm"
)

// Enums
type UserRole string

const (
    RoleUser  UserRole = "user"
    RoleAdmin UserRole = "admin"
    RoleMod   UserRole = "moderator"
)

// User model
type User struct {
    // Primary key
    ID uint `gorm:"primaryKey" json:"id"`

    // Basic info
    Username string `gorm:"size:50;uniqueIndex;not null" json:"username"`
    Email    string `gorm:"size:255;uniqueIndex;not null" json:"email"`

    // Password (never expose in JSON)
    Password     string `gorm:"-" json:"-"`
    PasswordHash string `gorm:"size:255;not null" json:"-"`

    // Profile
    FullName  *string `gorm:"size:100" json:"full_name,omitempty"`
    AvatarURL *string `gorm:"size:500" json:"avatar_url,omitempty"`
    Bio       *string `gorm:"type:text" json:"bio,omitempty"`

    // Status
    IsActive        bool      `gorm:"default:true" json:"is_active"`
    IsEmailVerified bool      `gorm:"default:false" json:"is_email_verified"`
    Role            UserRole  `gorm:"size:20;default:'user'" json:"role"`
    LastLoginAt     *time.Time `json:"last_login_at,omitempty"`

    // Metadata
    Settings JSON `gorm:"type:jsonb;default:'{}'" json:"settings,omitempty"`

    // Timestamps
    CreatedAt time.Time      `gorm:"autoCreateTime" json:"created_at"`
    UpdatedAt time.Time      `gorm:"autoUpdateTime" json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`

    // Relations (will cover in next lesson)
    Orders  []Order  `gorm:"foreignKey:UserID" json:"orders,omitempty"`
    Profile *Profile `gorm:"foreignKey:UserID" json:"profile,omitempty"`
}

// TableName - custom table name
func (User) TableName() string {
    return "users"
}

// Hooks
func (u *User) BeforeCreate(tx *gorm.DB) error {
    // Validate
    if err := u.validate(); err != nil {
        return err
    }

    // Hash password
    if u.Password != "" {
        hash, err := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)
        if err != nil {
            return err
        }
        u.PasswordHash = string(hash)
        u.Password = ""
    }

    // Normalize
    u.Email = strings.ToLower(strings.TrimSpace(u.Email))
    u.Username = strings.ToLower(strings.TrimSpace(u.Username))

    return nil
}

func (u *User) BeforeUpdate(tx *gorm.DB) error {
    if tx.Statement.Changed("Password") {
        hash, err := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)
        if err != nil {
            return err
        }
        tx.Statement.SetColumn("PasswordHash", string(hash))
        tx.Statement.SetColumn("Password", "")
    }
    return nil
}

// Methods
func (u *User) validate() error {
    if len(u.Username) < 3 {
        return errors.New("username must be at least 3 characters")
    }
    if !strings.Contains(u.Email, "@") {
        return errors.New("invalid email format")
    }
    if u.Password != "" && len(u.Password) < 8 {
        return errors.New("password must be at least 8 characters")
    }
    return nil
}

func (u *User) CheckPassword(password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(u.PasswordHash), []byte(password))
    return err == nil
}

func (u *User) SetPassword(password string) error {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return err
    }
    u.PasswordHash = string(hash)
    return nil
}

func (u *User) IsAdmin() bool {
    return u.Role == RoleAdmin
}
```

---

## 🔍 Scopes

<div dir="rtl">

Scopes بتسمح بإعادة استخدام query logic:

</div>

```go
// Define scopes
func ActiveUsers(db *gorm.DB) *gorm.DB {
    return db.Where("is_active = ?", true)
}

func AdminUsers(db *gorm.DB) *gorm.DB {
    return db.Where("role = ?", "admin")
}

func RecentlyCreated(days int) func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        return db.Where("created_at > ?", time.Now().AddDate(0, 0, -days))
    }
}

func Paginate(page, pageSize int) func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        if page <= 0 {
            page = 1
        }
        if pageSize <= 0 {
            pageSize = 10
        }
        offset := (page - 1) * pageSize
        return db.Offset(offset).Limit(pageSize)
    }
}

func OrderByRecent(db *gorm.DB) *gorm.DB {
    return db.Order("created_at DESC")
}

// استخدام الـ Scopes
func getActiveAdmins(db *gorm.DB) ([]User, error) {
    var users []User
    err := db.Scopes(ActiveUsers, AdminUsers, OrderByRecent).Find(&users).Error
    return users, err
}

func getRecentUsers(db *gorm.DB, page int) ([]User, error) {
    var users []User
    err := db.Scopes(
        ActiveUsers,
        RecentlyCreated(30),
        Paginate(page, 20),
    ).Find(&users).Error
    return users, err
}

// Scope on Model
func (User) ActiveScope(db *gorm.DB) *gorm.DB {
    return db.Where("is_active = ?", true)
}

// Usage
db.Scopes(User{}.ActiveScope).Find(&users)
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GORM Models Best Practices                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use pointer for nullable fields                                  │
│     FullName *string                                                │
│                                                                      │
│  2. Use json:"-" for sensitive fields                               │
│     PasswordHash string `json:"-"`                                  │
│                                                                      │
│  3. Validate in BeforeCreate/BeforeUpdate hooks                      │
│     if err := u.validate(); err != nil { return err }               │
│                                                                      │
│  4. Use embedded structs for reusable fields                         │
│     type Timestamps struct { CreatedAt, UpdatedAt }                 │
│                                                                      │
│  5. Define custom types for JSON, Arrays, etc.                       │
│     type JSON map[string]interface{}                                │
│                                                                      │
│  6. Use Scopes for reusable query logic                              │
│     db.Scopes(ActiveUsers, Paginate(1, 10))                         │
│                                                                      │
│  7. Keep models in separate package                                  │
│     models/user.go, models/order.go                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **gorm.Model** يضيف ID, CreatedAt, UpdatedAt, DeletedAt
2. استخدم **tags** لتحديد الـ schema بالتفصيل
3. **Hooks** للـ validation و business logic
4. **Custom types** للـ JSON و Arrays
5. **Scopes** لإعادة استخدام الـ queries
6. استخدم **pointer** للـ nullable fields

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [GORM Relations](./09-gorm-relations.md)**

</div>

---

<div align="center">

[⬅️ السابق](./07-gorm-basics.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./09-gorm-relations.md)

</div>
