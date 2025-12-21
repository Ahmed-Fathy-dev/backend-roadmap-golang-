# Authentication Security Checklist 🔐

<div dir="rtl">

## دليل شامل لتأمين Authentication في تطبيقك

</div>

---

## ✅ 1. Password Security

### Hashing

```go
// ✅ ALWAYS hash passwords
import "golang.org/x/crypto/bcrypt"

// Store
hashedPwd, _ := bcrypt.GenerateFromPassword(
    []byte(password),
    bcrypt.DefaultCost,  // Cost 10-12
)

// Verify
err := bcrypt.CompareHashAndPassword(
    []byte(hashedPassword),
    []byte(inputPassword),
)
```

### Password Requirements

```go
// ✅ Enforce strong passwords
type PasswordPolicy struct {
    MinLength      int  // 8-12 minimum
    RequireUpper   bool // A-Z
    RequireLower   bool // a-z
    RequireNumber  bool // 0-9
    RequireSpecial bool // !@#$%
}

// Example validation
func ValidatePassword(pwd string) error {
    if len(pwd) < 8 {
        return errors.New("password too short")
    }
    // Check other requirements...
}
```

### ❌ Never Do:

```go
// ❌ Plain text storage
user.Password = "mypassword"  // DANGER!

// ❌ Weak hashing (MD5, SHA1)
hash := md5.Sum([]byte(password))  // INSECURE!

// ❌ No salt
hash := sha256.Sum256([]byte(password))  // No salt!
```

---

## 🔑 2. JWT Security

### Token Configuration

```go
// ✅ Strong secret (256+ bits)
jwtSecret := os.Getenv("JWT_SECRET")  // From env!

// ✅ Set expiration
claims := jwt.MapClaims{
    "user_id": userID,
    "exp": time.Now().Add(15 * time.Minute).Unix(),  // Short!
}

// ✅ Use HS256 or RS256
token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
```

### Token Storage

```http
# ✅ httpOnly cookie (best for web)
Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict

# ⚠️ localStorage (vulnerable to XSS)
localStorage.setItem('token', token)  // Avoid if possible
```

### Refresh Tokens

```go
// ✅ Implement refresh tokens
accessToken := GenerateToken(user.ID, 15*time.Minute)   // Short
refreshToken := GenerateToken(user.ID, 7*24*time.Hour)  // Long

// Store refresh token in DB
StoreRefreshToken(user.ID, refreshToken)
```

---

## 🚫 3. Brute Force Protection

### Rate Limiting

```go
// ✅ Limit login attempts
import "github.com/ulule/limiter/v3"

rate := limiter.Rate{
    Period: 15 * time.Minute,
    Limit:  5,  // 5 attempts per 15 min
}

router.POST("/login", RateLimitMiddleware(rate), HandleLogin)
```

### Account Lockout

```go
// ✅ Lock account after failed attempts
type User struct {
    FailedAttempts int
    LockedUntil    *time.Time
}

func CheckLoginAttempts(user *User) error {
    if user.LockedUntil != nil && user.LockedUntil.After(time.Now()) {
        return errors.New("account locked")
    }

    if user.FailedAttempts >= 5 {
        lockUntil := time.Now().Add(30 * time.Minute)
        user.LockedUntil = &lockUntil
        return errors.New("account locked for 30 minutes")
    }

    return nil
}
```

---

## 🌐 4. HTTPS Only

```go
// ✅ Production: HTTPS only
router.RunTLS(":443", "cert.pem", "key.pem")

// ✅ Redirect HTTP to HTTPS
func RedirectToHTTPS() gin.HandlerFunc {
    return func(c *gin.Context) {
        if c.Request.TLS == nil {
            url := "https://" + c.Request.Host + c.Request.RequestURI
            c.Redirect(301, url)
            c.Abort()
        }
    }
}
```

---

## 🔒 5. CSRF Protection

```go
// ✅ Use CSRF tokens
import "github.com/utrack/gin-csrf"

router.Use(csrf.Middleware(csrf.Options{
    Secret: os.Getenv("CSRF_SECRET"),
    ErrorFunc: func(c *gin.Context) {
        c.JSON(403, gin.H{"error": "CSRF token invalid"})
    },
}))
```

---

## 🛡️ 6. Session Security

### Session Configuration

```go
// ✅ Secure session settings
store := cookie.NewStore([]byte(secret))
store.Options(sessions.Options{
    Path:     "/",
    MaxAge:   3600,      // 1 hour
    HttpOnly: true,      // No JavaScript access
    Secure:   true,      // HTTPS only
    SameSite: http.SameSiteStrictMode,  // CSRF protection
})
```

### Session Regeneration

```go
// ✅ Regenerate session ID after login
func Login(c *gin.Context) {
    // Verify credentials...

    // Destroy old session
    session := sessions.Default(c)
    session.Clear()

    // Create new session
    session.Set("user_id", user.ID)
    session.Save()
}
```

---

## ⏱️ 7. Timeout & Expiration

```go
// ✅ Implement timeouts
const (
    AccessTokenExpiry  = 15 * time.Minute  // Short-lived
    RefreshTokenExpiry = 7 * 24 * time.Hour  // Longer
    SessionExpiry      = 1 * time.Hour
)

// ✅ Absolute timeout (logout after X time regardless of activity)
session.Set("created_at", time.Now())
session.Set("user_id", userID)

// Check on each request
createdAt := session.Get("created_at").(time.Time)
if time.Since(createdAt) > 8*time.Hour {
    session.Clear()
    return errors.New("session expired")
}
```

---

## 📝 8. Audit Logging

```go
// ✅ Log all auth events
func LogAuthEvent(userID int, event string, success bool) {
    log.Printf(
        "AUTH: user_id=%d event=%s success=%v ip=%s time=%s",
        userID, event, success, ip, time.Now(),
    )
}

// Usage
LogAuthEvent(user.ID, "LOGIN", true)
LogAuthEvent(user.ID, "PASSWORD_CHANGE", true)
LogAuthEvent(0, "FAILED_LOGIN", false)
```

---

## 🔐 9. Multi-Factor Authentication (MFA)

```go
// ✅ Implement 2FA for sensitive operations
type User struct {
    TwoFactorEnabled bool
    TwoFactorSecret  string
}

func VerifyTOTP(user *User, code string) bool {
    if !user.TwoFactorEnabled {
        return true  // Skip if not enabled
    }

    // Verify TOTP code
    return totp.Validate(code, user.TwoFactorSecret)
}
```

---

## 📧 10. Email Verification

```go
// ✅ Verify email addresses
type User struct {
    Email         string
    EmailVerified bool
    VerifyToken   string
}

func SendVerificationEmail(user *User) error {
    token := GenerateRandomToken()
    user.VerifyToken = token

    verifyURL := fmt.Sprintf(
        "https://myapp.com/verify?token=%s",
        token,
    )

    return SendEmail(user.Email, "Verify Email", verifyURL)
}
```

---

## ✅ Complete Checklist

<div dir="rtl">

### Passwords

- [ ] **bcrypt** hashing (cost 10-12)
- [ ] **Strong password** requirements
- [ ] **No plain text** storage
- [ ] **Password reset** workflow secure

### JWT/Tokens

- [ ] **Strong secret** (256+ bits, from env)
- [ ] **Short expiration** (15-60 min for access)
- [ ] **Refresh tokens** implemented
- [ ] **httpOnly cookies** for web (if applicable)
- [ ] **Token revocation** mechanism

### Protection

- [ ] **Rate limiting** on login
- [ ] **Account lockout** after failed attempts
- [ ] **CSRF protection**
- [ ] **XSS prevention** (escape output)
- [ ] **SQL injection** prevention (prepared statements)

### Transport

- [ ] **HTTPS** in production
- [ ] **Secure cookies** (Secure, HttpOnly, SameSite)
- [ ] **HSTS** header

### Sessions

- [ ] **Secure configuration**
- [ ] **Session regeneration** after login
- [ ] **Timeout** configured
- [ ] **Logout** functionality

### Monitoring

- [ ] **Audit logging** for auth events
- [ ] **Failed login** monitoring
- [ ] **Anomaly detection**

### Advanced

- [ ] **Email verification**
- [ ] **2FA/MFA** (optional but recommended)
- [ ] **Password complexity** requirements
- [ ] **Remember me** (if needed)

</div>

---

## 🚨 Red Flags

<div dir="rtl">

**إذا وجدت أي من هذه - أصلحها فوراً:**

- ❌ Passwords in plain text
- ❌ Weak password requirements
- ❌ No rate limiting on login
- ❌ JWT secret in code (not env)
- ❌ No HTTPS in production
- ❌ Tokens in localStorage (XSS risk)
- ❌ No CSRF protection
- ❌ SQL queries with string concatenation
- ❌ No logging of auth events
- ❌ Same session ID before/after login

</div>

---

<div align="center">

[📚 Back to Module Home](../README.md)

</div>
