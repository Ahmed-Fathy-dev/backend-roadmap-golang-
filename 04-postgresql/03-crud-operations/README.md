# Module 03: CRUD Operations - عمليات CRUD 🔄

<div dir="rtl">

## نظرة عامة

CRUD هو اختصار لـ Create, Read, Update, Delete - العمليات الأساسية الأربعة للتعامل مع البيانات في أي قاعدة بيانات. في هذا الـ Module هنتعلم كل تفاصيل هذه العمليات بعمق.

**المستوى:** مبتدئ - متوسط
**المدة المتوقعة:** 4-5 ساعات

</div>

---

## 📚 فهرس الدروس

### 📥 Create (INSERT)

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 01 | [INSERT الأساسي](./lessons/01-basic-insert.md) | إدخال صف واحد وعدة صفوف | 20 دقيقة |
| 02 | [INSERT ... RETURNING](./lessons/02-insert-returning.md) | إرجاع البيانات بعد الإدخال | 15 دقيقة |
| 03 | [INSERT ... ON CONFLICT](./lessons/03-upsert.md) | Upsert: Update or Insert | 25 دقيقة |
| 04 | [INSERT ... SELECT](./lessons/04-insert-select.md) | الإدخال من Query آخر | 20 دقيقة |
| 05 | [COPY Command](./lessons/05-copy-command.md) | استيراد وتصدير البيانات | 25 دقيقة |

### 📖 Read (SELECT)

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 06 | [SELECT المتقدم](./lessons/06-advanced-select.md) | Subqueries و CTEs | 30 دقيقة |
| 07 | [Window Functions](./lessons/07-window-functions.md) | ROW_NUMBER, RANK, LAG, LEAD | 35 دقيقة |

### ✏️ Update (UPDATE)

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 08 | [UPDATE الأساسي](./lessons/08-basic-update.md) | تحديث البيانات | 20 دقيقة |
| 09 | [UPDATE المتقدم](./lessons/09-advanced-update.md) | UPDATE من JOIN و Subquery | 25 دقيقة |

### 🗑️ Delete (DELETE)

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 10 | [DELETE الأساسي](./lessons/10-basic-delete.md) | حذف البيانات | 15 دقيقة |
| 11 | [TRUNCATE و Soft Delete](./lessons/11-truncate-soft-delete.md) | طرق الحذف المختلفة | 20 دقيقة |

### 🔄 Transactions

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 12 | [Transactions](./lessons/12-transactions.md) | BEGIN, COMMIT, ROLLBACK | 25 دقيقة |
| 13 | [Isolation Levels](./lessons/13-isolation-levels.md) | مستويات العزل | 25 دقيقة |

---

## 🎯 أهداف الـ Module

<div dir="rtl">

بعد الانتهاء من هذا الـ Module، ستتمكن من:

1. **إدخال البيانات** بكفاءة مع التعامل مع conflicts
2. **قراءة البيانات** باستخدام queries متقدمة
3. **تحديث البيانات** بطرق مختلفة وآمنة
4. **حذف البيانات** مع فهم الفرق بين الطرق المختلفة
5. **استخدام Transactions** لضمان سلامة البيانات
6. **تطبيق Window Functions** للتحليلات المتقدمة

</div>

---

## 📊 CRUD Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CRUD Operations                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐ │
│   │  CREATE   │    │   READ    │    │  UPDATE   │    │  DELETE   │ │
│   │           │    │           │    │           │    │           │ │
│   │  INSERT   │    │  SELECT   │    │  UPDATE   │    │  DELETE   │ │
│   │           │    │           │    │           │    │           │ │
│   │  ➕ إنشاء  │    │  🔍 قراءة │    │  ✏️ تعديل │    │  🗑️ حذف  │ │
│   └───────────┘    └───────────┘    └───────────┘    └───────────┘ │
│                                                                      │
│   إضافة بيانات     استعلام عن       تحديث بيانات     إزالة بيانات  │
│   جديدة للجدول     البيانات          موجودة           من الجدول     │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                          Transactions                                │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  BEGIN → [Operations] → COMMIT / ROLLBACK                   │   │
│   │                                                             │   │
│   │  ضمان أن كل العمليات تنجح معاً أو تفشل معاً (ACID)         │   │
│   └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🗃️ البيانات التجريبية

```sql
-- سنستخدم هذه الجداول في كل الأمثلة

-- جدول المستخدمين
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    full_name VARCHAR(100),
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    role VARCHAR(20) DEFAULT 'user',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول المنتجات
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    is_available BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول الطلبات
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    status VARCHAR(20) DEFAULT 'pending',
    total_amount NUMERIC(10, 2) DEFAULT 0,
    shipping_address TEXT,
    notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول تفاصيل الطلبات
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id),
    quantity INT NOT NULL,
    unit_price NUMERIC(10, 2) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- بيانات تجريبية
INSERT INTO users (username, email, full_name, password_hash, role) VALUES
    ('ahmed', 'ahmed@example.com', 'Ahmed Ali', 'hash1', 'admin'),
    ('sara', 'sara@example.com', 'Sara Mohamed', 'hash2', 'user'),
    ('omar', 'omar@example.com', 'Omar Hassan', 'hash3', 'user');

INSERT INTO products (sku, name, price, stock, category_id) VALUES
    ('LAPTOP001', 'MacBook Pro 14"', 1999.99, 50, 1),
    ('PHONE001', 'iPhone 15 Pro', 999.99, 100, 2),
    ('WATCH001', 'Apple Watch Series 9', 399.99, 75, 3);
```

---

## 📖 المتطلبات السابقة

<div dir="rtl">

قبل البدء في هذا الـ Module، تأكد من إتمام:

</div>

- [x] [Module 01: Installation & Setup](../01-installation-setup/README.md)
- [x] [Module 02: SQL Basics](../02-sql-basics/README.md)

---

## 🚀 ابدأ التعلم

<div dir="rtl">

ابدأ بالدرس الأول:

</div>

**➡️ [INSERT الأساسي](./lessons/01-basic-insert.md)**

---

## 📂 هيكل الـ Module

```
03-crud-operations/
├── README.md
├── lessons/
│   ├── 01-basic-insert.md
│   ├── 02-insert-returning.md
│   ├── 03-upsert.md
│   ├── 04-insert-select.md
│   ├── 05-copy-command.md
│   ├── 06-advanced-select.md
│   ├── 07-window-functions.md
│   ├── 08-basic-update.md
│   ├── 09-advanced-update.md
│   ├── 10-basic-delete.md
│   ├── 11-truncate-soft-delete.md
│   ├── 12-transactions.md
│   └── 13-isolation-levels.md
├── examples/
│   ├── 01-e-commerce-crud.md
│   └── 02-inventory-system.md
└── resources/
    ├── crud-cheatsheet.md
    └── performance-tips.md
```

---

<div align="center">

[⬅️ Module السابق: SQL Basics](../02-sql-basics/README.md) | [🏠 الرئيسية](../README.md) | [Module التالي: Joins ➡️](../04-joins-relations/README.md)

</div>
