`os` 包提供了与操作系统交互的核心功能，包括文件操作、环境变量、进程管理等。以下是其核心功能的详细整理：

---
### **1. 文件操作**
#### **1.1 打开/创建文件**
- **`os.Open(name string) (*os.File, error)`**  
    打开文件（只读模式），失败返回错误（如文件不存在）。
    ```go
    file, err := os.Open("test.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close() // 确保关闭文件
    ```
- **`os.Create(name string) (*os.File, error)`**  
    创建文件（若存在则清空），权限默认 `0666`。
    ```go
    file, err := os.Create("newfile.txt")
    ```
#### **1.2 读写文件**
- **`file.Read(b []byte) (n int, err error)`**  
    从文件读取数据到字节切片。
    ```go
    data := make([]byte, 100)
    n, err := file.Read(data)
    ```
- **`file.Write(b []byte) (n int, err error)`**  
    将字节切片写入文件。
    ```go
    _, err = file.Write([]byte("Hello, Go!"))
    ```
- **`os.WriteFile(name string, data []byte, perm os.FileMode)`**  
    直接写入文件（覆盖模式）。
    ```go
    err := os.WriteFile("output.txt", []byte("data"), 0644)
    ```
#### **1.3 文件信息与权限**
- **`os.Stat(name string) (os.FileInfo, error)`**  
    获取文件信息（大小、权限、修改时间等）。
    ```go
    info, err := os.Stat("test.txt")
    fmt.Println("Size:", info.Size())
    fmt.Println("ModTime:", info.ModTime())
    ```
- **`os.Chmod(name string, mode os.FileMode)`**  
    修改文件权限。
    ```go
    os.Chmod("test.txt", 0755) // 设置可执行权限
    ```
#### **1.4 其他操作**
- **`os.Remove(name string) error`**  
    删除文件或空目录。
    ```go
    err := os.Remove("temp.txt")
    ```
- **`os.Rename(oldpath, newpath string) error`**  
    重命名或移动文件。
    ```go
    os.Rename("old.txt", "new.txt")
    ```
---
### **2. 目录操作**
#### **2.1 创建目录**
- **`os.Mkdir(name string, perm os.FileMode) error`**  
    创建单级目录。
    ```go
    err := os.Mkdir("mydir", 0755)
    ```
- **`os.MkdirAll(path string, perm os.FileMode) error`**  
    递归创建多级目录。
    ```go
    err := os.MkdirAll("parent/child", 0755)
    ```
#### **2.2 读取目录**
- **`os.ReadDir(name string) ([]os.DirEntry, error)`**  
    读取目录内容（返回 `DirEntry` 切片）。
    ```go    entries, err := os.ReadDir(".")
    for _, entry := range entries {
        fmt.Println(entry.Name(), entry.IsDir())
    }
    ```
#### **2.3 删除目录**
- **`os.RemoveAll(path string) error`**  
    递归删除目录及其内容。
    ```go
    err := os.RemoveAll("tempdir")
    ```
---
### **3. 环境变量**
- **`os.Getenv(key string) string`**  
    获取环境变量值。
    ```go
    path := os.Getenv("PATH")
    ```
- **`os.Setenv(key, value string) error`**  
    设置环境变量（仅当前进程有效）。
    ```go
    os.Setenv("MY_VAR", "123")
    ```
- **`os.Environ() []string`**  
    获取所有环境变量（格式为 `KEY=value`）。
    ```go
    for _, env := range os.Environ() {
        fmt.Println(env)
    }
    ```
---
### **4. 进程管理**
#### **4.1 进程信息**
- **`os.Getpid() int`**  
    获取当前进程ID。
    ```go
    fmt.Println("PID:", os.Getpid())
    ```
- **`os.Exit(code int)`**  
    终止当前进程（状态码 `0` 表示成功）。
    ```go
    os.Exit(0)
    ```
#### **4.2 执行外部命令**
- **`os.StartProcess`**  
    启动外部进程（复杂场景使用）。
- **推荐替代方案**：使用 `exec.Command`（来自 `os/exec` 包）。
    ```go
    cmd := exec.Command("ls", "-l")
    output, err := cmd.Output()
    ```
---
### **5. 错误处理**
- **`os.IsNotExist(err error) bool`**  
    检查错误是否为“文件不存在”。
    ```go
    if os.IsNotExist(err) {
        fmt.Println("文件不存在")
    }
    ```
- **`os.IsPermission(err error) bool`**  
    检查错误是否为权限不足。
    ```go
    if os.IsPermission(err) {
        fmt.Println("权限拒绝")
    }
    ```
---
### **6. 跨平台注意事项**

1. **路径分隔符**
    - 使用 `os.PathSeparator`（`/` 或 `\`）或 `filepath` 包处理路径。
    - 推荐使用 `filepath.Join` 构建跨平台路径：
        ```go
        path := filepath.Join("dir", "subdir", "file.txt")
        ```
2. **换行符**
    - Windows 使用 `\r\n`，Unix 使用 `\n`。
3. **文件权限**
    - Unix 权限模式（如 `0644`）在 Windows 下可能无效。
---
### **7. 常见应用场景**
- **配置文件读取**：结合 `os.Open` 和 `json`/`yaml` 解析。
- **日志记录**：使用 `os.Create` 或 `os.OpenFile` 写入日志文件。
- **命令行工具**：通过 `os.Args` 获取命令行参数。
	```go
    args := os.Args
    if len(args) > 1 {
        fmt.Println("第一个参数:", args[1])
    }
    ```
---
### **8. 完整示例**
```go
package main

import (
    "fmt"
    "log"
    "os"
)

func main() {
    // 创建文件并写入数据
    file, err := os.Create("demo.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()
    file.WriteString("Hello, OS Package!\n")

    // 读取文件内容
    data, err := os.ReadFile("demo.txt")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("文件内容: %s", data)

    // 删除文件
    os.Remove("demo.txt")
}
```
---
### **总结**

- **核心功能**：文件、目录、环境变量、进程管理。
    
- **关键注意点**：错误处理、资源释放（如 `defer file.Close()`）、跨平台兼容性。
    
- **适用场景**：系统工具开发、配置文件处理、日志系统等。