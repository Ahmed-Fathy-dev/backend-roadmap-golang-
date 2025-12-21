# Module 3.3: Structs 🏗️

<div dir="rtl">

## نظرة عامة

**Structs** = طريقة Go لإنشاء custom data types.

</div>

---

## 📚 Basic Structs

### Defining Structs

```go
// Define struct type
type Person struct {
    Name string
    Age  int
    Email string
}

// Create instance
person := Person{
    Name:  "Ahmed",
    Age:   25,
    Email: "ahmed@test.com",
}

// Access fields
fmt.Println(person.Name)  // Ahmed
person.Age = 26           // Modify
```

---

## 🎯 Advanced Features

### Nested Structs

```go
type Address struct {
    Street string
    City   string
}

type Employee struct {
    Name    string
    Age     int
    Address Address  // Nested
}

emp := Employee{
    Name: "Ahmed",
    Age:  25,
    Address: Address{
        Street: "123 Main St",
        City:   "Cairo",
    },
}

fmt.Println(emp.Address.City)  // Cairo
```

### Struct Tags

```go
type User struct {
    ID       int    `json:"id" db:"user_id"`
    Name     string `json:"name" db:"full_name"`
    Password string `json:"-"`  // Ignore in JSON
}

// Tags used by json.Marshal/Unmarshal
user := User{ID: 1, Name: "Ahmed", Password: "secret"}
jsonData, _ := json.Marshal(user)
// {"id":1,"name":"Ahmed"}  // Password excluded!
```

---

## 🔧 Complete Example

```go
package main

import "fmt"

type Product struct {
    ID      int
    Name    string
    Price   float64
    InStock bool
}

func (p Product) Display() {
    fmt.Printf("Product: %s\n", p.Name)
    fmt.Printf("Price: $%.2f\n", p.Price)
    status := "In Stock"
    if !p.InStock {
        status = "Out of Stock"
    }
    fmt.Printf("Status: %s\n", status)
}

func main() {
    product := Product{
        ID:      1,
        Name:    "Laptop",
        Price:   999.99,
        InStock: true,
    }

    product.Display()
}
```

---

## ✅ Best Practices

```go
// ✅ Use meaningful names
type User struct {
    Name  string  // Good
    Email string
}

// ✅ Use tags for JSON/DB mapping
type User struct {
    ID   int    `json:"id" db:"user_id"`
    Name string `json:"name"`
}

// ✅ Make unexported if internal
type internalConfig struct {  // lowercase = private
    secret string
}
```

---

## ⏭️ Next Module

**➡️ [Module 3.4: Methods](../04-methods/README.md)**

---

<div align="center">

[⬅️ Previous: Maps](../02-maps/README.md) | [🏠 Track 3](../README.md)

</div>
