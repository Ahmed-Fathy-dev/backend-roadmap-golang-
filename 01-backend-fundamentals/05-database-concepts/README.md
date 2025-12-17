# Module 1.5: Database Concepts 🗄️

<div dir="rtl">

## نظرة عامة

قواعد البيانات هي قلب أي Backend Application. في هذا الدرس سنتعلم المفاهيم الأساسية التي تحتاجها قبل البدء في PostgreSQL.

</div>

---

## 📖 Content

### 1. What is a Database?

<div dir="rtl">

**Database** هو نظام منظم لتخزين واسترجاع البيانات.

#### لماذا نحتاج Database؟

- ✅ تخزين البيانات بشكل دائم (Persistent Storage)
- ✅ البحث السريع
- ✅ الأمان والصلاحيات
- ✅ التعامل مع millions of records
- ✅ ضمان سلامة البيانات (Data Integrity)

</div>

---

### 2. SQL vs NoSQL

<div dir="rtl">

قواعد البيانات مقسمة لنوعين رئيسيين:

</div>

| Feature         | SQL (Relational)                         | NoSQL                                   |
| --------------- | ---------------------------------------- | --------------------------------------- |
| **الهيكل**      | <div dir="rtl">جداول بصفوف وأعمدة</div>  | <div dir="rtl">مرن(JSON-like)</div>     |
| **Schema**      | <div dir="rtl">ثابت (Fixed)</div>        | <div dir="rtl">ديناميكي (Dynamic)</div> |
| **العلاقات**    | <div dir="rtl">قوية (Joins)</div>        | <div dir="rtl">محدودة</div>             |
| **Scalability** | <div dir="rtl">Vertical</div>            | <div dir="rtl">Horizontal</div>         |
| **مثال**        | PostgreSQL, MySQL                        | MongoDB, Redis                          |
| **الاستخدام**   | <div dir="rtl">بيانات منظمة ومعقدة</div> | <div dir="rtl">بيانات مرنة وسريعة</div> |

<div dir="rtl">

### متى تستخدم SQL؟

- ✅ البيانات منظمة ولها علاقات واضحة
- ✅ تحتاج ACID Properties
- ✅ Complex queries و joins
- ✅ التطبيقات المالية، E-commerce، ERP

### متى تستخدم NoSQL؟

- ✅ البيانات غير منظمة أو متغيرة
- ✅ تحتاج scalability عالية جداً
- ✅ Real-time applications
- ✅ Social media, Logging, Caching

**في هذا المنهج سنركز على SQL (PostgreSQL)**

</div>

---

### 3. Relational Database Fundamentals

<div dir="rtl">

#### 3.1 Tables (الجداول)

Database تتكون من Tables، كل Table يشبه جدول Excel:

</div>

```
Table: users
┌────┬──────────┬──────────────────────┬──────────┬─────────────────────┐
│ id │   name   │       email          │   role   │     created_at      │
├────┼──────────┼──────────────────────┼──────────┼─────────────────────┤
│ 1  │ Ahmed    │ ahmed@example.com    │ user     │ 2024-01-15 10:30:00 │
│ 2  │ Sara     │ sara@example.com     │ admin    │ 2024-01-16 14:20:00 │
│ 3  │ Omar     │ omar@example.com     │ user     │ 2024-01-17 09:15:00 │
└────┴──────────┴──────────────────────┴──────────┴─────────────────────┘
```

<div dir="rtl">

- **Columns (أعمدة):** `id`, `name`, `email`, `role`, `created_at`
- **Rows (صفوف):** كل صف يمثل record واحد (مستخدم واحد)

#### 3.2 Primary Key

كل table يحتاج **Primary Key** (مفتاح أساسي):

- ✅ Unique لكل row
- ✅ لا يمكن أن يكون NULL
- ✅ غالباً يكون `id`

</div>

---

### 4. Data Types

<div dir="rtl">

أنواع البيانات الشائعة في PostgreSQL:

</div>

| Type             | الاستخدام                              | مثال                    |
| ---------------- | -------------------------------------- | ----------------------- |
| **INTEGER**      | <div dir="rtl">أرقام صحيحة</div>       | `5, 100, -20`           |
| **BIGINT**       | <div dir="rtl">أرقام صحيحة كبيرة</div> | `9999999999`            |
| **VARCHAR(n)**   | <div dir="rtl">نص بطول محدد</div>      | `'Ahmed'`               |
| **TEXT**         | <div dir="rtl">نص بطول غير محدد</div>  | `'Long description...'` |
| **BOOLEAN**      | <div dir="rtl">صح أو خطأ</div>         | `TRUE, FALSE`           |
| **DATE**         | <div dir="rtl">تاريخ</div>             | `'2024-12-10'`          |
| **TIMESTAMP**    | <div dir="rtl">تاريخ و وقت</div>       | `'2024-12-10 15:30:00'` |
| **DECIMAL(p,s)** | <div dir="rtl">أرقام عشرية دقيقة</div> | `99.99`                 |
| **JSON/JSONB**   | <div dir="rtl">بيانات JSON</div>       | `{"key": "value"}`      |

---

### 5. Relationships (العلاقات)

<div dir="rtl">

العلاقات بين Tables هي قوة SQL Databases.

#### 5.1 One-to-One (واحد لواحد)

</div>

```
users                    user_profiles
┌────┬────────┐         ┌────┬─────────┬──────┬──────┐
│ id │  name  │         │ id │ user_id │ bio  │ age  │
├────┼────────┤         ├────┼─────────┼──────┼──────┤
│ 1  │ Ahmed  │←───1:1──│ 1  │    1    │ ...  │  25  │
│ 2  │ Sara   │←───1:1──│ 2  │    2    │ ...  │  30  │
└────┴────────┘         └────┴─────────┴──────┴──────┘
```

<div dir="rtl">

**مثال:** كل User له Profile واحد فقط

---

#### 5.2 One-to-Many (واحد لكثير)

الأكثر شيوعاً:

</div>

```
users                    posts
┌────┬────────┐         ┌────┬─────────┬────────┐
│ id │  name  │         │ id │ user_id │ title  │
├────┼────────┤         ├────┼─────────┼────────┤
│ 1  │ Ahmed  │←──┬     │ 1  │    1    │ Post 1 │
│ 2  │ Sara   │   ├─1:M─│ 2  │    1    │ Post 2 │
└────┴────────┘   │     │ 3  │    1    │ Post 3 │
                  └─────│ 4  │    2    │ Post 4 │
                        └────┴─────────┴────────┘
```

<div dir="rtl">

**مثال:** كل User يمكن أن يكون له عدة Posts

---

#### 5.3 Many-to-Many (كثير لكثير)

تحتاج Junction Table (جدول وسيط):

</div>

```
students              enrollments            courses
┌────┬────────┐      ┌────┬────────────┬───────────┐      ┌────┬────────────┐
│ id │  name  │      │ id │ student_id │ course_id │      │ id │    name    │
├────┼────────┤      ├────┼────────────┼───────────┤      ├────┼────────────┤
│ 1  │ Ahmed  │←─┬   │ 1  │     1      │     1     │   ┌─→│ 1  │ Math       │
│ 2  │ Sara   │  ├─M:│ 2  │     1      │     2     │:M─┤  │ 2  │ Physics    │
└────┴────────┘  │   │ 3  │     2      │     1     │   └─→│ 3  │ Chemistry  │
                 └───│ 4  │     2      │     3     │──────→└────┴────────────┘
                     └────┴────────────┴───────────┘
```

<div dir="rtl">

**مثال:** كل Student يمكن التسجيل في عدة Courses، وكل Course له عدة Students

</div>

---

### 6. ACID Properties

<div dir="rtl">

**ACID** هي الخصائص التي تضمن سلامة البيانات:

#### A - Atomicity (الذرية)

العملية إما تنفذ كلها أو لا تنفذ أبداً (All or Nothing)

**مثال:**  
عند تحويل أموال من حساب لآخر:

</div>

```sql
-- يجب أن ينفذ الاثنين معاً أو لا شيء
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- خصم
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- إضافة
COMMIT;
```

<div dir="rtl">

إذا فشل أي واحد، يُلغى الكل.

#### C - Consistency (الاتساق)

البيانات تبقى في حالة صحيحة دائماً

**مثال:** لا يمكن أن يكون الرصيد سالب

#### I - Isolation (العزل)

Transactions لا تؤثر على بعضها

**مثال:** إذا شخصان يعدّلان نفس البيانات، لا يحدث تضارب

#### D - Durability (الديمومة)

بعد COMMIT، البيانات محفوظة حتى لو انهار Server

**مثال:** بعد تأكيد الطلب، لن يُفقد حتى لو انقطع الكهرباء

</div>

---

### 7. Transactions

<div dir="rtl">

**Transaction** هي مجموعة من العمليات تنفذ كوحدة واحدة:

</div>

```sql
BEGIN;                                    -- Start transaction

  INSERT INTO orders (user_id, total)
  VALUES (5, 100);

  UPDATE products SET stock = stock - 1
  WHERE id = 10;

  INSERT INTO order_items (order_id, product_id)
  VALUES (LAST_INSERT_ID(), 10);

COMMIT;                                   -- Confirm all changes
```

<div dir="rtl">

إذا حدث خطأ في أي عملية:

</div>

```sql
ROLLBACK;                                 -- Cancel all changes
```

---

### 8. Indexes (الفهارس)

<div dir="rtl">

**Index** يسرّع البحث في Database:

</div>

```
بدون Index:
البحث عن user_id = 5 في مليون record
→ يفحص المليون واحد واحد 😴 (SLOW)

مع Index:
→ يقفز مباشرة للـ record المطلوب ⚡ (FAST)
```

<div dir="rtl">

### متى تستخدم Index؟

- ✅ على Columns التي تبحث فيها كثيراً
- ✅ Foreign Keys
- ✅ Columns في WHERE, ORDER BY, JOIN

### متى لا تستخدم Index؟

- ❌ على Tables صغيرة
- ❌ على Columns نادراً ما تستخدمها
- ❌ على Columns التي تتغير كثيراً (INSERT/UPDATE بطيء)

**ملاحظة:** Primary Key له index تلقائياً

</div>

---

### 9. Constraints (القيود)

<div dir="rtl">

Constraints لضمان صحة البيانات:

</div>

| Constraint      | الوصف                                      | مثال                   |
| --------------- | ------------------------------------------ | ---------------------- |
| **PRIMARY KEY** | <div dir="rtl">Unique identifier</div>     | `id`                   |
| **FOREIGN KEY** | <div dir="rtl">ربط بـ table آخر</div>      | `user_id` →`users(id)` |
| **UNIQUE**      | <div dir="rtl">قيمة فريدة</div>            | `email`                |
| **NOT NULL**    | <div dir="rtl">لا يمكن أن تكون فارغة</div> | `name`                 |
| **CHECK**       | <div dir="rtl">شرط مخصص</div>              | `age >= 18`            |
| **DEFAULT**     | <div dir="rtl">قيمة افتراضية</div>         | `created_at = NOW()`   |

<div dir="rtl">

### مثال عملي:

</div>

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,                    -- Auto-increment, unique
  email VARCHAR(255) UNIQUE NOT NULL,       -- Must be unique, required
  age INTEGER CHECK (age >= 18),            -- Must be 18+
  role VARCHAR(50) DEFAULT 'user',          -- Default value
  created_at TIMESTAMP DEFAULT NOW()        -- Auto timestamp
);
```

---

### 10. Normalization (التطبيع)

<div dir="rtl">

**Normalization** هي عملية تنظيم Database لتقليل التكرار.

#### ❌ قبل Normalization (تكرار):

</div>

```
orders
┌────┬──────────────┬──────────────────────┬──────────┬─────────┐
│ id │ customer_name│ customer_email       │ product  │  price  │
├────┼──────────────┼──────────────────────┼──────────┼─────────┤
│ 1  │ Ahmed        │ ahmed@example.com    │ Laptop   │  1000   │
│ 2  │ Ahmed        │ ahmed@example.com    │ Mouse    │   20    │
│ 3  │ Sara         │ sara@example.com     │ Laptop   │  1000   │
└────┴──────────────┴──────────────────────┴──────────┴─────────┘
                      ↑ تكرار البيانات! ↑
```

<div dir="rtl">

#### ✅ بعد Normalization (منظم):

</div>

```
customers                    orders                      products
┌────┬────────┬─────────┐  ┌────┬─────────────┬────────┐  ┌────┬────────┬───────┐
│ id │  name  │  email  │  │ id │ customer_id │prod_id │  │ id │  name  │ price │
├────┼────────┼─────────┤  ├────┼─────────────┼────────┤  ├────┼────────┼───────┤
│ 1  │ Ahmed  │ahmed@..│←─│ 1  │      1      │   1    │─→│ 1  │ Laptop │ 1000  │
│ 2  │ Sara   │sara@.. │  │ 2  │      1      │   2    │  │ 2  │ Mouse  │  20   │
└────┴────────┴─────────┘  │ 3  │      2      │   1    │  └────┴────────┴───────┘
                           └────┴─────────────┴────────┘
```

<div dir="rtl">

**الفوائد:**

- ✅ لا تكرار في البيانات
- ✅ سهولة التحديث (تحديث email في مكان واحد فقط)
- ✅ توفير مساحة

### مستويات Normalization:

- **1NF:** كل column قيمة واحدة (atomic)
- **2NF:** لا توجد Partial Dependency
- **3NF:** لا توجد Transitive Dependency

**(سنتعلم التفصيل في Track 4)**

</div>

---

### 11. Basic SQL Operations

<div dir="rtl">

نظرة سريعة على SQL الأساسي:

#### CREATE - إنشاء table:

</div>

```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  stock INTEGER DEFAULT 0
);
```

<div dir="rtl">

#### INSERT - إضافة بيانات:

</div>

```sql
INSERT INTO products (name, price, stock)
VALUES ('Laptop', 1000.00, 10);
```

<div dir="rtl">

#### SELECT - قراءة بيانات:

</div>

```sql
SELECT * FROM products;                    -- جميع البيانات
SELECT name, price FROM products;          -- أعمدة محددة
SELECT * FROM products WHERE price > 500;  -- مع شرط
```

<div dir="rtl">

#### UPDATE - تحديث:

</div>

```sql
UPDATE products
SET price = 900, stock = 15
WHERE id = 1;
```

<div dir="rtl">

#### DELETE - حذف:

</div>

```sql
DELETE FROM products WHERE id = 1;
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **SQL Databases:** بيانات منظمة في Tables مع علاقات قوية
- ✅ **NoSQL Databases:** بيانات مرنة وسريعة
- ✅ **Primary Key:** معرّف unique لكل row
- ✅ **Relationships:** One-to-One, One-to-Many, Many-to-Many
- ✅ **ACID:** Atomicity, Consistency, Isolation, Durability
- ✅ **Transactions:** مجموعة عمليات تنفذ كوحدة واحدة
- ✅ **Indexes:** تسريع البحث
- ✅ **Constraints:** ضمان صحة البيانات
- ✅ **Normalization:** تقليل التكرار

</div>

---

## 🎯 Quiz

<div dir="rtl">

1. ما الفرق بين SQL و NoSQL؟
2. ما هي الأنواع الثلاثة للعلاقات؟
3. ما معنى ACID؟
4. متى تستخدم Index؟
5. ما الفرق بين PRIMARY KEY و FOREIGN KEY؟

</div>

---

## 🎉 Track 1 Complete!

<div dir="rtl">

تهانينا! أنهيت **Track 1: Backend Fundamentals**

الآن أنت تفهم:

- ✅ ما هو Backend وكيف يعمل
- ✅ HTTP Protocol بالتفصيل
- ✅ كيف تصمم REST APIs احترافية
- ✅ Authentication و Authorization
- ✅ أساسيات قواعد البيانات

</div>

---

## ⏭️ Next Track

<div dir="rtl">

استعد للغوص في عالم Go! 🚀

**➡️ [Track 2: Go Basics](../../02-go-basics/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Authentication](../04-auth-basics/README.md) | [🏠 Track 1 Home](../README.md) | [📚 Main](../../README.md)

</div>
