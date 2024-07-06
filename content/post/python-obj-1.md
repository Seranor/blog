---
title: 类和对象、属性查找及绑定方法
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-obj-01.html
toc: true
---

# 类和对象

- 类的意思：种类、分类、类别

对象是特征与技能的结合体，我可能有身高体重、而你也有身高体重，所以你会说你像我，但是你一定不会说你像阿猫阿狗。并且我和你其实就可以说成是一类，而你和选课系统不能说是一类，因此给出类的定义：类就是一系列对象相似的特征与技能的结合体。

在现实世界中：先有一个个具体存在的对象，然后随着人类文明的发展才了分类的概念，既然现实世界中有类这个概念，Python 程序中也一定有类这个概念，但是在 Python 程序中：必须先定义类，然后调用类来产生对象。

<!-- more -->

## 现实世界中定义类和对象

### 定义对象

就拿未来我们的选课系统来讲，我们先总结一套现实世界中的学生对象：

- 对象 1：
  - 特征：
    - 学校=’oldboy’
    - 姓名=’耗哥’
    - 年龄=18
    - 性别=’male’
  - 技能：
    - 选课
- 对象 2：
  - 特征：
    - 学校=’oldboy’
    - 姓名=’猪哥’
    - 年龄=17
    - 性别=’male’
  - 技能：
    - 选课

### 定义类

站在未来选课系统的角度，我们还可以总结现实世界中的学生类：

- 老男孩学生类：
  - 相似的特征：
    - 学校=’oldboy’
  - 相似的技能
    - 选课

## 程序中定义类和对象

### 定义类

```python
# 注意类中定义变量使用驼峰体
class OldboyStudent():
    school = 'oldboy'

    def choose_course(self):
        print('is choosing course')

```

- 曾经定义函数，函数只检测语法，不执行代码，但是定义类的时候，类体代码会在类定义阶段就立刻执行，并且会产生一个类的名称空间，也就是说类的本身其实就是一个容器/名称空间，是用来存放名字的，这是类的用途之一

```python
print(OldboyStudent.__dict__)
'''
{'__module__': '__main__', 'school': 'oldboy', 'choose_course': <function OldboyStudent.choose_course at 0x10d653ae8>, '__dict__': <attribute '__dict__' of 'OldboyStudent' objects>, '__weakref__': <attribute '__weakref__' of 'OldboyStudent' objects>, '__doc__': None}
'''

print(OldboyStudent.__dict__['school'])  # oldboy
print(OldboyStudent.__dict__['choose_course'])  # <function OldboyStudent.choose_course at 0x10d653ae8>
try:
    OldboyStudent.__dict__['choose_course']()
except Exception as e:
    print('error:', e)

'''
error: choose_course() missing 1 required positional argument: 'self'
'''


print(OldboyStudent.school)  # oldboy
OldboyStudent.choose_course(111)  # is choosing course
print(OldboyStudent.choose_course)  # <function OldboyStudent.choose_course at 0x10d653ae8>
OldboyStudent.__dict__['choose_course']  # <function __main__.OldboyStudent.choose_course(self)>
OldboyStudent.country='China'
OldboyStudent.__dict__['country']  # 'China'

OldboyStudent.country='CHINA'
OldboyStudent.__dict__['country']  # 'CHINA'

del OldboyStudent.school

print(OldboyStudent.__dict__)
'''
{'__module__': '__main__', 'school': 'oldboy', 'choose_course': <function OldboyStudent.choose_course at 0x10d653ae8>, '__dict__': <attribute '__dict__' of 'OldboyStudent' objects>, '__weakref__': <attribute '__weakref__' of 'OldboyStudent' objects>, '__doc__': None, 'country': 'CHINA'}
'''
```

### 定义对象

- 调用类即可产生对象，调用类的过程，又称为类的实例化，实例化的结果称为类的对象/实例

```python
stu1=OldboyStudent() # 调用类会得到一个返回值，该返回值就是类的一个具体存在的对象/实例
print(stu1.school)  # oldboy

stu2=OldboyStudent() # 调用类会得到一个返回值，该返回值就是类的一个具体存在的对象/实例
print(stu2.school)  # oldboy

stu3=OldboyStudent() # 调用类会得到一个返回值，该返回值就是类的一个具体存在的对象/实例
stu3.choose_course() # is choosing course
```

# 定制对象独有特征

类中定义的函数是类的函数属性，类可以使用，但使用的就是一个普通的函数而已，意味着需要完全遵循函数的参数规则，该传几个值就传几个

## 引入

```python
class OldboyStudent:
    school = 'oldboy'

    def choose_course(self):
        print('is choosing course')


stu1 = OldboyStudent()
stu2 = OldboyStudent()
stu3 = OldboyStudent()
```

- 对于上述的学生类，如果类的属性改了，则其他对象的属性也会随之改变

```python
OldboyStudent.school = 'OLDBOY'
print(stu1.school)  # OLDBOY

print(stu2.school)  # OLDBOY
```

## 定制对象独有特征

```python
print(stu1.__dict__)  # {}

print(stu2.__dict__)  # {}
```

- 对象本质类似于类，也是一个名称空间，但是对象的名称空间存放对象独有的名字，而类中存放的是对象们共有的名字。因此我们可以直接为对象单独定制名字。

```python
stu1.name = 'tank'
stu1.age = 18
stu1.gender = 'male'

print(stu1.name, stu1.age, stu1.gender)
tank 18 male
try:
    print(stu2.name, stu2.age, stu2.gender)
except Exception as e:
    print(e)

# 'OldboyStudent' object has no attribute 'name'


stu2.name = 'sean'
stu2.age = 19
stu2.gender = 'female'

print(stu2.name, stu2.age, stu2.gender)  # sean 19 female
```

## 属性查找

- 首先从自身查找，没找到往类中找，类中没有则会报错。即对象的属性查找顺序为：自身–》类–》报错

## 类定义阶段定制属性

```python
def init(obj, x, y, z):
    obj.name = x
    obj.age = y
    obj.gender = z


init(stu1, 'tank1', 181, 'male')
print(stu1.name, stu1.age, stu1.gender)  # tank1 181 male

init(stu2, 'sean1', 191, 'female')
print(stu2.name, stu2.age, stu2.gender)  # sean1 191 female

```

- 使用上述方法虽然让我们定制属性更简单，但是还是太麻烦了，如果可以在实例化对象的时候自动触发定时属性，那就更方便了，因此可以使用类的**init**方法。

```python
class OldboyStudent:
    school = 'oldboy'

    def __init__(self, name, age, gender):
        """调用类的时候自动触发"""
        self.name = name
        self.age = age
        self.gender = gender
        print('*' * 50)

    def choose_course(self):
        print('is choosing course')


try:
    stu1 = OldboyStudent()
except Exception as e:
    print(e)

# __init__() missing 3 required positional arguments: 'name', 'age', and 'gender'


stu1 = OldboyStudent('lqz', 18, 'male')
# **************************************************
```

- 通过上述现象可以发现，调用类时发生两件事：
  1. 创造一个空对象
  2. 自动触发类中**init**功能的执行，将 stu1 以及调用类括号内的参数一同传入

```python
print(stu1.__dict__)  # {'name': 'lqz', 'age': 18, 'gender': 'male'}
```

# 属性查找

- 先从对象自己的名称空间找，没有则去类中找，如果类也没有则报错

```python
class OldboyStudent:
    school = 'oldboy'
    count = 0
    aa = 10

    def __init__(self, x, y, z):  #会在调用类时自动触发
        self.name = x  # stu1.name='耗哥'
        self.age = y  # stu1.age=18
        self.sex = z  # stu1.sex='male'
        OldboyStudent.count += 1
        #         self.count += 5
        self.aa = 1

    def choose_course(self):
        print('is choosing course')
print(OldboyStudent.count)  # 0

stu1 = OldboyStudent('nick', 18, 'male')
print(stu1.count)  #  1

stu2 = OldboyStudent('sean', 17, 'male')
print(stu2.count)  # 2

stu3 = OldboyStudent('tank', 19, 'female')
print(stu3.count)  # 3

print(OldboyStudent.count) # 3
print(stu1.name)  # nick

```

- 由于上述修改的是类属性，类属性的 count 已经被修改为 3，所以其他实例的 count 都为 3

```python
print(stu1.count)  # 3

print(stu2.count)  # 3

print(stu3.count)  # 4

```

- 由于 aa 是私有属性，因此 stu 们都会用自己私有的 aa，不会用类的 aa

```python
print(stu1.__dict__)  # {'name': 'nick', 'age': 18, 'sex': 'male', 'aa': 1}

print(stu2.__dict__)  # {'name': 'sean', 'age': 17, 'sex': 'male', 'aa': 1}

print(stu3.__dict__)  # {'name': 'tank', 'age': 19, 'sex': 'female', 'aa': 1}
```

# 对象的绑定方法

```python
class OldboyStudent:
    school = 'oldboy'

    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.sex = gender

    def choose_course(self):
        print(f'{self.name} choosing course')

    def func(self):
        print('from func')
```

- 类名称空间中定义的数据属性和函数属性都是共享给所有对象用的
- 对象名称空间中定义的只有数据属性，而且是对象所独有的数据属性

## 类使用对象的绑定对象

```python
stu1 = OldboyStudent('lqz', 18, 'male')
stu2 = OldboyStudent('sean', 17, 'male')
stu3 = OldboyStudent('tank', 19, 'female')

print(stu1.name)  # lqz
print(stu1.school)  # oldboy
```

- 类中定义的函数是类的函数属性，类可以使用，但使用的就是一个普通的函数而已，意味着需要完全遵循函数的参数规则，该传几个值就传几个

```python
print(OldboyStudent.choose_course)  # <function OldboyStudent.choose_course at 0x10558e840>


try:
    OldboyStudent.choose_course(123)
except Exception as e:
    print(e)

'''
'int' object has no attribute 'name'
'''
```

## 对象使用对象的绑定方法

- 类中定义的函数是共享给所有对象的，对象也可以使用，而且是绑定给对象用的，
- 绑定的效果：绑定给谁，就应该由谁来调用，谁来调用就会将谁当作第一个参数自动传入

```python
print(id(stu1.choose_course))  # 4379911304
print(id(stu2.choose_course))  # 4379911304
print(id(stu3.choose_course))  # 4379911304
print(id(OldboyStudent.choose_course))  # 4384680000

print(id(stu1.school))  # 4380883688
print(id(stu2.school))  # 4380883688
print(id(stu3.school))  # 4380883688


print(id(stu1.name), id(stu2.name), id(stu3.name))
# 4384509600 4384506072 4384507864

stu1.choose_course()  # lqz choosing course

stu2.choose_course()  # sean choosing course

stu3.choose_course()  # tank choosing course
```

- 补充：类中定义的函数，类确实可以使用，但其实类定义的函数大多情况下都是绑定给对象用的，所以在类中定义的函数都应该自带一个参数 self

```python
stu1.func()  # from func

stu2.func()  # from func
```
