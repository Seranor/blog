---
title: python-函数使用(三)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-15.html
toc: true
---

### 递归函数

> 函数在运行过程中，直接或间接调用了自身

<!-- more -->

```python
# 官网表示:python默认的最大递归深度为1000次
# import sys
# print(sys.getrecursionlimit())  # 查看当前递归最大深度
# print(sys.setrecursionlimit(2000))  # 修改递归最大深度

# 无限自己调用自己，但是python限制了次数
# 无用递归1:
count = 1
def index():
    global count
    count += 1
    print(count)
    print('from index')
    index()
index()

# 无用递归2:
def func():
    print('from func')
    index()
def index():
    print('from index')
    func()
index()  # 两个函数互相调用
```

> 递归使用

```python
1.递推:一层层往下推导,每次往下推会相对于上一次复制度一定要有所下降
2.回溯:依据最后的结论往上推导出最初的答案
3.递归一定要有结束条件
'''
伪代码:可能无法运行,但是可以表述逻辑
'''

# 不打印列表，只打印数字
l1 = [1,[2,[3,[4,[5,[6,[7,[8,[9,[10,[11,[12,[13,[14,]]]]]]]]]]]]]]
def func1(l1):
    for i in l1:
        if type(i) is int:
            print(i)
        else:
            func1(i)
func1(l1)

# 实现阶乘
def factorial(n):
    if n == 1:
        return 1
    else:
        res = n*factorial(n-1)
        return res

print(factorial(4))  # 24
```

- 代码可视化运行: https://pythontutor.com/

### 二分法

```python
# 在列表中查出指定数字
# 第一种方法 直接用for循环从左往右依次查找

# 第二种使用二分法,二分法使用前提数据集必须有序
num = 567
l2 = [1, 11, 23, 43, 44, 57, 68, 76, 81, 99, 123, 222, 321, 432, 444, 567, 666, 712, 899, 999, 1111]

def get_num(num, l2):
    if len(l2) == 0:  # 列表中没有这个数
        return
    sli_num = len(l2) // 2  # 先从中间获取位置索引值
    if num > l2[sli_num]:  # 与列表中间的值做对比
        r_list = l2[sli_num + 1:]  # 如果大于中间值,截取优化右边为新列表
        print(r_list)
        get_num(num, r_list)  # 重新走到这个函数
    elif num < l2[sli_num]:  # 如果小于中间值,目标值就在左边
        l_list = l2[:sli_num]  # 将左边记录为新列表
        print(l_list)
        get_num(num, l_list)  # 继续走到这个函数
    else:
        print(num)  # 刚好等于中间值的情况

get_num(444, l2)
```

### 三元表达式

```python
1.当功能需求仅仅是二选一的情况下 那么推荐使用三元表达式
# 常规使用时
def max_num(a, b):
    if a > b:
      return a
    else:
      return b

# 使用三元表达式后
def max_num(a, b):
    return a if a > b else b

# 条件成立抛出if前面的值，否则就抛出else后面的值

2.虽然可以嵌套使用,但是不推荐

# 求三个数最大值
def max_num(a, b, c):
    return a if a > b else (b if b > c else c)

print(max_num(10, 22, 33))

# 案例二
is_free = input("是否收费Y/N:").strip().upper()
res = '收费'if is_free == 'Y' else '免费'
print(res)

# 案例三
username = input("username:").strip()
res = 'NB' if username == 'jason' else 'LB'
print(res)
```

### 列表生成式

```python
# 给列表中所有的人名加上_NEW后缀
name_list = ['jason', 'kevin', 'tony', 'jerry']

# 方式一:for循环加字符拼接
new_list = []
for i in name_list:
    new_list.append(i + '_NEW')
print(new_list)


# 方式二:列表生成式
new_list = ['%s_NEW' % i for i in name_list]
print(new_list)
```

### 字典生成式

```python
# 将两个列表合成为一个字典
l1 = ['jason', 18, 'read']
l2 = ['name', 'age', 'hobby']

new_dict = {}

for i in range(len(l2)):
    new_dict[l2[i]] = l1[i]
print(new_dict)


name_list = ['jason', 'tony', 'tom']
res = {i: j for i, j in enumerate(name_list, start=1)}

print(res)  # {1: 'jason', 2: 'tony', 3: 'tom'}

# 枚举
'''
enumerate(l1)
    针对该方法使用for循环取值 每次会产生两个结果
        第一个是从0开始的数字
        第二个是被循环对象里面的元素
    还可以通过start参数控制起始位置
'''
```

### 匿名函数

```python
# 匿名函数:没有名字的函数
'''
语法格式:
    lambda 形参:返回值
'''

# 简单使用
eg1:
a = (lambda x, y: x * y)(5, 6)
print(a)

eg2:
res = [x**2 for x in range(10)]
print(res)

res2 = [(lambda x:x**2)(i) for i in range(10)]
print(res2)

eg3:
print((lambda a, b: a if a > b else b)(10, 11))

# 匿名函数一般不会单独使用，会配合其他函数一起使用
# map() 映射
# 使用 map(function, iterable)
#         函数名,可迭代对象
l = [1, 2, 3, 4, 5, 6, 7, 8, 9]
def double(i):
    return i ** 2
print(list(map(double, l)))

print(list(map(lambda x: x ** 2, l)))
```
