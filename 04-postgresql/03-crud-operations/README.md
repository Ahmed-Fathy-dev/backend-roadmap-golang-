# Module 4.3: CRUD Operations 🔄

<div dir="rtl">

## نظرة عامة

تفاصيل عمليات **CRUD** - Create, Read, Update, Delete.

</div>

---

## ➕ CREATE (INSERT)

### Basic INSERT

```sql
-- Single insert
INSERT INTO products (name, price, stock)
VALUES ('Laptop', 999.99, 10);

-- Multiple rows
INSERT INTO products (name, price, stock) VALUES
    ('Mouse', 19.99, 50),
    ('Keyboard', 49.99, 30),
    ('Monitor', 299.99, 15);

-- RETURNING (get inserted data)
INSERT INTO products (name, price)
VALUES ('Headphones', 79.99)
RETURNING *;
```

### INSERT with DEFAULT values

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    views INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Uses DEFAULT for views and created_at
INSERT INTO posts (title, content)
VALUES ('My First Post', 'Hello World');
```

---

## 📖 READ (SELECT)

### Basic SELECT

```sql
-- All columns
SELECT * FROM products;

-- Specific columns
SELECT name, price FROM products;

-- DISTINCT (unique values)
SELECT DISTINCT category FROM products;
```

### WHERE Clause

```sql
-- Equality
SELECT * FROM products WHERE price = 99.99;

-- Comparison operators
SELECT * FROM products WHERE price > 50;
SELECT * FROM products WHERE stock <= 10;

-- Multiple conditions (AND)
SELECT * FROM products WHERE price > 50 AND stock > 0;

-- OR condition
SELECT * FROM products WHERE category = 'Electronics' OR category = 'Computers';

-- BETWEEN
SELECT * FROM products WHERE price BETWEEN 50 AND 200;

-- IN operator
SELECT * FROM products WHERE id IN (1, 3, 5, 7);

-- LIKE (pattern matching)
SELECT * FROM products WHERE name LIKE '%Laptop%';
SELECT * FROM products WHERE name ILIKE '%laptop%';  -- Case insensitive

-- IS NULL / IS NOT NULL
SELECT * FROM products WHERE description IS NULL;
```

### ORDER BY

```sql
-- Ascending (default)
SELECT * FROM products ORDER BY price;

-- Descending
SELECT * FROM products ORDER BY price DESC;

-- Multiple columns
SELECT * FROM products ORDER BY category ASC, price DESC;
```

### LIMIT & OFFSET

```sql
-- First 10 products
SELECT * FROM products LIMIT 10;

-- Skip first 10, get next 10 (pagination)
SELECT * FROM products LIMIT 10 OFFSET 10;

-- Page 3 (20 items per page)
SELECT * FROM products LIMIT 20 OFFSET 40;
```

### Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) FROM products;
SELECT COUNT(DISTINCT category) FROM products;

-- SUM
SELECT SUM(stock) FROM products;

-- AVG
SELECT AVG(price) FROM products;

-- MIN & MAX
SELECT MIN(price), MAX(price) FROM products;

-- GROUP BY
SELECT category, COUNT(*), AVG(price)
FROM products
GROUP BY category;

-- HAVING (filter groups)
SELECT category, AVG(price) as avg_price
FROM products
GROUP BY category
HAVING AVG(price) > 100;
```

---

## 🔄 UPDATE

### Basic UPDATE

```sql
-- Update single row
UPDATE products
SET price = 89.99
WHERE id = 1;

-- Update multiple columns
UPDATE products
SET price = 99.99, stock = 20
WHERE id = 2;

-- Update multiple rows
UPDATE products
SET stock = 0
WHERE price > 1000;

-- RETURNING updated data
UPDATE products
SET price = price * 0.9
WHERE category = 'Electronics'
RETURNING *;
```

### Conditional UPDATE

```sql
-- Increment value
UPDATE products
SET views = views + 1
WHERE id = 5;

-- Conditional update
UPDATE products
SET status = 'Out of Stock'
WHERE stock = 0;

-- Update based on calculation
UPDATE products
SET discount_price = price * 0.8
WHERE category = 'Clearance';
```

---

## ❌ DELETE

### Basic DELETE

```sql
-- Delete specific row
DELETE FROM products WHERE id = 10;

-- Delete multiple rows
DELETE FROM products WHERE price < 10;

-- Delete all (careful!)
DELETE FROM products;

-- RETURNING deleted data
DELETE FROM products
WHERE stock = 0
RETURNING *;
```

### Safer DELETE

```sql
-- Always use WHERE!
-- ❌ Dangerous:
DELETE FROM products;

-- ✅ Safe:
DELETE FROM products WHERE id = 5;

-- Check before deleting
SELECT * FROM products WHERE stock = 0;  -- Preview
DELETE FROM products WHERE stock = 0;     -- Then delete
```

---

## 🔄 Transactions

### Basic Transaction

```sql
BEGIN;

-- Step 1: Deduct from sender
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;

-- Step 2: Add to receiver
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

COMMIT;  -- Save changes

-- OR if error:
ROLLBACK;  -- Cancel all changes
```

### Transaction Example

```sql
-- Transfer money between accounts
BEGIN;

-- Check sender balance
SELECT balance FROM accounts WHERE id = 1;
-- If balance >= 1000, continue

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- If all successful:
COMMIT;

-- If any error occurs:
ROLLBACK;
```

---

## 🎯 Complete Example

```sql
-- Create orders table
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    total_price DECIMAL(10,2),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

-- CREATE: Insert order
INSERT INTO orders (customer_name, product_id, quantity, total_price)
VALUES ('Ahmed Ali', 1, 2, 199.98)
RETURNING *;

-- READ: Get all orders
SELECT * FROM orders;

-- READ: Get specific order
SELECT * FROM orders WHERE id = 1;

-- READ: Get pending orders
SELECT * FROM orders WHERE status = 'pending'
ORDER BY created_at DESC;

-- UPDATE: Mark order as completed
UPDATE orders
SET status = 'completed'
WHERE id = 1
RETURNING *;

-- UPDATE: Cancel old pending orders
UPDATE orders
SET status = 'cancelled'
WHERE status = 'pending'
AND created_at < NOW() - INTERVAL '7 days';

-- DELETE: Remove cancelled orders
DELETE FROM orders
WHERE status = 'cancelled'
AND created_at < NOW() - INTERVAL '30 days';
```

---

## ✅ Best Practices

```sql
-- ✅ Always use WHERE in UPDATE/DELETE
UPDATE products SET price = 99 WHERE id = 5;  -- Good
-- UPDATE products SET price = 99;  -- Bad! Updates ALL rows

-- ✅ Use transactions for multiple operations
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- ✅ Use RETURNING to see changes
UPDATE products SET stock = stock - 1 WHERE id = 5 RETURNING *;

-- ✅ Check data before deleting
SELECT * FROM products WHERE stock = 0;  -- Preview first
-- DELETE FROM products WHERE stock = 0;  -- Then delete

-- ✅ Use soft delete for important data
ALTER TABLE products ADD COLUMN deleted_at TIMESTAMP;
UPDATE products SET deleted_at = NOW() WHERE id = 5;  -- Soft delete
SELECT * FROM products WHERE deleted_at IS NULL;      -- Active only
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 4.4: Joins & Relations](../04-joins-relations/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: SQL Basics](../02-sql-basics/README.md) | [🏠 Track 4](../README.md)

</div>
