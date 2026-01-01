# أنواع التاريخ والوقت 📅

<div dir="rtl">

## مقدمة

التعامل مع التواريخ والأوقات من أصعب الأجزاء في البرمجة. PostgreSQL بيقدم مجموعة قوية من الأنواع والدوال اللي بتسهل التعامل مع الوقت بشكل احترافي.

**المدة المتوقعة:** 25-30 دقيقة

</div>

---

## 📊 نظرة عامة

```
┌─────────────────────────────────────────────────────────────────┐
│                  PostgreSQL Date/Time Types                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DATE         ─── تاريخ فقط (2024-12-21)                        │
│               │                                                  │
│  TIME         ─── وقت فقط (14:30:00)                            │
│               │                                                  │
│  TIMESTAMP    ─── تاريخ + وقت (2024-12-21 14:30:00)             │
│               │   بدون timezone                                  │
│               │                                                  │
│  TIMESTAMPTZ  ─── تاريخ + وقت + timezone  ✅                    │
│               │   (2024-12-21 14:30:00+02)                       │
│               │                                                  │
│  INTERVAL     ─── فترة زمنية (3 days, 2 hours)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📆 DATE - التاريخ

<div dir="rtl">

### الخصائص

- **الصيغة:** YYYY-MM-DD
- **المدى:** 4713 BC إلى 5874897 AD
- **الحجم:** 4 bytes

</div>

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    birth_date DATE,
    hire_date DATE NOT NULL,
    contract_end DATE
);

INSERT INTO employees (name, birth_date, hire_date)
VALUES ('Ahmed Ali', '1990-05-15', '2020-01-01');

-- طرق مختلفة لإدخال التاريخ
INSERT INTO employees (name, hire_date) VALUES
    ('Sara', '2024-12-21'),                    -- ISO format ✅
    ('Omar', 'December 21, 2024'),             -- Verbose
    ('Fatima', '21/12/2024');                  -- DD/MM/YYYY (حسب الإعدادات)
```

<div dir="rtl">

### دوال التاريخ

</div>

```sql
-- التاريخ الحالي
SELECT CURRENT_DATE;               -- 2024-12-21
SELECT NOW()::DATE;                -- 2024-12-21

-- استخراج أجزاء التاريخ
SELECT EXTRACT(YEAR FROM DATE '2024-12-21');   -- 2024
SELECT EXTRACT(MONTH FROM DATE '2024-12-21');  -- 12
SELECT EXTRACT(DAY FROM DATE '2024-12-21');    -- 21
SELECT EXTRACT(DOW FROM DATE '2024-12-21');    -- 6 (Saturday, 0=Sunday)
SELECT EXTRACT(DOY FROM DATE '2024-12-21');    -- 356 (day of year)

-- الفرق بين تاريخين
SELECT DATE '2024-12-31' - DATE '2024-01-01';  -- 365 days

-- إضافة/طرح
SELECT DATE '2024-12-21' + 7;                  -- 2024-12-28
SELECT DATE '2024-12-21' - 30;                 -- 2024-11-21
SELECT DATE '2024-12-21' + INTERVAL '1 month'; -- 2025-01-21
```

---

## ⏰ TIME - الوقت

<div dir="rtl">

### الخصائص

- **الصيغة:** HH:MM:SS.microseconds
- **المدى:** 00:00:00 إلى 24:00:00
- **الحجم:** 8 bytes

</div>

```sql
CREATE TABLE schedule (
    id SERIAL PRIMARY KEY,
    event_name VARCHAR(100),
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    CHECK (end_time > start_time)
);

INSERT INTO schedule (event_name, start_time, end_time)
VALUES ('Morning Meeting', '09:00:00', '10:30:00');

-- طرق مختلفة لإدخال الوقت
INSERT INTO schedule (event_name, start_time, end_time) VALUES
    ('Lunch', '12:00', '13:00'),               -- بدون ثواني
    ('Presentation', '14:30:00', '16:00:00'),  -- مع ثواني
    ('Review', '16:30:00.5', '17:00:00');      -- مع milliseconds
```

<div dir="rtl">

### دوال الوقت

</div>

```sql
-- الوقت الحالي
SELECT CURRENT_TIME;                           -- 14:30:00.123456+02
SELECT LOCALTIME;                              -- 14:30:00.123456

-- استخراج أجزاء الوقت
SELECT EXTRACT(HOUR FROM TIME '14:30:45');     -- 14
SELECT EXTRACT(MINUTE FROM TIME '14:30:45');   -- 30
SELECT EXTRACT(SECOND FROM TIME '14:30:45');   -- 45

-- حسابات الوقت
SELECT TIME '10:00:00' + INTERVAL '2 hours';   -- 12:00:00
SELECT TIME '14:00:00' - TIME '10:00:00';      -- 04:00:00
```

---

## 🕐 TIMESTAMP - التاريخ والوقت

<div dir="rtl">

### TIMESTAMP (بدون Timezone)

- **الصيغة:** YYYY-MM-DD HH:MM:SS
- **ما يخزنش:** الـ timezone
- **الاستخدام:** أوقات محلية ثابتة

</div>

```sql
CREATE TABLE local_events (
    id SERIAL PRIMARY KEY,
    event_name VARCHAR(100),
    event_time TIMESTAMP NOT NULL
);

INSERT INTO local_events (event_name, event_time)
VALUES ('Local Meeting', '2024-12-21 14:30:00');

-- هيتخزن كما هو: 2024-12-21 14:30:00
-- مفيش معلومات عن الـ timezone
```

<div dir="rtl">

### TIMESTAMPTZ (مع Timezone) ✅

- **الصيغة:** YYYY-MM-DD HH:MM:SS+TZ
- **بيخزن:** UTC داخلياً
- **بيعرض:** حسب timezone الـ session
- ✅ **الموصى به للتطبيقات**

</div>

```sql
CREATE TABLE global_events (
    id SERIAL PRIMARY KEY,
    event_name VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    scheduled_for TIMESTAMPTZ NOT NULL
);

-- إدخال مع timezone مختلفة
INSERT INTO global_events (event_name, scheduled_for) VALUES
    ('Cairo Meeting', '2024-12-21 14:30:00+02'),    -- توقيت القاهرة
    ('NYC Meeting', '2024-12-21 14:30:00-05'),      -- توقيت نيويورك
    ('UTC Event', '2024-12-21 14:30:00+00');        -- UTC

-- تغيير timezone العرض
SET timezone = 'Africa/Cairo';
SELECT * FROM global_events;
-- NYC Meeting هيظهر: 2024-12-21 21:30:00+02 (بتوقيت القاهرة)

SET timezone = 'America/New_York';
SELECT * FROM global_events;
-- Cairo Meeting هيظهر: 2024-12-21 07:30:00-05 (بتوقيت نيويورك)
```

<div dir="rtl">

### TIMESTAMP vs TIMESTAMPTZ

</div>

```
┌────────────────────────────────────────────────────────────────────┐
│  TIMESTAMP (without timezone)                                       │
├────────────────────────────────────────────────────────────────────┤
│  - بيخزن التاريخ والوقت كما هو                                     │
│  - مفيش timezone                                                    │
│  - استخدم لما الوقت محلي وثابت                                     │
│    (مثلاً: مواعيد الشغل، مواعيد المحلات)                           │
├────────────────────────────────────────────────────────────────────┤
│  TIMESTAMPTZ (with timezone)  ✅                                    │
├────────────────────────────────────────────────────────────────────┤
│  - بيخزن كـ UTC داخلياً                                            │
│  - بيحول للـ timezone المطلوب لما بيعرض                            │
│  - استخدم للتطبيقات الـ global                                     │
│    (مثلاً: created_at, updated_at, schedules)                      │
└────────────────────────────────────────────────────────────────────┘
```

---

## ⏳ INTERVAL - الفترات الزمنية

<div dir="rtl">

### الخصائص

- **الوظيفة:** تمثيل فترة زمنية
- **الأمثلة:** 3 أيام، ساعتين، شهر و10 أيام

</div>

```sql
-- إنشاء intervals
SELECT INTERVAL '1 year';
SELECT INTERVAL '2 months';
SELECT INTERVAL '3 days';
SELECT INTERVAL '4 hours';
SELECT INTERVAL '5 minutes';
SELECT INTERVAL '6 seconds';

-- مركّب
SELECT INTERVAL '1 year 2 months 3 days 4 hours 5 minutes 6 seconds';
SELECT INTERVAL '1 year' + INTERVAL '6 months';  -- 1 year 6 mons

-- استخدامات عملية
SELECT NOW() + INTERVAL '30 days';     -- بعد 30 يوم
SELECT NOW() - INTERVAL '1 week';      -- قبل أسبوع
SELECT NOW() + INTERVAL '2 hours';     -- بعد ساعتين
```

<div dir="rtl">

### استخدامات شائعة

</div>

```sql
-- المستخدمين اللي سجلوا في آخر 7 أيام
SELECT * FROM users
WHERE created_at > NOW() - INTERVAL '7 days';

-- المستخدمين اللي ماسجلوش دخول من شهر
SELECT * FROM users
WHERE last_login < NOW() - INTERVAL '1 month';

-- الطلبات اللي اتعملت في آخر ساعة
SELECT * FROM orders
WHERE created_at > NOW() - INTERVAL '1 hour';

-- انتهاء الـ subscription بعد سنة
INSERT INTO subscriptions (user_id, starts_at, ends_at)
VALUES (1, NOW(), NOW() + INTERVAL '1 year');
```

---

## 🛠️ دوال التاريخ والوقت المهمة

<div dir="rtl">

### دوال الحصول على الوقت

</div>

```sql
-- الوقت الحالي
SELECT NOW();                      -- timestamp with timezone
SELECT CURRENT_TIMESTAMP;          -- نفس NOW()
SELECT CURRENT_DATE;               -- التاريخ فقط
SELECT CURRENT_TIME;               -- الوقت فقط
SELECT LOCALTIME;                  -- الوقت المحلي (بدون timezone)
SELECT LOCALTIMESTAMP;             -- التاريخ+الوقت المحلي

-- بداية اليوم/الشهر/السنة
SELECT DATE_TRUNC('day', NOW());   -- 2024-12-21 00:00:00
SELECT DATE_TRUNC('month', NOW()); -- 2024-12-01 00:00:00
SELECT DATE_TRUNC('year', NOW());  -- 2024-01-01 00:00:00
SELECT DATE_TRUNC('hour', NOW());  -- 2024-12-21 14:00:00
```

<div dir="rtl">

### تنسيق التاريخ

</div>

```sql
-- TO_CHAR للتنسيق
SELECT TO_CHAR(NOW(), 'YYYY-MM-DD');           -- 2024-12-21
SELECT TO_CHAR(NOW(), 'DD/MM/YYYY');           -- 21/12/2024
SELECT TO_CHAR(NOW(), 'Day, DD Month YYYY');   -- Saturday, 21 December 2024
SELECT TO_CHAR(NOW(), 'HH24:MI:SS');           -- 14:30:45
SELECT TO_CHAR(NOW(), 'HH12:MI AM');           -- 02:30 PM

-- أنماط التنسيق الشائعة
-- YYYY = السنة (4 أرقام)
-- MM   = الشهر (01-12)
-- DD   = اليوم (01-31)
-- HH24 = الساعة (00-23)
-- HH12 = الساعة (01-12)
-- MI   = الدقائق (00-59)
-- SS   = الثواني (00-59)
-- Day  = اسم اليوم كامل
-- Mon  = اسم الشهر مختصر
```

<div dir="rtl">

### تحويل النصوص لتواريخ

</div>

```sql
-- TO_DATE للتحويل من نص لتاريخ
SELECT TO_DATE('21/12/2024', 'DD/MM/YYYY');    -- 2024-12-21
SELECT TO_DATE('December 21, 2024', 'Month DD, YYYY');

-- TO_TIMESTAMP للتحويل من نص لـ timestamp
SELECT TO_TIMESTAMP('21-12-2024 14:30:00', 'DD-MM-YYYY HH24:MI:SS');
```

---

## 📅 استخدامات عملية

<div dir="rtl">

### 1. حساب العمر

</div>

```sql
SELECT
    name,
    birth_date,
    AGE(birth_date) AS age,
    EXTRACT(YEAR FROM AGE(birth_date)) AS age_years
FROM employees;

-- النتيجة
-- name  | birth_date | age              | age_years
-- Ahmed | 1990-05-15 | 34 years 7 mons  | 34
```

<div dir="rtl">

### 2. مدة العمل

</div>

```sql
SELECT
    name,
    hire_date,
    AGE(NOW(), hire_date) AS employment_duration,
    EXTRACT(YEAR FROM AGE(NOW(), hire_date)) AS years_employed
FROM employees;
```

<div dir="rtl">

### 3. التقارير الزمنية

</div>

```sql
-- مبيعات اليوم
SELECT SUM(amount) FROM orders
WHERE created_at::DATE = CURRENT_DATE;

-- مبيعات الأسبوع ده
SELECT SUM(amount) FROM orders
WHERE created_at >= DATE_TRUNC('week', NOW());

-- مبيعات الشهر ده
SELECT SUM(amount) FROM orders
WHERE created_at >= DATE_TRUNC('month', NOW());

-- مبيعات حسب اليوم
SELECT
    created_at::DATE AS date,
    COUNT(*) AS orders_count,
    SUM(amount) AS total_amount
FROM orders
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY created_at::DATE
ORDER BY date DESC;
```

<div dir="rtl">

### 4. جدولة المهام

</div>

```sql
CREATE TABLE scheduled_tasks (
    id SERIAL PRIMARY KEY,
    task_name VARCHAR(100),
    scheduled_for TIMESTAMPTZ NOT NULL,
    completed_at TIMESTAMPTZ,
    is_completed BOOLEAN DEFAULT FALSE
);

-- المهام القادمة
SELECT * FROM scheduled_tasks
WHERE scheduled_for > NOW() AND NOT is_completed
ORDER BY scheduled_for;

-- المهام المتأخرة
SELECT * FROM scheduled_tasks
WHERE scheduled_for < NOW() AND NOT is_completed;
```

---

## ⚠️ أخطاء شائعة

<div dir="rtl">

### 1. استخدام TIMESTAMP بدل TIMESTAMPTZ

</div>

```sql
-- ❌ مشكلة مع المستخدمين في timezones مختلفة
created_at TIMESTAMP DEFAULT NOW()

-- ✅ الصح
created_at TIMESTAMPTZ DEFAULT NOW()
```

<div dir="rtl">

### 2. مقارنة التواريخ بطريقة غلط

</div>

```sql
-- ❌ وحش - بيقارن timestamp مع date string
WHERE created_at = '2024-12-21'  -- مش هيرجع حاجة!

-- ✅ كويس
WHERE created_at::DATE = '2024-12-21'
-- أو
WHERE created_at >= '2024-12-21' AND created_at < '2024-12-22'
```

<div dir="rtl">

### 3. نسيان timezone

</div>

```sql
-- ⚠️ هيستخدم timezone الـ server
INSERT INTO events (scheduled_for) VALUES ('2024-12-21 14:00:00');

-- ✅ حدد الـ timezone
INSERT INTO events (scheduled_for) VALUES ('2024-12-21 14:00:00+02');
```

---

## 📋 جدول مرجعي

| النوع | الوصف | الاستخدام |
|-------|-------|----------|
| `DATE` | تاريخ فقط | تاريخ الميلاد، تاريخ الانتهاء |
| `TIME` | وقت فقط | مواعيد يومية |
| `TIMESTAMP` | تاريخ + وقت | أوقات محلية |
| `TIMESTAMPTZ` | تاريخ + وقت + timezone ✅ | التطبيقات العالمية |
| `INTERVAL` | فترة زمنية | حسابات الوقت |

---

## ✅ Key Takeaways

<div dir="rtl">

1. استخدم **TIMESTAMPTZ** للـ created_at, updated_at وأي وقت عالمي
2. استخدم **DATE** للتواريخ فقط (تاريخ الميلاد)
3. استخدم **INTERVAL** لحسابات الوقت
4. **NOW()** يعطي الوقت الحالي مع timezone
5. **DATE_TRUNC** لقطع الوقت لأقرب يوم/شهر/سنة

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [الأنواع الخاصة](./05-special-types.md)**

</div>

---

<div align="center">

[⬅️ السابق: أنواع النصوص](./03-text-types.md) | [🏠 العودة للـ Module](../README.md)

</div>
