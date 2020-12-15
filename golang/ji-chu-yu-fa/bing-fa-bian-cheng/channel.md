# channel



### 一 channel简介

如果在程序中开启了多个goroutine，那么这些goroutine之间该如何通信呢？

go提供了一个channel（管道）数据类型，可以解决协程之间的通信问题！channel的本质是一个队列，遵循先进先出规则（FIFO），内部实现了同步，确保了并发安全！

### 二 channel的创建

#### 2.0 channel的语法

channel在创建时，可以设置一个可选参数：缓冲区容量

* 创建有缓冲channel：`make(chan int, 10)`，创建一个缓冲长度为10的channel
* 创建无缓冲channel：`make(chan int)`，其实就是第二个参数为0

channel内可以存储多种数据类型，如下所示：

```text
	ci := make(chan int)
	cs := make(chan string)
	cf := make(chan interface{})				
```

从管道中读取，或者向管道写入数据，使用运算符：`<-`，他在channel的左边则是读取，右边则代表写入：

```text
	ch := make(chan int)
	ch <- 10				// 写入数据10
	num := <- ch			// 读取数据
```

#### 2.1 无缓冲channel

无缓冲的channel是阻塞读写的，必须写端与读端同时存在，写入一个数据，则能读出一个数据：

```text
package main

import (
	"fmt"
	"time"
)

// 写端
func write(ch chan int) {
	ch <- 100
	fmt.Printf("ch addr：%v\n", ch) // 输出内存地址
	ch <- 200
	fmt.Printf("ch addr：%v\n", ch) // 输出内存地址
	ch <- 300                      // 该处数据未读取，后续操作直接阻塞
	fmt.Printf("ch addr：%v\n", ch) // 没有输出
}

// 读端
func read(ch chan int) {
	// 只读取两个数据
	fmt.Printf("取出的数据data1：%v\n", <-ch) // 100
	fmt.Printf("取出的数据data2：%v\n", <-ch) // 200
}

func main() {

	var ch chan int     // 声明一个无缓冲的channel
	ch = make(chan int) // 初始化

	// 向协程中写入数据
	go write(ch)

	// 向协程中读取数据
	go read(ch)

	// 防止主go程提前退出，导致其他协程未完成任务
	time.Sleep(time.Second * 3)
}
```

注意：无缓冲通道的收发操作必须在不同的两个`goroutine`间进行，因为通道的数据在没有接收方处理时，数据发送方会持续阻塞，所以通道的接收必定在另外一个 goroutine 中进行。

如果不按照该规则使用，则会引起经典的Golang错误`fatal error: all goroutines are asleep - deadlock!`：

```text
func main() {
	ch := make(chan int)
	ch <- 10
	<-ch
}
```

#### 2.2 有缓存channel

有缓存的channel是非阻塞的，但是写满缓冲长度后，也会阻塞写入。

```text
package main

import (
	"fmt"
	"time"
)

// 写端
func write(ch chan int) {
	ch <- 100
	fmt.Printf("ch addr：%v\n", ch) // 输出内存地址
	ch <- 200
	fmt.Printf("ch addr：%v\n", ch) // 输出内存地址
	ch <- 300                      // 写入第三个，造成阻塞
	fmt.Printf("ch addr：%v\n", ch) // 没有输出
}
func main() {

	var ch chan int        // 声明一个有缓冲的channel
	ch = make(chan int, 2) // 可以写入2个数据

	// 向协程中写入数据
	go write(ch)

	// 防止主go程提前退出，导致其他协程未完成任务
	time.Sleep(time.Second * 3)
}
```

同样的，当数据全部读取完毕后，再次读取也会造成阻塞，如下所示：

```text
func main() {
	ch := make(chan int, 1)
	ch <- 10
	<-ch
	// <-ch
}
```

此时程序可以顺序运行，不会报错，这是与无缓冲通道的区别，但是当继续打开 注释 部分代码时，通道阻塞，所有协程挂起，此时也会产生错误：`fatal error: all goroutines are asleep - deadlock!`。

#### 2.3 总结 无缓冲通道与有缓冲通道

无缓冲channel：

* 通道的容量为0，即 `cap(ch) = 0`
* 通道的个数为0，即 `len(ch) = 0`
* 可以让读、写两端具备并发同步的能力

有缓冲channel：

* 在make创建的时候设置非0的容量值
* 通道的个数为当前实际存储的数据个数
* 缓冲区具备数据存储的能力，到达存储上限后才会阻塞，相当于具备了异步的能力
* 有缓冲channel的阻塞产生条件：
  * 当缓冲通道被填满时，尝试再次发送数据会发生阻塞
  * 当缓冲通道为空时，尝试接收数据会发生阻塞

问题：为什么 Go语言对通道要限制长度而不提供无限长度的通道?

> channel是在两个 goroutine 间通信的桥梁。使用 goroutine 的代码必然有一方提供数据，一方消费数据 。通道如果不限制长度，在生产速度大于消费速度时，内存将不断膨胀直到应用崩溃。



### 三 channel的相关操作

#### 3.1 通道数据的遍历

channel只支持 for--range 的方式进行遍历：

```text
	ch := make(chan int)

	go func() {
		for i := 0; i <=3; i++ {
			ch <- i
			time.Sleep(time.Second)
		}
	}()

	for data := range ch {
		fmt.Println("data==", data)
		if data == 3 {
			break
		}
	}
```

#### 3.2 通道关闭

通道是一个引用对象，支持GC回收，但是通道也可以主动被关闭：

```text
	ch := make(chan int)
	close(ch)				// 关闭通道
	ch <- 1					// 报错：send on closed channel
```

从通道中接收数据时，可以利用多返回值判断通道是否已经关闭：

```text
func main() {

	ch := make(chan int, 10)

	go func(ch chan int) {
		for i := 0; i < 10; i++ {
			ch <- i
		}
		close(ch)
	}(ch)

	for {
		if num, ok := <-ch; ok == true {
			fmt.Println("读到数据：", num)
		} else {
			break
		}
	}
} 
```

如果channel已经关闭，此时需要注意：

* 不能再向其写入数据，否则会引起错误：`panic:send on closed channel`
* 可以从已经关闭的channel读取数据，如果通道中没有数据，会读取通道存储的默认值

#### 3.3 通道读写

默认情况下，管道的读写是双向的，但是为了对 channel 进行使用的限制，可以将 channel 声明为只读或只写。

```text
	var chan1 chan<- int		// 声明 只写channel
	var chan2 <-chan int 		// 声明 只读channel
```

单向chanel不能转换为双向channel，但是双向channel可以隐式转换为任意类型的单向channel：

```text
// 只写端
func write(ch chan<- int) {
	ch <- 100
	fmt.Printf("ch addr：%v\n", ch) // 输出内存地址
	ch <- 200
	fmt.Printf("ch addr：%v\n", ch) // 输出内存地址
	ch <- 300                      // 该处数据未读取，后续操作直接阻塞
	fmt.Printf("ch addr：%v\n", ch) // 没有输出
}

// 只读端
func read(ch <-chan int) {
	// 只读取两个数据
	fmt.Printf("取出的数据data1：%v\n", <-ch) // 100
	fmt.Printf("取出的数据data2：%v\n", <-ch) // 200
}

func main() {

	var ch chan int         // 声明一个双向
	ch = make(chan int, 10) // 初始化

	// 向协程中写入数据
	go write(ch)

	// 向协程中读取数据
	go read(ch)

	// 防止主go程提前退出，导致其他协程未完成任务
	time.Sleep(time.Second * 3)
}
```

双向channel进行显式转换：

```text
	ch := make(chan int)		// 声明普通channel
	ch1 := <-chan int(ch)		// 转换为 只读channel
	ch2 := chan<- int(ch)		// 转换为 只写channel
```



### 四 channel的一些示例

#### 4.1 示例一 限制并发

耗时操作为timeMore，现在有100个并发，限制为5个：

```text
package main

import (
	"time"
	"fmt"
)

func timeMore(ch chan string) {
	
	// 执行前先注册，写不进去就会阻塞
	ch <- "任务"			

	fmt.Println("模拟耗时操作")
	time.Sleep(time.Second)	// 模拟耗时操作

	// 任务执行完毕，则管道中销毁一个任务
	<-ch
}

func main() {

	ch := make(chan string, 5)

	// 开启100个协程
	for i := 0; i < 100; i++ {
		go timeMore(ch)
	}

	for {
		time.Sleep(time.Second * 5)
	}
}
```

#### 4.2 生产者消费者模型

```text
package main

import (
	"fmt"
	"time"
)

// 生产者
func producer(ch chan<- int) {
	i := 1
	for {
		ch <- i
		fmt.Println("Send:", i)
		i++
		time.Sleep(time.Second * 1) // 避免数据流动过快
	}
}

// 消费者
func consumer(ch <-chan int) {
	for {
		v := <-ch
		fmt.Println("Receive:", v)
		time.Sleep(time.Second * 2) // 避免数据流动过快
	}
}

func main() {

	// 生产消费模型中的缓冲区
	ch := make(chan int, 5)
	// 启动生产者
	go producer(ch)
	// 启动消费者
	go consumer(ch)
	// 阻塞主程序退出
	for {

	}
}
```

当然，该模型也可以使用无缓冲模型，区别如下：

* 无缓冲生产消费模型：同步通信
* 有缓冲生产消费模型：异步通信

#### 4.3 定时器

定时器`time.Timer`的底层其实也是一个管道。

```text
type Timer struct {
    C <-chan Time
    r runtimeTimer
}
```

在时间到达之前，没有数据写入，则timer.C会一直阻塞，直到时间到达，系统会自动向timer.C中写入当前时间，阻塞就会被解除：

```text
    // 创建定时器 定义延迟时间为2秒
	layTimer := time.NewTimer(time.Second * 2)
	// 从管道取数据，但是一直都是空的，阻塞中，直到2庙后有数据才能取出
	fmt.Println(<-layTimer.C)
```

定时器的其他操作：

* Stop\(\)：停止定时器，此时如果从管道中取数据，则会阻塞
* Reset\(\)：重置定时器，此时需要传入一个新的定时时间间隔
* timer.Tiker：可以创建一个周期定时器

