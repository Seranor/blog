---
title: python-生成器和常见内置函数
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-17.html
toc: true
---

### 异常捕获(二)

```python
try:
    name
except Exception as e:
    print("代码错误")
else:
    print('代码正常才会执行了')
finally:
    print('代码不管是否异常都会运行')

# 断言
name = 'jason'
assert isinstance(name, str)

# 主动抛出异常
raise ZeroDivisionError('除数不能为0')
```

<!-- more -->

### for 循环本质

```python
d = {'name': 'jason', 'age': 18}
res = d.__iter__()  # StopIteration的异常,该异常是在循环对象穷尽所有元素时的报错

# while实现循环打印
while True:
    try:
        print(res.__next__())
    except StopIteration as e:
        break

# for循环打印
for i in d:
    print(i)
```

### 迭代取值与索引取值对比

- 迭代取值

  1. 不依赖索引进行取值
  2. 取值的顺序都是固定的从左到右，无法重复获取

- 索引取值
  1. 可以重复取值
  2. 需要提供有序容器类型才可取值(不是通用方式)

### 生成器对象

```python
生成器其实就是自定义迭代器

# 定义阶段就是一个普通函数
def my_generator():
    print('first')
    yield 11
    print('second')
    yield 22

"""
当函数体内含有yield关键字 那么在第一次调用函数的时候
并不会执行函数体代码 而是将函数变成了生成器(迭代器)
"""

res = my_generator()  # 调用函数不执行函数体代码，而是将函数变成生成器(迭代器)
print(res)  # <generator object my_generator at 0x7fcc5f7d3888>
ret = res.__next__()  # 每执行一个__next__代码往下运行到yield停止 返回后面的数据
print(ret)  # first \n 11
ret = res.__next__()  # 再次执行__next__接着上次停止的地方继续往后 遇到yield再停止
print(ret)  # second \n 22
```

### 自定义 range 功能

```python
def my_range(start, stop=None, step=1):
    if not stop:
        stop, start = start, 0
    while start < stop:
        yield start
        start += step


for i in my_range(2, 10, 2):
    print(i)
```

### yield 传值

```python
def generator_func1(age):
    print('age is %s ' % age)
    while True:
        name = yield
        print('%s NB' % name)


res = generator_func1(18)  # 不会执行函数体代码，而是转换成生成器
res.__next__()
res.send('json')  # 给yield传值
res.send('xxx')  # 再次给yield传值
```

### yeild 与 return 对比

```python
相同点:可以返回值,支持多个并且组织成元组
不同点:
    yield:
      1. 函数体代码遇到yield不会结束,会'停住'
      2. yield可以将函数变成生成器,并且支持外界传值
    return:
      1. 函数体代码遇到return直接结束
```

### 生成器表达式

```python
# 列表生成式
l1 = [11, 22, 33, 44, 55, 66]
res = [i + 1 for i in l1 if i != 44]
print(res)  # [12, 23, 34, 56, 67]

# 生成器表达式
'''生成器表达式内部的代码只有在迭代取值的时候才会执行'''

res1 = (i + 1 for i in l1 if i != 44)
print(res1)  # <generator object <genexpr> at 0x7fbbb7e96ca8>
print(res1.__next__())  # 12
print(res1.__next__())  # 23
print(res1.__next__())  # 24
```

- 笔试题

  ```python
  # 求和
  def add(n, i):
      return n + i
  # 调用之前是函数 调用之后是生成器
  def test():
      for i in range(4):
          yield i
  g = test()  # 初始化生成器对象
  for n in [1, 10]:
      g = (add(n, i) for i in g)
      """
      第一次for循环
          g = (add(n, i) for i in g)
      第二次for循环
          g = (add(10, i) for i in (add(10, i) for i in g))
      """
  res = list(g)
  print(res)

  #A. res=[10,11,12,13]
  #B. res=[11,12,13,14]
  C. res=[20,21,22,23]
  #D. res=[21,22,23,24]
  ```

### 常见内置函数

```python
1. abs()  # 取绝对值
   print(abs(-10))  # 10

2. all()  any()
   l = [11, 22, 0]
   print(all(l))  # 所有元素为True才是True
   print(any(l))  # 所有元素有一个为True就是True

3. bin()  oct()  hex()
   print(bin(12))  # 0b1100  二进制
   print(oct(12))  # 0o14  八进制
   print(hex(12))  # 0xc  十六进制

4. bytes()  str()
   res = '测试'
   ret1 = bytes(res, 'utf8')
   print(ret1)  # b'\xe6\xb5\x8b\xe8\xaf\x95'
   ret2 = str(ret1, 'utf8')
   print(ret2)  # 测试

5. callable()  # 是否看调用(看是否能加括号运行)
   i = 1
	 def f():
       pass
   print(callable(i), callable(f))  # Flse True

6. chr()  ord()
   print(chr(65))  # A  按照ASICC码表的数字打印字符
   print(ord('A'))  # 65  按照ASICC码表的字符打印数字

7. complex()  复数
   print(complex(123))  # (123+0j)

8. dir()  # 查看当前对象可以调用的名字
   def f():
       pass
   print(dir(f))

9. divmod()  # 接收两个数字类型参数，返回一个包含商和余数的元组(a // b, a % b)
   print(divmod(101, 10))  # (10 1)
   应用:
      # 生成页数
      num, more = divmod(201, 10)
      if more:
          num += 1
      print('总共需要%s页' % num)

10. eval()  exec()  # 将字符串内的内容加载执行
    s1 = "print('hello')"
    s2 = '''
    for i in range(10):
        print(i)
    '''
    eval(s1)  # 只能执行简单的内容
    exec(s2)  # 可以执行复杂的内容

11. isinstance()  # 判断是否属于某个数据类型
    i = 1
		print(isinstance(i, int))  # True

12. pow()
		print(pow(4, 3))  # 64  4**3

13. round()
		print(round(4.5))  # 4
		print(round(4.6))  # 5

14. sum()  # 求和
    l = [11, 22, 33, 44]
    print(sum(l))  # 110 将列表 l 中的元素求和
```
