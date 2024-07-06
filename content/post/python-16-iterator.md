---
title: python-迭代器
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-16.html
toc: true
---

### 1. 常用内置函数

#### 1.1 map()

```python
# map()  # 映射
l = [11, 22, 33, 44]
res = map(lambda x: x + 1, l)  # 循环获取列表中的每个元素并交给匿名函数保存返回值
print(list(res))  # [12, 23, 34, 45]
```

<!-- more -->

#### 1.2 zip()

```python
# zip() 拉链  按最少元素
l1 = [1, 2, 3, 4, 5]
l2 = ['jason', 'tony', 'xxx', 'tom', 'bob']
res = zip(l1, l2)
print(list(res))
# [(1, 'jason'), (2, 'tony'), (3, 'xxx'), (4, 'tom'), (5, 'bob')]
```

#### 1.3 max()和 min()

```python
# max()求最大值 min()求最小值
l3 = [115, 292, 303, 414, 526, 809, 910, 101]
print(max(l3))  # 910
print(min(l3))  # 101

# 如果只接字典时会将K值进行比较
d1 = {"alex": 1000000,
      "tony": 20000,
      "jason": 3000000,
      "tom": 8900000000,
      }

print(max(d1, key=lambda key: d1[key]))  # tom  循环取值再比较大小，返回K
print(min(d1, key=lambda key: d1[key]))  # tony
```

#### 1.4 filter()

```python
# filter() 过滤
l3 = [115, 292, 303, 414, 526, 809, 910, 101]
res = filter(lambda x: x > 400, l3)
print(list(res))  # [414, 526, 809, 910]
```

#### 1.5 reduce()

```python
# reduce() 归总
from functools import reduce

l3 = [115, 292, 303, 414, 526, 809, 910, 101]
res1 = reduce(lambda x, y: x + y, l3)
res2 = reduce(lambda x, y: x + y, l3, 100)  # 还可以继续添加额外的元素
print(res1) # 3470
print(res2) # 3570
```

### 2. 可迭代对象

1. 迭代:迭代即更新换代,每次的更新都必须依赖上一次的结果
   迭代提供了一种不依赖索引取值的方
2. 可迭代对象:内置 `__intr__`方法的都称之为可迭代对象，内置可通过`.`的方式查看
3. 双下滑线开头双下滑线结尾的方法叫双下方法名，面向对象的时候为了与隐藏变量名区分开

```python
# 通过变量名.__查看是否有intr
i = 12  # 没有
f = 11.11  # 没有
s = 'jason'  # 有
l = [111,22,33,4]  # 有
d = {'username':'jason','pwd':123}  # 有
t = (11,22,33)  # 有
se = {11,22,33}  # 有
b = True  # 没有
file = open(r'a.txt','w',encoding='utf8')

"""
含有__iter__的有
    字符串 列表 字典 元组 集合 文件对象
上述通常为可迭代对象
"""

# 两种结果一样
print(d.__iter__()) # <dict_keyiterator object at 0x7fdf258b4a98>
print(iter(d))

"""
可迭代对象调用__iter__方法会变成迭代器对象(老母猪)

__iter__方法在调用的时候还有一个简便的写法iter()
    一般情况下所有的双下方法都会有一个与之对应的简化版本 方法名()
"""
```

### 3. 迭代器对象

1. 迭代器对象:即含有`__iter__`方法，又含有`__next__`方法
2. 可以让可迭代对象执行`__iter__`方法后就可以生成迭代器对象
3. 迭代器对象无论执行多少次`__iter__`方法还是迭代器对象(本身)
4. 迭代器提供了不依赖于索引取值的方式

```python
# 通过变量名.__查看是否有next方法
i = 12
f = 11.11
s = 'jason'
l = [111,22,33,4]
d = {'username':'jason','pwd':123}
t = (11,22,33)
se = {11,22,33}
b = True
file = open(r'a.txt','w',encoding='utf8')


res = s.__iter__()  # 转成迭代器对象
print(res.__next__())  # 迭代器对象执行__next__方法其实就是在迭代取值(for循环) j
print(res.__next__())  # 在取完元素之后会报错

# 下面结果是一样的，每次生成新的迭代器对象再执行__next__方法
print(s.__iter__().__next__())  # j
print(s.__iter__().__next__())  # j
```

### 4. for 循环本质

```python
# 循环打印每个元素,不使用for循环
# __iter__和__next__
l1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 22, 33, 44, 55]

# 将列表转为迭代器对象
res = l1.__iter__()
while True:
    print(res.__next__())  # 循环执行__next__取值，当取完元素之后会报错


for i in l1:
    print(i)

'''
for 循环内部原理
  1.将关键字in后面的数据先调用__iter__方法转为迭代器对象
  2.循环执行__next__方法
  3.在取完值后__next__会报错,但是for循环会自动捕获该错误并处理
'''
```

### 5. 异常捕获

```python
# 什么是异常
	代码运行出错会导致异常 异常发生后如果没有解决方案则会到底整个程序结束

# 异常三个重要组成部分
	1.traceback:提示错误的行
  2.XXXError:错误的类型
  3.错误类型冒号后面的内容:错误的详细原因(仔细看可能就会找到解决的方法)

# 错误的种类
    1.语法错误:不被允许的,出现了应该立刻修改!!!
    2.逻辑错误:可以被允许的,出现了之后尽快修改即可
   		'''修改逻辑错误的过程其实就是在从头到尾理清思路的过程'''

# 基本语法结构
    try:
        有可能会出错的代码
    except 错误类型 as e:
        出错之后对应的处理机制(e是错误的详细信息)
    except 错误类型 as e:
        出错之后对应的处理机制(e是错误的详细信息)
    except 错误类型 as e:
        出错之后对应的处理机制(e是错误的详细信息)

  eg:
  try:
      int('abc')
  except NameError as e:
      print('变量名name不存在',e)
  except ValueError:
      print('值不是纯数字')

# 捕获万能异常
  try:
      # int('abc')
      print(name)
      # l = [11]
      # l[100]
  except Exception:
      print('你来啥都行 无所谓')

"""
异常捕获句式和万能异常
    1.有可能会出现错误的代码才需要被监测
    2.被监测的代码一定要越少越好
    3.异常捕获使用频率越低越好
"""

# while 使用__next__方法抛出异常
l1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 22, 33, 44, 55]
res = l1.__iter__()

try:
    while True:
        print(res.__next__())

except Exception:
    pass
```
