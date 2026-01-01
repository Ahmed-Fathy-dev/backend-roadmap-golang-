# INSERT ... RETURNING - إرجاع البيانات 🔙

<div dir="rtl">

## مقدمة

`RETURNING` هي ميزة قوية في PostgreSQL بتسمحلك ترجّع البيانات اللي تم إدخالها مباشرةً - مفيدة جداً للحصول على الـ ID أو القيم المُحسّبة.

**المدة المتوقعة:** 15 دقيقة

</div>

---

## 📝 الصيغة

```sql
INSERT INTO table_name (columns)
VALUES (values)
RETURNING column1, column2, ... | *;
```

---

## 🔢 إرجاع الـ ID

<div dir="rtl">

### الحصول على الـ ID المُولّد

</div>

```sql
-- إرجاع الـ ID فقط
INSERT INTO users (username, email, password_hash)
VALUES ('ahmed', 'ahmed@example.com', 'hash123')
RETURNING id;

-- النتيجة:
--  id
-- ----
--  1
```

<div dir="rtl">

### لماذا RETURNING أفضل من SELECT بعد INSERT؟

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│  بدون RETURNING (طريقتين سيئتين):                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  الطريقة 1: lastval() أو currval()                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  INSERT INTO users (...) VALUES (...);                       │    │
│  │  SELECT lastval();  -- أو currval('users_id_seq')            │    │
│  │                                                               │    │
│  │  ⚠️ المشكلة: Race condition ممكن يحصل                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  الطريقة 2: SELECT بعد INSERT                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  INSERT INTO users (username, ...) VALUES ('ahmed', ...);    │    │
│  │  SELECT id FROM users WHERE username = 'ahmed';              │    │
│  │                                                               │    │
│  │  ⚠️ المشكلة: ممكن يرجع أكتر من صف، أو يخطئ                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│  مع RETURNING (الطريقة الصحيحة):                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  INSERT INTO users (...) VALUES (...)                        │    │
│  │  RETURNING id;                                                │    │
│  │                                                               │    │
│  │  ✅ Atomic: في statement واحد                                │    │
│  │  ✅ آمن: مفيش race conditions                                │    │
│  │  ✅ دقيق: بيرجع الـ ID الصحيح بالظبط                         │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 إرجاع أعمدة محددة

```sql
-- إرجاع عدة أعمدة
INSERT INTO users (username, email, full_name, password_hash)
VALUES ('sara', 'sara@example.com', 'Sara Mohamed', 'hash456')
RETURNING id, username, created_at;

-- النتيجة:
--  id | username |          created_at
-- ----+----------+-------------------------------
--   2 | sara     | 2024-12-21 14:30:00.123456+02

-- إرجاع مع alias
INSERT INTO orders (user_id, total_amount, status)
VALUES (1, 299.99, 'pending')
RETURNING
    id AS order_id,
    total_amount AS total,
    created_at AS ordered_at;
```

---

## ⭐ إرجاع كل الأعمدة

```sql
-- إرجاع كل الأعمدة باستخدام *
INSERT INTO products (sku, name, price, stock)
VALUES ('NEW001', 'New Product', 149.99, 25)
RETURNING *;

-- النتيجة:
--  id |  sku   |    name     | price  | stock | is_available |        created_at
-- ----+--------+-------------+--------+-------+--------------+---------------------------
--   1 | NEW001 | New Product | 149.99 |    25 | t            | 2024-12-21 14:30:00+02
```

---

## 🔢 RETURNING مع Bulk Insert

```sql
-- إرجاع IDs لكل الصفوف المُدخلة
INSERT INTO products (sku, name, price) VALUES
    ('BULK001', 'Product 1', 99.99),
    ('BULK002', 'Product 2', 149.99),
    ('BULK003', 'Product 3', 199.99)
RETURNING id, sku;

-- النتيجة:
--  id |   sku
-- ----+---------
--   1 | BULK001
--   2 | BULK002
--   3 | BULK003
```

---

## 🧮 RETURNING مع Expressions

```sql
-- إرجاع قيم محسوبة
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1, 5, 3, 99.99)
RETURNING
    id,
    quantity,
    unit_price,
    quantity * unit_price AS line_total;

-- النتيجة:
--  id | quantity | unit_price | line_total
-- ----+----------+------------+------------
--   1 |        3 |      99.99 |     299.97

-- إرجاع مع functions
INSERT INTO users (username, email, password_hash)
VALUES ('newuser', 'new@example.com', 'hash')
RETURNING
    id,
    UPPER(username) AS username_upper,
    LENGTH(email) AS email_length,
    created_at::DATE AS created_date;
```

---

## 🔗 استخدام RETURNING في CTE

<div dir="rtl">

### إدخال وقراءة في Query واحد

</div>

```sql
-- إنشاء order واستخدام الـ id لإنشاء order_items
WITH new_order AS (
    INSERT INTO orders (user_id, status, total_amount)
    VALUES (1, 'pending', 0)
    RETURNING id
)
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
SELECT new_order.id, 5, 2, 99.99
FROM new_order
RETURNING *;

-- سيناريو أكثر تعقيداً: إنشاء user و profile
WITH new_user AS (
    INSERT INTO users (username, email, password_hash)
    VALUES ('complete_user', 'complete@example.com', 'hash')
    RETURNING id, username
)
INSERT INTO user_profiles (user_id, bio, avatar_url)
SELECT id, 'New user bio', 'default_avatar.png'
FROM new_user
RETURNING *;
```

---

## 📊 استخدام RETURNING في Application Code

```go
// Go - استخدام RETURNING
var id int
err := db.QueryRow(
    `INSERT INTO users (username, email, password_hash)
     VALUES ($1, $2, $3)
     RETURNING id`,
    username, email, passwordHash,
).Scan(&id)

if err != nil {
    return fmt.Errorf("failed to insert user: %w", err)
}
fmt.Printf("Created user with ID: %d\n", id)
```

```python
# Python - استخدام RETURNING
cursor.execute("""
    INSERT INTO users (username, email, password_hash)
    VALUES (%s, %s, %s)
    RETURNING id, created_at
""", (username, email, password_hash))

result = cursor.fetchone()
user_id = result[0]
created_at = result[1]
```

```javascript
// Node.js - استخدام RETURNING
const result = await pool.query(
    `INSERT INTO users (username, email, password_hash)
     VALUES ($1, $2, $3)
     RETURNING id, created_at`,
    [username, email, passwordHash]
);

const { id, created_at } = result.rows[0];
```

---

## 💡 أمثلة عملية

<div dir="rtl">

### 1. إنشاء طلب كامل

</div>

```sql
-- Step 1: إنشاء الطلب والحصول على ID
WITH new_order AS (
    INSERT INTO orders (user_id, status, shipping_address)
    VALUES (1, 'pending', '123 Main St')
    RETURNING id, created_at
),
-- Step 2: إضافة المنتجات للطلب
order_products AS (
    INSERT INTO order_items (order_id, product_id, quantity, unit_price)
    SELECT
        new_order.id,
        p.id,
        c.quantity,
        p.price
    FROM new_order
    CROSS JOIN (
        VALUES (1, 2), (3, 1), (5, 3)  -- (product_id, quantity)
    ) AS c(product_id, quantity)
    JOIN products p ON p.id = c.product_id
    RETURNING order_id, quantity * unit_price AS line_total
)
-- Step 3: تحديث إجمالي الطلب
UPDATE orders
SET total_amount = (SELECT SUM(line_total) FROM order_products)
WHERE id = (SELECT DISTINCT order_id FROM order_products)
RETURNING id, total_amount;
```

<div dir="rtl">

### 2. تسجيل مستخدم مع إرسال email

</div>

```sql
-- إنشاء المستخدم وإرجاع البيانات اللازمة للـ email
INSERT INTO users (username, email, full_name, password_hash, verification_token)
VALUES (
    'new_user',
    'newuser@example.com',
    'New User',
    'hashed_password',
    gen_random_uuid()
)
RETURNING id, email, full_name, verification_token;

-- النتيجة تُستخدم لإرسال email التحقق
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **RETURNING** ترجّع البيانات بعد INSERT مباشرةً
2. **أفضل من SELECT** بعد INSERT - atomic وآمن
3. ممكن ترجّع **أعمدة محددة** أو **كل الأعمدة** (*)
4. تعمل مع **Bulk Insert** وترجع كل الصفوف
5. ممكن تستخدمها في **CTEs** لعمليات معقدة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [INSERT ... ON CONFLICT (Upsert)](./03-upsert.md)**

</div>

---

<div align="center">

[⬅️ السابق: INSERT الأساسي](./01-basic-insert.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./03-upsert.md)

</div>
