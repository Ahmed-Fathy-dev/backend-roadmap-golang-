# Module 1.3: REST API Fundamentals 🔌

<div dir="rtl">

## نظرة عامة

**REST (REpresentational State Transfer)** هو معمارية (Architecture Style) لتصميم APIs. معظم APIs الحديثة مبنية على مبادئ REST.

</div>

---

## 📖 Content

### 1. What is REST API?

<div dir="rtl">

**REST API** هي طريقة لتصميم APIs تعتمد على:

- استخدام HTTP Methods بشكل صحيح
- التعامل مع Resources (الموارد)
- Stateless Communication

### مثال:

بدلاً من:

- `/getUserById?id=5`
- `/createUser`
- `/updateUser?id=5`

REST API تستخدم:

- `GET /users/5` - قراءة
- `POST /users` - إنشاء
- `PUT /users/5` - تحديث

</div>

---

### 2. REST Principles

<div dir="rtl">

#### 1. Client-Server Separation

- Frontend و Backend منفصلين تماماً
- يتواصلان عبر API فقط

#### 2. Stateless

- كل Request يحتوي على كل المعلومات المطلوبة
- Server لا يحفظ حالة Client

#### 3. Cacheable

- Responses يمكن تخزينها مؤقتاً (Cache)
- لتحسين الأداء

#### 4. Uniform Interface

- طريقة موحدة للتعامل مع all Resources
- استخدام HTTP Methods بشكل صحيح

#### 5. Layered System

- يمكن وجود طبقات وسيطة (Load Balancers, Caches)

</div>

---

### 3. Resources & Endpoints

<div dir="rtl">

#### Resource هو أي شيء في نظامك:

- Users
- Products
- Orders
- Posts
- Comments

#### Endpoint هو المسار للوصول للـ Resource:

</div>

```
/users          ← Collection (مجموعة)
/users/5        ← Specific Resource (مورد معين)
/users/5/posts  ← Nested Resource (مورد متداخل)
```

---

### 4. RESTful Routing

<div dir="rtl">

جدول كامل لتصميم REST API:

</div>

| HTTP Method | Endpoint   | Action         | Description                                |
| ----------- | ---------- | -------------- | ------------------------------------------ |
| **GET**     | `/users`   | Index          | <div dir="rtl">قراءة جميع المستخدمين</div> |
| **GET**     | `/users/5` | Show           | <div dir="rtl">قراءة مستخدم محدد</div>     |
| **POST**    | `/users`   | Create         | <div dir="rtl">إنشاء مستخدم جديد</div>     |
| **PUT**     | `/users/5` | Update         | <div dir="rtl">تحديث مستخدم (كامل)</div>   |
| **PATCH**   | `/users/5` | Partial Update | <div dir="rtl">تحديث جزئي</div>            |
| **DELETE**  | `/users/5` | Destroy        | <div dir="rtl">حذف مستخدم</div>            |

---

### 5. Resource Naming Best Practices

<div dir="rtl">

#### ✅ DO (افعل):

</div>

```
✅ /users                    # جمع (Plural)
✅ /users/5                  # ID رقمي
✅ /users/5/posts            # علاقة واضحة
✅ /products?category=laptops # Query parameters للفلترة
```

<div dir="rtl">

#### ❌ DON'T (لا تفعل):

</div>

```
❌ /getUsers                 # لا تستخدم افعال في URL
❌ /user                     # استخدم جمع، ليس مفرد
❌ /users/five               # استخدم ID رقمي
❌ /USERS                    # استخدم lowercase
❌ /users_posts              # استخدم /users/5/posts
```

<div dir="rtl">

### القواعد الذهبية:

1. **استخدم الجمع (Plural):** `/users` ليس `/user`
2. **استخدم Nouns ليس Verbs:** `/users` ليس `/getUsers`
3. **استخدم Hierarchy للعلاقات:** `/users/5/posts`
4. **استخدم lowercase و hyphens:** `/blog-posts`
5. **لا تنتهي بـ `/`:** `/users` ليس `/users/`

</div>

---

### 6. HTTP Status Codes in REST

<div dir="rtl">

اختيار Status Code الصحيح مهم جداً:

</div>

| Operation                | Success Code   | Description                               |
| ------------------------ | -------------- | ----------------------------------------- |
| **GET** `/users`         | 200 OK         | <div dir="rtl">قائمة المستخدمين</div>     |
| **GET** `/users/5`       | 200 OK         | <div dir="rtl">بيانات المستخدم</div>      |
| **GET** `/users/999`     | 404 Not Found  | <div dir="rtl">مستخدم غير موجود</div>     |
| **POST** `/users`        | 201 Created    | <div dir="rtl">تم إنشاء مستخدم جديد</div> |
| **PUT/PATCH** `/users/5` | 200 OK         | <div dir="rtl">تم التحديث بنجاح</div>     |
| **DELETE** `/users/5`    | 204 No Content | <div dir="rtl">تم الحذف بنجاح</div>       |

---

### 7. Request & Response Examples

<div dir="rtl">

#### مثال 1: GET - قراءة قائمة

</div>

**Request:**

```http
GET /api/users HTTP/1.1
Host: myapi.com
```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    {
      "id": 1,
      "name": "Ahmed",
      "email": "ahmed@example.com"
    },
    {
      "id": 2,
      "name": "Sara",
      "email": "sara@example.com"
    }
  ],
  "total": 2
}
```

---

<div dir="rtl">

#### مثال 2: POST - إنشاء

</div>

**Request:**

```http
POST /api/users HTTP/1.1
Host: myapi.com
Content-Type: application/json

{
  "name": "Omar",
  "email": "omar@example.com",
  "password": "securepass123"
}
```

**Response:**

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/3

{
  "id": 3,
  "name": "Omar",
  "email": "omar@example.com",
  "created_at": "2024-12-10T05:30:00Z"
}
```

---

<div dir="rtl">

#### مثال 3: PUT - تحديث كامل

</div>

**Request:**

```http
PUT /api/users/3 HTTP/1.1
Host: myapi.com
Content-Type: application/json

{
  "name": "Omar Ali",
  "email": "omar.ali@example.com",
  "bio": "Software Engineer"
}
```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 3,
  "name": "Omar Ali",
  "email": "omar.ali@example.com",
  "bio": "Software Engineer",
  "updated_at": "2024-12-10T06:00:00Z"
}
```

---

<div dir="rtl">

#### مثال 4: DELETE - حذف

</div>

**Request:**

```http
DELETE /api/users/3 HTTP/1.1
Host: myapi.com
```

**Response:**

```http
HTTP/1.1 204 No Content
```

---

### 8. Query Parameters for Filtering

<div dir="rtl">

استخدم Query Parameters لـ:

</div>

#### Filtering (فلترة)

```
GET /products?category=laptops
GET /products?price_min=500&price_max=1000
GET /users?role=admin
```

#### Sorting (ترتيب)

```
GET /products?sort=price          # تصاعدي
GET /products?sort=-price         # تنازلي
GET /products?sort=name,price
```

#### Pagination (تقسيم الصفحات)

```
GET /products?page=2&limit=20
GET /products?offset=40&limit=20
```

#### Searching (بحث)

```
GET /products?search=laptop
GET /users?q=ahmed
```

#### Complete Example:

```
GET /products?category=laptops&sort=-price&page=1&limit=10
```

<div dir="rtl">

معناها: أعطني laptops، مرتبة حسب السعر تنازلياً، صفحة 1، 10 منتجات

</div>

---

### 9. Nested Resources

<div dir="rtl">

للعلاقات بين Resources:

</div>

```
GET  /users/5/posts           # جميع posts للمستخدم 5
POST /users/5/posts           # إنشاء post جديد للمستخدم 5
GET  /users/5/posts/12        # post محدد
PUT  /users/5/posts/12        # تحديث post
DELETE /users/5/posts/12      # حذف post
```

<div dir="rtl">

**ملحوظة:** لا تتعمق أكثر من مستويين:

- ✅ `/users/5/posts/12`
- ❌ `/users/5/posts/12/comments/7` (معقد جداً)
- ✅ بدلاً منه: `/comments/7` أو `/posts/12/comments/7`

</div>

---

### 10. API Versioning

<div dir="rtl">

عندما تحتاج لتغيير API بشكل كبير، استخدم Versioning:

</div>

#### Method 1: في URL (الأكثر شيوعاً)

```
/api/v1/users
/api/v2/users
```

#### Method 2: في Headers

```http
GET /api/users HTTP/1.1
Accept: application/vnd.myapi.v2+json
```

#### Method 3: Query Parameter

```
/api/users?version=2
```

<div dir="rtl">

**الطريقة الأولى (في URL) هي الأكثر وضوحاً واستخداماً.**

</div>

---

### 11. Error Responses

<div dir="rtl">

يجب أن تكون رسائل الأخطاء واضحة ومفيدة:

</div>

#### Bad Example ❌

```json
{
  "error": "Bad Request"
}
```

#### Good Example ✅

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request data is invalid",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters"
      }
    ]
  },
  "timestamp": "2024-12-10T06:00:00Z",
  "path": "/api/users"
}
```

---

### 12. REST API Best Practices

<div dir="rtl">

#### ✅ DO (افعل):

1. استخدم JSON كـ default format
2. استخدم HTTPS دائماً
3. استخدم plural nouns للـ resources
4. استخدم HTTP Methods الصحيحة
5. أضف API versioning من البداية
6. وفر pagination للبيانات الكبيرة
7. استخدم status codes صحيحة
8. وفر رسائل أخطاء واضحة
9. استخدم authentication للـ Protected Routes
10. وثّق API باستخدام Swagger/OpenAPI

#### ❌ DON'T (لا تفعل):

1. لا تستخدم verbs في URLs
2. لا تضع sensitive data في URLs
3. لا تتجاهل error handling
4. لا ترجع بيانات أكثر من المطلوب
5. لا تستخدم GET للعمليات التي تغير البيانات

</div>

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ REST هي معمارية لتصميم APIs باستخدام HTTP
- ✅ استخدم Resources (plural nouns) وليس Actions
- ✅ استخدم HTTP Methods بشكل صحيح: GET, POST, PUT, PATCH, DELETE
- ✅ استخدم Status Codes المناسبة
- ✅ استخدم Query Parameters للـ filtering, sorting, pagination
- ✅ وفر error messages واضحة ومفيدة
- ✅ استخدم versioning للـ backward compatibility

</div>

---

## 🎯 Practice

<div dir="rtl">

صمم REST API للمواضيع التالية:

1. مكتبة (Books, Authors, Categories)
2. متجر إلكتروني (Products, Orders, Customers)
3. مدونة (Posts, Comments, Users)

لكل واحدة، اكتب:

- Endpoints
- HTTP Methods
- Expected Status Codes

</div>

---

## 📚 Additional Resources

- 📖 [REST API Tutorial](https://restfulapi.net/)
- 📖 [Best Practices for REST API Design](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)

---

## ⏭️ Next Module

<div dir="rtl">

الآن بعد أن تعلمت تصميم REST APIs، دعنا نتعلم كيف نؤمنها:

**➡️ [Module 1.4: Authentication & Authorization](../04-auth-basics/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: HTTP Protocol](../02-http-protocol/README.md) | [🏠 Track 1 Home](../README.md) | [📚 Main](../../README.md)

</div>
