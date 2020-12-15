# 动态数组

### 一 动态数组扩容与缩容机制

数组的长度固定在使用上有极大约束力，如果数组的长度可以在适当的时机进行扩容、缩容，则是实际开发中所青睐的数据结构。

扩容/缩容的本质：重新申请了一个数组内存，将原有的数据拷贝入新申请的数组。

在制作数据结构的时候，比如`cap = 5`，那么在长度到 5 时，就需要一次扩容，那么本次操作肯定比较耗时，为了避免某次操作突然耗时较长，可以进行均摊，比如每次插入元素，都将`cap + 1`，v8具备这样的扩容缩容机制。

但是扩容和缩容设计不当，则会引起复杂度震荡。比如在某个节点进行添加收，需要扩容，而下次操作则是删除该节点，则执行缩容，反复增加删除增加删除，那么其时间复杂度会一直维持在最高水平。

解决办法：扩容的倍数缩容的时机相乘不等于1即可。

### 二 手动实现一个动态数组

```go
/*
 *  动态数组
 */

package array

import "fmt"

const DynamicArrayCap = 5		// 默认容量

type DynamicArray struct {
	items 	[]interface{}
	cap 	int            	// 动态数组容量
	length	int				// 动态数组元素个数
}

// 创建动态数组对象
func NewDynamicArray() *DynamicArray{
	return &DynamicArray{
		items:  make([]interface{}, DynamicArrayCap),
		cap:   DynamicArrayCap,
		length: 0,
	}
}

// 获取数组容量
func (arr *DynamicArray)Size() int {
	return arr.cap
}

// 获取数组元素个数
func (arr *DynamicArray)Length() int {
	return arr.length
}

// Push元素：添加元素到末尾
func (arr *DynamicArray)Push(e interface{}) {
	arr.length++
	arr.expandCap()			// 扩容
	arr.items[arr.Length() - 1] = e
}

// Pop元素：删除最后一个元素
func (arr *DynamicArray)Pop() interface{}{
	if arr.Length() <= 0 {
		fmt.Println("当前数组为空")
		return nil
	}

	e := arr.items[arr.Length() - 1]
	arr.items[arr.Length() - 1] = nil
	arr.length--

	arr.reduceCap()		// 缩容
	return e
}

// 扩容方法：超过数组容量，按照当前容量的 2 倍扩容
func (arr *DynamicArray)expandCap() {
	if arr.length <= arr.cap {
		return						// 无需扩容
	}

	newSize := arr.Size() * 2
	newItems := make([]interface{}, newSize)
	for i := 0; i < arr.Length() - 1; i++ {
		newItems[i] = arr.items[i]
	}
	arr.items = newItems
	arr.cap = newSize
}

// 缩容方法：数组元素个数为当前容量 1/4 时，缩容为当前容量的一半
func (arr *DynamicArray)reduceCap() {
	if arr.length > arr.cap / 4 {
		return						// 无需缩容
	}

	newSize := arr.Size() / 2
	newItems := make([]interface{}, newSize)
	for i := 0; i < arr.Length() - 1; i++ {
		newItems[i] = arr.items[i]
	}
	arr.items = newItems
	arr.cap = newSize
}

// 根据索引查找元素
func (arr *DynamicArray)Index(index int) interface{}{
	if index < 0 || index > arr.Length() - 1 {
		fmt.Println("数组索引越界")
		return nil
	}
	return arr.items[index]
}
```

### 三 Go中的专有数据结构-切片

Golang官方为了解决数组长度不能扩展，以及基本类型数组传递时产生副本的问题，官方包中引入了专有的数据结构切片（Slice）。

常用创建方式：

```text
    var s1 []int				// 和声明数组一样，只是没有长度，但是这样做没有意义，因为底层的数组指针为nil
    s2 := []byte {'a','b','c'}
    fmt.Println(s1)				//输出 []
    fmt.Print(s2)				//输出 [97 98 99]
```

使用make函数创建：

```text
    slice1 := make([]int,5)		// 创建长度为5，容量为5，初始值为0的切片
    slice2 := make([]int,5,7)	// 创建长度为5，容量为7，初始值为0的切片
    slice3 := []int{1,2,3,4,5}	// 创建长度为5，容量为5，并已经初始化的切片
```

贴士：为了方便，笔者在以后大多场景中，都使用切片来代替数组来模拟数据结构与算法。

