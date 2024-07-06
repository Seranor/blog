---
title: python-模块(一)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-18.html
toc: true
---

### 1. 面向过程编程

面向过程编程，核心是过程二字，过程指的是解决问题的步骤，即先干什么、后干什么、再干什么、然后干什么……

基于该思想编写程序就好比在设计一条流水线，面向对称编程其实是一种机械式的思维方式

优点: 复杂的问题流程化，进而简单化

缺点: 一旦要修改功能 那么需要整体改造(牵一发而动全身)

<!-- more -->

```python
# 用户注册功能
# 1.获取用户名和密码
# 2.组织成固定的格式
# 3.文件操作写入文件

def get_info():
    username = input('请输入用户名: ').strip()
    passwd = input('请输入密码: ').strip()
    if len(username) == 0 or len(passwd) == 0:
        print('用户名密码不能为空')
        return
    id_msg = {'1': 'admin',
              '2': 'user'}
    u_id = input('%s\n请输入身份: ' % id_msg).strip()
    if u_id in id_msg:
        user_id = id_msg.get(u_id)
        return deal_data(username, passwd, user_id)
    else:
        print('输入的信息不合法')


def deal_data(username, passwd, u_id):
    msg = '%s|%s|%s\n' % (username, passwd, u_id)
    return save_data(msg)


def save_data(msg):
    with open(r'info.txt', 'a', encoding='utf8') as f:
        f.write(msg)


get_info()
```

### 2. 模块简介

1. **什么是模块**

   模块是一系列功能的结合体

2. **为什么要用模块**

   为了提升开发效率

3. **模块的三种来源**

   1. 内置: Python 解释器自带的，能直接导入使用
   2. 第三方: 别人已经写好的，下载后可以直接拿来用
   3. 自定义: 自己写的模块

4. **模块的四种表现形式**

   1. 使用 Python 编写的代码(.py 文件)
   2. 已被编译为共享库活 DLL 或 C++扩展
   3. 包好一组模块的包(文件夹)，其实是多个 py 文件的集合，包内通常用`__init__.py`文件
   4. 使用 C 编写并链接到 Python 解释器的内置模块

PS: 在编写大型项目的时候，遇到一些复杂的功能可以先考虑是否有相应的模块可以调用

### 3. 模块的导入

#### 3.1 import

- 在同级目录下创建两个.py 文件

  mod-imp.py

  ```python
  import imtest

  money = 999
  print(imtest.money)  # 1000
  print(imtest.func1())  # from func1
  imtest.change()
  print(imtest.money)  # 10
  print(money)  # 999

  # 执行该文件时会首先打印 imtest模块
  ```

  imtest.py

  ```python
  print('imtest模块')
  money = 1000


  def func1():
      print('from func1')


  def change():
      global money
      money = 10
  ```

- 结论
  1. 多次导入相同模块时，只会执行一次
  2. 首次导入`imtest`模块的过程
     1. 运行导入文件(import 句式.py)产生该文件的全局名称空间
     2. 运行`imtest.py`文件
     3. 运行`imtest.py`内代码，将产生的名字全部存档于`imtest.py`名称空间
     4. 在导入文件名称空间产生一个`imtest`的名字指向`imtest.py`全局名称空间
  3. import 句式导入模块之后
     1. 通过`模块名.`的方式可以使用模块内所有的名字，并且肯定不会产生冲突

#### 3.2 from...import...

```python
from imtest import money, func1, change
```

1. 多次导入相同模块是，只会执行一次
2. 导入发生的过程
   1. 先产生执行文件的全局名称空间
   2. 执行模块文件，产生模块的全局名称空间
   3. 将模块中执行之后产生的名字全部存档于模块名称空间中
   4. 在执行文件中有一个`money`执行模块名称空间中`money`指向的值
3. 导入之后
   1. 在使用的时候直接写名字即可，但是当当前名称空间有相同名字的时候，就会产生冲突，使用的就变成了当前名称空间

#### 3.3 导入模块扩展用法

1. 起别名

   既可以给模块起别名也可以给模块中的某个起别名

   ```python
   import modisverylonglong  as m
   from modisverylonglong import name as n
   ```

2. 连续导入

   可以连续导入多个模块，但是只有当多个模块功能相似或属于同一系列，否则推荐分行导入

   ```python
   import mod1, mod2
   from mod1 import name1, name2
   ```

3. 通用导入

   将模块中所有名字全部导入

   ```python
   from mod import *  # * 表示所有
   ```

   ```python
   __all__ = ['name1', 'name2']  # 在被导入的模块文件中可以使用该方法指定可以被导入的名字,限制的是 *
   ```

#### 3.4 判断文件类型

- 判断 py 文件是作为模块文件还是执行文件

  `__name__`当文件是执行文件时会返回`__main__`

  文件被当做模块导入则返回文件名(模块名)

  执行 mod-imp.py 时

  ```python
  import imtest
  print(__name__)  # __main__
  print(imtest.__name__)  # imtest
  ```

  执行 imtest.py 时

  ```python
  print('imtest模块')
  money = 1000
  def func1():
      print('from func1')
  def change():
      global money
      money = 10
  print(__name__)  # __main__
  ```

- 应用

  ```python
  if __name__ == '__main__':
    func1()  # 可以在这里放测试代码，避免在别模块导入之后执行
  ```

  ps: 在 Pycharm 中打出 main 之后按 tab 自动补全

#### 3.5 循环导入

```python
当出现循环导入的情况时，程序设计不合理。所以在编写程序时不能出现循环导入现象
```

- 现象:

a.py

```python
from b import num_b
num_a = 10
```

b.py

```python
from a import num_a
num_b = 20
```

mod-imp.py

```python
from a import num_a  # 此时会出现异常
```

- 解决方案:
  1. 调换顺序，将彼此调用的句式放在代码的最后
  2. 函数形式，将导入的句式放入到函数体代码，等待所有的名字加载完毕之后再调用(本质等同于调换顺序)

#### 3.6 模块导入的顺序

- 查找顺序

  1. 先从内存中查找

  2. 再去内置模块中查找

  3. 最后去`sys.path`系统路劲中查找(自定义模块)

     如果都没找到就会报错

  ```python
  import sys
  print(sys.path)  # 结果中你的第一个元素永远是当前执行文件的路径
  ```

- 当自定义模块查找不到的时候解决方案

  1. 手动将该模块的路径添加到`sys.path`中

     ```python
     import sys
     sys.path.append(r'path')
     ```

  2. 使用`from...import...`

     1. `from` 文件夹名称.文件夹名称 `import` 模块名
     2. `from` 文件夹名称.模块名称 `import ` 名字

#### 3.7 绝对导入与相对导入

```python
"""在程序中涉及到多个文件之间导入模块的情况 一律按照执行文件所在的路径为准"""
绝对导入
	始终按照执行文件所在的sys.path查找模块
相对导入
	"""
	句点符(.)
		.表示当前文件路径
		..表示上一层文件路径
	"""
    能够打破始终以执行文件为准的规则 只考虑两个文件之间的位置
    # 相对导入只能用在模块文件中 不能在执行文件中使用
```
