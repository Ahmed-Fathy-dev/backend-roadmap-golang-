# Module 2.3: Data Types 🔢

<div dir="rtl">

## نظرة عامة

فهم أنواع البيانات في Go - Numbers, Strings, Booleans, Type Conversion.

</div>

---

## 📚 Topics

### 1. Numbers

#### Integers:

```go
// Signed integers
var age int8 = 25        // -128 to 127
var count int16 = 1000   // -32768 to 32767
var population int32 = 1000000
var bigNumber int64 = 9223372036854775807

// Unsigned integers (no negative)
var positive uint8 = 255   // 0 to 255
var id uint64 = 18446744073709551615

// Platform dependent
var x int   // 32 or 64 bit depending on system
var y uint  // 32 or 64 bit
```

#### Floats:

```go
var price float32 = 19.99
var pi float64 = 3.14159265359  // More precision

// Scientific notation
var avogadro float64 = 6.02e23
```

### 2. Strings

```go
// Single/double line
var name string = "Ahmed"
var greeting = "Hello, World!"

// Multi-line string
var message = `This is
a multi-line
string`

// String concatenation
firstName := "Ahmed"
lastName := "Ali"
fullName := firstName + " " + lastName  // "Ahmed Ali"

// String length
length := len("Hello")  // 5

// String indexing (returns byte)
first := "Hello"[0]  // 'H' (byte)

// Strings are immutable
// name[0] = 'M'  // ❌ Error!
```

#### String Operations:

```go
import "strings"

// Contains
strings.Contains("Hello", "ell")  // true

// Count
strings.Count("cheese", "e")  // 3

// HasPrefix/HasSuffix
strings.HasPrefix("Hello", "He")  // true
strings.HasSuffix("Hello", "lo")  // true

// Index
strings.Index("Hello", "l")  // 2

// Replace
strings.Replace("foo foo foo", "foo", "bar", 2)  // "bar bar foo"

// Split
strings.Split("a,b,c", ",")  // ["a", "b", "c"]

// Join
strings.Join([]string{"a", "b", "c"}, "-")  // "a-b-c"

// ToLower/ToUpper
strings.ToLower("HELLO")  // "hello"
strings.ToUpper("hello")  // "HELLO"
```

### 3. Booleans

```go
var isActive bool = true
var isCompleted = false

// Boolean operators
result := true && false   // AND: false
result = true || false    // OR: true
result = !true            // NOT: false

// Comparison operators
age := 25
isAdult := age >= 18      // true
isEqual := age == 25      // true
```

### 4. Type Conversion

```go
// int to float
var i int = 42
var f float64 = float64(i)  // 42.0

// float to int (truncates)
var price float64 = 19.99
var roundedPrice int = int(price)  // 19 (not 20!)

// string to int
import "strconv"

num, err := strconv.Atoi("123")  // 123
if err != nil {
    // Handle error
}

// int to string
str := strconv.Itoa(123)  // "123"

// string to float
price, err := strconv.ParseFloat("19.99", 64)  // 19.99

// float to string
priceStr := strconv.FormatFloat(19.99, 'f', 2, 64)  // "19.99"
```

---

## 🎯 Complete Example

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

func main() {
    // Numbers
    var age int = 25
    var price float64 = 19.99

    fmt.Printf("Age: %d\n", age)
    fmt.Printf("Price: %.2f\n", price)

    // Strings
    name := "Ahmed Ali"
    fmt.Printf("Name: %s\n", name)
    fmt.Printf("Length: %d\n", len(name))
    fmt.Printf("Uppercase: %s\n", strings.ToUpper(name))

    // Boolean
    isStudent := true
    fmt.Printf("Is Student: %t\n", isStudent)

    // Type conversion
    ageStr := strconv.Itoa(age)
    fmt.Printf("Age as string: '%s'\n", ageStr)

    priceInt := int(price)
    fmt.Printf("Price as int: %d\n", priceInt)
}
```

**Output:**

```
Age: 25
Price: 19.99
Name: Ahmed Ali
Length: 9
Uppercase: AHMED ALI
Is Student: true
Age as string: '25'
Price as int: 19
```

---

## 📊 Data Types Summary

| Type    | Size     | Range              | Example                    |
| ------- | -------- | ------------------ | -------------------------- |
| int8    | 1 byte   | -128 to 127        | `var x int8 = 100`         |
| int16   | 2 bytes  | -32768 to 32767    | `var x int16 = 1000`       |
| int32   | 4 bytes  | ~-2B to 2B         | `var x int32 = 1000000`    |
| int64   | 8 bytes  | Very large         | `var x int64 = 9999999999` |
| uint8   | 1 byte   | 0 to 255           | `var x uint8 = 200`        |
| float32 | 4 bytes  | ~6 decimal digits  | `var x float32 = 3.14`     |
| float64 | 8 bytes  | ~15 decimal digits | `var x float64 = 3.14159`  |
| string  | Variable | Text               | `var x = "Hello"`          |
| bool    | 1 byte   | true/false         | `var x = true`             |

---

## ✅ Best Practices

```go
// ✅ Use int for most cases (platform optimal)
var count int = 100

// ✅ Use float64 over float32 (more precision)
var price float64 = 19.99

// ✅ Always check errors in conversion
num, err := strconv.Atoi("abc")
if err != nil {
    fmt.Println("Conversion error!")
}

// ✅ Use strings package for string operations
import "strings"
```

---

## 🎯 Practice

<div dir="rtl">

**Task:** Create a program that:

1. Asks for name and age (as strings)
2. Converts age to int
3. Checks if adult (>= 18)
4. Prints result

</div>

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    var name, ageStr string

    fmt.Print("Enter your name: ")
    fmt.Scan(&name)

    fmt.Print("Enter your age: ")
    fmt.Scan(&ageStr)

    age, err := strconv.Atoi(ageStr)
    if err != nil {
        fmt.Println("Invalid age!")
        return
    }

    isAdult := age >= 18

    fmt.Printf("\nName: %s\n", name)
    fmt.Printf("Age: %d\n", age)
    fmt.Printf("Adult: %t\n", isAdult)
}
```

---

## ⏭️ Next Module

<div dir="rtl">

**➡️ [Module 2.4: Control Flow](../04-control-flow/README.md)**

</div>

---

<div align="center">

[⬅️ Previous: Syntax Basics](../02-syntax-basics/README.md) | [🏠 Track 2](../README.md)

</div>
