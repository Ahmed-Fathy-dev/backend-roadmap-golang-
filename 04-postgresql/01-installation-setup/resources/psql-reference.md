# مرجع أوامر psql 📋

<div dir="rtl">

دليل سريع لكل أوامر psql المهمة.

</div>

---

## 🔌 الاتصال

```bash
# اتصال بسيط
psql -U postgres

# اتصال بـ database معينة
psql -U postgres -d mydb

# اتصال بـ host معين
psql -h localhost -U postgres -d mydb -p 5432

# باستخدام Connection String
psql "postgresql://user:pass@localhost:5432/mydb"

# تنفيذ أمر واحد
psql -U postgres -c "SELECT version();"

# تنفيذ ملف SQL
psql -U postgres -d mydb -f script.sql
```

---

## 📋 أوامر المساعدة

| الأمر | الوظيفة |
|-------|---------|
| `\?` | مساعدة أوامر psql |
| `\h` | مساعدة SQL |
| `\h SELECT` | مساعدة أمر معين |

---

## 🗄️ أوامر الـ Databases

| الأمر | الوظيفة |
|-------|---------|
| `\l` | قائمة الـ Databases |
| `\l+` | قائمة تفصيلية |
| `\c dbname` | الاتصال بـ Database |
| `\conninfo` | معلومات الاتصال الحالي |

---

## 📊 أوامر الـ Tables

| الأمر | الوظيفة |
|-------|---------|
| `\dt` | قائمة الـ Tables |
| `\dt+` | قائمة تفصيلية |
| `\dt schema.*` | Tables في schema معين |
| `\d table` | وصف Table |
| `\d+ table` | وصف تفصيلي |

---

## 👤 أوامر الـ Users

| الأمر | الوظيفة |
|-------|---------|
| `\du` | قائمة الـ Users/Roles |
| `\du+` | قائمة تفصيلية |

---

## 🔍 أوامر استكشافية

| الأمر | الوظيفة |
|-------|---------|
| `\dn` | قائمة Schemas |
| `\di` | قائمة Indexes |
| `\dv` | قائمة Views |
| `\df` | قائمة Functions |
| `\ds` | قائمة Sequences |
| `\dx` | قائمة Extensions |

---

## 💻 أوامر التنفيذ

| الأمر | الوظيفة |
|-------|---------|
| `\i file.sql` | تنفيذ ملف SQL |
| `\e` | تحرير في Editor |
| `\g` | إعادة تنفيذ آخر Query |
| `\r` | مسح الـ Query الحالي |

---

## 📤 أوامر الـ Output

| الأمر | الوظيفة |
|-------|---------|
| `\x` | تفعيل/تعطيل العرض الموسع |
| `\x auto` | عرض موسع تلقائي |
| `\o file` | حفظ النتائج في ملف |
| `\o` | إيقاف الحفظ |
| `\timing` | عرض وقت التنفيذ |
| `\t` | إخفاء/إظهار Headers |
| `\a` | تبديل Alignment |

---

## 🔄 أوامر التصدير

```sql
-- تصدير لـ CSV
\copy (SELECT * FROM users) TO '/path/users.csv' CSV HEADER

-- استيراد من CSV
\copy users FROM '/path/users.csv' CSV HEADER

-- تصدير Table كامل
\copy users TO '/path/users.csv' CSV HEADER
```

---

## ⚙️ أوامر الإعدادات

| الأمر | الوظيفة |
|-------|---------|
| `\set VAR value` | تعيين متغير |
| `\unset VAR` | حذف متغير |
| `\echo text` | طباعة نص |
| `\pset format` | تغيير تنسيق الـ Output |

---

## 🚪 الخروج

| الأمر | الوظيفة |
|-------|---------|
| `\q` | خروج |
| `quit` | خروج |
| `exit` | خروج |
| `Ctrl+D` | خروج |

---

## 💡 اختصارات مفيدة

```sql
-- عرض الإصدار
SELECT version();

-- الـ Database الحالية
SELECT current_database();

-- الـ User الحالي
SELECT current_user;

-- الوقت الحالي
SELECT now();

-- حجم Database
SELECT pg_size_pretty(pg_database_size('mydb'));

-- حجم Table
SELECT pg_size_pretty(pg_total_relation_size('users'));
```

---

## 📁 ملف .psqlrc

```sql
-- ~/.psqlrc

-- إعدادات مفيدة
\timing on
\x auto
\pset null '[NULL]'
\set HISTSIZE 2000
\set PROMPT1 '%n@%M:%>/%/ %R%# '

-- اختصارات
\set show_slow 'SELECT pid, now() - pg_stat_activity.query_start AS duration, query FROM pg_stat_activity WHERE state != \'idle\' ORDER BY duration DESC;'
\set show_size 'SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database ORDER BY pg_database_size(datname) DESC;'
```

---

<div align="center">

[🏠 العودة للـ Module](../README.md)

</div>
