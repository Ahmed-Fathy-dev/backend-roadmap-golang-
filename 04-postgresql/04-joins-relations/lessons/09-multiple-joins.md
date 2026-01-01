# Multiple JOINs - ربط عدة جداول 🔗🔗🔗

<div dir="rtl">

## مقدمة

في التطبيقات الحقيقية، غالباً تحتاج تربط 3 جداول أو أكتر في query واحد.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 مثال E-Commerce

```sql
-- الطلب الكامل: User → Order → OrderItems → Product → Category
SELECT
    u.username AS customer,
    o.id AS order_id,
    o.status,
    p.name AS product,
    c.name AS category,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
JOIN categories c ON p.category_id = c.id
WHERE o.id = 1;
```

---

## 📝 ترتيب الـ JOINs

```sql
-- الترتيب الطبيعي: من الجدول الرئيسي للتفاصيل
FROM orders o                              -- 1. الجدول الأساسي
JOIN users u ON o.user_id = u.id           -- 2. معلومات العميل
JOIN order_items oi ON o.id = oi.order_id  -- 3. تفاصيل الطلب
JOIN products p ON oi.product_id = p.id    -- 4. معلومات المنتج
JOIN categories c ON p.category_id = c.id  -- 5. الفئة

-- 💡 الترتيب مهم للقراءة، الـ optimizer هيختار الترتيب الأفضل
```

---

## 🔢 أمثلة معقدة

```sql
-- تقرير المبيعات الكامل
SELECT
    c.name AS category,
    p.name AS product,
    COUNT(DISTINCT o.id) AS order_count,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.unit_price) AS revenue,
    ROUND(AVG(r.rating), 2) AS avg_rating
FROM categories c
JOIN products p ON c.id = p.category_id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id AND o.status = 'completed'
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY c.id, c.name, p.id, p.name
ORDER BY revenue DESC NULLS LAST;

-- أفضل عملاء كل فئة
WITH customer_spending AS (
    SELECT
        c.id AS category_id,
        c.name AS category,
        u.id AS user_id,
        u.username,
        SUM(oi.quantity * oi.unit_price) AS spent,
        ROW_NUMBER() OVER (
            PARTITION BY c.id
            ORDER BY SUM(oi.quantity * oi.unit_price) DESC
        ) AS rank
    FROM categories c
    JOIN products p ON c.id = p.category_id
    JOIN order_items oi ON p.id = oi.product_id
    JOIN orders o ON oi.order_id = o.id
    JOIN users u ON o.user_id = u.id
    WHERE o.status = 'completed'
    GROUP BY c.id, c.name, u.id, u.username
)
SELECT category, username, spent
FROM customer_spending
WHERE rank <= 3;
```

---

## 💡 Best Practices

```sql
-- 1. استخدم Aliases قصيرة وواضحة
FROM orders o          -- ✅
FROM orders ord        -- ✅
FROM orders            -- ❌ (بدون alias)

-- 2. حدد الأعمدة صراحةً
SELECT u.id, u.username, o.id AS order_id
-- مش SELECT *

-- 3. استخدم CTE للـ queries المعقدة
WITH order_totals AS (...)
SELECT * FROM order_totals ...

-- 4. Index على Foreign Keys
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

---

## ⏭️ الدرس التالي

**➡️ [JOIN Performance](./10-join-performance.md)**

---

<div align="center">

[⬅️ السابق](./08-self-join.md) | [🏠 العودة للـ Module](../README.md) | [التالي ➡️](./10-join-performance.md)

</div>
