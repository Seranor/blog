---
title: Golang基础
lastmod: 2021-10-20T16:43:23+08:00
date: 2021-10-20T11:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/golang-01.html
toc: true
---

## hello world

```go
package main

import "fmt"

func main() {
	fmt.Println("hello world1")
}
```

<!-- more -->

## 变量和常量

### 定义变量

#### 单声明变量

```go
// 指定变量类型，声明后若不赋值，使用默认值
var name1 string
name1 ="bob"

// 根据值自行判定变量类型(类型推断Type inference)如果一个变量有一个初始值，Go将自动能够使用初始值来推断该变量的类型。因此，如果变量具有初始值，则可以省略变量声明中的类型。
var name2 ="jason"

// 省略var, 注意 :=左侧的变量不应该是已经声明过的(多个变量同时声明时，至少保证一个是新变量)，否则会导致编译错误(简短声明)
var a int = 10
var b = 10
c : = 10


```

```go
package main

import "fmt"

func main() {
	var name1 string
	name1 = "bob"
	var name2 = "jason"
	name3 := "aaa"
	fmt.Println(name1, name2, name3)

}
```

#### 多声明变量

```go
// 以逗号分隔，声明与赋值分开，若不赋值，存在默认值
var name1, name2, name3 type
name1, name2, name3 = v1, v2, v3

// 直接赋值，下面的变量类型可以是不同的类型
var name1, name2, name3 = v1, v2, v3

// 集合类型
var (
    name string
    age int
)
```

#### 注意

- 变量必须先定义才能使用
- go 语言是静态语言，要求变量的类型和赋值的类型必须一致
- 变量名不能冲突。(同一个作用于域内不能冲突)
- 简短定义方式，左边的变量名至少有一个是新的
- 简短定义方式，不能定义全局变量
- 变量的零值。也叫默认值
- 变量定义了就要使用，否则无法通过编译

### 常量

#### 定义

```go
常量是一个简单值的标识符，在程序运行时，不会被修改的量。关键字 const

显式类型定义： const b string = "abc"
隐式类型定义： const b = "abc"

// 常量可以作为枚举，常量组
const (
    Unknown = 0
    Female = 1
    Male = 2
)
```

#### iota

```go
iota 特殊常量，可以认为是一个可以被编译器修改的常量
iota 可以被用作枚举值

  const (
      a = iota
      b = iota
      c = iota
  )

第一个 iota 等于 0，每当 iota 在新的一行被使用时，它的值都会自动加 1；所以 a=0, b=1, c=2 可以简写为如下形式：
  const (
      a = iota
      b
      c
  )
```

```go
package main

import "fmt"

func main() {
    const (
            a = iota   //0
            b          //1
            c          //2
            d = "ha"   //独立值，iota += 1
            e          //"ha"   iota += 1
            f = 100    //iota +=1
            g          //100  iota +=1
            h = iota   //7,恢复计数
            i          //8
    )
    fmt.Println(a,b,c,d,e,f,g,h,i)
}
```

### 匿名变量

```go
func test() (int, error)  {
	return 0, nil
}

func main() {
	_, err := test()
	if err != nil {
		fmt.Println("函数调用成功")
	}
}
```

### 变量的作用域

```go
package main

import "fmt"

// var a := 20 全局变量不能使用 := 的方式
var a = 20     // 全局变量没有被使用不会报错
// var c = 20  // 可以与局部变量名称相同

func main(){
  // 局部变量
  c := 10
  fmt.Println(c)

  sex := "Female"
	if sex == "Female"{
		outStr := "女"
		fmt.Println(outStr)  // outStr只在该作用域下
	}
	// fmt.Println(outStr) 无法使用 outStr 无法运行
}
```

## 基本数据类型

![image-20220623085914730](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220623085914730.png)

### bool 类型

布尔型的值只可以是常量 true 或者 false

```go
var b bool = true
```

### 数值类型

整数

```go
int8   有符号 8 位整型 (-128 到 127) 长度：8bit
int16  有符号 16 位整型 (-32768 到 32767)
int32  有符号 32 位整型 (-2147483648 到 2147483647)
int64  有符号 64 位整型 (-9223372036854775808 到 9223372036854775807)
uint8  无符号 8 位整型 (0 到 255) 8位都用于表示数值：
uint16 无符号 16 位整型 (0 到 65535)
uint32 无符号 32 位整型 (0 到 4294967295)
uint64 无符号 64 位整型 (0 到 18446744073709551615)
```

浮点型

```go
float32  32位浮点型数
float64  64位浮点型数
```

其他

```go
byte 等于 uint8
rune 等于 int32
uint  32 或 64 位
```

### 字符

```go
Golang中没有专门的字符类型，如果要存储单个字符(字母)，一般使用byte来保存。
字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。也就是说对于传统的字符串是由字符组成的，而Go的字符串不同，它是由字节组成的

字符常量只能使用单引号括起来，例如：var a byte = 'a' var a int = 'a'
字符本质是一个数字， 可以进行加减乘除
```

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var a byte
	a = 'a'
	// 输出ascii对应码值
	fmt.Println(a)
	fmt.Printf("a=%c", a)

 // 这里注意一下 1. a+1可以和数字计算 2.a+1的类型是32 3. int类型可以直接变成字符
	fmt.Println(reflect.TypeOf(a+1))
	fmt.Printf("a+1=%c", a+1)
}
```

### 字符串

```go
字符串就是一串固定长度的字符连接起来的字符序列
Go 的字符串是由单个字节连接起来的
Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本
```

### 数据类型的转换

#### 简单转换

```go
// 浮点数
a := 5.0

// 转换为int类型
b := int(a)

// Go允许在底层结构相同的两个类型之间互转。例如：
// IT类型的底层是int类型
type IT int

// a的类型为IT，底层是int
var a IT = 5

// 将a(IT)转换为int，b现在是int类型
b := int(5)

// 将b(int)转换为IT，c现在是IT类型
c := IT(b)

var a int32 = 1
var b int64 = 3
b = int64(a)
fmt.Println(a, b)

/*
不是所有数据类型都能转换的，例如字母格式的string类型"abcd"转换为int肯定会失败
低精度转换为高精度时是安全的，高精度的值转换为低精度时会丢失精度。例如int32转换为int16，float32转换为int
这种简单的转换方式不能对int(float)和string进行互转，要跨大类型转换，可以使用strconv包提供的函数
*/
```

#### `strconv`

`int`转换为`string`: Itoa()

`string`转换为`int`: Atoi()

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	// int 转 string Itoa
	a := 56
	strconv_a := strconv.Itoa(int(a))
	fmt.Printf("a的值是:%v 类型是: %T\n", strconv_a, strconv_a)

	// string 转 int Aoti
	b := "12"
	strconv_b, _ := strconv.Atoi(b) // Atoi 会返回 int err 用匿名 _ 接收结果是否成功 或者可以用该变量进行判断是否转换成功
	fmt.Printf("a的值是:%v 类型是: %T\n", strconv_b, strconv_b)

  // Atoi 转换失败
  i,err := strconv.Atoi("a")
  if err != nil {
      println("converted failed")
  }
  //由于string可能无法转换为int，所以这个函数有两个返回值：第一个返回值是转换成int的值，第二个返回值判断是否转换成功。
}
```

Parse 类函数用于转换字符串为给定类型的值：ParseBool()、ParseFloat()、ParseInt()、ParseUint()。

```go
b, err := strconv.ParseBool("true")
f, err := strconv.ParseFloat("3.1415", 64)
i, err := strconv.ParseInt("-42", 10, 64)
u, err := strconv.ParseUint("42", 10, 64)
```

```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (uint64, error)

说明：
  a. bitSize参数表示转换为什么位的int/uint，有效值为0、8、16、32、64。当bitSize=0的时候，表示转换为int或uint类型。例如bitSize=8表示转换后的值的类型为int8或uint8。
  b. base参数表示以什么进制的方式去解析给定的字符串，有效值为0、2-36。当base=0的时候，表示根据string的前缀来判断以什么进制去解析：0x开头的以16进制的方式去解析，0开头的以8进制方式去解析，其它的以10进制方式解析
```

Format 类函数，将给定类型格式化为 string 类型：FormatBool()、FormatFloat()、FormatInt()、FormatUint()

```go
s := strconv.FormatBool(true)
s := strconv.FormatFloat(3.1415, 'E', -1, 64)
s := strconv.FormatInt(-42, 16) //表示将-42转换为16进制数，转换的结果为-2a。
s := strconv.FormatUint(42, 16)

第二个参数base指定将第一个参数转换为多少进制，有效值为2<=base<=36。当指定的进制位大于10的时候，超出10的数值以a-z字母表示。例如16进制时，10-15的数字分别使用a-f表示，17进制时，10-16的数值分别使用a-g表示
```

## 运算符

## 字符串的基本操作

### len 求长度

```go
package main

import "fmt"

func main() {
	// 字符串的基本操作
	// 求字符串的长度
	var name string = "hello,世界"

	// Unicode 字符集 存储是时候需要 utf8 编码规则, utf8 是一个动态的编码规则
	// utf8 编码还可以用一个字节表示英文
	fmt.Println(len(name)) // hello, 6个字节 一个中文占3个字节

	// 类型转换
	name_arr := []rune(name)
	fmt.Println(len(name_arr)) // 长度就是 8 了

}
```

### 转义字符

| 转义字符 | 意义                                 | ASCII 码值（十进制） |
| -------- | ------------------------------------ | -------------------- |
| \n       | 换行(LF) ，将当前位置移到下一行开头  | 010                  |
| \r       | 回车(CR) ，将当前位置移到本行开头    | 013                  |
| \t       | 水平制表(HT) （跳到下一个 TAB 位置） | 009                  |
| \\\      | 代表一个反斜线字符''\'               | 092                  |
| \\'      | 代表一个单引号（撇号）字符           | 039                  |
| \\"      | 代表一个双引号字符                   | 034                  |
| \?       | 代表一个问号                         | 063                  |

```go
date1 := "2022\\06\\06"
date2 := `2022\06\06`

`` 符号内所见即所得
```

### 子串相关操作

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// 是否包含某个子串
	var name string = "test:\"这是测试\""
	res := strings.Contains(name, "测试")
	fmt.Println(res)

	// 返回字符串所在的位置
	fmt.Println(strings.Index(name, "测试"))

	// 统计出现的次数
	fmt.Println(strings.Count(name, "t"))

	// 前缀和后缀
	fmt.Println(strings.HasPrefix(name, "t"))  // 是否以 t 开头
	fmt.Println(strings.HasSuffix(name, "\"")) // 是否以 " 结尾

  	// 大小写转换
	fmt.Println(strings.ToUpper(name)) // 字母全变成大写
	fmt.Println(strings.ToLower(name)) // 字母全变成小写

	// 字符串的比较
	/*
		字符的比较就是 ascii 的比较
		a < b 返回 -1
		a = b 返回 0
		a > b 返回 1
	*/
	fmt.Println(strings.Compare("ab", "b")) // -1 比较的是第一个字符

	// 去掉空格和指定字符串
	fmt.Println(strings.TrimSpace(" Test "))
	fmt.Println(strings.TrimLeft("test", "t")) // 去掉左边的 t
	fmt.Println(strings.Trim("test", "t"))     // 去掉左右两边的 t

	// Split 方法
	fmt.Println(strings.Split("hello,world", ",")) // [hello world]

	// 合并 join 方法将字符串数组连接起来
	arrs := strings.Split("hello,world", ",")
	fmt.Println(strings.Join(arrs, "-")) // hello-world

	// 字符串替换
	fmt.Println(strings.Replace("age: 18, phone: 1888", "18", "19", 1))
	// age: 19, phone: 1888, 准备替换的字符串，需要被替换的子串，替换为什么子串，替换次数
}
```

### 格式化输入输出

```go

```

| 动词   | 描述                        |
| ------ | --------------------------- |
| `%v`   | 缺省格式                    |
| `%#v`  | go 语法打印                 |
| `%T`   | 类型打印                    |
| `%d`   | 十进制                      |
| `%+d`  | 必须显示正负符号            |
| `%4d`  | Pad 空格(宽度为 4，右对齐)  |
| `%-4d` | Pad 空格 (宽度为 4，左对齐) |
| `%b`   | 二进制                      |
| `%o`   | 八进制                      |
| `%x`   | 16 进制，小写               |
| `%c`   | 字符                        |
| `%q`   | 有引号的字符                |
| `%U`   | Unicode                     |
| `%#U`  | Unicode 有引号              |
| `%e`   | 科学计数                    |
| `%f`   | 十进制小数                  |
| `%s`   | 字符串原样输出              |
| `%6s`  | 宽度为 6，右对齐            |

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	// printf 格式化
	name := "lxx"
	age := 18
	fmt.Println("name:"+name+", age:"+strconv.Itoa(age))
	fmt.Printf("name:%T, age: %T\n", name, age)
	fmt.Printf("name:%s, age:%x,\n ", name, age)
	desc := fmt.Sprintf("name:%s, age:%x,\n ", name, age)
	fmt.Println(desc)

	data := 65
	fmt.Printf("%q\n", data)
	fmt.Printf("%e", 65.1)

	// 输入
	var n string
	var a int
	//fmt.Println("请输入你的姓名和年龄:")
	//fmt.Scanln(&n, &a)
	//fmt.Println(n, a)

	//通过scanf输入
	fmt.Println("请输入你的姓名和年龄:")
	//输入的内容必须符合指定的格式
	fmt.Scanf("请输入你的姓名和年龄:%s %d", &n, &a)
	fmt.Println(n, a)

}
```

## 条件和循环语句

go 语言的常用控制流程有 if 和 for，没有 while， 而 switch 和 goto 是为了简化代码，降低重复代码，属于扩展的流程控制

### `if`语句

结构

```go
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
}

if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
} else {
  /* 在布尔表达式为 false 时执行 */
}

if 布尔表达式1 {
   /* 在布尔表达式1为 true 时执行 */
} else if 布尔表达式2{
   /* 在布尔表达式1为 false ,布尔表达式2为true时执行 */
} else {
   /* 在上面两个布尔表达式都为false时，执行*/
}


```

```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 10

   /* 使用 if 语句判断布尔表达式 */
   if a < 20 {
       /* 如果条件为 true 则执行以下语句 */
       fmt.Printf("a 小于 20\n" )
   }
   fmt.Printf("a 的值为 : %d\n", a)
}
```

变体

```go
package main

import (
    "fmt"
)

func main() {
    if num := 10; num % 2 == 0 { //checks if number is even
        fmt.Println(num,"is even")
    }  else {
        fmt.Println(num,"is odd")
    }
}

```

```go
/*
  if err := Connect(); err != nil {

  }
*/

package main

import (
	"errors"
	"fmt"
)

func test() error {
	return errors.New("error")
}

func main() {
	if err := test(); err != nil {
		fmt.Println("error happen")
	}
	fmt.Println(err) //此处会报错
}
```

### `for`循环

```go
// 和 C 语言的 for 一样
for init; condition; post { }

// 和 C 的 while 一样
for condition { }

// 和 C 的 for(;;) 一样
for { }

init： 一般为赋值表达式，给控制变量赋初值
condition： 关系表达式或逻辑表达式，循环控制条件
post： 一般为赋值表达式，给控制变量增量或减量
```

for 循环的 range 格式可以对 slice、map、数组、字符串等进行迭代循环

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

```go
package main

import "fmt"

func main() {
        strings := []string{"hello", "world"}
        for i, s := range strings {
                fmt.Println(i, s)
        }


        numbers := [6]int{1, 2, 3, 5}
        for i,x:= range numbers {
                fmt.Printf("第 %d 位 x 的值 = %d\n", i,x)
        }
}
```

### `goto`语句

```go
Go 语言的 goto 语句可以无条件地转移到过程中指定的行
goto 语句通常与条件语句配合使用。可用来实现条件转移， 构成循环，跳出循环体等功能
但是，在结构化程序设计中一般不主张使用 goto 语句， 以免造成程序流程的混乱，使理解和调试程序都产生困难

goto 语法格式如下：
goto label;
..
.
label: statement;
```

![image.png](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/1589475164138-889bd94c-c943-40e2-97da-69cedcb3e209.png)

```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 10

   /* 循环 */
   LOOP: for a < 20 {
      if a == 15 {
         /* 跳过迭代 */
         a = a + 1
         goto LOOP
      }
      fmt.Printf("a的值为 : %d\n", a)
      a++
   }
}
```

使用 goto 退出多层循环

```go
package main
import "fmt"
func main() {
    var breakAgain bool
    // 外循环
    for x := 0; x < 10; x++ {
        // 内循环
        for y := 0; y < 10; y++ {
            // 满足某个条件时, 退出循环
            if y == 2 {
                // 设置退出标记
                breakAgain = true
                // 退出本次循环
                break
            }
        }
        // 根据标记, 还需要退出一次循环
        if breakAgain {
                break
        }
    }
    fmt.Println("done")
}
```

使用 goto 优化

```go
package main

import "fmt"

func main() {
    for x := 0; x < 10; x++ {
        for y := 0; y < 10; y++ {
            if y == 2 {
                // 跳转到标签
                goto breakHere
            }
        }
    }
    // 手动返回, 避免执行进入标签
    return
    // 标签
breakHere:
    fmt.Println("done")
}
```

### `swatch`语句

switch 语句用于基于不同条件执行不同动作，每一个 case 分支都是唯一的，从上至下逐一测试，直到匹配为止

```go
switch var1 {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
}
```

```go
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var grade string = "B"
   var marks int = 90

   switch marks {
      case 90: grade = "A"
      case 80: grade = "B"
      case 50,60,70 : grade = "C"
      default: grade = "D"
   }

   switch {
      case grade == "A" :
         fmt.Printf("优秀!\n" )
      case grade == "B", grade == "C" :
         fmt.Printf("良好\n" )
      case grade == "D" :
         fmt.Printf("及格\n" )
      case grade == "F":
         fmt.Printf("不及格\n" )
      default:
         fmt.Printf("差\n" );
   }
   fmt.Printf("你的等级是 %s\n", grade );
}
```

switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。

Type Switch 语法格式如下：

```go
switch x.(type){
    case type:
       statement(s);
    case type:
       statement(s);
    /* 你可以定义任意个数的case */
    default: /* 可选 */
       statement(s);
}
```

```go
package main

import "fmt"

func main() {
   var x interface{}

   switch i := x.(type) {
      case nil:
         fmt.Printf(" x 的类型 :%T",i)
      case int:
         fmt.Printf("x 是 int 型")
      case float64:
         fmt.Printf("x 是 float64 型")
      case func(int) float64:
         fmt.Printf("x 是 func(int) 型")
      case bool, string:
         fmt.Printf("x 是 bool 或 string 型" )
      default:
         fmt.Printf("未知型")
   }
}
```

一分支多值

```go
// 不同的 case 表达式使用逗号分隔。
var a = "mum"
switch a {
    case "mum", "daddy":
    fmt.Println("family")
}
```

```go
var r int = 11
switch {
    case r > 10 && r < 20:
    fmt.Println(r)
}
```

## 复杂数据类型

### 数组

```go
数组是具有相同唯一类型的一组已编号且长度固定的数据项序列，这种类型可以是任意的原始类型例如整形、字符串或者自定义类型

相对于去声明 number0, number1, ..., number99 的变量，使用数组形式 numbers[0], numbers[1] ..., numbers[99] 更加方便且易于扩展

数组元素可以通过索引（位置）来读取（或者修改），索引从 0 开始，第一个元素索引为 0，第二个索引为 1，以此类推
```

声明和初始化

```go
var balance [10] float32
var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}

初始化数组中 {} 中的元素个数不能大于 [] 中的数字。如果忽略 [] 中的数字不设置数组大小，Go 语言会根据元素的个数来设置数组的大小：
var balance = []float32{1000.0, 2.0, 3.4, 7.0, 50.0}
balance[4] = 50.0

// 数组的其他创建方式
var a [4] float32 // 等价于：var arr2 = [4]float32{}
fmt.Println(a) // [0 0 0 0]
var b = [5] string{"ruby", "王二狗", "rose"}
fmt.Println(b) // [ruby 王二狗 rose  ]
var c = [5] int{'A', 'B', 'C', 'D', 'E'} // byte
fmt.Println(c) // [65 66 67 68 69]
d := [...] int{1,2,3,4,5} // 根据元素的个数，设置数组的大小
fmt.Println(d) //[1 2 3 4 5]
e := [5] int{4: 100} // [0 0 0 0 100]
fmt.Println(e)
f := [...] int{0: 1, 4: 1, 9: 1} // [1 0 0 0 1 0 0 0 0 1]
fmt.Println(f)
```

访问数组

```go
package main

import "fmt"

func main() {
   var n [10]int /* n 是一个长度为 10 的数组 */
   var i,j int

   /* 为数组 n 初始化元素 */
   for i = 0; i < 10; i++ {
      n[i] = i + 100 /* 设置元素为 i + 100 */
   }

   /* 输出每个数组元素的值 */
   for j = 0; j < 10; j++ {
      fmt.Printf("Element[%d] = %d\n", j, n[j] )
   }
}
```

```go
package main

import (
	"fmt"
)

func printArray(toPrint [5]string) {
	//[]string这是切片类型
	toPrint[0] = "bobby"
	fmt.Println(toPrint)
}

func main() {
	//go语言中的数组和python的list可以对应起来理解，slice和python的list更像
	//静态语言中的数组： 1. 大小确定 2. 类型一致
	//数组的申明
	//var courses [10] string
	//var courses = [5]string{"django", "scrapy", "tornado"}
	course := [5]string{"django", "scrapy", "tornado"}
	//静态语言要求严格， 动态语言是一门动态类型的

	//1. 修改值， 取值： 删除值， 添加某一个值， 数组一开始就要指定大小
	//取值， 修改值
	fmt.Println(course[0])
	//修改值
	course[0] = "django3"
	fmt.Println(course)

	//数组的另一种创建方式
	//var a [4] float32
	var a = [4]float32{1.0}
	fmt.Println(a)

	var c = [5] int{'A', 'B'}
	fmt.Println(c)

	//首次接触到了省略号
	d := [...]int{1,2,3,4,5}
	fmt.Println(d)

	e := [5]int{4:100}
	fmt.Println(e)

	f := [...]int{0:1, 4:1, 9:100}
	fmt.Println(f)

	//数组操作第一种场景： 求长度
	fmt.Println(len(f))
	//数组操作第二种场景： 遍历数组

	for i, value := range course {
		fmt.Println(i, value)
	}

	//使用for range求和
	sum := 0
	for _, value := range f {
		sum += value
	}
	fmt.Println(sum)

	//使用for语句也可以遍历数组
	sum = 0
	for i := 0; i<len(course); i++{
		sum += f[i]
	}
	fmt.Println(sum)

	//数组是值类型
	courseA := [3]string{"django", "scrapy", "tornado"}
	courseB := [...]string{"django1", "scrapy1", "tornado1", "python+go", "asyncio"}
	//courseA和courseB应该是同一种类型， 都是数组类型
	//在go语言中，courseA和courseB都是数组，但是不是同一种类型
	fmt.Printf("%T\n", courseA)
	fmt.Printf("%T\n", courseB)
	//如果courseA和courseB是一种类型的话 为什么前面要加一个数组， 长度不一样的数组类型是不一样
	//正是基于这些，在go语言中函数传递参数的时候，数组作为参数 实际调用的时候是值传递
	printArray(courseB)
	fmt.Println(courseB)
}
```

### 切片

```go
package main

import "fmt"

func replace(mySlice []string) {
	mySlice[0] = "bobby"
}

func main() {
	//什么是切片
	//数组有一个很大的问题：大小确定，不能修改 - 切片 - 动态数组
	//var identifier []type
	//第一种方法： var courses []string //定义了一个切片
	//var courses = []string{"django", "scrapy", "tornado"}
	//fmt.Printf("%T", courses)

	//切片的第二种初始化方法 make
	//切片不是没有长度限制，为什么使用make初始化的需要我们传递一个长度， 那我传递了长度之后是不是意味着就像数组一样长度不能变了呢？
	//courses := make([]string, 5)
	//fmt.Println(len(courses))
	//slice对标的就是python中list

	//第三种方法：通过数组变成一个切片
	var courses = [5]string{"django", "scrapy", "tornado","python", "asyncio"}
	subCourse := courses[1:4] //亲切 //python中的用法叫切片 go语言中切片是一种数据结构 //python中切片的用法非常的多非常的灵活
	//切片
	replace(subCourse)
	fmt.Println(subCourse)
	//subCourse[0] = "bobby"
	//fmt.Println(courses)
	//fmt.Println(subCourse)
	//fmt.Printf("%T", subCourse)

	//第四种方式： new
	//subCourse2 := new([]int)
	//fmt.Println(subCourse2)

	//数组的传递是值传递 切片是引用传递
	//slice很重要，很常用

	//slice是动态数组，所以说我们需要动态添加值
	//fmt.Println(subCourse[1])
	//subCourse[1] = "bobby"
	//fmt.Println(subCourse)
	subCourse2 := subCourse[1:3]
	fmt.Printf("%T, %v\n", subCourse2, subCourse2)

	//append 可以向切片追加元素
	appendedCourse := []string{"imooc", "imooc2", "imooc3"}
	subCourse2 = append(subCourse2, appendedCourse...) //函数的参数传递规则
	fmt.Println(subCourse2)

	subCourse3 := make([]string, len(subCourse))
	fmt.Println(len(subCourse3))
	copy(subCourse3,subCourse2)

	//拷贝的时候 目标对象长度需要设置好
	fmt.Println(subCourse3)

	//append函数追加多个元素， 既然是切片，那为什么这个时候有来提出长度的要求
	//想从切片中删除元素怎么办？

	deleteCourses := [5]string{"django", "scrapy", "tornado","python", "asyncio"}
	courseSlice := deleteCourses[:]
	courseSlice = append(courseSlice[:1], courseSlice[2:]...) //取巧的做法
	fmt.Println(courseSlice)

	//如何判断某个元素是否在切片中

	//python和go语言的slice区别 玩出花样 go的slice更像是python的list， go语言的底层是基于数组实现的 python的list狄岑逛夜市基于数组实现的
	//slice进行的操作都会影响原来的数组， slice更像是一个指针 本身不存值

	//slice的原理，因为很多底层的知识相对来说很多时候并不难而是需要花费比较多的时间去慢慢理解
	//1. 第一个现象
	a := make([]int, 0)
	b := []int{1,2,3}
	fmt.Println(copy(a, b))
	fmt.Println(a)
	//不会去扩展a的空间
	//2. 第二个现象
	c := b[:]
	//c[0] = 8
	fmt.Println(b)
	fmt.Println(c)

	//3. 第三个现象
	c = append(c, 9)
	fmt.Println(b) //append函数没有影响到原来的数组
	fmt.Println(c)
	//这就是因为产生了扩容机制，扩容机制一旦产生 这个时候切片就会指向新的内存地址
	c[0] = 8
	fmt.Println(b)
	fmt.Println(c) //为什么append函数之后再调用c[0]=8不会影响到原来的数组
	//4. 第四个现象
	fmt.Println(len(c))
	fmt.Println(cap(c)) //cap指的是容量 长度和容量这两个概念
	//切片底层是使用数组实现的，既要使用数组 又要满足动态的功能 怎么实现？
	//假设有一个值 实际上申请数组的时候可能是两个，如果后续要增加数据那么就直接添加到数据的结尾，这个时候我不要额外重新申请
	//切片有不同的初始化方式
	//1. 使用make方法初始化 len和cap是多少. 不会有多余的预留空间
	d := make([]int, 0)
	fmt.Printf("len=%d, cap=%d\n", len(d), cap(d))

	//2. 通过数组取切片
	data := [10]int{0,1,2,3,4,5,6,7,8,9}
	slice := data[2:4]
	newSlice := data[3:6]
	for index, value := range slice {
		fmt.Println(index, value)
	}
	fmt.Printf("len=%d, cap=%d\n", len(slice), cap(slice))
	fmt.Printf("len=%d, cap=%d\n", len(newSlice), cap(newSlice))

	//3.
	slice2 := []int{1,2,3}
	fmt.Printf("len=%d, cap=%d\n", len(slice2), cap(slice2))

	//切片扩容问题， 扩容阶段会影响速度， python的list中底层实际上也是数组，也会面临动态扩容的问题，python的list中数据类型可以不一致
	oldSlice := make([]int, 0)
	fmt.Printf("len=%d, cap=%d\n", len(oldSlice), cap(oldSlice))
	oldSlice = append(oldSlice,1)
	fmt.Printf("len=%d, cap=%d\n", len(oldSlice), cap(oldSlice))
	oldSlice = append(oldSlice,2)
	fmt.Printf("len=%d, cap=%d\n", len(oldSlice), cap(oldSlice))
	oldSlice = append(oldSlice,3)
	fmt.Printf("len=%d, cap=%d\n", len(oldSlice), cap(oldSlice))
	oldSlice = append(oldSlice,4)
	oldSlice = append(oldSlice,5)
	oldSlice = append(oldSlice,4)
	oldSlice = append(oldSlice,5)
	oldSlice = append(oldSlice,4)
	oldSlice = append(oldSlice,5)
	fmt.Printf("len=%d, cap=%d\n", len(oldSlice), cap(oldSlice))
	/*
	Go 中切片扩容的策略是这样的：

	首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）
	否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap）
	否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的 1/4，即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
	如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）
	 */
	//如果小于1024 扩容的速度是2倍 如果大于了1024 扩容的速度就是1.25

	//切片来说 1. 底层是数组，如果是基于数组产生的 会有一个问题就是会影响原来的数组。
	//2. 切片的扩容机制
	//3. 切片的传递是引用传递
	oldArr := [3]int{1,2,3}
	newArr := oldArr
	newArr[0] = 5
	fmt.Println(newArr, oldArr)

	oldSlice = []int{1,2,3}
	newSlice = oldSlice
	newSlice[0] = 5
	fmt.Println(oldSlice)
	//go语言中slice的原理讲解很重要，这个有坑（对于初学者），有经验的程序员觉得这个不是坑
	//程序员也是消费者-java c++ go -静态语言是站在对处理器的角度考虑， python - 才会站在使用者的角度上去考虑，对于处理器就不友好
	//当make遇到了append容易出现的坑
	s1 := make([]int, 0)
	s1 = append(s1, 6)
	//对于很多初学者来说我们期望的是只有一个数字就是6
	fmt.Println(s1)
	//很多人对make函数产生了一个假象 ，s1 := make([]int, 5) 好比是python的 s1 = []

}
```

### `map`

```go
package main

import "fmt"

func main()  {
	//go中的map ->python中的dict
	//go语言中的map的key和value类型申明的就要指明
	//1. 字面值
	m1 := map[string]string{
		"m1": "v1",
	}
	fmt.Printf("%v\n", m1)

	//2. make函数 make函数可以创建slice 可以创建map
	m2 := make(map[string]string) //创建时，里面不添加元素
	m2["m2"] = "v2"
	fmt.Printf("%v\n", m2)

	//3. 定义一个空的map
	m3 := map[string]string{}
	fmt.Printf("%v\n", m3)

	//map中的key 不是所有的类型都支持，改类型需要支持 == 或者 != 操作
	//int rune
	//a := []int{1,2,3}
	//b := []int{1,2,3}
	//var m1 map[[]int]string

	//a := [3]int{1,2,3}
	//b := [3]int{1,2,3}
	//if a == b {
	//
	//}
	//map的基本
	m := map[string]string{
		"a": "va",
		"b": "vb",
		"d":"",
	}

	//1. 进行增加，修改
	m["c"] = "vc"
	m["b"] = "vb1"
	fmt.Printf("%v\n", m)

	//查询，你返回空的字符串到底是没有获取到还是值本身就是这样空字符串呢
	v, ok := m["d"]
	if ok {
		fmt.Println("找到了", v)
	}else{
		fmt.Println("没找到")
	}
	fmt.Println(v, ok)

	//删除
	//delete(m, "a")
	//delete(m, "e")
	//delete(m, "a")
	//fmt.Printf("%v", m)

	//遍历
	for k, v := range m {
		fmt.Println(k, v)
	}

	//go语言中也有一个list 就是数据结构中提到的链表
	//指针 //为什么指针在java python等很多语言中不存在
}
```

## 指针

```go
package main

import "fmt"

func swap(a *int, b *int){
	//用于交换a和b
	c := *a
	*a = *b
	*b = c
}
func main() {
	//什么指针，我们提一个问题
	a := 10
	b := 20
	//swap(a, b)
	fmt.Println(a, b)
	//为什么交换不成功， 这个函数运行完成以后 我想要把a和b的值变掉
	//指针 - 对于内存来说，每一个字节其实都是有地址-通过16进制打印出来
	fmt.Printf("%p\n", &a) //变量有地址
	//现在有一种特殊的变量类型，这个变量只能保存地址值
	var ip *int //这个变量里面就只能保存地址类型这种值
	ip = &a

	//如果要修改指针指向的变量的值，用法也比较特殊
	*ip = 30
	fmt.Println(a)
	//如何定义指针变量 如果修改指针变量指向的内存中的值。 通过指针去取值的时候不知道应该取多大的连续内存空间
	fmt.Printf("ip所指向的内存空间地址是：%p, 内存中的值是: %d\n",ip, *ip)
	swap(&a, &b)
	fmt.Println(a, b)
	//还不足以说服大家
	//但是go中数组是值传递 数组中有100万个值， 对于这种一般我们都采用切片来传递
	//在python中list和dict这种传递都是引用传递

	//指针还可以指向数组 指向数组的指针 数组是值类型
	arr := [3]int{1,2,3}
	var ip *[3]int = &arr
	//指针数组
	var ptrs [3]*int //创建能够存放三个指针变量的数组
	//很多时候都是函数参数的时候指明的类型
	//指针的默认值是nil
	if ip != nil {

	}
	//像python和java这种语言都是极力的屏蔽指针， c/c++ 都提供了指针 指针本身是很强大
	//c和c++中指针的功能很强大 指针的转换 指针的偏移 指针的运算
	//go语言没有屏蔽指针，但是go语言在指针上做了大量的限制，安全性高很多，相比较 c和c++灵活性就降低了
	//指针变量中涉及到两个符号 & 和 *

	//make， new， nil
}
```

### `make`和`new`

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	//make和new函数
	//new函数用法
	//var p *int //申明了一个变量p 但是变量没有初始值 没有内存
	//*p = 10
	//默认值 int byte rune float bool string 这些类型都有默认值
	//指针， 切片 map， 接口这些默认值是nil 理解为none
	var a int
	a = 10
	fmt.Println(a)


	//对于指针来说或者说其他的默认值是0的情况来说 如何一开始申明的时候就分配内存
	var p *int = new(int) //go的编译器就知道先申请一个内存空间，这里的内存中的值全部设置为0
	*p = 10

	//int使用make就不行
	//除了new函数可以申请内存以外 还有一个函数就是make,更加常用的是make函数， make函数一般用于切片 map
	var info map[string]string = make(map[string]string)
	info["c"] = "bobby"

	//new函数返回的是这个值的地址 指针 make函数返回的是指定类型的实例
	//nil的一些细节
	var info2 map[string]string
	if info2 == nil {
		fmt.Println("map的默认值是 nil")
	}

	var slice []string
	if slice == nil {
		fmt.Println("slice的默认值是 nil")
	}

	var err error
	if err == nil {
		fmt.Println("error的默认值是 nil")
	}

	//python中的None和go语言中的nil类型不一样，None是全局唯一的
	//go语言中 nil是唯一可以用来表示部分类型的零值的标识符， 它可以代表许多不同内存布局的值
	fmt.Println(unsafe.Sizeof(slice), unsafe.Sizeof(info2))
}
```

## 函数

### 函数的定义

```go
package main

import (
	"errors"
	"fmt"
)

//相比较其他静态语言，go语言的函数有很多两点
//函数的几个要素： 1. 函数名 2. 参数 3. 返回值
//函数第一种定义方法
func add(a, b int) int {
	var sum int
	sum = a + b
	return sum
}

//函数的第二种定义方法
func add2(a, b int) (sum int) {
	sum = a + b
	return sum
}

//函数的第三种定义方法
func add3(a, b int) (sum int) {
	sum = a + b
	return
}

//函数的第4种定义方法
// 被除数等于0 ，要返回多个值 -一个非常有用的特性
//go语言的返回设置花样很多
func div(a, b int) (int, error) {
	var result int
	var err error
	if b == 0 {
		err = errors.New("被除数不能为0")
	} else {
		result = a / b
	}

	return result, err
}

func div2(a, b int) (result int, err error) {
	if b == 0 {
		err = errors.New("被除数不能为0")
	} else {
		result = a / b
	}

	return
}

func main() {
	result, err := div2(12, 3)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(result)
	}
}
```

### 不定长参数

```go
package main

import "fmt"

//省略号
func add(params ...int) (sum int){
	//不能解决一个问题，我可能有不定个int值传递进来
	for _, v := range params {
		sum += v
	}
	params[0] = 9
	return
}

type sub func (a, b int) int //sub就等同于int map
func subImpl(a, b int) int {
	return a - b
}

func filter(score []int, f func(int) bool) []int{
	reSlice := make([]int, 0)
	for _, v := range score {
		if f(v) {
			reSlice = append(reSlice, v)
		}
	}
	return reSlice
}

func main() {
	//通过省略号去动态设置多个参数值
	slice := []int{1,2,3,4,5}
	fmt.Println(add(slice...)) //将slice打散
	fmt.Println(slice)
	//这种效果slice
	//区别，slice是一种类型， 还是引用传递， 我们要慎重

	//省略号的用途 1. 函数参数不定长 2. 将slice打散 3.
	arr := [...]int{1,2,3}
	fmt.Printf("%T\n", arr)
	//匿名函数
	fmt.Println(func (a, b int) int{
		return a+b
	}(1,2))
	//fmt.Println(result)
	//fmt.Printf("%T", myFunc)

	//go语言中非常重要的特性 函数 一些trick 一些语法糖 函数的一等公民特性 - 可以作为参数 返回值 复制给另一个变量
	//函数也可以当做参数传递给一个函数

	var mySub sub = func (a, b int) int {
		return a - b
	}
	fmt.Println(mySub(1,2))

	//将函数作为另一个函数的参数
	//写一个函数用于过滤一部分数据
	score := []int{10, 50, 80, 90, 85}
	//写一个函数过滤掉不合格的成绩
	fmt.Println(filter(score, func (a int) bool{
		if a>=90{
			return true
		}else {
			return false
		}
	}))
	//gin, python的装饰器

	//go语言并没有提供try except finally
	//1. 大量的嵌套try finally 2. 打开和关系逻辑离的太远
}
```

### `defer`

```go
package main

import "fmt"

//func main() {
//	fmt.Println("test1")
//	//defer之后只能是函数调用 不能是表达式 比如 a++
//	defer fmt.Println("defer test1")
//	defer fmt.Println("defer test2")
//	defer fmt.Println("defer test3")
//	/*
//	defer 语句是go体用的一种用于注册延迟调用的机制， 它可以让当前函数执行完毕之后执行
//	对于python的with语句来说，
//	 */
//	//此处有大量的逻辑需要读取文件
//	fmt.Println("test2")
//	//1. 如果有多个defer会出现什么情况 多个defer是按照先入后出的顺序执行
//}

//func main()  {
//	//defer语句执行时的拷贝机制
//	test := func () {
//		fmt.Println("test1")
//	}
//	defer test()
//	test = func () {
//		fmt.Println("test2")
//	}
//	fmt.Println("test3")
//}

//func main()  {
//	//defer语句执行时的拷贝机制
//	x := 10
//	defer func (a *int) {
//		fmt.Println(*a)
//	}(&x)
//	x++
//}

//func main()  {
//	//defer语句执行时的拷贝机制
//	x := 10
//	//此处的defer函数并没有参数，函数内部使用的值是全局的值
//	defer func (a int) {
//		fmt.Println(x)
//	}(x)
//	x++
//}

func f1() int {
	x := 10
	defer func () {
		x++
	}()
	tmp := x //x是int类型 值传递
	return tmp
}

func f2() *int {
	a := 10
	b := &a
	defer func () {
		*b++
	}()
	temp_data := b
	return temp_data
}
func main() {
	fmt.Println(f1()) //是不是就意味着 defer中影响不到外部的值呢
	fmt.Println(*f2())
	//defer本质上是注册了一个延迟函数，defer函数的执行顺序已经确定
	//defer 没有嵌套 defer的机制是要取代try except finally
	//https://www.cnblogs.com/zhangboyu/p/7911190.html
	//https://studygolang.com/articles/24044?fr=sidebar
}
```

### `panic`

```go
package main

import (
	"fmt"
)

func div(a, b int) (int, error)  {
	if b == 0 {
		panic("被除数不能为0")
	}
	return a/b, nil
}

func main()  {
	//错误就是能遇到可能出现的情况，这些情况会导致你的代码出问题， 参数检查， 数据库访问不了
	a := 12
	b := 0
	defer func() {
		err := recover()
		if err != nil {
			fmt.Println("异常被捕获到")
		}
		fmt.Println("bobby")
	}()

	fmt.Println(div(a, b))

	//panic的坑

	//strconv.Itoa(data) //go 认为这个Itoa的函数不可能出错， 没有必要返回error 内部代码出错这个时候应该抛出异常 panic
	//_, err := strconv.Atoi("abcd") //Atoi这个函数认为我的函数内部会出现一些预知错误情况
	//if err != nil {
	//	//错误
	//}

	//异常 go语言中如何抛出异常和如何捕捉异常
	//go语言认为错误就要自己处理， 就个人而言，go语言的这种想法是正确的。但是实际的使用中确实人有点烦人
}
```

## 结构体

### `type`

```go
package main

import "fmt"

type Course struct {
	name string
	price int
}

type Callable interface {

}

type handle func(str string)

func main()  {
	//go语言中的关键词 type
	//1. 给一个类型定义别名, 实际上为什么会有byte， 就是我为了强调我们现在处理的对象是字节类型 这种别名实际上还是为了代码的可读性， 这个实际上本质上仍然是uint8 无非就是在代码编码阶段可读性强而已
	type myByte = byte
	var b uint8
	fmt.Printf("%T\n", b)

	//2. 第二种 就是基于一个已有的类型定义一个新的类型
	type myInt int
	var i myInt
	fmt.Printf("%T\n", i)

	//3. 定义结构体
	//4. 定义接口
	//5. 定义函数别名
}
```

### `struct`

```go
package main

import (
	"fmt"
	"unsafe"
)

type Course struct {
	Name string
	Price int
	Url string
}

//函数的接收者
func (c Course) printCourseInfo()  {
	fmt.Printf("课程名:%s, 课程价格: %d, 课程的地址:%s", c.Name, c.Price, c.Url)
}
func (c *Course) setPrice(price int){
	c.Price = price
}

//1. 结构体的方法只能和结构体在同一个包中
//2. 内置的int类型不能加方法

func main()  {
	//go语言不支持面向对象
	//面向对象的三个基本特征： 1. 封装 2. 继承 3. 多态 4. 方法重载 4. 抽象基类
	//定义struct go语言没有class这个概念 所以说对于很多人来说会少理解很多面向对象抽象的概念

	//1. 实例化- kv形式
	var c Course = Course{
		Name: "django",
		Price: 100,
		Url: "https://www.imooc.com",
	}

	//访问
	fmt.Println(c.Name, c.Price, c.Url)

	//大小写在go语言中的重要性 可见性
	//一个包中的变量或者结构体如果首字母是小写 那么对于另一个包不可见
	//机构体定义的 名称 以及属性首字母大写很重要

	//2. 第二种实例化方式 - 顺序形式
	c2 := Course{"scrapy", 110, "https://www.imooc.com"}
	fmt.Println(c2.Name, c2.Price, c2.Url)

	//3. 如果一个指向结构体的指针, 通过结构体指针获取对象的值， 让很多人莫名其妙
	c3 := &Course{"tornado", 100, "https://www.imooc.com"}
	//fmt.Printf("%T", c3)
	//应该能看出来 go语言实际上在借鉴动态语言的特性 - 很多地方不管如何写都是正确的
	//另一个根本的原因 - go语言的指针是受限的
	fmt.Println(c3.Name, c3.Price, c3.Url) //这里其实是go语言的一个语法糖 go语言内部会将c3.Name转换成 (*c3).Name

	//4. 零值 如果不给结构体赋值， go语言会默认给每个字段采用默认值
	c4 := Course{}
	fmt.Println(c4.Price,)

	//5. 多种方式零值初始结构体
	var c5 Course = Course{}
	var c6 Course
	var c7 *Course = &Course{}
	//为什么c6和c8表现出来的结果不一样 指针如果只申明不赋值 默认值是nil c6不是指针 是结构体的类型
	//slice map

	fmt.Println("零值初始化")
	fmt.Println(c5.Price)
	fmt.Println(c6.Price)
	fmt.Println(c7.Price)

	//6. 结构体是值类型
	c8 := Course{"scrapy", 110, "https://www.imooc.com"}
	c9 := c8
	fmt.Println(c8)
	fmt.Println(c9)
	c8.Price = 200
	fmt.Println(c8)
	fmt.Println(c9)

	//go语言中struct无处不在
	//7. 结构体的大小 占用内存的大小 可以使用sizeof来查看对象占用的类型
	fmt.Println(unsafe.Sizeof(1))
	//go语言string的本质 其实string是一个结构体
	fmt.Println(unsafe.Sizeof(""))
	fmt.Println(unsafe.Sizeof(c8))

	//8. slice的大小
	type slice struct {
		array unsafe.Pointer  // 底层数组的地址
		len int // 长度
		cap int // 容量
	}

	s1 := []string{"django", "tornado", "scrapy", "celery", "snaic", "flask"}
	fmt.Println("切片占用的内存:", unsafe.Sizeof(s1))

	m1 := map[string]string {
		"bobby1": "django",
		"bobby2": "tornado",
		"bobby3": "scrapy",
		"bobby4": "celery",
	}
	fmt.Println(unsafe.Sizeof(m1))

	//结构体方法， 达到了封装数据和封装方法的效果
	c10 := Course{"scrapy", 110, "https://www.imooc.com"}
	//Course.setPrice(c10, 200)
	(&c10).setPrice(200) //修改c10的price? 为什么呢？ 语法糖 函数参数的传递是怎么传递的？ 结构体是值传递
	fmt.Println(c10.Price)
	//c10.printCourseInfo()

	//结构体的接收者有两种形式 1. 值传递 2. 指针传递 如果你想改结构体的值 如果结构体的数据很大
	//go语言不支持继承 但是有办法能达到同样的效果 组合

}
```

### 模拟构造函数

```go
package main

import "fmt"

type Student struct { // 声明结构体 值类型
	name   string
	age    int
	gender byte
	score  [3]float64
}

func NewStudent(name string, age int, gender byte, score [3]float64) *Student {
	return &Student{
		name:   name,
		age:    age,
		gender: gender,
		score:  score,
	}

}
func main() {
	var s1 = NewStudent("lzj", 18, 1, [3]float64{11.1, 20.5, 33.5})
	fmt.Println(s1)
}
```

### 方法接收器

```go
package main

import "fmt"

type Player struct {
	name        string
	healthPoint int
	magic       int
}

func (p *Player) attack() {
	fmt.Printf("%s 发起了攻击\n", p.name)
}
func (p *Player) attacked() {
	fmt.Printf("%s 被攻击\n", p.name)
	p.healthPoint -= 10
}
func main() {
	s1 := Player{
		name:        "b1",
		healthPoint: 100,
		magic:       10,
	}
	s2 := Player{
		name:        "b2",
		healthPoint: 50,
		magic:       20,
	}
	fmt.Println(s1.name)
	fmt.Println(s2.name)
	s1.attack()
	s2.attacked()
	fmt.Println(s2.healthPoint)
	s2.attack()
}
```

### 匿名成员变量

```go
package main

import "fmt"

// 结构体的嵌套

type Addr struct {
	country  string
	province string
	city     string
}

type Student struct {
	name string
	// int  // 默认会以类型为名称
	//address Addr
	Addr
}

func main() {
	s := Student{
		name: "a",
	}
	s.Addr = Addr{
		country:  "china",
		province: "bj",
		city:     "cp",
	}

	fmt.Println(s.Addr.province)
	fmt.Println(s.province)
}
```

### `struct tag`

```go
package main

import (
	"encoding/json"
	"fmt"
	"reflect"
)

//结构体能基本上达到类的一个效果 多态
type Info struct { //能表述的信息是有限的
	Name   string `json:"name"` //name是映射成mysql中char类型还是varchar类型还是text类型， 即使能够说明 但是额外的信息 max_length
	Age    int    `json:"age,omitempty"`
	Gender string `json:"-"`
}

//type Info struct { //能表述的信息是有限的
//	Name string `orm:"name, max_length=17, min_length=5"`
//	Age int `orm:"age, min=18, max=70"`
//	Gender string `orm:"gender, required"`
//}

//反射包

func main() {
	//结构体标签
	/*
		结构体的字段除了名字和类型外，还可以有一个可选的标签（tag）：
		它是一个附属于字段的字符串，可以是文档或其他的重要标记。
		比如在我们解析json或生成json文件时，常用到encoding/json包，
		它提供一些默认标签，例如：omitempty标签可以在序列化的时候忽略0值或者空值。
		而-标签的作用是不进行序列化，其效果和和直接将结构体中的字段写成小写的效果一样。
	*/
	info := Info{
		Name:   "bobby",
		Gender: "男",
	}
	re, _ := json.Marshal(info)
	fmt.Println(string(re))

	//通过反射包去识别这些tag 简单的体验了一下反射的威力 spring 底层都是反射
	t := reflect.TypeOf(info)
	fmt.Println("Type:", t.Name())
	fmt.Println("Kind:", t.Kind())

	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i) //获取结构体的每一个字段
		tag := field.Tag.Get("orm")
		fmt.Printf("%d. %v (%v), tag: '%v'\n", i+1, field.Name, field.Type.Name(), tag)
	}
	//具体的应用绝大部分情况之下我们是不需要使用到反射的 实际开发的项目中会用到的
	//接口 java 实际上在go语言中接口这个概念的地位和java中接口的地位是不一样， go语言的接口实际上就是python中的协议 - 鸭子类型
}
```

### 继承

```go
package main

import "fmt"

type Teacher struct {
	Name  string
	Age   int
	Title string
}

func (t Teacher) teacherInfo() {
	fmt.Printf("姓名:%s, 年龄:%d, 职称:%s", t.Name, t.Age, t.Title)
}

type Course struct {
	Teacher //如果讲师的信息比较多怎么办 将另一个结构体的变量放进来
	Name    string
	Price   int
	Url     string
}

//匿名嵌套, 这种做法其实就是 语法糖 还不算是继承 有了前面的方法和这里的内嵌结构体 实际上 继承 封装
func (c Course) courseInfo() {
	fmt.Printf("课程名:%s, 价格:%d, 讲师信息：%s %d %s", c.Name, c.Price, c.Teacher.Name, c.Age, c.Title)
}
//这种继承的效果说实话 有点说服不了人
func main() {
	//go语言的继承 组合
	t := Teacher{
		Name:"bobby",
		Age:18,
		Title:"程序员",
	}
	c := Course{
		Teacher:t,
		Price:100,
		Url:"",
		Name:"django",
	}
	c.courseInfo()

}
```

## 接口

### 接口的定义和使用

```go
package main

import (
	"fmt"
)

//接口是一个协议- 程序员 - 只要你能够 1. 写代码 2. 解决bug 其实就是一组方法的集合
type Programmer interface {
	Coding() string //方法只是申明
	Debug() string
}
type Designer interface {
	Design() string
}
type Manger interface {
	Programmer
	Designer
	Manage() string
}

//java的话 java里面一种类型只要继承一个接口 才行 如果你继承了这个接口的话 那么这个接口里面的所有方法你必须要全部实现
type UIDesigner struct {
}

func (d UIDesigner) Design() string{
	fmt.Println("我会ui设计")
	return "我会ui设计"
}

type Pythoner struct {
	UIDesigner
	lib []string
	kj []string
	years int
}

type G struct {

}

func (p G) Coding() string{
	fmt.Println("go开发者")
	return "go开发者"
}

func (p G) Debug() string{
	fmt.Println("我会go的debug")
	return "我会go的debug"
}

func (p Pythoner) Coding() string{
	fmt.Println("python开发者")
	return "python开发者"
}

func (p Pythoner) Debug() string{
	fmt.Println("我会python的debug")
	return "我会python的debug"
}

func (p Pythoner) Manage() string{
	fmt.Println("不好意思，管理我也懂")
	return "不好意思，管理我也懂"
}
//
//func (p Pythoner) Design() string{
//	fmt.Println("我是一个python开发者，但是我会ui设计")
//	return "我是一个python开发者，但是我会ui设计"
//}
//对于Pythoner这个结构体来说 你实现任何方法都可以 ，但是你只要不全部实现Coding Debug的话 那你Pythoner就不是一个Programmer类型
//1. Pythoner本身自己就是一个类型 那我何必在意我是不是Programmer
//2. 1. 封装 继承 多态 -多态的概念对于很多pythoner来说会有点陌生
//3. 在讲解多态之前 我们来对interface做一个说明：在go语言中接口是一种类型 int ， 是一种抽象类型

//开发中经常会遇到的问题
//开发一个电商网站， 支付环节 使用 微信、支付宝、银行卡 你的系统支持各种类型的支付 每一种支付类型都有统一的接口
// 定一个协议 1. 创建订单 2. 支付 3. 查询支付状态 4. 退款
//支付发起了
//type AliPay struct {
//
//}
//type WeChat struct {
//
//}
//
//type Bank struct {
//
//}
//
//var b Bank
//var a AliPay
//var w WeChat

//多态 什么类型的时候你申明的类型是一种兼容类型， 但是实际赋值的时候是另一种类型
//接口的强制性
//你现在有一个缓存 - 这个地方你一开始使用的缓存是redis 但是后期你考虑到可能使用其他的缓存技术 - 本地 memcache

//这种多态特性 其实在python中不需要多态 python是动态语言

//go语言中并不支持继承

//如果后期接入一种新的支付 或者取消已有的支付

func HandlePy(p Programmer){

}

type MyError struct {

}

func (m MyError) Error() string {
	return "错误"
}

func main()  {
	//新的语言出来了, 接口帮我们完成了go语言的多态
	//var pro Programmer = Pythoner{}

	var pros []Programmer
	pros = append(pros, Pythoner{})
	pros = append(pros, G{})

	//接口虽然是一种类型 但是和其他类型不太一样 接口是一种抽象类型 struct是具象
	p := Pythoner{}
	fmt.Printf("%T\n", p)
	var pro Programmer = Pythoner{}
	fmt.Printf("%T\n", pro)
	var pro2 Programmer = G{}
	fmt.Printf("%T", pro2)
	//如果大家对象面向对象理解的话 java 里面的抽象类型
	//1. go struct组合 组合一起实现了所有的接口的方法也是可以的
	//2. 接口本身也支持组合

	var m Manger = Pythoner{}
	m.Design()
	//python语言本身设计上是采用了完全的基于鸭子类型 - 协议 影响了python语法的 for len()
	//struct组合完成了接口 1. 接口支持组合 - 继承 2. 结构体组合实现了所有的接口方法也没有问题
	//go语言本身也推荐鸭子类型  error
	//var err error = errors.New(fmt.Sprintf(""))
	s := "文件不存在"
	var err error = fmt.Errorf("错误:%s", s)
	fmt.Println(err)
}
```

### 空接口

```go
package main

import "fmt"

type Course struct {
	name string
	price int
	url string
}
type Printer interface {
	printInfo() string
}

func (c Course) printInfo() string{
	return "课程信息"
}

//func print(x interface{}){
//	if v, ok := x.(int); ok{
//		fmt.Printf("%d(整数)\n", v)
//	}
//	if s, ok := x.(string); ok {
//		fmt.Printf("%s(字符串)\n", s)
//	}
//	//牵扯到go的另一个默认的问题
//	//fmt.Printf("%v\n", i)
//}

type AliOss struct {

}

type LocalFile struct {

}

func store(x interface{}){
	switch v := x.(type) {
	case AliOss:
		//此处要做一些特殊的处理，我设置阿里云的权限问题
		fmt.Println(v)
	case LocalFile:
		//检查路径的权限
		fmt.Println(v)
	}
}

func print(x interface{}){
	switch v := x.(type) {
	case string:
		fmt.Printf("%s(字符串)\n", v)
	case int:
		fmt.Printf("%d(整数)\n", v)
	}
}


func main()  {
	//空接口
	var i interface {} //空接口
	//空接口可以类似于我们java和python中的object
	//i = Course{}
	//fmt.Println(i)
	i = 10
	print(i)
	i = "bobby"
	print(i)
	i = []string{"django", "scrapy"}
	print(i)
	//空接口的第一个用途 可以把任何类型都赋值给空接口变量
	//2. 参数传递
	//3. 空接口可以作为map的值
	var teacherInfo = make(map[string]interface{})
	teacherInfo["name"] = "bobby"
	teacherInfo["age"] = 18
	teacherInfo["weight"] = 75.2
	teacherInfo["courses"] = []string{"django", "scrapy", "sanic"}
	fmt.Printf("%v", teacherInfo)
	//类型断言
	//接口的一个坑, 接口引入了
	// 接口有一个默认的规范  接口的名称一般以 er结尾
	//c := &Course{}
	//var c Printer = Course{}
	//c.printInfo()

}
```

### 接口实现`sort`

```go
package main

import (
	"fmt"
	"sort"
)

type Course struct {
	Name string
	Price int
	Url string
}

type Courses []Course

func (c Courses) Len() int{
	return len(c)
}

func (c Courses) Less(i, j int) bool{
	return c[i].Price < c[j].Price
}

func (c Courses) Swap(i, j int) {
	c[i], c[j] = c[j], c[i]
}

func main()  {
	//通过sort来排序
	//让你写一个排序算法， 冒泡 插入 快速 归并 桶排序 算法本质是一样 比较 计数排序
	//你的排序算法是否能应付各种类型的排序
	courses := Courses{
		Course{"django", 300, ""},
		Course{"scrapy", 100, ""},
		Course{"go", 400, ""},
		Course{"torando", 200, ""},
	}
	sort.Sort(courses) //协议 你的目的不是要告诉别人具体的类型，重要的是你的类型必要提供具体的方法
	for _, v := range courses{
		fmt.Println(v)
	}
	//grpc
	//go语言的包以及编码规范 包管理 1.12之前 go的没有包管理 python java maven go modules
}
```

## 值类型和引用类型

### 值类型

```go
值类型：基本数据类型(int,float,bool,string)以及数组和struct都属于值类型

特点：变量直接存储值，内存通常在栈中分配，栈在函数调用完会被释放，值类型变量声明后，不管是否以及赋值，编译器为其分配内存，此时该值存于栈上

var a int     // int类型默认值为0
var b string  // string类型默认值为nil空
var c bool    // bool类型默认值为false
var d [2]int  // 该数组默认值为[0 0] 根据类型而定

当使用 = 将一个变量的值赋值给另一个变量时，如 j = i，实际上是在内存中将 i 的值进行拷贝，可以通过 &i 获取 i 的内存地址，此时如果修改另一个变量的值不会影响到另一个
```

### 引用类型

```go
引用类型：指针 slice map chan interface等都是引用类型
特点：变量存储的是一个地址，这个地址存储最终的值，内存通常在堆上分配，通过GC回收

变量直接存放的就是一个内存地址，这个地址值指向的空间存的才是值，所以修改其中一个，另一个也会修改(同一个内存地址)，
引用类型必须申请内存才可以使用，make()是给引用类型申请内存空间
```

## 文件操作

### 读文件

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"io/ioutil"
	"os"
)

func read01(f *os.File) {
	// 读字节
	var b = make([]byte, 3)
	n, err := f.Read(b)
	if err != nil {
		panic(err)
	}
	fmt.Println("读取的字节个数: ", n)
	fmt.Println("读取的字节内容: ", b)
	fmt.Println("读取的字节的字符内容: ", string(b))
}

func read02(f *os.File) {
	// 按行读
	reader := bufio.NewReader(f)

	//for true {
	//	strs, err := reader.ReadString('\n') // 遇到换行符停止
	//	if err == io.EOF {                   // 已经读到末尾了
	//		//fmt.Println("err:", err)
	//		fmt.Println(err)
	//		break
	//	}
	//	fmt.Print(strs)
	//}

	for true {
		bytes, err := reader.ReadBytes('\n')
		//fmt.Print(bytes)
		fmt.Print(string(bytes))
		if err == io.EOF {
			break
		}
	}
	fmt.Println()
}

func read03() {
	bytes, err := ioutil.ReadFile("./a.txt")
	if err != nil {
		fmt.Println(err)
	}
	//fmt.Println(bytes)
	fmt.Println(string(bytes))
}
func main() {
	// 打开文件 获取文件描述符
	file, err := os.Open("./a.txt")
	if err != nil {
		fmt.Println("err:", err)
	}
	fmt.Println("file: ", file)
	//关闭文件句柄
	defer file.Close()

	// 读文件
	// 方式1 读字节和字符
	fmt.Println("===========read01===============")
	read01(file)

	// 方式2 按行读
	fmt.Println("===========read02===============")
	read02(file)

	// 方式3 读文件 使用于小文件
	fmt.Println("==========read03================")
	read03()
}
```

### 写文件

```go
package main

import (
	"bufio"
	"fmt"
	"io/ioutil"
	"os"
)

func write01(f *os.File) {
	str := "满江红\n"
	// 写入字节切片数据 把字符强转为 byte
	f.Write([]byte(str))
	// 直接写入字符串
	f.WriteString("怒发冲冠，凭栏处、萧雨歇歇。\n")
}

func write02(f *os.File) {
	writer := bufio.NewWriter(f)
	// 将数据先写入缓存，并不会到文件中
	writer.WriteString("抬望眼，仰天长啸，壮怀激烈。\n")
	// 必须flush将缓存中的内容写入文件
	writer.Flush()
}

func write03() {
	str := "三十功名尘与土，八千里路云和月。\n"
	err := ioutil.WriteFile("./a.txt", []byte(string(str)), 0666)
	if err != nil {
		fmt.Println("write file err:", err)
		return
	}
}
func main() {
	/*
		os.O_RDONLY 只读的方式打开
		os.O_WRONLY 只写的方式打开
		os.O_RDWR   读写的方式打开
		os.O_APPEND 追加的方式打开
		os.O_CREAT  创建并打开一个新文件
		os.O_TRUNC  打开文件并截断它的长度为0 必须有写权限  光标从零开始
		os.O_EXCL   如果指定的文件存在 返回错误
	*/
	file, err := os.OpenFile("./a.txt", os.O_CREATE|os.O_WRONLY|os.O_TRUNC, 0666)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// 方式1
	write01(file)

	// 方式2
	write02(file)

	// 方式3  创建文件 直接写入数据
	write03()
}
```

## 序列化

```go

```

## 包管理和编码规范

## 并发编程
