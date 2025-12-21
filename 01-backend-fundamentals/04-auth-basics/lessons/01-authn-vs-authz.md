# Lesson 1: Authentication vs Authorization 🔐

<div dir="rtl">

## المقدمة

**Authentication** و **Authorization** - كلمتين متشابهتين لكن مختلفتين تماماً!

فهم الفرق بينهما **ضروري** لبناء نظام أمان صحيح.

</div>

---

## 🆚 The Fundamental Difference

```
┌─────────────────────────────────────────────────────────┐
│  Authentication (المصادقة - التحقق من الهوية)          │
│                                                         │
│  السؤال: "من أنت؟"                                     │
│  الإجابة: "أنا أحمد" (مع إثبات)                       │
│                                                         │
│  Process: Login with username + password               │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Authorization (التفويض - التحقق من الصلاحيات)         │
│                                                         │
│  السؤال: "ماذا يمكنك أن تفعل؟"                         │
│  الإجابة: "يمكنني القراءة فقط" أو "يمكنني الحذف"      │
│                                                         │
│  Process: Check permissions for action                  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔐 Authentication (المصادقة)

<div dir="rtl">

### التعريف:

عملية **التحقق من هوية** المستخدم.

### مثال من الحيا ة اليومية:

</div>

```
المطار:
1. تعرض جواز السفر → Authentication
2. الموظف يتأكد: "هل هذا أنت حقاً؟"
3. ✅ تم التحقق من هويتك

تطبيق ويب:
1. تدخل email + password → Authentication
2. Server يتحقق: "هل البيانات صحيحة؟"
3. ✅ تسجيل دخول ناجح
```

### Authentication Methods:

#### 1. Password-Based

```go
// User submits credentials
email := "ahmed@test.com"
password := "mypassword123"

// Server verifies
user := GetUserByEmail(email)
if bcrypt.CompareHashAndPassword(user.Password, password) == nil {
    // ✅ Authenticated!
}
```

#### 2. Token-Based (JWT)

```go
// After successful login
token := GenerateJWT(user.ID)

// Future requests include token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Server verifies token
claims := VerifyToken(token)
// ✅ Authenticated as user ID = claims.UserID
```

#### 3. Social Login (OAuth)

```
"Login with Google"
→ Google verifies you
→ Sends confirmation to your app
→ ✅ Authenticated via Google
```

#### 4. Multi-Fact or Authentication (2FA)

```
Step 1: Password ✅
Step 2: 6-digit code from phone ✅
→ ✅ Fully authenticated
```

---

## 🔓 Authorization (التفويض)

<div dir="rtl">

### التعريف:

عملية **التحقق من الصلاحيات** - هل يُسمح للمستخدم بفعل هذا؟

### مثال من الحياة اليومية:

</div>

```
الفندق:
1. عندك مفتاح (authenticated)
2. تحاول تفتح غرفة 501
3. المفتاح يفتح فقط غرفة 402
→ ❌ Not authorized للغرفة 501

GitHub:
1. مسجل دخول (authenticated)
2. تحاول تحذف repository
3. أنت contributor فقط، مش owner
→ ❌ Not authorized to delete
```

### Authorization Methods:

#### 1. Role-Based (RBAC)

```go
type User struct {
    ID   int
    Role string  // "admin", "user", "moderator"
}

func DeleteProduct(c *gin.Context) {
    user := GetCurrentUser(c)

    // Check authorization
    if user.Role != "admin" {
        c.JSON(403, gin.H{"error": "Admin access required"})
        return
    }

    // ✅ Authorized - proceed
    DeleteProduct(productID)
}
```

#### 2. Permission-Based

```go
type User struct {
    ID          int
    Permissions []string  // ["read", "write", "delete"]
}

func DeletePost(c *gin.Context) {
    user := GetCurrentUser(c)

    if !HasPermission(user, "delete") {
        c.JSON(403, gin.H{"error": "No delete permission"})
        return
    }

    // ✅ Authorized
}
```

#### 3. Ownership-Based

```go
func UpdatePost(c *gin.Context) {
    user := GetCurrentUser(c)
    post := GetPost(postID)

    // Check if user owns this post
    if post.UserID != user.ID {
        c.JSON(403, gin.H{"error": "Not your post"})
        return
    }

    // ✅ Authorized - user owns this resource
}
```

---

## 🔄 The Complete Flow

```
1. User visits app
   ↓
2. AUTHENTICATION: User logs in
   username: ahmed@test.com
   password: ********
   ↓
3. Server verifies credentials
   ✅ Credentials correct
   ↓
4. Server creates session/token
   Token: eyJhbGc...
   User role: "moderator"
   ↓
5. User makes request: DELETE /posts/42
   Headers: Authorization: Bearer eyJhbGc...
   ↓
6. Server verifies token (AUTHENTICATION)
   ✅ Token valid → User is "ahmed"
   ↓
7. Server checks permissions (AUTHORIZATION)
   Action: DELETE post
   User role: "moderator"
   Post owner: "omar"
   ↓
8. Authorization check:
   - Is moderator? ✅
   - Can moderators delete? ✅
   - Is owner? ❌ (but moderator can delete anyone's)
   ↓
9. ✅ AUTHORIZED → Delete post
```

---

## 📊 Comparison Table

| Aspect       | Authentication                        | Authorization                            |
| ------------ | ------------------------------------- | ---------------------------------------- |
| **السؤال**   | <div dir="rtl">من أنت؟</div>          | <div dir="rtl">ماذا يمكنك أن تفعل؟</div> |
| **Process**  | Login, verify credentials             | Check permissions                        |
| **When**     | <div dir="rtl">عند تسجيل الدخول</div> | <div dir="rtl">عند كل action</div>       |
| **Methods**  | Password, JWT, OAuth, 2FA             | RBAC, Permissions, Ownership             |
| **Response** | 401 Unauthorized                      | 403 Forbidden                            |
| **Example**  | Login with password                   | Admin can delete users                   |

---

## 🚨 HTTP Status Codes

### 401 Unauthorized (Should be "Unauthenticated")

```http
GET /api/profile
# No token provided

Response: 401 Unauthorized
{
  "error": "Authentication required. Please login."
}
```

<div dir="rtl">

**المعنى:** لم تثبت هويتك - عرّف عن نفسك!

</div>

### 403 Forbidden

```http
DELETE /api/users/5
Authorization: Bearer token123

Response: 403 Forbidden
{
  "error": "Admin access required"
}
```

<div dir="rtl">

**المعنى:** أعرفك، لكن ليس لديك الصلاحية!

</div>

---

## 💡 Real-World Examples

### Example 1: Blog System

```
Scenario: Delete blog post

1. Authentication:
   - User logged in? ✅
   - Token valid? ✅
   → User is "Ahmed"

2. Authorization:
   - Is Ahmed the post owner? ❌ (owner is "Sara")
   - Is Ahmed admin? ✅
   - Can admins delete any post? ✅
   → ✅ Authorized!

Result: Post deleted
```

### Example 2: Bank Account

```
Scenario: Transfer money

1. Authentication:
   - Password correct? ✅
   - 2FA code correct? ✅
   → Authenticated as account #12345

2. Authorization:
   - Is this your account? ✅
   - Sufficient balance? ✅
   - Daily limit not exceeded? ✅
   → ✅ Authorized!

Result: Transfer completed
```

### Example 3: Google Drive

```
Scenario: Edit document

1. Authentication:
   - Logged into Google? ✅
   → Authenticated as "ahmed@gmail.com"

2. Authorization:
   - Do you have access to this document? ✅
   - Permission level: "Can edit"? ✅
   → ✅ Authorized!

Result: Can edit document
```

---

## 🔧 Implementation in Go

### Authentication Middleware:

```go
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Get token from header
        token := c.GetHeader("Authorization")

        if token == "" {
            c.AbortWithStatusJSON(401, gin.H{
                "error": "Authentication required",
            })
            return
        }

        // Verify token
        claims, err := VerifyJWT(token)
        if err != nil {
            c.AbortWithStatusJSON(401, gin.H{
                "error": "Invalid token",
            })
            return
        }

        // ✅ Authenticated - store user info
        c.Set("user_id", claims.UserID)
        c.Set("user_role", claims.Role)

        c.Next()  // Continue to handler
    }
}
```

### Authorization Middleware:

```go
func RequireRole(role string) gin.HandlerFunc {
    return func(c *gin.Context) {
        // User must be authenticated first
        userRole, exists := c.Get("user_role")
        if !exists {
            c.AbortWithStatusJSON(401, gin.H{
                "error": "Not authenticated",
            })
            return
        }

        // Check authorization
        if userRole != role {
            c.AbortWithStatusJSON(403, gin.H{
                "error": fmt.Sprintf("%s access required", role),
            })
            return
        }

        // ✅ Authorized
        c.Next()
    }
}

// Usage:
router.Use(AuthMiddleware())  // Authentication for all
router.DELETE("/users/:id", RequireRole("admin"), DeleteUser)  // Authorization
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Authentication** = التحقق من الهوية ("من أنت؟")
- ✅ **Authorization** = التحقق من الصلاحيات ("ماذا يمكنك أن تفعل؟")
- ✅ Authentication **أولاً**، ثم Authorization
- ✅ **401** للـ authentication errors
- ✅ **403** للـ authorization errors
- ✅ كل request يحتاج **authentication**
- ✅ بعض actions تحتاج **authorization**
- ✅ دائماً افحص الاثنين!

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 2: Session-Based Authentication](./02-session-auth.md)**

</div>

---

<div align="center">

[📚 Module Home](../README.md)

</div>
