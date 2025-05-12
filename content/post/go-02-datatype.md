---
title: golang基础-基本数据类型
lastmod: 2025-05-09T16:43:23+08:00
date: 2025-05-09T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-02.html
toc: true
---

## 基本数据类型

Go语言中有丰富的数据类型，除了基本的整型、浮点型、布尔型、字符串外，还有数组、切片、结构体、函数、map、通道（channel）等。Go 语言的基本类型和其他语言大同小异。

### 整型

| 类型   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| uint8  | 无符号 8位整型 (0 到 255)                                    |
| uint16 | 无符号 16位整型 (0 到 65535)                                 |
| uint32 | 无符号 32位整型 (0 到 4294967295)                            |
| uint64 | 无符号 64位整型 (0 到 18446744073709551615)                  |
| int8   | 有符号 8位整型 (-128 到 127)                                 |
| int16  | 有符号 16位整型 (-32768 到 32767)                            |
| int32  | 有符号 32位整型 (-2147483648 到 2147483647)                  |
| int64  | 有符号 64位整型 (-9223372036854775808 到 9223372036854775807) |

其中 uint8 是byte型

uint	 32位操作系统上就是uint32，64位操作系统上就是uint64
int	   32位操作系统上就是int32，64位操作系统上就是int64
uintptr    无符号整型，用于存放一个指针

```go
package main

import (
	"fmt"
)

func main() {
	n := 10
	fmt.Printf("%b\n", n) // 十进制打印为二进制
	fmt.Printf("%d\n", n) // 打印十进制

	// 八进制
	m := 075
	fmt.Printf("%d\n", m)
	fmt.Printf("%o\n", m)

	// 十六进制
	f := 0xff
	fmt.Printf("%x\n", f) // 十六进制
	fmt.Printf("%d\n", f) // 十进制

	// uint8
	var age uint8 // 0-255
	// age = 256 超过255 编译器提示报错
	fmt.Println(age) // 整型的默认值是0
}
```

### 浮点型

Go语言支持两种浮点型数：`float32`和`float64`。这两种浮点型数据格式遵循`IEEE 754 `标准： `float32` 的浮点数的最大范围约为 `3.4e38`，可以使用常量定义：`math.MaxFloat32`。 `float64` 的浮点数的最大范围约为 `1.8e308`，可以使用一个常量定义：`math.MaxFloat64`。

打印浮点数时，可以使用`fmt`包配合动词`%f`，代码如下：

```go
package main
import (
        "fmt"
        "math"
)
func main() {
        fmt.Printf("%f\n", math.Pi)
        fmt.Printf("%.2f\n", math.Pi)
    
    	// 浮点数最大值
        fmt.Println(math.MaxFloat32)
        fmt.Println(math.MaxFloat64)
}
```

### 复数

complex64和complex128

```go
var c1 complex64
c1 = 1 + 2i
var c2 complex128
c2 = 2 + 3i
fmt.Println(c1)
fmt.Println(c2)
```

### 布尔值

布尔数据只有 true 和false两个值，默认值是false，不允许将整型转换为布尔型，布尔型无法参与数值运算，也无法与其他类型进行转换

```go
	var b bool
	fmt.Println(b)
	b = true
	fmt.Println(b)
```

### 字符类型

Go语言的字符有以下两种：

- 一种是 uint8 类型，或者叫 byte 型，代表了 ASCII 码的一个字符。
- 另一种是 rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。rune 类型等价于 int32 类型。

**byte 类型是 uint8 的别名，rune 类型是int32的别名**

**ASCII 码的一个字符占一个字节**

```go
//使用单引号 表示一个字符
var ch byte = 'A'
//在 ASCII 码表中，A 的值是 65,也可以这么定义
var ch byte = 65
//65使用十六进制表示是41，所以也可以这么定义 \x 总是紧跟着长度为 2 的 16 进制数
var ch byte = '\x41'
//65的八进制表示是101，所以使用八进制定义 \后面紧跟着长度为 3 的八进制数
var ch byte = '\101'

fmt.Printf("%c",ch)
```

**Unicode** 是 ASCII 的超集，它定义了 1,114,112 个代码点的代码空间。 Unicode 版本 10.0 涵盖 139 个现代和历史文本集（包括符文字母，但不包括 Klingon ）以及多个符号集。

Go语言同样支持 Unicode（UTF-8）, `用rune来表示`, 在内存中使用 int 来表示。

在书写 Unicode 字符时，需要在 16 进制数之前加上前缀`\u`或者`\U`。如果需要使用到 4 字节，则使用`\u`前缀，如果需要使用到 8 个字节，则使用`\U`前缀。

```go
var ch rune = '\u0041'
	var ch1 int64 = '\U00000041'
	//格式化说明符%c用于表示字符，%v或%d会输出用于表示该字符的整数，%U输出格式为 U+hhhh 的字符串。
	fmt.Printf("%c,%c,%U",ch,ch1,ch
```

Unicode 包中内置了一些用于测试字符的函数，这些函数的返回值都是一个布尔值，如下所示（其中 ch 代表字符）：

- 判断是否为字母：unicode.IsLetter(ch)
- 判断是否为数字：unicode.IsDigit(ch)
- 判断是否为空白符号：unicode.IsSpace(ch)

```go
package main

import "fmt"

// 字符
func main() {
	// byte uint8 的别名 ASCII码
	// rune int32 的别名
	var c1 byte = 'c'
	var c2 rune = 'c'
	fmt.Println(c1, c2)
	fmt.Printf("c1:%T c2:%T\n", c1, c2)

	s := "hello世界"
	for _, r := range s {
		fmt.Printf("%c\n", r)
	}
	for i := 0; i < len(s); i++ {
		fmt.Printf("%c\n", s[i])
	}
}
	
```

修改字符串

要修改字符串，需要先将其转换成`[]rune`或`[]byte`，完成后再转换为`string`。无论哪种转换，都会重新分配内存，并复制字节数组。

```go
func changeString() {
	s1 := "big"
	// 强制类型转换
	byteS1 := []byte(s1)
	byteS1[0] = 'p'
	fmt.Println(string(byteS1))

	s2 := "白萝卜"
	runeS2 := []rune(s2)
	runeS2[0] = '红'
	fmt.Println(string(runeS2))
}
```

### 字符串

go语言中的字符串内部实现使用`UTF-8`编码，字符串的值为双引号("")中的内容，可以直接添加非ASCII码的字符

```go
s1 := "hello"
s2 := "你好"
```

转义符

| 转义符 | 含义                               |
| :----- | :--------------------------------- |
| `\r`   | 回车符（返回行首）                 |
| `\n`   | 换行符（直接跳到下一行的同列位置） |
| `\t`   | 制表符                             |
| `\'`   | 单引号                             |
| `\"`   | 双引号                             |
| `\\`   | 反斜杠                             |

```go
package main
import (
    "fmt"
)
func main() {
    fmt.Println("str := \"c:\\Code\\lesson1\\go.exe\"")
}
```

多行字符串

使用反引号 ` 字符表示

```go 
s1 := `aaa
bbb
ccc
`
// 原样输出
```



字符串常用操作

| 方法                                | 介绍           |
| :---------------------------------- | :------------- |
| len(str)                            | 求长度         |
| +或fmt.Sprintf                      | 拼接字符串     |
| strings.Split                       | 分割           |
| strings.contains                    | 判断是否包含   |
| strings.HasPrefix,strings.HasSuffix | 前缀/后缀判断  |
| strings.Index(),strings.LastIndex() | 子串出现的位置 |
| strings.Join(a[]string, sep string) | join操作       |

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// 字符串长度
	s1 := "hello"
	s2 := "hello世界"
	fmt.Println(len(s1))
	fmt.Println(len(s2))

	// 拼接字符串
	fmt.Println(s1 + s2)
	s3 := fmt.Sprintf("%s - %s", s1, s2)
	fmt.Println(s3)

	// 字符串的分割
	s4 := "how do you do"
	fmt.Println(strings.Split(s4, " "))
	fmt.Printf("切割之后的类型：%T\n", strings.Split(s4, " ")) // 字符串切片

	// 是否包含
	fmt.Println(strings.Contains(s4, "do"))

	// 前缀
	fmt.Println(strings.HasPrefix(s4, "how"))

	// 后缀
	fmt.Println(strings.HasSuffix(s4, "how"))

	// 判断子串的位置
	fmt.Println(strings.Index(s4, "do"))

	// 判断子串最后出现的位置
	fmt.Println(strings.LastIndex(s4, "do"))

	// join 拼接
	s5 := []string{"how", "do", "you", "do"}
	fmt.Println(s5)
	fmt.Println(strings.Join(s5, "+"))

}
```

拼接方式

```go
package main

import (
	"fmt"
	"strconv"
	"strings"
	"unicode/utf8"
)

func string_impl() {
	s1 := "My name is 李四"
	arr := []byte(s1)
	brr := []rune(s1)
	fmt.Printf("last byte %d\n", arr[len(arr)-1])
	fmt.Printf("last byte %c\n", arr[len(arr)-1])

	fmt.Printf("last rune %d\n", arr[len(brr)-1])
	fmt.Printf("last rune %c\n", arr[len(brr)-1])

	L := len(s1)
	fmt.Printf("string len %d byte array len %d rune array len %d\n", L, len(arr), len(brr))

	for _, ele := range s1 {
		fmt.Printf("%c\n", ele)
	}

	fmt.Println(utf8.RuneCountInString(s1), len([]rune(s1)))

}

func string_join() {
	s1 := "Hello"
	s2 := "how"
	s3 := "are"
	s4 := "you"
	merged := s1 + " " + s2 + " " + s3 + " " + s4 // 方式一
	fmt.Println(merged)

	merged = fmt.Sprintf("%s %s %s %s", s1, s2, s3, s4) // 方式二
	fmt.Println(merged)

	merged = strings.Join([]string{s1, s2, s3, s4}, " ") // 方式三
	fmt.Println(merged)

	sb := strings.Builder{} // 方式四
	sb.WriteString(s1)
	sb.WriteString(" ")
	sb.WriteString(s2)
	sb.WriteString(" ")
	sb.WriteString(s3)
	sb.WriteString(" ")
	sb.WriteString(s4)
	merged = sb.String()
	fmt.Println(merged)
}

func string_op() {
	s := "born to win, born to die."
	fmt.Printf("sentence length %d\n", len(s))
	fmt.Printf("\"s\" length %d\n", len("s"))
	fmt.Printf("\"中\" length %d\n", len("中"))

	arr := strings.Split(s, " ")
	fmt.Printf("arr[3]=%s\n", arr[3])

	fmt.Printf("contain die %t \n", strings.Contains(s, "die"))

	fmt.Printf("first index of bron %d\n", strings.Index(s, "born"))
	fmt.Printf("last index of bron %d\n", strings.LastIndex(s, "born"))

	fmt.Printf("begin with born %t\n", strings.HasPrefix(s, "born"))
	fmt.Printf("end with die. %t\n", strings.HasSuffix(s, "die."))

}
func string_other_convert() {
	var err error
	var i int = 8
	var i64 int64 = int64(i)
	var s string = strconv.Itoa(i)
	s = strconv.FormatInt(i64, 10)

	i, err = strconv.Atoi(s)
	if err != nil {
		fmt.Println(err)
	}
	i64, err = strconv.ParseInt(s, 10, 64)

	fmt.Println(i)
	fmt.Println(s)
	fmt.Println(i64)

	var f float64 = 8.123456789
	s = strconv.FormatFloat(f, 'f', 2, 64) // %.2f
	fmt.Println(s)
}
func main() {
	// string_impl()
	// string_join()
	// string_op()
	string_other_convert()
}

```

判断一个字符串中的汉字的数量

```go
func contString() {
	var s1 = "hello世界和平"
	var count int
	for _, ch := range s1 {
		if unicode.Is(unicode.Han, ch) {
			count++
		}
	}
	fmt.Printf("字符串中汉字的数量是: %d\n", count)

	// str := "hello世界和平"
	// temp := []rune(str)
	// var count int
	// for _, v := range temp {
	// 	if v > 256 {
	// 		count++
	// 		fmt.Println(string(v))
	// 	}
	// }
	// fmt.Println(count)
}
```



### 类型转换

Go语言中只有强制类型转换，没有隐式类型转换。该语法只能在两个类型之间支持相互转换的时候使用。

强制类型转换的基本语法如下：

```go
T(表达式)
```

其中，T表示要转换的类型。表达式包括变量、复杂算子和函数返回值等