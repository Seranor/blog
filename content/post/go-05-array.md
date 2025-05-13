---
title: golang基础-数组
lastmod: 2025-05-13T16:43:23+08:00
date: 2025-05-013T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-05.html
toc: true
---

## 数组

### 定义

数组是同一种数据类型元素的集合。 在Go语言中，数组从声明时就确定，使用时可以修改数组成员，但是数组大小不可变化。 基本语法：

```go
var a [3]int
```

数组定义：

```go
var 数组变量名 [元素数量]T
```

数组的长度必须是常量，并且长度是数组的一部分，定义之后长度不能变

数组的访问：通过下标进行访问，下标从0开始，最后一个元素下标是 len-1，访问越界之后会 panic

### 初始化

方法一

初始化数组时可以使用初始化列表来设置数组元素的值

```go
var arr1 [3]int // 初始化int类型的零值
var arr2 = [3]int{1,2} // 使用指定初始值完成初始化
var arr3 = [3]string{"北京","上海","深圳"}
```

方法二

自行推断数组的长度，使用 `[...]`

```go
func main() {
	var testArray [3]int
	var numArray = [...]int{1, 2}
	var cityArray = [...]string{"北京", "上海", "深圳"}
	fmt.Println(testArray)                          //[0 0 0]
	fmt.Println(numArray)                           //[1 2]
	fmt.Printf("type of numArray:%T\n", numArray)   //type of numArray:[2]int
	fmt.Println(cityArray)                          //[北京 上海 深圳]
	fmt.Printf("type of cityArray:%T\n", cityArray) //type of cityArray:[3]string
}
```

方法三

指定索引值初始化数组

```go
a := [...]int{1:1, 3:5} //[0 1 0 5]  类型是 [4]int
```

注意： [3]int  和 [4]int 的类型不是一样的

### 数组遍历

方法一

```go
func main() {
	var a = [...]string{"北京", "上海", "深圳"}
	for i := 0; i < len(a); i++ {
		fmt.Println(a[i])
	}
}

```

方法二

```go
func main() {
	var a = [...]string{"北京", "上海", "深圳"}
	for index, value := range a {
		fmt.Println(index, value)
	}
}
```

### 多维数组

数组中嵌套数组

以二维数组为例：

```go
func main() {
	a := [3][2]string{
		{"北京", "上海"},
		{"广州", "深圳"},
		{"成都", "重庆"},
	}
	fmt.Println(a) //[[北京 上海] [广州 深圳] [成都 重庆]]
	fmt.Println(a[2][1]) //支持索引取值:重庆
}
```

遍历

```go
func main() {
	a := [3][2]string{
		{"北京", "上海"},
		{"广州", "深圳"},
		{"成都", "重庆"},
	}
	for _, v1 := range a {
		for _, v2 := range v1 {
			fmt.Printf("%s\t", v2)
		}
		fmt.Println()
	}
}
```

关于 `...`  在多维数组中，只有第一层可以使用

```go
	var cityArr = [...][2]string{ // 只有最外层可以使用 ...
		{"北京", "上海"},
		{"广州", "深圳"},
		{"成都", "重庆"},
	}
	fmt.Println(cityArr)
```

### 数组是值类型

赋值和传参会复制整个数组。因此改变副本的值，不会改变本身的值。

```go
package main

import "fmt"

func main() {
	// 数组是值类型 会做完整的拷贝
	x := [3]int{1, 2, 3}
	fmt.Println(x)
	f1(x)
	fmt.Println(x)
	fmt.Println("-------------------------")
	for_range_array()
}

func f1(a [3]int) {
	a[0] = 100
	fmt.Println(a)
}

func for_range_array() {
	arr := [...]int{1, 2, 3}
	for i, ele := range arr {
		arr[i] += 8
		fmt.Printf("%d %d %d \n", i, arr[i], ele)
		ele += 1
		fmt.Printf("%d %d %d \n", i, arr[i], ele)
	}
	fmt.Println("-----------------")
	for i := 0; i < len(arr); i++ {
		fmt.Printf("%d %d \n", i, arr[i])
	}
}

```

**注意：**

1. 数组支持 “=="、”!=" 操作符，因为内存总是被初始化过的。
2. `[n]*T`表示指针数组，`*[n]T`表示数组指针 。