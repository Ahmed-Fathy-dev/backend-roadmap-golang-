# Lesson 6: OAuth 2.0 - المصادقة عبر طرف ثالث 🔐

<div dir="rtl">

## المقدمة

**OAuth 2.0** بيسمح للتطبيقات تصل لبيانات المستخدم من خدمات تانية (Google, Facebook, GitHub) بدون ما تعرف الـ password بتاعه!

**المدة المتوقعة:** 25 دقيقة

</div>

---

## 📊 What is OAuth 2.0?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OAuth 2.0 Overview                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  المشكلة:                                                            │
│  ─────────                                                           │
│  تطبيقك يحتاج يصل لبيانات المستخدم من Google                        │
│                                                                      │
│  ❌ الحل الخاطئ:                                                     │
│  اطلب من المستخدم يديك password الـ Google!                         │
│  (مخاطرة أمنية كبيرة)                                               │
│                                                                      │
│  ✅ الحل الصحيح (OAuth):                                             │
│  المستخدم يسجل دخول في Google مباشرة،                              │
│  Google يديك token محدود الصلاحيات                                  │
│                                                                      │
│  OAuth = "Open Authorization"                                        │
│  Allows third-party access WITHOUT sharing password                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ OAuth Roles

```
┌─────────────────────────────────────────────────────────────────────┐
│                      OAuth 2.0 Roles                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Resource Owner (المستخدم)                                        │
│     └─ الشخص اللي يملك البيانات                                     │
│     └─ مثال: أنت وحسابك في Google                                   │
│                                                                      │
│  2. Client (تطبيقك)                                                  │
│     └─ التطبيق اللي يريد الوصول للبيانات                            │
│     └─ مثال: تطبيق Todo يريد اسمك وإيميلك                           │
│                                                                      │
│  3. Authorization Server (سيرفر التصريح)                            │
│     └─ يتحقق من هوية المستخدم                                       │
│     └─ يصدر الـ tokens                                               │
│     └─ مثال: accounts.google.com                                    │
│                                                                      │
│  4. Resource Server (سيرفر البيانات)                                 │
│     └─ يحتوي البيانات الفعلية                                       │
│     └─ مثال: Gmail API, Google Drive API                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ OAuth Flow (Authorization Code)

```
┌─────────────────────────────────────────────────────────────────────┐
│              Authorization Code Flow (Most Secure)                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  User          Your App          Auth Server         Resource       │
│   │               │                   │               Server        │
│   │  1. Login     │                   │                  │          │
│   │──────────────▶│                   │                  │          │
│   │               │                   │                  │          │
│   │  2. Redirect to Auth Server       │                  │          │
│   │◀──────────────│                   │                  │          │
│   │               │                   │                  │          │
│   │  3. User logs in to Auth Server   │                  │          │
│   │──────────────────────────────────▶│                  │          │
│   │               │                   │                  │          │
│   │  4. User approves permissions     │                  │          │
│   │◀──────────────────────────────────│                  │          │
│   │               │                   │                  │          │
│   │  5. Redirect back with Code       │                  │          │
│   │──────────────▶│                   │                  │          │
│   │               │                   │                  │          │
│   │               │  6. Exchange Code │                  │          │
│   │               │     for Token     │                  │          │
│   │               │──────────────────▶│                  │          │
│   │               │                   │                  │          │
│   │               │◀──────────────────│                  │          │
│   │               │  7. Access Token  │                  │          │
│   │               │                   │                  │          │
│   │               │  8. Use Token     │                  │          │
│   │               │─────────────────────────────────────▶│          │
│   │               │                   │                  │          │
│   │               │◀─────────────────────────────────────│          │
│   │               │  9. User Data     │                  │          │
│   │               │                   │                  │          │
│   │◀──────────────│                   │                  │          │
│   │ 10. Logged in │                   │                  │          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Step by Step

```
Step 1-2: الـ User يضغط "Login with Google"
          تطبيقك يعمل redirect لـ Google

Step 3-4: الـ User يسجل دخول في Google
          يوافق على الصلاحيات المطلوبة

Step 5:   Google يرجع للـ callback URL مع authorization code

Step 6-7: تطبيقك يرسل الـ code لـ Google
          يستلم access token (و refresh token)

Step 8-9: تطبيقك يستخدم الـ token للوصول للبيانات

Step 10:  الـ User مسجل دخول في تطبيقك!
```

---

## 3️⃣ OAuth URLs

```
Authorization Request:
─────────────────────
https://accounts.google.com/o/oauth2/v2/auth
    ?client_id=YOUR_CLIENT_ID
    &redirect_uri=https://yourapp.com/callback
    &response_type=code
    &scope=email profile
    &state=random_string

Parameters:
• client_id     = تطبيقك مسجل في Google
• redirect_uri  = أين يرجع بعد الموافقة
• response_type = "code" للـ Authorization Code flow
• scope         = ما هي الصلاحيات المطلوبة
• state         = random string للحماية من CSRF


Token Exchange:
───────────────
POST https://oauth2.googleapis.com/token
Content-Type: application/x-www-form-urlencoded

client_id=YOUR_CLIENT_ID
&client_secret=YOUR_SECRET
&code=AUTHORIZATION_CODE
&grant_type=authorization_code
&redirect_uri=https://yourapp.com/callback


Token Response:
───────────────
{
  "access_token": "ya29.a0AfH6SM...",
  "expires_in": 3600,
  "refresh_token": "1//0eXxYz...",
  "scope": "email profile",
  "token_type": "Bearer"
}
```

---

## 4️⃣ Implementation Example

```go
// config.go
type OAuthConfig struct {
    ClientID     string
    ClientSecret string
    RedirectURL  string
    Scopes       []string
}

var googleOAuth = OAuthConfig{
    ClientID:     os.Getenv("GOOGLE_CLIENT_ID"),
    ClientSecret: os.Getenv("GOOGLE_CLIENT_SECRET"),
    RedirectURL:  "http://localhost:8080/auth/google/callback",
    Scopes:       []string{"email", "profile"},
}

// handlers.go

// Step 1-2: Redirect to Google
func GoogleLogin(c *gin.Context) {
    // Generate state for CSRF protection
    state := generateRandomState()
    setStateCookie(c, state)

    // Build authorization URL
    url := fmt.Sprintf(
        "https://accounts.google.com/o/oauth2/v2/auth?client_id=%s&redirect_uri=%s&response_type=code&scope=%s&state=%s",
        googleOAuth.ClientID,
        url.QueryEscape(googleOAuth.RedirectURL),
        url.QueryEscape(strings.Join(googleOAuth.Scopes, " ")),
        state,
    )

    c.Redirect(http.StatusTemporaryRedirect, url)
}

// Step 5-9: Handle callback
func GoogleCallback(c *gin.Context) {
    // Verify state (CSRF protection)
    state := c.Query("state")
    storedState := getStateCookie(c)
    if state != storedState {
        c.JSON(400, gin.H{"error": "Invalid state"})
        return
    }

    // Get authorization code
    code := c.Query("code")
    if code == "" {
        c.JSON(400, gin.H{"error": "No code provided"})
        return
    }

    // Step 6-7: Exchange code for token
    token, err := exchangeCodeForToken(code)
    if err != nil {
        c.JSON(500, gin.H{"error": "Failed to exchange token"})
        return
    }

    // Step 8-9: Get user info
    userInfo, err := getUserInfo(token.AccessToken)
    if err != nil {
        c.JSON(500, gin.H{"error": "Failed to get user info"})
        return
    }

    // Create or find user in our database
    user, err := findOrCreateUser(userInfo)
    if err != nil {
        c.JSON(500, gin.H{"error": "Failed to create user"})
        return
    }

    // Generate our own JWT
    ourToken, err := generateJWT(user)
    if err != nil {
        c.JSON(500, gin.H{"error": "Failed to generate token"})
        return
    }

    // Redirect to frontend with token
    c.Redirect(http.StatusTemporaryRedirect,
        fmt.Sprintf("http://localhost:3000/auth?token=%s", ourToken))
}

func exchangeCodeForToken(code string) (*TokenResponse, error) {
    data := url.Values{}
    data.Set("client_id", googleOAuth.ClientID)
    data.Set("client_secret", googleOAuth.ClientSecret)
    data.Set("code", code)
    data.Set("grant_type", "authorization_code")
    data.Set("redirect_uri", googleOAuth.RedirectURL)

    resp, err := http.PostForm("https://oauth2.googleapis.com/token", data)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var token TokenResponse
    json.NewDecoder(resp.Body).Decode(&token)
    return &token, nil
}

func getUserInfo(accessToken string) (*UserInfo, error) {
    req, _ := http.NewRequest("GET", "https://www.googleapis.com/oauth2/v2/userinfo", nil)
    req.Header.Set("Authorization", "Bearer "+accessToken)

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var userInfo UserInfo
    json.NewDecoder(resp.Body).Decode(&userInfo)
    return &userInfo, nil
}
```

---

## 5️⃣ OAuth Grant Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OAuth 2.0 Grant Types                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Authorization Code (الأكثر أماناً)                               │
│     └─ للـ Server-side apps                                          │
│     └─ Code → Token exchange on backend                             │
│     └─ مثال: Web apps with backend                                  │
│                                                                      │
│  2. Authorization Code + PKCE                                        │
│     └─ للـ Mobile/SPA apps                                           │
│     └─ يضيف code_verifier/code_challenge                            │
│     └─ أمان إضافي بدون client_secret                                │
│                                                                      │
│  3. Client Credentials                                               │
│     └─ للـ Machine-to-machine                                        │
│     └─ بدون user involvement                                        │
│     └─ مثال: Microservices communication                            │
│                                                                      │
│  4. Refresh Token                                                    │
│     └─ للحصول على access token جديد                                 │
│     └─ بدون تدخل المستخدم                                           │
│                                                                      │
│  ❌ Deprecated:                                                       │
│  5. Implicit (كان للـ SPAs - لم يعد آمناً)                           │
│  6. Resource Owner Password (يعطي password للتطبيق)                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6️⃣ Scopes

```
Scopes = الصلاحيات المطلوبة

Google Scopes Examples:
───────────────────────
• email                        → الإيميل فقط
• profile                      → الاسم والصورة
• openid                       → OpenID Connect
• https://www.googleapis.com/auth/calendar    → Calendar access
• https://www.googleapis.com/auth/drive       → Drive access


GitHub Scopes:
──────────────
• user:email                   → Read email
• read:user                    → Read profile
• repo                         → Full repo access
• public_repo                  → Public repos only


Best Practice:
──────────────
✅ Request minimum scopes needed
❌ Don't request everything "just in case"
```

---

## 7️⃣ Security Considerations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OAuth Security Checklist                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ MUST DO:                                                         │
│  ───────────                                                         │
│  □ Use HTTPS everywhere                                              │
│  □ Validate state parameter (CSRF protection)                       │
│  □ Store client_secret securely (env vars, secrets manager)        │
│  □ Validate redirect_uri (exact match, no wildcards)                │
│  □ Use short-lived access tokens                                    │
│  □ Store refresh tokens securely                                    │
│                                                                      │
│  ❌ DON'T DO:                                                        │
│  ────────────                                                        │
│  □ Don't expose client_secret in frontend                           │
│  □ Don't skip state validation                                       │
│  □ Don't use Implicit grant for new apps                            │
│  □ Don't request unnecessary scopes                                  │
│  □ Don't store tokens in localStorage (XSS risk)                    │
│                                                                      │
│  ⚠️ Common Attacks:                                                  │
│  ─────────────────                                                   │
│  • CSRF: State parameter prevents this                              │
│  • Token Theft: Use httpOnly cookies                                │
│  • Redirect Attacks: Strict redirect_uri validation                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8️⃣ OpenID Connect (OIDC)

```
OpenID Connect = OAuth 2.0 + Authentication

OAuth 2.0: للـ Authorization (access to resources)
OIDC:      للـ Authentication (who is the user)

OIDC Adds:
───────────
• ID Token (JWT with user info)
• /userinfo endpoint
• Standard claims (sub, email, name, picture)

Example ID Token:
────────────────
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",
  "email": "ahmed@gmail.com",
  "email_verified": true,
  "name": "Ahmed Mohamed",
  "picture": "https://...",
  "iat": 1704067200,
  "exp": 1704070800
}
```

---

## ✅ Key Takeaways

<div dir="rtl">

- ✅ **OAuth 2.0** = للـ authorization (وصول لموارد)
- ✅ **Authorization Code** = الـ flow الأكثر أماناً
- ✅ **PKCE** = للـ mobile/SPA apps
- ✅ **State parameter** = للحماية من CSRF
- ✅ **Scopes** = اطلب الحد الأدنى فقط
- ✅ **OIDC** = OAuth + identity (من هو المستخدم)

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد ما فهمت OAuth، خلينا نتعلم عن الـ Authorization - **RBAC**:

**➡️ [Lesson 7: RBAC](./07-rbac.md)**

</div>

---

<div align="center">

[⬅️ Previous: Password Security](./05-password-security.md) | [📚 Module Home](../README.md) | [➡️ Next: RBAC](./07-rbac.md)

</div>
