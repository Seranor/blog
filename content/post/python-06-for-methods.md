---
title: for循环及内置方法
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-06.html
toc: true
---

### 1. while 循环

#### 1.1 continue

<!-- more -->

```python
contiue可以让循环体代码直接回到条件判断处重新判断,相当于跳出本次循环
eg:
  # 使用while循环打印0-10但是不打印4
  num = 0
  while num < 11:
      if num == 4:
          num += 1  # 跳出的时候将num加1到num为5,好继续下次的循环,否则会卡在4一直循环
          continue  # 当num为4时跳出本次循环
      print(num)
      num += 1
```

#### 1.2 else

```python
当while循环没有被人为中断(break)的情况下才会走else
eg:
  count = 0
  while count < 5:
      print(count)
      count += 1
  else:
      print("被执行了")  # 结果:在打印了0-4之后,这段代码被执行了


  count = 0
  while count < 5:
      print(count)
      count += 1
      break  # 遇到break之后跳出了循环
  else:
      print("不被执行了")  # 这段就没有执行
```

#### 1.3 死循环

```python
一个靠自身控制无法终止的程序叫死循环,死循环会让CPU极度繁忙
eg：
  while True
      print(1)
```

### 2. for 循环

#### 2.1 基本使用

```python
1.for循环能做到的事情 while循环都可以做到,但是for循环语法更加简洁 并且在循环取值问题上更加方便
2.for循环一般用于遍历任意可迭代对象中的元素,可迭代对象包括字符串,列表,元组,集合和字典,字典默认只能取到K
3.变量名如果没有合适的名称,可以使用i,j,k,v,item
4.结构：
  for 变量 in 迭代对象:
      重复执行的代码
 5.eg:
   name_list = ['jason', 'tony', 'kevin', 'jack', 'xxx']
   # 循环打印列表中的每一个元素
   # while实现
     count = 0
     while count < 5:
         print(name_list[count])
         count += 1
   # for实现
     for i in name_list:
         print(i)
   # for循环打印字符串中每个字符
     for i in 'hello world'
         print(i)  # 每个字母都会被打印,中间的空格也回被打印
   # for循环字典,默认只能取到K
     d = {'username': 'jason', 'pwd': 123, 'hobby': 'read'}
     for k in d:
         print(k, d[k])  # 结果:前面是K值,后面是V值
```

![uVzeKa](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/uVzeKa.png)

#### 2.2 range

```python
1.第一种:一个参数,从零开始,顾头不顾尾
  eg:
    for i in range(10):
        print(i)  # 循环打印0-9
2.第二种:两个参数,自定义起始位置,顾头不顾尾
  eg:
    for i in range(4,10):
        print(i)  # 循环打印4-9
3.第三种:三个参数,第三个数字用来控制等差值
  eg:
    for i in range(2, 100, 10):
        print(i)  # 循环打印从2开始每隔10个数的值,2 12 22...92
"""
扩展:
    https://movie.douban.com/top250  第一页
    https://movie.douban.com/top250?start=25&filter=  第二页 每页相差25
    https://movie.douban.com/top250?start=50&filter=  第三页
    https://movie.douban.com/top250?start=75&filter=  第四页
"""
base_url = 'https://movie.douban.com/top250?start=%s&filter='
for i in range(0, 250, 25):
  print(base_url % i)  # 打印出了豆瓣top250每页的url

# range()在python2.x和python3.x返回值不同
  在python2.x中range()会生成一个列表,但有个xrange()也是迭代器
  在python3.x中range()是一个迭代器,相对于python2.x生成列表更加节省内存
# python2.x中的xrange()就是python3.x中的range()
```

#### 2.3 break

```python
结束本层循环
eg:
for i in range(10):
    if i == 4:
        break  # 当i等于4的时候遇到了break,直接结束这层的for循环
    print(i)  # 结果是打印0-3
```

#### 2.4 continue

```python
结束本次循环
eg:
for i in range(10):
    if i == 4:
        continue  # 当i等于4的时候,结束本次循环,不影响
    print(i)  # 打印0-3,5-9
```

#### 2.5 else

```python
在for循环正常结束的情况下才会被执行
eg:
for i in rnage(10)：
    print(i)
else:
    print("这段被执行了")  # 在循环打印了0-9之后,会继续执行这段代码,打印 这段被执行了

eg:
for i in range(10):
    if i == 4:
        break  # 在循环打印了0-3之后,就被跳出了本层循环
    print(i)
else:
    print("这段代码没有被执行")  #这段代码就没有被执行

```

#### 2.6 嵌套

```python
# for i in range(3):
#     for j in range(5):
#         print("*", end='')
#     print()


for i in range(1, 10):
    for j in range(1, i + 1):
        print('%s*%s=%s' % (i, j, i * j), end=' ')
    print()
```

### 3. 数据类型内置方法

#### 3.1 整型

```python
1.类型转换 int()
# 只能转换成纯数字,且在转换的时候只识别整数,遇到其他类型的都会报错,如小数,带字母的等
eg:
  res = '123'
  print(type(res))  # 输出结果是 str
  int(res)
  print(type(res))  # 输出结果是 int

  int(123.123)  # 报错,不识别小数
  int(t123)  # 报错，不识别除数字以外的

 2.进制转换
   print(bin(100))  # 将十进制的100转换成二进制  0b1100100
   print(oct(100))  # 将十进制的100转换成八进制  0o144
   print(hex(100))  # 将十进制的100转换成十六进制  0x64
  # 0b开头为二进制数  0o开头为八进制数  0x开头为十六进制数
    print(int('0b1100100', 2))  # 使用int()将0b1100100以二进制的方式转换为十进制
    print(int('0o144', 8))  # 使用int()将0o144以八进制的方式转换为十进制
    print(int('0x64', 16))  # 使用int()将0x64以十六进制的方式转换为十进制
```

#### 3.2 浮点型

```python
类型转换 float()
可以转换成小数,在转换的时候可以识别整数和小数,遇到其他类型的都会报错
eg:
  res1 = '123.123'
  res2 = '123'
  print(type(res1))  # 没被转换前类型是 str
  print(type(res2))  # 没被转换前类型是 str
  float(res1)  # 使用float()进行转换
  float(res2)  # 使用float()进行转换
  print(type(res1))  # 转换后res1类型是整型
  print(type(res2))  # 转换后res2类型是整型
  print(res2)  # res2的结果变成 123.0
```

#### 3.3 字符串

##### 3.3.1 类型转换 str()

```python
任何类型都可以转换成字符串类型 str()
str(123)
str(123.123)
str(['name', 'paswd'])
str({'name':'xxx', 'passwd': 123})
...
#最终结果都会是str类型
```

##### 3.3.2 索引取值

```python
res = 'hello world!'
print(res[1])  # 结果为e

#还可以支持负数索引
print(res[-1])  # 结果为 ！
```

##### 3.3.3 切片操作

```python
res = 'hello world!'
print(res[1:4])  # 结果为ell
#切片操作顾头不顾尾,左闭右开
```

##### 3.3.4 步长操作

```python
res = 'hello world!'
print(res[1:10:2])  # 结果为el ol
#先取到1到10的字符为在ello worl,同样顾头不顾尾
#再每隔两个取,结果就是el ol

# print(res[-5:-1])  # orld  顾头不顾尾
# print(res[-5:-1:-1])  # 方向冲突
```

##### 3.3.5 len()

```python
统计字符串内部字符的个数 len()
res = 'hello world!'
print(len(res))  # 结果为12
```

##### 3.3.6 strip()

```python
移除字符串首尾指定的字符,默认移除的是首位的空格 strip()
eg1:
  name = '  jason  '
  print(name, len(name))
  print(len(name.strip()))  # 默认移除首尾的空格
eg2:
  name1 = '$$jason$$'
  print(name1.strip('$'))  # 移除指定字符$ 结果为jason
  print(name1.lstrip('$'))  # 移除左边的$$ 结果为jason$$
  print(name1.rstrip('$'))  # 移除右边的$$ 结果为$$jason

# 在应用中,用户在输入的时候在首位手残输入了空格之后的解决办法
  username = input("请输入用户名:").strip()  # 用户在输入的时候前后输入了空格将不受影响
  if username == 'root':
      print('用户名输入正确')
  else:
      print("用户名输入错误")
```

##### 3.3.7 split()

```python
按照指定的字符切割字符串,该方法返回的是一个列表 split()
res = 'root|123|test'
print(res.split('|'))  # ['root', '123', 'test'] 以|进行分割,返回列表
print(res.split('|', maxsplit=1))  # ['root', '123|test']  maxsplit用于控制切割的次数
print(res.rsplit('|', maxsplit=1))  # ['root|123', 'test'] 从右边开始分割
```
