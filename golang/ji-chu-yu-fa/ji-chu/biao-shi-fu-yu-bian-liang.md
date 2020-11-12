# 1.变量

## 1**. 变量声明**

### 1.1 变量声明

Go变量声明的三种方式：

```go
var a int            // 声明一个变量，默认为0
var b = 10            // 声明并初始化，且自动推导类型
c := 20                // 初始化，且自动推导
```

注意：

* `:=`定义变量只能在函数内部使用，所以经常用var定义全局变量
* Go对已经声明但未使用的变量会在编译阶段报错：`** not used`
* Go中的标识符以字母或者下划线开头，大小写敏感
* Go推荐使用驼峰命名 

### **1.2 多变量声明**

```go
var a,b string
var a1,b1 string = "哼","哈"
var a2,b2 int = 1,2                             //类型可以直接省略
c,d := 1,2
var(
    e int
    f bool
)
```

### **1.3 变量值互换**

```go
m,n = n,m        //变量值互换
temp,_ = m,n        //匿名变量：变量值互换，且丢弃变量n
```

### **1.4 \_丢弃变量**

`_`是个特殊的变量名，任何赋予它的值都会被丢弃。该变量不占用命名空间，也不会分配内存。

```go
_, b := 34, 35      //将值`35`赋予`b`，并同时丢弃`34`：
```

### **1.5 := 声明的注意事项**

下面是正确的代码示例：

```go
in, err := os.Open(file)
out, err := os.Create(file)     // err已经在上方定义，此处的 err其实是赋值
```

但是如果在第二行赋值的变量名全部和第一行一致，则编译不通过：

```go
in, err := os.Open(file)
in, err := os.Create(file)     // 即 := 必须确保至少有一个变量是用于声明
```

`:=`只有对已经在同级词法域声明过的变量才和赋值操作语句等价，如果变量是在外部词法域声明的，那么`:=`将会在当前词法域重新声明一个新的变量。

### **1.6 多数据分组书写**

Go可以使用该方式声明多个数据：

```go
const(
    i = 100
    pi = 3.1415
    prefix = "Go_"
)

 var(
    i int
    pi float32
    prefix string
)
```



**Go对于已声明但未使用的变量会在编译阶段报错**

```go
package main

// 比如下面的代码就会产生一个错误：声明了i但未使用:
func main() {
    var i int
}
```

## **2. 数据类型**

### **2.1 内置基础类型\#**

Go 语言按类别有以下几种数据类型：

* 布尔型 --&gt; 在Go中，布尔值的类型为`bool`,值只可以是常量 true 或者 false。一个简单的例子：var b bool = true；
* 整数型 --&gt; 整型 int 和浮点型 float，Go 语言支持整型和浮点型数字，并且原生支持复数，其中位的运算采用补码；
* 字符串 --&gt; 字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。Go语言的字符串的字节使用UTF-8编码标识Unicode文本；
* 派生型 --&gt;
  * \(a\) 指针类型（Pointer）
  * \(b\) 数组类型
  * \(c\) 结构化类型\(struct\)
  * \(d\) 联合体类型 \(union\)
  * \(e\) 函数类型
  * \(f \) 切片类型
  * \(g\) 接口类型（interface）
  * \(h\) Map 类型
  * \(i\) Channel 类型

### **2.2 Boolean**

```text
//示例代码
var isActive bool  // 全局变量声明
var enabled, disabled = true, false  // 忽略类型的声明
func test() {
    var available bool  // 一般声明
    valid := false      // 简短声明
    available = true    // 赋值操作
}
```

**数值类型**

整数类型有无符号和带符号两种。Go同时支持`int`和`uint`，这两种类型的长度相同，但具体长度取决于不同编译器的实现。Go里面也有直接定义好位数的类型：`rune`, `int8`, `int16`, `int32`, `int64`和`byte`, `uint8`, `uint16`, `uint32`, `uint64`。其中`rune`是`int32`的别称，`byte`是`uint8`的别称。

```go
// 需要注意的一点是，这些类型的变量之间不允许互相赋值或操作，不然会在编译时引起编译器报错。
// 如下的代码会产生错误：invalid operation: a + b (mismatched types int8 and int32)
var a int8
var b int32
c:=a + b
// 另外，尽管int的长度是32 bit, 但int 与 int32并不可以互用。
// 浮点数的类型有float32和float64两种（没有float类型），默认是float64。
```

Go还支持复数。它的默认类型是`complex128`（64位实数+64位虚数）。如果需要小一些的，也有`complex64`\(32位实数+32位虚数\)。复数的形式为`RE + IMi`，其中`RE`是实数部分，`IM`是虚数部分，而最后的`i`是虚数单位。下面是一个使用复数的例子：

```go
var c complex64 = 5+5i
//output: (5+5i)
fmt.Printf("Value is: %v", c)
```

\*\*\*\*

## 3. 关键字iota

这个关键字用来声明`enum`的时候采用，它默认开始值是0，const中每增加一行加1：

```go
package main


const (
	x = iota // x == 0
	y
	z
	w 
)

const (
	h, i, j = iota, iota, iota //h=0,i=0,j=0 iota在同一行值相同
)

const (
	l       = iota //a=0
	m       = "B"
	n       = iota             //2
	o, p, q = iota, iota, iota //3,3,3
	r       = iota             //4
)

func main()  {
	println(x,y,z,w)
	println(h,i,j)
	println(l,m,n,o,p,q,r)
}

结果：
0 1 2 3
0 0 0
0 B 2 3 3 3 4
```

