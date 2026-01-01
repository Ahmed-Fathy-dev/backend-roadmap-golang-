# Aggregate Functions - دوال التجميع 📈

<div dir="rtl">

## مقدمة

Aggregate Functions بتاخد مجموعة قيم وترجع قيمة واحدة. مهمة جداً للتقارير والإحصائيات - زي حساب المجموع، المتوسط، العدد، وغيرها.

**المدة المتوقعة:** 20-25 دقيقة

</div>

---

## 📊 الدوال الأساسية

```
┌─────────────────────────────────────────────────────────────────┐
│                    Aggregate Functions                           │
├─────────────────────────────────────────────────────────────────┤
│  COUNT()   ─── عدد الصفوف                                       │
│  SUM()     ─── مجموع القيم                                      │
│  AVG()     ─── المتوسط                                          │
│  MIN()     ─── أصغر قيمة                                        │
│  MAX()     ─── أكبر قيمة                                        │
├─────────────────────────────────────────────────────────────────┤
│  STRING_AGG()  ─── دمج النصوص                                   │
│  ARRAY_AGG()   ─── تجميع في Array                               │
│  JSON_AGG()    ─── تجميع في JSON                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 COUNT - العدد

```sql
-- عدد كل الصفوف
SELECT COUNT(*) FROM employees;
-- Result: 10

-- عدد الصفوف اللي فيها قيمة (مش NULL)
SELECT COUNT(department) FROM employees;
-- لو فيه 2 NULL، هيرجع 8

-- عدد القيم الفريدة
SELECT COUNT(DISTINCT department) FROM employees;
-- Result: 4 (Engineering, Marketing, HR, Finance)

-- عدد مع شرط
SELECT COUNT(*) FROM employees WHERE is_active = TRUE;

-- عدة counts في query واحد
SELECT
    COUNT(*) AS total_employees,
    COUNT(DISTINCT department) AS unique_departments,
    COUNT(*) FILTER (WHERE is_active) AS active_count,
    COUNT(*) FILTER (WHERE salary > 70000) AS high_salary_count
FROM employees;
```

---

## ➕ SUM - المجموع

```sql
-- مجموع الرواتب
SELECT SUM(salary) FROM employees;

-- مجموع مع شرط
SELECT SUM(salary) FROM employees WHERE department = 'Engineering';

-- مجموع قيم فريدة (نادر الاستخدام)
SELECT SUM(DISTINCT salary) FROM employees;

-- مجموعات متعددة
SELECT
    SUM(salary) AS total_salaries,
    SUM(salary) FILTER (WHERE department = 'Engineering') AS engineering_total,
    SUM(salary) FILTER (WHERE department = 'Marketing') AS marketing_total
FROM employees;

-- مع حساب
SELECT SUM(quantity * unit_price) AS total_revenue FROM order_items;
```

---

## 📐 AVG - المتوسط

```sql
-- متوسط الرواتب
SELECT AVG(salary) FROM employees;
-- Result: 71000.0000000000000000

-- تقريب المتوسط
SELECT ROUND(AVG(salary), 2) AS avg_salary FROM employees;
-- Result: 71000.00

-- متوسط مع شرط
SELECT ROUND(AVG(salary), 2)
FROM employees
WHERE hire_date >= '2020-01-01';

-- متوسط قيم فريدة
SELECT AVG(DISTINCT salary) FROM employees;

-- ⚠️ AVG بيتجاهل NULL
-- لو عندك: 100, 200, NULL
-- AVG = (100 + 200) / 2 = 150
-- مش (100 + 200 + 0) / 3
```

---

## ⬇️⬆️ MIN و MAX

```sql
-- أصغر وأكبر راتب
SELECT
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees;

-- أقدم وأحدث تاريخ توظيف
SELECT
    MIN(hire_date) AS first_hire,
    MAX(hire_date) AS last_hire
FROM employees;

-- MIN و MAX للنصوص (أبجدياً)
SELECT
    MIN(first_name) AS first_alphabetically,
    MAX(first_name) AS last_alphabetically
FROM employees;

-- الفرق بين أعلى وأقل
SELECT
    MAX(salary) - MIN(salary) AS salary_range
FROM employees;

-- مع شرط
SELECT
    MIN(salary) AS min_engineering_salary,
    MAX(salary) AS max_engineering_salary
FROM employees
WHERE department = 'Engineering';
```

---

## 🔗 STRING_AGG - دمج النصوص

```sql
-- دمج الأسماء في نص واحد
SELECT STRING_AGG(first_name, ', ') FROM employees;
-- Result: "Ahmed, Sara, Omar, Fatima, ..."

-- مع ترتيب
SELECT STRING_AGG(first_name, ', ' ORDER BY first_name)
FROM employees;
-- Result: "Ahmed, Fatima, Hassan, ..." (مرتب أبجدياً)

-- دمج مع تصفية
SELECT STRING_AGG(first_name, ' | ')
FROM employees
WHERE department = 'Engineering';

-- DISTINCT قبل الدمج
SELECT STRING_AGG(DISTINCT department, ', ' ORDER BY department)
FROM employees;
-- Result: "Engineering, Finance, HR, Marketing"
```

---

## 📚 ARRAY_AGG - تجميع في Array

```sql
-- تجميع في Array
SELECT ARRAY_AGG(first_name) FROM employees;
-- Result: {Ahmed,Sara,Omar,Fatima,...}

-- مع ترتيب
SELECT ARRAY_AGG(first_name ORDER BY salary DESC)
FROM employees;

-- DISTINCT
SELECT ARRAY_AGG(DISTINCT department ORDER BY department)
FROM employees;
-- Result: {Engineering,Finance,HR,Marketing}

-- استخدام مفيد: كل الـ tags لمنتج
SELECT
    product_id,
    ARRAY_AGG(tag_name) AS tags
FROM product_tags
GROUP BY product_id;
```

---

## 📋 JSON_AGG - تجميع في JSON

```sql
-- تجميع كـ JSON Array
SELECT JSON_AGG(first_name) FROM employees;
-- Result: ["Ahmed", "Sara", "Omar", ...]

-- تجميع صفوف كاملة
SELECT JSON_AGG(row_to_json(e)) FROM employees e;
-- Result: [{"id":1,"first_name":"Ahmed",...}, ...]

-- JSONB_AGG (أفضل للتخزين)
SELECT JSONB_AGG(
    jsonb_build_object(
        'name', first_name,
        'salary', salary
    )
)
FROM employees;
```

---

## 📊 دوال إحصائية إضافية

```sql
-- الانحراف المعياري
SELECT
    STDDEV(salary) AS std_dev,
    STDDEV_POP(salary) AS population_std_dev,
    STDDEV_SAMP(salary) AS sample_std_dev
FROM employees;

-- التباين
SELECT
    VARIANCE(salary) AS variance,
    VAR_POP(salary) AS population_variance,
    VAR_SAMP(salary) AS sample_variance
FROM employees;

-- الارتباط
SELECT CORR(salary, years_experience)
FROM employees;

-- الوسيط (PostgreSQL)
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary)
FROM employees;
```

---

## 🔄 FILTER Clause

<div dir="rtl">

PostgreSQL بيدعم FILTER لتطبيق شرط على aggregate معين:

</div>

```sql
SELECT
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE department = 'Engineering') AS engineering_count,
    COUNT(*) FILTER (WHERE department = 'Marketing') AS marketing_count,
    COUNT(*) FILTER (WHERE salary > 70000) AS high_salary_count,
    AVG(salary) FILTER (WHERE is_active) AS active_avg_salary,
    SUM(salary) FILTER (WHERE hire_date >= '2020-01-01') AS recent_hires_total
FROM employees;

-- بديل بدون FILTER (أطول)
SELECT
    COUNT(*) AS total,
    SUM(CASE WHEN department = 'Engineering' THEN 1 ELSE 0 END) AS engineering_count,
    SUM(CASE WHEN department = 'Marketing' THEN 1 ELSE 0 END) AS marketing_count
FROM employees;
```

---

## ⚠️ NULL والـ Aggregates

```sql
-- معظم الـ aggregates بتتجاهل NULL

-- مثال: لو salary = [100, 200, NULL, 300]
SELECT COUNT(salary) FROM test;      -- 3 (مش 4)
SELECT SUM(salary) FROM test;        -- 600
SELECT AVG(salary) FROM test;        -- 200 (600/3, مش 600/4)
SELECT COUNT(*) FROM test;           -- 4 (بيحسب كل الصفوف)

-- للتعامل مع NULL
SELECT AVG(COALESCE(salary, 0)) FROM test;  -- 150 (600/4)

-- COUNT(*) vs COUNT(column)
SELECT
    COUNT(*) AS all_rows,            -- كل الصفوف
    COUNT(department) AS with_dept   -- الصفوف اللي فيها department
FROM employees;
```

---

## 📝 أمثلة عملية

<div dir="rtl">

### تقرير الموظفين

</div>

```sql
SELECT
    COUNT(*) AS total_employees,
    COUNT(*) FILTER (WHERE is_active) AS active_employees,
    COUNT(*) FILTER (WHERE NOT is_active) AS inactive_employees,
    COUNT(DISTINCT department) AS departments,
    ROUND(AVG(salary), 2) AS average_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    MAX(salary) - MIN(salary) AS salary_range,
    SUM(salary) AS total_payroll,
    MIN(hire_date) AS oldest_hire,
    MAX(hire_date) AS newest_hire
FROM employees;
```

<div dir="rtl">

### تقرير المبيعات

</div>

```sql
SELECT
    COUNT(*) AS total_orders,
    COUNT(DISTINCT user_id) AS unique_customers,
    SUM(total_amount) AS total_revenue,
    ROUND(AVG(total_amount), 2) AS average_order_value,
    MIN(total_amount) AS smallest_order,
    MAX(total_amount) AS largest_order,
    SUM(total_amount) FILTER (WHERE status = 'delivered') AS delivered_revenue,
    COUNT(*) FILTER (WHERE status = 'cancelled') AS cancelled_orders
FROM orders
WHERE created_at >= DATE_TRUNC('month', NOW());
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم Aliases واضحة

</div>

```sql
-- ✅ واضح
SELECT
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary
FROM employees;

-- ❌ مش واضح
SELECT COUNT(*), AVG(salary) FROM employees;
```

<div dir="rtl">

### 2. استخدم FILTER بدل CASE

</div>

```sql
-- ✅ أوضح (PostgreSQL)
COUNT(*) FILTER (WHERE condition)

-- ⚠️ يشتغل بس أطول
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

<div dir="rtl">

### 3. انتبه للـ NULL

</div>

```sql
-- فكر: هل NULL معناه 0 ولا "مش معروف"؟
AVG(salary)                    -- يتجاهل NULL
AVG(COALESCE(salary, 0))       -- يعامل NULL كـ 0
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **COUNT(*)** = كل الصفوف، **COUNT(column)** = غير NULL
2. **SUM, AVG** بيتجاهلوا NULL
3. **FILTER** لشروط على aggregate محدد
4. **STRING_AGG** لدمج النصوص
5. **ARRAY_AGG, JSON_AGG** لتجميع في structures

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [GROUP BY و HAVING](./14-group-by.md)**

</div>

---

<div align="center">

[⬅️ السابق: ORDER BY & LIMIT](./12-order-limit.md) | [🏠 العودة للـ Module](../README.md)

</div>
