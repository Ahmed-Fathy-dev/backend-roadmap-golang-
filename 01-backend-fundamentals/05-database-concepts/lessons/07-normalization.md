# Lesson 7: Normalization 📊

<div dir="rtl">

## المقدمة

**Normalization** = تنظيم البيانات لتقليل التكرار.

</div>

---

## ❌ Problem: Data Duplication

```sql
-- Bad: Denormalized
orders table:
┌────┬──────────┬────────────┬──────────────┬───────┐
│ id │ customer │ product    │ product_desc │ price │
├────┼──────────┼────────────┼──────────────┼───────┤
│ 1  │ Ahmed    │ Laptop     │ Gaming       │ 5000  │
│ 2  │ Sara     │ Laptop     │ Gaming       │ 5000  │ ← Duplicate!
│ 3  │ Omar     │ Laptop     │ Gaming       │ 5000  │ ← Duplicate!
└────┴──────────┴────────────┴──────────────┴───────┘

Problems:
❌ Wastes space (Laptop info repeated 3 times)
❌ Update anomaly (change price → must update all 3 rows!)
❌ Inconsistency risk
```

---

## ✅ Solution: Normalized

```sql
-- Good: Normalized
products table:
┌────┬────────┬──────────┬───────┐
│ id │  name  │   desc   │ price │
├────┼────────┼──────────┼───────┤
│ 1  │ Laptop │ Gaming   │ 5000  │
└────┴────────┴──────────┴───────┘

orders table:
┌────┬────────────┬────────────┐
│ id │ customer   │ product_id │
├────┼────────────┼────────────┤
│ 1  │ Ahmed      │ 1          │
│ 2  │ Sara       │ 1          │ ← Just reference!
│ 3  │ Omar       │ 1          │
└────┴────────────┴────────────┘

Benefits:
✅ No duplication
✅ Update price once
✅ Data consistency
```

---

## 📋 Normal Forms

### 1NF (First Normal Form)

```
✅ No repeating groups
✅ Atomic values (no lists in one cell)
```

```sql
-- ❌ Violates 1NF:
CREATE TABLE users (
    id INT,
    name VARCHAR,
    phones VARCHAR  -- "123,456,789" → Multiple values!
);

-- ✅ 1NF:
CREATE TABLE users (
    id INT,
    name VARCHAR
);

CREATE TABLE user_phones (
    user_id INT,
    phone VARCHAR  -- One value per row
);
```

### 2NF (Second Normal Form)

```
✅ Must be in 1NF
✅ No partial dependencies
```

### 3NF (Third Normal Form)

```
✅ Must be in 2NF
✅ No transitive dependencies
```

---

## ⚖️ Denormalization

<div dir="rtl">

**أحياناً** نعكس Normalization للأداء!

</div>

```sql
-- Normalized: Requires JOIN
SELECT u.name, p.title
FROM users u
JOIN posts p ON p.user_id = u.id;

-- Denormalized: Faster (duplicate user name)
SELECT name, title FROM posts;  -- No JOIN needed!
```

**When to denormalize:**

- ✅ Read-heavy workloads
- ✅ Performance critical
- ✅ Acceptable data duplication

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Normalization:** تقليل التكرار
- ✅ **1NF, 2NF, 3NF:** مستويات Normalization
- ✅ **Benefits:** لا تكرار، consistency
- ✅ **Trade-off:** أحياناً denormalize للأداء

</div>

---

<div align="center">

[⬅️ Previous: Indexes](./06-indexes.md) | [📚 Module Home](../README.md)

</div>
