---
title: python基础-01
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-03.html
toc: true
---

## 1、Pycharm 基本使用

### 1.1 新建项目

<!-- more -->

![Fg5CT2](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/Fg5CT2.png)

![koHl29](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/koHl29.png)

### 1.2 主题设置

![Yr8VSa](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/Yr8VSa.png)

### 1.3 Pycharm 切换解释器

![xy5MbY](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/xy5MbY.png)

### 1.4 调整字体

![kBm63I](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/kBm63I.png)

### 1.5 运行 python 脚本文件

鼠标右键到项目目录之后可以创建文件夹与文件，在代码空白处右键选择如下的 Run 即可运行 python 脚本

![dI0FCk](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/dI0FCk.png)

## 2、Python 的注释语法

### 2.1 注释

```python
"""注释是代码之母"""
注释：对代码的解释和说明，目的是为了让人们能够轻松的了解代码，注释不参与持续的运行
```

### 2.2 使用注释

```python
方式一:使用井号
  # 这是一行注释
方式二:使用三引号(单引号和双引号)
  """
  这是多行注释
  这是多行注释
  """
```

2.3 Pycharm 注释快捷键

```python
Windows： ctrl + ?
Mac:      command + ?
选中多行代码之后执行快捷键就会被一起注释
```

## 3、变量

### 3.1 什么是变量

```python
变量即变化的量，用于记录事物的某种状态，是模仿人类事物记忆能力
```

### 3.2 使用变量

```python
日常生活种:
  姓名:xxx
  年龄:28
  爱好:学习
程序中:
  username = 'xxx'
  age      = 18
  hobby    = 'music'
```

### 3.3 语法格式

```python
username = 'xxx'
变量名  赋值号 变量值
```

### 3.4 变量三要素

```python
1.变量的值
2.变量的内存地址
3.变量的数据类型

name = 'xxx'
print(name)         # 变量值
print(id(name))     # 返回一串数字 相当于是内存地址编号
print(type(name))   # 返回数据类型  <class 'str'>
```

![I6iPWT](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/I6iPWT.png)

### 3.5 底层原理

```python
# eg:
  age = 18

  '''
  遇到赋值号先看符号右边，再看到左边
  1.在内存中申请到了一块内存空间来存储18这个数字
  2.将18所在的内存空间地址指向绑定给变量名age
  3.后续如果要访问18可以直接通过访问变量age
  '''
```

![gyg2Ck](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/gyg2Ck.png)

### 3.6 Python 底层优化

```python
当值数据量很小的时候 如果有多个变量名需要使用 那么会指向同一块地址
"""
一个变量名只能指向一个内存地址
一个内存地址可以有多个变量名指向
"""
```

## 4、常量

```python
常量:主要记录一些不变的状态

在python中没有真正意义上的常量 我们墨守成规的将全大写的变量看成是常量
	HOST = '127.0.0.1'  # 一般情况下在配置文件中使用较多
在其他编程语言中是存在真正意义上的常量 定义了就无法修改
# JavaScript代码
	const pi = 3.14  # 定义常量
    pi = 4  # 不支持修改
# golang常量声明
const MAX = 1024
const (
  a = iota
  b = iota
  c = iota
)
```

## 5、垃圾回收机制

### 5.1 垃圾数据的定义

```python
在内存中没有任何变量名指向的数据
```

### 5.2 回收方案

#### 5.2.1 引用记数

```python
内存中变量值身上有几个变量名绑定引用计数就是几,只要不为0就不是垃圾
```

#### 5.2.2 标记清除

```python
当内存即将沾满的时候,python会自动暂停程序的执行,从头到尾将内存中数据进行扫描,并打上标记,之后一次性清除掉标记的数据
```

#### 5.2.3 分代回收

```python
会将数据的监管分为三个层次,随着层级的下降监督的频率降低
用时间换空间
```

![CR2fkx](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/CR2fkx.png)

## 6、数据类型

### 6.1 什么是数据类型

```python
存储数据的方式和表现形式有很多种，例如文本文件，视频文件，音频文件......
```

### 6.2 int 类型

```python
# int类型:整数类型,长度与其他语言对比无限
作用:可以记录人的年龄，人数......
eg:
  age = 18  # 直接些整数就是整型
```

### 6.3 float 类型

```python
# float可以理解为小数
作用:记录人的体重，薪资......
eg:
  salary = 3.14  # 直接写小数就是浮点型
```

## 7、代码规范

### 7.1 注释规范

```python
"""
1.警号与注释文本之间一定要有一个空格
2.如果单行注释跟在了一行代码的后面 需要先空两个再写
pycharm也提供自动化格式代码的功能
code
reformat code
Windows快捷键：ctrl + alt + l
Mac快捷键： option + command + L
"""

python代码编写规范  >>>: PEP8规范
# 如何快速掌握 借助于pycharm的自动化提示 前后对比 每天记忆
```

### 7.2 命名规范

```python
# 命名规范
    1.变量名只能由数字、字母、下划线任意组合
    	user@name(不对)、_(可以)、pwd_123_aaa(可以)
    2.变量名不能以数字开头，下划线建议不要开头因为有特殊含义
    3.变量名不能与关键字冲突
    4.变量名的命名一定要做到见名知意(重要)
    	'''变量名见名知意是核心 无论变量多长'''
# 命名风格
	1.驼峰体
    	大驼峰  # 所有单词首字母大写
        	UserNameFromDb
        小驼峰  # 第一首字母小写其余首字母大写
        	userNameFromDB
        """JavaScript推荐使用驼峰体"""
    2.下划线  # 单词与单词之间下划线隔开
    	user_name_from_db
        """python推荐使用下划线"""

# 好东西！！！输入中文即可给出对应英文命名
https://unbug.github.io/codelf/
```
