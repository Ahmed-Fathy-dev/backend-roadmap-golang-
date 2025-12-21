# Lesson 3: Relationships في Databases 🔗

<div dir="rtl">

## المقدمة

**Relationships** هي القلب النابض للـ Relational Databases!

فهم العلاقات بين Tables ضرور ي لتصميم database صحيح.

</div>

---

## 🎯 Types of Relationships

<div dir="rtl">

هناك **3 أنواع** رئيسية:

</div>

```
1. One-to-One (1:1)     - واحد لواحد
2. One-to-Many (1:M)    - واحد لكثير
3. Many-to-Many (M:M)   - كثير لكثير
```

---

## 1️⃣ One-to-One (1:1)

<div dir="rtl">

### المعنى:

كل row في Table A يرتبط بـ row **واحد فقط** في Table B، والعكس.

### مثال من الحياة:

- كل شخص له **رقم وطني واحد**
- كل رقم وطني لـ **شخص واحد**

</div>

### Database Example: User & Profile

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);

-- User profiles table (1:1 with users)
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INT UNIQUE NOT NULL,           -- ⭐ UNIQUE = 1:1
    full_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(255),
    phone VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

```
users table:
┌────┬──────────────────┐
│ id │      email       │
├────┼──────────────────┤
│ 1  │ ahmed@test.com   │
│ 2  │ sara@test.com    │
└────┴──────────────────┘
        │
        │ 1:1
        ↓
user_profiles table:
┌────┬─────────┬──────────────┐
│ id │ user_id │  full_name   │
├────┼─────────┼──────────────┤
│ 1  │    1    │ Ahmed Ali    │
│ 2  │    2    │ Sara Mohamed │
└────┴─────────┴──────────────┘
```

### Query Example:

```sql
-- Get user with profile
SELECT
    u.email,
    p.full_name,
    p.bio
FROM users u
JOIN user_profiles p ON p.user_id = u.id
WHERE u.id = 1;
```

### In Go (GORM):

```go
type User struct {
    ID      uint
    Email   string
    Profile UserProfile  // 1:1
}

type UserProfile struct {
    ID       uint
    UserID   uint    // Foreign key
    FullName string
    Bio      string
}

// Query
var user User
db.Preload("Profile").First(&user, 1)
```

---

## 2️⃣ One-to-Many (1:M) ⭐ الأكثر شيوعاً

<div dir="rtl">

### المعنى:

كل row في Table A يرتبط بـ **عدة rows** في Table B.

### مثال من الحياة:

- كل **مستخدم** له **عدة منشورات**
- كل **منشور** لـ **مستخدم واحد**

</div>

### Example: User & Posts

```sql
-- Users (one side)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Posts (many side)
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,                  -- Foreign key
    title VARCHAR(200),
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

```
users table:
┌────┬────────┐
│ id │  name  │
├────┼────────┤
│ 1  │ Ahmed  │
│ 2  │ Sara   │
└────┴────────┘
     │
     │ 1:Many
     ↓
posts table:
┌────┬─────────┬─────────────────────┐
│ id │ user_id │       title         │
├────┼─────────┼─────────────────────┤
│ 1  │    1    │ First Post          │ ← Ahmed's
│ 2  │    1    │ Second Post         │ ← Ahmed's
│ 3  │    1    │ Third Post          │ ← Ahmed's
│ 4  │    2    │ Sara's Post         │ ← Sara's
└────┴─────────┴─────────────────────┘
          ↑
    Foreign Key points to users.id
```

### Query Examples:

```sql
-- Get all posts by user 1
SELECT * FROM posts WHERE user_id = 1;

-- Get user with their posts
SELECT
    u.name,
    p.title,
    p.created_at
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
WHERE u.id = 1;

-- Count posts per user
SELECT
    u.name,
    COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
GROUP BY u.id, u.name;
```

### In Go:

```go
type User struct {
    ID    uint
    Name  string
    Posts []Post  // 1:Many (slice!)
}

type Post struct {
    ID      uint
    UserID  uint   // Foreign key
    Title   string
    Content string
}

// Get user with posts
var user User
db.Preload("Posts").First(&user, 1)

fmt.Println(user.Name)
for _, post := range user.Posts {
    fmt.Println("-", post.Title)
}
```

---

## 3️⃣ Many-to-Many (M:M)

<div dir="rtl">

### المعنى:

عدة rows في Table A يرتبطون بـ **عدة rows** في Table B.

### مثال من الحياة:

- كل **طالب** يدرس **عدة مواد**
- كل **مادة** يدرسها **عدة طلاب**

</div>

### Example: Students & Courses

```sql
-- Students table
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Courses table
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Junction/Pivot table (many-to-many)
CREATE TABLE student_courses (
    id SERIAL PRIMARY KEY,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at TIMESTAMP DEFAULT NOW(),
    grade CHAR(1),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id),
    UNIQUE(student_id, course_id)  -- Can't enroll twice
);
```

```
students:
┌────┬────────┐
│ id │  name  │
├────┼────────┤
│ 1  │ Ahmed  │
│ 2  │ Sara   │
│ 3  │ Omar   │
└────┴────────┘
         ╲
          ╲  Many:Many
           ╲
student_courses (Junction):        courses:
┌────┬────────────┬───────────┐   ┌────┬────────────┐
│ id │ student_id │ course_id │   │ id │    name    │
├────┼────────────┼───────────┤   ├────┼────────────┤
│ 1  │     1      │     1     │   │ 1  │ Math       │
│ 2  │     1      │     2     │   │ 2  │ Physics    │
│ 3  │     2      │     1     │   │ 3  │ Chemistry  │
│ 4  │     2      │     3     │   └────┴────────────┘
│ 5  │     3      │     2     │
└────┴────────────┴───────────┘

Ahmed (1) → Math (1), Physics (2)
Sara (2)  → Math (1), Chemistry (3)
Omar (3)  → Physics (2)
```

### Query Examples:

```sql
-- Get all courses for Ahmed (student_id = 1)
SELECT c.name
FROM courses c
JOIN student_courses sc ON sc.course_id = c.id
WHERE sc.student_id = 1;

-- Get all students in Math (course_id = 1)
SELECT s.name
FROM students s
JOIN student_courses sc ON sc.student_id = s.id
WHERE sc.course_id = 1;

-- Get student with all their courses
SELECT
    s.name AS student,
    c.name AS course,
    sc.grade
FROM students s
JOIN student_courses sc ON sc.student_id = s.id
JOIN courses c ON c.id = sc.course_id
WHERE s.id = 1;
```

### In Go:

```go
type Student struct {
    ID      uint
    Name    string
    Courses []Course `gorm:"many2many:student_courses"`
}

type Course struct {
    ID       uint
    Name     string
    Students []Student `gorm:"many2many:student_courses"`
}

// Junction table (optional if you need extra fields)
type StudentCourse struct {
    StudentID  uint
    CourseID   uint
    EnrolledAt time.Time
    Grade      string
}

// Get student with courses
var student Student
db.Preload("Courses").First(&student, 1)

// Add course to student
course := Course{ID: 2}
db.Model(&student).Association("Courses").Append(&course)
```

---

## 🎯 Choosing the Right Relationship

| Scenario                                               | Relationship |
| ------------------------------------------------------ | ------------ |
| <div dir="rtl">User → Profile</div>                    | 1:1          |
| <div dir="rtl">User → Posts</div>                      | 1:Many       |
| <div dir="rtl">User → Address (shipping/billing)</div> | 1:Many       |
| <div dir="rtl">Category → Products</div>               | 1:Many       |
| <div dir="rtl">Students ↔ Courses</div>                | Many:Many    |
| <div dir="rtl">Products ↔ Tags</div>                   | Many:Many    |
| <div dir="rtl">Authors ↔ Books</div>                   | Many:Many    |

---

## 💡 Best Practices

### 1. Always Use Foreign Keys

```sql
-- ✅ With foreign key (enforces integrity)
FOREIGN KEY (user_id) REFERENCES users(id)

-- ❌ Without (no integrity check!)
-- user_id INT  -- Just a number, no constraint
```

### 2. Index Foreign Keys

```sql
-- ✅ Index for faster joins
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_student_courses_student_id ON student_courses(student_id);
```

### 3. ON DELETE Behavior

```sql
-- Cascade: Delete related rows
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

-- Set NULL: Keep row but set FK to NULL
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL

-- Restrict: Prevent deletion if has related rows
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ **1:1** - UNIQUE constraint على Foreign Key
- ✅ **1:Many** - Foreign Key في "Many" side
- ✅ **Many:Many** - Junction table ضرورية
- ✅ دائماً استخدم **Foreign Keys**
- ✅ Index على **Foreign Keys**
- ✅ حدد **ON DELETE** behavior

</div>

---

<div align="center">

[⬅️ Previous: Relational Model](./02-relational-model.md) | [➡️ Next: ACID](./04-acid.md) | [📚 Module Home](../README.md)

</div>
