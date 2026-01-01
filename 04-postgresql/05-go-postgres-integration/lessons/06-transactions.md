# Transactions in Go - التعامل مع الـ Transactions 🔄

<div dir="rtl">

## مقدمة

الـ Transactions بتضمن إن مجموعة من العمليات تنفذ كلها مع بعض أو تفشل كلها. ده مهم جداً للحفاظ على سلامة البيانات.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 لماذا الـ Transactions؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Without Transaction                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Transfer $100 from Account A to Account B:                        │
│                                                                      │
│   Step 1: Deduct from A   → ✅ Success (A: $1000 → $900)           │
│   Step 2: Add to B        → ❌ Error! (server crashed)              │
│                                                                      │
│   Result: $100 lost! 💸                                              │
│   A has $900, B still has $500                                       │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                    With Transaction                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   BEGIN TRANSACTION                                                  │
│   Step 1: Deduct from A   → ✅ Success                              │
│   Step 2: Add to B        → ❌ Error!                                │
│   ROLLBACK                → 🔄 Both operations undone                │
│                                                                      │
│   Result: No money lost! ✅                                          │
│   A still has $1000, B still has $500                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Transaction باستخدام database/sql

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "log"

    _ "github.com/jackc/pgx/v5/stdlib"
)

func transferMoney(db *sql.DB, fromID, toID int, amount float64) error {
    ctx := context.Background()

    // بدء الـ Transaction
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin transaction: %w", err)
    }

    // defer Rollback - لو حصل error أو panic
    // لو عملنا Commit قبلها، الـ Rollback مش هيأثر
    defer tx.Rollback()

    // خصم من الحساب المصدر
    result, err := tx.ExecContext(ctx,
        "UPDATE accounts SET balance = balance - $1 WHERE id = $2 AND balance >= $1",
        amount, fromID,
    )
    if err != nil {
        return fmt.Errorf("deduct from source: %w", err)
    }

    rowsAffected, _ := result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("insufficient balance or account not found")
    }

    // إضافة للحساب المستلم
    result, err = tx.ExecContext(ctx,
        "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
        amount, toID,
    )
    if err != nil {
        return fmt.Errorf("add to destination: %w", err)
    }

    rowsAffected, _ = result.RowsAffected()
    if rowsAffected == 0 {
        return fmt.Errorf("destination account not found")
    }

    // تأكيد الـ Transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("commit transaction: %w", err)
    }

    return nil
}

func main() {
    db, _ := sql.Open("pgx", "postgres://postgres:password@localhost:5432/mydb")
    defer db.Close()

    err := transferMoney(db, 1, 2, 100.00)
    if err != nil {
        log.Printf("Transfer failed: %v\n", err)
    } else {
        log.Println("Transfer successful!")
    }
}
```

---

## ⚙️ Transaction Options

```go
func example(db *sql.DB) error {
    ctx := context.Background()

    // TxOptions للتحكم في سلوك الـ Transaction
    tx, err := db.BeginTx(ctx, &sql.TxOptions{
        // Isolation Level
        Isolation: sql.LevelSerializable,
        // Read-only transaction (للـ reports)
        ReadOnly: false,
    })
    if err != nil {
        return err
    }
    defer tx.Rollback()

    // ... operations ...

    return tx.Commit()
}
```

<div dir="rtl">

### Isolation Levels في Go

</div>

```go
// Isolation Levels المتاحة
sql.LevelDefault          // PostgreSQL default (READ COMMITTED)
sql.LevelReadUncommitted  // ⚠️ مش مستحسن
sql.LevelReadCommitted    // الـ default في PostgreSQL
sql.LevelRepeatableRead   // للـ reports
sql.LevelSerializable     // الأقوى، للعمليات الحساسة
```

---

## 🚀 Transaction باستخدام pgx

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5"
    "github.com/jackc/pgx/v5/pgxpool"
)

func transferMoneyPgx(pool *pgxpool.Pool, fromID, toID int, amount float64) error {
    ctx := context.Background()

    // BeginTx مع options
    tx, err := pool.BeginTx(ctx, pgx.TxOptions{
        IsoLevel: pgx.Serializable,
    })
    if err != nil {
        return err
    }
    defer tx.Rollback(ctx)

    // خصم
    tag, err := tx.Exec(ctx,
        "UPDATE accounts SET balance = balance - $1 WHERE id = $2 AND balance >= $1",
        amount, fromID,
    )
    if err != nil {
        return err
    }
    if tag.RowsAffected() == 0 {
        return fmt.Errorf("insufficient balance")
    }

    // إضافة
    _, err = tx.Exec(ctx,
        "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
        amount, toID,
    )
    if err != nil {
        return err
    }

    return tx.Commit(ctx)
}
```

---

## 🏭 Transaction Helper Pattern

<div dir="rtl">

Pattern مفيد لتبسيط التعامل مع الـ Transactions:

</div>

```go
package db

import (
    "context"
    "database/sql"
    "fmt"
)

// TxFunc is a function that runs within a transaction
type TxFunc func(tx *sql.Tx) error

// WithTransaction executes fn within a transaction
func WithTransaction(ctx context.Context, db *sql.DB, fn TxFunc) error {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin transaction: %w", err)
    }

    defer func() {
        if p := recover(); p != nil {
            tx.Rollback()
            panic(p) // re-throw panic after rollback
        }
    }()

    if err := fn(tx); err != nil {
        if rbErr := tx.Rollback(); rbErr != nil {
            return fmt.Errorf("rollback failed: %v (original error: %w)", rbErr, err)
        }
        return err
    }

    if err := tx.Commit(); err != nil {
        return fmt.Errorf("commit transaction: %w", err)
    }

    return nil
}

// استخدام الـ Helper
func createOrderWithItems(db *sql.DB, userID int, items []OrderItem) (int, error) {
    ctx := context.Background()
    var orderID int

    err := WithTransaction(ctx, db, func(tx *sql.Tx) error {
        // إنشاء الطلب
        err := tx.QueryRowContext(ctx,
            "INSERT INTO orders (user_id, status) VALUES ($1, 'pending') RETURNING id",
            userID,
        ).Scan(&orderID)
        if err != nil {
            return fmt.Errorf("create order: %w", err)
        }

        // إضافة العناصر
        stmt, err := tx.PrepareContext(ctx,
            "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)")
        if err != nil {
            return fmt.Errorf("prepare statement: %w", err)
        }
        defer stmt.Close()

        var total float64
        for _, item := range items {
            _, err := stmt.ExecContext(ctx, orderID, item.ProductID, item.Quantity, item.Price)
            if err != nil {
                return fmt.Errorf("add item %d: %w", item.ProductID, err)
            }
            total += item.Price * float64(item.Quantity)
        }

        // تحديث المجموع
        _, err = tx.ExecContext(ctx,
            "UPDATE orders SET total_amount = $1 WHERE id = $2",
            total, orderID,
        )
        if err != nil {
            return fmt.Errorf("update total: %w", err)
        }

        return nil
    })

    return orderID, err
}
```

---

## 🔄 pgx Transaction Helper

```go
package db

import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
    "github.com/jackc/pgx/v5/pgxpool"
)

// pgxpool.Pool has built-in BeginFunc!
func createOrderPgx(pool *pgxpool.Pool, userID int, items []OrderItem) (int, error) {
    ctx := context.Background()
    var orderID int

    err := pgx.BeginFunc(ctx, pool, func(tx pgx.Tx) error {
        // إنشاء الطلب
        err := tx.QueryRow(ctx,
            "INSERT INTO orders (user_id, status) VALUES ($1, 'pending') RETURNING id",
            userID,
        ).Scan(&orderID)
        if err != nil {
            return err
        }

        // إضافة العناصر
        var total float64
        for _, item := range items {
            _, err := tx.Exec(ctx,
                "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)",
                orderID, item.ProductID, item.Quantity, item.Price,
            )
            if err != nil {
                return err
            }
            total += item.Price * float64(item.Quantity)
        }

        // تحديث المجموع
        _, err = tx.Exec(ctx,
            "UPDATE orders SET total_amount = $1 WHERE id = $2",
            total, orderID,
        )
        return err
    })

    return orderID, err
}
```

---

## 📌 SAVEPOINT

<div dir="rtl">

الـ SAVEPOINT بيسمح بـ rollback جزئي داخل الـ transaction:

</div>

```go
func processWithSavepoint(db *sql.DB) error {
    ctx := context.Background()

    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    // العملية الأولى
    _, err = tx.ExecContext(ctx, "INSERT INTO logs (message) VALUES ($1)", "Started")
    if err != nil {
        return err
    }

    // إنشاء SAVEPOINT
    _, err = tx.ExecContext(ctx, "SAVEPOINT before_risky")
    if err != nil {
        return err
    }

    // عملية خطرة
    _, err = tx.ExecContext(ctx, "UPDATE accounts SET balance = balance - 1000 WHERE id = $1", 1)
    if err != nil {
        // Rollback للـ savepoint فقط
        tx.ExecContext(ctx, "ROLLBACK TO SAVEPOINT before_risky")
        // العملية الأولى (log) لسه موجودة!

        // Log the failure
        tx.ExecContext(ctx, "INSERT INTO logs (message) VALUES ($1)", "Risky operation failed")
    } else {
        // Release الـ savepoint لو نجحت
        tx.ExecContext(ctx, "RELEASE SAVEPOINT before_risky")
    }

    // العملية الأخيرة
    _, err = tx.ExecContext(ctx, "INSERT INTO logs (message) VALUES ($1)", "Completed")
    if err != nil {
        return err
    }

    return tx.Commit()
}
```

---

## ⚠️ التعامل مع الأخطاء

```go
import (
    "errors"
    "github.com/jackc/pgx/v5/pgconn"
)

func handleTransactionError(err error) error {
    if err == nil {
        return nil
    }

    // PostgreSQL-specific errors
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        switch pgErr.Code {
        case "23505": // unique_violation
            return fmt.Errorf("duplicate entry: %s", pgErr.Detail)
        case "23503": // foreign_key_violation
            return fmt.Errorf("referenced record not found")
        case "40001": // serialization_failure
            return fmt.Errorf("transaction conflict, please retry")
        case "40P01": // deadlock_detected
            return fmt.Errorf("deadlock detected, please retry")
        }
    }

    return err
}

// مع Retry للـ serialization errors
func executeWithRetry(db *sql.DB, maxRetries int, fn func(*sql.Tx) error) error {
    for attempt := 0; attempt < maxRetries; attempt++ {
        tx, err := db.Begin()
        if err != nil {
            return err
        }

        err = fn(tx)
        if err != nil {
            tx.Rollback()

            // Check if retryable
            var pgErr *pgconn.PgError
            if errors.As(err, &pgErr) {
                if pgErr.Code == "40001" || pgErr.Code == "40P01" {
                    // Serialization or deadlock - retry
                    time.Sleep(time.Duration(attempt*100) * time.Millisecond)
                    continue
                }
            }
            return err
        }

        err = tx.Commit()
        if err != nil {
            var pgErr *pgconn.PgError
            if errors.As(err, &pgErr) && pgErr.Code == "40001" {
                time.Sleep(time.Duration(attempt*100) * time.Millisecond)
                continue
            }
            return err
        }

        return nil // Success!
    }

    return fmt.Errorf("max retries exceeded")
}
```

---

## 🏭 Real-World Example: E-Commerce Order

```go
type OrderService struct {
    pool *pgxpool.Pool
}

type CreateOrderInput struct {
    UserID int
    Items  []struct {
        ProductID int
        Quantity  int
    }
}

func (s *OrderService) CreateOrder(ctx context.Context, input CreateOrderInput) (*Order, error) {
    var order Order

    err := pgx.BeginFunc(ctx, s.pool, func(tx pgx.Tx) error {
        // 1. التحقق من المستخدم
        var userExists bool
        err := tx.QueryRow(ctx,
            "SELECT EXISTS(SELECT 1 FROM users WHERE id = $1 AND is_active = TRUE)",
            input.UserID,
        ).Scan(&userExists)
        if err != nil {
            return fmt.Errorf("check user: %w", err)
        }
        if !userExists {
            return fmt.Errorf("user not found or inactive")
        }

        // 2. إنشاء الطلب
        err = tx.QueryRow(ctx,
            "INSERT INTO orders (user_id, status) VALUES ($1, 'pending') RETURNING id, created_at",
            input.UserID,
        ).Scan(&order.ID, &order.CreatedAt)
        if err != nil {
            return fmt.Errorf("create order: %w", err)
        }

        // 3. التحقق من المخزون وإضافة العناصر
        var totalAmount float64
        for _, item := range input.Items {
            // Lock the product row for update
            var product struct {
                ID    int
                Name  string
                Price float64
                Stock int
            }

            err := tx.QueryRow(ctx,
                "SELECT id, name, price, stock FROM products WHERE id = $1 FOR UPDATE",
                item.ProductID,
            ).Scan(&product.ID, &product.Name, &product.Price, &product.Stock)
            if err != nil {
                return fmt.Errorf("get product %d: %w", item.ProductID, err)
            }

            if product.Stock < item.Quantity {
                return fmt.Errorf("insufficient stock for product %s", product.Name)
            }

            // خصم من المخزون
            _, err = tx.Exec(ctx,
                "UPDATE products SET stock = stock - $1 WHERE id = $2",
                item.Quantity, item.ProductID,
            )
            if err != nil {
                return fmt.Errorf("update stock: %w", err)
            }

            // إضافة عنصر للطلب
            _, err = tx.Exec(ctx,
                `INSERT INTO order_items (order_id, product_id, product_name, quantity, unit_price)
                 VALUES ($1, $2, $3, $4, $5)`,
                order.ID, item.ProductID, product.Name, item.Quantity, product.Price,
            )
            if err != nil {
                return fmt.Errorf("add order item: %w", err)
            }

            totalAmount += product.Price * float64(item.Quantity)
        }

        // 4. تحديث مجموع الطلب
        _, err = tx.Exec(ctx,
            "UPDATE orders SET total_amount = $1 WHERE id = $2",
            totalAmount, order.ID,
        )
        if err != nil {
            return fmt.Errorf("update order total: %w", err)
        }

        order.UserID = input.UserID
        order.TotalAmount = totalAmount
        order.Status = "pending"

        return nil
    })

    if err != nil {
        return nil, err
    }

    return &order, nil
}
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Transaction Best Practices                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Always defer Rollback                                            │
│     tx, _ := db.Begin()                                             │
│     defer tx.Rollback() // Safe if already committed                │
│                                                                      │
│  2. Keep transactions short                                          │
│     ❌ Transaction with HTTP calls or slow operations               │
│     ✅ Only database operations inside transaction                  │
│                                                                      │
│  3. Use appropriate isolation level                                  │
│     READ COMMITTED for most cases                                   │
│     SERIALIZABLE for critical financial operations                  │
│                                                                      │
│  4. Handle serialization errors with retry                           │
│     for attempt := 0; attempt < maxRetries; attempt++ { ... }       │
│                                                                      │
│  5. Use FOR UPDATE when reading then updating                        │
│     SELECT * FROM products WHERE id = $1 FOR UPDATE                 │
│                                                                      │
│  6. Batch operations in single transaction                           │
│     One transaction for creating order + items + updating stock     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **defer tx.Rollback()** دايماً بعد BeginTx
2. خلي الـ **Transaction قصيرة** - database operations فقط
3. استخدم **Isolation Level** مناسب للـ use case
4. تعامل مع **serialization errors** بـ retry
5. استخدم **FOR UPDATE** لـ read-modify-write
6. استخدم **SAVEPOINT** للـ partial rollback

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [GORM Basics](./07-gorm-basics.md)**

</div>

---

<div align="center">

[⬅️ السابق](./05-prepared-statements.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./07-gorm-basics.md)

</div>
