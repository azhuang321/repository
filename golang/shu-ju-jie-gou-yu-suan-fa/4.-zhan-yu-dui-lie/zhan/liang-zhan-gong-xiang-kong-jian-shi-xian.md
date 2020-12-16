# 两栈共享空间实现

```go
/*
 *  共享顺序栈
 */

package stack

import (
	"fmt"
)

const ShareStackCap = 10				// 栈一般不考虑扩容的需要，这里不再设计扩容代码
 
 // 共享栈结构体
type ShareStack struct {
	items		[]interface{}		// 节点数组（切片）
	top1 		int					// 栈1栈顶指针
	top2		int 				// 栈2栈顶指针
	cap			int					// 栈的最大容量
 }
 
func NewShareStack() *ShareStack{
	return &ShareStack{
		items: 	make([]interface{}, ShareStackCap),
		top1:	-1,
		top2:	ShareStackCap,
		cap: 	ShareStackCap,
	}
 }

 
// 压栈操作
func (s *ShareStack)Push(e interface{}, stackID int) bool{		// num 表示元素 e Push到栈1还是栈2

	if stackID < 1 || stackID > 2 {
		fmt.Println("栈编号选择错误")
		return false
	}

	if s.top1 + 1 == s.top2 {
		fmt.Println("栈空间已满")
		return false
	}


	if stackID == 1 {		// 对第一个栈进行入栈
		s.items[s.top1 + 1] = e
		s.top1++
	} else {			// 对第二个栈进行入栈
		s.items[s.top2 - 1] = e
		s.top2--
	}

	return true
}

// 弹栈操作：返回出栈的值
func (s *ShareStack)Pop(stackID int) interface{}{

	if stackID < 1 || stackID > 2 {
		fmt.Println("栈编号选择错误")
		return false
	}

	if (stackID == 1 && s.top1 == -1) || (stackID ==2 && s.top2 == s.size) {
		fmt.Println("空栈无法pop")
		return nil
	}

	var e interface{}

	if stackID == 1 {
		e = s.items [s.top1]
		s.top1--
	} else {
		e = s.items[s.top2]
		s.top2++
	}

	return e
}

// 获取栈顶元素
func (s *ShareStack)Top(stackID int) interface{}{

	if stackID < 1 || stackID > 2 {
		fmt.Println("栈编号选择错误")
		return false
	}

	if (stackID == 1 && s.top1 == -1) || (stackID ==2 && s.top2 == s.size) {
		fmt.Println("空栈无法pop")
		return nil
	}

	var e interface{}

	if stackID == 1 {
		e = s.items [s.top1]
	} else {
		e = s.items[s.top2]
	}

	return e
}
```

