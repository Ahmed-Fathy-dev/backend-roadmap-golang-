# Lesson 5: Transactions 🔄

<div dir="rtl">

## المقدمة

**Transaction** = مجموعة من العمليات تُعامل كـ **وحدة واحدة**.

</div>

---

## 🎯 Transaction Basics

```sql
BEGIN;              -- Start transaction
  -- SQL statements here
COMMIT;             -- Save changes

-- OR

ROLLBACK;           -- Cancel all changes
```

---

## 💰 Example: Money Transfer

```sql
BEGIN;

-- Step 1: Check sender balance
SELECT balance FROM accounts WHERE id = 1;
-- balance = 5000

-- Step 2: Deduct from sender
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;

-- Step 3: Add to receiver
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

COMMIT;  -- ✅ All changes saved
```

### If Error Occurs:

```sql
BEGIN;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- ✅ Success

UPDATE accounts SET balance = balance + 1000 WHERE id = 999;
-- ❌ Error: Account 999 doesn't exist!

ROLLBACK;  -- ⚠️ First update also cancelled!
-- Account 1 back to original balance
```

---

## 🔧 In Go (GORM)

```go
func TransferMoney(fromID, toID int, amount float64) error {
    // Begin transaction
    tx := db.Begin()

    // Deduct from sender
    if err := tx.Model(&Account{}).
        Where("id = ?", fromID).
        Update("balance", gorm.Expr("balance - ?", amount)).
        Error; err != nil {
        tx.Rollback()  // Cancel transaction
        return err
    }

    // Add to receiver
    if err := tx.Model(&Account{}).
        Where("id = ?", toID).
        Update("balance", gorm.Expr("balance + ?", amount)).
        Error; err != nil {
        tx.Rollback()  // Cancel transaction
        return err
    }

    // Commit transaction
    return tx.Commit().Error
}
```

---

## 🎯 When to Use Transactions

```
✅ Money transfers
✅ Creating user + profile (both or none)
✅ Order + order items (atomic)
✅ Any multi-step operation that must be atomic

❌ Single SELECT (no need)
❌ Single INSERT (implicit transaction)
```

---

## ⚡ Transaction in Practice

### Creating User with Profile:

```go
func CreateUserWithProfile(email, name string) error {
    tx := db.Begin()

    // Create user
    user := &User{Email: email}
    if err := tx.Create(user).Error; err != nil {
        tx.Rollback()
        return err
    }

    // Create profile
    profile := &UserProfile{
        UserID:   user.ID,
        FullName: name,
    }
    if err := tx.Create(profile).Error; err != nil {
        tx.Rollback()  // User creation also cancelled!
        return err
    }

    return tx.Commit().Error
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **BEGIN** لبدء transaction
- ✅ **COMMIT** لحفظ التغييرات
- ✅ **ROLLBACK** لإلغاء كل شيء
- ✅ استخدم للعمليات المترابطة
- ✅ All or nothing!

</div>

---

<div align="center">

[⬅️ Previous: ACID](./04-acid.md) | [➡️ Next: Indexes](./06-indexes.md) | [📚 Module Home](../README.md)

</div>
