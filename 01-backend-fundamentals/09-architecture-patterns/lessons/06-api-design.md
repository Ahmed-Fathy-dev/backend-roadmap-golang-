# Lesson 6: API Design Best Practices - أفضل ممارسات تصميم API 🎨

<div dir="rtl">

## المقدمة

تصميم API جيد مهم جداً! API سيئ = developers غير سعيدين = adoption أقل. في هذا الدرس هنتعلم أفضل الممارسات.

**المدة المتوقعة:** 30 دقيقة

</div>

---

## 1️⃣ RESTful Naming Conventions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    URL Naming Conventions                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Good:                                                           │
│  ─────────                                                           │
│  GET    /users              List all users                          │
│  GET    /users/123          Get user 123                            │
│  POST   /users              Create new user                         │
│  PUT    /users/123          Update user 123 (full)                  │
│  PATCH  /users/123          Update user 123 (partial)               │
│  DELETE /users/123          Delete user 123                         │
│                                                                      │
│  ❌ Bad:                                                            │
│  ─────────                                                           │
│  GET    /getUsers           ❌ Verb in URL                          │
│  GET    /user/list          ❌ Inconsistent                         │
│  POST   /createUser         ❌ Verb in URL                          │
│  GET    /Users              ❌ Capital letters                      │
│  GET    /user_list          ❌ Underscores                          │
│                                                                      │
│  Rules:                                                             │
│  ───────                                                             │
│  • Use nouns, not verbs                                             │
│  • Use plural nouns (users, not user)                               │
│  • Use lowercase with hyphens                                       │
│  • Use HTTP methods for actions                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ Nested Resources

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Nested Resources                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  User's Orders:                                                     │
│  ──────────────                                                      │
│  GET    /users/123/orders           List user's orders              │
│  POST   /users/123/orders           Create order for user           │
│  GET    /users/123/orders/456       Get specific order              │
│                                                                      │
│  Post's Comments:                                                   │
│  ────────────────                                                    │
│  GET    /posts/1/comments           List post's comments            │
│  POST   /posts/1/comments           Add comment to post             │
│                                                                      │
│  ⚠️ Don't nest too deep:                                            │
│  ❌ /users/123/orders/456/items/789/variants/1                      │
│  ✅ /order-items/789                                                │
│                                                                      │
│  Rule: Max 2 levels of nesting                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ Query Parameters

```go
// Filtering
GET /users?status=active
GET /users?role=admin&status=active
GET /products?category=electronics&price_min=100&price_max=500

// Pagination
GET /users?page=2&per_page=20
GET /users?offset=20&limit=20
GET /users?cursor=abc123&limit=20  // Cursor-based (better for large datasets)

// Sorting
GET /users?sort=created_at         // Ascending
GET /users?sort=-created_at        // Descending (with -)
GET /users?sort=name,-created_at   // Multiple fields

// Field selection (sparse fieldsets)
GET /users?fields=id,name,email
GET /users/123?fields=name,email

// Search
GET /users?q=ahmed                 // Simple search
GET /users?search=ahmed@test.com   // Specific search

// Expanding relations
GET /orders/123?include=user,items
GET /posts?include=author,comments
```

### Implementation

```go
type ListUsersQuery struct {
    Status   string `form:"status"`
    Role     string `form:"role"`
    Page     int    `form:"page" binding:"min=1"`
    PerPage  int    `form:"per_page" binding:"min=1,max=100"`
    Sort     string `form:"sort"`
    Search   string `form:"q"`
}

func (h *UserHandler) List(c *gin.Context) {
    var query ListUsersQuery
    if err := c.ShouldBindQuery(&query); err != nil {
        c.JSON(400, ErrorResponse{Error: err.Error()})
        return
    }

    // Set defaults
    if query.Page == 0 {
        query.Page = 1
    }
    if query.PerPage == 0 {
        query.PerPage = 20
    }

    users, total, err := h.service.ListUsers(c.Request.Context(), query)
    if err != nil {
        c.JSON(500, ErrorResponse{Error: "Internal error"})
        return
    }

    c.JSON(200, ListResponse{
        Data: users,
        Meta: PaginationMeta{
            Page:       query.Page,
            PerPage:    query.PerPage,
            Total:      total,
            TotalPages: (total + query.PerPage - 1) / query.PerPage,
        },
    })
}
```

---

## 4️⃣ HTTP Status Codes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HTTP Status Codes                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  2xx Success:                                                       │
│  ─────────────                                                       │
│  200 OK              Successful GET, PUT, PATCH, DELETE             │
│  201 Created         Successful POST (resource created)             │
│  204 No Content      Successful DELETE (no body)                    │
│                                                                      │
│  4xx Client Errors:                                                 │
│  ───────────────────                                                 │
│  400 Bad Request     Invalid request body/params                    │
│  401 Unauthorized    Not authenticated                              │
│  403 Forbidden       Authenticated but not authorized               │
│  404 Not Found       Resource doesn't exist                         │
│  409 Conflict        Resource conflict (duplicate email)            │
│  422 Unprocessable   Validation error                               │
│  429 Too Many Reqs   Rate limit exceeded                            │
│                                                                      │
│  5xx Server Errors:                                                 │
│  ───────────────────                                                 │
│  500 Internal Error  Unexpected server error                        │
│  502 Bad Gateway     Upstream service error                         │
│  503 Unavailable     Server temporarily unavailable                 │
│  504 Gateway Timeout Upstream service timeout                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ Error Response Format

```go
// Consistent error format
type ErrorResponse struct {
    Error   string            `json:"error"`
    Code    string            `json:"code,omitempty"`
    Details map[string]string `json:"details,omitempty"`
}

// Validation errors
type ValidationErrorResponse struct {
    Error  string                 `json:"error"`
    Code   string                 `json:"code"`
    Fields map[string][]string    `json:"fields"`
}

// Examples:

// 400 Bad Request
{
    "error": "Invalid request body",
    "code": "INVALID_JSON"
}

// 401 Unauthorized
{
    "error": "Authentication required",
    "code": "AUTH_REQUIRED"
}

// 404 Not Found
{
    "error": "User not found",
    "code": "USER_NOT_FOUND"
}

// 422 Validation Error
{
    "error": "Validation failed",
    "code": "VALIDATION_ERROR",
    "fields": {
        "email": ["must be a valid email address"],
        "password": ["must be at least 8 characters"]
    }
}

// 500 Internal Error
{
    "error": "An unexpected error occurred",
    "code": "INTERNAL_ERROR"
}
// Note: Don't expose internal details in production!
```

### Error Handling Middleware

```go
// Custom error types
type AppError struct {
    Code       string
    Message    string
    StatusCode int
    Details    map[string]string
}

func (e AppError) Error() string {
    return e.Message
}

var (
    ErrNotFound = AppError{
        Code:       "NOT_FOUND",
        Message:    "Resource not found",
        StatusCode: 404,
    }
    ErrUnauthorized = AppError{
        Code:       "UNAUTHORIZED",
        Message:    "Authentication required",
        StatusCode: 401,
    }
    ErrForbidden = AppError{
        Code:       "FORBIDDEN",
        Message:    "Access denied",
        StatusCode: 403,
    }
)

// Error handling middleware
func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()

        if len(c.Errors) > 0 {
            err := c.Errors.Last().Err

            switch e := err.(type) {
            case AppError:
                c.JSON(e.StatusCode, ErrorResponse{
                    Error:   e.Message,
                    Code:    e.Code,
                    Details: e.Details,
                })
            case validator.ValidationErrors:
                fields := make(map[string][]string)
                for _, fe := range e {
                    field := strings.ToLower(fe.Field())
                    fields[field] = append(fields[field], getErrorMessage(fe))
                }
                c.JSON(422, ValidationErrorResponse{
                    Error:  "Validation failed",
                    Code:   "VALIDATION_ERROR",
                    Fields: fields,
                })
            default:
                // Don't expose internal errors
                c.JSON(500, ErrorResponse{
                    Error: "An unexpected error occurred",
                    Code:  "INTERNAL_ERROR",
                })
            }
        }
    }
}
```

---

## 6️⃣ API Versioning

```
┌─────────────────────────────────────────────────────────────────────┐
│                    API Versioning Strategies                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. URL Path (Most common)                                          │
│     GET /v1/users                                                   │
│     GET /v2/users                                                   │
│                                                                      │
│  2. Query Parameter                                                 │
│     GET /users?version=1                                            │
│     GET /users?version=2                                            │
│                                                                      │
│  3. Header                                                          │
│     GET /users                                                      │
│     Accept: application/vnd.myapi.v1+json                           │
│     X-API-Version: 1                                                │
│                                                                      │
│  Recommendation:                                                    │
│  • Use URL path versioning (/v1, /v2)                               │
│  • Simple, visible, easy to use                                     │
│  • Version the whole API, not individual endpoints                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```go
// URL path versioning
func SetupRouter() *gin.Engine {
    r := gin.Default()

    v1 := r.Group("/v1")
    {
        v1.GET("/users", v1Handlers.ListUsers)
        v1.POST("/users", v1Handlers.CreateUser)
    }

    v2 := r.Group("/v2")
    {
        v2.GET("/users", v2Handlers.ListUsers)  // New format
        v2.POST("/users", v2Handlers.CreateUser)
    }

    return r
}
```

---

## 7️⃣ Request/Response Design

### Consistent Response Format

```go
// Successful response
type SuccessResponse struct {
    Data interface{} `json:"data"`
    Meta interface{} `json:"meta,omitempty"`
}

// List response with pagination
type ListResponse struct {
    Data []interface{} `json:"data"`
    Meta PaginationMeta `json:"meta"`
}

type PaginationMeta struct {
    Page       int   `json:"page"`
    PerPage    int   `json:"per_page"`
    Total      int64 `json:"total"`
    TotalPages int   `json:"total_pages"`
}

// Single item response
// GET /users/123
{
    "data": {
        "id": 123,
        "name": "Ahmed",
        "email": "ahmed@test.com",
        "created_at": "2024-01-15T10:30:00Z"
    }
}

// List response
// GET /users
{
    "data": [
        {"id": 1, "name": "Ahmed"},
        {"id": 2, "name": "Mohamed"}
    ],
    "meta": {
        "page": 1,
        "per_page": 20,
        "total": 100,
        "total_pages": 5
    }
}

// Created response
// POST /users → 201 Created
{
    "data": {
        "id": 124,
        "name": "New User",
        "email": "new@test.com"
    }
}
```

### Request Body Design

```go
// Create request - only include required fields
type CreateUserRequest struct {
    Name     string `json:"name" binding:"required"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
}

// Update request - all fields optional (PATCH)
type UpdateUserRequest struct {
    Name  *string `json:"name,omitempty"`
    Email *string `json:"email,omitempty" binding:"omitempty,email"`
}

// Use pointers for optional fields to distinguish between:
// - Not provided (nil)
// - Provided as empty string ("")
```

---

## 8️⃣ HATEOAS (Optional but Useful)

```go
// Hypermedia As The Engine Of Application State
// Include links to related actions/resources

type UserResponse struct {
    ID    int64  `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
    Links Links  `json:"_links"`
}

type Links struct {
    Self   string `json:"self"`
    Orders string `json:"orders,omitempty"`
    Update string `json:"update,omitempty"`
    Delete string `json:"delete,omitempty"`
}

// Response:
{
    "data": {
        "id": 123,
        "name": "Ahmed",
        "email": "ahmed@test.com",
        "_links": {
            "self": "/v1/users/123",
            "orders": "/v1/users/123/orders",
            "update": "/v1/users/123",
            "delete": "/v1/users/123"
        }
    }
}
```

---

## 9️⃣ API Documentation

```go
// Use Swagger/OpenAPI
// go get -u github.com/swaggo/swag/cmd/swag
// go get -u github.com/swaggo/gin-swagger

// Add annotations to handlers
// @Summary Create a new user
// @Description Create a new user with the provided details
// @Tags users
// @Accept json
// @Produce json
// @Param user body CreateUserRequest true "User data"
// @Success 201 {object} SuccessResponse{data=UserResponse}
// @Failure 400 {object} ErrorResponse
// @Failure 422 {object} ValidationErrorResponse
// @Router /v1/users [post]
func (h *UserHandler) Create(c *gin.Context) {
    // ...
}

// Generate docs
// swag init -g cmd/server/main.go

// Serve Swagger UI
import swaggerFiles "github.com/swaggo/files"
import ginSwagger "github.com/swaggo/gin-swagger"

r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```

---

## ✅ API Design Checklist

```
┌─────────────────────────────────────────────────────────────────────┐
│                    API Design Checklist                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  URL Design:                                                        │
│  □ Use nouns, not verbs                                             │
│  □ Use plural nouns                                                 │
│  □ Use lowercase with hyphens                                       │
│  □ Max 2 levels of nesting                                          │
│                                                                      │
│  HTTP Methods:                                                      │
│  □ GET for read operations                                          │
│  □ POST for create                                                  │
│  □ PUT/PATCH for update                                             │
│  □ DELETE for delete                                                │
│                                                                      │
│  Responses:                                                         │
│  □ Consistent format                                                │
│  □ Proper status codes                                              │
│  □ Meaningful error messages                                        │
│  □ Pagination for lists                                             │
│                                                                      │
│  Security:                                                          │
│  □ Use HTTPS                                                        │
│  □ Validate input                                                   │
│  □ Authenticate requests                                            │
│  □ Rate limiting                                                    │
│                                                                      │
│  Documentation:                                                     │
│  □ OpenAPI/Swagger docs                                             │
│  □ Examples for all endpoints                                       │
│  □ Error code documentation                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Nouns** للـ URLs، **HTTP methods** للـ actions
- ✅ استخدم **plural nouns** (users, orders)
- ✅ **Consistent** error and success formats
- ✅ **Proper status codes** (201 للـ create، 404 للـ not found)
- ✅ **Pagination** للـ list endpoints
- ✅ **Version** الـ API (v1, v2)
- ✅ **Document** كل شيء!

</div>

---

## 🎉 Module Complete!

<div dir="rtl">

مبروك! أنت خلصت **Module 1.9: Architecture Patterns** 🎉

راجعنا:
- Monolith vs Microservices
- Layered Architecture
- Clean Architecture
- Repository Pattern
- Dependency Injection
- API Design Best Practices

</div>

---

<div align="center">

[⬅️ Previous: Dependency Injection](./05-dependency-injection.md) | [📚 Module Home](../README.md) | [🏠 Track 1](../../README.md)

</div>
