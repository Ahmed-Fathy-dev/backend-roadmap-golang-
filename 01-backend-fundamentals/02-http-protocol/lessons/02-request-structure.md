# Lesson 2: HTTP Request Structure 📤

<div dir="rtl">

## المقدمة

في هذا الدرس، سنفتح "صندوق" HTTP Request ونفهم كل جزء فيه بالتفصيل الممل! 😄

</div>

---

## 📦 HTTP Request Components

<div dir="rtl">

كل HTTP Request يتكون من **3 أجزاء رئيسية**:

</div>

```http
POST /api/users HTTP/1.1                    ← 1️⃣ Request Line
Host: api.example.com                        ← 2️⃣ Headers
Content-Type: application/json
Authorization: Bearer token123
                                             ← Blank Line (مهم!)
{                                            ← 3️⃣ Body (optional)
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

<div dir="rtl">

دعنا نفهم كل جزء على حدة...

</div>

---

##1️⃣ Request Line (سطر الطلب)

<div dir="rtl">

السطر الأول في كل Request، يحتوي على **3 أجزاء**:

</div>

```http
POST /api/users HTTP/1.1
 │      │         │
 │      │         └─ HTTP Version
 │      └─────────── Request Path/URI
 └────────────────── HTTP Method
```

### Part 1: HTTP Method

<div dir="rtl">

**Method** يحدد **نوع العملية** المطلوبة:

</div>

| Method    | الهدف                                       |
| --------- | ------------------------------------------- |
| `GET`     | <div dir="rtl">قراءة/جلب بيانات</div>       |
| `POST`    | <div dir="rtl">إنشاء بيانات جديدة</div>     |
| `PUT`     | <div dir="rtl">تحديث بيانات كاملة</div>     |
| `PATCH`   | <div dir="rtl">تحديث جزئي</div>             |
| `DELETE`  | <div dir="rtl">حذف بيانات</div>             |
| `HEAD`    | <div dir="rtl">مثل GET لكن بدون Body</div>  |
| `OPTIONS` | <div dir="rtl">معرفة Methods المدعومة</div> |

<div dir="rtl">

**مثال:**

</div>

```http
GET /products          ← "أعطني قائمة المنتجات"
POST /products         ← "أنشئ منتج جديد"
DELETE /products/5     ← "احذف المنتج رقم 5"
```

---

### Part 2: Request Path (URI)

<div dir="rtl">

**Path** هو "العنوان" الذي تريد الوصول إليه على السيرفر:

</div>

```
الشكل الكامل:
https://api.example.com:443/products?category=laptops#reviews
 │      │               │    │                │         │
 │      │               │    │                │         └─ Fragment (نادر في HTTP)
 │      │               │    │                └─────────── Query String
 │      │               │    └──────────────────────────── Path
 │      │               └───────────────────────────────── Port (optional)
 │      └───────────────────────────────────────────────── Domain/Host
 └──────────────────────────────────────────────────────── Scheme (http/https)

في Request Line يُكتب فقط:
/products?category=laptops
```

<div dir="rtl">

#### Query String Parameters

تستخدم لإرسال معلومات إضافية مع GET Request:

</div>

```
/products?category=laptops&price_min=500&price_max=1000
          │               │                 │
          └───────────────┴─────────────────┴─ Parameters
```

<div dir="rtl">

**القواعد:**

- يبدأ بـ `?`
- Parameters مفصولة بـ `&`
- `key=value` format
- يُستخدم مع GET غالباً

**مثال عملي:**

</div>

```
البحث في YouTube:
https://youtube.com/results?search_query=golang+tutorial&filter=today

يُترجم إلى:
GET /results?search_query=golang+tutorial&filter=today HTTP/1.1
Host: youtube.com
```

---

### Part 3: HTTP Version

<div dir="rtl">

يحدد إصدار HTTP المستخدم:

</div>

```http
HTTP/1.1     ← الأكثر شيوعاً
HTTP/2       ← الأحدث والأسرع
HTTP/3       ← الأحدث (يستخدم QUIC)
```

<div dir="rtl">

**في 99% من الحالات:** `HTTP/1.1`

</div>

---

## 2️⃣ Headers (الرؤوس)

<div dir="rtl">

**Headers** هي معلومات إضافية عن Request.

### الشكل العام:

</div>

```http
Header-Name: Header-Value
```

<div dir="rtl">

**ملاحظات:**

- ✅ Header Name بالـ **Title-Case** (كل كلمة تبدأ بحرف كبير)
- ✅ بعد الاسم نضع `:`
- ✅ كل Header في سطر منفصل
- ✅ Headers متعددة مسموحة

---

### أهم Request Headers:

#### 🏠 `Host` Header (إجباري in HTTP/1.1)

يحدد اسم Domain:

</div>

```http
Host: api.example.com
```

<div dir="rtl">

**لماذا ضروري؟**
لأن سيرفر واحد قد يستضيف عدة مواقع (Virtual Hosting):

</div>

```
Same IP: 192.168.1.1
├─ Host: site1.com     → Website 1
├─ Host: site2.com     → Website 2
└─ Host: api.site1.com → API
```

---

#### 📱 `User-Agent` Header

<div dir="rtl">

يخبر السيرفر **من** يرسل الطلب:

</div>

```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0
```

<div dir="rtl">

**معلومات تحتويها:**

- Browser (Chrome, Firefox, Safari)
- Operating System (Windows, Mac, Linux, Android, iOS)
- Device Type (Desktop, Mobile, Tablet)

**الفائدة:**

- Server يقدر يعرف Device ويرسل response مناسب
- مثلاً: Mobile user → Mobile-optimized page

</div>

---

#### 📝 `Content-Type` Header

<div dir="rtl">

يحدد **نوع البيانات** المرسلة في Body:

</div>

```http
Content-Type: application/json        ← JSON data
Content-Type: application/xml         ← XML data
Content-Type: text/html               ← HTML
Content-Type: multipart/form-data     ← File upload
Content-Type: application/x-www-form-urlencoded ← Form data
```

<div dir="rtl">

**مثال:**

</div>

```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json

{"name": "Ahmed", "age": 25}
```

---

#### 📏 `Content-Length` Header

<div dir="rtl">

يحدد حجم Body **بالـ bytes**:

</div>

```http
Content-Length: 45
```

<div dir="rtl">

**مثال:**

</div>

```http
POST /api/users HTTP/1.1
Content-Type: application/json
Content-Length: 45

{"name": "Ahmed", "email": "ahmed@test.com"}
 └────────── هذا يساوي 45 byte ─────────────┘
```

---

#### ✅ `Accept` Header

<div dir="rtl">

يخبر السيرفر **بأي Format** نريد الرد:

</div>

```http
Accept: application/json              ← أريد JSON
Accept: text/html                     ← أريد HTML
Accept: application/xml               ← أريد XML
Accept: */*                           ← أي format
```

<div dir="rtl">

**مثال:**

</div>

```http
GET /api/users/5 HTTP/1.1
Accept: application/json              ← رد علي بـ JSON please!
```

---

#### 🔐 `Authorization` Header

<div dir="rtl">

يحمل بيانات المصادقة (Authentication):

</div>

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

<div dir="rtl">

**أنواع:**

- **Bearer Token:** JWT Token (الأكثر شيوعاً)
- **Basic:** Username:Password مشفر بـ Base64

**(سنتعلم هذا بتفصيل في Module Authentication)**

</div>

---

#### 🍪 `Cookie` Header

<div dir="rtl">

يرسل Cookies محفوظة من requests سابقة:

</div>

```http
Cookie: session_id=abc123; user_pref=dark_mode
```

---

#### 🌐 `Origin` Header

<div dir="rtl">

يحدد من أين جاء Request (مهم لـ CORS):

</div>

```http
Origin: https://example.com
```

---

#### 🔄 `Referer` Header

<div dir="rtl">

URL الصفحة التي جاء منها User:

</div>

```http
Referer: https://google.com/search?q=golang
```

<div dir="rtl">

**الفائدة:**

- تتبع من أين يأتي الـ traffic
- Analytics

</div>

---

### 📋 مثال Headers كاملة:

```http
GET /api/products?category=laptops HTTP/1.1
Host: shop.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0
Accept: application/json
Accept-Language: ar-EG,ar;q=0.9,en;q=0.8
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: session_id=xyz789
Authorization: Bearer eyJhbGc...
Referer: https://google.com/
Cache-Control: no-cache
```

---

## 3️⃣ Body (الجسم)

<div dir="rtl">

**Body** يحتوي البيانات المُرسَلة.

### متى نستخدم Body؟

</div>

| Method   | Body?                       |
| -------- | --------------------------- |
| `GET`    | ❌ لا (نستخدم Query String) |
| `POST`   | ✅ نعم                      |
| `PUT`    | ✅ نعم                      |
| `PATCH`  | ✅ نعم                      |
| `DELETE` | ⚠️ أحياناً (نادر)           |

---

### Body Formats:

#### 1. JSON (الأكثر شيوعاً)

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Ahmed Ali",
  "email": "ahmed@example.com",
  "age": 25,
  "city": "Cairo"
}
```

---

#### 2. Form Data (URL Encoded)

```http
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

email=ahmed%40example.com&password=mypass123&remember=true
```

<div dir="rtl">

يُستخدم مع HTML Forms التقليدية.

</div>

---

#### 3. Multipart Form Data (File Upload)

```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="name"

Ahmed
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

[binary file data here]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

<div dir="rtl">

يُستخدم لرفع الملفات.

</div>

---

#### 4. XML

```http
POST /api/users HTTP/1.1
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<user>
  <name>Ahmed</name>
  <email>ahmed@example.com</email>
</user>
```

<div dir="rtl">

أقل شيوعاً، لكن مازال يُستخدم في بعض APIs القديمة.

</div>

---

## 📝 Complete Request Examples

### Example 1: Simple GET

```http
GET /api/products HTTP/1.1
Host: shop.com
```

<div dir="rtl">

**بسيط جداً:** Method + Path + Host فقط!

</div>

---

### Example 2: GET with Query Params

```http
GET /search?q=golang&page=2&limit=20 HTTP/1.1
Host: youtube.com
User-Agent: Mozilla/5.0
Accept: text/html
```

---

### Example 3: POST with JSON

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 85
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "name": "Ahmed Ali",
  "email": "ahmed@example.com",
  "password": "securepass123",
  "role": "user"
}
```

---

### Example 4: File Upload

```http
POST /api/upload HTTP/1.1
Host: cdn.example.com
Content-Type: multipart/form-data; boundary=----Boundary123
Authorization: Bearer token123

------Boundary123
Content-Disposition: form-data; name="title"

My Profile Photo
------Boundary123
Content-Disposition: form-data; name="image"; filename="profile.jpg"
Content-Type: image/jpeg

[...binary image data...]
------Boundary123--
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **Request Line:** Method + Path + HTTP Version
- ✅ **Headers:** معلومات إضافية (Host, User-Agent, Content-Type, etc.)
- ✅ **Blank Line:** يفصل Headers عن Body (ضروري!)
- ✅ **Body:** البيانات المرسلة (JSON, Form Data, Files)
- ✅ GET لا يحتاج Body، POST/PUT/PATCH يحتاجون
- ✅ Content-Type header يحدد نوع البيانات في Body

</div>

---

## 🎯 اختبر فهمك!

<div dir="rtl">

1. ما هي الأجزاء الثلاثة الرئيسية لـ HTTP Request؟
2. ما الفرق بين Path و Query String؟
3. متى نستخدم Body في Request؟
4. ما فائدة Content-Type header؟
5. لماذا Host header ضروري؟

</div>

---

## ⏭️ Next Lesson

<div dir="rtl">

الآن بعد أن فهمت Request، دعنا نتعلم **HTTP Methods** بالتفصيل:

**➡️ [Lesson 3: HTTP Methods](./03-http-methods.md)**

</div>

---

<div align="center">

[⬅️ Previous: What is HTTP](./01-what-is-http.md) | [📚 Module Home](../README.md)

</div>
