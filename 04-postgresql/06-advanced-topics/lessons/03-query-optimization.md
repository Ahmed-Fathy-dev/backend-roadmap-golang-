# Query Optimization - تحسين الـ Queries 🚀

<div dir="rtl">

## مقدمة

تحسين الـ queries ممكن يحول query من دقائق لـ milliseconds. هنتعلم التقنيات والـ patterns الأساسية.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 Optimization Checklist

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Query Optimization Checklist                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  □ 1. Use EXPLAIN ANALYZE to identify bottlenecks                   │
│  □ 2. Add appropriate indexes                                        │
│  □ 3. Select only needed columns (no SELECT *)                      │
│  □ 4. Filter early (WHERE before JOIN)                              │
│  □ 5. Use LIMIT for pagination                                       │
│  □ 6. Avoid functions on indexed columns                            │
│  □ 7. Use proper data types                                          │
│  □ 8. Update statistics (ANALYZE)                                   │
│  □ 9. Consider query rewriting                                       │
│  □ 10. Use CTEs or subqueries wisely                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Select Only Needed Columns

```sql
-- ❌ سيء: يقرأ كل الأعمدة
SELECT * FROM users WHERE id = 1;

-- ✅ جيد: يقرأ الأعمدة المطلوبة فقط
SELECT id, username, email FROM users WHERE id = 1;

-- ✅ أفضل: مع covering index
CREATE INDEX idx_users_email_covering ON users(email) INCLUDE (username);

SELECT username FROM users WHERE email = 'ahmed@example.com';
-- Index Only Scan!
```

---

## 2️⃣ Filter Early

```sql
-- ❌ سيء: يعمل JOIN أولاً ثم يفلتر
SELECT u.username, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed' AND o.created_at > '2024-01-01';

-- ✅ جيد: يفلتر في الـ subquery أولاً
SELECT u.username, o.total_amount
FROM users u
JOIN (
    SELECT user_id, total_amount
    FROM orders
    WHERE status = 'completed' AND created_at > '2024-01-01'
) o ON u.id = o.user_id;

-- أو PostgreSQL غالباً بيعمل optimize تلقائي
-- لكن تأكد بـ EXPLAIN ANALYZE
```

---

## 3️⃣ Avoid Functions on Indexed Columns

```sql
-- ❌ سيء: Function على الـ indexed column
SELECT * FROM users WHERE YEAR(created_at) = 2024;
-- Seq Scan! الـ index مش هيتستخدم

-- ✅ جيد: استخدم range
SELECT * FROM users
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
-- Index Scan!

-- ❌ سيء
SELECT * FROM users WHERE LOWER(email) = 'ahmed@example.com';

-- ✅ جيد: Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'ahmed@example.com';
-- Index Scan!

-- ✅ أفضل: استخدم citext
ALTER TABLE users ALTER COLUMN email TYPE citext;
```

---

## 4️⃣ Optimize JOINs

```sql
-- ❌ سيء: JOIN على non-indexed columns
SELECT u.username, o.total_amount
FROM users u
JOIN orders o ON u.email = o.user_email;  -- No index!

-- ✅ جيد: JOIN على indexed columns
SELECT u.username, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id;  -- FK with index

-- Index على الـ FK
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

```sql
-- ❌ سيء: Multiple JOINs بدون indexes
SELECT
    u.username,
    o.total_amount,
    p.name as product_name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE u.is_active = true;

-- ✅ جيد: مع الـ indexes الصحيحة
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_users_active ON users(id) WHERE is_active = true;
```

---

## 5️⃣ Pagination

```sql
-- ❌ سيء: OFFSET الكبير
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 10000;
-- يقرأ 10,020 صف ويرمي 10,000!

-- ✅ جيد: Keyset Pagination
SELECT * FROM posts
WHERE created_at < '2024-01-15 10:30:00'  -- Last seen timestamp
ORDER BY created_at DESC
LIMIT 20;

-- ✅ جيد: مع ID
SELECT * FROM posts
WHERE (created_at, id) < ('2024-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Index
CREATE INDEX idx_posts_created_id ON posts(created_at DESC, id DESC);
```

---

## 6️⃣ Subqueries vs JOINs

```sql
-- Correlated subquery (slow!)
SELECT u.username,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) as order_count
FROM users u;
-- يتنفذ مرة لكل user!

-- ✅ جيد: JOIN with GROUP BY
SELECT u.username, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- ✅ جيد: Subquery in FROM
SELECT u.username, COALESCE(oc.order_count, 0) as order_count
FROM users u
LEFT JOIN (
    SELECT user_id, COUNT(*) as order_count
    FROM orders
    GROUP BY user_id
) oc ON u.id = oc.user_id;
```

---

## 7️⃣ EXISTS vs IN

```sql
-- للـ large result sets، EXISTS أسرع
-- IN
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE status = 'completed');

-- EXISTS (usually faster for large tables)
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id AND o.status = 'completed'
);

-- EXISTS بيتوقف أول ما يلاقي match
-- IN بيجيب كل النتائج الأول
```

---

## 8️⃣ CTEs (Common Table Expressions)

```sql
-- CTE for readability
WITH active_users AS (
    SELECT id, username
    FROM users
    WHERE is_active = true
),
recent_orders AS (
    SELECT user_id, SUM(total_amount) as total
    FROM orders
    WHERE created_at > NOW() - INTERVAL '30 days'
    GROUP BY user_id
)
SELECT u.username, COALESCE(o.total, 0) as monthly_spend
FROM active_users u
LEFT JOIN recent_orders o ON u.id = o.user_id;

-- ⚠️ Note: In PostgreSQL < 12, CTEs are optimization barriers
-- In PostgreSQL 12+, they can be inlined
```

---

## 9️⃣ UNION vs UNION ALL

```sql
-- UNION: removes duplicates (slower)
SELECT email FROM users WHERE role = 'admin'
UNION
SELECT email FROM users WHERE is_active = true;

-- UNION ALL: keeps duplicates (faster)
SELECT email FROM users WHERE role = 'admin'
UNION ALL
SELECT email FROM users WHERE is_active = true AND role != 'admin';

-- Use UNION ALL when you know there are no duplicates
-- or when duplicates are acceptable
```

---

## 🔟 Batch Operations

```sql
-- ❌ سيء: Individual inserts
INSERT INTO logs (message) VALUES ('log1');
INSERT INTO logs (message) VALUES ('log2');
INSERT INTO logs (message) VALUES ('log3');
-- 3 round trips!

-- ✅ جيد: Batch insert
INSERT INTO logs (message) VALUES
    ('log1'),
    ('log2'),
    ('log3');
-- 1 round trip!

-- ✅ أفضل: COPY (fastest)
COPY logs (message) FROM '/path/to/file.csv';

-- For updates
-- ❌ سيء
UPDATE products SET price = 99.99 WHERE id = 1;
UPDATE products SET price = 149.99 WHERE id = 2;

-- ✅ جيد: Single statement
UPDATE products SET price = CASE id
    WHEN 1 THEN 99.99
    WHEN 2 THEN 149.99
END
WHERE id IN (1, 2);
```

---

## 📊 Real-World Optimization Example

```sql
-- Original slow query (10+ seconds)
SELECT
    u.username,
    u.email,
    COUNT(o.id) as order_count,
    SUM(o.total_amount) as total_spent,
    MAX(o.created_at) as last_order
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2023-01-01'
    AND (o.status = 'completed' OR o.status IS NULL)
GROUP BY u.id
ORDER BY total_spent DESC NULLS LAST
LIMIT 100;

-- Step 1: Add indexes
CREATE INDEX idx_users_created ON users(created_at);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Step 2: Rewrite with subquery
WITH user_orders AS (
    SELECT
        user_id,
        COUNT(*) as order_count,
        SUM(total_amount) as total_spent,
        MAX(created_at) as last_order
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
)
SELECT
    u.username,
    u.email,
    COALESCE(uo.order_count, 0) as order_count,
    COALESCE(uo.total_spent, 0) as total_spent,
    uo.last_order
FROM users u
LEFT JOIN user_orders uo ON u.id = uo.user_id
WHERE u.created_at > '2023-01-01'
ORDER BY total_spent DESC NULLS LAST
LIMIT 100;

-- Result: < 100ms
```

---

## 🔧 Configuration Tuning

```sql
-- Memory settings (postgresql.conf)
shared_buffers = '256MB'           -- 25% of RAM
effective_cache_size = '768MB'     -- 75% of RAM
work_mem = '64MB'                  -- Per operation
maintenance_work_mem = '128MB'     -- For VACUUM, CREATE INDEX

-- Planner settings
random_page_cost = 1.1             -- For SSD (default 4.0)
effective_io_concurrency = 200     -- For SSD

-- Check current settings
SHOW shared_buffers;
SHOW work_mem;
```

---

## 💡 Optimization Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Common Optimization Patterns                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Pattern              │ Before          │ After                     │
│  ─────────────────────┼─────────────────┼─────────────────────────  │
│  Covering Index       │ Index + Heap    │ Index Only Scan           │
│  Partial Index        │ Full Index      │ Smaller, Faster Index     │
│  Expression Index     │ Seq Scan        │ Index Scan                │
│  Keyset Pagination    │ OFFSET          │ WHERE + LIMIT             │
│  Batch Operations     │ Many INSERTs    │ Single INSERT             │
│  Denormalization      │ Many JOINs      │ Fewer JOINs               │
│  Materialized View    │ Complex Query   │ Precomputed               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **EXPLAIN ANALYZE** قبل أي optimization
2. **SELECT** الأعمدة المطلوبة فقط
3. **فلتر بدري** - WHERE قبل JOIN
4. تجنب **functions على indexed columns**
5. استخدم **Keyset pagination** بدل OFFSET
6. **EXISTS** أسرع من IN للجداول الكبيرة
7. **Batch** العمليات بدل one-by-one

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [JSON/JSONB Basics](./04-json-jsonb.md)**

</div>

---

<div align="center">

[⬅️ السابق](./02-explain-analyze.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./04-json-jsonb.md)

</div>
