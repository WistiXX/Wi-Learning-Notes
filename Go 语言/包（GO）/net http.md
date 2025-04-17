### **`net/http` 包详解**

**📌 核心作用**：提供 **HTTP 客户端和服务端的实现**，用于构建 Web 服务、发送 HTTP 请求或处理 API 接口。

---

## **🔧 核心功能与函数**

### **1. HTTP 服务端（Server）**

#### **关键函数/类型**

|函数/类型|作用|返回值/说明|
|---|---|---|
|`http.ListenAndServe(addr string, handler Handler)`|启动 HTTP 服务器|`error`（服务终止时返回错误）|
|`http.HandleFunc(pattern string, handler func(w ResponseWriter, r *Request))`|注册路由处理函数|无返回值|
|`http.Server`|自定义服务器配置|需调用 `server.ListenAndServe()`|
|`http.ResponseWriter`|写入 HTTP 响应|方法：`Write([]byte)`、`WriteHeader(statusCode)`|
|`http.Request`|封装 HTTP 请求信息|字段：`Method`、`URL`、`Header`、`Body`|

#### **示例：基础服务器**

go

复制

package main

import (
	"fmt"
	"net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %s!", r.URL.Query().Get("name"))
}

func main() {
	http.HandleFunc("/hello", helloHandler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Server error:", err)
	}
}

**访问**：

bash

复制

curl "http://localhost:8080/hello?name=Alice"
# 输出: Hello, Alice!

---

### **2. HTTP 客户端（Client）**

#### **关键函数/类型**

|函数/类型|作用|返回值/说明|
|---|---|---|
|`http.Get(url string)`|发送 GET 请求|`(*http.Response, error)`|
|`http.Post(url, contentType string, body io.Reader)`|发送 POST 请求|`(*http.Response, error)`|
|`http.Client`|自定义客户端（超时、重定向等）|需调用 `client.Do(req)`|
|`http.NewRequest(method, url string, body io.Reader)`|创建请求对象|`(*http.Request, error)`|
|`http.Response`|封装 HTTP 响应|字段：`StatusCode`、`Body`、`Header`|

#### **示例：发送 GET 请求**

go

复制

resp, err := http.Get("https://api.example.com/data")
if err != nil {
    panic(err)
}
defer resp.Body.Close() // 必须关闭 Body

body, _ := io.ReadAll(resp.Body)
fmt.Println("响应状态码:", resp.StatusCode)
fmt.Println("响应体:", string(body))

---

### **3. 路由与中间件**

#### **`http.ServeMux`（多路由处理）**

go

复制

mux := http.NewServeMux()
mux.HandleFunc("/api", func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("API Response"))
})
http.ListenAndServe(":8080", mux)

#### **中间件示例**

go

复制

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Printf("%s %s\n", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}
mux := http.NewServeMux()
mux.Handle("/", loggingMiddleware(http.HandlerFunc(helloHandler)))

---

## **📌 核心返回值详解**

### **1. `http.Response`（客户端响应）**

|字段/方法|类型/返回值|说明|
|---|---|---|
|`StatusCode`|`int`|HTTP 状态码（如 200、404）|
|`Body`|`io.ReadCloser`|响应体（需手动读取并关闭）|
|`Header`|`http.Header`|响应头部（如 `Content-Type`）|
|`Body.Close()`|`error`|关闭响应体（防止资源泄漏）|

### **2. `http.Request`（服务端请求）**

|字段/方法|类型/返回值|说明|
|---|---|---|
|`Method`|`string`|HTTP 方法（如 `GET`、`POST`）|
|`URL`|`*url.URL`|请求路径和查询参数|
|`Header`|`http.Header`|请求头部（如 `User-Agent`）|
|`Body`|`io.ReadCloser`|请求体（需手动读取并关闭）|

---

## **⚠️ 注意事项**

1. **资源释放**：
    
    - 必须调用 `resp.Body.Close()`（客户端）和 `r.Body.Close()`（服务端）。
        
2. **错误处理**：
    
    - 检查 `err`（如网络错误、JSON 解析失败）。
        
3. **性能优化**：
    
    - 复用 `http.Client`（避免重复创建）：
        
        go
        
        复制
        
        client := &http.Client{Timeout: 10 * time.Second}
        resp, _ := client.Get(url)
        

---

## **💡 进阶技巧**

### **1. 文件服务器**

go

复制

fs := http.FileServer(http.Dir("./static"))
http.Handle("/static/", http.StripPrefix("/static/", fs))

### **2. 解析 JSON 请求体**

go

复制

type RequestData struct {
    Name string `json:"name"`
}
var data RequestData
json.NewDecoder(r.Body).Decode(&data)

### **3. 自定义超时**

go

复制

server := &http.Server{
    Addr:         ":8080",
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
}
server.ListenAndServe()

---

## **🚀 总结**

- **服务端**：`http.HandleFunc` + `ListenAndServe`。
    
- **客户端**：`http.Get`/`Post` + 处理 `Response`。
    
- **核心返回值**：
    
    - `Response`：`StatusCode`、`Body`、`Header`。
        
    - `Request`：`Method`、`URL`、`Body`。
        
- **最佳实践**：
    
    - 使用中间件处理通用逻辑（如日志、鉴权）。
        
    - 生产环境配置超时和连接池。
        

需要 WebSocket 或 RESTful API 支持时，可结合 `gorilla/websocket` 或 `gin` 框架。