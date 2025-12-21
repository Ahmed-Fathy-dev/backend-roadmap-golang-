# Lesson 7: RBAC - Role-Based Access Control 👥

<div dir="rtl">

## المقدمة

**RBAC** = نظام لإدارة الصلاحيات بناءً على **الأدوار**.

</div>

---

## 🎯 Core Concepts

```
User → Role → Permissions

Ahmed → Admin → [create, read, update, delete]
Sara  → Editor → [create, read, update]
Omar  → Viewer → [read]
```

---

## 🔧 Implementation

```go
// Define roles
const (
    RoleAdmin  = "admin"
    RoleEditor = "editor"
    RoleViewer = "viewer"
)

type User struct {
    ID    uint
    Email string
    Role  string  // admin, editor, viewer
}

// Middleware to require specific role
func RequireRole(requiredRole string) gin.HandlerFunc {
    return func(c *gin.Context) {
        userRole, _ := c.Get("user_role")

        if userRole != requiredRole {
            c.AbortWithStatusJSON(403, gin.H{
                "error": fmt.Sprintf("%s access required", requiredRole),
            })
            return
        }

        c.Next()
    }
}

// Usage
router.DELETE("/users/:id", AuthMiddleware(), RequireRole(RoleAdmin), DeleteUser)
```

---

## 📋 Permission-Based Approach

```go
type User struct {
    ID          uint
    Email       string
    Permissions []string  // ["users:read", "users:write", "posts:delete"]
}

func HasPermission(user *User, permission string) bool {
    for _, p := range user.Permissions {
        if p == permission {
            return true
        }
    }
    return false
}

// Middleware
func RequirePermission(perm string) gin.HandlerFunc {
    return func(c *gin.Context) {
        user := GetCurrentUser(c)

        if !HasPermission(user, perm) {
            c.AbortWithStatusJSON(403, gin.H{
                "error": "Insufficient permissions",
            })
            return
        }

        c.Next()
    }
}

// Usage
router.DELETE("/posts/:id", RequirePermission("posts:delete"), DeletePost)
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Roles:** Admin, Editor, Viewer
- ✅ **Permissions:** create, read, update, delete
- ✅ **Middleware** للتحقق من الصلاحيات
- ✅ **403 Forbidden** للـ insufficient permissions

</div>

---

<div align="center">

[⬅️ Previous: OAuth2](./06-oauth2.md) | [➡️ Next: Security Threats](./08-security-threats.md) | [📚 Module Home](../README.md)

</div>
