# INSERT ... ON CONFLICT (Upsert) ⚔️

<div dir="rtl">

## مقدمة

**Upsert** = **Up**date + In**sert** - لو البيانات موجودة حدّثها، لو مش موجودة أضفها. PostgreSQL بتدعم ده من خلال `INSERT ... ON CONFLICT`.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
INSERT INTO table_name (columns)
VALUES (values)
ON CONFLICT (conflict_target)
DO NOTHING | DO UPDATE SET column = value;
```

---

## 🎯 ON CONFLICT DO NOTHING

<div dir="rtl">

### تجاهل الـ duplicates

</div>

```sql
-- لو الـ username موجود، متعملش حاجة
INSERT INTO users (username, email, password_hash)
VALUES ('ahmed', 'ahmed@example.com', 'hash')
ON CONFLICT (username) DO NOTHING;

-- لو الـ email موجود، متعملش حاجة
INSERT INTO users (username, email, password_hash)
VALUES ('new_user', 'existing@email.com', 'hash')
ON CONFLICT (email) DO NOTHING;

-- Bulk insert مع تجاهل الـ duplicates
INSERT INTO products (sku, name, price) VALUES
    ('SKU001', 'Product 1', 99.99),
    ('SKU002', 'Product 2', 149.99),
    ('SKU001', 'Duplicate', 199.99)  -- هيتجاهل لو SKU001 موجود
ON CONFLICT (sku) DO NOTHING;
```

---

## 🔄 ON CONFLICT DO UPDATE

<div dir="rtl">

### تحديث لو موجود

</div>

```sql
-- تحديث السعر والمخزون لو الـ SKU موجود
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU001', 'Updated Product', 149.99, 100)
ON CONFLICT (sku) DO UPDATE SET
    name = EXCLUDED.name,
    price = EXCLUDED.price,
    stock = EXCLUDED.stock,
    updated_at = NOW();

-- EXCLUDED = القيم الجديدة اللي حاولنا نضيفها
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                          EXCLUDED                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INSERT INTO products (sku, name, price)                            │
│  VALUES ('SKU001', 'New Name', 199.99)                              │
│  ON CONFLICT (sku) DO UPDATE SET                                     │
│      name = EXCLUDED.name,     -- 'New Name'                        │
│      price = EXCLUDED.price;   -- 199.99                            │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  EXCLUDED = الصف اللي حاولت تدخله                            │   │
│  │  ├── EXCLUDED.sku   = 'SKU001'                                │   │
│  │  ├── EXCLUDED.name  = 'New Name'                              │   │
│  │  └── EXCLUDED.price = 199.99                                  │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  products = الصف الموجود حالياً في الجدول                           │
│  ├── products.sku   = 'SKU001'                                      │
│  ├── products.name  = 'Old Name'                                    │
│  └── products.price = 99.99                                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Conflict Targets مختلفة

<div dir="rtl">

### على عمود واحد

</div>

```sql
-- على PRIMARY KEY
INSERT INTO users (id, username, email, password_hash)
VALUES (1, 'updated_user', 'updated@email.com', 'new_hash')
ON CONFLICT (id) DO UPDATE SET
    username = EXCLUDED.username,
    email = EXCLUDED.email;

-- على UNIQUE column
INSERT INTO products (sku, name, price)
VALUES ('EXISTING_SKU', 'New Name', 299.99)
ON CONFLICT (sku) DO UPDATE SET
    name = EXCLUDED.name,
    price = EXCLUDED.price;
```

<div dir="rtl">

### على عدة أعمدة (Composite)

</div>

```sql
-- جدول مع unique على أكثر من عمود
CREATE TABLE product_prices (
    product_id INT,
    region VARCHAR(10),
    price NUMERIC(10, 2),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (product_id, region)
);

-- Upsert على الـ composite unique
INSERT INTO product_prices (product_id, region, price)
VALUES (1, 'EG', 999.99)
ON CONFLICT (product_id, region) DO UPDATE SET
    price = EXCLUDED.price,
    updated_at = NOW();
```

<div dir="rtl">

### على Constraint بالاسم

</div>

```sql
-- استخدام اسم الـ constraint
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 99.99)
ON CONFLICT ON CONSTRAINT products_sku_key DO UPDATE SET
    price = EXCLUDED.price;

-- أو على PRIMARY KEY
INSERT INTO users (id, username, email, password_hash)
VALUES (1, 'user', 'email@test.com', 'hash')
ON CONFLICT ON CONSTRAINT users_pkey DO UPDATE SET
    username = EXCLUDED.username;
```

---

## 🎯 WHERE مع DO UPDATE

<div dir="rtl">

### تحديث بشرط

</div>

```sql
-- تحديث فقط لو السعر الجديد أعلى
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 199.99)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price
WHERE products.price < EXCLUDED.price;
-- لو السعر الحالي أعلى، مش هيحصل تحديث

-- تحديث فقط لو المنتج available
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU001', 'Updated', 149.99, 50)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price,
    stock = EXCLUDED.stock
WHERE products.is_available = TRUE;

-- زيادة المخزون فقط (مش استبدال)
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU001', 'Product', 99.99, 10)
ON CONFLICT (sku) DO UPDATE SET
    stock = products.stock + EXCLUDED.stock
WHERE products.stock + EXCLUDED.stock <= 1000;  -- حد أقصى للمخزون
```

---

## 📊 أنماط Upsert شائعة

<div dir="rtl">

### 1. Sync Pattern - مزامنة البيانات

</div>

```sql
-- مزامنة بيانات من API أو نظام خارجي
INSERT INTO customers (external_id, name, email, phone, synced_at)
VALUES
    ('EXT001', 'Ahmed Ali', 'ahmed@test.com', '01234567890', NOW()),
    ('EXT002', 'Sara Mohamed', 'sara@test.com', '01234567891', NOW()),
    ('EXT003', 'Omar Hassan', 'omar@test.com', '01234567892', NOW())
ON CONFLICT (external_id) DO UPDATE SET
    name = EXCLUDED.name,
    email = EXCLUDED.email,
    phone = EXCLUDED.phone,
    synced_at = NOW();
```

<div dir="rtl">

### 2. Counter Pattern - العدادات

</div>

```sql
-- جدول لتتبع views/clicks
CREATE TABLE page_views (
    page_url TEXT PRIMARY KEY,
    view_count INT DEFAULT 0,
    last_viewed_at TIMESTAMPTZ DEFAULT NOW()
);

-- زيادة العداد أو إنشاؤه
INSERT INTO page_views (page_url, view_count, last_viewed_at)
VALUES ('/products/laptop', 1, NOW())
ON CONFLICT (page_url) DO UPDATE SET
    view_count = page_views.view_count + 1,
    last_viewed_at = NOW();
```

<div dir="rtl">

### 3. Cache Pattern - التخزين المؤقت

</div>

```sql
-- جدول cache
CREATE TABLE api_cache (
    cache_key TEXT PRIMARY KEY,
    cache_value JSONB,
    expires_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- تخزين أو تحديث cache
INSERT INTO api_cache (cache_key, cache_value, expires_at)
VALUES (
    'user_profile_123',
    '{"name": "Ahmed", "email": "ahmed@test.com"}'::jsonb,
    NOW() + INTERVAL '1 hour'
)
ON CONFLICT (cache_key) DO UPDATE SET
    cache_value = EXCLUDED.cache_value,
    expires_at = EXCLUDED.expires_at;
```

<div dir="rtl">

### 4. Settings Pattern - الإعدادات

</div>

```sql
-- جدول إعدادات key-value
CREATE TABLE user_settings (
    user_id INT,
    setting_key VARCHAR(100),
    setting_value TEXT,
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (user_id, setting_key)
);

-- تحديث أو إنشاء إعداد
INSERT INTO user_settings (user_id, setting_key, setting_value)
VALUES (1, 'theme', 'dark')
ON CONFLICT (user_id, setting_key) DO UPDATE SET
    setting_value = EXCLUDED.setting_value,
    updated_at = NOW();
```

---

## 🔙 ON CONFLICT مع RETURNING

```sql
-- إرجاع البيانات سواء تم إدخالها أو تحديثها
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 149.99)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price,
    updated_at = NOW()
RETURNING id, sku, price, updated_at;

-- معرفة هل تم INSERT أو UPDATE
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 149.99)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price
RETURNING
    id,
    sku,
    CASE WHEN xmax = 0 THEN 'inserted' ELSE 'updated' END AS action;
-- xmax = 0 يعني INSERT جديد
```

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. نسيان تحديث updated_at

</div>

```sql
-- ❌ غلط: updated_at مش بيتحدث
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 99.99)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price;

-- ✅ صح: تحديث updated_at
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 99.99)
ON CONFLICT (sku) DO UPDATE SET
    price = EXCLUDED.price,
    updated_at = NOW();
```

<div dir="rtl">

### 2. استخدام عمود مش UNIQUE

</div>

```sql
-- ❌ خطأ: name مش UNIQUE
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 99.99)
ON CONFLICT (name) DO UPDATE SET price = EXCLUDED.price;
-- ERROR: there is no unique or exclusion constraint matching the ON CONFLICT

-- الحل: أنشئ UNIQUE constraint أولاً أو استخدم عمود UNIQUE
```

<div dir="rtl">

### 3. Conflict على أكثر من constraint

</div>

```sql
-- لو عندك UNIQUE على sku و email
-- لازم تختار واحد بس في ON CONFLICT
INSERT INTO products (sku, name, price, email)
VALUES ('SKU001', 'Product', 99.99, 'product@test.com')
ON CONFLICT (sku) DO UPDATE SET ...;  -- ✅

-- ❌ مش ممكن تحدد اثنين
ON CONFLICT (sku, email) DO UPDATE ...;  -- ده composite، مش اثنين منفصلين
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم WHERE لتجنب updates غير ضرورية

</div>

```sql
-- ✅ تحديث فقط لو فيه تغيير فعلي
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 99.99)
ON CONFLICT (sku) DO UPDATE SET
    name = EXCLUDED.name,
    price = EXCLUDED.price,
    updated_at = NOW()
WHERE products.name != EXCLUDED.name
   OR products.price != EXCLUDED.price;
```

<div dir="rtl">

### 2. استخدم RETURNING لمعرفة النتيجة

</div>

```sql
-- ✅ اعرف هل تم insert أو update
INSERT INTO products (sku, name, price)
VALUES ('SKU001', 'Product', 99.99)
ON CONFLICT (sku) DO UPDATE SET price = EXCLUDED.price
RETURNING *, (xmax = 0) AS was_inserted;
```

<div dir="rtl">

### 3. فكر في الـ Partial Index للـ Upsert المشروط

</div>

```sql
-- Unique فقط للمنتجات النشطة
CREATE UNIQUE INDEX products_active_sku_idx
ON products (sku)
WHERE is_available = TRUE;

-- الآن ممكن يكون عندك نفس الـ SKU للمنتجات المحذوفة
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **ON CONFLICT DO NOTHING** لتجاهل الـ duplicates
2. **ON CONFLICT DO UPDATE** للـ Upsert الكامل
3. **EXCLUDED** = القيم الجديدة اللي حاولت تدخلها
4. **WHERE** في DO UPDATE للتحديث المشروط
5. دايماً **حدّث updated_at** في DO UPDATE
6. استخدم **RETURNING** لمعرفة النتيجة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [INSERT ... SELECT](./04-insert-select.md)**

</div>

---

<div align="center">

[⬅️ السابق: RETURNING](./02-insert-returning.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./04-insert-select.md)

</div>
