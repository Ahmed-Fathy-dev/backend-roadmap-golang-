# Lesson 1: What is REST? 🔌

<div dir="rtl">

## المقدمة

**REST** هو المعيار الأكثر استخداماً لبناء Web APIs في العالم!

في هذا الدرس، سنفهم ما هو REST، ولماذا مهم جداً، وكيف يعمل.

</div>

---

## 🎯 What is REST?

**REST** = **RE**presentational **S**tate **T**ransfer

<div dir="rtl">

### التعريف البسيط:

REST هو **أسلوب** (architectural style) لبناء Web Services يستخدم HTTP بشكل صحيح.

### التعريف التقني:

معمارية (architecture) لتصميم networked applications تعتمد على:

- HTTP Methods (GET, POST, PUT, DELETE)
- Resources معرّفة بـ URLs
- Stateless communication
- Uniform interface

</div>

---

## 📖 A Brief History

```
2000: Roy Fielding يقدم REST في رسالة الدكتوراه
      ↓
2005-2010: REST يبدأ يحل محل SOAP
      ↓
2015+: REST = المعيار الصناعي للـ Web APIs
      ↓
Today: معظم APIs في العالم RESTful
```

<div dir="rtl">

**من يستخدم REST؟**

- Twitter API
- Facebook Graph API
- GitHub API
- Google APIs
- Amazon AWS
- تقريباً كل Web API حديث!

</div>

---

## 🔑 Core Concepts

### 1. Resources (الموارد)

<div dir="rtl">

**Resource** = أي شيء يمكن تسميته والوصول إليه.

**أمثلة:**

- User (مستخدم)
- Product (منتج)
- Order (طلب)
- Post (منشور)
- Comment (تعليق)

</div>

```
Resource في REST يشبه "Object" في OOP:
- له معرّف فريد (ID)
- له خصائص (properties)
- يمكن التعامل معه (CRUD operations)
```

### 2. URLs Identify Resources

<div dir="rtl">

كل Resource له URL فريد:

</div>

```
https://api.shop.com/products/42
                      │         │
                      │         └─ Resource ID
                      └─────────── Resource Type
```

**Examples:**

```
GET /api/users           # Collection of users
GET /api/users/5         # Specific user (ID = 5)
GET /api/products/123    # Product 123
GET /api/orders/99       # Order 99
```

### 3. HTTP Methods = Actions

<div dir="rtl">

بدلاً من URLs مثل:

</div>

```
❌ /getUser?id=5
❌ /createProduct
❌ /deleteOrder?id=99
```

<div dir="rtl">

REST يستخدم HTTP Methods:

</div>

```
✅ GET    /users/5        # Read user
✅ POST   /products       # Create product
✅ DELETE /orders/99      # Delete order
```

### 4. Representations

<div dir="rtl">

Resource يمكن تمثيله بأشكال مختلفة:

</div>

```json
// JSON (الأكثر شيوعاً)
{
  "id": 42,
  "name": "Ahmed",
  "email": "ahmed@test.com"
}
```

```xml
<!-- XML -->
<user>
  <id>42</id>
  <name>Ahmed</name>
  <email>ahmed@test.com</email>
</user>
```

---

## 🆚 REST vs SOAP vs GraphQL

### REST API:

```http
GET /api/users/5
Response: { "id": 5, "name": "Ahmed", "email": "..." }
```

<div dir="rtl">

**المميزات:**

- ✅ Simple & easy
- ✅ Uses standard HTTP
- ✅ Cacheable
- ✅ Stateless

**العيوب:**

- ❌ Over-fetching (يجيب بيانات زيادة)
- ❌ Under-fetching (أحياناً تحتاج requests متعددة)
- ❌ Versioning challenges

</div>

### SOAP (قديم):

```xml
<soap:Envelope>
  <soap:Body>
    <GetUser>
      <UserID>5</UserID>
    </GetUser>
  </soap:Body>
</soap:Envelope>
```

<div dir="rtl">

- ✅ Enterprise features
- ❌ Complex & verbose
- ❌ Harder to use
- 📉 Less popular now

</div>

### GraphQL (حديث):

```graphql
{
  user(id: 5) {
    name
    email
  }
}
```

<div dir="rtl">

- ✅ Get exactly what you need
- ✅ Single endpoint
- ❌ More complex setup
- ⚠️ Different mindset

</div>

---

## 🎨 REST Example: E-commerce API

### Resources:

```
Products
Users
Orders
Categories
Reviews
```

### Endpoints:

```
# Products
GET    /api/products           # List all products
GET    /api/products/42        # Get product 42
POST   /api/products           # Create new product
PUT    /api/products/42        # Update product 42
DELETE /api/products/42        # Delete product 42

# Users
GET    /api/users              # List users
GET    /api/users/5            # Get user 5
POST   /api/users              # Register new user

# Orders
GET    /api/orders             # List orders
GET    /api/orders/99          # Get order 99
POST   /api/orders             # Create new order

# User's orders (nested resource)
GET    /api/users/5/orders     # Get orders for user 5
```

---

## 📊 Complete REST Request Example

```http
# Request
POST /api/products HTTP/1.1
Host: shop.com
Content-Type: application/json
Authorization: Bearer eyJhbGc...

{
  "name": "Gaming Laptop",
  "price": 15000,
  "category": "electronics",
  "stock": 10
}

# Response
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/products/123

{
  "id": 123,
  "name": "Gaming Laptop",
  "price": 15000,
  "category": "electronics",
  "stock": 10,
  "created_at": "2024-12-20T15:00:00Z"
}
```

---

## ✅ REST Characteristics

### 1. Client-Server

<div dir="rtl">

Client و Server منفصلين تماماً:

</div>

```
Client (Frontend)  ◄──── HTTP ────►  Server (Backend)
```

### 2. Stateless

<div dir="rtl">

كل Request مستقل - Server لا يحفظ حالة Client:

</div>

```
Request 1: GET /products (with token)
Server processes, forgets

Request 2: GET /users (with token again)
Server doesn't remember Request 1
```

### 3. Cacheable

<div dir="rtl">

Responses يمكن حفظها في Cache:

</div>

```http
GET /api/products/42
Cache-Control: max-age=3600

# يمكن حفظ Response لمدة ساعة
```

### 4. Uniform Interface

<div dir="rtl">

نفس الطريقة للتعامل مع كل Resources:

</div>

```
# نفس النمط لكل resource:
GET    /{resource}         # List
GET    /{resource}/{id}    # Get one
POST   /{resource}         # Create
PUT    /{resource}/{id}    # Update
DELETE /{resource}/{id}    # Delete
```

---

## 🎯 Why REST?

### 1. Simplicity

```
REST easy to:
- Learn ✅
- Implement ✅
- Use ✅
- Debug ✅
```

### 2. Uses Standard HTTP

<div dir="rtl">

لا تحتاج بروتوكولات خاصة - فقط HTTP:

</div>

```
# Works with any HTTP client:
- curl
- Postman
- Browser
- Mobile apps
- IoT devices
```

### 3. Scalability

<div dir="rtl">

Stateless design = سهل التوسع:

</div>

```
User Request → Load Balancer → Server 1
                             → Server 2
                             → Server 3

أي server يقدر يتعامل مع any request
لأن كله stateless!
```

### 4. Flexibility

```json
// Same API, different formats:

GET /api/users/5
Accept: application/json
→ { "id": 5, "name": "Ahmed" }

GET /api/users/5
Accept: application/xml
→ <user><id>5</id><name>Ahmed</name></user>
```

---

## 🚫 What REST is NOT

<div dir="rtl">

**REST ليس:**

</div>

- ❌ Protocol (مثل HTTP, FTP)
- ❌ Standard (مثل HTML, JSON)
- ❌ Framework
- ❌ Library

<div dir="rtl">

**REST هو:**

</div>

- ✅ Architectural style
- ✅ Set of constraints/principles
- ✅ Best practices
- ✅ Design pattern

---

## 💡 RESTful vs REST-like

### Fully RESTful API:

```
✅ Uses HTTP methods correctly
✅ Resources identified by URLs
✅ Stateless
✅ Cacheable
✅ HATEOAS (links to related resources)
✅ Self-descriptive messages
```

### REST-like API (Most Real-World APIs):

```
✅ Uses HTTP methods
✅ Resources in URLs
✅ Stateless
✅ JSON responses
❌ Doesn't implement HATEOAS
❌ Not fully self-descriptive
```

<div dir="rtl">

**ملاحظة:** معظم APIs تُسمى "RESTful" لكن فعلياً "REST-like" - وهذا مقبول تماماً!

</div>

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **REST** = architectural style لبناء Web APIs
- ✅ يستخدم **HTTP methods** (GET, POST, PUT, DELETE)
- ✅ **Resources** معرّفة بـ URLs
- ✅ **Stateless** - كل request مستقل
- ✅ **JSON** هو الـ format الأكثر استخداماً
- ✅ **Simple** و easy to use
- ✅ المعيار الصناعي للـ Web APIs

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد أن فهمت ما هو REST، دعنا نتعمق في المبادئ الأساسية:

**➡️ [Lesson 2: REST Principles](./02-rest-principles.md)**

</div>

---

<div align="center">

[📚 Module Home](../README.md)

</div>
