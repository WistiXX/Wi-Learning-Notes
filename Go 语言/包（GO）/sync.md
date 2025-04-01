Go语言的`sync`包提供了基础的同步原语，用于协调多个goroutine之间的并发操作。以下是其主要组件及详细说明：

---

### 1. **Mutex（互斥锁）**

- **用途**：保护临界区资源，确保同一时刻仅一个goroutine访问共享数据。
    
- **方法**：
    
    - `Lock()`：获取锁，若锁被其他goroutine持有，则阻塞。
        
    - `Unlock()`：释放锁，未持有锁时调用会引发panic。
        
- **示例**：
```go
    var counter int
    var mu sync.Mutex
    
    func increment() {
        mu.Lock()
        defer mu.Unlock()
        counter++
    }
```
- **注意事项**：
    
    - 使用`defer`确保解锁，避免死锁。
        
    - 不可复制Mutex（如作为函数参数传递时需用指针）。
        

---

### 2. **RWMutex（读写锁）**

- **用途**：读多写少场景，允许多个读操作或单个写操作。
    
- **方法**：
    
    - `RLock()`：获取读锁（共享）。
        
    - `RUnlock()`：释放读锁。
        
    - `Lock()`：获取写锁（独占）。
        
    - `Unlock()`：释放写锁。
        
- **特点**：
    
    - 写锁优先级高：等待的写锁会阻塞后续读锁，避免写饥饿。
        
- **示例**：
```go
    var cache map[string]string
    var rwmu sync.RWMutex
    
    func read(key string) string {
        rwmu.RLock()
        defer rwmu.RUnlock()
        return cache[key]
    }
```
---

### 3. **WaitGroup（等待组）**

- **用途**：等待一组goroutine完成。
    
- **方法**：
    
    - `Add(delta int)`：增加计数器。
        
    - `Done()`：计数器减1（等价于`Add(-1)`）。
        
    - `Wait()`：阻塞直到计数器归零。
        
- **示例**：
```go
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            // 任务代码
        }()
    }
    wg.Wait()
```
- **注意事项**：
    
    - `Add`需在启动goroutine前调用。
        
    - 计数器不能为负数（否则panic）。
        

---

### 4. **Cond（条件变量）**

- **用途**：在条件满足时唤醒等待的goroutine。
    
- **方法**：
    
    - `Wait()`：释放锁并阻塞，被唤醒后重新获取锁。
        
    - `Signal()`：唤醒一个等待的goroutine。
        
    - `Broadcast()`：唤醒所有等待的goroutine。
        
- **示例**（生产者-消费者模型）：
```go
    var mu sync.Mutex
    cond := sync.NewCond(&mu)
    var data []interface{}
    
    // 生产者
    cond.L.Lock()
    data = append(data, newItem)
    cond.Signal() // 或 Broadcast()
    cond.L.Unlock()
    
    // 消费者
    cond.L.Lock()
    for len(data) == 0 {
        cond.Wait() // 自动释放锁，唤醒后重新获取
    }
    item := data[0]
    data = data[1:]
    cond.L.Unlock()
```
---

### 5. **Once（单次执行）**

- **用途**：确保函数仅执行一次（如初始化）。
    
- **方法**：
    
    - `Do(f func())`：无论调用多少次，`f`仅执行一次。
        
- **示例**：
```go
    var once sync.Once
    var config map[string]string
    
    func loadConfig() {
        once.Do(func() {
            config = readConfigFile()
        })
    }
```
- **注意事项**：
    
    - 若`f`发生panic，后续调用仍视为已完成。
        

---

### 6. **Pool（临时对象池）**

- **用途**：缓存可复用对象，减少内存分配。
    
- **方法**：
    
    - `Get() interface{}`：从池中获取对象（若池为空则调用`New`函数）。
        
    - `Put(x interface{})`：将对象放回池中。
        
- **示例**（缓冲区复用）：
```go
    var pool = sync.Pool{
        New: func() interface{} { return bytes.NewBuffer(nil) },
    }
    
    buf := pool.Get().(*bytes.Buffer)
    buf.Reset()
    defer pool.Put(buf)
    buf.WriteString("data")
```
- **注意事项**：
    
    - 池中对象可能随时被回收，不保证长期存在。
        
    - 取出后需重置对象状态。
        

---

### 7. **Map（并发安全字典）**

- **用途**：高并发读写的线程安全map，适合读多写少场景。
    
- **方法**：
    
    - `Store(key, value interface{})`
        
    - `Load(key interface{}) (value interface{}, ok bool)`
        
    - `Delete(key interface{})`
        
    - `Range(func(key, value interface{}) bool)`：遍历所有键值对。
        
- **示例**：
```go
    var sm sync.Map
    sm.Store("key", "value")
    if v, ok := sm.Load("key"); ok {
        fmt.Println(v)
    }
```
- **特点**：
    
    - 比`Mutex + map`性能更优的读操作。
        
    - 写操作较少时更高效。
        

---

### 注意事项与最佳实践

1. **锁的粒度**：尽量缩小临界区，减少锁持有时间。
    
2. **避免死锁**：按固定顺序获取锁，避免嵌套锁。
    
3. **竞态检测**：使用`go run -race`检测数据竞争。
    
4. **优先使用Channel**：在简单场景下，Channel可能更符合Go的并发哲学。
    
5. **不要复制同步对象**：如Mutex、WaitGroup等，传递时使用指针。