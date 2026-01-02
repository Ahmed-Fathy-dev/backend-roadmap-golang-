# Views - الـ Views في PostgreSQL 👁️

<div dir="rtl">

## مقدمة

الـ Views هي virtual tables بتعرض نتيجة query معينة. بتساعد في تبسيط الـ queries المعقدة، تنظيم الـ access، وتحسين الـ security.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 View Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                        PostgreSQL Views                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Type                 │ Storage │ Auto-Update │ Use Case            │
│   ─────────────────────┼─────────┼─────────────┼───────────────────  │
│   Regular View         │ No      │ Yes         │ Query simplification│
│   Materialized View    │ Yes     │ Manual      │ Performance cache   │
│   Updatable View       │ No      │ Yes         │ Data modification   │
│   Recursive View       │ No      │ Yes         │ Hierarchical data   │
│                                                                      │
│   Regular View = Stored query (virtual table)                       │
│   Materialized View = Cached results (physical storage)             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Regular Views

### إنشاء View بسيط

```sql
-- View للـ active users
CREATE VIEW active_users AS
SELECT id, username, email, created_at
FROM users
WHERE is_active = true;

-- استخدام الـ view
SELECT * FROM active_users;
SELECT * FROM active_users WHERE created_at > '2024-01-01';

-- View مع JOIN
CREATE VIEW order_summary AS
SELECT
    o.id as order_id,
    u.username,
    u.email,
    o.total_amount,
    o.status,
    o.created_at
FROM orders o
JOIN users u ON o.user_id = u.id;

-- استخدام
SELECT * FROM order_summary WHERE status = 'completed';
```

### View مع Calculations

```sql
-- View للإحصائيات
CREATE VIEW user_statistics AS
SELECT
    u.id,
    u.username,
    COUNT(o.id) as total_orders,
    COALESCE(SUM(o.total_amount), 0) as total_spent,
    COALESCE(AVG(o.total_amount), 0) as avg_order_value,
    MAX(o.created_at) as last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username;

-- استخدام
SELECT * FROM user_statistics ORDER BY total_spent DESC LIMIT 10;
```

### View مع Subqueries

```sql
-- View للمنتجات الأكثر مبيعاً
CREATE VIEW top_selling_products AS
SELECT
    p.id,
    p.name,
    p.price,
    stats.total_sold,
    stats.revenue
FROM products p
JOIN (
    SELECT
        product_id,
        SUM(quantity) as total_sold,
        SUM(quantity * price) as revenue
    FROM order_items
    GROUP BY product_id
) stats ON p.id = stats.product_id
ORDER BY stats.total_sold DESC;
```

---

## 2️⃣ View Options

### OR REPLACE

```sql
-- إنشاء أو تحديث view
CREATE OR REPLACE VIEW active_users AS
SELECT id, username, email, full_name, created_at
FROM users
WHERE is_active = true;
-- لو الـ view موجود بيتحدث، لو مش موجود بيتعمل
```

### WITH CHECK OPTION

```sql
-- View مع validation
CREATE VIEW premium_users AS
SELECT * FROM users
WHERE subscription_type = 'premium'
WITH CHECK OPTION;

-- الآن لو حاولت INSERT user مش premium هيفشل
INSERT INTO premium_users (username, email, subscription_type)
VALUES ('test', 'test@test.com', 'free');
-- Error: new row violates check option for view "premium_users"
```

### SECURITY DEFINER vs INVOKER

```sql
-- SECURITY INVOKER (default)
-- الـ view بتشتغل بصلاحيات الـ user اللي بيستخدمها
CREATE VIEW public_data AS
SELECT id, name FROM products;

-- SECURITY DEFINER
-- الـ view بتشتغل بصلاحيات الـ owner
CREATE VIEW sensitive_data
WITH (security_barrier = true)
AS
SELECT id, username, email FROM users
WHERE department = current_user;
```

---

## 3️⃣ Materialized Views

### إنشاء Materialized View

```sql
-- Materialized view للـ dashboard
CREATE MATERIALIZED VIEW dashboard_stats AS
SELECT
    DATE(created_at) as date,
    COUNT(*) as total_orders,
    SUM(total_amount) as revenue,
    AVG(total_amount) as avg_order,
    COUNT(DISTINCT user_id) as unique_customers
FROM orders
WHERE status = 'completed'
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- البيانات متخزنة فعلياً
SELECT * FROM dashboard_stats WHERE date >= CURRENT_DATE - 7;
```

### Refresh Materialized View

```sql
-- تحديث البيانات
REFRESH MATERIALIZED VIEW dashboard_stats;

-- تحديث بدون lock (يحتاج UNIQUE INDEX)
CREATE UNIQUE INDEX idx_dashboard_date ON dashboard_stats(date);
REFRESH MATERIALIZED VIEW CONCURRENTLY dashboard_stats;

-- ⚠️ CONCURRENTLY بيحتاج UNIQUE INDEX
-- بيسمح بـ reads أثناء الـ refresh
```

### Index على Materialized View

```sql
-- الـ materialized view ممكن يكون عليها indexes
CREATE INDEX idx_dashboard_stats_date ON dashboard_stats(date);
CREATE INDEX idx_dashboard_stats_revenue ON dashboard_stats(revenue DESC);
```

### Automatic Refresh

```sql
-- PostgreSQL مش بيدعم automatic refresh
-- لازم نعمله manually أو بـ extension/cron

-- Option 1: pg_cron extension
CREATE EXTENSION IF NOT EXISTS pg_cron;

SELECT cron.schedule(
    'refresh_dashboard',
    '*/15 * * * *',  -- كل 15 دقيقة
    'REFRESH MATERIALIZED VIEW CONCURRENTLY dashboard_stats'
);

-- Option 2: Application-level
-- Refresh في الـ Go code

-- Option 3: Trigger (بحذر!)
-- ممكن يبطئ الـ writes
```

---

## 4️⃣ Updatable Views

### Simple Updatable View

```sql
-- View بسيط على table واحد
CREATE VIEW user_public_info AS
SELECT id, username, email, bio
FROM users
WHERE is_active = true;

-- ممكن نعمل INSERT, UPDATE, DELETE
UPDATE user_public_info SET bio = 'New bio' WHERE id = 1;
DELETE FROM user_public_info WHERE id = 5;
INSERT INTO user_public_info (username, email, bio)
VALUES ('newuser', 'new@test.com', 'Hello');
-- ⚠️ is_active هياخد الـ default value
```

### Rules لتحويل View لـ Updatable

```
┌─────────────────────────────────────────────────────────────────────┐
│              Automatically Updatable View Requirements               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Requirements:                                                    │
│  • Single table in FROM                                              │
│  • No DISTINCT                                                       │
│  • No GROUP BY                                                       │
│  • No HAVING                                                         │
│  • No LIMIT / OFFSET                                                 │
│  • No UNION / INTERSECT / EXCEPT                                     │
│  • No Aggregate functions                                            │
│  • No Window functions                                               │
│  • No Set-returning functions                                        │
│                                                                      │
│  ❌ If any of above exists → Need INSTEAD OF trigger                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### INSTEAD OF Triggers

```sql
-- View معقد
CREATE VIEW order_details AS
SELECT
    o.id as order_id,
    o.status,
    o.total_amount,
    u.id as user_id,
    u.username,
    u.email
FROM orders o
JOIN users u ON o.user_id = u.id;

-- INSTEAD OF trigger للـ UPDATE
CREATE OR REPLACE FUNCTION update_order_details()
RETURNS trigger AS $$
BEGIN
    -- Update orders table
    UPDATE orders
    SET status = NEW.status,
        total_amount = NEW.total_amount
    WHERE id = NEW.order_id;

    -- Update users table (if needed)
    UPDATE users
    SET username = NEW.username,
        email = NEW.email
    WHERE id = NEW.user_id;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER order_details_update
    INSTEAD OF UPDATE ON order_details
    FOR EACH ROW
    EXECUTE FUNCTION update_order_details();

-- الآن نقدر نعمل
UPDATE order_details
SET status = 'shipped', username = 'updated_user'
WHERE order_id = 1;
```

---

## 5️⃣ Recursive Views

```sql
-- جدول الـ categories الهرمي
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    parent_id INT REFERENCES categories(id)
);

INSERT INTO categories (name, parent_id) VALUES
('Electronics', NULL),
('Phones', 1),
('Laptops', 1),
('iPhone', 2),
('Samsung', 2),
('MacBook', 3);

-- Recursive view للـ hierarchy
CREATE RECURSIVE VIEW category_tree (id, name, parent_id, level, path) AS
    -- Base case
    SELECT id, name, parent_id, 0 as level, name::text as path
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT c.id, c.name, c.parent_id, ct.level + 1, ct.path || ' > ' || c.name
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id;

-- استخدام
SELECT * FROM category_tree ORDER BY path;
-- Result:
-- Electronics
-- Electronics > Laptops
-- Electronics > Laptops > MacBook
-- Electronics > Phones
-- Electronics > Phones > iPhone
-- Electronics > Phones > Samsung
```

---

## 6️⃣ Security with Views

```sql
-- إخفاء بيانات حساسة
CREATE VIEW public_users AS
SELECT
    id,
    username,
    LEFT(email, 3) || '***' || SUBSTRING(email FROM POSITION('@' IN email)) as email,
    created_at
FROM users;

-- Row-Level Security simulation
CREATE VIEW my_orders AS
SELECT * FROM orders
WHERE user_id = current_user_id();  -- function ترجع الـ user id

-- Column-Level Security
CREATE VIEW employee_public AS
SELECT id, name, department, title
FROM employees;
-- مش بيشمل salary

-- Permissions
GRANT SELECT ON public_users TO readonly_role;
REVOKE ALL ON users FROM readonly_role;
```

---

## 📊 Practical Examples

### Dashboard View

```sql
CREATE MATERIALIZED VIEW admin_dashboard AS
WITH order_stats AS (
    SELECT
        COUNT(*) as total_orders,
        COUNT(*) FILTER (WHERE created_at >= CURRENT_DATE) as today_orders,
        COUNT(*) FILTER (WHERE created_at >= CURRENT_DATE - 7) as week_orders,
        SUM(total_amount) as total_revenue,
        SUM(total_amount) FILTER (WHERE created_at >= CURRENT_DATE) as today_revenue
    FROM orders
    WHERE status = 'completed'
),
user_stats AS (
    SELECT
        COUNT(*) as total_users,
        COUNT(*) FILTER (WHERE created_at >= CURRENT_DATE) as new_today,
        COUNT(*) FILTER (WHERE last_login >= CURRENT_DATE - 7) as active_weekly
    FROM users
),
product_stats AS (
    SELECT
        COUNT(*) as total_products,
        COUNT(*) FILTER (WHERE stock = 0) as out_of_stock
    FROM products
)
SELECT
    o.*,
    u.*,
    p.*,
    NOW() as last_updated
FROM order_stats o, user_stats u, product_stats p;

CREATE UNIQUE INDEX idx_admin_dashboard ON admin_dashboard(last_updated);
```

### Report View

```sql
CREATE VIEW monthly_sales_report AS
SELECT
    DATE_TRUNC('month', o.created_at) as month,
    p.category,
    COUNT(DISTINCT o.id) as order_count,
    COUNT(DISTINCT o.user_id) as customer_count,
    SUM(oi.quantity) as units_sold,
    SUM(oi.quantity * oi.price) as revenue,
    AVG(o.total_amount) as avg_order_value
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.status = 'completed'
GROUP BY DATE_TRUNC('month', o.created_at), p.category
ORDER BY month DESC, revenue DESC;
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Views Best Practices                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Use Regular Views for:                                           │
│     - Simplifying complex queries                                    │
│     - Security/access control                                        │
│     - API layer abstraction                                          │
│                                                                      │
│  2. Use Materialized Views for:                                      │
│     - Expensive aggregations                                         │
│     - Dashboard/reporting                                            │
│     - Data that changes infrequently                                 │
│                                                                      │
│  3. Naming conventions:                                              │
│     - v_users (regular view)                                         │
│     - mv_dashboard (materialized view)                               │
│                                                                      │
│  4. Always index materialized views                                  │
│     - On columns used in WHERE                                       │
│     - UNIQUE for CONCURRENTLY refresh                                │
│                                                                      │
│  5. Document views                                                   │
│     COMMENT ON VIEW v_users IS 'Active users only';                 │
│                                                                      │
│  6. Monitor materialized view freshness                              │
│     Store last_refreshed timestamp                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 View Management

```sql
-- List all views
SELECT viewname, viewowner FROM pg_views WHERE schemaname = 'public';

-- List materialized views
SELECT matviewname, matviewowner FROM pg_matviews WHERE schemaname = 'public';

-- View definition
SELECT pg_get_viewdef('active_users', true);

-- Drop views
DROP VIEW IF EXISTS active_users;
DROP VIEW IF EXISTS order_summary CASCADE;  -- drops dependent views

DROP MATERIALIZED VIEW IF EXISTS dashboard_stats;

-- Check if view is updatable
SELECT table_name, is_insertable_into, is_updatable
FROM information_schema.views
WHERE table_schema = 'public';

-- Dependencies
SELECT
    dependent_view.relname as view_name,
    source_table.relname as table_name
FROM pg_depend
JOIN pg_rewrite ON pg_depend.objid = pg_rewrite.oid
JOIN pg_class as dependent_view ON pg_rewrite.ev_class = dependent_view.oid
JOIN pg_class as source_table ON pg_depend.refobjid = source_table.oid
WHERE source_table.relname = 'users';
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **Regular Views** للـ simplification والـ security
2. **Materialized Views** للـ performance (مع REFRESH)
3. **WITH CHECK OPTION** للـ data validation
4. **INSTEAD OF triggers** للـ complex updatable views
5. **Indexes** على materialized views مهمة
6. **CONCURRENTLY** للـ refresh بدون downtime

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Partitioning](./09-partitioning.md)**

</div>

---

<div align="center">

[⬅️ السابق](./07-full-text-search.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./09-partitioning.md)

</div>
