# Module 3.1: Arrays & Slices 📊

<div dir="rtl">

## نظرة عامة

Arrays و Slices - أهم data structures في Go!

</div>

---

## 📚 Arrays

### Basic Array

```go
// Declare array with size
var numbers [5]int  // [0, 0, 0, 0, 0]

// Initialize with values
grades := [3]int{90, 85, 95}

// Let compiler count size
scores := [...]int{100, 95, 88, 92}  // size = 4

// Access elements
fmt.Println(grades[0])  // 90
grades[1] = 88          // Modify

// Array length
length := len(grades)  // 3
```

### Array Limitations

```go
// ❌ Arrays have FIXED size
var arr1 [3]int
var arr2 [5]int
// arr1 = arr2  // Error! Different types

// ✅ Use Slices instead!
```

---

## 📊 Slices (Dynamic Arrays)

### Creating Slices

```go
// Create empty slice
var numbers []int  // nil slice

// Literal
colors := []string{"red", "green", "blue"}

// make() function
scores := make([]int, 5)       // [0, 0, 0, 0, 0] - length 5
scores := make([]int, 5, 10)   // length 5, capacity 10
```

### Slice Operations

```go
numbers := []int{1, 2, 3, 4, 5}

// Append (most important!)
numbers = append(numbers, 6)         // [1, 2, 3, 4, 5, 6]
numbers = append(numbers, 7, 8, 9)   // Multiple values

// Slicing
numbers[1:4]   // [2, 3, 4] - from index 1 to 3
numbers[:3]    // [1, 2, 3] - from start to index 2
numbers[3:]    // [4, 5, ...] - from index 3 to end

// Copy
src := []int{1, 2, 3}
dst := make([]int, len(src))
copy(dst, src)  // dst = [1, 2, 3]
```

---

## 🎯 Complete Example

```go
package main

import "fmt"

func main() {
    grades := []int{85, 90, 78, 92, 88}

    // Calculate average
    sum := 0
    for _, grade := range grades {
        sum += grade
    }
    average := float64(sum) / float64(len(grades))

    fmt.Printf("Average: %.2f\n", average)

    // Add new grade
    grades = append(grades, 95)
    fmt.Println("Updated grades:", grades)
}
```

---

## ✅ Best Practices

```go
// ✅ Use slices, not arrays (99% of cases)
numbers := []int{1, 2, 3}  // Slice (good)

// ✅ Check nil before using
var slice []int
if slice != nil {
    // Use slice
}

// ✅ Preallocate capacity if size known
slice := make([]int, 0, 1000)  // Avoid reallocations

// ❌ Don't forget to reassign append result
numbers = append(numbers, 5)  // Correct
```

---

## ⏭️ Next Module

**➡️ [Module 3.2: Maps](../02-maps/README.md)**

---

<div align="center">

[🏠 Track 3](../README.md)

</div>
