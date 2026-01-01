# LEFT & RIGHT JOIN - الربط الخارجي ⬅️➡️

<div dir="rtl">

## مقدمة

Outer JOINs بترجع كل الصفوف من جدول واحد حتى لو مفيش match في الجدول التاني. مفيدة جداً لإيجاد البيانات الناقصة.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 كيف يعمل LEFT JOIN؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                         LEFT JOIN                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Table A (users)              Table B (orders)                      │
│  ┌────┬─────────┐            ┌────┬─────────┬─────────┐             │
│  │ id │  name   │            │ id │ user_id │ amount  │             │
│  ├────┼─────────┤            ├────┼─────────┼─────────┤             │
│  │ 1  │ Ahmed   │            │ 1  │    1    │  100    │             │
│  │ 2  │ Sara    │            │ 2  │    1    │  200    │             │
│  │ 3  │ Omar    │            │ 3  │    2    │  150    │             │
│  │ 4  │ Fatima  │            └────┴─────────┴─────────┘             │
│  └────┴─────────┘                                                   │
│                                                                      │
│  SELECT u.name, o.amount                                            │
│  FROM users u                                                        │
│  LEFT JOIN orders o ON u.id = o.user_id;                            │
│                                                                      │
│  Result:                                                            │
│  ┌─────────┬─────────┐                                              │
│  │  name   │ amount  │                                              │
│  ├─────────┼─────────┤                                              │
│  │ Ahmed   │  100    │                                              │
│  │ Ahmed   │  200    │                                              │
│  │ Sara    │  150    │                                              │
│  │ Omar    │  NULL   │  ← ظهر مع NULL لأنه مفيش match              │
│  │ Fatima  │  NULL   │  ← ظهرت مع NULL لأنه مفيش match             │
│  └─────────┴─────────┘                                              │
│                                                                      │
│  ✅ كل users ظهروا (الجدول اليسار)                                  │
│  NULL يظهر لما مفيش match في الجدول اليمين                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة

```sql
-- LEFT JOIN
SELECT columns
FROM left_table
LEFT JOIN right_table ON left_table.column = right_table.column;

-- LEFT OUTER JOIN (نفس المعنى)
SELECT columns
FROM left_table
LEFT OUTER JOIN right_table ON left_table.column = right_table.column;

-- RIGHT JOIN
SELECT columns
FROM left_table
RIGHT JOIN right_table ON left_table.column = right_table.column;
```

---

## ⬅️ LEFT JOIN أمثلة

```sql
-- كل المستخدمين وطلباتهم (حتى اللي معندهمش طلبات)
SELECT
    u.username,
    u.email,
    o.id AS order_id,
    o.total_amount,
    o.status
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- كل الفئات وعدد منتجاتها (حتى الفارغة)
SELECT
    c.name AS category,
    COUNT(p.id) AS product_count
FROM categories c
LEFT JOIN products p ON c.id = p.category_id
GROUP BY c.id, c.name;

-- المنتجات مع تقييماتها (حتى اللي ملهاش تقييمات)
SELECT
    p.name,
    p.price,
    COALESCE(ROUND(AVG(r.rating), 2), 0) AS avg_rating,
    COUNT(r.id) AS review_count
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name, p.price;
```

---

## 🔍 إيجاد البيانات الناقصة

```sql
-- المستخدمين اللي معندهمش طلبات
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;

-- الفئات الفارغة (مفيهاش منتجات)
SELECT c.*
FROM categories c
LEFT JOIN products p ON c.id = p.category_id
WHERE p.id IS NULL;

-- المنتجات اللي متباعتش
SELECT p.*
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
WHERE oi.id IS NULL;

-- المستخدمين اللي معملوش review
SELECT u.username, u.email
FROM users u
LEFT JOIN reviews r ON u.id = r.user_id
WHERE r.id IS NULL;
```

---

## ➡️ RIGHT JOIN

```sql
-- RIGHT JOIN: عكس LEFT JOIN
SELECT
    u.username,
    o.id AS order_id,
    o.total_amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- الطلبات اللي الـ user بتاعها محذوف
SELECT o.*
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id
WHERE u.id IS NULL;

-- ⚠️ معظم الناس تستخدم LEFT JOIN بدل RIGHT JOIN
-- بس تقلب الجداول
-- ده:
SELECT * FROM A RIGHT JOIN B ON ...
-- بيساوي ده:
SELECT * FROM B LEFT JOIN A ON ...
```

---

## 📊 LEFT JOIN مع عدة جداول

```sql
-- كل المستخدمين مع طلباتهم وتفاصيلها
SELECT
    u.username,
    o.id AS order_id,
    p.name AS product,
    oi.quantity
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN order_items oi ON o.id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.id;

-- تقرير المنتجات الكامل
SELECT
    c.name AS category,
    p.name AS product,
    p.price,
    COALESCE(AVG(r.rating), 0) AS avg_rating,
    COALESCE(SUM(oi.quantity), 0) AS units_sold
FROM categories c
LEFT JOIN products p ON c.id = p.category_id
LEFT JOIN reviews r ON p.id = r.product_id
LEFT JOIN order_items oi ON p.id = oi.product_id
GROUP BY c.id, c.name, p.id, p.name, p.price
ORDER BY c.name, p.name;
```

---

## 🎯 أنماط شائعة

<div dir="rtl">

### 1. Exclusive Left (اليسار فقط)

</div>

```sql
-- الصفوف اللي في A بس ومش في B
SELECT a.*
FROM table_a a
LEFT JOIN table_b b ON a.id = b.a_id
WHERE b.id IS NULL;

-- مثال: Users بدون Orders
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

<div dir="rtl">

### 2. Optional Relationship

</div>

```sql
-- المنتجات مع الخصم (الخصم optional)
SELECT
    p.name,
    p.price,
    COALESCE(d.percentage, 0) AS discount_pct,
    p.price * (1 - COALESCE(d.percentage, 0) / 100) AS final_price
FROM products p
LEFT JOIN discounts d ON p.id = d.product_id
    AND d.valid_until > NOW();
```

<div dir="rtl">

### 3. Count with Zero

</div>

```sql
-- عدد الطلبات لكل مستخدم (بما فيهم اللي عندهم 0)
SELECT
    u.username,
    COUNT(o.id) AS order_count  -- COUNT بيرجع 0 مش NULL
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username
ORDER BY order_count DESC;
```

---

## ⚠️ احترس من

<div dir="rtl">

### 1. WHERE بعد LEFT JOIN

</div>

```sql
-- ❌ غلط: WHERE بيحول LEFT JOIN لـ INNER JOIN
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed';
-- الـ users بدون orders مش هيظهروا!

-- ✅ صح: حط الشرط في ON
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
    AND o.status = 'completed';
-- الآن users بدون orders completed هيظهروا مع NULL
```

<div dir="rtl">

### 2. ترتيب الـ JOINs

</div>

```sql
-- الترتيب مهم!
-- لو عايز كل الـ users:
FROM users u
LEFT JOIN orders o ON ...

-- مش:
FROM orders o
LEFT JOIN users u ON ...  -- ده هيجيب كل الـ orders
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم COALESCE للـ NULL

</div>

```sql
SELECT
    u.username,
    COALESCE(COUNT(o.id), 0) AS order_count,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;
```

<div dir="rtl">

### 2. فضّل LEFT JOIN على RIGHT JOIN

</div>

```sql
-- ✅ أوضح
FROM users u
LEFT JOIN orders o ON u.id = o.user_id

-- ❌ أقل شيوعاً
FROM orders o
RIGHT JOIN users u ON o.user_id = u.id
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **LEFT JOIN** يرجع كل صفوف الجدول الأيسر
2. **NULL** يظهر لما مفيش match
3. **WHERE IS NULL** لإيجاد البيانات الناقصة
4. **RIGHT JOIN** = LEFT JOIN معكوس
5. انتبه لـ **WHERE بعد LEFT JOIN**

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [FULL OUTER JOIN](./06-full-outer-join.md)**

</div>

---

<div align="center">

[⬅️ السابق: INNER JOIN](./04-inner-join.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./06-full-outer-join.md)

</div>
