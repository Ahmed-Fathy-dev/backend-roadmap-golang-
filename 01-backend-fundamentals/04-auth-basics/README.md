# Module 1.4: Authentication & Authorization 🔐

<div dir="rtl">

## نظرة عامة

الأمان هو **أهم** جزء في Backend Development!

في هذا Module، سنتعلم كيف نحمي التطبيقات ونتحقق من هوية المستخدمين وصلاحياتهم.

</div>

---

## 📚 Lessons

### Authentication Basics

1. **[Authentication vs Authorization](./lessons/01-authn-vs-authz.md)**
   <div dir="rtl">- الفرق الواضح - متى تستخدم كل واحد</div>

2. **[Session-Based Authentication](./lessons/02-session-auth.md)**
   <div dir="rtl">- كيف يعمل - Cookies - Sessions - Pros & Cons</div>

3. **[Token-Based Authentication](./lessons/03-token-auth.md)**
   <div dir="rtl">- JWT - Access & Refresh Tokens - Stateless</div>

4. **[JWT Deep Dive](./lessons/04-jwt-deep-dive.md)**
   <div dir="rtl">- JWT Structure - Claims - Signing - Security</div>

5. **[Password Security](./lessons/05-password-security.md)**
   <div dir="rtl">- Hashing (bcrypt) - Salt - Best practices</div>

6. **[OAuth 2.0 Overview](./lessons/06-oauth2.md)**
   <div dir="rtl">- OAuth flow - Social login - Scopes</div>

7. **[RBAC - Role Based Access Control](./lessons/07-rbac.md)**
   <div dir="rtl">- Roles & Permissions - Implementation</div>

8. **[Security Threats](./lessons/08-security-threats.md)**
   <div dir="rtl">- XSS, CSRF, SQL Injection - Prevention</div>

---

## 💻 Examples

1. **[JWT Authentication Implementation](./examples/01-jwt-auth.md)**
   <div dir="rtl">Login/Register كامل مع JWT</div>

2. **[Session Management](./examples/02-session-management.md)**
   <div dir="rtl">Session-based auth example</div>

3. **[Password Reset Flow](./examples/03-password-reset.md)**
   <div dir="rtl">Forgot password implementation</div>

4. **[Role-Based Authorization](./examples/04-rbac-implementation.md)**
   <div dir="rtl">Roles & permissions في Go</div>

---

## 📖 Resources

- **[Authentication Security Checklist](./resources/auth-security-checklist.md)**
  <div dir="rtl">Checklist شامل للأمان</div>

- **[Password Security Guide](./resources/password-security.md)**
  <div dir="rtl">دليل حماية Passwords</div>

- **[JWT Best Practices](./resources/jwt-best-practices.md)**
  <div dir="rtl">أفضل ممارسات JWT</div>

---

## 💡 Quick Reference

<div dir="rtl">

**Authentication (المصادقة):**  
التحقق من الهوية - "من أنت؟"

**Authorization (التفويض):**  
التحقق من الصلاحيات - "ماذا يمكنك أن تفعل؟"

**مثال:**

- Login = Authentication ✅
- Admin يحذف مستخدم = Authorization ✅

</div>

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 1.5: Database Concepts](../05-database-concepts/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: REST API](../03-rest-api/README.md) | [🏠 Track 1](../README.md) | [📚 Main](../../README.md)

</div>
