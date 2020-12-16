# 实现

```go
/*
*  双向循环链表：带头节点的双向链表
*/
package list

import (
	"fmt"
)

// 结点结构体
type DCircleNode struct {
	data 	interface{}
	next 	*DCircleNode
	prev 	*DCircleNode
}

// 单链结构体
type DCircleList struct {
	head  	*DCircleNode	// 该双向循环链表的头节点
	length	int				// 笔者这里不再像单向链表那样把元素个数存储在头节点中
}

// 构造表
func NewDCircleList() *DCircleList {
	head := &DCircleNode{
		data: 0,
		next: nil,
		prev: nil,
	}
	head.next = head
	head.prev = head

	return &DCircleList{
		head:  	head,
		length: 0,
	}
}

func (l *DCircleList)PushBack(e interface{}) {
	// 构造要插入的节点
	insertNode := &DCircleNode{
		data: e,
		next: l.head,
		prev: nil,
	}
	// 当前循环到的节点
	currentNode := l.head
	for currentNode.next != l.head {			// 循环链表的末尾结束标志是 不等于 l.head
		currentNode = currentNode.next
	}
	currentNode.next = insertNode
	insertNode.prev = currentNode
	l.length++
}

func (l *DCircleList)PushFront(e interface{}) {

	// 构造要插入的节点
	insertNode := &DCircleNode{
		data: e,
		next: nil,
		prev: l.head,
	}

	if l.Length() == 0 {
		insertNode.next = l.head
		l.head.prev = insertNode
		l.head.next = insertNode
		l.length++
		return
	}

	// 插入位置的后继节点更改
	nextNode := l.head.next
	nextNode.prev = insertNode
	insertNode.next = nextNode

	// head节点更改
	l.head.next = insertNode
	l.length++
	return
}

// 插入元素
func (l *DCircleList)Insert(index int, e interface{}) bool {

	if index < 1 || index > l.length + 1 {
		fmt.Println("插入位序不正确")
		return false
	}

	if index == 1 {
		l.PushFront(e)
		return true
	}

	if index == l.Length() + 1 {
		l.PushBack(e)
		return true
	}

	// 常规插入:先获取插入位子
	currentNode := l.head
	for i := 1; i <= index; i++ {
		currentNode = currentNode.next
	}
	prevNode := currentNode.prev

	// 执行插入
	insertNode := &DCircleNode{
		data: e,
		next: currentNode,
		prev: prevNode,
	}
	currentNode.prev = insertNode
	prevNode.next = insertNode
	l.length++
	return true
}

// 获取长度
func (l *DCircleList)Length() int {
	return l.length
}

// 清空
func (l *DCircleList)Clear() {
	l.head.next = l.head
	l.head.prev = l.head
	l.length = 0
}
```

