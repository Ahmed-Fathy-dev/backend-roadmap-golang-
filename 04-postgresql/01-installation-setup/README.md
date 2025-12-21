# Module 4.1: Installation & Setup 🔧

<div dir="rtl">

## نظرة عامة

تثبيت PostgreSQL وإعداد البيئة للعمل.

</div>

---

## 🔧 Installing PostgreSQL

### Windows:

1. **Download:**
   - Go to [postgresql.org/download](https://www.postgresql.org/download/)
   - Download PostgreSQL 16+ installer
   - Run installer

2. **Installation Steps:**
   - Choose installation directory
   - Select components (PostgreSQL Server, pgAdmin, Command Line Tools)
   - Set port: **5432** (default)
   - Set password for **postgres** user (remember it!)

3. **Verify Installation:**
```powershell
psql --version
# Output: psql (PostgreSQL) 16.x
```

### Linux (Ubuntu/Debian):

```bash
# Update package list
sudo apt update

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Start service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verify
psql --version
```

---

## 💻 pgAdmin Setup

**pgAdmin** = GUI tool لإدارة PostgreSQL

1. Open pgAdmin (installed with PostgreSQL)
2. Create server connection:
   - Name: `Local PostgreSQL`
   - Host: `localhost`
   - Port: `5432`
   - Username: `postgres`
   - Password: (what you set during installation)

---

## 🎯 Creating Your First Database

### Using psql CLI:

```bash
# Connect to PostgreSQL
psql -U postgres

# Create database
CREATE DATABASE myapp;

# List databases
\l

# Connect to database
\c myapp

# Exit
\q
```

### Using pgAdmin:

1. Right-click **Databases**
2. Create → Database
3. Name: `myapp`
4. Save

---

## 📋 Essential psql Commands

```sql
-- List databases
\l

-- Connect to database
\c database_name

-- List tables
\dt

-- Describe table
\d table_name

-- List users
\du

-- Execute SQL file
\i /path/to/file.sql

-- Help
\?

-- Quit
\q
```

---

## 🔐 Creating Database User

```sql
-- Create user
CREATE USER myapp_user WITH PASSWORD 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE myapp TO myapp_user;

-- Grant table privileges
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myapp_user;
```

---

## 🎯 Your First Table

```sql
-- Connect to database
\c myapp

-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insert data
INSERT INTO users (name, email, age) 
VALUES ('Ahmed', 'ahmed@test.com', 25);

-- Query data
SELECT * FROM users;
```

**Output:**
```
 id |  name  |      email       | age |         created_at         
----+--------+------------------+-----+----------------------------
  1 | Ahmed  | ahmed@test.com   |  25 | 2024-12-21 20:00:00.123456
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ PostgreSQL installed
- [ ] ✅ psql command works
- [ ] ✅ pgAdmin installed & configured
- [ ] ✅ Created first database
- [ ] ✅ Created first table
- [ ] ✅ Can insert & query data

</div>

---

## 🔧 Connection String Format

```
postgresql://user:password@host:port/database

# Example:
postgresql://postgres:mypassword@localhost:5432/myapp
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 4.2: SQL Basics](../02-sql-basics/README.md)**

</div>

---

<div align="center">

[🏠 Track 4](../README.md) | [📚 Main](../../README.md)

</div>
