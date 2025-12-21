# Module 3.5: Interfaces 🔌

<div dir="rtl">

## نظرة عامة

**Interfaces** = مجموعة من method signatures - قلب polymorphism في Go!

</div>

---

## 📚 Interface Basics

### Defining Interfaces

```go
// Interface defines behavior
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Any type with these methods implements Shape
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Rectangle implements Shape automatically!
var s Shape = Rectangle{Width: 10, Height: 5}
```

---

## 🎯 Practical Example

```go
package main

import (
    "fmt"
    "math"
)

// Interface
type Shape interface {
    Area() float64
}

// Circle implements Shape
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// Rectangle implements Shape
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Function accepts any Shape
func PrintArea(s Shape) {
    fmt.Printf("Area: %.2f\n", s.Area())
}

func main() {
    circle := Circle{Radius: 5}
    rectangle := Rectangle{Width: 10, Height: 5}

    PrintArea(circle)     // Area: 78.54
    PrintArea(rectangle)  // Area: 50.00
}
```

---

## 🔍 Empty Interface

```go
// interface{} accepts ANY type
func Print(value interface{}) {
    fmt.Println(value)
}

Print(42)           // int
Print("hello")      // string
Print([]int{1,2,3}) // slice

// Type assertion
var i interface{} = "hello"
s := i.(string)     // "hello"
n, ok := i.(int)    // n=0, ok=false
```

---

## 🆚 Type Switch

```go
func ProcessValue(value interface{}) {
    switch v := value.(type) {
    case string:
        fmt.Printf("String: %s\n", v)
    case int:
        fmt.Printf("Integer: %d\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

---

## ✅ Best Practices

```go
// ✅ Keep interfaces small
type Reader interface {
    Read() ([]byte, error)
}

// ✅ Accept interfaces, return structs
func ProcessReader(r Reader) Data {
    // Returns concrete type
}

// ✅ Use empty interface sparingly
// Only when truly needed for any type
```

---

## 🎉 Congratulations!

<div dir="rtl">

أنهيت **Track 3: Go Advanced**! 🚀

**تعلمت:**

- ✅ Arrays & Slices
- ✅ Maps
- ✅ Structs
- ✅ Methods
- ✅ Interfaces
- ✅ Pointers

**التالي:**
**➡️ Track 4: PostgreSQL & Databases**

</div>

---

<div align="center">

[⬅️ Previous: Methods](../04-methods/README.md) | [🏠 Track 3](../README.md)

</div>
