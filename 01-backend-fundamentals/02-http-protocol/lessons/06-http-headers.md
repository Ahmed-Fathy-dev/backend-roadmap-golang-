# Lesson 6: HTTP Headers Deep Dive 📋

<div dir="rtl">

## المقدمة

**HTTP Headers** = metadata عن Request/Response.

فهمها ضروري لبناء APIs صحيحة!

</div>

---

## 📤 Request Headers

### 1. Authorization

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Types:
Authorization: Bearer <token>     # JWT
Authorization: Basic <base64>     # username:password
```

### 2. Content-Type

```http
Content-Type: application/json
Content-Type: application/x-www-form-urlencoded
Content-Type: multipart/form-data  # File uploads
```

### 3. Accept

```http
Accept: application/json          # Client wants JSON
Accept: application/xml           # Client wants XML
Accept: */*                       # Accept anything
```

### 4. User-Agent

```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

### 5. Cookie

```http
Cookie: session_id=abc123; theme=dark
```

---

## 📥 Response Headers

### 1. Content-Type

```http
Content-Type: application/json; charset=utf-8
```

### 2. Set-Cookie

```http
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict
```

### 3. Cache-Control

```http
Cache-Control: no-cache              # Don't cache
Cache-Control: max-age=3600          # Cache for 1 hour
Cache-Control: public, max-age=86400 # Cache for 1 day
```

### 4. Location (for redirects)

```http
HTTP/1.1 201 Created
Location: /api/products/123
```

---

## 🔐 Security Headers

```http
# Prevent clickjacking
X-Frame-Options: DENY

# Prevent MIME sniffing
X-Content-Type-Options: nosniff

# XSS protection
X-XSS-Protection: 1; mode=block

# Force HTTPS
Strict-Transport-Security: max-age=31536000

# Content Security Policy
Content-Security-Policy: default-src 'self'
```

---

## 🌐 CORS Headers

```http
# Server response:
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

---

## 🔧 In Go

```go
func HandleRequest(c *gin.Context) {
    // Read request headers
    authHeader := c.GetHeader("Authorization")
    contentType := c.GetHeader("Content-Type")

    // Set response headers
    c.Header("Content-Type", "application/json")
    c.Header("X-Custom-Header", "value")

    // Security headers
    c.Header("X-Frame-Options", "DENY")
    c.Header("X-Content-Type-Options", "nosniff")

    // CORS
    c.Header("Access-Control-Allow-Origin", "*")
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Authorization:** للـ authentication
- ✅ **Content-Type:** نوع البيانات
- ✅ **Security headers:** ضرورية!
- ✅ **CORS headers:** للـ cross-origin requests
- ✅ **Cache-Control:** لتحسين الأداء

</div>

---

<div align="center">

[⬅️ Previous: Status Codes](./05-status-codes.md) | [➡️ Next: HTTPS vs HTTP](./07-https-vs-http.md) | [📚 Module Home](../README.md)

</div>
