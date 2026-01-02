# Lesson 4: Input Validation - التحقق من المدخلات ✅

<div dir="rtl">

## المقدمة

**Rule #1:** Never trust user input!

كل data جاية من الـ client ممكن تكون malicious. الـ validation الصحيح بيحميك من معظم الهجمات.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 Why Validate?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Why Input Validation?                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  User Input Can Be:                                                  │
│  ─────────────────                                                   │
│  • Empty/Null             → App crashes                             │
│  • Wrong type             → Type errors                             │
│  • Too long               → Buffer overflow, DB errors              │
│  • SQL injection          → Database compromised                    │
│  • XSS script             → Users attacked                          │
│  • Path traversal         → File system access                      │
│  • Command injection      → Server compromised                      │
│                                                                      │
│  Input Sources:                                                      │
│  ──────────────                                                      │
│  • Request body (JSON, form data)                                   │
│  • Query parameters (?id=123)                                       │
│  • Path parameters (/users/:id)                                     │
│  • Headers (Authorization, Custom headers)                          │
│  • Cookies                                                          │
│  • File uploads                                                      │
│                                                                      │
│  Validate ALL of them!                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Validation vs Sanitization

```
Validation = Check if input is valid
Sanitization = Clean/transform input

Example: User email input

Input: "  AHMED@test.com <script>alert('xss')</script>  "

Validation:
├── Is it a valid email format? ❌ No (contains HTML)
└── Action: Reject with error

Sanitization:
├── Trim whitespace
├── Convert to lowercase
├── Remove HTML tags
└── Result: "ahmed@test.com"

Best Practice: Validate first, then sanitize
```

---

## 2️⃣ Validation in Go

### Using go-playground/validator

```go
import (
    "github.com/go-playground/validator/v10"
)

var validate = validator.New()

type CreateUserRequest struct {
    Email     string `json:"email" validate:"required,email,max=255"`
    Password  string `json:"password" validate:"required,min=8,max=100"`
    Name      string `json:"name" validate:"required,min=2,max=100"`
    Age       int    `json:"age" validate:"omitempty,gte=0,lte=150"`
    Role      string `json:"role" validate:"required,oneof=user admin moderator"`
    Website   string `json:"website" validate:"omitempty,url"`
    Phone     string `json:"phone" validate:"omitempty,e164"`
}

func CreateUser(c *gin.Context) {
    var req CreateUserRequest

    // Bind JSON
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": "Invalid JSON"})
        return
    }

    // Validate
    if err := validate.Struct(req); err != nil {
        errors := formatValidationErrors(err)
        c.JSON(400, gin.H{"errors": errors})
        return
    }

    // Input is valid, proceed
    // ...
}

func formatValidationErrors(err error) map[string]string {
    errors := make(map[string]string)

    for _, e := range err.(validator.ValidationErrors) {
        field := strings.ToLower(e.Field())
        switch e.Tag() {
        case "required":
            errors[field] = fmt.Sprintf("%s is required", field)
        case "email":
            errors[field] = "Invalid email format"
        case "min":
            errors[field] = fmt.Sprintf("%s must be at least %s characters", field, e.Param())
        case "max":
            errors[field] = fmt.Sprintf("%s must be at most %s characters", field, e.Param())
        default:
            errors[field] = fmt.Sprintf("Invalid %s", field)
        }
    }

    return errors
}
```

### Common Validation Tags

```go
// Required/Optional
`validate:"required"`           // Must be present
`validate:"omitempty"`          // Skip if empty

// String length
`validate:"min=2"`              // Minimum length
`validate:"max=100"`            // Maximum length
`validate:"len=10"`             // Exact length

// Numeric range
`validate:"gte=0"`              // Greater than or equal
`validate:"lte=100"`            // Less than or equal
`validate:"gt=0"`               // Greater than
`validate:"lt=100"`             // Less than

// Format
`validate:"email"`              // Valid email
`validate:"url"`                // Valid URL
`validate:"uuid"`               // Valid UUID
`validate:"datetime=2006-01-02"` // Date format

// Choices
`validate:"oneof=active inactive pending"`

// Custom patterns
`validate:"alphanum"`           // Letters and numbers only
`validate:"alpha"`              // Letters only
`validate:"numeric"`            // Numbers only

// Comparisons
`validate:"eqfield=ConfirmPassword"` // Must equal another field
`validate:"nefield=Username"`        // Must not equal another field
```

### Custom Validators

```go
func init() {
    validate.RegisterValidation("password_strength", validatePasswordStrength)
    validate.RegisterValidation("no_html", validateNoHTML)
}

func validatePasswordStrength(fl validator.FieldLevel) bool {
    password := fl.Field().String()

    hasUpper := false
    hasLower := false
    hasNumber := false
    hasSpecial := false

    for _, char := range password {
        switch {
        case unicode.IsUpper(char):
            hasUpper = true
        case unicode.IsLower(char):
            hasLower = true
        case unicode.IsNumber(char):
            hasNumber = true
        case unicode.IsPunct(char) || unicode.IsSymbol(char):
            hasSpecial = true
        }
    }

    return hasUpper && hasLower && hasNumber && hasSpecial
}

func validateNoHTML(fl validator.FieldLevel) bool {
    value := fl.Field().String()
    // Simple HTML tag detection
    matched, _ := regexp.MatchString("<[^>]*>", value)
    return !matched
}

// Usage
type SecureInput struct {
    Password string `validate:"required,min=8,password_strength"`
    Comment  string `validate:"required,no_html"`
}
```

---

## 3️⃣ Sanitization

### HTML Sanitization

```go
import (
    "github.com/microcosm-cc/bluemonday"
    "html"
)

// Option 1: Strip ALL HTML
func stripHTML(input string) string {
    p := bluemonday.StrictPolicy()
    return p.Sanitize(input)
}

// Option 2: Allow safe HTML
func sanitizeHTML(input string) string {
    p := bluemonday.UGCPolicy()  // User Generated Content
    return p.Sanitize(input)
}

// Option 3: Escape HTML (display as text)
func escapeHTML(input string) string {
    return html.EscapeString(input)
}

// Examples:
input := "<script>alert('xss')</script><b>Hello</b>"

stripHTML(input)   // "Hello"
sanitizeHTML(input) // "<b>Hello</b>"
escapeHTML(input)  // "&lt;script&gt;alert('xss')&lt;/script&gt;&lt;b&gt;Hello&lt;/b&gt;"
```

### String Sanitization

```go
func sanitizeString(input string) string {
    // Trim whitespace
    input = strings.TrimSpace(input)

    // Remove control characters
    input = strings.Map(func(r rune) rune {
        if r < 32 && r != '\n' && r != '\r' && r != '\t' {
            return -1
        }
        return r
    }, input)

    // Normalize whitespace
    input = strings.Join(strings.Fields(input), " ")

    return input
}

func sanitizeEmail(email string) string {
    email = strings.TrimSpace(email)
    email = strings.ToLower(email)
    return email
}

func sanitizeUsername(username string) string {
    username = strings.TrimSpace(username)
    username = strings.ToLower(username)
    // Remove non-alphanumeric except underscore
    reg := regexp.MustCompile(`[^a-z0-9_]`)
    username = reg.ReplaceAllString(username, "")
    return username
}
```

---

## 4️⃣ Path Parameter Validation

```go
// ❌ Dangerous: No validation
func GetUser(c *gin.Context) {
    id := c.Param("id")
    user, _ := db.Query("SELECT * FROM users WHERE id = " + id)
    // SQL Injection possible!
}

// ✅ Safe: Validate and use parameterized query
func GetUser(c *gin.Context) {
    idStr := c.Param("id")

    // Validate: must be numeric
    id, err := strconv.Atoi(idStr)
    if err != nil || id <= 0 {
        c.JSON(400, gin.H{"error": "Invalid user ID"})
        return
    }

    // Use parameterized query
    var user User
    err = db.QueryRow("SELECT * FROM users WHERE id = $1", id).Scan(&user)
    // ...
}

// For UUIDs
func GetResource(c *gin.Context) {
    idStr := c.Param("id")

    // Validate UUID format
    id, err := uuid.Parse(idStr)
    if err != nil {
        c.JSON(400, gin.H{"error": "Invalid resource ID"})
        return
    }

    // Use validated UUID
    // ...
}
```

---

## 5️⃣ Query Parameter Validation

```go
type ListUsersQuery struct {
    Page     int    `form:"page" validate:"omitempty,gte=1"`
    Limit    int    `form:"limit" validate:"omitempty,gte=1,lte=100"`
    Sort     string `form:"sort" validate:"omitempty,oneof=created_at name email"`
    Order    string `form:"order" validate:"omitempty,oneof=asc desc"`
    Status   string `form:"status" validate:"omitempty,oneof=active inactive pending"`
    Search   string `form:"search" validate:"omitempty,max=100"`
}

func ListUsers(c *gin.Context) {
    var query ListUsersQuery

    // Set defaults
    query.Page = 1
    query.Limit = 20
    query.Sort = "created_at"
    query.Order = "desc"

    // Bind and validate
    if err := c.ShouldBindQuery(&query); err != nil {
        c.JSON(400, gin.H{"error": "Invalid query parameters"})
        return
    }

    if err := validate.Struct(query); err != nil {
        c.JSON(400, gin.H{"error": "Invalid query parameters"})
        return
    }

    // Sanitize search
    query.Search = sanitizeString(query.Search)

    // Safe to use
    // ...
}
```

---

## 6️⃣ File Upload Validation

```go
func UploadAvatar(c *gin.Context) {
    file, header, err := c.Request.FormFile("avatar")
    if err != nil {
        c.JSON(400, gin.H{"error": "No file uploaded"})
        return
    }
    defer file.Close()

    // Validate file size (max 5MB)
    if header.Size > 5*1024*1024 {
        c.JSON(400, gin.H{"error": "File too large. Maximum 5MB"})
        return
    }

    // Validate file type by reading magic bytes
    buffer := make([]byte, 512)
    _, err = file.Read(buffer)
    if err != nil {
        c.JSON(400, gin.H{"error": "Cannot read file"})
        return
    }

    mimeType := http.DetectContentType(buffer)
    allowedTypes := map[string]bool{
        "image/jpeg": true,
        "image/png":  true,
        "image/gif":  true,
        "image/webp": true,
    }

    if !allowedTypes[mimeType] {
        c.JSON(400, gin.H{"error": "Invalid file type. Only images allowed"})
        return
    }

    // Reset file position
    file.Seek(0, 0)

    // Validate file name
    ext := filepath.Ext(header.Filename)
    if ext == "" || !strings.HasPrefix(mimeType, "image/") {
        c.JSON(400, gin.H{"error": "Invalid file"})
        return
    }

    // Generate safe filename
    newFilename := fmt.Sprintf("%s%s", uuid.New().String(), ext)

    // Save file...
}
```

---

## 7️⃣ Common Validation Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│                Common Validation Patterns                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Email:                                                              │
│  validate:"required,email,max=255"                                  │
│                                                                      │
│  Password:                                                           │
│  validate:"required,min=8,max=100"                                  │
│  + custom validator for complexity                                  │
│                                                                      │
│  Username:                                                           │
│  validate:"required,min=3,max=30,alphanum"                          │
│                                                                      │
│  Phone:                                                              │
│  validate:"required,e164" or custom regex                           │
│                                                                      │
│  URL:                                                                │
│  validate:"required,url"                                            │
│                                                                      │
│  ID (integer):                                                       │
│  validate:"required,gt=0"                                           │
│                                                                      │
│  ID (UUID):                                                          │
│  validate:"required,uuid"                                           │
│                                                                      │
│  Date:                                                               │
│  validate:"required,datetime=2006-01-02"                            │
│                                                                      │
│  Enum/Status:                                                        │
│  validate:"required,oneof=active inactive pending"                  │
│                                                                      │
│  Amount (money):                                                     │
│  validate:"required,gte=0" + decimal precision check               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **Never trust user input** - validate everything
- ✅ **Validate then sanitize** - reject bad input first
- ✅ **Use struct tags** - go-playground/validator
- ✅ **Parameterized queries** - prevent SQL injection
- ✅ **Whitelist file types** - check magic bytes, not extension
- ✅ **Validate on server** - client validation is not enough

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 5: Common Attacks](./05-common-attacks.md)**

</div>

---

<div align="center">

[⬅️ Previous: Rate Limiting](./03-rate-limiting.md) | [📚 Module Home](../README.md) | [➡️ Next: Common Attacks](./05-common-attacks.md)

</div>
