---
title: 绑定方法及面向对象小结
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-06.html
toc: true
---

# 绑定方法

## 对象的绑定方法

在类中没有被任何装饰器修饰的方法就是 绑定到对象的方法，这类方法专门为对象定制。

<!-- more -->

```python
class Person:
    country = "China"

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def speak(self):
        print(self.name + ', ' + str(self.age))


p = Person('Kitty', 18)


print(p.__dict__)  # {'name': 'Kitty', 'age': 18}
print(Person.__dict__['speak'])  # <function Person.speak at 0x10f0dd268>
```

speak 即为绑定到对象的方法，这个方法不在对象的名称空间中，而是在类的名称空间中。

通过对象调用绑定到对象的方法，会有一个自动传值的过程，即自动将当前对象传递给方法的第一个参数（self，一般都叫 self，也可以写成别的名称）；若是使用类调用，则第一个参数需要手动传值。

```python
p = Person('Kitty', 18)
# 通过对象调用
p.speak()  # Kitty, 18


 # 通过类调用
Person.speak(p)  # Kitty, 18
```

## 类的绑定方法

类中使用 @classmethod 修饰的方法就是绑定到类的方法。这类方法专门为类定制。通过类名调用绑定到类的方法时，会将类本身当做参数传给类方法的第一个参数。

```python
class Operate_database():
    host = '192.168.0.5'
    port = '3306'
    user = 'abc'
    password = '123456'

    @classmethod
    def connect(cls):  # 约定俗成第一个参数名为cls，也可以定义为其他参数名
        print(cls)
        print(cls.host + ':' + cls.port + ' ' + cls.user + '/' + cls.password)


Operate_database.connect()
'''
<class '__main__.Operate_database'>
192.168.0.5:3306 abc/123456
'''
```

通过对象也可以调用，只是默认传递的第一个参数还是这个对象对应的类。

```python
Operate_database().connect()  # 输出结果一致
'''
<class '__main__.Operate_database'>
192.168.0.5:3306 abc/123456
'''
```

# 非绑定方法

在类内部使用 @staticmethod 修饰的方法即为非绑定方法，这类方法和普通定义的函数没有区别，不与类或对象绑定，谁都可以调用，且没有自动传值的效果。

```python
import hashlib


class Operate_database():
    def __init__(self, host, port, user, password):
        self.host = host
        self.port = port
        self.user = user
        self.password = password

    @staticmethod
    def get_passwrod(salt, password):
        m = hashlib.md5(salt.encode('utf-8'))  # 加盐处理
        m.update(password.encode('utf-8'))
        return m.hexdigest()


hash_password = Operate_database.get_passwrod('lala', '123456')  # 通过类来调用
print(hash_password)  # f7a1cc409ed6f51058c2b4a94a7e1956


p = Operate_database('192.168.0.5', '3306', 'abc', '123456')
hash_password = p.get_passwrod(p.user, p.password)  # 也可以通过对象调用
print(hash_password)  # 0659c7992e268962384eb17fafe88364
```

简而言之，非绑定方法就是将普通方法放到了类的内部。

# 练习

假设我们现在有一个需求，需要让 Mysql 实例化出的对象可以从文件 settings.py 中读取数据。

```python
# settings.py
IP = '1.1.1.10'
PORT = 3306
NET = 27

# test.py
import uuid

class Mysql:
    def __init__(self, ip, port, net):
        self.uid = self.create_uid()
        self.ip = ip
        self.port = port
        self.net = net

    def tell_info(self):
        """查看ip地址和端口号"""
        print('%s:%s' % (self.ip, self.port))

    @classmethod
    def from_conf(cls):
        return cls(IP, NET, PORT)

    @staticmethod
    def func(x, y):
        print('不与任何人绑定')

    @staticmethod
    def create_uid():
        """随机生成一个字符串"""
        return uuid.uuid1()


# 默认的实例化方式：类名()
obj = Mysql('10.10.0.9', 3307, 27)
obj.tell_info()  # 10.10.0.9:3307
```

## 绑定方法小结

如果函数体代码需要用外部传入的类，则应该将该函数定义成绑定给类的方法

如果函数体代码需要用外部传入的对象，则应该将该函数定义成绑定给对象的方法

```python
# 一种新的实例化方式：从配置文件中读取配置完成实例化
obj1 = Mysql.from_conf()
obj1.tell_info()  # 1.1.1.10:27

print(obj.tell_info)  # <bound method Mysql.tell_info of <__main__.Mysql object at 0x10f469240>>

print(obj.from_conf)  # <bound method Mysql.from_conf of <class '__main__.Mysql'>>
```

## 非绑定方法小结

如果函数体代码既不需要外部传入的类也不需要外部传入的对象，则应该将该函数定义成非绑定方法/普通函数

```python
obj.func(1, 2)
# 不与任何人绑定
Mysql.func(3, 4)
# 不与任何人绑定

print(obj.func)  # <function Mysql.func at 0x10f10e620>

print(Mysql.func)  # <function Mysql.func at 0x10f10e620>

print(obj.uid)  # a78489ec-92a3-11e9-b4d7-acde48001122
```

# 面向对象进阶小结

## 类的继承

继承父类,则会有父类的所有属性和方法

```python
class ParentClass1():
	pass

class ParentClass2():
	pass

class SubClass(ParentClass1,ParentClass2):
	pass
```

## 类的派生

继承父类的同时自己有 init,然后也需要父类的 init

```python
class ParentClass1():
	def __init__(self,name):
		pass


class SubClass(ParentClass):
	def __init__(self,age):
		# 1. ParentClass1.__init__(self,name)
		# 2. super(SubClass,self).__init__(name)
		self.age = age
```

## 类的组合

类对象可以引用/当做参数传入/当做返回值/当做容器元素,类似于函数对象

```python
class ParentClass1():
	count = 0
	def __init__(self,name):
		pass

class SubClass(ParentClass):
	def __init__(self,age):
		self.age = age

pc = ParentClass1()
sc = SubClass()

sc.parent_class = pc  # 组合
sc.parent_class.count  # 0
```

## 菱形继承问题

新式类:继承 object 的类,python3 中全是新式类

经典类:没有继承 object 的类,只有 python2 中有

在菱形继承的时候,新式类是广度优先(老祖宗最后找);经典类深度优先(一路找到底,再找旁边的)

## 多态与多态性

一种事物的多种形态,动物–>人/猪/狗

```python
# 多态
import abc

class Animal(metaclass=abc.ABCmeta):
	@abc.abstractmethod
	def eat():
		print('eat')

class People(Animal):
	def eat():
		pass

class Pig(Animal):
	def eat():
		pass
    def run():
        pass

class Dog(Animal):  # 报错
	def run():
		pass

# 多态性
peo = People()
peo.eat()
peo1 = People()
peo1.eat()
pig = Pig()
pig.eat()

def func(obj):
	obj.eat()

class Cat(Animal):
	def eat():
		pass
cat = Cat()

func(cat)
```

鸭子类型:只要长得像鸭子,叫的像鸭子,游泳像鸭子,就是鸭子

```python
# 继承一个父类，父类中有方法，在子类中重写方法
# 鸭子类型：不需要显示继承一个类，只要多个类中有同样的属性或方法，我们把它们称之为一种类，python，go
# 非鸭子类类型语言：如果要属于同一类，必须显示的继承某个基类，这样才属于基类这个类型，java

# python语言建议使用鸭子类型（约定），但在实际开发中，我们经常不使用鸭子类型这种特性，出错概率低
# 实际编码：要么认为约定必须有哪些方法(符合鸭子类型)，可控性低；要么强制约定有哪些方法(abc模块，使用抛异常)

# java中：重写：重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变
# java中：重载：是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同
```

## 类的封装

隐藏属性,只有类内部可以访问,类外部不可以访问

```python
class Foo():
	__count = 0

	def get_count(self):
		return self.__count

f = Foo()
f.__count  # 报错
f._Foo__count # 不能这样做
```

## 类的 property 特性

把方法变成属性引用

```python
class People():
	def __init__(self,height,weight):
		self.height = height
		self.weight = weight

	@property
	def bmi(self):
		return weight/(height**2)

	@bmi.setter
	def bmi(self,value)
		print('setter')

    @bmi.deleter
    def bmi(self):
        print('delter')

peo = People
peo.bmi
```

## 类与对象的绑定方法和非绑定方法

没有任何装饰器装饰的方法就是对象的绑定方法, 类能调用, 但是必须得传参给 self

被 @classmethod 装饰器装饰的方法是类的绑定方法,参数写成 cls, cls 是类本身, 对象也能调用, 参数 cls 还是类本身

被 @staticmethod 装饰器装饰的方法就是非绑定方法, 就是一个普通的函数

# isinstance 与 type

在游戏项目中，我们会在每个接口验证客户端传过来的参数类型，如果验证不通过，返回给客户端“参数错误”错误码。

这样做不但便于调试，而且增加健壮性。因为客户端是可以作弊的，不要轻易相信客户端传过来的参数。

验证类型用 type 函数，非常好用，比如

```python
print(type('foo') == str)  # True

print(type(2.3) in (int, float))  # True
```

既然有了 type()来判断类型，为什么还有 isinstance()呢？

一个明显的区别是在判断子类。

type()不会认为子类是一种父类类型；isinstance()会认为子类是一种父类类型。

千言不如一码。

```python
class Foo(object):
    pass

class Bar(Foo):
    pass

print(type(Foo()) == Foo)  # True

print(type(Bar()) == Foo)  # False

# isinstance参数为对象和类
print(isinstance(Bar(),Foo))  # True
```

需要注意的是，旧式类跟新式类的 type()结果是不一样的。旧式类都是<type ‘instance’>。

```python
# python2.+
class A:
    pass

class B:
    pass

class C(object):
    pass

print('old style class',type(A()))  # old style class <type 'instance'>

print('old style class',type(B()))  # old style class <type 'instance'>

print('new style class',type(C()))  # new style class <class '__main__.C'>

print(type(A()) == type(B()))  # True
```

注意：**不存在说 isinstance 比 type 更好。只有哪个更适合需求。**

# issubclass

```python
class Parent:
    pass

class Sub(Parent):
    pass


print(issubclass(Sub, Parent))  # True
print(issubclass(Parent, object))  # True
```
