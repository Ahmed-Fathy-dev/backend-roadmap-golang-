# Module 06: Advanced Topics - مواضيع متقدمة 🚀

<div dir="rtl">

## نظرة عامة

في الـ Module ده هنتعلم مواضيع متقدمة في PostgreSQL: الـ Indexing، Performance Tuning، JSON/JSONB، Full-Text Search، Partitioning، وغيرها.

**المستوى:** متقدم
**المدة المتوقعة:** 5-6 ساعات

</div>

---

## 📚 فهرس الدروس

### 📊 Indexing & Performance

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 01 | [Index Types](./lessons/01-index-types.md) | أنواع الـ Indexes | 30 دقيقة |
| 02 | [EXPLAIN ANALYZE](./lessons/02-explain-analyze.md) | تحليل الـ Query Plans | 25 دقيقة |
| 03 | [Query Optimization](./lessons/03-query-optimization.md) | تحسين الـ Queries | 30 دقيقة |

### 📦 JSON & JSONB

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 04 | [JSON/JSONB Basics](./lessons/04-json-jsonb.md) | التعامل مع JSON | 30 دقيقة |
| 05 | [JSONB Operators](./lessons/05-jsonb-operators.md) | عمليات JSONB | 25 دقيقة |
| 06 | [JSONB Indexing](./lessons/06-jsonb-indexing.md) | فهرسة JSONB | 20 دقيقة |

### 🔍 Full-Text Search

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 07 | [Full-Text Search](./lessons/07-full-text-search.md) | البحث النصي الكامل | 30 دقيقة |

### 📊 Advanced Features

| # | الدرس | الوصف | المدة |
|---|-------|-------|-------|
| 08 | [Views & Materialized Views](./lessons/08-views.md) | الـ Views | 25 دقيقة |
| 09 | [Partitioning](./lessons/09-partitioning.md) | تقسيم الجداول | 30 دقيقة |
| 10 | [Stored Procedures](./lessons/10-stored-procedures.md) | الـ Functions و Procedures | 25 دقيقة |

---

## 🎯 أهداف الـ Module

<div dir="rtl">

بعد الانتهاء من هذا الـ Module، ستتمكن من:

1. **فهم واستخدام** أنواع الـ Indexes المختلفة
2. **تحليل وتحسين** أداء الـ Queries
3. **التعامل مع** JSON/JSONB بكفاءة
4. **بناء** Full-Text Search
5. **استخدام** Views و Materialized Views
6. **تقسيم** الجداول الكبيرة (Partitioning)

</div>

---

## 📊 Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Advanced PostgreSQL Topics                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Performance                                                        │
│   ├── B-tree, Hash, GIN, GiST indexes                               │
│   ├── EXPLAIN ANALYZE                                                │
│   ├── Query planning                                                 │
│   └── Optimization techniques                                        │
│                                                                      │
│   JSON/JSONB                                                         │
│   ├── Storing semi-structured data                                  │
│   ├── Querying JSON fields                                          │
│   ├── GIN indexes for JSONB                                         │
│   └── Combining relational + document models                        │
│                                                                      │
│   Full-Text Search                                                   │
│   ├── tsvector & tsquery                                            │
│   ├── Search ranking                                                │
│   └── Multi-language support                                        │
│                                                                      │
│   Scalability                                                        │
│   ├── Table partitioning                                            │
│   ├── Materialized views                                            │
│   └── Connection pooling (PgBouncer)                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📂 هيكل الـ Module

```
06-advanced-topics/
├── README.md
├── lessons/
│   ├── 01-index-types.md
│   ├── 02-explain-analyze.md
│   ├── 03-query-optimization.md
│   ├── 04-json-jsonb.md
│   ├── 05-jsonb-operators.md
│   ├── 06-jsonb-indexing.md
│   ├── 07-full-text-search.md
│   ├── 08-views.md
│   ├── 09-partitioning.md
│   └── 10-stored-procedures.md
├── examples/
│   └── performance-tuning/
└── resources/
    ├── index-cheatsheet.md
    └── performance-checklist.md
```

---

## 📖 المتطلبات السابقة

<div dir="rtl">

قبل البدء في هذا الـ Module، تأكد من إتمام:

</div>

- [x] [Module 01: Installation & Setup](../01-installation-setup/README.md)
- [x] [Module 02: SQL Basics](../02-sql-basics/README.md)
- [x] [Module 03: CRUD Operations](../03-crud-operations/README.md)
- [x] [Module 04: Joins & Relations](../04-joins-relations/README.md)
- [x] [Module 05: Go + PostgreSQL](../05-go-postgres-integration/README.md)

---

## 🚀 ابدأ التعلم

<div dir="rtl">

ابدأ بالدرس الأول:

</div>

**➡️ [Index Types](./lessons/01-index-types.md)**

---

<div align="center">

[⬅️ Module السابق: Go + PostgreSQL](../05-go-postgres-integration/README.md) | [🏠 الرئيسية](../README.md)

</div>
