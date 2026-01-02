# Partitioning - تقسيم الجداول 📊

<div dir="rtl">

## مقدمة

الـ Partitioning هو تقسيم جدول كبير لأجزاء أصغر (partitions). بيحسن الـ performance بشكل كبير للجداول الضخمة خصوصاً مع الـ time-series data.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 📊 Partitioning Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Partitioning Benefits                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Performance:                                                        │
│  ├── Query on one partition → Scan only that partition              │
│  ├── Parallel queries across partitions                              │
│  └── Smaller indexes per partition                                   │
│                                                                      │
│  Maintenance:                                                        │
│  ├── Drop old partitions instantly (vs DELETE)                      │
│  ├── VACUUM/ANALYZE on single partition                              │
│  └── Independent partition management                                │
│                                                                      │
│  Storage:                                                            │
│  ├── Different tablespaces per partition                             │
│  └── Compression per partition                                       │
│                                                                      │
│  Use Cases:                                                          │
│  • Time-series data (logs, events, metrics)                         │
│  • Large tables (> 100M rows)                                        │
│  • Data archival requirements                                        │
│  • Multi-tenant applications                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Partition Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Partition Strategies                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. RANGE Partitioning                                               │
│     ├── Best for: dates, timestamps, sequential IDs                 │
│     └── Example: orders_2024_01, orders_2024_02                     │
│                                                                      │
│  2. LIST Partitioning                                                │
│     ├── Best for: categorical data                                  │
│     └── Example: orders_egypt, orders_saudi                         │
│                                                                      │
│  3. HASH Partitioning                                                │
│     ├── Best for: even distribution                                 │
│     └── Example: users_0, users_1, users_2, users_3                 │
│                                                                      │
│  4. Composite Partitioning                                           │
│     └── Combination of above (e.g., RANGE then LIST)                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ RANGE Partitioning

### Time-Based Partitioning

```sql
-- إنشاء الـ parent table
CREATE TABLE events (
    id BIGSERIAL,
    event_type VARCHAR(50) NOT NULL,
    payload JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (id, created_at)  -- partition key لازم يكون في الـ PK
) PARTITION BY RANGE (created_at);

-- إنشاء الـ partitions
CREATE TABLE events_2024_01 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE events_2024_02 PARTITION OF events
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE events_2024_03 PARTITION OF events
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Default partition للبيانات خارج الـ ranges
CREATE TABLE events_default PARTITION OF events DEFAULT;

-- Insert بيروح للـ partition الصحيح تلقائياً
INSERT INTO events (event_type, payload, created_at)
VALUES ('user_signup', '{"user_id": 123}', '2024-01-15');
-- → Goes to events_2024_01

INSERT INTO events (event_type, payload, created_at)
VALUES ('purchase', '{"amount": 99}', '2024-02-20');
-- → Goes to events_2024_02
```

### Query with Partition Pruning

```sql
-- Query على شهر واحد
EXPLAIN ANALYZE
SELECT * FROM events
WHERE created_at >= '2024-01-01' AND created_at < '2024-02-01';
-- → Scans only events_2024_01

-- بدون الـ partition key
EXPLAIN ANALYZE
SELECT * FROM events WHERE event_type = 'purchase';
-- → Scans ALL partitions (slow!)
```

### Automatic Partition Creation

```sql
-- Function لإنشاء partitions تلقائياً
CREATE OR REPLACE FUNCTION create_monthly_partition(table_name TEXT, start_date DATE)
RETURNS void AS $$
DECLARE
    partition_name TEXT;
    end_date DATE;
BEGIN
    partition_name := table_name || '_' || TO_CHAR(start_date, 'YYYY_MM');
    end_date := start_date + INTERVAL '1 month';

    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF %I
         FOR VALUES FROM (%L) TO (%L)',
        partition_name,
        table_name,
        start_date,
        end_date
    );
END;
$$ LANGUAGE plpgsql;

-- إنشاء 12 شهر
DO $$
BEGIN
    FOR i IN 0..11 LOOP
        PERFORM create_monthly_partition(
            'events',
            DATE '2024-01-01' + (i || ' months')::INTERVAL
        );
    END LOOP;
END $$;
```

---

## 2️⃣ LIST Partitioning

### By Category

```sql
-- جدول orders مقسم بالـ region
CREATE TABLE orders (
    id BIGSERIAL,
    user_id INT NOT NULL,
    total_amount DECIMAL(10, 2),
    region VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (id, region)
) PARTITION BY LIST (region);

-- Partitions
CREATE TABLE orders_egypt PARTITION OF orders
    FOR VALUES IN ('egypt', 'eg');

CREATE TABLE orders_saudi PARTITION OF orders
    FOR VALUES IN ('saudi', 'sa', 'ksa');

CREATE TABLE orders_uae PARTITION OF orders
    FOR VALUES IN ('uae', 'ae');

CREATE TABLE orders_other PARTITION OF orders DEFAULT;

-- Insert
INSERT INTO orders (user_id, total_amount, region)
VALUES (1, 199.99, 'egypt');
-- → Goes to orders_egypt

-- Query specific region
SELECT * FROM orders WHERE region = 'egypt';
-- → Scans only orders_egypt
```

### By Status

```sql
-- Partition by order status
CREATE TABLE order_items (
    id BIGSERIAL,
    order_id BIGINT NOT NULL,
    product_id INT NOT NULL,
    status VARCHAR(20) NOT NULL,
    PRIMARY KEY (id, status)
) PARTITION BY LIST (status);

CREATE TABLE order_items_pending PARTITION OF order_items
    FOR VALUES IN ('pending', 'processing');

CREATE TABLE order_items_active PARTITION OF order_items
    FOR VALUES IN ('shipped', 'in_transit');

CREATE TABLE order_items_completed PARTITION OF order_items
    FOR VALUES IN ('delivered', 'completed');

CREATE TABLE order_items_cancelled PARTITION OF order_items
    FOR VALUES IN ('cancelled', 'refunded', 'returned');
```

---

## 3️⃣ HASH Partitioning

```sql
-- توزيع متساوي على 4 partitions
CREATE TABLE user_sessions (
    id BIGSERIAL,
    user_id INT NOT NULL,
    session_data JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (id, user_id)
) PARTITION BY HASH (user_id);

-- 4 partitions
CREATE TABLE user_sessions_0 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE user_sessions_1 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE user_sessions_2 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE user_sessions_3 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);

-- البيانات هتتوزع بالتساوي تقريباً
INSERT INTO user_sessions (user_id, session_data)
SELECT i, '{"active": true}'
FROM generate_series(1, 10000) i;

-- Check distribution
SELECT
    tableoid::regclass as partition,
    COUNT(*) as rows
FROM user_sessions
GROUP BY tableoid;
```

---

## 4️⃣ Composite (Sub-Partitioning)

```sql
-- Range ثم List
CREATE TABLE sales (
    id BIGSERIAL,
    region VARCHAR(20) NOT NULL,
    amount DECIMAL(10, 2),
    sale_date DATE NOT NULL,
    PRIMARY KEY (id, sale_date, region)
) PARTITION BY RANGE (sale_date);

-- Partition بالتاريخ
CREATE TABLE sales_2024_q1 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01')
    PARTITION BY LIST (region);  -- Sub-partition بالـ region

-- Sub-partitions
CREATE TABLE sales_2024_q1_egypt PARTITION OF sales_2024_q1
    FOR VALUES IN ('egypt');

CREATE TABLE sales_2024_q1_saudi PARTITION OF sales_2024_q1
    FOR VALUES IN ('saudi');

CREATE TABLE sales_2024_q1_other PARTITION OF sales_2024_q1 DEFAULT;

-- نفس الشيء لـ Q2
CREATE TABLE sales_2024_q2 PARTITION OF sales
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01')
    PARTITION BY LIST (region);

CREATE TABLE sales_2024_q2_egypt PARTITION OF sales_2024_q2
    FOR VALUES IN ('egypt');
-- ... etc
```

---

## 5️⃣ Indexes on Partitions

```sql
-- Index على الـ parent (ينطبق على كل الـ partitions)
CREATE INDEX idx_events_type ON events(event_type);
-- ينشئ index على كل partition

-- Index على partition معين
CREATE INDEX idx_events_2024_01_payload ON events_2024_01 USING gin(payload);

-- Check indexes
SELECT
    tablename,
    indexname
FROM pg_indexes
WHERE tablename LIKE 'events%';
```

---

## 6️⃣ Partition Management

### Adding Partitions

```sql
-- إضافة partition جديد
CREATE TABLE events_2024_04 PARTITION OF events
    FOR VALUES FROM ('2024-04-01') TO ('2024-05-01');

-- أو attach جدول موجود
CREATE TABLE events_2024_05 (LIKE events INCLUDING ALL);
ALTER TABLE events ATTACH PARTITION events_2024_05
    FOR VALUES FROM ('2024-05-01') TO ('2024-06-01');
```

### Removing Partitions

```sql
-- Detach بدون حذف
ALTER TABLE events DETACH PARTITION events_2024_01;
-- الجدول لسه موجود كـ standalone table

-- حذف البيانات القديمة (أسرع من DELETE!)
DROP TABLE events_2024_01;

-- أو move to archive
ALTER TABLE events_2024_01 SET TABLESPACE archive_space;
```

### Moving Data Between Partitions

```sql
-- لو البيانات اتحطت في الـ default partition
-- لازم نحركها للـ partition الصحيح

-- 1. إنشاء الـ partition الجديد
CREATE TABLE events_2024_06 PARTITION OF events
    FOR VALUES FROM ('2024-06-01') TO ('2024-07-01');

-- 2. نقل البيانات من default
INSERT INTO events_2024_06
SELECT * FROM events_default
WHERE created_at >= '2024-06-01' AND created_at < '2024-07-01';

DELETE FROM events_default
WHERE created_at >= '2024-06-01' AND created_at < '2024-07-01';
```

---

## 📊 Monitoring Partitions

```sql
-- List all partitions
SELECT
    parent.relname as parent_table,
    child.relname as partition,
    pg_get_expr(child.relpartbound, child.oid) as partition_bounds
FROM pg_inherits
JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
JOIN pg_class child ON pg_inherits.inhrelid = child.oid
WHERE parent.relname = 'events'
ORDER BY child.relname;

-- Partition sizes
SELECT
    child.relname as partition,
    pg_size_pretty(pg_relation_size(child.oid)) as size,
    pg_stat_get_live_tuples(child.oid) as row_count
FROM pg_inherits
JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
JOIN pg_class child ON pg_inherits.inhrelid = child.oid
WHERE parent.relname = 'events'
ORDER BY child.relname;

-- Check partition pruning
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM events
WHERE created_at BETWEEN '2024-01-15' AND '2024-01-20';
```

---

## 📦 Real-World Example: Logs System

```sql
-- Complete logs partitioning setup
CREATE TABLE application_logs (
    id BIGSERIAL,
    service VARCHAR(50) NOT NULL,
    level VARCHAR(10) NOT NULL,
    message TEXT,
    metadata JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (created_at);

-- Function لإنشاء daily partitions
CREATE OR REPLACE FUNCTION create_daily_log_partition()
RETURNS void AS $$
DECLARE
    tomorrow DATE := CURRENT_DATE + 1;
    partition_name TEXT;
BEGIN
    partition_name := 'application_logs_' || TO_CHAR(tomorrow, 'YYYY_MM_DD');

    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF application_logs
         FOR VALUES FROM (%L) TO (%L)',
        partition_name,
        tomorrow,
        tomorrow + 1
    );

    -- Create indexes
    EXECUTE format(
        'CREATE INDEX IF NOT EXISTS %I ON %I (service, level)',
        partition_name || '_service_level_idx',
        partition_name
    );
END;
$$ LANGUAGE plpgsql;

-- Function لحذف old partitions
CREATE OR REPLACE FUNCTION drop_old_log_partitions(retention_days INT)
RETURNS void AS $$
DECLARE
    cutoff_date DATE := CURRENT_DATE - retention_days;
    partition_record RECORD;
BEGIN
    FOR partition_record IN
        SELECT child.relname as partition_name
        FROM pg_inherits
        JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
        JOIN pg_class child ON pg_inherits.inhrelid = child.oid
        WHERE parent.relname = 'application_logs'
          AND child.relname < 'application_logs_' || TO_CHAR(cutoff_date, 'YYYY_MM_DD')
    LOOP
        EXECUTE format('DROP TABLE IF EXISTS %I', partition_record.partition_name);
        RAISE NOTICE 'Dropped partition: %', partition_record.partition_name;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- Schedule with pg_cron
-- Create tomorrow's partition daily
SELECT cron.schedule('create_log_partition', '0 23 * * *',
    'SELECT create_daily_log_partition()');

-- Drop old partitions (keep 30 days)
SELECT cron.schedule('drop_old_partitions', '0 2 * * *',
    'SELECT drop_old_log_partitions(30)');
```

---

## ⚠️ Limitations and Gotchas

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Partitioning Limitations                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Primary Key must include partition key                          │
│     PRIMARY KEY (id, created_at) -- not just (id)                  │
│                                                                      │
│  2. Unique constraints same rule                                     │
│     UNIQUE (email, created_at) -- not just (email)                 │
│                                                                      │
│  3. Foreign keys pointing TO partitioned table                      │
│     Not fully supported (workarounds exist)                         │
│                                                                      │
│  4. ON CONFLICT requires partition key                               │
│     Need to include it in the constraint                            │
│                                                                      │
│  5. Cross-partition queries slower                                   │
│     Always filter by partition key                                   │
│                                                                      │
│  6. TRUNCATE cascades to all partitions                              │
│     Use DROP partition instead for single partition                  │
│                                                                      │
│  7. Row movement between partitions                                  │
│     Need: ALTER TABLE ... ENABLE ROW MOVEMENT                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                 Partitioning Best Practices                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Choose the right partition key                                   │
│     - Most common filter column                                      │
│     - Usually timestamp for time-series                              │
│                                                                      │
│  2. Partition size: 100M-500M rows per partition                    │
│     - Too small = overhead                                           │
│     - Too large = defeats purpose                                    │
│                                                                      │
│  3. Always use DEFAULT partition                                     │
│     - Catches unexpected data                                        │
│     - Monitor its size                                               │
│                                                                      │
│  4. Create partitions in advance                                     │
│     - Don't wait until data arrives                                  │
│     - Automate creation                                              │
│                                                                      │
│  5. Include partition key in all queries                             │
│     - Enables partition pruning                                      │
│     - Much faster queries                                            │
│                                                                      │
│  6. Monitor and maintain                                             │
│     - Track partition sizes                                          │
│     - Drop old partitions (don't DELETE)                            │
│     - ANALYZE new partitions                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **RANGE** للـ time-series (الأكثر استخداماً)
2. **LIST** للـ categorical data
3. **HASH** للتوزيع المتساوي
4. **Partition key** لازم يكون في الـ PK
5. **DEFAULT partition** ضروري
6. **DROP** أسرع من DELETE للـ partitions
7. دايماً **filter بالـ partition key**

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [Stored Procedures](./10-stored-procedures.md)**

</div>

---

<div align="center">

[⬅️ السابق](./08-views.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./10-stored-procedures.md)

</div>
