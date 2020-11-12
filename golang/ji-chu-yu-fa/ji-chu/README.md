# 基础和语法

**字符串**

Go中的字符串都是采用`UTF-8`字符集编码,字符串是用一对双引号（`""`）或反引号（）括起来定义，它的类型是 `string`。

```text
func test(a,b int) {
   str := "hello world"
   m := "haha"
   result := str + m
   println(result)
   multStr := `hello
         world`
   println(multStr)
}
```

**在Go中字符串是不可变的，例如下面的代码编译时会报错：cannot assign to s\[0\]**

```text
var s string = "hello"
s[0] = 'k'
```

如果你想要修改一个字符串怎么办呢：

```text
s := "hello"
c := []byte(s)
c[0] = 'c'
s1 := string(c)
println(s1)
```

Go中可以使用 + 操作符来连接两个字符串：

```text
str := "hello world"
m := "haha"
result := str + m
println(result)
```

如果要声明一个多行的字符串怎么办？可以通过\`\` 来声明：

```text
multStr := `hello
         world`go
println(multStr)
```

括起的字符串为Raw字符串，即字符串在代码中的形式就是打印时的形式，它没有字符转义，换行也将原样输出。例如本例中会输出：

```text
hello
    world
```

**错误类型**

Go内置有一个`error`类型，专门用来处理错误信息，Go的`package`里面还专门有一个包`errors`来处理错误：

```text
func error()  {
   e := errors.New("this is a error demo")
   if e != nil {
      println(e)
   }
}
```

**1.5 array，slice，map\#**

**array**

array就是数组，定义方式如下：

```text
var arr [n]type
```

在`[n]type`中，`n`表示数组的长度，`type`表示存储元素的类型。对数组的操作和其它语言类似，都是通过`[]`来进行读取或赋值：

```text
var arr [10]int  // 声明了一个int类型的数组
arr[0] = 1      // 数组下标是从0开始的
arr[1] = 2      // 赋值操作
```

**由于长度也是数组类型的一部分，因此`[3]int`与`[4]int`是不同的类型，数组也就不能改变长度**。数组之间的赋值是值的赋值，即当把一个数组作为参数传入函数的时候，传入的其实是该数组的副本，而不是它的指针。如果要使用指针，那么就需要用到后面介绍的`slice`类型了。

数组可以使用另一种`:=`来声明:

```text
a := [3]int{1, 2, 3} // 声明了一个长度为3的int数组

b := [10]int{1, 2, 3} // 声明了一个长度为10的int数组，其中前三个元素初始化为1、2、3，其它默认为0

c := [...]int{4, 5, 6} // 可以省略长度而采用`...`的方式，Go会自动根据元素个数来计算长度
```

Go支持嵌套数组，即多维数组。比如下面的代码就声明了一个二维数组：

```text

// 声明了一个二维数组，该数组以两个数组作为元素，其中每个数组中又有4个int类型的元素
doubleArray := [2][4]int{[4]int{1, 2, 3, 4}, [4]int{5, 6, 7, 8}}

// 上面的声明可以简化，直接忽略内部的类型
easyArray := [2][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}}
```

**注意： \[2\]\[4\] int表示的是一个int型数组，数组内有两个数组，每个数组有四个元素组成。**

**slice**

知道python数组的就知道slice，跟python的实现是一样的。

slice有一些简便的操作

* `slice`的默认开始位置是0，`ar[:n]`等价于`ar[0:n]`
* `slice`的第二个序列默认是数组的长度，`ar[n:]`等价于`ar[n:len(ar)]`
* 如果从一个数组里面直接获取`slice`，可以这样`ar[:]`，因为默认第一个序列是0，第二个是数组的长度，即等价于`ar[0:len(ar)]`

下面有一些关于slice的示例：

```text
// 声明一个数组
var array = [10]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
// 声明两个slice
var aSlice, bSlice []byte

// 演示一些简便操作
aSlice = array[:3] // 等价于aSlice = array[0:3] aSlice包含元素: a,b,c
aSlice = array[5:] // 等价于aSlice = array[5:10] aSlice包含元素: f,g,h,i,j
aSlice = array[:]  // 等价于aSlice = array[0:10] 这样aSlice包含了全部的元素

// 从slice中获取slice
aSlice = array[3:7]  // aSlice包含元素: d,e,f,g，len=4，cap=7
bSlice = aSlice[1:3] // bSlice 包含aSlice[1], aSlice[2] 也就是含有: e,f
bSlice = aSlice[:3]  // bSlice 包含 aSlice[0], aSlice[1], aSlice[2] 也就是含有: d,e,f
bSlice = aSlice[0:5] // 对slice的slice可以在cap范围内扩展，此时bSlice包含：d,e,f,g,h
bSlice = aSlice[:]   // bSlice包含所有aSlice的元素: d,e,f,g

```

**重要：`slice`是引用类型，所以当引用改变其中元素的值时，其它的所有引用都会改变该值，例如上面的`aSlice`和`bSlice`，如果修改了`aSlice`中元素的值，那么`bSlice`相对应的值也会改变。**

对于`slice`有几个有用的内置函数：

* `len` 获取`slice`的长度
* `cap` 获取`slice`的最大容量
* `append` 向`slice`里面追加一个或者多个元素，然后返回一个和`slice`一样类型的`slice`
* `copy` 函数`copy`从源`slice`的`src`中复制元素到目标`dst`，并且返回复制的元素的个数

```text
//1.基于数组创建数组切片
var array  = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
var slice = array[1:7] //array[startIndex:endIndex] 不包含endIndex
//2.直接创建数组切片
slice2 := make([]int, 5, 10)
//3.直接创建并初始化数组切片
slice3 := []int{1, 2, 3, 4, 5, 6}
//4.基于数组切片创建数组切片
slice5 := slice3[:4]
println(slice5)
//5.遍历数组切片
for i, v := range slice3 {
println(i, v)
}
//6.len()和cap()
var len = len(slice2) //数组切片的长度
var cap = cap(slice)  //数组切片的容量
println("len(slice2) =", len)
println("cap(slice) =", cap)
//7.append() 会生成新的数组切片
slice4 := append(slice2, 6, 7, 8)
slice4 = append(slice4, slice3...)
println(slice4)
//8.copy() 如果进行操作的两个数组切片元素个数不一致，将会按照个数较小的数组切片进行复制
copy(slice2, slice3) //将slice3的前五个元素复制给slice2
println(slice2, slice3)
```

**map**

map就是Java中的Map，python中的字典。它的格式为 `map[keyType]valueType` 。

```text
fruit := map[string]int{"apple":5,"orange":7,"pineapple":3}
println(fruit)
var appleCount = fruit["apple"]
println(appleCount)
```

使用map过程中需要注意的几点：

* `map`是无序的，每次打印出来的`map`都会不一样，它不能通过`index`获取，而必须通过`key`获取
* `map`的长度是不固定的，也就是和`slice`一样，也是一种引用类型
* 内置的`len`函数同样适用于`map`，返回`map`拥有的`key`的数量
* `map`的值可以很方便的修改，通过`numbers["one"]=11`可以很容易的把key为`one`的字典值改为`11`
* `map`和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex lock机制

`map`的初始化可以通过`key:val`的方式初始化值，同时`map`内置有判断是否存在`key`的方式

**1.6 make,new操作\#**

`make`用于内建类型（`map`、`slice` 和`channel`）的内存分配。`new`用于各种类型的内存分配。

内建函数`new`本质上说跟其它语言中的同名函数功能一样：`new(T)`分配了零值填充的`T`类型的内存空间，并且返回其地址，即一个`*T`类型的值。用Go的术语说，它返回了一个指针，指向新分配的类型`T`的零值。有一点非常重要：

> `new`返回指针。

内建函数`make(T, args)`与`new(T)`有着不同的功能，make只能创建`slice`、`map`和`channel`，并且返回一个有初始值\(非零\)的`T`类型，而不是`*T`。本质来讲，导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化。例如，一个`slice`，是一个包含指向数据（内部`array`）的指针、长度和容量的三项描述符；在这些项目被初始化之前，`slice`为`nil`。对于`slice`、`map`和`channel`来说，`make`初始化了内部的数据结构，填充适当的值。

> `make`返回初始化后的（非零）值。

对于不同的数据类型，零值的意义是完全不一样的。比如，对于bool类型，零值为false；int的零值为0；string的零值是空字符串：

```text
b := new(bool)
println(*b)
i := new(int)
println(*i)
s := new(string)
println(*s)

输出：
false
0

```

**注意：上面最后string的输出是空值。**

**1.7 零值\#**

关于“零值”，所指并非是空值，而是一种“变量未填充前”的默认值，通常为0。 此处罗列 部分类型 的 “零值”

```text
int     0
int8    0
int32   0
int64   0
uint    0x0
rune    0 //rune的实际类型是 int32
byte    0x0 // byte的实际类型是 uint8
float32 0 //长度为 4 byte
float64 0 //长度为 8 byte
bool    false
string  ""
```

**1.8 一些技巧\#**

1.分组声明

在Go语言中，同时声明多个常量、变量，或者导入多个包时，可采用分组的方式进行声明。

例如下面的代码：

```text
import	"log"
import	"net/http"
import	"strings"

const a = 3
const b = 2
const c = 4

var str = "aa"
var prefix = "abc_"
```

可以写成如下分组形式：

```text
import (
	"log"
	"net/http"
	"strings"
)

const(
	a = 2,
    b = 2,
    c = 4
)

var(
	str = "aa"
    prefix = "abc_"
)
```

**Go程序设计的一些规则**

Go之所以会那么简洁，是因为它有一些默认的行为：

* 大写字母开头的变量是可导出的，也就是其它包可以读取的，是公有变量；小写字母开头的就是不可导出的，是私有变量。
* 大写字母开头的函数也是一样，相当于`class`中的带`public`关键词的公有函数；小写字母开头的就是有`private`关键词的私有函数。

#### 2. 流程和函数[\#](https://www.cnblogs.com/rickiyang/p/11074184.html#3825115472) <a id="3825115472"></a>

Go中流程控制分三大类：条件判断，循环控制和无条件跳转。

**2.1 流程\#**

**if**

Go里面`if`条件判断语句中不需要括号，如下代码所示 ：

```text
if x > 80{
    println("better")
} else {
    println("good")
}

```

Go的`if`还有一个强大的地方就是条件判断语句里面允许声明一个变量，这个变量的作用域只能在该条件逻辑块内，其他地方就不起作用了，如下所示

```text
if countScore := getCountScore(); countScore >= 80 {
    println("better")
} else if countScore >= 60 {
    println("good")
} else {
    println("e......")
}

//这里打印countScore是找不到这个变量的
println(countScore)
```

**goto**

Go有`goto`语句——请明智地使用它。用`goto`跳转到必须在当前函数内定义的标签。例如假设这样一个循环：

```text
a := 4
b := 5
if a > b {
	println(a * b)
} else {
	Ding:
		a = a+12
		b = a + (32/4)
	if a < b {
		goto Ding
	}
}

```

**注意：标签名是大小写敏感的**

**for**

Go里面的for除了基本的循环外，还可以读取slice和map的数据：

```text
sum := 0
for i:=0;i<10;i++ {
	sum += i
}
```

`for`配合`range`可以用于读取`slice`和`map`的数据：

```text
fruit := map[string]int{"apple":5,"orange":7,"pineapple":3}
for k,v := range fruit{
    println(k,v)
}
```

小插曲：

**"\_"的使用**

由于 Go 支持 “多值返回”, 而对于“声明而未被调用”的变量, 编译器会报错, 在这种情况下, 可以使用`_`来丢弃不需要的返回值 例如

```text
fruit := map[string]int{"apple":5,"orange":7,"pineapple":3}
for _,v := range fruit{
    println(v)
}
```

"\_"相当于占位符的作用，这个位置必须要有一个值来接收，但是这个值又没有用，可以用 “ \_”来占着位置。

**switch**

跟别的语言中的switch别无他致：

```text
var sex byte
switch sex {
case 0:
	println("男生")
case 1:
	println("女生")
default:
	println("纳尼。。。。。")
}
```

**2.2 函数\#**

函数是Go里面的核心设计，它通过关键字`func`来声明，它的格式如下：

```text
func funcName(input1 type1, input2 type2) (output1 type1, output2 type2) {
    //这里是处理逻辑代码
    //返回多个值
    return value1, value2
}
```

上面的代码我们看出:

* 关键字`func`用来声明一个函数`funcName`
* 函数可以有一个或者多个参数，每个参数后面带有类型，通过`,`分隔
* 函数可以返回多个值
* 上面返回值声明了两个变量`output1`和`output2`，如果你不想声明也可以，直接就两个类型
* 如果只有一个返回值且不声明返回值变量，那么你可以省略 包括返回值 的括号
* 如果没有返回值，那么就直接省略最后的返回信息
* 如果有返回值， 那么必须在函数的外层添加return语句

来看一个最简单的函数：

```text
package main

func add(a,b int) map[] {
	var resultMap = map[string]int{}
	resultMap["add"] = a + b
	resultMap["multi"] = a * b
	resultMap["subtract"] = a - b
	resultMap["division"] = a / b
	return resultMap
}

func main()  {
	a := 3
	b := 4
	count := add(a, b)
	println(count)
}
```

**多个返回值：**

Go中函数可以有多个返回值，这个比java强悍100倍：

```text
package main

func add(a,b int) (int,int) {
	return a+b,a-b
}

func main()  {
	a := 3
	b := 4
	add,sub := add(a, b)
	println(add,sub)
}

```

**可变参数：**

跟Java中差不多吧：

```text
func myFunc(arg ...int) {}
```

**defer**

defer是golang的一个特色功能，被称为“延迟调用函数”。当外部函数返回后执行defer。类似于其他语言的 try… catch … finally… 中的finally，当然差别还是明显的。

释放占用资源：

```text
func test() error {

	file, err := os.Open("path")
	if err != nil {
		return err
	}

	//放在判断err状态之后
	defer file.Close()

	//todo
	//...

	return nil
	//defer执行时机
}
```

异常处理：

```text
func test2() {
	defer func() {
		if err := recover(); err != nil {
			println(err)
		}
	}()

	file, err := os.Open("path")
	if err != nil {
		panic(err)
	}

	defer file.Close()

	//todo
	//...

	return
	//defer执行时机
}
```

日志输出：

```text
func test3() {

	t1 := time.Now()

	defer func() {
		println("耗时: %f s", time.Now().Sub(t1).Seconds())
	}()

	//todo
	//...

	return
	//defer执行时机
}
```

**3. struct类型\#**

Java中我们会去声明一些bean对象，里面包含字段和属性，在Go中可以声明一个struct类型的实体，在这个实体中声明一些属性，但是不可以在里面定义func。

```text
type person struct {
    name string
    age int
}
```

使用方式：

```text
package main

type person struct {
	name string
	age int
}


func main()  {
	var p person
	p.age = 13
	p.name = "xiaoming"
	println(p.name,p.age)
	
	p1 := person{"xiaoming",13}
	println(p1.name,p1.age)
	
	p2 := person{age:13,name:"xiaoming"}
	println(p2.name,p2.age)
	
}
```

**Go中的继承—匿名字段**

上面我们在定义struct的时候，里面的属性都是字段名和类型一一对应的。实际上Go也支持只提供类型而不写字段名的方式，也就是匿名字段。

当匿名字段是一个struct的时候，那么这个struct所拥有的全部字段都被隐式地引入了当前定义的这个struct。

```text
package main

type PersonOther struct {
	phone string
	address string
}

type Person struct {
	PersonOther
	name string
	age int
}


func main()  {
	p1 := Person{PersonOther{"13242342123","xxxxxxx"},"xiaoming",13}
	println(p1.address,p1.phone,p1.name,p1.age)


}

```

可以看到在p1中是可以看到address和phone属性的，这就跟Java中的继承一样。

既然说到继承，那么肯定会有这样的一种情况，就是在Person和PersonOther中都定义过了phone属性，当我们用p1去获取的时候到底获取的是哪个对象的phone属性呢？

**Go里面很简单的解决了这个问题，最外层的优先访问，也就是当你通过`Person.phone`访问的时候，是访问student里面的字段，而不是PersonOther里面的字段。**

当前如果你想访问父类中的phone也不是不可以，Go还保留着父类中的对象呢，你可以这样取出来：

```text
parentPhone := p1.PersonOther.phone
```

在Java中如果是这样的话，父类的同名字段就被子类覆写了，取不出来。

**4. 也谈谈面向对象编程----多态\#**

在Java中我们经常这样做：

定义一个关于计算面积的接口；

定义一个计算圆面积的类实现接口；

定义一个正方形计算面积的类实现接口；

定义一个计算长方形面积的类实现接口；

…

这样我们就抽象出来一套统一的计算面积的方案由一个接口把持，需要哪个面积计算就调用相关的实现。

在Go中同样也有这样的概念，但是实现方式确是不同。基于上面的原因所以就有了`method`的概念，`method`是附属在一个给定的类型上的，他的语法和函数的声明语法几乎一样，只是在`func`后面增加了一个receiver\(也就是method所依从的主体\)。

method的语法如下：

```text
func (r ReceiverType) funcName(parameters) (results)
```

看一个具体的示例：

```text
package main

type Circle struct {
	redius float64
}

type Square struct {
	width,height float64
}

func (c Circle) area() float64  {
	return c.redius * c.redius * 3.14
}

func (s Square) area() float64  {
	return s.height * s.width
}


func main()  {
	c := Circle{3.55}
	s := Square{3,5}
	println(c.area())
	println(s.area())
}

```

可以看到调用method通过Circle示例访问就像访问struct里面的字段一样。

