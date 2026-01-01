# INSERT - إدخال البيانات ➕

<div dir="rtl">

## مقدمة

`INSERT` هو الأمر المسؤول عن إضافة بيانات جديدة للجداول. في الدرس ده هنتعلم كل طرق الإدخال من الأساسية للمتقدمة.

**المدة المتوقعة:** 20-25 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

<div dir="rtl">

### مثال بسيط

</div>

```sql
-- إنشاء جدول للتجربة
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    age INT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- إدخال صف واحد
INSERT INTO users (username, email, age)
VALUES ('ahmed', 'ahmed@example.com', 25);

-- التحقق
SELECT * FROM users;
```

---

## 📋 طرق الإدخال المختلفة

<div dir="rtl">

### 1. إدخال كل الأعمدة

</div>

```sql
-- تحديد كل الأعمدة (الأفضل)
INSERT INTO users (username, email, age, created_at)
VALUES ('sara', 'sara@example.com', 28, NOW());

-- بدون تحديد الأعمدة (مش مفضل)
INSERT INTO users
VALUES (DEFAULT, 'omar', 'omar@example.com', 30, NOW());
-- لازم ترتيب القيم يطابق ترتيب الأعمدة بالظبط
-- ولازم تحط DEFAULT للـ SERIAL
```

<div dir="rtl">

### 2. إدخال أعمدة محددة

</div>

```sql
-- الأعمدة اللي مش موجودة هتاخد default أو NULL
INSERT INTO users (username, email)
VALUES ('fatima', 'fatima@example.com');
-- age = NULL, created_at = NOW()
```

<div dir="rtl">

### 3. إدخال عدة صفوف (Bulk Insert)

</div>

```sql
-- طريقة واحدة فعالة
INSERT INTO users (username, email, age) VALUES
    ('ali', 'ali@example.com', 22),
    ('mona', 'mona@example.com', 27),
    ('khaled', 'khaled@example.com', 35),
    ('nour', 'nour@example.com', 24),
    ('hassan', 'hassan@example.com', 31);

-- أسرع بكتير من إدخال كل صف لوحده!
```

---

## 🔙 INSERT ... RETURNING

<div dir="rtl">

### إرجاع البيانات المُدخلة

RETURNING بيرجّعلك البيانات بعد الإدخال - مفيد جداً للحصول على الـ ID!

</div>

```sql
-- إرجاع الـ ID
INSERT INTO users (username, email)
VALUES ('test_user', 'test@example.com')
RETURNING id;
-- Result: id = 10

-- إرجاع كل الأعمدة
INSERT INTO users (username, email, age)
VALUES ('new_user', 'new@example.com', 25)
RETURNING *;

-- إرجاع أعمدة محددة
INSERT INTO users (username, email)
VALUES ('another', 'another@example.com')
RETURNING id, username, created_at;

-- إرجاع مع alias
INSERT INTO users (username, email)
VALUES ('demo', 'demo@example.com')
RETURNING id AS user_id, created_at AS registered_at;

-- مع bulk insert
INSERT INTO users (username, email) VALUES
    ('user1', 'user1@example.com'),
    ('user2', 'user2@example.com'),
    ('user3', 'user3@example.com')
RETURNING id, username;
```

---

## ⚔️ INSERT ... ON CONFLICT (Upsert)

<div dir="rtl">

### التعامل مع التكرار

"Upsert" = UPDATE or INSERT - لو موجود حدّث، لو مش موجود أضف.

</div>

```sql
-- إنشاء جدول مع UNIQUE constraint
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    stock INT DEFAULT 0,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ON CONFLICT DO NOTHING (تجاهل لو موجود)
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU001', 'Laptop', 999.99, 10)
ON CONFLICT (sku) DO NOTHING;

-- لو SKU001 موجود، مش هيحصل حاجة
-- لو مش موجود، هيتضاف

-- ON CONFLICT DO UPDATE (تحديث لو موجود)
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU001', 'Laptop Pro', 1199.99, 15)
ON CONFLICT (sku) DO UPDATE SET
    name = EXCLUDED.name,
    price = EXCLUDED.price,
    stock = EXCLUDED.stock,
    updated_at = NOW();

-- EXCLUDED = القيم الجديدة اللي حاولنا نضيفها
```

<div dir="rtl">

### ON CONFLICT مع شرط

</div>

```sql
-- تحديث بشرط (لو السعر الجديد أعلى)
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Laptop', 899.99)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price
WHERE products.price < EXCLUDED.price;

-- تحديث أعمدة معينة فقط
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU001', 'Laptop Updated', 999.99, 20)
ON CONFLICT (sku) DO UPDATE SET
    stock = products.stock + EXCLUDED.stock,  -- زيادة المخزون
    updated_at = NOW();
```

<div dir="rtl">

### ON CONFLICT على Constraint معين

</div>

```sql
-- استخدام اسم الـ constraint
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 100)
ON CONFLICT ON CONSTRAINT products_sku_key DO UPDATE SET
    price = EXCLUDED.price;

-- أو على PRIMARY KEY
INSERT INTO users (id, username, email)
VALUES (1, 'updated_name', 'updated@email.com')
ON CONFLICT (id) DO UPDATE SET
    username = EXCLUDED.username,
    email = EXCLUDED.email;
```

---

## 📥 INSERT ... SELECT

<div dir="rtl">

### الإدخال من Query آخر

</div>

```sql
-- نسخ بيانات من جدول لجدول
INSERT INTO users_backup (username, email, created_at)
SELECT username, email, created_at
FROM users
WHERE is_active = true;

-- نسخ مع تعديل
INSERT INTO archived_orders (order_id, total, archived_at)
SELECT id, total_amount, NOW()
FROM orders
WHERE created_at < '2024-01-01';

-- نسخ من عدة جداول
INSERT INTO order_summary (user_name, order_count, total_spent)
SELECT
    u.username,
    COUNT(o.id),
    SUM(o.total_amount)
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;

-- مع RETURNING
INSERT INTO users_backup (username, email)
SELECT username, email FROM users WHERE is_active = false
RETURNING id, username;
```

---

## 🔢 إدخال أنواع بيانات مختلفة

<div dir="rtl">

### الأنواع الأساسية

</div>

```sql
CREATE TABLE examples (
    id SERIAL PRIMARY KEY,
    -- Numbers
    int_col INT,
    decimal_col NUMERIC(10, 2),
    -- Text
    text_col VARCHAR(100),
    -- Boolean
    bool_col BOOLEAN,
    -- Date/Time
    date_col DATE,
    time_col TIME,
    timestamp_col TIMESTAMPTZ,
    -- Special
    uuid_col UUID,
    json_col JSONB,
    array_col TEXT[]
);

INSERT INTO examples (
    int_col, decimal_col, text_col, bool_col,
    date_col, time_col, timestamp_col,
    uuid_col, json_col, array_col
) VALUES (
    42,                              -- INT
    99.99,                           -- NUMERIC
    'Hello World',                   -- VARCHAR
    TRUE,                            -- BOOLEAN
    '2024-12-21',                    -- DATE
    '14:30:00',                      -- TIME
    '2024-12-21 14:30:00+02',        -- TIMESTAMPTZ
    gen_random_uuid(),               -- UUID
    '{"key": "value", "num": 123}',  -- JSONB
    ARRAY['tag1', 'tag2', 'tag3']    -- TEXT[]
);
```

<div dir="rtl">

### NULL values

</div>

```sql
-- إدخال NULL صراحةً
INSERT INTO users (username, email, age)
VALUES ('test', 'test@test.com', NULL);

-- أو عدم تحديد العمود
INSERT INTO users (username, email)
VALUES ('test2', 'test2@test.com');
-- age سيكون NULL
```

---

## 🔐 الأمان والـ SQL Injection

<div dir="rtl">

### ❌ الطريقة الخطرة

</div>

```sql
-- في الـ Application Code (خطر!)
query = "INSERT INTO users (username) VALUES ('" + user_input + "')"
-- لو user_input = "'; DROP TABLE users; --"
-- النتيجة: INSERT INTO users (username) VALUES (''; DROP TABLE users; --')
```

<div dir="rtl">

### ✅ الطريقة الآمنة (Parameterized Queries)

</div>

```go
// Go code
_, err := db.Exec(
    "INSERT INTO users (username, email) VALUES ($1, $2)",
    username,  // $1
    email,     // $2
)
```

```python
# Python code
cursor.execute(
    "INSERT INTO users (username, email) VALUES (%s, %s)",
    (username, email)
)
```

---

## 📊 Performance Tips

<div dir="rtl">

### 1. استخدم Bulk Insert

</div>

```sql
-- ❌ بطيء (1000 round trip)
INSERT INTO logs (message) VALUES ('msg1');
INSERT INTO logs (message) VALUES ('msg2');
-- ... 998 more

-- ✅ سريع (1 round trip)
INSERT INTO logs (message) VALUES
    ('msg1'), ('msg2'), ('msg3'), ...;

-- أو باستخدام COPY (الأسرع)
COPY logs (message) FROM '/path/to/data.csv' CSV;
```

<div dir="rtl">

### 2. استخدم COPY للبيانات الكبيرة

</div>

```sql
-- COPY من ملف
COPY users (username, email, age)
FROM '/tmp/users.csv'
WITH (FORMAT CSV, HEADER true);

-- COPY من stdin
COPY users (username, email) FROM stdin;
ahmed    ahmed@test.com
sara     sara@test.com
\.

-- COPY لملف (export)
COPY users TO '/tmp/users_export.csv' WITH (FORMAT CSV, HEADER true);
```

<div dir="rtl">

### 3. تعطيل Triggers/Indexes مؤقتاً

</div>

```sql
-- للبيانات الكبيرة جداً
BEGIN;

-- تعطيل الـ triggers
ALTER TABLE huge_table DISABLE TRIGGER ALL;

-- إدراج البيانات
INSERT INTO huge_table ...;

-- إعادة تفعيل الـ triggers
ALTER TABLE huge_table ENABLE TRIGGER ALL;

COMMIT;

-- إعادة بناء الـ indexes
REINDEX TABLE huge_table;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. حدد الأعمدة دايماً

</div>

```sql
-- ✅ كويس
INSERT INTO users (username, email) VALUES ('test', 'test@test.com');

-- ❌ وحش
INSERT INTO users VALUES (DEFAULT, 'test', 'test@test.com', NULL, NOW());
```

<div dir="rtl">

### 2. استخدم RETURNING للحصول على ID

</div>

```sql
-- ✅ استخدم RETURNING
INSERT INTO orders (user_id, total)
VALUES (1, 100)
RETURNING id;

-- ❌ لا تعمل SELECT بعد INSERT
INSERT INTO orders (user_id, total) VALUES (1, 100);
SELECT id FROM orders WHERE ... -- معقد وممكن يرجع غلط
```

<div dir="rtl">

### 3. استخدم ON CONFLICT للـ Upsert

</div>

```sql
-- ✅ في statement واحد
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 100)
ON CONFLICT (sku) DO UPDATE SET price = EXCLUDED.price;

-- ❌ لا تعمل SELECT ثم INSERT أو UPDATE
SELECT * FROM products WHERE sku = 'SKU001';
-- IF exists: UPDATE
-- ELSE: INSERT
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **حدد الأعمدة** صراحةً في INSERT
2. **RETURNING** للحصول على القيم بعد الإدخال
3. **ON CONFLICT** للتعامل مع التكرار (Upsert)
4. **Bulk INSERT** أسرع بكتير من إدخال صف صف
5. **COPY** للبيانات الكبيرة جداً
6. استخدم **Parameterized Queries** للأمان

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [SELECT - قراءة البيانات](./10-select.md)**

</div>

---

<div align="center">

[⬅️ السابق: تعديل الجداول](./08-alter-table.md) | [🏠 العودة للـ Module](../README.md)

</div>
