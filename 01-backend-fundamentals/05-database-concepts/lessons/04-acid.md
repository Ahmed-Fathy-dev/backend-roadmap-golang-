# Lesson 4: ACID Properties 🔒

<div dir="rtl">

## المقدمة

**ACID** = المبادئ الأربعة التي تضمن موثوقية Database transactions.

</div>

---

## 🎯 ACID Overview

```
A = Atomicity    (الذرية)
C = Consistency  (الاتساق)
I = Isolation    (العزل)
D = Durability   (الديمومة)
```

---

## ⚛️ Atomicity (الذرية)

<div dir="rtl">

**المعنى:** Transaction إما تنفذ **كلها** أو **لا شيء**.

### مثال: تحويل مال

</div>

```sql
BEGIN;
  -- Step 1: Deduct from Account A
  UPDATE accounts SET balance = balance - 1000 WHERE id = 1;

  -- Step 2: Add to Account B
  UPDATE accounts SET balance = balance + 1000 WHERE id = 2;
COMMIT;
```

<div dir="rtl">

**سيناريوهات:**

**✅ Success:** كل الـ steps تنفذ → Transaction committed

**❌ Failure:** أي step يفشل → كل Transaction يرجع (rollback)

</div>

```
Scenario: خطأ في Step 2

Step 1 ✅: Account A = 9000 (- 1000)
Step 2 ❌: Error!
→ ROLLBACK: Account A back to 10000
→ لا يضيع المال! 💰
```

---

## ✔️ Consistency (الاتساق)

<div dir="rtl">

**المعنى:** Database دائماً في حالة **صحيحة** قبل وبعد Transaction.

</div>

```sql
-- Constraint: balance >= 0
CREATE TABLE accounts (
    id INT,
    balance DECIMAL CHECK (balance >= 0)
);

BEGIN;
  UPDATE accounts SET balance = balance - 5000 WHERE id = 1;
  -- If balance < 0 → Transaction fails!
COMMIT;
```

<div dir="rtl">

**القواعد محفوظة:**

- Primary/Foreign key constraints
- Unique constraints
- Check constraints
- Triggers

</div>

---

## 🔒 Isolation (العزل)

<div dir="rtl">

**المعنى:** Transactions معزولة عن بعض - كل واحد لا يرى التغييرات غير المكتملة للآخر.

</div>

```
Transaction A:           Transaction B:
BEGIN                    BEGIN
  Read: balance = 1000     Read: balance = 1000
  balance = 1000 - 500     balance = 1000 + 200
  (uncommitted)            (uncommitted)
COMMIT                   COMMIT

With Isolation ✅:
- A sees 1000, writes 500
- B sees 1000, writes 1200
- Final: 1200 (B's commit overwrites A)

Without Isolation ❌:
- Dirty reads, lost updates possible!
```

---

## 💾 Durability (الديمومة)

<div dir="rtl">

**المعنى:** بعد COMMIT، التغييرات **دائمة** - حتى لو السيرفر انطفأ!

</div>

```
Transaction commits ✅
→ Written to disk
→ Server crashes 💥
→ Server restarts
→ Data still there! ✅
```

---

## 🎯 Real Example: Bank Transfer

```go
func TransferMoney(fromID, toID int, amount float64) error {
    // Start transaction
    tx := db.Begin()

    // Atomicity: All or nothing
    // Deduct from sender
    if err := tx.Exec(
        "UPDATE accounts SET balance = balance - ? WHERE id = ?",
        amount, fromID,
    ).Error; err != nil {
        tx.Rollback()  // ⚛️ Atomicity
        return err
    }

    // Add to receiver
    if err := tx.Exec(
        "UPDATE accounts SET balance = balance + ? WHERE id = ?",
        amount, toID,
    ).Error; err != nil {
        tx.Rollback()  // ⚛️ Atomicity
        return err
    }

    // Consistency: Check constraints enforced
    // Isolation: Other transactions don't see partial updates
    // Durability: After commit, changes permanent

    return tx.Commit().Error  // 💾 Durability
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Atomicity:** All or nothing
- ✅ **Consistency:** Rules always valid
- ✅ **Isolation:** Transactions don't interfere
- ✅ **Durability:** Commits are permanent
- ✅ ACID = موثوقية Database

</div>

---

<div align="center">

[⬅️ Previous: Relationships](./03-relationships.md) | [➡️ Next: Transactions](./05-transactions.md) | [📚 Module Home](../README.md)

</div>
