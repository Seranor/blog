---
title: 派生、继承、分类及菱形问题
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-03.html
toc: true
---

# 类的派生

- 派生：子类中新定义的属性的这个过程叫做派生，并且需要记住子类在使用派生的属性时始终以自己的为准

![90-类的派生-基因遗传.jpg](https://gitee.com/gengff/blogimage/raw/master/images/0081Kckwgy1glvtuz78n9j30dw08hdfz-20211206102016340.jpg)

<!-- more -->

## 派生方法一(类调用)

- 指名道姓访问某一个类的函数：该方式与继承无关

```python
class OldboyPeople:
    """由于学生和老师都是人，因此人都有姓名、年龄、性别"""
    school = 'oldboy'

    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.gender = gender


class OldboyStudent(OldboyPeople):
    """由于学生类没有独自的__init__()方法，因此不需要声明继承父类的__init__()方法，会自动继承"""

    def choose_course(self):
        print('%s is choosing course' % self.name)


class OldboyTeacher(OldboyPeople):
    """由于老师类有独自的__init__()方法，因此需要声明继承父类的__init__()"""

    def __init__(self, name, age, gender, level):
        OldboyPeople.__init__(self, name, age, gender)
        self.level = level  # 派生

    def score(self, stu_obj, num):
        print('%s is scoring' % self.name)
        stu_obj.score = num


stu1 = OldboyStudent('tank', 18, 'male')
tea1 = OldboyTeacher('lqz', 18, 'male', 10)


print(stu1.__dict__)  # {'name': 'tank', 'age': 18, 'gender': 'male'}
print(tea1.__dict__)  # {'name': 'lqz', 'age': 18, 'gender': 'male', 'level': 10}
```

## 派生方法二(super)

- 严格以来继承属性查找关系
- super()会得到一个特殊的对象，该对象就是专门用来访问父类中的属性的（按照继承的关系）
- super().**init**(不用为 self 传值)
- super 的完整用法是 super(自己的类名,self),在 python2 中需要写完整，而 python3 中可以简写为 super()

```python
class OldboyPeople:
    school = 'oldboy'

    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex


class OldboyStudent(OldboyPeople):
    def __init__(self, name, age, sex, stu_id):
        # OldboyPeople.__init__(self,name,age,sex)
        # super(OldboyStudent, self).__init__(name, age, sex)
        super().__init__(name, age, sex)
        self.stu_id = stu_id

    def choose_course(self):
        print('%s is choosing course' % self.name)


stu1 = OldboyStudent('tank', 19, 'male', 1)


print(stu1.__dict__)  # {'name': 'tank', 'age': 19, 'sex': 'male', 'stu_id': 1}
```

# 类的组合

## 什么是组合

- 组合就是一个类的对象具备某一个属性，该属性的值是指向另外外一个类的对象

## 为什么用组合

- 组合是用来解决类与类之间代码冗余的问题
- 首先我们先写一个简单版的选课系统

```python
class OldboyPeople:
    school = 'oldboy'

    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex


class OldboyStudent(OldboyPeople):
    def __init__(self, name, age, sex, stu_id):
        OldboyPeople.__init__(self, name, age, sex)
        self.stu_id = stu_id

    def choose_course(self):
        print('%s is choosing course' % self.name)


class OldboyTeacher(OldboyPeople):
    def __init__(self, name, age, sex, level):
        OldboyPeople.__init__(self, name, age, sex)
        self.level = level

    def score(self, stu, num):
        stu.score = num
        print('老师[%s]为学生[%s]打分[%s]' % (self.name, stu.name, num))


stu1 = OldboyStudent('tank', 19, 'male', 1)
tea1 = OldboyTeacher('lqz', 18, 'male', 10)

stu1.choose_course()  # tank is choosing course
tea1.score(stu1, 100)  # 老师[lqz]为学生[tank]打分[100]

print(stu1.__dict__)  # {'name': 'tank', 'age': 19, 'sex': 'male', 'stu_id': 1, 'score': 100}
```

- 如上设计了一个选课系统，但是这个选课系统在未来一定是要修改、扩展的，因此我们需要修改上述的代码

## 如何用组合

- 需求：假如我们需要给学生增添课程属性，但是又不是所有的老男孩学生一进学校就有课程属性，课程属性是学生来老男孩后选出来的，也就是说课程需要后期学生们添加进去的
- 实现思路：如果我们直接在学生中添加课程属性，那么学生刚被定义就需要添加课程属性，这就不符合我们的要求，因此我们可以使用组合能让学生未来添加课程属性

```python
class Course:
    def __init__(self, name, period, price):
        self.name = name
        self.period = period
        self.price = price

    def tell_info(self):
        msg = """
        课程名：%s
        课程周期：%s
        课程价钱：%s
        """ % (self.name, self.period, self.price)
        print(msg)


class OldboyPeople:
    school = 'oldboy'

    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex


class OldboyStudent(OldboyPeople):
    def __init__(self, name, age, sex, stu_id):
        OldboyPeople.__init__(self, name, age, sex)
        self.stu_id = stu_id

    def choose_course(self):
        print('%s is choosing course' % self.name)


class OldboyTeacher(OldboyPeople):
    def __init__(self, name, age, sex, level):
        OldboyPeople.__init__(self, name, age, sex)
        self.level = level

    def score(self, stu, num):
        stu.score = num
        print('老师[%s]为学生[%s]打分[%s]' % (self.name, stu.name, num))


# 创造课程
python = Course('python全栈开发', '5mons', 3000)
python.tell_info()
'''
课程名：python全栈开发
课程周期：5mons
课程价钱：3000
'''

linux = Course('linux运维', '5mons', 800)
linux.tell_info()
'''
课程名：linux运维
课程周期：5mons
课程价钱：800
'''

# 创造学生与老师
stu1 = OldboyStudent('tank', 19, 'male', 1)
tea1 = OldboyTeacher('lqz', 18, 'male', 10)
```

- 组合

```python
# 将学生、老师与课程对象关联/组合
stu1.course = python
tea1.course = linux

stu1.course.tell_info()
'''
课程名：python全栈开发
课程周期：5mons
课程价钱：3000
'''

tea1.course.tell_info()
'''
课程名：linux运维
课程周期：5mons
课程价钱：800
'''
```

- 组合可以理解成多个人去造一个机器人，有的人造头、有的人造脚、有的人造手、有的人造躯干，大家都完工后，造躯干的人把头、脚、手拼接到自己的躯干上，因此一个机器人便造出来了

# 类的分类

## 新式类

- 继承了 object 的类以及该类的子类，都是新式类
- Python3 中所有的类都是新式类

## 经典类

- 没有继承 object 的类以及该类的子类，都是经典类
- 只有 Python2 中才有经典类

# 菱形继承问题

在 Java 和 C#中子类只能继承一个父类，而 Python 中子类可以同时继承多个父类，如 A(B,C,D)

如果继承关系为非菱形结构，则会按照先找 B 这一条分支，然后再找 C 这一条分支，最后找 D 这一条分支的顺序直到找到我们想要的属性

如果继承关系为菱形结构，即子类的父类最后继承了同一个类，那么属性的查找方式有两种：

- 经典类下：深度优先
- 广度优先：广度优先
- 经典类：一条路走到黑，深度优先

```
class G(object):
    # def test(self):
    #     print('from G')
    pass


print(G.__bases__)


class E(G):
    # def test(self):
    #     print('from E')
    pass


class B(E):
    # def test(self):
    #     print('from B')
    pass


class F(G):
    # def test(self):
    #     print('from F')
    pass


class C(F):
    # def test(self):
    #     print('from C')
    pass


class D(G):
    # def test(self):
    #     print('from D')
    pass


class A(B, C, D):
    def test(self):
        print('from A')


obj = A()



```

```python
(<class 'object'>,)
```

```python
obj.test()  # A->B->E-C-F-D->G-object
```

```py
from A
```

# C3 算法与 mro()方法介绍

python 到底是如何实现继承的，对于你定义的每一个类，python 会计算出一个方法解析顺序(MRO)列表，这个 MRO 列表就是一个简单的所有基类的线性顺序列表，如：

```python
print(A.mro())  # A.__mro__
'''
[<class '__main__.A'>, <class '__main__.B'>, <class '__main__.E'>, <class '__main__.C'>, <class '__main__.F'>, <class '__main__.D'>, <class '__main__.G'>, <class 'object'>]
'''


for i in A.mro():
    print(i)
'''
<class '__main__.A'>
<class '__main__.B'>
<class '__main__.E'>
<class '__main__.C'>
<class '__main__.F'>
<class '__main__.D'>
<class '__main__.G'>
<class 'object'>
'''
```

为了实现继承，python 会在 MRO 列表上从左到右开始查找基类，直到找到第一个匹配这个属性的类为止。

而这个 MRO 列表的构造是通过一个 C3 线性化算法来实现的。我们不去深究这个算法的数学原理，它实际上就是合并所有父类的 MRO 列表并遵循如下三条准则:

1. 子类会先于父类被检查
2. 多个父类会根据它们在列表中的顺序被检查
3. 如果对下一个类存在两个合法的选择，选择第一个父类
