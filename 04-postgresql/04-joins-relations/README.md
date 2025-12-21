# Module 4.4: Joins & Relations 🔗

<div dir="rtl">

## نظرة عامة

**JOINs** و **Relations** - ربط البيانات بين Tables.

</div>

---

## 🔑 Primary & Foreign Keys

### Primary Key

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- Auto-increment primary key
    email VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100)
);
```

### Foreign Key

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    user_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Foreign Key with Actions

```sql
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);
```

---

## 🔗 INNER JOIN

```sql
-- Get posts with user names
SELECT
    posts.title,
    posts.content,
    users.name AS author
FROM posts
INNER JOIN users ON posts.user_id = users.id;

-- Shorter with aliases
SELECT
    p.title,
    p.content,
    u.name AS author
FROM posts p
INNER JOIN users u ON p.user_id = u.id;
```

---

## ⬅️ LEFT JOIN

```sql
-- Get all users and their posts (even users with no posts)
SELECT
    u.name,
    p.title
FROM users u
LEFT JOIN posts p ON p.user_id = u.id;

-- Users without posts
SELECT u.name
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
WHERE p.id IS NULL;
```

---

## ➡️ RIGHT JOIN

```sql
-- Get all posts and their users
SELECT
    p.title,
    u.name
FROM users u
RIGHT JOIN posts p ON p.user_id = u.id;
```

---

## 🎯 Complete Example: Blog System

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Posts table
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    user_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Comments table
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Insert sample data
INSERT INTO users (name, email) VALUES
    ('Ahmed', 'ahmed@test.com'),
    ('Sara', 'sara@test.com'),
    ('Omar', 'omar@test.com');

INSERT INTO posts (title, content, user_id) VALUES
    ('First Post', 'Hello World', 1),
    ('Go Tutorial', 'Learn Go...', 1),
    ('PostgreSQL Guide', 'Database...', 2);

INSERT INTO comments (content, post_id, user_id) VALUES
    ('Great post!', 1, 2),
    ('Thanks!', 1, 1),
    ('Very helpful', 2, 3);

-- Get posts with authors
SELECT
    p.title,
    p.content,
    u.name AS author,
    p.created_at
FROM posts p
INNER JOIN users u ON p.user_id = u.id
ORDER BY p.created_at DESC;

-- Get post with comments
SELECT
    p.title,
    c.content AS comment,
    u.name AS commenter
FROM posts p
INNER JOIN comments c ON c.post_id = p.id
INNER JOIN users u ON c.user_id = u.id
WHERE p.id = 1;

-- Count posts per user
SELECT
    u.name,
    COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON p.user_id = u.id
GROUP BY u.id, u.name;
```

---

## 🔗 Many-to-Many Relationship

```sql
-- Students table
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- Courses table
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- Junction table (many-to-many)
CREATE TABLE student_courses (
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at TIMESTAMP DEFAULT NOW(),
    grade INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);

-- Insert data
INSERT INTO students (name) VALUES ('Ahmed'), ('Sara'), ('Omar');
INSERT INTO courses (name) VALUES ('Math'), ('Physics'), ('Chemistry');

INSERT INTO student_courses (student_id, course_id) VALUES
    (1, 1),  -- Ahmed enrolled in Math
    (1, 2),  -- Ahmed enrolled in Physics
    (2, 1),  -- Sara enrolled in Math
    (3, 2);  -- Omar enrolled in Physics

-- Get student's courses
SELECT
    s.name AS student,
    c.name AS course
FROM students s
INNER JOIN student_courses sc ON sc.student_id = s.id
INNER JOIN courses c ON c.id = sc.course_id
WHERE s.id = 1;

-- Get course's students
SELECT
    c.name AS course,
    s.name AS student
FROM courses c
INNER JOIN student_courses sc ON sc.course_id = c.id
INNER JOIN students s ON s.id = sc.student_id
WHERE c.id = 1;
```

---

## 📈 Indexes

```sql
-- Create index on frequently queried column
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Composite index
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Check query performance
EXPLAIN ANALYZE
SELECT * FROM posts WHERE user_id = 1;
```

---

## ✅ Best Practices

```sql
-- ✅ Always create indexes on foreign keys
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_comments_post_id ON comments(post_id);

-- ✅ Use appropriate ON DELETE actions
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;  -- Delete related
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL; -- Keep but null

-- ✅ Use meaningful aliases
SELECT p.title, u.name
FROM posts p
INNER JOIN users u ON p.user_id = u.id;

-- ✅ Specify columns instead of SELECT *
SELECT p.title, p.content, u.name  -- Better
-- SELECT *  -- Slower
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 4.5: Go Integration](../05-go-postgres-integration/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: CRUD](../03-crud-operations/README.md) | [🏠 Track 4](../README.md)

</div>
