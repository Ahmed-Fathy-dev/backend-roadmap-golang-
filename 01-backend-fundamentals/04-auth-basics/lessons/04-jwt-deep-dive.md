# Lesson 4: JWT Deep Dive 🎫

<div dir="rtl">

## المقدمة

**JWT (JSON Web Token)** هو المعيار الأكثر استخداماً للـ token-based authentication!

في هذا الدرس، سنفهم JWT من الداخل - structure, security, best practices.

</div>

---

## 🎯 What is JWT?

**JWT** = **J**SON **W**eb **T**oken

<div dir="rtl">

Token مشفر يحتوي معلومات عن المستخدم، يُستخدم للـ authentication.

</div>

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo1LCJleHAiOjE2MzQwNDgwMDB9.5mXt8Z9vY3nN8kL2jD4pQ6wR7mF3hK9sT1uA6cB2eI
```

---

## 📦 JWT Structure

<div dir="rtl">

JWT يتكون من **3 أجزاء** مفصولة بنقطة:

</div>

```
HEADER.PAYLOAD.SIGNATURE
  │       │         │
  │       │         └─ التوقيع (للتحقق من الصحة)
  │       └─────────── البيانات (claims)
  └─────────────────── نوع و algorithm
```

### Part 1: Header

```json
{
  "alg": "HS256", // Algorithm
  "typ": "JWT" // Type
}
```

<div dir="rtl">

يُشفّر بـ **Base64URL**:

</div>

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### Part 2: Payload (Claims)

```json
{
  "user_id": 5,
  "email": "ahmed@test.com",
  "role": "admin",
  "exp": 1735660800, // Expiration timestamp
  "iat": 1735657200 // Issued at timestamp
}
```

<div dir="rtl">

يُشفّر بـ **Base64URL**:

</div>

```
eyJ1c2VyX2lkIjo1LCJlbWFpbCI6ImFobWVkQHRlc3QuY29tIiwicm9sZSI6ImFkbWluIiwiZXhwIjoxNzM1NjYwODAwLCJpYXQiOjE3MzU2NTcyMDB9
```

### Part 3: Signature

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

<div dir="rtl">

النتيجة:

</div>

```
5mXt8Z9vY3nN8kL2jD4pQ6wR7mF3hK9sT1uA6cB2eI
```

---

## 🔐 How JWT Works

### 1. User Login:

```go
func Login(c *gin.Context) {
    // 1. Verify credentials
    user := VerifyCredentials(email, password)

    // 2. Create JWT
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "user_id": user.ID,
        "email":   user.Email,
        "role":    user.Role,
        "exp":     time.Now().Add(24 * time.Hour).Unix(),
        "iat":     time.Now().Unix(),
    })

    // 3. Sign with secret
    secret := os.Getenv("JWT_SECRET")
    tokenString, _ := token.SignedString([]byte(secret))

    // 4. Send to client
    c.JSON(200, gin.H{
        "token": tokenString,
    })
}
```

### 2. Client Stores Token:

```javascript
// Option 1: localStorage (vulnerable to XSS)
localStorage.setItem("token", token);

// Option 2: httpOnly cookie (more secure)
// Server sets cookie automatically
```

### 3. Client Sends Token:

```http
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 4. Server Verifies Token:

```go
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 1. Extract token
        authHeader := c.GetHeader("Authorization")
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")

        // 2. Parse & verify
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            // Verify signing algorithm
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method")
            }
            return []byte(os.Getenv("JWT_SECRET")), nil
        })

        if err != nil || !token.Valid {
            c.AbortWithStatusJSON(401, gin.H{"error": "Invalid token"})
            return
        }

        // 3. Extract claims
        claims := token.Claims.(jwt.MapClaims)
        c.Set("user_id", claims["user_id"])
        c.Set("user_role", claims["role"])

        c.Next()
    }
}
```

---

## 📋 Standard Claims

<div dir="rtl">

Claims معيارية يجب استخدامها:

</div>

| Claim   | Name       | المعنى                                      |
| ------- | ---------- | ------------------------------------------- |
| **iss** | Issuer     | <div dir="rtl">من أصدر Token</div>          |
| **sub** | Subject    | <div dir="rtl">الموضوع (User ID عادة)</div> |
| **aud** | Audience   | <div dir="rtl">لمن هذا Token</div>          |
| **exp** | Expiration | <div dir="rtl">متى ينتهي</div>              |
| **iat** | Issued At  | <div dir="rtl">متى صدر</div>                |
| **nbf** | Not Before | <div dir="rtl">لا يُستخدم قبل</div>         |
| **jti** | JWT ID     | <div dir="rtl">معرّف فريد للـ Token</div>   |

### Example with Standard Claims:

```go
claims := jwt.MapClaims{
    "sub":   user.ID,                                    // Subject
    "iss":   "myapp.com",                               // Issuer
    "aud":   "myapp-clients",                           // Audience
    "exp":   time.Now().Add(15 * time.Minute).Unix(),  // Expiration
    "iat":   time.Now().Unix(),                         // Issued at
    "nbf":   time.Now().Unix(),                         // Not before
    "jti":   uuid.New().String(),                       // JWT ID

    // Custom claims
    "email": user.Email,
    "role":  user.Role,
}
```

---

## 🔒 JWT Security

### 1. Use Strong Secret

```go
// ❌ WEAK
secret := "mysecret"

// ✅ STRONG (256+ bits)
secret := os.Getenv("JWT_SECRET")  // From environment
// e.g., "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6"
```

### 2. Set Expiration

```go
// ❌ No expiration (dangerous!)
claims := jwt.MapClaims{
    "user_id": 5,
}

// ✅ Short expiration
claims := jwt.MapClaims{
    "user_id": 5,
    "exp": time.Now().Add(15 * time.Minute).Unix(),  // 15 min
}
```

### 3. Don't Store Sensitive Data

```go
// ❌ NEVER put sensitive data in JWT
claims := jwt.MapClaims{
    "password": user.Password,        // DANGER!
    "ssn": "123-45-6789",            // DANGER!
    "credit_card": "4532-...",       // DANGER!
}

// ✅ Only non-sensitive identifiers
claims := jwt.MapClaims{
    "user_id": user.ID,              // OK
    "email": user.Email,             // OK
    "role": user.Role,               // OK
}
```

<div dir="rtl">

**لماذا؟** JWT مُشفّر بـ Base64 فقط - **أي حد يقدر يفك التشفير ويقرأه!**

</div>

### 4. Use HTTPS

```
❌ HTTP: Token مكشوف في network
✅ HTTPS: Token مشفر في transit
```

---

## 🔄 Access & Refresh Tokens

### The Problem:

```
Short expiration (15 min):
✅ More secure
❌ User has to login every 15 minutes!

Long expiration (30 days):
✅ User doesn't login often
❌ If stolen, attacker has 30 days access!
```

### The Solution: Refresh Tokens

```go
// Access Token: Short-lived (15 min)
accessToken := GenerateJWT(user.ID, 15*time.Minute)

// Refresh Token: Long-lived (7 days), stored in DB
refreshToken := GenerateRefreshToken(user.ID, 7*24*time.Hour)
StoreRefreshToken(user.ID, refreshToken)

// Send both
c.JSON(200, gin.H{
    "accessToken":  accessToken,
    "refreshToken": refreshToken,
})
```

### Refresh Flow:

```
1. Access token expires (15 min)
   ↓
2. Client sends refresh token to /refresh endpoint
   ↓
3. Server validates refresh token (checks DB)
   ↓
4. Server issues NEW access token
   ↓
5. Client uses new access token
```

```go
func RefreshToken(c *gin.Context) {
    refreshToken := c.PostForm("refresh_token")

    // Verify refresh token in DB
    storedToken, err := GetRefreshToken(refreshToken)
    if err != nil {
        c.JSON(401, gin.H{"error": "Invalid refresh token"})
        return
    }

    // Check expiration
    if storedToken.ExpiresAt.Before(time.Now()) {
        c.JSON(401, gin.H{"error": "Refresh token expired"})
        return
    }

    // Generate new access token
    newAccessToken := GenerateJWT(storedToken.UserID, 15*time.Minute)

    c.JSON(200, gin.H{
        "accessToken": newAccessToken,
    })
}
```

---

## ⚡ Performance Tips

### Don't Query DB on Every Request

```go
// ❌ BAD: Query DB on every request
func AuthMiddleware() gin.HandlerFunc {
    claims := ExtractClaims(token)
    user := db.GetUser(claims.UserID)  // DB query! Slow!
}

// ✅ GOOD: Trust the token
func AuthMiddleware() gin.HandlerFunc {
    claims := ExtractClaims(token)
    c.Set("user_id", claims.UserID)   // No DB query! Fast!
    // Only query DB when you actually need full user data
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ JWT = **header.payload.signature**
- ✅ **Stateless** - Server لا يحفظ tokens
- ✅ **Self-contained** - كل المعلومات في Token
- ✅ Use **strong secret** (256+ bits)
- ✅ Set **short expiration** (15-60 min)
- ✅ **Never** store sensitive data in JWT
- ✅ Use **HTTPS** always
- ✅ Implement **refresh tokens**
- ✅ JWT readable by anyone - don't trust content from client!

</div>

---

<div align="center">

[⬅️ Previous: Token Auth](./03-token-auth.md) | [➡️ Next: Password Security](./05-password-security.md) | [📚 Module Home](../README.md)

</div>
