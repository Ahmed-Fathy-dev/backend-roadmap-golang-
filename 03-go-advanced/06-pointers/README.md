# Module 3.6: Pointers 👉

<div dir="rtl">

## نظرة عامة

**Pointers** = عناوين الذاكرة - أساسية لفهم Go!

</div>

---

## 📚 Pointer Basics

```go
// Regular variable
x := 10

// Pointer to x
var p *int = &x  // & = address-of operator
fmt.Println(p)   // 0xc000014070 (memory address)

// Dereference pointer
fmt.Println(*p)  // 10 (* = dereference operator)

// Modify via pointer
*p = 20
fmt.Println(x)   // 20 (original changed!)
```

---

## 🆚 Pass by Value vs Reference

```go
// Pass by value (copy)
func doubleValue(x int) {
    x *= 2  // Only modifies copy
}

// Pass by pointer (reference)
func doublePointer(x *int) {
    *x *= 2  // Modifies original!
}

func main() {
    num := 10

    doubleValue(num)
    fmt.Println(num)  // Still 10

    doublePointer(&num)
    fmt.Println(num)  // Now 20
}
```

---

## 🎯 Practical Examples

### Example 1: Swap Function

```go
func swap(a, b *int) {
    *a, *b = *b, *a
}

func main() {
    x, y := 5, 10
    swap(&x, &y)
    fmt.Println(x, y)  // 10, 5
}
```

### Example 2: Modifying Struct

```go
type Person struct {
    Name string
    Age  int
}

func updateAge(p *Person, age int) {
    p.Age = age  // Modifies original
}

func main() {
    person := Person{Name: "Ahmed", Age: 25}
    updateAge(&person, 26)
    fmt.Println(person.Age)  // 26
}
```

---

## ⚠️ nil Pointers

```go
var p *int
fmt.Println(p)  // nil

// Dereferencing nil pointer = panic!
// fmt.Println(*p)  // ❌ Runtime panic!

// Always check nil
if p != nil {
    fmt.Println(*p)
}

// Create pointer with new
p = new(int)  // Allocates memory
*p = 42
fmt.Println(*p)  // 42
```

---

## ✅ When to Use Pointers

```go
// ✅ To modify function arguments
func updateValue(x *int) {
    *x = 100
}

// ✅ With large structs (avoid copying)
type LargeStruct struct {
    Data [1000000]int
}

func process(s *LargeStruct) {  // Pointer to avoid copy
    // ...
}

// ✅ To return "optional" values
func findUser(id int) *User {
    // ...
    if notFound {
        return nil  // Indicates not found
    }
    return &user
}
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ `&` = أخذ العنوان (address-of)
- ✅ `*` = dereference (الوصول للقيمة)
- ✅ Pointers للتعديل والتجنب copying
- ✅ دائماً افحص `nil` قبل الاستخدام
- ✅ Go handles most pointer details automatically

</div>

---

## 🎉 Track 3 Complete!

<div dir="rtl">

**تهانينا!** أنهيت Track 3 🚀

**تعلمت:**

- ✅ Arrays & Slices - dynamic data
- ✅ Maps - key-value pairs
- ✅ Structs - custom types
- ✅ Methods - OOP in Go
- ✅ Interfaces - polymorphism
- ✅ Pointers - memory & performance

**جاهز للتطبيق العملي؟**
**➡️ Track 4: PostgreSQL & Database Integration**

</div>

---

<div align="center">

[⬅️ Previous: Interfaces](../05-interfaces/README.md) | [🏠 Track 3](../README.md)

</div>
