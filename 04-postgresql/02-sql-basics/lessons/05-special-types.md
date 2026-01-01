# الأنواع الخاصة في PostgreSQL 🎯

<div dir="rtl">

## مقدمة

PostgreSQL مش بس بيدعم الأنواع التقليدية، لكن كمان بيقدم أنواع خاصة قوية جداً زي JSON, UUID, Arrays, وغيرهم. الأنواع دي بتخلي PostgreSQL قادر يعمل حاجات كتير بدون ما تحتاج tools خارجية.

**المدة المتوقعة:** 30-35 دقيقة

</div>

---

## ✅ BOOLEAN

<div dir="rtl">

### الخصائص

- **القيم:** `TRUE`, `FALSE`, `NULL`
- **الحجم:** 1 byte
- **الاستخدام:** flags, states, yes/no fields

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    is_admin BOOLEAN DEFAULT FALSE,
    email_notifications BOOLEAN DEFAULT TRUE
);

-- إدخال البيانات
INSERT INTO users (username, is_active, is_verified) VALUES
    ('ahmed', TRUE, TRUE),
    ('sara', true, false),      -- lowercase يشتغل
    ('omar', 't', 'f'),         -- اختصار
    ('fatima', 'yes', 'no'),    -- yes/no
    ('ali', '1', '0');          -- 1/0

-- الاستعلام
SELECT * FROM users WHERE is_active = TRUE;
SELECT * FROM users WHERE is_active;           -- نفس الشيء
SELECT * FROM users WHERE NOT is_verified;     -- غير موثقين
SELECT * FROM users WHERE is_admin IS NULL;    -- غير محدد
```

<div dir="rtl">

### قيم مقبولة للـ TRUE و FALSE

</div>

```sql
-- TRUE يقبل:
TRUE, 't', 'true', 'y', 'yes', 'on', '1'

-- FALSE يقبل:
FALSE, 'f', 'false', 'n', 'no', 'off', '0'
```

---

## 🔑 UUID - Universal Unique Identifier

<div dir="rtl">

### الخصائص

- **الصيغة:** 32 رقم hex مفصولين بـ dashes
- **مثال:** `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11`
- **الحجم:** 16 bytes
- **الاستخدام:** IDs فريدة عالمياً، Public IDs

</div>

```sql
-- تفعيل extension (مرة واحدة)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- أو استخدم gen_random_uuid() (PostgreSQL 13+)

CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    internal_id SERIAL,  -- للاستخدام الداخلي
    name VARCHAR(200) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- الإدخال
INSERT INTO products (name) VALUES ('Laptop');
-- id هيتولد تلقائي: 'f47ac10b-58cc-4372-a567-0e02b2c3d479'

-- أو تحديد UUID معين
INSERT INTO products (id, name)
VALUES ('a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'Phone');

-- الاستعلام
SELECT * FROM products WHERE id = 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11';
```

<div dir="rtl">

### UUID vs SERIAL للـ IDs

</div>

```
┌─────────────────────────────────────────────────────────────────┐
│  SERIAL (Auto-increment)                                         │
├─────────────────────────────────────────────────────────────────┤
│  ✅ أصغر حجماً (4 bytes)                                        │
│  ✅ أسرع في الـ indexing                                        │
│  ✅ قابل للقراءة (1, 2, 3...)                                   │
│  ❌ يكشف عدد الـ records                                        │
│  ❌ مشاكل في الـ distributed systems                            │
│  ❌ ممكن يتخمن                                                  │
├─────────────────────────────────────────────────────────────────┤
│  UUID                                                            │
├─────────────────────────────────────────────────────────────────┤
│  ✅ فريد عالمياً                                                │
│  ✅ آمن للـ public APIs                                         │
│  ✅ يشتغل في distributed systems                                │
│  ❌ أكبر حجماً (16 bytes)                                       │
│  ❌ أبطأ شوية في الـ indexing                                   │
│  ❌ صعب القراءة                                                 │
└─────────────────────────────────────────────────────────────────┘

💡 نصيحة: استخدم الاتنين!
   - SERIAL للـ internal ID
   - UUID للـ public/external ID
```

---

## 📋 JSON و JSONB

<div dir="rtl">

### الفرق بين JSON و JSONB

</div>

```
┌─────────────────────────────────────────────────────────────────┐
│  JSON                                                            │
├─────────────────────────────────────────────────────────────────┤
│  - يخزن النص كما هو                                             │
│  - يحافظ على الترتيب والمسافات                                  │
│  - أبطأ في الاستعلام                                            │
│  - لا يدعم indexing                                             │
├─────────────────────────────────────────────────────────────────┤
│  JSONB (Binary JSON)  ✅ الأفضل                                  │
├─────────────────────────────────────────────────────────────────┤
│  - يخزن بصيغة binary محسّنة                                     │
│  - أسرع في الاستعلام                                            │
│  - يدعم indexing                                                │
│  - لا يحافظ على ترتيب المفاتيح                                  │
└─────────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### استخدام JSONB

</div>

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    attributes JSONB,           -- خصائص متغيرة
    metadata JSONB DEFAULT '{}' -- metadata
);

-- إدخال البيانات
INSERT INTO products (name, attributes) VALUES
    ('iPhone 15', '{
        "brand": "Apple",
        "storage": "256GB",
        "color": "Blue",
        "specs": {
            "screen": "6.1 inch",
            "camera": "48MP"
        },
        "tags": ["smartphone", "premium", "ios"]
    }'),
    ('Galaxy S24', '{
        "brand": "Samsung",
        "storage": "128GB",
        "color": "Black",
        "specs": {
            "screen": "6.2 inch",
            "camera": "50MP"
        },
        "tags": ["smartphone", "android"]
    }');
```

<div dir="rtl">

### استعلامات JSONB

</div>

```sql
-- الوصول لقيمة (-> يرجع JSON, ->> يرجع text)
SELECT name, attributes->>'brand' AS brand FROM products;
SELECT name, attributes->'specs'->>'screen' AS screen FROM products;

-- البحث في JSONB
SELECT * FROM products WHERE attributes->>'brand' = 'Apple';
SELECT * FROM products WHERE attributes->'specs'->>'camera' = '48MP';

-- البحث في array داخل JSONB
SELECT * FROM products WHERE attributes->'tags' ? 'premium';
-- المنتجات اللي فيها tag اسمه 'premium'

-- البحث بـ contains
SELECT * FROM products
WHERE attributes @> '{"brand": "Apple"}';

-- البحث بـ key exists
SELECT * FROM products WHERE attributes ? 'color';

-- تحديث قيمة في JSONB
UPDATE products
SET attributes = jsonb_set(attributes, '{color}', '"Red"')
WHERE name = 'iPhone 15';

-- إضافة key جديد
UPDATE products
SET attributes = attributes || '{"warranty": "2 years"}'
WHERE name = 'iPhone 15';

-- حذف key
UPDATE products
SET attributes = attributes - 'warranty'
WHERE name = 'iPhone 15';
```

<div dir="rtl">

### Indexing JSONB

</div>

```sql
-- GIN index للبحث العام
CREATE INDEX idx_products_attributes ON products USING GIN(attributes);

-- Index لـ key معين
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));
```

---

## 📚 ARRAY - المصفوفات

<div dir="rtl">

### إنشاء واستخدام Arrays

</div>

```sql
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    tags TEXT[],                    -- array of text
    ratings INTEGER[],              -- array of integers
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- إدخال البيانات
INSERT INTO articles (title, tags, ratings) VALUES
    ('Learn PostgreSQL', ARRAY['database', 'sql', 'backend'], ARRAY[5, 4, 5, 5]),
    ('Go Programming', '{"golang", "programming", "backend"}', '{4, 5, 4}'),
    ('React Basics', ARRAY['frontend', 'javascript', 'react'], ARRAY[5, 5, 4, 5, 5]);

-- الوصول للعناصر (index يبدأ من 1!)
SELECT title, tags[1] AS first_tag FROM articles;

-- slice
SELECT title, tags[1:2] AS first_two_tags FROM articles;
```

<div dir="rtl">

### استعلامات Arrays

</div>

```sql
-- البحث عن قيمة في array
SELECT * FROM articles WHERE 'backend' = ANY(tags);
SELECT * FROM articles WHERE tags @> ARRAY['backend'];

-- البحث عن كل القيم
SELECT * FROM articles WHERE tags @> ARRAY['database', 'sql'];

-- عدد العناصر
SELECT title, array_length(tags, 1) AS tags_count FROM articles;

-- تحويل array لـ rows
SELECT title, unnest(tags) AS tag FROM articles;

-- تجميع قيم في array
SELECT array_agg(title) FROM articles WHERE 'backend' = ANY(tags);

-- إضافة عنصر
UPDATE articles
SET tags = array_append(tags, 'tutorial')
WHERE id = 1;

-- حذف عنصر
UPDATE articles
SET tags = array_remove(tags, 'tutorial')
WHERE id = 1;

-- متوسط التقييمات
SELECT
    title,
    ROUND(AVG(r)::numeric, 2) AS avg_rating
FROM articles, unnest(ratings) AS r
GROUP BY title;
```

---

## 🏷️ ENUM - القيم المحددة

<div dir="rtl">

### إنشاء واستخدام ENUM

</div>

```sql
-- إنشاء نوع ENUM
CREATE TYPE order_status AS ENUM (
    'pending',
    'processing',
    'shipped',
    'delivered',
    'cancelled'
);

CREATE TYPE user_role AS ENUM ('admin', 'moderator', 'user', 'guest');

-- استخدام ENUM
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    status order_status DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    role user_role DEFAULT 'user'
);

-- إدخال البيانات
INSERT INTO orders (user_id, status) VALUES (1, 'pending');
INSERT INTO orders (user_id, status) VALUES (2, 'shipped');

-- قيمة غير موجودة في الـ ENUM
INSERT INTO orders (user_id, status) VALUES (3, 'unknown');
-- ERROR: invalid input value for enum order_status: "unknown"
```

<div dir="rtl">

### إدارة ENUM

</div>

```sql
-- عرض قيم ENUM
SELECT enum_range(NULL::order_status);
-- {pending,processing,shipped,delivered,cancelled}

-- إضافة قيمة جديدة
ALTER TYPE order_status ADD VALUE 'returned';
ALTER TYPE order_status ADD VALUE 'on_hold' AFTER 'processing';

-- ⚠️ لا يمكن حذف قيمة من ENUM بسهولة!

-- مقارنة (حسب ترتيب التعريف)
SELECT * FROM orders WHERE status > 'processing';
-- هيرجع shipped, delivered, cancelled
```

<div dir="rtl">

### ENUM vs VARCHAR مع CHECK

</div>

```sql
-- طريقة 1: ENUM
status order_status DEFAULT 'pending'

-- طريقة 2: VARCHAR مع CHECK
status VARCHAR(20) DEFAULT 'pending'
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))

-- ENUM أفضل لما:
-- ✅ القيم ثابتة ونادراً تتغير
-- ✅ محتاج type safety
-- ✅ محتاج ترتيب معين

-- VARCHAR + CHECK أفضل لما:
-- ✅ القيم ممكن تتغير كتير
-- ✅ محتاج flexibility
-- ✅ مش محتاج migrations معقدة
```

---

## 🌐 Network Types

<div dir="rtl">

### INET و CIDR

</div>

```sql
CREATE TABLE servers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    ip_address INET NOT NULL,
    network CIDR
);

INSERT INTO servers (name, ip_address, network) VALUES
    ('Web Server', '192.168.1.100', '192.168.1.0/24'),
    ('DB Server', '10.0.0.50', '10.0.0.0/8');

-- استعلامات
SELECT * FROM servers WHERE ip_address << '192.168.0.0/16';
-- IP addresses في الـ range ده

SELECT * FROM servers WHERE ip_address && '192.168.1.0/24';
-- overlaps مع الـ network
```

<div dir="rtl">

### MACADDR

</div>

```sql
CREATE TABLE devices (
    id SERIAL PRIMARY KEY,
    device_name VARCHAR(100),
    mac_address MACADDR NOT NULL UNIQUE
);

INSERT INTO devices (device_name, mac_address)
VALUES ('Router', '08:00:2b:01:02:03');
```

---

## 💰 Money Type

<div dir="rtl">

### ⚠️ تحذير: لا تستخدم MONEY!

</div>

```sql
-- ❌ لا تستخدم MONEY type
price MONEY  -- مشاكل مع الـ locale والتحويلات

-- ✅ استخدم NUMERIC بدلاً منه
price NUMERIC(10, 2)

-- ليه؟
-- MONEY بيعتمد على locale setting
-- MONEY صعب في التحويل بين العملات
-- NUMERIC أكثر دقة وتحكم
```

---

## 📍 Geometric Types

<div dir="rtl">

### أنواع الأشكال الهندسية

</div>

```sql
-- POINT
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    coordinates POINT
);

INSERT INTO locations (name, coordinates)
VALUES ('Cairo Tower', POINT(31.2243, 30.0459));

-- LINE, LSEG, BOX, PATH, POLYGON, CIRCLE
-- للاستخدامات الهندسية المتقدمة

-- للـ Geographic data الحقيقي، استخدم PostGIS extension
```

---

## 🔧 Composite Types

<div dir="rtl">

### إنشاء أنواع مركبة

</div>

```sql
-- إنشاء نوع مركب
CREATE TYPE address AS (
    street VARCHAR(200),
    city VARCHAR(100),
    country VARCHAR(100),
    postal_code VARCHAR(20)
);

CREATE TABLE companies (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    headquarters address,
    branch_office address
);

-- الإدخال
INSERT INTO companies (name, headquarters, branch_office)
VALUES (
    'Tech Corp',
    ROW('123 Main St', 'Cairo', 'Egypt', '12345'),
    ROW('456 Branch Ave', 'Alexandria', 'Egypt', '54321')
);

-- الاستعلام
SELECT name, (headquarters).city, (branch_office).city FROM companies;
```

---

## 📋 ملخص الأنواع الخاصة

| النوع | الاستخدام | ملاحظات |
|-------|----------|---------|
| `BOOLEAN` | Flags, states | TRUE/FALSE/NULL |
| `UUID` | Unique identifiers | للـ public IDs |
| `JSONB` | Flexible data | أفضل من JSON |
| `ARRAY` | Lists | index يبدأ من 1 |
| `ENUM` | Fixed values | صعب التعديل |
| `INET` | IP addresses | مع network functions |

---

## ✅ Key Takeaways

<div dir="rtl">

1. **JSONB** أفضل من JSON - استخدمه دايماً
2. **UUID** ممتاز للـ public-facing IDs
3. **ARRAY** مفيد بس استخدمه بحذر
4. **ENUM** كويس للقيم الثابتة
5. **لا تستخدم MONEY** - استخدم NUMERIC

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [إنشاء الجداول](./06-create-table.md)**

</div>

---

<div align="center">

[⬅️ السابق: أنواع التاريخ والوقت](./04-datetime-types.md) | [🏠 العودة للـ Module](../README.md)

</div>
