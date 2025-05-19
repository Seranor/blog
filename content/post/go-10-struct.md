---
title: golang基础-结构体
lastmod: 2025-05-15T16:43:23+08:00
date: 2025-05-015T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-10.html
toc: true
---

## type关键字

### 自定义类型

在Go语言中有一些基本的数据类型，还可以使用 `type` 关键字来定义自定义类型

自定义类型是定义了一个全新的类型。我们可以基于内置的基本类型定义，也可以通过struct定义。例如：

```go
//将MyInt定义为int类型
type MyInt int
```

### 类型别名

类型别名规定：TypeAlias只是Type的别名，本质上TypeAlias与Type是同一个类型。

```go
type TypeAlias = Type
```

我们之前见过的`rune`和`byte`就是类型别名，他们的定义如下：

```go
type byte = uint8
type rune = int32
```

区别：

```go
package main

import "fmt"

// 定义类型
type MyInt int

// 别名
type NewInt = int

func main() {
	var i MyInt
	fmt.Printf("type:%T value:%v\n", i, i) // type:main.MyInt value:0

	var x NewInt
	fmt.Printf("type:%T value:%v\n", x, x) // type:int value:0
}
```

结果显示 i 的类型是`main.NewInt`，表示main包下定义的`NewInt`类型。x 的类型是`int`。`MyInt`类型只会在代码中存在，编译完成时并不会有`MyInt`类型。

## 结构体

Go语言中没有“类”的概念，也不支持“类”的继承等面向对象的概念。Go语言中通过结构体的内嵌再配合接口比面向对象具有更高的扩展性和灵活性。

Go语言中的基础数据类型可以表示一些事物的基本属性，但是当我们想表达一个事物的全部或部分属性时，这时候再用单一的基本数据类型明显就无法满足需求了，Go语言提供了一种自定义数据类型，可以封装多个基本数据类型，这种数据类型叫结构体，英文名称`struct`。 也就是我们可以通过`struct`来定义自己的类型了。

Go语言中通过`struct`来实现面向对象。

### 定义

使用关键字 **type** 可以将各种基本类型定义为自定义类型，基本类型包括整型、字符串、布尔等。结构体是一种复合的基本类型，通过 type 定义为自定义类型后，使结构体更便于使用。

```go
type 类型名 struct {
    字段名 字段类型
    字段名 字段类型
    ...
}
```

- 类型名：标识自定义结构体的名称，在同一个包内不能重复。
- struct{}：表示结构体类型，`type 类型名 struct{}`可以理解为将 struct{} 结构体定义为类型名的类型。
- 字段1、字段2……：表示结构体字段名，结构体中的字段名必须唯一。
- 字段1类型、字段2类型……：表示结构体各个字段的类型。

```go
type User struct {
	// 成员变量
	Id            int
	Score         float64
	Name, Address string
}
```

### 例化

只有当结构体实例化时，才会真正地分配内存。也就是必须实例化后才能使用结构体的字段。

结构体本身也是一种类型，我们可以像声明内置类型一样使用`var`关键字声明结构体类型。

```go
var 结构体实例 结构体类型
```

#### 基本实例化

```go
package main

import "fmt"

// 声明结构体： 类: 值类型

type Student struct {
	name   string
	age    int
	gender byte
	score  [3]float32
}

func main() {
	// 声明一个结构体对象
	var stu01 Student
	fmt.Println(stu01.name)   // 默认零值
	fmt.Println(stu01.age)    // 默认零值
	fmt.Println(stu01.gender) // 默认零值
	fmt.Println(stu01.score)  // 默认零值

	// 初始化
	stu01.name = "xm"
	stu01.age = 22
	stu01.gender = 1
	stu01.score[0] = 84
	stu01.score[1] = 79
	stu01.score[2] = 88

	fmt.Println(stu01.name)
	fmt.Println(stu01.age)
	fmt.Println(stu01.gender)
	fmt.Println(stu01.score)
}
```

我们通过`.`来访问结构体的字段（成员变量）

#### 匿名结构体

在定义一些临时数据结构等场景下还可以使用匿名结构体。

```go
package main
     
import (
    "fmt"
)
     
func main() {
	// 匿名结构体
	var student struct {
		Name string
		Age  int
	}
	student.Name = "zs"
	student.Age = 18
    fmt.Printf("%#v\n", student)
}
```

#### 指针类型结构体

通过 `new` 关键字对结构体进行实例化，得到的时候结构体地址

```go
package main

import "fmt"

// 声明结构体： 类: 值类型

type Student struct {
	name   string
	age    int
	gender byte
	score  [3]float32
}

func main() {
	var stu2 = new(Student)
	fmt.Printf("%T\n", stu2)
	fmt.Printf("%#v\n", stu2)
}
```

也支持对结构体指针直接使用`.`来访问结构体的成员。

#### 取结构体的地址实例化

使用`&`对结构体进行取地址操作相当于对该结构体类型进行了一次`new`实例化操作。

```go
package main

import (
	"fmt"
)

type Student struct {
	name   string
	age    int
	gender byte
	score  [3]float32
}

func main() {
	var stu3 = &Student{}
	fmt.Printf("%T\n", stu3)
	fmt.Printf("%#v\n", stu3)

	stu3.name = "zhansan"
	stu3.age = 16
	stu3.gender = 1
	stu3.score = [3]float32{88, 77, 56}
	fmt.Println(stu3)
}
```

`stu3.name = "zhansan"`其实在底层是`(*stu3).name = "zhansan"`，这是Go语言帮我们实现的语法糖。

### 结构体初始化

没有初始化的结构体，其成员变量都是对应其类型的零值。

#### 键值对初始化

```go
package main

import "fmt"

type Student struct {
	name   string
	age    int
	gender byte
	score  [3]float32
}

func main() {
	stu4 := Student{
		name:   "lisi",
		age:    18,
		gender: 1,
		score:  [3]float32{88, 89, 79},
	}
	fmt.Printf("%#v\n", stu4) // main.Student{name:"lisi", age:18, gender:0x1, score:[3]float32{88, 89, 79}}
}
```

也可以对结构体指针进行键值对初始化

```go
	stu4 := &Student{
		name:   "lisi",
		age:    18,
		gender: 1,
		score:  [3]float32{88, 89, 79},
	}
```

当某些字段没有初始值的时候，该字段可以不写。此时，没有指定初始值的字段的值就是该字段类型的零值。

#### 使用值的列表初始化

初始化结构体的时候可以简写，也就是初始化的时候不写键，直接写值：

```go
package main

import "fmt"

type Student struct {
	name   string
	age    int
	gender byte
	score  [3]float32
}

func main() {
	stu4 := Student{
		"lisi",
		18,
		1,
		[3]float32{88, 89, 79},
	}
	fmt.Printf("%#v\n", stu4)
}
```

使用这种格式初始化时，需要注意：

1. 必须初始化结构体的所有字段。
2. 初始值的填充顺序必须与字段在结构体中的声明顺序一致。
3. 该方式不能和键值初始化方式混用。

### 结构体内存布局

结构体占用一块连续的内存。

```go
type test struct {
	a int8
	b int8
	c int8
	d int8
}
n := test{
	1, 2, 3, 4,
}
fmt.Printf("n.a %p\n", &n.a)
fmt.Printf("n.b %p\n", &n.b)
fmt.Printf("n.c %p\n", &n.c)
fmt.Printf("n.d %p\n", &n.d)
```

#### 空结构体

空结构体是不占用空间的。

```go
package main

import (
	"fmt"
	"reflect"
	"time"
	"unsafe"
)

type ETS struct{}

func allEmptyStructIsSame() {
	// 所有的空结构体指向的地址是一样的  内核完全一样的
	var a ETS
	var b ETS
	var c struct{} // 空结构体类型
	fmt.Printf("address of a %p b%p c%p\n", &a, &b, &c)
	fmt.Printf("size of a %d b %d\n", unsafe.Sizeof(a), unsafe.Sizeof(b))
	fmt.Printf("size of a %d b %d\n", reflect.TypeOf(a).Size(), reflect.TypeOf(b).Size())
}

// 空结构体的应用场景
func scenariosOfEmptyStruct() {
	set := map[int]struct{}{
		1: struct{}{},
		2: struct{}{},
		4: struct{}{},
	}
	if _, exists := set[5]; exists {
		fmt.Println("5存在")
	} else {
		fmt.Println("5不存在")
	}

	blocker := make(chan struct{}, 0)
	go func() {
		time.Sleep(2 * time.Second)
		fmt.Println("done")
		blocker <- struct{}{}
	}()
	<-blocker // 等待子协程结束
}

func main() {
	allEmptyStructIsSame()
	scenariosOfEmptyStruct()
}
```

### 构造函数

没有传统意义上的构造函数（像其他编程语言中的 `class` 构造函数那样）。

但是，我们可以通过定义一个函数来创建并初始化结构体的实例，从而模拟构造函数的行为。

因为`struct`是值类型，如果结构体比较复杂的话，值拷贝性能开销会比较大，所以该构造函数返回的是结构体指针类型。

```go
package main

import "fmt"

type Student5 struct {
	name   string
	age    int
	gender byte
	score  [3]float64
}

// 构造函数

func NewStudent(name string, age int, gender byte, score [3]float64) *Student5 {
	return &Student5{
		name:   name,
		age:    age,
		gender: gender,
		score:  score}
}
func main() {
	var s = NewStudent("xiaoxiao", 18, 1, [3]float64{100, 99, 88})
	fmt.Println(s)
}
```

### 方法和接收者

Go语言中的`方法（Method）`是一种作用于特定类型变量的函数。这种特定类型变量叫做`接收者（Receiver）`。接收者的概念就类似于其他语言中的`this`或者 `self`。

方法的定义格式如下：

```go
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
    函数体
}
```

其中，

- 接收者变量：接收者中的参数变量名在命名时，官方建议使用接收者类型名称首字母的小写，而不是`self`、`this`之类的命名。例如，`Person`类型的接收者变量应该命名为 `p`，`Connector`类型的接收者变量应该命名为`c`等。
- 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
- 方法名、参数列表、返回参数：具体格式与函数定义相同。

```go
package main

import "fmt"

type Player struct {
	name        string
	healthPoint int
	magic       int
}

func NewPlayer(name string, healthPoint int, magic int) *Player {
	return &Player{
		name:        name,
		healthPoint: healthPoint,
		magic:       magic}
}

// attach 归属到 Player 结构体，只有Player结构体的对线才能调用此方法
func (p *Player) attack() {
	fmt.Printf("%s attack 10\n", p.name)
	p.healthPoint -= 10
}

func (p *Player) attacked() {
	fmt.Printf("%s attacked ... \n", p.name)
}
func main() {
	p1 := NewPlayer("wawa", 100, 100)
	p2 := NewPlayer("dada", 100, 100)
	p1.attack() // 会将 p1 结构体对象赋值给p
	p2.attack()
	p1.attacked()
	fmt.Println(p1.healthPoint)
}
```

方法与函数的区别是，函数不属于任何类型，方法属于特定的类型。

#### 指针类型的接收者

指针类型的接收者由一个结构体的指针组成，由于指针的特性，调用方法时修改接收者指针的任意成员变量，在方法结束后，修改都是有效的。

这种方式就十分接近于其他语言中面向对象中的`this`或者`self`。

#### 值类型的接收者

当方法作用于值类型接收者时，Go语言会在代码运行时将接收者的值复制一份。在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对副本，无法修改接收者变量本身。

```go
func (p *Player) attack() {  // 如果这个不是 *Player  最后的结果就是无法修改healthPoint的值
	fmt.Printf("%s attack 10\n", p.name)
	p.healthPoint -= 10
}
```

#### 指针类型接收者场景

1. 需要修改接收者中的值
2. 接收者是拷贝代价比较大的大对象
3. 保证一致性，如果有某个方法使用了指针接收者，那么其他的方法也应该使用指针接收者。

### 任意类型添加方法

在Go语言中，接收者的类型可以是任何类型，不仅仅是结构体，任何类型都可以拥有方法。 举个例子，我们基于内置的`int`类型使用type关键字可以定义新的自定义类型，然后为我们的自定义类型添加方法。

```go
//MyInt 将int定义为自定义MyInt类型
type MyInt int

//SayHello 为MyInt添加一个SayHello的方法
func (m MyInt) SayHello() {
	fmt.Println("Hello, 我是一个int。")
}
func main() {
	var m1 MyInt
	m1.SayHello() //Hello, 我是一个int。
	m1 = 100
	fmt.Printf("%#v  %T\n", m1, m1) //100  main.MyInt
}
```

**非本地类型不能定义方法，也就是说我们不能给别的包的类型定义方法。**

### 结构体的匿名字段

结构体允许其成员字段在声明时没有字段名而只有类型，这种没有名字的字段就称为匿名字段。

```go
type Person struct {
    string
    int
}

func main() {
    p1 := Person {
        "小王子",
        18,
    }
    fmt.Printf("%#v\n",p1)
    fmt.Printf(p1.string, p1.int)
}
```

**注意：**这里匿名字段的说法并不代表没有字段名，而是默认会采用类型名作为字段名，结构体要求字段名称必须唯一，因此一个结构体中同种类型的匿名字段只能有一个。

### 嵌套结构体

一个结构体中可以嵌套包含另一个结构体或结构体指针，就像下面的示例代码那样。

```go
//Address 地址结构体
type Address struct {
	Province string
	City     string
}

//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address Address
}

func main() {
	user1 := User{
		Name:   "小王子",
		Gender: "男",
		Address: Address{
			Province: "山东",
			City:     "威海",
		},
	}
	fmt.Printf("user1=%#v\n", user1)//user1=main.User{Name:"小王子", Gender:"男", Address:main.Address{Province:"山东", City:"威海"}}
}
```

#### 匿名字段

上面user结构体中嵌套的`Address`结构体也可以采用匿名字段的方式，例如：

```go
//Address 地址结构体
type Address struct {
	Province string
	City     string
}

//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address //匿名字段
}

func main() {
	var user2 User
	user2.Name = "小王子"
	user2.Gender = "男"
	user2.Address.Province = "山东"    // 匿名字段默认使用类型名作为字段名
	user2.City = "威海"                // 匿名字段可以省略
	fmt.Printf("user2=%#v\n", user2) //user2=main.User{Name:"小王子", Gender:"男", Address:main.Address{Province:"山东", City:"威海"}}
}
```

当访问结构体成员时会先在结构体中查找该字段，找不到再去嵌套的匿名字段中查找。

#### 字段冲突

嵌套结构体内部可能存在相同的字段名。在这种情况下为了避免歧义需要通过指定具体的内嵌结构体字段名。

```go
//Address 地址结构体
type Address struct {
	Province   string
	City       string
	CreateTime string
}

//Email 邮箱结构体
type Email struct {
	Account    string
	CreateTime string
}

//User 用户结构体
type User struct {
	Name   string
	Gender string
	Address
	Email
}

func main() {
	var user3 User
	user3.Name = "沙河娜扎"
	user3.Gender = "男"
	// user3.CreateTime = "2019" //ambiguous selector user3.CreateTime
	user3.Address.CreateTime = "2000" //指定Address结构体中的CreateTime
	user3.Email.CreateTime = "2000"   //指定Email结构体中的CreateTime
}
```

### 继承

Go语言中使用结构体也可以实现其他编程语言中面向对象的继承。

```go
package main

import "fmt"

type Animal struct {
	name string
}

func (a *Animal) eat() {
	fmt.Printf("%s 在 eat\n", a.name)
}
func (a *Animal) sleep() {
	fmt.Printf("%s 在 sleep\n", a.name)
}

type Dog struct {
	kind    string
	*Animal // Dog 继承了 Animal
}

func (d Dog) bark() {
	fmt.Printf("%s 吠\n", d.name)
}

type Cat struct {
	kind    string
	*Animal // Cat 继承了 Animal
}

func main() {
	d1 := &Dog{
		kind: "hashiqi",
		Animal: &Animal{
			name: "huahua",
		},
	}
	fmt.Println(d1.kind)
	fmt.Println(d1.name)
	d1.eat()
	d1.bark()

	c1 := &Cat{
		kind: "jumao",
		Animal: &Animal{
			name: "jindou",
		},
	}
	fmt.Println(c1.kind)
	fmt.Println(c1.name)
	c1.sleep()
	c1.eat()
}
```

### 字段可见性

结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问）。

### JSON序列化

JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。JSON键值对是用来保存JS对象的一种方式，键/值对组合中的键名写在前面并用双引号`""`包裹，使用冒号`:`分隔，然后紧接着值；多个键值之间使用英文`,`分隔。



```go
package main

type Address struct {
	Province, City string
}

type Stu struct {
	// 这边首字母不能用小写 序列化失败 可见性的问题
	// 然后在序列化数据的时候，可以使用标签  `json:"name"`  这样在序列化数据的时候就是小写 name
	Name    string `json:"name"`
	Age     int
	Address Address
}

func main() {
	/*	var map01 = map[int]string{1: "111", 2: "222", 3: "333"}
		var stu01 = Stu{Name: "yy", Age: 18, Address: Address{Province: "sc", City: "cd"}}
	*/

	// 序列化
	/*	map01Json, _ := json.Marshal(map01)
		fmt.Println(string(map01Json))

		stu01Json, _ := json.Marshal(stu01)
		fmt.Println(stu01)
		fmt.Println(string(stu01Json))*/

	// 写入文件
	/*	err := os.WriteFile("data.json", stu01Json, 0666)
		if err != nil {
			fmt.Println(err)
		}*/

	// 反序列化
	/*	bytes, _ := os.ReadFile("data.json")
		fmt.Println(string(bytes))
		// 将json字符串转换成go的数据结构
		var stu Stu
		err := json.Unmarshal(bytes, &stu)
		if err != nil {
			fmt.Println(err)
		}
		fmt.Println(stu)*/

	/*	var s = `{"name":"oi"}`
		var m map[string]string
		json.Unmarshal([]byte(s), &m)
		fmt.Println(m)*/
}
```

### 标签Tag

`Tag`是结构体的元信息，可以在运行的时候通过反射的机制读取出来。 `Tag`在结构体字段的后方定义，由一对**反引号**包裹起来，具体的格式如下：

```go
`key1:"value1" key2:"value2"`
```

结构体tag由一个或多个键值对组成。键与值使用冒号分隔，值用双引号括起来。同一个结构体字段可以设置多个键值对tag，不同的键值对之间使用空格分隔。

**注意事项：** 为结构体编写`Tag`时，必须严格遵守键值对的规则。结构体标签的解析代码的容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，通过反射也无法正确取值。例如不要在key和value之间添加空格。

例如我们为`Student`结构体的每个字段定义json序列化时使用的Tag：

```go
//Student 学生
type Student struct {
	ID     int    `json:"id"` //通过指定tag实现json序列化该字段时的key
	Gender string //json序列化是默认使用字段名作为key
	name   string //私有不能被json包访问
}

func main() {
	s1 := Student{
		ID:     1,
		Gender: "男",
		name:   "沙河娜扎",
	}
	data, err := json.Marshal(s1)
	if err != nil {
		fmt.Println("json marshal failed!")
		return
	}
	fmt.Printf("json str:%s\n", data) //json str:{"id":1,"Gender":"男"}
}
```

### 补充

因为slice和map这两种数据类型都包含了指向底层数据的指针，因此我们在需要复制它们时要特别注意。我们来看下面的例子：

```go
type Person struct {
	name   string
	age    int8
	dreams []string
}

func (p *Person) SetDreams(dreams []string) {
	p.dreams = dreams
}

func main() {
	p1 := Person{name: "小王子", age: 18}
	data := []string{"吃饭", "睡觉", "打豆豆"}
	p1.SetDreams(data)

	// 你真的想要修改 p1.dreams 吗？
	data[1] = "不睡觉"
	fmt.Println(p1.dreams)  // ?
}
```

正确的做法是在方法中使用传入的slice的拷贝进行结构体赋值。

```go
func (p *Person) SetDreams(dreams []string) {
	p.dreams = make([]string, len(dreams))
	copy(p.dreams, dreams)
}
```

同样的问题也存在于返回值slice和map的情况，在实际编码过程中一定要注意这个问题。
