# أنواع النصوص في PostgreSQL 📝

<div dir="rtl">

## مقدمة

النصوص (Strings) من أكثر أنواع البيانات استخداماً. الأسماء، الإيميلات، العناوين، المحتوى - كلها نصوص. PostgreSQL بيقدم عدة أنواع للنصوص، كل واحد له استخدامه المناسب.

**المدة المتوقعة:** 20-25 دقيقة

</div>

---

## 📊 نظرة عامة

```
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Text Types                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VARCHAR(n)    ─── نص متغير الطول (max n characters)            │
│                    "Hello" → 5 bytes + overhead                  │
│                                                                  │
│  CHAR(n)       ─── نص ثابت الطول (exactly n characters)         │
│                    "Hi" in CHAR(5) → "Hi   " (padded)            │
│                                                                  │
│  TEXT          ─── نص غير محدود الطول                           │
│                    أي طول (حتى 1GB)                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📏 VARCHAR(n) - Variable Character

<div dir="rtl">

### الخصائص

- **الطول:** متغير حتى `n` حرف
- **التخزين:** على حسب المحتوى الفعلي
- **الأداء:** ممتاز للنصوص القصيرة والمتوسطة
- ✅ **الأكثر استخداماً**

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,      -- 50 حرف max
    email VARCHAR(255) NOT NULL,        -- 255 حرف (طول email قياسي)
    phone VARCHAR(20),                  -- 20 حرف للتليفون
    country_code VARCHAR(2)             -- 2 حرف (EG, US, SA, etc.)
);

-- الإدخال
INSERT INTO users (username, email, phone, country_code)
VALUES ('ahmed_ali', 'ahmed@example.com', '+201234567890', 'EG');

-- لو تجاوزت الحد
INSERT INTO users (username, email)
VALUES ('this_is_a_very_long_username_that_exceeds_fifty_characters', 'test@test.com');
-- ERROR: value too long for type character varying(50)
```

<div dir="rtl">

### أطوال شائعة لـ VARCHAR

</div>

```sql
-- القاعدة العامة: اختار حد معقول
username VARCHAR(50)        -- أسماء المستخدمين
email VARCHAR(255)          -- الإيميلات (RFC standard)
password_hash VARCHAR(255)  -- الـ Hashes
phone VARCHAR(20)           -- أرقام التليفون
url VARCHAR(2048)           -- URLs
title VARCHAR(200)          -- عناوين
slug VARCHAR(100)           -- URL slugs
country_code VARCHAR(2)     -- ISO country codes
currency_code VARCHAR(3)    -- ISO currency codes (USD, EGP)
```

---

## 📐 CHAR(n) - Fixed Character

<div dir="rtl">

### الخصائص

- **الطول:** ثابت بالظبط `n` حرف
- **الـ Padding:** يملأ الباقي بمسافات
- **الاستخدام:** أكواد ثابتة الطول (نادر الاستخدام)

</div>

```sql
CREATE TABLE countries (
    code CHAR(2) PRIMARY KEY,    -- EG, US, SA (دايماً حرفين)
    name VARCHAR(100) NOT NULL
);

INSERT INTO countries (code, name) VALUES ('EG', 'Egypt');
INSERT INTO countries (code, name) VALUES ('US', 'United States');

-- لو أدخلت أقل من 2
INSERT INTO countries (code, name) VALUES ('A', 'Test');
-- هيتخزن كـ 'A ' (مسافة في الآخر)

SELECT code, LENGTH(code) FROM countries;
-- 'EG', 2
-- 'A ', 2  -- المسافة محسوبة
```

<div dir="rtl">

### متى تستخدم CHAR؟

</div>

```sql
-- ✅ استخدم CHAR لما الطول دايماً ثابت
country_code CHAR(2)      -- ISO country (EG, US)
currency_code CHAR(3)     -- ISO currency (USD, EGP)
language_code CHAR(2)     -- ISO language (ar, en)
gender CHAR(1)            -- M/F
status_code CHAR(3)       -- مثلاً ACT, PND, CMP

-- ❌ لا تستخدم CHAR لـ
username CHAR(50)         -- استخدم VARCHAR
email CHAR(255)           -- استخدم VARCHAR
```

---

## 📜 TEXT - Unlimited Text

<div dir="rtl">

### الخصائص

- **الطول:** غير محدود (حتى 1GB تقريباً)
- **الأداء:** نفس VARCHAR (في PostgreSQL)
- **الاستخدام:** محتوى طويل (مقالات، تعليقات)

</div>

```sql
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    slug VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,           -- المحتوى الطويل
    summary TEXT,                    -- الملخص
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    article_id INT REFERENCES articles(id),
    author_name VARCHAR(100),
    body TEXT NOT NULL,              -- نص التعليق
    created_at TIMESTAMP DEFAULT NOW()
);
```

<div dir="rtl">

### VARCHAR vs TEXT في PostgreSQL

</div>

```sql
-- في PostgreSQL، الأداء متساوي تقريباً!
-- الفرق الوحيد هو الـ validation

-- VARCHAR(n): يتحقق من الطول
name VARCHAR(100)  -- لو أكتر من 100، error

-- TEXT: لا يتحقق
content TEXT       -- أي طول مقبول

-- 💡 نصيحة:
-- استخدم VARCHAR(n) لما عندك حد منطقي
-- استخدم TEXT لما مفيش حد (مقالات، تعليقات)
```

---

## 🔤 دوال النصوص المهمة

<div dir="rtl">

### 1. الطول والقطع

</div>

```sql
-- الطول
SELECT LENGTH('Hello');                    -- 5
SELECT CHAR_LENGTH('مرحبا');              -- 5 (عدد الحروف)
SELECT OCTET_LENGTH('مرحبا');             -- 10 (عدد الـ bytes)

-- القطع
SELECT SUBSTRING('Hello World' FROM 1 FOR 5);  -- 'Hello'
SELECT LEFT('Hello', 2);                       -- 'He'
SELECT RIGHT('Hello', 2);                      -- 'lo'

-- إزالة المسافات
SELECT TRIM('  Hello  ');                      -- 'Hello'
SELECT LTRIM('  Hello');                       -- 'Hello'
SELECT RTRIM('Hello  ');                       -- 'Hello'
```

<div dir="rtl">

### 2. التحويل

</div>

```sql
-- الحالة (Case)
SELECT UPPER('hello');                     -- 'HELLO'
SELECT LOWER('HELLO');                     -- 'hello'
SELECT INITCAP('hello world');             -- 'Hello World'

-- الاستبدال
SELECT REPLACE('Hello World', 'World', 'PostgreSQL');
-- 'Hello PostgreSQL'

-- عكس النص
SELECT REVERSE('Hello');                   -- 'olleH'
```

<div dir="rtl">

### 3. البحث

</div>

```sql
-- موقع نص داخل نص
SELECT POSITION('World' IN 'Hello World');  -- 7 (يبدأ من 1)
SELECT STRPOS('Hello World', 'World');      -- 7

-- هل يحتوي؟
SELECT 'Hello World' LIKE '%World%';        -- true
SELECT 'Hello World' ILIKE '%world%';       -- true (case-insensitive)

-- هل يبدأ/ينتهي بـ
SELECT 'Hello' LIKE 'He%';                  -- true (يبدأ بـ He)
SELECT 'Hello' LIKE '%lo';                  -- true (ينتهي بـ lo)
```

<div dir="rtl">

### 4. الدمج

</div>

```sql
-- دمج نصوص
SELECT 'Hello' || ' ' || 'World';           -- 'Hello World'
SELECT CONCAT('Hello', ' ', 'World');       -- 'Hello World'
SELECT CONCAT_WS(' ', 'Hello', 'World');    -- 'Hello World' (with separator)

-- مع NULL
SELECT 'Hello' || NULL;                     -- NULL (كل حاجة)
SELECT CONCAT('Hello', NULL, 'World');      -- 'HelloWorld' (يتجاهل NULL)
```

---

## 🔍 LIKE و Pattern Matching

<div dir="rtl">

### LIKE

</div>

```sql
-- % = أي عدد من الأحرف (صفر أو أكتر)
SELECT * FROM users WHERE email LIKE '%@gmail.com';      -- ينتهي بـ @gmail.com
SELECT * FROM users WHERE name LIKE 'Ahmed%';            -- يبدأ بـ Ahmed
SELECT * FROM users WHERE name LIKE '%Ali%';             -- يحتوي Ali

-- _ = حرف واحد بالظبط
SELECT * FROM products WHERE code LIKE 'A__';            -- A + حرفين
SELECT * FROM users WHERE phone LIKE '+20__________';    -- +20 + 10 أرقام

-- Case-insensitive
SELECT * FROM users WHERE name ILIKE '%ahmed%';          -- Ahmed, AHMED, ahmed
```

<div dir="rtl">

### Regular Expressions

</div>

```sql
-- ~ للـ regex (case-sensitive)
-- ~* للـ regex (case-insensitive)

-- إيميلات valid
SELECT * FROM users WHERE email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';

-- أرقام تليفون مصرية
SELECT * FROM users WHERE phone ~ '^\+20[0-9]{10}$';

-- يبدأ بحرف
SELECT * FROM users WHERE username ~ '^[a-zA-Z]';
```

---

## 🌍 Unicode والعربية

<div dir="rtl">

PostgreSQL يدعم Unicode بشكل كامل:

</div>

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL
);

-- النصوص العربية تشتغل تمام
INSERT INTO posts (title, content)
VALUES (
    'مرحباً بالعالم',
    'هذا منشور باللغة العربية. PostgreSQL يدعم جميع اللغات!'
);

-- البحث يشتغل
SELECT * FROM posts WHERE title LIKE '%مرحباً%';

-- الترتيب
SELECT * FROM posts ORDER BY title;  -- الترتيب حسب الـ Collation
```

<div dir="rtl">

### Collation للغة العربية

</div>

```sql
-- تحديد Collation
CREATE TABLE arabic_names (
    name VARCHAR(100) COLLATE "ar_EG"
);

-- أو على مستوى الـ Query
SELECT * FROM users ORDER BY name COLLATE "ar_EG";
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. اختيار النوع المناسب

</div>

```sql
-- ✅ استخدم VARCHAR(n) للنصوص القصيرة/المتوسطة
username VARCHAR(50)
email VARCHAR(255)
title VARCHAR(200)

-- ✅ استخدم TEXT للنصوص الطويلة
content TEXT
bio TEXT
description TEXT

-- ✅ استخدم CHAR(n) للأكواد الثابتة فقط
country_code CHAR(2)
currency_code CHAR(3)
```

<div dir="rtl">

### 2. Constraints مناسبة

</div>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,

    -- تأكد من عدم الفراغ
    CONSTRAINT username_not_empty CHECK (LENGTH(TRIM(username)) > 0),

    -- تأكد من شكل الإيميل (regex بسيط)
    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);
```

<div dir="rtl">

### 3. Indexes للبحث

</div>

```sql
-- Index عادي
CREATE INDEX idx_users_email ON users(email);

-- Index للـ LIKE searches
CREATE INDEX idx_users_name_pattern ON users(name varchar_pattern_ops);
-- هيحسن أداء: WHERE name LIKE 'Ahmed%'

-- Full-text search index
CREATE INDEX idx_articles_content ON articles USING GIN(to_tsvector('english', content));
```

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. استخدام CHAR بدل VARCHAR

</div>

```sql
-- ❌ وحش
CREATE TABLE users (
    username CHAR(50)  -- هيملأ الباقي بمسافات!
);

-- ✅ كويس
CREATE TABLE users (
    username VARCHAR(50)
);
```

<div dir="rtl">

### 2. VARCHAR بدون حد معقول

</div>

```sql
-- ❌ وحش
username VARCHAR(10000)  -- كتير أوي!

-- ✅ كويس
username VARCHAR(50)     -- حد منطقي
```

<div dir="rtl">

### 3. نسيان NULL handling

</div>

```sql
-- ⚠️ NULL || 'text' = NULL
SELECT first_name || ' ' || last_name FROM users;
-- لو last_name NULL، النتيجة كلها NULL!

-- ✅ استخدم COALESCE أو CONCAT
SELECT CONCAT(first_name, ' ', last_name) FROM users;
SELECT first_name || ' ' || COALESCE(last_name, '') FROM users;
```

---

## 📋 جدول مرجعي

| النوع | الوصف | الاستخدام |
|-------|-------|----------|
| `VARCHAR(n)` | نص متغير (max n) | معظم الاستخدامات ✅ |
| `CHAR(n)` | نص ثابت (exactly n) | أكواد ثابتة فقط |
| `TEXT` | نص غير محدود | محتوى طويل |

---

## ✅ Key Takeaways

<div dir="rtl">

1. **VARCHAR(n)** هو الأكثر استخداماً - اختار حد معقول
2. **TEXT** للنصوص الطويلة (مقالات، تعليقات)
3. **CHAR(n)** فقط للأكواد الثابتة الطول
4. PostgreSQL يدعم **Unicode والعربية** بالكامل
5. استخدم **ILIKE** للبحث case-insensitive
6. استخدم **TRIM** لإزالة المسافات الزائدة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [أنواع التاريخ والوقت](./04-datetime-types.md)**

</div>

---

<div align="center">

[⬅️ السابق: أنواع الأرقام](./02-numeric-types.md) | [🏠 العودة للـ Module](../README.md)

</div>
