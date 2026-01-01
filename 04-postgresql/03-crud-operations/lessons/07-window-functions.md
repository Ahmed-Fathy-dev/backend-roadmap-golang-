# Window Functions - دوال النوافذ 🪟

<div dir="rtl">

## مقدمة

Window Functions من أقوى الأدوات في SQL - بتسمحلك تعمل حسابات على مجموعة صفوف مرتبطة بالصف الحالي، من غير ما تدمج الصفوف زي GROUP BY.

**المدة المتوقعة:** 35 دقيقة

</div>

---

## 📊 ما هي Window Function؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                Window Function vs Aggregate                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Aggregate (GROUP BY):                                              │
│  ┌──────────────────────┐                                           │
│  │  Name    │ Dept │Sal │        SELECT dept, AVG(salary)          │
│  ├──────────┼──────┼────┤        FROM employees                     │
│  │  Ahmed   │ Eng  │70k │        GROUP BY dept;                     │
│  │  Sara    │ Eng  │80k │                                           │
│  │  Omar    │ HR   │60k │   →    ┌──────┬─────────┐                 │
│  │  Fatima  │ HR   │55k │        │ Dept │  AVG    │                 │
│  └──────────┴──────┴────┘        ├──────┼─────────┤                 │
│       4 صفوف                      │ Eng  │  75k    │                 │
│                                   │ HR   │  57.5k  │                 │
│                                   └──────┴─────────┘                 │
│                                        2 صفوف                        │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│  Window Function (OVER):                                            │
│  ┌──────────────────────┐                                           │
│  │  Name    │ Dept │Sal │        SELECT name, dept, salary,         │
│  ├──────────┼──────┼────┤          AVG(salary) OVER (PARTITION BY   │
│  │  Ahmed   │ Eng  │70k │            dept) AS dept_avg              │
│  │  Sara    │ Eng  │80k │        FROM employees;                    │
│  │  Omar    │ HR   │60k │                                           │
│  │  Fatima  │ HR   │55k │   →    ┌───────┬────┬─────┬────────┐     │
│  └──────────┴──────┴────┘        │ Name  │Dept│ Sal │dept_avg│     │
│       4 صفوف                      ├───────┼────┼─────┼────────┤     │
│                                   │ Ahmed │Eng │ 70k │  75k   │     │
│                                   │ Sara  │Eng │ 80k │  75k   │     │
│                                   │ Omar  │ HR │ 60k │ 57.5k  │     │
│                                   │Fatima │ HR │ 55k │ 57.5k  │     │
│                                   └───────┴────┴─────┴────────┘     │
│                                        4 صفوف (زي ما هي!)           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة الأساسية

```sql
function_name() OVER (
    [PARTITION BY column1, column2, ...]
    [ORDER BY column3 [ASC|DESC], ...]
    [frame_clause]
)
```

---

## 🔢 Ranking Functions

<div dir="rtl">

### ROW_NUMBER()

</div>

```sql
-- رقم تسلسلي فريد
SELECT
    first_name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- ترتيب داخل كل قسم
SELECT
    first_name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS dept_rank
FROM employees;

-- النتيجة:
-- first_name | department  | salary | dept_rank
-- Khaled     | Engineering | 90000  | 1
-- Omar       | Engineering | 80000  | 2
-- Ahmed      | Engineering | 75000  | 3
-- Youssef    | Finance     | 85000  | 1
-- Hassan     | Finance     | 70000  | 2
```

<div dir="rtl">

### RANK() و DENSE_RANK()

</div>

```sql
-- RANK: بيسيب فجوات
-- DENSE_RANK: مش بيسيب فجوات
SELECT
    first_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- النتيجة (لو فيه رواتب متساوية):
-- first_name | salary | rank | dense_rank
-- Khaled     | 90000  |   1  |     1
-- Ahmed      | 80000  |   2  |     2
-- Omar       | 80000  |   2  |     2      ← نفس الترتيب
-- Sara       | 75000  |   4  |     3      ← RANK فات 3، DENSE_RANK لأ
```

<div dir="rtl">

### NTILE()

</div>

```sql
-- تقسيم لمجموعات
SELECT
    first_name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;

-- النتيجة: تقسيم لـ 4 مجموعات (Top 25%, Next 25%, ...)
-- first_name | salary | quartile
-- Khaled     | 90000  |    1     ← Top 25%
-- Omar       | 80000  |    1
-- Ahmed      | 75000  |    2
-- Sara       | 65000  |    2
-- Hassan     | 60000  |    3
-- Fatima     | 55000  |    3
-- Mona       | 52000  |    4
-- Nour       | 50000  |    4
```

---

## 📊 Aggregate Window Functions

```sql
-- SUM, AVG, COUNT, MIN, MAX كـ window functions
SELECT
    first_name,
    department,
    salary,
    SUM(salary) OVER (PARTITION BY department) AS dept_total,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    COUNT(*) OVER (PARTITION BY department) AS dept_count,
    MIN(salary) OVER (PARTITION BY department) AS dept_min,
    MAX(salary) OVER (PARTITION BY department) AS dept_max
FROM employees;

-- مقارنة بالمتوسط
SELECT
    first_name,
    salary,
    AVG(salary) OVER () AS company_avg,
    salary - AVG(salary) OVER () AS diff_from_avg,
    ROUND(salary / AVG(salary) OVER () * 100, 2) AS pct_of_avg
FROM employees;
```

---

## 📈 Running Totals و Moving Averages

<div dir="rtl">

### Running Total

</div>

```sql
-- المجموع التراكمي
SELECT
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- النتيجة:
-- order_date | total_amount | running_total
-- 2024-01-01 |    100.00    |    100.00
-- 2024-01-02 |    150.00    |    250.00
-- 2024-01-03 |     75.00    |    325.00
-- 2024-01-04 |    200.00    |    525.00

-- Running total لكل user
SELECT
    user_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY user_id
        ORDER BY order_date
    ) AS user_running_total
FROM orders;
```

<div dir="rtl">

### Moving Average

</div>

```sql
-- متوسط آخر 3 قيم
SELECT
    order_date,
    total_amount,
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM orders;

-- متوسط آخر 7 أيام
SELECT
    order_date,
    daily_revenue,
    AVG(daily_revenue) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
    ) AS weekly_avg
FROM daily_sales;
```

---

## ⏮️ LAG و LEAD

<div dir="rtl">

### LAG: القيمة السابقة

</div>

```sql
-- مقارنة بالشهر السابق
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    revenue - LAG(revenue) OVER (ORDER BY month) AS change
FROM monthly_sales;

-- النتيجة:
-- month   | revenue | prev_month | change
-- 2024-01 | 10000   | NULL       | NULL
-- 2024-02 | 12000   | 10000      | 2000
-- 2024-03 | 11500   | 12000      | -500

-- LAG مع قيمة افتراضية
SELECT
    month,
    revenue,
    LAG(revenue, 1, 0) OVER (ORDER BY month) AS prev_month
FROM monthly_sales;
-- لو مفيش قيمة سابقة، يرجع 0
```

<div dir="rtl">

### LEAD: القيمة التالية

</div>

```sql
-- مقارنة بالشهر التالي
SELECT
    month,
    revenue,
    LEAD(revenue) OVER (ORDER BY month) AS next_month,
    LEAD(revenue) OVER (ORDER BY month) - revenue AS expected_change
FROM monthly_sales;

-- LEAD بخطوات أكتر
SELECT
    order_date,
    total_amount,
    LEAD(total_amount, 2) OVER (ORDER BY order_date) AS amount_in_2_days
FROM orders;
```

---

## 🪟 Frame Clause

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Frame Clause                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ROWS BETWEEN start AND end                                         │
│                                                                      │
│  Options:                                                           │
│  ├── UNBOUNDED PRECEDING  - من أول صف                              │
│  ├── n PRECEDING          - n صفوف قبل الحالي                       │
│  ├── CURRENT ROW          - الصف الحالي                            │
│  ├── n FOLLOWING          - n صفوف بعد الحالي                       │
│  └── UNBOUNDED FOLLOWING  - لآخر صف                                 │
│                                                                      │
│  Examples:                                                          │
│  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  ← Running total  │
│  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW          ← آخر 3 صفوف    │
│  ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING          ← 3 صفوف محيطة  │
│  ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING  ← من هنا للنهاية│
└─────────────────────────────────────────────────────────────────────┘
```

```sql
-- أمثلة Frame Clause
SELECT
    order_date,
    total_amount,
    -- من البداية للحالي
    SUM(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    -- آخر 3 صفوف
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3,
    -- الصف الحالي والتاليين
    SUM(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
    ) AS next_3_sum
FROM orders;
```

---

## 🎯 FIRST_VALUE و LAST_VALUE

```sql
-- أول وآخر قيمة في الـ window
SELECT
    first_name,
    department,
    salary,
    FIRST_VALUE(first_name) OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS highest_paid,
    LAST_VALUE(first_name) OVER (
        PARTITION BY department
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid
FROM employees;

-- ⚠️ ملحوظة: LAST_VALUE تحتاج frame clause صريح
-- لأن الـ default frame هو UNBOUNDED PRECEDING TO CURRENT ROW
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. تقرير النمو الشهري

</div>

```sql
WITH monthly_data AS (
    SELECT
        DATE_TRUNC('month', created_at)::DATE AS month,
        SUM(total_amount) AS revenue,
        COUNT(*) AS orders
    FROM orders
    WHERE status = 'completed'
    GROUP BY DATE_TRUNC('month', created_at)
)
SELECT
    month,
    revenue,
    orders,
    LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
    ROUND(
        (revenue - LAG(revenue) OVER (ORDER BY month)) /
        NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100,
        2
    ) AS growth_pct,
    SUM(revenue) OVER (ORDER BY month) AS ytd_revenue
FROM monthly_data
ORDER BY month;
```

<div dir="rtl">

### 2. Top 3 في كل فئة

</div>

```sql
WITH ranked_products AS (
    SELECT
        p.name,
        c.name AS category,
        SUM(oi.quantity) AS total_sold,
        ROW_NUMBER() OVER (
            PARTITION BY p.category_id
            ORDER BY SUM(oi.quantity) DESC
        ) AS rank
    FROM products p
    JOIN categories c ON p.category_id = c.id
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.id, p.name, c.id, c.name
)
SELECT name, category, total_sold, rank
FROM ranked_products
WHERE rank <= 3
ORDER BY category, rank;
```

<div dir="rtl">

### 3. تحليل سلوك العملاء

</div>

```sql
SELECT
    user_id,
    order_date,
    total_amount,
    -- الطلب السابق
    LAG(order_date) OVER (PARTITION BY user_id ORDER BY order_date) AS prev_order,
    -- الأيام بين الطلبات
    order_date - LAG(order_date) OVER (
        PARTITION BY user_id ORDER BY order_date
    ) AS days_between,
    -- رقم الطلب للعميل
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date) AS order_number,
    -- إجمالي العميل التراكمي
    SUM(total_amount) OVER (
        PARTITION BY user_id
        ORDER BY order_date
    ) AS cumulative_total,
    -- متوسط الطلبات للعميل
    AVG(total_amount) OVER (PARTITION BY user_id) AS customer_avg
FROM orders
WHERE status = 'completed'
ORDER BY user_id, order_date;
```

<div dir="rtl">

### 4. الفرق عن المتوسط

</div>

```sql
SELECT
    first_name,
    department,
    salary,
    AVG(salary) OVER () AS company_avg,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER () AS diff_company,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_dept,
    CASE
        WHEN salary > AVG(salary) OVER (PARTITION BY department) * 1.2
        THEN 'Above Average'
        WHEN salary < AVG(salary) OVER (PARTITION BY department) * 0.8
        THEN 'Below Average'
        ELSE 'Average'
    END AS salary_band
FROM employees
ORDER BY department, salary DESC;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم Named Windows

</div>

```sql
-- ✅ أوضح
SELECT
    first_name,
    salary,
    SUM(salary) OVER w AS running_total,
    AVG(salary) OVER w AS running_avg
FROM employees
WINDOW w AS (ORDER BY hire_date);

-- بدل تكرار OVER clause
```

<div dir="rtl">

### 2. فكر في الـ Frame

</div>

```sql
-- Default frame مع ORDER BY:
-- RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- ⚠️ LAST_VALUE محتاج frame صريح
LAST_VALUE(x) OVER (
    ORDER BY y
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

<div dir="rtl">

### 3. استخدم CTE للوضوح

</div>

```sql
-- ✅ سهل القراءة
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (...) AS rn
    FROM table
)
SELECT * FROM ranked WHERE rn = 1;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Window Functions** تحافظ على كل الصفوف (عكس GROUP BY)
2. **ROW_NUMBER, RANK, DENSE_RANK** للترتيب
3. **LAG, LEAD** للقيم السابقة/التالية
4. **Frame Clause** للتحكم في نطاق الحساب
5. **Named Windows** لتجنب التكرار

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [UPDATE الأساسي](./08-basic-update.md)**

</div>

---

<div align="center">

[⬅️ السابق: SELECT المتقدم](./06-advanced-select.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./08-basic-update.md)

</div>
