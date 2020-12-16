# 栈的顺序存储实现

```go
/*
 *  顺序栈
 */

package stack

import (
	"fmt"
)

const SqStackCap = 5				// 这里不再设计扩容代码，直接给出了一个固定容量

// 顺序栈结构体
type SqStack struct {
	items		[]interface{}		// 节点数组（切片）
	cap			int					// 栈的最大容量
	top 		int					// 栈顶指针，其实就是当前索引位置
}

func NewSqStack() *SqStack{
	return &SqStack{
		items: 	make([]interface{}, SqStackCap),
		cap: 	SqStackCap,
		top:	-1,					// 默认-1，表示是空栈，到达0则是数组的第一个索引成为栈顶
	}
}

// 压栈操作
func (s *SqStack)Push(e interface{}) bool{

	if s.top == s.cap - 1 {
		fmt.Println("栈已满")
		return false
	}

	s.items[s.top + 1] = e
	s.top++
	return true
}

// 弹栈操作：返回出栈的值
func (s *SqStack)Pop() interface{}{

	if s.top == -1 {
		fmt.Println("空栈无法pop")
		return nil
	}

	e := s.items[s.top]
	s.top--
	return e
}

// 获取栈顶元素
func (s *SqStack)Top() interface{}{
	if s.top == -1 {
		fmt.Println("当前栈为空栈")
		return nil
	}
	return s.items[s.top]
}
```

