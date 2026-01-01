# FULL OUTER JOIN - الربط الخارجي الكامل 🔄

<div dir="rtl">

## مقدمة

`FULL OUTER JOIN` بيرجع كل الصفوف من الجدولين، مع NULL لما مفيش match.

**المدة المتوقعة:** 15 دقيقة

</div>

---

## 📊 كيف يعمل FULL OUTER JOIN؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                      FULL OUTER JOIN                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Table A              Table B              Result                   │
│  ┌────┬─────┐        ┌────┬─────┐         ┌─────┬─────┐            │
│  │ id │ val │        │ id │ val │         │  A  │  B  │            │
│  ├────┼─────┤        ├────┼─────┤         ├─────┼─────┤            │
│  │ 1  │  a  │        │ 1  │  x  │         │  a  │  x  │ ← match   │
│  │ 2  │  b  │        │ 3  │  y  │         │  b  │NULL │ ← A only  │
│  └────┴─────┘        │ 4  │  z  │         │NULL │  y  │ ← B only  │
│                      └────┴─────┘         │NULL │  z  │ ← B only  │
│                                           └─────┴─────┘            │
│                                                                      │
│  = LEFT JOIN + RIGHT JOIN                                           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة

```sql
SELECT columns
FROM table_a
FULL OUTER JOIN table_b ON table_a.id = table_b.a_id;

-- أو
FULL JOIN table_b ON ...
```

---

## 🔢 أمثلة

```sql
-- مقارنة بيانات من نظامين
SELECT
    COALESCE(old.product_id, new.product_id) AS product_id,
    old.name AS old_name,
    new.name AS new_name,
    CASE
        WHEN old.product_id IS NULL THEN 'New'
        WHEN new.product_id IS NULL THEN 'Deleted'
        WHEN old.name != new.name THEN 'Modified'
        ELSE 'Unchanged'
    END AS status
FROM old_products old
FULL OUTER JOIN new_products new ON old.product_id = new.product_id;

-- إيجاد الفروقات
SELECT *
FROM table_a a
FULL OUTER JOIN table_b b ON a.id = b.id
WHERE a.id IS NULL OR b.id IS NULL;
```

---

## 💡 متى تستخدم FULL OUTER JOIN؟

- **Data reconciliation**: مقارنة بيانات
- **Finding differences**: إيجاد الفروقات
- **Merging datasets**: دمج بيانات من مصادر مختلفة

---

## ⏭️ الدرس التالي

**➡️ [CROSS JOIN](./07-cross-join.md)**

---

<div align="center">

[⬅️ السابق](./05-outer-joins.md) | [🏠 العودة للـ Module](../README.md) | [التالي ➡️](./07-cross-join.md)

</div>
