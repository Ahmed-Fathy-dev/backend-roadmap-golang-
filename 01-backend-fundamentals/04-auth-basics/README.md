# Module 1.4: Authentication & Authorization Basics 🔐

<div dir="rtl">

## نظرة عامة

**Authentication** و **Authorization** هما أساس الأمان في أي Backend Application. في هذا الدرس سنتعلم الفرق بينهما والطرق المختلفة لتطبيقهما.

</div>

---

## 📖 Content

### 1. Authentication vs Authorization

<div dir="rtl">

| Authentication       | Authorization           |
| -------------------- | ----------------------- |
| **من أنت؟**          | **ماذا يمكنك أن تفعل؟** |
| التحقق من الهوية     | التحقق من الصلاحيات     |
| Login/Register       | Permissions/Roles       |
| Username + Password  | User Role (Admin, User) |
| **401 Unauthorized** | **403 Forbidden**       |

### مثال من الحياة:

- **Authentication:** بطاقة الهوية عند دخول المبنى (إثبات من أنت)
- **Authorization:** بطاقة الدخول للطوابق (ما الذي يمكنك الوصول إليه)

### مثال تقني:

- **Authentication:** تسجيل دخول بـ email + password
- **Authorization:** هل يمكنك حذف هذا Post؟ (إذا كنت صاحبه أو admin)

</div>

---

### 2. Authentication Methods

<div dir="rtl">

#### 2.1 Session-Based Authentication

الطريقة التقليدية:

</div>

```
┌────────┐                          ┌────────┐
│ Client │                          │ Server │
└────────┘                          └────────┘
     │                                   │
     │   POST /login                     │
     │   {email, password}               │
     ├──────────────────────────────────>│
     │                                   │
     │                              ┌────┴────┐
     │                              │ Verify  │
     │                              │ Create  │
     │                              │ Session │
     │                              └────┬────┘
     │                                   │
     │   200 OK                          │
     │   Set-Cookie: session_id=abc123   │
     │<──────────────────────────────────┤
     │                                   │
     │   GET /profile                    │
     │   Cookie: session_id=abc123       │
     ├──────────────────────────────────>│
     │                                   │
     │                              ┌────┴────┐
     │                              │ Check   │
     │                              │ Session │
     │                              └────┬────┘
     │                                   │
     │   200 OK {user data}              │
     │<──────────────────────────────────┤
```

<div dir="rtl">

**كيف تعمل:**

1. User يرسل email + password
2. Server يتحقق من البيانات
3. Server ينشئ Session ويحفظها في memory أو database
4. Server يرسل Session ID للـ Client في Cookie
5. Client يرسل Cookie مع كل Request
6. Server يتحقق من Session ID

**المميزات:**

- ✅ سهلة الفهم
- ✅ يمكن إلغاء Sessions من Server

**العيوب:**

- ❌ تحتاج تخزين Sessions في Server (صعب مع Scaling)
- ❌ لا تعمل جيداً مع Mobile Apps
- ❌ CORS issues

</div>

---

<div dir="rtl">

#### 2.2 Token-Based Authentication

الطريقة الحديثة (الأكثر استخداماً):

</div>

```
┌────────┐                          ┌────────┐
│ Client │                          │ Server │
└────────┘                          └────────┘
     │                                   │
     │   POST /login                     │
     │   {email, password}               │
     ├──────────────────────────────────>│
     │                                   │
     │                              ┌────┴────┐
     │                              │ Verify  │
     │                              │ Create  │
     │                              │ JWT     │
     │                              └────┬────┘
     │                                   │
     │   200 OK                          │
     │   {token: "eyJhbGc..."}           │
     │<──────────────────────────────────┤
     │                                   │
── Client saves token in localStorage ──
     │                                   │
     │   GET /profile                    │
     │   Authorization: Bearer eyJhbGc...│
     ├──────────────────────────────────>│
     │                                   │
     │                              ┌────┴────┐
     │                              │ Verify  │
     │                              │ JWT     │
     │                              └────┬────┘
     │                                   │
     │   200 OK {user data}              │
     │<──────────────────────────────────┤
```

<div dir="rtl">

**كيف تعمل:**

1. User يرسل email + password
2. Server يتحقق من البيانات
3. Server ينشئ **JWT Token** يحتوي على معلومات User
4. Client يحفظ Token (في localStorage أو memory)
5. Client يرسل Token في Authorization Header مع كل Request
6. Server يتحقق من صحة Token

**المميزات:**

- ✅ Stateless (Server لا يحفظ شيء)
- ✅ يعمل مع Mobile, Web, Desktop
- ✅ سهل Scaling
- ✅ لا CORS issues

**العيوب:**

- ❌ صعب إلغاء Token قبل انتهاء صلاحيته
- ❌ إذا سُرق Token، يمكن استخدامه حتى ينتهي

</div>

---

### 3. JWT (JSON Web Token)

<div dir="rtl">

JWT هو Token مكون من 3 أجزاء مفصولة بـ `.`:

</div>

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFobWVkIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
     └──────── Header ─────────┘  └───────────────────── Payload ─────────────────────┘  └──────────── Signature ────────────┘
```

<div dir="rtl">

#### 3.1 Header

معلومات عن نوع Token والخوارزمية:

</div>

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

<div dir="rtl">

#### 3.2 Payload

المعلومات (Claims):

</div>

```json
{
  "sub": "5", // User ID
  "name": "Ahmed", // User name
  "email": "ahmed@example.com",
  "role": "user",
  "iat": 1516239022, // Issued at (timestamp)
  "exp": 1516242622 // Expiration (timestamp)
}
```

<div dir="rtl">

#### 3.3 Signature

لضمان أن Token لم يتم تعديله:

</div>

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

<div dir="rtl">

**ملاحظات مهمة:**

- ⚠️ Payload مشفرة بـ Base64 فقط (ليست encrypted)
- ⚠️ لا تضع sensitive data (passwords, credit cards) في JWT
- ✅ ضع فقط معلومات user البسيطة (id, name, role)

</div>

---

### 4. Token Security Best Practices

<div dir="rtl">

#### 4.1 Token Expiration

استخدم expiration قصيرة:

- **Access Token:** 15 دقيقة - ساعة
- **Refresh Token:** أسبوع - شهر

#### 4.2 Refresh Tokens

</div>

```
Access Token (15 min) ───┐
                          ├──▶ للطلبات العادية
Refresh Token (7 days) ──┘    للحصول على access token جديد
```

<div dir="rtl">

**الفكرة:**

1. Client يستخدم Access Token للطلبات
2. عند انتهاء Access Token، يستخدم Refresh Token للحصول على واحد جديد
3. إذا سُرق Access Token، سينتهي بعد 15 دقيقة
4. Refresh Token محفوظ بشكل آمن ونادراً ما يُرسل

#### 4.3 Storage

- ✅ **Best:** HttpOnly Cookies (للـ Web)
- ✅ **Good:** Secure Storage (للـ Mobile)
- ⚠️ **Acceptable:** localStorage (مع الحذر)
- ❌ **Never:** في URL أو sessionStorage للبيانات الحساسة

#### 4.4 HTTPS Only

- ✅ استخدم HTTPS دائماً في Production
- ❌ لا ترسل Tokens عبر HTTP

</div>

---

### 5. Authorization & Roles

<div dir="rtl">

بعد Authentication، نحتاج التحقق من **Permissions**.

#### 5.1 Role-Based Access Control (RBAC)

</div>

```
User ──▶ Role ──▶ Permissions

مثال:
Ahmed ──▶ Admin ──▶ [create, read, update, delete]
Sara  ──▶ User  ──▶ [read]
```

<div dir="rtl">

**الأدوار الشائعة:**

- **Admin:** كل الصلاحيات
- **Moderator:** تعديل وحذف المحتوى
- **User:** قراءة وإنشاء محتوى خاص

#### 5.2 Middleware Protection

في Backend، نستخدم Middleware للتحقق:

</div>

```go
// Pseudocode example
func RequireAuth(next Handler) Handler {
    return func(request, response) {
        // Extract token from header
        token := request.Header.Get("Authorization")

        // Verify token
        if !isValidToken(token) {
            return 401 Unauthorized
        }

        // Token valid, proceed
        next(request, response)
    }
}

func RequireRole(role string) Middleware {
    return func(next Handler) Handler {
        return func(request, response) {
            user := request.User // من RequireAuth

            if user.Role != role {
                return 403 Forbidden
            }

            next(request, response)
        }
    }
}
```

<div dir="rtl">

**الاستخدام:**

</div>

```go
// Public route - no auth required
GET  /products          ──▶ getAllProducts

// Protected route - auth required
GET  /profile           ──▶ RequireAuth ──▶ getProfile

// Admin only route
POST /products          ──▶ RequireAuth ──▶ RequireRole("admin") ──▶ createProduct
```

---

### 6. OAuth 2.0 Overview

<div dir="rtl">

OAuth 2.0 للـ "Login with Google/Facebook/GitHub"

</div>

```
┌────────┐           ┌─────────┐           ┌──────────┐
│  User  │           │  Your   │           │  Google  │
│        │           │  App    │           │  (OAuth) │
└────────┘           └─────────┘           └──────────┘
     │                    │                      │
     │  1. Click "Login   │                      │
     │   with Google"     │                      │
     ├───────────────────>│                      │
     │                    │                      │
     │  2. Redirect to    │                      │
     │   Google login     │                      │
     │<───────────────────┤                      │
     │                    │                      │
     │  3. Login with     │                      │
     │   Google account   │                      │
     ├────────────────────┼─────────────────────>│
     │                    │                      │
     │  4. Google asks:   │                      │
     │   "Allow App XYZ?" │                      │
     │<───────────────────┼──────────────────────┤
     │                    │                      │
     │  5. User approves  │                      │
     ├────────────────────┼─────────────────────>│
     │                    │                      │
     │  6. Redirect back  │                      │
     │   with code        │                      │
     │<───────────────────┼──────────────────────┤
     │                    │                      │
     │  7. Pass code      │                      │
     │   to your app      │                      │
     ├───────────────────>│                      │
     │                    │  8. Exchange code    │
     │                    │   for access token   │
     │                    ├─────────────────────>│
     │                    │                      │
     │                    │  9. Return token     │
     │                    │<─────────────────────┤
     │                    │                      │
     │  10. Create user   │                      │
     │   session/JWT      │                      │
     │<───────────────────┤                      │
```

<div dir="rtl">

**الفوائد:**

- ✅ No password management (Google يدير كلمات السر)
- ✅ Trusted authentication
- ✅ Better user experience

**متى تستخدمه:**

- للتطبيقات العامة (Social, E-commerce)
- عندما تريد تسهيل التسجيل

</div>

---

### 7. Password Security

<div dir="rtl">

#### ❌ NEVER Store Passwords in Plain Text!

</div>

```
❌ Database:
users
├── id: 1
├── name: "Ahmed"
└── password: "mypassword123"    ← خطر جداً!
```

<div dir="rtl">

#### ✅ Always Hash Passwords

</div>

```
✅ Database:
users
├── id: 1
├── name: "Ahmed"
└── password: "$2a$10$N9qo8uL..."     ← Hashed (bcrypt)
```

<div dir="rtl">

**استخدم bcrypt:**

- ✅ Slow by design (يصعّب Brute Force Attacks)
- ✅ يضيف Salt تلقائياً
- ✅ Industry standard

**عند التسجيل:**

</div>

```
1. User sends: password = "mypassword123"
2. Server hashes: "$2a$10$N9qo8uL..."
3. Server stores hash في Database
```

<div dir="rtl">

**عند Login:**

</div>

```
1. User sends: password = "mypassword123"
2. Server gets hash from Database
3. Server compares: bcrypt.Compare(hash, password)
4. If match ✅ → Login successful
```

---

### 8. Common Security Threats

<div dir="rtl">

#### 8.1 Brute Force Attack

محاولة تجريب كلمات سر كثيرة

**الحماية:**

- Rate Limiting (حد أقصى للمحاولات)
- CAPTCHA بعد محاولات فاشلة
- Account lockout

#### 8.2 Man-in-the-Middle (MITM)

اعتراض البيانات بين Client و Server

**الحماية:**

- ✅ HTTPS دائماً
- ✅ HSTS Headers

#### 8.3 XSS (Cross-Site Scripting)

حقن JavaScript ضار

**الحماية:**

- Sanitize user inputs
- Use HttpOnly cookies
- Content Security Policy (CSP)

#### 8.4 CSRF (Cross-Site Request Forgery)

إرسال طلبات نيابة عن المستخدم

**الحماية:**

- CSRF Tokens
- SameSite Cookie attribute

</div>

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Authentication:** من أنت؟ (401 Unauthorized)
- ✅ **Authorization:** ماذا يمكنك أن تفعل؟ (403 Forbidden)
- ✅ **Session-based:** تقليدية، تحفظ state في Server
- ✅ **Token-based (JWT):** حديثة، stateless، الأكثر استخداماً
- ✅ JWT مكون من: Header.Payload.Signature
- ✅ استخدم short-lived access tokens + refresh tokens
- ✅ **NEVER** store passwords in plain text - استخدم bcrypt
- ✅ استخدم HTTPS دائماً في Production
- ✅ طبق RBAC للـ Authorization (Admin, User, etc.)

</div>

---

## 🎯 Practice Questions

<div dir="rtl">

1. ما الفرق بين 401 و 403؟
2. ما الفرق بين Session-based و Token-based authentication؟
3. ما هي أجزاء JWT الثلاثة؟
4. لماذا نستخدم Refresh Tokens؟
5. لماذا لا نخزن Passwords كـ plain text؟

</div>

---

## ⏭️ Next Module

<div dir="rtl">

الآن بعد أن فهمت الأمان، دعنا نتعلم أساسيات قواعد البيانات:

**➡️ [Module 1.5: Database Concepts](../05-database-concepts/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: REST API](../03-rest-api/README.md) | [🏠 Track 1 Home](../README.md) | [📚 Main](../../README.md)

</div>
