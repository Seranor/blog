---
title: 描述符
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-08.html
toc: true
---

# 描述符

- 描述符是什么：描述符本质就是一个新式类，在这个新式类中，至少实现了

  get()，set()，delete()中的一个，这也被称为描述符协议

  - **get**()：调用一个属性时，触发
  - **set**()：为一个属性赋值时，触发
  - **delete**()：采用 del 删除属性时，触发

- 定义一个描述符
<!-- more -->

```python
class Foo:  # 在python3中Foo是新式类，它实现了__get__()，__set__()，__delete__()中的一个三种方法的一个，这个类就被称作一个描述符
    def __get__(self, instance, owner):
        pass

    def __set__(self, instance, value):
        pass

    def __delete__(self, instance):
        pass
```

# 描述符的作用

- 描述符是干什么的：描述符的作用是用来代理另外一个类的属性的，必须把描述符定义成这个类的类属性，不能定义到构造函数中

```python
class Foo:
    def __get__(self, instance, owner):
        print('触发get')

    def __set__(self, instance, value):
        print('触发set')

    def __delete__(self, instance):
        print('触发delete')


f1 = Foo()
```

- 包含这三个方法的新式类称为描述符，由这个类产生的实例进行属性的调用/赋值/删除，并不会触发这三个方法

```python
f1.name = 'lqz'
f1.name
del f1.name
```

## 何时，何地，会触发这三个方法的执行

```python
class Str:
    """描述符Str"""

    def __get__(self, instance, owner):
        print('Str调用')

    def __set__(self, instance, value):
        print('Str设置...')

    def __delete__(self, instance):
        print('Str删除...')


class Int:
    """描述符Int"""

    def __get__(self, instance, owner):
        print('Int调用')

    def __set__(self, instance, value):
        print('Int设置...')

    def __delete__(self, instance):
        print('Int删除...')


class People:
    name = Str()
    age = Int()

    def __init__(self, name, age):  # name被Str类代理，age被Int类代理
        self.name = name
        self.age = age


# 何地？：定义成另外一个类的类属性

# 何时？：且看下列演示

p1 = People('alex', 18)


```

Str 设置...
Int 设置...

- 描述符 Str 的使用

```python
p1.name  # Str调用

p1.name = 'lqz'  # Str设置...

del p1.name  # Str删除...
```

- 描述符 Int 的使用

```python
p1.age  # Int调用

p1.age = 18  # Int设置...

del p1.age  # Int删除...
```

- 我们来瞅瞅到底发生了什么

```python
print(p1.__dict__)  # {}
print(People.__dict__)
'''
{'__module__': '__main__', 'name': <__main__.Str object at 0x107a86940>, 'age': <__main__.Int object at 0x107a863c8>, '__init__': <function People.__init__ at 0x107ba2ae8>, '__dict__': <attribute '__dict__' of 'People' objects>, '__weakref__': <attribute '__weakref__' of 'People' objects>, '__doc__': None}
'''
```

- 补充

```python
print(type(p1) == People)  # True type(obj)其实是查看obj是由哪个类实例化来的

print(type(p1).__dict__ == People.__dict__)  # True
```

# 两种描述符

## 数据描述符

- 至少实现了**get**()和**set**()

```python
class Foo:
    def __set__(self, instance, value):
        print('set')

    def __get__(self, instance, owner):
        print('get')
```

## 非数据描述符

- 没有实现**set**()

```python
class Foo:
    def __get__(self, instance, owner):
        print('get')
```

# 描述符注意事项

![103-描述符(get,set,delete)-注意.jpg](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2qmdn5b5g309m07iwfh.gif)

1. 描述符本身应该定义成新式类，被代理的类也应该是新式类

2. 必须把描述符定义成这个类的类属性，不能为定义到构造函数中

3. 要严格遵循该优先级，优先级由高到底分别是

   - 1.类属性

   - 2.数据描述符

   - 3.实例属性

   - 4.非数据描述符

   - 5.找不到的属性触发**getattr**()

# 使用描述符

- 众所周知，python 是弱类型语言，即参数的赋值没有类型限制，下面我们通过描述符机制来实现类型限制功能

## 牛刀小试

```python
class Str:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        print('get--->', instance, owner)
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print('set--->', instance, value)
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        print('delete--->', instance)
        instance.__dict__.pop(self.name)


class People:
    name = Str('name')

    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary



p1 = People('lqz', 18, 3231.3)

# set---> <__main__.People object at 0x107a86198> lqz
```

- 调用

```python
print(p1.__dict__)  # {'name': 'lqz', 'age': 18, 'salary': 3231.3}
print(p1.name)  # get---> <__main__.People object at 0x107a86198> <class '__main__.People'>
lqz
```

- 赋值

```python
print(p1.__dict__)  # {'name': 'lqz', 'age': 18, 'salary': 3231.3}

p1.name = 'lqzlin'
print(p1.__dict__)
'''
set---> <__main__.People object at 0x107a86198> lqzlin
{'name': 'lqzlin', 'age': 18, 'salary': 3231.3}
'''
```

- 删除

```python
print(p1.__dict__)  # {'name': 'lqzlin', 'age': 18, 'salary': 3231.3}
del p1.name
print(p1.__dict__)
'''
delete---> <__main__.People object at 0x107a86198>
{'age': 18, 'salary': 3231.3}
'''
```

## 拔刀相助

```python
class Str:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        print('get--->', instance, owner)
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print('set--->', instance, value)
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        print('delete--->', instance)
        instance.__dict__.pop(self.name)


class People:
    name = Str('name')

    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


# 疑问：如果我用类名去操作属性呢
try:
    People.name  # 报错，错误的根源在于类去操作属性时，会把None传给instance
except Exception as e:
    print(e)

'''
get---> None <class '__main__.People'>
'NoneType' object has no attribute '__dict__'
'''
```

- 修订**get**方法

```
class Str:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        print('get--->', instance, owner)
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print('set--->', instance, value)
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        print('delete--->', instance)
        instance.__dict__.pop(self.name)


class People:
    name = Str('name')

    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


print(People.name)  # 完美，解决
get---> None <class '__main__.People'>
<__main__.Str object at 0x107a86da0>
```

## 磨刀霍霍

```python
class Str:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __get__(self, instance, owner):
        print('get--->', instance, owner)
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print('set--->', instance, value)
        if not isinstance(value, self.expected_type):  # 如果不是期望的类型，则抛出异常
            raise TypeError('Expected %s' % str(self.expected_type))
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        print('delete--->', instance)
        instance.__dict__.pop(self.name)


class People:
    name = Str('name', str)  # 新增类型限制str

    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


try:
    p1 = People(123, 18, 3333.3)  # 传入的name因不是字符串类型而抛出异常
except Exception as e:
    print(e)

'''
set---> <__main__.People object at 0x1084cd940> 123
Expected <class 'str'>
'''
```

## 大刀阔斧

```python
class Typed:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __get__(self, instance, owner):
        print('get--->', instance, owner)
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print('set--->', instance, value)
        if not isinstance(value, self.expected_type):
            raise TypeError('Expected %s' % str(self.expected_type))
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        print('delete--->', instance)
        instance.__dict__.pop(self.name)


class People:
    name = Typed('name', str)
    age = Typed('name', int)
    salary = Typed('name', float)

    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


try:
    p1 = People(123, 18, 3333.3)
except Exception as e:
    print(e)
'''
set---> <__main__.People object at 0x1082c7908> 123
Expected <class 'str'>
'''


try:
    p1 = People('lqz', '18', 3333.3)
except Exception as e:
    print(e)
'''
set---> <__main__.People object at 0x1078dd438> lqz
set---> <__main__.People object at 0x1078dd438> 18
Expected <class 'int'>
'''


p1 = People('lqz', 18, 3333.3)
'''
set---> <__main__.People object at 0x1081b3da0> lqz
set---> <__main__.People object at 0x1081b3da0> 18
set---> <__main__.People object at 0x1081b3da0> 3333.3
'''
```

- 大刀阔斧之后我们已然能实现功能了，但是问题是，如果我们的类有很多属性，你仍然采用在定义一堆类属性的方式去实现，low，这时候我需要教你一招：独孤九剑

![103-描述符(get,set,delete)-独孤九剑.jpg](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1gm2qmz3cm4j30ci09eglo.jpg)

### 类的装饰器:无参

```python
def decorate(cls):
    print('类的装饰器开始运行啦------>')
    return cls


@decorate  # 无参：People = decorate(People)
class People:
    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


p1 = People('lqz', 18, 3333.3)
类的装饰器开始运行啦------>
```

### 类的装饰器:有参

```python
def typeassert(**kwargs):
    def decorate(cls):
        print('类的装饰器开始运行啦------>', kwargs)
        return cls

    return decorate


@typeassert(
    name=str, age=int, salary=float
)  # 有参：1.运行typeassert(...)返回结果是decorate，此时参数都传给kwargs 2.People=decorate(People)
class People:
    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


p1 = People('lqz', 18, 3333.3)

'''
类的装饰器开始运行啦------> {'name': <class 'str'>, 'age': <class 'int'>, 'salary': <class 'float'>}
'''
```

## 刀光剑影

```python
class Typed:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __get__(self, instance, owner):
        print('get--->', instance, owner)
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print('set--->', instance, value)
        if not isinstance(value, self.expected_type):
            raise TypeError('Expected %s' % str(self.expected_type))
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        print('delete--->', instance)
        instance.__dict__.pop(self.name)


def typeassert(**kwargs):
    def decorate(cls):
        print('类的装饰器开始运行啦------>', kwargs)
        for name, expected_type in kwargs.items():
            setattr(cls, name, Typed(name, expected_type))
        return cls

    return decorate


@typeassert(
    name=str, age=int, salary=float
)  # 有参：1.运行typeassert(...)返回结果是decorate，此时参数都传给kwargs 2.People=decorate(People)
class People:
    def __init__(self, name, age, salary):
        self.name = name
        self.age = age
        self.salary = salary


print(People.__dict__)
p1 = People('lqz', 18, 3333.3)

'''
类的装饰器开始运行啦------> {'name': <class 'str'>, 'age': <class 'int'>, 'salary': <class 'float'>}
{'__module__': '__main__', '__init__': <function People.__init__ at 0x10797a400>, '__dict__': <attribute '__dict__' of 'People' objects>, '__weakref__': <attribute '__weakref__' of 'People' objects>, '__doc__': None, 'name': <__main__.Typed object at 0x1080b2a58>, 'age': <__main__.Typed object at 0x1080b2ef0>, 'salary': <__main__.Typed object at 0x1080b2c18>}
set---> <__main__.People object at 0x1080b22e8> lqz
set---> <__main__.People object at 0x1080b22e8> 18
set---> <__main__.People object at 0x1080b22e8> 3333.3
'''
```

# 描述符总结

- 描述符是可以实现大部分 python 类特性中的底层魔法，包括@classmethod，@staticmethd，@property 甚至是**slots**属性
- 描述父是很多高级库和框架的重要工具之一，描述符通常是使用到装饰器或者元类的大型框架中的一个组件.

![103-描述符(get,set,delete)-私人订制.jpg](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2qovssrxj30j60900sy.jpg)

# 自定制@property

- 利用描述符原理完成一个自定制@property，实现延迟计算（本质就是把一个函数属性利用装饰器原理做成一个描述符：类的属性字典中函数名为 key，value 为描述符类产生的对象）

## property 回顾

```python
class Room:
    def __init__(self, name, width, length):
        self.name = name
        self.width = width
        self.length = length

    @property
    def area(self):
        return self.width * self.length


r1 = Room('alex', 1, 1)

print(r1.area)  # 1
```

## 自定制 property

```python
class Lazyproperty:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        print('这是我们自己定制的静态属性，r1.area实际是要执行r1.area()')
        if instance is None:
            return self
        return self.func(instance)  # 此时你应该明白，到底是谁在为你做自动传递self的事情


class Room:
    def __init__(self, name, width, length):
        self.name = name
        self.width = width
        self.length = length

    @Lazyproperty  # area=Lazyproperty(area) 相当于定义了一个类属性,即描述符
    def area(self):
        return self.width * self.length


r1 = Room('alex', 1, 1)
print(r1.area)
'''
这是我们自己定制的静态属性，r1.area实际是要执行r1.area()
1
'''
```

## 实现延迟计算功能

```python
class Lazyproperty:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        print('这是我们自己定制的静态属性，r1.area实际是要执行r1.area()')
        if instance is None:
            return self
        else:
            print('--->')
            value = self.func(instance)
            setattr(instance, self.func.__name__, value)  # 计算一次就缓存到实例的属性字典中
            return value


class Room:
    def __init__(self, name, width, length):
        self.name = name
        self.width = width
        self.length = length

    @Lazyproperty  # area=Lazyproperty(area) 相当于'定义了一个类属性,即描述符'
    def area(self):
        return self.width * self.length


r1 = Room('alex', 1, 1)
print(r1.area)  # 先从自己的属性字典找,没有再去类的中找,然后出发了area的__get__方法
'''
这是我们自己定制的静态属性，r1.area实际是要执行r1.area()
--->
1
'''

print(r1.area)  # 1 先从自己的属性字典找,找到了,是上次计算的结果,这样就不用每执行一次都去计算

```

# 打破延迟计算

- 一个小的改动，延迟计算的美梦就破碎了

```python
class Lazyproperty:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        print('这是我们自己定制的静态属性，r1.area实际是要执行r1.area()')
        if instance is None:
            return self
        else:
            value = self.func(instance)
            instance.__dict__[self.func.__name__] = value
            return value
        # return self.func(instance) # 此时你应该明白,到底是谁在为你做自动传递self的事情
    def __set__(self, instance, value):
        print('hahahahahah')


class Room:
    def __init__(self, name, width, length):
        self.name = name
        self.width = width
        self.length = length

    @Lazyproperty  # area=Lazyproperty(area) 相当于定义了一个类属性,即描述符
    def area(self):
        return self.width * self.length



print(Room.__dict__)
'''
{'__module__': '__main__', '__init__': <function Room.__init__ at 0x107d53620>, 'area': <__main__.Lazyproperty object at 0x107ba3860>, '__dict__': <attribute '__dict__' of 'Room' objects>, '__weakref__': <attribute '__weakref__' of 'Room' objects>, '__doc__': None}
'''

r1 = Room('alex', 1, 1)
print(r1.area)
print(r1.area)
print(r1.area)
'''
这是我们自己定制的静态属性，r1.area实际是要执行r1.area()
1
这是我们自己定制的静态属性，r1.area实际是要执行r1.area()
1
这是我们自己定制的静态属性，r1.area实际是要执行r1.area()
1
'''

print(r1.area)  #缓存功能失效,每次都去找描述符了,为何,因为描述符实现了set方法,它由非数据描述符变成了数据描述符,数据描述符比实例属性有更高的优先级,因而所有的属性操作都去找描述符了
'''
这是我们自己定制的静态属性，r1.area实际是要执行r1.area()
1
'''
```

# 自定制@classmethod

```python
class ClassMethod:
    def __init__(self, func):
        self.func = func

    def __get__(
            self, instance,
            owner):  #类来调用,instance为None,owner为类本身,实例来调用,instance为实例,owner为类本身,
        def feedback():
            print('在这里可以加功能啊...')
            return self.func(owner)

        return feedback


class People:
    name = 'lqz'

    @ClassMethod  # say_hi=ClassMethod(say_hi)
    def say_hi(cls):
        print('你好啊,帅哥 %s' % cls.name)


People.say_hi()

p1 = People()
'''
在这里可以加功能啊...
你好啊,帅哥 lqz
'''


p1.say_hi()
'''
在这里可以加功能啊...
你好啊,帅哥 lqz
'''
```

- 疑问,类方法如果有参数呢,好说,好说

```python
class ClassMethod:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner
                ):  # 类来调用,instance为None,owner为类本身,实例来调用,instance为实例,owner为类本身,
        def feedback(*args, **kwargs):
            print('在这里可以加功能啊...')
            return self.func(owner, *args, **kwargs)

        return feedback


class People:
    name = 'lqz'

    @ClassMethod  # say_hi=ClassMethod(say_hi)
    def say_hi(cls, msg):
        print('你好啊,帅哥 %s %s' % (cls.name, msg))


People.say_hi('你是那偷心的贼')

p1 = People()
'''
在这里可以加功能啊...
你好啊,帅哥 lqz 你是那偷心的贼
'''

p1.say_hi('你是那偷心的贼')

'''
在这里可以加功能啊...
你好啊,帅哥 lqz 你是那偷心的贼
'''
```

# 自定制@staticmethod

```python
class StaticMethod:
    def __init__(self, func):
        self.func = func

    def __get__(
            self, instance,
            owner):  # 类来调用，instance为None，owner为类本身，实例来调用，instance为实例，owner为类本身
        def feedback(*args, **kwargs):
            print('在这里可以加功能啊...')
            return self.func(*args, **kwargs)

        return feedback


class People:
    @StaticMethod  # say_hi = StaticMethod(say_hi)
    def say_hi(x, y, z):
        print('------>', x, y, z)


People.say_hi(1, 2, 3)

p1 = People()
'''
在这里可以加功能啊...
------> 1 2 3
'''

p1.say_hi(4, 5, 6)
'''
在这里可以加功能啊...
------> 4 5 6
'''
```

![103-描述符(get,set,delete)-放假.jpg](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2qo716n3j3073073mx7.jpg)
