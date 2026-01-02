# Lesson 2: CORS - Cross-Origin Resource Sharing 🌐

<div dir="rtl">

## المقدمة

**CORS** هو آلية أمان تمنع الـ websites من الوصول لـ APIs على domains تانية بدون إذن.

لو عندك frontend على `myapp.com` وـ API على `api.myapp.com` - لازم تفهم CORS!

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 The Problem

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Same-Origin Policy                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Browser Security Rule:                                              │
│  ──────────────────────                                              │
│  JavaScript can only make requests to the SAME origin               │
│                                                                      │
│  Origin = Protocol + Domain + Port                                   │
│                                                                      │
│  Examples:                                                           │
│  ─────────                                                           │
│  https://myapp.com     ───▶ https://myapp.com/api     ✅ Same       │
│  https://myapp.com     ───▶ http://myapp.com/api      ❌ Different  │
│  https://myapp.com     ───▶ https://api.myapp.com     ❌ Different  │
│  https://myapp.com     ───▶ https://myapp.com:8080    ❌ Different  │
│  https://myapp.com     ───▶ https://google.com/api    ❌ Different  │
│                                                                      │
│  Without CORS:                                                       │
│  ─────────────                                                       │
│  evil-site.com could read your data from bank-api.com!             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ How CORS Works

### Simple Requests

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Simple CORS Request                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Browser (myapp.com)              API (api.myapp.com)               │
│        │                                  │                         │
│        │  GET /users                      │                         │
│        │  Origin: https://myapp.com       │                         │
│        │ ────────────────────────────────▶│                         │
│        │                                  │                         │
│        │                                  │ Check: Is myapp.com     │
│        │                                  │ allowed?                │
│        │                                  │                         │
│        │  200 OK                          │                         │
│        │  Access-Control-Allow-Origin:    │                         │
│        │    https://myapp.com             │                         │
│        │ ◀────────────────────────────────│                         │
│        │                                  │                         │
│  Browser checks header → Origin matches → Request allowed! ✅       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Preflight Requests (Complex Requests)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Preflight Request (OPTIONS)                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Complex requests trigger preflight:                                 │
│  • Methods: PUT, DELETE, PATCH                                      │
│  • Custom headers: Authorization, X-Custom-Header                   │
│  • Content-Type: application/json                                   │
│                                                                      │
│  Browser (myapp.com)              API (api.myapp.com)               │
│        │                                  │                         │
│        │  1. OPTIONS /users (preflight)   │                         │
│        │  Origin: https://myapp.com       │                         │
│        │  Access-Control-Request-Method:  │                         │
│        │    POST                          │                         │
│        │  Access-Control-Request-Headers: │                         │
│        │    Content-Type, Authorization   │                         │
│        │ ────────────────────────────────▶│                         │
│        │                                  │                         │
│        │  2. 204 No Content               │                         │
│        │  Access-Control-Allow-Origin:    │                         │
│        │    https://myapp.com             │                         │
│        │  Access-Control-Allow-Methods:   │                         │
│        │    GET, POST, PUT, DELETE        │                         │
│        │  Access-Control-Allow-Headers:   │                         │
│        │    Content-Type, Authorization   │                         │
│        │  Access-Control-Max-Age: 86400   │                         │
│        │ ◀────────────────────────────────│                         │
│        │                                  │                         │
│        │  3. Actual POST /users           │                         │
│        │  (with Authorization header)     │                         │
│        │ ────────────────────────────────▶│                         │
│        │                                  │                         │
│        │  4. 201 Created                  │                         │
│        │ ◀────────────────────────────────│                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ CORS Headers

### Response Headers (Server → Browser)

```http
# Required: Which origin is allowed
Access-Control-Allow-Origin: https://myapp.com
# Or allow all (NOT recommended for APIs with auth):
Access-Control-Allow-Origin: *

# Which methods are allowed
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS

# Which headers client can send
Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With

# Allow credentials (cookies, auth headers)
Access-Control-Allow-Credentials: true

# How long to cache preflight response (seconds)
Access-Control-Max-Age: 86400

# Which headers client can read from response
Access-Control-Expose-Headers: X-Total-Count, X-Page-Count
```

### Request Headers (Browser → Server)

```http
# Browser sends automatically:
Origin: https://myapp.com

# In preflight requests:
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization
```

---

## 3️⃣ Implementation in Go

### Basic CORS Middleware

```go
func CORSMiddleware(allowedOrigins []string) gin.HandlerFunc {
    return func(c *gin.Context) {
        origin := c.Request.Header.Get("Origin")

        // Check if origin is allowed
        allowed := false
        for _, o := range allowedOrigins {
            if o == origin || o == "*" {
                allowed = true
                break
            }
        }

        if allowed {
            c.Header("Access-Control-Allow-Origin", origin)
            c.Header("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
            c.Header("Access-Control-Allow-Headers", "Content-Type, Authorization")
            c.Header("Access-Control-Allow-Credentials", "true")
            c.Header("Access-Control-Max-Age", "86400")
        }

        // Handle preflight
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }

        c.Next()
    }
}

// Usage
func main() {
    r := gin.Default()

    // Allow specific origins
    r.Use(CORSMiddleware([]string{
        "https://myapp.com",
        "https://www.myapp.com",
        "http://localhost:3000",  // for development
    }))

    r.GET("/api/users", getUsers)
    r.Run(":8080")
}
```

### Using gin-cors Package

```go
import "github.com/gin-contrib/cors"

func main() {
    r := gin.Default()

    // Configure CORS
    config := cors.Config{
        AllowOrigins:     []string{"https://myapp.com", "http://localhost:3000"},
        AllowMethods:     []string{"GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"},
        AllowHeaders:     []string{"Origin", "Content-Type", "Authorization"},
        ExposeHeaders:    []string{"Content-Length", "X-Total-Count"},
        AllowCredentials: true,
        MaxAge:           12 * time.Hour,
    }

    r.Use(cors.New(config))

    // Or allow all origins (development only!)
    // r.Use(cors.Default())

    r.Run(":8080")
}
```

---

## 4️⃣ Common Scenarios

### Scenario 1: Frontend & Backend on Same Domain

```
Frontend: https://myapp.com
Backend:  https://myapp.com/api

CORS needed? NO! ✅ Same origin
```

### Scenario 2: Separate Subdomains

```
Frontend: https://myapp.com
Backend:  https://api.myapp.com

CORS needed? YES! ❌ Different origin

Config:
Access-Control-Allow-Origin: https://myapp.com
```

### Scenario 3: Development vs Production

```go
func getCORSConfig() cors.Config {
    if os.Getenv("ENV") == "production" {
        return cors.Config{
            AllowOrigins: []string{
                "https://myapp.com",
                "https://www.myapp.com",
            },
            AllowCredentials: true,
            // ... other settings
        }
    }

    // Development: more permissive
    return cors.Config{
        AllowOrigins:     []string{"http://localhost:3000", "http://localhost:5173"},
        AllowMethods:     []string{"*"},
        AllowHeaders:     []string{"*"},
        AllowCredentials: true,
    }
}
```

### Scenario 4: Public API

```go
// Public API (no auth, read-only)
config := cors.Config{
    AllowOrigins: []string{"*"},  // Anyone can access
    AllowMethods: []string{"GET", "OPTIONS"},  // Read-only
    AllowHeaders: []string{"Content-Type"},
    // AllowCredentials: false (default)
}

// ⚠️ Don't use * with AllowCredentials: true!
```

---

## 5️⃣ Common Mistakes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CORS Common Mistakes                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ Mistake 1: Using * with credentials                             │
│  ─────────────────────────────────────                               │
│  Access-Control-Allow-Origin: *                                     │
│  Access-Control-Allow-Credentials: true                             │
│  → Browser will reject! Cannot use * with credentials               │
│                                                                      │
│  ✅ Fix: Echo back the specific origin                              │
│  Access-Control-Allow-Origin: https://myapp.com                     │
│                                                                      │
│  ────────────────────────────────────────────────────────────────── │
│                                                                      │
│  ❌ Mistake 2: Forgetting preflight handler                         │
│  ─────────────────────────────────────────                           │
│  OPTIONS /api/users returns 404                                     │
│  → Browser blocks the actual request                                │
│                                                                      │
│  ✅ Fix: Handle OPTIONS method                                       │
│  if method == "OPTIONS" { return 204 }                              │
│                                                                      │
│  ────────────────────────────────────────────────────────────────── │
│                                                                      │
│  ❌ Mistake 3: Missing headers in Allow-Headers                     │
│  ─────────────────────────────────────────────                       │
│  Client sends: Authorization, X-Custom-Header                       │
│  Server allows: Content-Type (only)                                 │
│  → Request blocked!                                                 │
│                                                                      │
│  ✅ Fix: Include all headers client needs to send                   │
│                                                                      │
│  ────────────────────────────────────────────────────────────────── │
│                                                                      │
│  ❌ Mistake 4: CORS in wrong order                                  │
│  ───────────────────────────────────                                 │
│  r.GET("/api/users", getUsers)                                      │
│  r.Use(CORSMiddleware())  // Too late!                              │
│                                                                      │
│  ✅ Fix: Add CORS middleware first                                   │
│  r.Use(CORSMiddleware())                                            │
│  r.GET("/api/users", getUsers)                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6️⃣ Debugging CORS

### Browser Developer Tools

```javascript
// Console error you might see:
Access to XMLHttpRequest at 'https://api.myapp.com/users'
from origin 'https://myapp.com' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

### Check Response Headers

```bash
# Using curl to see CORS headers
curl -I -X OPTIONS \
  -H "Origin: https://myapp.com" \
  -H "Access-Control-Request-Method: POST" \
  https://api.myapp.com/users

# Expected response:
HTTP/2 204
access-control-allow-origin: https://myapp.com
access-control-allow-methods: GET, POST, PUT, DELETE
access-control-allow-headers: Content-Type, Authorization
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **CORS** = Browser security لمنع الـ cross-origin requests
- ✅ **Same Origin** = نفس Protocol + Domain + Port
- ✅ **Preflight** = OPTIONS request قبل الـ complex requests
- ✅ **لا تستخدم `*`** مع credentials
- ✅ **Handle OPTIONS** method
- ✅ **Add middleware first** قبل الـ routes

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 3: Rate Limiting](./03-rate-limiting.md)**

</div>

---

<div align="center">

[⬅️ Previous: HTTPS & TLS](./01-https-tls.md) | [📚 Module Home](../README.md) | [➡️ Next: Rate Limiting](./03-rate-limiting.md)

</div>
