---
title: golang基础-流程控制
lastmod: 2025-05-12T16:43:23+08:00
date: 2025-05-012T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-04.html
toc: true
---
## 流程控制

### if-else

if条件判断基本写法，格式如下

```go
if 表达式1 {
    分支1
}else if 表达式2 {
    分支2
} else {
    分支3
}
```

固定格式，`{ `不能另起一行

使用例子

```go
package main

import "fmt"

func main() {
	/* 	var score = 65
	   	if score >= 90 {
	   		fmt.Println("A")
	   	} else if score > 75 {
	   		fmt.Println("B")
	   	} else if score >= 60 {
	   		fmt.Println("C")
	   	} else {
	   		fmt.Println("D")
	   	} */

	if score := 65; score >= 90 {
		fmt.Println("A")
	} else if score > 75 {
		fmt.Println("B")
	} else {
		fmt.Println("C")
	}
}
```

### for循环

go中的循环都是用 for 实现，并未没有 while 语句

#### 基本结构

```go
for 初始语句;条件表达式;结束语句{
    循环体
}
```

当表达式返回为 true 时，循环体会不停的循环，直到条件表达式返回 false 时会自动退出循环

```go
func forDemo() {
	for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
}
```

初始语句和结束语句都可以省略

```go
func forDemo3() {
	i := 0
	for i < 10 {
		fmt.Println(i)
		i++
	}
}
```

#### 无限循环

```go
for {
    循环体
}
```

for循环可以通过 break、goto、return、panic等强制退出循环

#### for-range

循环遍历数组、切片、字符串、map以及管道(channel)。通过`for range`遍历的返回值有以下规律：

1. 数组、切片、字符串返回索引和值。
2. map返回键和值。
3. 通道（channel）只返回通道内的值。

Go1.22版本开始支持 for range 整数。

```go
for i := range 5 {
	fmt.Println(i)
}

for range 2 {
	fmt.Println("hello world")
}
```

### switch-case

使用 switch 可以方便的对大量值进行条件判断。

switch 只能有一个 default 分支

一个分支可以有多个值，多个case值中间使用英文逗号分隔。

分支还可以使用表达式，这时候switch语句后面不需要再跟判断变量。

`fallthrough`语法可以执行满足条件的case的下一个case，是为了兼容C语言中的case设计的

```go
package main

import (
	"fmt"
)

func add(a int) int {
	return a + 10
}

func switch_expression() {
	a := 5
	switch add(a) {
	case 15:
		fmt.Println("right")
	default:
		fmt.Println("wrong")
	}
}

// fallthrough 强行执行下一个case 
func fall_throth(age int) {
	fmt.Printf("您的年龄是%d,您可以: \n", age)
	switch {
	case age > 50:
		fmt.Println("出任国家首脑")
		fallthrough
	case age > 25:
		fmt.Println("生儿育女")
		fallthrough
	case age > 22:
		fmt.Println("结婚")
		fallthrough
	case age > 18:
		fmt.Println("开车")
		fallthrough
	case age > 16:
		fmt.Println("参加工作")
		fallthrough
	case age > 3:
		fmt.Println("上幼儿园")
	}
}

func main() {
	finger := 5
	switch finger {
	case 1:
		fmt.Println("大拇指")
	case 2:
		fmt.Println("食指")
	case 3:
		fmt.Println("中指")
	case 4:
		fmt.Println("无名指")
	case 5:
		fmt.Println("小拇指")
	default:
		fmt.Println("无效的")
	}

	num := 5
	switch num {
	case 1, 3, 5, 7, 9:
		fmt.Println("奇数")
	case 2, 4, 6, 8, 10:
		fmt.Println("偶数")
	default:
		fmt.Println("不是10以内的数据")
	}

	a := "a"
	switch a {
	case "a":
		fmt.Println(a)
	default:
		fmt.Println("not a")
	}

	switch {
	case add(5) > 0:
		fmt.Println("right")
	default:
		fmt.Println("wrong")
	}
	fall_throth(18)
}
```

### goto

通过标签进行代码间的无条件跳转，可以快速跳出循环、避免重复退出上有一定的帮助

双层嵌套的for循环要退出的时：

```go
func gotoDemo1() {
	var breakFlag bool
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				// 设置退出标签
				breakFlag = true
				break
			}
			fmt.Printf("%v-%v\n", i, j)
		}
		// 外层for循环判断
		if breakFlag {
			break
		}
	}
}
```

使用goto

```go
func gotoDemo2() {
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				// 设置退出标签
				goto breakFlag
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
    return
    // 标签
breakFlag:
    fmt.Println("结束循环")
}
```

案例2

```go
package main

import "fmt"

func basicGoto() {
	i := 4
MY_LABEL:
	i += 3
	i *= 2
	fmt.Println(i)
	if i > 200 {
		return
	}
	goto MY_LABEL
}

func ifGoto() {
	i := 4
	if i%2 == 0 {
		goto L1
	} else {
		goto L2 // 先使用，后定义
	}
L1:
	i += 3
	fmt.Println(i)

L2:
	i *= 3
	fmt.Println(i)
}

func main() {
}
```



### break

break 可以跳出 for、switch 和 select 的代码块

break 还可以在后面添加标签。表示退出某个标签对应的代码块，标签要求必须定义在对应的`for`、`switch`和 `select`的代码块上

```go
func breakDemo1() {
BREAKDEMO1:
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				break BREAKDEMO1
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
	fmt.Println("...")
}

```

### continue

结束本次循环，仅限在`for`循环内使用

在 `continue`语句后添加标签时，表示开始标签对应的循环。例如：

```go
func continueDemo() {
forloop1:
	for i := 0; i < 5; i++ {
		// forloop2:
		for j := 0; j < 5; j++ {
			if i == 2 && j == 2 {
				continue forloop1
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
}
```

### 九九乘法表

```go
package main

import "fmt"

func main() {
	for i := 1; i <= 9; i++ {
		for j := 1; j <= i; j++ {
			fmt.Printf("%d * %d = %d", j, i, i*j)
		}
		fmt.Println()
	}
}
```

