# SELECT - قراءة البيانات 🔍

<div dir="rtl">

## مقدمة

`SELECT` هو أكثر أمر SQL استخداماً. بيسمحلك تقرأ وتستعلم عن البيانات بأي طريقة تحتاجها. في الدرس ده هنتعلم أساسيات SELECT بالتفصيل.

**المدة المتوقعة:** 25-30 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
SELECT column1, column2, ...
FROM table_name;
```

---

## 🗃️ بيانات للتجربة

```sql
-- إنشاء وتعبئة جدول للتجارب
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    department VARCHAR(50),
    salary NUMERIC(10, 2),
    hire_date DATE,
    is_active BOOLEAN DEFAULT TRUE
);

INSERT INTO employees (first_name, last_name, email, department, salary, hire_date, is_active) VALUES
    ('Ahmed', 'Ali', 'ahmed@company.com', 'Engineering', 75000, '2020-01-15', TRUE),
    ('Sara', 'Mohamed', 'sara@company.com', 'Marketing', 65000, '2019-06-01', TRUE),
    ('Omar', 'Hassan', 'omar@company.com', 'Engineering', 80000, '2018-03-20', TRUE),
    ('Fatima', 'Ibrahim', 'fatima@company.com', 'HR', 55000, '2021-09-10', TRUE),
    ('Khaled', 'Mahmoud', 'khaled@company.com', 'Engineering', 90000, '2017-11-30', TRUE),
    ('Nour', 'Ahmed', 'nour@company.com', 'Marketing', 60000, '2022-02-14', TRUE),
    ('Hassan', 'Ali', 'hassan@company.com', 'Finance', 70000, '2020-07-22', FALSE),
    ('Mona', 'Saeed', 'mona@company.com', 'HR', 52000, '2023-01-05', TRUE),
    ('Youssef', 'Kamal', 'youssef@company.com', 'Finance', 85000, '2019-04-18', TRUE),
    ('Layla', 'Omar', 'layla@company.com', 'Engineering', 78000, '2021-08-25', TRUE);
```

---

## ⭐ SELECT الأساسي

<div dir="rtl">

### اختيار كل الأعمدة

</div>

```sql
-- كل الأعمدة وكل الصفوف
SELECT * FROM employees;

-- ⚠️ تجنب * في Production code
-- استخدمها للاستكشاف فقط
```

<div dir="rtl">

### اختيار أعمدة محددة

</div>

```sql
-- أعمدة محددة
SELECT first_name, last_name, email FROM employees;

-- بترتيب مختلف
SELECT email, first_name, department FROM employees;
```

---

## 🏷️ Aliases (الأسماء البديلة)

<div dir="rtl">

### Column Aliases

</div>

```sql
-- باستخدام AS
SELECT
    first_name AS "First Name",
    last_name AS "Last Name",
    salary AS "Annual Salary"
FROM employees;

-- بدون AS (يشتغل بس مش واضح)
SELECT
    first_name "First Name",
    salary monthly_salary
FROM employees;

-- للحسابات
SELECT
    first_name,
    salary AS annual_salary,
    salary / 12 AS monthly_salary,
    salary * 0.10 AS bonus
FROM employees;
```

<div dir="rtl">

### Table Aliases

</div>

```sql
-- مفيد في الـ JOINs
SELECT e.first_name, e.last_name, e.department
FROM employees AS e;

-- بدون AS
SELECT e.first_name, e.salary
FROM employees e;
```

---

## 🧮 التعبيرات والحسابات

```sql
-- عمليات حسابية
SELECT
    first_name,
    salary,
    salary + 5000 AS with_raise,
    salary * 1.10 AS ten_percent_raise,
    salary * 12 AS annual_salary
FROM employees;

-- دمج النصوص
SELECT
    first_name || ' ' || last_name AS full_name,
    CONCAT(first_name, ' ', last_name) AS full_name_2
FROM employees;

-- مع formatting
SELECT
    first_name || ' ' || last_name AS name,
    '$' || TO_CHAR(salary, 'FM999,999.00') AS formatted_salary
FROM employees;
```

---

## 🎯 DISTINCT - إزالة التكرار

```sql
-- القيم الفريدة لعمود واحد
SELECT DISTINCT department FROM employees;
-- Result: Engineering, Marketing, HR, Finance

-- القيم الفريدة لعدة أعمدة
SELECT DISTINCT department, is_active FROM employees;

-- عدد القيم الفريدة
SELECT COUNT(DISTINCT department) AS unique_departments FROM employees;
-- Result: 4

-- DISTINCT ON (PostgreSQL specific)
-- أول صف من كل مجموعة
SELECT DISTINCT ON (department)
    department, first_name, salary
FROM employees
ORDER BY department, salary DESC;
-- هيرجع أعلى راتب في كل department
```

---

## 📊 Functions في SELECT

<div dir="rtl">

### String Functions

</div>

```sql
SELECT
    first_name,
    UPPER(first_name) AS upper_name,
    LOWER(first_name) AS lower_name,
    LENGTH(first_name) AS name_length,
    LEFT(email, 5) AS email_prefix,
    SUBSTRING(email FROM 1 FOR POSITION('@' IN email) - 1) AS email_username
FROM employees;
```

<div dir="rtl">

### Numeric Functions

</div>

```sql
SELECT
    first_name,
    salary,
    ROUND(salary / 12, 2) AS monthly,
    CEIL(salary / 12) AS monthly_ceil,
    FLOOR(salary / 12) AS monthly_floor,
    ABS(salary - 70000) AS diff_from_70k
FROM employees;
```

<div dir="rtl">

### Date Functions

</div>

```sql
SELECT
    first_name,
    hire_date,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month,
    AGE(hire_date) AS tenure,
    DATE_PART('year', AGE(hire_date)) AS years_employed,
    hire_date + INTERVAL '1 year' AS one_year_anniversary
FROM employees;
```

---

## ❓ CASE Expressions

<div dir="rtl">

### Simple CASE

</div>

```sql
SELECT
    first_name,
    department,
    CASE department
        WHEN 'Engineering' THEN 'Tech'
        WHEN 'Marketing' THEN 'Business'
        WHEN 'HR' THEN 'People'
        WHEN 'Finance' THEN 'Business'
        ELSE 'Other'
    END AS department_category
FROM employees;
```

<div dir="rtl">

### Searched CASE

</div>

```sql
SELECT
    first_name,
    salary,
    CASE
        WHEN salary >= 80000 THEN 'Senior'
        WHEN salary >= 60000 THEN 'Mid-Level'
        WHEN salary >= 50000 THEN 'Junior'
        ELSE 'Entry'
    END AS salary_level
FROM employees;

-- مع عدة شروط
SELECT
    first_name,
    salary,
    hire_date,
    CASE
        WHEN salary >= 80000 AND hire_date < '2019-01-01' THEN 'Senior Veteran'
        WHEN salary >= 80000 THEN 'Senior'
        WHEN salary >= 60000 THEN 'Mid-Level'
        ELSE 'Junior'
    END AS employee_class
FROM employees;
```

---

## 🔄 COALESCE و NULLIF

<div dir="rtl">

### COALESCE - أول قيمة غير NULL

</div>

```sql
-- إرجاع أول قيمة غير NULL
SELECT
    first_name,
    COALESCE(department, 'Unassigned') AS department
FROM employees;

-- مع عدة خيارات
SELECT
    COALESCE(phone, mobile, email, 'No Contact') AS contact_info
FROM contacts;

-- مفيد في الحسابات
SELECT
    product_name,
    price * COALESCE(discount, 0) AS discount_amount
FROM products;
```

<div dir="rtl">

### NULLIF - إرجاع NULL لو متساويين

</div>

```sql
-- تجنب القسمة على صفر
SELECT
    name,
    total / NULLIF(count, 0) AS average
FROM stats;

-- NULLIF(count, 0): لو count = 0، ترجع NULL
-- total / NULL = NULL (بدل error)
```

---

## 📝 Subqueries في SELECT

```sql
-- Scalar subquery (قيمة واحدة)
SELECT
    first_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_salary,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;

-- Subquery مع IN
SELECT first_name, department
FROM employees
WHERE department IN (
    SELECT DISTINCT department
    FROM employees
    WHERE salary > 70000
);

-- Correlated subquery
SELECT
    e.first_name,
    e.department,
    e.salary,
    (SELECT AVG(salary) FROM employees WHERE department = e.department) AS dept_avg
FROM employees e;
```

---

## 🪟 Window Functions Preview

<div dir="rtl">

(هنشرحها بالتفصيل في Advanced Topics)

</div>

```sql
-- ترتيب داخل كل department
SELECT
    first_name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Running total
SELECT
    first_name,
    salary,
    SUM(salary) OVER (ORDER BY hire_date) AS running_total
FROM employees;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. تجنب SELECT *

</div>

```sql
-- ❌ وحش (في Production)
SELECT * FROM employees;

-- ✅ كويس
SELECT id, first_name, last_name, email FROM employees;
```

<div dir="rtl">

### 2. استخدم Aliases واضحة

</div>

```sql
-- ❌ غير واضح
SELECT a, b, c FROM t;

-- ✅ واضح
SELECT
    first_name AS name,
    salary * 12 AS annual_salary
FROM employees;
```

<div dir="rtl">

### 3. تنسيق الـ Query

</div>

```sql
-- ❌ صعب القراءة
SELECT first_name,last_name,salary,department FROM employees WHERE is_active=true ORDER BY salary DESC;

-- ✅ سهل القراءة
SELECT
    first_name,
    last_name,
    salary,
    department
FROM employees
WHERE is_active = true
ORDER BY salary DESC;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **SELECT** هو أساس قراءة البيانات
2. **تجنب SELECT *** في Production
3. **Aliases** لتسمية واضحة
4. **DISTINCT** لإزالة التكرار
5. **CASE** للـ conditional logic
6. **COALESCE** للتعامل مع NULL

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [WHERE - تصفية البيانات](./11-where.md)**

</div>

---

<div align="center">

[⬅️ السابق: INSERT](./09-insert.md) | [🏠 العودة للـ Module](../README.md)

</div>
