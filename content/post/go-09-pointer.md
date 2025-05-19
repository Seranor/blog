---
title: golang基础-指针
lastmod: 2025-05-15T16:43:23+08:00
date: 2025-05-015T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-09.html
toc: true
---

### 指针

区别于C/C++中的指针，Go语言中的指针不能进行偏移和运算，是安全指针。

指针（pointer）在Go语言中以被拆分为两个核心概念：

- 类型指针，允许对这个指针类型的数据进行修改，传递数据可以直接使用指针，而无须拷贝数据，类型指针不能进行偏移和运算。
- 切片，由指向起始元素的原始指针、元素数量和容量组成。

受益于这样的约束和拆分，Go语言的指针类型变量即拥有指针高效访问的特点，又不会发生指针偏移，从而避免了非法修改关键性数据的问题。

同时，`垃圾回收`也比较容易对不会发生偏移的指针进行检索和回收。

切片比原始指针具备更强大的特性，而且更为安全。

切片在发生越界时，运行时会报出宕机，并打出堆栈，而原始指针只会崩溃。

### 指针地址和指针类型

每个变量在运行时都有一个地址，这个地址代表变量在内存中的位置。

使用 `&` 字符放在变量前面对变量进行取地址操作。

Go语言中的值类型（int、float、bool、string、array、struct）都有对应的指针类型，如：`*int`、`*int64`、`*string`等。

取变量指针的语法如下：

```go
ptr := &v // v的类型为T
```

- v:代表被取地址的变量，类型为`T`
- ptr:用于接收地址的变量，ptr的类型就为`*T`，称做T的指针类型。*代表指针。

```go
func main() {
	a := 10
	b := &a
	fmt.Printf("a:%d ptr:%p\n", a, &a) // a:10 ptr:0xc00001a078
	fmt.Printf("b:%p type:%T\n", b, b) // b:0xc00001a078 type:*int
	fmt.Println(&b)                    // 0xc00000e018
}
```

%p 需要搭配 &

![取变量地址图示](https://seranor-1251900471.cos.ap-chengdu.myqcloud.com/typora/ptr.png)

### 指针取值

在对普通变量使用&操作符取地址后会获得这个变量的指针，然后可以对指针使用*操作，也就是指针取值，代码如下。

```go
func main() {
	//指针取值
	a := 10
	b := &a // 取变量a的地址，将指针保存到b中
	fmt.Printf("type of b:%T\n", b)
	c := *b // 指针取值（根据指针去内存取值）
	fmt.Printf("type of c:%T\n", c)
	fmt.Printf("value of c:%v\n", c)
}
```

取地址操作符`&`和取值操作符`*`是一对互补操作符，`&`取出地址，`*`根据地址取出地址指向的值。

变量、指针地址、指针变量、取地址、取值的相互关系和特性如下：

- 对变量进行取地址（&）操作，可以获得这个变量的指针变量。
- 指针变量的值是指针地址。
- 对指针变量进行取值（*）操作，可以获得指针变量指向的原变量的值。

```go
package main

import "fmt"

func main() {
	var cat int = 1
	var str string = "学习教程"
	ptr1 := &cat
	fmt.Printf("%p,%p\n", &cat, &str) // 内存地址
	fmt.Println(*ptr1)

	/*
		变量、指针和地址三者的关系是，每个变量都拥有地址，指针的值就是地址
	*/
	var room int = 18
	var ptr2 = &room
	fmt.Printf("%p\n", &room)
	fmt.Printf("%T,%p\n", ptr2, ptr2)

	fmt.Println("指针地址", ptr2)
	fmt.Println("指针地址的值", *ptr2)
}
```



```go
package main

import "fmt"

func modify1(x int) {
	x = 100
}

func modify2(y *int) {
	*y = 200
}

func agr1(a, b *int) {
	*a = *a + *b
	*b = 888
}
func arg2(a, b int) {
	a = a + b
}
func return1(a, b int) int {
	a = a + b
	c := a
	return c
}
func main() {
	a := 1
	modify1(a)
	fmt.Println(a)
	modify2(&a)
	fmt.Println(a)

	c := 5
	d := 6
	// arg2(c, d)
	// agr1(&c, &d)
	e := return1(c, d)

	fmt.Println(c, d, e)

}

```

### new()创建指针

```go
new(类型)
```

```go
str := new(string)
*str = "go"
fmt.Println(*str)
```

**new() 函数可以创建一个对应类型的指针，创建过程会分配内存，被创建的指针指向默认值。**
