---
title: golang基础-变量和常量
lastmod: 2025-05-08T16:43:23+08:00
date: 2025-05-08T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-01.html
toc: true
---

## hello world

```go
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}

```

## 变量

标识符：由字母数字和下划线(`_`)组成，并且只能以字母和`_`开头

关键字

```
    break        default      func         interface    select
    case         defer        go           map          struct
    chan         else         goto         package      switch
    const        fallthrough  if           range        type
    continue     for          import       return       var
```

保留字

```     Constants:    true  false  iota  nil
    Constants:    true  false  iota  nil

        Types:    int  int8  int16  int32  int64  
                  uint  uint8  uint16  uint32  uint64  uintptr
                  float32  float64  complex128  complex64
                  bool  byte  rune  string  error

    Functions:   make  len  cap  new  append  copy  close  delete
                 complex  real  imag
                 panic  recover
```

### 声明变量

格式:

```
var 变量名 变量类型
```

例子:

```go
var name string
var age int
var is_ok bool
```

批量声明

```go
var (
	name string
    age int
    is_ok bool
)
```

### 变量的初始化

```go
var 变量名 类型 = 表达式
```

```go
var name string = "jin"

// 一次性初始化多个变量
var name,age = "jin",20
```

### 类型推导

有时候会将变量的类型省略，编译器会自动将右边的值来推导变量的类型完成初始化

```go
var name = "jin"
```

### 短变量声明

在函数内部，可以使用更简略的 `:=` 方式声明并初始化变量。

```go
name5 := "jin"
age5 := 20
```

### 匿名变量

如果想忽略某个值的时候，可以使用匿名变量，使用一个 `_` 下划线表示，匿名变量不占用命名空间，不会分配内存，所以匿名变量不会存在重复声明

```go
func foo()(int, string){
    return 10, "a"
}
func main(){
    x,_ := foo()
    _,y := foo()
    fmt.Println(x,y)
}
```

注意事项

1. 函数外的每个语句都必须以关键字开始(var、const、 func等)
2. := 不能使用在函数外
3. _多用于占位，表示忽略值

## 常量
相对于变量，常量是恒定不变的值，多用于定义程序运行期间不会改变的值，常量在定义的时候必须赋值，常量的值一旦赋值，就不能再改变。常量可以是数字、字符、字符串或布尔值。
```go
const pi = 3.1415
const e = 2.7182
```

也可以一起声明

```go
const (
    pi = 3.1415
    e = 2.7182
)
```

const同时声明多个常量时，如果省略了值则表示和上面一行的值相同。 例如：

```go
const (
    n1 = 100
    n2
    n3
)
```

常量不能作为变量的地址引用，也不能用于作为数组大小等需要动态值的地方。

在1.18版本之后go常量不使用编译器不会报错，但是会进行提示，

### iota

`iota`是go语言的常量计数器，只能在常量的表达式中使用。

`iota`在const关键字出现时将被重置为0。const中每新增一行常量声明将使`iota`计数一次(iota可理解为const语句块中的行索引)。 使用iota能简化定义，在定义枚举时很有用。

```go
const (
		n1 = iota //0
		n2        //1
		n3        //2
		n4        //3
	)
```



代码总结

```go
package main

import "fmt"

func foo() (int, string) {
	return 10, "a"
}

// 常量
// const pi = 3.1415
// const e = 2.7182

const (
	pi = 3.1415
	e  = 2.7182
)

const (
	n1 = 100
	n2
	n3
)

func main() {
	// 批量声明变量
	var (
		name1  string
		age1   int
		is_ok1 bool
	)
	// 标准格式声明变量
	var name string
	var age int
	var is_ok bool
	fmt.Println(name, age, is_ok)
	fmt.Println(name1, age1, is_ok1)

	// 变量的初始化
	var name2 string = "jin"
	var age2 int = 20
	fmt.Println(name2, age2)

	// 初始化多个变量
	var name3, age3 = "jin", 20
	fmt.Println(name3, age3)

	// 类型推导
	var name4 = "jin"
	var age4 = 20
	fmt.Println(name4, age4)

	// 短变量声明
	name5 := "jin"
	age5 := 20
	fmt.Println(name5, age5)

	// 匿名变量
	x, _ := foo()
	fmt.Println(x)
}
```
