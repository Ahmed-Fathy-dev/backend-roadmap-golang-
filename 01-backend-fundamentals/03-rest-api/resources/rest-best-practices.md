# REST API Best Practices 🌟

<div dir="rtl">

## نظرة عامة

دليل شامل لأفضل الممارسات في تصميم REST APIs احترافية.

</div>

---

## 🎯 1. Resource Naming

### ✅ DO:

```
# Use nouns (not verbs)
GET /api/products           ✅
GET /api/users              ✅
GET /api/orders             ✅

# Use plural nouns
GET /api/products           ✅ (not /api/product)
GET /api/users              ✅ (not /api/user)

# Use lowercase
GET /api/products           ✅
GET /api/product-categories ✅

# Use hyphens (not underscores)
GET /api/product-categories ✅
GET /api/user-profiles      ✅
```

### ❌ DON'T:

```
# Don't use verbs
GET /api/getProducts        ❌
POST /api/createUser        ❌
DELETE /api/deleteProduct/5 ❌

# Don't use singular
GET /api/product            ❌
GET /api/user               ❌

# Don't use camelCase in URLs
GET /api/productCategories  ❌
GET /api/userProfiles       ❌

# Don't use underscores
GET /api/product_categories ❌
```

---

## 📍 2. Endpoint Design

### Collection & Resource Pattern:

```
# Collection (plural)
GET    /api/products       # Get all products
POST   /api/products       # Create new product

# Specific resource (singular ID)
GET    /api/products/123   # Get product 123
PUT    /api/products/123   # Update product 123
PATCH  /api/products/123   # Partial update
DELETE /api/products/123   # Delete product 123
```

### Nested Resources:

```
# ✅ Good: Maximum 2 levels
GET /api/users/5/posts           # User 5's posts
GET /api/posts/10/comments       # Post 10's comments

# ⚠️ Avoid: Too deep nesting
GET /api/users/5/posts/10/comments/3/likes  # Too deep!

# ✅ Better: Flat structure
GET /api/comments/3/likes        # Direct access
```

---

## 🔧 3. HTTP Methods Usage

### CRUD Mapping:

| Operation            | HTTP Method | Example                  |
| -------------------- | ----------- | ------------------------ |
| **Create**           | POST        | `POST /api/products`     |
| **Read (all)**       | GET         | `GET /api/products`      |
| **Read (one)**       | GET         | `GET /api/products/5`    |
| **Update (full)**    | PUT         | `PUT /api/products/5`    |
| **Update (partial)** | PATCH       | `PATCH /api/products/5`  |
| **Delete**           | DELETE      | `DELETE /api/products/5` |

### ✅ Best Practices:

```go
// ✅ POST for creation
POST /api/products
Body: { "name": "Laptop", "price": 1000 }
Response: 201 Created

// ✅ GET has no body
GET /api/products?category=laptops

// ✅ PUT replaces entirely
PUT /api/products/5
Body: { "name": "...", "price": ..., "stock": ... }  // ALL fields

// ✅ PATCH updates partially
PATCH /api/products/5
Body: { "price": 900 }  // Only changed field

// ✅ DELETE returns 204
DELETE /api/products/5
Response: 204 No Content
```

---

## 📊 4. Status Codes

### Use Correct Codes:

```go
// Success
200 OK              // GET, PUT, PATCH success
201 Created         // POST success (creation)
204 No Content      // DELETE success

// Client Errors
400 Bad Request     // Invalid request format
401 Unauthorized    // Not authenticated
403 Forbidden       // Not authorized
404 Not Found       // Resource doesn't exist
409 Conflict        // Duplicate/conflict
422 Unprocessable   // Validation error
429 Too Many        // Rate limit exceeded

// Server Errors
500 Internal Error  // Server bug
503 Unavailable     // Server down/maintenance
```

---

## 🔍 5. Filtering, Sorting, Pagination

### Query Parameters:

```
# Filtering
GET /api/products?category=laptops
GET /api/products?price_min=500&price_max=2000
GET /api/users?status=active&role=admin

# Sorting
GET /api/products?sort=price           # Ascending
GET /api/products?sort=-price          # Descending (- prefix)
GET /api/products?sort=name,created_at # Multiple fields

# Pagination
GET /api/products?page=2&limit=20
GET /api/products?offset=40&limit=20   # Alternative

# Combined
GET /api/products?category=laptops&sort=-price&page=1&limit=10
```

### Response Format:

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```

---

## 📝 6. Request & Response Format

### Consistent JSON Structure:

```json
// ✅ Success Response
{
  "success": true,
  "data": {
    "id": 123,
    "name": "Product Name"
  }
}

// ✅ Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}

// ✅ List Response
{
  "success": true,
  "data": [...],
  "pagination": {...}
}
```

---

## 🔐 7. Security

### Authentication:

```http
# ✅ Use Bearer token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# ❌ Don't put credentials in URL
GET /api/products?api_key=secret123  # WRONG!
```

### Input Validation:

```go
// ✅ Always validate
type CreateProductRequest struct {
    Name  string  `binding:"required,min=3,max=100"`
    Price float64 `binding:"required,gt=0"`
    Stock int     `binding:"required,gte=0"`
}

// ❌ Never trust input
```

---

## 📌 8. Versioning

### URL Versioning (Most Common):

```
# ✅ Version in URL
GET /api/v1/products
GET /api/v2/products

# ✅ Major versions only
/api/v1/...  ✅
/api/v2/...  ✅
/api/v1.2/... ❌ (too specific)
```

### Header Versioning:

```http
GET /api/products
Accept: application/vnd.myapi.v1+json
```

---

## 🎯 9. Error Handling

### Standard Error Format:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Product with ID 999 not found",
    "status": 404,
    "timestamp": "2024-12-20T15:30:00Z"
  }
}
```

### Validation Errors:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "code": "INVALID_EMAIL"
      },
      {
        "field": "age",
        "message": "Must be at least 18",
        "code": "AGE_TOO_LOW"
      }
    ]
  }
}
```

---

## 📖 10. Documentation

### Use OpenAPI/Swagger:

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /api/products:
    get:
      summary: Get all products
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        "200":
          description: Success
```

---

## ✅ Complete Checklist

<div dir="rtl">

- [ ] Resources بأسماء **nouns** و **plural**
- [ ] استخدام **HTTP methods** الصحيحة
- [ ] **Status codes** مناسبة
- [ ] **Filtering, sorting, pagination** implemented
- [ ] **Consistent JSON** structure
- [ ] **Error handling** شامل
- [ ] **Input validation** على كل endpoint
- [ ] **Authentication & Authorization** implemented
- [ ] **Rate limiting** على Sensitive endpoints
- [ ] **Versioning** strategy محددة
- [ ] **HTTPS** في Production
- [ ] **API documentation** (Swagger)
- [ ] **Logging** لكل Requests
- [ ] **Testing** comprehensive

</div>

---

## 💡 Key Principles

<div dir="rtl">

1. **Consistency** - نفس النمط في كل API
2. **Simplicity** - سهل الفهم والاستخدام
3. **Predictability** - سلوك متوقع
4. **Security** - آمن دائماً
5. **Performance** - سريع وكفء
6. **Documentation** - موثق جيداً

</div>

---

<div align="center">

[📚 Back to Module Home](../README.md)

</div>
