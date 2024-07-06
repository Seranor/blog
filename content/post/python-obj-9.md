---
title: 魔术方法(二)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-09.html
toc: true
---

```
class Foo:
    def __init__(self, name):
        self.name = name

    def __getitem__(self, item):
        print('getitem执行', self.__dict__[item])

    def __setitem__(self, key, value):
        print('setitem执行')
        self.__dict__[key] = value

    def __delitem__(self, key):
        print('del obj[key]时，delitem执行')
        self.__dict__.pop(key)

    def __delattr__(self, item):
        print('del obj.key时，delattr执行')
        self.__dict__.pop(item)


f1 = Foo('sb')
```

<!-- more -->

# **setitem**

- 中括号赋值时触发

```python
f1['age'] = 18  # setitem执行

f1['age1'] = 19  # setitem执行
```

# **getitem**

- 中括号取值时触发

```python
f1['age']  # getitem执行 18

f1['name'] = 'tank'  # setitem执行
```

# **delitem**与**delattr**

- **delitem**：中括号删除时触发
- **delattr**：.删除时触发

```python
del f1.age1  # del obj.key时，delattr执行

del f1['age']  # del obj[key]时，delitem执行

print(f1.__dict__)  # {'name': 'tank'}
```

# **format**

- 自定制格式化字符串

```python
date_dic = {
    'ymd': '{0.year}:{0.month}:{0.day}',
    'dmy': '{0.day}/{0.month}/{0.year}',
    'mdy': '{0.month}-{0.day}-{0.year}',
}


class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    def __format__(self, format_spec):
        # 默认打印ymd的{0.year}:{0.month}:{0.day}格式
        if not format_spec or format_spec not in date_dic:
            format_spec = 'ymd'
        fmt = date_dic[format_spec]
        return fmt.format(self)


d1 = Date(2016, 12, 29)

print(format(d1))  # 2016:12:29

print('{:mdy}'.format(d1))  # 12-29-2016
```

# **del**

- **del**也称之为析构方法
- **del**会在对象被删除之前自动触发

```python
class People:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.f = open('test.txt', 'w', encoding='utf-8')

    def __del__(self):
        print('run======>')
        # 做回收系统资源相关的事情
        self.f.close()


obj = People('egon', 18)
del obj  # del obj会间接删除f的内存占用，但是还需要自定制__del__删除文件的系统占用
print('主')
'''
run=-====>
主
'''
```

# slots

## 什么是**slots**

- **slots**是一个类变量，变量值可以是列表，元祖，或者可迭代对象，也可以是一个字符串(意味着所有实例只有一个数据属性)
- 使用点来访问属性本质就是在访问类或者对象的**dict**属性字典(类的字典是共享的，而每个实例的是独立的)

![107-slots-内存.jpg](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1gm2qsa5w7hj30f807zt8r-20211206135642069.jpg)

## 为什么用**slots**

- 字典会占用大量内存，如果你有一个属性很少的类，但是有很多实例，为了节省内存可以使用**slots**取代实例的**dict**

- 当你定义**slots**后，**slots**就会为实例使用一种更加紧凑的内部表示。实例通过一个很小的固定大小的数组来构建，而不是为每个实例定义一个字典，这跟元组或列表很类似。在**slots**中列出的属性名在内部被映射到这个数组的指定小标上。使用**slots**一个不好的地方就是我们不能再给实例添加新的属性了，只能使用在**slots**中定义的那些属性名。

  ```python
  class Foo:
      __slots__='x'


  f1=Foo()
  f1.x=1
  f1.y=2  # 报错
  print(f1.__slots__)  # f1不再有__dict__

  class Bar:
      __slots__=['x','y']

  n=Bar()
  n.x,n.y=1,2
  n.z=3  # 报错
  ```

- 注意：**slots**的很多特性都依赖于普通的基于字典的实现。另外，定义了**slots**后的类不再 支持一些普通类特性了，比如多继承。大多数情况下，你应该只在那些经常被使用到 的用作数据结构的类上定义**slots**比如在程序中需要创建某个类的几百万个实例对象 。

- 关于**slots**的一个常见误区是它可以作为一个封装工具来防止用户给实例增加新的属性。尽管使用**slots**可以达到这样的目的，但是这个并不是它的初衷。它更多的是用来作为一个内存优化工具。

## 刨根问底

![107-slots-刨根问底.jpg](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1gm2qsqvvbaj30dv0bbq33-20211206135642109.jpg)

```python
class Foo:
    __slots__=['name','age']


f1=Foo()
f1.name='alex'
f1.age=18
print(f1.__slots__)  # ['name', 'age']


f2=Foo()
f2.name='egon'
f2.age=19
print(f2.__slots__)  # ['name', 'age']
```

- f1 与 f2 都没有属性字典**dict**了，统一归**slots**管，节省内存

```python
print(Foo.__dict__)
'''
{'__module__': '__main__', '__slots__': ['name', 'age'], 'age': <member 'age' of 'Foo' objects>, 'name': <member 'name' of 'Foo' objects>, '__doc__': None}
'''
```

# **doc**

- 返回类的注释信息

```python
class Foo:
    '我是描述信息'
    pass


print(Foo.__doc__)
我是描述信息
```

- 该属性无法被继承

```python
class Foo:
    '我是描述信息'
    pass

class Bar(Foo):
    pass
print(Bar.__doc__) #该属性无法继承给子类
None
```

# **call**

- 对象后面加括号时，触发执行。
- 注：构造方法的执行是由创建对象触发的，即：对象 = 类名() ；而对于 **call** 方法的执行是由对象后加括号触发的，即：对象() 或者 类()()

```python
class Foo:
    def __init__(self):
        print('__init__触发了')

    def __call__(self, *args, **kwargs):

        print('__call__触发了')

# 执行 __init__
obj = Foo()  # __init__触发了

# 执行 __call__
obj()  # __call__
```

# init 和 new

曾经我幼稚的以为认识了 python 的**init**()方法就相当于认识了类构造器，结果，**new**()方法突然出现在我眼前，让我突然认识到原来**new**才是老大。为什么这么说呢？

我们首先得从**new**(cls[,…])的参数说说起，**new**方法的第一个参数是这个类，而其余的参数会在调用成功后全部传递给**init**方法初始化，这一下子就看出了谁是老子谁是小子的关系。

所以，**new**方法（第一个执行）先于**init**方法执行：

```python
class A:
    pass


class B(A):
    def __new__(cls):
        print("__new__方法被执行")
        return super().__new__(cls)

    def __init__(self):
        print("__init__方法被执行")


b = B()
'''
__new__方法被执行
__init__方法被执行
'''
```

我们比较两个方法的参数，可以发现**new**方法是传入类(cls)，而**init**方法传入类的实例化对象(self)，而有意思的是，**new**方法返回的值就是一个实例化对象（ps:如果**new**方法返回 None，则**init**方法不会被执行，并且返回值只能调用父类中的**new**方法，而不能调用毫无关系的类的**new**方法）。我们可以这么理解它们之间的关系，**new**是开辟疆域的大将军，而**init**是在这片疆域上辛勤劳作的小老百姓，只有**new**执行完后，开辟好疆域后，**init**才能工作。

绝大多数情况下，我们都不需要自己重写**new**方法，但在当继承一个不可变的类型（例如 str 类,int 类等）时，它的特性就尤显重要了。我们举下面这个例子：

```python
class CapStr(str):
    def __init__(self, string):
        string = string.upper()


a = CapStr("I love China!")
print(a)
I love China!
class CapStr(str):
    def __new__(cls, string):
        string = string.upper()
        return super().__new__(cls, string)


a = CapStr("I love China!")
print(a)  # I LOVE CHINA!
```

我们可以根据上面的理论可以这样分析，我们知道字符串是不可改变的，所以第一个例子中，传入的字符串相当于已经被打下的疆域，而这块疆域除了将军其他谁也无法改变，**init**只能在这块领地上干瞪眼，此时这块疆域就是”I love China!“。而第二个例子中，**new**大将军重新去开辟了一块疆域，所以疆域上的内容也发生了变化，此时这块疆域变成了”I LOVE CHINA!“。

## 小结

小结：**\*\*new\*\*和\*\*init\*\*想配合才是 python 中真正的类构造器**

# **str**

- 打印时触发

```python
class Foo:
    pass


obj = Foo()

print(obj)  # <__main__.Foo object at 0x10d2b8f98>

dic = {'a': 1}  # d = dict({'x':1})
print(dic)  # {'a': 1}
```

- obj 和 dic 都是实例化的对象，但是 obj 打印的是内存地址，而 dic 打印的是有用的信息，很明显 dic 的打印是非常好的

```python
class Foo:
    def __init__(self, name, age):
        """对象实例化的时候自动触发"""
        self.name = name
        self.age = age

    def __str__(self):
        print('打印的时候自动触发，但是其实不需要print即可打印')
        return f'{self.name}:{self.age}'  # 如果不返回字符串类型，则会报错


obj = Foo('nick', 18)
print(obj)  # obj.__str__() # 打印的时候就是在打印返回值
'''
打印的时候自动触发，但是其实不需要print即可打印
nick:18
'''

obj2 = Foo('tank', 30)
print(obj2)
'''
打印的时候自动触发，但是其实不需要print即可打印
tank:30
'''
```

# **repr**

- str 函数或者 print 函数—>obj.**str**()
- repr 或者交互式解释器—>obj.**repr**()
- 如果**str**没有被定义，那么就会使用**repr**来代替输出
- 注意：这俩方法的返回值必须是字符串，否则抛出异常

```python
class School:
    def __init__(self, name, addr, type):
        self.name = name
        self.addr = addr
        self.type = type

    def __repr__(self):
        return 'School(%s,%s)' % (self.name, self.addr)

    def __str__(self):
        return '(%s,%s)' % (self.name, self.addr)


s1 = School('oldboy1', '北京', '私立')
print('from repr: ', repr(s1))  # # from repr:  School(oldboy1,北京)

print('from str: ', str(s1))  # from str:  (oldboy1,北京)

print(s1)  # (oldboy1,北京)
```

# module 和 class

```python
# lib/aa.py
class C:
    def __init__(self):
        self.name = 'SB'
# index.py

from lib.aa import C

obj = C()
```

## **module**

- **module** 表示当前操作的对象在那个模块

```python
print(obj.__module__)  # 输出 lib.aa，即：输出模块
```

## **class**

- **class**表示当前操作的对象的类是什么

```python
print(obj.__class__)  # 输出 lib.aa.C，即：输出类
```
