### **1. 什么是JSON？**

- **全称**：JavaScript Object Notation
    
- **本质**：一种**轻量级的数据交换格式**，用于在不同系统间传输和存储结构化数据。
    
- **核心特点**：
    
    - 文本格式（纯字符串）
        
    - 独立于编程语言（几乎所有语言都支持）
        
    - 易读、易写、易解析
        

---

### **2. JSON 的核心结构**

#### **基本元素**

1. **键值对**：`"键": 值`
    
    - 键必须用**双引号**包裹
        
    - 值可以是：字符串、数字、布尔值（`true`/`false`）、数组、对象、`null`
	```json
    "name": "张三",
    "age": 30
	```
2. **对象**（Object）：用 `{}` 包裹，表示一组无序的键值对集合。
    ```json
    {
      "name": "李四",
      "isStudent": true
    }
	```
3. **数组**（Array）：用 `[]` 包裹，表示有序的值集合，元素可以是任意类型。
    ```json
    "hobbies": ["篮球", "音乐", "旅行"]
	```

---

### **3. JSON 完整示例**
```json
{
  "user": {
    "id": 1001,
    "name": "小明",
    "scores": [85, 92, 78],
    "address": {
      "city": "上海",
      "postcode": "200000"
    }
  },
  "isActive": true
}
```
---

### **4. JSON 的常见用途**

#### **(1) 前后端数据交互（Web API）**

- 前端发送请求 → 后端返回 JSON 数据
    ```json
    // 后端返回的用户数据示例
    {
      "code": 200,
      "data": {
        "userId": "U1001",
        "username": "xiaoming"
      }
    }
	```

#### **(2) 配置文件**

- 软件设置（如 VSCode 的 `settings.json`、npm 的 `package.json`）
    ```json
    // package.json 示例（定义项目依赖）
    {
      "name": "my-project",
      "version": "1.0.0",
      "dependencies": {
        "react": "^18.2.0"
      }
    }
	```

#### **(3) 数据库存储**

- NoSQL 数据库（如 MongoDB）直接存储 JSON 格式数据。
    
- 灵活存储非结构化数据（如用户表单、日志）。
    

---

### **5. JSON 与其他格式的对比**

#### **JSON vs XML**

|**特性**|**JSON**|**XML**|
|---|---|---|
|**数据量**|更小（无冗余标签）|更大（标签重复）|
|**可读性**|高（结构简洁）|较低（标签嵌套复杂）|
|**解析速度**|快（直接转为数据结构）|慢（需 DOM 解析）|
|**典型应用**|Web API、配置文件|旧式 Web 服务（SOAP）、文档标记|

#### **JSON vs Markdown**

|**特性**|**JSON**|**Markdown**|
|---|---|---|
|**设计目的**|存储和传输数据（给程序用）|编写格式化文档（给人看）|
|**语法重点**|严格的键值对、引号、逗号|符号标记（`#`、`*`、`>`）|
|**容错性**|低（格式错误直接报错）|高（排版错误仍可阅读）|

---

### **6. 注意事项（常见错误）**

1. **键必须用双引号**
    ```json
    { "name": "张三" }  // ✅  
    { name: "张三" }    // ❌  
	```
2. **末尾不能有多余逗号**
    ```json
    { "a": 1, "b": 2 }       // ✅  
    { "a": 1, "b": 2, }      // ❌  
	```
3. **不支持注释**
    ```json
    {
      "name": "张三",  // 这是注释（JSON不支持！）
      "age": 30
    }
	```
4. **字符串必须用双引号**
    ```json
    { "city": "北京" }  // ✅  
    { 'city': '北京' }  // ❌  
	```

---

### **7. 在代码中操作 JSON**

#### **JavaScript 示例**

javascript

```json
// 1. 将对象转为JSON字符串（序列化）
const user = { name: "李雷", age: 25 };
const jsonStr = JSON.stringify(user); 
// 结果：'{"name":"李雷","age":25}'

// 2. 将JSON字符串转为对象（反序列化）
const jsonStrFromServer = '{"name":"韩梅梅","age":24}';
const userObj = JSON.parse(jsonStrFromServer);
console.log(userObj.name); // 输出：韩梅梅
```
#### **Python 示例**

python

```json
import json

# 将字典转为JSON字符串
data = {"name": "王五", "score": 90}
json_str = json.dumps(data)  # '{"name": "\\u738b\\u4e94", "score": 90}'

# 将JSON字符串转为字典
json_str_from_api = '{"city": "广州", "population": 1500}'
data_dict = json.loads(json_str_from_api)
print(data_dict["city"])  # 输出：广州
```
---

### **8. 总结：JSON 的核心价值**

- **跨平台**：几乎所有编程语言和系统都支持。
    
- **高效**：体积小、解析快，适合网络传输。
    
- **灵活**：可嵌套对象和数组，适应复杂数据结构。