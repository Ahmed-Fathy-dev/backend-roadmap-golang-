# القيود (Constraints) 🔒

<div dir="rtl">

## مقدمة

الـ Constraints هي قواعد بتفرضها على البيانات عشان تضمن سلامتها وصحتها. من أهم المفاهيم في تصميم قواعد البيانات لأنها بتمنع إدخال بيانات خاطئة.

**المدة المتوقعة:** 30-35 دقيقة

</div>

---

## 📊 أنواع الـ Constraints

```
┌─────────────────────────────────────────────────────────────────┐
│                      PostgreSQL Constraints                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PRIMARY KEY   ─── معرّف فريد للصف                              │
│                                                                  │
│  FOREIGN KEY   ─── ربط بجدول آخر                                │
│                                                                  │
│  UNIQUE        ─── قيمة فريدة (لا تتكرر)                        │
│                                                                  │
│  NOT NULL      ─── لا يقبل NULL                                 │
│                                                                  │
│  CHECK         ─── شرط مخصص                                     │
│                                                                  │
│  DEFAULT       ─── قيمة افتراضية                                │
│                                                                  │
│  EXCLUDE       ─── منع التداخل (advanced)                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 PRIMARY KEY

<div dir="rtl">

### الخصائص

- معرّف فريد لكل صف
- **UNIQUE + NOT NULL** تلقائياً
- جدول واحد = PRIMARY KEY واحد فقط
- يمكن أن يتكون من عمود أو أكثر (Composite)

</div>

```sql
-- Primary Key على عمود واحد
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);

-- أو بصيغة منفصلة
CREATE TABLE users (
    id SERIAL,
    username VARCHAR(50),
    PRIMARY KEY (id)
);

-- أو مع اسم
CREATE TABLE users (
    id SERIAL,
    username VARCHAR(50),
    CONSTRAINT users_pkey PRIMARY KEY (id)
);
```

<div dir="rtl">

### Composite Primary Key

</div>

```sql
-- Primary Key من عمودين
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id)  -- Composite PK
);

-- مثال: جدول Many-to-Many
CREATE TABLE user_roles (
    user_id INT REFERENCES users(id),
    role_id INT REFERENCES roles(id),
    assigned_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (user_id, role_id)
);
```

<div dir="rtl">

### اختيار Primary Key

</div>

```sql
-- ✅ Surrogate Key (الأفضل غالباً)
CREATE TABLE products (
    id SERIAL PRIMARY KEY,  -- Surrogate
    sku VARCHAR(50) UNIQUE,  -- Natural key كـ UNIQUE
    name VARCHAR(200)
);

-- ⚠️ Natural Key (في حالات معينة)
CREATE TABLE countries (
    code CHAR(2) PRIMARY KEY,  -- ISO country code
    name VARCHAR(100)
);

-- ❌ لا تستخدم بيانات قابلة للتغيير كـ PK
CREATE TABLE users (
    email VARCHAR(255) PRIMARY KEY,  -- ممكن يتغير!
    name VARCHAR(100)
);
```

---

## 🔗 FOREIGN KEY

<div dir="rtl">

### الخصائص

- يربط جدول بجدول آخر
- يضمن Referential Integrity
- القيمة لازم تكون موجودة في الجدول المرجعي (أو NULL)

</div>

```sql
-- إنشاء الجداول
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    category_id INT REFERENCES categories(id)
);

-- أو بصيغة كاملة
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    category_id INT,
    CONSTRAINT fk_product_category
        FOREIGN KEY (category_id)
        REFERENCES categories(id)
);
```

<div dir="rtl">

### ON DELETE Actions

</div>

```sql
-- ON DELETE CASCADE: حذف الأبناء مع الأب
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE
);
-- لو حذفت user، كل orders بتاعته هتتحذف

-- ON DELETE SET NULL: تحويل لـ NULL
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    category_id INT REFERENCES categories(id) ON DELETE SET NULL
);
-- لو حذفت category، products هتبقى category_id = NULL

-- ON DELETE RESTRICT: منع الحذف (Default)
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE RESTRICT
);
-- مش هتقدر تحذف user لو عنده orders

-- ON DELETE SET DEFAULT: تحويل لـ default value
CREATE TABLE products (
    category_id INT DEFAULT 1 REFERENCES categories(id) ON DELETE SET DEFAULT
);

-- ON DELETE NO ACTION: زي RESTRICT بس يتأجل للـ transaction end
```

<div dir="rtl">

### ON UPDATE Actions

</div>

```sql
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    product_id INT REFERENCES products(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE  -- لو الـ product.id اتغير، هنا كمان يتغير
);
```

<div dir="rtl">

### مقارنة ON DELETE Actions

</div>

```
┌────────────────────────────────────────────────────────────────────┐
│  Action          │  السلوك                                        │
├────────────────────────────────────────────────────────────────────┤
│  CASCADE         │  حذف/تحديث كل الـ children                     │
│  SET NULL        │  تحويل FK لـ NULL                              │
│  SET DEFAULT     │  تحويل FK للقيمة الافتراضية                    │
│  RESTRICT        │  منع الحذف/التحديث (فوراً)                     │
│  NO ACTION       │  منع الحذف/التحديث (آخر الـ transaction)      │
└────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 UNIQUE

<div dir="rtl">

### الخصائص

- القيمة لازم تكون فريدة
- يسمح بـ NULL (وممكن أكتر من NULL!)
- يمكن إنشاء أكتر من UNIQUE constraint

</div>

```sql
-- على عمود واحد
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(255) UNIQUE
);

-- UNIQUE مركب
CREATE TABLE memberships (
    id SERIAL PRIMARY KEY,
    user_id INT,
    organization_id INT,
    UNIQUE (user_id, organization_id)  -- كل user في organization مرة واحدة
);

-- مع اسم
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    CONSTRAINT unique_sku UNIQUE (sku)
);
```

<div dir="rtl">

### UNIQUE vs PRIMARY KEY

</div>

```
┌────────────────────────────────────────────────────────────────────┐
│                 │  PRIMARY KEY          │  UNIQUE                  │
├────────────────────────────────────────────────────────────────────┤
│  NULL           │  ❌ ممنوع            │  ✅ مسموح               │
│  العدد          │  واحد فقط            │  متعدد                  │
│  Index          │  Clustered (غالباً)  │  Non-clustered          │
│  الغرض          │  معرّف الصف          │  منع التكرار            │
└────────────────────────────────────────────────────────────────────┘
```

---

## ⛔ NOT NULL

<div dir="rtl">

### الخصائص

- العمود لازم يكون له قيمة
- لا يقبل NULL
- أبسط وأهم constraint

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,      -- مطلوب
    email VARCHAR(255) NOT NULL,        -- مطلوب
    phone VARCHAR(20),                  -- اختياري (يقبل NULL)
    bio TEXT                            -- اختياري
);

-- محاولة إدخال NULL
INSERT INTO users (username, email) VALUES ('ahmed', NULL);
-- ERROR: null value in column "email" violates not-null constraint
```

<div dir="rtl">

### متى تستخدم NOT NULL؟

</div>

```sql
-- ✅ استخدم NOT NULL لـ:
username VARCHAR(50) NOT NULL,    -- مطلوب للتعريف
email VARCHAR(255) NOT NULL,      -- مطلوب للتواصل
password_hash VARCHAR(255) NOT NULL,  -- مطلوب للأمان
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),  -- مطلوب للتتبع

-- ⚠️ لا تستخدم NOT NULL لـ:
phone VARCHAR(20),           -- اختياري
address TEXT,                -- اختياري
deleted_at TIMESTAMPTZ,      -- NULL = not deleted
```

---

## ✅ CHECK

<div dir="rtl">

### الخصائص

- شرط مخصص على القيم
- يمكن أن يشمل عمود أو أكثر
- مرن جداً

</div>

```sql
-- CHECK على عمود واحد
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price NUMERIC(10, 2) CHECK (price > 0),
    stock INT CHECK (stock >= 0),
    discount NUMERIC(5, 2) CHECK (discount >= 0 AND discount <= 100)
);

-- CHECK مع اسم
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    salary NUMERIC(12, 2),
    bonus NUMERIC(12, 2),
    CONSTRAINT salary_positive CHECK (salary > 0),
    CONSTRAINT bonus_limit CHECK (bonus <= salary * 0.5)  -- الـ bonus ≤ 50% من الراتب
);

-- CHECK على أعمدة متعددة
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    start_date TIMESTAMPTZ NOT NULL,
    end_date TIMESTAMPTZ NOT NULL,
    CONSTRAINT valid_dates CHECK (end_date > start_date)
);

-- CHECK مع IN
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    status VARCHAR(20) CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);

-- CHECK مع regex
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    phone VARCHAR(20) CHECK (phone ~ '^\+?[0-9]{10,15}$')
);
```

<div dir="rtl">

### أمثلة CHECK متقدمة

</div>

```sql
-- التحقق من طول النص
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) CHECK (LENGTH(title) >= 5),
    content TEXT CHECK (LENGTH(content) >= 100)
);

-- التحقق من نطاق التاريخ
CREATE TABLE reservations (
    id SERIAL PRIMARY KEY,
    check_in DATE NOT NULL,
    check_out DATE NOT NULL,
    CONSTRAINT valid_reservation CHECK (
        check_out > check_in AND
        check_in >= CURRENT_DATE
    )
);

-- التحقق من JSON structure
CREATE TABLE settings (
    id SERIAL PRIMARY KEY,
    config JSONB CHECK (
        config ? 'theme' AND
        config ? 'language'
    )
);
```

---

## 📋 DEFAULT

<div dir="rtl">

### الخصائص

- قيمة افتراضية لو مااتحددتش
- يمكن استخدام قيم ثابتة أو functions

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    role VARCHAR(20) DEFAULT 'user',
    is_active BOOLEAN DEFAULT TRUE,
    login_count INT DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    price NUMERIC(10, 2) DEFAULT 0.00,
    stock INT DEFAULT 0,
    public_id UUID DEFAULT gen_random_uuid(),
    rating NUMERIC(3, 2) DEFAULT 0.00
);

-- Default مع expression
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    slug VARCHAR(200) DEFAULT '',  -- هيتحسب في trigger أو application
    published_at TIMESTAMPTZ,
    is_draft BOOLEAN DEFAULT TRUE
);
```

---

## 🚫 EXCLUDE (Advanced)

<div dir="rtl">

### منع التداخل

EXCLUDE constraint بيمنع التداخل في ranges أو geometric types.

</div>

```sql
-- تثبيت extension
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- منع حجوزات متداخلة
CREATE TABLE room_bookings (
    id SERIAL PRIMARY KEY,
    room_id INT NOT NULL,
    during TSTZRANGE NOT NULL,
    EXCLUDE USING GIST (
        room_id WITH =,
        during WITH &&
    )
);

-- مثال الإدخال
INSERT INTO room_bookings (room_id, during)
VALUES (1, '[2024-12-21 10:00, 2024-12-21 12:00)');

-- هذا هيفشل (تداخل!)
INSERT INTO room_bookings (room_id, during)
VALUES (1, '[2024-12-21 11:00, 2024-12-21 13:00)');
-- ERROR: conflicting key value violates exclusion constraint
```

---

## 🔧 إضافة وتعديل Constraints

<div dir="rtl">

### إضافة Constraint لجدول موجود

</div>

```sql
-- إضافة NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;

-- إضافة UNIQUE
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);

-- إضافة CHECK
ALTER TABLE products ADD CONSTRAINT positive_price CHECK (price > 0);

-- إضافة FOREIGN KEY
ALTER TABLE orders ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users(id);

-- إضافة PRIMARY KEY
ALTER TABLE logs ADD PRIMARY KEY (id);
```

<div dir="rtl">

### حذف Constraint

</div>

```sql
-- حذف constraint باسمه
ALTER TABLE users DROP CONSTRAINT unique_email;

-- حذف NOT NULL
ALTER TABLE users ALTER COLUMN phone DROP NOT NULL;

-- حذف PRIMARY KEY
ALTER TABLE users DROP CONSTRAINT users_pkey;
```

<div dir="rtl">

### تعديل Constraint

</div>

```sql
-- مفيش ALTER CONSTRAINT مباشر
-- لازم تحذف وتضيف من جديد

ALTER TABLE products DROP CONSTRAINT positive_price;
ALTER TABLE products ADD CONSTRAINT positive_price CHECK (price >= 0);  -- غيرنا > لـ >=
```

---

## 📋 عرض الـ Constraints

```sql
-- كل constraints لجدول معين
SELECT
    conname AS constraint_name,
    contype AS type,
    pg_get_constraintdef(oid) AS definition
FROM pg_constraint
WHERE conrelid = 'users'::regclass;

-- أو باستخدام information_schema
SELECT
    constraint_name,
    constraint_type
FROM information_schema.table_constraints
WHERE table_name = 'users';

-- تفاصيل Foreign Keys
SELECT
    tc.constraint_name,
    tc.table_name,
    kcu.column_name,
    ccu.table_name AS foreign_table,
    ccu.column_name AS foreign_column
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
    ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu
    ON tc.constraint_name = ccu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. سمّي الـ Constraints

</div>

```sql
-- ✅ أفضل - أسماء واضحة
CONSTRAINT users_email_unique UNIQUE (email),
CONSTRAINT orders_user_fk FOREIGN KEY (user_id) REFERENCES users(id),
CONSTRAINT products_price_positive CHECK (price > 0)

-- ❌ أسماء تلقائية صعبة القراءة
-- users_email_key, orders_user_id_fkey, products_price_check
```

<div dir="rtl">

### 2. استخدم NOT NULL بحكمة

</div>

```sql
-- ✅ NOT NULL للحقول المطلوبة
username VARCHAR(50) NOT NULL,
email VARCHAR(255) NOT NULL,

-- ✅ اسمح بـ NULL للحقول الاختيارية
phone VARCHAR(20),  -- بدون NOT NULL
middle_name VARCHAR(50),
```

<div dir="rtl">

### 3. ON DELETE المناسب

</div>

```sql
-- CASCADE: للعلاقات التبعية (تفاصيل الطلب)
order_items → orders: ON DELETE CASCADE

-- RESTRICT: للعلاقات المستقلة (منع حذف user له orders)
orders → users: ON DELETE RESTRICT

-- SET NULL: للعلاقات الاختيارية (category محذوفة)
products → categories: ON DELETE SET NULL
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **PRIMARY KEY** = UNIQUE + NOT NULL
2. **FOREIGN KEY** يضمن Referential Integrity
3. **CHECK** للـ business rules
4. **سمّي الـ constraints** بأسماء واضحة
5. **ON DELETE CASCADE** للتفاصيل التابعة
6. **NOT NULL** للحقول المطلوبة فقط

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [تعديل الجداول - ALTER TABLE](./08-alter-table.md)**

</div>

---

<div align="center">

[⬅️ السابق: إنشاء الجداول](./06-create-table.md) | [🏠 العودة للـ Module](../README.md)

</div>
