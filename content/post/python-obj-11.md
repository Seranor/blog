---
title: 面向对象高级小结
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-11.html
toc: true
---

# 面向对象高级小结

## isinstance,issubclass

isinstance 判断是否为类的实例化对象,会检测父类,而 type 不会检测父类

issubclass,判断是否为其子类

<!-- more -->

## 反射

1. hasattr:通过字符串判断是否类属性存在
2. getattr:通过字符串获取类属性
3. setattr:通过字符串修改类属性
4. delattr:通过字符串删除类属性

## call

```python
class Foo:
    def __init__(self):
        print('Foo()会触发我')
    def __call__(self):
        print('Foo()()/f()会触发我')

f = Foo()
f()
```

## new

```python
class Foo:
    def __new__(self):
        print('new')
        obj = object.__new__(self)
        return obj

    def __init__(self):
        print('init')

f = Foo()
```

## 元类

元类用来造类的

元类()–>类–>init

元类()()–>对象—>call

类分为几部分:类名/类体名称空间/父类们

```python
class Mymeta(type):
    def __init__(self,class_name,class_bases,class_dic):
        # 控制类的逻辑代码
        super().__init__(class_name,class_bases,class_dic)

    def __call__(self,*args,**kwargs):
        # 控制类实例化的参数

        obj = self.__new__(self)  # obj就是实例化的对象
        self.__init__(obj,*args,**kwargs)
        print(obj.__dict__)

        # 控制类实例化的逻辑

        return obj

class People(metaclass=Mymeta):
    def __init__(self,name,age):
        self.name = name
        self.age = age
```

# 单例模式

### 利用类的绑定方法的特性

```python
NAME = 'lqz'
AGE = 18

class People():

    __instance = None

    @classmethod
    def from_conf(cls):
        if cls.__instance:
            return cls.__instance

        cls.__instance = cls(NAME,AGE)
        return cls.__instance
```

People.from_conf()

People.from_conf()

### 利用装饰器

```python
NAME = 'lqz'
AGE = 18

def deco(cls):
    cls.__instance = cls(NAME,AGE)

    def wrapper(*args,**kwargs):
        if len(args) == 0 and len(kwargs) == 0:
            return cls.__instance

        res = cls(*args,**kwargs)
        return res

    return wrapper

@deco
class People():
    def __init__(self,name,age):
        self.name = name
        self.age = age
```

peo1 = People()

peo2 = People()

### 利用元类(正宗的)

```python
NAME = 'lqz'
AGE = 18

class Mymeta(type):
    def __init__(self,class_name,class_bases,class_dict):
        super().__init__(class_name,class_bases,class_dict)
        self.__instance = self(NAME,AGE)

    def __call__(self,*args,**kwargs):

        if len(args) == 0 and len(kwargs) == 0:
            return self.__instance

        obj = object.__new__(self)
        self.__init__(obj,*args,**kwargs)

        return obj

class People(metaclass=Mymeta):
    def __init__(self,name,age):
        self.name = name
        self.age = age

peo1 = People()
peo2 = People()
```

# 实战之单例模式

**单例模式（Singleton Pattern）**是一种常用的软件设计模式，该模式的主要目的是确保**某一个类只有一个实例存在**。当你希望在整个系统中，某个类只能出现一个实例时，单例对象就能派上用场。

比如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个 AppConfig 的类来读取配置文件的信息。如果在程序运行期间，有很多地方都需要使用配置文件的内容，也就是说，很多地方都需要创建 AppConfig 对象的实例，这就导致系统中存在多个 AppConfig 的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似 AppConfig 这样的类，我们希望在程序运行期间只存在一个实例对象。

## 实现单例模式的几种方式

### 使用模块

其实，**Python 的模块就是天然的单例模式**，因为模块在第一次导入时，会生成 `.pyc` 文件，当第二次导入时，就会直接加载 `.pyc` 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做：

**mysingleton.py**

```python
class Singleton(object):
    def foo(self):
        pass
singleton = Singleton()
```

将上面的代码保存在文件 `mysingleton.py` 中，要使用时，直接在其他文件中导入此文件中的对象，这个对象即是单例模式的对象

```
from a import singleton
```

### 使用装饰器

```python
def Singleton(cls):
    instance = None

    def _singleton(*args, **kargs):
        nonlocal instance
        if not instance:
            instance = cls(*args, **kargs)
        return instance

    return _singleton


@Singleton
class A(object):
    def __init__(self, x=0):
        self.x = x


a1 = A(2)
a2 = A(3)
print(a1.x)
print(a2.x)

print(a1 is a2)
```

### 使用类方法

```python
class Singleton(object):
    _instance=None
    def __init__(self):
        pass
    @classmethod
    def instance(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance=cls(*args, **kwargs)
        return cls._instance

a1=Singleton.instance()
a2=Singleton().instance()

print(a1 is a2)
```

### 基于**new**方法实现

```python
class Singleton(object):
    _instance=None
    def __init__(self):
        pass


    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = object.__new__(cls)
        return cls._instance

obj1 = Singleton()
obj2 = Singleton()
print(obj1 is obj2)
```

### 基于 metaclass 方式实现

```python
class SingletonType(type):
    _instance=None
    def __call__(cls, *args, **kwargs):
        if not cls._instance:
            # cls._instance = super().__call__(*args, **kwargs)
            cls._instance = object.__new__(cls)
            cls._instance.__init__(*args, **kwargs)
        return cls._instance

class Foo(metaclass=SingletonType):
    def __init__(self,name):
        self.name = name


obj1 = Foo('name')
obj2 = Foo('name')
print(obj1.name)
print(obj1 is obj2)
```
