# Example 1: GET Request 📖

<div dir="rtl">

## نظرة عامة

هذا مثال عملي كامل على **GET Request** - الطريقة الأكثر استخداماً لقراءة البيانات.

</div>

---

## 🎯 السيناريو

<div dir="rtl">

**المطلوب:** جلب قائمة المنتجات من متجر إلكتروني

**URL:** `https://shop.example.com/api/products`

</div>

---

## 📤 The Request

```http
GET /api/products HTTP/1.1
Host: shop.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0
Accept: application/json
Accept-Language: ar-EG,ar;q=0.9
Connection: keep-alive
```

---

## 🔍 تحليل Request

<div dir="rtl">

### Request Line

</div>

```http
GET /api/products HTTP/1.1
 │      │            │
 │      │            └─ HTTP Version 1.1
 │      └────────────── Path to products API
 └───────────────────── GET method (reading data)
```

<div dir="rtl">

### Headers

**`Host: shop.example.com`**

- يحدد اسم Domain
- ضروري في HTTP/1.1

**`User-Agent: Mozilla/5.0...`**

- معلومات عن Browser و Operating System
- Server يعرف هل Desktop ولا Mobile

**`Accept: application/json`**

- نطلب الرد يكون JSON format
- لو Server ما عندوش JSON، يرد بـ 406 Not Acceptable

**`Accept-Language: ar-EG,ar;q=0.9`**

- نفضل المحتوى بالعربي المصري
- لو مش متوفر، عربي عادي
- `q=0.9` تعني الأولوية (0-1)

**`Connection: keep-alive`**

- نبقي الاتصال مفتوح للـ Requests القادمة
- أسرع من فتح اتصال جديد كل مرة

</div>

---

## 📥 The Response

```http
HTTP/1.1 200 OK
Date: Tue, 10 Dec 2024 12:30:00 GMT
Server: nginx/1.18.0
Content-Type: application/json; charset=UTF-8
Content-Length: 342
Cache-Control: max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"

{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "price": 25000,
      "category": "laptops",
      "stock": 15,
      "image": "https://cdn.shop.com/laptop-dell.jpg"
    },
    {
      "id": 2,
      "name": "iPhone 15 Pro",
      "price": 30000,
      "category": "phones",
      "stock": 8,
      "image": "https://cdn.shop.com/iphone15.jpg"
    },
    {
      "id": 3,
      "name": "Sony Headphones WH-1000XM5",
      "price": 5000,
      "category": "audio",
      "stock": 25,
      "image": "https://cdn.shop.com/sony-headphones.jpg"
    }
  ],
  "total": 3,
  "page": 1
}
```

---

## 🔍 تحليل Response

<div dir="rtl">

### Status Line

</div>

```http
HTTP/1.1 200 OK
   │      │   │
   │      │   └─ Status Message (OK)
   │      └───── Status Code (200 = Success)
   └──────────── HTTP Version
```

<div dir="rtl">

### Response Headers

**`Date:`** التاريخ والوقت من السيرفر

**`Server: nginx/1.18.0`** نوع السيرفر المستخدم

**`Content-Type: application/json; charset=UTF-8`**

- الرد JSON format
- UTF-8 encoding (يدعم العربية ✅)

**`Content-Length: 342`** حجم الـ Body = 342 byte

**`Cache-Control: max-age=3600`**

- يمكن حفظ النتيجة في Cache لمدة 3600 ثانية (ساعة)
- بعد ساعة، اطلب من السيرفر مرة ثانية

**`ETag:`** معرّف unique للمحتوى

- لو المحتوى اتغير، الـ ETag يتغير
- يستخدم للـ Caching الذكي

### Response Body

JSON object يحتوي:

- `success`: هل العملية نجحت
- `data`: array من المنتجات
- `total`: عدد المنتجات
- `page`: رقم الصفحة (للـ pagination)

</div>

---

## 🔄 GET with Query Parameters

<div dir="rtl">

### السيناريو: نريد laptops فقط، مرتبة حسب السعر

</div>

```http
GET /api/products?category=laptops&sort=price&order=asc HTTP/1.1
Host: shop.example.com
Accept: application/json
```

<div dir="rtl">

**Query Parameters:**

- `category=laptops` - فقط فئة laptops
- `sort=price` - رتب حسب السعر
- `order=asc` - تصاعدياً (من الأرخص للأغلى)

</div>

---

## 🔐 GET with Authentication

<div dir="rtl">

### السيناريو: جلب بيانات profile المستخدم (محمي)

</div>

```http
GET /api/user/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1IiwibmFtZSI6IkFobWVkIn0.xyz
Accept: application/json
```

<div dir="rtl">

**Authorization header** يحمل JWT Token للمصادقة

</div>

---

## ⚠️ GET Error Example

<div dir="rtl">

### السيناريو: طلب منتج غير موجود

</div>

**Request:**

```http
GET /api/products/999 HTTP/1.1
Host: shop.example.com
Accept: application/json
```

**Response:**

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "success": false,
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "Product with ID 999 does not exist"
  }
}
```

---

## 💡 Key Points

<div dir="rtl">

### ✅ خصائص GET:

- **Safe:** لا يغير البيانات على السيرفر
- **Idempotent:** تنفيذه 100 مرة = نفس النتيجة
- **Cacheable:** يمكن حفظه في Cache

### ✅ لا يحتوي على Body

كل البيانات في:

- URL Path: `/api/products/5`
- Query String: `?category=laptops&page=2`

### ✅ متى نستخدم GET؟

- قراءة/جلب بيانات
- عرض قوائم
- البحث
- الفلترة

### ❌ متى لا نستخدم GET؟

- إنشاء بيانات جديدة → استخدم POST
- تحديث بيانات → استخدم PUT/PATCH
- حذف بيانات → استخدم DELETE
- إرسال بيانات حساسة (passwords) → لا تستخدم Query String!

</div>

---

## 🎯 Try It Yourself!

<div dir="rtl">

### استخدم أي أداة:

- **Browser DevTools** (F12 → Network tab)
- **Postman**
- **Thunder Client** (VS Code extension)
- **curl** command line

### جرب:

1. GET request لأي API عام
2. أضف Query Parameters
3. راقب الـ Headers في الـ Response
4. لاحظ الـ Status Code

</div>

---

<div align="center">

[📚 Back to Examples](../README.md#-examples-أمثلة-عملية)

</div>
