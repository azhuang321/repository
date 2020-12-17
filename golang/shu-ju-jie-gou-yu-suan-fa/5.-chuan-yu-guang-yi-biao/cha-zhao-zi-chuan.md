# 查找子串

### 一 模式匹配

串的最常见操作就是查找子串，也称为模式匹配。

比如要从`S="goodgoogle"`中找到`T="google"`这个子串的位置，通常需要如下步骤：

* 1 索引从主串S第一位开始匹配，前三个匹配成功，第四个字母失败，索引返回第二个位置
* 2 索引从主串S第二位开始重新匹配，匹配失败，依次类推
* 3 到达google匹配成功

在上述案例中，n为主串长度，m为要匹配的子串长度，根据等概率原则，平均是`(n+m)/2`次查找，时间复杂度为O\(n+m\)，在最坏的情况下，比如`S="00000001"`，要匹配`T="01"`，时间复杂度是O\(m\*\(n-m+1\)\)，这种算法的低效程度令人发指。

代码如下：

```go
package main

import (
	"errors"
	"fmt"
)

// 给定字符串s，子串 t，查找t在str中的位置（索引）
func Index(s string, t string) (pos int, err error){

	sLength := len(s)
	tLength := len(t)

	if sLength == 0 || tLength == 0 || sLength < tLength{
		err = errors.New("字符串长度不合法")
		return
	}

	for i := 0; i <= sLength - tLength; i++ {
		// s 截取和 t 一样长度的子串
		sonStr := s[i: i + tLength]
		if sonStr == t {
			err = errors.New("未匹配成功")
			continue
		} else {
			pos = i
			err = nil
			break
		}
	}

	return

}

func main() {

	s1 := "goodgoogle"
	s2 := "google"

	r, e := Index(s1, s2)

	fmt.Println("最终e:", e)
	fmt.Println("最终r:", r)

}
```

### 二 朴素模式匹配算法的不足

上一节的源码中使用了投机取巧的办法：

```go
// s 截取和 t 一样长度的子串
sonStr := s[i: i + tLength]
```

这样的操作并不符合题目解题思路中的步骤，朴素模式匹配是让模式串的首字符从主串的首字符开始向右匹配，这里会存在指针回溯的情况，其匹配规则如下：

* 第一步：在主串的第i个位置，让pos=i，如果主串pos位置的字符等于模式串的第一个字符，则pos++，再与模式串的第二个字符进行匹配。
* 第二步：如果又相等则继续向下匹配，直到匹配到模式串的结尾，就说明主串中存在该模式串；如果不相等，则停止匹配，回到主串中，并i++。
* 第三步：然后再让pos=i，重复上述步骤，直到i到达主串结尾。

如果不利用golang的该特性，那么真实的朴素模式匹配算法应该如下：

```go
package main

import (
	"errors"
	"fmt"
)

// 给定字符串s，子串 t，查找t在str中的位置（索引）
func Index(s string, t string) (pos int, err error){

	sLength := len(s)
	tLength := len(t)
	if sLength == 0 || tLength == 0 || sLength < tLength {
		return 0, errors.New("字符串长度不合法")
	}

	for i := 0; i <= sLength - tLength; i++ {

		pos = i

		for j := 0; j < tLength; j++ {

			if s[i] != t[j] {
				i = i - j + 1							// 与到不相等 i 回退
				pos = 0
				err = errors.New("未匹配到数据")
				break
			} else {
				i++
				err = nil
				continue
			}
		}

	}

	return
}

func main() {

	s1 := "goodgoogle"
	s2 := "google"

	r, e := Index(s1, s2)

	fmt.Println("最终e:", e)
	fmt.Println("最终r:", r)

}
```

每次i值的回退都会耗费一定的时间，为了避免一些无谓的浪费，可以使用KMP算法。

