# أنواع الأرقام في PostgreSQL 🔢

<div dir="rtl">

## مقدمة

الأرقام من أهم أنواع البيانات في أي Database. PostgreSQL بيقدم مجموعة كبيرة من أنواع الأرقام لتناسب كل الاحتياجات - من أرقام صغيرة لأرقام ضخمة، ومن أرقام صحيحة لأرقام عشرية.

**المدة المتوقعة:** 25-30 دقيقة

</div>

---

## 📊 نظرة عامة

```
┌─────────────────────────────────────────────────────────────────┐
│                     PostgreSQL Numeric Types                     │
├──────────────────────┬───────────────────┬──────────────────────┤
│   Integer Types      │  Floating-Point   │   Exact Numeric      │
├──────────────────────┼───────────────────┼──────────────────────┤
│ SMALLINT (2 bytes)   │ REAL (4 bytes)    │ NUMERIC(p,s)         │
│ INTEGER  (4 bytes)   │ DOUBLE (8 bytes)  │ DECIMAL(p,s)         │
│ BIGINT   (8 bytes)   │                   │                      │
├──────────────────────┴───────────────────┴──────────────────────┤
│                        Auto-Increment                            │
├─────────────────────────────────────────────────────────────────┤
│ SMALLSERIAL (2 bytes) │ SERIAL (4 bytes) │ BIGSERIAL (8 bytes)  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Integer Types (الأعداد الصحيحة)

<div dir="rtl">

### SMALLINT

- **الحجم:** 2 bytes
- **المدى:** -32,768 إلى 32,767
- **الاستخدام:** أرقام صغيرة جداً (العمر، الكمية الصغيرة)

</div>

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    age SMALLINT CHECK (age >= 0 AND age <= 120),
    grade SMALLINT CHECK (grade >= 1 AND grade <= 12)
);

INSERT INTO students (age, grade) VALUES (15, 10);
```

<div dir="rtl">

### INTEGER (أو INT)

- **الحجم:** 4 bytes
- **المدى:** -2,147,483,648 إلى 2,147,483,647 (حوالي ±2 مليار)
- **الاستخدام:** معظم الأرقام الصحيحة (IDs, counts, etc.)
- ✅ **الأكثر شيوعاً**

</div>

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    stock INTEGER DEFAULT 0,
    views INTEGER DEFAULT 0,
    orders_count INTEGER DEFAULT 0
);

-- مثال على القيم
INSERT INTO products (stock, views)
VALUES (1000000, 2000000000);  -- مليون و2 مليار
```

<div dir="rtl">

### BIGINT

- **الحجم:** 8 bytes
- **المدى:** حوالي ±9 كوينتليون (9 × 10^18)
- **الاستخدام:** أرقام ضخمة جداً (financial, analytics)

</div>

```sql
CREATE TABLE analytics (
    id SERIAL PRIMARY KEY,
    total_views BIGINT DEFAULT 0,
    total_bytes_transferred BIGINT DEFAULT 0
);

-- مثال: YouTube views
INSERT INTO analytics (total_views)
VALUES (10000000000);  -- 10 مليار view
```

<div dir="rtl">

### مقارنة Integer Types

</div>

| النوع | الحجم | المدى | الاستخدام |
|-------|-------|-------|----------|
| `SMALLINT` | 2 bytes | ±32,767 | العمر، الترتيب |
| `INTEGER` | 4 bytes | ±2.1 مليار | معظم الاستخدامات |
| `BIGINT` | 8 bytes | ±9 كوينتليون | إحصائيات ضخمة |

---

## 🔄 Serial Types (Auto-Increment)

<div dir="rtl">

Serial هو نوع خاص للـ IDs التي تزيد تلقائياً.

</div>

```sql
-- الطريقة القديمة (SERIAL)
CREATE TABLE users_old (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- الطريقة الحديثة (IDENTITY) - PostgreSQL 10+
CREATE TABLE users_new (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100)
);

-- BIGSERIAL لـ IDs ضخمة
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    event_data JSONB
);
```

<div dir="rtl">

### الفرق بين SERIAL و IDENTITY

</div>

```sql
-- SERIAL: يمكن تجاوزه بسهولة
INSERT INTO users_old (id, name) VALUES (999, 'Ahmed');  -- ✅ هيشتغل

-- IDENTITY: أكثر أماناً
INSERT INTO users_new (id, name) VALUES (999, 'Ahmed');
-- ERROR: cannot insert into column "id"

-- لو محتاج تحدد الـ ID
INSERT INTO users_new (id, name)
OVERRIDING SYSTEM VALUE
VALUES (999, 'Ahmed');  -- هيشتغل بـ OVERRIDING
```

---

## 💰 Exact Numeric Types (للأرقام الدقيقة)

<div dir="rtl">

### NUMERIC / DECIMAL

**مهم جداً للفلوس!** الـ floating-point types (REAL, DOUBLE) ممكن تعمل أخطاء في الأرقام العشرية.

</div>

```sql
-- Syntax: NUMERIC(precision, scale)
-- precision = إجمالي الأرقام
-- scale = الأرقام بعد العلامة العشرية

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price NUMERIC(10, 2),        -- 10 أرقام، 2 منهم عشري
    tax_rate NUMERIC(5, 4),      -- 5 أرقام، 4 منهم عشري (مثل 0.1500)
    weight NUMERIC(8, 3)         -- 8 أرقام، 3 منهم عشري
);

INSERT INTO products (name, price, tax_rate, weight)
VALUES ('Laptop', 1299.99, 0.1400, 2.500);
```

<div dir="rtl">

### أمثلة على Precision و Scale

</div>

```sql
-- NUMERIC(10, 2) يقدر يخزن:
-- 99999999.99   ← أكبر قيمة
-- -99999999.99  ← أصغر قيمة
-- 1234.56       ← عادي
-- 0.99          ← عادي

-- NUMERIC(5, 4) يقدر يخزن:
-- 9.9999        ← أكبر قيمة
-- 0.1234        ← عادي
-- 0.0001        ← عادي
```

<div dir="rtl">

### ليه NUMERIC للفلوس؟

</div>

```sql
-- مشكلة REAL/DOUBLE
SELECT 0.1::REAL + 0.2::REAL;
-- Result: 0.30000001192092896 ← مش 0.3 بالظبط!

-- حل NUMERIC
SELECT 0.1::NUMERIC + 0.2::NUMERIC;
-- Result: 0.3 ← الإجابة الصحيحة!

-- مثال عملي
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    amount NUMERIC(15, 2) NOT NULL,  -- ✅ استخدم NUMERIC للفلوس
    -- amount REAL NOT NULL          -- ❌ لا تستخدم REAL للفلوس!
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🌊 Floating-Point Types (الأعداد العشرية التقريبية)

<div dir="rtl">

### REAL (4 bytes)

- **الدقة:** 6 أرقام عشرية تقريباً
- **المدى:** ±1E-37 إلى ±1E+37
- **الاستخدام:** إحصائيات، قياسات علمية غير مالية

</div>

```sql
CREATE TABLE sensors (
    id SERIAL PRIMARY KEY,
    temperature REAL,
    humidity REAL,
    pressure REAL
);

INSERT INTO sensors (temperature, humidity, pressure)
VALUES (23.5, 65.2, 1013.25);
```

<div dir="rtl">

### DOUBLE PRECISION (8 bytes)

- **الدقة:** 15 رقم عشري تقريباً
- **المدى:** ±1E-307 إلى ±1E+308
- **الاستخدام:** حسابات علمية عالية الدقة

</div>

```sql
CREATE TABLE scientific_data (
    id SERIAL PRIMARY KEY,
    measurement DOUBLE PRECISION,
    latitude DOUBLE PRECISION,
    longitude DOUBLE PRECISION
);

INSERT INTO scientific_data (measurement, latitude, longitude)
VALUES (3.14159265358979, 30.0444, 31.2357);  -- القاهرة
```

<div dir="rtl">

### مقارنة Floating-Point Types

</div>

| النوع | الحجم | الدقة | الاستخدام |
|-------|-------|-------|----------|
| `REAL` | 4 bytes | ~6 أرقام | قياسات عادية |
| `DOUBLE PRECISION` | 8 bytes | ~15 رقم | حسابات علمية |

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. استخدام REAL للفلوس

</div>

```sql
-- ❌ غلط
CREATE TABLE orders (
    total REAL  -- لا تفعل هذا!
);

-- ✅ صح
CREATE TABLE orders (
    total NUMERIC(12, 2)  -- استخدم NUMERIC للفلوس
);
```

<div dir="rtl">

### 2. INTEGER لـ IDs ضخمة

</div>

```sql
-- ❌ لو متوقع أكتر من 2 مليار صف
CREATE TABLE events (
    id SERIAL PRIMARY KEY  -- هيفشل بعد 2 مليار
);

-- ✅ استخدم BIGSERIAL
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY
);
```

<div dir="rtl">

### 3. NUMERIC بدون precision

</div>

```sql
-- ⚠️ بيشتغل بس مش مفضل
CREATE TABLE products (
    price NUMERIC  -- هيخزن أي عدد من الأرقام
);

-- ✅ حدد precision و scale
CREATE TABLE products (
    price NUMERIC(10, 2)
);
```

---

## 🧮 العمليات الحسابية

```sql
-- الجمع
SELECT 10 + 5;           -- 15

-- الطرح
SELECT 10 - 5;           -- 5

-- الضرب
SELECT 10 * 5;           -- 50

-- القسمة (Integer division)
SELECT 10 / 3;           -- 3 (مش 3.33!)

-- القسمة (Decimal)
SELECT 10.0 / 3;         -- 3.3333333333333333
SELECT 10::NUMERIC / 3;  -- 3.3333333333333333

-- باقي القسمة
SELECT 10 % 3;           -- 1

-- الأس
SELECT 2 ^ 10;           -- 1024
SELECT POWER(2, 10);     -- 1024

-- الجذر التربيعي
SELECT SQRT(16);         -- 4
SELECT |/ 16;            -- 4

-- القيمة المطلقة
SELECT ABS(-10);         -- 10
SELECT @ -10;            -- 10

-- التقريب
SELECT ROUND(3.7);       -- 4
SELECT ROUND(3.14159, 2); -- 3.14
SELECT CEIL(3.1);        -- 4
SELECT FLOOR(3.9);       -- 3
SELECT TRUNC(3.9);       -- 3
```

---

## 📋 جدول مرجعي سريع

```
┌────────────────────┬──────────┬─────────────────────────────────────┐
│      النوع         │  الحجم   │            الاستخدام               │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ SMALLINT           │ 2 bytes  │ أرقام صغيرة (العمر، الترتيب)       │
│ INTEGER/INT        │ 4 bytes  │ معظم الأرقام الصحيحة ✅            │
│ BIGINT             │ 8 bytes  │ أرقام ضخمة جداً                    │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ SERIAL             │ 4 bytes  │ Auto-increment IDs                 │
│ BIGSERIAL          │ 8 bytes  │ Auto-increment IDs ضخمة            │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ NUMERIC(p,s)       │ variable │ الفلوس والأرقام الدقيقة ✅         │
│ DECIMAL(p,s)       │ variable │ نفس NUMERIC                        │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ REAL               │ 4 bytes  │ أرقام عشرية تقريبية                │
│ DOUBLE PRECISION   │ 8 bytes  │ أرقام عشرية عالية الدقة            │
└────────────────────┴──────────┴─────────────────────────────────────┘
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. اختيار النوع المناسب

</div>

```sql
-- ✅ للـ IDs
id SERIAL PRIMARY KEY          -- أو BIGSERIAL لو متوقع أكتر من 2 مليار

-- ✅ للفلوس
price NUMERIC(10, 2)           -- 10 أرقام، 2 عشري
amount NUMERIC(15, 2)          -- للمبالغ الكبيرة

-- ✅ للنسب المئوية
percentage NUMERIC(5, 2)       -- 0.00 to 100.00

-- ✅ للإحداثيات
latitude DOUBLE PRECISION
longitude DOUBLE PRECISION

-- ✅ للعمر
age SMALLINT CHECK (age >= 0 AND age <= 150)
```

<div dir="rtl">

### 2. استخدم CHECK constraints

</div>

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price NUMERIC(10, 2) CHECK (price > 0),
    stock INTEGER CHECK (stock >= 0),
    discount NUMERIC(5, 2) CHECK (discount >= 0 AND discount <= 100)
);
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **INTEGER** هو الأكثر استخداماً للأعداد الصحيحة
2. **BIGINT** للأرقام الضخمة جداً
3. **NUMERIC/DECIMAL** للفلوس - لا تستخدم REAL أو DOUBLE!
4. **SERIAL** للـ Auto-increment IDs
5. **DOUBLE PRECISION** للحسابات العلمية

</div>

---

## 🧪 اختبر نفسك

<div dir="rtl">

1. ما الفرق بين INTEGER و BIGINT؟
2. لماذا لا نستخدم REAL للفلوس؟
3. ما معنى NUMERIC(10, 2)؟
4. ما الفرق بين SERIAL و IDENTITY؟
5. اكتب CREATE TABLE لجدول منتجات مع سعر وكمية

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [أنواع النصوص](./03-text-types.md)**

</div>

---

<div align="center">

[⬅️ السابق: مقدمة في SQL](./01-intro-to-sql.md) | [🏠 العودة للـ Module](../README.md)

</div>
