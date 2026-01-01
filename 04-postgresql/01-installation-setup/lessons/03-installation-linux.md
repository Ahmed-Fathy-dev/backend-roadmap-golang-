# تثبيت PostgreSQL على Linux 🐧

<div dir="rtl">

## مقدمة

Linux هو البيئة الأكثر شيوعاً لتشغيل PostgreSQL في Production. في الدرس ده هنتعلم التثبيت على أشهر توزيعات Linux: Ubuntu/Debian و Fedora/CentOS.

**المدة المتوقعة:** 10-15 دقيقة

</div>

---

## 🐧 Ubuntu / Debian

<div dir="rtl">

### الطريقة 1: من الـ Official Repository (الأسهل)

</div>

```bash
# تحديث الـ Package List
sudo apt update

# تثبيت PostgreSQL
sudo apt install postgresql postgresql-contrib

# التحقق من الإصدار
psql --version
```

<div dir="rtl">

### الطريقة 2: من PostgreSQL Official Repository (أحدث إصدار)

</div>

```bash
# إضافة الـ PostgreSQL Repository

# 1. Import the repository signing key
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# 2. إنشاء ملف الـ Repository
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# 3. تحديث وتثبيت
sudo apt update
sudo apt install postgresql-16 postgresql-contrib-16
```

<div dir="rtl">

### التحقق من التثبيت

</div>

```bash
# تحقق من الإصدار
psql --version
# Output: psql (PostgreSQL) 16.x

# تحقق إن الـ Service شغال
sudo systemctl status postgresql
```

**النتيجة المتوقعة:**
```
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled)
     Active: active (exited) since ...
```

---

## 🎩 Fedora / CentOS / RHEL

<div dir="rtl">

### على Fedora

</div>

```bash
# تثبيت PostgreSQL
sudo dnf install postgresql-server postgresql-contrib

# تهيئة الـ Database
sudo postgresql-setup --initdb

# تشغيل الـ Service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# التحقق
psql --version
```

<div dir="rtl">

### على CentOS / RHEL

</div>

```bash
# إضافة الـ PostgreSQL Repository
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# تعطيل الـ Built-in PostgreSQL
sudo dnf -qy module disable postgresql

# تثبيت PostgreSQL 16
sudo dnf install -y postgresql16-server postgresql16-contrib

# تهيئة الـ Database
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb

# تشغيل الـ Service
sudo systemctl start postgresql-16
sudo systemctl enable postgresql-16
```

---

## 🔧 إعداد الـ Service

<div dir="rtl">

### أوامر systemctl الأساسية

</div>

```bash
# تشغيل PostgreSQL
sudo systemctl start postgresql

# إيقاف PostgreSQL
sudo systemctl stop postgresql

# إعادة تشغيل
sudo systemctl restart postgresql

# إعادة تحميل الإعدادات (بدون إيقاف)
sudo systemctl reload postgresql

# التحقق من الحالة
sudo systemctl status postgresql

# تفعيل التشغيل التلقائي
sudo systemctl enable postgresql

# تعطيل التشغيل التلقائي
sudo systemctl disable postgresql
```

---

## 👤 الاتصال بـ PostgreSQL

<div dir="rtl">

### فهم نظام المستخدمين

PostgreSQL على Linux بيستخدم نظام "Peer Authentication" افتراضياً:

```
┌─────────────────────────────────────────────────────┐
│  Linux User    ──────►   PostgreSQL User            │
│  postgres      ──────►   postgres (superuser)       │
└─────────────────────────────────────────────────────┘
```

### الاتصال كـ postgres user

</div>

```bash
# الطريقة 1: التحويل لـ postgres user
sudo -i -u postgres
psql

# الطريقة 2: في أمر واحد
sudo -u postgres psql
```

**النتيجة:**
```
psql (16.1)
Type "help" for help.

postgres=#
```

<div dir="rtl">

### تنفيذ أمر تجريبي

</div>

```sql
-- داخل psql
SELECT version();

-- النتيجة
                              version
--------------------------------------------------------------------
 PostgreSQL 16.1 on x86_64-pc-linux-gnu, compiled by gcc ...

-- للخروج
\q
```

---

## 🔐 إعداد Password للـ postgres User

<div dir="rtl">

### لماذا محتاجين Password؟

بشكل افتراضي، مفيش password للـ postgres user (بيستخدم Peer Authentication). بس لو عايز تتصل من:
- Application (Go, Python, etc.)
- pgAdmin
- جهاز تاني

محتاج تعمل Password.

### إعداد الـ Password

</div>

```bash
# الاتصال كـ postgres
sudo -u postgres psql

# تغيير الـ Password
ALTER USER postgres WITH PASSWORD 'your_secure_password';

# للخروج
\q
```

<div dir="rtl">

### تعديل ملف pg_hba.conf

عشان تقدر تتصل بـ Password، لازم تعدل ملف الـ Authentication:

</div>

```bash
# إيجاد مسار الملف
sudo -u postgres psql -c "SHOW hba_file;"
# Usually: /etc/postgresql/16/main/pg_hba.conf
# Or:      /var/lib/pgsql/16/data/pg_hba.conf
```

```bash
# تعديل الملف
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

<div dir="rtl">

### قبل التعديل:

</div>

```
# TYPE  DATABASE  USER      ADDRESS         METHOD
local   all       all                       peer
host    all       all       127.0.0.1/32    ident
```

<div dir="rtl">

### بعد التعديل:

</div>

```
# TYPE  DATABASE  USER      ADDRESS         METHOD
local   all       all                       scram-sha-256
host    all       all       127.0.0.1/32    scram-sha-256
host    all       all       ::1/128         scram-sha-256
```

```bash
# إعادة تحميل الإعدادات
sudo systemctl reload postgresql
```

<div dir="rtl">

### الآن تقدر تتصل بـ Password

</div>

```bash
psql -U postgres -h localhost
# Password for user postgres: ********
```

---

## 🌐 السماح بالاتصال من الخارج

<div dir="rtl">

### 1. تعديل postgresql.conf

</div>

```bash
# إيجاد مسار الملف
sudo -u postgres psql -c "SHOW config_file;"

# تعديل الملف
sudo nano /etc/postgresql/16/main/postgresql.conf
```

<div dir="rtl">

### تغيير listen_addresses

</div>

```conf
# قبل
#listen_addresses = 'localhost'

# بعد (للسماح من أي IP)
listen_addresses = '*'

# أو لـ IPs محددة
listen_addresses = 'localhost, 192.168.1.100'
```

<div dir="rtl">

### 2. تعديل pg_hba.conf

</div>

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

```
# إضافة سطر للسماح بالاتصال من شبكة معينة
host    all    all    192.168.1.0/24    scram-sha-256

# أو من أي IP (خطير في Production!)
host    all    all    0.0.0.0/0         scram-sha-256
```

<div dir="rtl">

### 3. إعادة تشغيل PostgreSQL

</div>

```bash
sudo systemctl restart postgresql
```

<div dir="rtl">

### 4. فتح الـ Firewall

</div>

```bash
# Ubuntu (UFW)
sudo ufw allow 5432/tcp

# CentOS/Fedora (firewalld)
sudo firewall-cmd --permanent --add-port=5432/tcp
sudo firewall-cmd --reload
```

---

## 📁 مسارات الملفات المهمة

<div dir="rtl">

### Ubuntu/Debian

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Config Files:   /etc/postgresql/16/main/                   │
│  Data Directory: /var/lib/postgresql/16/main/               │
│  Log Files:      /var/log/postgresql/                       │
│  Binary Files:   /usr/lib/postgresql/16/bin/                │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### CentOS/Fedora

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Config Files:   /var/lib/pgsql/16/data/                    │
│  Data Directory: /var/lib/pgsql/16/data/                    │
│  Log Files:      /var/lib/pgsql/16/data/log/                │
│  Binary Files:   /usr/pgsql-16/bin/                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 🐳 Docker Alternative

<div dir="rtl">

### لو مش عايز تثبت PostgreSQL مباشرة

</div>

```bash
# تشغيل PostgreSQL في Docker
docker run --name postgres-dev \
    -e POSTGRES_PASSWORD=mysecretpassword \
    -p 5432:5432 \
    -d postgres:16

# الاتصال
psql -h localhost -U postgres
```

<div dir="rtl">

### Docker Compose

</div>

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:16
    container_name: postgres-dev
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
# تشغيل
docker-compose up -d

# الاتصال
psql -h localhost -U postgres -d myapp
```

---

## ❌ حل المشاكل الشائعة

<div dir="rtl">

### مشكلة 1: Permission denied

</div>

```bash
# لو ظهر:
psql: error: could not connect to server: Permission denied

# الحل: استخدم sudo -u postgres
sudo -u postgres psql
```

<div dir="rtl">

### مشكلة 2: Service مش بيشتغل

</div>

```bash
# شوف الـ Logs
sudo journalctl -u postgresql -n 50

# أو
sudo tail -f /var/log/postgresql/postgresql-16-main.log
```

<div dir="rtl">

### مشكلة 3: Port مشغول

</div>

```bash
# شوف مين شغال على الـ Port
sudo lsof -i :5432

# أو
sudo netstat -tulpn | grep 5432
```

<div dir="rtl">

### مشكلة 4: Authentication failed

</div>

```bash
# تأكد من pg_hba.conf
sudo cat /etc/postgresql/16/main/pg_hba.conf

# تأكد إن الـ Method هو scram-sha-256 أو md5
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ PostgreSQL installed
- [ ] ✅ Service running (`systemctl status postgresql`)
- [ ] ✅ يقدر يتصل (`sudo -u postgres psql`)
- [ ] ✅ Password configured
- [ ] ✅ يقدر يتصل بـ Password (`psql -U postgres -h localhost`)

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [تثبيت PostgreSQL على macOS](./04-installation-macos.md)**

أو لو مش محتاج macOS:

**➡️ [إعداد pgAdmin](./05-pgadmin-setup.md)**

</div>

---

<div align="center">

[⬅️ السابق: تثبيت على Windows](./02-installation-windows.md) | [🏠 العودة للـ Module](../README.md)

</div>
