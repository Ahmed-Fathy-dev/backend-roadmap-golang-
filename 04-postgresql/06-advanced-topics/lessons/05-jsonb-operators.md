# JSONB Operators - عمليات JSONB المتقدمة 🔧

<div dir="rtl">

## مقدمة

JSONB بيقدم مجموعة قوية من الـ operators للبحث والمقارنة والتعديل. هنتعلم كل operator وإزاي نستخدمه بكفاءة.

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 Operators Reference

```
┌─────────────────────────────────────────────────────────────────────┐
│                      JSONB Operators                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Extraction Operators                                                │
│  ─────────────────────────────────────────────────────────────────  │
│  ->    │ Get JSON element by key/index     │ '{"a":1}' -> 'a'       │
│  ->>   │ Get element as TEXT               │ '{"a":1}' ->> 'a'      │
│  #>    │ Get element at path               │ '{"a":{"b":1}}' #> '{a,b}' │
│  #>>   │ Get element at path as TEXT       │ Same as above, returns TEXT │
│                                                                      │
│  Containment Operators                                               │
│  ─────────────────────────────────────────────────────────────────  │
│  @>    │ Contains (left contains right)    │ '{"a":1,"b":2}' @> '{"a":1}' │
│  <@    │ Contained by                      │ '{"a":1}' <@ '{"a":1,"b":2}' │
│                                                                      │
│  Existence Operators                                                 │
│  ─────────────────────────────────────────────────────────────────  │
│  ?     │ Key exists                        │ '{"a":1}' ? 'a'        │
│  ?|    │ Any key exists                    │ '{"a":1}' ?| array['a','b'] │
│  ?&    │ All keys exist                    │ '{"a":1,"b":2}' ?& array['a','b'] │
│                                                                      │
│  Modification Operators                                              │
│  ─────────────────────────────────────────────────────────────────  │
│  ||    │ Concatenate                       │ '{"a":1}' || '{"b":2}' │
│  -     │ Delete key/index                  │ '{"a":1,"b":2}' - 'a'  │
│  #-    │ Delete at path                    │ '{"a":{"b":1}}' #- '{a,b}' │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Extraction Operators

### -> (Get JSON)

```sql
-- استخراج بـ key
SELECT '{"name": "Ahmed", "age": 30}'::jsonb -> 'name';
-- Result: "Ahmed" (JSON string)

-- استخراج من array بـ index
SELECT '["a", "b", "c"]'::jsonb -> 1;
-- Result: "b"

-- استخراج nested
SELECT '{"user": {"name": "Ahmed"}}'::jsonb -> 'user' -> 'name';
-- Result: "Ahmed"

-- Null إذا مش موجود
SELECT '{"name": "Ahmed"}'::jsonb -> 'email';
-- Result: NULL
```

### ->> (Get TEXT)

```sql
-- الفرق بين -> و ->>
SELECT
    '{"name": "Ahmed"}'::jsonb -> 'name' as json_result,
    '{"name": "Ahmed"}'::jsonb ->> 'name' as text_result;
-- json_result: "Ahmed" (مع quotes)
-- text_result: Ahmed (بدون quotes)

-- مفيد للمقارنات
SELECT * FROM users
WHERE profile ->> 'country' = 'Egypt';  -- TEXT comparison

-- مع casting
SELECT * FROM products
WHERE (metadata ->> 'price')::numeric > 100;
```

### #> و #>> (Path Extraction)

```sql
-- استخراج بـ path
SELECT '{"a": {"b": {"c": 123}}}'::jsonb #> '{a,b,c}';
-- Result: 123

-- كـ TEXT
SELECT '{"a": {"b": {"c": 123}}}'::jsonb #>> '{a,b,c}';
-- Result: "123" (TEXT)

-- مع array في الـ path
SELECT '{"users": [{"name": "Ahmed"}, {"name": "Sara"}]}'::jsonb
    #>> '{users,0,name}';
-- Result: "Ahmed"

-- Practical example
SELECT
    id,
    metadata #>> '{shipping,address,city}' as city
FROM orders
WHERE metadata #>> '{shipping,address,country}' = 'Egypt';
```

---

## 2️⃣ Containment Operators

### @> (Contains)

```sql
-- هل الـ JSON بيحتوي على الـ subset ده؟
SELECT '{"name": "Ahmed", "age": 30, "city": "Cairo"}'::jsonb
    @> '{"name": "Ahmed"}'::jsonb;
-- Result: true

-- البحث في جدول
SELECT * FROM products
WHERE metadata @> '{"brand": "Apple", "in_stock": true}'::jsonb;

-- Nested containment
SELECT * FROM products
WHERE metadata @> '{"specs": {"cpu": "M3"}}'::jsonb;

-- Array containment
SELECT '{"colors": ["red", "blue", "green"]}'::jsonb
    @> '{"colors": ["red"]}'::jsonb;
-- Result: true

-- ⚠️ Array order matters for exact match
SELECT '[1, 2, 3]'::jsonb @> '[1, 2]'::jsonb;  -- true
SELECT '[1, 2, 3]'::jsonb @> '[2, 1]'::jsonb;  -- true (order doesn't matter!)
SELECT '[1, 2, 3]'::jsonb @> '[1, 4]'::jsonb;  -- false
```

### <@ (Contained By)

```sql
-- العكس من @>
SELECT '{"name": "Ahmed"}'::jsonb
    <@ '{"name": "Ahmed", "age": 30}'::jsonb;
-- Result: true

-- مفيد للتحقق من subset
SELECT * FROM user_permissions
WHERE permissions <@ '{"read": true, "write": true, "admin": true}'::jsonb;
```

---

## 3️⃣ Existence Operators

### ? (Key Exists)

```sql
-- هل الـ key موجود؟
SELECT '{"name": "Ahmed", "email": "a@b.com"}'::jsonb ? 'name';
-- Result: true

SELECT '{"name": "Ahmed"}'::jsonb ? 'email';
-- Result: false

-- البحث في جدول
SELECT * FROM products WHERE metadata ? 'warranty';

-- مع arrays: يبحث عن element
SELECT '["apple", "banana", "orange"]'::jsonb ? 'banana';
-- Result: true
```

### ?| (Any Key Exists)

```sql
-- هل أي key من دول موجود؟
SELECT '{"name": "Ahmed", "age": 30}'::jsonb ?| array['email', 'phone', 'name'];
-- Result: true (name موجود)

-- البحث
SELECT * FROM products
WHERE metadata ?| array['discount', 'sale_price', 'promotion'];
```

### ?& (All Keys Exist)

```sql
-- هل كل الـ keys موجودين؟
SELECT '{"name": "Ahmed", "age": 30}'::jsonb ?& array['name', 'age'];
-- Result: true

SELECT '{"name": "Ahmed", "age": 30}'::jsonb ?& array['name', 'email'];
-- Result: false (email مش موجود)

-- Validation
SELECT * FROM user_profiles
WHERE profile ?& array['first_name', 'last_name', 'email', 'phone'];
```

---

## 4️⃣ Modification Operators

### || (Concatenate/Merge)

```sql
-- دمج objects
SELECT '{"name": "Ahmed"}'::jsonb || '{"age": 30}'::jsonb;
-- Result: {"age": 30, "name": "Ahmed"}

-- Override existing keys
SELECT '{"name": "Ahmed", "age": 25}'::jsonb || '{"age": 30}'::jsonb;
-- Result: {"age": 30, "name": "Ahmed"}

-- دمج arrays
SELECT '[1, 2]'::jsonb || '[3, 4]'::jsonb;
-- Result: [1, 2, 3, 4]

-- إضافة element لـ array
SELECT '[1, 2, 3]'::jsonb || '4'::jsonb;
-- Result: [1, 2, 3, 4]

-- UPDATE with merge
UPDATE users
SET profile = profile || '{"verified": true, "verified_at": "2024-01-15"}'::jsonb
WHERE id = 1;

-- Deep merge (لازم function)
-- || بيعمل shallow merge فقط
SELECT '{"a": {"b": 1}}'::jsonb || '{"a": {"c": 2}}'::jsonb;
-- Result: {"a": {"c": 2}} -- الـ nested object اتبدل بالكامل!
```

### - (Delete)

```sql
-- حذف key
SELECT '{"name": "Ahmed", "age": 30, "city": "Cairo"}'::jsonb - 'age';
-- Result: {"city": "Cairo", "name": "Ahmed"}

-- حذف multiple keys
SELECT '{"a": 1, "b": 2, "c": 3}'::jsonb - 'a' - 'b';
-- Result: {"c": 3}

-- حذف من array بـ index
SELECT '["a", "b", "c", "d"]'::jsonb - 1;
-- Result: ["a", "c", "d"]

-- حذف بـ negative index
SELECT '["a", "b", "c", "d"]'::jsonb - -1;
-- Result: ["a", "b", "c"]

-- حذف element من array بـ value (مش supported مباشرة!)
-- لازم نستخدم function
SELECT jsonb_agg(elem)
FROM jsonb_array_elements('["a", "b", "c"]'::jsonb) as elem
WHERE elem != '"b"'::jsonb;
-- Result: ["a", "c"]
```

### #- (Delete at Path)

```sql
-- حذف nested key
SELECT '{"a": {"b": {"c": 1, "d": 2}}}'::jsonb #- '{a,b,c}';
-- Result: {"a": {"b": {"d": 2}}}

-- حذف من nested array
SELECT '{"users": [{"name": "Ahmed"}, {"name": "Sara"}]}'::jsonb
    #- '{users,0}';
-- Result: {"users": [{"name": "Sara"}]}

-- UPDATE with path delete
UPDATE products
SET metadata = metadata #- '{specs,old_feature}'
WHERE id = 1;
```

---

## 5️⃣ Comparison Operators

```sql
-- Equality
SELECT '{"a": 1, "b": 2}'::jsonb = '{"b": 2, "a": 1}'::jsonb;
-- Result: true (order doesn't matter for objects)

SELECT '[1, 2, 3]'::jsonb = '[1, 2, 3]'::jsonb;
-- Result: true

SELECT '[1, 2, 3]'::jsonb = '[3, 2, 1]'::jsonb;
-- Result: false (order matters for arrays)

-- Not equal
SELECT '{"a": 1}'::jsonb <> '{"a": 2}'::jsonb;
-- Result: true
```

---

## 📊 Advanced Patterns

### Pattern 1: Dynamic Filtering

```sql
-- Function للبحث الديناميكي
CREATE OR REPLACE FUNCTION search_products(filters jsonb)
RETURNS TABLE (id int, name varchar, metadata jsonb)
AS $$
BEGIN
    RETURN QUERY
    SELECT p.id, p.name, p.metadata
    FROM products p
    WHERE p.metadata @> filters;
END;
$$ LANGUAGE plpgsql;

-- استخدام
SELECT * FROM search_products('{"brand": "Apple", "in_stock": true}');
```

### Pattern 2: JSON Validation

```sql
-- Check required fields
ALTER TABLE products
ADD CONSTRAINT valid_metadata CHECK (
    metadata ?& array['brand', 'category']
);

-- أو مع type checking
ALTER TABLE products
ADD CONSTRAINT valid_metadata CHECK (
    jsonb_typeof(metadata -> 'price') = 'number'
    AND jsonb_typeof(metadata -> 'tags') = 'array'
);
```

### Pattern 3: Conditional Updates

```sql
-- Update فقط لو الـ key موجود
UPDATE products
SET metadata = jsonb_set(metadata, '{discount}', '20')
WHERE metadata ? 'discount';

-- Add if not exists
UPDATE products
SET metadata = CASE
    WHEN metadata ? 'views'
    THEN jsonb_set(metadata, '{views}',
         ((metadata->>'views')::int + 1)::text::jsonb)
    ELSE metadata || '{"views": 1}'
END
WHERE id = 1;
```

### Pattern 4: Array Operations

```sql
-- إضافة لـ array مع تجنب duplicates
UPDATE products
SET metadata = jsonb_set(
    metadata,
    '{tags}',
    (
        SELECT jsonb_agg(DISTINCT elem)
        FROM (
            SELECT jsonb_array_elements(metadata -> 'tags') as elem
            UNION
            SELECT '"new_tag"'::jsonb
        ) t
    )
)
WHERE id = 1;

-- حذف من array
UPDATE products
SET metadata = jsonb_set(
    metadata,
    '{tags}',
    (
        SELECT COALESCE(jsonb_agg(elem), '[]'::jsonb)
        FROM jsonb_array_elements(metadata -> 'tags') as elem
        WHERE elem != '"tag_to_remove"'::jsonb
    )
)
WHERE id = 1;
```

---

## 🔄 Operator Precedence

```sql
-- ترتيب الأولويات (من الأعلى للأقل)
-- 1. -> ->> #> #>> (Extraction)
-- 2. @> <@ (Containment)
-- 3. ? ?| ?& (Existence)
-- 4. || (Concatenation)
-- 5. - #- (Deletion)

-- مثال
SELECT metadata -> 'brand' ->> 'name' || ' Inc'
FROM products
WHERE metadata @> '{"active": true}' AND metadata ? 'brand';
```

---

## 💡 Performance Tips

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Operator Performance Tips                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Operator    │ GIN Index Support │ Performance                      │
│  ────────────┼───────────────────┼────────────────────────────────  │
│  @>          │ ✅ Yes            │ Fast with GIN                    │
│  <@          │ ❌ No (usually)   │ Slower                           │
│  ?           │ ✅ Yes            │ Fast with GIN                    │
│  ?| ?&       │ ✅ Yes            │ Fast with GIN                    │
│  -> ->>      │ ❌ No             │ Use expression index             │
│  #> #>>      │ ❌ No             │ Use expression index             │
│                                                                      │
│  Recommendations:                                                    │
│  • Use @> instead of multiple ? checks                              │
│  • Create GIN index for containment queries                         │
│  • Use expression index for specific paths                          │
│  • Avoid <@ if possible (harder to index)                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```sql
-- GIN index للـ containment
CREATE INDEX idx_products_metadata ON products USING gin(metadata);

-- Expression index لـ specific path
CREATE INDEX idx_products_brand ON products((metadata ->> 'brand'));

-- الآن ده هيستخدم الـ index
SELECT * FROM products WHERE metadata ->> 'brand' = 'Apple';
```

---

## ✅ Key Takeaways

<div dir="rtl">

1. **->** للـ JSON، **->>** للـ TEXT
2. **@>** للـ containment (الأكثر استخداماً للبحث)
3. **?** للـ key existence
4. **||** للدمج، **-** للحذف
5. **#>** و **#-** للـ nested paths
6. استخدم **GIN index** للـ @> و ? operators

</div>

---

## ⏭️ الدرس التالي

<div dir="rtl">

**➡️ [JSONB Indexing](./06-jsonb-indexing.md)**

</div>

---

<div align="center">

[⬅️ السابق](./04-json-jsonb.md) | [🏠 العودة للـ Module](../README.md) | [الدرس التالي ➡️](./06-jsonb-indexing.md)

</div>
