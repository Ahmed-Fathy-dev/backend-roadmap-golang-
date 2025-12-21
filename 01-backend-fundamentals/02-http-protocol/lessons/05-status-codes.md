# Lesson 5: HTTP Status Codes - The Complete Guide 📊

<div dir="rtl">

## المقدمة

**HTTP Status Codes** هي أرقام ثلاثية تخبرك بنتيجة Request.

هل نجح؟ هل فشل؟ ما المشكلة؟ - كل هذا في رقم بسيط!

</div>

---

## 🎯 Status Code Structure

```
┌─────────────────────────┐
│    HTTP/1.1  200  OK    │
│             │    │      │
│             │    └─ Reason Phrase
│             └────── Status Code
└─────────────────────────┘
```

<div dir="rtl">

### التقسيم حسب الرقم الأول:

</div>

| Range   | Category      | المعنى                                |
| ------- | ------------- | ------------------------------------- |
| **1xx** | Informational | <div dir="rtl">معلومات (نادرة)</div>  |
| **2xx** | Success       | <div dir="rtl">نجاح ✅</div>          |
| **3xx** | Redirection   | <div dir="rtl">إعادة توجيه</div>      |
| **4xx** | Client Error  | <div dir="rtl">خطأ من Client ❌</div> |
| **5xx** | Server Error  | <div dir="rtl">خطأ من Server💥</div>  |

---

## ✅ 2xx Success Codes

### 200 OK

<div dir="rtl">

**المعنى:** كل شيء تمام!  
**الاستخدام:** GET, PUT, PATCH requests ناجحة

</div>

```http
GET /api/products/42
Response:
HTTP/1.1 200 OK
{ "id": 42, "name": "Laptop" }
```

### 201 Created ⭐

<div dir="rtl">

**المعنى:** تم إنشاء مورد جديد  
**الاستخدام:** POST requests ناجحة

</div>

```http
POST /api/products
Response:
HTTP/1.1 201 Created
Location: /api/products/123
{ "id": 123, "name": "New Product" }
```

```go
// ✅ Correct
c.JSON(201, gin.H{
    "id": product.ID,
})

// ❌ Wrong - don't use 200 for creation!
c.JSON(200, product)
```

### 204 No Content

<div dir="rtl">

**المعنى:** نجح لكن لا يوجد محتوى للإرجاع  
**الاستخدام:** DELETE, PUT ناجحة

</div>

```http
DELETE /api/products/42
Response:
HTTP/1.1 204 No Content
[No body]
```

```go
// ✅ Perfect for DELETE
c.Status(204)
```

---

## ↪️ 3xx Redirection Codes

### 301 Moved Permanently

```http
GET /old-url
Response:
HTTP/1.1 301 Moved Permanently
Location: /new-url
```

### 302 Found (Temporary Redirect)

```http
GET /login
Response:
HTTP/1.1 302 Found
Location: /auth/login
```

### 304 Not Modified

<div dir="rtl">

**للـ Caching:** المحتوى لم يتغير

</div>

```http
GET /api/products
If-None-Match: "abc123"

Response:
HTTP/1.1 304 Not Modified
[No body - use cached version]
```

---

## ❌ 4xx Client Error Codes

### 400 Bad Request

<div dir="rtl">

**المعنى:** الطلب غير صحيح / invalid syntax  
**متى:** Invalid JSON, missing required fields

</div>

```go
c.JSON(400, gin.H{
    "error": "Invalid request format",
})
```

### 401 Unauthorized ⭐

<div dir="rtl">

**المعنى:** غير مصادق عليه (No/Invalid auth)  
**متى:** Missing or invalid token/credentials

</div>

```go
// No token provided
c.JSON(401, gin.H{
    "error": "Authentication required",
})

// Invalid token
c.JSON(401, gin.H{
    "error": "Invalid or expired token",
})
```

### 403 Forbidden ⭐

<div dir="rtl">

**المعنى:** مصادق عليه لكن **غير مسموح** (No permission)  
**متى:** Authenticated but lacks permission

</div>

```go
// User is logged in but not admin
c.JSON(403, gin.H{
    "error": "Admin access required",
})
```

**401 vs 403:**

```
401: "من أنت؟ عرّف عن نفسك"
403: "أعرفك، لكن غير مسموح لك"
```

### 404 Not Found ⭐

<div dir="rtl">

**المعنى:** المورد غير موجود

</div>

```go
c.JSON(404, gin.H{
    "error": "Product not found",
})
```

### 409 Conflict

<div dir="rtl">

**المعنى:** تعارض مع حالة حالية  
**متى:** Duplicate entry, concurrent update

</div>

```go
// Email already exists
c.JSON(409, gin.H{
    "error": "Email already registered",
})
```

### 422 Unprocessable Entity ⭐

<div dir="rtl">

**المعنى:** الطلب مفهوم لكن البيانات غير صالحة  
**متى:** Validation errors

</div>

```go
c.JSON(422, gin.H{
    "error": "Validation failed",
    "details": [
        {"field": "email", "message": "Invalid email format"},
        {"field": "age", "message": "Must be at least 18"},
    ],
})
```

### 429 Too Many Requests

<div dir="rtl">

**المعنى:** تجاوز Rate Limit

</div>

```go
c.JSON(429, gin.H{
    "error": "Too many requests. Try again in 60 seconds",
    "retry_after": 60,
})
```

---

## 💥 5xx Server Error Codes

### 500 Internal Server Error ⭐

<div dir="rtl">

**المعنى:** خطأ عام في السيرفر  
**متى:** Unhandled exception, bug

</div>

```go
// Unexpected error
c.JSON(500, gin.H{
    "error": "Internal server error",
})

// Log the actual error for debugging
log.Printf("Error: %v", err)
```

### 502 Bad Gateway

<div dir="rtl">

**المعنى:** السيرفر كـ gateway تلقى رد غير صحيح

</div>

### 503 Service Unavailable

<div dir="rtl">

**المعنى:** السيرفر مشغول أو Maintenance

</div>

```http
HTTP/1.1 503 Service Unavailable
Retry-After: 3600

{
  "error": "Service temporarily unavailable"
}
```

### 504 Gateway Timeout

<div dir="rtl">

**المعنى:** السيرفر لم يتلق رد في الوقت المحدد

</div>

---

## 📋 Complete Reference Table

### Most Important Codes

| Code    | Name          | Use Case                                       |
| ------- | ------------- | ---------------------------------------------- |
| **200** | OK            | <div dir="rtl">GET, PUT, PATCH نجحوا</div>     |
| **201** | Created       | <div dir="rtl">POST نجح</div>                  |
| **204** | No Content    | <div dir="rtl">DELETE نجح</div>                |
| **400** | Bad Request   | <div dir="rtl">Invalid request</div>           |
| **401** | Unauthorized  | <div dir="rtl">No/Invalid authentication</div> |
| **403** | Forbidden     | <div dir="rtl">No permission</div>             |
| **404** | Not Found     | <div dir="rtl">Resource لا يوجد</div>          |
| **409** | Conflict      | <div dir="rtl">Duplicate/Conflict</div>        |
| **422** | Unprocessable | <div dir="rtl">Validation errors</div>         |
| **429** | Too Many      | <div dir="rtl">Rate limit exceeded</div>       |
| **500** | Server Error  | <div dir="rtl">Server bug</div>                |
| **503** | Unavailable   | <div dir="rtl">Server busy/maintenance</div>   |

---

## ✅ Best Practices

### 1. Use Correct Status Code

```go
// ❌ WRONG: Everything returns 200
c.JSON(200, gin.H{"error": "User not found"})  // Should be 404!
c.JSON(200, gin.H{"error": "Unauthorized"})    // Should be 401!

// ✅ CORRECT: Use appropriate codes
c.JSON(404, gin.H{"error": "User not found"})
c.JSON(401, gin.H{"error": "Unauthorized"})
c.JSON(201, newUser)  // For creation
```

### 2. Consistent Error Format

```go
// ✅ Standard error response structure
type ErrorResponse struct {
    Error   string                 `json:"error"`
    Code    string                 `json:"code,omitempty"`
    Details map[string]interface{} `json:"details,omitempty"`
}

// Usage
c.JSON(422, ErrorResponse{
    Error: "Validation failed",
    Code:  "VALIDATION_ERROR",
    Details: map[string]interface{}{
        "fields": []string{"email", "password"},
    },
})
```

### 3. Don't Expose Internal Errors

```go
// ❌ DANGEROUS: Exposes internal details
c.JSON(500, gin.H{
    "error": err.Error(),  // "sql: no rows in result set"
})

// ✅ SAFE: Generic message + log internally
log.Printf("Database error: %v", err)
c.JSON(500, gin.H{
    "error": "Internal server error",
})
```

---

## 💡 Decision Tree

```
هل Request نجح؟
├─ نعم
│  ├─ تم إنشاء شيء جديد؟ → 201 Created
│  ├─ لا يوجد body للإرجاع؟ → 204 No Content
│  └─ عادي → 200 OK
│
└─ لا
   ├─ الخطأ من Client؟
   │  ├─ Request format غلط؟ → 400 Bad Request
   │  ├─ لم يعرّف عن نفسه؟ → 401 Unauthorized
   │  ├─ ليس له صلاحية؟ → 403 Forbidden
   │  ├─ Resource مش موجود؟ → 404 Not Found
   │  ├─ Duplicate/Conflict؟ → 409 Conflict
   │  ├─ البيانات غير صالحة؟ → 422 Unprocessable
   │  └─ Too many requests؟ → 429 Too Many
   │
   └─ الخطأ من Server؟
      ├─ خطأ عام → 500 Internal Server Error
      ├─ Server مشغول → 503 Service Unavailable
      └─ Timeout → 504 Gateway Timeout
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **2xx** = Success
- ✅ **4xx** = Client error (المستخدم غلط)
- ✅ **5xx** = Server error (السيرفر غلط)
- ✅ استخدم **201** لـ POST (ليس 200)
- ✅ استخدم **204** لـ DELETE (ليس 200)
- ✅ **401** = not authenticated, **403** = not authorized
- ✅ **422** للـ validation errors
- ✅ لا تكشف Internal errors للمستخدم

</div>

---

<div align="center">

[⬅️ Previous: Response Structure](./04-response-structure.md) | [➡️ Next: HTTP Headers](./06-http-headers.md) | [📚 Module Home](../README.md)

</div>
