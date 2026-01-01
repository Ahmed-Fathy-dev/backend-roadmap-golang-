# DELETE الأساسي - حذف البيانات 🗑️

<div dir="rtl">

## مقدمة

`DELETE` بيحذف صفوف من الجدول. عملية خطيرة ومش قابلة للتراجع (بدون backup أو transaction)، فلازم نكون حذرين.

**المدة المتوقعة:** 15 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
DELETE FROM table_name
WHERE condition;
```

---

## ⚠️ تحذير مهم جداً!

```
┌─────────────────────────────────────────────────────────────────────┐
│  ⚠️⚠️⚠️  DELETE بدون WHERE بيحذف كل البيانات!  ⚠️⚠️⚠️              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ خطر جداً! هيحذف كل الصفوف:                                      │
│  DELETE FROM products;                                              │
│                                                                      │
│  ✅ آمن: هيحذف صف واحد:                                             │
│  DELETE FROM products WHERE id = 5;                                 │
│                                                                      │
│  💡 خطوات آمنة:                                                     │
│  1. SELECT * FROM table WHERE condition;  -- شوف إيه هيتحذف        │
│  2. BEGIN;                                -- ابدأ transaction        │
│  3. DELETE FROM table WHERE condition;   -- احذف                    │
│  4. SELECT COUNT(*) FROM table;          -- تحقق                    │
│  5. COMMIT; أو ROLLBACK;                 -- أكد أو ارجع            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 DELETE صف واحد

```sql
-- حذف بالـ ID
DELETE FROM users WHERE id = 5;

-- حذف بشرط unique
DELETE FROM users WHERE email = 'test@example.com';

-- حذف منتج
DELETE FROM products WHERE sku = 'DELETED_SKU';
```

---

## 📊 DELETE عدة صفوف

```sql
-- حذف كل الطلبات الملغية
DELETE FROM orders WHERE status = 'cancelled';

-- حذف البيانات القديمة
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days';

-- حذف المستخدمين غير النشطين
DELETE FROM users
WHERE is_active = FALSE
  AND last_login < NOW() - INTERVAL '1 year';

-- حذف بعدة شروط
DELETE FROM products
WHERE stock = 0
  AND is_available = FALSE
  AND created_at < '2023-01-01';
```

---

## 🔙 DELETE مع RETURNING

```sql
-- إرجاع الصفوف المحذوفة
DELETE FROM users
WHERE is_active = FALSE
RETURNING id, username, email;

-- حفظ المحذوف في جدول آخر
WITH deleted AS (
    DELETE FROM orders
    WHERE status = 'cancelled'
      AND created_at < NOW() - INTERVAL '90 days'
    RETURNING *
)
INSERT INTO deleted_orders_archive
SELECT *, NOW() AS deleted_at
FROM deleted;

-- عدد الصفوف المحذوفة
DELETE FROM logs
WHERE created_at < NOW() - INTERVAL '7 days'
RETURNING COUNT(*);
```

---

## 🔗 DELETE مع Subquery

```sql
-- حذف باستخدام IN
DELETE FROM order_items
WHERE order_id IN (
    SELECT id FROM orders WHERE status = 'cancelled'
);

-- حذف باستخدام EXISTS
DELETE FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
)
AND u.created_at < NOW() - INTERVAL '1 year';

-- حذف باستخدام subquery
DELETE FROM products
WHERE category_id IN (
    SELECT id FROM categories WHERE is_active = FALSE
);
```

---

## 🔗 DELETE مع USING (JOIN)

```sql
-- PostgreSQL syntax: DELETE USING
DELETE FROM order_items oi
USING orders o
WHERE oi.order_id = o.id
  AND o.status = 'cancelled';

-- مثال آخر
DELETE FROM products p
USING categories c
WHERE p.category_id = c.id
  AND c.name = 'Discontinued';

-- حذف مع join معقد
DELETE FROM reviews r
USING users u, products p
WHERE r.user_id = u.id
  AND r.product_id = p.id
  AND u.is_banned = TRUE;
```

---

## 🔄 DELETE مع CTE

```sql
-- استخدام CTE لتحديد الصفوف
WITH old_orders AS (
    SELECT id
    FROM orders
    WHERE created_at < NOW() - INTERVAL '2 years'
      AND status IN ('completed', 'cancelled')
)
DELETE FROM orders
WHERE id IN (SELECT id FROM old_orders)
RETURNING id;

-- CTE معقد مع archiving
WITH
orders_to_delete AS (
    SELECT id, user_id, total_amount, created_at
    FROM orders
    WHERE status = 'cancelled'
      AND created_at < NOW() - INTERVAL '90 days'
),
archived AS (
    INSERT INTO orders_archive
    SELECT *, NOW() AS archived_at
    FROM orders_to_delete
    RETURNING id
)
DELETE FROM orders
WHERE id IN (SELECT id FROM archived);
```

---

## 📊 Cascade Delete

<div dir="rtl">

### ON DELETE CASCADE

</div>

```sql
-- عند إنشاء الجدول
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id),
    quantity INT
);

-- الآن لما تحذف order، الـ items بتتحذف تلقائياً
DELETE FROM orders WHERE id = 1;
-- كل الـ order_items اللي order_id = 1 اتحذفوا تلقائياً
```

<div dir="rtl">

### ON DELETE SET NULL

</div>

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE SET NULL,
    title TEXT
);

-- لما تحذف user، الـ posts بتفضل بس user_id = NULL
DELETE FROM users WHERE id = 1;
-- posts.user_id هيبقى NULL
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. تنظيف Logs

</div>

```sql
-- حذف logs قديمة
DELETE FROM application_logs
WHERE created_at < NOW() - INTERVAL '30 days'
  AND level NOT IN ('ERROR', 'CRITICAL');

-- حذف batch by batch (للجداول الكبيرة)
DO $$
DECLARE
    deleted_count INT;
BEGIN
    LOOP
        DELETE FROM application_logs
        WHERE id IN (
            SELECT id FROM application_logs
            WHERE created_at < NOW() - INTERVAL '30 days'
            LIMIT 10000
        );

        GET DIAGNOSTICS deleted_count = ROW_COUNT;
        EXIT WHEN deleted_count = 0;

        COMMIT;
    END LOOP;
END $$;
```

<div dir="rtl">

### 2. حذف مع Archive

</div>

```sql
-- نقل للـ archive ثم حذف
WITH to_archive AS (
    DELETE FROM users
    WHERE is_active = FALSE
      AND last_login < NOW() - INTERVAL '2 years'
    RETURNING *
)
INSERT INTO users_archive (
    original_id, username, email, created_at, deleted_at
)
SELECT id, username, email, created_at, NOW()
FROM to_archive;
```

<div dir="rtl">

### 3. Cleanup Orphaned Records

</div>

```sql
-- حذف order_items بدون orders
DELETE FROM order_items
WHERE order_id NOT IN (SELECT id FROM orders);

-- أو باستخدام NOT EXISTS (أسرع)
DELETE FROM order_items oi
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.id = oi.order_id
);

-- حذف reviews لمنتجات محذوفة
DELETE FROM reviews r
WHERE NOT EXISTS (
    SELECT 1 FROM products p WHERE p.id = r.product_id
);
```

---

## ⚠️ تجنب الأخطاء

<div dir="rtl">

### 1. نسيان WHERE

</div>

```sql
-- ❌ كارثة!
DELETE FROM users;  -- هيحذف كل المستخدمين!

-- ✅ آمن
DELETE FROM users WHERE id = 5;
```

<div dir="rtl">

### 2. Foreign Key Violations

</div>

```sql
-- ❌ هيفشل لو فيه orders مرتبطة
DELETE FROM users WHERE id = 1;
-- ERROR: update or delete violates foreign key constraint

-- ✅ احذف الـ dependent records أولاً
DELETE FROM orders WHERE user_id = 1;
DELETE FROM users WHERE id = 1;

-- أو استخدم CASCADE في الـ schema
```

<div dir="rtl">

### 3. حذف بدون تأكد

</div>

```sql
-- ❌ حذف مباشر
DELETE FROM important_data WHERE condition;

-- ✅ تحقق أولاً
BEGIN;
SELECT COUNT(*) FROM important_data WHERE condition;  -- شوف كام صف
DELETE FROM important_data WHERE condition;
-- لو كويس:
COMMIT;
-- لو غلط:
-- ROLLBACK;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. دايماً SELECT قبل DELETE

</div>

```sql
-- شوف إيه هيتحذف
SELECT * FROM users WHERE condition LIMIT 10;
SELECT COUNT(*) FROM users WHERE condition;

-- بعدين احذف
DELETE FROM users WHERE condition;
```

<div dir="rtl">

### 2. استخدم Transactions

</div>

```sql
BEGIN;

DELETE FROM order_items WHERE order_id = 5;
DELETE FROM orders WHERE id = 5;

-- تحقق
SELECT COUNT(*) FROM orders WHERE id = 5;

COMMIT;  -- أو ROLLBACK لو فيه مشكلة
```

<div dir="rtl">

### 3. استخدم RETURNING للتوثيق

</div>

```sql
-- سجل المحذوف
INSERT INTO deletion_log (table_name, deleted_data, deleted_at)
SELECT 'users', row_to_json(d), NOW()
FROM (
    DELETE FROM users
    WHERE is_active = FALSE
    RETURNING *
) d;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **دايماً WHERE** - DELETE بدونها كارثة
2. **SELECT قبل DELETE** - تأكد من الشرط
3. **Transaction** - للأمان والتراجع
4. **RETURNING** - لمعرفة المحذوف
5. **CASCADE** - للـ foreign keys

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [TRUNCATE و Soft Delete](./11-truncate-soft-delete.md)**

</div>

---

<div align="center">

[⬅️ السابق: UPDATE المتقدم](./09-advanced-update.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./11-truncate-soft-delete.md)

</div>
