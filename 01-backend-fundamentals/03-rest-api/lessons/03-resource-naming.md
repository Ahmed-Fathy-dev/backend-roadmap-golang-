# Lesson 3: Resource Naming Conventions 📝

<div dir="rtl">

## المقدمة

**تسمية Resources** من أهم قرارات تصميم API!

أسماء سيئة = API صعب الاستخدام  
أسماء جيدة = API واضح وسهل

</div>

---

## ✅ Golden Rules

### Rule 1: Use Nouns (Not Verbs)

```
# ❌ WRONG: Using verbs
/getProducts
/createUser
/deleteOrder
/updateProduct

# ✅ CORRECT: Using nouns
/products        # GET → get products
/users           # POST → create user
/orders/5        # DELETE → delete order
/products/10     # PUT → update product
```

<div dir="rtl">

**لماذا؟**  
HTTP Method يمثل الفعل، URL يمثل الاسم:

</div>

```
GET /products       → "Get products"
POST /products      → "Create product"
DELETE /orders/5    → "Delete order 5"
```

---

### Rule 2: Use Plural Nouns

```
# ❌ WRONG: Singular
/product
/user
/order

# ✅ CORRECT: Plural
/products
/users
/orders
```

<div dir="rtl">

**لماذا؟**  
Consistency! Collection و Resource:

</div>

```
GET /products       # Collection (many)
GET /products/5     # Resource (one from collection)

# Both use same base: /products
```

---

### Rule 3: Use Lowercase

```
# ❌ WRONG: Mixed case
/Products
/USERS
/MyOrders
/productCategories

# ✅ CORRECT: All lowercase
/products
/users
/my-orders
/product-categories
```

---

### Rule 4: Use Hyphens (Not Underscores)

```
# ❌ WRONG: Underscores or camelCase
/product_categories
/productCategories
/user_accounts

# ✅ CORRECT: Hyphens (kebab-case)
/product-categories
/user-accounts
/shopping-carts
```

<div dir="rtl">

**لماذا؟**

- URLs قد تكون underlined في بعض الأماكن
- Hyphens أسهل قراءة
- Industry standard

</div>

---

## 📍 Resource Naming Patterns

### Pattern 1: Collection & Resource

```
# Collection (plural)
GET    /products        # Get all products
POST   /products        # Create product

# Specific resource
GET    /products/42     # Get product 42
PUT    /products/42     # Update product 42
PATCH  /products/42     # Partial update
DELETE /products/42     # Delete product 42
```

### Pattern 2: Nested Resources

```
# User's posts
GET /users/5/posts              # All posts by user 5
GET /users/5/posts/10           # Post 10 by user 5

# Product reviews
GET /products/42/reviews        # Reviews for product 42
POST /products/42/reviews       # Add review to product 42

# Order items
GET /orders/99/items            # Items in order 99
```

<div dir="rtl">

**⚠️ تحذير:** لا تتعمق أكثر من مستويين!

</div>

```
# ❌ BAD: Too deep
/users/5/posts/10/comments/3/likes

# ✅ BETTER: Flatten
/comments/3/likes
/posts/10/comments
```

### Pattern 3: Actions (Non-CRUD)

<div dir="rtl">

أحياناً تحتاج actions ليست CRUD:

</div>

```
# ✅ Use verbs for actions (exception to Rule 1)
POST /users/5/activate          # Activate user
POST /orders/99/cancel          # Cancel order
POST /products/42/publish       # Publish product
POST /cart/checkout             # Checkout

# Alternative: Use resource for action
POST /activations               # Body: { "user_id": 5 }
POST /cancellations             # Body: { "order_id": 99 }
```

### Pattern 4: Filtering

```
# Use query parameters for filtering
GET /products?category=laptops
GET /products?price_min=500&price_max=2000
GET /users?status=active&role=admin
GET /orders?date_from=2024-01-01&date_to=2024-12-31
```

### Pattern 5: Sorting

```
GET /products?sort=price           # Ascending
GET /products?sort=-price          # Descending (- prefix)
GET /products?sort=name,created_at # Multiple fields
```

### Pattern 6: Pagination

```
# Page-based
GET /products?page=2&limit=20

# Offset-based
GET /products?offset=40&limit=20

# Cursor-based (for large datasets)
GET /products?cursor=abc123&limit=20
```

---

## 🎯 Real-World Examples

### E-Commerce API:

```
# Products
/products
/products/42
/products/42/reviews
/products/42/images
/product-categories
/product-categories/5/products

# Users
/users
/users/5
/users/5/orders
/users/5/addresses
/users/5/wishlist

# Orders
/orders
/orders/99
/orders/99/items
/orders/99/tracking

# Cart
/cart
/cart/items
/cart/checkout

# Search
/search/products?q=laptop
```

### Social Media API:

```
# Users
/users/5
/users/5/posts
/users/5/followers
/users/5/following

# Posts
/posts
/posts/10
/posts/10/comments
/posts/10/likes

# Actions
POST /posts/10/like
POST /users/5/follow
```

---

## ❌ Common Mistakes

### Mistake 1: Verbs in URLs

```
# ❌ WRONG
/getAllUsers
/createProduct
/deleteOrder/5
/updateUser

# ✅ CORRECT
GET    /users
POST   /products
DELETE /orders/5
PUT    /users/5
```

### Mistake 2: File Extensions

```
# ❌ WRONG
/users.json
/products.xml
/orders/5.json

# ✅ CORRECT
/users
/products
/orders/5

# Use Accept header instead
Accept: application/json
```

### Mistake 3: Trailing Slashes

```
# ❌ INCONSISTENT
/users/     # With slash
/products   # Without slash

# ✅ CONSISTENT: Pick one (without is common)
/users
/products
```

### Mistake 4: CRUD in URL

```
# ❌ WRONG
/users/create
/users/5/update
/users/5/delete

# ✅ CORRECT
POST   /users
PUT    /users/5
DELETE /users/5
```

---

## 📐 Naming Checklist

<div dir="rtl">

قبل ما تنشر API، تأكد:

</div>

- [ ] ✅ **Nouns** used (not verbs) - except actions
- [ ] ✅ **Plural** nouns for collections
- [ ] ✅ **Lowercase** everything
- [ ] ✅ **Hyphens** for multi-word (not underscores)
- [ ] ✅ **Consistent** naming across API
- [ ] ✅ **No file extensions** (.json, .xml)
- [ ] ✅ **No trailing slashes** (or always trailing - pick one)
- [ ] ✅ **Maximum 2 levels** for nested resources
- [ ] ✅ **Query parameters** for filtering/sorting/pagination
- [ ] ✅ **Meaningful** and **descriptive** names

---

## 💡 Quick Reference

### Resource Types:

| Type       | Singular | Plural | Example           |
| ---------- | -------- | ------ | ----------------- |
| Simple     | ❌       | ✅     | `/products`       |
| Composite  | ❌       | ✅     | `/shopping-carts` |
| Collection | ❌       | ✅     | `/users`          |
| Singleton  | ✅       | ❌     | `/profile`        |

### Multi-word Resources:

```
# ✅ Use hyphens
/product-categories
/user-profiles
/shopping-carts
/order-items
/payment-methods
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Nouns** not verbs (عدا الـ actions)
- ✅ **Plural** للـ collections
- ✅ **Lowercase** + **hyphens**
- ✅ **Nested resources** (max 2 levels)
- ✅ **Query params** للـ filtering/sorting
- ✅ **Consistency** أهم شيء!
- ✅ Think من وجهة نظر المستخدم - What makes sense?

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 4: HTTP Methods in REST](./04-http-methods-rest.md)**

</div>

---

<div align="center">

[⬅️ Previous: REST Principles](./02-rest-principles.md) | [📚 Module Home](../README.md)

</div>
