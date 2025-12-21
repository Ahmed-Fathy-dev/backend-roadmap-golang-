# Lesson 8: Error Handling in REST APIs 🚨

<div dir="rtl">

## المقدمة

**Error handling** الجيد = API سهل الاستخدام!

</div>

---

## ✅ Standard Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "status": 422,
    "timestamp": "2024-12-20T18:00:00Z",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
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

## 🔧 Implementation

```go
type ErrorResponse struct {
    Error ErrorDetail `json:"error"`
}

type ErrorDetail struct {
    Code      string                 `json:"code"`
    Message   string                 `json:"message"`
    Status    int                    `json:"status"`
    Timestamp time.Time              `json:"timestamp"`
    Details   []ValidationError      `json:"details,omitempty"`
}

type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
    Code    string `json:"code"`
}

func HandleError(c *gin.Context, status int, code, message string) {
    c.JSON(status, ErrorResponse{
        Error: ErrorDetail{
            Code:      code,
            Message:   message,
            Status:    status,
            Timestamp: time.Now(),
        },
    })
}

// Usage
HandleError(c, 404, "NOT_FOUND", "Product not found")
```

---

## 📋 Common Error Scenarios

### 1. Validation Errors (422)

```go
func CreateProduct(c *gin.Context) {
    var product Product
    if err := c.ShouldBindJSON(&product); err != nil {
        c.JSON(422, ErrorResponse{
            Error: ErrorDetail{
                Code:    "VALIDATION_ERROR",
                Message: "Invalid input data",
                Status:  422,
                Details: ParseValidationErrors(err),
            },
        })
        return
    }
}
```

### 2. Not Found (404)

```go
product, err := GetProduct(id)
if err != nil {
    HandleError(c, 404, "NOT_FOUND", "Product not found")
    return
}
```

### 3. Unauthorized (401)

```go
if token == "" {
    HandleError(c, 401, "UNAUTHORIZED", "Authentication required")
    return
}
```

### 4. Forbidden (403)

```go
if user.Role != "admin" {
    HandleError(c, 403, "FORBIDDEN", "Admin access required")
    return
}
```

### 5. Conflict (409)

```go
if EmailExists(email) {
    HandleError(c, 409, "CONFLICT", "Email already registered")
    return
}
```

---

## 🛡️ Don't Expose Internal Errors

```go
// ❌ DANGEROUS
c.JSON(500, gin.H{
    "error": err.Error(),  // "sql: duplicate key value..."
})

// ✅ SAFE
log.Printf("Database error: %v", err)  // Log internally
HandleError(c, 500, "INTERNAL_ERROR", "Internal server error")
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Consistent format** لكل errors
- ✅ **Error codes** بـ strings
- ✅ **Validation details** واضحة
- ✅ **Don't expose** internal errors
- ✅ **Log** errors internally
- ✅ Use **proper status codes**

</div>

---

<div align="center">

[⬅️ Previous: Pagination](./07-pagination-filtering.md) | [📚 Module Home](../README.md)

</div>
