---
title: python基础-02
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-04.html
toc: true
---

## 1、数据类型

### 1.1 字符串 str

<!-- more -->

```python
作用:主要记录描述性质的数据，例如姓名、地址、邮箱......

定义:
  方式1:使用单引号,eg:
      name = 'hello'
  方式2:使用双引号,eg:
      name = "hello"
  方式3:使用三引号,eg:
      name = '''hello'''
  方式4:使用三引号,eg:
      name = """hello"""

  类型查看:
    str_a = 'hello'
		print(type(str_a))
    结果:<class 'str'>

  三引号说明:
    1.两个三引号都支持换行
    2.定义字符的多种方式原因
    	print('鲁迅说：'我没说过这句话'')  # 这语句就是错误的了
			print('鲁迅说："我没说过这句话"')  # 这条语句就正确了
    3.print('It\'s a dog')  # 可以用\ 进行转义为本身含义
```

### 1.2 列表 list

```python 
作用:能够存储多个数据并且可以方便的取出任意个数
特征:中括号括起来,内部可以存放多个元素,元素与元素之间用逗号隔开,元素可以是任意类型
eg:
  first_list = ["hello", "world", 123, 12344, ["test", 555, ["two", "results", 777]]]
  print(type(first_list))    # 结果:<class 'list'>
索引取值(从0开始的连续数字)
  print(first_list[1])  # world
  print(first_list[3])  # 12344

取值练习:
  取例中的results
  方法一:逐一提取
    l1 = first_list[4]  # ['test', 555, ['two', 'results', 777]]
    l2 = l1[2]  # ['two', 'results', 777]
    l3 = l2[1]  # results
    print(l3)
  方法二:熟悉之后一步到位
    print(first_list[4][2][1])
```

### 1.3 字典 dict

```python
作用:能够更加精准的存储数据
定义:大括号括起来,内存可以存放多个元素,元素与元素之间逗号隔开,元素是K:V键值对的形式
    K是对数据的描述,V是所存的数据
eg:
  first_dict = {
    'username': 'root',
    'passwd': '123456',
    'hostname': 'node1'
  }
  print(type(first_dict))  # <class 'dict'>
取值:
  1.字典无法索取值
  2.字典取值需要借助K
  eg:
    print(first_dict['username'])  # root

取值练习:
info = {'username': 'root', 'addr': ['安徽', '芜湖', {'国家': '中国', '编号': [11, 22, '中国最牛逼']}]}
方式一:
    d1 = info['addr']  # '安徽', '芜湖', {'国家': '中国', '编号': [11, 22, '中国最牛逼']}]
  	d2 = d1[2]  # {'国家': '中国', '编号': [11, 22, '中国最牛逼']}
    d3 = d2['编号']  # [11, 22, '中国最牛逼']
    d4 = d3[2]  # 中国最牛逼
方式二:
		print(info['addr'][2]['编号'][2])

```

### 1.4 布尔值 bool

```python
作用:用于判断事物的对错
定义:
  True  # 正确的
  False  # 错误的
  #ps: 首字母大写

布尔变量的命名一般采用is开头,eg:
  is_right = True
  is_delete = False

  print(type(is_right))  # <class 'bool'>
数据类型转换为布尔值的注意点:
  0, None, '', [], {}
以上转换为布尔值的False,其他情况都是True

其他:生活中数据存储的销户,很大概率并没有删除用户的数据,而是通过数据的唯一标识进行过滤掉,从而对外显示已删除
```

### 1.5 元组 tuple

```python
作用:与列表几乎一致,内部可以存放多个元素(可以看成是不可变的列表)
定义:用小括号括起来,存放多个元素,元素与元素之间逗号隔开,元素不支持修改
eg:
  t = (11, 22, 33, 44)
  print(type(t))  # <class 'tuple'>
```

### 1.6 集合 set

```python
作用:去重和关系运算
定义:用大括号括起来,内存可以存放多个元素,元素与元素之间逗号隔开,元素不是K:V键值对
eg:
  s = {11, 22, 33, 44}
	print(type(s))  # <class 'set'>

定义空集合
  s = set()
  print(type(s))  # <class 'set'>

默认情况下使用{}是空字典
  s = {}
  print(type(s))  # <class 'dict'>

```

## 2、输入与输出

### 2.1 输入

```python
输入:程序接收用户输入的数据功能,使用内置函数input()
  input()
  	1.接收到的任意输入的数据都会处理为字符串类型
    2.程序执行到input时会等待输入数据才开始进行下一步操作
eg:
  username = input("请输入你的名字:")
  age = input("请输入你的年龄:")
  print(type(username))		# 查看username的数据类型
  print(type(age))  # 查看age的数据类型
  print(username, age)  # 将输入的数据进行打印

res:
  请输入你的名字:tom  # 输入的tom
  请输入你的年龄:18   # 输入的18
  <class 'str'>  # 显示username的数据类型为str
  <class 'str'>  # 显示age的数据类型也是str
  tom 18  # 输出结果与输入结果一致
```

### 2.2 输出

```python
输出:程序输出内容给用户,内置函数print()
  print()
    1.括号内可以使用逗号将多个元素一起打印
    2.自带end参数控制打印的排版
    	eg:
        print('test', end='&')
        print(123)
      res:
        test&123
```

### 2.3 格式化输出

```python
格式化输出:将字符串中某些内容替换掉再输出就是格式化输出
1.先使用占位符 %s
2.再使用%按照位置进行替换
eg:
  res = '亲爱的%s你好！你%s月的话费是%s，余额是%s'
	print(res % ('jason', 11, 100, 999))
	print(res % ('tony', 11, 200, -100))
	print(res % ('kevin', 11, 500, -999))
res:
  亲爱的jason你好！你11月的话费是100，余额是999
	亲爱的tony你好！你11月的话费是200，余额是-100
	亲爱的kevin你好！你11月的话费是500，余额是-999

%d占位符只能给数字占位
print('%08d' % 123)
print('%08d' % 6666666666666)

res:
  00000123
  6666666666666
# 08导致输出结果会保留8位,不足的用0补齐,超过的直接显示源数据
```

## 3、基本运算符

### 3.1 算数运算符

a = 10 , b = 20

| 运算符 |                    描述                    | 实例             |
| :----: | :----------------------------------------: | ---------------- |
|   +    |                两个对象相加                | a + b 值为 30    |
|   -    |       得到负数或是一个数减去另一个数       | a - b 值为 -10   |
|   \*   | 两个数相乘或是返回一个被重复若干次的字符串 | a \* b 值为 200  |
|   /    |                  x 除以 y                  | b / a 值为 2     |
|   //   | 取整除 - 返回商的整数部分（**向下取整**）  | 9 // 2 值为 4    |
|   %    |            取模,返回除法的余数             | b % a 输出结果 0 |
|  \*\*  |              返回 x 的 y 次幂              | 2 \*\* 3 值为 8  |

### 3.2 比较运算符

| 比较运算符 | 描述                                    |
| :--------: | --------------------------------------- |
|     ==     | 等于,两边相等为 True,否则返回 False     |
|     !=     | 不等于,两边不相等为 True,否则返回 False |
|     >      | 大于                                    |
|     >=     | 大于等于                                |
|     <      | 小于                                    |
|     <=     | 小于等于                                |

### 3.3 赋值运算符

#### 3.3.1 增量赋值

| 赋值运算符 | 描述         | 实例                  |
| ---------- | ------------ | --------------------- |
| =          | 简单赋值运算 | a = 10                |
| +=         | 加法赋值运算 | a +=1 相当于 a = a+1  |
| -=         | 减法赋值运算 | a -= 1 相当于 a = a-1 |
| \*=        | 乘法赋值运算 |                       |
| /=         | 除法赋值运算 |                       |
| //=        | 取整赋值运算 |                       |
| %=         | 取余赋值运算 |                       |
| \*\*=      | 幂赋值运算   |                       |

#### 3.3.2 链式赋值

```python
可以把同一个值同时赋值个多个变量名
eg:
  a = 10
  b = a
  c = b

  # 链式赋值可以一行解决
  a = b = c = 10
```

#### 3.3.3 交叉赋值

```python
eg:
  a = 10
  b = 22

  需要a和b交换
  方式1:
 		tmp = a  # 引入第三变量暂存a的值
		a = b  # 变量a指向变量b,此时a的值为22
		b = tmp  # 变量b指向tmp,此时b的值就是10，完成互换吧
		print(a, b)
  方式2:
    a, b = 22, 10  # 简单粗暴
```

#### 3.3.4 解压赋值

```python
将列表中的多个值取出来依次赋值给多个变量名
eg:
  eg_list = [12, 13, 14, 15]
  a = eg_list[0]
  b = eg_list[1]
  c = eg_list[2]
  d = eg_list[3]
  print(a, b, c, d,)  # 输出为12 13 14 15

  解压赋值可以这样写:
    a, b, c, d = eg_list

  解压赋值注意事项:
    1.等号左边的变量名格式化必须与右面包含的格式相同
    2.可以使用*_打破上述规则
      eg:
        a, *_, d = eg_list
        print(a, _, b)  # 结果为 12 [13, 14] 15
     说明:
      * 可以接收多余的元素,组成列表赋值给后面的变量名
      _ 作为单独变量名时,通常表达指向的值无用
```

### 3.4 逻辑运算符

```python
在python逻辑运算符就三个
and	与:
	用于连接多个条件并且多个条件必须都成立才可以
or	或:
	用于连接多个条件并且多个条件只要有一个成立即可
not 非:
	取反

print(2 > 1 and 1 != 1 and True and 3 > 2)  # False
print(2 > 1 or 1 != 1 or True or 3 > 2)  # True
print(not True)  # False

# ps:三个连接符在混合使用的时候是有优先级的,但是我们在编写的时候应该人为的规定好优先级,()优先级最高
```

### 3.5 成员运算

```python
定义:判断某个个体在不在某个群体内
关键字:
  in  			(在)
  not in    (不在)
eg:
   name_list = ['jason', 'kevin', 'tony', 'jackson']
    name = input('请输入您要查询的学生姓名>>>:')
    print(name in name_list)
    print(name not in name_list)
    # 最终返回的是True或者False

    #字典默认暴露给外界的只有K
    print('jason' in {'username': 'jason', 'age': 18})  # False
    print('username' in {'username': 'jason', 'age': 18}) # True
```

### 3.6 身份运算

```python
定义:判断两个数据 值和内存地址是否相等
符号:
  ==  (只判断值)
  is  (判断内存地址)
eg:
    s1 = ['jason', 'kevin', 'tony', 'jackson']
    s2 = ['jason', 'kevin', 'tony', 'jackson']
    print(s1 == s2)  # True
    print(id(s1),id(s2))  # 查看相当于内存地址的数字
    print(s1 is s2)  # False
结论:
  值相等内存地址不一定相等
  内存地址相等值一定相等
```
