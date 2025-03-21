高级特性部分有点难，所以就先搁置去先学其他内容了
***
# 一、 函数定义
## 1. 基本格式
```go
func 函数名 (参数列表) 返回值类型 {
	函数体
}
```

```go
//例：加法函数
func add(n1 int, n2 int) int {
	return n1 + n2
}
```
* **参数**：输入进函数的数据
* **返回值**：输出的结果
	* 可选的，同上方的hello例子
	* 可以返回多个值
```go
func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("1.周杰伦", "2.陶喆")
   fmt.Println(a, b)
}

//输出
2.陶喆 1.周杰伦"
```

***
## 2. 调用
```go
变量名 := 函数名(参数列表)
```
以上面的加法函数为例
```
sum := add(3,5,1)
```
**输出为：`9`**
# 二、 函数参数
## 1. 值传递
**函数默认使用值传递。** 传递到函数的参数是原有参数的拷贝，所以在调用过程中不会影响原有参数
## 2. 引用传递
将参数的地址传递到函数中，函数调用过程中对参数做修改时会影响到原有参数
```go
//例子：交换值函数。通过指针使用了引用传递
func swap(x, y *int) {
   var temp int
   temp = *x    /* 保存 x 地址上的值 */
   *x = *y      /* 将 y 值赋给 x */
   *y = temp    /* 将 temp 值赋给 y */
}

func main() {
   var a int = 100
   var b int= 200

   fmt.Printf("交换前，a 的值 : %d\n", a )
   fmt.Printf("交换前，b 的值 : %d\n", b )

   /* 调用 swap() 函数
   * &a 指向 a 指针，a 变量的地址
   * &b 指向 b 指针，b 变量的地址
   */
   swap(&a, &b)

   fmt.Printf("\n")

   fmt.Printf("交换后，a 的值 : %d\n", a )
   fmt.Printf("交换后，b 的值 : %d\n", b )
}

//输出为：
交换前，a 的值 : 100
交换前，b 的值 : 200

交换后，a 的值 : 200
交换后，b 的值 : 100
```
# 三、函数高级特性

## 1. 可变函数
让函数可以接受**任意数量**的同类型参数

在参数类型前加`...`，让他变成一个切片
```go
// 例子：计算任意数量数字的和
func Sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

// 调用
fmt.Println(Sum(1, 2, 3))       //输出为：6
fmt.Println(Sum([]int{4, 5, 6}...)) // 切片展开
```
## 2. 函数作为值
在go中，函数是**一等公民**，可以像变量一样被赋值，传递和返回，并且无需特殊处理
### **函数类型声明**
```go
type 类型名 func(参数类型列表) 返回值类型
```
***
### **一、函数赋值给变量**
```go
//声明了一个函数类型，接收两个int，返回int
type cal func(int int) int

// 定义加法函数
func Add(a, b int) int {
    return a + b
}

// 将 Add 函数赋值给 Cal（上面声明的函数类型）类型的变量v1
var v1 Cal = Add

// 调用
result := v1(3, 5) // result = 8
```
#### **匿名函数直接赋值**
```go
// 无需提前定义函数，直接将cal类型的变量mult赋值
var Mult Cal = func(x, y int) int {
    return x * y
}

fmt.Println(Multiply(2, 4)) // 8
```
***
### **二、函数作为参数（回调函数）**
***
### **三、函数作为返回值**
***
## 3. 闭包
