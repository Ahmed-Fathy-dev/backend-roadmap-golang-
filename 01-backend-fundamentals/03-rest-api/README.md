# Module 1.3: REST API Fundamentals 🔌

<div dir="rtl">

## نظرة عامة

في هذا Module، سنتعلم **REST API** - المعيار الأكثر استخداماً لبناء APIs حديثة.

</div>

---

## 📚 Lessons (الدروس)

### REST Fundamentals

1. **[What is REST?](./lessons/01-what-is-rest.md)**
   <div dir="rtl">- مفهوم REST - REST Principles - لماذا REST؟</div>

2. **[REST Principles Deep Dive](./lessons/02-rest-principles.md)**
   <div dir="rtl">- 6 مبادئ أساسية - Stateless - Client-Server - Cacheable</div>

3. **[Resource Naming Conventions](./lessons/03-resource-naming.md)**
   <div dir="rtl">- كيف تسمي Resources - Best practices - Common mistakes</div>

4. **[HTTP Methods in REST](./lessons/04-http-methods-rest.md)**
   <div dir="rtl">- CRUD operations - Idempotency - Safe methods</div>

5. **[Status Codes in REST](./lessons/05-status-codes-rest.md)**
   <div dir="rtl">- متى تستخدم كل code - Error responses - Best practices</div>

6. **[API Versioning](./lessons/06-api-versioning.md)**
   <div dir="rtl">- استراتيجيات Versioning - URL vs Header vs Media Type</div>

7. **[Pagination, Filtering & Sorting](./lessons/07-pagination-filtering.md)**
   <div dir="rtl">- Pagination strategies - Query parameters - Performance</div>

8. **[Error Handling](./lessons/08-error-handling.md)**
   <div dir="rtl">- Error response format - Validation errors - Best practices</div>

---

## 💻 Examples (أمثلة عملية)

1. **[Complete REST API Design](./examples/01-complete-api-design.md)**
   <div dir="rtl">E-commerce API كامل</div>

2. **[CRUD Operations](./examples/02-crud-operations.md)**
   <div dir="rtl">أمثلة على Create, Read, Update, Delete</div>

3. **[Nested Resources](./examples/03-nested-resources.md)**
   <div dir="rtl">التعامل مع Resources المتداخلة</div>

4. **[Error Responses](./examples/04-error-responses.md)**
   <div dir="rtl">أمثلة على Error handling</div>

---

## 📖 Resources

- **[REST API Best Practices](./resources/rest-best-practices.md)**
  <div dir="rtl">قائمة شاملة بأفضل الممارسات</div>

- **[API Design Checklist](./resources/design-checklist.md)**
  <div dir="rtl">Checklist لتصميم API احترافي</div>

---

## 💡 Quick Reference

<div dir="rtl">

### REST في نقاط:

- **RE**presentational **S**tate **T**ransfer
- معمارية لبناء Web Services
- يستخدم HTTP Methods (GET, POST, PUT, DELETE)
- Stateless - كل request مستقل
- Resources تُعرّف بـ URLs

### مثال سريع:

```
GET    /api/products      # Get all products
GET    /api/products/5    # Get product 5
POST   /api/products      # Create product
PUT    /api/products/5    # Update product 5
DELETE /api/products/5    # Delete product 5
```

</div>

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 1.4: Authentication & Authorization](../04-auth-basics/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: HTTP Protocol](../02-http-protocol/README.md) | [🏠 Track 1](../README.md) | [📚 Main](../../README.md)

</div>
