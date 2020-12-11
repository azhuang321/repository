# 接口

### 一 接口 interface

接口（interface）是调用方和实现方均需要遵守的一种约束，约束开发者按照统一的方法命名、参数类型、数量来处理具体业务。实际上，接口就是一组没有实现的方法声明，到某个自定义类型要使用该方法时，根据具体情况把这些方法实现出来。接口语法：

```go
type 接口类型名 interface {
	方法名1(参数列表) 返回值列表
	方法名2(参数列表) 返回值列表
	...
}
```

示例：

```go
package main

import "fmt"

// 运输方式
type Transporter interface {
	BicycleTran()
	CarTran()
}

// 驾驶员
type Driver struct {
	Name string
	Age  int
}

// 实现运输方式接口
func (d *Driver) BicycleTran() {
	fmt.Println("使用自行车运输")
}
func (d *Driver) CarTran() {
	fmt.Println("使用小汽车运输")
}

func main() {
	d := &Driver{
		"张三",
		27,
	}
	trans(d)
}

// 只要实现了 Transporter接口的类型都可以作为参数
func trans(t Transporter) {
	t.BicycleTran()
}
```

注意：

* Go语言的接口在命名时，一般会在单词后面添加er，如写操作的接口叫做Writer
* 当方法名首字母大写，且实现的接口首字母也是大写，则该方法可以被接口所在包之外的代码访问
* 方法与接口中的方法签名一致（方法名、参数列表、返回列表都必须一致）
* 参数列表和返回值列表中的变量名可以被忽略，如：type writer interfae{ Write\(\[\]byte\) error}
* 接口中所有的方法都必须被实现
* 如果编译时发现实现接口的方法签名不一致，则会报错：`does not implement`。

### 二 Go接口的特点

在上述示例中，Go无须像Java那样显式声明实现了哪个接口，即为非侵入式，接口编写者无需知道接口被哪些类型实现，接口实现者只需要知道实现的是什么样子的接口，但无需指明实现了哪个接口。编译器知道最终编译时使用哪个类型实现哪个接口，或者接口应该由谁来实现。

类型和接口之间有一对多和多对一的关系，即：

* 一个类型可以实现多个接口，接口间是彼此独立的，互相不知道对方的实现
* 多个类型也可以实现相同的接口。

```go
type Service interface {
	Start()
	Log(string)
}

// 日志器
type Logger struct {
}
//日志输出方法
func (g *Logger) Log(s string){
	fmt.Println("日志：", s)
}

// 游戏服务
type GameService struct {
	Logger
}
// 实现游戏服务的Start方法
func (g *GameService) Start() {
	fmt.Println("游戏服务启动")
}

func main() {
	s := new(GameService)
	s.Start()
	s.Log("hello")
}
```

在上述案例中，即使没有接口也能运行，但是当存在接口时，会隐式实现接口，让接口给类提供约束。

使用接口调用了结构体中的方法，也可以理解为实现了面向对象中的多态。

### 三 接口嵌套

Go中不仅结构体之间可以嵌套，接口之间也可以嵌套。接口与接口嵌套形成了新的接口，只要接口的所有方法被实现，则这个接口中所有嵌套接口的方法均可以被调用。

```go
// 定义一个 写 接口
type Writer interface {
	Write(p []byte) (n int, e error)
}

// 定义一个 读 接口
type Reader interface {
	Read() error
}

// 定义一个 嵌套接口
type IO interface {
	Writer
	Closer
}
```

### 四 空接口

**4.1 空接口定义**

空接口是接口的特殊形式，没有任何方法，因此任何具体的类型都可以认为实现了空接口。

```go
	var any interface{}

	any = 1
	fmt.Println(any)

	any = "hello"
	fmt.Println(any)
```

空接口作为函数参数：

```go
func Test(i interface{}) {
	fmt.Printf("%T\n", i)
}

func main() {
	Test(3)			// int
	Test("hello")	// sting
}
```

利用空接口，可以实现任意类型的存储：

```go
	m := make(map[string]interface{})
	m["name"] = "李四"
	m["age"] = 30	
```

**4.2 从空接口获取值**

保存到空接口的值，如果直接取出指定类型的值时，会发生编译错误：

```go
	var a int = 1
	var i interface{} = a
	var b int = i				//这里编译报错（类型不一致），可以这样做：b := i
```

**4.3 空接口值比较**

类型不同的空接口比较：

```go
	var a interface{} = 100
	var b interface{} = "hi"

	fmt.Println(a == b)			//false
```

不能比较空接口中的动态值：

```go
	var c interface{} = []int{10}
	var d interface{} = []int{20}
	fmt.Println(c == d)					//运行报错
```

还原相关类型:

```go
func main() {
	var a interface{} = 1
	var b interface{} = '2'
	var c interface{} = "3"
	var d interface{} = true
	var e interface{} = 1.1

	fmt.Println(a.(int))
	fmt.Println(b.(int32))
	fmt.Println(c.(string))
	fmt.Println(d.(bool))
	fmt.Println(e.(float64))
}
```

空接口的类型和可比较性：

| 类型 | 说明 |
| :--- | :--- |
| map | 不可比较，会发生宕机错误 |
| 切片 | 不可比较，会发生宕机错误 |
| 通道 | 可比较，必须由同一个make生成，即同一个通道才是true |
| 数组 | 可比较，编译期即可知道是否一致 |
| 结构体 | 可比较，可逐个比较结构体的值 |
| 函数 | 可比较 |



### 五 断言

接口是编程的规范，他也可以作为函数的参数，以让函数更具备适用性。在下列示例中，有三个接口 动物接口、飞翔接口、游泳接口，两个实现类鸟类与鱼类：

* 鸟类：实现了动物接口，飞翔接口
* 鱼类：实现了动物接口，游泳接口

```go
package main

import "fmt"

// 定义一个通用接口：动物接口
type Animal interface {
	Breath()			// 动物都具备 呼吸方法
}

type Flyer interface {
	Fly()
}
type Swimer interface {
	Swim()
}

// 定义一个鸟类：其呼吸的方式是在陆地
type Bird struct {
	Name string
	Food string
	Kind string
}
func (b *Bird) Breath() {
	fmt.Println("鸟 在 陆地 呼吸")
}
func (b *Bird) Fly() {
	fmt.Printf("%s 在 飞\n", b.Name)
}

// 一定一个鱼类：其呼吸方式是在水下
type Fish struct {
	Name string
	Kind string
}
func (f *Fish) Breath() {
	fmt.Println("鱼 在 水下 呼吸")
}
func (f *Fish) Swim() {
	fmt.Printf("%s 在游泳\n", f.Name)
}

// 一个普通函数，参数是动物接口
func Display(a Animal) {
	// 直接调用接口中的方法
	a.Breath()
	// 调用实现类的成员：此时会报错
	fmt.Println(a.Name)
}

func main() {
	var b = &Bird{
		"斑鸠",
		"蚂蚱",
		"鸟类"
	}
	Display(b)
}
```

接口类型无法直接访问其具体实现类的成员，需要使用断言（type assertions），对接口的类型进行判断，类型断言格式：

```go
t := i.(T)			//不安全写法：如果i没有完全实现T接口的方法，这个语句将会触发宕机
t, ok := i.(T)		// 安全写法：如果接口未实现接口，将会把ok掷为false，t掷为T类型的0值
```

* i代表接口变量
* T代表转换的目标类型
* t代表转换后的变量

上述案例的Dsiplay就可以书写为：

```go
func Display(a Animal) {
	// 直接调用接口中的方法
	a.Breath()
	// 调用实现类的成员：此时会报错
	instance, ok := a.(*Bird)	// 注意：这里必须是 *Bird类型，因为是*Bird实现了接口，不是Bird实现了接口
	if ok {
		// 得到了具体的实现类，才能访问实现类的成员
		fmt.Println("该鸟类的名字是：", instance.Name)
	} else {
		fmt.Println("该动物不是鸟类")
	}
}
```

### 二 接口类型转换

在接口定义时，其类型已经确定，因为接口的本质是方法签名的集合，如果两个接口的方法签名结合相同（顺序可以不同），则这2个接口之间不需要强制类型转换就可以相互赋值，因为go编译器在校验接口是否能赋值时，比较的是二者的方法集。

在上一节中，函数Display接收的是Animal接口类型，在断言后转换为了别的类型：\*Bird\(实现类指针类型\)：

```go
func Display(a Animal) {
	instance, ok := a.(*Bird)		// 动物接口转换为了 *Bird实现类
	if ok {
		// 得到了具体的实现类，才能访问实现类的成员
		fmt.Println("该鸟类的名字是：", instance.Name)
	} else {
		fmt.Println("该动物不是鸟类")
	}
}
```

其实，断言还可以将接口转换成另外一个接口：

```go
func Display(a Animal) {
	instance, ok := a.(Flyer)			// 动物接口转换为了飞翔接口
	if ok {
		instance.Fly()
	} else {
		fmt.Println("该动物不会飞")
	}

}
```

一个实现类往往实现了很多接口，为了精准类型查询，可以使用switch语句来判断对象类型：

```go
var v1 interfaceP{} = ...
switch v := v1.(type) {
	case int:
	case string:
	...
}
```

### 三 多态

多态是面向对象的三大特性之一，即一个类型具备多种具体的表现形式。

上述示例中，鸟和鱼都实现了动物接口的 Breath方法，即动物的Breath方法在鸟和鱼中具备不同的体现。我们在new出动物的具体对象实例时，这个对象实例也就实现了对应自己的接口方法。

```go
// New出Animal的函数
func NewAnimal(kind string) Animal{

	switch kind {
	case "鸟类":
		return &Bird{}
	case "鱼类":
		return &Fish{}
	default:
		return nil
	}

}

func main() {
	// 获取的是动物接口类型，但是实现类是鸟类
	a1 := NewAnimal("鸟类")
	a1.Breath()		// 鸟 在 陆地 呼吸

	// 获取的是动物接口类型，但是实现类是鱼类
	a2 := NewAnimal("鱼类")
	a2.Breath()		// 鱼 在 水下 呼吸
}
```

