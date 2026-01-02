# Lesson 6: Security Headers - هيدرز الحماية 🛡️

<div dir="rtl">

## المقدمة

**Security Headers** هي HTTP headers بتقول للـ browser إزاي يتعامل مع الـ content بشكل آمن. إضافتها سهلة وبتحسن الـ security بشكل كبير.

**المدة المتوقعة:** 15 دقيقة

</div>

---

## 📊 Essential Security Headers

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Security Headers Overview                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Header                          │ Purpose                          │
│  ────────────────────────────────┼──────────────────────────────── │
│  Strict-Transport-Security       │ Force HTTPS                     │
│  Content-Security-Policy         │ Prevent XSS, code injection    │
│  X-Content-Type-Options          │ Prevent MIME sniffing          │
│  X-Frame-Options                 │ Prevent clickjacking           │
│  X-XSS-Protection                │ Browser XSS filter             │
│  Referrer-Policy                 │ Control referrer info          │
│  Permissions-Policy              │ Control browser features       │
│  Cache-Control                   │ Control caching                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Strict-Transport-Security (HSTS)

```
Purpose: Force browser to ALWAYS use HTTPS
```

```go
// Header
c.Header("Strict-Transport-Security", "max-age=31536000; includeSubDomains; preload")

// Parameters:
// max-age=31536000     → Remember for 1 year
// includeSubDomains    → Apply to all subdomains
// preload              → Add to browser preload list

// What it does:
// 1. User types "http://example.com"
// 2. Browser automatically upgrades to "https://example.com"
// 3. User can't bypass certificate errors
```

---

## 2️⃣ Content-Security-Policy (CSP)

```
Purpose: Control what resources can be loaded - MOST IMPORTANT!
```

```go
// Strict policy for API
c.Header("Content-Security-Policy", "default-src 'none'; frame-ancestors 'none'")

// For web application
policy := strings.Join([]string{
    "default-src 'self'",                           // Default: only same origin
    "script-src 'self' 'unsafe-inline'",            // Scripts
    "style-src 'self' 'unsafe-inline'",             // Styles
    "img-src 'self' data: https:",                  // Images
    "font-src 'self'",                              // Fonts
    "connect-src 'self' https://api.example.com",   // AJAX, WebSocket
    "frame-ancestors 'none'",                       // Prevent framing
    "base-uri 'self'",                              // Restrict <base>
    "form-action 'self'",                           // Form submissions
}, "; ")

c.Header("Content-Security-Policy", policy)

// Common directives:
// 'self'         → Same origin
// 'none'         → Block all
// 'unsafe-inline' → Allow inline scripts/styles (avoid if possible)
// 'unsafe-eval'  → Allow eval() (avoid!)
// https:         → Only HTTPS sources
// data:          → Allow data: URIs
```

---

## 3️⃣ X-Content-Type-Options

```
Purpose: Prevent MIME type sniffing
```

```go
c.Header("X-Content-Type-Options", "nosniff")

// What it prevents:
// Server sends: Content-Type: text/plain
// File contains: <script>alert('xss')</script>
//
// Without nosniff: Browser might execute as JavaScript!
// With nosniff: Browser respects the declared type
```

---

## 4️⃣ X-Frame-Options

```
Purpose: Prevent clickjacking attacks
```

```go
// Options:
c.Header("X-Frame-Options", "DENY")           // Never allow framing
c.Header("X-Frame-Options", "SAMEORIGIN")     // Only same origin can frame

// Clickjacking attack:
// 1. Attacker creates page with invisible iframe
// 2. iframe loads your banking site
// 3. User thinks they're clicking attacker's page
// 4. Actually clicking "Transfer $1000" on bank site!
```

---

## 5️⃣ Referrer-Policy

```
Purpose: Control referrer information
```

```go
// Options (most to least restrictive):
c.Header("Referrer-Policy", "no-referrer")                    // Never send
c.Header("Referrer-Policy", "same-origin")                    // Same origin only
c.Header("Referrer-Policy", "strict-origin")                  // Origin only, HTTPS→HTTP blocked
c.Header("Referrer-Policy", "strict-origin-when-cross-origin") // Recommended

// Example:
// User on: https://myapp.com/secret-page?token=abc123
// Clicks link to: https://external.com
//
// no-referrer:           No referrer sent
// same-origin:           No referrer (different origin)
// strict-origin:         "https://myapp.com" (origin only)
// no-referrer-downgrade: Full URL if HTTPS→HTTPS, nothing if HTTPS→HTTP
```

---

## 6️⃣ Permissions-Policy (Feature-Policy)

```
Purpose: Disable browser features you don't need
```

```go
policy := strings.Join([]string{
    "camera=()",           // Disable camera
    "microphone=()",       // Disable microphone
    "geolocation=()",      // Disable geolocation
    "payment=()",          // Disable Payment API
    "usb=()",              // Disable USB
    "fullscreen=(self)",   // Only self can fullscreen
}, ", ")

c.Header("Permissions-Policy", policy)
```

---

## 7️⃣ Cache-Control

```
Purpose: Prevent sensitive data caching
```

```go
// For sensitive data (account info, etc.)
c.Header("Cache-Control", "no-store, no-cache, must-revalidate, private")
c.Header("Pragma", "no-cache")  // HTTP/1.0 compatibility
c.Header("Expires", "0")

// For public static assets
c.Header("Cache-Control", "public, max-age=31536000, immutable")
```

---

## 📦 Complete Implementation

```go
func SecurityHeadersMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // HTTPS
        c.Header("Strict-Transport-Security", "max-age=31536000; includeSubDomains")

        // XSS Protection
        c.Header("X-XSS-Protection", "1; mode=block")
        c.Header("X-Content-Type-Options", "nosniff")

        // Clickjacking
        c.Header("X-Frame-Options", "DENY")

        // CSP - Strict for API
        c.Header("Content-Security-Policy", "default-src 'none'; frame-ancestors 'none'")

        // Referrer
        c.Header("Referrer-Policy", "strict-origin-when-cross-origin")

        // Permissions
        c.Header("Permissions-Policy", "camera=(), microphone=(), geolocation=()")

        // Cache
        if strings.HasPrefix(c.Request.URL.Path, "/api/") {
            c.Header("Cache-Control", "no-store")
        }

        c.Next()
    }
}

func main() {
    r := gin.Default()
    r.Use(SecurityHeadersMiddleware())
    // ...
}
```

---

## 🔍 Testing Your Headers

```bash
# Check headers
curl -I https://yoursite.com

# Online tools:
# https://securityheaders.com/
# https://observatory.mozilla.org/
```

### Target Grade: A+

```
┌─────────────────────────────────────────────────────────────────────┐
│                SecurityHeaders.com Grade                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  A+    Excellent! All headers present                               │
│  A     Good, minor improvements possible                            │
│  B     Missing some important headers                               │
│  C     Missing critical headers                                     │
│  D     Basic security issues                                        │
│  F     Serious security problems                                    │
│                                                                      │
│  Required for A+:                                                    │
│  ✅ Strict-Transport-Security                                       │
│  ✅ Content-Security-Policy                                         │
│  ✅ X-Content-Type-Options                                          │
│  ✅ X-Frame-Options                                                 │
│  ✅ Referrer-Policy                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **HSTS** = Force HTTPS always
- ✅ **CSP** = أهم header - يمنع XSS
- ✅ **X-Frame-Options** = يمنع clickjacking
- ✅ **nosniff** = يمنع MIME sniffing
- ✅ **Referrer-Policy** = يحمي URLs الحساسة
- ✅ اختبر على **securityheaders.com**

</div>

---

## 🎉 Module Complete!

<div dir="rtl">

مبروك! أنت خلصت **Module 1.6: API Security** 🎉

راجعنا:
- HTTPS & TLS
- CORS
- Rate Limiting
- Input Validation
- Common Attacks
- Security Headers

**➡️ [Next Module: Caching Basics](../07-caching-basics/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Common Attacks](./05-common-attacks.md) | [📚 Module Home](../README.md) | [🏠 Track 1](../../README.md)

</div>
