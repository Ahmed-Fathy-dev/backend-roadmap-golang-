# Lesson 2: Session-Based Authentication 🍪

<div dir="rtl">

## المقدمة

**Session-based auth** = الطريقة التقليدية للـ authentication باستخدام **Sessions** و **Cookies**.

</div>

---

## 🎯 How It Works

```
1. User logs in
   ↓
2. Server creates session
   Session ID: "abc123xyz"
   ↓
3. Server stores session in memory/database
   ↓
4. Server sends session ID to client in cookie
   Set-Cookie: session_id=abc123xyz
   ↓
5. Client stores cookie
   ↓
6. Client sends cookie with every request
   Cookie: session_id=abc123xyz
   ↓
7. Server looks up session
   ↓
8. Server knows who you are!
```

---

## 🔧 Implementation in Go

```go
import (
    "github.com/gin-contrib/sessions"
    "github.com/gin-contrib/sessions/cookie"
)

func main() {
    router := gin.Default()

    // Setup session store
    store := cookie.NewStore([]byte("secret-key-123"))
    router.Use(sessions.Sessions("mysession", store))

    router.POST("/login", Login)
    router.GET("/profile", AuthRequired(), GetProfile)
    router.POST("/logout", Logout)

    router.Run()
}

// Login handler
func Login(c *gin.Context) {
    var req struct {
        Email    string `json:"email"`
        Password string `json:"password"`
    }
    c.BindJSON(&req)

    // Verify credentials
    user, err := VerifyCredentials(req.Email, req.Password)
    if err != nil {
        c.JSON(401, gin.H{"error": "Invalid credentials"})
        return
    }

    // Create session
    session := sessions.Default(c)
    session.Set("user_id", user.ID)
    session.Set("email", user.Email)
    session.Save()

    c.JSON(200, gin.H{"message": "Logged in"})
}

// Auth middleware
func AuthRequired() gin.HandlerFunc {
    return func(c *gin.Context) {
        session := sessions.Default(c)
        userID := session.Get("user_id")

        if userID == nil {
            c.AbortWithStatusJSON(401, gin.H{
                "error": "Please login",
            })
            return
        }

        c.Set("user_id", userID)
        c.Next()
    }
}

// Logout
func Logout(c *gin.Context) {
    session := sessions.Default(c)
    session.Clear()
    session.Save()

    c.JSON(200, gin.H{"message": "Logged out"})
}
```

---

## 🍪 Cookies

```http
# Server sends cookie
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict

# Client sends cookie automatically
Cookie: session_id=abc123
```

### Cookie Attributes:

```
HttpOnly  → JavaScript can't access (XSS protection)
Secure    → Only sent over HTTPS
SameSite  → CSRF protection
```

---

## ✅ Pros & Cons

**Pros:**

- ✅ Simple to implement
- ✅ Server controls sessions
- ✅ Easy to revoke (just delete session)
- ✅ Works great for web browsers

**Cons:**

- ❌ Scalability issues (sessions in memory/DB)
- ❌ Doesn't work well for mobile apps
- ❌ Requires server-side storage
- ❌ Hard to scale horizontally

---

## 🆚 Session vs Token

| Aspect          | Session     | Token (JWT) |
| --------------- | ----------- | ----------- |
| **Storage**     | Server-side | Client-side |
| **Scalability** | Harder      | Easier      |
| **Revocation**  | Easy        | Harder      |
| **Mobile**      | Harder      | Better      |
| **Stateless**   | No          | Yes         |

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ Session ID stored in **cookie**
- ✅ Session data stored on **server**
- ✅ **HttpOnly** cookies للأمان
- ✅ Good for **web apps**
- ✅ Requires **server-side storage**

</div>

---

<div align="center">

[⬅️ Previous: Authentication vs Authorization](./01-authn-vs-authz.md) | [➡️ Next: Token Auth](./03-token-auth.md) | [📚 Module Home](../README.md)

</div>
