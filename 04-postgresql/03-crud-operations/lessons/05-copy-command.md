# COPY Command - استيراد وتصدير البيانات 📁

<div dir="rtl">

## مقدمة

`COPY` هو أسرع طريقة لاستيراد وتصدير كميات كبيرة من البيانات في PostgreSQL - أسرع بكتير من INSERT العادي.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 مقارنة الأداء

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Performance Comparison                          │
│                      (1,000,000 rows)                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Single INSERTs:        █████████████████████████████  ~5-10 دقائق │
│                                                                      │
│  Bulk INSERT:           █████████                       ~1-2 دقيقة  │
│                                                                      │
│  COPY:                  █                               ~10-20 ثانية│
│                                                                      │
│  ⚡ COPY أسرع 10-50x من INSERT!                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📥 COPY FROM - استيراد البيانات

<div dir="rtl">

### من ملف CSV

</div>

```sql
-- استيراد من ملف CSV
COPY users (username, email, full_name)
FROM '/path/to/users.csv'
WITH (FORMAT CSV, HEADER true);

-- مع خيارات إضافية
COPY products (sku, name, price, stock)
FROM '/tmp/products.csv'
WITH (
    FORMAT CSV,
    HEADER true,
    DELIMITER ',',
    QUOTE '"',
    ESCAPE '\',
    NULL 'NULL'
);
```

<div dir="rtl">

### خيارات COPY FROM

</div>

```sql
-- FORMAT: csv, text, binary
COPY table FROM '/file.csv' WITH (FORMAT CSV);
COPY table FROM '/file.txt' WITH (FORMAT TEXT);
COPY table FROM '/file.bin' WITH (FORMAT BINARY);

-- DELIMITER: الفاصل بين القيم
COPY table FROM '/file.csv' WITH (FORMAT CSV, DELIMITER ',');   -- comma
COPY table FROM '/file.tsv' WITH (FORMAT CSV, DELIMITER E'\t'); -- tab
COPY table FROM '/file.psv' WITH (FORMAT CSV, DELIMITER '|');   -- pipe

-- HEADER: هل السطر الأول headers؟
COPY table FROM '/file.csv' WITH (FORMAT CSV, HEADER true);

-- NULL: كيف تُمثل NULL؟
COPY table FROM '/file.csv' WITH (FORMAT CSV, NULL '');
COPY table FROM '/file.csv' WITH (FORMAT CSV, NULL 'NULL');
COPY table FROM '/file.csv' WITH (FORMAT CSV, NULL '\N');

-- QUOTE: حرف الاقتباس
COPY table FROM '/file.csv' WITH (FORMAT CSV, QUOTE '"');

-- ENCODING: ترميز الملف
COPY table FROM '/file.csv' WITH (FORMAT CSV, ENCODING 'UTF8');
```

<div dir="rtl">

### من stdin (داخل psql)

</div>

```sql
-- COPY من stdin
COPY users (username, email) FROM stdin;
ahmed	ahmed@example.com
sara	sara@example.com
omar	omar@example.com
\.

-- أو مع CSV format
COPY users (username, email) FROM stdin WITH (FORMAT CSV);
ahmed,ahmed@example.com
sara,sara@example.com
omar,omar@example.com
\.
```

---

## 📤 COPY TO - تصدير البيانات

<div dir="rtl">

### لملف

</div>

```sql
-- تصدير لملف CSV
COPY users TO '/tmp/users_export.csv'
WITH (FORMAT CSV, HEADER true);

-- تصدير أعمدة محددة
COPY users (id, username, email)
TO '/tmp/users_basic.csv'
WITH (FORMAT CSV, HEADER true);

-- تصدير نتيجة query
COPY (
    SELECT u.username, u.email, COUNT(o.id) AS order_count
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id, u.username, u.email
) TO '/tmp/user_orders.csv'
WITH (FORMAT CSV, HEADER true);
```

<div dir="rtl">

### لـ stdout (داخل psql)

</div>

```sql
-- تصدير للشاشة
COPY users TO stdout WITH (FORMAT CSV, HEADER true);

-- تصدير query معين
COPY (SELECT * FROM users WHERE is_active = TRUE)
TO stdout WITH (FORMAT CSV);
```

---

## 🔧 \copy في psql Client

<div dir="rtl">

### الفرق بين COPY و \copy

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                      COPY vs \copy                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COPY (Server-side):                                                │
│  ├── يتنفذ على السيرفر                                              │
│  ├── الملف لازم يكون على السيرفر                                    │
│  ├── يحتاج SUPERUSER أو pg_read_server_files/pg_write_server_files  │
│  └── أسرع للملفات الكبيرة                                          │
│                                                                      │
│  \copy (Client-side):                                               │
│  ├── يتنفذ من الـ psql client                                       │
│  ├── الملف على جهازك المحلي                                         │
│  ├── مش محتاج صلاحيات خاصة                                          │
│  └── أبطأ قليلاً (البيانات بتعدي عبر الـ connection)                │
└─────────────────────────────────────────────────────────────────────┘
```

```bash
# من psql client
psql -d mydb

# استيراد من ملف محلي
\copy users (username, email) FROM './users.csv' WITH (FORMAT CSV, HEADER true)

# تصدير لملف محلي
\copy users TO './users_backup.csv' WITH (FORMAT CSV, HEADER true)

# تصدير query
\copy (SELECT * FROM users WHERE is_active) TO './active_users.csv' CSV HEADER
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. استيراد بيانات من Excel

</div>

```sql
-- 1. احفظ Excel كـ CSV أولاً
-- 2. استورد

-- products.csv:
-- sku,name,price,stock,category
-- "SKU001","Laptop",999.99,50,"Electronics"
-- "SKU002","Phone",499.99,100,"Electronics"

COPY products (sku, name, price, stock, category_name)
FROM '/tmp/products.csv'
WITH (FORMAT CSV, HEADER true, ENCODING 'UTF8');
```

<div dir="rtl">

### 2. Backup و Restore

</div>

```sql
-- Backup جدول كامل
COPY users TO '/backup/users_20241221.csv'
WITH (FORMAT CSV, HEADER true);

-- Restore
TRUNCATE users;  -- أو DELETE
COPY users FROM '/backup/users_20241221.csv'
WITH (FORMAT CSV, HEADER true);
```

<div dir="rtl">

### 3. تصدير تقرير

</div>

```sql
-- تقرير مبيعات شهري
COPY (
    SELECT
        TO_CHAR(o.created_at, 'YYYY-MM') AS month,
        u.full_name AS customer,
        COUNT(o.id) AS orders,
        SUM(o.total_amount) AS revenue
    FROM orders o
    JOIN users u ON o.user_id = u.id
    WHERE o.status = 'completed'
    GROUP BY TO_CHAR(o.created_at, 'YYYY-MM'), u.id, u.full_name
    ORDER BY month, revenue DESC
) TO '/tmp/monthly_sales_report.csv'
WITH (FORMAT CSV, HEADER true);
```

<div dir="rtl">

### 4. Data Migration

</div>

```sql
-- تصدير من database قديمة
-- في الـ old database:
COPY (
    SELECT
        customer_id AS id,
        TRIM(first_name || ' ' || last_name) AS full_name,
        LOWER(email) AS email,
        COALESCE(phone, '') AS phone
    FROM old_customers
) TO '/tmp/customers_migration.csv'
WITH (FORMAT CSV, HEADER true);

-- استيراد في database جديدة
COPY customers (id, full_name, email, phone)
FROM '/tmp/customers_migration.csv'
WITH (FORMAT CSV, HEADER true);
```

---

## ⚠️ التعامل مع الأخطاء

<div dir="rtl">

### مشاكل شائعة

</div>

```sql
-- 1. خطأ في عدد الأعمدة
-- ERROR: missing data for column "email"
-- الحل: تأكد من تطابق الأعمدة في CSV والجدول

-- 2. قيمة NULL غير صحيحة
-- ERROR: invalid input syntax
-- الحل: حدد كيف تُمثل NULL
COPY table FROM '/file.csv' WITH (FORMAT CSV, NULL '');

-- 3. encoding خاطئ
-- ERROR: invalid byte sequence for encoding "UTF8"
-- الحل: حدد الـ encoding الصحيح
COPY table FROM '/file.csv' WITH (FORMAT CSV, ENCODING 'LATIN1');

-- 4. duplicate key
-- ERROR: duplicate key value violates unique constraint
-- الحل: نظف البيانات قبل الاستيراد أو استخدم staging table
```

<div dir="rtl">

### استخدام Staging Table

</div>

```sql
-- 1. إنشاء جدول مؤقت
CREATE TEMP TABLE users_staging (LIKE users INCLUDING DEFAULTS);
ALTER TABLE users_staging DROP CONSTRAINT IF EXISTS users_staging_pkey;

-- 2. استيراد للجدول المؤقت (بدون constraints)
COPY users_staging FROM '/tmp/users.csv' WITH (FORMAT CSV, HEADER true);

-- 3. تنظيف البيانات
DELETE FROM users_staging WHERE email IS NULL;
DELETE FROM users_staging s
WHERE EXISTS (SELECT 1 FROM users u WHERE u.email = s.email);

-- 4. نقل للجدول الأصلي
INSERT INTO users
SELECT * FROM users_staging
ON CONFLICT (email) DO NOTHING;

-- 5. حذف المؤقت
DROP TABLE users_staging;
```

---

## 🚀 تحسين الأداء

<div dir="rtl">

### 1. تعطيل Triggers و Indexes

</div>

```sql
BEGIN;

-- تعطيل triggers
ALTER TABLE big_table DISABLE TRIGGER ALL;

-- استيراد البيانات
COPY big_table FROM '/tmp/big_data.csv' WITH (FORMAT CSV);

-- إعادة تفعيل triggers
ALTER TABLE big_table ENABLE TRIGGER ALL;

COMMIT;

-- إعادة بناء indexes
REINDEX TABLE big_table;

-- أو تحليل الجدول
ANALYZE big_table;
```

<div dir="rtl">

### 2. استخدام UNLOGGED Tables

</div>

```sql
-- للبيانات اللي مش محتاج recovery ليها
CREATE UNLOGGED TABLE temp_import (
    col1 TEXT,
    col2 TEXT,
    col3 TEXT
);

COPY temp_import FROM '/tmp/data.csv' WITH (FORMAT CSV);

-- معالجة البيانات ثم نقلها
INSERT INTO final_table
SELECT DISTINCT col1, col2::INT, col3::DATE
FROM temp_import;

DROP TABLE temp_import;
```

<div dir="rtl">

### 3. Parallel Import

</div>

```bash
# تقسيم الملف الكبير
split -l 100000 huge_file.csv chunk_

# استيراد بالتوازي (في terminals مختلفة)
psql -c "\copy table FROM 'chunk_aa' CSV" &
psql -c "\copy table FROM 'chunk_ab' CSV" &
psql -c "\copy table FROM 'chunk_ac' CSV" &
wait
```

---

## 🔒 الأمان

```sql
-- COPY يحتاج صلاحيات خاصة
-- للقراءة من ملفات السيرفر:
GRANT pg_read_server_files TO etl_user;

-- للكتابة لملفات السيرفر:
GRANT pg_write_server_files TO etl_user;

-- أو استخدم \copy من psql (مش محتاج صلاحيات خاصة)
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم \copy للملفات المحلية

</div>

```bash
# أسهل ومش محتاج صلاحيات superuser
\copy users FROM './users.csv' CSV HEADER
```

<div dir="rtl">

### 2. استخدم Staging Table للبيانات غير النظيفة

</div>

```sql
-- استورد لجدول مؤقت، نظف، ثم انقل
CREATE TEMP TABLE staging (...);
COPY staging FROM '...';
-- تنظيف
INSERT INTO production SELECT * FROM staging;
```

<div dir="rtl">

### 3. حدد الأعمدة صراحةً

</div>

```sql
-- ✅ صح: أعمدة محددة
COPY users (username, email, full_name) FROM '/file.csv' CSV HEADER;

-- ❌ غلط: بدون تحديد (حساس لتغيير ترتيب الأعمدة)
COPY users FROM '/file.csv' CSV HEADER;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **COPY** أسرع طريقة لاستيراد/تصدير البيانات
2. **\copy** للملفات المحلية، **COPY** للملفات على السيرفر
3. استخدم **WITH (FORMAT CSV, HEADER true)** للـ CSV
4. **Staging Table** للتعامل مع البيانات غير النظيفة
5. **تعطيل Triggers/Indexes** لتحسين الأداء

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [SELECT المتقدم](./06-advanced-select.md)**

</div>

---

<div align="center">

[⬅️ السابق: INSERT SELECT](./04-insert-select.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./06-advanced-select.md)

</div>
