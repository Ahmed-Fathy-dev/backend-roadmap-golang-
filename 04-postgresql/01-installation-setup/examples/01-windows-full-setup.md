# مثال: تثبيت كامل على Windows 🪟

<div dir="rtl">

## السيناريو

أنا مطور اسمي Ahmed، عندي Windows 11، وعايز أثبت PostgreSQL للتطوير المحلي. هنمشي مع بعض خطوة بخطوة.

</div>

---

## 📋 قبل البدء

<div dir="rtl">

### متطلبات Ahmed

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  نظام التشغيل: Windows 11 Pro                               │
│  RAM: 16GB                                                   │
│  Disk: 50GB free على C:                                     │
│  صلاحيات: Administrator                                     │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### خطة Ahmed

1. تثبيت PostgreSQL 16
2. إعداد pgAdmin
3. إنشاء database للمشروع
4. إنشاء user للتطبيق
5. اختبار الاتصال

</div>

---

## 🔽 الخطوة 1: التحميل

<div dir="rtl">

Ahmed فتح المتصفح وراح على:

</div>

```
https://www.postgresql.org/download/windows/
```

<div dir="rtl">

### اختيار الإصدار

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  EnterpriseDB Installer                                      │
│                                                              │
│  PostgreSQL 16.1 - Windows x86-64                           │
│  Download Size: 311 MB                                       │
│                                                              │
│  [Download] ← Ahmed ضغط هنا                                 │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

الملف اتحمل:
`postgresql-16.1-1-windows-x64.exe`

</div>

---

## ⚙️ الخطوة 2: التثبيت

<div dir="rtl">

### 2.1 تشغيل الـ Installer

Ahmed عمل double-click على الملف، واختار Yes في UAC.

### 2.2 اختيار المكونات

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Select Components                                           │
├─────────────────────────────────────────────────────────────┤
│  ☑️ PostgreSQL Server       ← السيرفر نفسه                  │
│  ☑️ pgAdmin 4               ← واجهة رسومية                  │
│  ☑️ Stack Builder           ← لتثبيت إضافات                 │
│  ☑️ Command Line Tools      ← psql وغيره                    │
│                                                              │
│  Space Required: 280 MB                                      │
│                                                              │
│  [Next]                                                      │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### 2.3 تحديد المسارات

Ahmed خلى المسارات default:

</div>

```
Installation Directory: C:\Program Files\PostgreSQL\16
Data Directory:         C:\Program Files\PostgreSQL\16\data
```

<div dir="rtl">

### 2.4 إعداد الـ Password

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Password                                                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Password: Postgres@Dev2024                                  │
│  Retype:   Postgres@Dev2024                                  │
│                                                              │
│  ⚠️ Ahmed كتب الـ Password في ملاحظاته المشفرة             │
│                                                              │
│  [Next]                                                      │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### 2.5 Port و Locale

</div>

```
Port: 5432 (default)
Locale: [Default locale]
```

<div dir="rtl">

### 2.6 بدء التثبيت

</div>

```
Installing PostgreSQL...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100%

✅ Installation complete!

☐ Launch Stack Builder at exit?  ← Ahmed شال العلامة دي
```

---

## ✅ الخطوة 3: التحقق

<div dir="rtl">

### 3.1 اختبار من PowerShell

Ahmed فتح PowerShell:

</div>

```powershell
PS C:\> psql --version
psql (PostgreSQL) 16.1
```

<div dir="rtl">

### 3.2 اختبار الاتصال

</div>

```powershell
PS C:\> psql -U postgres
Password for user postgres: ********

psql (16.1)
Type "help" for help.

postgres=#
```

<div dir="rtl">

### 3.3 التحقق من الـ Service

</div>

```powershell
PS C:\> Get-Service postgresql*

Status   Name               DisplayName
------   ----               -----------
Running  postgresql-x64-16  postgresql-x64-16 - PostgreSQL Server
```

---

## 🖥️ الخطوة 4: إعداد pgAdmin

<div dir="rtl">

### 4.1 فتح pgAdmin

Ahmed فتح pgAdmin من Start Menu

### 4.2 إنشاء Master Password

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Set Master Password                                         │
├─────────────────────────────────────────────────────────────┤
│  Password: pgAdmin@Master2024                                │
│  Confirm:  pgAdmin@Master2024                                │
│                                                              │
│  ℹ️ هذا الـ Password لـ pgAdmin نفسه، مش لـ PostgreSQL      │
│                                                              │
│  [OK]                                                        │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### 4.3 إضافة Server

Ahmed ضغط Right-click على "Servers" → Register → Server

**Tab: General**

</div>

```
Name: Local Development
```

<div dir="rtl">

**Tab: Connection**

</div>

```
Host: localhost
Port: 5432
Maintenance database: postgres
Username: postgres
Password: Postgres@Dev2024
☑️ Save password
```

<div dir="rtl">

### 4.4 التحقق من الاتصال

</div>

```
✅ Connected!

Servers
└── Local Development
    ├── Databases
    │   ├── postgres
    │   ├── template0
    │   └── template1
    └── Login/Group Roles
        └── postgres
```

---

## 📦 الخطوة 5: إنشاء Database للمشروع

<div dir="rtl">

Ahmed عايز يعمل database لمشروع Task Manager:

### 5.1 من psql

</div>

```sql
postgres=# CREATE DATABASE taskmanager;
CREATE DATABASE

postgres=# \l
                              List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype
-------------+----------+----------+-------------+-------------
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 taskmanager | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8
```

<div dir="rtl">

### 5.2 الاتصال بالـ Database الجديدة

</div>

```sql
postgres=# \c taskmanager
You are now connected to database "taskmanager" as user "postgres".
taskmanager=#
```

---

## 👤 الخطوة 6: إنشاء User للتطبيق

<div dir="rtl">

Ahmed عايز user منفصل للتطبيق بتاعه:

</div>

```sql
-- إنشاء User
taskmanager=# CREATE USER taskapp WITH PASSWORD 'TaskApp@2024';
CREATE ROLE

-- منح صلاحيات
taskmanager=# GRANT ALL PRIVILEGES ON DATABASE taskmanager TO taskapp;
GRANT

-- التحقق
taskmanager=# \du
                             List of roles
 Role name |                   Attributes
-----------+------------------------------------------------
 postgres  | Superuser, Create role, Create DB, Replication
 taskapp   |
```

---

## 📊 الخطوة 7: إنشاء Tables

```sql
-- الاتصال بـ taskmanager
taskmanager=# \c taskmanager

-- جدول المستخدمين
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- جدول المهام
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    status VARCHAR(20) DEFAULT 'pending',
    due_date DATE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- التحقق
taskmanager=# \dt
          List of relations
 Schema |  Name  | Type  |  Owner
--------+--------+-------+----------
 public | tasks  | table | postgres
 public | users  | table | postgres
```

---

## 🔐 الخطوة 8: ضبط صلاحيات التطبيق

```sql
-- صلاحيات على Schema
GRANT USAGE ON SCHEMA public TO taskapp;

-- صلاحيات على Tables
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO taskapp;

-- صلاحيات على Sequences
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO taskapp;

-- صلاحيات تلقائية للـ Tables الجديدة
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO taskapp;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT USAGE, SELECT ON SEQUENCES TO taskapp;
```

---

## ✅ الخطوة 9: اختبار الاتصال من التطبيق

<div dir="rtl">

Ahmed كتب Connection String في ملف `.env`:

</div>

```bash
# .env
DATABASE_URL=postgresql://taskapp:TaskApp@2024@localhost:5432/taskmanager?sslmode=disable
```

<div dir="rtl">

### اختبار من psql

</div>

```powershell
PS C:\> psql -U taskapp -d taskmanager -h localhost
Password for user taskapp: ********

taskmanager=>
```

<div dir="rtl">

### اختبار الصلاحيات

</div>

```sql
-- هيشتغل
taskmanager=> INSERT INTO users (username, email) VALUES ('test', 'test@test.com');
INSERT 0 1

taskmanager=> SELECT * FROM users;
 id | username |     email      |         created_at
----+----------+----------------+----------------------------
  1 | test     | test@test.com  | 2024-12-21 15:30:00.123456

taskmanager=> DELETE FROM users WHERE id = 1;
DELETE 1
```

---

## 📋 ملخص

<div dir="rtl">

### ما أنجزه Ahmed

</div>

```
✅ PostgreSQL 16.1 installed
✅ pgAdmin configured
✅ Database: taskmanager
✅ User: taskapp
✅ Tables: users, tasks
✅ Permissions configured
✅ Connection tested
```

<div dir="rtl">

### المعلومات المهمة

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  PostgreSQL Connection                                       │
├─────────────────────────────────────────────────────────────┤
│  Host:     localhost                                         │
│  Port:     5432                                              │
│  Database: taskmanager                                       │
│  User:     taskapp                                           │
│  Password: TaskApp@2024 (في ملف .env)                       │
│                                                              │
│  Connection String:                                          │
│  postgresql://taskapp:TaskApp@2024@localhost:5432/taskmanager│
├─────────────────────────────────────────────────────────────┤
│  Admin Access (للطوارئ)                                      │
│  User:     postgres                                          │
│  Password: Postgres@Dev2024 (في ملاحظات مشفرة)              │
└─────────────────────────────────────────────────────────────┘
```

---

<div align="center">

[🏠 العودة للـ Module](../README.md)

</div>
