# SELECT المتقدم - Subqueries و CTEs 🔍

<div dir="rtl">

## مقدمة

في الدرس ده هنتعلم Subqueries و Common Table Expressions (CTEs) - أدوات قوية جداً لكتابة queries معقدة بطريقة منظمة وقابلة للقراءة.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 Subqueries

<div dir="rtl">

### ما هي Subquery؟

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│  Subquery = Query داخل Query                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SELECT * FROM orders                                               │
│  WHERE user_id IN (                                                 │
│      SELECT id FROM users WHERE is_active = TRUE  ← Subquery       │
│  );                                                                 │
│                                                                      │
│  أنواع Subqueries:                                                  │
│  ├── Scalar: ترجع قيمة واحدة                                       │
│  ├── Row: ترجع صف واحد                                              │
│  ├── Table: ترجع عدة صفوف                                          │
│  └── Correlated: مرتبطة بالـ outer query                           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Scalar Subquery

<div dir="rtl">

### ترجع قيمة واحدة

</div>

```sql
-- في SELECT: إضافة قيمة محسوبة
SELECT
    first_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS company_avg,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;

-- في WHERE: مقارنة بقيمة
SELECT *
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- مقارنة بـ MAX/MIN
SELECT *
FROM products
WHERE price = (SELECT MAX(price) FROM products);

-- في UPDATE
UPDATE employees
SET salary = salary * 1.10
WHERE salary < (SELECT AVG(salary) FROM employees);
```

---

## 📋 Table Subquery

<div dir="rtl">

### ترجع عدة صفوف

</div>

```sql
-- IN: قيمة من مجموعة
SELECT *
FROM orders
WHERE user_id IN (
    SELECT id FROM users WHERE role = 'premium'
);

-- NOT IN
SELECT *
FROM products
WHERE category_id NOT IN (
    SELECT id FROM categories WHERE is_active = FALSE
);

-- ANY/SOME: أي قيمة تحقق الشرط
SELECT *
FROM employees
WHERE salary > ANY (
    SELECT salary FROM employees WHERE department = 'HR'
);
-- الراتب أكبر من أي راتب في HR (يعني أكبر من أقل راتب)

-- ALL: كل القيم تحقق الشرط
SELECT *
FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE department = 'HR'
);
-- الراتب أكبر من كل رواتب HR (يعني أكبر من أعلى راتب)
```

---

## 🔗 Correlated Subquery

<div dir="rtl">

### مرتبطة بالـ Outer Query

</div>

```sql
-- متوسط راتب القسم لكل موظف
SELECT
    e.first_name,
    e.department,
    e.salary,
    (SELECT AVG(salary)
     FROM employees
     WHERE department = e.department) AS dept_avg
FROM employees e;

-- الموظفين اللي راتبهم أعلى من متوسط قسمهم
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department = e.department
);

-- أحدث طلب لكل مستخدم
SELECT *
FROM orders o
WHERE created_at = (
    SELECT MAX(created_at)
    FROM orders
    WHERE user_id = o.user_id
);
```

---

## ✅ EXISTS و NOT EXISTS

```sql
-- EXISTS: هل يوجد صفوف؟
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id
);
-- المستخدمين اللي عندهم طلبات

-- NOT EXISTS
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id
);
-- المستخدمين اللي معندهمش طلبات

-- EXISTS أسرع من IN في كتير من الحالات
-- لأنه بيوقف أول ما يلاقي صف
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IN vs EXISTS                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  IN:                                                                │
│  - بينفذ الـ subquery كلها أولاً                                    │
│  - بيبني list من القيم                                              │
│  - ⚠️ NULL handling ممكن يسبب مشاكل                                │
│                                                                      │
│  EXISTS:                                                            │
│  - بيوقف أول ما يلاقي match                                        │
│  - أسرع للـ correlated subqueries                                   │
│  - مش بيتأثر بـ NULL                                                │
│                                                                      │
│  القاعدة العامة:                                                     │
│  - استخدم IN للـ small, static lists                                │
│  - استخدم EXISTS للـ correlated subqueries                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Subquery في FROM (Derived Table)

```sql
-- Subquery كجدول
SELECT
    dept_stats.department,
    dept_stats.avg_salary,
    dept_stats.employee_count
FROM (
    SELECT
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE dept_stats.employee_count >= 5;

-- استخدام عملي
SELECT
    u.username,
    order_summary.total_orders,
    order_summary.total_spent
FROM users u
JOIN (
    SELECT
        user_id,
        COUNT(*) AS total_orders,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
) AS order_summary ON u.id = order_summary.user_id
WHERE order_summary.total_spent > 1000;
```

---

## 🏗️ Common Table Expressions (CTEs)

<div dir="rtl">

### ما هي CTE؟

</div>

```sql
-- CTE = Subquery مسماة، أسهل في القراءة
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

<div dir="rtl">

### CTE بسيطة

</div>

```sql
-- بدل derived table
WITH dept_stats AS (
    SELECT
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department
)
SELECT *
FROM dept_stats
WHERE employee_count >= 5;

-- أسهل في القراءة من:
SELECT * FROM (SELECT department, AVG(salary)... GROUP BY department) AS dept_stats WHERE ...;
```

---

## 📊 CTEs متعددة

```sql
-- عدة CTEs
WITH
-- CTE 1: المستخدمين النشطين
active_users AS (
    SELECT id, username, email
    FROM users
    WHERE is_active = TRUE
),
-- CTE 2: إحصائيات الطلبات
order_stats AS (
    SELECT
        user_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
),
-- CTE 3: دمج البيانات
user_summary AS (
    SELECT
        u.id,
        u.username,
        u.email,
        COALESCE(os.order_count, 0) AS orders,
        COALESCE(os.total_spent, 0) AS spent
    FROM active_users u
    LEFT JOIN order_stats os ON u.id = os.user_id
)
-- الـ Query الرئيسي
SELECT *
FROM user_summary
WHERE orders >= 5
ORDER BY spent DESC;
```

---

## 🔄 Recursive CTEs

<div dir="rtl">

### للبيانات الهرمية

</div>

```sql
-- جدول الموظفين مع المدير
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT REFERENCES employees(id)
);

-- استعلام recursive: كل المرؤوسين
WITH RECURSIVE subordinates AS (
    -- Base case: المدير
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE id = 1  -- CEO

    UNION ALL

    -- Recursive case: المرؤوسين
    SELECT e.id, e.name, e.manager_id, s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

<div dir="rtl">

### مثال: تسلسل الـ Categories

</div>

```sql
-- جدول categories هرمي
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    parent_id INT REFERENCES categories(id)
);

-- كل الـ subcategories
WITH RECURSIVE category_tree AS (
    -- Base: الفئة الرئيسية
    SELECT id, name, parent_id, name::TEXT AS path, 0 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive: الفئات الفرعية
    SELECT c.id, c.name, c.parent_id,
           ct.path || ' > ' || c.name,
           ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree
ORDER BY path;

-- النتيجة:
-- Electronics
-- Electronics > Phones
-- Electronics > Phones > iPhones
-- Electronics > Phones > Android
-- Electronics > Laptops
```

---

## 🎯 أمثلة عملية

<div dir="rtl">

### 1. أفضل منتج في كل فئة

</div>

```sql
WITH product_rankings AS (
    SELECT
        p.id,
        p.name,
        p.price,
        c.name AS category,
        RANK() OVER (PARTITION BY p.category_id ORDER BY p.price DESC) AS rank
    FROM products p
    JOIN categories c ON p.category_id = c.id
)
SELECT id, name, price, category
FROM product_rankings
WHERE rank = 1;
```

<div dir="rtl">

### 2. تقرير مقارنة الشهور

</div>

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', created_at)::DATE AS month,
        SUM(total_amount) AS revenue,
        COUNT(*) AS orders
    FROM orders
    WHERE status = 'completed'
    GROUP BY DATE_TRUNC('month', created_at)
),
with_previous AS (
    SELECT
        month,
        revenue,
        orders,
        LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
        LAG(orders) OVER (ORDER BY month) AS prev_orders
    FROM monthly_sales
)
SELECT
    month,
    revenue,
    orders,
    revenue - COALESCE(prev_revenue, 0) AS revenue_change,
    CASE
        WHEN prev_revenue > 0 THEN
            ROUND(((revenue - prev_revenue) / prev_revenue * 100)::NUMERIC, 2)
        ELSE 0
    END AS revenue_growth_pct
FROM with_previous
ORDER BY month;
```

<div dir="rtl">

### 3. المستخدمين المتكررين

</div>

```sql
WITH user_activity AS (
    SELECT
        user_id,
        DATE(created_at) AS order_date
    FROM orders
),
consecutive_days AS (
    SELECT
        user_id,
        order_date,
        order_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date)::INT AS grp
    FROM user_activity
),
streaks AS (
    SELECT
        user_id,
        MIN(order_date) AS streak_start,
        MAX(order_date) AS streak_end,
        COUNT(*) AS streak_length
    FROM consecutive_days
    GROUP BY user_id, grp
)
SELECT
    u.username,
    s.streak_start,
    s.streak_end,
    s.streak_length
FROM streaks s
JOIN users u ON s.user_id = u.id
WHERE s.streak_length >= 3
ORDER BY s.streak_length DESC;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم CTEs للقراءة

</div>

```sql
-- ❌ صعب القراءة
SELECT * FROM (SELECT * FROM (SELECT ...)) AS a JOIN (SELECT ...) AS b ...

-- ✅ سهل القراءة
WITH a AS (...), b AS (...)
SELECT * FROM a JOIN b ...
```

<div dir="rtl">

### 2. EXISTS أسرع من IN للـ Correlated

</div>

```sql
-- ✅ أسرع
WHERE EXISTS (SELECT 1 FROM related WHERE id = outer.id)

-- ⚠️ أبطأ في بعض الحالات
WHERE id IN (SELECT id FROM related)
```

<div dir="rtl">

### 3. تجنب الـ Correlated Subqueries في SELECT الكبيرة

</div>

```sql
-- ❌ بطيء (N+1 queries)
SELECT e.name, (SELECT AVG(salary) FROM employees WHERE dept = e.dept)
FROM employees e;

-- ✅ أسرع (JOIN)
SELECT e.name, d.avg_salary
FROM employees e
JOIN (SELECT dept, AVG(salary) AS avg_salary FROM employees GROUP BY dept) d
ON e.dept = d.dept;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Scalar Subquery** ترجع قيمة واحدة
2. **EXISTS** أسرع من IN للـ correlated queries
3. **CTEs** تخلي الـ queries أسهل في القراءة
4. **Recursive CTEs** للبيانات الهرمية
5. تجنب **Correlated Subqueries** في SELECT الكبيرة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Window Functions](./07-window-functions.md)**

</div>

---

<div align="center">

[⬅️ السابق: COPY](./05-copy-command.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./07-window-functions.md)

</div>
