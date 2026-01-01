# Module 04: Joins & Relations - الربط والعلاقات 🔗

<div dir="rtl">

## نظرة عامة

JOINs هي القلب النابض لقواعد البيانات العلائقية - بتسمحلك تربط بيانات من جداول مختلفة. في هذا الـ Module هنتعلم كل أنواع الـ JOINs والعلاقات.

**المستوى:** متوسط
**المدة المتوقعة:** 4-5 ساعات

</div>

---

## 📚 فهرس الدروس

### 🔑 أساسيات العلاقات

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 01 | [Database Relationships](./lessons/01-relationships.md) | أنواع العلاقات: 1:1, 1:N, M:N | 25 دقيقة |
| 02 | [Primary & Foreign Keys](./lessons/02-keys.md) | المفاتيح الأساسية والخارجية | 20 دقيقة |
| 03 | [Referential Integrity](./lessons/03-referential-integrity.md) | سلامة المراجع و ON DELETE/UPDATE | 20 دقيقة |

### 🔗 أنواع الـ JOINs

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 04 | [INNER JOIN](./lessons/04-inner-join.md) | الربط الداخلي | 25 دقيقة |
| 05 | [LEFT & RIGHT JOIN](./lessons/05-outer-joins.md) | الربط الخارجي | 25 دقيقة |
| 06 | [FULL OUTER JOIN](./lessons/06-full-outer-join.md) | الربط الخارجي الكامل | 15 دقيقة |
| 07 | [CROSS JOIN](./lessons/07-cross-join.md) | الربط التقاطعي | 15 دقيقة |
| 08 | [Self JOIN](./lessons/08-self-join.md) | ربط الجدول بنفسه | 20 دقيقة |

### 🎯 تقنيات متقدمة

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 09 | [Multiple JOINs](./lessons/09-multiple-joins.md) | ربط أكتر من جدولين | 25 دقيقة |
| 10 | [JOIN Performance](./lessons/10-join-performance.md) | تحسين أداء الـ JOINs | 30 دقيقة |

---

## 🎯 أهداف الـ Module

<div dir="rtl">

بعد الانتهاء من هذا الـ Module، ستتمكن من:

1. **فهم العلاقات** بين الجداول (1:1, 1:N, M:N)
2. **تصميم** Primary و Foreign Keys صحيحة
3. **استخدام** كل أنواع JOINs
4. **تحسين أداء** الـ queries المعقدة
5. **حل مشاكل** الـ data integrity

</div>

---

## 📊 Joins Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Types of JOINs                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INNER JOIN                    LEFT JOIN                            │
│  ┌─────┐ ┌─────┐              ┌─────┐ ┌─────┐                       │
│  │  A  │∩│  B  │              │█████│∩│  B  │                       │
│  │     │█│     │              │█████│█│     │                       │
│  └─────┘ └─────┘              └─────┘ └─────┘                       │
│  التقاطع فقط                  كل A + التقاطع                        │
│                                                                      │
│  RIGHT JOIN                    FULL OUTER JOIN                      │
│  ┌─────┐ ┌─────┐              ┌─────┐ ┌─────┐                       │
│  │  A  │∩│█████│              │█████│∩│█████│                       │
│  │     │█│█████│              │█████│█│█████│                       │
│  └─────┘ └─────┘              └─────┘ └─────┘                       │
│  كل B + التقاطع               كل A + كل B                           │
│                                                                      │
│  CROSS JOIN                    SELF JOIN                            │
│  ┌─────┐ × ┌─────┐            ┌─────────────┐                       │
│  │  A  │   │  B  │            │   A    ↔    │                       │
│  │     │   │     │            │      (A)    │                       │
│  └─────┘   └─────┘            └─────────────┘                       │
│  كل تركيبة ممكنة              ربط الجدول بنفسه                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🗃️ البيانات التجريبية

```sql
-- جدول المستخدمين
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول الفئات
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id INT REFERENCES categories(id),
    description TEXT
);

-- جدول المنتجات
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    category_id INT REFERENCES categories(id),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول الطلبات
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    status VARCHAR(20) DEFAULT 'pending',
    total_amount NUMERIC(10, 2) DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول تفاصيل الطلبات
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id),
    quantity INT NOT NULL,
    unit_price NUMERIC(10, 2) NOT NULL
);

-- جدول التقييمات
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    product_id INT REFERENCES products(id),
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (user_id, product_id)  -- تقييم واحد لكل منتج لكل مستخدم
);

-- بيانات تجريبية
INSERT INTO users (username, email, full_name) VALUES
    ('ahmed', 'ahmed@example.com', 'Ahmed Ali'),
    ('sara', 'sara@example.com', 'Sara Mohamed'),
    ('omar', 'omar@example.com', 'Omar Hassan');

INSERT INTO categories (name, description) VALUES
    ('Electronics', 'Electronic devices'),
    ('Clothing', 'Fashion items'),
    ('Books', 'Books and publications');

INSERT INTO products (name, price, category_id) VALUES
    ('Laptop', 1299.99, 1),
    ('Smartphone', 799.99, 1),
    ('T-Shirt', 29.99, 2),
    ('Novel', 19.99, 3);
```

---

## 📖 المتطلبات السابقة

<div dir="rtl">

قبل البدء في هذا الـ Module، تأكد من إتمام:

</div>

- [x] [Module 01: Installation & Setup](../01-installation-setup/README.md)
- [x] [Module 02: SQL Basics](../02-sql-basics/README.md)
- [x] [Module 03: CRUD Operations](../03-crud-operations/README.md)

---

## 🚀 ابدأ التعلم

<div dir="rtl">

ابدأ بالدرس الأول:

</div>

**➡️ [Database Relationships](./lessons/01-relationships.md)**

---

## 📂 هيكل الـ Module

```
04-joins-relations/
├── README.md
├── lessons/
│   ├── 01-relationships.md
│   ├── 02-keys.md
│   ├── 03-referential-integrity.md
│   ├── 04-inner-join.md
│   ├── 05-outer-joins.md
│   ├── 06-full-outer-join.md
│   ├── 07-cross-join.md
│   ├── 08-self-join.md
│   ├── 09-multiple-joins.md
│   └── 10-join-performance.md
├── examples/
│   ├── 01-e-commerce-schema.md
│   └── 02-blog-system.md
└── resources/
    └── joins-cheatsheet.md
```

---

<div align="center">

[⬅️ Module السابق: CRUD Operations](../03-crud-operations/README.md) | [🏠 الرئيسية](../README.md) | [Module التالي: Go Integration ➡️](../05-go-postgres-integration/README.md)

</div>
