# UPDATE المتقدم - تحديث متقدم 🚀

<div dir="rtl">

## مقدمة

في الدرس ده هنتعلم تقنيات متقدمة في UPDATE: التحديث من JOIN، من Subquery، و CTEs.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 🔗 UPDATE مع FROM (JOIN)

<div dir="rtl">

### الصيغة

</div>

```sql
UPDATE target_table t
SET column = value
FROM source_table s
WHERE t.foreign_key = s.id;
```

<div dir="rtl">

### أمثلة

</div>

```sql
-- تحديث أسعار المنتجات من جدول الفئات
UPDATE products p
SET price = p.price * c.price_multiplier
FROM categories c
WHERE p.category_id = c.id
  AND c.name = 'Electronics';

-- تحديث معلومات الطلبات من المستخدمين
UPDATE orders o
SET customer_email = u.email,
    customer_name = u.full_name
FROM users u
WHERE o.user_id = u.id
  AND o.customer_email IS NULL;

-- تحديث المخزون من جدول shipments
UPDATE products p
SET stock = p.stock + s.quantity
FROM shipments s
WHERE p.id = s.product_id
  AND s.status = 'received'
  AND s.processed = FALSE;

-- تحديث ratings من aggregated data
UPDATE products p
SET
    avg_rating = r.avg_rating,
    review_count = r.review_count
FROM (
    SELECT
        product_id,
        AVG(rating) AS avg_rating,
        COUNT(*) AS review_count
    FROM reviews
    GROUP BY product_id
) r
WHERE p.id = r.product_id;
```

---

## 📊 UPDATE مع Subquery

<div dir="rtl">

### في SET

</div>

```sql
-- تحديث بقيمة من subquery
UPDATE employees
SET salary = (
    SELECT AVG(salary)
    FROM employees
    WHERE department = 'Engineering'
)
WHERE id = 1;

-- تحديث بقيمة محسوبة
UPDATE products
SET price = (
    SELECT AVG(price) * 1.1
    FROM products
    WHERE category_id = products.category_id
)
WHERE is_featured = TRUE;
```

<div dir="rtl">

### في WHERE

</div>

```sql
-- تحديث صفوف بناءً على subquery
UPDATE orders
SET status = 'vip_processing'
WHERE user_id IN (
    SELECT id FROM users WHERE role = 'vip'
);

-- تحديث المنتجات الأكثر مبيعاً
UPDATE products
SET is_bestseller = TRUE
WHERE id IN (
    SELECT product_id
    FROM order_items
    GROUP BY product_id
    HAVING SUM(quantity) > 100
);

-- تحديث باستخدام EXISTS
UPDATE users
SET has_orders = TRUE
WHERE EXISTS (
    SELECT 1 FROM orders
    WHERE orders.user_id = users.id
);

-- تحديث باستخدام NOT EXISTS
UPDATE products
SET is_available = FALSE
WHERE NOT EXISTS (
    SELECT 1 FROM inventory
    WHERE inventory.product_id = products.id
      AND inventory.quantity > 0
);
```

---

## 🔄 UPDATE مع CTE

```sql
-- استخدام CTE لتحضير البيانات
WITH sales_summary AS (
    SELECT
        product_id,
        SUM(quantity) AS total_sold,
        SUM(quantity * unit_price) AS total_revenue
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.id
    WHERE o.status = 'completed'
    GROUP BY product_id
)
UPDATE products p
SET
    units_sold = s.total_sold,
    total_revenue = s.total_revenue,
    updated_at = NOW()
FROM sales_summary s
WHERE p.id = s.product_id;

-- CTE معقد
WITH
-- حساب متوسط كل قسم
dept_averages AS (
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
),
-- الموظفين تحت المتوسط
below_average AS (
    SELECT e.id
    FROM employees e
    JOIN dept_averages d ON e.department = d.department
    WHERE e.salary < d.avg_salary * 0.9
)
UPDATE employees
SET
    salary = salary * 1.05,
    last_raise_date = NOW()
WHERE id IN (SELECT id FROM below_average);
```

---

## 📊 UPDATE مع Window Functions

```sql
-- تحديث ترتيب المنتجات
WITH ranked_products AS (
    SELECT
        id,
        ROW_NUMBER() OVER (
            PARTITION BY category_id
            ORDER BY total_sales DESC
        ) AS rank
    FROM products
)
UPDATE products p
SET category_rank = r.rank
FROM ranked_products r
WHERE p.id = r.id;

-- تحديث percentile
WITH percentiles AS (
    SELECT
        id,
        NTILE(100) OVER (ORDER BY salary) AS percentile
    FROM employees
)
UPDATE employees e
SET salary_percentile = p.percentile
FROM percentiles p
WHERE e.id = p.id;
```

---

## 🔒 UPDATE مع Locking

```sql
-- SELECT FOR UPDATE للـ locking
BEGIN;

-- قفل الصفوف قبل التحديث
SELECT * FROM products
WHERE id = 5
FOR UPDATE;

-- الآن ممكن تحدث بأمان
UPDATE products
SET stock = stock - 1
WHERE id = 5;

COMMIT;

-- UPDATE مع NOWAIT
BEGIN;

SELECT * FROM products
WHERE id = 5
FOR UPDATE NOWAIT;  -- لو مقفول، يفشل فوراً

UPDATE products SET stock = stock - 1 WHERE id = 5;

COMMIT;

-- UPDATE مع SKIP LOCKED
UPDATE products
SET is_processing = TRUE
WHERE id IN (
    SELECT id FROM products
    WHERE is_processing = FALSE
    ORDER BY created_at
    LIMIT 10
    FOR UPDATE SKIP LOCKED  -- تخطي الصفوف المقفولة
)
RETURNING id;
```

---

## 📊 Bulk Update Patterns

<div dir="rtl">

### تحديث من Values List

</div>

```sql
-- تحديث عدة صفوف بقيم مختلفة
UPDATE products AS p
SET price = v.new_price
FROM (VALUES
    (1, 99.99),
    (2, 149.99),
    (3, 199.99),
    (4, 249.99)
) AS v(id, new_price)
WHERE p.id = v.id;

-- مع عدة أعمدة
UPDATE products AS p
SET
    price = v.price,
    stock = v.stock
FROM (VALUES
    (1, 99.99, 50),
    (2, 149.99, 30),
    (3, 199.99, 20)
) AS v(id, price, stock)
WHERE p.id = v.id;
```

<div dir="rtl">

### تحديث من Temp Table

</div>

```sql
-- إنشاء temp table بالقيم الجديدة
CREATE TEMP TABLE price_updates (
    product_id INT,
    new_price NUMERIC(10, 2)
);

INSERT INTO price_updates VALUES
    (1, 99.99),
    (2, 149.99),
    (3, 199.99);

-- تحديث من الـ temp table
UPDATE products p
SET
    price = u.new_price,
    updated_at = NOW()
FROM price_updates u
WHERE p.id = u.product_id;

DROP TABLE price_updates;
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. تحديث أسعار المنتجات

</div>

```sql
-- تطبيق خصم على فئة معينة
WITH discount_products AS (
    SELECT p.id
    FROM products p
    JOIN categories c ON p.category_id = c.id
    WHERE c.name = 'Summer Collection'
      AND p.is_available = TRUE
)
UPDATE products
SET
    original_price = price,
    price = price * 0.80,
    is_on_sale = TRUE,
    sale_ends_at = NOW() + INTERVAL '7 days',
    updated_at = NOW()
WHERE id IN (SELECT id FROM discount_products)
RETURNING id, name, original_price, price;
```

<div dir="rtl">

### 2. Denormalize Data

</div>

```sql
-- تحديث cached counts
WITH order_counts AS (
    SELECT
        user_id,
        COUNT(*) AS total_orders,
        SUM(total_amount) AS total_spent,
        MAX(created_at) AS last_order_at
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
)
UPDATE users u
SET
    order_count = oc.total_orders,
    total_spent = oc.total_spent,
    last_order_at = oc.last_order_at,
    updated_at = NOW()
FROM order_counts oc
WHERE u.id = oc.user_id;
```

<div dir="rtl">

### 3. Batch Processing

</div>

```sql
-- تحديث batch من الطلبات
WITH orders_to_process AS (
    SELECT id
    FROM orders
    WHERE status = 'pending'
      AND payment_verified = TRUE
    ORDER BY created_at
    LIMIT 100
    FOR UPDATE SKIP LOCKED
)
UPDATE orders
SET
    status = 'processing',
    processed_at = NOW(),
    updated_at = NOW()
WHERE id IN (SELECT id FROM orders_to_process)
RETURNING id, user_id, total_amount;
```

<div dir="rtl">

### 4. Conditional Bulk Update

</div>

```sql
-- تحديث رواتب بناءً على معايير متعددة
UPDATE employees e
SET salary = CASE
    WHEN d.dept_budget_increase > 0.15 AND e.performance_rating >= 4
        THEN e.salary * 1.15
    WHEN d.dept_budget_increase > 0.10 AND e.performance_rating >= 3
        THEN e.salary * 1.10
    WHEN d.dept_budget_increase > 0.05
        THEN e.salary * 1.05
    ELSE e.salary
END
FROM departments d
WHERE e.department_id = d.id
  AND e.is_active = TRUE;
```

---

## 💡 Performance Tips

<div dir="rtl">

### 1. Index على WHERE columns

</div>

```sql
-- تأكد من وجود index
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_products_category ON products(category_id);

-- الآن الـ UPDATE هيكون سريع
UPDATE orders SET status = 'archived' WHERE status = 'completed';
```

<div dir="rtl">

### 2. Batch Large Updates

</div>

```sql
-- ❌ بطيء للبيانات الكبيرة
UPDATE huge_table SET column = value WHERE condition;

-- ✅ أسرع: batch by batch
DO $$
DECLARE
    batch_size INT := 10000;
    affected INT := 1;
BEGIN
    WHILE affected > 0 LOOP
        UPDATE huge_table
        SET column = value
        WHERE id IN (
            SELECT id FROM huge_table
            WHERE condition AND column IS DISTINCT FROM value
            LIMIT batch_size
        );
        GET DIAGNOSTICS affected = ROW_COUNT;
        COMMIT;
    END LOOP;
END $$;
```

<div dir="rtl">

### 3. Avoid Unnecessary Updates

</div>

```sql
-- ✅ تجنب التحديث لو القيمة مش متغيرة
UPDATE products
SET name = 'New Name'
WHERE id = 5
  AND name IS DISTINCT FROM 'New Name';
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **UPDATE FROM** للتحديث من جدول آخر
2. **Subquery** في SET أو WHERE
3. **CTE** للـ queries المعقدة
4. **FOR UPDATE** للـ locking الآمن
5. **Batch updates** للبيانات الكبيرة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [DELETE الأساسي](./10-basic-delete.md)**

</div>

---

<div align="center">

[⬅️ السابق: UPDATE الأساسي](./08-basic-update.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./10-basic-delete.md)

</div>
