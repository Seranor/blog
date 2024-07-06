---
title: 类和数据类型、类的继承
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-02.html
toc: true
---

# 类和数据类型

## python3 中统一了类与类型的概念，类就是类型

<!-- more -->

```python
class Foo:
    pass


obj = Foo()
print(type(obj))  # <class '__main__.Foo'>

lis = [1, 2, 3]
lis2 = [4, 5, 6]
print(type(lis))  # <class 'list'>
```

- lis 和 lis2 都是实例化的对象，因此 lis 使用 append 方法和 lis2 无关

```python
lis.append(7)
print(lis)  # [1, 2, 3, 7]
print(lis2)  # [4, 5, 6]
```

## list.append()方法原理

```python
class OldboyStudent:
    school = 'oldboy'

    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.sex = gender

    def choose_course(self, name):
        print(f'{name} choosing course')


stu1 = OldboyStudent('nick', 18, 'male')
stu1.choose_course(1)  # 1 choosing course
OldboyStudent.choose_course(stu1, 1)  # 1 choosing course

lis = [1, 2, 3]  # lis = list([1,2,3])
print(type(lis))  # <class 'list'>

lis.append(4)  # list.append(lis,4)
print(lis)  # [1, 2, 3, 4]

list.append(lis, 5)
print(lis)  # [1, 2, 3, 4, 5]
```

# 对象的高度整合

## 没有对象

- 以未来我们要连接数据库举例，如果没有面向对象的思想，我们只要想要使用一个方法，就必须得这样做

```python
import pymysql  # 连接mysql的三方库，可以pip3 install pymysql安装


def exc1(host, port, db, charset, sql):
    conn = pymysql.connect(host, port, db, charset)
    conn.execute(sql)
    return xxx


def exc2(proc_name):
    conn = pymysql.connect(host, port, db, charsett)
    conn.call_proc(sql)
    return xxx


exc1('1.1.1.1', 3306, 'db1', 'utf-8', 'select * from t1')
exc1('1.1.1.1', 3306, 'db1', 'utf-8', 'select * from t2')
exc1('1.1.1.1', 3306, 'db1', 'utf-8', 'select * from t3')
exc1('1.1.1.1', 3306, 'db1', 'utf-8', 'select * from t4')
```

- 由于 host、port、db、charset 可能是固定不变的，sql 一直在变化，因此我们通过上述的方法实现不同的 sql 语句，非常麻烦，因此我们可以改用默认形参

```python
def exc1(sql, host='1.1.1.1', port=3306, db='db1', charset='utf-8'):
    conn = pymysql.connect(host, port, db, charset)
    conn.execute(sql)
    return xxx

exc1('select * from t1')
exc1('select * from t2')
exc1('select * from t3')
exc1('select * from t4')
```

- 虽然是用默认参数简化了操作，但是对于不同引用的对象，参数并不是一成不变的，或者我们需要对 exc2 方法进行修改，这是非常麻烦的，因此可以考虑使用面向对象

## 有对象

- 有了面向对象之后，对于上述的例子，我们可以这样做

```python
import pymysql


class Foo:
    def __init__(self, host, port, db, chartset):
        self.host = host
        self.port = port
        self.db = db
        self.charset = chartset

    def exc1(self, sql):
        conn = pymysql.connect(self.host, self.port, self.db, self.charset)
        conn.execute(sql)
        return xxx

    def exc2(self, proc_name):
        conn = pymysql.connect(self.host, self.port, self.db, self.charsett)
        conn.call_proc(sql)
        return xxx


obj1 = Foo('1.1.1.1', 3306, 'db1', 'utf-8')
obj1.exc1('select * from t1')
obj1.exc1('select * from t2')
obj1.exc1('select * from t3')
obj1.exc1('select * from t4')

obj2 = Foo('1.1.1.2', 3306, 'db1', 'utf-8')
obj2.exc1('select * from t4')
```

- 对于上述发生的现象，我们可以总结对象其实就是一个高度整合的产物，整合数据与专门操作该数据的方法（绑定方法）

# 面向对象基础小结

## 面向对象编程

面向过程编程：类似于工厂的流水线

- 优点：逻辑清晰
- 缺点：扩展性差

面向对象编程：核心是对象二字，对象属性和方法的集合体，面向对象编程就是一堆对象交互

- 优点：扩展性强
- 缺点：逻辑非常乱

## 类与对象

- 对象：属性和方法的集合体
- 类：一系列相同属性和方法的集合体

现实世界中先有对象后有类，python 中先有类，再实例化出对象

## 对象的属性的查找顺序

先对象本身-–>类–->父类-–>父类的父类-–>object–->自己定制的元类-–>type

## 给对象定制独有属性

```python
class People:
	pass

p1 = Peolple()
p1.name = 'nick'

p2 = People()
p2.name = 'tank'
```

## 对象的绑定方法

```python
class People:
	def eat(self):
		print(self, 'eat....')

p1 = Peolple()
p1.eat()
p1.name = 'nick'

p2 = People()
p2.eat()
p2.name = 'tank'
```

## 类与数据类型

```python
lis = [1,2,3]  # lis = list([1,2,3])

class foo:
	def __init__(self,name):
    	self.name = name

f = foo('name')

lis.append(4)  # 对象调对象绑定的方法,会自动传参
list.append(lis,4)  # 类调用对象绑定的方法,必须得传参
```

# 类的继承

#### 什么是继承

- 继承是一种新建类的方式，新建的类称为子类，被继承的类称为父类
- 继承的特性是：子类会遗传父类的属性
- 继承是类与类之间的关系

![89-类的继承-继承.jpg](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1glvts5mfutj30dv085gm8.jpg)

#### 为什么用继承

- 使用继承可以减少代码的冗余

## 对象的继承

- Python 中支持一个类同时继承多个父类

```python
class Parent1:
    pass


class Parent2:
    pass


class Sub1(Parent1, Parent2):
    pass
```

- 使用**bases**方法可以获取对象继承的类

```python
print(Sub1.__bases__)  # (<class '__main__.Parent1'>, <class '__main__.Parent2'>)
```

- 在 Python3 中如果一个类没有继承任何类，则默认继承 object 类
- 在 Python2 中如果一个类没有继承任何类，不会继承 object 类

```python
print(Parent1.__bases__)  # (<class 'object'>,)
```

## 类的分类

### 新式类

- 继承了 object 的类以及该类的子类，都是新式类
- Python3 中所有的类都是新式类

### 经典类

- 没有继承 object 的类以及该类的子类，都是经典类
- 只有 Python2 中才有经典类

## 继承与抽象

继承描述的是子类与父类之间的关系，是一种什么是什么的关系。要找出这种关系，必须先抽象再继承，抽象即抽取类似或者说比较像的部分。

抽象分成两个层次：

1. 将奥巴马和梅西这俩对象比较像的部分抽取成类；
2. 将人，猪，狗这三个类比较像的部分抽取成父类。

抽象最主要的作用是划分类别（可以隔离关注点，降低复杂度），如下图所示：

![89-类的继承-抽象图.png](https://tva1.sinaimg.cn/large/0081Kckwgy1glvtsjiddoj30lq0afgp4.jpg)

继承：基于抽象的结果，通过编程语言去实现它，肯定是先经历抽象这个过程，才能通过继承的方式去表达出抽象的结构。

抽象只是分析和设计的过程中，一个动作或者说一种技巧，通过抽象可以得到类，如下图所示：

![89-类的继承-继承图.png](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1glvtst5jygj30nu0bjn1o.jpg)

## 继承的应用

- 牢记对象是特征与功能的集合体，我们可以拿选课系统举例

```python
class OldboyPeople:
    """由于学生和老师都是人，因此人都有姓名、年龄、性别"""
    school = 'oldboy'

    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.gender = gender


class OldboyStudent(OldboyPeople):
    def choose_course(self):
        print('%s is choosing course' % self.name)


class OldboyTeacher(OldboyPeople):
    def score(self, stu_obj, num):
        print('%s is scoring' % self.name)
        stu_obj.score = num


stu1 = OldboyStudent('tank', 18, 'male')
tea1 = OldboyTeacher('lqz', 18, 'male')
```

- 对象查找属性的顺序：对象自己---》对象的类---》父类---》父类。。。

![89-类的继承-查找.jpg](https://tva1.sinaimg.cn/large/0081Kckwgy1glvttee3l6j308c094t8s.jpg)

```python
print(stu1.school)  # oldboy

print(tea1.school)  # oldboy

print(stu1.__dict__)  # {'name': 'tank', 'age': 18, 'gender': 'male'}

tea1.score(stu1, 99)  # lqz is scoring

print(stu1.__dict__)  # {'name': 'tank', 'age': 18, 'gender': 'male', 'score': 99}
```

### 属性查找练习

```python
class Foo:
    def f1(self):
        print('Foo.f1')

    def f2(self):
        print('Foo.f2')
        self.f1()


class Bar(Foo):
    def f1(self):
        print('Bar.f1')


# 对象查找属性的顺序：对象自己-》对象的类-》父类-》父类。。。
obj = Bar()  # self是obj本身，即找到Bar的f1()
obj.f2()
Foo.f2
Bar.f1
```
