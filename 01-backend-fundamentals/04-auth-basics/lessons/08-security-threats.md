# Lesson 8: Security Threats & Prevention 🛡️

<div dir="rtl">

## المقدمة

**أهم التهديدات الأمنية** وكيف تحمي منها!

</div>

---

## 1️⃣ SQL Injection 💉

### The Attack:

```sql
-- User input: email = "x' OR '1'='1"
SELECT * FROM users WHERE email = 'x' OR '1'='1';
→ Returns ALL users! 😱
```

### Prevention:

```go
// ❌ NEVER concatenate SQL
query := "SELECT * FROM users WHERE email = '" + email + "'"

// ✅ ALWAYS use prepared statements
db.Where("email = ?", email).First(&user)
```

---

## 2️⃣ XSS (Cross-Site Scripting) 🔓

### The Attack:

```javascript
// Attacker input:
username = "<script>alert('Hacked!')</script>"

// If displayed without escaping:
<div>Welcome, <script>alert('Hacked!')</script></div>
→ Script executes! 💥
```

### Prevention:

```go
// ✅ Go templates auto-escape HTML
import "html/template"
tmpl.Execute(w, data)

// ✅ For JSON APIs
c.Header("Content-Type", "application/json")
c.Header("X-Content-Type-Options", "nosniff")

// ✅ Set CSP header
c.Header("Content-Security-Policy", "default-src 'self'")
```

---

## 3️⃣ CSRF (Cross-Site Request Forgery) 🎣

### The Attack:

```html
<!-- Evil site -->
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker" />
  <input name="amount" value="1000000" />
</form>
<script>
  document.forms[0].submit();
</script>

<!-- If user is logged into bank → money transferred! -->
```

### Prevention:

```go
// ✅ Use CSRF tokens
import "github.com/utrack/gin-csrf"

router.Use(csrf.Middleware(csrf.Options{
    Secret: os.Getenv("CSRF_SECRET"),
}))

// ✅ SameSite cookies
c.SetCookie("session", token, 3600, "/", "", true, true)
//                                              ↑
//                                         SameSite=Strict
```

---

## 4️⃣ Brute Force Attacks 🔨

### The Attack:

```
Try login with:
- password1
- password123
- password1234
... (millions of attempts)
```

### Prevention:

```go
// ✅ Rate limiting
import "github.com/ulule/limiter/v3"

rate := limiter.Rate{
    Period: 15 * time.Minute,
    Limit:  5,  // 5 attempts per 15 min
}

router.POST("/login", RateLimitMiddleware(rate), Login)

// ✅ Account lockout
if user.FailedAttempts >= 5 {
    return errors.New("Account locked for 30 minutes")
}
```

---

## 5️⃣ Weak Passwords 🔑

### Prevention:

```go
// ✅ Enforce password policy
type PasswordPolicy struct {
    MinLength      int   // 8+
    RequireUpper   bool
    RequireLower   bool
    RequireNumber  bool
    RequireSpecial bool
}

// ✅ Use bcrypt
hashedPwd, _ := bcrypt.GenerateFromPassword(
    []byte(password),
    bcrypt.DefaultCost,
)
```

---

## 🔐 Security Headers

```go
func SecurityHeaders() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("X-Frame-Options", "DENY")
        c.Header("X-Content-Type-Options", "nosniff")
        c.Header("X-XSS-Protection", "1; mode=block")
        c.Header("Strict-Transport-Security", "max-age=31536000")
        c.Header("Content-Security-Policy", "default-src 'self'")
        c.Next()
    }
}
```

---

## ✅ Security Checklist

<div dir="rtl">

- [ ] **SQL Injection:** Prepared statements ✅
- [ ] **XSS:** Escape output ✅
- [ ] **CSRF:** CSRF tokens ✅
- [ ] **Brute Force:** Rate limiting ✅
- [ ] **Passwords:** bcrypt + policy ✅
- [ ] **HTTPS:** In production ✅
- [ ] **Headers:** Security headers ✅
- [ ] **Validation:** All input validated ✅

</div>

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Never trust user input!**
- ✅ Use **prepared statements**
- ✅ **Escape** all output
- ✅ **CSRF** tokens للـ forms
- ✅ **Rate limiting** على login
- ✅ **Strong passwords** required
- ✅ **HTTPS** always in production

</div>

---

<div align="center">

[⬅️ Previous: RBAC](./07-rbac.md) | [📚 Module Home](../README.md)

</div>
