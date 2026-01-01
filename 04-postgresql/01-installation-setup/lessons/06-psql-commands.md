# أوامر psql الأساسية 💻

<div dir="rtl">

## مقدمة

**psql** هو الـ Command Line Interface الرسمي لـ PostgreSQL. أداة قوية جداً ومهمة لأي Developer أو DBA. في الدرس ده هنتعلم كل الأوامر اللي هتحتاجها.

**المدة المتوقعة:** 20-30 دقيقة

</div>

---

## 🚀 الاتصال بـ PostgreSQL

<div dir="rtl">

### الصيغة الأساسية

</div>

```bash
psql [options] [database] [username]
```

<div dir="rtl">

### أمثلة الاتصال

</div>

```bash
# الاتصال الافتراضي (على Linux/macOS)
psql

# الاتصال كـ postgres user
psql -U postgres

# الاتصال بـ database محددة
psql -U postgres -d myapp

# الاتصال بـ host محدد
psql -h localhost -U postgres -d myapp

# الاتصال بـ port مختلف
psql -h localhost -p 5433 -U postgres -d myapp

# باستخدام Connection String
psql "postgresql://postgres:password@localhost:5432/myapp"
```

<div dir="rtl">

### Options الشائعة

| Option | الوظيفة |
|--------|---------|
| `-h` | Host (عنوان السيرفر) |
| `-p` | Port (الافتراضي 5432) |
| `-U` | Username |
| `-d` | Database |
| `-W` | طلب Password |
| `-w` | بدون Password |
| `-c` | تنفيذ أمر واحد |
| `-f` | تنفيذ ملف SQL |

</div>

---

## 📋 Meta-Commands (أوامر Backslash)

<div dir="rtl">

أوامر psql الخاصة بتبدأ بـ `\` (backslash). دي مش SQL، دي أوامر psql نفسه.

### أوامر المساعدة

</div>

```sql
-- قائمة كل الأوامر
\?

-- مساعدة SQL
\h

-- مساعدة لأمر SQL محدد
\h SELECT
\h CREATE TABLE
```

---

## 🗄️ أوامر الـ Databases

<div dir="rtl">

### عرض الـ Databases

</div>

```sql
-- قائمة كل الـ Databases
\l
-- أو
\list

-- معلومات تفصيلية
\l+
```

**النتيجة:**
```
                              List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype
-----------+----------+----------+-------------+-------------
 myapp     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
```

<div dir="rtl">

### التنقل بين الـ Databases

</div>

```sql
-- الاتصال بـ database
\c myapp
-- أو
\connect myapp

-- الاتصال بـ user مختلف
\c myapp admin_user

-- معرفة الـ Database الحالية
SELECT current_database();
```

---

## 📊 أوامر الـ Tables

<div dir="rtl">

### عرض الـ Tables

</div>

```sql
-- قائمة Tables في الـ Schema الحالي
\dt

-- كل الـ Tables (مع system tables)
\dt *.*

-- معلومات تفصيلية
\dt+

-- Tables في schema معين
\dt public.*
\dt myschema.*
```

**النتيجة:**
```
            List of relations
 Schema |   Name   | Type  |  Owner
--------+----------+-------+----------
 public | users    | table | postgres
 public | products | table | postgres
 public | orders   | table | postgres
```

<div dir="rtl">

### وصف Table

</div>

```sql
-- هيكل الـ Table
\d users

-- معلومات تفصيلية (مع indexes و constraints)
\d+ users
```

**النتيجة:**
```
                                      Table "public.users"
   Column   |          Type          | Collation | Nullable |              Default
------------+------------------------+-----------+----------+-----------------------------------
 id         | integer                |           | not null | nextval('users_id_seq'::regclass)
 name       | character varying(100) |           | not null |
 email      | character varying(100) |           | not null |
 created_at | timestamp              |           |          | now()
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "users_email_key" UNIQUE CONSTRAINT, btree (email)
```

---

## 👤 أوامر الـ Users و Roles

<div dir="rtl">

### عرض الـ Users

</div>

```sql
-- قائمة Users/Roles
\du
-- أو
\dg

-- معلومات تفصيلية
\du+
```

**النتيجة:**
```
                             List of roles
 Role name |                   Attributes
-----------+------------------------------------------------
 postgres  | Superuser, Create role, Create DB, Replication
 myuser    |
 readonly  | Cannot login
```

---

## 🔍 أوامر استكشافية أخرى

<div dir="rtl">

### Schemas

</div>

```sql
-- قائمة Schemas
\dn
\dn+
```

<div dir="rtl">

### Views

</div>

```sql
-- قائمة Views
\dv
\dv+
```

<div dir="rtl">

### Indexes

</div>

```sql
-- قائمة Indexes
\di
\di+

-- Indexes لـ table معين
\di users*
```

<div dir="rtl">

### Functions

</div>

```sql
-- قائمة Functions
\df

-- Functions معينة
\df *user*
```

<div dir="rtl">

### Sequences

</div>

```sql
-- قائمة Sequences
\ds
```

---

## 💻 تنفيذ SQL

<div dir="rtl">

### كتابة Queries

</div>

```sql
-- Query عادي (لازم ينتهي بـ ;)
SELECT * FROM users;

-- Query متعدد الأسطر
SELECT
    id,
    name,
    email
FROM
    users
WHERE
    age > 20;
-- اضغط Enter بعد ;

-- إلغاء Query قبل التنفيذ
\r
-- أو Ctrl+C
```

<div dir="rtl">

### تنفيذ ملف SQL

</div>

```sql
-- تنفيذ ملف
\i /path/to/script.sql

-- على Windows
\i C:/Users/ahmed/scripts/setup.sql
-- أو
\i 'C:\\Users\\ahmed\\scripts\\setup.sql'
```

<div dir="rtl">

### تنفيذ أمر من خارج psql

</div>

```bash
# تنفيذ أمر واحد
psql -U postgres -d myapp -c "SELECT * FROM users;"

# تنفيذ ملف
psql -U postgres -d myapp -f setup.sql

# تنفيذ عدة أوامر
psql -U postgres -d myapp <<EOF
SELECT * FROM users;
SELECT COUNT(*) FROM orders;
EOF
```

---

## 📤 التحكم في الـ Output

<div dir="rtl">

### تغيير طريقة العرض

</div>

```sql
-- عرض موسع (سطر لكل column)
\x
-- أو
\x on
\x off
\x auto  -- تلقائي حسب حجم البيانات

-- مثال النتيجة مع \x on
-[ RECORD 1 ]--------------------
id         | 1
name       | Ahmed
email      | ahmed@test.com
created_at | 2024-12-21 10:30:00
```

<div dir="rtl">

### حفظ النتائج في ملف

</div>

```sql
-- بداية التسجيل في ملف
\o output.txt

-- نفذ أي queries
SELECT * FROM users;
SELECT * FROM products;

-- إنهاء التسجيل
\o

-- أو تصدير query واحد
\copy (SELECT * FROM users) TO '/path/to/users.csv' CSV HEADER
```

<div dir="rtl">

### تنسيق الـ Output

</div>

```sql
-- بدون Headers
\t
\t on
\t off

-- بدون Alignment
\a

-- تغيير Separator
\pset fieldsep ','
\pset fieldsep '|'

-- HTML output
\pset format html

-- إرجاع للـ default
\pset format aligned
```

---

## ⚙️ إعدادات psql

<div dir="rtl">

### معلومات الاتصال الحالي

</div>

```sql
-- معلومات الاتصال
\conninfo
-- Output: You are connected to database "myapp" as user "postgres"...
```

<div dir="rtl">

### تغيير الـ Prompt

</div>

```sql
-- تغيير الـ prompt
\set PROMPT1 '%n@%M:%>/%/ %# '

-- Prompt الافتراضي
-- database_name=#
```

<div dir="rtl">

### Timing

</div>

```sql
-- تفعيل/تعطيل وقت التنفيذ
\timing
\timing on
\timing off

-- مثال
myapp=# \timing on
Timing is on.
myapp=# SELECT * FROM users;
...
Time: 1.234 ms
```

<div dir="rtl">

### Auto-commit

</div>

```sql
-- إيقاف Auto-commit (مفيد للـ transactions)
\set AUTOCOMMIT off

-- إرجاعه
\set AUTOCOMMIT on
```

---

## 🔄 History والتحرير

<div dir="rtl">

### استخدام History

</div>

```sql
-- عرض آخر Query
\p

-- تعديل آخر Query في editor
\e

-- تنفيذ آخر Query
\g

-- البحث في History
-- استخدم Arrow Up/Down
-- أو Ctrl+R للبحث
```

<div dir="rtl">

### التحرير في Editor خارجي

</div>

```sql
-- فتح Editor لكتابة Query
\e

-- تحديد Editor معين
\setenv EDITOR nano
\setenv EDITOR vim
\setenv EDITOR code  -- VS Code

-- تعديل function في Editor
\ef my_function
```

---

## 🔐 أوامر إدارية

<div dir="rtl">

### إنشاء Database

</div>

```sql
-- من psql
CREATE DATABASE myapp;

-- أو من command line
createdb -U postgres myapp
```

<div dir="rtl">

### حذف Database

</div>

```sql
-- من psql
DROP DATABASE myapp;

-- أو من command line
dropdb -U postgres myapp
```

<div dir="rtl">

### إنشاء User

</div>

```sql
-- من psql
CREATE USER myuser WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE myapp TO myuser;

-- أو من command line
createuser -U postgres myuser
```

---

## 🚪 الخروج من psql

```sql
-- الخروج
\q
-- أو
quit
-- أو
exit
-- أو
Ctrl+D
```

---

## 📋 ملخص الأوامر الأساسية

<div dir="rtl">

### Quick Reference

</div>

```
┌─────────────────────────────────────────────────────────────────┐
│                    psql Quick Reference                          │
├─────────────────────────────────────────────────────────────────┤
│  CONNECTION                                                      │
│  \c database        التنقل لـ database                          │
│  \conninfo          معلومات الاتصال                              │
│  \q                 الخروج                                       │
├─────────────────────────────────────────────────────────────────┤
│  LISTING                                                         │
│  \l                 قائمة Databases                              │
│  \dt                قائمة Tables                                 │
│  \du                قائمة Users                                  │
│  \dn                قائمة Schemas                                │
│  \di                قائمة Indexes                                │
│  \dv                قائمة Views                                  │
├─────────────────────────────────────────────────────────────────┤
│  DESCRIBE                                                        │
│  \d table           وصف table                                    │
│  \d+ table          وصف تفصيلي                                   │
├─────────────────────────────────────────────────────────────────┤
│  EXECUTION                                                       │
│  \i file.sql        تنفيذ ملف                                    │
│  \e                 تحرير في editor                              │
│  \g                 إعادة تنفيذ آخر query                        │
├─────────────────────────────────────────────────────────────────┤
│  OUTPUT                                                          │
│  \x                 تفعيل/تعطيل العرض الموسع                     │
│  \o file            حفظ النتائج في ملف                          │
│  \timing            عرض وقت التنفيذ                              │
├─────────────────────────────────────────────────────────────────┤
│  HELP                                                            │
│  \?                 مساعدة psql                                  │
│  \h                 مساعدة SQL                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💡 نصائح Pro

<div dir="rtl">

### 1. ملف .psqlrc

أنشئ ملف `~/.psqlrc` لحفظ إعداداتك:

</div>

```sql
-- ~/.psqlrc

-- تفعيل timing دايماً
\timing on

-- تفعيل expanded display تلقائي
\x auto

-- تغيير الـ prompt
\set PROMPT1 '%n@%M:%>/%/ %R%# '

-- إظهار null بشكل واضح
\pset null '[NULL]'

-- حفظ History
\set HISTSIZE 2000
\set HISTCONTROL ignoredups

-- ألوان (على terminals تدعم)
\set COMP_KEYWORD_CASE upper
```

<div dir="rtl">

### 2. Aliases مفيدة

</div>

```sql
-- في .psqlrc
\set clear '\\! clear'
\set show_slow_queries 'SELECT pid, now() - pg_stat_activity.query_start AS duration, query FROM pg_stat_activity WHERE state != \'idle\' ORDER BY duration DESC;'
\set show_connections 'SELECT count(*) FROM pg_stat_activity;'
```

<div dir="rtl">

### 3. Password File

لتجنب كتابة الـ Password كل مرة:

</div>

```bash
# أنشئ ملف ~/.pgpass
echo "localhost:5432:*:postgres:your_password" > ~/.pgpass
chmod 600 ~/.pgpass
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ يقدر يتصل بـ psql
- [ ] ✅ يعرف يستخدم `\l`, `\dt`, `\d`
- [ ] ✅ يقدر يتنقل بين الـ databases
- [ ] ✅ يقدر ينفذ SQL queries
- [ ] ✅ يقدر ينفذ ملف SQL

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [إنشاء أول Database](./07-first-database.md)**

</div>

---

<div align="center">

[⬅️ السابق: إعداد pgAdmin](./05-pgadmin-setup.md) | [🏠 العودة للـ Module](../README.md)

</div>
