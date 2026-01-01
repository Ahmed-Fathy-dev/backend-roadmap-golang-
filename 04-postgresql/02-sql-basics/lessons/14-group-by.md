# GROUP BY و HAVING - التجميع والتصفية 📊

<div dir="rtl">

## مقدمة

`GROUP BY` بيجمّع الصفوف اللي ليها نفس القيم في مجموعات، و`HAVING` بيصفّي المجموعات بناءً على شرط. مهمين جداً للتقارير والإحصائيات.

**المدة المتوقعة:** 25-30 دقيقة

</div>

---

## 📊 GROUP BY الأساسي

<div dir="rtl">

### كيف يعمل GROUP BY؟

</div>

```
┌─────────────────────────────────────────────────────────────────┐
│                     بدون GROUP BY                                │
├─────────────────────────────────────────────────────────────────┤
│  employees table:                                                │
│  ┌────┬─────────┬──────────────┬─────────┐                      │
│  │ id │  name   │  department  │ salary  │                      │
│  ├────┼─────────┼──────────────┼─────────┤                      │
│  │ 1  │ Ahmed   │ Engineering  │ 75000   │                      │
│  │ 2  │ Sara    │ Marketing    │ 65000   │                      │
│  │ 3  │ Omar    │ Engineering  │ 80000   │                      │
│  │ 4  │ Fatima  │ Marketing    │ 60000   │                      │
│  │ 5  │ Khaled  │ Engineering  │ 90000   │                      │
│  └────┴─────────┴──────────────┴─────────┘                      │
│                                                                  │
│  SELECT COUNT(*) FROM employees;  → 5                           │
├─────────────────────────────────────────────────────────────────┤
│                     مع GROUP BY                                  │
├─────────────────────────────────────────────────────────────────┤
│  SELECT department, COUNT(*) FROM employees GROUP BY department; │
│                                                                  │
│  ┌──────────────┬───────┐                                       │
│  │  department  │ count │                                       │
│  ├──────────────┼───────┤                                       │
│  │ Engineering  │   3   │  ← (Ahmed, Omar, Khaled)              │
│  │ Marketing    │   2   │  ← (Sara, Fatima)                     │
│  └──────────────┴───────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة الأساسية

```sql
SELECT
    column1,           -- العمود/الأعمدة اللي بنجمّع بيها
    AGG_FUNC(column2)  -- Aggregate function
FROM table_name
WHERE condition        -- تصفية قبل التجميع (اختياري)
GROUP BY column1       -- التجميع
HAVING agg_condition   -- تصفية بعد التجميع (اختياري)
ORDER BY column1;      -- الترتيب (اختياري)
```

---

## 🔢 أمثلة أساسية

```sql
-- عدد الموظفين في كل قسم
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- مجموع الرواتب لكل قسم
SELECT
    department,
    SUM(salary) AS total_salaries
FROM employees
GROUP BY department;

-- متوسط الرواتب لكل قسم
SELECT
    department,
    ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department;

-- أعلى وأقل راتب في كل قسم
SELECT
    department,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

---

## 📊 GROUP BY مع عدة أعمدة

```sql
-- التجميع بأكثر من عمود
SELECT
    department,
    is_active,
    COUNT(*) AS count
FROM employees
GROUP BY department, is_active
ORDER BY department, is_active;

-- النتيجة:
-- department   | is_active | count
-- Engineering  | false     | 1
-- Engineering  | true      | 4
-- Marketing    | true      | 2
-- HR           | true      | 2

-- تقرير المبيعات بالسنة والشهر
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY year, month;

-- المنتجات حسب الفئة والحالة
SELECT
    category_id,
    status,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price
FROM products
GROUP BY category_id, status
ORDER BY category_id, status;
```

---

## 🎯 HAVING - تصفية المجموعات

<div dir="rtl">

### الفرق بين WHERE و HAVING

</div>

```
┌─────────────────────────────────────────────────────────────────┐
│  WHERE  vs  HAVING                                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHERE:                                                          │
│  ├── يعمل على الصفوف الفردية                                    │
│  ├── يتنفذ قبل GROUP BY                                         │
│  └── لا يمكن استخدام Aggregate functions                        │
│                                                                  │
│  HAVING:                                                         │
│  ├── يعمل على المجموعات                                         │
│  ├── يتنفذ بعد GROUP BY                                         │
│  └── يستخدم مع Aggregate functions                              │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  ترتيب التنفيذ:                                                  │
│                                                                  │
│  FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT   │
│         ↑                   ↑                                    │
│    صفوف فردية           مجموعات                                  │
└─────────────────────────────────────────────────────────────────┘
```

```sql
-- WHERE: تصفية الصفوف قبل التجميع
SELECT department, COUNT(*)
FROM employees
WHERE is_active = TRUE    -- صفوف الموظفين النشطين فقط
GROUP BY department;

-- HAVING: تصفية المجموعات بعد التجميع
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;      -- الأقسام اللي فيها أكتر من 2 موظفين

-- الاثنين معاً
SELECT department, COUNT(*) AS emp_count
FROM employees
WHERE is_active = TRUE          -- 1. فلترة: الموظفين النشطين فقط
GROUP BY department             -- 2. تجميع: حسب القسم
HAVING COUNT(*) > 2             -- 3. فلترة المجموعات: أكتر من 2
ORDER BY emp_count DESC;        -- 4. ترتيب
```

---

## 📈 أمثلة HAVING

```sql
-- الأقسام اللي متوسط راتبها أكبر من 70000
SELECT
    department,
    ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;

-- المنتجات اللي إجمالي مبيعاتها أكبر من 10000
SELECT
    product_id,
    SUM(quantity) AS total_sold,
    SUM(quantity * price) AS total_revenue
FROM order_items
GROUP BY product_id
HAVING SUM(quantity * price) > 10000
ORDER BY total_revenue DESC;

-- العملاء اللي عملوا أكتر من 5 طلبات
SELECT
    user_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5
ORDER BY total_spent DESC;

-- HAVING مع عدة شروط
SELECT
    department,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) >= 3 AND AVG(salary) > 60000;
```

---

## 🔄 GROUP BY مع Expressions

```sql
-- التجميع بتعبير (Expression)
SELECT
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    COUNT(*) AS employees_hired
FROM employees
GROUP BY EXTRACT(YEAR FROM hire_date)
ORDER BY hire_year;

-- التجميع بـ CASE
SELECT
    CASE
        WHEN salary >= 80000 THEN 'Senior'
        WHEN salary >= 60000 THEN 'Mid'
        ELSE 'Junior'
    END AS salary_level,
    COUNT(*) AS count,
    ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY
    CASE
        WHEN salary >= 80000 THEN 'Senior'
        WHEN salary >= 60000 THEN 'Mid'
        ELSE 'Junior'
    END
ORDER BY avg_salary DESC;

-- التجميع بالشهر واليوم
SELECT
    TO_CHAR(created_at, 'YYYY-MM') AS month,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY TO_CHAR(created_at, 'YYYY-MM')
ORDER BY month;

-- التجميع بيوم الأسبوع
SELECT
    TO_CHAR(created_at, 'Day') AS day_name,
    EXTRACT(DOW FROM created_at) AS day_number,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    TO_CHAR(created_at, 'Day'),
    EXTRACT(DOW FROM created_at)
ORDER BY day_number;
```

---

## 📋 GROUPING SETS (PostgreSQL)

<div dir="rtl">

### تجميعات متعددة في Query واحد

</div>

```sql
-- GROUPING SETS: تجميعات مختلفة
SELECT
    department,
    is_active,
    COUNT(*) AS count
FROM employees
GROUP BY GROUPING SETS (
    (department),           -- تجميع بالقسم
    (is_active),            -- تجميع بالحالة
    (department, is_active) -- تجميع بالاثنين
);

-- النتيجة:
-- department   | is_active | count
-- Engineering  | NULL      | 5      ← تجميع بالقسم فقط
-- Marketing    | NULL      | 2
-- NULL         | true      | 6      ← تجميع بالحالة فقط
-- NULL         | false     | 1
-- Engineering  | true      | 4      ← تجميع بالاثنين
-- Engineering  | false     | 1
-- Marketing    | true      | 2
```

---

## 🎲 ROLLUP

<div dir="rtl">

### تجميع هرمي مع Subtotals

</div>

```sql
-- ROLLUP: يعطي subtotals و grand total
SELECT
    COALESCE(department, 'TOTAL') AS department,
    COALESCE(is_active::TEXT, 'ALL') AS status,
    COUNT(*) AS count,
    SUM(salary) AS total_salary
FROM employees
GROUP BY ROLLUP (department, is_active);

-- النتيجة (مع subtotals):
-- department   | status | count | total_salary
-- Engineering  | true   | 4     | 320000
-- Engineering  | false  | 1     | 70000
-- Engineering  | ALL    | 5     | 390000       ← Subtotal
-- Marketing    | true   | 2     | 125000
-- Marketing    | ALL    | 2     | 125000       ← Subtotal
-- TOTAL        | ALL    | 7     | 515000       ← Grand Total

-- تقرير مبيعات مع ROLLUP
SELECT
    COALESCE(TO_CHAR(order_date, 'YYYY'), 'Total') AS year,
    COALESCE(TO_CHAR(order_date, 'MM'), 'All') AS month,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY ROLLUP (
    TO_CHAR(order_date, 'YYYY'),
    TO_CHAR(order_date, 'MM')
)
ORDER BY year, month;
```

---

## 🧊 CUBE

<div dir="rtl">

### كل التجميعات الممكنة

</div>

```sql
-- CUBE: كل التجميعات الممكنة
SELECT
    department,
    is_active,
    COUNT(*) AS count
FROM employees
GROUP BY CUBE (department, is_active);

-- CUBE يعطي:
-- 1. تجميع بالقسم فقط
-- 2. تجميع بالحالة فقط
-- 3. تجميع بالاثنين
-- 4. Grand total (بدون تجميع)

-- مقارنة:
-- ROLLUP (A, B) = (A,B), (A), ()
-- CUBE (A, B)   = (A,B), (A), (B), ()
```

---

## 🏷️ GROUPING() Function

```sql
-- GROUPING() بيحدد إذا كان NULL من التجميع أم من البيانات
SELECT
    CASE WHEN GROUPING(department) = 1 THEN 'All Depts' ELSE department END AS dept,
    CASE WHEN GROUPING(is_active) = 1 THEN 'All Status' ELSE is_active::TEXT END AS status,
    COUNT(*) AS count
FROM employees
GROUP BY ROLLUP (department, is_active);

-- GROUPING() = 1: القيمة NULL بسبب التجميع (subtotal)
-- GROUPING() = 0: القيمة من البيانات (قد تكون NULL أو لأ)
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. تقرير الموظفين

</div>

```sql
SELECT
    department,
    COUNT(*) AS total_employees,
    COUNT(*) FILTER (WHERE is_active) AS active_employees,
    COUNT(*) FILTER (WHERE NOT is_active) AS inactive_employees,
    ROUND(AVG(salary), 2) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    SUM(salary) AS total_payroll
FROM employees
GROUP BY department
HAVING COUNT(*) >= 2
ORDER BY total_employees DESC;
```

<div dir="rtl">

### 2. تقرير المبيعات الشهري

</div>

```sql
SELECT
    TO_CHAR(order_date, 'YYYY-MM') AS month,
    COUNT(*) AS total_orders,
    COUNT(DISTINCT user_id) AS unique_customers,
    SUM(total_amount) AS total_revenue,
    ROUND(AVG(total_amount), 2) AS avg_order_value,
    MAX(total_amount) AS largest_order
FROM orders
WHERE order_date >= DATE_TRUNC('year', NOW())
GROUP BY TO_CHAR(order_date, 'YYYY-MM')
ORDER BY month;
```

<div dir="rtl">

### 3. أفضل المنتجات

</div>

```sql
SELECT
    p.name AS product_name,
    COUNT(oi.id) AS times_ordered,
    SUM(oi.quantity) AS total_quantity_sold,
    SUM(oi.quantity * oi.price) AS total_revenue,
    ROUND(AVG(oi.price), 2) AS avg_selling_price
FROM products p
JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name
HAVING SUM(oi.quantity) >= 10
ORDER BY total_revenue DESC
LIMIT 10;
```

<div dir="rtl">

### 4. تحليل العملاء

</div>

```sql
SELECT
    CASE
        WHEN total_spent >= 10000 THEN 'VIP'
        WHEN total_spent >= 5000 THEN 'Premium'
        WHEN total_spent >= 1000 THEN 'Regular'
        ELSE 'New'
    END AS customer_tier,
    COUNT(*) AS customer_count,
    ROUND(AVG(total_spent), 2) AS avg_spent,
    SUM(total_spent) AS tier_revenue
FROM (
    SELECT
        user_id,
        SUM(total_amount) AS total_spent
    FROM orders
    GROUP BY user_id
) customer_totals
GROUP BY
    CASE
        WHEN total_spent >= 10000 THEN 'VIP'
        WHEN total_spent >= 5000 THEN 'Premium'
        WHEN total_spent >= 1000 THEN 'Regular'
        ELSE 'New'
    END
ORDER BY tier_revenue DESC;
```

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. عمود مش في GROUP BY

</div>

```sql
-- ❌ خطأ: first_name مش في GROUP BY
SELECT department, first_name, COUNT(*)
FROM employees
GROUP BY department;
-- ERROR: column "first_name" must appear in GROUP BY

-- ✅ صح: كل الأعمدة غير الـ aggregate في GROUP BY
SELECT department, COUNT(*)
FROM employees
GROUP BY department;

-- ✅ أو استخدم aggregate على العمود
SELECT department, STRING_AGG(first_name, ', ') AS names, COUNT(*)
FROM employees
GROUP BY department;
```

<div dir="rtl">

### 2. استخدام Aggregate في WHERE

</div>

```sql
-- ❌ خطأ: لا يمكن استخدام COUNT في WHERE
SELECT department, COUNT(*)
FROM employees
WHERE COUNT(*) > 2
GROUP BY department;
-- ERROR: aggregate functions are not allowed in WHERE

-- ✅ صح: استخدم HAVING
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

<div dir="rtl">

### 3. الخلط بين WHERE و HAVING

</div>

```sql
-- ❌ مش مثالي: شرط على صف في HAVING
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING department = 'Engineering';  -- يشتغل بس مش المكان المناسب

-- ✅ أفضل: WHERE للصفوف الفردية
SELECT department, COUNT(*)
FROM employees
WHERE department = 'Engineering'
GROUP BY department;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم Aliases واضحة

</div>

```sql
-- ✅ واضح
SELECT
    department AS dept,
    COUNT(*) AS employee_count,
    ROUND(AVG(salary), 2) AS average_salary
FROM employees
GROUP BY department;

-- ❌ مش واضح
SELECT department, COUNT(*), AVG(salary)
FROM employees
GROUP BY department;
```

<div dir="rtl">

### 2. WHERE قبل HAVING

</div>

```sql
-- ✅ فلتر الصفوف بـ WHERE أولاً (أسرع)
SELECT department, AVG(salary)
FROM employees
WHERE is_active = TRUE
GROUP BY department
HAVING AVG(salary) > 70000;

-- ❌ لا تستخدم HAVING لشروط الصفوف
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000; -- بس!
-- الـ inactive employees هيتحسبوا
```

<div dir="rtl">

### 3. Indexes للـ GROUP BY

</div>

```sql
-- أنشئ Index للأعمدة اللي بتجمّع بيها كتير
CREATE INDEX idx_employees_department ON employees(department);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Index مركب لو بتجمّع بأكتر من عمود
CREATE INDEX idx_orders_date_user ON orders(order_date, user_id);
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **GROUP BY** يجمّع الصفوف المتشابهة
2. **كل عمود** غير aggregate لازم يكون في GROUP BY
3. **WHERE** للصفوف الفردية (قبل التجميع)
4. **HAVING** للمجموعات (بعد التجميع)
5. **ROLLUP** للـ subtotals، **CUBE** لكل التجميعات
6. **فلتر بـ WHERE الأول** لأداء أفضل

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Module 03: CRUD Operations](../../03-crud-operations/README.md)**

</div>

---

<div align="center">

[⬅️ السابق: Aggregate Functions](./13-aggregate-functions.md) | [🏠 العودة للـ Module](../README.md)

</div>
