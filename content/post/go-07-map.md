---
title: golang基础-map
lastmod: 2025-05-13T16:43:23+08:00
date: 2025-05-013T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-07.html
toc: true
---

## map

map 是一种无序的`键值对`的集合。

map 最重要的一点是通过 key 来快速检索数据，key 类似于索引，指向数据的值。

map 是一种集合，所以我们可以像迭代数组和切片那样迭代它。不过，map 是无序的，我们无法决定它的返回顺序，这是因为 map 是使用 hash 表来实现的。

map是引用类型，必须初始化才能使用

### 定义

```go
var mapname map[keyType]ValueType
```

- KeyType:表示键的类型。
- ValueType:表示键对应的值的类型。

map类型的变量默认初始值为nil，需要使用make()函数来分配内存。语法为：

```go
make(map[KeyType]ValueType, [cap])
```

```go
	var m1 map[string]int // 未初始化为 nil
	fmt.Println(m1 == nil) // true
```

**切记不要使用new创建map，否则会得到一个空引用的指针**

其中cap表示map的容量，该参数虽然不是必须的，但是我们应该在初始化map的时候就为其指定一个合适的容量。

`当 map 增长到容量上限的时候，如果再增加新的 key-value，map 的大小会自动加 1，所以出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明。`

### 基本使用

```go
	m2 := make(map[int]string, 8)
	fmt.Println(m2)
	m2[0] = "a"
	m2[1] = "b"
	fmt.Printf("%#v\n", m2)
	fmt.Printf("%T\n", m2)

	// 声明时填充元素
	m3 := map[int]bool{
		1: true,
		2: false,
	}
	fmt.Println(m3)
```

### map嵌套时key的赋值

```go
	var m4 = map[int]map[int]string{}
	m4[0] = map[int]string{1: "aaa"}
	fmt.Println(m4) // map[0:map[1:aaa]]
```

### 判断是键否存在

Go语言中有个判断map中键是否存在的特殊写法，格式如下:

```go
value, ok := map[key]
```

```go
	var scorpMap = map[string]int{
		"张三": 80,
		"李四": 85,
	}
	v, ok := scorpMap["张三五"]
	if ok {
		fmt.Println("张三五成绩在scorpMap中", v)
	} else {
		fmt.Println("张三五不在scoreMap中")
	}
```

### 遍历

使用 `for range` 遍历 map

```go
func main() {
	scoreMap := make(map[string]int)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	scoreMap["娜扎"] = 60
	for k, v := range scoreMap {
		fmt.Println(k, v)
	}
}
```

可以只遍历 key

```go
func main() {
	scoreMap := make(map[string]int)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	scoreMap["娜扎"] = 60
	for k := range scoreMap {
		fmt.Println(k)
	}
}
```

**注意：** 遍历map时的元素顺序与添加键值对的顺序无关。

### 删除键值对

使用内置函数 delete() 删除map中一组键值对，使用方法

```go
delete(map, key)
```
- map:表示要删除键值对的map
- key:表示要删除的键值对的键

```go
	// 删除
	delete(scorpMap, "张三")
	fmt.Println(scorpMap)
```

### 指定顺序遍历

```go
package main

import (
	"fmt"
	"math/rand"
	"sort"
)

func main() {
	var scorpMap = make(map[string]int, 100)
	for i := 0; i < 51; i++ {
		key := fmt.Sprintf("stu%02d", i)
		value := rand.Intn(100)
		scorpMap[key] = value
	}
	keys := make([]string, 0, 100)
	for key := range scorpMap {
		keys = append(keys, key)
	}
	sort.Strings(keys)

	for _, k := range keys {
		fmt.Println(k, scorpMap[k])
	}
}
```

### 元素类型为map的切片

```go
	var mapSlice = make([]map[string]int, 8) // 完成切片的初始化
	mapSlice[0] = make(map[string]int, 8)    // 完成map的初始化
	mapSlice[0]["张三"] = 90
	mapSlice[1] = map[string]int{"李四": 80} // 这种方式也可以
```

### 值为切片的map

```go
	var sliceMap = make(map[string][]int, 8)
	v1, ok1 := sliceMap["成都"]
	if ok1 {
		fmt.Println(sliceMap["成都"], v1)
	} else {
		sliceMap["成都"] = make([]int, 8)
		sliceMap["成都"][0] = 100
		sliceMap["成都"][1] = 200
		sliceMap["成都"][2] = 300
	}
	sliceMap["北京"] = make([]int, 8)
	sliceMap["北京"][0] = 1001
	sliceMap["北京"][1] = 1002
	// 遍历
	fmt.Println(sliceMap)
	for k2, v2 := range sliceMap {
		fmt.Println(k2, v2)
	}
```



```go
	// 统计一个字符串中每个单词出现的次数
	// "how do you do" 中每个单词出现的次数
	var s = "how do you do"
	var wordCount = make(map[string]int, 8)
	words := strings.Split(s, " ")
	for _, w := range words {
		k, ok := wordCount[w]
		if ok {
			wordCount[w] = k + 1
		} else {
			wordCount[w] = 1
		}
	}
	fmt.Println(wordCount)
```

