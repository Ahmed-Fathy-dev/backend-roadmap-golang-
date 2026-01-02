# Lesson 2: Relational Database Model 📊

<div dir="rtl">

## المقدمة

الـ **Relational Model** هو الأساس اللي بُني عليه كل SQL databases. فهمه صح هيخليك تصمم databases أفضل بكتير!

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 Core Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Relational Model Terminology                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Academic Term          Common Term         المصطلح العربي          │
│  ─────────────          ───────────         ────────────────         │
│  Relation        →      Table               جدول                    │
│  Tuple           →      Row/Record          صف/سجل                  │
│  Attribute       →      Column/Field        عمود/حقل                │
│  Domain          →      Data Type           نوع البيانات            │
│  Schema          →      Table Structure     هيكل الجدول             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Tables (Relations)

### Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Table: users                                │
│                    (Relation = جدول)                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Column Names (Attributes)                                           │
│  ↓         ↓           ↓            ↓           ↓                   │
├────────┬────────────┬─────────────────────┬──────────┬──────────────┤
│   id   │    name    │        email        │   age    │  created_at  │
├────────┼────────────┼─────────────────────┼──────────┼──────────────┤
│   1    │   Ahmed    │  ahmed@test.com     │    25    │  2024-01-15  │ ← Row (Tuple)
│   2    │   Sara     │  sara@test.com      │    30    │  2024-01-16  │ ← Row (Tuple)
│   3    │   Mohamed  │  mohamed@test.com   │    28    │  2024-01-17  │ ← Row (Tuple)
└────────┴────────────┴─────────────────────┴──────────┴──────────────┘
            ↑
         Each column has a specific data type (domain)
```

### Table Properties

```
خصائص الـ Table في Relational Model:

1. Unique Name
   └─ كل table له اسم فريد في الـ database

2. Atomic Values
   └─ كل cell فيها قيمة واحدة (لا lists, لا objects)

3. Unique Rows
   └─ لا يوجد صفين متطابقين تماماً

4. No Row Order
   └─ الصفوف ليس لها ترتيب افتراضي
   └─ استخدم ORDER BY لو محتاج ترتيب

5. Unique Column Names
   └─ أسماء الأعمدة فريدة داخل نفس الـ table
```

---

## 2️⃣ Columns (Attributes)

### Data Types (Domains)

```sql
-- PostgreSQL Data Types

-- Numeric Types
id          SERIAL              -- Auto-incrementing integer
price       DECIMAL(10, 2)      -- Exact numeric (10 digits, 2 after decimal)
quantity    INTEGER             -- Whole numbers
rating      REAL                -- Floating point

-- String Types
name        VARCHAR(100)        -- Variable length (max 100)
code        CHAR(5)             -- Fixed length (exactly 5)
description TEXT                -- Unlimited length

-- Date/Time Types
created_at  TIMESTAMP           -- Date and time
birth_date  DATE                -- Date only
check_in    TIME                -- Time only
updated_at  TIMESTAMPTZ         -- Timestamp with timezone

-- Boolean
is_active   BOOLEAN             -- true/false

-- Other Common Types
uuid        UUID                -- Universally unique identifier
data        JSONB               -- JSON data (binary)
tags        TEXT[]              -- Array of text
```

### Column Constraints

```sql
CREATE TABLE users (
    -- NOT NULL: لا يقبل قيم فارغة
    id          SERIAL PRIMARY KEY,
    email       VARCHAR(255) NOT NULL UNIQUE,

    -- DEFAULT: قيمة افتراضية
    is_active   BOOLEAN DEFAULT true,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- CHECK: تحقق من القيمة
    age         INTEGER CHECK (age >= 0 AND age <= 150),

    -- REFERENCES: Foreign Key
    department_id INTEGER REFERENCES departments(id)
);
```

---

## 3️⃣ Keys

### Primary Key (PK)

<div dir="rtl">

**Primary Key** = المعرّف الفريد لكل صف

</div>

```sql
-- Option 1: Auto-increment (الأكثر شيوعاً)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Option 2: UUID
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100)
);

-- Option 3: Composite Primary Key
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)  -- مفتاحين معاً
);
```

### Primary Key Rules

```
✅ قواعد الـ Primary Key:

1. Unique
   └─ لا يتكرر أبداً

2. Not Null
   └─ لا يكون فارغاً أبداً

3. Immutable
   └─ لا يتغير (best practice)

4. Simple
   └─ يفضل يكون integer أو UUID

❌ Don't use as PK:
   └─ Email (قد يتغير)
   └─ Phone (قد يتغير)
   └─ SSN (privacy concerns)
   └─ Composite keys (when possible to avoid)
```

### Foreign Key (FK)

<div dir="rtl">

**Foreign Key** = مرجع لـ Primary Key في table آخر

</div>

```sql
-- Parent table
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- Child table with Foreign Key
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES departments(id)
    --             ↑ Foreign Key points to departments.id
);

-- Explicit FK with name and options
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER,

    CONSTRAINT fk_department
        FOREIGN KEY (department_id)
        REFERENCES departments(id)
        ON DELETE SET NULL        -- عند حذف department
        ON UPDATE CASCADE         -- عند تحديث department.id
);
```

### Foreign Key Actions

```
ON DELETE / ON UPDATE actions:

CASCADE     → التغيير ينتشر للأبناء
             (حذف department = حذف كل employees فيها)

SET NULL    → القيمة تصبح NULL
             (حذف department = employees.department_id = NULL)

SET DEFAULT → القيمة تصبح default
             (يحتاج DEFAULT معرّف)

RESTRICT    → يمنع العملية لو في أبناء (default)
             (لا يمكن حذف department لها employees)

NO ACTION   → مثل RESTRICT (فرق تقني بسيط)
```

```sql
-- Real example
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    total DECIMAL(10, 2),

    -- لو حذفنا user، نحذف orders تبعه
    CONSTRAINT fk_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER NOT NULL,
    user_id INTEGER,
    content TEXT,

    -- لو حذفنا post، نحذف comments
    CONSTRAINT fk_post
        FOREIGN KEY (post_id)
        REFERENCES posts(id)
        ON DELETE CASCADE,

    -- لو حذفنا user، نخلي user_id = NULL
    CONSTRAINT fk_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE SET NULL
);
```

---

## 4️⃣ Schema Design Example

### E-commerce Schema

```
┌─────────────────────────────────────────────────────────────────────┐
│                    E-commerce Database Schema                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐         ┌──────────────┐                          │
│  │   users      │         │  categories  │                          │
│  ├──────────────┤         ├──────────────┤                          │
│  │ PK id        │         │ PK id        │                          │
│  │    email     │         │    name      │                          │
│  │    name      │         │    slug      │                          │
│  │    password  │         └──────┬───────┘                          │
│  │    created_at│                │                                  │
│  └──────┬───────┘                │                                  │
│         │                        │                                  │
│         │ 1:N                    │ 1:N                              │
│         │                        │                                  │
│         ▼                        ▼                                  │
│  ┌──────────────┐         ┌──────────────┐                          │
│  │   orders     │         │   products   │                          │
│  ├──────────────┤         ├──────────────┤                          │
│  │ PK id        │         │ PK id        │                          │
│  │ FK user_id   │─────┐   │    name      │                          │
│  │    total     │     │   │    price     │                          │
│  │    status    │     │   │    stock     │                          │
│  │    created_at│     │   │ FK category_id                          │
│  └──────┬───────┘     │   └──────┬───────┘                          │
│         │             │          │                                  │
│         │ 1:N         │          │                                  │
│         │             │          │                                  │
│         ▼             │          │                                  │
│  ┌──────────────┐     │          │                                  │
│  │ order_items  │     │          │                                  │
│  ├──────────────┤     │          │                                  │
│  │ PK id        │     └──────────┼──── (orders.user_id → users.id) │
│  │ FK order_id  │────────────────┼──── (order_items.order_id →     │
│  │ FK product_id│────────────────┘     orders.id)                  │
│  │    quantity  │                                                   │
│  │    price     │                                                   │
│  └──────────────┘                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### SQL Implementation

```sql
-- Categories
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status VARCHAR(20) NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    total DECIMAL(10, 2) NOT NULL DEFAULT 0,
    shipping_address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Order Items (Junction table for Order-Product relationship)
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price DECIMAL(10, 2) NOT NULL,  -- Price at time of order
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- Prevent duplicate product in same order
    UNIQUE (order_id, product_id)
);

-- Indexes for performance
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

---

## 5️⃣ Integrity Constraints

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Types of Integrity                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Entity Integrity                                                 │
│     └─ Primary Key must be unique and not null                      │
│     └─ Each row must be uniquely identifiable                       │
│                                                                      │
│  2. Referential Integrity                                            │
│     └─ Foreign Key must reference valid PK or be NULL               │
│     └─ No orphan records                                            │
│                                                                      │
│  3. Domain Integrity                                                 │
│     └─ Values must be within defined domain (data type)             │
│     └─ CHECK constraints                                            │
│                                                                      │
│  4. User-Defined Integrity                                           │
│     └─ Business rules (e.g., order total = sum of items)            │
│     └─ Custom constraints and triggers                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```sql
-- Examples of constraints

-- Entity Integrity
id SERIAL PRIMARY KEY

-- Referential Integrity
user_id INTEGER REFERENCES users(id) ON DELETE CASCADE

-- Domain Integrity
age INTEGER CHECK (age >= 0 AND age <= 150)
status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'pending'))
email VARCHAR(255) NOT NULL UNIQUE

-- User-Defined (Trigger)
CREATE OR REPLACE FUNCTION update_order_total()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE orders
    SET total = (
        SELECT COALESCE(SUM(quantity * price), 0)
        FROM order_items
        WHERE order_id = NEW.order_id
    )
    WHERE id = NEW.order_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER order_items_changed
    AFTER INSERT OR UPDATE OR DELETE ON order_items
    FOR EACH ROW
    EXECUTE FUNCTION update_order_total();
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Relational Model Best Practices                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Naming:                                                             │
│  ─────────                                                           │
│  • Tables: plural, lowercase, snake_case (users, order_items)       │
│  • Columns: singular, lowercase, snake_case (user_id, created_at)   │
│  • Primary Key: id أو table_id                                      │
│  • Foreign Key: referenced_table_id (user_id, category_id)          │
│                                                                      │
│  Design:                                                             │
│  ───────                                                             │
│  • Always have a Primary Key                                         │
│  • Use appropriate data types (don't VARCHAR everything)            │
│  • Add indexes on FK columns                                         │
│  • Use constraints to enforce rules                                  │
│  • Consider soft delete (is_deleted) vs hard delete                 │
│                                                                      │
│  Avoid:                                                              │
│  ──────                                                              │
│  • Storing multiple values in one column (use separate table)       │
│  • Using reserved words as names (order, user, table)               │
│  • Over-using TEXT (use VARCHAR with limit when possible)           │
│  • Forgetting timestamps (created_at, updated_at)                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Table** = Collection of related data (rows & columns)
- ✅ **Primary Key** = Unique identifier for each row
- ✅ **Foreign Key** = Reference to another table's PK
- ✅ **Constraints** = Rules to maintain data integrity
- ✅ **Relationships** = Links between tables via FK
- ✅ **Schema** = Overall structure of your database

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد ما فهمت الـ Relational Model، خلينا نتعمق في أنواع الـ Relationships:

**➡️ [Lesson 3: Relationships](./03-relationships.md)**

</div>

---

<div align="center">

[⬅️ Previous: SQL vs NoSQL](./01-sql-vs-nosql.md) | [📚 Module Home](../README.md) | [➡️ Next: Relationships](./03-relationships.md)

</div>
