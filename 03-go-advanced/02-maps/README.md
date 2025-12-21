# Module 3.2: Maps 🗺️

<div dir="rtl">

## نظرة عامة

**Maps** = Key-Value pairs (مثل Dictionary/HashMap).

</div>

---

## 📚 Basic Maps

### Creating Maps

```go
// make() function
ages := make(map[string]int)

// Literal initialization
scores := map[string]int{
    "Ahmed": 95,
    "Sara":  88,
    "Omar":  92,
}

// Empty map
var colors map[string]string  // nil map
```

### CRUD Operations

```go
ages := make(map[string]int)

// Create/Update
ages["Ahmed"] = 25
ages["Sara"] = 30

// Read
age := ages["Ahmed"]  // 25

// Check if key exists
age, exists := ages["Omar"]
if exists {
    fmt.Println(age)
} else {
    fmt.Println("Not found")
}

// Delete
delete(ages, "Sara")

// Length
fmt.Println(len(ages))  // Number of key-value pairs
```

---

## 🔄 Iterating Maps

```go
scores := map[string]int{
    "Ahmed": 95,
    "Sara":  88,
    "Omar":  92,
}

// Iterate over map
for name, score := range scores {
    fmt.Printf("%s: %d\n", name, score)
}

// Only keys
for name := range scores {
    fmt.Println(name)
}

// Only values
for _, score := range scores {
    fmt.Println(score)
}
```

---

## 🎯 Complete Examples

### Example 1: Word Count

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "hello world hello go world"
    words := strings.Split(text, " ")

    wordCount := make(map[string]int)

    for _, word := range words {
        wordCount[word]++
    }

    for word, count := range wordCount {
        fmt.Printf("%s: %d\n", word, count)
    }
}
```

### Example 2: Student Grades

```go
package main

import "fmt"

func main() {
    students := map[string][]int{
        "Ahmed": {85, 90, 88},
        "Sara":  {92, 95, 90},
        "Omar":  {78, 82, 85},
    }

    for name, grades := range students {
        sum := 0
        for _, grade := range grades {
            sum += grade
        }
        avg := float64(sum) / float64(len(grades))
        fmt.Printf("%s: %.2f\n", name, avg)
    }
}
```

---

## ✅ Best Practices

```go
// ✅ Check if key exists before using
if value, exists := myMap[key]; exists {
    // Use value
}

// ✅ Initialize with make or literal
ages := make(map[string]int)  // Good
// var ages map[string]int  // nil map - can't add to it!

// ✅ Use _ to ignore values you don't need
for key := range myMap {
    // Only need keys
}
```

---

## ⏭️ Next Module

**➡️ [Module 3.3: Structs](../03-structs/README.md)**

---

<div align="center">

[⬅️ Previous: Arrays & Slices](../01-arrays-slices/README.md) | [🏠 Track 3](../README.md)

</div>
