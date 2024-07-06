---
title: python-文件与函数初识
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-11.html
toc: true
---

## 1. 文件

### 1.1 二进制模式读

```python
with open(r'a.txt', 'rb') as f:
    print(f.read())  # 读取的是二进制内容
    print(f.read().decode('utf8'))  # 经过住解码，能正常读取内容
    print(f.read(3).decode('utf8'))  # 三个字节为一个中文


'''
read()  括号内可以放数字
    在t模式下表示字符个数
    在b模式下表示字节个数
英文字符统一使用一个bytes来表示
中文字符统一使用三个bytes来表示
'''
```

<!-- more -->

### 1.2 文件内光标的移动

```python
with open(r'a.txt', 'rb') as f:
    print(f.read(6).decode('utf8'))
    print(f.tell())  # 查看光标移动了多少字节
    print(f.seek(8, 1))  # 从上面第6字节位置开始往后移动8个字节
    print(f.read().decode('utf8'))  # 查看上面移动后的结果

'''
seek()
  控制文件光标的移动,eg:
  f.seek(offset,whence)
      offset表示位移量,始终以字节为最小单位,正数从左往右，负数从右往左

      whence表示模式
          0:以文件开头为参考(支持tb两种模式)
          1:以当前位置为参考,只支持b模式
          2:以文件末尾为参考,只支持b模
'''
```

通过光标实现动态查看日志

```python
import time

with open(r'a.txt', 'rb') as f:
    # 直接将光标移动到文件末尾
    f.seek(0, 2)
    while True:
        # 从文件末尾一直读取文件内容
        line = f.readline()
        # 判断读取的内容是否为0
        if len(line) == 0:
            time.sleep(0.5)
        else:
            print(line.decode('utf8'), end='')
```

### 1.3 文件内容修改

#### 1.3.1 覆盖

```python
with open(r'c.txt','r',encoding='utf8') as f:
     data = f.read()  # 取出内容
     # print(type(data))
with open(r'c.txt','w',encoding='utf8') as f1:
     new_data = data.replace('tony','jason')  # 替换
     f1.write(new_data)  # 重新写入
```

#### 1.3.2 新建

```python
import os
with open('c.txt', mode='rt', encoding='utf-8') as read_f, \
        open('c.txt.swap', mode='wt', encoding='utf-8') as write_f:
    for line in read_f:
        write_f.write(line.replace('SB', 'kevin'))
os.remove('c.txt')  # 删除原文件
os.rename('c.txt.swap', 'c.txt')  # 重命名文件
```

## 2. 函数

### 2.1 函数语法结构

```python
def 函数名(参数1,参数2):
  '''函数注释'''
  函数体代码
  return 返回值

'''
1.def 定义函数的关键字 必要的
2.函数名  相当于变量名,命名规范与风格遵循变量名 必要的
3.参数 参数可以没有也可以有多个,表示在使用函数之前需要满足的一些条件 非必要
4.函数注释 介绍函数功能 参数使用,以及其他情况 非必要
5.函数体代码 函数核心的代码逻辑 必要的
6.return返回值 使用函数之后反馈给使用者的结果 非必要
'''
```

### 2.2 定义与调用

```python
1.函数必须先定义后调用(顺序不能乱)
2.函数在定义阶段只检测语法不执行代码
3.函数在调用阶段才会执行函数体代码
调用函数:函数名加括号,如果函数在定义阶段有参数则在调用阶段也需要给参数

函数在定义与调用阶段底层原理
1.在内存空间中申请一块空间存储函数体代码
2.将函数体代码所在的空间地址绑定给函数名
3.函数名加括号则会执行函数体代码
```

### 2.3 分类

```python
1.内置函数
  python解释器自带的(提前已经定义好,直接使用即可)

2.自定义函数
  自己写的函数
    1.无参函数:在定义函数阶段括号内没有写参数(变量名)
        def my_func():
            print("hello")
    2.有参函数:在定义函数阶段括号内写了参数(变量名)
        def my_func(a, b)
            print("hello")
    3.空函数:函数体代码为空(pass),虽然空函数本身没有意义,但是空函数可以提前规定好编写代码的思路
        def run():
          pass
```
