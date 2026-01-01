# تثبيت PostgreSQL على macOS 🍎

<div dir="rtl">

## مقدمة

macOS من أفضل البيئات للتطوير، وتثبيت PostgreSQL عليه سهل جداً. هنشرح طريقتين: Homebrew (الأسهل) و Postgres.app (الأبسط).

**المدة المتوقعة:** 5-10 دقائق

</div>

---

## 🍺 الطريقة 1: Homebrew (الموصى بها)

<div dir="rtl">

### تثبيت Homebrew (لو مش موجود)

</div>

```bash
# تثبيت Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# التحقق
brew --version
```

<div dir="rtl">

### تثبيت PostgreSQL

</div>

```bash
# تحديث Homebrew
brew update

# تثبيت PostgreSQL
brew install postgresql@16

# ربط الإصدار (لو عندك إصدارات متعددة)
brew link postgresql@16
```

<div dir="rtl">

### تشغيل PostgreSQL

</div>

```bash
# تشغيل كـ Service (يشتغل تلقائياً مع الجهاز)
brew services start postgresql@16

# أو تشغيل يدوي (مرة واحدة)
pg_ctl -D /opt/homebrew/var/postgresql@16 start
```

<div dir="rtl">

### التحقق من التثبيت

</div>

```bash
# الإصدار
psql --version
# Output: psql (PostgreSQL) 16.x

# حالة الـ Service
brew services list | grep postgresql
# Output: postgresql@16 started

# الاتصال
psql postgres
```

---

## 🐘 الطريقة 2: Postgres.app (الأبسط)

<div dir="rtl">

### ما هو Postgres.app؟

تطبيق macOS عادي - بتحمله وتشغله وخلاص! مثالي للتطوير.

### التحميل

1. روح على: https://postgresapp.com/
2. حمّل أحدث إصدار
3. اسحب التطبيق لـ Applications

### التشغيل

1. افتح **Postgres** من Applications
2. اضغط **Initialize** (أول مرة بس)
3. خلاص! PostgreSQL شغال

</div>

```
┌─────────────────────────────────────────────┐
│  🐘 Postgres                                │
├─────────────────────────────────────────────┤
│                                             │
│  Server: localhost                          │
│  Port: 5432                                 │
│  Status: ● Running                          │
│                                             │
│  Databases:                                 │
│  • postgres                                 │
│  • template0                                │
│  • template1                                │
│  • [your_username]                          │
│                                             │
└─────────────────────────────────────────────┘
```

<div dir="rtl">

### إضافة Command Line Tools

</div>

```bash
# أضف للـ PATH
sudo mkdir -p /etc/paths.d && \
echo /Applications/Postgres.app/Contents/Versions/latest/bin | \
sudo tee /etc/paths.d/postgresapp

# افتح Terminal جديد وجرب
psql --version
```

---

## 🔧 الفرق بين الطريقتين

<div dir="rtl">

| الميزة | Homebrew | Postgres.app |
|--------|----------|--------------|
| **سهولة التثبيت** | ⚙️ Command line | ✅ Drag & Drop |
| **تحديثات** | `brew upgrade` | Download جديد |
| **تشغيل تلقائي** | ✅ brew services | ✅ مدمج |
| **إدارة إصدارات** | ✅ ممتازة | ⚠️ محدودة |
| **حجم** | أصغر | أكبر |
| **CLI tools** | ✅ تلقائي | محتاج إعداد |

**نصيحتي:**
- للمطورين: **Homebrew**
- للمبتدئين: **Postgres.app**

</div>

---

## 👤 الاتصال بـ PostgreSQL

<div dir="rtl">

### الطريقة الافتراضية

على macOS، PostgreSQL بينشئ database باسم الـ username بتاعك:

</div>

```bash
# الاتصال بدون parameters
# (هيتصل بـ database اسمها = username)
psql

# أو تحديد الـ database
psql postgres

# أو بـ parameters كاملة
psql -h localhost -p 5432 -U $USER -d postgres
```

<div dir="rtl">

### إنشاء Superuser

بشكل افتراضي، الـ macOS user بتاعك هو superuser:

</div>

```sql
-- داخل psql
-- شوف الـ users
\du

-- النتيجة
                             List of roles
 Role name |                   Attributes
-----------+------------------------------------------------
 ahmed     | Superuser, Create role, Create DB, Replication
```

---

## 🔐 إعداد postgres User

<div dir="rtl">

### إنشاء postgres user (لو مش موجود)

</div>

```bash
# من Terminal
createuser -s postgres

# أو من psql
psql -c "CREATE USER postgres WITH SUPERUSER PASSWORD 'your_password';"
```

<div dir="rtl">

### تعيين Password

</div>

```bash
psql -c "ALTER USER postgres WITH PASSWORD 'your_password';"
```

---

## 🛠️ أوامر الإدارة

<div dir="rtl">

### Homebrew

</div>

```bash
# تشغيل
brew services start postgresql@16

# إيقاف
brew services stop postgresql@16

# إعادة تشغيل
brew services restart postgresql@16

# حالة
brew services list

# Logs
tail -f /opt/homebrew/var/log/postgresql@16.log
```

<div dir="rtl">

### Postgres.app

- **تشغيل:** افتح التطبيق
- **إيقاف:** أقفل التطبيق (أو Quit من menu bar)
- **Logs:** من الـ menu bar → Show Logs

</div>

---

## 📁 مسارات الملفات

<div dir="rtl">

### Homebrew (Apple Silicon / M1/M2/M3)

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Binary:         /opt/homebrew/bin/psql                     │
│  Data Directory: /opt/homebrew/var/postgresql@16/           │
│  Config:         /opt/homebrew/var/postgresql@16/postgresql.conf │
│  Log:            /opt/homebrew/var/log/postgresql@16.log    │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### Homebrew (Intel Mac)

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Binary:         /usr/local/bin/psql                        │
│  Data Directory: /usr/local/var/postgresql@16/              │
│  Config:         /usr/local/var/postgresql@16/postgresql.conf │
│  Log:            /usr/local/var/log/postgresql@16.log       │
└─────────────────────────────────────────────────────────────┘
```

<div dir="rtl">

### Postgres.app

</div>

```
┌─────────────────────────────────────────────────────────────┐
│  Binary:         /Applications/Postgres.app/.../bin/psql    │
│  Data Directory: ~/Library/Application Support/Postgres/    │
│  Config:         ~/Library/Application Support/Postgres/.../postgresql.conf │
└─────────────────────────────────────────────────────────────┘
```

---

## 🗑️ إلغاء التثبيت

<div dir="rtl">

### Homebrew

</div>

```bash
# إيقاف الـ Service
brew services stop postgresql@16

# إلغاء التثبيت
brew uninstall postgresql@16

# حذف البيانات (اختياري - خطير!)
rm -rf /opt/homebrew/var/postgresql@16
```

<div dir="rtl">

### Postgres.app

</div>

```bash
# 1. أقفل التطبيق
# 2. احذف من Applications
rm -rf /Applications/Postgres.app

# 3. احذف البيانات (اختياري)
rm -rf ~/Library/Application\ Support/Postgres
```

---

## ❌ حل المشاكل الشائعة

<div dir="rtl">

### مشكلة 1: psql not found

</div>

```bash
# Homebrew - تأكد من الـ PATH
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Postgres.app - أضف للـ PATH
echo 'export PATH="/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

<div dir="rtl">

### مشكلة 2: Port 5432 مشغول

</div>

```bash
# شوف مين شغال
lsof -i :5432

# أقفل الـ Process
kill -9 <PID>

# أو غيّر الـ Port في postgresql.conf
```

<div dir="rtl">

### مشكلة 3: Permission denied

</div>

```bash
# تأكد من صلاحيات Data Directory
sudo chown -R $(whoami) /opt/homebrew/var/postgresql@16
```

<div dir="rtl">

### مشكلة 4: Database not found

</div>

```bash
# أنشئ database باسم الـ user
createdb $(whoami)
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ PostgreSQL installed
- [ ] ✅ `psql --version` شغال
- [ ] ✅ PostgreSQL running
- [ ] ✅ يقدر يتصل (`psql postgres`)
- [ ] ✅ الـ PATH مُعد صح

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [إعداد pgAdmin](./05-pgadmin-setup.md)**

</div>

---

<div align="center">

[⬅️ السابق: تثبيت على Linux](./03-installation-linux.md) | [🏠 العودة للـ Module](../README.md)

</div>
