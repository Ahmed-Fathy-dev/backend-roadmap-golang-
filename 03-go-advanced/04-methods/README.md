# Module 3.4: Methods 🔧

<div dir="rtl">

## نظرة عامة

**Methods** = Functions مرتبطة بـ type معين.

</div>

---

## 📚 Methods Basics

### Defining Methods

```go
type Rectangle struct {
    Width, Height float64
}

// Method with value receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Method with pointer receiver
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// Usage
rect := Rectangle{Width: 10, Height: 5}
area := rect.Area()      // 50
rect.Scale(2)            // Modifies rect
fmt.Println(rect.Width)  // 20
```

---

## 🆚 Value vs Pointer Receivers

```go
type Counter struct {
    Count int
}

// Value receiver - doesn't modify original
func (c Counter) Increment() {
    c.Count++  // Only modifies copy!
}

// Pointer receiver - modifies original
func (c *Counter) IncrementPtr() {
    c.Count++  // Modifies original ✅
}

func main() {
    counter := Counter{Count: 0}

    counter.Increment()     // counter.Count still 0
    counter.IncrementPtr()  // counter.Count now 1
}
```

---

## 🎯 When to Use Pointer Receivers

```go
// ✅ Use pointer receiver when:
// 1. Method needs to modify receiver
func (p *Person) UpdateAge(age int) {
    p.Age = age
}

// 2. Receiver is large struct (avoid copying)
type BigStruct struct {
    Data [1000000]int
}

func (b *BigStruct) Process() {  // Pointer to avoid copying
    // ...
}

// 3. For consistency (if some methods use pointer)
type User struct {
    Name string
}

func (u *User) SetName(name string) { u.Name = name }
func (u *User) GetName() string { return u.Name }  // Pointer for consistency
```

---

## 🔧 Complete Example

```go
package main

import (
    "fmt"
    "math"
)

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

func (c *Circle) Scale(factor float64) {
    c.Radius *= factor
}

func main() {
    circle := Circle{Radius: 5}

    fmt.Printf("Area: %.2f\n", circle.Area())
    fmt.Printf("Perimeter: %.2f\n", circle.Perimeter())

    circle.Scale(2)
    fmt.Printf("New radius: %.2f\n", circle.Radius)  // 10
}
```

---

## ✅ Best Practices

```go
// ✅ Use pointer receivers for consistency
type Person struct {
    Name string
    Age  int
}

// All methods use pointer receiver
func (p *Person) SetName(name string) { p.Name = name }
func (p *Person) GetName() string { return p.Name }
func (p *Person) SetAge(age int) { p.Age = age }

// ✅ Value receivers for small, immutable types
type Point struct {
    X, Y int
}

func (p Point) Distance(other Point) float64 {
    // Read-only, small struct
}
```

---

## ⏭️ Next Module

**➡️ [Module 3.5: Interfaces](../05-interfaces/README.md)**

---

<div align="center">

[⬅️ Previous: Structs](../03-structs/README.md) | [🏠 Track 3](../README.md)

</div>
