用于延迟执行一个函数调用，**在当前函数返回前**（无论是否 panic）执行，常用于资源清理（如关闭文件、解锁）

通俗点说就是：
* 无论写在函数哪一行（开头、中间、甚至在嵌套在if里）
* 无论函数如何结束（自然执行完毕、`return`退出、`panic`崩溃）

**defer都会在函数结束前最后一刻执行，且保证执行（除非程序强行终止）**
***
## 核心特征
1. **执行时机**：
    - 在 `return` **后**，但返回值**被返回前**执行（可修改命名返回值）。
    - 若发生 `panic`，`defer` 仍会执行（用于 recover）。
2. **栈顺序**：**后进先出**（LIFO），最后一个 `defer` 最先执行。
3. **参数预计算**：`defer` 的参数在**定义时**立即求值，而非调用时。
***
## 典型用法
```go
// 1. 关闭文件（确保资源释放）
func readFile() error {
    f, err := os.Open("file.txt")
    if err != nil {
        return err
    }
    defer f.Close()  // 无论后续代码是否报错都会执行
    // ...处理文件...
}

// 2. 修改命名返回值
func count() (n int) {
    defer func() { n *= 2 }()  // n=1 → return 前变为 2
    return 1
}

// 3. 捕获 panic
func safeCall() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered:", r)
        }
    }()
    panic("oops!")
}
```