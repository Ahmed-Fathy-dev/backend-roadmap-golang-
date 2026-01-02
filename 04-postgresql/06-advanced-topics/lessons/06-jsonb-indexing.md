# JSONB Indexing - فهرسة JSONB 📇

<div dir="rtl">

## مقدمة

الـ indexing صحيح للـ JSONB ممكن يحول query من ثواني لـ milliseconds. هنتعلم أنواع الـ indexes وإمتى نستخدم كل نوع.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Index Types for JSONB

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JSONB Index Types                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Index Type        │ Best For                │ Operators Supported   │
│  ──────────────────┼─────────────────────────┼─────────────────────  │
│  GIN (default)     │ General JSONB queries   │ @> ? ?& ?|            │
│  GIN (jsonb_ops)   │ Same as default         │ @> ? ?& ?|            │
│  GIN (path_ops)    │ Path containment only   │ @> @?                 │
│  B-tree            │ Specific extracted val  │ = < > <= >=           │
│  Hash              │ Equality on extracted   │ =                     │
│                                                                      │
│  Recommendation:                                                     │
│  • GIN jsonb_ops: للاستعلامات المتنوعة                              │
│  • GIN path_ops: للـ containment فقط (أصغر حجماً)                   │
│  • B-tree expression: للقيم المحددة                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ GIN Index (Default - jsonb_ops)

```sql
-- إنشاء GIN index
CREATE INDEX idx_products_metadata ON products USING gin(metadata);

-- ده بيدعم:
-- @> containment
-- ? key existence
-- ?& all keys exist
-- ?| any key exists

-- Queries اللي هتستفيد من الـ index:
SELECT * FROM products WHERE metadata @> '{"brand": "Apple"}';
SELECT * FROM products WHERE metadata ? 'warranty';
SELECT * FROM products WHERE metadata ?& array['brand', 'price'];

-- ⚠️ مش بيدعم:
-- <@ (contained by) - مش هيستخدم الـ index
-- -> أو ->> extraction - لازم expression index
```

### Check Index Usage

```sql
-- نتأكد إن الـ index بيتستخدم
EXPLAIN ANALYZE
SELECT * FROM products WHERE metadata @> '{"brand": "Apple"}';

-- Output المتوقع:
-- Bitmap Heap Scan on products
--   -> Bitmap Index Scan on idx_products_metadata
--      Index Cond: (metadata @> '{"brand": "Apple"}'::jsonb)
```

---

## 2️⃣ GIN Index (jsonb_path_ops)

```sql
-- أصغر وأسرع للـ containment فقط
CREATE INDEX idx_products_metadata_path ON products
    USING gin(metadata jsonb_path_ops);

-- ده بيدعم @> فقط!
-- مش بيدعم ? أو ?& أو ?|

-- Query supported:
SELECT * FROM products WHERE metadata @> '{"brand": "Apple"}';

-- Query NOT supported (won't use this index):
SELECT * FROM products WHERE metadata ? 'brand';
```

### Comparison: jsonb_ops vs jsonb_path_ops

```sql
-- Setup للمقارنة
CREATE TABLE test_json (id serial, data jsonb);

-- Insert 1 million rows
INSERT INTO test_json (data)
SELECT jsonb_build_object(
    'field_' || i, i,
    'nested', jsonb_build_object('value', i)
)
FROM generate_series(1, 1000000) i;

-- Index sizes comparison
CREATE INDEX idx_default ON test_json USING gin(data);
CREATE INDEX idx_path ON test_json USING gin(data jsonb_path_ops);

SELECT
    indexrelname,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE relname = 'test_json';

-- Typical results:
-- idx_default: ~150MB
-- idx_path:    ~80MB (almost half!)
```

---

## 3️⃣ Expression Indexes (B-tree)

```sql
-- Index على قيمة محددة
CREATE INDEX idx_products_brand ON products((metadata ->> 'brand'));

-- الآن ده هيستخدم الـ index:
SELECT * FROM products WHERE metadata ->> 'brand' = 'Apple';

-- Index على nested value
CREATE INDEX idx_products_cpu ON products((metadata #>> '{specs,cpu}'));

SELECT * FROM products WHERE metadata #>> '{specs,cpu}' = 'M3 Pro';

-- Index على numeric value
CREATE INDEX idx_products_price ON products(((metadata ->> 'price')::numeric));

SELECT * FROM products WHERE (metadata ->> 'price')::numeric > 1000;

-- Index على boolean
CREATE INDEX idx_products_in_stock ON products(((metadata ->> 'in_stock')::boolean));

SELECT * FROM products WHERE (metadata ->> 'in_stock')::boolean = true;
```

### Partial Expression Index

```sql
-- Index فقط للمنتجات اللي عندها brand
CREATE INDEX idx_products_brand_partial ON products((metadata ->> 'brand'))
    WHERE metadata ? 'brand';

-- أصغر وأسرع من الـ full index
```

---

## 4️⃣ Composite Indexes

```sql
-- Index على أكتر من expression
CREATE INDEX idx_products_brand_price ON products(
    (metadata ->> 'brand'),
    ((metadata ->> 'price')::numeric)
);

-- Useful للـ queries:
SELECT * FROM products
WHERE metadata ->> 'brand' = 'Apple'
ORDER BY (metadata ->> 'price')::numeric;

-- أو
SELECT * FROM products
WHERE metadata ->> 'brand' = 'Apple'
  AND (metadata ->> 'price')::numeric < 1000;
```

---

## 5️⃣ Indexes على JSON Arrays

```sql
-- Table مع array
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    data JSONB  -- {"tags": ["tech", "news"], "categories": [...]}
);

-- GIN index للـ array containment
CREATE INDEX idx_articles_data ON articles USING gin(data);

-- البحث في الـ tags
SELECT * FROM articles WHERE data @> '{"tags": ["tech"]}';

-- ⚠️ لو عايز تبحث عن أي tag من list
-- ده مش هيستخدم الـ GIN index بكفاءة:
SELECT * FROM articles
WHERE data -> 'tags' ?| array['tech', 'news'];

-- الحل: index على الـ tags specifically
CREATE INDEX idx_articles_tags ON articles USING gin((data -> 'tags'));
```

---

## 📊 Index Selection Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                    When to Use Each Index                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Query Pattern                    │ Recommended Index                │
│  ─────────────────────────────────┼────────────────────────────────  │
│  metadata @> '{"key": "value"}'   │ GIN (jsonb_path_ops)            │
│  metadata ? 'key'                 │ GIN (jsonb_ops)                 │
│  metadata ?| array['a','b']       │ GIN (jsonb_ops)                 │
│  metadata ->> 'key' = 'value'     │ B-tree expression               │
│  (metadata->>'num')::int > 100    │ B-tree expression               │
│  metadata @> AND metadata ?       │ GIN (jsonb_ops)                 │
│  Frequent specific path access    │ B-tree expression               │
│  Varied/unknown queries           │ GIN (jsonb_ops)                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Real-World Examples

### Example 1: E-commerce Products

```sql
-- Products table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    base_price DECIMAL(10,2),
    attributes JSONB
);

-- الـ attributes بيحتوي:
-- {"brand": "...", "category": "...", "specs": {...}, "tags": [...]}

-- 1. GIN للـ general queries
CREATE INDEX idx_products_attrs ON products USING gin(attributes);

-- 2. Expression indexes للـ frequent filters
CREATE INDEX idx_products_brand ON products((attributes ->> 'brand'));
CREATE INDEX idx_products_category ON products((attributes ->> 'category'));

-- 3. Composite للـ filtering + sorting
CREATE INDEX idx_products_cat_brand ON products(
    (attributes ->> 'category'),
    (attributes ->> 'brand')
);

-- Typical queries:
-- Fast with GIN:
SELECT * FROM products WHERE attributes @> '{"brand": "Apple"}';

-- Fast with expression index:
SELECT * FROM products WHERE attributes ->> 'brand' = 'Apple';

-- Fast with composite:
SELECT * FROM products
WHERE attributes ->> 'category' = 'Electronics'
ORDER BY attributes ->> 'brand';
```

### Example 2: User Profiles

```sql
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    profile JSONB
);

-- {"name": "...", "settings": {...}, "social": {...}, "verified": true}

-- Index للـ verified users (partial)
CREATE INDEX idx_verified_users ON user_profiles((profile ->> 'name'))
    WHERE (profile ->> 'verified')::boolean = true;

-- Query المستفيد:
SELECT profile ->> 'name'
FROM user_profiles
WHERE (profile ->> 'verified')::boolean = true;
```

### Example 3: Event Logs

```sql
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(50),
    payload JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Composite: event_type + time + JSONB
CREATE INDEX idx_events_type_time ON events(event_type, created_at DESC);
CREATE INDEX idx_events_payload ON events USING gin(payload jsonb_path_ops);

-- Index على specific fields
CREATE INDEX idx_events_user_id ON events((payload ->> 'user_id'))
    WHERE payload ? 'user_id';

-- Query:
SELECT * FROM events
WHERE event_type = 'purchase'
  AND created_at > NOW() - INTERVAL '7 days'
  AND payload @> '{"status": "completed"}';
```

---

## 📈 Index Maintenance

```sql
-- Check index sizes
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE tablename = 'products'
ORDER BY pg_relation_size(indexrelid) DESC;

-- Check index usage
SELECT
    indexrelname,
    idx_scan as scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
WHERE relname = 'products';

-- Find unused indexes
SELECT
    schemaname || '.' || relname as table,
    indexrelname as index,
    pg_size_pretty(pg_relation_size(indexrelid)) as size,
    idx_scan as scans
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE '%_pkey';

-- Reindex if needed
REINDEX INDEX idx_products_metadata;
-- أو
REINDEX TABLE products;
```

---

## ⚠️ Common Mistakes

```sql
-- ❌ Mistake 1: Index على <@ operator
CREATE INDEX idx_wrong ON products USING gin(metadata);
SELECT * FROM products WHERE metadata <@ '{"brand": "Apple"}'::jsonb;
-- هذا الـ query مش هيستخدم الـ GIN index!

-- ✅ Solution: rewrite using @>
SELECT * FROM products WHERE '{"brand": "Apple"}'::jsonb @> metadata;
-- لكن ده semantically مختلف!

-- ❌ Mistake 2: Expression مش matching
CREATE INDEX idx_brand ON products((metadata ->> 'brand'));
SELECT * FROM products WHERE metadata -> 'brand' = '"Apple"';
-- مش هيستخدم الـ index! لازم ->> مش ->

-- ✅ Correct:
SELECT * FROM products WHERE metadata ->> 'brand' = 'Apple';

-- ❌ Mistake 3: Casting mismatch
CREATE INDEX idx_price ON products(((metadata ->> 'price')::numeric));
SELECT * FROM products WHERE (metadata ->> 'price')::int > 100;
-- int مش numeric! مش هيستخدم الـ index

-- ✅ Correct:
SELECT * FROM products WHERE (metadata ->> 'price')::numeric > 100;
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  JSONB Indexing Best Practices                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Start with GIN (jsonb_ops) للـ flexibility                      │
│     Add expression indexes للـ hot paths لاحقاً                     │
│                                                                      │
│  2. استخدم jsonb_path_ops لو بتستخدم @> فقط                         │
│     أصغر 40-50% من jsonb_ops                                        │
│                                                                      │
│  3. Expression indexes للـ:                                          │
│     - Equality checks على specific paths                            │
│     - Range queries على numeric values                              │
│     - Sorting على JSON fields                                        │
│                                                                      │
│  4. Partial indexes للـ:                                             │
│     - Keys اللي مش موجودة في كل الـ rows                            │
│     - Filtering على subset معين                                     │
│                                                                      │
│  5. Monitor index usage                                              │
│     Remove unused indexes (بتبطئ writes)                            │
│                                                                      │
│  6. EXPLAIN ANALYZE كل query مهم                                     │
│     تأكد إن الـ index بيتستخدم                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **GIN jsonb_ops** الـ default - بيدعم كل operators
2. **GIN path_ops** أصغر لكن @> فقط
3. **Expression indexes** للقيم المحددة والـ sorting
4. **Partial indexes** لتوفير المساحة
5. **EXPLAIN ANALYZE** دايماً للتأكد
6. **Monitor** الـ indexes وشيل الـ unused

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Full-Text Search](./07-full-text-search.md)**

</div>

---

<div align="center">

[⬅️ السابق](./05-jsonb-operators.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./07-full-text-search.md)

</div>
