# WHERE - تصفية البيانات 🎯

<div dir="rtl">

## مقدمة

`WHERE` بيسمحلك تفلتر البيانات وتجيب اللي محتاجه بس. من أهم الأجزاء في SQL لأنك نادراً ما هتحتاج كل البيانات.

**المدة المتوقعة:** 25-30 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
SELECT columns
FROM table_name
WHERE condition;
```

---

## 🔢 Comparison Operators

<div dir="rtl">

### المقارنات الأساسية

</div>

```sql
-- يساوي
SELECT * FROM employees WHERE department = 'Engineering';

-- لا يساوي
SELECT * FROM employees WHERE department != 'Engineering';
SELECT * FROM employees WHERE department <> 'Engineering';  -- نفس الشيء

-- أكبر من
SELECT * FROM employees WHERE salary > 70000;

-- أكبر من أو يساوي
SELECT * FROM employees WHERE salary >= 70000;

-- أصغر من
SELECT * FROM employees WHERE salary < 60000;

-- أصغر من أو يساوي
SELECT * FROM employees WHERE salary <= 60000;
```

<div dir="rtl">

### مع التواريخ

</div>

```sql
-- بعد تاريخ معين
SELECT * FROM employees WHERE hire_date > '2020-01-01';

-- قبل تاريخ معين
SELECT * FROM employees WHERE hire_date < '2020-01-01';

-- في سنة معينة
SELECT * FROM employees
WHERE hire_date >= '2020-01-01' AND hire_date < '2021-01-01';

-- باستخدام EXTRACT
SELECT * FROM employees WHERE EXTRACT(YEAR FROM hire_date) = 2020;
```

---

## 🔗 Logical Operators

<div dir="rtl">

### AND - كل الشروط

</div>

```sql
-- لازم كل الشروط تتحقق
SELECT * FROM employees
WHERE department = 'Engineering'
  AND salary > 70000;

SELECT * FROM employees
WHERE department = 'Engineering'
  AND salary > 70000
  AND is_active = TRUE;
```

<div dir="rtl">

### OR - أي شرط

</div>

```sql
-- أي شرط يكفي
SELECT * FROM employees
WHERE department = 'Engineering'
   OR department = 'Marketing';

SELECT * FROM employees
WHERE salary > 80000
   OR department = 'Finance';
```

<div dir="rtl">

### NOT - نفي الشرط

</div>

```sql
SELECT * FROM employees WHERE NOT is_active;
SELECT * FROM employees WHERE NOT department = 'HR';
SELECT * FROM employees WHERE department NOT IN ('HR', 'Finance');
```

<div dir="rtl">

### تركيب الشروط (مع الأقواس)

</div>

```sql
-- ⚠️ الأقواس مهمة جداً!

-- بدون أقواس (نتيجة غير متوقعة)
SELECT * FROM employees
WHERE department = 'Engineering' OR department = 'Marketing'
  AND salary > 70000;
-- AND بيتنفذ قبل OR!
-- ده معناه: Engineering أو (Marketing AND salary > 70000)

-- ✅ مع أقواس (النتيجة المتوقعة)
SELECT * FROM employees
WHERE (department = 'Engineering' OR department = 'Marketing')
  AND salary > 70000;
-- ده معناه: (Engineering أو Marketing) AND salary > 70000

-- مثال معقد
SELECT * FROM employees
WHERE (department = 'Engineering' AND salary > 80000)
   OR (department = 'Finance' AND salary > 70000)
   OR (is_active = FALSE);
```

---

## 📊 IN و NOT IN

<div dir="rtl">

### IN - قيمة من مجموعة

</div>

```sql
-- بدل OR متكرر
SELECT * FROM employees
WHERE department IN ('Engineering', 'Marketing', 'Finance');

-- نفس النتيجة بـ OR (أطول)
SELECT * FROM employees
WHERE department = 'Engineering'
   OR department = 'Marketing'
   OR department = 'Finance';

-- مع أرقام
SELECT * FROM products WHERE category_id IN (1, 2, 5, 10);

-- مع Subquery
SELECT * FROM employees
WHERE department IN (
    SELECT department FROM employees WHERE salary > 80000
);
```

<div dir="rtl">

### NOT IN

</div>

```sql
SELECT * FROM employees
WHERE department NOT IN ('HR', 'Finance');

-- ⚠️ احذر من NULL في NOT IN!
SELECT * FROM employees
WHERE department NOT IN ('HR', NULL);
-- هيرجع 0 صفوف! لأن مقارنة أي حاجة بـ NULL = NULL (unknown)
```

---

## 📏 BETWEEN

<div dir="rtl">

### نطاق القيم

</div>

```sql
-- الرواتب من 60000 لـ 80000 (شامل)
SELECT * FROM employees
WHERE salary BETWEEN 60000 AND 80000;

-- نفس النتيجة بـ >= و <=
SELECT * FROM employees
WHERE salary >= 60000 AND salary <= 80000;

-- نطاق التواريخ
SELECT * FROM employees
WHERE hire_date BETWEEN '2020-01-01' AND '2021-12-31';

-- NOT BETWEEN
SELECT * FROM employees
WHERE salary NOT BETWEEN 60000 AND 80000;
```

---

## 🔍 LIKE و Pattern Matching

<div dir="rtl">

### LIKE - البحث بالنمط

</div>

```sql
-- Wildcards:
-- % = أي عدد من الأحرف (صفر أو أكتر)
-- _ = حرف واحد بالظبط

-- يبدأ بـ
SELECT * FROM employees WHERE first_name LIKE 'A%';

-- ينتهي بـ
SELECT * FROM employees WHERE email LIKE '%@company.com';

-- يحتوي على
SELECT * FROM employees WHERE first_name LIKE '%ah%';

-- حرف واحد
SELECT * FROM employees WHERE first_name LIKE '_a%';
-- الحرف التاني هو 'a'

-- طول محدد
SELECT * FROM products WHERE sku LIKE '___';  -- 3 حروف بالظبط
SELECT * FROM products WHERE sku LIKE 'A__';  -- يبدأ بـ A + حرفين
```

<div dir="rtl">

### ILIKE - بدون حساسية للحالة (PostgreSQL)

</div>

```sql
-- LIKE حساس للحالة
SELECT * FROM employees WHERE first_name LIKE 'ahmed';  -- مش هيلاقي Ahmed

-- ILIKE مش حساس للحالة
SELECT * FROM employees WHERE first_name ILIKE 'ahmed';  -- هيلاقي Ahmed

-- البحث case-insensitive
SELECT * FROM products WHERE name ILIKE '%laptop%';
```

<div dir="rtl">

### NOT LIKE

</div>

```sql
SELECT * FROM employees WHERE email NOT LIKE '%@gmail.com';
```

<div dir="rtl">

### Escaping في LIKE

</div>

```sql
-- البحث عن % أو _ كحرف عادي
SELECT * FROM products WHERE name LIKE '%50\%%' ESCAPE '\';
-- البحث عن "50%"
```

---

## 🎭 Regular Expressions

<div dir="rtl">

### SIMILAR TO (SQL Standard)

</div>

```sql
-- مزيج من LIKE و Regex
SELECT * FROM employees
WHERE first_name SIMILAR TO '(Ahmed|Omar|Hassan)';

SELECT * FROM employees
WHERE email SIMILAR TO '%@(gmail|yahoo|hotmail)\.com';
```

<div dir="rtl">

### ~ و ~* (PostgreSQL Regex)

</div>

```sql
-- ~ للـ case-sensitive regex
SELECT * FROM employees WHERE email ~ '^[a-z]+@company\.com$';

-- ~* للـ case-insensitive regex
SELECT * FROM employees WHERE first_name ~* '^ah';

-- !~ للنفي
SELECT * FROM employees WHERE email !~ '@gmail\.com$';

-- أمثلة متقدمة
-- إيميلات valid
SELECT * FROM users
WHERE email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';

-- أرقام التليفون المصرية
SELECT * FROM contacts
WHERE phone ~ '^\+20[0-9]{10}$';
```

---

## ❓ NULL Handling

<div dir="rtl">

### IS NULL و IS NOT NULL

</div>

```sql
-- الصفوف اللي فيها department = NULL
SELECT * FROM employees WHERE department IS NULL;

-- الصفوف اللي فيها department مش NULL
SELECT * FROM employees WHERE department IS NOT NULL;

-- ⚠️ لا تستخدم = للمقارنة بـ NULL!
SELECT * FROM employees WHERE department = NULL;     -- ❌ مش هيشتغل!
SELECT * FROM employees WHERE department IS NULL;    -- ✅ صح
```

<div dir="rtl">

### COALESCE في WHERE

</div>

```sql
-- معاملة NULL كقيمة معينة
SELECT * FROM products
WHERE COALESCE(discount, 0) > 10;

-- البحث مع default
SELECT * FROM contacts
WHERE COALESCE(phone, mobile, '') != '';
```

<div dir="rtl">

### NULLIF في WHERE

</div>

```sql
-- تجاهل قيمة معينة
SELECT * FROM products
WHERE NULLIF(status, 'unknown') IS NOT NULL;
```

---

## 🎨 شروط متقدمة

<div dir="rtl">

### ANY / SOME

</div>

```sql
-- أي قيمة من array
SELECT * FROM products WHERE category_id = ANY(ARRAY[1, 2, 3]);

-- مع Subquery
SELECT * FROM employees
WHERE salary > ANY(SELECT salary FROM employees WHERE department = 'HR');
```

<div dir="rtl">

### ALL

</div>

```sql
-- أكبر من كل القيم
SELECT * FROM employees
WHERE salary > ALL(SELECT salary FROM employees WHERE department = 'HR');
```

<div dir="rtl">

### EXISTS

</div>

```sql
-- لو فيه صفوف في الـ Subquery
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.department = d.name AND e.salary > 80000
);
```

---

## 📅 فلترة التواريخ الشائعة

```sql
-- اليوم
SELECT * FROM orders WHERE created_at::DATE = CURRENT_DATE;

-- الأسبوع ده
SELECT * FROM orders
WHERE created_at >= DATE_TRUNC('week', NOW());

-- الشهر ده
SELECT * FROM orders
WHERE created_at >= DATE_TRUNC('month', NOW());

-- السنة دي
SELECT * FROM orders
WHERE EXTRACT(YEAR FROM created_at) = EXTRACT(YEAR FROM NOW());

-- آخر 7 أيام
SELECT * FROM orders
WHERE created_at >= NOW() - INTERVAL '7 days';

-- آخر 30 يوم
SELECT * FROM orders
WHERE created_at >= NOW() - INTERVAL '30 days';

-- بين تاريخين
SELECT * FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31 23:59:59';
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم Indexes

</div>

```sql
-- ✅ الـ Index هيشتغل
WHERE email = 'test@test.com'

-- ❌ الـ Index مش هيشتغل
WHERE LOWER(email) = 'test@test.com'
-- الحل: أنشئ functional index
```

<div dir="rtl">

### 2. تجنب Functions على Columns

</div>

```sql
-- ❌ بطيء (full table scan)
WHERE YEAR(created_at) = 2024

-- ✅ أسرع (يستخدم index)
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
```

<div dir="rtl">

### 3. استخدم الأقواس

</div>

```sql
-- ✅ واضح
WHERE (condition1 OR condition2) AND condition3

-- ❌ غامض
WHERE condition1 OR condition2 AND condition3
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Comparison**: `=`, `!=`, `>`, `<`, `>=`, `<=`
2. **Logical**: `AND`, `OR`, `NOT` (مع أقواس!)
3. **Range**: `BETWEEN` للنطاقات
4. **Pattern**: `LIKE` و `ILIKE` للنصوص
5. **NULL**: استخدم `IS NULL` / `IS NOT NULL`
6. **Performance**: تجنب functions على columns

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [ORDER BY & LIMIT](./12-order-limit.md)**

</div>

---

<div align="center">

[⬅️ السابق: SELECT](./10-select.md) | [🏠 العودة للـ Module](../README.md)

</div>
