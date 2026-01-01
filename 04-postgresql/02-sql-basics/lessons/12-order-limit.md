# ORDER BY & LIMIT - الترتيب والتحديد 📊

<div dir="rtl">

## مقدمة

`ORDER BY` بيرتب النتائج حسب عمود أو أكتر، و`LIMIT` بيحدد عدد الصفوف المرجعة. مهمين جداً للـ Pagination وعرض البيانات.

**المدة المتوقعة:** 20-25 دقيقة

</div>

---

## 📊 ORDER BY

<div dir="rtl">

### الترتيب الأساسي

</div>

```sql
-- ترتيب تصاعدي (الافتراضي)
SELECT * FROM employees ORDER BY salary;
SELECT * FROM employees ORDER BY salary ASC;  -- نفس الشيء

-- ترتيب تنازلي
SELECT * FROM employees ORDER BY salary DESC;

-- ترتيب أبجدي
SELECT * FROM employees ORDER BY first_name;

-- ترتيب بالتاريخ
SELECT * FROM employees ORDER BY hire_date;          -- الأقدم أولاً
SELECT * FROM employees ORDER BY hire_date DESC;     -- الأحدث أولاً
```

<div dir="rtl">

### ترتيب بعدة أعمدة

</div>

```sql
-- الترتيب الأول، ثم الثاني لو متساويين
SELECT * FROM employees
ORDER BY department, salary DESC;
-- ترتيب بالقسم (أبجدي)، ثم بالراتب (تنازلي) داخل كل قسم

SELECT * FROM employees
ORDER BY department ASC, salary DESC, first_name ASC;

-- مثال: ترتيب المنتجات
SELECT * FROM products
ORDER BY category_id, price DESC, name;
-- في كل category، الأغلى أولاً، ولو نفس السعر بالاسم
```

<div dir="rtl">

### الترتيب برقم العمود

</div>

```sql
-- الترتيب بموقع العمود (مش مفضل بس بيشتغل)
SELECT first_name, last_name, salary
FROM employees
ORDER BY 3 DESC;  -- العمود الثالث (salary)

-- ⚠️ لا تستخدمها في Production code
-- لأن لو تغير ترتيب الأعمدة، النتيجة هتتغير
```

---

## 🔠 الترتيب والـ NULL

```sql
-- الترتيب الافتراضي: NULLs في الآخر (ASC) أو الأول (DESC)
SELECT * FROM employees ORDER BY department;
-- NULL values هتظهر في الآخر

-- التحكم في موقع NULL
SELECT * FROM employees ORDER BY department NULLS FIRST;
SELECT * FROM employees ORDER BY department NULLS LAST;

SELECT * FROM employees ORDER BY department DESC NULLS FIRST;
SELECT * FROM employees ORDER BY department DESC NULLS LAST;
```

---

## 🔤 الترتيب والـ Collation

<div dir="rtl">

### الترتيب للنصوص

</div>

```sql
-- الترتيب الافتراضي
SELECT * FROM users ORDER BY name;

-- ترتيب case-insensitive
SELECT * FROM users ORDER BY LOWER(name);

-- ترتيب بـ Collation معين
SELECT * FROM users ORDER BY name COLLATE "en_US";

-- للعربية
SELECT * FROM users ORDER BY name COLLATE "ar_EG";
```

---

## 📐 الترتيب بـ Expressions

```sql
-- ترتيب بحساب
SELECT
    first_name,
    salary,
    salary * 12 AS annual_salary
FROM employees
ORDER BY salary * 12 DESC;

-- أو باستخدام الـ Alias
SELECT
    first_name,
    salary * 12 AS annual_salary
FROM employees
ORDER BY annual_salary DESC;

-- ترتيب بـ CASE
SELECT
    first_name,
    department,
    CASE department
        WHEN 'Engineering' THEN 1
        WHEN 'Finance' THEN 2
        WHEN 'Marketing' THEN 3
        ELSE 4
    END AS dept_order
FROM employees
ORDER BY dept_order, first_name;

-- ترتيب عشوائي
SELECT * FROM products ORDER BY RANDOM();
```

---

## 📏 LIMIT

<div dir="rtl">

### تحديد عدد النتائج

</div>

```sql
-- أول 10 صفوف
SELECT * FROM employees LIMIT 10;

-- أول 5 موظفين براتب عالي
SELECT * FROM employees
ORDER BY salary DESC
LIMIT 5;

-- أحدث 3 طلبات
SELECT * FROM orders
ORDER BY created_at DESC
LIMIT 3;
```

---

## 📄 OFFSET

<div dir="rtl">

### تخطي صفوف

</div>

```sql
-- تخطي أول 5 وجيب الباقي
SELECT * FROM employees
ORDER BY id
OFFSET 5;

-- تخطي 10 وجيب 5
SELECT * FROM employees
ORDER BY id
LIMIT 5 OFFSET 10;
```

---

## 📑 Pagination (ترقيم الصفحات)

<div dir="rtl">

### حساب الـ Pagination

</div>

```
Page Size = 10
Page 1: LIMIT 10 OFFSET 0   (rows 1-10)
Page 2: LIMIT 10 OFFSET 10  (rows 11-20)
Page 3: LIMIT 10 OFFSET 20  (rows 21-30)
...
Page N: LIMIT 10 OFFSET (N-1) * 10
```

<div dir="rtl">

### أمثلة Pagination

</div>

```sql
-- الصفحة الأولى (10 عناصر)
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 0;

-- الصفحة الثانية
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 10;

-- صفحة ديناميكية
-- page_number = 3, page_size = 10
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 20;  -- (3-1) * 10 = 20
```

<div dir="rtl">

### Pagination مع COUNT

</div>

```sql
-- إجمالي العناصر (لحساب عدد الصفحات)
SELECT COUNT(*) AS total FROM products;

-- البيانات
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 0;

-- في query واحد (PostgreSQL)
SELECT
    *,
    COUNT(*) OVER() AS total_count
FROM products
ORDER BY id
LIMIT 10 OFFSET 0;
```

---

## 🚀 Keyset Pagination (أسرع)

<div dir="rtl">

### المشكلة مع OFFSET

</div>

```sql
-- ⚠️ OFFSET بطيء مع أرقام كبيرة
SELECT * FROM huge_table
ORDER BY id
LIMIT 10 OFFSET 1000000;
-- PostgreSQL لازم يقرأ 1,000,000 صف ويتخطاهم!
```

<div dir="rtl">

### الحل: Keyset/Cursor Pagination

</div>

```sql
-- الصفحة الأولى
SELECT * FROM products
WHERE id > 0  -- أو بدون WHERE للأولى
ORDER BY id
LIMIT 10;

-- الصفحة التالية (بعد آخر id في الصفحة السابقة)
SELECT * FROM products
WHERE id > 1234  -- آخر id كان 1234
ORDER BY id
LIMIT 10;

-- مع عدة أعمدة للترتيب
SELECT * FROM products
WHERE (created_at, id) > ('2024-01-15 10:00:00', 5678)
ORDER BY created_at, id
LIMIT 10;
```

<div dir="rtl">

### مقارنة طرق الـ Pagination

</div>

```
┌────────────────────────────────────────────────────────────────────┐
│  OFFSET/LIMIT                                                       │
├────────────────────────────────────────────────────────────────────┤
│  ✅ سهل الاستخدام                                                  │
│  ✅ يدعم القفز لأي صفحة                                            │
│  ❌ بطيء مع OFFSET كبير                                            │
│  ❌ مشاكل مع البيانات المتغيرة (ممكن يكرر أو يفقد صفوف)             │
├────────────────────────────────────────────────────────────────────┤
│  Keyset Pagination                                                  │
├────────────────────────────────────────────────────────────────────┤
│  ✅ أداء ثابت بغض النظر عن الصفحة                                  │
│  ✅ صحيح مع البيانات المتغيرة                                      │
│  ❌ يحتاج unique, orderable column                                 │
│  ❌ لا يدعم القفز لصفحة معينة                                      │
└────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 FETCH (SQL Standard)

<div dir="rtl">

### بديل LIMIT القياسي

</div>

```sql
-- SQL Standard syntax
SELECT * FROM employees
ORDER BY salary DESC
FETCH FIRST 10 ROWS ONLY;

-- مع OFFSET
SELECT * FROM employees
ORDER BY salary DESC
OFFSET 5 ROWS
FETCH FIRST 10 ROWS ONLY;

-- FETCH NEXT = FETCH FIRST
SELECT * FROM employees
ORDER BY salary DESC
FETCH NEXT 5 ROWS ONLY;
```

---

## 🔝 TOP N Queries

<div dir="rtl">

### أعلى/أقل N

</div>

```sql
-- أعلى 5 رواتب
SELECT * FROM employees
ORDER BY salary DESC
LIMIT 5;

-- أقل 5 رواتب
SELECT * FROM employees
ORDER BY salary ASC
LIMIT 5;

-- أعلى راتب في كل قسم
SELECT DISTINCT ON (department)
    department, first_name, salary
FROM employees
ORDER BY department, salary DESC;

-- أو باستخدام Window Function
SELECT * FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
) sub
WHERE rn <= 3;  -- أعلى 3 في كل قسم
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. دايماً ORDER BY مع LIMIT

</div>

```sql
-- ❌ النتيجة غير متوقعة
SELECT * FROM products LIMIT 10;

-- ✅ النتيجة ثابتة ومتوقعة
SELECT * FROM products
ORDER BY id
LIMIT 10;
```

<div dir="rtl">

### 2. استخدم Keyset للـ Large Datasets

</div>

```sql
-- ❌ بطيء
SELECT * FROM logs ORDER BY id LIMIT 10 OFFSET 1000000;

-- ✅ سريع
SELECT * FROM logs WHERE id > 1000000 ORDER BY id LIMIT 10;
```

<div dir="rtl">

### 3. Index للـ ORDER BY

</div>

```sql
-- أنشئ Index للأعمدة اللي بترتب بيها كتير
CREATE INDEX idx_products_price ON products(price DESC);
CREATE INDEX idx_orders_date ON orders(created_at DESC);
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **ORDER BY** لترتيب النتائج
2. **ASC** تصاعدي (default)، **DESC** تنازلي
3. **LIMIT** لتحديد العدد
4. **OFFSET** للـ Pagination (بحذر!)
5. **Keyset Pagination** أفضل للـ large datasets
6. دايماً **ORDER BY مع LIMIT**

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Aggregate Functions](./13-aggregate-functions.md)**

</div>

---

<div align="center">

[⬅️ السابق: WHERE](./11-where.md) | [🏠 العودة للـ Module](../README.md)

</div>
