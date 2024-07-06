---
title: python-函数使用(二)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-13.html
toc: true
---

## 1. 名称空间作用域

1. 作用域

   ```python
   名称空间所能够作用的范围
   ```

2. 内置名称空间

   ```python
   程序任何阶段位置均可使用(全局有效)
   ```

   <!-- more -->

3. 全局名称空间

   ```python
   程序任何阶段位置均可使用(全局有效)
   ```

4. 局部名称空间

   ```python
   一般情况下只在各自的局部名称空间中有效
   ```

## 2. global 与 nonlocal

### 2.1 global

```python
eg1:
x = 111
def index():
    global x  # 局部修改全局变量
    x = 222
index()  # 调用函数之后,因为有了global，x可以被修改
print(x)  # 结果为222

eg2:
name_list = ['jason', 'xxx']
def index(a):
    name_list.append(a)
index('tony')
print(name_list)  # ['jason', 'xxx', 'tony']

结论:
  如果想在局部修改全局数据
    数据为不可变类型则需要关键字global声明
    数据为可变类型则不需要关键字global声明
```

### 2.2 nolocal

```python
eg1:
def index():
    x = 111
    def func():
        nonlocal x  # 内部的局部修改外部的局部数据使用 nonlocal
        x = 222
    func()  # 此时已经将 x 重新赋值为222
    print(x)  # 打印x
index()  # 调用index函数,结果为222


eg2:
def index():
    l1 = ['jason', 18]
    def func():
        l1.append('male')
    func()
    print(l1)  # ['jason', 18, 'male']
```

## 3. 函数使用

### 3.1 函数对象(函数名)

> **函数名遇到括号就会被调用**

1. 用法一：函数名可以当做变量名赋值

   ```python
   def index():
       print("from index")
   a = index
   print(a)  # <function index at 0x7fcb278c5ea0> 相当于函数的内存地址
   a()  # 相当于调用index() 结果为 from index
   ```

2. 用法二：函数名可以当做函数的参数

   ```python
   def index():
       print("from index")
   def func(a):
       print("from func")
       print(a)  # 函数的内存地址
       a()
   func(index)
   # from func
   # from index
   ```

3. 用法三：函数名可以当做函数的返回值

   ```python
   def index():
       print("from index")
   def func():
       print("from func")
       return index
   res = func()  # 先得到 from func
   print(res)  # 得到func函数的返回值，是index函数的内存地址
   res()  # 得到 from index
   ```

4. 用法四：函数名可以当做容器类型(内部可以存放多个数据)的元素

   ```python
   eg1:
   def index():
       print("from index")
   def func():
       print("from func")
   l1 = [index, func]
   l1[0]()  # 本质就是index() 即调用函数index
   l1[1]()  # 本质就是func()  即调用函数func
   print(l1)  # 得到函数index和func的内存地址

   eg2:
   def func1():
       print("功能1")
   def func2():
       print("功能2")
   choice_dict = {'1': func1,
                  '2': func2}
   while True:
       print('''
           1.功能1
           2.功能2
           3.退出
       ''')
       choice = input("请输入选项>>>: ").strip()
       if choice == '3':
           print("退出")
           break
       else:
           # 比if-elif-else格式精简
           if choice in choice_dict:
               func_name = choice_dict.get(choice)
               func_name()
           else:
               print("选项不存在")
   ```

### 3.2 函数嵌套调用

- 函数内部调用其他函数

  ```python
  eg1:
  def index():
      print('from index')
  def func():
      index()
      print('from func')
  func()

  eg2:
  def two_max(a, b):
      if a > b:
          return a
      else:
          return b
  def four_max(x, y, m, n):
      res1 = two_max(x, y)
      res2 = two_max(res1, m)
      res3 = two_max(res2, n)
      return res3
  max = four_max(11, 23, 9, 59)
  print(max)
  ```

### 3.3 函数嵌套定义

- 函数体内部定义其他函数

  ```python
  # 将复杂的功能全部隐藏起来，暴露一个简单的接口
  def all_func(type):
      def func1():
          print('功能一')
      def func2():
          print('功能二')
      def func3():
          print('功能三')
      all_dict = {'1': func1,
                  '2': func2,
                  '3': func3}
      if type in all_dict:
          func_name = all_dict.get(type)
          func_name()
      else:
          print("功能不存在")

  all_func('2')  # 功能二
  ```

### 3.4 闭包函数

1. 闭包函数定义

   ```python
   闭:定义在函数内部的函数
   包:内部函数使用了外部函数名称空间中的名字

   eg:
     def outer():
         x = 111
         def func():
             print('form func', x)
         return func
     print(outer())
     a = outer()
     a()
   ```

2. 函数传参的两种方式

   1. 方式一:

      ```python
      # 函数体代码需要用到的数据直接在括号内定义形参即可
      def index(username):
          print(username)
      index('jason')
      ```

   2. 方式二:

      ```python
      # 闭包函数
      eg1:
        def outer(x, y):
            def func():
                if x > y:
                    return x
                else:
                    return y
            return func
        res = outer(23, 5)
        print(res())  # 23
        print(res())  # 23

      eg2:
        import requests
        def outer(url):
            def get_content():
                res = requests.get(url)
                if res.status_code == 200:
                    with open(r'xxx.html', 'wb') as f:
                        f.write(res.content)
            return get_content

        res = outer('https://jd.com')  # 需要爬哪个网站直接替换实参，比定义全局url更灵活
        res()
      ```
