# Lesson 2: REST Principles 📐

<div dir="rtl">

## المقدمة

**6 مبادئ** تجعل API "RESTful" حقيقي!

</div>

---

## 1️⃣ Client-Server Separation

<div dir="rtl">

**المعنى:** Client و Server منفصلين تماماً.

</div>

```
Client (Frontend) ◄──── HTTP ────► Server (Backend)
     │                                    │
  UI/UX Logic                      Business Logic
  Data Display                     Data Storage
```

**Benefits:**

- ✅ يمكن تطوير كل واحد بشكل مستقل
- ✅ Multiple clients (web, mobile, desktop)

---

## 2️⃣ Stateless

<div dir="rtl">

**المعنى:** Server لا يحفظ أي state عن Client.

</div>

```
Request 1: GET /products (with token)
Server processes → returns data → forgets everything

Request 2: GET /users (with token again!)
Server doesn't remember Request 1
```

**Benefits:**

- ✅ سهل التوسع (any server can handle any request)
- ✅ Reliability (no session to lose)

---

## 3️⃣ Cacheable

<div dir="rtl">

**المعنى:** Responses يمكن حفظها في Cache.

</div>

```http
GET /api/products/42
Cache-Control: max-age=3600  # Cache for 1 hour

# Next request within 1 hour:
→ Served from cache ⚡ (no server hit!)
```

---

## 4️⃣ Uniform Interface

<div dir="rtl">

**المعنى:** نفس الطريقة للتعامل مع كل Resources.

</div>

```
# Same pattern for all:
GET    /{resource}         # List
GET    /{resource}/{id}    # Get one
POST   /{resource}         # Create
PUT    /{resource}/{id}    # Update
DELETE /{resource}/{id}    # Delete
```

---

## 5️⃣ Layered System

<div dir="rtl">

**المعنى:** يمكن إضافة layers بين Client و Server.

</div>

```
Client → Load Balancer → Cache → API Server → Database
```

---

## 6️⃣ Code on Demand (Optional)

<div dir="rtl">

**المعنى:** Server يمكنه إرسال executable code.

**نادراً الاستخدام!**

</div>

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Client-Server:** منفصلين
- ✅ **Stateless:** لا state على Server
- ✅ **Cacheable:** يمكن الـ caching
- ✅ **Uniform:** نفس الطريقة للكل
- ✅ **Layered:** يمكن إضافة layers
- ✅ **Code on Demand:** (نادر)

</div>

---

<div align="center">

[⬅️ Previous: What is REST](./01-what-is-rest.md) | [➡️ Next: Resource Naming](./03-resource-naming.md) | [📚 Module Home](../README.md)

</div>
