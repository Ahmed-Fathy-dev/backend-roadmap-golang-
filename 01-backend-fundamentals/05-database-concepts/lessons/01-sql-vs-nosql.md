# Lesson 1: SQL vs NoSQL 🗄️

<div dir="rtl">

## المقدمة

**متى تستخدم SQL؟ متى NoSQL؟**

الاختيار الصحيح = فرق كبير في الأداء!

</div>

---

## 🔍 SQL (Relational) Databases

**Examples:** PostgreSQL, MySQL, SQL Server

### Structure:

```
Tables with fixed schema:

users table:
┌────┬────────┬─────────────────┬─────┐
│ id │  name  │      email      │ age │
├────┼────────┼─────────────────┼─────┤
│ 1  │ Ahmed  │ ahmed@test.com  │ 25  │
│ 2  │ Sara   │ sara@test.com   │ 30  │
└────┴────────┴─────────────────┴─────┘
```

### Characteristics:

```
✅ Fixed schema (structure defined)
✅ ACID transactions
✅ Relationships (Foreign Keys)
✅ SQL language (standardized)
✅ Vertical scaling (bigger server)
```

### When to Use SQL:

```
✅ Complex relationships
✅ ACID requirements (banking, finance)
✅ Complex queries (JOINs, aggregations)
✅ Structured data
✅ Data integrity critical

Examples:
- Banking systems
- E-commerce (orders, products)
- CRM systems
- Traditional apps
```

---

## 📦 NoSQL Databases

**Examples:** MongoDB, Redis, Cassandra, DynamoDB

### Structure:

```json
// Document (MongoDB):
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "Ahmed",
  "email": "ahmed@test.com",
  "age": 25,
  "addresses": [
    // Nested data!
    {
      "type": "home",
      "city": "Cairo"
    }
  ]
}
```

### Characteristics:

```
✅ Flexible schema (no fixed structure)
✅ Horizontal scaling (more servers)
✅ Fast for simple queries
✅ Good for unstructured data
❌ Limited transactions
❌ Weaker consistency (eventual)
```

### When to Use NoSQL:

```
✅ Flexible/changing schema
✅ Massive scale (millions of users)
✅ Simple queries (key-value)
✅ Real-time data
✅ Horizontal scaling needed

Examples:
- Social media (posts, feeds)
- IoT data
- Caching (Redis)
- Analytics
- Big data
```

---

## 🆚 Comparison

| Aspect             | SQL             | NoSQL        |
| ------------------ | --------------- | ------------ |
| **Schema**         | Fixed           | Flexible     |
| **Scaling**        | Vertical        | Horizontal   |
| **Transactions**   | ACID ✅         | Limited      |
| **Relationships**  | Complex ✅      | Simple       |
| **Query Language** | SQL             | Varies       |
| **Consistency**    | Strong          | Eventual     |
| **Best For**       | Complex queries | Simple, fast |

---

## 🎯 Real Examples

### E-commerce (SQL):

```sql
-- Complex relationships
SELECT
    o.id,
    u.name AS customer,
    p.name AS product,
    oi.quantity
FROM orders o
JOIN users u ON u.id = o.user_id
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
WHERE o.status = 'completed';
```

### Social Media Feeds (NoSQL):

```javascript
// Simple, fast lookups
db.posts.find({ user_id: "123" }).sort({ created_at: -1 }).limit(20);
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **SQL:** Complex relationships, ACID, structured
- ✅ **NoSQL:** Flexible, scalable, simple queries
- ✅ **اختر حسب الاحتياج** - ليس واحد أفضل من الآخر!
- ✅ يمكن استخدام **الاثنين معاً**!

</div>

---

<div align="center">

[➡️ Next: Relational Model](./02-relational-model.md) | [📚 Module Home](../README.md)

</div>
