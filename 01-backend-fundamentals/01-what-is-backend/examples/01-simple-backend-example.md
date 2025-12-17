# Example: Simple Backend Workflow 🔄

<div dir="rtl">

## نظرة عامة

مثال عملي بسيط يوضح **workflow كامل** من Frontend للـ Backend والعكس.

السيناريو: **إضافة منتج للسلة (Add to Cart)**

</div>

---

## 🎯 The Scenario

<div dir="rtl">

**المستخدم:** يتصفح متجر إلكتروني  
**الفعل:** يضغط "Add to Cart" على منتج  
**النتيجة:** المنتج يضاف للسلة

**خلف الكواليس:** رحلة كاملة من Browser للـ Server!

</div>

---

## 📱 Part 1: Frontend (The Request)

### HTML Structure

```html
<!-- Product Card -->
<div class="product-card" data-product-id="42">
  <img src="/images/laptop.jpg" alt="Gaming Laptop" />
  <h3 class="product-name">ASUS ROG Gaming Laptop</h3>
  <p class="product-price">35,000 EGP</p>
  <p class="product-stock">In Stock: 5</p>

  <button class="add-to-cart-btn" onclick="addToCart(42)">
    <span class="btn-text">Add to Cart</span>
    <span class="btn-icon">🛒</span>
  </button>
</div>

<!-- Cart Counter (in navbar) -->
<div class="cart-counter"><span id="cart-count">0</span> items</div>
```

---

### JavaScript Logic

```javascript
// Function to add product to cart
async function addToCart(productId) {
  const button = event.target.closest(".add-to-cart-btn");
  const btnText = button.querySelector(".btn-text");

  // Step 1: Visual feedback (UX)
  btnText.textContent = "Adding...";
  button.disabled = true;

  try {
    // Step 2: Prepare request data
    const requestBody = {
      product_id: productId,
      quantity: 1,
    };

    // Step 3: Get auth token from localStorage
    const token = localStorage.getItem("auth_token");

    // Step 4: Send HTTP POST request to Backend
    const response = await fetch("https://api.shop.com/api/cart/add", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`, // Authentication
      },
      body: JSON.stringify(requestBody),
    });

    // Step 5: Parse response
    const data = await response.json();

    // Step 6: Handle response
    if (response.ok && data.success) {
      // Success!
      btnText.textContent = "Added ✓";
      button.classList.add("success");

      // Update cart counter
      document.getElementById("cart-count").textContent = data.cart_count;

      // Show notification
      showNotification("Product added to cart!", "success");

      // Reset button after 2 seconds
      setTimeout(() => {
        btnText.textContent = "Add to Cart";
        button.disabled = false;
        button.classList.remove("success");
      }, 2000);
    } else {
      // Error from Backend
      throw new Error(data.error || "Failed to add product");
    }
  } catch (error) {
    // Network error or Backend error
    console.error("Error:", error);

    btnText.textContent = "Add to Cart";
    button.disabled = false;

    showNotification(
      error.message || "Failed to add to cart. Please try again.",
      "error"
    );
  }
}

// Helper: Show notification
function showNotification(message, type) {
  const notification = document.createElement("div");
  notification.className = `notification notification-${type}`;
  notification.textContent = message;

  document.body.appendChild(notification);

  setTimeout(() => notification.remove(), 3000);
}
```

---

## 🔌 The HTTP Request

<div dir="rtl">

ما يُرسل من Frontend:

</div>

```http
POST /api/cart/add HTTP/1.1
Host: api.shop.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo1fQ...
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json

{
  "product_id": 42,
  "quantity": 1
}
```

---

## ⚙️ Part 2: Backend (The Processing)

### Go Backend Code

```go
package main

import (
    "database/sql"
    "net/http"
    "github.com/gin-gonic/gin"
    _ "github.com/lib/pq"
)

// Product struct
type Product struct {
    ID    int     `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price"`
    Stock int     `json:"stock"`
}

// CartItem struct
type CartItem struct {
    ID        int     `json:"id"`
    UserID    int     `json:"user_id"`
    ProductID int     `json:"product_id"`
    Quantity  int     `json:"quantity"`
    AddedAt   string  `json:"added_at"`
}

// Request body structure
type AddToCartRequest struct {
    ProductID int `json:"product_id" binding:"required"`
    Quantity  int `json:"quantity" binding:"required,min=1"`
}

// Handler: Add to Cart
func HandleAddToCart(c *gin.Context) {
    // Step 1: Extract user ID from JWT token (set by auth middleware)
    userID, exists := c.Get("user_id")
    if !exists {
        c.JSON(http.StatusUnauthorized, gin.H{
            "success": false,
            "error":   "Unauthorized",
        })
        return
    }

    // Step 2: Parse and validate request body
    var req AddToCartRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "success": false,
            "error":   "Invalid request: " + err.Error(),
        })
        return
    }

    // Step 3: Check if product exists and has stock
    product, err := getProduct(req.ProductID)
    if err != nil {
        c.JSON(http.StatusNotFound, gin.H{
            "success": false,
            "error":   "Product not found",
        })
        return
    }

    if product.Stock < req.Quantity {
        c.JSON(http.StatusBadRequest, gin.H{
            "success": false,
            "error":   "Insufficient stock",
            "available_stock": product.Stock,
        })
        return
    }

    // Step 4: Check if product already in cart
    existingItem, err := getCartItem(userID.(int), req.ProductID)

    if err == nil {
        // Product already in cart - update quantity
        newQuantity := existingItem.Quantity + req.Quantity

        // Check total quantity doesn't exceed stock
        if newQuantity > product.Stock {
            c.JSON(http.StatusBadRequest, gin.H{
                "success": false,
                "error":   "Cannot add more items. Total would exceed available stock.",
            })
            return
        }

        err = updateCartItemQuantity(existingItem.ID, newQuantity)
        if err != nil {
            c.JSON(http.StatusInternalServerError, gin.H{
                "success": false,
                "error":   "Failed to update cart",
            })
            return
        }
    } else {
        // New item - add to cart
        err = addCartItem(userID.(int), req.ProductID, req.Quantity)
        if err != nil {
            c.JSON(http.StatusInternalServerError, gin.H{
                "success": false,
                "error":   "Failed to add to cart",
            })
            return
        }
    }

    // Step 5: Get updated cart count
    cartCount, err := getCartCount(userID.(int))
    if err != nil {
        cartCount = 0
    }

    // Step 6: Send success response
    c.JSON(http.StatusOK, gin.H{
        "success":    true,
        "message":    "Product added to cart successfully",
        "cart_count": cartCount,
        "product": gin.H{
            "id":   product.ID,
            "name": product.Name,
        },
    })
}

// Database functions
func getProduct(productID int) (*Product, error) {
    var product Product
    query := `
        SELECT id, name, price, stock
        FROM products
        WHERE id = $1 AND deleted_at IS NULL
    `

    err := db.QueryRow(query, productID).Scan(
        &product.ID,
        &product.Name,
        &product.Price,
        &product.Stock,
    )

    return &product, err
}

func getCartItem(userID, productID int) (*CartItem, error) {
    var item CartItem
    query := `
        SELECT id, user_id, product_id, quantity, created_at
        FROM cart_items
        WHERE user_id = $1 AND product_id = $2
    `

    err := db.QueryRow(query, userID, productID).Scan(
        &item.ID,
        &item.UserID,
        &item.ProductID,
        &item.Quantity,
        &item.AddedAt,
    )

    return &item, err
}

func addCartItem(userID, productID, quantity int) error {
    query := `
        INSERT INTO cart_items (user_id, product_id, quantity, created_at)
        VALUES ($1, $2, $3, NOW())
    `

    _, err := db.Exec(query, userID, productID, quantity)
    return err
}

func updateCartItemQuantity(cartItemID, newQuantity int) error {
    query := `
        UPDATE cart_items
        SET quantity = $1, updated_at = NOW()
        WHERE id = $2
    `

    _, err := db.Exec(query, newQuantity, cartItemID)
    return err
}

func getCartCount(userID int) (int, error) {
    var count int
    query := `
        SELECT COALESCE(SUM(quantity), 0)
        FROM cart_items
        WHERE user_id = $1
    `

    err := db.QueryRow(query, userID).Scan(&count)
    return count, err
}
```

---

## 📤 The HTTP Response

### Success Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 10 Dec 2024 15:30:00 GMT

{
  "success": true,
  "message": "Product added to cart successfully",
  "cart_count": 3,
  "product": {
    "id": 42,
    "name": "ASUS ROG Gaming Laptop"
  }
}
```

### Error Response (Out of Stock)

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "success": false,
  "error": "Insufficient stock",
  "available_stock": 2
}
```

---

## 🗄️ Database Changes

<div dir="rtl">

### قبل الإضافة:

**Table: `cart_items`**

</div>

```
┌────┬─────────┬────────────┬──────────┬─────────────────────┐
│ id │ user_id │ product_id │ quantity │     created_at      │
├────┼─────────┼────────────┼──────────┼─────────────────────┤
│ 1  │    5    │     10     │    1     │ 2024-12-10 10:00:00 │
│ 2  │    5    │     15     │    2     │ 2024-12-10 11:30:00 │
└────┴─────────┴────────────┴──────────┴─────────────────────┘
```

<div dir="rtl">

### بعد الإضافة:

</div>

```
┌────┬─────────┬────────────┬──────────┬─────────────────────┐
│ id │ user_id │ product_id │ quantity │     created_at      │
├────┼─────────┼────────────┼──────────┼─────────────────────┤
│ 1  │    5    │     10     │    1     │ 2024-12-10 10:00:00 │
│ 2  │    5    │     15     │    2     │ 2024-12-10 11:30:00 │
│ 3  │    5    │     42     │    1     │ 2024-12-10 15:30:00 │ ← NEW!
└────┴─────────┴────────────┴──────────┴─────────────────────┘
```

---

## 🔄 Complete Flow Diagram

```
USER
  │
  └─► Clicks "Add to Cart" button
          │
          ▼
    ┌─────────────────┐
    │   FRONTEND      │
    │   (Browser)     │
    └─────────────────┘
          │
          │ 1. Collect data (product_id: 42)
          │ 2. Get auth token
          │ 3. Build HTTP Request
          │
          ▼
    POST /api/cart/add
    Authorization: Bearer xxx
    Body: { product_id: 42, quantity: 1 }
          │
          ▼
    ┌─────────────────┐
    │   BACKEND       │
    │   (Go Server)   │
    └─────────────────┘
          │
          │ 4. Verify JWT token
          │ 5. Validate request
          │ 6. Check product exists
          │ 7. Check stock available
          │ 8. Check if already in cart
          │
          ▼
    ┌─────────────────┐
    │   DATABASE      │
    │  (PostgreSQL)   │
    └─────────────────┘
          │
          │ 9. INSERT cart_item
          │ 10. SELECT cart count
          │
          ▼
    ┌─────────────────┐
    │   BACKEND       │
    └─────────────────┘
          │
          │ 11. Build response
          │
          ▼
    HTTP 200 OK
    { success: true, cart_count: 3 }
          │
          ▼
    ┌─────────────────┐
    │   FRONTEND      │
    └─────────────────┘
          │
          │ 12. Update UI
          │      - Button: "Added ✓"
          │      - Cart counter: "3 items"
          │      - Show notification
          │
          ▼
       USER SEES
    "Product added!"
```

---

## 💡 Key Points

<div dir="rtl">

### ✅ Frontend المسؤوليات:

- جمع البيانات من UI
- إرسال Request للـ Backend
- التعامل مع Response
- تحديث UI

### ✅ Backend المسؤوليات:

- Validation (Product exists? Stock available?)
- Business Logic (Already in cart? Update or add?)
- Database Operations
- Response building

### ✅ Security:

- JWT Token للـ Authentication
- Backend يتحقق من كل شيء
- Frontend لا يقدر يغش (كل الـ validation في Backend)

### ✅ Error Handling:

- Frontend: Network errors, UI feedback
- Backend: Business logic errors, Database errors

</div>

---

<div align="center">

[📚 Back to Examples](../README.md)

</div>
