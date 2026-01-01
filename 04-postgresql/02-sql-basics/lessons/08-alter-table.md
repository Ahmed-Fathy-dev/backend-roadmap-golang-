# تعديل الجداول - ALTER TABLE 🔧

<div dir="rtl">

## مقدمة

بعد إنشاء الجدول، هتحتاج تعدّله - تضيف أعمدة، تغير أنواع، تضيف constraints. الأمر `ALTER TABLE` بيخليك تعمل كل ده بدون ما تحذف الجدول وتنشئه من جديد.

**المدة المتوقعة:** 20-25 دقيقة

</div>

---

## ➕ إضافة عمود (ADD COLUMN)

```sql
-- إضافة عمود بسيط
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- إضافة عمود مع NOT NULL و DEFAULT
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT TRUE;

-- إضافة عمود مع constraint
ALTER TABLE products ADD COLUMN discount NUMERIC(5,2) CHECK (discount >= 0 AND discount <= 100);

-- إضافة عدة أعمدة
ALTER TABLE users
    ADD COLUMN middle_name VARCHAR(50),
    ADD COLUMN suffix VARCHAR(10),
    ADD COLUMN nickname VARCHAR(50);
```

<div dir="rtl">

### إضافة عمود NOT NULL لجدول فيه بيانات

</div>

```sql
-- ❌ هيفشل لو فيه بيانات
ALTER TABLE users ADD COLUMN role VARCHAR(20) NOT NULL;
-- ERROR: column "role" contains null values

-- ✅ الطريقة الصحيحة
-- 1. أضف العمود مع default
ALTER TABLE users ADD COLUMN role VARCHAR(20) NOT NULL DEFAULT 'user';

-- أو
-- 1. أضف العمود بدون NOT NULL
ALTER TABLE users ADD COLUMN role VARCHAR(20);
-- 2. حدّث البيانات الموجودة
UPDATE users SET role = 'user' WHERE role IS NULL;
-- 3. أضف NOT NULL
ALTER TABLE users ALTER COLUMN role SET NOT NULL;
```

---

## ➖ حذف عمود (DROP COLUMN)

```sql
-- حذف عمود
ALTER TABLE users DROP COLUMN middle_name;

-- حذف لو موجود (تجنب الأخطاء)
ALTER TABLE users DROP COLUMN IF EXISTS middle_name;

-- حذف مع الـ dependencies
ALTER TABLE users DROP COLUMN created_by CASCADE;
-- هيحذف أي views أو constraints معتمدة على العمود

-- حذف عدة أعمدة
ALTER TABLE users
    DROP COLUMN suffix,
    DROP COLUMN nickname;
```

<div dir="rtl">

### ⚠️ تحذير

</div>

```sql
-- DROP COLUMN عملية خطيرة!
-- البيانات هتتحذف نهائياً

-- خد backup قبل ما تعمل DROP
pg_dump -U postgres -t users mydb > users_backup.sql

-- أو استخدم soft delete بدلاً من حذف العمود
-- (أضف deprecated_ prefix وخليه nullable)
ALTER TABLE users RENAME COLUMN old_field TO deprecated_old_field;
ALTER TABLE users ALTER COLUMN deprecated_old_field DROP NOT NULL;
```

---

## 🔄 تعديل عمود (ALTER COLUMN)

<div dir="rtl">

### تغيير نوع البيانات

</div>

```sql
-- تغيير نوع العمود
ALTER TABLE users ALTER COLUMN phone TYPE VARCHAR(30);

-- تغيير من VARCHAR لـ TEXT
ALTER TABLE products ALTER COLUMN description TYPE TEXT;

-- تغيير مع تحويل البيانات
ALTER TABLE users ALTER COLUMN age TYPE VARCHAR(10);
-- Integer هيتحول لـ string تلقائياً

-- تحويل معقد (محتاج USING)
ALTER TABLE users
ALTER COLUMN age TYPE INTEGER
USING age::INTEGER;

-- مثال: تحويل string لـ timestamp
ALTER TABLE events
ALTER COLUMN event_date TYPE TIMESTAMPTZ
USING event_date::TIMESTAMPTZ;
```

<div dir="rtl">

### تغيير DEFAULT

</div>

```sql
-- إضافة/تغيير default
ALTER TABLE users ALTER COLUMN is_active SET DEFAULT TRUE;
ALTER TABLE products ALTER COLUMN stock SET DEFAULT 0;
ALTER TABLE orders ALTER COLUMN status SET DEFAULT 'pending';

-- حذف default
ALTER TABLE users ALTER COLUMN is_active DROP DEFAULT;
```

<div dir="rtl">

### تغيير NOT NULL

</div>

```sql
-- إضافة NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;

-- حذف NOT NULL
ALTER TABLE users ALTER COLUMN phone DROP NOT NULL;
```

---

## 📝 إعادة التسمية (RENAME)

<div dir="rtl">

### إعادة تسمية عمود

</div>

```sql
-- تغيير اسم عمود
ALTER TABLE users RENAME COLUMN username TO user_name;

-- تغيير اسم عمود في علاقة
ALTER TABLE orders RENAME COLUMN user_id TO customer_id;
-- ⚠️ لازم تحدث أي Foreign Keys أو Code يستخدم الاسم القديم
```

<div dir="rtl">

### إعادة تسمية جدول

</div>

```sql
-- تغيير اسم الجدول
ALTER TABLE users RENAME TO app_users;

-- تغيير اسم مع Schema
ALTER TABLE public.users RENAME TO customers;
```

---

## 🔒 إدارة Constraints

<div dir="rtl">

### إضافة Constraints

</div>

```sql
-- إضافة PRIMARY KEY
ALTER TABLE logs ADD PRIMARY KEY (id);

-- إضافة UNIQUE
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);

-- إضافة FOREIGN KEY
ALTER TABLE orders ADD CONSTRAINT fk_orders_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE;

-- إضافة CHECK
ALTER TABLE products ADD CONSTRAINT check_price_positive
    CHECK (price > 0);

-- إضافة CHECK مع validation للبيانات الموجودة
ALTER TABLE products ADD CONSTRAINT check_stock_non_negative
    CHECK (stock >= 0)
    NOT VALID;  -- لا يتحقق من البيانات الموجودة

-- التحقق لاحقاً
ALTER TABLE products VALIDATE CONSTRAINT check_stock_non_negative;
```

<div dir="rtl">

### حذف Constraints

</div>

```sql
-- حذف constraint باسمه
ALTER TABLE users DROP CONSTRAINT unique_email;

-- حذف PRIMARY KEY
ALTER TABLE users DROP CONSTRAINT users_pkey;

-- حذف FOREIGN KEY
ALTER TABLE orders DROP CONSTRAINT fk_orders_user;
```

<div dir="rtl">

### إعادة تسمية Constraints

</div>

```sql
ALTER TABLE users RENAME CONSTRAINT unique_email TO users_email_unique;
```

---

## 📑 إدارة Indexes

```sql
-- إضافة Index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_created ON orders(created_at DESC);

-- حذف Index
DROP INDEX idx_users_email;

-- إعادة تسمية Index
ALTER INDEX idx_users_email RENAME TO users_email_idx;

-- Rebuild Index (لتحسين الأداء)
REINDEX INDEX idx_users_email;
REINDEX TABLE users;  -- كل الـ indexes للجدول
```

---

## 🔧 عمليات متقدمة

<div dir="rtl">

### تغيير Owner

</div>

```sql
-- تغيير مالك الجدول
ALTER TABLE users OWNER TO new_owner;
```

<div dir="rtl">

### تغيير Schema

</div>

```sql
-- نقل جدول لـ schema تاني
ALTER TABLE users SET SCHEMA archive;
```

<div dir="rtl">

### تغيير Tablespace

</div>

```sql
-- نقل جدول لـ tablespace تاني
ALTER TABLE users SET TABLESPACE fast_storage;
```

---

## 📋 أمثلة عملية

<div dir="rtl">

### مثال 1: تطوير جدول المستخدمين

</div>

```sql
-- الجدول الأصلي
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

-- التطويرات
-- 1. فصل الاسم
ALTER TABLE users ADD COLUMN first_name VARCHAR(50);
ALTER TABLE users ADD COLUMN last_name VARCHAR(50);
UPDATE users SET
    first_name = SPLIT_PART(name, ' ', 1),
    last_name = SPLIT_PART(name, ' ', 2);
ALTER TABLE users DROP COLUMN name;

-- 2. إضافة حقول الأمان
ALTER TABLE users ADD COLUMN password_hash VARCHAR(255);
ALTER TABLE users ADD COLUMN is_verified BOOLEAN DEFAULT FALSE;
ALTER TABLE users ADD COLUMN verification_token VARCHAR(100);

-- 3. إضافة timestamps
ALTER TABLE users ADD COLUMN created_at TIMESTAMPTZ DEFAULT NOW();
ALTER TABLE users ADD COLUMN updated_at TIMESTAMPTZ DEFAULT NOW();

-- 4. إضافة Constraints
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);
ALTER TABLE users ADD CONSTRAINT valid_email
    CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

<div dir="rtl">

### مثال 2: Migration Script

</div>

```sql
-- migration_001_add_user_roles.sql

-- Up migration
BEGIN;

-- إضافة جدول الأدوار
CREATE TABLE IF NOT EXISTS roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- إضافة أدوار افتراضية
INSERT INTO roles (name, description) VALUES
    ('admin', 'Full system access'),
    ('moderator', 'Content moderation access'),
    ('user', 'Standard user access')
ON CONFLICT (name) DO NOTHING;

-- إضافة العمود للمستخدمين
ALTER TABLE users ADD COLUMN IF NOT EXISTS role_id INT;

-- تعيين الـ default role
UPDATE users SET role_id = (SELECT id FROM roles WHERE name = 'user')
WHERE role_id IS NULL;

-- إضافة Foreign Key
ALTER TABLE users ADD CONSTRAINT fk_users_role
    FOREIGN KEY (role_id) REFERENCES roles(id)
    ON DELETE SET NULL;

COMMIT;

-- Down migration (للرجوع)
-- BEGIN;
-- ALTER TABLE users DROP CONSTRAINT IF EXISTS fk_users_role;
-- ALTER TABLE users DROP COLUMN IF EXISTS role_id;
-- DROP TABLE IF EXISTS roles;
-- COMMIT;
```

---

## ⚠️ Locking و Performance

<div dir="rtl">

### العمليات اللي بتعمل Lock

</div>

```sql
-- ⚠️ عمليات بتعمل EXCLUSIVE LOCK (بتوقف الـ reads والـ writes)
ALTER TABLE users ADD COLUMN new_col VARCHAR(100) NOT NULL DEFAULT 'value';
ALTER TABLE users ALTER COLUMN name TYPE TEXT;
ALTER TABLE users ADD CONSTRAINT ... CHECK (...);

-- ✅ عمليات سريعة (metadata change فقط)
ALTER TABLE users ADD COLUMN new_col VARCHAR(100);  -- بدون NOT NULL/DEFAULT
ALTER TABLE users DROP COLUMN old_col;
ALTER TABLE users RENAME COLUMN a TO b;
```

<div dir="rtl">

### تجنب الـ Locks الطويلة

</div>

```sql
-- ❌ هيعمل lock طويل على جدول كبير
ALTER TABLE huge_table ADD COLUMN status VARCHAR(20) NOT NULL DEFAULT 'active';

-- ✅ الطريقة الأفضل
-- 1. أضف العمود nullable
ALTER TABLE huge_table ADD COLUMN status VARCHAR(20);

-- 2. حدّث على batches
UPDATE huge_table SET status = 'active' WHERE id BETWEEN 1 AND 100000;
UPDATE huge_table SET status = 'active' WHERE id BETWEEN 100001 AND 200000;
-- ... continue in batches

-- 3. أضف NOT NULL والـ DEFAULT
ALTER TABLE huge_table ALTER COLUMN status SET NOT NULL;
ALTER TABLE huge_table ALTER COLUMN status SET DEFAULT 'active';
```

<div dir="rtl">

### استخدام CONCURRENTLY

</div>

```sql
-- Index عادي (يعمل lock)
CREATE INDEX idx_users_email ON users(email);

-- Index بدون lock (أبطأ بس ما يوقفش الـ app)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- ⚠️ CONCURRENTLY مش بيشتغل داخل transaction
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. خد Backup قبل أي تغيير

</div>

```bash
# Backup الجدول
pg_dump -U postgres -t users mydb > users_backup.sql

# أو Backup كامل
pg_dump -U postgres mydb > full_backup.sql
```

<div dir="rtl">

### 2. اختبر على Copy

</div>

```sql
-- أنشئ نسخة للاختبار
CREATE TABLE users_test AS SELECT * FROM users LIMIT 1000;

-- جرب التغييرات
ALTER TABLE users_test ADD COLUMN new_field VARCHAR(100);

-- لو نجح، نفذ على الأصلي
ALTER TABLE users ADD COLUMN new_field VARCHAR(100);

-- امسح النسخة
DROP TABLE users_test;
```

<div dir="rtl">

### 3. استخدم Transactions

</div>

```sql
BEGIN;

ALTER TABLE users ADD COLUMN role VARCHAR(20);
UPDATE users SET role = 'user';
ALTER TABLE users ALTER COLUMN role SET NOT NULL;

-- لو حصل خطأ
-- ROLLBACK;

-- لو كل حاجة تمام
COMMIT;
```

<div dir="rtl">

### 4. خطط للـ Downtime

</div>

```sql
-- للتغييرات الكبيرة:
-- 1. أعلن عن maintenance window
-- 2. أوقف الـ application
-- 3. نفذ التغييرات
-- 4. اختبر
-- 5. شغّل الـ application
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **ADD COLUMN** سهل، بس NOT NULL محتاج معالجة خاصة
2. **DROP COLUMN** خطير - خد backup أولاً
3. **ALTER COLUMN TYPE** ممكن يحتاج USING للتحويل
4. **Constraints** ممكن تتضاف بـ NOT VALID للسرعة
5. استخدم **CONCURRENTLY** للـ indexes في Production
6. دايماً **اختبر قبل** ما تنفذ في Production

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [INSERT - إدخال البيانات](./09-insert.md)**

</div>

---

<div align="center">

[⬅️ السابق: القيود (Constraints)](./07-constraints.md) | [🏠 العودة للـ Module](../README.md)

</div>
