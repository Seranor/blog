---
title: python-内置方法(二)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-07.html
toc: true
---

## 1. 字符串内置方法

### 1.1 大小写转换

```python
res = 'jasOn123 JAsOn'

1.全转大写upper()
  print(res.upper())  # JASON123 JASON
```

<!-- more -->

```python
2.全转小写lower()
  print(res.lower())  # jason123 jason

3.eg:
  old_code = 'jAson123'
  code = input("请输入验证码:%s>>>:" % old_code).strip()
  if code.upper() == old_code.upper():  # 这里也可以用lower()
      print("验证码输入正确")
  else:
      print("验证码输入错误")
 # 忽略大小写,全部转为大写或者小写
```

### 1.2 判断大小写

```python
res1 = 'JASON'
res2 = 'jason'

1.判断是否纯大写isupper()
  print(res1.isupper())  # True
  print(res2.isupper())  # False

2.判断是否纯小写islower()
  print(res1.islower())  # False
  print(res2.islower())  # True
```

### 1.3 判断指定字符开头、结尾

```python
s1 = 'jason 123 newapeman heiheihei oldgirl'

1.判断字符串是否以指定的字符开头startswith()
  print(s1.startswith('jon'))  # False
  print(s1.startswith('jas'))  # True
  print(s1.startswith('jason 123'))  #True

2.判断字符串是否以指定的字符结尾endswith()
  print(s1.endswith('oldboy'))  # False
  print(s1.endswith('girl'))  # True
  print(s1.endswith('hei oldgirl'))  # True
```

### 1.4 格式化输出 format

```python
1.之前使用的是占位符 %s %d
2.字符串内置方法 format()

使用方式1:相当于占位符
  s1 = 'my name is {} my age is {}'
  print(s1.format('jason', 18))  # my name is jason my age is 18

使用方式2:大括号内写索引值可以打破顺序,并且可以反复使用相同位置的数据
  s2 = '{1} my name is {0} my age is {1} {1} {0}'  # 18 my name is jason my age is 18 18 jason
  print(s2.format('jason', 18))

使用方式3:大括号内写变量名
  s3 = ' my name is {name} my age is  {age}'
  print(s3.format(name='jason', age=18))  # my name is jason my age is  18
```

### 1.5 字符串的拼接

```python
1.使用 +
print('hello' + 'world')  # hello world

2.使用join()
  l = ['jason', 'tony', 'kevin']
  print(l[0] + '|' + l[1] + '|' + l[2])  # jason|tony|kevin
  print('|'.join(l))  # jason|tony|kevin

# l1 = ['jason', 123, 'tony']
# print('$'.join(l1))  # 报错
"""必须是字符串类型 (在python不同数据类型之间无法直接操作)"""
```

### 1.6 替换字符串中指定的字符

```python
replace()

eg:
  s4 = 'my name is tony tony tony my age is 18'
  print(s4.replace('tony', 'Bob'))  # my name is Bob Bob Bob my age is 18

# 替换指定字符的次数
  s4 = 'my name is tony tony tony my age is 18'
  print(s4.replace('tony', 'Bob', 1))  # my name is Bob tony tony my age is 18
```

### 1.7 判断是否纯数字

```python
isdigit()

eg:
  s1 = 'asd123'
  s2 = '123'
  print(s1.isdigit())  # False
  print(s2.isdigit())  # True

# 案例:判断用户输入的是否是纯数字
  real_age = 20
  while True:
      age = input("请输入猜测的年龄:").strip()  # 去除首尾的空格
      if age.isdigit():  # 判断输入的是否是纯数字
          if int(age) == real_age:  # 将输入的整数字符串转换为整型
              print("猜对了")
              break  # 猜对了就退出循环
          else:
              print("猜错了")
      else:
          print("请输入正确的数字")  # 如果用户输入的不是纯数字提示并再次循环
```

### 1.8 字体格式相关

```python
str1 = 'my namE iS Bob'
str2 = 'but'
str3 = 'tony123'

1.title()  # 所有单词首字母大写
  print(str1.title())  # My Name Is Bob

2.capitalize()  # 第一个单词首字母大写
  print(str1.capitalize())  # My name is bob

3.swapcase()  # 大小写互换
  print(str1.swapcase())  # MY NAMe Is bOB

4.find()  # 查看指定字符对应的起始索引值,从左往右找到一个就结束
  print(str1.find('n'))  # 3
  print(str1.find('nam'))  # 3  返回的是第三个字母n的索引值

5.center()  # 指定字符补齐指定个数,居中显示
  print(str2.center(15, '$'))  # $$$$$$but$$$$$$

6.ljust()  # 指定字符补齐指定个数,左对齐
  print(str2.ljust(15, '*'))  # but************

7.rjust()  # 指定字符补齐指定个数,右对齐
  print(str2.rjust(15, '%'))  # %%%%%%%%%%%%but

8.isalnum()  # 字符串中即可以包含数字也可以包含字母,返回布尔值
  print(str2.isalnum())  # True
  print(str3.isalnum())  # True

9.isalpha()  # 字符串中只包含字母,返回布尔值
  print(str2.isalpha())  # True
  print(str3.isalpha())  # False
```

## 2.列表内置方法

### 2.1 基本方法

```python
1.列表内一般都会存储相同数据类型的数据
2.list()  转换为列表类型,可以将支持for循环的数据类型转换成列表
eg:
  # print(list(123))  # 报错
  # print(list(123.21))  # 报错
  print(list('hello'))  # ['h', 'e', 'l', 'l', 'o']
  print(list({'username': 'jason', 'pwd': 123}))  # ['username', 'pwd']
  print(list((11, 22, 33)))  # [11, 22, 33]
  print(list({11, 22, 33}))  # [33, 11, 22]
```

### 2.2 列表增改数据

```python
name_list = ['jason', 'kevin', 'tony', 'jack']

1.改
  name_list[0] = 666
  print(name_list)  # [666, 'kevin', 'tony', 'jack']

2.增
  方式1: append()
    # 尾部追加
    name_list.appernd(666)
    print(name_list)  # ['jason', 'kevin', 'tony', 'jack', 666]

    # 尾部追加(将括号内的数据当成一个整体追加到列表末尾)
    name_list.append([111, 222, 333])
	  print(name_list)  # ['jason', 'kevin', 'tony', 'jack', [111, 222, 333]]

  方式2: insert()
    # 可以在指定索引值插入元素
    name_list.insert(3, 'vae')
		print(name_list)  # ['jason', 'kevin', 'tony', 'vae', 'jack']

    # 元素会被当成一个整体插入到指定索引位置
    name_list.insert(2, [11, 22])
		print(name_list)  # ['jason', 'kevin', [11, 22], 'tony', 'jack']

 方式3: extend()
    # 扩展元素
    name_list.extend([11, 22])
		print(name_list)  # ['jason', 'kevin', 'tony', 'jack', 11, 22]

    # 相当于for循环+append()
    l1 = [11, 22, 33]
    l2 = [1, 2, 3]
    for i in l2:
    		l1.append(i)
		print(l1)
```

### 2.3 列表删除数据

```python
name_list = ['jason', 'kevin', 'tony', 'jack']
1.通过del删除
  del name_list[0]  # 直接删除索引为0的值
  print(name_list)  # ['kevin', 'tony', 'jack']

2.remove()
  # 移除括号内的元素值
  name_list.remove('jack')
	print(name_list)  # ['jason', 'kevin', 'tony']
  print(name_list.remove('jack'))  # None

3.pop()
  # 弹出括号内的元素索引值,如果括号没有值,则默认弹出列表尾部的元素
  print(name_list.pop(0))  # jason
  print(name_list)  # ['kevin', 'tony', 'jack']
```

## 可变类型与不可变类型

```python
可变类型: 列表
    值改变,内存地址不变,修改的是原值
不可变类型: 整型 浮点型 字符串
    值改变,内存地址肯定变,产生了新值

eg:
  # 不可变类型
  str1 = 'hello world'
  print(str1.title())  # Hello World 结果操作之后的值,是一个新的值
  print(str1)  # hello world 原值并没有改变

  # 可变类型
  name_list = ['jason', 'kevin', 'tony', 'jack']
  print(id(name_list))  # 现在的值为 140618704502856 (不固定值)
	name_list[0] = 666
  print(name_list)  # [666, 'kevin', 'tony', 'jack']
	print(id(name_list))  # 修改之后值还是 140618704502856
```

## 实现队列与堆栈

```python
list1 = list()

# 进入
for i in range(10):
    list1.append(i)
    print(list1)  # 一个个进,从0开始
print(list1)

# 队列 先进先出
for j in range(10):
    del list1[0]  # 一个个删除,相当于一个个出来,每次出来都从索引0开始,从0开始删除到9
    print(list1)  # 根据循环打印出过程,可以直观看出先进先出
print(list1)

# 堆栈 先进后出
for m in range(10):
    list1.pop()  # 从尾部开始删除,相当于最后进来的先删除
    print(list1)
print(list1)
```
