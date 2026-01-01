# GORM Relations - العلاقات في GORM 🔗

<div dir="rtl">

## مقدمة

GORM بيسهّل التعامل مع العلاقات بين الجداول. في هذا الدرس هنتعلم كل أنواع العلاقات وكيفية استخدامها.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 أنواع العلاقات

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GORM Relationship Types                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Belongs To (1:1)                                                │
│     Profile belongs to User                                         │
│     ┌──────────┐     ┌──────────┐                                   │
│     │ Profile  │────▶│   User   │                                   │
│     │ user_id  │     │   id     │                                   │
│     └──────────┘     └──────────┘                                   │
│                                                                      │
│  2. Has One (1:1)                                                   │
│     User has one Profile                                            │
│     ┌──────────┐     ┌──────────┐                                   │
│     │   User   │◀────│ Profile  │                                   │
│     │   id     │     │ user_id  │                                   │
│     └──────────┘     └──────────┘                                   │
│                                                                      │
│  3. Has Many (1:N)                                                  │
│     User has many Orders                                            │
│     ┌──────────┐     ┌──────────┐                                   │
│     │   User   │◀────│  Order   │                                   │
│     │   id     │     │ user_id  │                                   │
│     └──────────┘     └──────────┘                                   │
│                      │  Order   │                                   │
│                      │ user_id  │                                   │
│                      └──────────┘                                   │
│                                                                      │
│  4. Many to Many (M:N)                                              │
│     Product has many Tags (and vice versa)                          │
│     ┌─────────┐    ┌─────────────┐    ┌──────────┐                 │
│     │ Product │◀──▶│product_tags │◀──▶│   Tag    │                 │
│     │   id    │    │ product_id  │    │   id     │                 │
│     └─────────┘    │  tag_id     │    └──────────┘                 │
│                    └─────────────┘                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 Belongs To

<div dir="rtl">

العلاقة من جهة الـ child (اللي عنده الـ foreign key):

</div>

```go
// Profile belongs to User
type Profile struct {
    ID        uint   `gorm:"primaryKey"`
    Bio       string
    AvatarURL string

    // Foreign key
    UserID uint `gorm:"not null"`
    // Belongs to relation
    User User `gorm:"constraint:OnDelete:CASCADE"`
}

type User struct {
    ID       uint   `gorm:"primaryKey"`
    Username string
    Email    string
}

// Query مع الـ relation
func getProfileWithUser(db *gorm.DB, profileID uint) (*Profile, error) {
    var profile Profile
    err := db.Preload("User").First(&profile, profileID).Error
    return &profile, err
}

// Create مع الـ relation
func createProfile(db *gorm.DB) {
    profile := Profile{
        Bio:    "Software Developer",
        UserID: 1, // User must exist
    }
    db.Create(&profile)

    // أو إنشاء User جديد مع Profile
    profile2 := Profile{
        Bio: "Designer",
        User: User{
            Username: "sara",
            Email:    "sara@example.com",
        },
    }
    db.Create(&profile2) // Creates both!
}
```

---

## 🔗 Has One

<div dir="rtl">

العلاقة من جهة الـ parent:

</div>

```go
type User struct {
    ID       uint   `gorm:"primaryKey"`
    Username string
    Email    string

    // Has One relation
    Profile Profile `gorm:"foreignKey:UserID;constraint:OnDelete:CASCADE"`
}

type Profile struct {
    ID        uint `gorm:"primaryKey"`
    UserID    uint `gorm:"uniqueIndex"` // unique for 1:1
    Bio       string
    AvatarURL string
}

// Query
func getUserWithProfile(db *gorm.DB, userID uint) (*User, error) {
    var user User
    err := db.Preload("Profile").First(&user, userID).Error
    return &user, err
}

// Create User with Profile
func createUserWithProfile(db *gorm.DB) {
    user := User{
        Username: "ahmed",
        Email:    "ahmed@example.com",
        Profile: Profile{
            Bio:       "Backend Developer",
            AvatarURL: "https://example.com/avatar.jpg",
        },
    }
    db.Create(&user)
}
```

---

## 🔗 Has Many

```go
type User struct {
    ID       uint    `gorm:"primaryKey"`
    Username string
    Email    string

    // Has Many relation
    Orders []Order `gorm:"foreignKey:UserID"`
}

type Order struct {
    ID          uint    `gorm:"primaryKey"`
    UserID      uint    `gorm:"index;not null"`
    TotalAmount float64
    Status      string
    CreatedAt   time.Time
}

// Query user with orders
func getUserWithOrders(db *gorm.DB, userID uint) (*User, error) {
    var user User
    err := db.Preload("Orders").First(&user, userID).Error
    return &user, err
}

// Query with conditions on relation
func getUserWithCompletedOrders(db *gorm.DB, userID uint) (*User, error) {
    var user User
    err := db.Preload("Orders", "status = ?", "completed").First(&user, userID).Error
    return &user, err
}

// Query with ordering
func getUserWithRecentOrders(db *gorm.DB, userID uint) (*User, error) {
    var user User
    err := db.Preload("Orders", func(db *gorm.DB) *gorm.DB {
        return db.Order("created_at DESC").Limit(10)
    }).First(&user, userID).Error
    return &user, err
}

// Append to relation
func addOrderToUser(db *gorm.DB, userID uint, order *Order) error {
    return db.Model(&User{ID: userID}).Association("Orders").Append(order)
}

// Count
func countUserOrders(db *gorm.DB, userID uint) int64 {
    return db.Model(&User{ID: userID}).Association("Orders").Count()
}

// Clear all
func clearUserOrders(db *gorm.DB, userID uint) error {
    return db.Model(&User{ID: userID}).Association("Orders").Clear()
}
```

---

## 🔗 Many to Many

```go
type Product struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string
    Price float64

    // Many to Many
    Tags []Tag `gorm:"many2many:product_tags;"`
}

type Tag struct {
    ID   uint   `gorm:"primaryKey"`
    Name string `gorm:"uniqueIndex"`

    // Reverse relation (optional)
    Products []Product `gorm:"many2many:product_tags;"`
}

// GORM creates product_tags table automatically:
// product_id, tag_id

// Create with tags
func createProductWithTags(db *gorm.DB) {
    product := Product{
        Name:  "MacBook Pro",
        Price: 1999.99,
        Tags: []Tag{
            {Name: "electronics"},
            {Name: "laptop"},
            {Name: "apple"},
        },
    }
    db.Create(&product)
}

// Query with tags
func getProductWithTags(db *gorm.DB, productID uint) (*Product, error) {
    var product Product
    err := db.Preload("Tags").First(&product, productID).Error
    return &product, err
}

// Add tags
func addTagsToProduct(db *gorm.DB, productID uint, tags []Tag) error {
    return db.Model(&Product{ID: productID}).Association("Tags").Append(tags)
}

// Replace all tags
func replaceProductTags(db *gorm.DB, productID uint, tags []Tag) error {
    return db.Model(&Product{ID: productID}).Association("Tags").Replace(tags)
}

// Remove tag
func removeTagFromProduct(db *gorm.DB, productID, tagID uint) error {
    return db.Model(&Product{ID: productID}).Association("Tags").Delete(&Tag{ID: tagID})
}

// Find products by tag
func getProductsByTag(db *gorm.DB, tagName string) ([]Product, error) {
    var products []Product
    err := db.Joins("JOIN product_tags ON products.id = product_tags.product_id").
        Joins("JOIN tags ON tags.id = product_tags.tag_id").
        Where("tags.name = ?", tagName).
        Find(&products).Error
    return products, err
}
```

---

## 📦 Custom Join Table

```go
// Custom join table with extra fields
type ProductTag struct {
    ProductID uint      `gorm:"primaryKey"`
    TagID     uint      `gorm:"primaryKey"`
    AddedAt   time.Time `gorm:"autoCreateTime"`
    AddedBy   uint
}

type Product struct {
    ID   uint   `gorm:"primaryKey"`
    Name string
    Tags []Tag `gorm:"many2many:product_tags;"`
}

type Tag struct {
    ID   uint   `gorm:"primaryKey"`
    Name string
}

// Setup join table
func setupJoinTable(db *gorm.DB) {
    db.SetupJoinTable(&Product{}, "Tags", &ProductTag{})
}

// Add with extra fields
func addTagWithMeta(db *gorm.DB, productID, tagID, userID uint) error {
    return db.Create(&ProductTag{
        ProductID: productID,
        TagID:     tagID,
        AddedBy:   userID,
    }).Error
}
```

---

## 🔍 Preloading

<div dir="rtl">

طرق تحميل الـ relations:

</div>

```go
// Single Preload
db.Preload("Orders").Find(&users)

// Multiple Preloads
db.Preload("Orders").Preload("Profile").Find(&users)

// Nested Preload
db.Preload("Orders.Items").Find(&users)
// أو
db.Preload("Orders.Items.Product").Find(&users)

// Preload All
db.Preload(clause.Associations).Find(&users)

// Conditional Preload
db.Preload("Orders", "status = ?", "completed").Find(&users)

// Preload with custom query
db.Preload("Orders", func(db *gorm.DB) *gorm.DB {
    return db.Where("status = ?", "completed").Order("created_at DESC")
}).Find(&users)

// Joins instead of Preload (more efficient for filtering)
db.Joins("Profile").Where("profiles.bio LIKE ?", "%developer%").Find(&users)

// Joins with select
db.Joins("Profile").Select("users.*, profiles.bio").Find(&users)
```

---

## 🏭 Full E-Commerce Example

```go
package models

import (
    "time"
    "gorm.io/gorm"
)

// User model
type User struct {
    ID        uint   `gorm:"primaryKey"`
    Username  string `gorm:"uniqueIndex"`
    Email     string `gorm:"uniqueIndex"`
    CreatedAt time.Time
    UpdatedAt time.Time

    // Relations
    Profile   *Profile  `gorm:"constraint:OnDelete:CASCADE"`
    Orders    []Order   `gorm:"foreignKey:UserID"`
    Addresses []Address `gorm:"foreignKey:UserID"`
}

// Profile (1:1 with User)
type Profile struct {
    ID        uint   `gorm:"primaryKey"`
    UserID    uint   `gorm:"uniqueIndex"`
    FullName  string
    AvatarURL string
    Bio       string
}

// Address (User has many Addresses)
type Address struct {
    ID        uint   `gorm:"primaryKey"`
    UserID    uint   `gorm:"index"`
    Type      string // "billing", "shipping"
    Street    string
    City      string
    Country   string
    IsDefault bool
}

// Order (User has many Orders)
type Order struct {
    ID              uint    `gorm:"primaryKey"`
    UserID          uint    `gorm:"index"`
    Status          string  `gorm:"default:'pending'"`
    TotalAmount     float64
    ShippingAddress string
    CreatedAt       time.Time
    UpdatedAt       time.Time

    // Relations
    User  User        `gorm:"foreignKey:UserID"`
    Items []OrderItem `gorm:"foreignKey:OrderID;constraint:OnDelete:CASCADE"`
}

// OrderItem (Order has many Items)
type OrderItem struct {
    ID        uint    `gorm:"primaryKey"`
    OrderID   uint    `gorm:"index"`
    ProductID uint    `gorm:"index"`
    Quantity  int
    UnitPrice float64

    // Relations
    Product Product `gorm:"foreignKey:ProductID"`
}

// Product
type Product struct {
    ID          uint    `gorm:"primaryKey"`
    SKU         string  `gorm:"uniqueIndex"`
    Name        string
    Description string
    Price       float64
    Stock       int
    CategoryID  *uint   `gorm:"index"`
    CreatedAt   time.Time
    UpdatedAt   time.Time

    // Relations
    Category *Category `gorm:"foreignKey:CategoryID"`
    Tags     []Tag     `gorm:"many2many:product_tags"`
    Images   []ProductImage `gorm:"foreignKey:ProductID"`
}

// Category (Product belongs to Category)
type Category struct {
    ID       uint   `gorm:"primaryKey"`
    Name     string `gorm:"uniqueIndex"`
    ParentID *uint  `gorm:"index"`

    // Self-referential
    Parent   *Category  `gorm:"foreignKey:ParentID"`
    Children []Category `gorm:"foreignKey:ParentID"`
    Products []Product  `gorm:"foreignKey:CategoryID"`
}

// Tag (Many to Many with Product)
type Tag struct {
    ID       uint      `gorm:"primaryKey"`
    Name     string    `gorm:"uniqueIndex"`
    Products []Product `gorm:"many2many:product_tags"`
}

// ProductImage (Product has many Images)
type ProductImage struct {
    ID        uint   `gorm:"primaryKey"`
    ProductID uint   `gorm:"index"`
    URL       string
    IsPrimary bool
    SortOrder int
}

// Queries
func GetOrderWithDetails(db *gorm.DB, orderID uint) (*Order, error) {
    var order Order
    err := db.Preload("User.Profile").
        Preload("Items.Product.Category").
        Preload("Items.Product.Images", "is_primary = ?", true).
        First(&order, orderID).Error
    return &order, err
}

func GetUserDashboard(db *gorm.DB, userID uint) (*User, error) {
    var user User
    err := db.Preload("Profile").
        Preload("Addresses").
        Preload("Orders", func(db *gorm.DB) *gorm.DB {
            return db.Order("created_at DESC").Limit(5)
        }).
        Preload("Orders.Items").
        First(&user, userID).Error
    return &user, err
}

func GetProductWithAllRelations(db *gorm.DB, productID uint) (*Product, error) {
    var product Product
    err := db.Preload("Category.Parent").
        Preload("Tags").
        Preload("Images", func(db *gorm.DB) *gorm.DB {
            return db.Order("sort_order")
        }).
        First(&product, productID).Error
    return &product, err
}

func GetCategoryTree(db *gorm.DB, categoryID uint) (*Category, error) {
    var category Category
    err := db.Preload("Children.Children"). // 2 levels deep
        Preload("Products", func(db *gorm.DB) *gorm.DB {
            return db.Limit(10)
        }).
        First(&category, categoryID).Error
    return &category, err
}
```

---

## ⚙️ Association Mode

```go
// Association operations
func associationExamples(db *gorm.DB) {
    var user User
    db.First(&user, 1)

    // Find associations
    var orders []Order
    db.Model(&user).Association("Orders").Find(&orders)

    // Append
    db.Model(&user).Association("Orders").Append(&Order{TotalAmount: 100})

    // Replace
    db.Model(&user).Association("Orders").Replace(&newOrders)

    // Delete
    db.Model(&user).Association("Orders").Delete(&order)

    // Clear
    db.Model(&user).Association("Orders").Clear()

    // Count
    count := db.Model(&user).Association("Orders").Count()
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  GORM Relations Best Practices                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use constraint tags for cascade delete                           │
│     `gorm:"constraint:OnDelete:CASCADE"`                            │
│                                                                      │
│  2. Add indexes on foreign keys                                      │
│     UserID uint `gorm:"index"`                                      │
│                                                                      │
│  3. Use Preload wisely - avoid N+1                                   │
│     db.Preload("Orders").Find(&users)                               │
│                                                                      │
│  4. Use Joins for filtering on relations                             │
│     db.Joins("Profile").Where("profiles.bio LIKE ?", "%dev%")       │
│                                                                      │
│  5. Limit nested preloads                                            │
│     Don't preload too deep - it can be slow                         │
│                                                                      │
│  6. Consider lazy loading for large relations                        │
│     Load orders separately when needed                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Belongs To** - من جهة الـ child (عنده foreign key)
2. **Has One/Many** - من جهة الـ parent
3. **Many to Many** - جدول وسيط
4. استخدم **Preload** لتحميل الـ relations
5. استخدم **Joins** للفلترة على الـ relations
6. أضف **index** على الـ foreign keys

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [GORM Advanced](./10-gorm-advanced.md)**

</div>

---

<div align="center">

[⬅️ السابق](./08-gorm-models.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./10-gorm-advanced.md)

</div>
