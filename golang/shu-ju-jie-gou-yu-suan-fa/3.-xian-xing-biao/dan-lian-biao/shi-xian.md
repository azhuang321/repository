# 实现

```go
/*
 *  单链表：带头节点
 */
package list

import (
	"fmt"
)

// 结点结构体
type LinkedNode struct {
	data	interface{}
	next	*LinkedNode
}

// 单链表结构体
type LinkedList struct {
	head	*LinkedNode		// 头节点，头节点之后的第一个节点可以称呼为首元节点
}

// 构造表
func NewLinkedList() *LinkedList {
	head := &LinkedNode{
		data: 0,				// 这里使用头节点存储链表的长度，不包含头节点
		next: nil,
	}
	return &LinkedList{
		head:  head,
	}
}

func (l *LinkedList)PushBack(e interface{}) {
	// 构造要插入的节点
	insertNode := &LinkedNode{
		data: e,
		next: nil,
	}
	// 当前循环到的节点
	currentNode := l.head
	for currentNode.next != nil {
		currentNode = currentNode.next
	}
	currentNode.next = insertNode
	l.head.data = l.head.data.(int) + 1
}

func (l *LinkedList)PushFront(e interface{}) {
	// 构造要插入的节点
	insertNode := &LinkedNode{
		data: e,
		next: nil,
	}
	insertNode.next = l.head.next
	l.head.next = insertNode
	l.head.data = l.head.data.(int) + 1
}

// 插入元素
func (l *LinkedList)Insert(index int, e interface{}) bool {

	if index < 1 || index > l.Length() + 1 {
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

	// 构造要插入的节点
	insertNode := &LinkedNode{
		data: e,
		next: nil,
	}

	// 找到要插入位置的前一个节点
	prevNode := l.head
	for i := 1; i <= index - 1; i++ {
		prevNode = prevNode.next
	}

	// 执行插入
	insertNode.next = prevNode.next
	prevNode.next = insertNode
	return true
}

// 删除元素
func (l *LinkedList)Delete(e interface{}) bool {

	// 找到该元素
	delNode := l.curNode(e)
	if delNode == nil {
		fmt.Println("未找到该元素")
		return false
	}

	// 获取前驱：这时候就是单链表的不足了，必须再次循环，或者在查找节点的时候制作多返回值
	prevNode := l.prevNode(e)
	if prevNode == nil {
		fmt.Println("未找到前驱元素")
		return false
	}

	prevNode.next = delNode.next
	l.head.data = l.head.data.(int) - 1
	return true

}

// 根据元素获取节点
func (l *LinkedList)curNode(e interface{}) *LinkedNode {

	if l.Length() <= 0 {
		fmt.Println("链表为空")
		return nil
	}

	currentNode := l.head
	for currentNode.next != nil {
		currentNode = currentNode.next
		if currentNode.data == e {
			return currentNode
		}
	}

	fmt.Println("未找到该元素")
	return nil
}

// 获取前驱节点：注意是节点，不是值，有了节点，再写取值方法、任意位置插入方法也很方便
func (l *LinkedList)prevNode(e interface{}) *LinkedNode {

	if l.Length() <= 0 {
		fmt.Println("链表为空")
		return nil
	}

	// 注意：如果没有头节点，还需要额外判断是否是首元素，因为首元素无前驱
	currentNode := l.head
	var prevNode *LinkedNode
	for currentNode.next != nil {
		prevNode = currentNode
		currentNode = currentNode.next
		if currentNode.data == e {
			return prevNode
		}
	}

	fmt.Println("未查找到当前节点")
	return nil
}

// 获取后继节点：注意是节点，不是值，有了节点，再写取值方法、任意位置插入方法也很方便
func (l *LinkedList)nextNode(e interface{}) interface{}{

	if l.Length() <= 0 {
		fmt.Println("链表为空")
		return nil
	}

	currentNode := l.head
	for currentNode.next != nil {
		currentNode = currentNode.next
		if currentNode.data == e {
			if currentNode.next == nil {
				fmt.Println("末尾元素无后继节点")
				return nil
			} else {
				return currentNode.next
			}
		}
	}
	fmt.Println("未查找到当前节点")
	return nil
}



// 获取长度
func (l *LinkedList)Length() int {
	return l.head.data.(int)
}

// 清空
func (l *LinkedList) Clear() {
	l.head.data = 0
	l.head.next = nil
}
```

