# Isolation Levels - مستويات العزل 🔒

<div dir="rtl">

## مقدمة

Isolation Levels بتحدد إزاي الـ transactions بتشوف بعض. مهم جداً لفهم الـ concurrency والتعامل مع المشاكل اللي ممكن تحصل.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 المشاكل الممكنة

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Concurrency Problems                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Dirty Read (قراءة متسخة)                                        │
│     قراءة بيانات من transaction لسه ما عملتش commit                 │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │  T1: UPDATE balance = 500 (من 1000)                        │  │
│     │  T2: SELECT balance → 500 (قرأ القيمة الجديدة!)            │  │
│     │  T1: ROLLBACK (رجع لـ 1000)                                │  │
│     │  T2: شاف 500 بس القيمة الحقيقية 1000!                      │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. Non-Repeatable Read (قراءة غير قابلة للتكرار)                   │
│     نفس الـ query بترجع نتائج مختلفة في نفس الـ transaction        │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │  T1: SELECT balance → 1000                                 │  │
│     │  T2: UPDATE balance = 500; COMMIT;                         │  │
│     │  T1: SELECT balance → 500 (مختلف!)                         │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  3. Phantom Read (قراءة شبحية)                                       │
│     ظهور صفوف جديدة في نفس الـ query                                │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │  T1: SELECT COUNT(*) WHERE status='active' → 10            │  │
│     │  T2: INSERT INTO... status='active'; COMMIT;               │  │
│     │  T1: SELECT COUNT(*) WHERE status='active' → 11 (زاد!)     │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  4. Serialization Anomaly                                           │
│     نتيجة مختلفة عن لو الـ transactions اتنفذت بالتتابع            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📋 مستويات العزل

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Isolation Levels                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Level              │ Dirty │ Non-Rep │ Phantom │ Serialization    │
│                     │ Read  │  Read   │  Read   │   Anomaly        │
│  ───────────────────┼───────┼─────────┼─────────┼─────────────────  │
│  READ UNCOMMITTED   │  ✓    │   ✓     │    ✓    │      ✓           │
│  READ COMMITTED     │  ✗    │   ✓     │    ✓    │      ✓           │
│  REPEATABLE READ    │  ✗    │   ✗     │    ✓*   │      ✓           │
│  SERIALIZABLE       │  ✗    │   ✗     │    ✗    │      ✗           │
│                                                                      │
│  ✓ = ممكن يحصل    ✗ = محمي منه                                     │
│  * PostgreSQL بيحمي من Phantom Reads في REPEATABLE READ            │
│                                                                      │
│  PostgreSQL defaults: READ COMMITTED                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 تعيين Isolation Level

```sql
-- على مستوى Transaction
BEGIN ISOLATION LEVEL READ COMMITTED;
-- أو
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- على مستوى Session
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- التحقق من المستوى الحالي
SHOW transaction_isolation;
```

---

## 📖 READ COMMITTED (Default)

<div dir="rtl">

### الخصائص

- كل query بتشوف البيانات المؤكدة فقط (committed)
- Queries مختلفة في نفس الـ transaction ممكن تشوف changes مختلفة

</div>

```sql
-- Session 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 1000
-- في هذه اللحظة Session 2 بتحدث وتعمل commit
SELECT balance FROM accounts WHERE id = 1;  -- 500 (اتغيرت!)
COMMIT;

-- Session 2
BEGIN;
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;
```

<div dir="rtl">

### متى تستخدمه

- معظم الـ applications
- لما مش محتاج consistency قوية
- Default في PostgreSQL

</div>

---

## 🔄 REPEATABLE READ

<div dir="rtl">

### الخصائص

- الـ transaction بتشوف snapshot من البيانات وقت بدايتها
- كل الـ queries بترجع نفس النتائج

</div>

```sql
-- Session 1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- 1000
-- Session 2 بتحدث وتعمل commit
SELECT balance FROM accounts WHERE id = 1;  -- 1000 (نفس القيمة!)
COMMIT;

-- Session 2
BEGIN;
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;  -- Session 1 مش هتشوف التغيير
```

<div dir="rtl">

### Serialization Failure

</div>

```sql
-- Session 1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM accounts WHERE id = 1;  -- balance = 1000
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- هنا Session 2 بتحدث نفس الصف

-- Session 2
BEGIN ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance - 200 WHERE id = 1;
COMMIT;  -- نجح

-- Session 1
COMMIT;
-- ERROR: could not serialize access due to concurrent update
-- لازم تعيد المحاولة!
```

<div dir="rtl">

### متى تستخدمه

- Reports اللي محتاج تكون consistent
- Read-heavy workloads
- Analytics queries

</div>

---

## 🔒 SERIALIZABLE

<div dir="rtl">

### الخصائص

- أقوى مستوى عزل
- الـ transactions بتتصرف كأنها اتنفذت واحدة بعد التانية
- PostgreSQL بيستخدم SSI (Serializable Snapshot Isolation)

</div>

```sql
-- جدول
CREATE TABLE counters (
    name VARCHAR(50) PRIMARY KEY,
    value INT NOT NULL
);
INSERT INTO counters VALUES ('total', 0);

-- Session 1
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT value FROM counters WHERE name = 'total';  -- 0
UPDATE counters SET value = value + 1 WHERE name = 'total';

-- Session 2 (في نفس الوقت)
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT value FROM counters WHERE name = 'total';  -- 0
UPDATE counters SET value = value + 1 WHERE name = 'total';

-- واحد منهم هيفشل بـ serialization error
-- Session 1: COMMIT; → نجح
-- Session 2: COMMIT; → ERROR: could not serialize access
```

<div dir="rtl">

### متى تستخدمه

- العمليات المالية الحساسة
- لما محتاج consistency قصوى
- لما الـ application logic معقد

</div>

---

## 📊 مثال عملي: Inventory

```sql
-- المشكلة: Lost Update
-- Session 1 و 2 بيقرأوا stock = 10
-- Session 1: بيبيع 3 → يحدث لـ 7
-- Session 2: بيبيع 5 → يحدث لـ 5
-- النتيجة المتوقعة: 10 - 3 - 5 = 2
-- النتيجة الفعلية: 5 (آخر update كسب!)

-- الحل 1: REPEATABLE READ مع retry
BEGIN ISOLATION LEVEL REPEATABLE READ;
UPDATE products SET stock = stock - 3 WHERE id = 1;
COMMIT;
-- لو فشل، حاول تاني

-- الحل 2: SELECT FOR UPDATE
BEGIN;
SELECT stock FROM products WHERE id = 1 FOR UPDATE;
-- الآن الصف مقفول، الـ session التاني هيستنى
UPDATE products SET stock = stock - 3 WHERE id = 1;
COMMIT;

-- الحل 3: Optimistic Locking
BEGIN;
SELECT stock, version FROM products WHERE id = 1;
-- stock = 10, version = 1

UPDATE products
SET stock = stock - 3, version = version + 1
WHERE id = 1 AND version = 1;
-- لو الـ version اتغيرت، 0 rows affected → retry

COMMIT;
```

---

## 🔄 Handling Serialization Errors

```go
// Go: Retry logic
func executeWithRetry(db *sql.DB, maxRetries int, fn func(*sql.Tx) error) error {
    for i := 0; i < maxRetries; i++ {
        tx, err := db.BeginTx(ctx, &sql.TxOptions{
            Isolation: sql.LevelSerializable,
        })
        if err != nil {
            return err
        }

        err = fn(tx)
        if err != nil {
            tx.Rollback()
            // Check if serialization error
            if isSerializationError(err) {
                continue  // Retry
            }
            return err
        }

        err = tx.Commit()
        if err != nil {
            if isSerializationError(err) {
                continue  // Retry
            }
            return err
        }
        return nil  // Success
    }
    return fmt.Errorf("max retries exceeded")
}

func isSerializationError(err error) bool {
    // PostgreSQL error code 40001
    return strings.Contains(err.Error(), "could not serialize")
}
```

---

## 📊 مقارنة الأداء

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Performance vs Consistency                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  READ COMMITTED:                                                    │
│  ├── Performance: ⭐⭐⭐⭐⭐                                          │
│  ├── Consistency: ⭐⭐                                               │
│  └── Use: معظم الـ applications                                     │
│                                                                      │
│  REPEATABLE READ:                                                   │
│  ├── Performance: ⭐⭐⭐⭐                                            │
│  ├── Consistency: ⭐⭐⭐⭐                                            │
│  └── Use: Reports, Analytics                                        │
│                                                                      │
│  SERIALIZABLE:                                                      │
│  ├── Performance: ⭐⭐⭐                                              │
│  ├── Consistency: ⭐⭐⭐⭐⭐                                          │
│  └── Use: Financial, Critical operations                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. استخدم READ COMMITTED كـ default

</div>

```sql
-- للعمليات العادية
BEGIN;
-- operations
COMMIT;
```

<div dir="rtl">

### 2. REPEATABLE READ للـ Reports

</div>

```sql
-- تقرير محتاج يكون consistent
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT SUM(amount) FROM transactions;
SELECT COUNT(*) FROM transactions;
-- الرقمين متوافقين
COMMIT;
```

<div dir="rtl">

### 3. Retry logic للـ SERIALIZABLE

</div>

```sql
-- في Application code
-- while (retries < max) {
--     try { execute(); break; }
--     catch (SerializationError) { retries++; }
-- }
```

<div dir="rtl">

### 4. استخدم FOR UPDATE للـ Critical Sections

</div>

```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- الآن الصف مقفول
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **READ COMMITTED** للعمليات العادية (default)
2. **REPEATABLE READ** للـ reports و analytics
3. **SERIALIZABLE** للعمليات الحساسة جداً
4. دايماً **handle serialization errors**
5. **FOR UPDATE** لقفل صفوف معينة

</div>

---

## 🎉 انتهى Module 03!

<div dir="rtl">

**➡️ الـ Module التالي: [Joins & Relations](../../04-joins-relations/README.md)**

</div>

---

<div align="center">

[⬅️ السابق: Transactions](./12-transactions.md) | [🏠 العودة للـ Module](../README.md) | [Module التالي ➡️](../../04-joins-relations/README.md)

</div>
