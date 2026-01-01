# UPDATE الأساسي - تحديث البيانات ✏️

<div dir="rtl">

## مقدمة

`UPDATE` بيسمحلك تعدّل البيانات الموجودة في الجدول. في الدرس ده هنتعلم كل طرق التحديث من البسيطة للمتقدمة.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📝 الصيغة الأساسية

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition;
```

---

## ⚠️ تحذير مهم!

```
┌─────────────────────────────────────────────────────────────────────┐
│  ⚠️  دايماً استخدم WHERE مع UPDATE!                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ خطر! هيحدّث كل الصفوف:                                          │
│  UPDATE products SET price = 99.99;                                 │
│                                                                      │
│  ✅ آمن: هيحدّث صف واحد:                                            │
│  UPDATE products SET price = 99.99 WHERE id = 5;                    │
│                                                                      │
│  💡 نصيحة: اعمل SELECT أولاً بنفس الـ WHERE                         │
│  SELECT * FROM products WHERE id = 5;  -- شوف إيه هيتحدث           │
│  UPDATE products SET price = 99.99 WHERE id = 5;                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 UPDATE صف واحد

```sql
-- تحديث عمود واحد
UPDATE users
SET email = 'newemail@example.com'
WHERE id = 1;

-- تحديث عدة أعمدة
UPDATE users
SET
    email = 'newemail@example.com',
    full_name = 'Ahmed Mohamed Ali',
    updated_at = NOW()
WHERE id = 1;

-- تحديث منتج
UPDATE products
SET
    price = 149.99,
    stock = 75,
    updated_at = NOW()
WHERE sku = 'LAPTOP001';
```

---

## 📊 UPDATE عدة صفوف

```sql
-- تحديث كل الموظفين في قسم معين
UPDATE employees
SET salary = salary * 1.10
WHERE department = 'Engineering';

-- تحديث حالة الطلبات القديمة
UPDATE orders
SET status = 'cancelled'
WHERE status = 'pending'
  AND created_at < NOW() - INTERVAL '7 days';

-- تحديث المنتجات منخفضة المخزون
UPDATE products
SET is_available = FALSE
WHERE stock = 0;
```

---

## 🧮 UPDATE مع Expressions

```sql
-- زيادة نسبية
UPDATE employees
SET salary = salary * 1.10
WHERE performance_rating >= 4;

-- زيادة ثابتة
UPDATE employees
SET salary = salary + 5000
WHERE department = 'Engineering';

-- تقليل
UPDATE products
SET price = price * 0.90  -- خصم 10%
WHERE category_id = 5;

-- عمليات على strings
UPDATE users
SET email = LOWER(email)
WHERE email != LOWER(email);

-- concatenation
UPDATE products
SET name = 'SALE: ' || name
WHERE is_on_sale = TRUE;
```

---

## 🔄 UPDATE مع CASE

```sql
-- تحديث شرطي
UPDATE employees
SET salary = CASE
    WHEN department = 'Engineering' THEN salary * 1.15
    WHEN department = 'Marketing' THEN salary * 1.10
    WHEN department = 'HR' THEN salary * 1.08
    ELSE salary * 1.05
END
WHERE is_active = TRUE;

-- تحديث status بناءً على شرط
UPDATE orders
SET status = CASE
    WHEN total_amount >= 1000 THEN 'priority'
    WHEN total_amount >= 500 THEN 'standard'
    ELSE 'economy'
END
WHERE status = 'pending';

-- تحديث مع عدة شروط
UPDATE products
SET
    price = CASE
        WHEN stock > 100 THEN price * 0.95
        WHEN stock > 50 THEN price * 1.00
        ELSE price * 1.05
    END,
    is_available = CASE
        WHEN stock > 0 THEN TRUE
        ELSE FALSE
    END;
```

---

## 🔙 UPDATE مع RETURNING

```sql
-- إرجاع الصفوف المحدّثة
UPDATE users
SET is_active = FALSE
WHERE last_login < NOW() - INTERVAL '90 days'
RETURNING id, username, email;

-- إرجاع القيم القديمة والجديدة
UPDATE products
SET price = price * 1.10
WHERE category_id = 1
RETURNING id, name, price AS new_price;

-- استخدام النتيجة في CTE
WITH updated AS (
    UPDATE orders
    SET status = 'processing'
    WHERE status = 'pending' AND total_amount > 500
    RETURNING id, user_id, total_amount
)
INSERT INTO order_logs (order_id, action, details)
SELECT id, 'status_change', 'Changed to processing'
FROM updated;
```

---

## 📅 تحديث Timestamps

```sql
-- تحديث updated_at يدوياً
UPDATE products
SET
    name = 'New Product Name',
    updated_at = NOW()
WHERE id = 1;

-- أو أوتوماتيكياً بـ Trigger
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON products
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

-- الآن updated_at هيتحدث تلقائياً
UPDATE products SET name = 'Updated Name' WHERE id = 1;
```

---

## 🔢 UPDATE مع NULL

```sql
-- تعيين قيمة NULL
UPDATE users
SET phone = NULL
WHERE id = 1;

-- تحديث فقط لو مش NULL
UPDATE users
SET email = 'backup@email.com'
WHERE email IS NULL;

-- استخدام COALESCE
UPDATE products
SET
    description = COALESCE(description, 'No description available')
WHERE description IS NULL;

-- NULLIF: تحويل قيمة لـ NULL
UPDATE products
SET discount = NULLIF(discount, 0)
WHERE discount = 0;
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. تحديث المخزون بعد البيع

</div>

```sql
-- تقليل المخزون
UPDATE products
SET
    stock = stock - 1,
    updated_at = NOW()
WHERE id = 5 AND stock > 0
RETURNING id, name, stock;

-- تقليل بكمية معينة
UPDATE products
SET stock = stock - 3
WHERE sku = 'PHONE001'
  AND stock >= 3;  -- تأكد إن المخزون كافي
```

<div dir="rtl">

### 2. تحديث حالة الطلبات

</div>

```sql
-- تحديث الطلب لـ shipped
UPDATE orders
SET
    status = 'shipped',
    shipped_at = NOW(),
    updated_at = NOW()
WHERE id = 123 AND status = 'processing'
RETURNING *;

-- تحديث bulk للطلبات الجاهزة
UPDATE orders
SET
    status = 'processing',
    updated_at = NOW()
WHERE status = 'pending'
  AND payment_status = 'paid'
  AND created_at < NOW() - INTERVAL '1 hour'
RETURNING id;
```

<div dir="rtl">

### 3. تحديث معلومات المستخدم

</div>

```sql
-- تحديث Profile
UPDATE users
SET
    full_name = 'Ahmed Mohamed Ali',
    email = 'ahmed.new@example.com',
    phone = '+201234567890',
    updated_at = NOW()
WHERE id = 1
  AND is_active = TRUE
RETURNING id, username, email, updated_at;

-- تحديث كلمة السر
UPDATE users
SET
    password_hash = 'new_hashed_password',
    password_changed_at = NOW(),
    updated_at = NOW()
WHERE id = 1;

-- تحديث last_login
UPDATE users
SET last_login = NOW()
WHERE id = 1;
```

---

## ⚠️ تجنب الأخطاء الشائعة

<div dir="rtl">

### 1. نسيان WHERE

</div>

```sql
-- ❌ خطر! هيحدث كل الصفوف
UPDATE products SET price = 0;

-- ✅ آمن
UPDATE products SET price = 0 WHERE id = 5;
```

<div dir="rtl">

### 2. Race Conditions

</div>

```sql
-- ❌ Race condition ممكن
-- Session 1: stock = 10
UPDATE products SET stock = 9 WHERE id = 5;
-- Session 2: stock = 10 (قبل ما Session 1 يخلص)
UPDATE products SET stock = 9 WHERE id = 5;
-- النتيجة: stock = 9، بس المفروض يكون 8!

-- ✅ آمن: استخدم الـ expression
UPDATE products SET stock = stock - 1 WHERE id = 5;
-- كل session هيقرأ القيمة الحالية
```

<div dir="rtl">

### 3. التحديث بدون تحقق

</div>

```sql
-- ❌ مش بيتحقق من المخزون
UPDATE products SET stock = stock - 5 WHERE id = 5;
-- ممكن يخلي المخزون سالب!

-- ✅ مع تحقق
UPDATE products
SET stock = stock - 5
WHERE id = 5 AND stock >= 5;
-- لو المخزون أقل من 5، مش هيحصل تحديث
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. دايماً اعمل SELECT أولاً

</div>

```sql
-- 1. شوف إيه هيتحدث
SELECT * FROM products WHERE category_id = 5;

-- 2. لو الناتج صح، نفذ الـ UPDATE
UPDATE products SET price = price * 0.9 WHERE category_id = 5;
```

<div dir="rtl">

### 2. استخدم Transaction للعمليات المهمة

</div>

```sql
BEGIN;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- لو كل حاجة تمام
COMMIT;
-- لو فيه مشكلة
-- ROLLBACK;
```

<div dir="rtl">

### 3. استخدم RETURNING للتحقق

</div>

```sql
-- ✅ تأكد إن التحديث تم
UPDATE products
SET price = 99.99
WHERE id = 5
RETURNING *;

-- لو مرجعش حاجة، يبقى الـ WHERE مش صح
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **دايماً استخدم WHERE** - تجنب تحديث كل الصفوف
2. **اعمل SELECT الأول** - تأكد من الـ WHERE
3. **استخدم expressions** - `stock = stock - 1` أأمن
4. **RETURNING** للتحقق من التحديث
5. **Transactions** للعمليات المرتبطة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [UPDATE المتقدم](./09-advanced-update.md)**

</div>

---

<div align="center">

[⬅️ السابق: Window Functions](./07-window-functions.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./09-advanced-update.md)

</div>
