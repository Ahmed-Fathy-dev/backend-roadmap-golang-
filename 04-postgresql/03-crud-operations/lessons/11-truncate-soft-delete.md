# TRUNCATE و Soft Delete 🗑️

<div dir="rtl">

## مقدمة

في الدرس ده هنتعلم TRUNCATE لحذف كل البيانات بسرعة، و Soft Delete كبديل آمن للحذف النهائي.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## ⚡ TRUNCATE

<div dir="rtl">

### ما هو TRUNCATE؟

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DELETE vs TRUNCATE                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DELETE FROM table;                                                 │
│  ├── يحذف صف صف                                                     │
│  ├── يسجل كل حذف في الـ WAL                                         │
│  ├── Triggers بتشتغل                                                │
│  ├── ممكن تعمل ROLLBACK                                             │
│  ├── بطيء للجداول الكبيرة                                           │
│  └── يحافظ على الـ sequence value                                   │
│                                                                      │
│  TRUNCATE TABLE table;                                              │
│  ├── يحذف كل البيانات مرة واحدة                                     │
│  ├── مش بيسجل كل صف (أسرع)                                         │
│  ├── Triggers مش بتشتغل (إلا لو TRUNCATE trigger)                  │
│  ├── ممكن تعمل ROLLBACK (في PostgreSQL)                            │
│  ├── سريع جداً                                                      │
│  └── بيعيد الـ sequence للـ 1 (لو RESTART IDENTITY)                 │
└─────────────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### الصيغة

</div>

```sql
-- TRUNCATE بسيط
TRUNCATE TABLE logs;

-- TRUNCATE مع إعادة الـ sequence
TRUNCATE TABLE users RESTART IDENTITY;

-- TRUNCATE عدة جداول
TRUNCATE TABLE orders, order_items, payments;

-- TRUNCATE مع CASCADE (للجداول المرتبطة)
TRUNCATE TABLE users CASCADE;
-- هيحذف من كل الجداول اللي ليها foreign key على users
```

<div dir="rtl">

### أمثلة استخدام

</div>

```sql
-- تنظيف جدول test
TRUNCATE TABLE test_data RESTART IDENTITY;

-- تنظيف جداول staging
TRUNCATE TABLE
    staging_customers,
    staging_orders,
    staging_products
RESTART IDENTITY;

-- TRUNCATE في transaction
BEGIN;
TRUNCATE TABLE temp_imports;
-- لو حصل مشكلة
ROLLBACK;
-- أو
COMMIT;
```

---

## 🔄 Soft Delete

<div dir="rtl">

### ما هو Soft Delete؟

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Hard Delete vs Soft Delete                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Hard Delete:                                                       │
│  ┌─────────────────────────────────────┐                            │
│  │  DELETE FROM users WHERE id = 5;    │                            │
│  │                                      │                            │
│  │  ❌ البيانات راحت للأبد              │                            │
│  │  ❌ مش ممكن نرجعها                   │                            │
│  │  ❌ مفيش audit trail                 │                            │
│  └─────────────────────────────────────┘                            │
│                                                                      │
│  Soft Delete:                                                       │
│  ┌─────────────────────────────────────┐                            │
│  │  UPDATE users                        │                            │
│  │  SET deleted_at = NOW()             │                            │
│  │  WHERE id = 5;                      │                            │
│  │                                      │                            │
│  │  ✅ البيانات موجودة                  │                            │
│  │  ✅ ممكن نرجعها                      │                            │
│  │  ✅ فيه audit trail                  │                            │
│  └─────────────────────────────────────┘                            │
└─────────────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### تطبيق Soft Delete

</div>

```sql
-- 1. إضافة عمود deleted_at
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;

-- 2. إنشاء index على deleted_at
CREATE INDEX idx_users_deleted_at ON users(deleted_at)
WHERE deleted_at IS NULL;

-- 3. Soft delete
UPDATE users
SET deleted_at = NOW()
WHERE id = 5;

-- 4. Query للبيانات النشطة فقط
SELECT * FROM users WHERE deleted_at IS NULL;

-- 5. استعادة البيانات
UPDATE users
SET deleted_at = NULL
WHERE id = 5;
```

---

## 🏗️ Soft Delete Schema

```sql
-- جدول مع soft delete
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    deleted_at TIMESTAMPTZ,  -- NULL = active, timestamp = deleted
    deleted_by INT           -- من حذفه (optional)
);

-- Unique constraint مع soft delete
-- نريد username فريد للمستخدمين النشطين فقط
CREATE UNIQUE INDEX idx_users_username_active
ON users (username)
WHERE deleted_at IS NULL;

-- نفس الـ email ممكن يتكرر للمحذوفين
CREATE UNIQUE INDEX idx_users_email_active
ON users (email)
WHERE deleted_at IS NULL;
```

---

## 📊 Views للـ Active Records

```sql
-- View للمستخدمين النشطين
CREATE VIEW active_users AS
SELECT id, username, email, created_at, updated_at
FROM users
WHERE deleted_at IS NULL;

-- View للمحذوفين
CREATE VIEW deleted_users AS
SELECT id, username, email, created_at, deleted_at, deleted_by
FROM users
WHERE deleted_at IS NOT NULL;

-- استخدام الـ View
SELECT * FROM active_users WHERE username LIKE 'a%';
```

---

## 🔄 Soft Delete Functions

```sql
-- Function للـ soft delete
CREATE OR REPLACE FUNCTION soft_delete_user(user_id INT, deleted_by_id INT)
RETURNS VOID AS $$
BEGIN
    UPDATE users
    SET
        deleted_at = NOW(),
        deleted_by = deleted_by_id,
        updated_at = NOW()
    WHERE id = user_id
      AND deleted_at IS NULL;
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT soft_delete_user(5, 1);  -- حذف user 5 بواسطة user 1

-- Function للاستعادة
CREATE OR REPLACE FUNCTION restore_user(user_id INT)
RETURNS VOID AS $$
BEGIN
    UPDATE users
    SET
        deleted_at = NULL,
        deleted_by = NULL,
        updated_at = NOW()
    WHERE id = user_id
      AND deleted_at IS NOT NULL;
END;
$$ LANGUAGE plpgsql;
```

---

## 🎯 Soft Delete مع Cascade

```sql
-- عند soft delete user، نريد soft delete لطلباته
CREATE OR REPLACE FUNCTION cascade_soft_delete_user(user_id INT)
RETURNS VOID AS $$
BEGIN
    -- Soft delete user
    UPDATE users
    SET deleted_at = NOW()
    WHERE id = user_id;

    -- Soft delete orders
    UPDATE orders
    SET deleted_at = NOW()
    WHERE user_id = user_id AND deleted_at IS NULL;

    -- Soft delete reviews
    UPDATE reviews
    SET deleted_at = NOW()
    WHERE user_id = user_id AND deleted_at IS NULL;
END;
$$ LANGUAGE plpgsql;
```

---

## 📊 Hard Delete للبيانات القديمة

```sql
-- حذف نهائي للبيانات المحذوفة soft أكتر من سنة
DELETE FROM users
WHERE deleted_at IS NOT NULL
  AND deleted_at < NOW() - INTERVAL '1 year';

-- Scheduled cleanup job
CREATE OR REPLACE FUNCTION cleanup_old_deleted_records()
RETURNS INTEGER AS $$
DECLARE
    total_deleted INTEGER := 0;
    deleted_count INTEGER;
BEGIN
    -- Users
    DELETE FROM users
    WHERE deleted_at < NOW() - INTERVAL '1 year';
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    total_deleted := total_deleted + deleted_count;

    -- Orders
    DELETE FROM orders
    WHERE deleted_at < NOW() - INTERVAL '1 year';
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    total_deleted := total_deleted + deleted_count;

    -- Products
    DELETE FROM products
    WHERE deleted_at < NOW() - INTERVAL '2 years';
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    total_deleted := total_deleted + deleted_count;

    RETURN total_deleted;
END;
$$ LANGUAGE plpgsql;
```

---

## 📊 مقارنة شاملة

```
┌────────────────────────────────────────────────────────────────────┐
│                  متى تستخدم كل طريقة؟                               │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  DELETE:                                                            │
│  ├── بيانات مؤقتة (temp tables, cache)                             │
│  ├── بيانات test                                                   │
│  ├── لما محتاج Triggers تشتغل                                      │
│  └── حذف selective (WHERE condition)                               │
│                                                                     │
│  TRUNCATE:                                                          │
│  ├── مسح جدول كامل                                                 │
│  ├── Reset جداول test/staging                                       │
│  ├── Performance critical (جداول كبيرة)                            │
│  └── إعادة الـ sequences                                            │
│                                                                     │
│  Soft Delete:                                                       │
│  ├── بيانات المستخدمين                                             │
│  ├── بيانات مالية/قانونية                                          │
│  ├── محتاج audit trail                                              │
│  ├── محتاج استعادة                                                 │
│  └── Compliance requirements (GDPR, etc.)                          │
└────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. Soft Delete للبيانات المهمة

</div>

```sql
-- ✅ للمستخدمين والطلبات
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;
ALTER TABLE orders ADD COLUMN deleted_at TIMESTAMPTZ;

-- للـ logs مش محتاج
-- DELETE أو TRUNCATE كفاية
```

<div dir="rtl">

### 2. Index على deleted_at

</div>

```sql
-- ✅ Partial index للأداء
CREATE INDEX idx_users_active
ON users (id)
WHERE deleted_at IS NULL;
```

<div dir="rtl">

### 3. Views للسهولة

</div>

```sql
-- ✅ بدل ما تكتب WHERE deleted_at IS NULL كل مرة
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;
```

<div dir="rtl">

### 4. Cleanup Policy

</div>

```sql
-- ✅ حدد متى تحذف نهائياً
-- بعد سنة من الـ soft delete
DELETE FROM users WHERE deleted_at < NOW() - INTERVAL '1 year';
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **TRUNCATE** أسرع من DELETE للجداول الكاملة
2. **Soft Delete** للبيانات اللي محتاج ترجعها
3. **deleted_at** عمود standard للـ soft delete
4. **Partial Index** للأداء مع soft delete
5. **Cleanup jobs** للحذف النهائي

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Transactions](./12-transactions.md)**

</div>

---

<div align="center">

[⬅️ السابق: DELETE الأساسي](./10-basic-delete.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./12-transactions.md)

</div>
