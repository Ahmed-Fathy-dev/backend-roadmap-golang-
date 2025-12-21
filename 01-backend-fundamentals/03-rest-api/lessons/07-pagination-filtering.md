# Lesson 7: Pagination, Filtering & Sorting ⚡

<div dir="rtl">

## نظرة عامة

**لا ترجع millions of rows** في request واحد! استخدم pagination, filtering, sorting.

</div>

---

## 📄 Pagination

### Page-Based (الأكثر شيوعاً)

```
GET /api/products?page=1&limit=20
GET /api/products?page=2&limit=20
```

**Response:**

```json
{
  "data": [...20 products...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 500,
    "pages": 25
  }
}
```

**Implementation:**

```go
func GetProducts(c *gin.Context) {
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "20"))

    offset := (page - 1) * limit

    var products []Product
    var total int64

    db.Model(&Product{}).Count(&total)
    db.Limit(limit).Offset(offset).Find(&products)

    c.JSON(200, gin.H{
        "data": products,
        "pagination": gin.H{
            "page":  page,
            "limit": limit,
            "total": total,
            "pages": int(math.Ceil(float64(total) / float64(limit))),
        },
    })
}
```

---

## 🔍 Filtering

```
GET /api/products?category=laptops
GET /api/products?price_min=500&price_max=2000
GET /api/users?status=active&role=admin
```

**Implementation:**

```go
func GetProducts(c *gin.Context) {
    query := db.Model(&Product{})

    // Filter by category
    if category := c.Query("category"); category != "" {
        query = query.Where("category = ?", category)
    }

    // Filter by price range
    if minPrice := c.Query("price_min"); minPrice != "" {
        query = query.Where("price >= ?", minPrice)
    }
    if maxPrice := c.Query("price_max"); maxPrice != "" {
        query = query.Where("price <= ?", maxPrice)
    }

    var products []Product
    query.Find(&products)

    c.JSON(200, products)
}
```

---

## 🔽 Sorting

```
GET /api/products?sort=price           # Ascending
GET /api/products?sort=-price          # Descending (- prefix)
GET /api/products?sort=name,created_at # Multiple
```

**Implementation:**

```go
func GetProducts(c *gin.Context) {
    query := db.Model(&Product{})

    // Sorting
    if sort := c.Query("sort"); sort != "" {
        // Handle descending (-)
        if strings.HasPrefix(sort, "-") {
            field := strings.TrimPrefix(sort, "-")
            query = query.Order(field + " DESC")
        } else {
            query = query.Order(sort + " ASC")
        }
    }

    var products []Product
    query.Find(&products)

    c.JSON(200, products)
}
```

---

## 🎯 Complete Example

```
GET /api/products?category=laptops&price_min=500&sort=-price&page=1&limit=10

Returns:
- Category: laptops
- Price >= 500
- Sorted by price (descending)
- Page 1, 10 items per page
```

```go
func GetProducts(c *gin.Context) {
    query := db.Model(&Product{})

    // Filtering
    if category := c.Query("category"); category != "" {
        query = query.Where("category = ?", category)
    }
    if minPrice := c.Query("price_min"); minPrice != "" {
        query = query.Where("price >= ?", minPrice)
    }

    // Sorting
    sort := c.DefaultQuery("sort", "created_at")
    if strings.HasPrefix(sort, "-") {
        query = query.Order(strings.TrimPrefix(sort, "-") + " DESC")
    } else {
        query = query.Order(sort + " ASC")
    }

    // Count total
    var total int64
    query.Count(&total)

    // Pagination
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "20"))
    offset := (page - 1) * limit

    var products []Product
    query.Limit(limit).Offset(offset).Find(&products)

    c.JSON(200, gin.H{
        "data": products,
        "pagination": gin.H{
            "page":  page,
            "limit": limit,
            "total": total,
            "pages": int(math.Ceil(float64(total) / float64(limit))),
        },
    })
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Pagination:** page/limit parameters
- ✅ **Filtering:** query parameters
- ✅ **Sorting:** sort parameter (- for DESC)
- ✅ Always **limit** maximum results
- ✅ Return **total count** for pagination

</div>

---

<div align="center">

[⬅️ Previous](./06-api-versioning.md) | [➡️ Next](./08-error-handling.md) | [📚 Module Home](../README.md)

</div>
