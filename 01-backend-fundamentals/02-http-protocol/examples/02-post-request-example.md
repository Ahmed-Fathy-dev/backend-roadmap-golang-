# Example 2: POST Request 📝

<div dir="rtl">

## نظرة عامة

مثال عملي متكامل على **POST Request** - المستخدم لإنشاء بيانات جديدة.

</div>

---

## 🎯 السيناريو

<div dir="rtl">

**المطلوب:** إنشاء مستخدم جديد في النظام

**URL:** `https://api.example.com/api/users`

</div>

---

## 📤 The Request

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 156
Authorization: Bearer admin_token_xyz123
User-Agent: PostmanRuntime/7.32.0
Accept: application/json

{
  "name": "Ahmed Mohamed",
  "email": "ahmed.mohamed@example.com",
  "password": "SecurePass123!",
  "age": 28,
  "city": "Cairo",
  "role": "user"
}
```

---

## 🔍 تحليل Request

<div dir="rtl">

### Request Line

</div>

```http
POST /api/users HTTP/1.1
  │      │          │
  │      │          └─ HTTP Version
  │      └──────────── Path (collection endpoint)
  └─────────────────── POST = Create new resource
```

<div dir="rtl">

### لماذا POST وليس GET؟

- ✅ نريد **إنشاء** مورد جديد (User)
- ✅ نرسل **بيانات حساسة** (Password)
- ✅ العملية **تغير** حالة السيرفر

---

### Headers التحليل

**`Host: api.example.com`**

- Domain name

**`Content-Type: application/json`**

- ⭐ **مهم جداً!**
- يخبر السيرفر: البيانات في الـ Body هي JSON
- بدونه، السيرفر لن يفهم الـ Body

**`Content-Length: 156`**

- حجم الـ Body = 156 bytes
- السيرفر يعرف كم بيانات يستقبل

**`Authorization: Bearer admin_token_xyz123`**

- JWT Token للمصادقة
- فقط Admins يقدروا يعملوا users جدد مثلاً

**`Accept: application/json`**

- نطلب الرد يكون JSON

---

### Body التحليل

</div>

```json
{
  "name": "Ahmed Mohamed", // User's full name
  "email": "ahmed.mohamed@...", // Unique email (will be validated)
  "password": "SecurePass123!", // Plain text (will be hashed by server)
  "age": 28, // Integer
  "city": "Cairo", // Optional field
  "role": "user" // Default role
}
```

---

## 📥 Successful Response (201 Created)

```http
HTTP/1.1 201 Created
Date: Tue, 10 Dec 2024 14:20:00 GMT
Server: Express/4.18.0
Content-Type: application/json; charset=utf-8
Content-Length: 245
Location: /api/users/42
ETag: "15-Ymfja9c83zJCpzKY1zZz8w"

{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": 42,
    "name": "Ahmed Mohamed",
    "email": "ahmed.mohamed@example.com",
    "age": 28,
    "city": "Cairo",
    "role": "user",
    "created_at": "2024-12-10T14:20:00.000Z",
    "updated_at": "2024-12-10T14:20:00.000Z"
  }
}
```

---

## 🔍 تحليل Response

<div dir="rtl">

### Status Code: 201 Created

**لماذا 201 وليس 200؟**

- `200 OK` = عملية نجحت
- `201 Created` = عملية نجحت **وتم إنشاء** مورد جديد
- 201 أكثر دقة ووضوح!

---

### Response Headers المهمة

**`Location: /api/users/42`**

- ⭐ **مهم جداً في POST!**
- يخبر الـ Client بـ URL الـ resource الجديد
- الآن يمكن الوصول للمستخدم عبر: `GET /api/users/42`

**`Content-Type: application/json`**

- الرد JSON format

---

### Response Body

لاحظ الفرق بين Request و Response:

- ✅ **تم إضافة:** `id` (auto-generated)
- ✅ **تم إضافة:** `created_at` و `updated_at`
- ❌ **تم حذف:** `password` (لأمان!)
  - Password موجود في Database مشفر
  - لكن لا نرجعه أبداً في Response

</div>

---

## ⚠️ Error Responses

### السيناريو 1: Email مستخدم مسبقاً

**Request:**

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Sara Ali",
  "email": "ahmed.mohamed@example.com",   ← نفس Email أحمد!
  "password": "pass123"
}
```

**Response:**

```http
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "success": false,
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "A user with this email already exists",
    "field": "email",
    "value": "ahmed.mohamed@example.com"
  }
}
```

<div dir="rtl">

**`409 Conflict`** = تعارض مع بيانات موجودة

</div>

---

### السيناريو 2: Validation Errors

**Request:**

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "A",                       ← أقصر من 3 أحرف
  "email": "invalid-email",          ← ليس email صحيح
  "password": "123",                 ← أقل من 8 أحرف
  "age": 15                          ← أقل من 18
}
```

**Response:**

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The provided data is invalid",
    "details": [
      {
        "field": "name",
        "message": "Name must be at least 3 characters long",
        "value": "A"
      },
      {
        "field": "email",
        "message": "Invalid email format",
        "value": "invalid-email"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters long",
        "value": "123"
      },
      {
        "field": "age",
        "message": "User must be at least 18 years old",
        "value": 15
      }
    ]
  }
}
```

<div dir="rtl">

**`422 Unprocessable Entity`** = البيانات مفهومة لكن غير صالحة

</div>

---

### السيناريو 3: Missing Required Fields

**Request:**

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Omar Khaled"
  // Missing: email, password
}
```

**Response:**

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "success": false,
  "error": {
    "code": "MISSING_REQUIRED_FIELDS",
    "message": "Required fields are missing",
    "missing_fields": ["email", "password"]
  }
}
```

<div dir="rtl">

**`400 Bad Request`** = الطلب نفسه غير صحيح

</div>

---

### السيناريو 4: Unauthorized

**Request:**

```http
POST /api/users HTTP/1.1
Content-Type: application/json
// Missing: Authorization header

{
  "name": "Hacker",
  "email": "hacker@example.com",
  "password": "pass123"
}
```

**Response:**

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json
WWW-Authenticate: Bearer

{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required. Please provide a valid token."
  }
}
```

<div dir="rtl">

**`401 Unauthorized`** = غير مسجل دخول / Token غير موجود

</div>

---

## 🔐 POST with File Upload

<div dir="rtl">

### السيناريو: إنشاء منتج مع صورة

</div>

```http
POST /api/products HTTP/1.1
Host: shop.example.com
Content-Type: multipart/form-data; boundary=----WebKitBoundary123
Authorization: Bearer token_xyz

------WebKitBoundary123
Content-Disposition: form-data; name="name"

Gaming Laptop
------WebKitBoundary123
Content-Disposition: form-data; name="price"

15000
------WebKitBoundary123
Content-Disposition: form-data; name="category"

laptops
------WebKitBoundary123
Content-Disposition: form-data; name="image"; filename="laptop.jpg"
Content-Type: image/jpeg

[...binary image data...]
------WebKitBoundary123--
```

<div dir="rtl">

**لاحظ:**

- `Content-Type: multipart/form-data`
- كل field له section منفصل
- الصورة binary data

</div>

---

## 💡 POST Best Practices

<div dir="rtl">

### ✅ DO:

1. استخدم **201 Created** للنجاح (ليس 200)
2. أضف **Location header** بـ URL الـ resource الجديد
3. أرجع الـ **resource الكامل** في Response
4. عمل **Validation** شاملة
5. أرجع **error messages واضحة** مع Details
6. **لا ترجع sensitive data** (passwords, tokens)
7. Hash passwords قبل الحفظ

### ❌ DON'T:

1. لا تستخدم GET لإنشاء بيانات
2. لا تنسى Content-Type header
3. لا ترجع passwords في Response
4. لا تستخدم 200 OK، استخدم 201 Created

</div>

---

## 🎯 Try It Yourself!

<div dir="rtl">

استخدم Postman أو Thunder Client:

1. جرب POST request لـ API test مثل:

   - `https://jsonplaceholder.typicode.com/users`
   - `https://reqres.in/api/users`

2. جرب إرسال بيانات خاطئة وشاهد Errors

3. لاحظ الفرق بين Status Codes

</div>

---

<div align="center">

[⬅️ Previous: GET Example](./01-get-request-example.md) | [📚 Back to Examples](../README.md#-examples-أمثلة-عملية)

</div>
