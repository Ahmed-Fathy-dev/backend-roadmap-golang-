# Module 4.2: SQL Basics 📝

<div dir="rtl">

## نظرة عامة

أساسيات SQL - Data Types, Creating Tables, Basic Queries.

</div>

---

## 📊 Data Types

```sql
-- Numbers
INT, BIGINT, SMALLINT          -- Integers
DECIMAL(10,2), NUMERIC(10,2)   -- Exact numbers
REAL, DOUBLE PRECISION         -- Floating point

-- Strings
VARCHAR(100)                    -- Variable length
CHAR(10)                        -- Fixed length
TEXT                            -- Unlimited length

-- Date & Time
DATE                            -- 2024-12-21
TIME                            -- 20:30:00
TIMESTAMP                       -- 2024-12-21 20:30:00
TIMESTAMPTZ                     -- With timezone

-- Boolean
BOOLEAN                         -- TRUE/FALSE

-- JSON
JSON, JSONB                     -- JSON data

-- Special
SERIAL, BIGSERIAL              -- Auto-increment
UUID                            -- Universal ID
```

---

## 🏗️ Creating Tables

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## ➕ INSERT Data

```sql
-- Single insert
INSERT INTO products (name, price, stock)
VALUES ('Laptop', 999.99, 10);

-- Multiple inserts
INSERT INTO products (name, price, stock) VALUES
    ('Mouse', 19.99, 50),
    ('Keyboard', 49.99, 30),
    ('Monitor', 299.99, 15);

-- Return inserted data
INSERT INTO products (name, price)
VALUES ('Headphones', 79.99)
RETURNING *;
```

---

## 🔍 SELECT Data

```sql
-- All columns
SELECT * FROM products;

-- Specific columns
SELECT name, price FROM products;

-- WHERE clause
SELECT * FROM products WHERE price > 50;

-- Multiple conditions
SELECT * FROM products 
WHERE price > 50 AND stock > 10;

-- OR condition
SELECT * FROM products 
WHERE name = 'Laptop' OR name = 'Monitor';

-- LIKE (pattern matching)
SELECT * FROM products WHERE name LIKE '%top%';

-- IN operator
SELECT * FROM products WHERE name IN ('Laptop', 'Mouse');

-- BETWEEN
SELECT * FROM products WHERE price BETWEEN 20 AND 100;
```

---

## 📊 ORDER BY & LIMIT

```sql
-- Order ascending
SELECT * FROM products ORDER BY price;

-- Order descending
SELECT * FROM products ORDER BY price DESC;

-- Multiple columns
SELECT * FROM products ORDER BY price DESC, name ASC;

-- LIMIT
SELECT * FROM products LIMIT 5;

-- OFFSET (skip first 5)
SELECT * FROM products LIMIT 5 OFFSET 5;

-- Pagination (page 2, 10 items per page)
SELECT * FROM products 
LIMIT 10 OFFSET 10;
```

---

## 🎯 Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) FROM products;
SELECT COUNT(*) FROM products WHERE price > 100;

-- SUM
SELECT SUM(stock) FROM products;

-- AVG
SELECT AVG(price) FROM products;

-- MIN & MAX
SELECT MIN(price), MAX(price) FROM products;

-- GROUP BY
SELECT is_active, COUNT(*) 
FROM products 
GROUP BY is_active;
```

---

## 🔧 Complete Example

```sql
-- Create students table
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    grade INT CHECK (grade >= 0 AND grade <= 100),
    enrolled_at TIMESTAMP DEFAULT NOW()
);

-- Insert students
INSERT INTO students (name, email, grade) VALUES
    ('Ahmed Ali', 'ahmed@test.com', 85),
    ('Sara Mohamed', 'sara@test.com', 92),
    ('Omar Hassan', 'omar@test.com', 78),
    ('Fatima Ahmed', 'fatima@test.com', 95),
    ('Youssef Ibrahim', 'youssef@test.com', 88);

-- Get all students
SELECT * FROM students;

-- Top 3 students
SELECT name, grade 
FROM students 
ORDER BY grade DESC 
LIMIT 3;

-- Average grade
SELECT AVG(grade) as average_grade FROM students;

-- Students with grade > 85
SELECT name, grade 
FROM students 
WHERE grade > 85
ORDER BY grade DESC;
```

---

## ✅ Best Practices

```sql
-- ✅ Always use NOT NULL for required fields
CREATE TABLE users (
    email VARCHAR(100) NOT NULL
);

-- ✅ Use UNIQUE for unique fields
CREATE TABLE users (
    email VARCHAR(100) UNIQUE NOT NULL
);

-- ✅ Add CHECK constraints
CREATE TABLE products (
    price DECIMAL(10,2) CHECK (price > 0)
);

-- ✅ Use DEFAULT values
CREATE TABLE posts (
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 4.3: CRUD Operations](../03-crud-operations/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Installation](../01-installation-setup/README.md) | [🏠 Track 4](../README.md)

</div>
