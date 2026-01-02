# Stored Procedures & Functions - الـ Functions و Procedures 🔧

<div dir="rtl">

## مقدمة

PostgreSQL بيدعم Functions و Procedures لتنفيذ logic معقد على الـ database server. بيقللوا الـ network overhead وبيحسنوا الـ performance والـ security.

**المدة المتوقعة:** 35 دقيقة

</div>

---

## 📊 Functions vs Procedures

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Functions vs Procedures                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Feature              │ Function          │ Procedure                │
│  ─────────────────────┼───────────────────┼────────────────────────  │
│  Returns value        │ Yes (required)    │ No (optional OUT)        │
│  Transaction control  │ No                │ Yes (COMMIT/ROLLBACK)    │
│  Can be in SELECT     │ Yes               │ No                       │
│  Call syntax          │ SELECT func()     │ CALL proc()              │
│  Use in expressions   │ Yes               │ No                       │
│  Introduced           │ Always            │ PostgreSQL 11+           │
│                                                                      │
│  When to use:                                                        │
│  • Functions: calculations, transformations, queries                │
│  • Procedures: multi-step operations, transaction control           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Basic Functions

### Simple Function

```sql
-- Function ترجع قيمة واحدة
CREATE OR REPLACE FUNCTION calculate_tax(amount DECIMAL)
RETURNS DECIMAL AS $$
BEGIN
    RETURN amount * 0.14;  -- 14% VAT
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT calculate_tax(100);  -- 14.00
SELECT name, price, calculate_tax(price) as tax FROM products;
```

### Function مع Multiple Parameters

```sql
-- حساب السعر بعد الخصم
CREATE OR REPLACE FUNCTION apply_discount(
    original_price DECIMAL,
    discount_percent DECIMAL DEFAULT 0
)
RETURNS DECIMAL AS $$
DECLARE
    discount_amount DECIMAL;
    final_price DECIMAL;
BEGIN
    discount_amount := original_price * (discount_percent / 100);
    final_price := original_price - discount_amount;
    RETURN GREATEST(final_price, 0);  -- لا يكون سالب
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT apply_discount(100, 20);  -- 80.00
SELECT apply_discount(100);       -- 100.00 (default discount)
```

### Function ترجع Table

```sql
-- Function ترجع مجموعة rows
CREATE OR REPLACE FUNCTION get_user_orders(p_user_id INT)
RETURNS TABLE (
    order_id INT,
    total_amount DECIMAL,
    status VARCHAR,
    created_at TIMESTAMPTZ
) AS $$
BEGIN
    RETURN QUERY
    SELECT o.id, o.total_amount, o.status, o.created_at
    FROM orders o
    WHERE o.user_id = p_user_id
    ORDER BY o.created_at DESC;
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT * FROM get_user_orders(1);
SELECT * FROM get_user_orders(1) WHERE status = 'completed';
```

### Function مع SETOF

```sql
-- ترجع نوع موجود
CREATE OR REPLACE FUNCTION get_active_users()
RETURNS SETOF users AS $$
BEGIN
    RETURN QUERY
    SELECT * FROM users WHERE is_active = true;
END;
$$ LANGUAGE plpgsql;

-- أو custom type
CREATE TYPE order_summary AS (
    user_id INT,
    username VARCHAR,
    total_orders BIGINT,
    total_spent DECIMAL
);

CREATE OR REPLACE FUNCTION get_top_customers(limit_count INT DEFAULT 10)
RETURNS SETOF order_summary AS $$
BEGIN
    RETURN QUERY
    SELECT
        u.id,
        u.username,
        COUNT(o.id)::BIGINT,
        COALESCE(SUM(o.total_amount), 0)
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id
    ORDER BY SUM(o.total_amount) DESC NULLS LAST
    LIMIT limit_count;
END;
$$ LANGUAGE plpgsql;
```

---

## 2️⃣ Procedures

### Basic Procedure

```sql
-- Procedure لنقل الفلوس
CREATE OR REPLACE PROCEDURE transfer_funds(
    from_account INT,
    to_account INT,
    amount DECIMAL
)
LANGUAGE plpgsql AS $$
BEGIN
    -- خصم من الحساب الأول
    UPDATE accounts
    SET balance = balance - amount
    WHERE id = from_account;

    -- إضافة للحساب الثاني
    UPDATE accounts
    SET balance = balance + amount
    WHERE id = to_account;

    -- Commit داخل الـ procedure
    COMMIT;
END;
$$;

-- استخدام
CALL transfer_funds(1, 2, 500.00);
```

### Procedure مع Transaction Control

```sql
CREATE OR REPLACE PROCEDURE process_order(
    p_order_id INT
)
LANGUAGE plpgsql AS $$
DECLARE
    v_user_id INT;
    v_total DECIMAL;
    v_item RECORD;
BEGIN
    -- Get order info
    SELECT user_id, total_amount INTO v_user_id, v_total
    FROM orders WHERE id = p_order_id;

    -- Start processing
    BEGIN
        -- Update order status
        UPDATE orders SET status = 'processing' WHERE id = p_order_id;

        -- Deduct stock
        FOR v_item IN
            SELECT product_id, quantity FROM order_items WHERE order_id = p_order_id
        LOOP
            UPDATE products
            SET stock = stock - v_item.quantity
            WHERE id = v_item.product_id;

            -- Check if stock went negative
            IF NOT FOUND OR (SELECT stock FROM products WHERE id = v_item.product_id) < 0 THEN
                RAISE EXCEPTION 'Insufficient stock for product %', v_item.product_id;
            END IF;
        END LOOP;

        -- Charge customer
        UPDATE customer_balance
        SET balance = balance - v_total
        WHERE user_id = v_user_id;

        -- Complete
        UPDATE orders SET status = 'completed' WHERE id = p_order_id;

        COMMIT;

    EXCEPTION WHEN OTHERS THEN
        ROLLBACK;
        UPDATE orders SET status = 'failed' WHERE id = p_order_id;
        COMMIT;
        RAISE;
    END;
END;
$$;

-- استخدام
CALL process_order(123);
```

### Procedure مع OUT Parameters

```sql
CREATE OR REPLACE PROCEDURE get_order_stats(
    p_user_id INT,
    OUT total_orders INT,
    OUT total_spent DECIMAL,
    OUT avg_order DECIMAL
)
LANGUAGE plpgsql AS $$
BEGIN
    SELECT
        COUNT(*)::INT,
        COALESCE(SUM(total_amount), 0),
        COALESCE(AVG(total_amount), 0)
    INTO total_orders, total_spent, avg_order
    FROM orders
    WHERE user_id = p_user_id AND status = 'completed';
END;
$$;

-- استخدام
CALL get_order_stats(1, NULL, NULL, NULL);

-- أو
DO $$
DECLARE
    v_orders INT;
    v_spent DECIMAL;
    v_avg DECIMAL;
BEGIN
    CALL get_order_stats(1, v_orders, v_spent, v_avg);
    RAISE NOTICE 'Orders: %, Spent: %, Avg: %', v_orders, v_spent, v_avg;
END $$;
```

---

## 3️⃣ Control Structures

### IF-THEN-ELSE

```sql
CREATE OR REPLACE FUNCTION get_discount_tier(total_spent DECIMAL)
RETURNS VARCHAR AS $$
BEGIN
    IF total_spent >= 10000 THEN
        RETURN 'platinum';
    ELSIF total_spent >= 5000 THEN
        RETURN 'gold';
    ELSIF total_spent >= 1000 THEN
        RETURN 'silver';
    ELSE
        RETURN 'bronze';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

### CASE

```sql
CREATE OR REPLACE FUNCTION translate_status(status VARCHAR)
RETURNS VARCHAR AS $$
BEGIN
    RETURN CASE status
        WHEN 'pending' THEN 'قيد الانتظار'
        WHEN 'processing' THEN 'قيد المعالجة'
        WHEN 'shipped' THEN 'تم الشحن'
        WHEN 'delivered' THEN 'تم التسليم'
        WHEN 'cancelled' THEN 'ملغي'
        ELSE 'غير معروف'
    END;
END;
$$ LANGUAGE plpgsql;
```

### Loops

```sql
-- FOR loop
CREATE OR REPLACE FUNCTION generate_numbers(n INT)
RETURNS TABLE(num INT) AS $$
BEGIN
    FOR i IN 1..n LOOP
        num := i;
        RETURN NEXT;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- WHILE loop
CREATE OR REPLACE FUNCTION factorial(n INT)
RETURNS BIGINT AS $$
DECLARE
    result BIGINT := 1;
    i INT := n;
BEGIN
    WHILE i > 1 LOOP
        result := result * i;
        i := i - 1;
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;

-- FOREACH (for arrays)
CREATE OR REPLACE FUNCTION sum_array(arr INT[])
RETURNS INT AS $$
DECLARE
    total INT := 0;
    elem INT;
BEGIN
    FOREACH elem IN ARRAY arr LOOP
        total := total + elem;
    END LOOP;
    RETURN total;
END;
$$ LANGUAGE plpgsql;
```

### Loop Over Query Results

```sql
CREATE OR REPLACE FUNCTION update_all_prices(increase_percent DECIMAL)
RETURNS INT AS $$
DECLARE
    product_record RECORD;
    updated_count INT := 0;
BEGIN
    FOR product_record IN SELECT id, price FROM products LOOP
        UPDATE products
        SET price = product_record.price * (1 + increase_percent / 100)
        WHERE id = product_record.id;

        updated_count := updated_count + 1;
    END LOOP;

    RETURN updated_count;
END;
$$ LANGUAGE plpgsql;
```

---

## 4️⃣ Error Handling

```sql
CREATE OR REPLACE FUNCTION safe_divide(a DECIMAL, b DECIMAL)
RETURNS DECIMAL AS $$
BEGIN
    IF b = 0 THEN
        RAISE EXCEPTION 'Division by zero' USING ERRCODE = '22012';
    END IF;
    RETURN a / b;

EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'Attempted division by zero';
        RETURN NULL;
    WHEN OTHERS THEN
        RAISE NOTICE 'Error: %', SQLERRM;
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Custom exceptions
CREATE OR REPLACE FUNCTION validate_order(p_order_id INT)
RETURNS BOOLEAN AS $$
DECLARE
    v_total DECIMAL;
    v_status VARCHAR;
BEGIN
    SELECT total_amount, status INTO v_total, v_status
    FROM orders WHERE id = p_order_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'Order not found: %', p_order_id
            USING ERRCODE = 'P0001',
                  HINT = 'Check if order ID is correct';
    END IF;

    IF v_status = 'cancelled' THEN
        RAISE EXCEPTION 'Order is cancelled'
            USING ERRCODE = 'P0002';
    END IF;

    IF v_total <= 0 THEN
        RAISE WARNING 'Order has zero total amount';
        RETURN FALSE;
    END IF;

    RETURN TRUE;

EXCEPTION
    WHEN SQLSTATE 'P0001' THEN
        RAISE NOTICE 'Order validation failed: %', SQLERRM;
        RETURN FALSE;
    WHEN SQLSTATE 'P0002' THEN
        RAISE NOTICE 'Order is cancelled';
        RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

---

## 5️⃣ Triggers

```sql
-- Trigger function
CREATE OR REPLACE FUNCTION update_modified_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_timestamp();

-- Audit trigger
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    old_data JSONB,
    new_data JSONB,
    changed_by VARCHAR(50) DEFAULT current_user,
    changed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_data, new_data)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP = 'DELETE' OR TG_OP = 'UPDATE' THEN row_to_json(OLD)::jsonb END,
        CASE WHEN TG_OP = 'INSERT' OR TG_OP = 'UPDATE' THEN row_to_json(NEW)::jsonb END
    );

    IF TG_OP = 'DELETE' THEN
        RETURN OLD;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_users
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW
    EXECUTE FUNCTION audit_trigger();
```

### Constraint Trigger

```sql
-- Check stock before order
CREATE OR REPLACE FUNCTION check_stock()
RETURNS TRIGGER AS $$
DECLARE
    available_stock INT;
BEGIN
    SELECT stock INTO available_stock
    FROM products
    WHERE id = NEW.product_id;

    IF available_stock < NEW.quantity THEN
        RAISE EXCEPTION 'Insufficient stock. Available: %, Requested: %',
            available_stock, NEW.quantity;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_order_stock
    BEFORE INSERT ON order_items
    FOR EACH ROW
    EXECUTE FUNCTION check_stock();
```

---

## 6️⃣ SQL Functions (Simpler Syntax)

```sql
-- SQL function (without plpgsql)
CREATE OR REPLACE FUNCTION get_user_email(p_user_id INT)
RETURNS VARCHAR AS $$
    SELECT email FROM users WHERE id = p_user_id;
$$ LANGUAGE sql STABLE;

-- Inline SQL function (often faster)
CREATE OR REPLACE FUNCTION is_premium_user(p_user_id INT)
RETURNS BOOLEAN AS $$
    SELECT EXISTS (
        SELECT 1 FROM users
        WHERE id = p_user_id AND subscription_type = 'premium'
    );
$$ LANGUAGE sql STABLE;

-- Aggregate-like function
CREATE OR REPLACE FUNCTION user_order_stats(p_user_id INT)
RETURNS TABLE (total_orders BIGINT, total_spent DECIMAL) AS $$
    SELECT COUNT(*), COALESCE(SUM(total_amount), 0)
    FROM orders
    WHERE user_id = p_user_id AND status = 'completed';
$$ LANGUAGE sql STABLE;
```

---

## 📦 Real-World Examples

### User Registration

```sql
CREATE OR REPLACE FUNCTION register_user(
    p_username VARCHAR,
    p_email VARCHAR,
    p_password_hash VARCHAR
)
RETURNS INT AS $$
DECLARE
    new_user_id INT;
BEGIN
    -- Check if username exists
    IF EXISTS (SELECT 1 FROM users WHERE username = p_username) THEN
        RAISE EXCEPTION 'Username already exists: %', p_username
            USING ERRCODE = 'P0001';
    END IF;

    -- Check if email exists
    IF EXISTS (SELECT 1 FROM users WHERE email = p_email) THEN
        RAISE EXCEPTION 'Email already exists: %', p_email
            USING ERRCODE = 'P0002';
    END IF;

    -- Insert user
    INSERT INTO users (username, email, password_hash, is_active)
    VALUES (p_username, p_email, p_password_hash, false)
    RETURNING id INTO new_user_id;

    -- Create default settings
    INSERT INTO user_settings (user_id, settings)
    VALUES (new_user_id, '{"notifications": true, "theme": "light"}');

    -- Log event
    INSERT INTO events (event_type, payload)
    VALUES ('user_registered', jsonb_build_object('user_id', new_user_id));

    RETURN new_user_id;
END;
$$ LANGUAGE plpgsql;
```

### Create Order

```sql
CREATE OR REPLACE FUNCTION create_order(
    p_user_id INT,
    p_items JSONB  -- [{"product_id": 1, "quantity": 2}, ...]
)
RETURNS INT AS $$
DECLARE
    new_order_id INT;
    item JSONB;
    v_product_id INT;
    v_quantity INT;
    v_price DECIMAL;
    v_total DECIMAL := 0;
BEGIN
    -- Create order
    INSERT INTO orders (user_id, status, total_amount)
    VALUES (p_user_id, 'pending', 0)
    RETURNING id INTO new_order_id;

    -- Process items
    FOR item IN SELECT * FROM jsonb_array_elements(p_items)
    LOOP
        v_product_id := (item->>'product_id')::INT;
        v_quantity := (item->>'quantity')::INT;

        -- Get price and check stock
        SELECT price INTO v_price
        FROM products
        WHERE id = v_product_id AND stock >= v_quantity;

        IF NOT FOUND THEN
            RAISE EXCEPTION 'Product % not available or insufficient stock', v_product_id;
        END IF;

        -- Insert order item
        INSERT INTO order_items (order_id, product_id, quantity, price)
        VALUES (new_order_id, v_product_id, v_quantity, v_price);

        -- Update stock
        UPDATE products
        SET stock = stock - v_quantity
        WHERE id = v_product_id;

        v_total := v_total + (v_price * v_quantity);
    END LOOP;

    -- Update order total
    UPDATE orders SET total_amount = v_total WHERE id = new_order_id;

    RETURN new_order_id;

EXCEPTION WHEN OTHERS THEN
    -- Rollback is automatic in function
    RAISE;
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT create_order(1, '[
    {"product_id": 1, "quantity": 2},
    {"product_id": 3, "quantity": 1}
]');
```

---

## 🔧 Function Management

```sql
-- List all functions
SELECT
    proname as function_name,
    pg_get_function_arguments(oid) as arguments,
    pg_get_function_result(oid) as return_type
FROM pg_proc
WHERE pronamespace = 'public'::regnamespace;

-- Function definition
SELECT pg_get_functiondef('calculate_tax'::regproc);

-- Drop function
DROP FUNCTION IF EXISTS calculate_tax(DECIMAL);
DROP FUNCTION IF EXISTS get_user_orders(INT) CASCADE;

-- Permissions
GRANT EXECUTE ON FUNCTION calculate_tax(DECIMAL) TO app_user;
REVOKE EXECUTE ON FUNCTION create_order(INT, JSONB) FROM PUBLIC;
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│              Functions & Procedures Best Practices                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use IMMUTABLE/STABLE/VOLATILE correctly                         │
│     • IMMUTABLE: Same input = same output (can cache)               │
│     • STABLE: Same within transaction                               │
│     • VOLATILE: Can change anytime (default)                        │
│                                                                      │
│  2. Always handle errors                                             │
│     • Use EXCEPTION blocks                                           │
│     • Custom error codes for different cases                        │
│                                                                      │
│  3. Use SECURITY DEFINER carefully                                   │
│     • Runs with owner's permissions                                  │
│     • Set search_path explicitly                                     │
│                                                                      │
│  4. Avoid side effects in functions used in queries                 │
│     • PostgreSQL may call multiple times                            │
│     • Use procedures for side effects                               │
│                                                                      │
│  5. Prefer SQL functions when possible                               │
│     • Often inlined and optimized                                    │
│     • Use plpgsql only when needed                                  │
│                                                                      │
│  6. Document your functions                                          │
│     COMMENT ON FUNCTION func IS 'Description';                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Functions** ترجع قيمة وممكن في SELECT
2. **Procedures** للـ transaction control و CALL
3. **RETURNS TABLE** للـ queries
4. **Triggers** للـ automatic actions
5. **EXCEPTION** للـ error handling
6. **IMMUTABLE/STABLE** للـ optimization

</div>

---

## 🎉 نهاية Module 06

<div dir="rtl">

مبروك! خلصت Module 06: Advanced Topics. دلوقتي عندك معرفة متقدمة بـ:

- Index Types و EXPLAIN ANALYZE
- Query Optimization
- JSON/JSONB والـ operators والـ indexing
- Full-Text Search
- Views و Materialized Views
- Partitioning
- Stored Procedures و Functions

</div>

---

<div align="center">

[⬅️ السابق](./09-partitioning.md) | [🏠 العودة للـ Module](../README.md) | [📚 العودة للـ Track](../../README.md)

</div>
