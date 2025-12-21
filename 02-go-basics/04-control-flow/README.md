# Module 2.4: Control Flow 🔄

<div dir="rtl">

## نظرة عامة

Control Flow - التحكم في تدفق البرنامج: if/else, switch, loops.

</div>

---

## 📚 Topics

### 1. if/else

```go
// Basic if
age := 25
if age >= 18 {
    fmt.Println("Adult")
}

// if-else
if age >= 18 {
    fmt.Println("Adult")
} else {
    fmt.Println("Minor")
}

// if-else if-else
score := 85
if score >= 90 {
    fmt.Println("A")
} else if score >= 80 {
    fmt.Println("B")
} else if score >= 70 {
    fmt.Println("C")
} else {
    fmt.Println("F")
}

// if with short statement
if num := 10; num > 5 {
    fmt.Println("Greater than 5")
}
// num not available here
```

### 2. switch

```go
// Basic switch
day := "Monday"
switch day {
case "Monday":
    fmt.Println("Start of week")
case "Friday":
    fmt.Println("End of week")
case "Saturday", "Sunday":  // Multiple cases
    fmt.Println("Weekend!")
default:
    fmt.Println("Midweek")
}

// Switch with no condition (like if-else chain)
age := 25
switch {
case age < 13:
    fmt.Println("Child")
case age < 18:
    fmt.Println("Teenager")
case age < 65:
    fmt.Println("Adult")
default:
    fmt.Println("Senior")
}

// Type switch
var x interface{} = "hello"
switch v := x.(type) {
case string:
    fmt.Printf("String: %s\n", v)
case int:
    fmt.Printf("Integer: %d\n", v)
default:
    fmt.Printf("Unknown type")
}
```

### 3. for Loop (The Only Loop in Go!)

```go
// Standard for loop
for i := 0; i < 5; i++ {
    fmt.Println(i)  // 0, 1, 2, 3, 4
}

// While-style loop
count := 0
for count < 5 {
    fmt.Println(count)
    count++
}

// Infinite loop
for {
    fmt.Println("Forever...")
    break  // Exit loop
}

// Range over slice
numbers := []int{1, 2, 3, 4, 5}
for index, value := range numbers {
    fmt.Printf("Index: %d, Value: %d\n", index, value)
}

// Ignore index with _
for _, value := range numbers {
    fmt.Println(value)
}

// Range over string (iterates over runes)
for index, char := range "Hello" {
    fmt.Printf("%d: %c\n", index, char)
}

// Range over map
ages := map[string]int{"Ahmed": 25, "Sara": 30}
for name, age := range ages {
    fmt.Printf("%s is %d years old\n", name, age)
}
```

### 4. break & continue

```go
// break - exit loop
for i := 0; i < 10; i++ {
    if i == 5 {
        break  // Stops at 5
    }
    fmt.Println(i)  // 0, 1, 2, 3, 4
}

// continue - skip iteration
for i := 0; i < 5; i++ {
    if i == 2 {
        continue  // Skip 2
    }
    fmt.Println(i)  // 0, 1, 3, 4
}

// Label with break (nested loops)
outer:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if j == 1 {
            break outer  // Breaks outer loop
        }
        fmt.Printf("%d,%d ", i, j)
    }
}
```

---

## 🎯 Complete Examples

### Example 1: Grade Calculator

```go
package main

import "fmt"

func main() {
    var score int

    fmt.Print("Enter score (0-100): ")
    fmt.Scan(&score)

    // Validate input
    if score < 0 || score > 100 {
        fmt.Println("Invalid score!")
        return
    }

    // Calculate grade
    var grade string
    switch {
    case score >= 90:
        grade = "A"
    case score >= 80:
        grade = "B"
    case score >= 70:
        grade = "C"
    case score >= 60:
        grade = "D"
    default:
        grade = "F"
    }

    fmt.Printf("Grade: %s\n", grade)

    // Pass/Fail
    if score >= 60 {
        fmt.Println("Status: PASS ✅")
    } else {
        fmt.Println("Status: FAIL ❌")
    }
}
```

### Example 2: FizzBuzz

```go
package main

import "fmt"

func main() {
    for i := 1; i <= 30; i++ {
        switch {
        case i%15 == 0:
            fmt.Println("FizzBuzz")
        case i%3 == 0:
            fmt.Println("Fizz")
        case i%5 == 0:
            fmt.Println("Buzz")
        default:
            fmt.Println(i)
        }
    }
}
```

### Example 3: Find Prime Numbers

```go
package main

import "fmt"

func main() {
    fmt.Println("Prime numbers from 1 to 50:")

    for num := 2; num <= 50; num++ {
        isPrime := true

        for i := 2; i*i <= num; i++ {
            if num%i == 0 {
                isPrime = false
                break
            }
        }

        if isPrime {
            fmt.Printf("%d ", num)
        }
    }
    fmt.Println()
}
```

---

## ✅ Best Practices

```go
// ✅ Use switch instead of long if-else chains
// Instead of:
if x == 1 {
    // ...
} else if x == 2 {
    // ...
} else if x == 3 {
    // ...
}

// Use:
switch x {
case 1:
    // ...
case 2:
    // ...
case 3:
    // ...
}

// ✅ Use range for iterating slices/maps
for i, v := range slice {
    // Better than: for i := 0; i < len(slice); i++
}

// ✅ Use _, to ignore unused values
for _, value := range numbers {
    // Don't need index
}
```

---

## 🎯 Practice

<div dir="rtl">

**Task:** Create a number guessing game:

1. Computer picks random number (1-100)
2. User guesses
3. Show "Too high" or "Too low"
4. Count attempts

</div>

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
    target := rand.Intn(100) + 1
    attempts := 0

    fmt.Println("Guess the number (1-100)!")

    for {
        var guess int
        fmt.Print("Your guess: ")
        fmt.Scan(&guess)
        attempts++

        if guess < target {
            fmt.Println("Too low! Try again.")
        } else if guess > target {
            fmt.Println("Too high! Try again.")
        } else {
            fmt.Printf("Correct! You won in %d attempts! 🎉\n", attempts)
            break
        }
    }
}
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 2.5: Functions](../05-functions/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Data Types](../03-data-types/README.md) | [🏠 Track 2](../README.md)

</div>
