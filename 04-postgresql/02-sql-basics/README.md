# Module 4.2: SQL Basics 📝

<div dir="rtl">

## نظرة عامة

SQL (Structured Query Language) هي اللغة اللي بنتكلم بيها مع الـ Database. في الـ Module ده هنتعلم كل أساسيات SQL من الصفر للاحتراف.

**المدة المتوقعة:** 1-2 أسبوع

</div>

---

## 📚 Lessons (الدروس)

### Part 1: Data Types (أنواع البيانات)

1. **[مقدمة في SQL](./lessons/01-intro-to-sql.md)**
   <div dir="rtl">- ما هو SQL - تاريخه - أنواع الأوامر - DDL, DML, DCL, TCL</div>

2. **[أنواع الأرقام](./lessons/02-numeric-types.md)**
   <div dir="rtl">- INTEGER, BIGINT, SMALLINT - DECIMAL, NUMERIC - REAL, DOUBLE PRECISION - SERIAL</div>

3. **[أنواع النصوص](./lessons/03-text-types.md)**
   <div dir="rtl">- VARCHAR, CHAR, TEXT - الفروق والاستخدامات - أفضل الممارسات</div>

4. **[أنواع التاريخ والوقت](./lessons/04-datetime-types.md)**
   <div dir="rtl">- DATE, TIME, TIMESTAMP - TIMESTAMPTZ - INTERVAL - الـ Timezone</div>

5. **[الأنواع الخاصة](./lessons/05-special-types.md)**
   <div dir="rtl">- BOOLEAN - UUID - JSON, JSONB - ARRAY - ENUM</div>

### Part 2: Creating Tables (إنشاء الجداول)

6. **[إنشاء الجداول](./lessons/06-create-table.md)**
   <div dir="rtl">- CREATE TABLE - الأعمدة - القيم الافتراضية</div>

7. **[القيود (Constraints)](./lessons/07-constraints.md)**
   <div dir="rtl">- PRIMARY KEY - FOREIGN KEY - UNIQUE - NOT NULL - CHECK - DEFAULT</div>

8. **[تعديل الجداول](./lessons/08-alter-table.md)**
   <div dir="rtl">- ADD COLUMN - DROP COLUMN - MODIFY - RENAME</div>

### Part 3: Basic Queries (الاستعلامات الأساسية)

9. **[INSERT - إدخال البيانات](./lessons/09-insert.md)**
   <div dir="rtl">- Single INSERT - Multiple INSERT - INSERT ... RETURNING - INSERT ... ON CONFLICT</div>

10. **[SELECT - قراءة البيانات](./lessons/10-select.md)**
    <div dir="rtl">- SELECT الأساسي - Columns - Aliases - DISTINCT</div>

11. **[WHERE - تصفية البيانات](./lessons/11-where.md)**
    <div dir="rtl">- Comparison Operators - AND, OR, NOT - IN, BETWEEN - LIKE, ILIKE - NULL handling</div>

12. **[ORDER BY & LIMIT](./lessons/12-order-limit.md)**
    <div dir="rtl">- Sorting - ASC, DESC - LIMIT - OFFSET - Pagination</div>

### Part 4: Aggregate Functions (دوال التجميع)

13. **[Aggregate Functions](./lessons/13-aggregate-functions.md)**
    <div dir="rtl">- COUNT - SUM - AVG - MIN, MAX</div>

14. **[GROUP BY و HAVING](./lessons/14-group-by.md)**
    <div dir="rtl">- تجميع البيانات - HAVING للفلترة - أمثلة عملية</div>

---

## 💻 Examples (أمثلة عملية)

1. **[بناء قاعدة بيانات متجر إلكتروني](./examples/01-ecommerce-database.md)**
2. **[بناء قاعدة بيانات مدونة](./examples/02-blog-database.md)**
3. **[استعلامات تحليلية للمبيعات](./examples/03-sales-analytics.md)**
4. **[البحث والفلترة المتقدمة](./examples/04-advanced-search.md)**

---

## 📖 Resources (موارد إضافية)

1. **[مرجع Data Types الكامل](./resources/data-types-reference.md)**
2. **[أنماط SQL الشائعة](./resources/sql-patterns.md)**
3. **[أخطاء SQL الشائعة](./resources/common-mistakes.md)**

---

## 🎯 ماذا ستتعلم؟

<div dir="rtl">

بعد إنهاء هذا Module، هتكون قادر على:

- ✅ فهم وإتقان أنواع البيانات في PostgreSQL
- ✅ تصميم وإنشاء Tables بشكل احترافي
- ✅ كتابة Constraints صحيحة
- ✅ إدخال وقراءة البيانات
- ✅ تصفية وترتيب النتائج
- ✅ استخدام Aggregate Functions
- ✅ كتابة تقارير وتحليلات

</div>

---

## ✅ Checklist

<div dir="rtl">

قبل الانتقال للـ Module التالي، تأكد إنك:

- [ ] فهمت الفرق بين أنواع البيانات
- [ ] تقدر تنشئ Table مع Constraints
- [ ] تقدر تكتب INSERT, SELECT, WHERE
- [ ] تقدر تستخدم ORDER BY, LIMIT
- [ ] تقدر تستخدم Aggregate Functions
- [ ] عملت تمارين على كل موضوع

</div>

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 4.3: CRUD Operations](../03-crud-operations/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Installation & Setup](../01-installation-setup/README.md) | [🏠 Track 4](../README.md)

</div>
