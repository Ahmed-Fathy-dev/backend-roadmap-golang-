# INNER JOIN - الربط الداخلي 🔗

<div dir="rtl">

## مقدمة

`INNER JOIN` بيرجع الصفوف اللي ليها match في الجدولين. لو مفيش match، الصف مش بيظهر.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 كيف يعمل INNER JOIN؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                        INNER JOIN                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Table A (users)              Table B (orders)                      │
│  ┌────┬─────────┐            ┌────┬─────────┬─────────┐             │
│  │ id │  name   │            │ id │ user_id │ amount  │             │
│  ├────┼─────────┤            ├────┼─────────┼─────────┤             │
│  │ 1  │ Ahmed   │            │ 1  │    1    │  100    │             │
│  │ 2  │ Sara    │            │ 2  │    1    │  200    │             │
│  │ 3  │ Omar    │            │ 3  │    2    │  150    │             │
│  │ 4  │ Fatima  │            │ 4  │    5    │  300    │             │
│  └────┴─────────┘            └────┴─────────┴─────────┘             │
│                                                                      │
│  SELECT u.name, o.amount                                            │
│  FROM users u                                                        │
│  INNER JOIN orders o ON u.id = o.user_id;                           │
│                                                                      │
│  Result:                                                            │
│  ┌─────────┬─────────┐                                              │
│  │  name   │ amount  │                                              │
│  ├─────────┼─────────┤                                              │
│  │ Ahmed   │  100    │  ← user 1 له order                          │
│  │ Ahmed   │  200    │  ← user 1 له order تاني                     │
│  │ Sara    │  150    │  ← user 2 له order                          │
│  └─────────┴─────────┘                                              │
│                                                                      │
│  ❌ Omar (id=3) مش ظاهر - معندوش orders                             │
│  ❌ Fatima (id=4) مش ظاهرة - معندهاش orders                         │
│  ❌ Order 4 (user_id=5) مش ظاهر - مفيش user بـ id=5                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;

-- أو بدون كلمة INNER (نفس المعنى)
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;
```

---

## 🔢 أمثلة أساسية

```sql
-- المنتجات مع أسماء الفئات
SELECT
    p.name AS product_name,
    p.price,
    c.name AS category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.id;

-- الطلبات مع أسماء المستخدمين
SELECT
    o.id AS order_id,
    u.username,
    u.email,
    o.total_amount,
    o.status
FROM orders o
INNER JOIN users u ON o.user_id = u.id;

-- تفاصيل الطلب مع أسماء المنتجات
SELECT
    oi.order_id,
    p.name AS product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.id
WHERE oi.order_id = 1;
```

---

## 🔗 INNER JOIN مع عدة جداول

```sql
-- الطلبات مع المستخدمين والمنتجات
SELECT
    o.id AS order_id,
    u.username AS customer,
    p.name AS product,
    oi.quantity,
    oi.unit_price
FROM orders o
INNER JOIN users u ON o.user_id = u.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;

-- المنتجات مع الفئات والتقييمات
SELECT
    p.name AS product,
    c.name AS category,
    r.rating,
    r.comment,
    u.username AS reviewer
FROM products p
INNER JOIN categories c ON p.category_id = c.id
INNER JOIN reviews r ON p.id = r.product_id
INNER JOIN users u ON r.user_id = u.id;
```

---

## 📊 INNER JOIN مع Aggregations

```sql
-- عدد الطلبات لكل مستخدم
SELECT
    u.username,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM users u
INNER JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;

-- متوسط تقييم كل منتج
SELECT
    p.name AS product,
    ROUND(AVG(r.rating), 2) AS avg_rating,
    COUNT(r.id) AS review_count
FROM products p
INNER JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name
HAVING COUNT(r.id) >= 3
ORDER BY avg_rating DESC;

-- المبيعات حسب الفئة
SELECT
    c.name AS category,
    COUNT(DISTINCT o.id) AS order_count,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM categories c
INNER JOIN products p ON c.id = p.category_id
INNER JOIN order_items oi ON p.id = oi.product_id
INNER JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY c.id, c.name
ORDER BY revenue DESC;
```

---

## 🎯 INNER JOIN مع WHERE

```sql
-- طلبات مستخدم معين
SELECT o.id, o.total_amount, o.status
FROM orders o
INNER JOIN users u ON o.user_id = u.id
WHERE u.username = 'ahmed';

-- المنتجات في فئة معينة بتقييم عالي
SELECT
    p.name,
    p.price,
    AVG(r.rating) AS avg_rating
FROM products p
INNER JOIN categories c ON p.category_id = c.id
INNER JOIN reviews r ON p.id = r.product_id
WHERE c.name = 'Electronics'
GROUP BY p.id, p.name, p.price
HAVING AVG(r.rating) >= 4;

-- الطلبات في فترة معينة
SELECT
    o.id,
    u.username,
    o.total_amount,
    o.created_at
FROM orders o
INNER JOIN users u ON o.user_id = u.id
WHERE o.created_at BETWEEN '2024-01-01' AND '2024-12-31'
ORDER BY o.created_at DESC;
```

---

## 🔄 USING بدل ON

```sql
-- لو اسم العمود متطابق
-- بدل:
SELECT * FROM orders o
INNER JOIN users u ON o.user_id = u.id;

-- ممكن (لو user_id موجود في الاتنين):
SELECT * FROM orders o
INNER JOIN users u USING (user_id);
-- بس ده نادر لأن عادةً user_id في orders و id في users

-- مثال أفضل:
SELECT * FROM order_items oi
INNER JOIN orders o USING (order_id);
-- لو الجدولين فيهم order_id
```

---

## 📊 Natural JOIN

```sql
-- Natural JOIN: بيربط تلقائياً بالأعمدة المتطابقة الاسم
SELECT *
FROM orders
NATURAL JOIN users;
-- ⚠️ خطر! لو فيه أعمدة بنفس الاسم مش المقصودة

-- ❌ مش مفضل - استخدم INNER JOIN ON صراحةً
-- ✅ أفضل:
SELECT *
FROM orders o
INNER JOIN users u ON o.user_id = u.id;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم Aliases واضحة

</div>

```sql
-- ✅ واضح
SELECT
    u.username,
    o.total_amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- ❌ مش واضح
SELECT username, total_amount
FROM users, orders
WHERE users.id = orders.user_id;
```

<div dir="rtl">

### 2. حدد الأعمدة صراحةً

</div>

```sql
-- ✅ كويس
SELECT u.id, u.username, o.id AS order_id, o.total_amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- ❌ مش كويس
SELECT *
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

<div dir="rtl">

### 3. Index على الـ Foreign Keys

</div>

```sql
-- ✅ أنشئ index للأداء
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **INNER JOIN** يرجع فقط الصفوف اللي ليها match
2. **ON** لتحديد شرط الربط
3. ممكن تربط **عدة جداول** في query واحد
4. استخدم **Aliases** للوضوح
5. **Index** على Foreign Keys للأداء

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [LEFT & RIGHT JOIN](./05-outer-joins.md)**

</div>

---

<div align="center">

[⬅️ السابق: Referential Integrity](./03-referential-integrity.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./05-outer-joins.md)

</div>
