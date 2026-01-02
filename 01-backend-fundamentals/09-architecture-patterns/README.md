# Module 1.9: Architecture Patterns - أنماط التصميم المعماري 🏗️

<div dir="rtl">

## نظرة عامة

**Architecture Patterns** هي أنماط وحلول مثبتة لبناء تطبيقات قابلة للتوسع والصيانة. في هذا الـ Module هنتعلم أهم الـ patterns للـ Backend.

**المدة المتوقعة:** 4-5 ساعات

</div>

---

## 🎯 Learning Objectives

<div dir="rtl">

بعد إكمال هذا الـ Module، ستتمكن من:

</div>

- ✅ Understand Monolith vs Microservices
- ✅ Apply Layered Architecture
- ✅ Implement Clean Architecture in Go
- ✅ Use Repository Pattern
- ✅ Apply Dependency Injection
- ✅ Design APIs following best practices

---

## 📊 Why Architecture Matters?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Why Architecture Matters?                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Poor Architecture:                                                 │
│  ───────────────────                                                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Spaghetti Code 🍝                          │    │
│  │                                                               │    │
│  │    Handler ←─┬─→ Database ←─┬─→ Business Logic               │    │
│  │       ↑      │       ↑      │         ↑                      │    │
│  │       └──────┼───────┴──────┼─────────┘                      │    │
│  │              │              │                                 │    │
│  │    Everything connected to everything!                       │    │
│  │    Change one thing = break everything                       │    │
│  │                                                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Good Architecture:                                                 │
│  ────────────────────                                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                               │    │
│  │    ┌─────────┐    ┌─────────┐    ┌─────────┐                │    │
│  │    │ Handler │───▶│ Service │───▶│  Repo   │                │    │
│  │    └─────────┘    └─────────┘    └─────────┘                │    │
│  │                        │              │                      │    │
│  │                   Interfaces     Interfaces                  │    │
│  │                        │              │                      │    │
│  │    Clear boundaries, easy to test and maintain              │    │
│  │                                                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Module Contents

### Lesson 1: Monolith vs Microservices
<div dir="rtl">

- ما الفرق بينهم؟
- متى تستخدم كل واحد؟
- Monolith First approach

</div>

**➡️ [Start Lesson 1](./lessons/01-monolith-vs-microservices.md)**

---

### Lesson 2: Layered Architecture
<div dir="rtl">

- الطبقات الأساسية
- Separation of Concerns
- Go implementation

</div>

**➡️ [Start Lesson 2](./lessons/02-layered-architecture.md)**

---

### Lesson 3: Clean Architecture
<div dir="rtl">

- مبادئ Clean Architecture
- Dependency Rule
- تطبيق في Go

</div>

**➡️ [Start Lesson 3](./lessons/03-clean-architecture.md)**

---

### Lesson 4: Repository Pattern
<div dir="rtl">

- ما هو Repository Pattern؟
- لماذا نستخدمه؟
- Implementation في Go

</div>

**➡️ [Start Lesson 4](./lessons/04-repository-pattern.md)**

---

### Lesson 5: Dependency Injection
<div dir="rtl">

- ما هو DI؟
- Constructor Injection
- Wire for Go

</div>

**➡️ [Start Lesson 5](./lessons/05-dependency-injection.md)**

---

### Lesson 6: API Design Best Practices
<div dir="rtl">

- RESTful conventions
- Versioning
- Error handling patterns

</div>

**➡️ [Start Lesson 6](./lessons/06-api-design.md)**

---

## 🛠️ Prerequisites

<div dir="rtl">

قبل البدء، تأكد من:

</div>

- ✅ Completed Go basics (Module 2)
- ✅ Completed REST API (Module 1.3)
- ✅ Basic understanding of interfaces in Go

---

## 📊 Common Architecture Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Architecture Patterns Overview                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Monolithic                                                      │
│     └─ Single deployable unit                                       │
│     └─ Good for: Small teams, MVPs, simple apps                    │
│                                                                      │
│  2. Layered (N-Tier)                                                │
│     └─ Horizontal layers (Presentation, Business, Data)            │
│     └─ Good for: Most web applications                             │
│                                                                      │
│  3. Clean/Hexagonal                                                 │
│     └─ Domain-centric, dependency inversion                        │
│     └─ Good for: Complex business logic                            │
│                                                                      │
│  4. Microservices                                                   │
│     └─ Independent deployable services                             │
│     └─ Good for: Large teams, complex systems, scaling             │
│                                                                      │
│  5. Event-Driven                                                    │
│     └─ Async communication via events                              │
│     └─ Good for: Decoupling, real-time systems                     │
│                                                                      │
│  6. CQRS                                                            │
│     └─ Separate read and write models                              │
│     └─ Good for: High-performance, complex queries                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Quick Decision Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Architecture Decision Guide                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Team Size:                                                         │
│  ──────────                                                          │
│  1-5 developers    → Monolith with clean internal structure         │
│  5-20 developers   → Modular Monolith or few Microservices          │
│  20+ developers    → Microservices (if justified)                   │
│                                                                      │
│  Project Stage:                                                     │
│  ──────────────                                                      │
│  MVP/Startup       → Monolith (iterate fast!)                       │
│  Growing product   → Modular Monolith                               │
│  Mature product    → Consider Microservices for bottlenecks         │
│                                                                      │
│  Complexity:                                                        │
│  ───────────                                                         │
│  Simple CRUD       → Layered Architecture                           │
│  Complex domain    → Clean Architecture                             │
│  Multiple domains  → Microservices or Modular Monolith              │
│                                                                      │
│  ⚠️ Start Simple! You can always evolve the architecture.          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📁 Recommended Go Project Structure

```
myapp/
├── cmd/
│   └── server/
│       └── main.go           # Entry point
├── internal/
│   ├── config/               # Configuration
│   ├── domain/               # Business entities
│   │   ├── user.go
│   │   └── order.go
│   ├── repository/           # Data access
│   │   ├── user_repository.go
│   │   └── order_repository.go
│   ├── service/              # Business logic
│   │   ├── user_service.go
│   │   └── order_service.go
│   ├── handler/              # HTTP handlers
│   │   ├── user_handler.go
│   │   └── order_handler.go
│   └── middleware/           # HTTP middleware
├── pkg/                      # Reusable packages
├── migrations/               # Database migrations
├── config/                   # Config files
├── scripts/                  # Build/deploy scripts
├── go.mod
├── go.sum
└── Makefile
```

---

## ⏭️ Start Learning

<div dir="rtl">

ابدأ بالـ Lesson الأول:

**➡️ [Lesson 1: Monolith vs Microservices](./lessons/01-monolith-vs-microservices.md)**

</div>

---

<div align="center">

[⬅️ Previous: Docker Basics](../08-docker-basics/README.md) | [🏠 Track 1 Home](../README.md)

</div>
