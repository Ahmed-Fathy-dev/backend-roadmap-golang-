# Lesson 8: Query Optimization - تحسين الـ Queries 🚀

<div dir="rtl">

## المقدمة

Query بطيء ممكن يقتل الـ application بتاعك! في هذا الدرس هنتعلم إزاي نحلل ونحسن الـ queries.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Why Queries Get Slow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Common Causes of Slow Queries                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Missing Indexes                                                  │
│     └─ Full table scan instead of index lookup                      │
│     └─ الأكثر شيوعاً!                                                │
│                                                                      │
│  2. Poor Query Design                                                │
│     └─ SELECT * بدل columns محددة                                   │
│     └─ Unnecessary JOINs                                             │
│     └─ N+1 Query Problem                                             │
│                                                                      │
│  3. Large Result Sets                                                │
│     └─ Fetching millions of rows                                    │
│     └─ مفيش LIMIT                                                    │
│                                                                      │
│  4. Complex Calculations                                             │
│     └─ Functions on indexed columns                                  │
│     └─ Heavy aggregations                                            │
│                                                                      │
│  5. Lock Contention                                                  │
│     └─ Waiting for other transactions                               │
│     └─ Long-running transactions                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ EXPLAIN ANALYZE

### Basic Usage

```sql
-- EXPLAIN: يشرح الخطة
EXPLAIN SELECT * FROM users WHERE email = 'ahmed@test.com';

-- EXPLAIN ANALYZE: يشرح + ينفذ + يقيس الوقت
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'ahmed@test.com';
```

### Reading the Output

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 5;

-- Output without index:
Seq Scan on orders  (cost=0.00..15406.00 rows=157 width=52)
                     (actual time=0.025..89.234 rows=150 loops=1)
  Filter: (user_id = 5)
  Rows Removed by Filter: 999850
Planning Time: 0.123 ms
Execution Time: 89.456 ms
│
└── ⚠️ Seq Scan = Full Table Scan (BAD!)
    89ms for 150 rows (SLOW!)

-- After adding index:
CREATE INDEX idx_orders_user ON orders(user_id);

EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 5;

-- Output with index:
Index Scan using idx_orders_user on orders
                     (cost=0.42..8.56 rows=157 width=52)
                     (actual time=0.025..0.234 rows=150 loops=1)
  Index Cond: (user_id = 5)
Planning Time: 0.098 ms
Execution Time: 0.312 ms
│
└── ✅ Index Scan (GOOD!)
    0.3ms for 150 rows (FAST!)
```

### Key Terms in EXPLAIN

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EXPLAIN Output Terms                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Scan Types (من الأسوأ للأفضل):                                     │
│  ────────────────────────────────                                    │
│  Seq Scan        → Full table scan (عادة سيء)                        │
│  Index Scan      → استخدام index (جيد)                               │
│  Index Only Scan → البيانات من index فقط (أفضل)                      │
│  Bitmap Scan     → للـ OR conditions أو ranges كبيرة                │
│                                                                      │
│  Join Types:                                                         │
│  ───────────                                                         │
│  Nested Loop     → للـ small tables أو indexed                      │
│  Hash Join       → للـ large tables بدون index                      │
│  Merge Join      → للـ sorted data أو مع index                      │
│                                                                      │
│  Cost Numbers:                                                       │
│  ──────────────                                                      │
│  cost=X..Y       → X = startup cost, Y = total cost                 │
│  rows=N          → عدد الصفوف المتوقع                                │
│  width=N         → حجم الصف بالـ bytes                              │
│                                                                      │
│  Actual vs Estimated:                                                │
│  ────────────────────                                                │
│  rows=100 vs actual rows=5000 → Statistics out of date!            │
│  Run: ANALYZE table_name;                                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ Common Optimizations

### A. Use Indexes Properly

```sql
-- ❌ Bad: Function on indexed column (won't use index)
SELECT * FROM users WHERE LOWER(email) = 'ahmed@test.com';

-- ✅ Good: Expression index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'ahmed@test.com';

-- ✅ Better: Store lowercase
SELECT * FROM users WHERE email = 'ahmed@test.com';
-- (ensure email is stored lowercase)

---

-- ❌ Bad: Type mismatch
SELECT * FROM products WHERE id = '42';  -- id is INTEGER, '42' is TEXT

-- ✅ Good: Correct type
SELECT * FROM products WHERE id = 42;

---

-- ❌ Bad: Leading wildcard (can't use index)
SELECT * FROM products WHERE name LIKE '%laptop%';

-- ✅ Good: Trailing wildcard (uses index)
SELECT * FROM products WHERE name LIKE 'laptop%';

-- ✅ For full-text search, use proper FTS
CREATE INDEX idx_products_name_gin ON products USING gin(to_tsvector('english', name));
SELECT * FROM products WHERE to_tsvector('english', name) @@ to_tsquery('laptop');
```

### B. Optimize JOINs

```sql
-- ❌ Bad: N+1 Query Problem
-- In application code:
users = db.query("SELECT * FROM users")
for user in users:
    orders = db.query(f"SELECT * FROM orders WHERE user_id = {user.id}")
-- 1 query for users + N queries for orders = N+1 queries!

-- ✅ Good: Single JOIN query
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
-- 1 query only!

---

-- ❌ Bad: Joining large tables without index
SELECT *
FROM orders o
JOIN order_items oi ON oi.order_id = o.id;  -- slow if no index

-- ✅ Good: Ensure FK has index
CREATE INDEX idx_order_items_order ON order_items(order_id);
```

### C. Limit Results

```sql
-- ❌ Bad: Fetching everything
SELECT * FROM orders;  -- millions of rows!

-- ✅ Good: Pagination
SELECT * FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;  -- Page 1

-- ✅ Better: Keyset pagination (for large datasets)
SELECT * FROM orders
WHERE created_at < '2024-01-15 10:30:00'  -- last item from previous page
ORDER BY created_at DESC
LIMIT 20;
```

### D. Select Only Needed Columns

```sql
-- ❌ Bad: Select everything
SELECT * FROM products;

-- ✅ Good: Select specific columns
SELECT id, name, price FROM products;

-- Why?
-- 1. Less data transferred
-- 2. May enable Index Only Scan
-- 3. Clearer code intention
```

### E. Avoid Expensive Operations

```sql
-- ❌ Bad: DISTINCT on large result
SELECT DISTINCT category FROM products;

-- ✅ Good: Use GROUP BY
SELECT category FROM products GROUP BY category;

-- ✅ Or: Create a categories table!

---

-- ❌ Bad: ORDER BY random column
SELECT * FROM products ORDER BY random() LIMIT 10;  -- full scan!

-- ✅ Good: Use materialized approach
-- Store random order periodically, or use better algorithm

---

-- ❌ Bad: Subquery in SELECT (executed per row!)
SELECT
    name,
    (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;

-- ✅ Good: LEFT JOIN with aggregation
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

---

## 3️⃣ Index Strategies

```
┌─────────────────────────────────────────────────────────────────────┐
│                    When to Create Indexes                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Create Index On:                                                 │
│  ─────────────────────                                               │
│  • Primary Keys (automatic)                                          │
│  • Foreign Keys                                                      │
│  • Columns in WHERE clauses                                          │
│  • Columns in JOIN conditions                                        │
│  • Columns in ORDER BY                                               │
│  • Columns with UNIQUE constraint                                    │
│                                                                      │
│  ❌ Don't Index:                                                     │
│  ────────────────                                                    │
│  • Small tables (< 1000 rows)                                        │
│  • Low cardinality columns (boolean, status)                        │
│  • Frequently updated columns                                        │
│  • Rarely queried columns                                            │
│  • Wide columns (TEXT, large VARCHAR)                               │
│                                                                      │
│  ⚠️ Trade-offs:                                                      │
│  ──────────────                                                      │
│  • Indexes speed up SELECTs                                          │
│  • Indexes slow down INSERTs/UPDATEs                                │
│  • Indexes use disk space                                            │
│  • Too many indexes = slow writes + maintenance                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Composite Index Order

```sql
-- Composite index for common query pattern
-- Query: WHERE category = ? AND price < ? ORDER BY created_at

-- ✅ Index that matches query pattern:
CREATE INDEX idx_products_cat_price_date
    ON products(category, price, created_at);

-- Index column order rules:
-- 1. Equality columns first (category = ?)
-- 2. Range/inequality columns next (price < ?)
-- 3. ORDER BY columns last (ORDER BY created_at)

-- This index helps:
SELECT * FROM products WHERE category = 'laptops' AND price < 5000;  ✅
SELECT * FROM products WHERE category = 'laptops';  ✅
SELECT * FROM products WHERE category = 'laptops' ORDER BY created_at;  ✅

-- This won't use the index fully:
SELECT * FROM products WHERE price < 5000;  ❌ (category not specified)
```

---

## 4️⃣ Practical Checklist

```
┌─────────────────────────────────────────────────────────────────────┐
│               Query Optimization Checklist                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Before Writing Query:                                               │
│  ─────────────────────                                               │
│  □ What data do I actually need?                                    │
│  □ Can I filter at database level (not in app)?                     │
│  □ What indexes exist?                                               │
│                                                                      │
│  Writing Query:                                                      │
│  ──────────────                                                      │
│  □ SELECT specific columns (not *)                                  │
│  □ Use appropriate JOINs                                             │
│  □ Add WHERE conditions to reduce result set                        │
│  □ Use LIMIT for pagination                                          │
│  □ Avoid functions on indexed columns in WHERE                      │
│                                                                      │
│  After Writing Query:                                                │
│  ────────────────────                                                │
│  □ EXPLAIN ANALYZE the query                                         │
│  □ Check for Seq Scans on large tables                              │
│  □ Verify estimated vs actual row counts                            │
│  □ Add missing indexes if needed                                    │
│  □ Test with production-like data volume                            │
│                                                                      │
│  Monitoring:                                                         │
│  ───────────                                                         │
│  □ Log slow queries (pg_stat_statements)                            │
│  □ Monitor query performance over time                              │
│  □ Run ANALYZE after bulk data changes                              │
│  □ Check for unused indexes periodically                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ PostgreSQL Tools

```sql
-- Enable timing
\timing on

-- Show execution plan
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE status = 'pending';

-- Find slow queries (requires pg_stat_statements)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

SELECT
    calls,
    round(total_exec_time::numeric, 2) as total_time_ms,
    round(mean_exec_time::numeric, 2) as avg_time_ms,
    query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Find missing indexes (approximate)
SELECT
    schemaname,
    relname as table,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    seq_tup_read / NULLIF(seq_scan, 0) as avg_rows_per_scan
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC
LIMIT 20;

-- Find unused indexes
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
AND indexrelname NOT LIKE '%_pkey';

-- Table bloat (dead rows)
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    round(100 * n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;

-- Update statistics
ANALYZE table_name;
ANALYZE;  -- all tables
```

---

## 💡 Quick Wins

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Quick Optimization Wins                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Add index on Foreign Keys                                        │
│     └─ CREATE INDEX idx_orders_user ON orders(user_id);             │
│                                                                      │
│  2. Add LIMIT to every SELECT                                        │
│     └─ SELECT * FROM orders LIMIT 100;                              │
│                                                                      │
│  3. Use EXISTS instead of COUNT for checking                        │
│     ❌ SELECT COUNT(*) FROM users WHERE id = 5 > 0                  │
│     ✅ SELECT EXISTS(SELECT 1 FROM users WHERE id = 5)              │
│                                                                      │
│  4. Cache expensive queries (Redis)                                  │
│     └─ Don't hit DB for same data repeatedly                        │
│                                                                      │
│  5. Batch operations                                                 │
│     ❌ 1000 individual INSERTs                                       │
│     ✅ 1 INSERT with 1000 rows                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **EXPLAIN ANALYZE** = أداتك الأولى لفهم الـ performance
- ✅ **Seq Scan** على جدول كبير = مشكلة
- ✅ **Index Scan** = جيد
- ✅ **أضف indexes** على FKs و WHERE و JOIN columns
- ✅ **تجنب functions** على indexed columns
- ✅ **استخدم LIMIT** دايماً
- ✅ **N+1 Problem** = مشكلة شائعة - استخدم JOINs

</div>

---

## ⏭️ Module Completed!

<div dir="rtl">

مبروك! أنت خلصت **Module 1.5: Database Concepts** 🎉

خلينا نراجع اللي اتعلمناه:
- SQL vs NoSQL
- Relational Model
- Relationships
- ACID Properties
- Transactions
- Indexes
- Normalization
- Query Optimization

**➡️ [Back to Track 1](../../README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Normalization](./07-normalization.md) | [📚 Module Home](../README.md) | [🏠 Track 1](../../README.md)

</div>
