# Lesson 6: API Versioning 🔢

<div dir="rtl">

## المقدمة

**API Versioning** ضروري لتطوير API بدون كسر التطبيقات الموجودة!

</div>

---

## 🎯 Why Versioning?

```
Scenario: تريد تغيير API response structure

OLD (v1):
{
  "name": "Ahmed Ali",
  "age": 25
}

NEW (v2):
{
  "firstName": "Ahmed",
  "lastName": "Ali",
  "dateOfBirth": "1999-01-01"
}

❌ Without versioning: كل التطبيقات القديمة تتعطل!
✅ With versioning: v1 يعمل, v2 للجديد
```

---

## 📋 Versioning Strategies

### 1. URL Versioning (Most Common) ⭐

```
GET /api/v1/products
GET /api/v2/products
GET /api/v3/products
```

**Pros:**

- ✅ واضح جداً
- ✅ سهل الاستخدام
- ✅ سهل الـ routing

**Cons:**

- ❌ URLs مختلفة لنفس Resource

**Implementation:**

```go
router := gin.Default()

// v1 routes
v1 := router.Group("/api/v1")
{
    v1.GET("/products", GetProductsV1)
    v1.GET("/users", GetUsersV1)
}

// v2 routes
v2 := router.Group("/api/v2")
{
    v2.GET("/products", GetProductsV2)
    v2.GET("/users", GetUsersV2)
}
```

---

### 2. Header Versioning

```http
GET /api/products
Accept: application/vnd.myapi.v1+json

GET /api/products
Accept: application/vnd.myapi.v2+json
```

**Pros:**

- ✅ URL ثابت
- ✅ RESTful أكثر

**Cons:**

- ❌ أصعب في الاستخدام
- ❌ صعب اختباره في Browser

---

### 3. Query Parameter

```
GET /api/products?version=1
GET /api/products?version=2
```

**Pros:**

- ✅ بسيط

**Cons:**

- ❌ غير قياسي
- ❌ مش best practice

---

## ✅ Best Practices

### 1. Major Versions Only

```
✅ /api/v1/...
✅ /api/v2/...
✅ /api/v3/...

❌ /api/v1.2/...
❌ /api/v1.2.5/...
```

### 2. Support Multiple Versions

```go
// Support v1 and v2 simultaneously
v1 := router.Group("/api/v1")
v2 := router.Group("/api/v2")

// Deprecate gradually
```

### 3. Document Breaking Changes

```
v1 → v2 Breaking Changes:
- Response structure changed
- /users endpoint removed
- Authentication method changed
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **URL versioning** الأكثر شيوعاً
- ✅ **Major versions** only (v1, v2, v3)
- ✅ Support **multiple versions** للـ backward compatibility
- ✅ Document **breaking changes**

</div>

---

<div align="center">

[⬅️ Previous](./05-status-codes-rest.md) | [➡️ Next](./07-pagination-filtering.md) | [📚 Module Home](../README.md)

</div>
