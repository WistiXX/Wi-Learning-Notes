`io` 包是 Go 语言标准库中用于处理 I/O(输入/输出)操作的核心包，它提供了一系列接口和函数，用于抽象和统一各种 I/O 操作。

## 核心接口

### 1. `io.Reader` 接口
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

- 表示可读取数据的来源
    
- 每次调用 `Read()` 方法会读取数据到字节切片 `p` 中
    
- 返回读取的字节数和可能的错误
    
- 当数据读取完毕时返回 `io.EOF` 错误
    
```
### 2. `io.Writer` 接口

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}

- 表示可写入数据的目标
    
- 将字节切片 `p` 中的数据写入目标
    
- 返回实际写入的字节数和可能的错误
    
```
### 3. `io.Closer` 接口

```go
type Closer interface {
    Close() error
}

- 表示可关闭的资源
    
- 通常用于文件、网络连接等需要显式关闭的资源
    
```
### 4. `io.Seeker` 接口

```go
type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}

- 表示可随机访问的数据源
    
- `whence` 可以是 `io.SeekStart`、`io.SeekCurrent` 或 `io.SeekEnd`
    
```
### 5. 组合接口

```go
type ReadWriter interface {
    Reader
    Writer
}

type ReadCloser interface {
    Reader
    Closer
}

type WriteCloser interface {
    Writer
    Closer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

type ReadSeeker interface {
    Reader
    Seeker
}

type WriteSeeker interface {
    Writer
    Seeker
}

type ReadWriteSeeker interface {
    Reader
    Writer
    Seeker
}
```
## 常用函数

### 1. 数据拷贝

```go
func Copy(dst Writer, src Reader) (written int64, err error)

- 从 `src` 读取数据并写入 `dst`，直到遇到 EOF 或错误
    
- 返回拷贝的字节数和可能的错误
    
```

```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)

- 从 `src` 拷贝指定字节数 `n` 到 `dst`
    
```
### 2. 数据读取

```go
func ReadAll(r Reader) ([]byte, error)

- 从 `r` 读取所有数据直到 EOF
    
- 返回读取的数据和可能的错误
    
```

```go
func ReadAtLeast(r Reader, buf []byte, min int) (n int, err error)

- 从 `r` 读取至少 `min` 字节到 `buf`
    
- 如果读取的字节数不足 `min` 且没有错误，返回 `io.ErrUnexpectedEOF`
    
```

```go
func ReadFull(r Reader, buf []byte) (n int, err error)

- 从 `r` 读取恰好 `len(buf)` 字节到 `buf`
    
- 相当于 `ReadAtLeast(r, buf, len(buf))`
    
```
### 3. 实用工具

```go
func Pipe() (*PipeReader, *PipeWriter)

- 创建同步的内存管道
    
- 适用于需要将 `io.Reader` 接口连接到 `io.Writer` 接口的场景
    
```

```go
func MultiReader(readers ...Reader) Reader

- 创建一个逻辑串联的 `Reader`
    
- 按顺序从多个 `Reader` 读取数据
    
```

```go
func MultiWriter(writers ...Writer) Writer

- 创建一个将数据写入多个 `Writer` 的 `Writer`
    
- 写入操作会同时写入所有提供的 `Writer`
    
```
## 常用实现

### 1. `bytes.Buffer`

- 实现了 `Reader`、`Writer` 等接口
    
- 可变大小的字节缓冲区
    

### 2. `strings.Reader`

- 实现了 `Reader`、`Seeker` 等接口
    
- 从字符串读取数据
    

### 3. `os.File`

- 实现了 `Reader`、`Writer`、`Seeker`、`Closer` 等接口
    
- 文件操作
    

### 4. `bufio.Reader`/`bufio.Writer`

- 带缓冲的 I/O
    
- 减少系统调用次数，提高性能
    

## 使用示例

### 1. 基本读写
```go
// 从 Reader 读取数据
data := make([]byte, 1024)
n, err := r.Read(data)
if err != nil {
    if err != io.EOF {
        log.Fatal(err)
    }
}
fmt.Printf("Read %d bytes: %q\n", n, data[:n])

// 向 Writer 写入数据
n, err := w.Write([]byte("Hello, World!"))
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Wrote %d bytes\n", n)
```
### 2. 文件拷贝
```go
srcFile, err := os.Open("source.txt")
if err != nil {
    log.Fatal(err)
}
defer srcFile.Close()

dstFile, err := os.Create("destination.txt")
if err != nil {
    log.Fatal(err)
}
defer dstFile.Close()

bytesCopied, err := io.Copy(dstFile, srcFile)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Copied %d bytes\n", bytesCopied)
```
### 3. 使用管道
```go
pr, pw := io.Pipe()

go func() {
    defer pw.Close()
    pw.Write([]byte("Hello, Pipe!"))
}()

data, err := io.ReadAll(pr)
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(data)) // 输出: Hello, Pipe!
```
## 总结

`io` 包提供了一套简洁而强大的 I/O 抽象接口，使得 Go 程序可以以统一的方式处理各种 I/O 操作，无论是文件、网络连接、内存缓冲区还是其他自定义的数据源和目标。通过组合这些基本接口，可以构建出复杂的 I/O 处理逻辑，同时保持代码的清晰和可维护性。