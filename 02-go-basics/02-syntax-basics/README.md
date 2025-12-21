# Module 2.2: Syntax Basics 📝

<div dir="rtl">

## نظرة عامة

أساسيات كتابة Code في Go - Variables, Constants, I/O.

</div>

---

## 📚 Topics

### 1. Variables

```go
// Declare with type
var name string = "Ahmed"
var age int = 25

// Type inference
var email = "ahmed@test.com"  // string inferred

// Short declaration (inside functions only)
city := "Cairo"

// Multiple variables
var x, y int = 10, 20
a, b := "hello", "world"

// Zero values (default values)
var count int      // 0
var price float64  // 0.0
var name string    // ""
var active bool    // false
```

### 2. Constants

```go
// Single constant
const PI = 3.14159

// Multiple constants
const (
    StatusOK = 200
    StatusNotFound = 404
    StatusError = 500
)

// Typed constant
const MaxUsers int = 1000

// Constants can't be changed
const Name = "Ahmed"
// Name = "Sara"  // ❌ Error!
```

### 3. Basic Input/Output

```go
package main

import "fmt"

func main() {
    // Print
    fmt.Println("Hello")           // With newline
    fmt.Print("World")             // Without newline
    fmt.Printf("Age: %d\n", 25)    // Formatted

    // Input
    var name string
    fmt.Print("Enter your name: ")
    fmt.Scan(&name)  // Read input
    fmt.Println("Hello,", name)

    // Formatted input
    var age int
    fmt.Print("Enter age: ")
    fmt.Scanf("%d", &age)
}
```

### 4. Comments

```go
// Single line comment

/*
Multi-line comment
Can span multiple lines
*/

// Good comment: explains WHY
// Calculate tax based on income bracket
tax := calculateTax(income)

// Bad comment: states the obvious
// Set x to 10
x := 10
```

---

## 🎯 Complete Example

```go
package main

import "fmt"

func main() {
    // Constants
    const AppName = "My Go App"
    const Version = "1.0.0"

    // Variables
    var username string
    var age int
    city := "Cairo"  // Short declaration

    // Input
    fmt.Printf("Welcome to %s v%s\n", AppName, Version)
    fmt.Print("Enter your name: ")
    fmt.Scan(&username)

    fmt.Print("Enter your age: ")
    fmt.Scanf("%d", &age)

    // Output
    fmt.Println("\n--- User Info ---")
    fmt.Printf("Name: %s\n", username)
    fmt.Printf("Age: %d\n", age)
    fmt.Printf("City: %s\n", city)
}
```

**Output:**

```
Welcome to My Go App v1.0.0
Enter your name: Ahmed
Enter your age: 25

--- User Info ---
Name: Ahmed
Age: 25
City: Cairo
```

---

## 🔧 Format Specifiers

```go
// %d - integer
fmt.Printf("Count: %d\n", 42)

// %f - float
fmt.Printf("Price: %.2f\n", 19.99)  // 2 decimal places

// %s - string
fmt.Printf("Name: %s\n", "Ahmed")

// %t - boolean
fmt.Printf("Active: %t\n", true)

// %v - default format
fmt.Printf("Value: %v\n", anything)

// %T - type
fmt.Printf("Type: %T\n", 42)  // int
```

---

## ✅ Best Practices

```go
// ✅ Use short declaration inside functions
func example() {
    name := "Ahmed"  // Good
}

// ✅ Use var for package-level
var GlobalConfig = "production"

// ✅ Group related constants
const (
    Monday = 1
    Tuesday = 2
    Wednesday = 3
)

// ✅ Use meaningful names
userCount := 10  // Good
x := 10          // Bad
```

---

## 🎯 Practice

<div dir="rtl">

**Task:** Create a simple calculator that:

1. Asks for two numbers
2. Prints sum, difference, product

</div>

```go
package main

import "fmt"

func main() {
    var a, b float64

    fmt.Print("Enter first number: ")
    fmt.Scan(&a)

    fmt.Print("Enter second number: ")
    fmt.Scan(&b)

    fmt.Println("\n--- Results ---")
    fmt.Printf("Sum: %.2f\n", a+b)
    fmt.Printf("Difference: %.2f\n", a-b)
    fmt.Printf("Product: %.2f\n", a*b)

    if b != 0 {
        fmt.Printf("Division: %.2f\n", a/b)
    }
}
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 2.3: Data Types](../03-data-types/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Installation](../01-installation-setup/README.md) | [🏠 Track 2](../README.md)

</div>
