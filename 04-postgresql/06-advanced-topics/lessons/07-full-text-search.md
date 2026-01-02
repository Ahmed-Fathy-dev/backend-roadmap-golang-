# Full-Text Search - البحث النصي الكامل 🔍

<div dir="rtl">

## مقدمة

PostgreSQL بيقدم Full-Text Search قوي ومدمج. بيدعم البحث بالـ stemming، ranking، وحتى البحث fuzzy. بديل قوي لـ Elasticsearch للتطبيقات المتوسطة.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 Full-Text Search Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Full-Text Search Flow                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Document Text                                                      │
│        │                                                             │
│        ▼                                                             │
│   ┌─────────────┐                                                   │
│   │  to_tsvector │  ← Convert to searchable tokens                  │
│   └─────────────┘                                                   │
│        │                                                             │
│        ▼                                                             │
│   tsvector: 'postgresql':1 'databas':2 'power':3                   │
│                                                                      │
│   Search Query                                                       │
│        │                                                             │
│        ▼                                                             │
│   ┌─────────────┐                                                   │
│   │  to_tsquery  │  ← Convert to search query                       │
│   └─────────────┘                                                   │
│        │                                                             │
│        ▼                                                             │
│   tsquery: 'postgresql' & 'power'                                   │
│                                                                      │
│   Match: tsvector @@ tsquery                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Basic Full-Text Search

### tsvector و tsquery

```sql
-- تحويل نص لـ tsvector
SELECT to_tsvector('english', 'PostgreSQL is a powerful database system');
-- Result: 'databas':5 'postgresql':1 'power':4 'system':6

-- لاحظ:
-- 1. الكلمات اتحولت لـ stems (powerful → power)
-- 2. Stop words اتشالت (is, a)
-- 3. كل كلمة معاها position

-- تحويل بحث لـ tsquery
SELECT to_tsquery('english', 'powerful & database');
-- Result: 'power' & 'databas'

-- البحث
SELECT to_tsvector('english', 'PostgreSQL is a powerful database')
    @@ to_tsquery('english', 'powerful & database');
-- Result: true
```

### البحث في جدول

```sql
-- جدول المقالات
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    body TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insert sample data
INSERT INTO articles (title, body) VALUES
('Introduction to PostgreSQL', 'PostgreSQL is a powerful open-source database system with many advanced features.'),
('Database Design Best Practices', 'Good database design is essential for building scalable applications.'),
('PostgreSQL Full-Text Search', 'Learn how to implement full-text search in PostgreSQL for your applications.');

-- البحث الأساسي
SELECT title, body
FROM articles
WHERE to_tsvector('english', title || ' ' || body)
    @@ to_tsquery('english', 'postgresql');

-- البحث مع AND
SELECT title FROM articles
WHERE to_tsvector('english', title || ' ' || body)
    @@ to_tsquery('english', 'postgresql & search');

-- البحث مع OR
SELECT title FROM articles
WHERE to_tsvector('english', title || ' ' || body)
    @@ to_tsquery('english', 'design | search');
```

---

## 2️⃣ Stored tsvector Column

```sql
-- إضافة column للـ tsvector
ALTER TABLE articles ADD COLUMN search_vector tsvector;

-- Update الـ column
UPDATE articles
SET search_vector = to_tsvector('english', title || ' ' || body);

-- إنشاء trigger للتحديث التلقائي
CREATE OR REPLACE FUNCTION articles_search_trigger()
RETURNS trigger AS $$
BEGIN
    NEW.search_vector := to_tsvector('english', NEW.title || ' ' || NEW.body);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER articles_search_update
    BEFORE INSERT OR UPDATE ON articles
    FOR EACH ROW
    EXECUTE FUNCTION articles_search_trigger();

-- أو استخدم tsvector_update_trigger المدمج
CREATE TRIGGER articles_search_update
    BEFORE INSERT OR UPDATE ON articles
    FOR EACH ROW
    EXECUTE FUNCTION tsvector_update_trigger(search_vector, 'pg_catalog.english', title, body);
```

### البحث باستخدام الـ stored column

```sql
-- أسرع بكتير!
SELECT title, body
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql');
```

---

## 3️⃣ GIN Index للـ Full-Text Search

```sql
-- إنشاء GIN index
CREATE INDEX idx_articles_search ON articles USING gin(search_vector);

-- الآن البحث سريع جداً
EXPLAIN ANALYZE
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql');

-- Output:
-- Bitmap Heap Scan on articles
--   -> Bitmap Index Scan on idx_articles_search
```

### Expression Index (بدون stored column)

```sql
-- لو مش عايز تضيف column
CREATE INDEX idx_articles_search_expr ON articles
    USING gin(to_tsvector('english', title || ' ' || body));

-- ⚠️ لازم الـ query يطابق الـ expression بالظبط
SELECT title FROM articles
WHERE to_tsvector('english', title || ' ' || body)
    @@ to_tsquery('english', 'postgresql');
```

---

## 4️⃣ Search Query Operators

```sql
-- AND: كل الكلمات
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql & database');

-- OR: أي كلمة
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql | mysql');

-- NOT: استثناء
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'database & !mysql');

-- Phrase search (adjacent words)
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'full <-> text');
-- full-text متجاورين

-- Distance operator
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql <2> features');
-- postgresql وبعدها features خلال كلمتين

-- Prefix search
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'post:*');
-- كل الكلمات اللي بتبدأ بـ post
```

### plainto_tsquery و phraseto_tsquery

```sql
-- plainto_tsquery: يحول النص لـ AND query
SELECT plainto_tsquery('english', 'powerful database');
-- Result: 'power' & 'databas'

-- phraseto_tsquery: يحول لـ phrase query
SELECT phraseto_tsquery('english', 'full text search');
-- Result: 'full' <-> 'text' <-> 'search'

-- websearch_to_tsquery: مثل Google search
SELECT websearch_to_tsquery('english', 'postgresql -mysql "full text"');
-- Result: 'postgresql' & !'mysql' & 'full' <-> 'text'
```

---

## 5️⃣ Ranking Results

```sql
-- ts_rank: ranking بسيط
SELECT
    title,
    ts_rank(search_vector, to_tsquery('english', 'postgresql')) as rank
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql')
ORDER BY rank DESC;

-- ts_rank_cd: cover density ranking
SELECT
    title,
    ts_rank_cd(search_vector, to_tsquery('english', 'postgresql & database')) as rank
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql & database')
ORDER BY rank DESC;

-- Ranking مع weights
-- A = 1.0, B = 0.4, C = 0.2, D = 0.1
-- Title = A, Body = D
ALTER TABLE articles DROP COLUMN search_vector;
ALTER TABLE articles ADD COLUMN search_vector tsvector;

UPDATE articles
SET search_vector =
    setweight(to_tsvector('english', title), 'A') ||
    setweight(to_tsvector('english', body), 'B');

-- الآن matches في الـ title عندها weight أعلى
SELECT
    title,
    ts_rank(search_vector, query) as rank
FROM articles, to_tsquery('english', 'postgresql') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

---

## 6️⃣ Highlighting Results

```sql
-- ts_headline: highlight matching words
SELECT
    title,
    ts_headline('english', body, to_tsquery('english', 'postgresql'),
        'StartSel=<b>, StopSel=</b>, MaxWords=35, MinWords=15') as excerpt
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql');

-- Result:
-- title: Introduction to PostgreSQL
-- excerpt: <b>PostgreSQL</b> is a powerful open-source database system...

-- Options:
-- StartSel, StopSel: العلامات
-- MaxWords: أقصى عدد كلمات
-- MinWords: أقل عدد كلمات
-- MaxFragments: عدد الـ fragments
-- FragmentDelimiter: الفاصل بين الـ fragments
```

---

## 7️⃣ Arabic Full-Text Search

```sql
-- PostgreSQL مش بيدعم Arabic by default
-- لكن ممكن نستخدم 'simple' configuration

SELECT to_tsvector('simple', 'مرحبا بكم في PostgreSQL');
-- Result: 'postgresql':4 'بكم':2 'في':3 'مرحبا':1

-- أو install Arabic dictionary
-- CREATE EXTENSION IF NOT EXISTS pg_arabic;

-- Workaround: استخدم simple + trigram
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- للبحث العربي البسيط
SELECT * FROM articles
WHERE title ILIKE '%بحث%'
   OR body ILIKE '%بحث%';

-- مع trigram index
CREATE INDEX idx_articles_title_trgm ON articles USING gin(title gin_trgm_ops);
CREATE INDEX idx_articles_body_trgm ON articles USING gin(body gin_trgm_ops);
```

---

## 8️⃣ Fuzzy Search with pg_trgm

```sql
-- Install extension
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Similarity search
SELECT title, similarity(title, 'Postgre') as sim
FROM articles
WHERE title % 'Postgre'  -- % means similarity > threshold
ORDER BY sim DESC;

-- Set threshold
SET pg_trgm.similarity_threshold = 0.3;

-- Word similarity
SELECT title, word_similarity('database', title) as sim
FROM articles
ORDER BY sim DESC;

-- Index للـ similarity
CREATE INDEX idx_articles_title_trgm ON articles USING gin(title gin_trgm_ops);

-- أو gist للـ nearest neighbor
CREATE INDEX idx_articles_title_gist ON articles USING gist(title gist_trgm_ops);

-- Query
SELECT title FROM articles
WHERE title % 'Postgre'
ORDER BY title <-> 'Postgre'
LIMIT 5;
```

---

## 📦 Complete Search Implementation

```sql
-- Products table مع full-text search
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    category VARCHAR(50),
    price DECIMAL(10, 2),
    search_vector tsvector,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Trigger للتحديث التلقائي
CREATE OR REPLACE FUNCTION products_search_trigger()
RETURNS trigger AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', COALESCE(NEW.name, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.category, '')), 'B') ||
        setweight(to_tsvector('english', COALESCE(NEW.description, '')), 'C');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER products_search_update
    BEFORE INSERT OR UPDATE OF name, description, category ON products
    FOR EACH ROW
    EXECUTE FUNCTION products_search_trigger();

-- Indexes
CREATE INDEX idx_products_search ON products USING gin(search_vector);
CREATE INDEX idx_products_name_trgm ON products USING gin(name gin_trgm_ops);

-- Search function
CREATE OR REPLACE FUNCTION search_products(
    search_query TEXT,
    category_filter VARCHAR DEFAULT NULL,
    min_price DECIMAL DEFAULT NULL,
    max_price DECIMAL DEFAULT NULL,
    page_size INT DEFAULT 20,
    page_num INT DEFAULT 1
)
RETURNS TABLE (
    id INT,
    name VARCHAR,
    description TEXT,
    category VARCHAR,
    price DECIMAL,
    rank REAL,
    headline TEXT
) AS $$
DECLARE
    query tsquery;
    offset_val INT;
BEGIN
    -- Convert search to tsquery
    query := websearch_to_tsquery('english', search_query);
    offset_val := (page_num - 1) * page_size;

    RETURN QUERY
    SELECT
        p.id,
        p.name,
        p.description,
        p.category,
        p.price,
        ts_rank(p.search_vector, query) as rank,
        ts_headline('english', p.description, query,
            'StartSel=<mark>, StopSel=</mark>, MaxWords=30') as headline
    FROM products p
    WHERE p.search_vector @@ query
        AND (category_filter IS NULL OR p.category = category_filter)
        AND (min_price IS NULL OR p.price >= min_price)
        AND (max_price IS NULL OR p.price <= max_price)
    ORDER BY rank DESC
    LIMIT page_size
    OFFSET offset_val;
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT * FROM search_products('wireless headphones', 'Electronics', 50, 200);
```

---

## 🔧 Configuration Options

```sql
-- Available configurations
SELECT cfgname FROM pg_ts_config;
-- simple, english, arabic, french, german, ...

-- Check current configuration
SHOW default_text_search_config;

-- Set default
SET default_text_search_config = 'english';

-- Configuration details
SELECT
    alias,
    description
FROM ts_debug('english', 'PostgreSQL databases are powerful');

-- Dictionaries
SELECT dictname FROM pg_ts_dict;

-- Custom configuration (advanced)
CREATE TEXT SEARCH CONFIGURATION my_english (COPY = english);
ALTER TEXT SEARCH CONFIGURATION my_english
    ALTER MAPPING FOR word WITH simple;
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                Full-Text Search Best Practices                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use stored tsvector column                                       │
│     أسرع من حسابها في كل query                                      │
│                                                                      │
│  2. Use GIN index                                                    │
│     Essential للـ performance                                        │
│                                                                      │
│  3. Use weights for ranking                                          │
│     Title (A) > Tags (B) > Description (C)                          │
│                                                                      │
│  4. Combine with pg_trgm                                             │
│     للـ fuzzy search وtypo tolerance                                 │
│                                                                      │
│  5. Use websearch_to_tsquery                                         │
│     User-friendly search syntax                                      │
│                                                                      │
│  6. Limit headline processing                                        │
│     ts_headline is expensive                                         │
│                                                                      │
│  7. Consider Elasticsearch for:                                      │
│     - Very large datasets (billions)                                 │
│     - Complex analytics                                              │
│     - Multi-language (especially Arabic)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **tsvector** للـ documents، **tsquery** للـ search
2. استخدم **stored column** + **GIN index**
3. **Weights** للـ ranking (A, B, C, D)
4. **ts_headline** للـ highlighting
5. **pg_trgm** للـ fuzzy search
6. **websearch_to_tsquery** لـ Google-like syntax

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Views](./08-views.md)**

</div>

---

<div align="center">

[⬅️ السابق](./06-jsonb-indexing.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./08-views.md)

</div>
