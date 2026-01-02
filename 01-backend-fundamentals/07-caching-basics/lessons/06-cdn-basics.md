# Lesson 6: CDN Basics - أساسيات شبكات توزيع المحتوى 🌍

<div dir="rtl">

## المقدمة

**CDN** (Content Delivery Network) هي شبكة من السيرفرات موزعة حول العالم لتوصيل المحتوى للمستخدمين بسرعة.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 What is a CDN?

```
┌─────────────────────────────────────────────────────────────────────┐
│                        What is CDN?                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Without CDN:                                                       │
│  ─────────────                                                       │
│                         ┌─────────────┐                              │
│                         │   Origin    │  (New York)                 │
│                         │   Server    │                              │
│                         └─────────────┘                              │
│                               ▲                                      │
│           ┌───────────────────┼───────────────────┐                  │
│           │                   │                   │                  │
│      200ms│              100ms│              300ms│                  │
│           │                   │                   │                  │
│      🇪🇬 Egypt          🇺🇸 USA            🇯🇵 Japan               │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  With CDN:                                                          │
│  ──────────                                                          │
│       🌍 Edge Servers Around the World                              │
│                                                                      │
│      ┌────────┐         ┌────────┐         ┌────────┐               │
│      │  Edge  │         │  Edge  │         │  Edge  │               │
│      │ Cairo  │         │ NYC    │         │ Tokyo  │               │
│      └────────┘         └────────┘         └────────┘               │
│           ▲                 ▲                   ▲                    │
│       10ms│              5ms│               10ms│                    │
│           │                 │                   │                    │
│      🇪🇬 Egypt          🇺🇸 USA            🇯🇵 Japan               │
│                                                                      │
│  Content served from nearest edge server!                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ How CDN Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CDN Request Flow                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  First Request (Cache Miss):                                        │
│  ─────────────────────────────                                       │
│                                                                      │
│  User (Egypt) → Edge (Cairo) → Origin (USA)                         │
│       │             │               │                               │
│       │  1. Request │   2. Forward  │                               │
│       │─────────────▶│──────────────▶│                               │
│       │             │               │                               │
│       │             │  3. Response  │                               │
│       │             │◀──────────────│                               │
│       │             │               │                               │
│       │  4. Cache & │               │                               │
│       │     Return  │               │                               │
│       │◀────────────│               │                               │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  Subsequent Requests (Cache Hit):                                   │
│  ─────────────────────────────────                                   │
│                                                                      │
│  User (Egypt) → Edge (Cairo)                                        │
│       │             │                                               │
│       │  1. Request │                                               │
│       │─────────────▶│                                               │
│       │             │ 2. Serve from cache                           │
│       │◀────────────│    (No origin hit!)                           │
│       │             │                                               │
│                                                                      │
│  ⚡ Much faster! Origin server load reduced!                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ What CDN Caches

```
┌─────────────────────────────────────────────────────────────────────┐
│                     What CDN Can Cache                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ Perfect for CDN:                                                │
│  ────────────────────                                                │
│  • Static files (images, videos, PDFs)                              │
│  • CSS and JavaScript files                                         │
│  • Fonts                                                            │
│  • HTML pages (static sites)                                        │
│  • API responses (with Cache-Control)                               │
│                                                                      │
│  ⚠️ Careful with CDN:                                               │
│  ──────────────────────                                              │
│  • Dynamic content (personalized pages)                             │
│  • API responses with authentication                                │
│  • Real-time data                                                   │
│                                                                      │
│  ❌ Don't cache on CDN:                                             │
│  ───────────────────────                                             │
│  • User-specific data                                               │
│  • Sensitive/private content                                        │
│  • Frequently changing data                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ CDN Cache Headers

<div dir="rtl">

الـ Cache-Control headers بتتحكم في سلوك الـ CDN.

</div>

```go
// Static assets (long cache)
func serveStaticFile(c *gin.Context) {
    c.Header("Cache-Control", "public, max-age=31536000, immutable")
    // public      = CDN can cache
    // max-age     = 1 year in seconds
    // immutable   = never changes (use versioned URLs)

    c.File("static/app.123abc.js")
}

// API response (short cache)
func getProducts(c *gin.Context) {
    c.Header("Cache-Control", "public, max-age=60, s-maxage=300")
    // max-age   = browser cache for 1 minute
    // s-maxage  = CDN cache for 5 minutes (overrides max-age for CDN)

    c.JSON(200, products)
}

// Private content (no CDN cache)
func getUserProfile(c *gin.Context) {
    c.Header("Cache-Control", "private, no-store")
    // private   = only browser can cache
    // no-store  = don't cache at all

    c.JSON(200, user)
}

// With revalidation
func getArticle(c *gin.Context) {
    c.Header("Cache-Control", "public, max-age=0, must-revalidate")
    c.Header("ETag", `"abc123"`)
    // Browser/CDN must check with origin if content changed
}
```

### Cache-Control Directives

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Cache-Control Directives                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Directive        │ Meaning                                         │
│  ─────────────────┼───────────────────────────────────────────────  │
│  public           │ Anyone can cache (browser, CDN, proxies)        │
│  private          │ Only browser can cache (not CDN)                │
│  no-cache         │ Must revalidate before using cache              │
│  no-store         │ Don't cache at all                              │
│  max-age=N        │ Cache for N seconds (browser & CDN)             │
│  s-maxage=N       │ Cache for N seconds (CDN only)                  │
│  immutable        │ Never changes, don't revalidate                 │
│  must-revalidate  │ Must check if stale before using                │
│  stale-while-     │ Serve stale while fetching fresh                │
│    revalidate=N   │                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ CDN Invalidation

<div dir="rtl">

لما تحتاج تحذف محتوى من الـ CDN قبل انتهاء صلاحيته.

</div>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CDN Cache Invalidation                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Method 1: Purge/Invalidate                                         │
│  ──────────────────────────────                                      │
│  • Tell CDN to delete cached content                                │
│  • Usually via API or dashboard                                     │
│  • Can be slow (propagate to all edges)                             │
│                                                                      │
│  Method 2: Versioned URLs (Recommended!)                            │
│  ─────────────────────────────────────────                           │
│  • Instead of: /static/app.js                                       │
│  • Use:        /static/app.abc123.js                                │
│  • New version = new URL = no cache conflict!                       │
│                                                                      │
│  Method 3: Query String                                             │
│  ──────────────────────────                                          │
│  • /static/app.js?v=123                                             │
│  • Easy but not all CDNs cache query strings                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Cloudflare API Example

```go
func purgeCloudflareCache(zoneID string, urls []string) error {
    apiToken := os.Getenv("CLOUDFLARE_API_TOKEN")

    payload := map[string]interface{}{
        "files": urls,
    }
    body, _ := json.Marshal(payload)

    req, _ := http.NewRequest(
        "POST",
        fmt.Sprintf("https://api.cloudflare.com/client/v4/zones/%s/purge_cache", zoneID),
        bytes.NewReader(body),
    )
    req.Header.Set("Authorization", "Bearer "+apiToken)
    req.Header.Set("Content-Type", "application/json")

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    if resp.StatusCode != 200 {
        return fmt.Errorf("purge failed: %d", resp.StatusCode)
    }

    return nil
}

// Usage
purgeCloudflareCache("zone123", []string{
    "https://example.com/api/products",
    "https://example.com/images/logo.png",
})
```

### Versioned Assets Pattern

```go
// Build time: Generate versioned filenames
// app.js → app.abc123.js (hash of content)

import (
    "crypto/md5"
    "fmt"
    "io"
    "os"
    "path/filepath"
)

func hashFile(path string) (string, error) {
    f, err := os.Open(path)
    if err != nil {
        return "", err
    }
    defer f.Close()

    h := md5.New()
    io.Copy(h, f)

    return fmt.Sprintf("%x", h.Sum(nil))[:8], nil
}

func versionedPath(originalPath string) (string, error) {
    hash, err := hashFile(originalPath)
    if err != nil {
        return "", err
    }

    ext := filepath.Ext(originalPath)
    base := originalPath[:len(originalPath)-len(ext)]

    return fmt.Sprintf("%s.%s%s", base, hash, ext), nil
}

// app.js → app.abc12345.js
```

---

## 5️⃣ Popular CDN Providers

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Popular CDN Providers                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Provider          │ Strengths              │ Best For              │
│  ──────────────────┼────────────────────────┼───────────────────────│
│  Cloudflare        │ Free tier, DDoS,       │ Websites, APIs        │
│                    │ Security features      │                        │
│                    │                        │                        │
│  AWS CloudFront    │ AWS integration,       │ AWS-hosted apps       │
│                    │ Lambda@Edge            │                        │
│                    │                        │                        │
│  Google Cloud CDN  │ GCP integration,       │ GCP-hosted apps       │
│                    │ Global load balancing  │                        │
│                    │                        │                        │
│  Fastly            │ Real-time purge,       │ High-traffic APIs     │
│                    │ Edge computing         │                        │
│                    │                        │                        │
│  Akamai            │ Enterprise, largest    │ Enterprise apps       │
│                    │ network                │                        │
│                    │                        │                        │
│  Bunny CDN         │ Cheap, simple          │ Small-medium sites    │
│                    │                        │                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6️⃣ CDN for APIs

<div dir="rtl">

CDN مش بس للـ static files - يمكن استخدامه للـ APIs!

</div>

```go
// API endpoint with CDN caching
func getPublicProducts(c *gin.Context) {
    // Cache on CDN for 5 minutes
    c.Header("Cache-Control", "public, s-maxage=300")

    // Add Vary header for proper caching
    c.Header("Vary", "Accept-Encoding, Accept-Language")

    products := getProductsFromDB()
    c.JSON(200, products)
}

// API with conditional caching
func getProduct(c *gin.Context) {
    productID := c.Param("id")
    product := getProductFromDB(productID)

    // Generate ETag from content
    etag := generateETag(product)
    c.Header("ETag", etag)
    c.Header("Cache-Control", "public, max-age=0, must-revalidate")

    // Check If-None-Match
    if c.GetHeader("If-None-Match") == etag {
        c.Status(304) // Not Modified
        return
    }

    c.JSON(200, product)
}

func generateETag(data interface{}) string {
    bytes, _ := json.Marshal(data)
    hash := md5.Sum(bytes)
    return fmt.Sprintf(`"%x"`, hash)
}
```

### Vary Header

```go
// Vary tells CDN what makes requests "different"
func getContent(c *gin.Context) {
    lang := c.GetHeader("Accept-Language")

    // Cache different versions for different languages
    c.Header("Vary", "Accept-Language")
    c.Header("Cache-Control", "public, s-maxage=3600")

    if strings.HasPrefix(lang, "ar") {
        c.JSON(200, arabicContent)
    } else {
        c.JSON(200, englishContent)
    }
}

// CDN will cache:
// - /content (Accept-Language: en) → English version
// - /content (Accept-Language: ar) → Arabic version
```

---

## 7️⃣ CDN Configuration Example (Cloudflare)

```yaml
# Example: cloudflare page rules (conceptual)

# Static assets - long cache
Rule 1:
  URL: example.com/static/*
  Cache Level: Standard
  Edge Cache TTL: 1 month
  Browser Cache TTL: 1 year

# API - short cache
Rule 2:
  URL: example.com/api/public/*
  Cache Level: Standard
  Edge Cache TTL: 5 minutes
  Browser Cache TTL: 1 minute

# Private API - no cache
Rule 3:
  URL: example.com/api/user/*
  Cache Level: Bypass
  Security: High

# HTML pages - cache with revalidation
Rule 4:
  URL: example.com/*.html
  Cache Level: Standard
  Edge Cache TTL: 1 hour
  Always Online: On
```

---

## 8️⃣ CDN Benefits Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CDN Benefits                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Performance 🚀                                                  │
│     • Lower latency (content closer to users)                       │
│     • Faster page loads                                             │
│     • Better Core Web Vitals                                        │
│                                                                      │
│  2. Scalability 📈                                                  │
│     • Handle traffic spikes                                         │
│     • Reduce origin server load                                     │
│     • Global distribution                                           │
│                                                                      │
│  3. Security 🔒                                                     │
│     • DDoS protection                                               │
│     • WAF (Web Application Firewall)                                │
│     • SSL/TLS termination                                           │
│                                                                      │
│  4. Cost Savings 💰                                                 │
│     • Less bandwidth on origin                                      │
│     • Smaller origin servers needed                                 │
│     • Often cheaper than scaling origin                             │
│                                                                      │
│  5. Reliability ✅                                                  │
│     • Redundancy across edges                                       │
│     • Failover capabilities                                         │
│     • Always-on caching                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **CDN** = شبكة سيرفرات موزعة لتوصيل المحتوى بسرعة
- ✅ **Edge Servers** = أقرب للمستخدم = أسرع
- ✅ **Cache-Control** = للتحكم في سلوك الـ CDN
- ✅ **Versioned URLs** = أفضل طريقة للـ cache invalidation
- ✅ **s-maxage** = خاص بالـ CDN فقط
- ✅ CDN للـ **static + APIs** العامة

</div>

---

## 🎉 Module Complete!

<div dir="rtl">

مبروك! أنت خلصت **Module 1.7: Caching Basics** 🎉

راجعنا:
- ما هو Caching ولماذا مهم
- Caching Strategies (Cache-Aside, Write-Through, etc.)
- Redis Fundamentals
- Redis في Go
- Cache Invalidation
- CDN Basics

**➡️ [Next Module: Docker Basics](../../08-docker-basics/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Cache Invalidation](./05-cache-invalidation.md) | [📚 Module Home](../README.md) | [🏠 Track 1](../../README.md)

</div>
