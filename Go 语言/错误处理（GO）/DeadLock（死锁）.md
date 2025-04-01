**死锁（Deadlock）** 是并发编程中一个经典问题，尤其在多线程/多协程场景下容易发生。它指的是 **两个或多个执行单元（如 goroutine）互相等待对方释放资源，导致所有单元永久阻塞，程序无法继续执行**。

---

## **1. 死锁的四个必要条件**

死锁的发生必须同时满足以下四个条件（缺一不可）：

|条件|解释|Go 示例|
|---|---|---|
|**互斥条件**|资源一次只能被一个 goroutine 占用（如锁、无缓冲通道）。|`var mu sync.Mutex`（锁是独占的）|
|**占有并等待**|goroutine 持有资源的同时，还在等待其他资源。|`goroutine1` 锁住 `A`，同时想拿 `B`；`goroutine2` 锁住 `B`，同时想拿 `A`。|
|**非抢占条件**|goroutine 已占用的资源不能被强制剥夺，必须主动释放。|Go 的 `Mutex` 不会自动解锁，必须手动 `Unlock()`。|
|**循环等待条件**|多个 goroutine 形成资源依赖的环形链。|`A 等 B → B 等 C → C 等 A`|

**只要破坏任意一个条件，死锁就不会发生！**

---

## **2. Go 中常见的死锁场景**

### **(1) 无缓冲通道的互相阻塞**
```go
func main() {
    ch := make(chan int) // 无缓冲通道
    ch <- 1              // 发送阻塞（没有接收方）
    fmt.Println(<-ch)    // 接收阻塞（永远执行不到这里）
}
// 报错：fatal error: all goroutines are asleep - deadlock!
```
**原因**：

- 主 goroutine 发送 `ch <- 1` 时，没有其他 goroutine 接收，导致永久阻塞。
    
- Go 检测到所有 goroutine 都阻塞，直接报死锁。
    

**修复**：用 goroutine 异步接收或改用缓冲通道。
```go
go func() { fmt.Println(<-ch) }() // 启动接收方
ch <- 1                           // 发送成功
```
---
### **(2) 锁的嵌套导致互相等待**
```go
var muA, muB sync.Mutex

func goroutine1() {
    muA.Lock()
    defer muA.Unlock()
    time.Sleep(time.Second) // 模拟耗时操作
    muB.Lock()              // 等待 goroutine2 释放 muB
    defer muB.Unlock()
}

func goroutine2() {
    muB.Lock()
    defer muB.Unlock()
    time.Sleep(time.Second)
    muA.Lock()              // 等待 goroutine1 释放 muA
    defer muA.Unlock()
}

func main() {
    go goroutine1()
    go goroutine2()
    time.Sleep(5 * time.Second) // 死锁发生！
}
```
**原因**：

- `goroutine1` 持有 `muA`，等待 `muB`；
    
- `goroutine2` 持有 `muB`，等待 `muA`；
    
- 形成**循环等待**，两者永远阻塞。
    

**修复**：保证锁的获取顺序一致（如总是先 `muA` 后 `muB`）。

---

### **(3) WaitGroup 使用不当**
```go
func main() {
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        wg.Wait()  // 阻塞等待 Done()，但永远不会执行
    }()
    wg.Wait()      // 主 goroutine 也阻塞
}
// 死锁：所有 goroutine 都在 Wait()
```
**修复**：确保 `wg.Done()` 能被调用。
```go
go func() {
    defer wg.Done() // 正确释放
    // do work
}()
```
---
## **3. 如何避免死锁？**

### **(1) 超时机制**

用 `select` + `time.After` 避免无限阻塞：
```go
select {
case <-ch:
    // 收到数据
case <-time.After(1 * time.Second):
    fmt.Println("超时！")
}
```
### **(2) 按固定顺序获取锁**

所有 goroutine 必须按相同顺序请求锁（如总是先 `A` 后 `B`）。

### **(3) 使用缓冲通道**

避免无缓冲通道的同步阻塞：
```go
ch := make(chan int, 1) // 缓冲大小为1
ch <- 1                 // 不会阻塞
```
### **(4) 工具检测**

- Go 的运行时能检测到**所有 goroutine 都阻塞**的情况，直接报 `deadlock`。
    
- 第三方工具如 [go-deadlock](https://github.com/sasha-s/go-deadlock) 可以提前预警。
    

---

## **4. 一句话总结**

死锁就是 **“你等我，我等你，大家一起卡死”**。  
在 Go 中，**无缓冲通道、锁、WaitGroup 是死锁高发区**，务必注意资源释放顺序和异步设计！