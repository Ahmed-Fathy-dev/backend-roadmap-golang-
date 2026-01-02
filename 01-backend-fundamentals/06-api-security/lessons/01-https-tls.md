# Lesson 1: HTTPS & TLS - التشفير 🔐

<div dir="rtl">

## المقدمة

**HTTPS** هو HTTP + **S**ecure. بيشفر كل البيانات بين الـ Client والـ Server، فلا أحد يقدر يقرأها في الوسط.

**المدة المتوقعة:** 20 دقيقة

</div>

---

## 📊 HTTP vs HTTPS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HTTP vs HTTPS                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HTTP (Port 80):                                                     │
│  ───────────────                                                     │
│  Client ──────── "password=secret123" ────────▶ Server              │
│                          ↑                                           │
│                     Anyone can read!                                │
│                     (Man-in-the-Middle)                             │
│                                                                      │
│  HTTPS (Port 443):                                                   │
│  ────────────────                                                    │
│  Client ──────── "X#$@!kL9..." (encrypted) ───▶ Server              │
│                          ↑                                           │
│                     Unreadable!                                     │
│                     (Only client & server can decrypt)              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ What HTTPS Provides

### The CIA Triad

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HTTPS Provides                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Confidentiality (السرية)                                        │
│     └─ البيانات مشفرة                                               │
│     └─ لا أحد يقدر يقرأها في الوسط                                  │
│     └─ Passwords, credit cards, personal data protected            │
│                                                                      │
│  2. Integrity (سلامة البيانات)                                      │
│     └─ البيانات لم تُعدّل                                           │
│     └─ لا أحد يقدر يغير الـ response                                │
│     └─ Prevents tampering with data                                 │
│                                                                      │
│  3. Authentication (التحقق)                                         │
│     └─ السيرفر هو فعلاً من يدعي                                     │
│     └─ Certificate verification                                      │
│     └─ Prevents impersonation                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ TLS (Transport Layer Security)

<div dir="rtl">

**TLS** = الـ protocol المسؤول عن التشفير (الـ "S" في HTTPS).

**SSL** = الإصدار القديم (deprecated) - لكن الناس لسه بتقول "SSL certificate".

</div>

```
Version History:
────────────────
SSL 1.0    → Never released (too insecure)
SSL 2.0    → 1995 (deprecated, insecure)
SSL 3.0    → 1996 (deprecated, POODLE attack)
TLS 1.0    → 1999 (deprecated)
TLS 1.1    → 2006 (deprecated)
TLS 1.2    → 2008 (still widely used ✅)
TLS 1.3    → 2018 (current, most secure ✅)
```

---

## 3️⃣ TLS Handshake

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TLS 1.3 Handshake                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Client                                        Server                │
│     │                                             │                  │
│     │  1. Client Hello                            │                  │
│     │     - TLS version                           │                  │
│     │     - Cipher suites supported               │                  │
│     │     - Random number                         │                  │
│     │ ───────────────────────────────────────────▶│                  │
│     │                                             │                  │
│     │  2. Server Hello                            │                  │
│     │     - Selected cipher suite                 │                  │
│     │     - Server certificate                    │                  │
│     │     - Server random                         │                  │
│     │ ◀───────────────────────────────────────────│                  │
│     │                                             │                  │
│     │  3. Client verifies certificate             │                  │
│     │     - Check CA signature                    │                  │
│     │     - Check domain matches                  │                  │
│     │     - Check not expired                     │                  │
│     │                                             │                  │
│     │  4. Key Exchange                            │                  │
│     │     - Generate session keys                 │                  │
│     │ ◀─────────────────────────────────────────▶ │                  │
│     │                                             │                  │
│     │  5. Encrypted Communication Begins          │                  │
│     │ ═══════════════════════════════════════════ │                  │
│                                                                      │
│  Total: 1 Round Trip (TLS 1.3) - Very fast!                        │
│  TLS 1.2: 2 Round Trips                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ Certificates

### What is a Certificate?

```
Certificate = Digital ID card for your server

Contains:
─────────
• Domain name (example.com)
• Organization name
• Public key
• Expiration date
• Issuer (Certificate Authority)
• Digital signature
```

### Certificate Authority (CA)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Certificate Trust Chain                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Root CA (Built into browsers/OS)                                   │
│     │    - DigiCert, Let's Encrypt, GlobalSign                     │
│     │    - Ultimate trust anchor                                    │
│     ▼                                                                │
│  Intermediate CA                                                     │
│     │    - Signed by Root CA                                        │
│     │    - Issues certificates to websites                          │
│     ▼                                                                │
│  Your Server Certificate                                             │
│          - Signed by Intermediate CA                                │
│          - Proves your server is legitimate                         │
│                                                                      │
│  Browser verifies the chain:                                         │
│  Your cert → signed by Intermediate → signed by Root (trusted!)    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Certificate Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Certificate Types                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Domain Validated (DV) - الأبسط                                     │
│  ─────────────────────                                               │
│  • Verifies domain ownership only                                   │
│  • Free (Let's Encrypt)                                             │
│  • Good for: Blogs, personal sites                                  │
│                                                                      │
│  Organization Validated (OV)                                         │
│  ──────────────────────────                                          │
│  • Verifies domain + organization                                   │
│  • Shows company name in cert details                               │
│  • Good for: Business websites                                      │
│                                                                      │
│  Extended Validation (EV) - الأشمل                                  │
│  ─────────────────────────                                           │
│  • Thorough verification                                            │
│  • Most expensive                                                    │
│  • Good for: Banks, e-commerce                                      │
│                                                                      │
│  Wildcard (*.example.com)                                            │
│  ────────────────────────                                            │
│  • Covers all subdomains                                            │
│  • api.example.com, www.example.com, etc.                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ Getting a Certificate

### Let's Encrypt (Free!)

```bash
# Install certbot
sudo apt install certbot

# Get certificate (standalone)
sudo certbot certonly --standalone -d example.com -d www.example.com

# Or with nginx
sudo certbot --nginx -d example.com -d www.example.com

# Auto-renewal (add to cron)
sudo certbot renew

# Certificates stored at:
# /etc/letsencrypt/live/example.com/fullchain.pem
# /etc/letsencrypt/live/example.com/privkey.pem
```

### Using in Go

```go
package main

import (
    "log"
    "net/http"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, HTTPS!"))
    })

    // Simple TLS server
    log.Fatal(http.ListenAndServeTLS(
        ":443",
        "/etc/letsencrypt/live/example.com/fullchain.pem",  // Certificate
        "/etc/letsencrypt/live/example.com/privkey.pem",    // Private Key
        mux,
    ))
}
```

### Production Configuration

```go
package main

import (
    "crypto/tls"
    "log"
    "net/http"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", handler)

    // Secure TLS configuration
    tlsConfig := &tls.Config{
        MinVersion: tls.VersionTLS12,  // Minimum TLS 1.2
        CurvePreferences: []tls.CurveID{
            tls.CurveP256,
            tls.X25519,
        },
        PreferServerCipherSuites: true,
        CipherSuites: []uint16{
            tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,
            tls.TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,
            tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
            tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
        },
    }

    server := &http.Server{
        Addr:      ":443",
        Handler:   mux,
        TLSConfig: tlsConfig,
    }

    // Redirect HTTP to HTTPS
    go func() {
        http.ListenAndServe(":80", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            http.Redirect(w, r, "https://"+r.Host+r.URL.String(), http.StatusMovedPermanently)
        }))
    }()

    log.Fatal(server.ListenAndServeTLS(
        "/etc/letsencrypt/live/example.com/fullchain.pem",
        "/etc/letsencrypt/live/example.com/privkey.pem",
    ))
}
```

---

## 6️⃣ Common Mistakes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HTTPS Common Mistakes                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ❌ Mixed Content                                                    │
│     └─ HTTPS page loading HTTP resources                           │
│     └─ Solution: Use relative URLs or HTTPS everywhere             │
│                                                                      │
│  ❌ Certificate Expiry                                               │
│     └─ Forgetting to renew                                          │
│     └─ Solution: Auto-renewal with certbot                          │
│                                                                      │
│  ❌ Self-Signed Certificates in Production                          │
│     └─ Browsers show scary warning                                  │
│     └─ Solution: Use Let's Encrypt (free)                          │
│                                                                      │
│  ❌ Not Redirecting HTTP to HTTPS                                   │
│     └─ Users can still access insecure version                     │
│     └─ Solution: 301 redirect + HSTS                                │
│                                                                      │
│  ❌ Old TLS Versions                                                 │
│     └─ TLS 1.0/1.1 are deprecated                                  │
│     └─ Solution: Minimum TLS 1.2                                    │
│                                                                      │
│  ❌ Missing HSTS                                                     │
│     └─ Browser doesn't enforce HTTPS                               │
│     └─ Solution: Add HSTS header                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Testing Your HTTPS

```bash
# Check certificate details
openssl s_client -connect example.com:443 -servername example.com

# Check TLS version support
nmap --script ssl-enum-ciphers -p 443 example.com

# Online tools:
# - https://www.ssllabs.com/ssltest/
# - https://securityheaders.com/
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **HTTPS** = HTTP مشفر بـ TLS
- ✅ **TLS 1.2/1.3** = الإصدارات الآمنة
- ✅ **Certificate** = هوية السيرفر
- ✅ **Let's Encrypt** = certificates مجانية
- ✅ **Always redirect** HTTP to HTTPS
- ✅ **Use HSTS** لإجبار المتصفح على HTTPS

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

**➡️ [Lesson 2: CORS](./02-cors.md)**

</div>

---

<div align="center">

[📚 Module Home](../README.md) | [➡️ Next: CORS](./02-cors.md)

</div>
