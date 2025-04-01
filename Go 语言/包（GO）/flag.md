**`flag` 包是 Go 标准库中用于解析命令行参数的模块**，专门用来处理程序启动时通过 `-flag=value` 形式传入的参数。以下是它的核心功能解析：

---

### 📌 **核心用途**

1. **定义参数**  
    自动解析 `-name=Alice` 或 `--port=8080` 这类命令行输入。
    
2. **生成帮助信息**  
    自动支持 `-h` 或 `--help` 显示参数说明。
    
3. **类型安全**  
    直接支持 `string`、`int`、`bool` 等类型，避免手动转换。
    

---

### 🎯 **基础使用场景**
```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	// 1. 定义参数
	port := flag.Int("port", 8080, "服务监听端口") // -port=数值
	debug := flag.Bool("debug", false, "启用调试模式") // -debug

	// 2. 解析命令行
	flag.Parse()

	// 3. 使用参数
	fmt.Printf("Server running on port %d (debug: %t)\n", *port, *debug)
}
```
**运行效果**：
```bash
go run main.go -port=9000 -debug
# 输出: Server running on port 9000 (debug: true)
```
---

### 🔧 **关键函数速查**

|函数|作用|示例|
|---|---|---|
|`flag.String()`|定义字符串参数|`-user=root`|
|`flag.Int()`|定义整数参数|`-port=8080`|
|`flag.Bool()`|定义布尔参数|`-verbose=true`|
|`flag.Parse()`|触发实际参数解析|必须调用|
|`flag.Args()`|获取非标志参数|`./app file.txt`|

---
#### **1. `flag.String()` / `flag.Int()` / `flag.Bool()`**

#### 📌 作用

定义命令行参数，并返回参数的 **指针**（`*string` / `*int` / `*bool`）。

#### 🔧 函数签名
```go
func String(标志名称, 默认值, 帮助信息) *string //以下相同
func Int(name string, value int, usage string) *int  
func Bool(name string, value bool, usage string) *bool
```
#### 参数说明

|参数|类型|说明|
|---|---|---|
|`name`|string|标志名称（如 `"port"` 对应 `-port=8080`）|
|`value`|对应类型|默认值（未提供参数时使用）|
|`usage`|string|帮助信息（`-h` 时显示）|

#### 🌰 示例
```go
port := flag.Int("port", 8080, "服务监听端口") // -port=8080
name := flag.String("name", "Guest", "用户名")  // -name=Alice
debug := flag.Bool("debug", false, "启用调试")  // -debug=true
```
---

### **2. `flag.StringVar()` / `flag.IntVar()` / `flag.BoolVar()`**

#### 📌 作用

将命令行参数绑定到 **已存在的变量**（避免使用指针）。

#### 🔧 函数签名
```go
func StringVar(p *string, name string, value string, usage string)
func IntVar(p *int, name string, value int, usage string)
func BoolVar(p *bool, name string, value bool, usage string)
```
#### 🌰 示例
```go
var port int
var name string
flag.IntVar(&port, "port", 8080, "端口号")  // 绑定到 port 变量
flag.StringVar(&name, "name", "Guest", "用户名")
```
---

### **3. `flag.Parse()`**

#### 📌 作用

**解析命令行参数**，必须在所有 `flag.Xxx()` 定义后调用。

#### ⚠️ 注意

- 未调用时，所有参数保持默认值。
    
- 解析失败时会自动调用 `os.Exit(2)` 终止程序。
    

#### 🌰 示例
```go
flag.Parse() // 必须调用！
```
---

### **4. `flag.Args()`**

#### 📌 作用

获取 **非标志参数**（即不带 `-` 的参数）。

#### 🌰 示例
```go
files := flag.Args() // ["file1.txt", "file2.txt"]
```

```bash
go run main.go -port=8080 file1.txt file2.txt
```
---

### **5. `flag.Usage`**

#### 📌 作用

**自定义帮助信息**（覆盖默认的 `-h` 输出）。

#### 🌰 示例
```go
flag.Usage = func() {
    fmt.Fprintf(os.Stderr, "用法: %s [选项] <文件>\n", os.Args[0])
    flag.PrintDefaults()
}
```
---

### **📚 完整代码示例**
```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	// 定义参数
	port := flag.Int("port", 8080, "服务端口")
	name := flag.String("name", "Guest", "用户名")
	debug := flag.Bool("debug", false, "调试模式")

	// 自定义帮助信息
	flag.Usage = func() {
		fmt.Printf("用法: %s -port=8080 -name=Alice\n", os.Args[0])
		flag.PrintDefaults()
	}

	// 解析参数
	flag.Parse()

	// 使用参数
	fmt.Printf("端口: %d, 用户: %s, 调试: %t\n", *port, *name, *debug)

	// 非标志参数
	if args := flag.Args(); len(args) > 0 {
		fmt.Println("其他参数:", args)
	}
}
```
***
### 🌰 **实际案例：CLI 工具**
```go
// 文件处理工具示例
func main() {
	input := flag.String("i", "", "输入文件路径")
	output := flag.String("o", "out.txt", "输出文件路径")
	flag.Parse()

	if *input == "" {
		flag.Usage() // 显示帮助信息
		return
	}
	fmt.Printf("Processing %s → %s\n", *input, *output)
}
```
**帮助信息自动生成**：
```go
go run main.go -h
Usage:
  -i string
        输入文件路径
  -o string
        输出文件路径 (default "out.txt")
```
---

### ⚠️ **注意事项**

1. **指针返回值**  
    `flag.String()` 返回 `*string`，使用时需解引用（如 `*port`）。
    
2. **必须调用 Parse**  
    参数定义后必须执行 `flag.Parse()` 才会生效。
    
3. **参数顺序自由**  
    `-a=1 -b=2` 和 `-b=2 -a=1` 等效。
    

---

### 💡 **进阶技巧**

1. **自定义标志名**
```go
var port int
flag.IntVar(&port, "p", 8080, "端口号") // 绑定到变量
```
2. **子命令支持**  
    通过 `flag.NewFlagSet()` 实现类似 `git commit` 的子命令。
---

📌 **一句话总结**：`flag` 包让命令行参数处理变得简单规范，是构建 CLI 工具（如 `docker`、`kubectl`）的基础组件。需要处理复杂参数时，可考虑第三方库如 `cobra`。