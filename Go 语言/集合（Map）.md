无序键值对的集合
# 一、基本操作

### 声明
```go
map名 := map[key类型]value类型  //必须初始化才能使用
```
### 初始化
```go
// 使用 make 初始化
m := make(map[string]int) //make初始化的map是非nil的，但内部没有任何键值对，可以进行动态补充

// 直接初始化（字面量）
m := map[string]int{
    "apple":  5,
    "banana": 3,
}
```
***
## 增删改查
### 添加/修改元素
```go
m["orange"] = 4 // 添加新键值对
m["apple"] = 10  // 修改已有键的值
```
### 获取元素
```go
变量名 := m["apple"] // 若键不存在，返回值类型的零值（如 int 返回 0）
```
### 删除元素
使用`delete`
```go
delete(目标map, 键) //若键不存在则无操作
```

```go
delete(m, "banana")
```
### 查询键是否存在
如果键存在，会返回`true`，否则就返回`false`
```go
value, ok := m["apple"] //ok是变量名，没有实际意义
if ok {
    fmt.Println("存在，值为:", value)
} else {
    fmt.Println("不存在")
}
```
`value, ok := m["apple"]`使用了**双返回值**
* **第一个值**是键对应的值。在这里是`5`，返回给`value`
* **第二个值**是布尔类型。在这里是`true`，返回给`ok`
***
# 二、特性与注意事项
## 1. 无序性
`map`的遍历顺序是随机的
## 2. 零值（nil map）
未初始化的`map`是`nil`，直接写入会导致panic[^1]：
```go
var m map[string]int
m["key"] = 1 // panic: assignment to nil map
```
## 3.  键类型的限制
* 键的类型必须支持`==`操作（如基本类型，结构体，数组等）
* 不可用类型：切片，集合，函数等不可比较的类型
***
# 三、高级用法
## 1. 遍历map
```go
for key, value := range m {
    fmt.Println(key, "->", value)
}
```
## 2. 值类型是复杂结构
值可以是任意类型，例如`map`嵌套
```go
//第一个map[string]的值类型为map[string]，第二个map[string]的值类型为int
m:= map[string]map[string]int{
    "fruit": {"apple": 5},
    "drink": {"coffee": 2},
}
```
## 3. 作为函数参数
`map`是引用类型，函数内修改会影响原数据
```go
func function(m map[string]int) {
    m["new"] = 100
}
```
***
# 示例代码
```go
package main

import "fmt"

func main() {
    // 初始化
    m := map[string]int{"apple": 5, "banana": 3}
    
    // 添加/修改
    m["orange"] = 4
    m["apple"] = 10
    
    // 删除
    delete(m, "banana")
    
    // 遍历
    for k, v := range m {
        fmt.Printf("%s: %d\n", k, v)
    }
    
    // 检查存在性
    if val, ok := m["pear"]; ok {
        fmt.Println("pear exists:", val)
    } else {
        fmt.Println("pear does not exist")
    }
}
```

[^1]: 表示程序发生了无法恢复的错误或异常情况，需要立即终止程序的执行。
