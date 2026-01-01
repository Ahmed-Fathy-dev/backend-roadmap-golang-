# INSERT الأساسي - إدخال البيانات ➕

<div dir="rtl">

## مقدمة

`INSERT` هو أمر إضافة بيانات جديدة للجدول. في الدرس ده هنتعلم كل طرق الإدخال من البسيطة للمتقدمة.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

---

## 🔢 إدخال صف واحد

<div dir="rtl">

### تحديد الأعمدة صراحةً (الطريقة المفضلة)

</div>

```sql
-- إدخال مستخدم جديد
INSERT INTO users (username, email, full_name, password_hash)
VALUES ('ahmed', 'ahmed@example.com', 'Ahmed Ali', 'hashed_password_here');

-- إدخال منتج جديد
INSERT INTO products (sku, name, price, stock, category_id)
VALUES ('LAPTOP001', 'MacBook Pro 14"', 1999.99, 50, 1);

-- إدخال طلب جديد
INSERT INTO orders (user_id, status, total_amount, shipping_address)
VALUES (1, 'pending', 2499.98, '123 Main St, Cairo, Egypt');
```

<div dir="rtl">

### بدون تحديد الأعمدة (مش مفضل)

</div>

```sql
-- ⚠️ لازم القيم تكون بنفس ترتيب الأعمدة في الجدول
INSERT INTO users
VALUES (DEFAULT, 'sara', 'sara@example.com', 'Sara Mohamed', 'hash', TRUE, 'user', NOW(), NOW());

-- ❌ مشاكل هذه الطريقة:
-- 1. لو تغير ترتيب الأعمدة، الـ query هيبوظ
-- 2. لازم تحط كل الأعمدة حتى لو عندها default
-- 3. صعب القراءة والفهم
```

---

## 📦 إدخال عدة صفوف (Bulk Insert)

<div dir="rtl">

### طريقة واحدة فعالة

</div>

```sql
-- إدخال عدة مستخدمين في statement واحد
INSERT INTO users (username, email, full_name, password_hash) VALUES
    ('ahmed', 'ahmed@example.com', 'Ahmed Ali', 'hash1'),
    ('sara', 'sara@example.com', 'Sara Mohamed', 'hash2'),
    ('omar', 'omar@example.com', 'Omar Hassan', 'hash3'),
    ('fatima', 'fatima@example.com', 'Fatima Ibrahim', 'hash4'),
    ('khaled', 'khaled@example.com', 'Khaled Mahmoud', 'hash5');

-- إدخال منتجات
INSERT INTO products (sku, name, price, stock, category_id) VALUES
    ('PHONE001', 'iPhone 15 Pro', 999.99, 100, 2),
    ('PHONE002', 'Samsung S24 Ultra', 899.99, 80, 2),
    ('PHONE003', 'Google Pixel 8', 699.99, 60, 2),
    ('LAPTOP002', 'Dell XPS 15', 1499.99, 40, 1),
    ('LAPTOP003', 'ThinkPad X1 Carbon', 1299.99, 35, 1);
```

<div dir="rtl">

### مقارنة الأداء

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Single vs Bulk Insert                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Single Inserts (1000 صف):                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  INSERT INTO ... VALUES (...);  -- Round trip 1              │    │
│  │  INSERT INTO ... VALUES (...);  -- Round trip 2              │    │
│  │  INSERT INTO ... VALUES (...);  -- Round trip 3              │    │
│  │  ...                                                          │    │
│  │  INSERT INTO ... VALUES (...);  -- Round trip 1000           │    │
│  │                                                               │    │
│  │  الوقت: ~5-10 ثواني                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Bulk Insert (1000 صف):                                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  INSERT INTO ... VALUES                                       │    │
│  │      (...), (...), (...), ...;  -- Round trip واحد فقط!      │    │
│  │                                                               │    │
│  │  الوقت: ~100-200 مللي ثانية                                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ⚡ Bulk Insert أسرع 50x-100x من Single Inserts!                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 القيم الافتراضية (DEFAULT)

```sql
-- استخدام DEFAULT صراحةً
INSERT INTO products (sku, name, price, stock, is_available)
VALUES ('TEST001', 'Test Product', 99.99, DEFAULT, DEFAULT);
-- stock = 0 (default), is_available = TRUE (default)

-- عدم تحديد العمود = استخدام DEFAULT تلقائياً
INSERT INTO products (sku, name, price)
VALUES ('TEST002', 'Another Product', 49.99);
-- stock, is_available, created_at, updated_at كلهم هياخدوا default

-- كل القيم default (نادر)
INSERT INTO test_table DEFAULT VALUES;
```

---

## 🔢 أنواع البيانات المختلفة

```sql
-- Numbers
INSERT INTO products (sku, name, price, stock)
VALUES ('NUM001', 'Number Test', 1234.56, 100);

-- Strings (النصوص)
INSERT INTO users (username, email, full_name, password_hash)
VALUES ('test_user', 'test@test.com', 'Test User', 'password_hash');

-- Boolean
INSERT INTO users (username, email, full_name, password_hash, is_active)
VALUES ('inactive_user', 'inactive@test.com', 'Inactive', 'hash', FALSE);

-- Dates
INSERT INTO events (name, event_date, event_time)
VALUES ('Conference', '2024-06-15', '09:00:00');

-- Timestamp
INSERT INTO logs (message, created_at)
VALUES ('User logged in', '2024-12-21 14:30:00+02');

-- Timestamp with NOW()
INSERT INTO logs (message, created_at)
VALUES ('Current time log', NOW());

-- NULL
INSERT INTO users (username, email, full_name, password_hash)
VALUES ('no_name', 'noname@test.com', NULL, 'hash');

-- Arrays (PostgreSQL)
INSERT INTO products (sku, name, price, tags)
VALUES ('ARR001', 'Tagged Product', 99.99, ARRAY['electronics', 'sale', 'new']);

-- JSON/JSONB
INSERT INTO products (sku, name, price, metadata)
VALUES ('JSON001', 'JSON Product', 149.99, '{"color": "red", "size": "large"}'::jsonb);

-- UUID
INSERT INTO sessions (id, user_id, token)
VALUES (gen_random_uuid(), 1, 'session_token_here');
```

---

## ⚠️ التعامل مع الأخطاء الشائعة

<div dir="rtl">

### 1. مخالفة UNIQUE Constraint

</div>

```sql
-- لو حاولت تدخل username موجود
INSERT INTO users (username, email, full_name, password_hash)
VALUES ('ahmed', 'new@test.com', 'New User', 'hash');
-- ERROR: duplicate key value violates unique constraint "users_username_key"

-- الحل: تأكد إن القيمة مش موجودة أو استخدم ON CONFLICT
```

<div dir="rtl">

### 2. مخالفة NOT NULL Constraint

</div>

```sql
-- لو حاولت تدخل NULL في عمود NOT NULL
INSERT INTO users (username, email, full_name, password_hash)
VALUES ('test', NULL, 'Test', 'hash');
-- ERROR: null value in column "email" violates not-null constraint

-- الحل: حط قيمة صحيحة
```

<div dir="rtl">

### 3. مخالفة Foreign Key Constraint

</div>

```sql
-- لو حاولت تربط بـ id مش موجود
INSERT INTO orders (user_id, status, total_amount)
VALUES (999, 'pending', 100.00);
-- ERROR: insert or update violates foreign key constraint

-- الحل: تأكد إن الـ user_id موجود في جدول users
```

<div dir="rtl">

### 4. مخالفة CHECK Constraint

</div>

```sql
-- لو عندك CHECK constraint على السعر
-- CHECK (price > 0)
INSERT INTO products (sku, name, price)
VALUES ('NEG001', 'Negative Price', -50.00);
-- ERROR: new row violates check constraint "products_price_check"

-- الحل: تأكد إن القيمة تحقق الشرط
```

---

## 🔒 الأمان: تجنب SQL Injection

<div dir="rtl">

### ❌ الطريقة الخطرة (لا تستخدمها أبداً!)

</div>

```python
# Python - الطريقة الخطرة
username = user_input  # ممكن يكون: "'; DROP TABLE users; --"
query = f"INSERT INTO users (username) VALUES ('{username}')"
# النتيجة: INSERT INTO users (username) VALUES (''; DROP TABLE users; --')
```

<div dir="rtl">

### ✅ الطريقة الآمنة (Parameterized Queries)

</div>

```python
# Python - الطريقة الآمنة
cursor.execute(
    "INSERT INTO users (username, email) VALUES (%s, %s)",
    (username, email)
)
```

```go
// Go - الطريقة الآمنة
_, err := db.Exec(
    "INSERT INTO users (username, email) VALUES ($1, $2)",
    username, email,
)
```

```javascript
// Node.js - الطريقة الآمنة
await pool.query(
    'INSERT INTO users (username, email) VALUES ($1, $2)',
    [username, email]
);
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. حدد الأعمدة دائماً

</div>

```sql
-- ✅ صح
INSERT INTO users (username, email, password_hash)
VALUES ('test', 'test@test.com', 'hash');

-- ❌ غلط
INSERT INTO users VALUES (DEFAULT, 'test', 'test@test.com', ...);
```

<div dir="rtl">

### 2. استخدم Bulk Insert للبيانات الكثيرة

</div>

```sql
-- ✅ سريع
INSERT INTO logs (message) VALUES
    ('msg1'), ('msg2'), ('msg3'), ...;

-- ❌ بطيء
INSERT INTO logs (message) VALUES ('msg1');
INSERT INTO logs (message) VALUES ('msg2');
INSERT INTO logs (message) VALUES ('msg3');
```

<div dir="rtl">

### 3. استخدم Transaction للعمليات المرتبطة

</div>

```sql
BEGIN;
INSERT INTO orders (user_id, total_amount) VALUES (1, 100) RETURNING id;
-- لو الـ id = 5
INSERT INTO order_items (order_id, product_id, quantity) VALUES (5, 1, 2);
INSERT INTO order_items (order_id, product_id, quantity) VALUES (5, 2, 1);
COMMIT;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **حدد الأعمدة صراحةً** في INSERT
2. **Bulk Insert** أسرع بكتير من إدخالات فردية
3. **DEFAULT** للقيم الافتراضية
4. **Parameterized Queries** للأمان
5. انتبه للـ **Constraints** (UNIQUE, NOT NULL, FK, CHECK)

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [INSERT ... RETURNING](./02-insert-returning.md)**

</div>

---

<div align="center">

[🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./02-insert-returning.md)

</div>
