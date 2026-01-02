# Index Types - أنواع الـ Indexes 📊

<div dir="rtl">

## مقدمة

الـ Indexes هي هياكل بيانات بتسرّع عمليات البحث في الـ database. PostgreSQL بيوفر أنواع مختلفة من الـ indexes لـ use cases مختلفة.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 ما هو الـ Index؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Without Index                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   SELECT * FROM users WHERE email = 'ahmed@example.com'             │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                     Table Scan                               │   │
│   │   Row 1: john@... ❌                                         │   │
│   │   Row 2: sara@... ❌                                         │   │
│   │   Row 3: ahmed@... ✅ Found!                                 │   │
│   │   Row 4: omar@... (still scanning...)                        │   │
│   │   ...                                                        │   │
│   │   Row 1,000,000: last@... ❌                                 │   │
│   │                                                               │   │
│   │   ⏱️ O(n) - Scans ALL rows!                                  │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                       With Index                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────┐    ┌───────────────────────────────────┐     │
│   │   B-tree Index  │    │         Table                      │     │
│   │                 │    │                                     │     │
│   │  ahmed@ ──────────────▶ Row 3 ✅                           │     │
│   │  john@ ─────────│    │                                     │     │
│   │  omar@ ─────────│    │                                     │     │
│   │  sara@ ─────────│    │                                     │     │
│   │                 │    │                                     │     │
│   └─────────────────┘    └───────────────────────────────────┘     │
│                                                                      │
│   ⚡ O(log n) - Much faster!                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🌳 B-tree Index (Default)

<div dir="rtl">

الـ B-tree هو الـ default index في PostgreSQL. مناسب لمعظم الحالات.

</div>

```sql
-- إنشاء B-tree index
CREATE INDEX idx_users_email ON users(email);

-- أو صراحة
CREATE INDEX idx_users_email ON users USING btree(email);

-- Composite index (multiple columns)
CREATE INDEX idx_users_name ON users(last_name, first_name);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Partial index (on subset of data)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- Index with expression
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

<div dir="rtl">

### متى تستخدم B-tree؟

</div>

```
✅ Equality comparisons:    WHERE email = 'ahmed@example.com'
✅ Range queries:           WHERE created_at > '2024-01-01'
✅ Sorting:                 ORDER BY created_at DESC
✅ BETWEEN:                 WHERE age BETWEEN 18 AND 65
✅ IS NULL / IS NOT NULL:   WHERE deleted_at IS NULL
✅ LIKE prefix:             WHERE name LIKE 'Ahmed%'
❌ LIKE suffix:             WHERE name LIKE '%ahmed' (لا يستخدم index!)
```

---

## 🔢 Hash Index

<div dir="rtl">

أسرع من B-tree للـ equality comparisons فقط.

</div>

```sql
-- إنشاء Hash index
CREATE INDEX idx_users_email_hash ON users USING hash(email);
```

<div dir="rtl">

### متى تستخدم Hash؟

</div>

```
✅ Equality comparisons:    WHERE email = 'ahmed@example.com'
❌ Range queries:           WHERE created_at > '2024-01-01'  (لا يعمل!)
❌ Sorting:                 ORDER BY                          (لا يعمل!)
❌ LIKE:                    WHERE name LIKE 'Ahmed%'          (لا يعمل!)

📝 Note: In modern PostgreSQL (10+), Hash indexes are WAL-logged and safe.
         But B-tree is usually preferred for its flexibility.
```

---

## 📝 GIN Index (Generalized Inverted Index)

<div dir="rtl">

مثالي للـ arrays، JSONB، و full-text search.

</div>

```sql
-- للـ Arrays
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    tags TEXT[]
);

CREATE INDEX idx_articles_tags ON articles USING gin(tags);

-- Query
SELECT * FROM articles WHERE tags @> ARRAY['postgresql'];

-- للـ JSONB
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    data JSONB
);

CREATE INDEX idx_products_data ON products USING gin(data);

-- Query
SELECT * FROM products WHERE data @> '{"category": "electronics"}';

-- للـ Full-text search
CREATE INDEX idx_articles_content ON articles USING gin(to_tsvector('english', content));
```

<div dir="rtl">

### متى تستخدم GIN؟

</div>

```
✅ Array containment:       WHERE tags @> ARRAY['go']
✅ JSONB containment:       WHERE data @> '{"key": "value"}'
✅ Full-text search:        WHERE to_tsvector(content) @@ to_tsquery('go')
✅ Array overlap:           WHERE tags && ARRAY['go', 'python']

⚠️ GIN indexes are:
   - Slower to build and update
   - Larger in size
   - But MUCH faster for these query types
```

---

## 🗺️ GiST Index (Generalized Search Tree)

<div dir="rtl">

للـ geometric data، ranges، و full-text search.

</div>

```sql
-- للـ Geometric data
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name TEXT,
    coordinates POINT
);

CREATE INDEX idx_locations_coords ON locations USING gist(coordinates);

-- Find points within a box
SELECT * FROM locations
WHERE coordinates <@ box '((0,0),(10,10))';

-- للـ Range types
CREATE TABLE reservations (
    id SERIAL PRIMARY KEY,
    room_id INT,
    during TSTZRANGE
);

CREATE INDEX idx_reservations_during ON reservations USING gist(during);

-- Find overlapping reservations
SELECT * FROM reservations
WHERE during && '[2024-01-01, 2024-01-07)';

-- للـ Full-text search (alternative to GIN)
CREATE INDEX idx_articles_fts ON articles USING gist(to_tsvector('english', content));
```

<div dir="rtl">

### GiST vs GIN للـ Full-text Search

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                       GiST vs GIN                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Feature              │   GIN         │   GiST                     │
│   ─────────────────────┼───────────────┼───────────────────────     │
│   Build speed          │   Slower      │   Faster                   │
│   Update speed         │   Slower      │   Faster                   │
│   Query speed          │   Faster      │   Slower                   │
│   Index size           │   Larger      │   Smaller                  │
│                                                                      │
│   Use GIN for:  Read-heavy, exact matches                           │
│   Use GiST for: Write-heavy, range queries                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔤 BRIN Index (Block Range Index)

<div dir="rtl">

Index صغير جداً للجداول الكبيرة المرتبة طبيعياً.

</div>

```sql
-- مثالي للـ time-series data
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_time TIMESTAMPTZ DEFAULT NOW(),
    event_type TEXT,
    data JSONB
);

-- BRIN index (much smaller than B-tree!)
CREATE INDEX idx_events_time ON events USING brin(event_time);

-- Query
SELECT * FROM events
WHERE event_time BETWEEN '2024-01-01' AND '2024-01-31';
```

<div dir="rtl">

### متى تستخدم BRIN؟

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                       When to Use BRIN                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ✅ GOOD for:                                                       │
│   ├── Very large tables (millions of rows)                          │
│   ├── Naturally ordered data (timestamps, sequential IDs)           │
│   ├── Append-only tables (logs, events)                             │
│   └── When index size is a concern                                  │
│                                                                      │
│   ❌ NOT good for:                                                   │
│   ├── Randomly ordered data                                         │
│   ├── Frequently updated data                                       │
│   └── Point queries (exact match)                                   │
│                                                                      │
│   📊 Size comparison for 10M rows:                                   │
│   ├── B-tree:  ~200 MB                                              │
│   └── BRIN:    ~50 KB (4000x smaller!)                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📋 Index Types Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Index Types Summary                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Type    │ Use Case                      │ Operators               │
│   ────────┼───────────────────────────────┼──────────────────────   │
│   B-tree  │ General purpose (default)     │ <, <=, =, >=, >, LIKE   │
│   Hash    │ Equality only                 │ =                        │
│   GIN     │ Arrays, JSONB, Full-text      │ @>, &&, @@               │
│   GiST    │ Geometric, Ranges, Full-text  │ <@, &&, @>               │
│   BRIN    │ Large sequential tables       │ <, <=, =, >=, >          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 إدارة الـ Indexes

```sql
-- عرض كل الـ indexes
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'users';

-- حجم الـ indexes
SELECT pg_size_pretty(pg_indexes_size('users')) as index_size;

-- حجم كل index
SELECT
    indexrelname as index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE relname = 'users'
ORDER BY pg_relation_size(indexrelid) DESC;

-- إعادة بناء index
REINDEX INDEX idx_users_email;

-- إعادة بناء كل indexes الجدول
REINDEX TABLE users;

-- حذف index
DROP INDEX idx_users_email;

-- إنشاء index بدون قفل الجدول (CONCURRENTLY)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
```

---

## ⚠️ متى لا تستخدم Index؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                    When NOT to Index                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ❌ Small tables (< 1000 rows)                                      │
│      Table scan is faster than index lookup                         │
│                                                                      │
│   ❌ Low selectivity columns                                         │
│      e.g., gender (only M/F), boolean flags                         │
│      Exception: partial index on the rare value                     │
│                                                                      │
│   ❌ Write-heavy tables with rare reads                              │
│      Indexes slow down INSERT/UPDATE/DELETE                         │
│                                                                      │
│   ❌ Columns that are rarely queried                                 │
│      Waste of space and maintenance                                 │
│                                                                      │
│   📊 Rule of thumb:                                                  │
│      Index columns that appear in WHERE, JOIN, ORDER BY             │
│      Only if queries are slow and table is large                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Best Practices

```sql
-- 1. Index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 2. Use partial indexes for common filters
CREATE INDEX idx_active_orders ON orders(created_at)
WHERE status = 'pending';

-- 3. Use expression indexes for function calls
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- 4. Consider covering indexes (include extra columns)
CREATE INDEX idx_users_email ON users(email) INCLUDE (username, full_name);

-- 5. Use CONCURRENTLY in production
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- 6. Analyze after creating index
ANALYZE users;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **B-tree** الـ default ومناسب لمعظم الحالات
2. **GIN** للـ arrays، JSONB، full-text
3. **GiST** للـ geometric و ranges
4. **BRIN** للجداول الكبيرة المرتبة
5. استخدم **partial indexes** للفلاتر الشائعة
6. استخدم **CONCURRENTLY** في production

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [EXPLAIN ANALYZE](./02-explain-analyze.md)**

</div>

---

<div align="center">

[🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./02-explain-analyze.md)

</div>
