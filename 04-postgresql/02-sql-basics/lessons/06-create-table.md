# إنشاء الجداول - CREATE TABLE 🏗️

<div dir="rtl">

## مقدمة

الجداول (Tables) هي الوحدة الأساسية لتخزين البيانات في PostgreSQL. في الدرس ده هنتعلم كيفية تصميم وإنشاء جداول احترافية مع كل التفاصيل.

**المدة المتوقعة:** 25-30 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    column3 datatype constraints,
    ...
    table_constraints
);
```

<div dir="rtl">

### مثال بسيط

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🏛️ تشريح CREATE TABLE

<div dir="rtl">

### المكونات الأساسية

</div>

```sql
CREATE TABLE products (
    -- Column definitions
    id SERIAL PRIMARY KEY,              -- اسم + نوع + constraint
    name VARCHAR(200) NOT NULL,         -- اسم + نوع + constraint
    price NUMERIC(10,2) DEFAULT 0.00,   -- اسم + نوع + default
    description TEXT,                    -- اسم + نوع (nullable)

    -- Table-level constraints
    CONSTRAINT price_positive CHECK (price >= 0)
);
```

```
┌─────────────────────────────────────────────────────────────────┐
│  CREATE TABLE products (                                         │
│      id SERIAL PRIMARY KEY,                                      │
│      ↑    ↑        ↑                                             │
│      │    │        └── Constraint                                │
│      │    └─────────── Data Type                                 │
│      └──────────────── Column Name                               │
│                                                                  │
│      name VARCHAR(200) NOT NULL,                                 │
│           ↑            ↑                                         │
│           │            └── Column Constraint                     │
│           └─────────────── Data Type with length                 │
│                                                                  │
│      CONSTRAINT price_positive CHECK (price >= 0)                │
│      ↑          ↑              ↑                                 │
│      │          │              └── Constraint Expression         │
│      │          └───────────────── Constraint Name               │
│      └──────────────────────────── Table-level Constraint        │
│  );                                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 أمثلة عملية متكاملة

<div dir="rtl">

### 1. جدول المستخدمين (Users)

</div>

```sql
CREATE TABLE users (
    -- Primary Key
    id SERIAL PRIMARY KEY,

    -- Authentication
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,

    -- Profile Info
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone VARCHAR(20),
    avatar_url VARCHAR(500),

    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    role VARCHAR(20) DEFAULT 'user',

    -- Timestamps
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    last_login_at TIMESTAMPTZ,

    -- Constraints
    CONSTRAINT valid_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT valid_role CHECK (role IN ('admin', 'moderator', 'user', 'guest'))
);

-- إضافة comment للجدول
COMMENT ON TABLE users IS 'جدول المستخدمين الرئيسي';
COMMENT ON COLUMN users.password_hash IS 'كلمة السر المشفرة - لا تخزن plain text!';
```

<div dir="rtl">

### 2. جدول المنتجات (Products)

</div>

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    slug VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    parent_id INT REFERENCES categories(id) ON DELETE SET NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE products (
    -- IDs
    id SERIAL PRIMARY KEY,
    public_id UUID DEFAULT gen_random_uuid() UNIQUE,
    sku VARCHAR(50) UNIQUE,

    -- Basic Info
    name VARCHAR(200) NOT NULL,
    slug VARCHAR(200) NOT NULL UNIQUE,
    description TEXT,
    short_description VARCHAR(500),

    -- Pricing
    price NUMERIC(10, 2) NOT NULL,
    compare_at_price NUMERIC(10, 2),
    cost_price NUMERIC(10, 2),

    -- Inventory
    stock INT DEFAULT 0,
    low_stock_threshold INT DEFAULT 10,
    track_inventory BOOLEAN DEFAULT TRUE,

    -- Classification
    category_id INT REFERENCES categories(id) ON DELETE SET NULL,
    brand VARCHAR(100),
    tags TEXT[],

    -- Media
    images JSONB DEFAULT '[]',
    thumbnail_url VARCHAR(500),

    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    is_featured BOOLEAN DEFAULT FALSE,

    -- SEO
    meta_title VARCHAR(200),
    meta_description VARCHAR(500),

    -- Timestamps
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    published_at TIMESTAMPTZ,

    -- Constraints
    CONSTRAINT positive_price CHECK (price > 0),
    CONSTRAINT valid_compare_price CHECK (compare_at_price IS NULL OR compare_at_price >= price),
    CONSTRAINT non_negative_stock CHECK (stock >= 0)
);
```

<div dir="rtl">

### 3. جدول الطلبات (Orders)

</div>

```sql
CREATE TABLE orders (
    -- IDs
    id SERIAL PRIMARY KEY,
    order_number VARCHAR(20) NOT NULL UNIQUE,

    -- Customer
    user_id INT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,

    -- Amounts
    subtotal NUMERIC(12, 2) NOT NULL,
    discount_amount NUMERIC(12, 2) DEFAULT 0,
    tax_amount NUMERIC(12, 2) DEFAULT 0,
    shipping_amount NUMERIC(12, 2) DEFAULT 0,
    total_amount NUMERIC(12, 2) NOT NULL,

    -- Status
    status VARCHAR(20) DEFAULT 'pending',
    payment_status VARCHAR(20) DEFAULT 'unpaid',

    -- Addresses (JSONB for flexibility)
    shipping_address JSONB NOT NULL,
    billing_address JSONB,

    -- Payment
    payment_method VARCHAR(50),
    payment_reference VARCHAR(100),

    -- Shipping
    shipping_method VARCHAR(50),
    tracking_number VARCHAR(100),
    shipped_at TIMESTAMPTZ,
    delivered_at TIMESTAMPTZ,

    -- Notes
    customer_notes TEXT,
    internal_notes TEXT,

    -- Timestamps
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),

    -- Constraints
    CONSTRAINT valid_status CHECK (
        status IN ('pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded')
    ),
    CONSTRAINT valid_payment_status CHECK (
        payment_status IN ('unpaid', 'paid', 'refunded', 'partially_refunded')
    ),
    CONSTRAINT valid_total CHECK (total_amount >= 0)
);

-- Order Items
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(id) ON DELETE RESTRICT,

    -- Item Details
    product_name VARCHAR(200) NOT NULL,  -- Snapshot at order time
    quantity INT NOT NULL,
    unit_price NUMERIC(10, 2) NOT NULL,
    total_price NUMERIC(12, 2) GENERATED ALWAYS AS (quantity * unit_price) STORED,

    -- Constraints
    CONSTRAINT positive_quantity CHECK (quantity > 0),
    CONSTRAINT positive_unit_price CHECK (unit_price > 0)
);
```

---

## 🔧 IF NOT EXISTS

<div dir="rtl">

لتجنب الأخطاء لو الجدول موجود:

</div>

```sql
-- بدون IF NOT EXISTS
CREATE TABLE users (...);
-- لو موجود: ERROR: relation "users" already exists

-- مع IF NOT EXISTS
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
-- لو موجود: NOTICE: relation "users" already exists, skipping
```

---

## 🗂️ Schemas

<div dir="rtl">

### إنشاء جدول في Schema معين

</div>

```sql
-- إنشاء Schema
CREATE SCHEMA IF NOT EXISTS inventory;
CREATE SCHEMA IF NOT EXISTS auth;

-- إنشاء جدول في Schema
CREATE TABLE auth.users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);

CREATE TABLE inventory.products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200)
);

-- الوصول للجداول
SELECT * FROM auth.users;
SELECT * FROM inventory.products;

-- أو تغيير search_path
SET search_path TO inventory, public;
SELECT * FROM products;  -- هيدور في inventory أولاً
```

---

## 📋 Temporary Tables

<div dir="rtl">

### جداول مؤقتة

</div>

```sql
-- جدول مؤقت (يختفي بعد انتهاء الـ session)
CREATE TEMPORARY TABLE temp_results (
    id INT,
    value NUMERIC
);

-- أو
CREATE TEMP TABLE temp_data AS
SELECT id, name FROM users WHERE is_active = true;

-- استخدامات:
-- - تخزين نتائج مؤقتة
-- - تحسين الأداء في queries معقدة
-- - اختبار بدون تأثير على البيانات الحقيقية
```

---

## 🔄 CREATE TABLE AS

<div dir="rtl">

### إنشاء جدول من Query

</div>

```sql
-- إنشاء جدول من نتيجة query
CREATE TABLE active_users AS
SELECT id, username, email, created_at
FROM users
WHERE is_active = true;

-- مع تعديل الأعمدة
CREATE TABLE monthly_sales AS
SELECT
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS orders_count,
    SUM(total_amount) AS total_sales
FROM orders
WHERE created_at >= '2024-01-01'
GROUP BY DATE_TRUNC('month', created_at);

-- إنشاء هيكل فقط (بدون بيانات)
CREATE TABLE users_backup (LIKE users INCLUDING ALL);
-- أو
CREATE TABLE users_empty AS SELECT * FROM users WHERE false;
```

---

## 📝 Generated Columns

<div dir="rtl">

### أعمدة محسوبة تلقائياً

</div>

```sql
CREATE TABLE rectangles (
    id SERIAL PRIMARY KEY,
    width NUMERIC NOT NULL,
    height NUMERIC NOT NULL,
    -- Computed columns
    area NUMERIC GENERATED ALWAYS AS (width * height) STORED,
    perimeter NUMERIC GENERATED ALWAYS AS (2 * (width + height)) STORED
);

INSERT INTO rectangles (width, height) VALUES (10, 5);
SELECT * FROM rectangles;
-- id | width | height | area | perimeter
-- 1  | 10    | 5      | 50   | 30

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    full_name VARCHAR(101) GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED,
    birth_date DATE,
    age INT GENERATED ALWAYS AS (EXTRACT(YEAR FROM AGE(birth_date))) STORED
);
```

---

## 🔒 Partitioned Tables

<div dir="rtl">

### تقسيم الجداول الكبيرة

</div>

```sql
-- جدول الطلبات مقسم بالتاريخ
CREATE TABLE orders_partitioned (
    id SERIAL,
    order_number VARCHAR(20) NOT NULL,
    user_id INT NOT NULL,
    total_amount NUMERIC(12, 2),
    created_at TIMESTAMPTZ NOT NULL,
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (created_at);

-- إنشاء Partitions
CREATE TABLE orders_2024_q1 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

CREATE TABLE orders_2024_q3 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

CREATE TABLE orders_2024_q4 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-10-01') TO ('2025-01-01');

-- الإدخال يروح للـ partition المناسب تلقائياً
INSERT INTO orders_partitioned (order_number, user_id, total_amount, created_at)
VALUES ('ORD-001', 1, 100.00, '2024-05-15');
-- هيروح لـ orders_2024_q2
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. تسمية الجداول

</div>

```sql
-- ✅ كويس: lowercase, snake_case, plural
CREATE TABLE users (...);
CREATE TABLE order_items (...);
CREATE TABLE product_categories (...);

-- ❌ وحش
CREATE TABLE User (...);           -- PascalCase
CREATE TABLE orderItems (...);     -- camelCase
CREATE TABLE tbl_users (...);      -- prefixes
CREATE TABLE user (...);           -- singular (debatable)
```

<div dir="rtl">

### 2. تسمية الأعمدة

</div>

```sql
-- ✅ كويس
id, user_id, created_at, is_active, first_name

-- ❌ وحش
ID, userId, CreatedAt, isactive, FirstName
```

<div dir="rtl">

### 3. دايماً حدد Primary Key

</div>

```sql
-- ✅ كويس
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    ...
);

-- ❌ وحش (لا تنشئ جدول بدون PK)
CREATE TABLE users (
    username VARCHAR(50),
    ...
);
```

<div dir="rtl">

### 4. استخدم Constraints

</div>

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,           -- NOT NULL للـ required
    email VARCHAR(255) UNIQUE,            -- UNIQUE للـ unique
    price NUMERIC(10,2) CHECK (price > 0), -- CHECK للـ validation
    category_id INT REFERENCES categories(id) -- FK للـ relations
);
```

<div dir="rtl">

### 5. Timestamps

</div>

```sql
-- دايماً أضف timestamps
CREATE TABLE any_table (
    ...
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. نسيان Primary Key

</div>

```sql
-- ❌
CREATE TABLE logs (
    message TEXT,
    created_at TIMESTAMP
);

-- ✅
CREATE TABLE logs (
    id BIGSERIAL PRIMARY KEY,
    message TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

<div dir="rtl">

### 2. أسماء محجوزة

</div>

```sql
-- ❌ Error: "user" is reserved
CREATE TABLE user (...);
CREATE TABLE order (...);

-- ✅
CREATE TABLE users (...);
CREATE TABLE orders (...);
-- أو استخدم quotes (مش مفضل)
CREATE TABLE "user" (...);
```

<div dir="rtl">

### 3. VARCHAR بدون حد

</div>

```sql
-- ⚠️ مش أفضل ممارسة
name VARCHAR  -- unlimited

-- ✅ حدد حد معقول
name VARCHAR(200)
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. دايماً أضف **PRIMARY KEY**
2. استخدم **snake_case** للأسماء
3. أضف **timestamps** (created_at, updated_at)
4. استخدم **IF NOT EXISTS** لتجنب الأخطاء
5. استخدم **GENERATED** columns للحسابات التلقائية
6. **Partition** الجداول الكبيرة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [القيود (Constraints)](./07-constraints.md)**

</div>

---

<div align="center">

[⬅️ السابق: الأنواع الخاصة](./05-special-types.md) | [🏠 العودة للـ Module](../README.md)

</div>
