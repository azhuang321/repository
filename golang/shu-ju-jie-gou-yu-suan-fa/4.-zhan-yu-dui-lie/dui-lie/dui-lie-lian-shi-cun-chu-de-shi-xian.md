# 队列链式存储的实现

```go
/*
	链式存储队列
 */

package queue

import (
	"fmt"
)

type LinkedNode struct {
	data   	interface{}
	next 	*LinkedNode
}

type LinkedQueue struct {
	length	int
	front	*LinkedNode
	rear    *LinkedNode
}

func NewLinkQueue() *LinkedQueue{
	return &LinkedQueue{
		length: 0,
		front:  nil,
		rear:   nil,
	}
}

func (q *LinkedQueue)EnQueue(e interface{}) {

	enNode := &LinkedNode{
		data: e,
		next: nil,
	}

	// 笔者使用无头节点方式，需要额外判断是否是第一次插入
	if q.length == 0 {
		q.front = enNode
		q.rear = enNode
		q.length++
		return
	}

	currentNode := q.front
	for currentNode.next != nil {
		currentNode = currentNode.next
	}

	// 把新结点赋值给原队尾结点的后继
	currentNode.next = enNode
	q.rear = enNode
	q.length++
}

func (q *LinkedQueue)DeQueue() interface{}{

	if q.Length() == 0 {
		fmt.Println("队列为空")
		return nil
	}

	// 笔者这里使用的是无头节点方式，直接进行出队操作
	temp := q.front
	q.front = q.front.next

	// 若队头是队尾，则出队后将tail指向head
	if q.rear == temp {
		q.rear = q.front
	}
	q.length--
	return temp.data
}

func (q *LinkedQueue)Length() int {
	return q.length
}
```

