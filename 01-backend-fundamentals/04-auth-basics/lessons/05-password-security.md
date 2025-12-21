# Lesson 5: Password Security 🔐

<div dir="rtl">

## المقدمة

**Password security** من أهم الأشياء في Backend Development!

خطأ واحد = كل passwords المستخدمين مكشوفة!

</div>

---

## ❌ NEVER Do This!

```go
// ❌ EXTREMELY DANGEROUS!
type User struct {
    Email    string
    Password string  // Plain text! 💀
}

db.Create(&User{
    Email: "ahmed@test.com",
    Password: "mypassword123",  // Stored as-is!
})
```

<div dir="rtl">

**لماذا خطير؟**

- Database breach → كل passwords مكشوفة
- Admin يقدر يشوف passwords
- لو user يستخدم نفس password في مواقع أخرى → كل حساباته في خطر!

</div>

---

## ✅ The Correct Way: Hashing

### What is Hashing?

```
Password: "mypassword123"
    ↓ Hash Function (bcrypt)
Hash: "$2a$10$N9qo8uLOickgx2ZMRZoMye..."

خصائص:
✅ One-way: لا يمكن العكس (hash → password)
✅ Same input = same output
✅ Different input = completely different output
✅ Slow (intentionally) → يمنع brute force
```

---

## 🔐 bcrypt - The Gold Standard

### Why bcrypt?

```
✅ Slow (good for passwords!)
✅ Built-in salt
✅ Adjustable cost/work factor
✅ Industry standard
```

### Implementation:

```go
import "golang.org/x/crypto/bcrypt"

// STORING password
func CreateUser(email, password string) error {
    // Hash password
    hashedPassword, err := bcrypt.GenerateFromPassword(
        []byte(password),
        bcrypt.DefaultCost,  // Cost = 10 (good balance)
    )
    if err != nil {
        return err
    }

    // Store hash (NOT plain password!)
    user := &User{
        Email:    email,
        Password: string(hashedPassword),  // Hash stored
    }

    return db.Create(user).Error
}

// VERIFYING password
func Login(email, password string) (*User, error) {
    // Get user
    var user User
    if err := db.Where("email = ?", email).First(&user).Error; err != nil {
        return nil, errors.New("invalid credentials")
    }

    // Compare password hash
    err := bcrypt.CompareHashAndPassword(
        []byte(user.Password),  // Stored hash
        []byte(password),       // User input
    )

    if err != nil {
        return nil, errors.New("invalid credentials")
    }

    // ✅ Password correct!
    return &user, nil
}
```

---

## 🧂 Salt (Automatic in bcrypt)

<div dir="rtl">

**Salt** = قيمة عشوائية تُضاف للـ password قبل hashing

</div>

```
Without Salt (BAD):
User 1: password="123456" → hash="abc..."
User 2: password="123456" → hash="abc..."  ← Same hash!
→ Attacker knows both have same password

With Salt (GOOD):
User 1: password="123456" + salt="xyz" → hash="def..."
User 2: password="123456" + salt="pqr" → hash="ghi..."  ← Different!
→ Attacker can't tell they're the same
```

<div dir="rtl">

**bcrypt يضيف salt تلقائياً - مش لازم تعمله بنفسك!**

</div>

---

## ⚙️ Cost/Work Factor

```go
// Higher cost = more secure but slower
bcrypt.GenerateFromPassword(password, 10)  // Default (good)
bcrypt.GenerateFromPassword(password, 12)  // More secure (slower)
bcrypt.GenerateFromPassword(password, 14)  // Very secure (very slow)
```

<div dir="rtl">

**توصية:** 10-12 للـ production

</div>

---

## 🔒 Password Requirements

```go
type PasswordPolicy struct {
    MinLength      int
    RequireUpper   bool
    RequireLower   bool
    RequireNumber  bool
    RequireSpecial bool
}

func ValidatePassword(pwd string, policy PasswordPolicy) error {
    if len(pwd) < policy.MinLength {
        return fmt.Errorf("password must be at least %d characters", policy.MinLength)
    }

    if policy.RequireUpper && !regexp.MustCompile(`[A-Z]`).MatchString(pwd) {
        return errors.New("password must contain uppercase letter")
    }

    if policy.RequireLower && !regexp.MustCompile(`[a-z]`).MatchString(pwd) {
        return errors.New("password must contain lowercase letter")
    }

    if policy.RequireNumber && !regexp.MustCompile(`[0-9]`).MatchString(pwd) {
        return errors.New("password must contain number")
    }

    if policy.RequireSpecial && !regexp.MustCompile(`[!@#$%^&*]`).MatchString(pwd) {
        return errors.New("password must contain special character")
    }

    return nil
}

// Usage
policy := PasswordPolicy{
    MinLength:      8,
    RequireUpper:   true,
    RequireLower:   true,
    RequireNumber:  true,
    RequireSpecial: true,
}

if err := ValidatePassword("weak", policy); err != nil {
    // Password doesn't meet requirements
}
```

---

## 📋 Complete Registration Flow

```go
func Register(c *gin.Context) {
    var req struct {
        Email    string `json:"email" binding:"required,email"`
        Password string `json:"password" binding:"required,min=8"`
    }

    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }

    // 1. Validate password strength
    if err := ValidatePassword(req.Password, passwordPolicy); err != nil {
        c.JSON(422, gin.H{"error": err.Error()})
        return
    }

    // 2. Check if email exists
    var existingUser User
    if db.Where("email = ?", req.Email).First(&existingUser).Error == nil {
        c.JSON(409, gin.H{"error": "Email already registered"})
        return
    }

    // 3. Hash password
    hashedPassword, err := bcrypt.GenerateFromPassword(
        []byte(req.Password),
        bcrypt.DefaultCost,
    )
    if err != nil {
        c.JSON(500, gin.H{"error": "Error creating user"})
        return
    }

    // 4. Create user
    user := &User{
        Email:    req.Email,
        Password: string(hashedPassword),
    }

    if err := db.Create(user).Error; err != nil {
        c.JSON(500, gin.H{"error": "Error creating user"})
        return
    }

    // 5. Return success (DON'T return password!)
    c.JSON(201, gin.H{
        "id":    user.ID,
        "email": user.Email,
    })
}
```

---

## 🚨 Common Mistakes

### 1. Comparing Passwords Directly

```go
// ❌ WRONG!
if user.Password == inputPassword {
    // This compares hashes, will never work!
}

// ✅ CORRECT
err := bcrypt.CompareHashAndPassword(
    []byte(user.Password),
    []byte(inputPassword),
)
if err == nil {
    // Password correct
}
```

### 2. Weak Hashing

```go
// ❌ NEVER use MD5/SHA1 for passwords!
hash := md5.Sum([]byte(password))     // INSECURE!
hash := sha1.Sum([]byte(password))    // INSECURE!

// ✅ Use bcrypt
hash := bcrypt.GenerateFromPassword(password, bcrypt.DefaultCost)
```

### 3. No Requirements

```go
// ❌ Accepting weak passwords
password := "123"  // Too weak!

// ✅ En force requirements
MinLength: 8
Require: upper, lower, number, special
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **NEVER** store passwords as plain text
- ✅ Use **bcrypt** for hashing
- ✅ **Salt** automatic in bcrypt
- ✅ Use **cost 10-12**
- ✅ Enforce **password requirements**
- ✅ Use `CompareHashAndPassword` للمقارنة
- ✅ **Don't** return password in responses

</div>

---

<div align="center">

[⬅️ Previous: JWT](./04-jwt-deep-dive.md) | [➡️ Next: OAuth2](./06-oauth2.md) | [📚 Module Home](../README.md)

</div>
