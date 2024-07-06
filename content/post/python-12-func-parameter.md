---
title: python-函数参数(一)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-12.html
toc: true
---

### 1. 函数参数分类

#### 1.1 形式参数

在函数定义阶段括号内所填写的参数，简称形参

<!-- more -->

```python
def func(a, b):
    pass
# a和b就是函数func的形参
```

#### 1.2 实际参数

在函数调用阶段括号内传入的参数，简称实参

```python
func(1, 2)  # 数据1和2就是函数func的实参
```

#### 1.3 关系

```python
形参可以看成变量名,实参可以看成变量值
两者在函数调用阶段临时绑定,函数运行结束断开

形参的表现形式只有一种就是变量名
实参的表现形式有很多中(核心是数据值)
```

### 2. 位置参数

1. 位置参数

   按照从左往右的顺序依次填入的参数

2. 位置形参

   在函数定义阶段括号内按照从左往右的顺序依次填入的变量名

3. 位置实参

   在函数调用阶段括号内按照从左往右的顺序依次传入的数据值

4. 关键字实参

   可以打破位置顺序

   在函数调用阶段可以通过 `形参名=数据值` 的形式进行传值

5. 总结

   ```python
   1.位置形参与位置实参在函数调用阶段，按照位置一一对应
   2.位置参数在绑定的时候个数一致
   3.格式越简单的越靠前，格式越复杂的越靠后
   ```

### 3. 默认参数

```python
默认参数描述的是默认形参:
  1.函数在定义阶段就给形参赋值了
  2.该形参在调用阶段如果不给值，则使用默认值
  3.该形参在函数调用阶段也可以继续给值，则使用所给的值
  4.位置形参与默认形参在定义的时候，位置形参必须在默认形参的前面

  eg:
    def register(name, age, gender='male'):
        print('%s:%s:%s' % (name, age, gender))

    register('jason', 18)  # jason:18:male
    register('lili', 18, 'female')  # lili:18:female
```

### 4. 可变长参数

#### 4.1 形参

##### 4.1.1 位置参数

```python
函数无论传入多少位置参数都可以正常运行

eg:
  def func(a, b, *c):
      print(a, b, c)

  func(1, 2)  # 1 2 ()
  func(1, 2, 3)  # 1 2 (3,)
  func(1, 2, 3, 4)  # 1 2 (3, 4)

* 在形参数中的使用，用于接收多余位置的参数，并组织成元组的形式赋值给*号后面的变量名
```

##### 4.1.2 关键字参数

```python
函数无论传入多少关键字参数都可以正常运行

eg:
  def func(a, b, **c):
      print(a, b, c)


  func(a=1, b=2)  # 1 2 {}
  func(a=1, b=2, c=3)  # 1 2 {'c': 3}
  func(a=1, b=2, c=3, d=4)  # 1 2 {'c': 3, 'd': 4}

** 号在形参中的使用，用于接收多余的关键字参数，并组织成字典的形式赋值给给**号后面的变量名
```

##### 4.1.3 位置参数+关键字参数

```python
函数无论传入多少位置参数和关键字参数都可以正常运行

eg:
  def func(*a, **b):
      print(a, b)

  func()  # () {}
  func(1, 2)  # (1, 2) {}
  func(a=1, b=2  # () {'a': 1, 'b': 2}
  func(1, 2, a=1, b=2)  # (1, 2) {'a': 1, 'b': 2}

规定:
    可变长形参 *和**后面的变量名可以随便定义，但是python中推荐使用
       *args
       **kwargs
```

#### 4.2 实参

##### 4.2.1 `*`在实参中使用

```python
def func(a, b, c):
    print(a, b, c)


list1 = [11, 22, 33]
# func(list1)  # 报错，list1是一个整体给了a，后面的b和c参数没有传入
func(list1[0], list1[1], list1[2])  # 需要拆散一一传值

当形参是*args，列表里面的元素如何按照位置一一传值
eg:
  def func(*args):
      print(args)
  list1 = [11, 22, 33, 44]
  func(*list1)  # (11, 22, 33, 44)
*号在实参中的使用，会将列表、元组内的元素打散成位置参数的形式一一传值
```

##### 4.2.3 `**`在实参中使用

```python
def func(**kwargs):
    print(kwargs)
dict1 = {'name': 'xxx', 'pwd': 123}
func(**dict1)  # {'name': 'xxx', 'pwd': 123}
# name='xxx'  --> 'name': 'xxx'
# pwd=123     --> 'pwd': 123

**号在实参中的使用，会将字典内的键值对打散成关键字参数传入
```

### 5. 函数参数补充

```python
def func(name, age, *a, sex, height):
    print(name, age, a, sex, height)

# func('jason', 18, 'male', 183)  # 报错
func('lili', 18, sex='male', height=183)

当*号后面还有参数时，后面传入实参的时候必须以关键字参数的形式，该形式不常用
```

### 6. 名称空间

用于存放变量名与变量值绑定关系的地方

##### 6.1 分类

```python
1.内置名称空间
  python解释器定义好的，如 len()

2.全局名称空间
	在py文件中顶格编写的代码运行之后都会存入全局名称空间
	        	name = 'jason'  # name全局
            def func():  # func全局
                pass
            if 1:
                a = 123  # a全局
            for i in range(10):
                print(i)  # i全局
            while True:
                a = 123  # a全局

3.局部名称空间
  函数体代码运行之后产生的都是局部名称空间
```

##### 6.2 存活周期

```python
1.内置名称空间
  python解释器启动与关闭而创建和销毁
2.全局名称空间
  随着py文件的运行与结束而创建和销毁
3.局部名称空间
  随着函数体代码的执行与结束而创建和销毁
```

##### 6.3 查找顺序

```python
1.查找名字的时候，先确定自己当前在哪儿
  如果在局部
    局部 --> 全局 --> 内置
  如果在全局
    全局 --> 内置

2.局部名称空间的嵌套
x = 111
def f1():
    x = 222
    def f2():
        x = 333
        def f3():
            x = 444  # 如果没在这定义 x 会向上查找，依次类推
            print(x)
            # x = 444 # 当x定义在这时，会报错
        f3()
    f2()

# 调用f1() 执行所有函数
f1()  # 444
```

```python
def register():
    pass

def login():
    pass

func_dic = {'1': register, '2': login}
while True:
    print("""
    1.注册
    2.登录
    """)
    choice = input('please>>>:').strip()
    if choice in func_dic:
        func_name = func_dic.get(choice)
        func_name()
    else:
        print('功能编号不存在')
```
