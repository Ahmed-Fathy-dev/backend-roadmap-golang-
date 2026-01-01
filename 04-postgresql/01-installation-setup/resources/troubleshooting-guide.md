# دليل حل المشاكل الشائعة 🔧

<div dir="rtl">

## مقدمة

هذا الدليل يغطي أشهر المشاكل اللي ممكن تواجهك مع PostgreSQL وكيفية حلها.

</div>

---

## 🚫 مشاكل التثبيت

### المشكلة 1: Installation Failed - Port in Use

<div dir="rtl">

**الوصف:** الـ Installer فشل لأن Port 5432 مشغول.

</div>

```
Error: The specified port 5432 is already in use.
```

<div dir="rtl">

**الأسباب:**
- PostgreSQL قديم مثبت
- تطبيق آخر يستخدم الـ Port

**الحل:**

</div>

```powershell
# شوف مين على الـ Port
netstat -ano | findstr 5432

# النتيجة
TCP    0.0.0.0:5432    0.0.0.0:0    LISTENING    1234
                                                  ↑ PID

# شوف الـ Process
tasklist | findstr 1234

# لو PostgreSQL قديم
net stop postgresql-x64-15
# أو أقفله من Task Manager

# أو استخدم port تاني في التثبيت: 5433
```

---

### المشكلة 2: Installation Failed - Permission Denied

<div dir="rtl">

**الوصف:** مفيش صلاحيات كافية.

</div>

```
Error: Unable to write to the selected directory
```

<div dir="rtl">

**الحل:**
1. شغّل الـ Installer كـ Administrator (Right-click → Run as administrator)
2. أو غيّر مسار التثبيت لمكان عندك صلاحيات عليه

</div>

---

## 🔌 مشاكل الاتصال

### المشكلة 3: Connection Refused

<div dir="rtl">

**الوصف:** مش قادر يتصل بالـ Server.

</div>

```
psql: error: connection to server at "localhost" (127.0.0.1), port 5432 failed:
Connection refused
Is the server running on that host and accepting TCP/IP connections?
```

<div dir="rtl">

**الأسباب:**
- PostgreSQL مش شغال
- الـ Service متوقف

**الحل:**

</div>

```powershell
# Windows - تشغيل الـ Service
net start postgresql-x64-16

# أو من Services
services.msc → postgresql-x64-16 → Start
```

```bash
# Linux
sudo systemctl start postgresql
sudo systemctl status postgresql
```

---

### المشكلة 4: Password Authentication Failed

<div dir="rtl">

**الوصف:** الـ Password غلط.

</div>

```
psql: error: connection to server at "localhost" (127.0.0.1), port 5432 failed:
FATAL: password authentication failed for user "postgres"
```

<div dir="rtl">

**الحل 1: تأكد من الـ Password**

- تأكد إنك بتكتب الـ Password الصح
- تأكد من الـ Caps Lock

**الحل 2: Reset الـ Password**

</div>

```sql
-- 1. عدّل pg_hba.conf
-- غيّر: scram-sha-256 → trust

-- 2. أعد تشغيل PostgreSQL

-- 3. اتصل بدون password
psql -U postgres

-- 4. غيّر الـ Password
ALTER USER postgres WITH PASSWORD 'new_password';

-- 5. رجّع pg_hba.conf للأصل
-- غيّر: trust → scram-sha-256

-- 6. أعد تشغيل PostgreSQL
```

<div dir="rtl">

**مسار pg_hba.conf:**

</div>

```
Windows: C:\Program Files\PostgreSQL\16\data\pg_hba.conf
Linux:   /etc/postgresql/16/main/pg_hba.conf
macOS:   /opt/homebrew/var/postgresql@16/pg_hba.conf
```

---

### المشكلة 5: psql Command Not Found

<div dir="rtl">

**الوصف:** الـ Terminal مش عارف الأمر `psql`.

</div>

```
'psql' is not recognized as an internal or external command
```

<div dir="rtl">

**الحل: أضف PostgreSQL للـ PATH**

</div>

```powershell
# Windows - أضف للـ PATH
[Environment]::SetEnvironmentVariable("Path",
    "$env:Path;C:\Program Files\PostgreSQL\16\bin",
    "Machine")

# أو يدوياً:
# System Properties → Environment Variables → Path → Edit → Add:
# C:\Program Files\PostgreSQL\16\bin
```

```bash
# Linux/macOS
echo 'export PATH="/usr/lib/postgresql/16/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

### المشكلة 6: Peer Authentication Failed (Linux)

<div dir="rtl">

**الوصف:** على Linux، الـ peer authentication فاشل.

</div>

```
psql: error: FATAL: Peer authentication failed for user "postgres"
```

<div dir="rtl">

**الحل 1: استخدم sudo**

</div>

```bash
sudo -u postgres psql
```

<div dir="rtl">

**الحل 2: غيّر الـ Authentication Method**

عدّل `/etc/postgresql/16/main/pg_hba.conf`:

</div>

```
# قبل
local   all   all                 peer

# بعد
local   all   all                 scram-sha-256
```

```bash
sudo systemctl reload postgresql
```

---

## ⚙️ مشاكل الأداء

### المشكلة 7: Database بطيئة

<div dir="rtl">

**الأسباب والحلول:**

**1. مفيش Indexes**

</div>

```sql
-- شوف الـ Slow Queries
SELECT
    query,
    calls,
    mean_time,
    total_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- أضف Index للأعمدة اللي بتبحث فيها كتير
CREATE INDEX idx_users_email ON users(email);
```

<div dir="rtl">

**2. الإحصائيات قديمة**

</div>

```sql
ANALYZE;
-- أو لـ table معين
ANALYZE users;
```

<div dir="rtl">

**3. محتاج VACUUM**

</div>

```sql
VACUUM ANALYZE;
-- أو Full vacuum (أبطأ بس أعمق)
VACUUM FULL;
```

---

### المشكلة 8: Too Many Connections

<div dir="rtl">

**الوصف:** وصلت لأقصى عدد اتصالات.

</div>

```
FATAL: too many connections for role "postgres"
```

<div dir="rtl">

**الحل 1: شوف الاتصالات الحالية**

</div>

```sql
SELECT count(*) FROM pg_stat_activity;

-- تفاصيل الاتصالات
SELECT pid, usename, application_name, state, query
FROM pg_stat_activity
WHERE state != 'idle';
```

<div dir="rtl">

**الحل 2: أقفل الاتصالات القديمة**

</div>

```sql
-- أقفل اتصالات idle لأكتر من ساعة
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle'
  AND state_change < now() - interval '1 hour';
```

<div dir="rtl">

**الحل 3: زوّد max_connections**

في `postgresql.conf`:

</div>

```conf
max_connections = 200  # الافتراضي 100
```

---

## 🗄️ مشاكل البيانات

### المشكلة 9: Disk Full

<div dir="rtl">

**الوصف:** مفيش مساحة كافية.

</div>

```
ERROR: could not extend file: No space left on device
```

<div dir="rtl">

**الحل:**

</div>

```sql
-- 1. شوف حجم الـ Databases
SELECT
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- 2. شوف حجم الـ Tables
SELECT
    tablename,
    pg_size_pretty(pg_total_relation_size(tablename::text)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::text) DESC;

-- 3. امسح البيانات القديمة
DELETE FROM logs WHERE created_at < now() - interval '30 days';

-- 4. اعمل VACUUM FULL
VACUUM FULL;
```

---

### المشكلة 10: Table Corrupted

<div dir="rtl">

**الوصف:** البيانات متلفة.

</div>

```
ERROR: invalid page in block X of relation Y
```

<div dir="rtl">

**الحل:**

</div>

```sql
-- 1. جرّب Reindex
REINDEX TABLE table_name;

-- 2. لو ما نفع، جرب
SET zero_damaged_pages = on;
VACUUM FULL table_name;
SET zero_damaged_pages = off;

-- 3. لو مازال فيه مشكلة، Restore من Backup
pg_restore -d dbname backup.dump
```

---

## 🔐 مشاكل الأمان

### المشكلة 11: SSL Connection Required

<div dir="rtl">

**الوصف:** السيرفر يتطلب SSL.

</div>

```
FATAL: no pg_hba.conf entry for host "...", SSL off
```

<div dir="rtl">

**الحل:**

</div>

```bash
# أضف sslmode للـ Connection String
psql "postgresql://user:pass@host/db?sslmode=require"
```

---

### المشكلة 12: Permission Denied on Table

<div dir="rtl">

**الوصف:** الـ User مالوش صلاحيات.

</div>

```
ERROR: permission denied for table users
```

<div dir="rtl">

**الحل:**

</div>

```sql
-- كـ superuser (postgres)
GRANT SELECT, INSERT, UPDATE, DELETE ON users TO myuser;

-- أو كل الـ Tables
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;
```

---

## 🐳 مشاكل Docker

### المشكلة 13: Container Exits Immediately

<div dir="rtl">

**الوصف:** الـ Container بيقفل فوراً.

**الحل:**

</div>

```bash
# شوف الـ Logs
docker logs postgres-container

# غالباً محتاج POSTGRES_PASSWORD
docker run -e POSTGRES_PASSWORD=secret -d postgres
```

---

### المشكلة 14: Data Not Persisted

<div dir="rtl">

**الوصف:** البيانات بتتمسح لما الـ Container يتوقف.

**الحل: استخدم Volume**

</div>

```bash
docker run \
    -e POSTGRES_PASSWORD=secret \
    -v postgres_data:/var/lib/postgresql/data \
    -d postgres
```

---

## 📋 Quick Troubleshooting Checklist

<div dir="rtl">

### لما حاجة مش شغالة:

</div>

```
□ PostgreSQL Service شغال؟
  → net start postgresql-x64-16 (Windows)
  → sudo systemctl status postgresql (Linux)

□ Port 5432 مفتوح ومش مشغول؟
  → netstat -ano | findstr 5432

□ الـ User والـ Password صح؟
  → جرب من psql مباشرة

□ الـ Database موجودة؟
  → \l في psql

□ الـ pg_hba.conf مظبوط؟
  → تأكد من الـ authentication method

□ الـ Firewall مش بيمنع؟
  → ufw allow 5432 (Linux)

□ الـ Logs بتقول إيه؟
  → tail -f /var/log/postgresql/*.log
  → C:\Program Files\PostgreSQL\16\data\log\
```

---

## 📞 محتاج مساعدة أكتر؟

<div dir="rtl">

1. **Official Documentation:** https://www.postgresql.org/docs/
2. **Stack Overflow:** ابحث عن الـ Error message
3. **PostgreSQL Mailing Lists:** https://www.postgresql.org/list/

</div>

---

<div align="center">

[🏠 العودة للـ Module](../README.md)

</div>
