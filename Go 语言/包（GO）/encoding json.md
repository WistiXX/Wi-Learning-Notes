### **`encoding/json` 包详解**

**📌 核心作用**：实现 **JSON 数据与 Go 结构体之间的相互转换**，支持序列化（`Marshal`）与反序列化（`Unmarshal`）。

---

## **🔧 常用函数/方法**

|函数/方法|作用|示例场景|
|---|---|---|
|`json.Marshal(v)`|Go 结构体 → JSON 字节|API 响应数据格式化|
|`json.Unmarshal(data, &v)`|JSON 字节 → Go 结构体|解析请求体|
|`json.NewEncoder(w)`|流式写入 JSON 到 `io.Writer`|HTTP 响应流|
|`json.NewDecoder(r)`|从 `io.Reader` 流式读取 JSON|解析大 JSON 文件|

---

## **📚 核心功能详解**

### **1. 序列化（Go → JSON）**

#### **`json.Marshal(v interface{}) ([]byte, error)`**

将 Go 结构体转换为 JSON 格式的字节切片。

**示例**：
```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name string `json:"name"`  // 字段标签控制 JSON 键名
	Age  int    `json:"age"`
}

func main() {
	user := User{Name: "Alice", Age: 25}
	jsonData, _ := json.Marshal(user)
	fmt.Println(string(jsonData))
}
```
**输出**：
```json
{"name":"Alice","age":25}
```
**关键点**：

- 结构体字段首字母必须**大写**（公开字段）才能被序列化。
    
- 通过 `json:"xxx"` 标签自定义 JSON 字段名。

---

### **2. 反序列化（JSON → Go）**

#### **`json.Unmarshal(data []byte, v interface{}) error`**

将 JSON 数据解析到 Go 结构体。

**示例**：
```go
jsonStr := `{"name":"Bob","age":30}`
var user User
err := json.Unmarshal([]byte(jsonStr), &user)
if err != nil {
    panic(err)
}
fmt.Printf("%+v", user)  // 输出: {Name:Bob Age:30}
```
**注意事项**：

- 目标结构体字段需与 JSON 键名匹配（不区分大小写）。
    
- 未知字段会被忽略。
    

---

### **3. 流式处理（大文件友好）**

#### **编码到 `io.Writer`**
```go
file, _ := os.Create("user.json")
defer file.Close()

encoder := json.NewEncoder(file)
encoder.Encode(User{Name: "Charlie"})  // 直接写入文件
```
#### **从 `io.Reader` 解码**
```go
file, _ := os.Open("user.json")
defer file.Close()

var user User
decoder := json.NewDecoder(file)
decoder.Decode(&user)  // 从文件流式读取
```
---

## **⚠️ 常见问题与技巧**

### **1. 字段控制**

- **忽略空字段**：
    
    ```go
    type User struct {
        Name string `json:"name,omitempty"`  // 如果 Name 为空，则跳过该字段
    }
    ```
- **忽略字段**：
    
    ```go
    Password string `json:"-"`  // 不序列化此字段
    ```

### **2. 处理特殊类型**

- **时间类型**：
    
    ```go
    type Log struct {
        Time time.Time `json:"time"`  // 默认转为 RFC3339 格式字符串
    }
    ```
- **自定义序列化**：  
    实现 `json.Marshaler` 接口：
    
    ```go
    func (u User) MarshalJSON() ([]byte, error) {
        return []byte(`{"custom":"format"}`), nil
    }
    ```

### **3. 性能优化**

- **预分配缓冲区**（高频调用时）：
    
    ```go
    var buf bytes.Buffer
    encoder := json.NewEncoder(&buf)
    encoder.Encode(data)
    ```

---

## **💡 进阶应用**

### **1. 动态 JSON（`map[string]interface{}`）**
```go
var data map[string]interface{}
jsonStr := `{"name":"Alice","details":{"age":25}}`
json.Unmarshal([]byte(jsonStr), &data)
fmt.Println(data["details"].(map[string]interface{})["age"])  // 输出: 25
```
### **2. 美化输出（缩进格式化）**
```go
jsonData, _ := json.MarshalIndent(user, "", "  ")  // 两个空格缩进
fmt.Println(string(jsonData))
```
**输出**：
```json
{
  "name": "Alice",
  "age": 25
}
```
---

## **📌 总结**

- **序列化**：`json.Marshal`（结构体 → JSON）。
    
- **反序列化**：`json.Unmarshal`（JSON → 结构体）。
    
- **流式处理**：`NewEncoder`/`NewDecoder` 适合大文件。
    
- **字段控制**：通过结构体标签灵活配置。
    

遇到复杂 JSON 结构时，可结合 `json.RawMessage` 延迟解析部分字段。