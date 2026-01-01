# إنشاء أول Database 🎯

<div dir="rtl">

## مقدمة

دلوقتي هنطبق كل اللي اتعلمناه! هننشئ أول Database حقيقية، ونعمل User، ونظبط الصلاحيات. الدرس ده عملي 100%.

**المدة المتوقعة:** 15-20 دقيقة

</div>

---

## 🎯 السيناريو

<div dir="rtl">

هننشئ بيئة لمشروع **E-commerce** بسيط:

```
┌─────────────────────────────────────────────────────────────┐
│                     Our Project                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Database: ecommerce_db                                      │
│                                                              │
│  Users:                                                      │
│  ├─ admin_user     → صلاحيات كاملة                          │
│  ├─ app_user       → للتطبيق (read/write)                   │
│  └─ readonly_user  → للتقارير (read only)                   │
│                                                              │
│  Tables:                                                     │
│  ├─ users          → بيانات المستخدمين                      │
│  ├─ products       → المنتجات                               │
│  └─ orders         → الطلبات                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

</div>

---

## 📦 الخطوة 1: إنشاء الـ Database

<div dir="rtl">

### الاتصال بـ PostgreSQL

</div>

```bash
# Windows
psql -U postgres

# Linux
sudo -u postgres psql

# macOS
psql postgres
```

<div dir="rtl">

### إنشاء الـ Database

</div>

```sql
-- إنشاء Database
CREATE DATABASE ecommerce_db;

-- التحقق
\l

-- النتيجة
                              List of databases
     Name      |  Owner   | Encoding |   Collate   |    Ctype
---------------+----------+----------+-------------+-------------
 ecommerce_db  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 postgres      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
```

<div dir="rtl">

### إنشاء Database مع خيارات إضافية

</div>

```sql
-- Database مع إعدادات مخصصة
CREATE DATABASE ecommerce_db
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0
    CONNECTION LIMIT = 100;

-- إضافة تعليق للـ Database
COMMENT ON DATABASE ecommerce_db IS 'E-commerce application database';
```

---

## 👥 الخطوة 2: إنشاء الـ Users

<div dir="rtl">

### 1. Admin User (صلاحيات كاملة)

</div>

```sql
-- إنشاء Admin User
CREATE USER admin_user WITH
    PASSWORD 'Admin@2024Secure!'
    CREATEDB
    CREATEROLE
    LOGIN;

-- منحه صلاحيات على الـ Database
GRANT ALL PRIVILEGES ON DATABASE ecommerce_db TO admin_user;
```

<div dir="rtl">

### 2. Application User (للتطبيق)

</div>

```sql
-- إنشاء App User
CREATE USER app_user WITH
    PASSWORD 'AppUser@2024!'
    LOGIN;

-- صلاحيات على الـ Database
GRANT CONNECT ON DATABASE ecommerce_db TO app_user;
```

<div dir="rtl">

### 3. Readonly User (للتقارير)

</div>

```sql
-- إنشاء Readonly User
CREATE USER readonly_user WITH
    PASSWORD 'ReadOnly@2024!'
    LOGIN;

-- صلاحيات على الـ Database
GRANT CONNECT ON DATABASE ecommerce_db TO readonly_user;
```

<div dir="rtl">

### التحقق من الـ Users

</div>

```sql
\du

-- النتيجة
                             List of roles
   Role name   |                   Attributes
---------------+------------------------------------------------
 admin_user    | Create role, Create DB
 app_user      |
 postgres      | Superuser, Create role, Create DB, Replication
 readonly_user |
```

---

## 🔗 الخطوة 3: الاتصال بالـ Database الجديدة

<div dir="rtl">

### التنقل للـ Database

</div>

```sql
\c ecommerce_db
-- أو
\connect ecommerce_db

-- النتيجة
You are now connected to database "ecommerce_db" as user "postgres".
ecommerce_db=#
```

<div dir="rtl">

### أو من command line

</div>

```bash
psql -U postgres -d ecommerce_db
```

---

## 📊 الخطوة 4: إنشاء الـ Tables

<div dir="rtl">

### جدول Users

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone VARCHAR(20),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- إضافة تعليق
COMMENT ON TABLE users IS 'جدول المستخدمين';
COMMENT ON COLUMN users.password_hash IS 'كلمة السر المشفرة - لا تخزن plain text!';
```

<div dir="rtl">

### جدول Products

</div>

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL CHECK (price > 0),
    stock INT NOT NULL DEFAULT 0 CHECK (stock >= 0),
    category VARCHAR(100),
    image_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

COMMENT ON TABLE products IS 'جدول المنتجات';
```

<div dir="rtl">

### جدول Orders

</div>

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total_amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending'
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    shipping_address TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

COMMENT ON TABLE orders IS 'جدول الطلبات';
```

<div dir="rtl">

### جدول Order Items

</div>

```sql
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    total_price DECIMAL(10, 2) GENERATED ALWAYS AS (quantity * unit_price) STORED
);

COMMENT ON TABLE order_items IS 'تفاصيل الطلبات';
```

<div dir="rtl">

### التحقق من الـ Tables

</div>

```sql
\dt

-- النتيجة
            List of relations
 Schema |    Name     | Type  |  Owner
--------+-------------+-------+----------
 public | order_items | table | postgres
 public | orders      | table | postgres
 public | products    | table | postgres
 public | users       | table | postgres
```

```sql
-- وصف table معين
\d+ users
```

---

## 🔐 الخطوة 5: إعداد الصلاحيات

<div dir="rtl">

### صلاحيات App User

</div>

```sql
-- صلاحيات على Schema
GRANT USAGE ON SCHEMA public TO app_user;

-- صلاحيات على كل الـ Tables الحالية
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;

-- صلاحيات على الـ Sequences (للـ SERIAL columns)
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- صلاحيات تلقائية للـ Tables الجديدة
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT USAGE, SELECT ON SEQUENCES TO app_user;
```

<div dir="rtl">

### صلاحيات Readonly User

</div>

```sql
-- صلاحيات على Schema
GRANT USAGE ON SCHEMA public TO readonly_user;

-- صلاحيات قراءة فقط
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- صلاحيات تلقائية للـ Tables الجديدة
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO readonly_user;
```

---

## 📝 الخطوة 6: إضافة بيانات تجريبية

<div dir="rtl">

### إضافة Users

</div>

```sql
INSERT INTO users (username, email, password_hash, first_name, last_name, phone)
VALUES
    ('ahmed_ali', 'ahmed@example.com', '$2a$10$abc...', 'Ahmed', 'Ali', '01012345678'),
    ('sara_mohamed', 'sara@example.com', '$2a$10$def...', 'Sara', 'Mohamed', '01098765432'),
    ('omar_hassan', 'omar@example.com', '$2a$10$ghi...', 'Omar', 'Hassan', '01122334455');
```

<div dir="rtl">

### إضافة Products

</div>

```sql
INSERT INTO products (name, description, price, stock, category)
VALUES
    ('iPhone 15 Pro', 'Latest Apple smartphone', 999.99, 50, 'Electronics'),
    ('MacBook Pro 14"', 'Apple M3 Pro laptop', 1999.99, 25, 'Electronics'),
    ('AirPods Pro 2', 'Wireless earbuds with ANC', 249.99, 100, 'Electronics'),
    ('Nike Air Max', 'Running shoes', 149.99, 75, 'Shoes'),
    ('Levi\'s 501 Jeans', 'Classic blue jeans', 79.99, 200, 'Clothing');
```

<div dir="rtl">

### إضافة Order

</div>

```sql
-- إضافة طلب
INSERT INTO orders (user_id, total_amount, shipping_address)
VALUES
    (1, 1249.98, '123 Main Street, Cairo, Egypt');

-- إضافة تفاصيل الطلب
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES
    (1, 1, 1, 999.99),   -- iPhone
    (1, 3, 1, 249.99);   -- AirPods
```

---

## ✅ الخطوة 7: اختبار كل شيء

<div dir="rtl">

### اختبار الـ Queries

</div>

```sql
-- عرض كل المستخدمين
SELECT id, username, email, first_name, last_name
FROM users;

-- عرض المنتجات مع الـ Stock
SELECT name, price, stock, category
FROM products
WHERE is_active = TRUE
ORDER BY price DESC;

-- عرض الطلبات مع تفاصيلها
SELECT
    o.id AS order_id,
    u.username,
    p.name AS product,
    oi.quantity,
    oi.unit_price,
    oi.total_price
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;

-- إحصائيات
SELECT
    COUNT(*) AS total_products,
    SUM(stock) AS total_stock,
    AVG(price)::DECIMAL(10,2) AS avg_price
FROM products;
```

<div dir="rtl">

### اختبار الـ Users

</div>

```bash
# اختبار app_user
psql -U app_user -d ecommerce_db -h localhost
# Password: AppUser@2024!
```

```sql
-- هيشتغل
SELECT * FROM users;
INSERT INTO users (username, email, password_hash) VALUES ('test', 'test@test.com', 'hash');
DELETE FROM users WHERE username = 'test';

-- التحقق
\q
```

```bash
# اختبار readonly_user
psql -U readonly_user -d ecommerce_db -h localhost
# Password: ReadOnly@2024!
```

```sql
-- هيشتغل
SELECT * FROM products;

-- مش هيشتغل (Permission denied)
INSERT INTO products (name, price, stock) VALUES ('Test', 100, 10);
-- ERROR: permission denied for table products
```

---

## 🔄 الخطوة 8: Backup الـ Database

<div dir="rtl">

### إنشاء Backup

</div>

```bash
# Backup كامل
pg_dump -U postgres ecommerce_db > ecommerce_backup.sql

# Backup مضغوط
pg_dump -U postgres -Fc ecommerce_db > ecommerce_backup.dump

# Backup للـ Schema فقط (بدون بيانات)
pg_dump -U postgres --schema-only ecommerce_db > ecommerce_schema.sql

# Backup للبيانات فقط
pg_dump -U postgres --data-only ecommerce_db > ecommerce_data.sql
```

<div dir="rtl">

### استعادة Backup

</div>

```bash
# استعادة من SQL file
psql -U postgres -d ecommerce_db < ecommerce_backup.sql

# استعادة من dump file
pg_restore -U postgres -d ecommerce_db ecommerce_backup.dump
```

---

## 📋 ملخص ما أنشأناه

<div dir="rtl">

### الهيكل النهائي

</div>

```
ecommerce_db/
├── Users:
│   ├── admin_user    (CREATEDB, CREATEROLE)
│   ├── app_user      (SELECT, INSERT, UPDATE, DELETE)
│   └── readonly_user (SELECT only)
│
├── Tables:
│   ├── users         (id, username, email, password_hash, ...)
│   ├── products      (id, name, price, stock, ...)
│   ├── orders        (id, user_id, total_amount, status, ...)
│   └── order_items   (id, order_id, product_id, quantity, ...)
│
└── Relations:
    ├── orders.user_id → users.id
    ├── order_items.order_id → orders.id
    └── order_items.product_id → products.id
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. الأسماء

</div>

```sql
-- ✅ كويس
CREATE TABLE users (...);
CREATE TABLE order_items (...);

-- ❌ وحش
CREATE TABLE Users (...);
CREATE TABLE orderItems (...);
CREATE TABLE tbl_users (...);
```

<div dir="rtl">

### 2. الصلاحيات

</div>

```
✅ Principle of Least Privilege:
   - كل user ياخد أقل صلاحيات يحتاجها بس
   - التطبيق مش محتاج DROP أو CREATE
   - التقارير محتاجة SELECT بس

❌ لا تعمل ده:
   - GRANT ALL PRIVILEGES للتطبيق
   - استخدام postgres user في التطبيق
```

<div dir="rtl">

### 3. كلمات السر

</div>

```
✅ كويس:
   - كلمات سر قوية ومختلفة لكل user
   - تخزين في Environment Variables
   - تغيير دوري

❌ وحش:
   - 123456, password, postgres
   - نفس الـ password لكل الـ users
   - كتابتها في الكود
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ أنشأت الـ Database
- [ ] ✅ أنشأت 3 Users بصلاحيات مختلفة
- [ ] ✅ أنشأت 4 Tables
- [ ] ✅ أضفت بيانات تجريبية
- [ ] ✅ اختبرت الصلاحيات
- [ ] ✅ عملت Backup

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Connection Strings](./08-connection-strings.md)**

</div>

---

<div align="center">

[⬅️ السابق: أوامر psql](./06-psql-commands.md) | [🏠 العودة للـ Module](../README.md)

</div>
