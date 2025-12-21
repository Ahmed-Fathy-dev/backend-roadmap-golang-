# Security Guide: HTTP & Backend Security 🔐

<div dir="rtl">

## نظرة عامة

هذا الدليل يغطي **أهم مفاهيم الأمان** في HTTP و Backend Development.

**تذكر:** الأمان ليس "ميزة إضافية" - إنه **ضرورة**!

</div>

---

## 🛡️ 1. HTTPS - Always!

### HTTP vs HTTPS

```
❌ HTTP (Plain Text):
Browser ──► GET /login?password=123 ──► Server
         👁️ Anyone can read this!

✅ HTTPS (Encrypted):
Browser ──► 🔒 encrypted data 🔒 ──► Server
         🛡️ Only browser & server can read
```

### Why HTTPS?

<div dir="rtl">

1. **Encryption**: البيانات مشفرة
2. **Integrity**: لا يمكن تعديل البيانات في المنتصف
3. **Authentication**: التحقق من هوية السيرفر
4. **Trust**: Google تفضّل HTTPS في Search Results

</div>

### Implementation:

```go
// Production: ALWAYS use HTTPS
router.RunTLS(":443", "cert.pem", "key.pem")

// Development only
router.Run(":8080")
```

---

## 🔐 2. Authentication & Authorization

### Hash Passwords - ALWAYS!

```go
import "golang.org/x/crypto/bcrypt"

// ❌ NEVER store passwords as plain text!
user.Password = "mypassword123"  // DANGER!

// ✅ ALWAYS hash
hashedPwd, _ := bcrypt.GenerateFromPassword(
    []byte(password),
    bcrypt.DefaultCost,  // Cost 10-12 recommended
)
user.Password = string(hashedPwd)
```

### JWT Best Practices:

```go
// ✅ Use strong secret (256+ bits)
jwtSecret := os.Getenv("JWT_SECRET")  // From env, not hardcoded!

// ✅ Set expiration
token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
    "user_id": userID,
    "exp": time.Now().Add(24 * time.Hour).Unix(),  // Expires in 24h
})

// ✅ Use HTTPS to transmit tokens
// ✅ Store in httpOnly cookies (not localStorage)
c.SetCookie("token", tokenString, 86400, "/", "", true, true)
                                                    ↑     ↑
                                                  secure httpOnly
```

---

## 🔒 3. Input Validation & Sanitization

### SQL Injection Prevention

```go
// ❌ EXTREMELY DANGEROUS!
query := "SELECT * FROM users WHERE email = '" + email + "'"
db.Exec(query)

// Attacker input: email = "x' OR '1'='1"
// Result: SELECT * FROM users WHERE email = 'x' OR '1'='1'
//         → Returns ALL users! 💥

// ✅ ALWAYS use prepared statements
query := "SELECT * FROM users WHERE email = $1"
db.QueryRow(query, email)  // Safe!
```

### Validate ALL Input:

```go
import "github.com/go-playground/validator/v10"

type CreateUserRequest struct {
    Email    string `binding:"required,email"`
    Password string `binding:"required,min=8,max=100"`
    Age      int    `binding:"required,gte=18,lte=120"`
}

func CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    // req is now validated!
}
```

---

## 🚫 4. XSS (Cross-Site Scripting) Prevention

### The Attack:

```javascript
// Attacker submits:
username = "<script>alert('Hacked!')</script>"

// If you display this without escaping:
<div>Welcome, <script>alert('Hacked!')</script></div>
→ Script executes! 💥
```

### Prevention:

```go
import "html/template"

// ✅ Go templates auto-escape
tmpl := template.Must(template.ParseFiles("page.html"))
tmpl.Execute(w, data)  // Automatically escapes HTML

// ✅ For JSON APIs: Use Content-Type headers
c.Header("Content-Type", "application/json")
c.Header("X-Content-Type-Options", "nosniff")
```

---

## 🔐 5. CSRF (Cross-Site Request Forgery) Prevention

### The Attack:

```html
<!-- Evil site creates hidden form -->
<form action="https://yourbank.com/transfer" method="POST">
  <input name="to" value="attacker" />
  <input name="amount" value="1000000" />
</form>
<script>
  document.forms[0].submit();
</script>

<!-- If user is logged into bank → money transferred! 💰💸 -->
```

### Prevention:

```go
import "github.com/utrack/gin-csrf"

// Add CSRF middleware
router.Use(csrf.Middleware(csrf.Options{
    Secret: "your-secret-key",
    ErrorFunc: func(c *gin.Context) {
        c.JSON(403, gin.H{"error": "CSRF token validation failed"})
    },
}))

// Include token in forms
c.HTML(200, "form.html", gin.H{
    "csrf_token": csrf.GetToken(c),
})
```

---

## ⏱️ 6. Rate Limiting

### Prevent Brute Force:

```go
import "github.com/ulule/limiter/v3"

// 10 requests per minute
rate := limiter.Rate{
    Period: 1 * time.Minute,
    Limit:  10,
}

store := memory.NewStore()
limiter := limiter.New(store, rate)

// Apply to sensitive endpoints
router.POST("/login", RateLimitMiddleware(limiter), HandleLogin)
```

---

## 🔑 7. Security Headers

```go
func SecurityHeaders() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Prevent clickjacking
        c.Header("X-Frame-Options", "DENY")

        // Prevent MIME sniffing
        c.Header("X-Content-Type-Options", "nosniff")

        // Enable XSS filter
        c.Header("X-XSS-Protection", "1; mode=block")

        // HTTPS only
        c.Header("Strict-Transport-Security", "max-age=31536000")

        // Content Security Policy
        c.Header("Content-Security-Policy", "default-src 'self'")

        c.Next()
    }
}

router.Use(SecurityHeaders())
```

---

## 🚨 8. Common Vulnerabilities Checklist

<div dir="rtl">

### ✅ افحص دائماً:

- [ ] **HTTPS** في Production
- [ ] **Passwords** مُهَشّٓة (bcrypt)
- [ ] **SQL Injection** - استخدم Prepared Statements
- [ ] **XSS** - Escape output
- [ ] **CSRF** - استخدم CSRF tokens
- [ ] **Rate Limiting** على Login/Sensitive endpoints
- [ ] **Input Validation** - كل input من المستخدم
- [ ] **Authentication** - JWT أو Sessions آمنة
- [ ] **Authorization** - تحقق من Permissions
- [ ] **Security Headers** - أضف كل Headers المهمة
- [ ] **Secrets** في Environment Variables (ليس في Code)
- [ ] **Error Messages** - لا تكشف تفاصيل داخلية
- [ ] **File Uploads** - Validate type & size
- [ ] **CORS** - Configure بشكل صحيح

</div>

---

## 💡 Security Mindset

<div dir="rtl">

**القاعدة الذهبية:**

> "Never trust user input!"

**كل ما يأتي من المستخدم محتمل أن يكون:**

- ❌ Malicious (ضار)
- ❌ Invalid (غير صحيح)
- ❌ Unexpected (غير متوقع)

**دائماً:**

- ✅ Validate
- ✅ Sanitize
- ✅ Escape
- ✅ Check permissions
- ✅ Log security events

</div>

---

<div align="center">

[📚 Back to Module Home](../README.md)

</div>
