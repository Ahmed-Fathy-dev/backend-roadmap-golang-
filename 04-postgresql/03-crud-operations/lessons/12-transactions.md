# Transactions - المعاملات 🔄

<div dir="rtl">

## مقدمة

Transactions بتضمن إن مجموعة عمليات تنجح كلها أو تفشل كلها - ده أساس سلامة البيانات في أي تطبيق.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 ما هي Transaction؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Transaction                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  مجموعة عمليات تُعامل كوحدة واحدة:                                  │
│  - إما تنجح كلها (COMMIT)                                           │
│  - أو تفشل كلها (ROLLBACK)                                          │
│                                                                      │
│  مثال: تحويل فلوس                                                   │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  BEGIN;                                                       │   │
│  │                                                               │   │
│  │  1. UPDATE accounts SET balance = balance - 1000             │   │
│  │     WHERE id = 1;  -- خصم من أحمد                            │   │
│  │                                                               │   │
│  │  2. UPDATE accounts SET balance = balance + 1000             │   │
│  │     WHERE id = 2;  -- إضافة لسارة                            │   │
│  │                                                               │   │
│  │  COMMIT;  -- الاثنين ينفذوا معاً أو يترجعوا معاً              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  لو أي خطوة فشلت:                                                   │
│  - الـ ROLLBACK يرجع كل التغييرات                                   │
│  - أحمد فلوسه ترجع، سارة متاخدش حاجة                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 ACID Properties

```
┌─────────────────────────────────────────────────────────────────────┐
│                          ACID                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  A - Atomicity (الذرية)                                             │
│      كل العمليات تنجح أو تفشل معاً                                  │
│      "All or Nothing"                                               │
│                                                                      │
│  C - Consistency (الاتساق)                                          │
│      الـ database تبقى في حالة valid قبل وبعد                       │
│      Constraints تتحقق دايماً                                       │
│                                                                      │
│  I - Isolation (العزل)                                              │
│      كل transaction معزولة عن غيرها                                 │
│      مش بتشوف تغييرات الـ transactions التانية                      │
│                                                                      │
│  D - Durability (الديمومة)                                          │
│      بعد COMMIT، البيانات محفوظة حتى لو السيرفر وقع                  │
│      مكتوبة على الـ disk                                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة الأساسية

```sql
-- بدء transaction
BEGIN;
-- أو
BEGIN TRANSACTION;
-- أو
START TRANSACTION;

-- تنفيذ العمليات
INSERT INTO ...;
UPDATE ...;
DELETE ...;

-- تأكيد التغييرات
COMMIT;

-- أو إلغاء كل التغييرات
ROLLBACK;
```

---

## 💰 مثال: تحويل الأموال

```sql
-- جدول الحسابات
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    owner_name VARCHAR(100),
    balance NUMERIC(15, 2) NOT NULL DEFAULT 0,
    CONSTRAINT positive_balance CHECK (balance >= 0)
);

-- بيانات
INSERT INTO accounts (owner_name, balance) VALUES
    ('Ahmed', 5000.00),
    ('Sara', 3000.00);

-- تحويل 1000 من Ahmed لـ Sara
BEGIN;

-- خطوة 1: خصم من أحمد
UPDATE accounts
SET balance = balance - 1000
WHERE owner_name = 'Ahmed';

-- خطوة 2: إضافة لسارة
UPDATE accounts
SET balance = balance + 1000
WHERE owner_name = 'Sara';

-- تحقق من النتيجة
SELECT * FROM accounts;

-- لو كل حاجة تمام
COMMIT;

-- لو فيه مشكلة
-- ROLLBACK;
```

---

## 🛡️ SAVEPOINT

<div dir="rtl">

### نقاط حفظ داخل Transaction

</div>

```sql
BEGIN;

INSERT INTO users (username, email) VALUES ('user1', 'user1@test.com');

-- حفظ نقطة
SAVEPOINT sp1;

INSERT INTO users (username, email) VALUES ('user2', 'user2@test.com');

-- حفظ نقطة تانية
SAVEPOINT sp2;

INSERT INTO users (username, email) VALUES ('user3', 'invalid_email');  -- هيفشل

-- رجوع لآخر savepoint
ROLLBACK TO SAVEPOINT sp2;
-- user3 اترجع، user1 و user2 لسه موجودين

-- أو رجوع لـ sp1
-- ROLLBACK TO SAVEPOINT sp1;
-- user2 و user3 اترجعوا، user1 بس موجود

COMMIT;  -- هيحفظ user1 و user2
```

<div dir="rtl">

### Release Savepoint

</div>

```sql
BEGIN;

SAVEPOINT sp1;
INSERT INTO logs (message) VALUES ('Step 1');

SAVEPOINT sp2;
INSERT INTO logs (message) VALUES ('Step 2');

-- لو Step 2 نجحت، مش محتاج sp1 تاني
RELEASE SAVEPOINT sp1;

-- الآن مش ممكن نرجع لـ sp1

COMMIT;
```

---

## 📊 أمثلة عملية

<div dir="rtl">

### 1. إنشاء طلب كامل

</div>

```sql
BEGIN;

-- إنشاء الطلب
INSERT INTO orders (user_id, status, total_amount)
VALUES (1, 'pending', 0)
RETURNING id INTO order_id;

-- إضافة المنتجات
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES
    (order_id, 1, 2, 99.99),
    (order_id, 3, 1, 149.99);

-- تحديث إجمالي الطلب
UPDATE orders
SET total_amount = (
    SELECT SUM(quantity * unit_price)
    FROM order_items
    WHERE order_id = order_id
)
WHERE id = order_id;

-- تقليل المخزون
UPDATE products SET stock = stock - 2 WHERE id = 1;
UPDATE products SET stock = stock - 1 WHERE id = 3;

-- التحقق من المخزون
DO $$
BEGIN
    IF EXISTS (SELECT 1 FROM products WHERE stock < 0) THEN
        RAISE EXCEPTION 'Insufficient stock';
    END IF;
END $$;

COMMIT;
```

<div dir="rtl">

### 2. تسجيل مستخدم مع Profile

</div>

```sql
BEGIN;

-- إنشاء المستخدم
INSERT INTO users (username, email, password_hash)
VALUES ('newuser', 'newuser@example.com', 'hashed_password')
RETURNING id AS new_user_id;

-- إنشاء Profile
INSERT INTO user_profiles (user_id, bio, avatar_url)
VALUES (new_user_id, '', 'default_avatar.png');

-- إنشاء Settings
INSERT INTO user_settings (user_id, theme, notifications_enabled)
VALUES (new_user_id, 'light', TRUE);

-- إرسال Welcome Email (إضافة للـ queue)
INSERT INTO email_queue (to_email, template, data)
VALUES (
    'newuser@example.com',
    'welcome',
    '{"username": "newuser"}'::jsonb
);

COMMIT;
```

<div dir="rtl">

### 3. تحديث مع Validation

</div>

```sql
BEGIN;

-- قفل الصف للتحديث
SELECT * FROM products WHERE id = 5 FOR UPDATE;

-- التحقق من الشروط
DO $$
DECLARE
    current_stock INT;
    requested_quantity INT := 10;
BEGIN
    SELECT stock INTO current_stock FROM products WHERE id = 5;

    IF current_stock < requested_quantity THEN
        RAISE EXCEPTION 'Not enough stock. Available: %, Requested: %',
            current_stock, requested_quantity;
    END IF;
END $$;

-- تنفيذ التحديث
UPDATE products
SET stock = stock - 10
WHERE id = 5;

COMMIT;
```

---

## 🔒 Transaction في Application Code

```go
// Go code
func TransferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    // بدء transaction
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    // defer للـ rollback في حالة الـ error
    defer tx.Rollback()

    // خصم من المرسل
    _, err = tx.Exec(
        "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
        amount, fromID,
    )
    if err != nil {
        return err
    }

    // إضافة للمستلم
    _, err = tx.Exec(
        "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
        amount, toID,
    )
    if err != nil {
        return err
    }

    // Commit
    return tx.Commit()
}
```

```python
# Python code
def transfer_money(conn, from_id, to_id, amount):
    try:
        with conn.cursor() as cur:
            # خصم
            cur.execute(
                "UPDATE accounts SET balance = balance - %s WHERE id = %s",
                (amount, from_id)
            )
            # إضافة
            cur.execute(
                "UPDATE accounts SET balance = balance + %s WHERE id = %s",
                (amount, to_id)
            )
        conn.commit()
    except Exception as e:
        conn.rollback()
        raise e
```

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. نسيان COMMIT أو ROLLBACK

</div>

```sql
-- ❌ Transaction معلقة
BEGIN;
UPDATE users SET name = 'test' WHERE id = 1;
-- نسيت COMMIT!
-- الـ connection لسه مفتوح والـ lock موجود

-- ✅ دايماً أنهي الـ transaction
BEGIN;
UPDATE users SET name = 'test' WHERE id = 1;
COMMIT;
```

<div dir="rtl">

### 2. Transaction طويلة جداً

</div>

```sql
-- ❌ غلط: transaction طويلة
BEGIN;
SELECT * FROM big_table;  -- query بطيء
-- عمليات كتير
-- الـ locks مستمرة طول الوقت
COMMIT;

-- ✅ صح: transactions قصيرة
-- قسم العمليات لـ transactions أصغر
```

<div dir="rtl">

### 3. تجاهل الـ errors

</div>

```sql
-- ❌ غلط: مش بتتعامل مع الـ errors
BEGIN;
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- ممكن يفشل لو balance < 1000 (CHECK constraint)
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;
COMMIT;
-- الـ COMMIT هيفشل لو الـ UPDATE الأول فشل

-- ✅ صح: تعامل مع الـ errors
BEGIN;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- في Application code: لو فشل، ROLLBACK

UPDATE accounts SET balance = balance + 1000 WHERE id = 2;
-- في Application code: لو فشل، ROLLBACK

COMMIT;
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. Transactions قصيرة

</div>

```sql
-- ✅ قصيرة ومحددة
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

<div dir="rtl">

### 2. استخدم SAVEPOINT للعمليات المعقدة

</div>

```sql
BEGIN;
-- خطوة مهمة
SAVEPOINT after_critical;

-- خطوة ممكن تفشل
-- لو فشلت: ROLLBACK TO after_critical

COMMIT;
```

<div dir="rtl">

### 3. تعامل مع Deadlocks

</div>

```sql
-- في Application: retry logic
-- لو حصل deadlock، حاول تاني
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Transaction** = مجموعة عمليات atomic
2. **BEGIN...COMMIT** لتنفيذ التغييرات
3. **ROLLBACK** لإلغاء التغييرات
4. **SAVEPOINT** لنقاط حفظ داخلية
5. **ACID** يضمن سلامة البيانات
6. Transactions **قصيرة** أفضل

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Isolation Levels](./13-isolation-levels.md)**

</div>

---

<div align="center">

[⬅️ السابق: TRUNCATE و Soft Delete](./11-truncate-soft-delete.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./13-isolation-levels.md)

</div>
