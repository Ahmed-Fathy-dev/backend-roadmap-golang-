# مقدمة في SQL 📖

<div dir="rtl">

## ما هو SQL؟

**SQL** (Structured Query Language) هي اللغة القياسية للتعامل مع قواعد البيانات العلائقية. كل حاجة بتعملها مع PostgreSQL - من إنشاء جداول لإضافة بيانات لقراءتها - بتستخدم SQL.

**المدة المتوقعة:** 20-30 دقيقة

</div>

---

## 📖 تاريخ SQL

<div dir="rtl">

### الجدول الزمني

</div>

```
1970 ─── Edgar F. Codd ينشر نظرية Relational Database في IBM
   │
1974 ─── IBM تطور SEQUEL (Structured English Query Language)
   │
1979 ─── Oracle تطلق أول منتج SQL تجاري
   │
1986 ─── ANSI تعتمد SQL كمعيار رسمي (SQL-86)
   │
1992 ─── SQL-92 (الإصدار الأكثر انتشاراً)
   │
1999 ─── SQL:1999 (Object-Relational features)
   │
2003 ─── SQL:2003 (XML support)
   │
2011 ─── SQL:2011 (Temporal data)
   │
2016 ─── SQL:2016 (JSON support)
   │
2023 ─── SQL:2023 (Property Graph Queries)
```

<div dir="rtl">

### ليه اسمها SQL؟

- **S** = Structured (منظمة)
- **Q** = Query (استعلام)
- **L** = Language (لغة)

البعض بينطقها "سيكويل" والبعض بينطقها "إس كيو إل" - الاتنين صح!

</div>

---

## 🏗️ أنواع أوامر SQL

<div dir="rtl">

SQL مقسمة لأربع فئات رئيسية:

</div>

```
┌─────────────────────────────────────────────────────────────────┐
│                        SQL Commands                              │
├───────────────┬───────────────┬───────────────┬────────────────┤
│      DDL      │      DML      │      DCL      │      TCL       │
│ Data Definition│ Data Manip.  │ Data Control  │ Transaction    │
├───────────────┼───────────────┼───────────────┼────────────────┤
│ CREATE        │ SELECT        │ GRANT         │ BEGIN          │
│ ALTER         │ INSERT        │ REVOKE        │ COMMIT         │
│ DROP          │ UPDATE        │               │ ROLLBACK       │
│ TRUNCATE      │ DELETE        │               │ SAVEPOINT      │
└───────────────┴───────────────┴───────────────┴────────────────┘
```

---

## 📝 DDL - Data Definition Language

<div dir="rtl">

أوامر DDL بتتحكم في **هيكل** قاعدة البيانات (الجداول، الأعمدة، etc.)

</div>

### CREATE - إنشاء

```sql
-- إنشاء Database
CREATE DATABASE myapp;

-- إنشاء Table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE
);

-- إنشاء Index
CREATE INDEX idx_users_email ON users(email);
```

### ALTER - تعديل

```sql
-- إضافة عمود
ALTER TABLE users ADD COLUMN age INT;

-- تعديل عمود
ALTER TABLE users ALTER COLUMN name TYPE VARCHAR(200);

-- حذف عمود
ALTER TABLE users DROP COLUMN age;
```

### DROP - حذف

```sql
-- حذف Table (بكل البيانات!)
DROP TABLE users;

-- حذف لو موجود فقط
DROP TABLE IF EXISTS users;

-- حذف Database
DROP DATABASE myapp;
```

### TRUNCATE - إفراغ

```sql
-- حذف كل البيانات (أسرع من DELETE)
TRUNCATE TABLE users;

-- مع إعادة تعيين الـ SERIAL
TRUNCATE TABLE users RESTART IDENTITY;
```

---

## 📊 DML - Data Manipulation Language

<div dir="rtl">

أوامر DML بتتحكم في **البيانات** نفسها.

</div>

### SELECT - قراءة

```sql
-- قراءة كل البيانات
SELECT * FROM users;

-- قراءة أعمدة محددة
SELECT name, email FROM users;

-- مع شروط
SELECT * FROM users WHERE age > 18;

-- مع ترتيب
SELECT * FROM users ORDER BY name ASC;
```

### INSERT - إدخال

```sql
-- إدخال صف واحد
INSERT INTO users (name, email)
VALUES ('Ahmed', 'ahmed@example.com');

-- إدخال عدة صفوف
INSERT INTO users (name, email) VALUES
    ('Sara', 'sara@example.com'),
    ('Omar', 'omar@example.com');
```

### UPDATE - تحديث

```sql
-- تحديث صف واحد
UPDATE users SET name = 'Ahmed Ali' WHERE id = 1;

-- تحديث عدة أعمدة
UPDATE users
SET name = 'Ahmed Ali', email = 'ahmed.ali@example.com'
WHERE id = 1;

-- تحديث كل الصفوف (خطير!)
UPDATE users SET is_active = true;
```

### DELETE - حذف

```sql
-- حذف صف واحد
DELETE FROM users WHERE id = 1;

-- حذف بشرط
DELETE FROM users WHERE created_at < '2024-01-01';

-- حذف كل البيانات (بطيء - استخدم TRUNCATE)
DELETE FROM users;
```

---

## 🔐 DCL - Data Control Language

<div dir="rtl">

أوامر DCL بتتحكم في **الصلاحيات**.

</div>

### GRANT - منح صلاحيات

```sql
-- صلاحيات على Database
GRANT CONNECT ON DATABASE myapp TO myuser;

-- صلاحيات على Table
GRANT SELECT, INSERT ON users TO myuser;

-- كل الصلاحيات
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;
```

### REVOKE - سحب صلاحيات

```sql
-- سحب صلاحية معينة
REVOKE INSERT ON users FROM myuser;

-- سحب كل الصلاحيات
REVOKE ALL PRIVILEGES ON users FROM myuser;
```

---

## 🔄 TCL - Transaction Control Language

<div dir="rtl">

أوامر TCL بتتحكم في **المعاملات** (Transactions).

</div>

### Transaction مثال

```sql
-- بداية Transaction
BEGIN;

-- عمليات متعددة
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- لو كل حاجة تمام
COMMIT;

-- أو لو فيه مشكلة
ROLLBACK;
```

### SAVEPOINT

```sql
BEGIN;

INSERT INTO users (name) VALUES ('Ahmed');
SAVEPOINT after_ahmed;

INSERT INTO users (name) VALUES ('Sara');
-- لو حصل خطأ هنا
ROLLBACK TO after_ahmed;
-- Ahmed هيتحفظ، Sara لا

COMMIT;
```

---

## ✍️ قواعد كتابة SQL

<div dir="rtl">

### 1. الـ Keywords

</div>

```sql
-- ✅ Convention: UPPERCASE للـ keywords
SELECT name FROM users WHERE age > 18;

-- ⚠️ بيشتغل بس مش best practice
select name from users where age > 18;

-- ⚠️ Mixed case (مش مفضل)
Select Name From Users Where Age > 18;
```

<div dir="rtl">

### 2. الأسماء (Identifiers)

</div>

```sql
-- ✅ lowercase مع underscore (snake_case)
CREATE TABLE user_profiles (...);
SELECT first_name, last_name FROM users;

-- ⚠️ بيشتغل بس مش مفضل
CREATE TABLE UserProfiles (...);

-- ❌ محتاج quotes (وحش)
CREATE TABLE "User Profiles" (...);
```

<div dir="rtl">

### 3. الـ Semicolon

</div>

```sql
-- كل SQL statement لازم ينتهي بـ ;
SELECT * FROM users;
INSERT INTO users (name) VALUES ('Ahmed');

-- في psql، الـ ; بيقول "نفذ دلوقتي"
```

<div dir="rtl">

### 4. التعليقات

</div>

```sql
-- تعليق سطر واحد

/*
   تعليق
   متعدد
   الأسطر
*/

SELECT * FROM users; -- تعليق في نهاية السطر
```

<div dir="rtl">

### 5. التنسيق المقروء

</div>

```sql
-- ❌ صعب القراءة
SELECT u.id, u.name, u.email, COUNT(o.id) as order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id WHERE u.is_active = true AND u.created_at > '2024-01-01' GROUP BY u.id, u.name, u.email HAVING COUNT(o.id) > 5 ORDER BY order_count DESC LIMIT 10;

-- ✅ سهل القراءة
SELECT
    u.id,
    u.name,
    u.email,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.is_active = true
  AND u.created_at > '2024-01-01'
GROUP BY u.id, u.name, u.email
HAVING COUNT(o.id) > 5
ORDER BY order_count DESC
LIMIT 10;
```

---

## 🔠 NULL في SQL

<div dir="rtl">

`NULL` قيمة خاصة تعني "غير معروف" أو "غير موجود".

</div>

```sql
-- NULL مش بتساوي أي حاجة (حتى نفسها!)
SELECT NULL = NULL;        -- NULL (مش TRUE!)
SELECT NULL != NULL;       -- NULL (مش TRUE!)

-- استخدم IS NULL / IS NOT NULL
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM users WHERE phone IS NOT NULL;

-- COALESCE لاستبدال NULL
SELECT COALESCE(phone, 'No phone') FROM users;

-- NULLIF لإنتاج NULL
SELECT NULLIF(status, 'unknown') FROM users;
-- لو status = 'unknown'، هترجع NULL
```

---

## 🎯 ترتيب تنفيذ SQL

<div dir="rtl">

هام جداً! SQL بيتنفذ بترتيب مختلف عن الكتابة:

</div>

```
ترتيب الكتابة:                    ترتيب التنفيذ:
─────────────                     ──────────────
1. SELECT                         1. FROM        ← أول حاجة
2. FROM                           2. WHERE
3. WHERE                          3. GROUP BY
4. GROUP BY                       4. HAVING
5. HAVING                         5. SELECT      ← هنا بنختار الأعمدة
6. ORDER BY                       6. ORDER BY
7. LIMIT                          7. LIMIT       ← آخر حاجة
```

<div dir="rtl">

### مثال

</div>

```sql
SELECT department, AVG(salary) AS avg_salary  -- 5. نختار الأعمدة
FROM employees                                 -- 1. نجيب البيانات
WHERE is_active = true                         -- 2. نفلتر الصفوف
GROUP BY department                            -- 3. نجمّع
HAVING AVG(salary) > 50000                     -- 4. نفلتر المجموعات
ORDER BY avg_salary DESC                       -- 6. نرتب
LIMIT 5;                                       -- 7. نحدد العدد
```

<div dir="rtl">

### ليه ده مهم؟

لأنه بيحدد:
- إيه الـ aliases اللي تقدر تستخدمها فين
- ليه WHERE مش بتشتغل مع Aggregate functions
- ليه محتاج HAVING بدل WHERE للتجميعات

</div>

---

## 💡 نصائح للمبتدئين

<div dir="rtl">

### 1. ابدأ بسيط

</div>

```sql
-- ابدأ بـ SELECT بسيط
SELECT * FROM users;

-- بعدين أضف WHERE
SELECT * FROM users WHERE is_active = true;

-- بعدين أضف ORDER
SELECT * FROM users WHERE is_active = true ORDER BY name;

-- وهكذا...
```

<div dir="rtl">

### 2. استخدم LIMIT في التجارب

</div>

```sql
-- لو الـ Table فيه ملايين صفوف
SELECT * FROM huge_table LIMIT 10;
```

<div dir="rtl">

### 3. اختبر في Transaction

</div>

```sql
BEGIN;
DELETE FROM users WHERE age < 18;  -- شوف إيه اللي هيتحذف
-- لو مش متأكد
ROLLBACK;
-- لو متأكد
-- COMMIT;
```

<div dir="rtl">

### 4. استخدم EXPLAIN

</div>

```sql
-- شوف خطة التنفيذ
EXPLAIN SELECT * FROM users WHERE email = 'test@test.com';
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **SQL** = لغة التعامل مع Databases
2. **DDL** = هيكل (CREATE, ALTER, DROP)
3. **DML** = بيانات (SELECT, INSERT, UPDATE, DELETE)
4. **DCL** = صلاحيات (GRANT, REVOKE)
5. **TCL** = معاملات (BEGIN, COMMIT, ROLLBACK)
6. **Keywords** بـ UPPERCASE (convention)
7. **Identifiers** بـ snake_case
8. **NULL** = غير معروف (مش صفر أو empty string)

</div>

---

## 🧪 اختبر نفسك

<div dir="rtl">

1. ما الفرق بين DDL و DML؟
2. ما الفرق بين DELETE و TRUNCATE؟
3. ما الفرق بين COMMIT و ROLLBACK؟
4. لماذا `NULL = NULL` ترجع `NULL` وليس `TRUE`؟
5. اكتب SQL statement يحذف كل الـ users اللي عمرهم أقل من 18

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [أنواع الأرقام](./02-numeric-types.md)**

</div>

---

<div align="center">

[🏠 العودة للـ Module](../README.md)

</div>
