# EXPLAIN ANALYZE - تحليل الـ Query Plans ⚡

<div dir="rtl">

## مقدمة

EXPLAIN ANALYZE بيوريك كيف PostgreSQL بينفذ الـ query وكام الوقت اللي بياخده كل خطوة. أداة أساسية لتحسين الأداء.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 EXPLAIN vs EXPLAIN ANALYZE

```sql
-- EXPLAIN: يوضح الخطة بدون تنفيذ
EXPLAIN SELECT * FROM users WHERE email = 'ahmed@example.com';

-- EXPLAIN ANALYZE: ينفذ فعلاً ويقيس الوقت
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'ahmed@example.com';

-- مع تفاصيل أكتر
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM users WHERE email = 'ahmed@example.com';
```

---

## 🔍 قراءة الـ Output

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'ahmed@example.com';
```

```
                                                    QUERY PLAN
------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_users_email on users  (cost=0.42..8.44 rows=1 width=156) (actual time=0.025..0.026 rows=1 loops=1)
   Index Cond: (email = 'ahmed@example.com'::text)
 Planning Time: 0.095 ms
 Execution Time: 0.045 ms
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Understanding the Output                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Index Scan using idx_users_email on users                         │
│   └── نوع العملية: Index Scan (جيد!)                                │
│                                                                      │
│   (cost=0.42..8.44 rows=1 width=156)                                │
│   ├── cost=0.42 : Startup cost (قبل إرجاع أول صف)                   │
│   ├── ..8.44    : Total cost (التكلفة الكلية)                       │
│   ├── rows=1    : Estimated rows (التقدير)                          │
│   └── width=156 : Average row size in bytes                        │
│                                                                      │
│   (actual time=0.025..0.026 rows=1 loops=1)                         │
│   ├── time=0.025 : Actual startup time (ms)                         │
│   ├── ..0.026    : Actual total time (ms)                           │
│   ├── rows=1     : Actual rows returned                             │
│   └── loops=1    : Number of times executed                         │
│                                                                      │
│   Planning Time: 0.095 ms   ← وقت التخطيط                           │
│   Execution Time: 0.045 ms  ← وقت التنفيذ الفعلي                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📋 أنواع الـ Scan Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Scan Types                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Seq Scan (Sequential Scan)                                         │
│  ├── يقرأ الجدول كله صف بصف                                         │
│  ├── ❌ بطيء للجداول الكبيرة                                        │
│  └── ✅ مقبول للجداول الصغيرة أو لما محتاج معظم الصفوف              │
│                                                                      │
│  Index Scan                                                          │
│  ├── يستخدم الـ index للعثور على الصفوف                             │
│  ├── ثم يقرأ البيانات من الجدول                                     │
│  └── ✅ جيد للـ queries اللي بترجع صفوف قليلة                       │
│                                                                      │
│  Index Only Scan                                                     │
│  ├── يقرأ من الـ index فقط (بدون الجدول!)                           │
│  └── ✅ الأسرع - لما كل الـ columns موجودة في الـ index             │
│                                                                      │
│  Bitmap Index Scan + Bitmap Heap Scan                               │
│  ├── يبني bitmap من الـ index                                       │
│  ├── ثم يقرأ الصفوف بترتيب فيزيائي                                  │
│  └── ✅ جيد لـ medium selectivity                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 أنواع الـ Join Operations

```sql
EXPLAIN ANALYZE
SELECT u.username, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed';
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Join Types                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Nested Loop                                                         │
│  ├── لكل صف في الجدول الأول، يبحث في الثاني                         │
│  ├── O(n × m) بدون index                                            │
│  └── ✅ جيد: جداول صغيرة أو indexed lookup                          │
│                                                                      │
│  Hash Join                                                           │
│  ├── يبني hash table من الجدول الأصغر                               │
│  ├── يبحث في الـ hash للجدول الأكبر                                 │
│  └── ✅ جيد: جداول متوسطة، equality joins                           │
│                                                                      │
│  Merge Join                                                          │
│  ├── يرتب الجدولين ويمشي عليهم بالتوازي                             │
│  └── ✅ جيد: جداول كبيرة مرتبة                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. Seq Scan (سيء للجداول الكبيرة)

</div>

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE full_name = 'Ahmed Ali';
```

```
Seq Scan on users  (cost=0.00..1234.00 rows=1 width=156) (actual time=45.123..45.234 rows=1 loops=1)
  Filter: (full_name = 'Ahmed Ali'::text)
  Rows Removed by Filter: 99999
Planning Time: 0.123 ms
Execution Time: 45.456 ms
```

```
❌ المشكلة: Seq Scan على 100,000 صف!
✅ الحل: إضافة index
```

```sql
CREATE INDEX idx_users_full_name ON users(full_name);
```

<div dir="rtl">

### 2. Index Scan (جيد)

</div>

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'ahmed@example.com';
```

```
Index Scan using idx_users_email on users  (cost=0.42..8.44 rows=1 width=156) (actual time=0.025..0.026 rows=1 loops=1)
  Index Cond: (email = 'ahmed@example.com'::text)
Planning Time: 0.095 ms
Execution Time: 0.045 ms
```

```
✅ ممتاز: استخدم الـ index، التنفيذ < 1ms
```

<div dir="rtl">

### 3. Bitmap Scan (متوسط)

</div>

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';
```

```
Bitmap Heap Scan on orders  (cost=12.34..456.78 rows=5000 width=120) (actual time=5.123..25.456 rows=4987 loops=1)
  Recheck Cond: (status = 'pending'::text)
  Heap Blocks: exact=234
  ->  Bitmap Index Scan on idx_orders_status  (cost=0.00..11.09 rows=5000 width=0) (actual time=4.567..4.567 rows=4987 loops=1)
        Index Cond: (status = 'pending'::text)
Planning Time: 0.234 ms
Execution Time: 26.789 ms
```

```
✅ جيد: Bitmap scan لـ medium selectivity
   5000 صف من أصل (مثلاً) 100,000
```

---

## 🔍 EXPLAIN Options

```sql
-- الكل مع بعض
EXPLAIN (
    ANALYZE,          -- تنفيذ فعلي
    BUFFERS,          -- I/O statistics
    COSTS,            -- التكاليف
    TIMING,           -- التوقيت
    VERBOSE,          -- تفاصيل أكتر
    FORMAT JSON       -- JSON output
)
SELECT * FROM users WHERE email = 'ahmed@example.com';
```

<div dir="rtl">

### BUFFERS output

</div>

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE id = 1;
```

```
Index Scan using users_pkey on users  (cost=0.29..8.31 rows=1 width=156) (actual time=0.015..0.016 rows=1 loops=1)
  Index Cond: (id = 1)
  Buffers: shared hit=3
Planning Time: 0.045 ms
Execution Time: 0.029 ms
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Buffer Statistics                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  shared hit=3     : 3 pages found in cache (good!)                  │
│  shared read=5    : 5 pages read from disk (slower)                 │
│  shared dirtied=2 : 2 pages modified (written later)                │
│  shared written=2 : 2 pages written to disk                         │
│                                                                      │
│  ✅ High hit ratio = good cache utilization                         │
│  ❌ High read count = might need more memory or indexes             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ⚠️ المشاكل الشائعة

<div dir="rtl">

### 1. Estimated vs Actual Rows

</div>

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;
```

```
Index Scan using idx_orders_user_id on orders  (cost=0.42..12.45 rows=10 width=120) (actual time=0.025..0.456 rows=5000 loops=1)
```

```
❌ المشكلة: التقدير (10) مختلف جداً عن الفعلي (5000)
   هذا قد يسبب خطط تنفيذ سيئة

✅ الحل: تحديث الإحصائيات
```

```sql
ANALYZE orders;
-- أو
VACUUM ANALYZE orders;
```

<div dir="rtl">

### 2. Function Calls Preventing Index Use

</div>

```sql
-- ❌ لن يستخدم index
EXPLAIN ANALYZE SELECT * FROM users WHERE LOWER(email) = 'ahmed@example.com';
-- Seq Scan...

-- ✅ الحل: Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- أو استخدم citext type
```

<div dir="rtl">

### 3. OR Conditions

</div>

```sql
-- ❌ ممكن Seq Scan
EXPLAIN ANALYZE SELECT * FROM users
WHERE email = 'ahmed@example.com' OR username = 'ahmed';

-- ✅ الحل: استخدم UNION
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'ahmed@example.com'
UNION
SELECT * FROM users WHERE username = 'ahmed';
```

---

## 🛠️ أدوات مساعدة

```sql
-- عرض إعدادات الـ planner
SHOW enable_seqscan;
SHOW enable_indexscan;
SHOW random_page_cost;
SHOW effective_cache_size;

-- تعطيل Seq Scan مؤقتاً (للاختبار فقط!)
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM users WHERE full_name = 'Ahmed';
SET enable_seqscan = on;

-- إحصائيات الجدول
SELECT
    relname,
    n_live_tup as rows,
    n_dead_tup as dead_rows,
    last_analyze
FROM pg_stat_user_tables
WHERE relname = 'users';
```

---

## 📊 Visual EXPLAIN

```sql
-- JSON format للـ visualization tools
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM users WHERE id = 1;
```

<div dir="rtl">

### أدوات مفيدة:

- **explain.depesz.com** - Online visualizer
- **pgAdmin** - Built-in graphical explain
- **DBeaver** - Visual explain plan

</div>

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                   EXPLAIN ANALYZE Best Practices                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Always use ANALYZE to get actual times                           │
│     EXPLAIN alone only shows estimates                              │
│                                                                      │
│  2. Use BUFFERS to understand I/O                                    │
│     High reads = might need more indexes or memory                  │
│                                                                      │
│  3. Compare estimated vs actual rows                                 │
│     Large difference = run ANALYZE                                  │
│                                                                      │
│  4. Look for Seq Scans on large tables                               │
│     Usually means missing index                                     │
│                                                                      │
│  5. Watch for high loops count                                       │
│     Nested loops with many iterations = slow                        │
│                                                                      │
│  6. Run multiple times                                               │
│     First run might include cache warmup                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **EXPLAIN** للتقدير، **ANALYZE** للتنفيذ الفعلي
2. راقب الفرق بين **estimated و actual rows**
3. **Seq Scan** على جداول كبيرة = مشكلة
4. استخدم **BUFFERS** لفهم الـ I/O
5. **ANALYZE** الجداول بعد تغييرات كبيرة
6. استخدم **Visual tools** لفهم أسهل

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Query Optimization](./03-query-optimization.md)**

</div>

---

<div align="center">

[⬅️ السابق](./01-index-types.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./03-query-optimization.md)

</div>
