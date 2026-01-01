# Primary & Foreign Keys - المفاتيح 🔑

<div dir="rtl">

## مقدمة

المفاتيح هي أساس ربط الجداول ببعض. Primary Key بيحدد هوية الصف، و Foreign Key بيربط الجداول.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 🔑 Primary Key

```sql
-- تعريف Primary Key
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- Auto-increment
    email VARCHAR(255) UNIQUE NOT NULL
);

-- أو صراحةً
CREATE TABLE products (
    id SERIAL,
    sku VARCHAR(50) NOT NULL,
    PRIMARY KEY (id)
);

-- Composite Primary Key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- إضافة PK لجدول موجود
ALTER TABLE existing_table
ADD PRIMARY KEY (id);
```

### خصائص Primary Key
- **UNIQUE**: لا تتكرر
- **NOT NULL**: لا تقبل NULL
- **واحد فقط** لكل جدول
- تُنشئ **Index** تلقائياً

---

## 🔗 Foreign Key

```sql
-- تعريف Foreign Key
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- أو inline
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id)
);

-- مع اسم للـ constraint
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id) REFERENCES users(id)
);

-- إضافة FK لجدول موجود
ALTER TABLE orders
ADD CONSTRAINT fk_orders_user
FOREIGN KEY (user_id) REFERENCES users(id);

-- حذف FK
ALTER TABLE orders
DROP CONSTRAINT fk_orders_user;
```

---

## 🔄 ON DELETE / ON UPDATE Actions

```sql
-- CASCADE: احذف/حدّث المرتبط
CREATE TABLE orders (
    user_id INT REFERENCES users(id) ON DELETE CASCADE
);
-- لو حذفت user، الـ orders بتتحذف معاه

-- SET NULL: حط NULL
CREATE TABLE posts (
    author_id INT REFERENCES users(id) ON DELETE SET NULL
);
-- لو حذفت user، الـ author_id يبقى NULL

-- SET DEFAULT: حط القيمة الافتراضية
CREATE TABLE logs (
    user_id INT DEFAULT 0 REFERENCES users(id) ON DELETE SET DEFAULT
);

-- RESTRICT (default): امنع الحذف
CREATE TABLE accounts (
    user_id INT REFERENCES users(id) ON DELETE RESTRICT
);
-- مش هتقدر تحذف user لو له accounts

-- NO ACTION: زي RESTRICT بس deferred
CREATE TABLE profiles (
    user_id INT REFERENCES users(id) ON DELETE NO ACTION
);
```

---

## 💡 Best Practices

```sql
-- ✅ استخدم SERIAL أو UUID للـ Primary Key
CREATE TABLE users (
    id SERIAL PRIMARY KEY  -- أو UUID
);

-- ✅ أنشئ Index على Foreign Keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- ✅ استخدم ON DELETE CASCADE للـ child tables
CREATE TABLE order_items (
    order_id INT REFERENCES orders(id) ON DELETE CASCADE
);

-- ✅ سمّي الـ constraints
CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users(id)
```

---

## ⏭️ الدرس التالي

**➡️ [Referential Integrity](./03-referential-integrity.md)**

---

<div align="center">

[⬅️ السابق](./01-relationships.md) | [🏠 العودة للـ Module](../README.md) | [التالي ➡️](./03-referential-integrity.md)

</div>
