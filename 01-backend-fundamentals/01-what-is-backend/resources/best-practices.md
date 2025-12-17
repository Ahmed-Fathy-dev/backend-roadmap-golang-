# Best Practices: Backend Development 🌟

<div dir="rtl">

## نظرة عامة

هذا الملف يحتوي على **أفضل الممارسات** في Backend Development التي يجب على كل Backend Developer احترافي معرفتها وتطبيقها.

</div>

---

## 🔐 Security Best Practices

### 1. Never Trust User Input

<div dir="rtl">

**القاعدة الذهبية:** كل input من المستخدم محتمل أن يكون خطير!

</div>

#### ❌ خطأ شائع:

```go
// DON'T DO THIS!
func GetUser(c *gin.Context) {
    id := c.Query("id")

    // Direct use of user input in SQL query - SQL INJECTION!
    query := "SELECT * FROM users WHERE id = " + id
    db.Query(query)
}
```

#### ✅ الطريقة الصحيحة:

```go
func GetUser(c *gin.Context) {
    // 1. Validate input
    idStr := c.Query("id")
    id, err := strconv.Atoi(idStr)
    if err != nil || id <= 0 {
        c.JSON(400, gin.H{"error": "Invalid user ID"})
        return
    }

    // 2. Use prepared statements (prevents SQL injection)
    query := "SELECT * FROM users WHERE id = $1"
    db.QueryRow(query, id)
}
```

---

### 2. Hash Passwords - ALWAYS!

#### ❌ NEVER do this:

```go
// EXTREMELY DANGEROUS!
user := User{
    Email: "user@test.com",
    Password: "mypassword123",  // Plain text!
}
db.Create(&user)
```

#### ✅ Always hash:

```go
import "golang.org/x/crypto/bcrypt"

func CreateUser(user *User) error {
    // Hash password before storing
    hashedPassword, err := bcrypt.GenerateFromPassword(
        []byte(user.Password),
        bcrypt.DefaultCost,  // Cost = 10 (good balance)
    )
    if err != nil {
        return err
    }

    user.Password = string(hashedPassword)
    return db.Create(user).Error
}

// Login: Compare hashed password
func Login(email, password string) (*User, error) {
    user, err := GetUserByEmail(email)
    if err != nil {
        return nil, err
    }

    // Compare hash
    err = bcrypt.CompareHashAndPassword(
        []byte(user.Password),
        []byte(password),
    )
    if err != nil {
        return nil, errors.New("invalid credentials")
    }

    return user, nil
}
```

---

### 3. Use Environment Variables for Secrets

#### ❌ خطأ فادح:

```go
// NEVER commit secrets to Git!
const DB_PASSWORD = "mySecretPassword123"
const JWT_SECRET = "super-secret-key"
const API_KEY = "sk_live_abc123xyz"
```

#### ✅ الطريقة الصحيحة:

```go
import "os"

// Load from environment variables
dbPassword := os.Getenv("DB_PASSWORD")
jwtSecret := os.Getenv("JWT_SECRET")
apiKey := os.Getenv("API_KEY")

// Use .env file (with godotenv)
import "github.com/joho/godotenv"

func init() {
    godotenv.Load()  // Load .env file
}
```

**📝 .env file (never commit to Git!):**

```bash
DB_PASSWORD=mySecretPassword123
JWT_SECRET=super-secret-key-xyz
API_KEY=sk_live_abc123xyz
```

**📝 .gitignore:**

```
.env
*.env
```

---

### 4. Implement Rate Limiting

<div dir="rtl">

لمنع Brute Force Attacks:

</div>

```go
import "github.com/ulule/limiter/v3"

// Allow 10 requests per minute
rate := limiter.Rate{
    Period: 1 * time.Minute,
    Limit:  10,
}

// Apply to login endpoint
router.POST("/login", RateLimitMiddleware(rate), HandleLogin)
```

---

### 5. Validate & Sanitize Input

```go
import "github.com/go-playground/validator/v10"

type CreateUserRequest struct {
    Name     string `json:"name" binding:"required,min=3,max=50"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
    Age      int    `json:"age" binding:"required,gte=18,lte=120"`
}

func CreateUser(c *gin.Context) {
    var req CreateUserRequest

    // Automatic validation
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }

    // req is now validated!
}
```

---

## ⚡ Performance Best Practices

### 1. Use Database Indexing

#### ❌ Without Index (SLOW):

```sql
-- Searching 1 million users
SELECT * FROM users WHERE email = 'user@example.com';
→ Full table scan: checks all 1,000,000 rows 😱
→ Time: ~500ms
```

#### ✅ With Index (FAST):

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Now same query:
SELECT * FROM users WHERE email = 'user@example.com';
→ Index lookup: finds row instantly ⚡
→ Time: ~2ms
```

**في Go:**

```go
// GORM migration
db.AutoMigrate(&User{})

// Add index
db.Exec("CREATE INDEX idx_users_email ON users(email)")
```

---

### 2. Use Connection Pooling

```go
import "database/sql"

// Configure connection pool
db, err := sql.Open("postgres", dsn)

// Set pool limits
db.SetMaxOpenConns(25)          // Max connections
db.SetMaxIdleConns(5)           // Idle connections
db.SetConnMaxLifetime(5 * time.Minute)
```

**لماذا؟**

- Opening DB connection = expensive (~100ms)
- Pooling = reuse connections = faster! ⚡

---

### 3. Implement Caching

```go
import "github.com/go-redis/redis/v8"

// Cache frequently accessed data
func GetUser(id int) (*User, error) {
    // 1. Check cache first
    cacheKey := fmt.Sprintf("user:%d", id)
    cached, err := redis.Get(ctx, cacheKey).Result()

    if err == nil {
        // Cache hit! ⚡
        var user User
        json.Unmarshal([]byte(cached), &user)
        return &user, nil
    }

    // 2. Cache miss - query database
    user, err := db.GetUser(id)
    if err != nil {
        return nil, err
    }

    // 3. Store in cache for next time
    jsonData, _ := json.Marshal(user)
    redis.Set(ctx, cacheKey, jsonData, 10*time.Minute)

    return user, nil
}
```

---

### 4. Use Pagination

#### ❌ Don't load everything:

```go
// Returns 100,000 products! 💥
func GetProducts(c *gin.Context) {
    var products []Product
    db.Find(&products)  // Loads ALL!
    c.JSON(200, products)
}
```

#### ✅ Use pagination:

```go
func GetProducts(c *gin.Context) {
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "20"))

    offset := (page - 1) * limit

    var products []Product
    var total int64

    db.Model(&Product{}).Count(&total)
    db.Limit(limit).Offset(offset).Find(&products)

    c.JSON(200, gin.H{
        "data":  products,
        "total": total,
        "page":  page,
        "pages": (total + int64(limit) - 1) / int64(limit),
    })
}
```

---

### 5. Optimize Database Queries

#### ❌ N+1 Query Problem:

```go
// Gets users
users := GetUsers()  // 1 query

// For each user, get their posts
for _, user := range users {
    user.Posts = GetUserPosts(user.ID)  // N queries!
}
// Total: 1 + N queries 😱
```

#### ✅ Use Eager Loading:

```go
// GORM: Load users WITH posts in ONE query
var users []User
db.Preload("Posts").Find(&users)  // 1 query only! ⚡
```

---

## 🏗️ Architecture Best Practices

### 1. Separation of Concerns

```
project/
├── handlers/       # HTTP handlers (controllers)
├── services/       # Business logic
├── models/         # Data models
├── repositories/   # Database operations
├── middleware/     # Middleware functions
└── utils/          # Helper functions
```

**Example:**

```go
// ❌ DON'T mix everything:
func CreateUser(c *gin.Context) {
    // Parsing, validation, business logic, database, response
    // all in one function! 😱
}

// ✅ DO separate:

// Handler: HTTP layer
func CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }

    user, err := userService.Create(req)
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }

    c.JSON(201, user)
}

// Service: Business logic
func (s *UserService) Create(req CreateUserRequest) (*User, error) {
    hashedPwd, _ := bcrypt.GenerateFromPassword(...)
    user := &User{...}
    return s.repo.Create(user)
}

// Repository: Database
func (r *UserRepository) Create(user *User) error {
    return r.db.Create(user).Error
}
```

---

### 2. Error Handling

#### ✅ Proper error handling:

```go
func GetUser(id int) (*User, error) {
    user, err := db.GetUser(id)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, ErrUserNotFound  // Custom error
        }
        // Log the error
        log.Printf("Database error: %v", err)
        return nil, err
    }
    return user, nil
}

// Custom errors
var (
    ErrUserNotFound = errors.New("user not found")
    ErrInvalidInput = errors.New("invalid input")
)
```

---

### 3. Use Middleware for Common Tasks

```go
// Authentication middleware
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")

        if token == "" {
            c.AbortWithStatusJSON(401, gin.H{"error": "unauthorized"})
            return
        }

        claims, err := VerifyToken(token)
        if err != nil {
            c.AbortWithStatusJSON(401, gin.H{"error": "invalid token"})
            return
        }

        // Store user info in context
        c.Set("user_id", claims.UserID)
        c.Next()  // Continue to handler
    }
}

// Apply to protected routes
router.Group("/api").Use(AuthMiddleware()).{
    // All routes here require auth
}
```

---

## 📝 Code Quality Best Practices

### 1. Use Meaningful Names

```go
// ❌ Bad:
func p(u User) {}
func calc(a, b int) int {}
var x = "test"

// ✅ Good:
func ProcessUser(user User) {}
func CalculateTotal(price, quantity int) int {}
var emailAddress = "test@example.com"
```

---

### 2. Write Tests

```go
func TestCreateUser(t *testing.T) {
    // Arrange
    user := &User{
        Name:  "Ahmed",
        Email: "ahmed@test.com",
    }

    // Act
    result, err := service.CreateUser(user)

    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, result)
    assert.Greater(t, result.ID, 0)
}
```

---

### 3. Add Logging

```go
import "log"

func ProcessOrder(orderID int) error {
    log.Printf("Processing order %d", orderID)

    err := validateOrder(orderID)
    if err != nil {
        log.Printf("Order validation failed: %v", err)
        return err
    }

    log.Printf("Order %d processed successfully", orderID)
    return nil
}
```

---

## 💡 General Best Practices

<div dir="rtl">

### ✅ افعل:

1. **Validate input** - دائماً
2. **Hash passwords** - bcrypt
3. **Use HTTPS** - في Production
4. **Implement rate limiting** - لمنع attacks
5. **Log errors** - للـ debugging
6. **Write tests** - Unit + Integration
7. **Use connection pooling** - للأداء
8. **Index database** - للسرعة
9. **Implement caching** - للبيانات الثابتة
10. **Separate concerns** - Clean Architecture

### ❌ لا تفعل:

1. **لا تخزن passwords** كـ plain text
2. **لا تعرض secrets** في Git
3. **لا تثق في user input** أبداً
4. **لا تحمّل كل البيانات** مرة واحدة
5. **لا تتجاهل errors** - handle them!
6. **لا تستخدم GET** للعمليات التي تغير البيانات
7. **لا تنسى CORS** configuration
8. **لا تترك SQL injection** vulnerabilities

</div>

---

<div align="center">

[📚 Back to Module Home](../README.md)

</div>
