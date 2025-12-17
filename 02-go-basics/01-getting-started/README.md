# Module 2.1: Getting Started with Go 🎬

<div dir="rtl">

## نظرة عامة

في هذا الدرس، سنبدأ رحلتنا مع Go! سنقوم بتثبيت Go، إعداد بيئة العمل، وكتابة أول برنامج.

</div>

---

## 📖 Content

### 1. Installing Go

<div dir="rtl">

#### Windows:

1. اذهب إلى [https://go.dev/dl/](https://go.dev/dl/)
2. حمّل المثبت لـ Windows (`.msi`)
3. شغّل المثبت واتبع الخطوات
4. تحقق من التثبيت:

</div>

```powershell
go version
# Should output: go version go1.22.x windows/amd64
```

<div dir="rtl">

#### Linux/Mac:

</div>

```bash
# Download and extract
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin

# Verify
go version
```

---

### 2. Setting Up Your Workspace

<div dir="rtl">

#### إعداد GOPATH (اختياري في Go Modules):

Go Modules هي الطريقة الحديثة (لا تحتاج GOPATH)، لكن من الجيد معرفتها:

</div>

```bash
# Check current GOPATH
go env GOPATH

# On Windows usually: C:\Users\YourName\go
# On Linux/Mac usually: ~/go
```

<div dir="rtl">

#### هikal المشروع الموصى به:

</div>

```
my-go-project/
├── go.mod          # Module definition
├── go.sum          # Dependencies checksums
├── main.go         # Entry point
├── handlers/       # Your packages
│   └── user.go
└── models/
    └── user.go
```

---

### 3. Your First Go Program

<div dir="rtl">

#### Step 1: إنشاء مجلد المشروع:

</div>

```powershell
mkdir hello-go
cd hello-go
```

<div dir="rtl">

#### Step 2: Initialize Go Module:

</div>

```powershell
go mod init hello-go
# Creates go.mod file
```

<div dir="rtl">

#### Step 3: إنشاء `main.go`:

</div>

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, Go!")
}
```

<div dir="rtl">

**شرح الكود:**

</div>

```go
package main                    // Every Go file starts with package declaration
                               // "main" package = executable program

import "fmt"                   // Import fmt package for formatted I/O
                               // fmt = format

func main() {                  // main function = entry point of program
	fmt.Println("Hello, Go!")  // Print to console
}                              // Println = Print Line (adds newline)
```

<div dir="rtl">

#### Step 4: تشغيل البرنامج:

</div>

```powershell
go run main.go
# Output: Hello, Go!
```

<div dir="rtl">

#### Step 5: Build executable:

</div>

```powershell
go build
# Creates: hello-go.exe (Windows) or hello-go (Linux/Mac)

# Run it:
./hello-go      # Linux/Mac
.\hello-go.exe  # Windows
```

---

### 4. Go Modules

<div dir="rtl">

**Go Modules** هي نظام إدارة Dependencies في Go.

#### ما هو `go.mod`؟

</div>

```go
module hello-go           // Module name

go 1.22                  // Go version

// Dependencies (will be added automatically)
require (
    github.com/gin-gonic/gin v1.9.1
)
```

<div dir="rtl">

#### أوامر Module الأساسية:

</div>

```bash
# Initialize module
go mod init module-name

# Add dependency
go get github.com/gin-gonic/gin

# Remove unused dependencies
go mod tidy

# Download dependencies
go mod download

# Verify dependencies
go mod verify
```

---

### 5. Go Tooling

<div dir="rtl">

Go تأتي بأدوات قوية مدمجة:

#### `go run` - تشغيل مباشر:

</div>

```bash
go run main.go
```

<div dir="rtl">

#### `go build` - بناء executable:

</div>

```bash
go build              # Creates executable in current dir
go build -o myapp     # Custom name
```

<div dir="rtl">

#### `go test` - تشغيل الاختبارات:

</div>

```bash
go test              # Run tests in current package
go test ./...        # Run all tests
go test -v           # Verbose output
```

<div dir="rtl">

#### `go fmt` - تنسيق الكود:

</div>

```bash
go fmt main.go       # Format single file
go fmt ./...         # Format all files
```

<div dir="rtl">

#### `go vet` - فحص الأخطاء المحتملة:

</div>

```bash
go vet main.go
go vet ./...
```

<div dir="rtl">

#### `go doc` - عرض الdocumentation:

</div>

```bash
go doc fmt.Println
go doc -all fmt
```

---

### 6. VS Code Setup

<div dir="rtl">

#### Step 1: تثبيت Go Extension:

1. افتح VS Code
2. Extensions (Ctrl+Shift+X)
3. ابحث عن "Go"
4. ثبّت الـ Extension الرسمي من Go Team

#### Step 2: Install Go Tools:

عند فتح ملف `.go` لأول مرة، ستظهر رسالة:

- اضغط "Install All"

أو يدوياً:

</div>

```bash
# Install useful tools
go install golang.org/x/tools/gopls@latest
go install github.com/go-delve/delve/cmd/dlv@latest
```

<div dir="rtl">

#### Step 3: Recommended VS Code Settings:

أضف إلى `settings.json`:

</div>

```json
{
  "go.useLanguageServer": true,
  "go.formatTool": "gofmt",
  "go.lintTool": "golangci-lint",
  "editor.formatOnSave": true,
  "go.testOnSave": false
}
```

---

### 7. Package System

<div dir="rtl">

#### ما هو Package؟

Package هو مجموعة من ملفات Go في نفس المجلد.

#### أنواع Packages:

**1. Executable Package (main):**

</div>

```go
package main

func main() {
    // Entry point
}
```

<div dir="rtl">

**2. Library Package (أي اسم آخر):**

</div>

```go
package mathutils

func Add(a, b int) int {
    return a + b
}
```

<div dir="rtl">

#### استيراد Packages:

**Standard Library:**

</div>

```go
import "fmt"
import "net/http"
import "encoding/json"
```

<div dir="rtl">

**External Packages:**

</div>

```go
import "github.com/gin-gonic/gin"
```

<div dir="rtl">

**Your Own Packages:**

</div>

```
myproject/
├── main.go
└── utils/
    └── helper.go
```

```go
// main.go
package main

import "myproject/utils"

func main() {
    utils.SomeFunction()
}
```

<div dir="rtl">

#### Multiple Imports:

</div>

```go
import (
    "fmt"
    "net/http"
    "github.com/gin-gonic/gin"
)
```

---

### 8. Exported vs Unexported

<div dir="rtl">

قاعدة مهمة جداً في Go:

</div>

| Case          | Visibility                                | Example         |
| ------------- | ----------------------------------------- | --------------- |
| **Uppercase** | <div dir="rtl">Exported (public)</div>    | `Add()`, `User` |
| **lowercase** | <div dir="rtl">Unexported (private)</div> | `add()`, `user` |

```go
package mathutils

// Exported function (can be used from other packages)
func Add(a, b int) int {
    return a + b
}

// Unexported function (only within this package)
func subtract(a, b int) int {
    return a - b
}
```

<div dir="rtl">

**من package آخر:**

</div>

```go
import "myproject/mathutils"

mathutils.Add(5, 3)       // ✅ Works
mathutils.subtract(5, 3)  // ❌ Error: unexported
```

---

### 9. Common Go Commands

```bash
# Run without building
go run main.go

# Build executable
go build

# Build for different OS
GOOS=linux go build
GOOS=windows go build
GOOS=darwin go build    # Mac

# Install to $GOPATH/bin
go install

# Get dependency
go get github.com/package/name

# Update dependencies
go get -u

# Clean build cache
go clean

# Show environment
go env

# Format code
go fmt ./...

# Run tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Generate coverage report
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

---

## 💡 Key Takeaways

<div dir="rtl">

- ✅ Go بسيطة وسريعة وقوية
- ✅ `package main` + `func main()` = نقطة البداية
- ✅ `go run` للتشغيل المباشر، `go build` للـ executable
- ✅ Go Modules لإدارة Dependencies
- ✅ Uppercase = Exported, lowercase = Unexported
- ✅ Go تأتي بأدوات مدمجة قوية (fmt, test, vet, etc.)

</div>

---

## 🎯 Practice

<div dir="rtl">

### تمرين 1: Hello World مخصص

اكتب برنامج يطبع:

```
مرحباً، اسمي [اسمك]
أنا أتعلم Go!
```

### تمرين 2: Multiple Files

أنشئ:

- `main.go`: يستدعي function من package آخر
- `greetings/greet.go`: يحتوي على function `SayHello(name string)`

### تمرين 3: Build

Build برنامجك لـ:

- Windows
- Linux
- Mac

</div>

---

## 📚 Additional Resources

- 📖 [Official Go Tour](https://go.dev/tour/)
- 📖 [Go by Example - Hello World](https://gobyexample.com/hello-world)
- 📺 [Go in 100 Seconds](https://www.youtube.com/watch?v=446E-r0rXHI)

---

## ⏭️ Next Module

<div dir="rtl">

الآن بعد أن أعددت بيئة العمل، دعنا نتعلم أساسيات اللغة:

**➡️ [Module 2.2: Core Language Features](../02-core-features/README.md)**

</div>

---

<div align="center">

[🏠 Track 2 Home](../README.md) | [📚 Main](../../README.md)

</div>
