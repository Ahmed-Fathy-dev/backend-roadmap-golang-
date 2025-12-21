# Module 2.1: Installation & Setup ⚙️

<div dir="rtl">

## نظرة عامة

في هذا Module، سنثبت Go ونجهز بيئة التطوير.

</div>

---

## 📋 What You'll Learn

<div dir="rtl">

1. تثبيت Go على نظامك
2. إعداد VS Code
3. كتابة أول برنامج Go
4. فهم Go Modules
5. استخدام Go commands

</div>

---

## 🔧 Installing Go

### Windows:

1. **Download:**

   - Go to [go.dev/dl](https://go.dev/dl/)
   - Download latest version (1.22+)
   - Run installer

2. **Verify Installation:**

```powershell
go version
# Output: go version go1.22.0 windows/amd64
```

### Linux/Mac:

```bash
# Download and install
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Add to PATH
export PATH=$PATH:/usr/local/go/bin

# Verify
go version
```

---

## 💻 Setting Up VS Code

### 1. Install VS Code

- Download from [code.visualstudio.com](https://code.visualstudio.com/)

### 2. Install Go Extension

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search "Go"
4. Install official Go extension by Google

### 3. Install Go Tools

```powershell
# VS Code will prompt to install tools
# Or manually:
go install golang.org/x/tools/gopls@latest
go install github.com/go-delve/delve/cmd/dlv@latest
```

---

## 🎯 Your First Go Program

### Create Project:

```powershell
# Create directory
mkdir hello-go
cd hello-go

# Initialize Go module
go mod init hello-go

# Create main.go
```

### `main.go`:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go! 🚀")
}
```

### Run:

```powershell
go run main.go
# Output: Hello, Go! 🚀
```

---

## 📦 Go Modules

<div dir="rtl">

**Go Modules** = نظام إدارة Dependencies في Go.

</div>

### Initialize Module:

```powershell
go mod init myproject
```

Creates `go.mod`:

```go
module myproject

go 1.22
```

### Add Dependencies:

```powershell
# Install package
go get github.com/gin-gonic/gin

# go.mod automatically updated
```

---

## 🔨 Essential Go Commands

```powershell
# Run program
go run main.go

# Build executable
go build
# Creates: hello-go.exe (Windows) or hello-go (Linux/Mac)

# Run executable
./hello-go

# Format code
go fmt ./...

# Download dependencies
go mod download

# Tidy dependencies (remove unused)
go mod tidy

# Run tests
go test ./...

# Install package
go install
```

---

## 📂 Project Structure

```
hello-go/
├── go.mod          # Module definition
├── go.sum          # Dependency checksums
├── main.go         # Main entry point
└── README.md
```

---

## ✅ Checklist

<div dir="rtl">

- [ ] ✅ Go installed (`go version` works)
- [ ] ✅ VS Code + Go extension installed
- [ ] ✅ Created first Go program
- [ ] ✅ `go run` works
- [ ] ✅ `go build` creates executable
- [ ] ✅ Understanding Go modules basics

</div>

---

## 🎯 Practice Exercise

<div dir="rtl">

**Task:** Create a program that prints your name and age.

**Expected Output:**

```
My name is Ahmed
I am 25 years old
```

**Solution:**

</div>

```go
package main

import "fmt"

func main() {
    name := "Ahmed"
    age := 25

    fmt.Printf("My name is %s\n", name)
    fmt.Printf("I am %d years old\n", age)
}
```

---

## ⏭️ Next Module

<div dir="rtl">

جاهز للتعمق في Go؟

**➡️ [Module 2.2: Syntax Basics](../02-syntax-basics/README.md)**

</div>

---

<div align="center">

[🏠 Track 2 Home](../README.md) | [📚 Main](../../README.md)

</div>
