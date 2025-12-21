# Module 2.5: Functions ⚡

<div dir="rtl">

## نظرة عامة

Functions في Go - كيف تكتب وتستخدم functions بكفاءة.

</div>

---

## 📚 Topics

### 1. Basic Functions

```go
// Simple function
func sayHello() {
    fmt.Println("Hello!")
}

// Function with parameters
func greet(name string) {
    fmt.Printf("Hello, %s!\n", name)
}

// Function with return value
func add(a int, b int) int {
    return a + b
}

// Shorthand for same type parameters
func multiply(a, b int) int {
    return a * b
}

// Call functions
sayHello()           // Hello!
greet("Ahmed")       // Hello, Ahmed!
result := add(5, 3)  // 8
```

### 2. Multiple Return Values

```go
// Return multiple values
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
}

// Usage
result, err := divide(10, 2)
if err != nil {
    fmt.Println("Error:", err)
} else {
    fmt.Println("Result:", result)
}

// Ignore return value with _
result, _ := divide(10, 2)  // Ignore error

// Named return values
func calculate(a, b int) (sum, product int) {
    sum = a + b
    product = a * b
    return  // Naked return
}

sum, product := calculate(5, 3)  // 8, 15
```

### 3. Variadic Functions

```go
// Accept variable number of arguments
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

// Usage
sum(1, 2)           // 3
sum(1, 2, 3, 4, 5)  // 15

// Pass slice
nums := []int{1, 2, 3, 4}
sum(nums...)  // Spread operator

// Variadic with other parameters
func greetAll(greeting string, names ...string) {
    for _, name := range names {
        fmt.Printf("%s, %s!\n", greeting, name)
    }
}

greetAll("Hello", "Ahmed", "Sara", "Omar")
```

### 4. Anonymous Functions & Closures

```go
// Anonymous function
add := func(a, b int) int {
    return a + b
}
result := add(5, 3)  // 8

// Immediately invoked
func() {
    fmt.Println("Anonymous function executed!")
}()

// Closure (captures outer variable)
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

increment := counter()
fmt.Println(increment())  // 1
fmt.Println(increment())  // 2
fmt.Println(increment())  // 3
```

### 5. Defer

```go
// Defer delays execution until function returns
func example() {
    defer fmt.Println("World")  // Executed last
    fmt.Println("Hello")        // Executed first
}
// Output:
// Hello
// World

// Common use: cleanup
func readFile(filename string) {
    file, err := os.Open(filename)
    if err != nil {
        return
    }
    defer file.Close()  // Ensures file closes

    // Read file...
}

// Multiple defers (LIFO - Last In First Out)
func multipleDefers() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
}
// Output: 3, 2, 1
```

---

## 🎯 Complete Examples

### Example 1: Calculator

```go
package main

import (
    "fmt"
    "errors"
)

func add(a, b float64) float64 {
    return a + b
}

func subtract(a, b float64) float64 {
    return a - b
}

func multiply(a, b float64) float64 {
    return a * b
}

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func main() {
    var a, b float64
    var op string

    fmt.Print("Enter first number: ")
    fmt.Scan(&a)

    fmt.Print("Enter operator (+, -, *, /): ")
    fmt.Scan(&op)

    fmt.Print("Enter second number: ")
    fmt.Scan(&b)

    var result float64
    var err error

    switch op {
    case "+":
        result = add(a, b)
    case "-":
        result = subtract(a, b)
    case "*":
        result = multiply(a, b)
    case "/":
        result, err = divide(a, b)
        if err != nil {
            fmt.Println("Error:", err)
            return
        }
    default:
        fmt.Println("Invalid operator!")
        return
    }

    fmt.Printf("Result: %.2f\n", result)
}
```

### Example 2: Fibonacci Generator

```go
package main

import "fmt"

func fibonacci(n int) []int {
    if n <= 0 {
        return []int{}
    }
    if n == 1 {
        return []int{0}
    }

    fibs := make([]int, n)
    fibs[0], fibs[1] = 0, 1

    for i := 2; i < n; i++ {
        fibs[i] = fibs[i-1] + fibs[i-2]
    }

    return fibs
}

func main() {
    fmt.Println(fibonacci(10))  // [0 1 1 2 3 5 8 13 21 34]
}
```

### Example 3: Higher-Order Functions

```go
package main

import "fmt"

// Function that takes function as parameter
func applyOperation(a, b int, op func(int, int) int) int {
    return op(a, b)
}

func main() {
    add := func(x, y int) int { return x + y }
    multiply := func(x, y int) int { return x * y }

    result1 := applyOperation(5, 3, add)       // 8
    result2 := applyOperation(5, 3, multiply)  // 15

    fmt.Println(result1, result2)
}
```

---

## ✅ Best Practices

```go
// ✅ Return errors, don't panic
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")  // Good
        // panic("division by zero")  // Bad!
    }
    return a / b, nil
}

// ✅ Use defer for cleanup
func processFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()  // Always closes

    // Process file...
    return nil
}

// ✅ Keep functions small and focused
// Bad: function does too much
func processEverything() { /* 200 lines */ }

// Good: split into smaller functions
func validateInput() { }
func processData() { }
func saveResult() { }

// ✅ Use named return values fوr clarity
func getUserInfo(id int) (name string, age int, err error) {
    // Clear what function returns
}
```

---

## 🎯 Practice

<div dir="rtl">

**Task:** Create utility functions:

1. `isPrime(n int) bool` - check if prime
2. `factorial(n int) int` - calculate factorial
3. `reverse(s string) string` - reverse string

</div>

```go
package main

import "fmt"

func isPrime(n int) bool {
    if n < 2 {
        return false
    }
    for i := 2; i*i <= n; i++ {
        if n%i == 0 {
            return false
        }
    }
    return true
}

func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}

func reverse(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

func main() {
    fmt.Println(isPrime(17))       // true
    fmt.Println(factorial(5))      // 120
    fmt.Println(reverse("Hello"))  // olleH
}
```

---

## 🎉 Congratulations!

<div dir="rtl">

أنهيت **Track 2: Go Basics**! 🚀

**تعلمت:**

- ✅ Installation & Setup
- ✅ Variables & Constants
- ✅ Data Types
- ✅ Control Flow
- ✅ Functions

**التالي:**
**➡️ Track 3: Go Advanced** (Arrays, Slices, Maps, Structs, Interfaces)

</div>

---

<div align="center">

[⬅️ Previous: Control Flow](../04-control-flow/README.md) | [🏠 Track 2](../README.md)

</div>
