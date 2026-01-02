# JSON/JSONB Basics - التعامل مع JSON 📦

<div dir="rtl">

## مقدمة

PostgreSQL بيدعم تخزين والاستعلام عن JSON data بشكل native. ده بيسمح بمرونة في الـ schema مع الحفاظ على قوة الـ relational database.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 JSON vs JSONB

```
┌─────────────────────────────────────────────────────────────────────┐
│                       JSON vs JSONB                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Feature              │   JSON         │   JSONB                   │
│   ─────────────────────┼────────────────┼─────────────────────────  │
│   Storage              │ Text (exact)   │ Binary (decomposed)       │
│   Preserves whitespace │ Yes            │ No                        │
│   Preserves key order  │ Yes            │ No                        │
│   Duplicate keys       │ Allowed        │ Last one wins             │
│   Indexing             │ No             │ Yes (GIN)                 │
│   Write speed          │ Faster         │ Slower (parsing)          │
│   Read speed           │ Slower         │ Faster (binary)           │
│   Operators            │ Limited        │ Full (containment, etc)   │
│                                                                      │
│   Recommendation: استخدم JSONB في معظم الحالات!                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 إنشاء جداول مع JSON

```sql
-- جدول منتجات مع metadata
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول أحداث (events log)
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    payload JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول إعدادات المستخدم
CREATE TABLE user_settings (
    user_id INT PRIMARY KEY REFERENCES users(id),
    settings JSONB DEFAULT '{
        "notifications": true,
        "theme": "light",
        "language": "en"
    }'
);
```

---

## ➕ إدخال JSON Data

```sql
-- إدخال object
INSERT INTO products (name, price, metadata) VALUES
(
    'MacBook Pro',
    1999.99,
    '{
        "brand": "Apple",
        "specs": {
            "cpu": "M3 Pro",
            "ram": "16GB",
            "storage": "512GB"
        },
        "colors": ["Space Gray", "Silver"],
        "in_stock": true
    }'
);

-- إدخال باستخدام jsonb_build_object
INSERT INTO products (name, price, metadata) VALUES
(
    'iPhone 15',
    999.99,
    jsonb_build_object(
        'brand', 'Apple',
        'specs', jsonb_build_object(
            'storage', '256GB',
            'color', 'Blue'
        ),
        'features', ARRAY['5G', 'USB-C', 'Dynamic Island']
    )
);

-- إدخال array
INSERT INTO events (event_type, payload) VALUES
(
    'user_signup',
    '{"user_id": 123, "email": "ahmed@example.com", "source": "google"}'
);
```

---

## 🔍 استخراج البيانات

```sql
-- Operators الأساسية
-- -> : يرجع JSON
-- ->> : يرجع TEXT
-- #> : path إلى JSON
-- #>> : path إلى TEXT

-- استخراج قيمة واحدة
SELECT
    name,
    metadata -> 'brand' as brand_json,      -- "Apple" (JSON)
    metadata ->> 'brand' as brand_text      -- Apple (TEXT)
FROM products;

-- استخراج قيمة متداخلة
SELECT
    name,
    metadata -> 'specs' -> 'cpu' as cpu_json,
    metadata -> 'specs' ->> 'cpu' as cpu_text,
    metadata #>> '{specs, cpu}' as cpu_path  -- نفس النتيجة
FROM products;

-- استخراج من array
SELECT
    name,
    metadata -> 'colors' -> 0 as first_color,      -- "Space Gray"
    metadata -> 'colors' ->> 0 as first_color_text -- Space Gray
FROM products;

-- استخراج كل عناصر array
SELECT name, jsonb_array_elements_text(metadata -> 'colors') as color
FROM products
WHERE metadata ? 'colors';
```

---

## 🔍 البحث والفلترة

```sql
-- البحث بقيمة محددة
SELECT * FROM products
WHERE metadata ->> 'brand' = 'Apple';

-- البحث في قيمة متداخلة
SELECT * FROM products
WHERE metadata -> 'specs' ->> 'cpu' = 'M3 Pro';

-- البحث بـ containment (@>)
SELECT * FROM products
WHERE metadata @> '{"brand": "Apple"}';

-- البحث في array
SELECT * FROM products
WHERE metadata -> 'colors' ? 'Silver';

-- البحث بـ key existence
SELECT * FROM products WHERE metadata ? 'brand';
SELECT * FROM products WHERE metadata ?| ARRAY['brand', 'warranty'];  -- أي منهم
SELECT * FROM products WHERE metadata ?& ARRAY['brand', 'specs'];      -- كلهم

-- البحث بـ path
SELECT * FROM products
WHERE metadata #>> '{specs, storage}' = '512GB';

-- مقارنات
SELECT * FROM products
WHERE (metadata ->> 'price')::numeric > 500;

-- NULL check
SELECT * FROM products
WHERE metadata -> 'warranty' IS NULL;
```

---

## ✏️ تحديث JSON

```sql
-- تحديث قيمة واحدة
UPDATE products
SET metadata = jsonb_set(metadata, '{brand}', '"Samsung"')
WHERE id = 1;

-- تحديث قيمة متداخلة
UPDATE products
SET metadata = jsonb_set(metadata, '{specs, ram}', '"32GB"')
WHERE id = 1;

-- إضافة key جديد
UPDATE products
SET metadata = metadata || '{"warranty": "2 years"}'
WHERE id = 1;

-- إضافة لـ array
UPDATE products
SET metadata = jsonb_set(
    metadata,
    '{colors}',
    metadata -> 'colors' || '"Gold"'::jsonb
)
WHERE id = 1;

-- حذف key
UPDATE products
SET metadata = metadata - 'warranty'
WHERE id = 1;

-- حذف من path
UPDATE products
SET metadata = metadata #- '{specs, color}'
WHERE id = 1;

-- تحديث أو إضافة (upsert في JSON)
UPDATE products
SET metadata = metadata || '{"rating": 4.5}'::jsonb
WHERE id = 1;
```

---

## 📊 Functions مفيدة

```sql
-- jsonb_each: تحويل object لـ rows
SELECT key, value
FROM products, jsonb_each(metadata -> 'specs')
WHERE id = 1;

-- jsonb_each_text: نفس الشيء لكن TEXT
SELECT key, value
FROM products, jsonb_each_text(metadata -> 'specs')
WHERE id = 1;

-- jsonb_object_keys: الـ keys فقط
SELECT jsonb_object_keys(metadata)
FROM products WHERE id = 1;

-- jsonb_array_elements: عناصر الـ array
SELECT jsonb_array_elements(metadata -> 'colors')
FROM products WHERE id = 1;

-- jsonb_array_length: طول الـ array
SELECT name, jsonb_array_length(metadata -> 'colors') as color_count
FROM products;

-- jsonb_typeof: نوع الـ JSON
SELECT jsonb_typeof(metadata -> 'brand') as brand_type,
       jsonb_typeof(metadata -> 'colors') as colors_type,
       jsonb_typeof(metadata -> 'in_stock') as stock_type
FROM products WHERE id = 1;
-- Result: string, array, boolean

-- jsonb_pretty: formatting
SELECT jsonb_pretty(metadata) FROM products WHERE id = 1;

-- jsonb_strip_nulls: إزالة null values
SELECT jsonb_strip_nulls('{"a": 1, "b": null, "c": 3}'::jsonb);
-- Result: {"a": 1, "c": 3}
```

---

## 🏗️ Aggregation مع JSON

```sql
-- تجميع rows كـ JSON array
SELECT jsonb_agg(jsonb_build_object(
    'id', id,
    'name', name,
    'price', price
)) as products
FROM products
WHERE price > 500;

-- تجميع كـ object
SELECT jsonb_object_agg(name, price) as price_list
FROM products;

-- مع GROUP BY
SELECT
    metadata ->> 'brand' as brand,
    jsonb_agg(name) as products,
    COUNT(*) as count
FROM products
GROUP BY metadata ->> 'brand';
```

---

## 📦 Use Cases

<div dir="rtl">

### 1. User Settings

</div>

```sql
-- جدول الإعدادات
CREATE TABLE user_preferences (
    user_id INT PRIMARY KEY,
    prefs JSONB DEFAULT '{}'
);

-- إعدادات default
INSERT INTO user_preferences (user_id, prefs)
VALUES (1, '{
    "notifications": {
        "email": true,
        "push": true,
        "sms": false
    },
    "theme": "dark",
    "language": "ar",
    "timezone": "Africa/Cairo"
}');

-- تحديث إعداد معين
UPDATE user_preferences
SET prefs = jsonb_set(prefs, '{theme}', '"light"')
WHERE user_id = 1;

-- دمج مع defaults
SELECT
    COALESCE(prefs ->> 'theme', 'light') as theme,
    COALESCE(prefs -> 'notifications' ->> 'email', 'true')::boolean as email_notif
FROM user_preferences
WHERE user_id = 1;
```

<div dir="rtl">

### 2. Product Variants

</div>

```sql
CREATE TABLE product_variants (
    id SERIAL PRIMARY KEY,
    product_id INT REFERENCES products(id),
    sku VARCHAR(50) UNIQUE,
    attributes JSONB NOT NULL,  -- {"size": "L", "color": "Red"}
    price DECIMAL(10, 2),
    stock INT DEFAULT 0
);

-- البحث عن variant معين
SELECT * FROM product_variants
WHERE product_id = 1
  AND attributes @> '{"size": "L", "color": "Red"}';

-- كل الـ sizes المتاحة
SELECT DISTINCT attributes ->> 'size' as size
FROM product_variants
WHERE product_id = 1;
```

<div dir="rtl">

### 3. Event Logging

</div>

```sql
CREATE TABLE event_logs (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    actor_id INT,
    payload JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Log event
INSERT INTO event_logs (event_type, actor_id, payload)
VALUES ('order_created', 123, '{
    "order_id": 456,
    "items": [
        {"product_id": 1, "quantity": 2},
        {"product_id": 3, "quantity": 1}
    ],
    "total": 199.99
}');

-- Query events
SELECT
    created_at,
    payload ->> 'order_id' as order_id,
    payload ->> 'total' as total,
    jsonb_array_length(payload -> 'items') as item_count
FROM event_logs
WHERE event_type = 'order_created'
  AND created_at > NOW() - INTERVAL '7 days';
```

---

## ⚠️ Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                      JSONB Best Practices                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. استخدم JSONB مش JSON                                             │
│     أسرع للقراءة وبيدعم indexing                                    │
│                                                                      │
│  2. لا تخزن كل شيء في JSON                                           │
│     الأعمدة المستخدمة كثيراً في WHERE = أعمدة عادية                  │
│                                                                      │
│  3. استخدم GIN index للبحث                                           │
│     CREATE INDEX ON products USING gin(metadata);                   │
│                                                                      │
│  4. Validate JSON في الـ application                                 │
│     أو استخدم CHECK constraint                                       │
│                                                                      │
│  5. Document الـ schema المتوقع                                       │
│     JSON flexible لكن محتاج documentation                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. استخدم **JSONB** مش JSON
2. **-> للـ JSON، ->> للـ TEXT**
3. **@>** للـ containment
4. **?** للـ key existence
5. **jsonb_set** للتحديث
6. **||** للدمج، **-** للحذف

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [JSONB Operators](./05-jsonb-operators.md)**

</div>

---

<div align="center">

[⬅️ السابق](./03-query-optimization.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./05-jsonb-operators.md)

</div>
