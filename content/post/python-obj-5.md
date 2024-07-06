---
title: 封装和property特性
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-05.html
toc: true
---

# 封装什么

- 你钱包的有多少钱（数据的封装）
- 你的性取向（数据的封装）
- 你撒尿的具体功能是怎么实现的（方法的封装）
<!-- more -->

# 为什么要封装

封装数据的主要原因是：保护隐私（作为男人的你，脸上就写着：我喜欢男人，你害怕么？）

封装方法的主要原因是：隔离复杂度（快门就是傻瓜相机为傻瓜们提供的方法，该方法将内部复杂的照相功能都隐藏起来了，比如你不必知道你自己的尿是怎么流出来的，你直接掏出自己的接口就能用尿这个功能）

提示：在编程语言里，对外提供的接口（接口可理解为了一个入口），就是函数，称为接口函数，这与接口的概念还不一样，接口代表一组接口函数的集合体。

# 两个层面的封装

封装其实分为两个层面，但无论哪种层面的封装，都要对外界提供好访问你内部隐藏内容的接口（接口可以理解为入口，有了这个入口，使用者无需且不能够直接访问到内部隐藏的细节，只能走接口，并且我们可以在接口的实现上附加更多的处理逻辑，从而严格控制使用者的访问）

## 第一个层面

第一个层面的封装（什么都不用做）：创建类和对象会分别创建二者的名称空间，我们只能用类名.或者 obj.的方式去访问里面的名字，这本身就是一种封装

注意：对于这一层面的封装（隐藏），类名.和实例名.就是访问隐藏属性的接口

## 第二个层面

第二个层面的封装：类中把某些属性和方法隐藏起来(或者说定义成私有的)，只在类的内部使用、外部无法访问，或者留下少量接口（函数）供外部访问。

在 python 中用双下划线的方式实现隐藏属性（设置成私有的）

类中所有双下划线开头的名称如**x 都会自动变形成：\_类名**x 的形式：

```python
class A:
    __N = 0  # 类的数据属性就应该是共享的,但是语法上是可以把类的数据属性设置成私有的如__N,会变形为_A__N

    def __init__(self):
        self.__X = 10  # 变形为self._A__X

    def __foo(self):  # 变形为_A__foo
        print('from A')

    def bar(self):
        self.__foo()  # 只有在类内部才可以通过__foo的形式访问到.
```

这种自动变形的特点：

1. 类中定义的**x 只能在内部使用，如 self.**x，引用的就是变形的结果。
2. 这种变形其实正是针对内部的变形，在外部是无法通过\_\_x 这个名字访问到的。
3. 在子类定义的**x 不会覆盖在父类定义的**x，因为子类中变形成了：*子类名\*\*x,而父类中变形成了：*父类名\*\*x，即双下滑线开头的属性在继承给子类时，子类是无法覆盖的。

**注意：对于这一层面的封装（隐藏），我们需要在类中定义一个函数（接口函数）在它内部访问被隐藏的属性，然后外部就可以使用了**

这种变形需要注意的问题是：

1. 这种机制也并没有真正意义上限制我们从外部直接访问属性，知道了类名和属性名就可以拼出名字：\_类名**属性，然后就可以访问了，如 a.\_A**N

```python
# 对象测试
a = A()
print(a._A__N)  # 0

# 对象测试
print(a._A__X)  # 10

# 类测试
print(A._A__N)  # 0

# 类测试
try:
    print(A._A__X)  # 对象私有的属性
except Exception as e:
    print(e)

# type object 'A' has no attribute '_A__X'
```

1. 变形的过程只在类的定义时发生一次,在定义后的赋值操作，不会变形

```python
a = A()
print(a.__dict__)  # {'_A__X': 10}

a.__Y = 1
print(a.__dict__)  # {'_A__X': 10, '__Y': 1}
```

1. 在继承中，父类如果不想让子类覆盖自己的方法，可以将方法定义为私有的

```python
# 正常情况
class A:
    def fa(self):
        print('from A')

    def test(self):
        self.fa()


class B(A):
    def fa(self):
        print('from B')

b = B()
b.test()

from B


# 把fa定义成私有的，即__fa
class A:
    def __fa(self):  # 在定义时就变形为_A__fa
        print('from A')

    def test(self):
        self.__fa()  # 只会与自己所在的类为准,即调用_A__fa


class B(A):
    def __fa(self):
        print('from B')


b = B()
b.test()

from A
```

# 私有模块

python 并不会真的阻止你访问私有的属性，模块也遵循这种约定，如果模块中的变量名\_private_module 以单下划线开头，那么 from module import \*时不能被导入该变量，但是你 from module import \_private_module 依然是可以导入该变量的

其实很多时候你去调用一个模块的功能时会遇到单下划线开头的(socket.\_socket,sys.\_home,sys.\_clear_type_cache),这些都是私有的，原则上是供内部调用的，作为外部的你，一意孤行也是可以用的，只不过显得稍微傻逼一点点

python 要想与其他编程语言一样，严格控制属性的访问权限，只能借助内置方法如**getattr**，详见面向对象高级部分。

# 练习

- 多态是在定义角度
- 多态性是在调用角度（使用角度）

```python
class A:
    def fa(self):
        print('from A')

    def test(self):
        self.fa()


class B(A):
    def fa(self):
        print('from B')


b = B()
b.test()

from B



class A:
    def __fa(self):
        print('from A')

    def test(self):
        self.__fa()


class B(A):
    def __fa(self):
        print('from B')


b = B()
b.test()

from A
```

注：\_\_名字，这种语法只在定义的时候才有变形的效果，如果类或对象已经产生了，就不会有变形的效果了。

# 什么是 property 特性

- property 装饰器用于将被装饰的方法伪装成一个数据属性，在使用时可以不用加括号而直接使用

```python
# ############### 定义 ###############
class Foo:
    def func(self):
        pass

    # 定义property属性
    @property
    def prop(self):
        pass


# ############### 调用 ###############
foo_obj = Foo()
foo_obj.func()  # 调用实例方法
foo_obj.prop  # 调用property属性
```

如下的例子用于说明如何定一个简单的 property 属性：

```python
class Goods(object):
    @property
    def size(self):
        return 100


g = Goods()
print(g.size)  # 100
```

property 属性的定义和调用要注意一下几点：

1. 定义时，在实例方法的基础上添加 @property 装饰器；并且仅有一个 self 参数

2. 调用时，无需括号

# 简单示例

对于京东商城中显示电脑主机的列表页面，每次请求不可能把数据库中的所有内容都显示到页面上，而是通过分页的功能局部显示，所以在向数据库中请求数据时就要显示的指定获取从第 m 条到第 n 条的所有数据 这个分页的功能包括：

1. 根据用户请求的当前页和总数据条数计算出 m 和 n
2. 根据 m 和 n 去数据库中请求数据

```python
# ############### 定义 ###############
class Pager:
    def __init__(self, current_page):
        # 用户当前请求的页码（第一页、第二页...）
        self.current_page = current_page
        # 每页默认显示10条数据
        self.per_items = 10

    @property
    def start(self):
        val = (self.current_page - 1) * self.per_items
        return val

    @property
    def end(self):
        val = self.current_page * self.per_items
        return val


# ############### 调用 ###############
p = Pager(1)
print(p.start)  # 0 就是起始值，即：m

print(p.end)  # 10 就是结束值，即：n

```

从上述可见 Python 的 property 属性的功能是：property 属性内部进行一系列的逻辑计算，最终将计算结果返回。

# property 属性的两种方式

1. 装饰器 即：在方法上应用装饰器（推荐使用）
2. 类属性 即：在类中定义值为 property 对象的类属性（Python2 历史遗留）

## 装饰器

在类的实例方法上应用 @property 装饰器

Python 中的类有经典类和新式类，新式类的属性比经典类的属性丰富。（ 如果类继 object，那么该类是新式类 ）

经典类，具有一种 @property 装饰器：

```python
# ############### 定义 ###############
class Goods:
    @property
    def price(self):
        return "laowang"


# ############### 调用 ###############
obj = Goods()
result = obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
print(result)  # laowang
```

新式类，具有三种 @property 装饰器：

```python
#coding=utf-8
# ############### 定义 ###############
class Goods:
    """python3中默认继承object类
        以python2、3执行此程序的结果不同，因为只有在python3中才有@xxx.setter  @xxx.deleter
    """

    @property
    def price(self):
        print('@property')

    @price.setter
    def price(self, value):
        print('@price.setter')

    @price.deleter
    def price(self):
        print('@price.deleter')


# ############### 调用 ###############
obj = Goods()
obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值

@property

obj.price = 123  # 自动执行 @price.setter 修饰的 price 方法，并将  123 赋值给方法的参数

@price.setter

del obj.price  # 自动执行 @price.deleter 修饰的 price 方法

@price.deleter
```

注意：

- **经典类中的属性只有一种访问方式，其对应被 @property 修饰的方法**
- **新式类中的属性有三种访问方式，并分别对应了三个被 @property、@方法名.setter、@方法名.deleter 修饰的方法**

由于新式类中具有三种访问方式，我们可以根据它们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除

```python
class Goods(object):
    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    @property
    def price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    @price.setter
    def price(self, value):
        self.original_price = value

    @price.deleter
    def price(self):
        print('del')
        del self.original_price


obj = Goods()
print(obj.price)  # 80.0 获取商品价格

obj.price = 200  # 修改商品原价
print(obj.price) # 160.0

del obj.price  # 删除商品原价

```

## 类属性方式

创建值为 property 对象的类属性

注意：**当使用类属性的方式创建 property 属性时，经典类和新式类无区别**

```python
class Foo:
    def get_bar(self):
        return 'laowang'

    BAR = property(get_bar)


obj = Foo()
reuslt = obj.BAR  # 自动调用get_bar方法，并获取方法的返回值
print(reuslt)  # laowang
```

property 方法中有个四个参数

1. 第一个参数是方法名，调用 对象.属性 时自动触发执行方法
2. 第二个参数是方法名，调用 对象.属性 ＝ XXX 时自动触发执行方法
3. 第三个参数是方法名，调用 del 对象.属性 时自动触发执行方法
4. 第四个参数是字符串，调用 对象.属性.**doc** ，此参数是该属性的描述信息

```python
#coding=utf-8
class Foo(object):
    def get_bar(self):
        print("getter...")
        return 'laowang'

    def set_bar(self, value):
        """必须两个参数"""
        print("setter...")
        return 'set value' + value

    def del_bar(self):
        print("deleter...")
        return 'laowang'

    BAR = property(get_bar, set_bar, del_bar, "description...")


obj = Foo()

# 自动调用第一个参数中定义的方法：get_bar
obj.BAR
'''
getter...


'laowang'
'''

# 自动调用第二个参数中定义的方法：set_bar方法，并将“alex”当作参数传入
obj.BAR = "alex"  # setter...

desc = Foo.BAR.__doc__  # 自动获取第四个参数中设置的值：description...
print(desc)  # description...

# 自动调用第三个参数中定义的方法：del_bar方法
del obj.BAR  # deleter...
```

由于类属性方式创建 property 属性具有 3 种访问方式，我们可以根据它们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除

```python
class Goods(object):
    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    def get_price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    def set_price(self, value):
        self.original_price = value

    def del_price(self):
        del self.original_price

    PRICE = property(get_price, set_price, del_price, '价格属性描述...')


obj = Goods()
obj.PRICE  # 80.0 获取商品价格

obj.PRICE = 200  # 修改商品原价
print(obj.PRICE)  # 160.0

del obj.PRICE  # 删除商品原价
```

综上所述:

- 定义 property 属性共有两种方式，分别是【装饰器】和【类属性】，而【装饰器】方式针对经典类和新式类又有所不同。
- 通过使用 property 属性，能够简化调用者在获取数据的流程

# property+类的封装

```python
class People:
    def __init__(self, name):
        self.__name = name

    @property  # 查看obj.name
    def name(self):
        return '<名字是：%s>' % self.__name


peo1 = People('lqz')
print(peo1.name)  # <名字是：lqz>

try:
    peo1.name = 'EGON'
except Exception as e:
    print(e)
# can't set attribute
```

# 应用

## 私有属性添加 getter 和 setter 方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    def getMoney(self):
        return self.__money

    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")
```

## 使用 property 升级 getter 和 setter 方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    def getMoney(self):
        return self.__money

    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")

    # 定义一个属性，当对这个money设置值时调用setMoney,当获取值时调用getMoney
    money = property(getMoney, setMoney)


a = Money()
a.money = 100  # 调用setMoney方法
print(a.money)  # 100 调用getMoney方法
```

## 使用 property 取代 getter 和 setter 方法

重新实现一个属性的设置和读取方法,可做边界判定

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    # 使用装饰器对money进行装饰，那么会自动添加一个叫money的属性，当调用获取money的值时，调用装饰的方法
    @property
    def money(self):
        return self.__money

    # 使用装饰器对money进行装饰，当对money设置值时，调用装饰的方法
    @money.setter
    def money(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")


a = Money()
a.money = 100
print(a.money)  # 100
```

# 练习

计算圆的周长和面积

```python
import math


class Circle:
    def __init__(self, radius):  # 圆的半径radius
        self.radius = radius

    @property
    def area(self):
        return math.pi * self.radius**2  # 计算面积

    @property
    def perimeter(self):
        return 2 * math.pi * self.radius  # 计算周长


c = Circle(10)
print(c.radius)  # 10


# 可以向访问数据属性一样去访问area，会触发一个函数的执行，动态计算出一个值
print(c.area)  # 314.1592653589793

print(c.perimeter)  # 62.83185307179586 同上

```
