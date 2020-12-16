# 实现

```go
/**
 *  静态链表：StaticList
 *	链表内部结构应该杜绝指针，因为静态链表本就是为了无指针场景
 *  一般静态链表内部的数组第一个元素、最后一个元素不存储数据
 *  游标cur为0时，表示无指向
 *  下标为0的元素的cur存放备用链表的第一个结点的下标（备用链表即未被使用的数组元素）
 */

package staticlist

import (
	"errors"
	"fmt"
)
 
// 静态链表中元素节点
type node struct {
	data		interface{}
	cur			int
}
 
// 静态链表结构体
type StaticList struct {
	size		int
	length		int
	arr			[]node
}
 
 // 构造实例：其实是构造一个备用链表
func NewStaticList(size int) *StaticList {

	if size < 3 {
		fmt.Println("数组长度最少为3")
		panic("数组长度最少为3")
	}

	arr := make([]node, size)
	for i := 0; i < size - 1; i++ {
		arr[i].cur = i + 1
	}
	// 数组的最后一位的cur为0
	arr[size - 1].cur = 0

	return &StaticList{
		size: 	size,
		length:	0,
		arr:	arr,
	}
}
 
// 打印线性表
func (l *StaticList) Display() {
 
	fmt.Printf("数据结构长度：%d，容量：%d\n", l.length, l.size)
 
	if l.length == 0 {
		 return
	} 
 
	fmt.Printf("数据元素显示：")
	for i := 1; i <= l.length; i++ {
		fmt.Printf("%d ", l.arr[i].data)
	}
	fmt.Println("")
}

// 增：任意位置插入元素
func (l *StaticList) Insert(index int, e interface{}) error {
	
	if index < 1 || index > l.length + 1 {
		fmt.Println("位序不合法")
		return errors.New("位序不合法")
	}

	// 获取空闲分量的下标:取出一个元素，并将数组第一位的cur修改为数组的下一个元素的cur
	freeIndex := l.arr[0].cur
	l.arr[0].cur = l.arr[freeIndex].cur		// 必须用下一个分量作为备用

	
	// 将数据赋值给取出的这个分量
	l.arr[freeIndex].data = e

	// 找到第index个元素之前的位置
	var updIndex = l.size - 1
	for preIndex := 1; preIndex <= index - 1; preIndex++ {
		updIndex = l.arr[updIndex].cur
	}

	// 把第i个元素之前的cur赋值给新元素的cur
	l.arr[freeIndex].cur = l.arr[updIndex].cur
	l.arr[updIndex].cur = freeIndex

	l.length++
	return nil

}

// 删除
func (l *StaticList)Delete(index int) error {

	if index < 1 || index > l.length {
		fmt.Println("位序不合法")
		return errors.New("位序不合法")
	}

	i := l.arr[l.size - 1].cur
	j := 1
	for i > 0 && j < index - 1 {
		j++
		i = l.arr[i].cur
	}

	temp := l.arr[i].cur
	l.arr[i].cur = l.arr[temp].cur

	// 回收链表
	// 把第一个元素的cur值付给要删除的分量cur
	l.arr[temp].cur = l.arr[0].cur
	// 把要删除的分量下标赋值给第一个元素的cur
	l.arr[0].cur = temp


	l.length--
	return nil
}
```

