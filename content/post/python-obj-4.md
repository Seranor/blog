---
title: 调用父类、super()、多态
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-04.html
toc: true
---

# 单独调用父类的方法

需求：编写一个类，然后再写一个子类进行继承，使用子类去调用父类的方法 1。

使用方法 1 打印： 胖子老板，来包槟榔。

<!-- more -->

> 那么先写一个胖子老板的父类，执行一下：

```python
class FatFather(object):
    def __init__(self, name):
        print('FatFather的init开始被调用')
        self.name = name
        print('FatFather的name是%s' % self.name)
        print('FatFather的init调用结束')


def main():
    ff = FatFather("胖子老板的父亲")
```

运行一下这个胖子老板父类的构造方法**init** 如下：

```python
if __name__ == "__main__":
    main()
'''
FatFather的init开始被调用
FatFather的name是胖子老板的父亲
FatFather的init调用结束
'''
```

> 好了，那么下面来写一个子类，也就是胖子老板类，继承上面的类

```python
# 胖子老板的父类
class FatFather(object):
    def __init__(self, name):
        print('FatFather的init开始被调用')
        self.name = name
        print('调用FatFather类的name是%s' % self.name)
        print('FatFather的init调用结束')


# 胖子老板类 继承 FatFather 类
class FatBoss(FatFather):
    def __init__(self, name, hobby):
        print('胖子老板的类被调用啦！')
        self.hobby = hobby
        FatFather.__init__(self, name)  # 直接调用父类的构造方法
        print("%s 的爱好是 %s" % (name, self.hobby))


def main():
    #ff = FatFather("胖子老板的父亲")
    fatboss = FatBoss("胖子老板", "打斗地主")
```

在这上面的代码中，我使用 FatFather.**init**(self,name)直接调用父类的方法。
运行结果如下：

```python
if __name__ == "__main__":
    main()
'''
胖子老板的类被调用啦！
FatFather的init开始被调用
调用FatFather类的name是胖子老板
FatFather的init调用结束
胖子老板 的爱好是 打斗地主
'''
```

# super() 方法基本概念

> 除了直接使用 FatFather.**init**(self,name) 的方法，还可以使用 super()方法来调用。

那么首先需要看 super()方法的描述和语法理解一下 super() 方法的使用。

## 描述

**super() 函数是用于调用父类(超类)的一个方法。**

super 是用来解决多重继承问题的，直接用类名调用父类方法在使用单继承的时候没问题，但是如果使用多继承，会涉及到查找顺序（MRO）、重复调用（钻石继承）等种种问题。

MRO 就是类的方法解析顺序表, 其实也就是继承父类方法时的顺序表。

## 语法

以下是 super() 方法的语法:

```python
super(type[, object-or-type])
```

> 参数

- type – 类
- object-or-type – 类，一般是 self

> Python3.x 和 Python2.x 的一个区别是: Python 3 可以使用直接使用 super().xxx 代替 super(Class, self).xxx :

- Python3.x 实例：

```python
class A:
    pass
class B(A):
    def add(self, x):
        super().add(x)
```

- Python2.x 实例：

```
class A(object):   # Python2.x 记得继承 object
    pass
class B(A):
    def add(self, x):
        super(B, self).add(x)
```

## 单继承使用 super()

- 使用 super() 方法来改写刚才胖子老板继承父类的 **init** 构造方法

```python
# 胖子老板的父类
class FatFather(object):
    def __init__(self, name):
        print('FatFather的init开始被调用')
        self.name = name
        print('调用FatFather类的name是%s' % self.name)
        print('FatFather的init调用结束')


# 胖子老板类 继承 FatFather 类
class FatBoss(FatFather):
    def __init__(self, name, hobby):
        print('胖子老板的类被调用啦！')
        self.hobby = hobby
        #FatFather.__init__(self,name)   # 直接调用父类的构造方法
        super().__init__(name)
        print("%s 的爱好是 %s" % (name, self.hobby))


def main():
    #ff = FatFather("胖子老板的父亲")
    fatboss = FatBoss("胖子老板", "打斗地主")
```

从上面使用 super 方法的时候，因为是单继承，直接就可以使用了。
运行如下：

```python
if __name__ == "__main__":
    main()
'''
胖子老板的类被调用啦！
FatFather的init开始被调用
调用FatFather类的name是胖子老板
FatFather的init调用结束
胖子老板 的爱好是 打斗地主
'''
```

那么为什么说**单继承**直接使用就可以呢？因为 super()方法如果多继承的话，会涉及到一个 MRO(**继承父类方法时的顺序表**) 的调用排序问题。

> 下面可以打印一下看看单继承的 MRO 顺序(FatBoss.**mro**)。

```python
# 胖子老板的父类
class FatFather(object):
    def __init__(self, name):
        print('FatFather的init开始被调用')
        self.name = name
        print('调用FatFather类的name是%s' % self.name)
        print('FatFather的init调用结束')


# 胖子老板类 继承 FatFather 类
class FatBoss(FatFather):
    def __init__(self, name, hobby):
        print('胖子老板的类被调用啦！')
        self.hobby = hobby
        #FatFather.__init__(self,name)   # 直接调用父类的构造方法
        super().__init__(name)
        print("%s 的爱好是 %s" % (name, self.hobby))


def main():
    print("打印FatBoss类的MRO")
    print(FatBoss.__mro__)

    print()
    print("=========== 下面按照 MRO 顺序执行super方法 =============")
    fatboss = FatBoss("胖子老板", "打斗地主")
```

上面的代码使用 FatBoss.**mro** 可以打印出 FatBoss 这个类经过 python 解析器的 C3 算法计算过后的继承调用顺序。
运行如下：

```python
if __name__ == "__main__":
    main()
'''
打印FatBoss类的MRO
(<class '__main__.FatBoss'>, <class '__main__.FatFather'>, <class 'object'>)

=========== 下面按照 MRO 顺序执行super方法 ===========
胖子老板的类被调用啦！
FatFather的init开始被调用
调用FatFather类的name是胖子老板
FatFather的init调用结束
胖子老板 的爱好是 打斗地主
'''
```

从上面的结果 (<class ‘**main**.FatBoss’>, <class ‘**main**.FatFather’>, <class ‘object’>) 可以看出，super() 方法在 FatBoss 会直接调用父类是 FatFather ，所以单继承是没问题的。

那么如果多继承的话，会有什么问题呢？

## 多继承使用 super()

假设再写一个胖子老板的女儿类，和 胖子老板的老婆类，此时女儿需要同时继承 两个类（胖子老板类，胖子老板老婆类）。

因为胖子老板有一个爱好，胖子老板的老婆需要干活干家务，那么女儿需要帮忙同时兼顾。

此时女儿就是需要继承使用这两个父类的方法了，那么该如何去写呢？

下面来看看实现代码：

```python
# 胖子老板的父类
class FatFather(object):
    def __init__(self, name, *args, **kwargs):
        print()
        print("=============== 开始调用 FatFather  ========================")
        print('FatFather的init开始被调用')
        self.name = name
        print('调用FatFather类的name是%s' % self.name)
        print('FatFather的init调用结束')
        print()
        print("=============== 结束调用 FatFather  ========================")


# 胖子老板类 继承 FatFather 类
class FatBoss(FatFather):
    def __init__(self, name, hobby, *args, **kwargs):
        print()
        print("=============== 开始调用 FatBoss  ========================")
        print('胖子老板的类被调用啦！')
        #super().__init__(name)
        ## 因为多继承传递的参数不一致，所以使用不定参数
        super().__init__(name, *args, **kwargs)
        print("%s 的爱好是 %s" % (name, hobby))
        print()
        print("=============== 结束调用 FatBoss  ========================")


# 胖子老板的老婆类 继承 FatFather类
class FatBossWife(FatFather):
    def __init__(self, name, housework, *args, **kwargs):
        print()
        print("=============== 开始调用 FatBossWife  ========================")
        print('胖子老板的老婆类被调用啦！要学会干家务')
        #super().__init__(name)
        ## 因为多继承传递的参数不一致，所以使用不定参数
        super().__init__(name, *args, **kwargs)
        print("%s 需要干的家务是 %s" % (name, housework))
        print()
        print("=============== 结束调用 FatBossWife  ========================")


# 胖子老板的女儿类 继承 FatBoss FatBossWife类
class FatBossGril(FatBoss, FatBossWife):
    def __init__(self, name, hobby, housework):
        print('胖子老板的女儿类被调用啦！要学会干家务，还要会帮胖子老板斗地主')
        super().__init__(name, hobby, housework)


def main():

    print("打印FatBossGril类的MRO")
    print(FatBossGril.__mro__)

    print()
    print("========== 下面按照 MRO 顺序执行super方法 ============")
    gril = FatBossGril("胖子老板", "打斗地主", "拖地")
```

运行结果如下：

```python
if __name__ == "__main__":
    main()

'''
打印FatBossGril类的MRO
(<class '__main__.FatBossGril'>, <class '__main__.FatBoss'>, <class '__main__.FatBossWife'>, <class '__main__.FatFather'>, <class 'object'>)


=========== 下面按照 MRO 顺序执行super方法 =============
胖子老板的女儿类被调用啦！要学会干家务，还要会帮胖子老板斗地主

=============== 开始调用 FatBoss  ========================
胖子老板的类被调用啦！

=============== 开始调用 FatBossWife =====================
胖子老板的老婆类被调用啦！要学会干家务

=============== 开始调用 FatFather  ========================
FatFather的init开始被调用
调用FatFather类的name是胖子老板
FatFather的init调用结束

=============== 结束调用 FatFather  ========================
胖子老板 需要干的家务是 拖地

=============== 结束调用 FatBossWife  ======================
胖子老板 的爱好是 打斗地主

=============== 结束调用 FatBoss  =======================
'''
```

从上面的运行结果来看，我特意给每个类的调用开始以及结束都进行打印标识，可以看到。

每个类开始调用是根据 MRO 顺序进行开始，然后逐个进行结束的。

还有就是由于因为需要继承不同的父类，参数不一定。

所以，所有的父类都应该加上不定参数\*args , \*\*kwargs ，不然参数不对应是会报错的。

# 注意事项

- super().**init**相对于类名.**init**，在单继承上用法基本无差
- 但在多继承上有区别，super 方法能保证每个父类的方法只会执行一次，而使用类名的方法会导致方法被执行多次，可以尝试写个代码来看输出结果
- 多继承时，使用 super 方法，对父类的传参数，应该是由于 python 中 super 的算法导致的原因，必须把参数全部传递，否则会报错
- 单继承时，使用 super 方法，则不能全部传递，只能传父类方法所需的参数，否则会报错
- 多继承时，相对于使用类名.**init**方法，要把每个父类全部写一遍, 而使用 super 方法，只需写一句话便执行了全部父类的方法，这也是为何多继承需要全部传参的一个原因

# 练习

> 以下的代码的输出将是什么? 说出你的答案并解释。

```python
class Parent(object):
    x = 1


class Child1(Parent):
    pass


class Child2(Parent):
    pass


print(Parent.x, Child1.x, Child2.x)  # 1 1 1

Child1.x = 2
print(Parent.x, Child1.x, Child2.x)  # 1 2 1
```

- 注意：Child1 已经拥有了属于自己的 x

```python
Parent.x = 3
print(Parent.x, Child1.x, Child2.x)  # 3 2 3
```

很多人喜欢将多态与多态性二者混为一谈，然后百思不得其解，其实只要分开看，就会很明朗。

# 多态

多态指的是一类事物有多种形态，（一个抽象类有多个子类，因而多态的概念依赖于继承）

1. 序列数据类型有多种形态：字符串，列表，元组
2. 动物有多种形态：人，狗，猪

## 动物的多种形态

```python
# 动物有多种形态：人类、猪、狗
class Animal:
    def run(self):  # 子类约定俗称的必须实现这个方法
        raise AttributeError('子类必须实现这个方法')


class People(Animal):
    def run(self):
        print('人正在走')


class Pig(Animal):
    def run(self):
        print('pig is walking')


class Dog(Animal):
    def run(self):
        print('dog is running')


peo1 = People()
pig1 = Pig()
d1 = Dog()

peo1.run()  # 人正在走
pig1.run()  # pig is walking
d1.run()  # dog is running
```

```python
import abc


class Animal(metaclass=abc.ABCMeta):  # 同一类事物:动物
    @abc.abstractmethod  # 上述代码子类是约定俗称的实现这个方法，加上@abc.abstractmethod装饰器后严格控制子类必须实现这个方法
    def talk(self):
        raise AttributeError('子类必须实现这个方法')


class People(Animal):  # 动物的形态之一:人
    def talk(self):
        print('say hello')


class Dog(Animal):  # 动物的形态之二:狗
    def talk(self):
        print('say wangwang')


class Pig(Animal):  # 动物的形态之三:猪
    def talk(self):
        print('say aoao')


peo2 = People()
pig2 = Pig()
d2 = Dog()

peo2.talk()  # say hello
pig2.talk()  # say aoao
d2.talk()  # say wangwang
```

## 文件的多种形态

```python
# 文件有多种形态：文件、文本文件、可执行文件
import abc


class File(metaclass=abc.ABCMeta):  # 同一类事物:文件
    @abc.abstractmethod
    def click(self):
        pass


class Text(File):  # 文件的形态之一:文本文件
    def click(self):
        print('open file')


class ExeFile(File):  # 文件的形态之二:可执行文件
    def click(self):
        print('execute file')


text = Text()
exe_file = ExeFile()

text.click()  # open file
exe_file.click()  # execute file
```

# 多态性

注意：多态与多态性是两种概念

多态性是指具有不同功能的函数可以使用相同的函数名，这样就可以用一个函数名调用不同内容的函数。在面向对象方法中一般是这样表述多态性：向不同的对象发送同一条消息，不同的对象在接收时会产生不同的行为（即方法）。也就是说，每个对象可以用自己的方式去响应共同的消息。所谓消息，就是调用函数，不同的行为就是指不同的实现，即执行不同的函数。

## 动物形态多态性的使用

```python
# 多态性：一种调用方式，不同的执行效果（多态性）
def func(obj):
    obj.run()


func(peo1)  # 人正在走
func(pig1)  # pig is walking
func(d1)  # dog is running




# 多态性依赖于：继承
# 多态性：定义统一的接口
def func(obj):  # obj这个参数没有类型限制，可以传入不同类型的值
    obj.talk()  # 调用的逻辑都一样，执行的结果却不一样


func(peo2)  # say hello
func(pig2)  # say aoao
func(d2)  # say wangwang
```

## 文件形态多态性的使用

```python
def func(obj):
    obj.click()


func(text)  # open file
func(exe_file)  # execute file
```

## 序列数据类型多态性的使用

```python
def func(obj):
    print(len(obj))


func('hello')  # 5
func([1, 2, 3])  # 3
func((1, 2, 3))  # 3
```

综上可以说，多态性是一个接口（函数 func）的多种实现（如 obj.run()，obj.talk()，obj.click()，len(obj)）

# 多态性的好处

其实大家从上面多态性的例子可以看出，我们并没有增加新的知识，也就是说 Python 本身就是支持多态性的，这么做的好处是什么呢？

1. 增加了程序的灵活性：以不变应万变，不论对象千变万化，使用者都是同一种形式去调用，如 func(animal)
2. 增加了程序额可扩展性：通过继承 Animal 类创建了一个新的类，使用者无需更改自己的代码，还是用 func(animal)去调用

```python
class Cat(Animal):  # 属于动物的另外一种形态：猫
    def talk(self):
        print('say miao')


def func(animal):  # 对于使用者来说，自己的代码根本无需改动
    animal.talk()


cat1 = Cat()  # 实例出一只猫
func(cat1)  # say miao 甚至连调用方式也无需改变，就能调用猫的talk功能
```

- 上述代码我们新增了一个形态 Cat，由 Cat 类产生的实例 cat1，使用者可以在完全不需要修改自己代码的情况下。使用和人、狗、猪一样的方式调用 cat1 的 talk 方法，即 func(cat1)

# 小结

多态：同一种事物的多种形态，动物分为人类，猪类（在定义角度）
多态性：一种调用方式，不同的执行效果（多态性）
