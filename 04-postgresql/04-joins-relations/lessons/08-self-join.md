# Self JOIN - ربط الجدول بنفسه 🔄

<div dir="rtl">

## مقدمة

Self JOIN هو ربط الجدول بنفسه - مفيد للبيانات الهرمية زي الموظفين والمديرين.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 متى تحتاج Self JOIN؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Self JOIN                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  employees                                                          │
│  ┌────┬─────────┬────────────┐                                      │
│  │ id │  name   │ manager_id │                                      │
│  ├────┼─────────┼────────────┤     Hierarchy:                       │
│  │ 1  │ Ahmed   │    NULL    │     Ahmed (CEO)                      │
│  │ 2  │ Sara    │     1      │       ├── Sara                       │
│  │ 3  │ Omar    │     1      │       ├── Omar                       │
│  │ 4  │ Fatima  │     2      │       │     └── Khaled               │
│  │ 5  │ Khaled  │     3      │       └── Fatima                     │
│  └────┴─────────┴────────────┘                                      │
│                                                                      │
│  Self JOIN: ربط employees بـ employees                              │
│  لإيجاد اسم المدير لكل موظف                                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

## 🔢 أمثلة عملية

```sql
-- الموظفين مع مديريهم
SELECT
    e.name AS employee,
    e.position,
    COALESCE(m.name, 'No Manager') AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- الموظفين اللي تحت مدير معين
SELECT e.name, e.position
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE m.name = 'Ahmed';

-- Categories الهرمية
SELECT
    c.name AS category,
    p.name AS parent_category
FROM categories c
LEFT JOIN categories p ON c.parent_id = p.id;

-- إيجاد duplicates
SELECT a.id, b.id, a.email
FROM users a
JOIN users b ON a.email = b.email AND a.id < b.id;
```

---

## 🌳 Recursive CTE للهياكل العميقة

```sql
-- كل المرؤوسين (مهما كان المستوى)
WITH RECURSIVE subordinates AS (
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE id = 1  -- CEO

    UNION ALL

    SELECT e.id, e.name, e.manager_id, s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

---

## ⏭️ الدرس التالي

**➡️ [Multiple JOINs](./09-multiple-joins.md)**

---

<div align="center">

[⬅️ السابق](./07-cross-join.md) | [🏠 العودة للـ Module](../README.md) | [التالي ➡️](./09-multiple-joins.md)

</div>
