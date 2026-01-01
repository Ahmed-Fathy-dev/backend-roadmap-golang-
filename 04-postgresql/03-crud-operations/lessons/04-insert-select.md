# INSERT ... SELECT - الإدخال من Query 📥

<div dir="rtl">

## مقدمة

`INSERT ... SELECT` بيسمحلك تدخل بيانات من query آخر - مفيد جداً لنسخ البيانات، إنشاء جداول مؤقتة، وعمليات الـ ETL.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
INSERT INTO target_table (column1, column2, ...)
SELECT value1, value2, ...
FROM source_table
WHERE condition;
```

---

## 📋 نسخ بيانات بسيطة

```sql
-- نسخ كل المستخدمين النشطين لجدول backup
INSERT INTO users_backup (id, username, email, created_at)
SELECT id, username, email, created_at
FROM users
WHERE is_active = TRUE;

-- نسخ مع تحويل البيانات
INSERT INTO archived_orders (order_id, total, archived_at)
SELECT id, total_amount, NOW()
FROM orders
WHERE created_at < '2024-01-01';

-- نسخ جدول كامل
INSERT INTO products_copy
SELECT * FROM products;
```

---

## 🔄 نسخ مع تعديل

```sql
-- نسخ مع تغيير قيم
INSERT INTO order_history (
    original_order_id,
    user_id,
    total_amount,
    status,
    archived_reason,
    archived_at
)
SELECT
    id,
    user_id,
    total_amount,
    'archived',           -- قيمة ثابتة
    'Monthly cleanup',    -- قيمة ثابتة
    NOW()                 -- timestamp حالي
FROM orders
WHERE status = 'completed'
  AND created_at < NOW() - INTERVAL '1 year';

-- نسخ مع حساب
INSERT INTO monthly_sales (year, month, total_revenue, order_count)
SELECT
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at),
    SUM(total_amount),
    COUNT(*)
FROM orders
WHERE status = 'completed'
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at);
```

---

## 🔗 INSERT SELECT مع JOIN

```sql
-- نسخ بيانات من عدة جداول
INSERT INTO order_report (
    order_id,
    customer_name,
    customer_email,
    order_total,
    order_date
)
SELECT
    o.id,
    u.full_name,
    u.email,
    o.total_amount,
    o.created_at
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'completed';

-- تقرير مفصل
INSERT INTO sales_summary (
    product_name,
    category_name,
    total_sold,
    total_revenue
)
SELECT
    p.name,
    c.name,
    SUM(oi.quantity),
    SUM(oi.quantity * oi.unit_price)
FROM order_items oi
JOIN products p ON oi.product_id = p.id
JOIN categories c ON p.category_id = c.id
JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY p.id, p.name, c.id, c.name;
```

---

## 📊 INSERT SELECT مع Subquery

```sql
-- إدخال المستخدمين اللي عملوا طلبات
INSERT INTO active_customers (user_id, total_orders, total_spent)
SELECT
    user_id,
    COUNT(*),
    SUM(total_amount)
FROM orders
WHERE user_id IN (
    SELECT id FROM users WHERE is_active = TRUE
)
GROUP BY user_id
HAVING COUNT(*) >= 5;

-- نسخ المنتجات الأكثر مبيعاً
INSERT INTO bestsellers (product_id, product_name, units_sold)
SELECT
    p.id,
    p.name,
    (SELECT SUM(quantity) FROM order_items WHERE product_id = p.id)
FROM products p
WHERE p.id IN (
    SELECT product_id
    FROM order_items
    GROUP BY product_id
    HAVING SUM(quantity) > 100
);
```

---

## 🏗️ إنشاء جدول من Query

```sql
-- CREATE TABLE AS (أسرع لجدول جديد)
CREATE TABLE high_value_customers AS
SELECT
    u.id,
    u.username,
    u.email,
    SUM(o.total_amount) AS lifetime_value,
    COUNT(o.id) AS order_count
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
GROUP BY u.id, u.username, u.email
HAVING SUM(o.total_amount) > 10000;

-- SELECT INTO (طريقة بديلة)
SELECT
    id,
    username,
    email,
    created_at
INTO new_users_table
FROM users
WHERE created_at >= '2024-01-01';
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                CREATE TABLE AS vs INSERT SELECT                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CREATE TABLE new_table AS SELECT ...                               │
│  ├── ينشئ جدول جديد                                                 │
│  ├── يحدد الـ columns تلقائياً من الـ SELECT                        │
│  ├── مش بينسخ constraints أو indexes                                │
│  └── أسرع لإنشاء جدول من الصفر                                      │
│                                                                      │
│  INSERT INTO existing_table SELECT ...                               │
│  ├── الجدول لازم يكون موجود                                         │
│  ├── بيستخدم الـ constraints الموجودة                               │
│  ├── الـ triggers بتشتغل                                            │
│  └── مناسب لإضافة بيانات لجدول موجود                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 INSERT SELECT مع CTE

```sql
-- استخدام CTE لتحضير البيانات
WITH monthly_stats AS (
    SELECT
        user_id,
        DATE_TRUNC('month', created_at) AS month,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id, DATE_TRUNC('month', created_at)
)
INSERT INTO user_monthly_summary (user_id, month, order_count, total_spent)
SELECT user_id, month, order_count, total_spent
FROM monthly_stats
WHERE order_count >= 3;

-- CTE معقد
WITH
active_users AS (
    SELECT id, username, email
    FROM users
    WHERE is_active = TRUE
),
user_orders AS (
    SELECT
        user_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
),
combined AS (
    SELECT
        u.id,
        u.username,
        u.email,
        COALESCE(uo.order_count, 0) AS order_count,
        COALESCE(uo.total_spent, 0) AS total_spent
    FROM active_users u
    LEFT JOIN user_orders uo ON u.id = uo.user_id
)
INSERT INTO customer_report (user_id, username, email, orders, revenue)
SELECT id, username, email, order_count, total_spent
FROM combined;
```

---

## 🎯 INSERT SELECT مع ON CONFLICT

```sql
-- Upsert من query
INSERT INTO product_stats (product_id, total_sold, last_sold_at)
SELECT
    product_id,
    SUM(quantity),
    MAX(created_at)
FROM order_items
GROUP BY product_id
ON CONFLICT (product_id) DO UPDATE SET
    total_sold = product_stats.total_sold + EXCLUDED.total_sold,
    last_sold_at = GREATEST(product_stats.last_sold_at, EXCLUDED.last_sold_at);

-- تحديث جدول lookup
INSERT INTO category_counts (category_id, product_count)
SELECT category_id, COUNT(*)
FROM products
WHERE is_available = TRUE
GROUP BY category_id
ON CONFLICT (category_id) DO UPDATE SET
    product_count = EXCLUDED.product_count;
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. Data Migration

</div>

```sql
-- نقل بيانات من schema قديم لجديد
INSERT INTO new_schema.customers (
    id,
    name,
    email,
    phone,
    address,
    created_at
)
SELECT
    customer_id,
    TRIM(first_name || ' ' || last_name),
    LOWER(email_address),
    COALESCE(mobile_phone, home_phone),
    street || ', ' || city || ', ' || country,
    registration_date
FROM old_schema.customer_records
WHERE status != 'deleted';
```

<div dir="rtl">

### 2. Data Aggregation

</div>

```sql
-- تجميع يومي
INSERT INTO daily_stats (date, orders, revenue, avg_order_value)
SELECT
    created_at::DATE,
    COUNT(*),
    SUM(total_amount),
    AVG(total_amount)
FROM orders
WHERE status = 'completed'
  AND created_at::DATE = CURRENT_DATE - 1  -- أمس
GROUP BY created_at::DATE;
```

<div dir="rtl">

### 3. Data Denormalization

</div>

```sql
-- إنشاء جدول denormalized للـ reporting
INSERT INTO order_flat (
    order_id,
    order_date,
    customer_name,
    customer_email,
    product_names,
    total_items,
    total_amount
)
SELECT
    o.id,
    o.created_at,
    u.full_name,
    u.email,
    STRING_AGG(p.name, ', '),
    SUM(oi.quantity),
    o.total_amount
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY o.id, o.created_at, u.full_name, u.email, o.total_amount;
```

<div dir="rtl">

### 4. Audit Trail

</div>

```sql
-- نسخ للـ audit قبل الحذف
INSERT INTO deleted_users_audit (
    original_id,
    username,
    email,
    deleted_at,
    deleted_by
)
SELECT
    id,
    username,
    email,
    NOW(),
    'admin'
FROM users
WHERE is_active = FALSE
  AND updated_at < NOW() - INTERVAL '90 days';
```

---

## 🔙 INSERT SELECT مع RETURNING

```sql
-- إرجاع البيانات المُدخلة
INSERT INTO archived_orders (original_id, user_id, total)
SELECT id, user_id, total_amount
FROM orders
WHERE created_at < '2024-01-01'
RETURNING id, original_id;

-- استخدام النتيجة
WITH inserted AS (
    INSERT INTO order_backup (order_id, data, backed_up_at)
    SELECT id, row_to_json(orders), NOW()
    FROM orders
    WHERE status = 'completed'
    RETURNING order_id
)
UPDATE orders
SET backed_up = TRUE
WHERE id IN (SELECT order_id FROM inserted);
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. تأكد من تطابق الأعمدة

</div>

```sql
-- ✅ صح: ترتيب ونوع الأعمدة متطابق
INSERT INTO target (col1, col2, col3)
SELECT a, b, c FROM source;

-- ❌ غلط: عدد الأعمدة مختلف
INSERT INTO target (col1, col2)
SELECT a, b, c FROM source;
```

<div dir="rtl">

### 2. استخدم Transaction للعمليات الكبيرة

</div>

```sql
BEGIN;

-- نسخ البيانات
INSERT INTO archive_table
SELECT * FROM main_table WHERE condition;

-- حذف من الجدول الأصلي
DELETE FROM main_table WHERE condition;

COMMIT;
```

<div dir="rtl">

### 3. اختبر الـ SELECT أولاً

</div>

```sql
-- ✅ اختبر الـ query الأول
SELECT id, username, email
FROM users
WHERE is_active = TRUE
LIMIT 10;

-- بعد التأكد، نفذ الـ INSERT
INSERT INTO target_table (id, username, email)
SELECT id, username, email
FROM users
WHERE is_active = TRUE;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **INSERT SELECT** لنسخ بيانات من query
2. ممكن تستخدم **JOIN** و **WHERE** و **GROUP BY**
3. **CREATE TABLE AS** لإنشاء جدول جديد من query
4. استخدم **CTE** للـ queries المعقدة
5. ممكن تدمج مع **ON CONFLICT** للـ Upsert
6. **اختبر الـ SELECT** قبل الـ INSERT

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [COPY Command](./05-copy-command.md)**

</div>

---

<div align="center">

[⬅️ السابق: ON CONFLICT](./03-upsert.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./05-copy-command.md)

</div>
