---
title: python-装饰器
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-14.html
toc: true
---

### 1. 装饰器定义

```python
装饰器由名称空间，函数对象，闭包函数组合而来

装饰:给被装饰对象添加额外的功能
器:指的是工具

装饰器的原则:开放封闭原则
  开放:对扩展开放
  封闭:对修改封闭

装饰器核心思想:在不改变被"装饰对象内部代码"和"原有调用方式"的基础上添加额外的功能
```

<!-- more -->

```python
eg:
  import time
  def index():
      time.sleep(3)  # 阻塞3秒
      print('form index')
  start_time = time.time()  # 记录函数运行之前的时间戳(1970年1月1日开始计算的秒数)
  index()  # 调用函数
  end_time = time.time()  # 记录函数运行之后的时间戳
  print(end_time - start_time)  # 计算出函数运行的时间值
```

### 2. 装饰器简易版

> 给函数添加统计执行时间的功能

```python
import time
def run_time(func):
    def r_time():
        start_time = time.time()
        func()
        end_time = time.time()
        print('程序运行时间为%s' % (end_time - start_time))
    return r_time
def for_loop():
    for i in range(100000):
        pass
def while_loop():
    i = 0
    while i < 100000:
        i += 1
for_loop = run_time(for_loop)  # 左侧的for_loop其实是r_time函数名，赋值给一个叫for_loop的变量名
for_loop()  # 只是在使用上感觉还是原来的函数名
while_loop = run_time(while_loop)
while_loop()
```

### 3. 装饰器参数问题

```python
import time
def outer(func):
    def run_time(*args, **kwargs):  # 将传入的位置参数组织成元组，关键字参数组织成字典
        start_time = time.time()
        func(*args, **kwargs)  # 将组织成元组的参数进行拆分成位置参数，将组织成字典的参数拆分成关键字参数
        end_time = time.time()
        print("程序运行时间%s" % (end_time - start_time))
    return run_time
def get_name(name):
    time.sleep(1)
    print("from index name is %s" % name)
def get_age(age):
    time.sleep(1)
    print("from func age is %s" % age)

get_name = outer(get_name)
get_name('jason')
get_age = outer(get_age)
get_age(18)
```

### 4. 装饰器返回值问题

```python
def outer(func):
    def run_time(*args, **kwargs):
        start_time = time.time()
        res = func(*args, **kwargs)  # res用于接收被装饰函数的返回值
        end_time = time.time()
        print("程序运行时间%s" % (end_time - start_time))
        return res  # 此时run_time函数的返回值就是被装饰函数的返回值了
    return run_time
def num_max(a, b):
    time.sleep(1)
    if a > b:
        return a
    else:
        return b
num_max = outer(num_max)
print(num_max(11, 22))
```

### 5. 认证装饰器

> 小功能实现

```python
# 每次运行都校验用户名和密码
def login(func):
    def auth():
        username = input('username:').strip()
        passwd = input('passwd:').strip()
        if username == 'jason' and passwd == '123':
            func()
        else:
            print("认证失败")
    return auth
def func1():
    print("功能1")
def func2():
    print("功能2")

func1 = login(func1)
func1()

func2 = login(func2)
func2()
```

```python
# 记录用户登录状态,只需要认证一次
login_flag = {'flag': False}  # 数据为可变类型，则函数名称空间内可以进行修改

def login(func):
    def auth():
        if login_flag.get('flag'):
            func()
        else:
            username = input("username:").strip()
            passwd = input("passwd:").strip()
            if username == 'jason' and passwd == '123':
                func()
                login_flag['flag'] = True
            else:
                print("认证失败")
    return auth
def func1():
    print("功能1")
def func2():
    print("功能2")

func1 = login(func1)
func1()

func2 = login(func2)
func2()
```

### 6. 装饰器固定模板

```python
def outer(func):
    def inner(*args, **kwargs):
        print('执行函数之前可以添加的额外功能')
        res = func(*args, **kwargs)  # 执行被装饰的函数
        print('执行函数之后可以添加的额外功能')
        return res  # 将被装饰函数执行之后的返回值返回
    return inner
```

### 7. 装饰器语法糖

```python
def outer(func):
    def inner(*args, **kwargs):
        print('执行函数之前可以添加的额外功能')
        res = func(*args, **kwargs)  # 执行被装饰的函数
        print('执行函数之后可以添加的额外功能')
        return res  # 将被装饰函数执行之后的返回值返回
    return inner
@outer  # index = outer(index)
def index(*args, **kwargs):
    print('from index')
@outer  # home = outer(home)
def home():
    print('from home')

"""
装饰器语法糖书写规范
    语法糖必须紧贴在被装饰对象的上方
装饰器语法糖内部原理
    会自动将下面紧贴着的被装饰对象名字当做参数传给装饰器函数调用
"""
```

### 8. 装饰器双层语法糖

```python
# 带参数带返回值的双层装饰器
login_flag = {'flag': False}
def all_time(func):
    def run_time(*args, **kwargs):
        start_time = time.time()
        res = func(*args, **kwargs)  # 接收的返回值是func1和func2的
        end_time = time.time()
        print("程序执行时间%s" % (end_time - start_time))
        return res
    return run_time
def login(func):
    def auth(*args, **kwargs):
        if login_flag.get('flag'):
            res = func(*args, **kwargs)
            return res
        else:
            username = input("username:").strip()
            passwd = input("passwd:").strip()
            if username == 'jason' and passwd == '123':
                res = func(*args, **kwargs)  # 接收的返回值是run_time的
                login_flag['flag'] = True
                return res
            else:
                print("用户名或密码错误")
    return auth

@all_time
@login
def func1(name):
    print("功能1%s" % name)
    time.sleep(1)
    return '功能1->%s' % name

@all_time
@login
def func2(name):
    print("功能2%s" % name)
    time.sleep(1)
    return '功能2->%s' % name

print(func1('购物'))
print(func2('付款'))

```

### 9. 装饰器三层语法糖

```python
# 判断七句print执行顺序
def outter1(func1):
    print('加载了outter1')
    def wrapper1(*args, **kwargs):
        print('执行了wrapper1')
        res1 = func1(*args, **kwargs)
        return res1
    return wrapper1

def outter2(func2):
    print('加载了outter2')
    def wrapper2(*args, **kwargs):
        print('执行了wrapper2')
        res2 = func2(*args, **kwargs)
        return res2
    return wrapper2

def outter3(func3):
    print('加载了outter3')
    def wrapper3(*args, **kwargs):
        print('执行了wrapper3')
        res3 = func3(*args, **kwargs)
        return res3
    return wrapper3


@outter1
@outter2
@outter3
def index():
    print('from index')

# 加载了outter3
# 加载了outter2
# 加载了outter1
# 执行了wrapper1
# 执行了wrapper2
# 执行了wrapper3
# from index
```

### 10. 装饰器修复技术

```python
from functools import wraps
def outer(func):
    @wraps(func)  # 修复技术就是为了让被装饰对象更加不容易被察觉装饰了
    def inner(*args, **kwargs):
        print('执行函数之前可以添加的额外功能')
        res = func(*args, **kwargs)  # 执行被装饰的函数
        print('执行函数之后可以添加的额外功能')
        return res  # 将被装饰函数执行之后的返回值返回
    return inner

@outer  # index = outer(index)
def index():
    print('from index')
print(index)
help(index)

def home():
    """这是一个home函数"""
    print('from home')

# help(index)
# help(home)
# print(index)
# help(len)
```

### 11. 有参装饰器

```python
# 通过第三层传值
def outer(source_data):
    # source_data = 'file'
    def login_auth(func):
        def auth(*args,**kwargs):
            # 2.校验用户名和密码是否正确
            # 数据的校验方式可以切换多种
            if source_data == 'file':
                # 从文件中获取用户数据并比对
                print('file文件获取')
            elif source_data == 'MySQL':
                # 从MySQL数据库中获取数据比对
                print('MySQL数据库获取')
            elif source_data == 'postgreSQL':
                # 从postgreSQL数据库中获取数据对比
                print('postgreSQL数据库获取')
            else:
                print('用户名或密码错误 无法执行函数')
        return auth
    return login_auth

@outer('file')
def index():
    print('from index')
@outer('MySQL')
def home():
    print('from home')

index()
home()
```
