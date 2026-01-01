# Database Relationships - أنواع العلاقات 🔗

<div dir="rtl">

## مقدمة

العلاقات هي أساس قواعد البيانات العلائقية (Relational Databases). بتحدد كيف الجداول مرتبطة ببعض.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 أنواع العلاقات

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Database Relationships                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  One-to-One (1:1)                                                   │
│  ┌─────────┐         ┌─────────────┐                               │
│  │  User   │─────────│   Profile   │                               │
│  └─────────┘         └─────────────┘                               │
│  كل user له profile واحد فقط                                       │
│                                                                      │
│  One-to-Many (1:N)                                                  │
│  ┌─────────┐         ┌─────────────┐                               │
│  │  User   │────┬────│   Order 1   │                               │
│  └─────────┘    ├────│   Order 2   │                               │
│                  └────│   Order 3   │                               │
│                       └─────────────┘                               │
│  كل user له طلبات كتير                                              │
│                                                                      │
│  Many-to-Many (M:N)                                                 │
│  ┌─────────┐         ┌─────────────┐         ┌─────────┐           │
│  │Student 1│─────┬───│Enrollment   │───┬─────│Course 1 │           │
│  │Student 2│─────┼───│  (Junction) │───┼─────│Course 2 │           │
│  │Student 3│─────┴───│    Table    │───┴─────│Course 3 │           │
│  └─────────┘         └─────────────┘         └─────────┘           │
│  طلاب كتير في كورسات كتير                                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 One-to-One (1:1)

<div dir="rtl">

### متى تُستخدم؟

- بيانات اختيارية أو نادرة الاستخدام
- فصل البيانات الحساسة
- تحسين الأداء (جدول أصغر = أسرع)

</div>

```sql
-- جدول المستخدمين (الأساسي)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- جدول الـ Profile (1:1)
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INT UNIQUE NOT NULL,  -- UNIQUE يضمن 1:1
    bio TEXT,
    avatar_url VARCHAR(500),
    date_of_birth DATE,
    phone VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- أو الـ Foreign Key هو نفسه الـ Primary Key
CREATE TABLE user_settings (
    user_id INT PRIMARY KEY,  -- هو الـ FK و PK في نفس الوقت
    theme VARCHAR(20) DEFAULT 'light',
    language VARCHAR(10) DEFAULT 'ar',
    notifications_enabled BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Query
SELECT
    u.username,
    u.email,
    p.bio,
    p.avatar_url
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id
WHERE u.id = 1;
```

---

## 📚 One-to-Many (1:N)

<div dir="rtl">

### أكثر العلاقات شيوعاً

</div>

```sql
-- جدول الفئات (الـ One)
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

-- جدول المنتجات (الـ Many)
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    category_id INT,  -- كل منتج في فئة واحدة
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- مثال: User و Orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    total_amount NUMERIC(10, 2),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Query: كل طلبات المستخدم
SELECT
    u.username,
    o.id AS order_id,
    o.total_amount,
    o.status
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.id = 1;

-- Query: عدد المنتجات في كل فئة
SELECT
    c.name AS category,
    COUNT(p.id) AS product_count
FROM categories c
LEFT JOIN products p ON c.id = p.category_id
GROUP BY c.id, c.name;
```

---

## 🔄 Many-to-Many (M:N)

<div dir="rtl">

### تحتاج Junction Table (جدول وسيط)

</div>

```sql
-- جدول الطلاب
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE
);

-- جدول الكورسات
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    credits INT
);

-- Junction Table (جدول الربط)
CREATE TABLE enrollments (
    id SERIAL PRIMARY KEY,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at TIMESTAMPTZ DEFAULT NOW(),
    grade CHAR(2),
    -- Composite unique: طالب ما ينفعش يسجل في نفس الكورس مرتين
    UNIQUE (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);

-- تسجيل طالب في كورس
INSERT INTO enrollments (student_id, course_id)
VALUES (1, 2);

-- كورسات الطالب
SELECT
    s.name AS student,
    c.name AS course,
    e.enrolled_at,
    e.grade
FROM students s
JOIN enrollments e ON s.id = e.student_id
JOIN courses c ON e.course_id = c.id
WHERE s.id = 1;

-- طلاب الكورس
SELECT
    c.name AS course,
    s.name AS student,
    e.grade
FROM courses c
JOIN enrollments e ON c.id = e.course_id
JOIN students s ON e.student_id = s.id
WHERE c.id = 1;
```

<div dir="rtl">

### مثال آخر: Products و Tags

</div>

```sql
-- جدول Tags
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- Junction Table
CREATE TABLE product_tags (
    product_id INT NOT NULL,
    tag_id INT NOT NULL,
    PRIMARY KEY (product_id, tag_id),  -- Composite Primary Key
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);

-- إضافة tags لمنتج
INSERT INTO product_tags (product_id, tag_id) VALUES
    (1, 1), (1, 2), (1, 3);

-- منتجات بـ tag معين
SELECT p.*
FROM products p
JOIN product_tags pt ON p.id = pt.product_id
JOIN tags t ON pt.tag_id = t.id
WHERE t.name = 'electronics';

-- Tags لمنتج معين
SELECT t.name
FROM tags t
JOIN product_tags pt ON t.id = pt.tag_id
WHERE pt.product_id = 1;
```

---

## 🌳 Self-Referencing Relationship

<div dir="rtl">

### جدول مرتبط بنفسه

</div>

```sql
-- الموظفين مع المدير
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    manager_id INT,  -- مرجع لنفس الجدول
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);

-- Categories هرمية
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES categories(id)
);

-- الموظفين ومديريهم
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Subcategories
SELECT
    c.name AS category,
    p.name AS parent_category
FROM categories c
LEFT JOIN categories p ON c.parent_id = p.id;
```

---

## 📊 ER Diagram Example

```
┌─────────────────────────────────────────────────────────────────────┐
│                    E-Commerce ER Diagram                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐         ┌──────────────┐                         │
│  │    Users     │         │   Products   │                         │
│  ├──────────────┤         ├──────────────┤                         │
│  │ id (PK)      │         │ id (PK)      │                         │
│  │ username     │         │ name         │                         │
│  │ email        │         │ price        │                         │
│  └──────────────┘         │ category_id  │──┐                      │
│         │                 └──────────────┘  │                      │
│         │ 1:N                    │          │                      │
│         ▼                        │          │ N:1                  │
│  ┌──────────────┐               │          ▼                      │
│  │    Orders    │               │   ┌──────────────┐              │
│  ├──────────────┤               │   │  Categories  │              │
│  │ id (PK)      │               │   ├──────────────┤              │
│  │ user_id (FK) │               │   │ id (PK)      │              │
│  │ total_amount │               │   │ name         │              │
│  └──────────────┘               │   │ parent_id    │──┐ Self      │
│         │                       │   └──────────────┘  │ Reference │
│         │ 1:N                   │                     └───────────┘
│         ▼                       │ M:N                              │
│  ┌──────────────┐               │                                  │
│  │ Order_Items  │◄──────────────┘                                  │
│  ├──────────────┤                                                  │
│  │ id (PK)      │                                                  │
│  │ order_id (FK)│                                                  │
│  │ product_id   │                                                  │
│  │ quantity     │                                                  │
│  └──────────────┘                                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Best Practices

<div dir="rtl">

### 1. اختر النوع الصحيح

</div>

```
1:1 → عندما البيانات optional أو sensitive
1:N → الأكثر شيوعاً (User-Orders, Category-Products)
M:N → استخدم Junction Table دايماً
```

<div dir="rtl">

### 2. الـ Junction Table

</div>

```sql
-- ✅ ممكن تحتوي على بيانات إضافية
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrolled_at TIMESTAMP,  -- متى سجل؟
    grade CHAR(2),          -- الدرجة
    PRIMARY KEY (student_id, course_id)
);
```

<div dir="rtl">

### 3. تسمية الجداول

</div>

```
users, products, orders          → Plural (جمع)
user_profiles, order_items       → Relationship tables
enrollments, product_tags        → Junction tables
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **1:1** للبيانات الاختيارية أو الحساسة
2. **1:N** الأكثر شيوعاً في التطبيقات
3. **M:N** تحتاج Junction Table
4. **Self-Reference** للبيانات الهرمية
5. اختر العلاقة بناءً على الـ **business requirements**

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Primary & Foreign Keys](./02-keys.md)**

</div>

---

<div align="center">

[🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./02-keys.md)

</div>
