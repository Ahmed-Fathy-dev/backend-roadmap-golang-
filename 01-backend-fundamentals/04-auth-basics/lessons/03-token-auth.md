# Lesson 3: Token-Based Authentication 🎫

<div dir="rtl">

## المقدمة

**Token-based Authentication** هو الأسلوب الأكثر استخداماً في الـ Modern APIs!

بدل ما نحفظ session على السيرفر، بنعطي الـ client token يحمله معاه.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 Session vs Token

```
┌─────────────────────────────────────────────────────────────────────┐
│              Session-Based vs Token-Based                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Session-Based:                                                      │
│  ─────────────                                                       │
│  ┌────────┐    1. Login            ┌────────┐                       │
│  │ Client │ ──────────────────────▶│ Server │                       │
│  └────────┘                        └────────┘                       │
│       ▲                                 │                           │
│       │    2. Session ID (Cookie)       │ Stores in DB/Memory       │
│       └─────────────────────────────────┘                           │
│                                                                      │
│  Token-Based:                                                        │
│  ────────────                                                        │
│  ┌────────┐    1. Login            ┌────────┐                       │
│  │ Client │ ──────────────────────▶│ Server │                       │
│  └────────┘                        └────────┘                       │
│       ▲                                 │                           │
│       │    2. Token (JWT)               │ Stateless!                │
│       └─────────────────────────────────┘ (No storage needed)       │
│                                                                      │
│  Key Difference:                                                     │
│  • Session: Server must store session data                          │
│  • Token: Server just verifies signature (stateless)                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ How Token Auth Works

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Token Authentication Flow                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Step 1: Login                                                       │
│  ─────────────                                                       │
│  Client                              Server                          │
│    │                                    │                            │
│    │  POST /login                       │                            │
│    │  {email, password}                 │                            │
│    │ ──────────────────────────────────▶│                            │
│    │                                    │ Verify credentials        │
│    │                                    │ Generate token            │
│    │◀────────────────────────────────── │                            │
│    │  {token: "eyJhbG..."}             │                            │
│    │                                    │                            │
│  Step 2: Store Token                                                 │
│  ───────────────────                                                 │
│    │                                    │                            │
│    │  Store in localStorage             │                            │
│    │  or httpOnly cookie                │                            │
│    │                                    │                            │
│  Step 3: Use Token                                                   │
│  ─────────────────                                                   │
│    │                                    │                            │
│    │  GET /api/profile                  │                            │
│    │  Authorization: Bearer eyJhbG...   │                            │
│    │ ──────────────────────────────────▶│                            │
│    │                                    │ Verify token signature    │
│    │                                    │ Extract user data         │
│    │◀────────────────────────────────── │                            │
│    │  {user data}                       │                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Code Example

```go
// Login endpoint
func Login(c *gin.Context) {
    var credentials struct {
        Email    string `json:"email"`
        Password string `json:"password"`
    }
    c.BindJSON(&credentials)

    // 1. Verify credentials
    user, err := db.FindUserByEmail(credentials.Email)
    if err != nil || !CheckPassword(credentials.Password, user.PasswordHash) {
        c.JSON(401, gin.H{"error": "Invalid credentials"})
        return
    }

    // 2. Generate token
    token, err := GenerateJWT(user.ID, user.Email, user.Role)
    if err != nil {
        c.JSON(500, gin.H{"error": "Could not generate token"})
        return
    }

    // 3. Return token
    c.JSON(200, gin.H{
        "token": token,
        "user": gin.H{
            "id":    user.ID,
            "email": user.Email,
            "name":  user.Name,
        },
    })
}

// Auth middleware
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 1. Get token from header
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.AbortWithStatusJSON(401, gin.H{"error": "No token provided"})
            return
        }

        // 2. Extract token (remove "Bearer ")
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")

        // 3. Verify token
        claims, err := ValidateJWT(tokenString)
        if err != nil {
            c.AbortWithStatusJSON(401, gin.H{"error": "Invalid token"})
            return
        }

        // 4. Set user info in context
        c.Set("user_id", claims.UserID)
        c.Set("user_email", claims.Email)
        c.Set("user_role", claims.Role)

        c.Next()
    }
}

// Protected endpoint
func GetProfile(c *gin.Context) {
    userID := c.GetInt("user_id")

    user, err := db.FindUserByID(userID)
    if err != nil {
        c.JSON(404, gin.H{"error": "User not found"})
        return
    }

    c.JSON(200, user)
}
```

---

## 2️⃣ Token Types

### Bearer Token

```http
GET /api/users HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ...
```

<div dir="rtl">

**Bearer** = "حامل" - يعني أي حد يحمل الـ token يقدر يستخدمه.

</div>

### API Key

```http
GET /api/data HTTP/1.1
Host: api.example.com
X-API-Key: sk_live_abc123xyz
```

<div dir="rtl">

**API Key** = مفتاح ثابت للـ machine-to-machine authentication.

</div>

### Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Token Types Comparison                            │
├────────────────┬────────────────────┬───────────────────────────────┤
│     Type       │      Use Case      │           Notes               │
├────────────────┼────────────────────┼───────────────────────────────┤
│ JWT            │ User auth          │ Contains user data, expires   │
│ API Key        │ Service auth       │ Long-lived, no user data      │
│ OAuth Token    │ Third-party auth   │ Scoped access, refresh token  │
│ Session Token  │ Web apps           │ Opaque, server-stored         │
└────────────────┴────────────────────┴───────────────────────────────┘
```

---

## 3️⃣ Token Storage

### Client-Side Options

```javascript
// Option 1: localStorage (NOT recommended for auth tokens)
// ❌ Vulnerable to XSS attacks
localStorage.setItem('token', token);
const token = localStorage.getItem('token');

// Option 2: sessionStorage (slightly better)
// ❌ Still vulnerable to XSS
sessionStorage.setItem('token', token);

// Option 3: httpOnly Cookie (RECOMMENDED)
// ✅ Not accessible via JavaScript = XSS safe
// Server sets this:
// Set-Cookie: token=eyJ...; HttpOnly; Secure; SameSite=Strict

// Option 4: Memory (for SPAs)
// ✅ Lost on refresh (combine with refresh token)
let token = null;
function setToken(t) { token = t; }
function getToken() { return token; }
```

### Recommendation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Token Storage Recommendation                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  For Web Applications:                                               │
│  ─────────────────────                                               │
│  Access Token  → Memory (JS variable)                               │
│  Refresh Token → httpOnly Cookie                                    │
│                                                                      │
│  Why?                                                                │
│  • Access token: Short-lived, lost on refresh is OK                │
│  • Refresh token: Long-lived, must be protected from XSS           │
│                                                                      │
│  For Mobile Applications:                                            │
│  ────────────────────────                                            │
│  Access Token  → Secure storage (Keychain/Keystore)                 │
│  Refresh Token → Secure storage (Keychain/Keystore)                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ Token Transmission

### Authorization Header

```http
GET /api/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# Most common and recommended
```

### Cookie

```http
GET /api/profile HTTP/1.1
Host: api.example.com
Cookie: token=eyJhbGciOiJIUzI1NiIs...

# Automatically sent with same-origin requests
# Needs CSRF protection
```

### Query Parameter (NOT RECOMMENDED)

```http
GET /api/profile?token=eyJhbGciOiJIUzI1NiIs... HTTP/1.1

# ❌ Token visible in logs, browser history, referrer
# Only use for temporary use cases (password reset links)
```

---

## 5️⃣ Advantages & Disadvantages

### Advantages ✅

```
1. Stateless
   └─ Server doesn't store sessions
   └─ Easy to scale horizontally
   └─ Any server can validate token

2. Cross-Domain
   └─ Works across different domains
   └─ Good for microservices
   └─ Mobile apps, SPAs, APIs

3. Self-Contained
   └─ Token contains user info
   └─ No database lookup needed
   └─ Faster authentication

4. Decentralized
   └─ Auth server can be separate
   └─ Microservices can validate independently
```

### Disadvantages ❌

```
1. Token Size
   └─ Larger than session ID
   └─ Sent with every request
   └─ Can be several KB

2. Cannot Invalidate Easily
   └─ Token valid until expiry
   └─ Need blacklist for logout
   └─ Tricky for password change

3. Secret Key Management
   └─ If secret is compromised, all tokens are
   └─ Need secure key rotation

4. XSS Risk (if stored in localStorage)
   └─ Attacker can steal token
   └─ Use httpOnly cookies
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                 Token Auth Best Practices                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Short Expiration                                                 │
│     └─ Access token: 15 minutes                                     │
│     └─ Use refresh tokens for longer sessions                       │
│                                                                      │
│  2. Secure Transmission                                              │
│     └─ Always use HTTPS                                              │
│     └─ Use Authorization header (not URL)                           │
│                                                                      │
│  3. Secure Storage                                                   │
│     └─ httpOnly cookies for web                                      │
│     └─ Keychain/Keystore for mobile                                 │
│                                                                      │
│  4. Minimal Claims                                                   │
│     └─ Don't put sensitive data in token                            │
│     └─ Only include necessary info                                  │
│                                                                      │
│  5. Token Rotation                                                   │
│     └─ Issue new tokens regularly                                    │
│     └─ Invalidate old refresh tokens                                │
│                                                                      │
│  6. Validate Everything                                              │
│     └─ Check expiration                                              │
│     └─ Verify signature                                              │
│     └─ Validate issuer and audience                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Token Auth** = الـ client يحمل الـ token مع كل request
- ✅ **Stateless** = السيرفر مش محتاج يحفظ sessions
- ✅ **JWT** = أشهر نوع tokens (هنتعلمه بالتفصيل)
- ✅ **Bearer Token** = الطريقة المعيارية للإرسال
- ✅ **httpOnly Cookie** = أأمن طريقة للتخزين
- ✅ **Short Expiration** = لتقليل المخاطر

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد ما فهمت Token Auth، خلينا نتعمق في **JWT**:

**➡️ [Lesson 4: JWT Deep Dive](./04-jwt-deep-dive.md)**

</div>

---

<div align="center">

[⬅️ Previous: Session Auth](./02-session-auth.md) | [📚 Module Home](../README.md) | [➡️ Next: JWT Deep Dive](./04-jwt-deep-dive.md)

</div>
