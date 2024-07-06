---
title: python-流程控制
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-05.html
toc: true
---

## 流程控制

### 1. 定义及说明

```python
流程控制即控制事物的执行过程
任何使用执行流程只有三种情况:
  1.顺序结构  # 自上而下依次运行
  2.分支结构  # 在运行过程中根据条件的不同可能会执行不同的流程
  3.循环结构  # 在运行过程中有些代码需要反复执行
```

<!-- more -->

```python
1.条件都会转成布尔值  从而决定子代码是否执行
2.在python中 使用缩进来表示代码的从属关系
3.并不是所有的代码都可以拥有子代码
4.同属于某个代码的多行子代码 必须要保持相同的缩进量
	在python中推荐使用四个空格来缩进
ps:小技巧 上一行代码的结尾如果是冒号 那么下一行代码必缩进
```

### 2. 顺序结构

```python
从上到下依次执行
```

![T8iFeL](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/T8iFeL.png)

### 3. 分支结构

#### 3.1 if 基本使用

##### 3.1.1 if 单分支

```python
格式:
  if 条件:
    条件成立之后执行的子代码块
eg:
  age = 20
  if age < 28:
    print("小姐姐好")
```

![sizMEI](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/sizMEI.png)

##### 3.1.2 if-else 使用

```python
格式:
  if 条件:
    条件成立之后执行的子代码块
  else:
    条件不成立执行的子代码块

 # if与else连用情况下,两者子代码永远只会执行一个
eg:
  age = 20
  if age < 28:
    print('小姐姐好')
  else:
    print('认错人了')
```

![alOv1n](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/alOv1n.png)

##### 3.1.3 if-elif-else 使用

```python
格式:
  if 条件1:
    条件1成立之后执行的子代码块
  elif 条件2:
    条件1不成立 条件2成立之后执行的子代码块
  elif 条件3:
    条件1和2都不成立 条件3成立之后执行的子代码块
  ...
  else:
   上述条件都不成立 执行的子代码块
  # elif 可以有多个,三者连用也只会执行其中一个子代码块
eg:
    score = 79
    if score > 90:
        print('优秀')
    elif score > 80:
        print('良好')
    elif score > 70:
        print('一般')
    elif score 78> 60:
        print('及格')
    else:
        print('挂科重修')
```

![GYHxdd](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/GYHxdd.png)

#### 3.2 if 嵌套使用

```python
格式:
if 条件 1:
	条件 1 成立执行的代码
	if 条件 2:
		条件 2 成立执行的代码
eg:
  age = 26
  height = 165
  weight = 99
  is_beautiful = True
  is_success = True
  if age < 28 and height > 160 and weight < 100 and is_beautiful:
      print('小姐姐能否加个微信')
      # 判断小姐姐是否会给微信
      if is_success:
          print('吃饭 看电影 天黑了...')
      else:
          print('去你妹的 变态!')
  else:
      print('可惜了')
```

![voz9xF](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/voz9xF.png)

#### 3.3 小练习

1.编写一个用户登录功能 ,用户名是 jason, 密码是 123,用户如果输入正确则打印来宾三位,否则登录失败

```python
# 定义默认用户名和密码
NAME = 'jason'
PASSWD = 123

# 将用户输入的用户名和密码传给username和passwd
username = input("请输入用户名:")
passwd = int(input("请输入密码:"))

# 判断用户输入的用户名和密码是否和定义默认的用户名密码相同
if NAME == username and PASSWD == passwd:
    print("来宾三位")  # 如果相同,则打印来宾三位
else:
    print("登录失败")  # 其中一个不同都会显示登录失败
```

![Ma5Orf](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/Ma5Orf.png)

2.根据用户名的不同打印不同的用户身份
jason 管理员 tony 安保人员 kevin 财务 jack 销售 其他普通员工

```python
# 将用户输入的用户名给变量username,然后判断
username = input("请输入用户名:")
if username == "jason":
    print("管理员")
elif username == 'tony':
    print("安保人员")
elif username == 'kevin':
    print("财务")
elif username == 'jack':
    print("销售")
else:
    print("普通员工")
```

![eqPxPs](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/eqPxPs.png)

### 4. 循环结构

#### 4.1 while 循环

```python
格式:
  while 条件:
    条件成立之后循环执行的子代码
eg:
  while True:
      # 1.获取用户输入的用户名和密码
      username = input('username>>>:')
      password = input('password>>>:')
      # 2.判断用户名和密码是否正确
      if username == 'jason' and password == '123':
          print('来宾三位')
      else:
          print('登录失败')
 # 含义:这段代码执行后,当用户输入用户名密码进行判断是否是jason和123,不管用户输入对错与否,都会一直执行下去,因为True一直成立,是个死循环
```

![7bpWCy](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/7bpWCy.png)

#### 4.2 while-break

```python
break:结束本层循环
eg:
  while True:
     # 1.获取用户输入的用户名和密码
     username = input('username>>>:')
     password = input('password>>>:')
     # 2.判断用户名和密码是否正确
     if username == 'jason' and password == '123':
         print('来宾三位')
         break  # 直接跳出本层循环
     else:
         print('登录失败')
```

#### 4.3 全局标志位

```python
# 标志位的使用
flag = True
while flag:
    # 1.获取用户输入的用户名和密码
    username = input('username>>>:')
    password = input('password>>>:')
    # 2.判断用户名和密码是否正确
    if username == 'jason' and password == '123':
        print('来宾三位')
        while flag:
            cmd = input('请输入您的指令>>>:')
            # 判断用户是否想退出
            if cmd == 'q':
                flag = False
            print('正在执行您的指令:%s' % cmd)
    else:
        print('去你妹的 没钱滚蛋')
```

### 5. 练习

```python
猜年龄的游戏
	1.要求1
    	用户可以有三次猜错的机会 如果过程中猜对了直接退出
  2.要求2
    	三次机会用完之后提示用户是否继续尝试 如果是则再给三次机会 如果否则直接结束

    # 数据类型转换提示
		age = input('age>>>:')
    real_age = 18
    # 将字符串的数字转换成整型
    age = int(age)
```

```python
# 要求1:
AGE = 26
count = 0

while count < 3:
    age = int(input("请输入你猜的年龄:"))

    if age == AGE:
        print("猜对了,年龄是%s" % age)
        break
    else:
        print("猜错了")
    count += 1
```

```python
# 要求2:
AGE = 26
count = 0
while count < 3:
    count += 1
    age = int(input("请输入你猜的年龄: "))
    if age == AGE:
        print("猜对了,年龄是%s" % age)
        break
    elif count == 3:
        while True:
            again = input("已经答错三次,是否继续三次Y/N: ")
            if again == 'Y':
                count = 0
                break
            elif again == 'N':
                break
            else:
                print("输入正确的字符")
    else:
        print("答错了请继续!")
```
