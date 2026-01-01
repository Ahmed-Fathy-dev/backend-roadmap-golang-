# CROSS JOIN - الربط التقاطعي ✖️

<div dir="rtl">

## مقدمة

`CROSS JOIN` بينتج Cartesian Product - كل صف من الجدول الأول مع كل صف من الجدول التاني.

**المدة المتوقعة:** 15 دقيقة

</div>

---

## 📊 كيف يعمل CROSS JOIN؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CROSS JOIN                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  colors (3 rows)     sizes (2 rows)      Result (3 × 2 = 6 rows)   │
│  ┌─────────┐         ┌─────────┐         ┌─────────┬─────────┐     │
│  │  color  │    ×    │  size   │    =    │  color  │  size   │     │
│  ├─────────┤         ├─────────┤         ├─────────┼─────────┤     │
│  │  Red    │         │   S     │         │  Red    │   S     │     │
│  │  Blue   │         │   M     │         │  Red    │   M     │     │
│  │  Green  │         └─────────┘         │  Blue   │   S     │     │
│  └─────────┘                             │  Blue   │   M     │     │
│                                          │  Green  │   S     │     │
│                                          │  Green  │   M     │     │
│                                          └─────────┴─────────┘     │
│                                                                      │
│  ⚠️ كل تركيبة ممكنة!                                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📝 الصيغة

```sql
SELECT columns
FROM table_a
CROSS JOIN table_b;

-- أو بالطريقة القديمة
SELECT columns
FROM table_a, table_b;
```

---

## 🔢 أمثلة عملية

```sql
-- توليد كل تركيبات الألوان والمقاسات
SELECT
    c.name AS color,
    s.name AS size,
    c.name || ' - ' || s.name AS variant
FROM colors c
CROSS JOIN sizes s;

-- توليد كل أيام الأسبوع مع ساعات العمل
SELECT
    d.day_name,
    h.hour
FROM (
    VALUES ('Sunday'), ('Monday'), ('Tuesday'),
           ('Wednesday'), ('Thursday')
) AS d(day_name)
CROSS JOIN generate_series(9, 17) AS h(hour);

-- ربط كل منتج بكل متجر (لتتبع المخزون)
INSERT INTO store_inventory (store_id, product_id, quantity)
SELECT s.id, p.id, 0
FROM stores s
CROSS JOIN products p;
```

---

## ⚠️ تحذير الأداء

```sql
-- ⚠️ CROSS JOIN ينتج rows كتير جداً!
-- Table A: 1,000 rows
-- Table B: 1,000 rows
-- Result: 1,000,000 rows!

-- استخدمه بحذر ومع LIMIT
SELECT *
FROM large_table_a
CROSS JOIN large_table_b
LIMIT 100;
```

---

## ⏭️ الدرس التالي

**➡️ [Self JOIN](./08-self-join.md)**

---

<div align="center">

[⬅️ السابق](./06-full-outer-join.md) | [🏠 العودة للـ Module](../README.md) | [التالي ➡️](./08-self-join.md)

</div>
