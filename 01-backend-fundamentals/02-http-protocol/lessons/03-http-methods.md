# Lesson 3: HTTP Methods in Detail 🔧

<div dir="rtl">

## المقدمة

في هذا الدرس، سنتعلم **HTTP Methods** بالتفصيل الممل - كل method، استخدامه، خصائصه، و best practices.

</div>

---

## 🎯 What are HTTP Methods?

<div dir="rtl">

**HTTP Method** (أو HTTP Verb) يحدد **نوع العملية** المطلوب تنفيذها على المورد (Resource).

### التشبيه:

تخيل Method كأنه "فعل" في الجملة:

- "**احصل على** المنتجات" → GET
- "**أنشئ** مستخدم جديد" → POST
- "**حدّث** بيانات المستخدم" → PUT
- "**احذف** المنتج" → DELETE

</div>

---

## 📚 The Main HTTP Methods

### Overview Table

| Method      | الغرض                                       | Has Body? | Safe? | Idempotent? |
| ----------- | ------------------------------------------- | --------- | ----- | ----------- |
| **GET**     | <div dir="rtl">قراءة</div>                  | ❌        | ✅    | ✅          |
| **POST**    | <div dir="rtl">إنشاء</div>                  | ✅        | ❌    | ❌          |
| **PUT**     | <div dir="rtl">تحديث كامل</div>             | ✅        | ❌    | ✅          |
| **PATCH**   | <div dir="rtl">تحديث جزئي</div>             | ✅        | ❌    | ❌\*        |
| **DELETE**  | <div dir="rtl">حذف</div>                    | ❌\*      | ❌    | ✅          |
| **HEAD**    | <div dir="rtl">Headers فقط</div>            | ❌        | ✅    | ✅          |
| **OPTIONS** | <div dir="rtl">معرفة Methods المدعومة</div> | ❌        | ✅    | ✅          |

<div dir="rtl">

### مصطلحات مهمة:

**Safe (آمن):**  
لا يغير حالة السيرفر - قراءة فقط

**Idempotent (متماثل):**  
تنفيذه مرة = تنفيذه 100 مرة (نفس النتيجة)

</div>

---

## 1️⃣ GET Method

<div dir="rtl">

### الاستخدام:

**قراءة/جلب البيانات** من السيرفر

</div>

### Characteristics:

- ✅ **Safe**: لا يغير البيانات
- ✅ **Idempotent**: القراءة 100 مرة = نفس النتيجة
- ✅ **Cacheable**: يمكن حفظه في Cache
- ❌ **No Body**: البيانات في URL (Query String)

### Syntax:

```http
GET /api/products HTTP/1.1
Host: shop.com
```

### Examples:

```http
# Get all products
GET /api/products

# Get specific product
GET /api/products/42

# With filters (Query String)
GET /api/products?category=laptops&price_min=500&price_max=2000

# With pagination
GET /api/products?page=2&limit=20

# With sorting
GET /api/products?sort=price&order=desc
```

### ✅ Best Practices:

```go
// ✅ DO: Use GET for reading
GET /api/users           // Get all users
GET /api/users/5         // Get user with ID 5
GET /api/users/5/posts   // Get posts for user 5

// ❌ DON'T: Use GET for actions that change data
GET /api/users/5/delete  // WRONG! Use DELETE
GET /api/users/create    // WRONG! Use POST
```

### ⚠️ Security Warning:

```http
# ❌ NEVER put sensitive data in URL!
GET /api/login?email=user@test.com&password=mypass123

# Why? URLs are:
# - Logged in browser history
# - Logged in server logs
# - Visible in network traffic
# - Shared/bookmarked

# ✅ Use POST with body instead
POST /api/login
Body: { "email": "...", "password": "..." }
```

---

## 2️⃣ POST Method

<div dir="rtl">

### الاستخدام:

**إنشاء مورد جديد** على السيرفر

</div>

### Characteristics:

- ✅ **Has Body**: البيانات في Request Body
- ❌ **Not Safe**: يغير حالة السيرفر
- ❌ **Not Idempotent**: كل طلب ينشئ مورد جديد
- ❌ **Not Cacheable**: لا يمكن حفظه

### Syntax:

```http
POST /api/products HTTP/1.1
Host: shop.com
Content-Type: application/json

{
  "name": "Gaming Laptop",
  "price": 15000
}
```

### When to Use POST:

```http
# ✅ Creating new resources
POST /api/users          # Create new user
POST /api/products       # Create new product
POST /api/orders         # Create new order

# ✅ Actions/Operations
POST /api/auth/login     # Login action
POST /api/password/reset # Reset password
POST /api/cart/checkout  # Checkout action

# ✅ File uploads
POST /api/upload         # Upload file

# ✅ Search with complex criteria
POST /api/search         # Complex search (body has filters)
```

### Response Codes:

```http
# Success
201 Created              # Resource created successfully
Location: /api/products/123

# Errors
400 Bad Request          # Invalid input
409 Conflict             # Resource already exists
422 Unprocessable Entity # Validation failed
```

### ✅ Best Practices:

```go
// ✅ Return 201 Created (not 200 OK)
c.JSON(201, gin.H{
    "id": newProduct.ID,
    "message": "Product created successfully",
})

// ✅ Include Location header
c.Header("Location", fmt.Sprintf("/api/products/%d", newProduct.ID))

// ✅ Return the created resource
c.JSON(201, newProduct)

// ❌ DON'T return 200 OK for creation
c.JSON(200, ...) // Wrong! Use 201
```

---

## 3️⃣ PUT Method

<div dir="rtl">

### الاستخدام:

**تحديث كامل** لمورد موجود (Replace)

</div>

### Characteristics:

- ✅ **Has Body**: البيانات الكاملة في Body
- ✅ **Idempotent**: نفس الطلب 100 مرة = نفس النتيجة
- ❌ **Not Safe**: يغير البيانات
- ⚠️ **Replaces Entirely**: يستبدل المورد بالكامل!

### Syntax:

```http
PUT /api/products/42 HTTP/1.1
Content-Type: application/json

{
  "name": "Updated Laptop",
  "price": 16000,
  "stock": 10,
  "category": "electronics"
}
```

### PUT Behavior:

```
قبل PUT:
Product 42: { name: "Laptop", price: 15000, stock: 5 }

PUT /api/products/42
Body: { name: "New Laptop", price: 16000 }

بعد PUT:
Product 42: { name: "New Laptop", price: 16000, stock: NULL }
                                               ↑ Missing fields = NULL!
```

<div dir="rtl">

**⚠️ مهم:** PUT يستبدل المورد **بالكامل**!  
إذا لم ترسل field، يصبح NULL أو يُحذف.

</div>

### ✅ Best Practices:

```go
// ✅ PUT requires ALL fields
type UpdateProductRequest struct {
    Name     string  `json:"name" binding:"required"`
    Price    float64 `json:"price" binding:"required"`
    Stock    int     `json:"stock" binding:"required"`
    Category string  `json:"category" binding:"required"`
}

// ✅ Replace entirely
func UpdateProduct(c *gin.Context) {
    var req UpdateProductRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": "All fields required"})
        return
    }

    // Replace product completely
    product = Product{
        Name:     req.Name,
        Price:    req.Price,
        Stock:    req.Stock,
        Category: req.Category,
    }

    db.Save(&product)
    c.JSON(200, product)
}
```

---

## 4️⃣ PATCH Method

<div dir="rtl">

### الاستخدام:

**تحديث جزئي** (Partial Update)

</div>

### Characteristics:

- ✅ **Has Body**: الحقول المراد تحديثها فقط
- ❌ **Not Idempotent**: (في بعض الحالات)
- ❌ **Not Safe**: يغير البيانات
- ✅ **Partial**: يحدّث الحقول المحددة فقط

### Syntax:

```http
PATCH /api/products/42 HTTP/1.1
Content-Type: application/json

{
  "price": 14000
}
```

### PATCH Behavior:

```
قبل PATCH:
Product 42: { name: "Laptop", price: 15000, stock: 5 }

PATCH /api/products/42
Body: { price: 14000 }

بعد PATCH:
Product 42: { name: "Laptop", price: 14000, stock: 5 }
                              ↑ Only this changed!
```

### PUT vs PATCH:

| Aspect             | PUT                                     | PATCH                                     |
| ------------------ | --------------------------------------- | ----------------------------------------- |
| **Updates**        | <div dir="rtl">كل الحقول</div>          | <div dir="rtl">حقول محددة فقط</div>       |
| **Missing Fields** | <div dir="rtl">تصبح NULL</div>          | <div dir="rtl">تبقى كما هي</div>          |
| **Body Size**      | <div dir="rtl">كبير (كل البيانات)</div> | <div dir="rtl">صغير (التغييرات فقط)</div> |
| **Idempotent**     | ✅ Yes                                  | ⚠️ Depends                                |

### ✅ Best Practices:

```go
// ✅ PATCH: Optional fields
type PatchProductRequest struct {
    Name     *string  `json:"name"`      // Pointer = optional
    Price    *float64 `json:"price"`
    Stock    *int     `json:"stock"`
    Category *string  `json:"category"`
}

func PatchProduct(c *gin.Context) {
    var req PatchProductRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }

    // Get existing product
    product, _ := GetProduct(id)

    // Update only provided fields
    if req.Name != nil {
        product.Name = *req.Name
    }
    if req.Price != nil {
        product.Price = *req.Price
    }
    if req.Stock != nil {
        product.Stock = *req.Stock
    }

    db.Save(&product)
    c.JSON(200, product)
}
```

---

## 5️⃣ DELETE Method

<div dir="rtl">

### الاستخدام:

**حذف مورد**

</div>

### Characteristics:

- ✅ **Idempotent**: حذف نفس المورد 100 مرة = نفس النتيجة
- ❌ **Not Safe**: يغير البيانات
- ❌ **Usually No Body**: Body اختياري

### Syntax:

```http
DELETE /api/products/42 HTTP/1.1
```

### Response Codes:

```http
# Success
200 OK                   # Deleted, returns deleted resource
204 No Content           # Deleted, no body (recommended)

# Errors
404 Not Found            # Resource doesn't exist
403 Forbidden            # Not allowed to delete
409 Conflict             # Can't delete (has dependencies)
```

### ✅ Best Practices:

```go
// ✅ Soft Delete (Recommended)
func DeleteProduct(c *gin.Context) {
    id := c.Param("id")

    // Don't actually delete - mark as deleted
    db.Model(&Product{}).
       Where("id = ?", id).
       Update("deleted_at", time.Now())

    c.Status(204)  // No Content
}

// Alternative: Hard Delete
func HardDeleteProduct(c *gin.Context) {
    id := c.Param("id")

    result := db.Delete(&Product{}, id)

    if result.RowsAffected == 0 {
        c.JSON(404, gin.H{"error": "Product not found"})
        return
    }

    c.Status(204)
}
```

### Idempotent DELETE:

```
First DELETE:
DELETE /api/products/42
Response: 204 No Content (deleted successfully)

Second DELETE:
DELETE /api/products/42
Response: 204 No Content (still success! idempotent)

Or: 404 Not Found (also acceptable)
```

---

## 6️⃣ HEAD Method

<div dir="rtl">

### الاستخدام:

مثل GET لكن **Headers فقط** (بدون Body)

</div>

### Use Cases:

```http
# Check if resource exists
HEAD /api/products/42
Response: 200 OK (exists) or 404 Not Found

# Check resource size before downloading
HEAD /api/files/large-video.mp4
Content-Length: 1073741824  # 1GB
→ Decide if you want to download

# Check if resource was modified
HEAD /api/products/42
Last-Modified: Mon, 09 Dec 2024 10:30:00 GMT
ETag: "abc123"
→ Use for caching decisions
```

---

## 7️⃣ OPTIONS Method

<div dir="rtl">

### الاستخدام:

معرفة **HTTP Methods المدعومة** للمورد

</div>

### Example:

```http
OPTIONS /api/products HTTP/1.1

Response:
HTTP/1.1 200 OK
Allow: GET, POST, HEAD, OPTIONS
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

### CORS Preflight:

```
Browser wants to make:
DELETE /api/products/42

Browser first sends:
OPTIONS /api/products/42
Origin: https://myapp.com

Server responds:
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, DELETE

→ Browser now knows DELETE is allowed
→ Proceeds with actual DELETE request
```

---

## 💡 Method Selection Guide

<div dir="rtl">

### كيف تختار Method المناسب؟

</div>

| I want to...                               | Use         |
| ------------------------------------------ | ----------- |
| <div dir="rtl">قراءة بيانات</div>          | **GET**     |
| <div dir="rtl">إنشاء مورد جديد</div>       | **POST**    |
| <div dir="rtl">تحديث كل حقوله</div>        | **PUT**     |
| <div dir="rtl">تحديث بعض حقوله</div>       | **PATCH**   |
| <div dir="rtl">حذف مورد</div>              | **DELETE**  |
| <div dir="rtl">فحص وجود مورد</div>         | **HEAD**    |
| <div dir="rtl">معرفة Methods المتاحة</div> | **OPTIONS** |

---

## 🔐 Security Best Practices

### 1. Use Correct Method

```go
// ❌ WRONG: Using GET for actions
GET /api/products/delete/42    // Dangerous!
GET /api/users/update?id=5&name=Hacker

// Why dangerous?
// - GET requests cached
// - URLs logged
// - Browser prefetch might trigger them!

// ✅ CORRECT:
DELETE /api/products/42
PATCH /api/users/5
```

### 2. Never Trust Method Alone

```go
// ❌ WRONG: Trusting method for security
func DeleteProduct(c *gin.Context) {
    // Anyone can delete! 😱
    db.Delete(&Product{}, c.Param("id"))
}

// ✅ CORRECT: Check authorization
func DeleteProduct(c *gin.Context) {
    userID := c.GetInt("user_id")  // From JWT
    product := GetProduct(c.Param("id"))

    // Check permission
    if product.OwnerID != userID && !IsAdmin(userID) {
        c.JSON(403, gin.H{"error": "Forbidden"})
        return
    }

    db.Delete(&Product{}, product.ID)
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **GET**: قراءة (Safe, Idempotent, Cacheable)
- ✅ **POST**: إنشاء (Not Safe, Not Idempotent)
- ✅ **PUT**: تحديث كامل (Idempotent, Replaces)
- ✅ **PATCH**: تحديث جزئي (Partial update)
- ✅ **DELETE**: حذف (Idempotent)
- ✅ استخدم Method الصحيح للعملية الصحيحة
- ✅ لا تضع Sensitive Data في GET URLs
- ✅ استخدم 201 لـ POST, 204 لـ DELETE

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد أن فهمت Methods، دعنا نفهم Response Structure:

**➡️ [Lesson 4: HTTP Response Structure](./04-response-structure.md)**

</div>

---

<div align="center">

[⬅️ Previous: Request Structure](./02-request-structure.md) | [📚 Module Home](../README.md)

</div>
