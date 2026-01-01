# JOIN Performance - تحسين أداء الـ JOINs ⚡

<div dir="rtl">

## مقدمة

الـ JOINs ممكن تكون بطيئة لو مش مصممة صح. هنتعلم كيف نحسّن أداءها.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 EXPLAIN ANALYZE

```sql
-- شوف خطة التنفيذ
EXPLAIN ANALYZE
SELECT u.username, COUNT(o.id)
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- النتيجة بتوريك:
-- - نوع الـ scan (Seq Scan, Index Scan)
-- - نوع الـ join (Hash, Nested Loop, Merge)
-- - الوقت والتكلفة
```

---

## 🔑 Indexes على Foreign Keys

```sql
-- ❌ بدون index: Seq Scan (بطيء)
-- الـ JOIN هيقرأ الجدول كله!

-- ✅ مع index: Index Scan (سريع)
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_products_category_id ON products(category_id);

-- Composite index لـ queries معينة
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

---

## 📊 أنواع الـ JOIN Algorithms

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JOIN Algorithms                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Nested Loop:                                                    │
│     ├── لكل صف في A، ابحث في B                                     │
│     ├── مناسب: جداول صغيرة أو indexed lookups                      │
│     └── O(n × m) بدون index                                        │
│                                                                      │
│  2. Hash Join:                                                      │
│     ├── ابني hash table من الجدول الأصغر                           │
│     ├── ابحث في الـ hash للجدول الأكبر                             │
│     └── مناسب: جداول متوسطة، equality joins                        │
│                                                                      │
│  3. Merge Join:                                                     │
│     ├── رتب الجدولين، امشي عليهم بالتوازي                          │
│     ├── مناسب: جداول كبيرة مرتبة                                   │
│     └── الأسرع لو الجدولين مرتبين                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 تحسينات

```sql
-- 1. SELECT الأعمدة اللي محتاجها بس
SELECT u.id, u.username, o.total_amount  -- ✅
-- SELECT *  -- ❌

-- 2. فلتر بدري
SELECT u.username, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'  -- فلتر هنا
  AND o.created_at > '2024-01-01';

-- 3. استخدم LIMIT لو مش محتاج كل النتائج
SELECT u.username, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
ORDER BY o.created_at DESC
LIMIT 100;

-- 4. Materialized View للـ queries المتكررة
CREATE MATERIALIZED VIEW order_summary AS
SELECT
    u.id AS user_id,
    u.username,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;

-- Refresh بشكل دوري
REFRESH MATERIALIZED VIEW order_summary;
```

---

## 📊 مقارنة الأداء

```sql
-- قبل التحسين
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'completed';
-- Seq Scan, 500ms

-- بعد إضافة indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);

EXPLAIN ANALYZE
SELECT o.id, u.username FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'completed';
-- Index Scan, 5ms
```

---

## 💡 Best Practices

1. **Index** على كل Foreign Key
2. **EXPLAIN ANALYZE** لفهم الخطة
3. **SELECT** الأعمدة المطلوبة فقط
4. **فلتر بدري** بـ WHERE
5. **Materialized Views** للتقارير

---

## 🎉 انتهى Module 04!

**➡️ الـ Module التالي: [Go + PostgreSQL](../../05-go-postgres-integration/README.md)**

---

<div align="center">

[⬅️ السابق](./09-multiple-joins.md) | [🏠 العودة للـ Module](../README.md)

</div>
