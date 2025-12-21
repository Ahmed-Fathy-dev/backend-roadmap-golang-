# Database Performance Guide ⚡

<div dir="rtl">

## نظرة عامة

دليل شامل لتحسين أداء قواعد البيانات وجعل Queries أسرع.

</div>

---

## 🎯 1. Indexing

### What is an Index?

<div dir="rtl">

**Index** مثل فهرس الكتاب - يساعدك تجد المعلومة بسرعة بدل ما تقرأ الكتاب كله.

</div>

### Without Index:

```sql
-- Search 1 million users
SELECT * FROM users WHERE email = 'ahmed@test.com';

→ Full table scan: checks ALL 1,000,000 rows 😱
→ Time: ~500ms
```

### With Index:

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Same query now:
SELECT * FROM users WHERE email = 'ahmed@test.com';

→ Index lookup: finds row instantly ⚡
→ Time: ~2ms (250x faster!)
```

### When to Create Indexes:

```sql
-- ✅ Index columns used in WHERE
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_products_category ON products(category);

-- ✅ Index Foreign Keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- ✅Index columns used in ORDER BY
CREATE INDEX idx_products_created_at ON products(created_at);

-- ✅ Composite indexes for multiple columns
CREATE INDEX idx_products_category_price ON products(category, price);
```

### When NOT to Index:

```go
// ❌ Don't index:
// - Small tables (< 1000 rows)
// - Columns with low cardinality (e.g., boolean, status with few values)
// - Columns that change frequently
// - Rarely queried columns
```

---

## 🔍 2. Query Optimization

### Use EXPLAIN ANALYZE:

```sql
-- See query execution plan
EXPLAIN ANALYZE
SELECT * FROM products WHERE category = 'laptops';

-- Output shows:
-- - Seq Scan vs Index Scan
-- - Time taken
-- - Rows examined
```

### N+1 Query Problem:

```go
// ❌ BAD: N+1 queries
users := GetAllUsers()  // 1 query
for _, user := range users {
    user.Posts = GetUserPosts(user.ID)  // N queries!
}
// Total: 1 + N queries

// ✅ GOOD: Use JOIN or eager loading
SELECT users.*, posts.*
FROM users
LEFT JOIN posts ON posts.user_id = users.id;
// Only 1 query!

// In GORM:
db.Preload("Posts").Find(&users)  // 1 query
```

### Select Only Needed Columns:

```sql
-- ❌ BAD: Select everything
SELECT * FROM users;

-- ✅ GOOD: Select only needed
SELECT id, name, email FROM users;
```

### Use LIMIT:

```sql
-- ❌ BAD: Load everything
SELECT * FROM products;  -- 1 million rows!

-- ✅ GOOD: Paginate
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;
```

---

## 💾 3. Connection Pooling

### The Problem:

```
Request 1: Open DB connection (100ms) + Query (2ms) = 102ms
Request 2: Open DB connection (100ms) + Query (2ms) = 102ms
Request 3: Open DB connection (100ms) + Query (2ms) = 102ms
...
```

### The Solution:

```go
import "database/sql"

db, _ := sql.Open("postgres", dsn)

// Configure pool
db.SetMaxOpenConns(25)          // Max 25 concurrent connections
db.SetMaxIdleConns(5)           // Keep 5 idle connections
db.SetConnMaxLifetime(5 * time.Minute)

// Now connections are reused!
// Request 1: Reuse connection + Query (2ms) = 2ms ⚡
```

---

## 🗄️ 4. Caching

### Application-Level Caching:

```go
import "github.com/go-redis/redis/v8"

func GetProduct(id int) (*Product, error) {
    // 1. Check cache first
    cacheKey := fmt.Sprintf("product:%d", id)
    cached, err := redisClient.Get(ctx, cacheKey).Result()

    if err == nil {
        // Cache HIT! ⚡
        var product Product
        json.Unmarshal([]byte(cached), &product)
        return &product, nil
    }

    // 2. Cache MISS - query database
    product, err := db.GetProduct(id)
    if err != nil {
        return nil, err
    }

    // 3. Store in cache
    jsonData, _ := json.Marshal(product)
    redisClient.Set(ctx, cacheKey, jsonData, 10*time.Minute)

    return product, nil
}
```

### Database Query Caching:

```sql
-- PostgreSQL automatically caches queries
-- But you can help by:
-- 1. Using prepared statements
-- 2. Consistent query patterns
```

---

## 📊 5. Database Design

### Normalization:

```sql
-- ❌ BAD: Denormalized (data duplication)
CREATE TABLE orders (
    id INT,
    product_name VARCHAR,     -- Repeated!
    product_price DECIMAL,    -- Repeated!
    quantity INT
);

-- ✅ GOOD: Normalized
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR,
    price DECIMAL
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    product_id INT REFERENCES products(id),
    quantity INT
);
```

### Use Appropriate Data Types:

```sql
-- ❌ BAD: Wrong types
CREATE TABLE users (
    age VARCHAR(50),          -- Should be INT!
    price VARCHAR(100),       -- Should be DECIMAL!
    is_active VARCHAR(10)     -- Should be BOOLEAN!
);

-- ✅ GOOD: Correct types
CREATE TABLE users (
    age INT,
    price DECIMAL(10,2),
    is_active BOOLEAN
);
```

---

## 🔄 6. Batch Operations

### Instead of Multiple Inserts:

```go
// ❌ BAD: Insert one by one
for _, product := range products {
    db.Exec(
        "INSERT INTO products (name, price) VALUES ($1, $2)",
        product.Name,
        product.Price,
    )
}
// 100 products = 100 queries!

// ✅ GOOD: Batch insert
values := []interface{}{}
query := "INSERT INTO products (name, price) VALUES "
for i, product := range products {
    query += fmt.Sprintf("($%d, $%d)", i*2+1, i*2+2)
    if i < len(products)-1 {
        query += ", "
    }
    values = append(values, product.Name, product.Price)
}
db.Exec(query, values...)
// 100 products = 1 query! ⚡
```

---

## ⏱️ 7. Slow Query Log

### Enable in PostgreSQL:

```sql
-- postgresql.conf
log_min_duration_statement = 1000  -- Log queries > 1 second

-- Check logs
SELECT * FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

---

## 📈 8. Monitoring

### Key Metrics to Track:

```go
// 1. Query execution time
// 2. Connection pool usage
// 3. Cache hit ratio
// 4. Slow queries
// 5. Database CPU/Memory
```

---

## ✅ Performance Checklist

<div dir="rtl">

- [ ] **Indexes** على columns مستخدمة في WHERE, ORDER BY
- [ ] **Foreign Keys** indexed
- [ ] **N+1 queries** eliminated (use JOINs/Preload)
- [ ] **SELECT \*** replaced with specific columns
- [ ] **LIMIT** used for large result sets
- [ ] **Connection pooling** configured
- [ ] **Caching** implemented (Redis/Memcached)
- [ ] **Batch operations** للـ bulk inserts/updates
- [ ] **EXPLAIN ANALYZE** للـ slow queries
- [ ] **Slow query log** enabled
- [ ] **Appropriate data types** used
- [ ] **Database normalized** (3NF usually)
- [ ] **Monitoring** in place

</div>

---

## 💡 Quick Wins

<div dir="rtl">

### أسرع التحسينات:

1. **Add index** على email, foreign keys → 10-100x faster
2. **Connection pooling** → 50x faster
3. **Cache frequently accessed data** → 100x faster
4. **Fix N+1 queries** → 10x faster
5. **Use LIMIT** → Don't load millions of rows

</div>

---

<div align="center">

[📚 Back to Module Home](../README.md)

</div>
