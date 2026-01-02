# Track 4: PostgreSQL & Databases 🐘

<div dir="rtl">

## نظرة عامة

في هذا Track، هتتعلم PostgreSQL من الصفر للاحتراف! من التثبيت والـ SQL الأساسي لحد الـ Advanced Topics زي الـ Partitioning والـ Full-Text Search.

**المدة المتوقعة:** 4-5 أسابيع

</div>

---

## 📊 Track Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Learning Path                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Module 1: Installation & Setup                                      │
│  ├── PostgreSQL Installation                                         │
│  ├── pgAdmin & psql CLI                                              │
│  └── Database Creation & Configuration                               │
│                                                                      │
│  Module 2: SQL Basics                                                │
│  ├── Data Types & Table Creation                                     │
│  ├── SELECT, WHERE, ORDER BY                                         │
│  └── Aggregate Functions & Grouping                                  │
│                                                                      │
│  Module 3: CRUD Operations                                           │
│  ├── INSERT, UPDATE, DELETE                                          │
│  ├── Subqueries & CTEs                                               │
│  └── Transactions & ACID                                             │
│                                                                      │
│  Module 4: Joins & Relations                                         │
│  ├── Primary & Foreign Keys                                          │
│  ├── All JOIN Types                                                  │
│  └── Indexes & Constraints                                           │
│                                                                      │
│  Module 5: Go + PostgreSQL                                           │
│  ├── database/sql & pgx                                              │
│  ├── GORM ORM                                                        │
│  └── Migrations & Best Practices                                     │
│                                                                      │
│  Module 6: Advanced Topics                                           │
│  ├── Query Optimization & EXPLAIN                                    │
│  ├── JSON/JSONB                                                      │
│  └── Partitioning & Stored Procedures                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Modules

### Module 4.1: Installation & Setup 🔧

<div dir="rtl">

**10 دروس شاملة:**

- تثبيت PostgreSQL (Windows, macOS, Linux, Docker)
- إعداد pgAdmin 4
- استخدام psql CLI
- إنشاء Databases و Users
- الـ Configuration الأساسي
- الـ Backup و Restore

**[➡️ Start Module 4.1](./01-installation-setup/README.md)**

</div>

---

### Module 4.2: SQL Basics 📝

<div dir="rtl">

**14 درس من الصفر:**

- مقدمة SQL و أنواعها
- Data Types (Numeric, String, Date, etc.)
- إنشاء وتعديل الجداول
- SELECT و الـ Expressions
- WHERE و Operators
- ORDER BY و LIMIT و OFFSET
- DISTINCT و IN و BETWEEN
- NULL Handling
- Aggregate Functions
- GROUP BY و HAVING
- String و Date Functions
- Type Casting
- CASE Expressions

**[➡️ Start Module 4.2](./02-sql-basics/README.md)**

</div>

---

### Module 4.3: CRUD Operations 🔄

<div dir="rtl">

**13 درس عملي:**

- INSERT (Single & Batch)
- RETURNING Clause
- SELECT المتقدم
- UPDATE (Simple & Complex)
- DELETE و TRUNCATE
- UPSERT (INSERT ON CONFLICT)
- Subqueries
- CTEs (Common Table Expressions)
- Transactions و ACID
- Savepoints
- Locking و Concurrency
- تمارين عملية

**[➡️ Start Module 4.3](./03-crud-operations/README.md)**

</div>

---

### Module 4.4: Joins & Relations 🔗

<div dir="rtl">

**10 دروس:**

- Database Design Basics
- Primary و Foreign Keys
- Relationships (1:1, 1:N, N:N)
- INNER JOIN
- LEFT/RIGHT/FULL OUTER JOIN
- CROSS JOIN
- Self Joins
- Multiple Joins
- Indexes
- Constraints (UNIQUE, CHECK, etc.)

**[➡️ Start Module 4.4](./04-joins-relations/README.md)**

</div>

---

### Module 4.5: Go + PostgreSQL Integration 🔌

<div dir="rtl">

**15 درس للربط مع Go:**

- database/sql Package
- pgx Driver (Native)
- Connection Pooling
- Basic CRUD Operations
- Prepared Statements
- Transactions في Go
- GORM Basics
- GORM Models و Tags
- GORM Relations
- GORM Advanced Features
- Migrations Concepts
- golang-migrate Tool
- Repository Pattern
- Error Handling
- Testing Database Code

**[➡️ Start Module 4.5](./05-go-postgres-integration/README.md)**

</div>

---

### Module 4.6: Advanced Topics 🚀

<div dir="rtl">

**10 دروس متقدمة:**

- Index Types (B-tree, Hash, GIN, GiST, BRIN)
- EXPLAIN ANALYZE
- Query Optimization
- JSON/JSONB Basics
- JSONB Operators
- JSONB Indexing
- Full-Text Search
- Views و Materialized Views
- Table Partitioning
- Stored Procedures و Functions

**[➡️ Start Module 4.6](./06-advanced-topics/README.md)**

</div>

---

## 🎯 What You'll Learn

<div dir="rtl">

بعد إنهاء هذا Track، هتكون قادر على:

### المهارات الأساسية
- ✅ تثبيت وإدارة PostgreSQL
- ✅ كتابة SQL Queries من البسيط للمعقد
- ✅ تصميم Database schemas
- ✅ فهم الـ Relations و الـ Joins
- ✅ استخدام Transactions بشكل صحيح

### المهارات المتقدمة
- ✅ ربط PostgreSQL مع Go
- ✅ استخدام GORM ORM
- ✅ كتابة Migrations
- ✅ تحسين الـ Performance
- ✅ التعامل مع JSON/JSONB
- ✅ Full-Text Search
- ✅ Partitioning للـ Big Data
- ✅ كتابة Stored Procedures

</div>

---

## 🛠️ What You'll Build

<div dir="rtl">

خلال الـ Track ده، هتبني مشاريع حقيقية:

1. **User Management System**
   - CRUD operations
   - Authentication data
   - Role-based access

2. **Blog Platform Database**
   - Posts, Comments, Tags
   - Full-text search
   - User relationships

3. **E-commerce Database**
   - Products, Categories
   - Orders, Order Items
   - Customer management
   - Inventory tracking

4. **Analytics System**
   - Event logging
   - Time-series data
   - Partitioned tables
   - Dashboard queries

</div>

---

## 📖 Prerequisites

<div dir="rtl">

**يجب إنهاء:**

- ✅ Track 1 (Backend Fundamentals)
- ✅ Track 2 (Go Basics)
- ✅ Track 3 (Go Advanced)

**المتطلبات التقنية:**

- Computer مع 4GB RAM على الأقل
- مساحة تخزين 2GB للـ PostgreSQL
- Go 1.21+ مثبت
- Terminal/Command Line access

</div>

---

## ⏱️ Estimated Time

<div dir="rtl">

| Module | المدة | الدروس |
|--------|-------|--------|
| Installation & Setup | 2-3 أيام | 10 |
| SQL Basics | 4-5 أيام | 14 |
| CRUD Operations | 3-4 أيام | 13 |
| Joins & Relations | 3-4 أيام | 10 |
| Go + PostgreSQL | 5-6 أيام | 15 |
| Advanced Topics | 4-5 أيام | 10 |
| **المجموع** | **4-5 أسابيع** | **72 درس** |

</div>

---

## 🚀 Getting Started

<div dir="rtl">

**جاهز للبدء؟**

1. تأكد من إنهاء الـ Prerequisites
2. جهز بيئة العمل (Terminal, Editor)
3. ابدأ بـ Module 4.1

**➡️ [Module 4.1: Installation & Setup](./01-installation-setup/README.md)**

</div>

---

## 💡 Tips for Success

<div dir="rtl">

1. **اكتب كل الـ queries بنفسك** - لا تعتمد على copy/paste
2. **جرب variations** - غير في الـ queries وشوف النتايج
3. **استخدم EXPLAIN** - افهم إزاي PostgreSQL بينفذ الـ queries
4. **اعمل مشاريع جانبية** - طبق اللي بتتعلمه
5. **راجع الـ documentation** - PostgreSQL docs ممتازة

</div>

---

<div align="center">

## 📚 Module Navigation

| Module | الوصف | الحالة |
|--------|-------|--------|
| [4.1 Installation](./01-installation-setup/README.md) | التثبيت والإعداد | ✅ |
| [4.2 SQL Basics](./02-sql-basics/README.md) | أساسيات SQL | ✅ |
| [4.3 CRUD](./03-crud-operations/README.md) | عمليات CRUD | ✅ |
| [4.4 Joins](./04-joins-relations/README.md) | الـ Joins والعلاقات | ✅ |
| [4.5 Go Integration](./05-go-postgres-integration/README.md) | الربط مع Go | ✅ |
| [4.6 Advanced](./06-advanced-topics/README.md) | مواضيع متقدمة | ✅ |

</div>

---

<div align="center">

[⬅️ Back: Track 3](../03-go-advanced/README.md) | [🏠 Main](../README.md) | [Track 5 ➡️](../05-rest-api/README.md)

</div>
