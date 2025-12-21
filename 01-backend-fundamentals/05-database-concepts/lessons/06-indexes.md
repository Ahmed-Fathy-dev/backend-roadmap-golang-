# Lesson 6: Indexes في Databases ⚡

<div dir="rtl">

## المقدمة

**Index** هو السر لجعل Database **سريع**!

بدون indexes = slow queries 🐌  
مع indexes = fast queries ⚡

</div>

---

## 🎯 What is an Index?

<div dir="rtl">

**Index** مثل فهرس الكتاب - يساعدك تلاقي الصفحة بسرعة بدل ما تقرأ الكتاب كله.

</div>

---

## 📊 Performance Impact

### Without Index:

```sql
SELECT * FROM users WHERE email = 'ahmed@test.com';

→ Full Table Scan: checks ALL 1,000,000 rows 😱
→ Time: ~500ms
```

### With Index:

```sql
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'ahmed@test.com';

→ Index Lookup: finds row instantly ⚡
→ Time: ~2ms (250x faster!)
```

---

## 🔧 Creating Indexes

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (multiple columns)
CREATE INDEX idx_products_category_price ON products(category, price);

-- Unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

---

## ✅ When to Create Indexes

```sql
-- ✅ Columns in WHERE clause
CREATE INDEX idx_products_category ON products(category);
SELECT * FROM products WHERE category = 'laptops';

-- ✅ Foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- ✅ Columns in ORDER BY
CREATE INDEX idx_products_created_at ON products(created_at);
SELECT * FROM products ORDER BY created_at DESC;

-- ✅ Columns in JOIN
CREATE INDEX idx_posts_user_id ON posts(user_id);
SELECT * FROM users u JOIN posts p ON p.user_id = u.id;
```

---

## ❌ When NOT to Index

```
❌ Small tables (< 1000 rows)
❌ Columns with low cardinality (boolean, status)
❌ Columns that change frequently
❌ Rarely queried columns
```

---

## 📋 Index Types

### 1. B-Tree (Default)

```sql
CREATE INDEX idx_name ON table(column);

-- Best for:
- Equality: column = value
- Range: column > value, column BETWEEN x AND y
- Sorting: ORDER BY column
```

### 2. Hash

```sql
CREATE INDEX idx_name ON table USING HASH (column);

-- Best for:
- Equality only: column = value
-- NOT for range or sorting
```

---

## 💡 Composite Indexes

```sql
-- Order matters!
CREATE INDEX idx_products_cat_price ON products(category, price);

-- ✅ Will use index:
SELECT * FROM products WHERE category = 'laptops' AND price > 500;
SELECT * FROM products WHERE category = 'laptops';

-- ❌ Won't use index fully:
SELECT * FROM products WHERE price > 500;  -- category not in WHERE
```

---

## 🔍 Check Index Usage

```sql
-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM products WHERE category = 'laptops';

-- Shows:
-- - Seq Scan (bad - not using index)
-- - Index Scan (good - using index)
```

---

## ⚖️ Trade-offs

**Pros:**

- ✅ Faster SELECT queries
- ✅ Faster JOIN operations
- ✅ Faster ORDER BY

**Cons:**

- ❌ Slower INSERT/UPDATE/DELETE
- ❌ More disk space
- ❌ Maintenance overhead

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ Index على columns في **WHERE, JOIN, ORDER BY**
- ✅ Index على **Foreign Keys**
- ✅ **Composite indexes** order matters
- ✅ استخدم **EXPLAIN** لفحص queries
- ✅ Don't over-index!

</div>

---

<div align="center">

[⬅️ Previous: Transactions](./05-transactions.md) | [➡️ Next: Normalization](./07-normalization.md) | [📚 Module Home](../README.md)

</div>
