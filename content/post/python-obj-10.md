---
title: 迭代器、with、元类
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-10.html
toc: true
---

# 实现迭代器

## 简单示例

- 死循环

```python
class Foo:
    def __init__(self, x):
        self.x = x

    def __iter__(self):
        return self

    def __next__(self):
        self.x += 1
        return self.x


f = Foo(3)
for i in f:
    print(i)
```

<!-- more -->

## StopIteration 异常版

- 加上 StopIteration 异常

```python
class Foo:
    def __init__(self, start, stop):
        self.num = start
        self.stop = stop

    def __iter__(self):
        return self

    def __next__(self):
        if self.num >= self.stop:
            raise StopIteration
        n = self.num
        self.num += 1
        return n


f = Foo(1, 5)
from collections import Iterable, Iterator
print(isinstance(f, Iterator))
True
for i in Foo(1, 5):
    print(i)

'''
1
2
3
4
'''
```

## 模拟 range

```python
class Range:
    def __init__(self, n, stop, step):
        self.n = n
        self.stop = stop
        self.step = step

    def __next__(self):
        if self.n >= self.stop:
            raise StopIteration
        x = self.n
        self.n += self.step
        return x

    def __iter__(self):
        return self


for i in Range(1, 7, 3):
    print(i)

'''
1
4
'''
```

## 斐波那契数列

```python
class Fib:
    def __init__(self):
        self._a = 0
        self._b = 1

    def __iter__(self):
        return self

    def __next__(self):
        self._a, self._b = self._b, self._a + self._b
        return self._a


f1 = Fib()
for i in f1:
    if i > 100:
        break
    print('%s ' % i, end='')  # 1 1 2 3 5 8 13 21 34 55 89
```

# 实现文件上下文管理

- 我们知道在操作文件对象的时候可以这么写

```python
with open('a.txt') as f:
    '代码块'
```

- 上述叫做上下文管理协议，即 with 语句，为了让一个对象兼容 with 语句，必须在这个对象的类中声明**enter**和**exit**方法

## 上下文管理协议

```python
class Open:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        print('出现with语句，对象的__enter__被触发，有返回值则赋值给as声明的变量')
        # return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        print('with中代码块执行完毕时执行我啊')


with Open('a.txt') as f:
    print('=====>执行代码块')
    # print(f,f.name)
'''
出现with语句,对象的__enter__被触发,有返回值则赋值给as声明的变量
=====>执行代码块
with中代码块执行完毕时执行我啊
'''
```

- **exit**()中的三个参数分别代表异常类型，异常值和追溯信息,with 语句中代码块出现异常，则 with 后的代码都无法执行

```python
class Open:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        print('出现with语句，对象的__enter__被触发，有返回值则赋值给as声明的变量')

    def __exit__(self, exc_type, exc_val, exc_tb):
        print('with中代码块执行完毕时执行我啊')
        print(exc_type)
        print(exc_val)
        print(exc_tb)


try:
    with Open('a.txt') as f:
        print('=====>执行代码块')
        raise AttributeError('***着火啦，救火啊***')
except Exception as e:
    print(e)

'''
出现with语句，对象的__enter__被触发，有返回值则赋值给as声明的变量
=====>执行代码块
with中代码块执行完毕时执行我啊
<class 'AttributeError'>
***着火啦，救火啊***
<traceback object at 0x1065f1f88>
***着火啦，救火啊***
'''
```

- 如果\_\_exit()返回值为 True,那么异常会被清空，就好像啥都没发生一样，with 后的语句正常执行

```python
class Open:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        print('出现with语句，对象的__enter__被触发，有返回值则赋值给as声明的变量')

    def __exit__(self, exc_type, exc_val, exc_tb):
        print('with中代码块执行完毕时执行我啊')
        print(exc_type)
        print(exc_val)
        print(exc_tb)
        return True


with Open('a.txt') as f:
    print('=====>执行代码块')
    raise AttributeError('***着火啦，救火啊***')
print('0' * 100)  #------------->会执行

'''
出现with语句，对象的__enter__被触发，有返回值则赋值给as声明的变量
=====>执行代码块
with中代码块执行完毕时执行我啊
<class 'AttributeError'>
***着火啦，救火啊***
<traceback object at 0x1062ab048>
0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
'''
```

## 模拟 open

```python
class Open:
    def __init__(self, filepath, mode='r', encoding='utf-8'):
        self.filepath = filepath
        self.mode = mode
        self.encoding = encoding

    def __enter__(self):
        # print('enter')
        self.f = open(self.filepath, mode=self.mode, encoding=self.encoding)
        return self.f

    def __exit__(self, exc_type, exc_val, exc_tb):
        # print('exit')
        self.f.close()
        return True

    def __getattr__(self, item):
        return getattr(self.f, item)


with Open('a.txt', 'w') as f:
    print(f)
    f.write('aaaaaa')
    f.wasdf  #抛出异常，交给__exit__处理

'''
<_io.TextIOWrapper name='a.txt' mode='w' encoding='utf-8'>
'''
```

## 优点

1. 使用 with 语句的目的就是把代码块放入 with 中执行，with 结束后，自动完成清理工作，无须手动干预
2. 在需要管理一些资源比如文件，网络连接和锁的编程环境中，可以在**exit**中定制自动释放资源的机制，你无须再去关系这个问题，这将大有用处

# 什么是元类

- 在 python 中一切皆对象，那么我们用 class 关键字定义的类本身也是一个对象，负责产生该对象的类称之为元类，即元类可以简称为类的类

```
class Foo:  # Foo=元类()
    pass
```

![114-元类metaclass-类的创建.png](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1gm2qxadyxlj30i70463ye.jpg)

# 为什么用元类

- 元类是负责产生类的，所以我们学习元类或者自定义元类的目的：是为了控制类的产生过程，还可以控制对象的产生过程

# 内置函数 exec(储备)

```python
cmd = """
x=1
print('exec函数运行了')
def func(self):
    pass
"""
class_dic = {}
# 执行cmd中的代码，然后把产生的名字丢入class_dic字典中

exec(cmd, {}, class_dic)  # exec函数运行了

print(class_dic)  # {'x': 1, 'func': <function func at 0x10a0bc048>}
```

# class 创建类

- 如果说类也是对象，那么用 class 关键字的去创建类的过程也是一个实例化的过程，该实例化的目的是为了得到一个类，调用的是元类
- 用 class 关键字创建一个类，用的默认的元类 type，因此以前说不要用 type 作为类别判断

```python
class People:  # People=type(...)
    country = 'China'

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print('%s is eating' % self.name)


print(type(People))  # <class 'type'>
```

![114-元类metaclass-class关键字.png](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2qxvspp3j30vu0epdgx.jpg)

## 5.1 type 实现

- 创建类的 3 个要素：类名，基类，类的名称空间
- People = type(类名，基类，类的名称空间)

```python
class_name = 'People'  # 类名

class_bases = (object, )  # 基类

# 类的名称空间
class_dic = {}
class_body = """
country='China'
def __init__(self,name,age):
    self.name=name
    self.age=age
def eat(self):
    print('%s is eating' %self.name)
"""

exec(
    class_body,
    {},
    class_dic,
)


print(class_name)  # People

print(class_bases)  # (<class 'object'>,)

print(class_dic)  # 类的名称空间
'''
{'country': 'China', '__init__': <function __init__ at 0x10a0bc048>, 'eat': <function eat at 0x10a0bcd08>}
'''
```

- People = type(类名，基类，类的名称空间)

```python
People1 = type(class_name, class_bases, class_dic)
print(People1)  # <class '__main__.People'>

obj1 = People1(1, 2)
obj1.eat()  # 1 is eating
```

- class 创建的类的调用

```python
print(People)  # <class '__main__.People'>

obj = People1(1, 2)
obj.eat()  # 1 is eating
```

# 自定义元类控制类的创建

- 使用自定义的元类

```
class Mymeta(type):  # 只有继承了type类才能称之为一个元类，否则就是一个普通的自定义类
    def __init__(self, class_name, class_bases, class_dic):
        print('self:', self)  # 现在是People
        print('class_name:', class_name)
        print('class_bases:', class_bases)
        print('class_dic:', class_dic)
        super(Mymeta, self).__init__(class_name, class_bases,
                                     class_dic)  # 重用父类type的功能
```

- 分析用 class 自定义类的运行原理（而非元类的的运行原理）：
  1. 拿到一个字符串格式的类名 class_name=’People’
  2. 拿到一个类的基类们 class_bases=(obejct,)
  3. 执行类体代码，拿到一个类的名称空间 class_dic={…}
  4. 调用 People=type(class_name,class_bases,class_dic)

```python
class People(object, metaclass=Mymeta):  # People=Mymeta(类名,基类们,类的名称空间)
    country = 'China'

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print('%s is eating' % self.name)


'''
self: <class '__main__.People'>
class_name: People
class_bases: (<class 'object'>,)
class_dic: {'__module__': '__main__', '__qualname__': 'People', 'country': 'China', '__init__': <function People.__init__ at 0x10a0bcbf8>, 'eat': <function People.eat at 0x10a0bc2f0>}
'''
```

## 应用

- 自定义元类控制类的产生过程，类的产生过程其实就是元类的调用过程
- 我们可以控制类必须有文档，可以使用如下的方式实现

```python
class Mymeta(type):  # 只有继承了type类才能称之为一个元类，否则就是一个普通的自定义类
    def __init__(self, class_name, class_bases, class_dic):
        if class_dic.get('__doc__') is None or len(
                class_dic.get('__doc__').strip()) == 0:
            raise TypeError('类中必须有文档注释，并且文档注释不能为空')
        if not class_name.istitle():
            raise TypeError('类名首字母必须大写')
        super(Mymeta, self).__init__(class_name, class_bases,
                                     class_dic)  # 重用父类的功能


try:

    class People(object, metaclass=Mymeta
                 ):  # People  = Mymeta('People',(object,),{....})
        # """这是People类"""
        country = 'China'

        def __init__(self, name, age):
            self.name = name
            self.age = age

        def eat(self):
            print('%s is eating' % self.name)
except Exception as e:
    print(e)

"""类中必须有文档注释，并且文档注释不能为空"""
```

# **call**(储备)

- 要想让 obj 这个对象变成一个可调用的对象，需要在该对象的类中定义一个方法、、**call**方法，该方法会在调用对象时自动触发

```python
class Foo:
    def __call__(self, *args, **kwargs):
        print(args)
        print(kwargs)
        print('__call__实现了，实例化对象可以加括号调用了')


obj = Foo()
obj('lqz', age=18)
('lqz',)
{'age': 18}
__call__实现了，实例化对象可以加括号调用了
```

# **new**(储备)

我们之前说类实例化第一个调用的是**init**，但**init**其实不是实例化一个类的时候第一个被调用 的方法。当使用 Persion(name, age) 这样的表达式来实例化一个类时，最先被调用的方法 其实是 **new** 方法。

**new**方法接受的参数虽然也是和**init**一样，但**init**是在类实例创建之后调用，而 **new**方法正是创建这个类实例的方法。

注意：**\*\*new\*\*() 函数只能用于从 object 继承的新式类。**

```python
class A:
    pass


class B(A):
    def __new__(cls):
        print("__new__方法被执行")
        return cls.__new__(cls)

    def __init__(self):
        print("__init__方法被执行")


b = B()
```

# 自定义元类控制类的实例化

```python
class Mymeta(type):
    def __call__(self, *args, **kwargs):
        print(self)  # self是People
        print(args)  # args = ('lqz',)
        print(kwargs)  # kwargs = {'age':18}
        # return 123
        # 1. 先造出一个People的空对象，申请内存空间
        # __new__方法接受的参数虽然也是和__init__一样，但__init__是在类实例创建之后调用，而 __new__方法正是创建这个类实例的方法。
        obj = self.__new__(self)  # 虽然和下面同样是People，但是People没有，找到的__new__是父类的
        # 2. 为该对空对象初始化独有的属性
        self.__init__(obj, *args, **kwargs)
        # 3. 返回一个初始化好的对象
        return obj
```

- People = Mymeta()，People()则会触发**call**

```python
class People(object, metaclass=Mymeta):
    country = 'China'

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print('%s is eating' % self.name)


#     在调用Mymeta的__call__的时候，首先会找自己（如下函数）的，自己的没有才会找父类的
#     def __new__(cls, *args, **kwargs):
#         # print(cls)  # cls是People
#         # cls.__new__(cls) # 错误，无限死循环，自己找自己的，会无限递归
#         obj = super(People, cls).__new__(cls)  # 使用父类的，则是去父类中找__new__
#         return obj
```

- 类的调用，即类实例化就是元类的调用过程，可以通过元类 Mymeta 的**call**方法控制
- 分析：调用 Pepole 的目的
  1. 先造出一个 People 的空对象
  2. 为该对空对象初始化独有的属性
  3. 返回一个初始化好的对象

```python
obj = People('lqz', age=18)
'''
<class '__main__.People'>
('lqz',)
{'age': 18}
'''

print(obj.__dict__)  # {'name': 'lqz', 'age': 18}
```

# 自定义元类后类的继承顺序

结合 python 继承的实现原理+元类重新看属性的查找应该是什么样子呢？？？

在学习完元类后，其实我们用 class 自定义的类也全都是对象（包括 object 类本身也是元类 type 的 一个实例，可以用 type(object)查看），我们学习过继承的实现原理，如果把类当成对象去看，将下述继承应该说成是：对象 OldboyTeacher 继承对象 Foo，对象 Foo 继承对象 Bar，对象 Bar 继承对象 object

```python
class Mymeta(type):  # 只有继承了type类才能称之为一个元类，否则就是一个普通的自定义类
    n = 444

    def __call__(self, *args,
                 **kwargs):  #self=<class '__main__.OldboyTeacher'>
        obj = self.__new__(self)
        self.__init__(obj, *args, **kwargs)
        return obj


class Bar(object):
    n = 333


class Foo(Bar):
    n = 222


class OldboyTeacher(Foo, metaclass=Mymeta):
    n = 111

    school = 'oldboy'

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def say(self):
        print('%s says welcome to the oldboy to learn Python' % self.name)


print(OldboyTeacher.n)  # 111 # 自下而上依次注释各个类中的n=xxx，然后重新运行程序，发现n的查找顺序为OldboyTeacher->Foo->Bar->object->Mymeta->type


print(OldboyTeacher.n)  # 111
```

- 查找顺序：
  1. 先对象层：OldoyTeacher->Foo->Bar->object
  2. 然后元类层：Mymeta->type

依据上述总结，我们来分析下元类 Mymeta 中**call**里的 self.**new**的查找

```python
class Mymeta(type):
    n = 444

    def __call__(self, *args,
                 **kwargs):  #self=<class '__main__.OldboyTeacher'>
        obj = self.__new__(self)
        print(self.__new__ is object.__new__)  #True


class Bar(object):
    n = 333

    # def __new__(cls, *args, **kwargs):
    #     print('Bar.__new__')


class Foo(Bar):
    n = 222

    # def __new__(cls, *args, **kwargs):
    #     print('Foo.__new__')


class OldboyTeacher(Foo, metaclass=Mymeta):
    n = 111

    school = 'oldboy'

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def say(self):
        print('%s says welcome to the oldboy to learn Python' % self.name)

    # def __new__(cls, *args, **kwargs):
    #     print('OldboyTeacher.__new__')


OldboyTeacher('lqz',
              18)  # 触发OldboyTeacher的类中的__call__方法的执行，进而执行self.__new__开始查找
```

总结，Mymeta 下的**call**里的 self.**new**在 OldboyTeacher、Foo、Bar 里都没有找到**new**的情况下，会去找 object 里的**new**，而 object 下默认就有一个**new**，所以即便是之前的类均未实现**new**,也一定会在 object 中找到一个，根本不会、也根本没必要再去找元类 Mymeta->type 中查找**new**

# 练习

需求：使用元类修改属性为隐藏属性

```python
class Mymeta(type):
    def __init__(self, class_name, class_bases, class_dic):
        # 加上逻辑，控制类Foo的创建
        super(Mymeta, self).__init__(class_name, class_bases, class_dic)

    def __call__(self, *args, **kwargs):
        # 加上逻辑，控制Foo的调用过程，即Foo对象的产生过程
        obj = self.__new__(self)
        self.__init__(obj, *args, **kwargs)
        # 修改属性为隐藏属性
        obj.__dict__ = {
            '_%s__%s' % (self.__name__, k): v
            for k, v in obj.__dict__.items()
        }

        return obj


class Foo(object, metaclass=Mymeta):  # Foo = Mymeta(...)
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex


obj = Foo('lqz', 18, 'male')


print(obj.__dict__)  # {'_Foo__name': 'egon', '_Foo__age': 18, '_Foo__sex': 'male'}
```
