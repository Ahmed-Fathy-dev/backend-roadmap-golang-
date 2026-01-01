# Referential Integrity - سلامة المراجع 🛡️

<div dir="rtl">

## مقدمة

Referential Integrity بتضمن إن العلاقات بين الجداول تفضل صحيحة - مثلاً مينفعش يكون عندك order لـ user مش موجود.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 ما هي Referential Integrity؟

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Referential Integrity                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Valid Reference:                                                │
│  users                    orders                                    │
│  ┌────┬─────────┐        ┌────┬─────────┐                          │
│  │ id │  name   │        │ id │ user_id │                          │
│  ├────┼─────────┤        ├────┼─────────┤                          │
│  │ 1  │ Ahmed   │◄───────│ 1  │    1    │  ✅ user 1 موجود        │
│  │ 2  │ Sara    │◄───────│ 2  │    2    │  ✅ user 2 موجود        │
│  └────┴─────────┘        └────┴─────────┘                          │
│                                                                      │
│  ❌ Invalid Reference (Orphan):                                     │
│  users                    orders                                    │
│  ┌────┬─────────┐        ┌────┬─────────┐                          │
│  │ id │  name   │        │ id │ user_id │                          │
│  ├────┼─────────┤        ├────┼─────────┤                          │
│  │ 1  │ Ahmed   │        │ 1  │    1    │                          │
│  │ 2  │ Sara    │    ✗───│ 2  │    5    │  ❌ user 5 مش موجود!    │
│  └────┴─────────┘        └────┴─────────┘                          │
│                                                                      │
│  Foreign Key Constraint بيمنع ده!                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔒 كيف تعمل الحماية؟

```sql
-- مثال: محاولة إدخال order لـ user مش موجود
INSERT INTO orders (user_id, total_amount)
VALUES (999, 100.00);
-- ERROR: insert or update violates foreign key constraint
-- Key (user_id)=(999) is not present in table "users"

-- محاولة حذف user له orders
DELETE FROM users WHERE id = 1;
-- ERROR: update or delete violates foreign key constraint
-- Key (id)=(1) is still referenced from table "orders"
```

---

## 🔄 ON DELETE Actions

```sql
-- 1. CASCADE: احذف كل المرتبط
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE
);
-- لو حذفت user، كل الـ orders بتتحذف تلقائياً

-- 2. SET NULL: حط NULL
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    author_id INT REFERENCES users(id) ON DELETE SET NULL
);
-- لو حذفت user، الـ author_id يبقى NULL

-- 3. RESTRICT (default): امنع الحذف
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE RESTRICT
);
-- مينفعش تحذف user لو له accounts

-- 4. NO ACTION: زي RESTRICT
-- الفرق: NO ACTION deferred (يتشيك في نهاية الـ transaction)
```

---

## 🔄 ON UPDATE Actions

```sql
-- لو الـ Primary Key اتغير
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON UPDATE CASCADE
);
-- لو الـ order.id اتغير، الـ order_items.order_id يتغير معاه

-- ⚠️ نادراً ما تحتاج ON UPDATE
-- لأن الـ Primary Key المفروض ما يتغيرش
```

---

## 🛡️ Deferrable Constraints

```sql
-- Constraint يتشيك في نهاية الـ transaction
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    manager_id INT,
    CONSTRAINT fk_manager
        FOREIGN KEY (manager_id) REFERENCES employees(id)
        DEFERRABLE INITIALLY DEFERRED
);

-- مفيد في:
BEGIN;
-- إدخال manager و employee في نفس الـ transaction
INSERT INTO employees (id, manager_id) VALUES (1, NULL);  -- Manager
INSERT INTO employees (id, manager_id) VALUES (2, 1);     -- Employee
COMMIT;  -- الـ constraint يتشيك هنا
```

---

## 💡 Best Practices

```sql
-- 1. دايماً استخدم Foreign Keys
CREATE TABLE orders (
    user_id INT NOT NULL REFERENCES users(id)
);

-- 2. فكر في الـ ON DELETE action المناسب
-- CASCADE للـ child tables (order_items)
-- SET NULL للـ optional relationships
-- RESTRICT للـ important relationships

-- 3. أنشئ Index على Foreign Keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 4. سمّي الـ constraints
CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users(id)
```

---

## ⏭️ الدرس التالي

**➡️ [INNER JOIN](./04-inner-join.md)**

---

<div align="center">

[⬅️ السابق](./02-keys.md) | [🏠 العودة للـ Module](../README.md) | [التالي ➡️](./04-inner-join.md)

</div>
