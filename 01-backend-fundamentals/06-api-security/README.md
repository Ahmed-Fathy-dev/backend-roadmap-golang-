# Module 1.6: API Security 🔒

<div dir="rtl">

## نظرة عامة

أي API على الإنترنت هو **هدف للهجمات**!

في هذا Module، هنتعلم كيف نحمي الـ APIs بتاعتنا من التهديدات الشائعة.

**المدة المتوقعة:** 1-2 يوم

</div>

---

## 📚 Lessons

### Security Fundamentals

1. **[HTTPS & TLS](./lessons/01-https-tls.md)**
   <div dir="rtl">- ما هو HTTPS - Certificates - TLS Handshake</div>

2. **[CORS](./lessons/02-cors.md)**
   <div dir="rtl">- Cross-Origin Resource Sharing - Headers - Configuration</div>

3. **[Rate Limiting](./lessons/03-rate-limiting.md)**
   <div dir="rtl">- حماية من الـ abuse - Algorithms - Implementation</div>

4. **[Input Validation](./lessons/04-input-validation.md)**
   <div dir="rtl">- Sanitization - Validation Rules - Common Mistakes</div>

5. **[Common Attacks](./lessons/05-common-attacks.md)**
   <div dir="rtl">- SQL Injection - XSS - CSRF - وكيفية الحماية</div>

6. **[Security Headers](./lessons/06-security-headers.md)**
   <div dir="rtl">- CSP - X-Frame-Options - HSTS - وغيرهم</div>

---

## 🎯 Learning Objectives

<div dir="rtl">

بنهاية هذا Module، ستكون قادراً على:

</div>

- ✅ فهم HTTPS وكيف يحمي البيانات
- ✅ تكوين CORS بشكل صحيح
- ✅ تطبيق Rate Limiting
- ✅ Validate و sanitize الـ input
- ✅ الحماية من الهجمات الشائعة
- ✅ إضافة Security Headers

---

## 🔐 Why API Security?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    API Security Importance                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Real Breaches:                                                      │
│  ──────────────                                                      │
│  • 2021: Facebook - 533M users data leaked                          │
│  • 2019: Capital One - 100M+ customers exposed                      │
│  • 2017: Equifax - 147M people affected                             │
│                                                                      │
│  Common Causes:                                                      │
│  ──────────────                                                      │
│  • Missing authentication                                            │
│  • Broken access control                                             │
│  • Excessive data exposure                                           │
│  • Lack of rate limiting                                             │
│  • Security misconfiguration                                         │
│                                                                      │
│  Cost:                                                               │
│  ─────                                                               │
│  • Average data breach: $4.24 million (2021)                        │
│  • Reputation damage: Priceless                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📖 OWASP API Security Top 10

```
┌─────────────────────────────────────────────────────────────────────┐
│                 OWASP API Security Top 10 (2023)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Broken Object Level Authorization                                │
│     └─ User A can access User B's data                              │
│                                                                      │
│  2. Broken Authentication                                            │
│     └─ Weak credentials, missing MFA                                │
│                                                                      │
│  3. Broken Object Property Level Authorization                       │
│     └─ Exposing sensitive properties                                │
│                                                                      │
│  4. Unrestricted Resource Consumption                                │
│     └─ No rate limits, DoS possible                                 │
│                                                                      │
│  5. Broken Function Level Authorization                              │
│     └─ Regular user can access admin functions                      │
│                                                                      │
│  6. Unrestricted Access to Sensitive Business Flows                  │
│     └─ Automation abuse (ticket scalping, etc.)                     │
│                                                                      │
│  7. Server Side Request Forgery (SSRF)                               │
│     └─ Tricking server to make internal requests                    │
│                                                                      │
│  8. Security Misconfiguration                                        │
│     └─ Default configs, verbose errors, missing headers            │
│                                                                      │
│  9. Improper Inventory Management                                    │
│     └─ Exposed debug endpoints, old API versions                   │
│                                                                      │
│  10. Unsafe Consumption of APIs                                      │
│      └─ Trusting third-party APIs blindly                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💡 Quick Security Checklist

<div dir="rtl">

استخدم هذه القائمة لكل API تبنيها:

</div>

```
□ HTTPS enabled (no HTTP)
□ Authentication required
□ Authorization checked (user can only access their data)
□ Rate limiting implemented
□ Input validation on all endpoints
□ CORS configured properly
□ Security headers added
□ Sensitive data not exposed
□ Errors don't leak information
□ Logging for security events
```

---

## ⏭️ Start Learning

<div dir="rtl">

جاهز للبدء؟

**➡️ [Lesson 1: HTTPS & TLS](./lessons/01-https-tls.md)**

</div>

---

<div align="center">

[⬅️ Previous: Database Concepts](../05-database-concepts/README.md) | [🏠 Track 1](../README.md) | [➡️ Next: Caching](../07-caching-basics/README.md)

</div>
