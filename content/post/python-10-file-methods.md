---
title: python-文件操作(二)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-10.html
toc: true
---

### 1. 文件操作方法

#### 1.1 读方法

```python
with open(r'a.txt','r', encoding='utf8') as f:
    print(f.read())  # 一次性读取文件所有的内容
    print(f.readline())  # 每次值读文件一行内容
    print(f.readlines())  # 读取文件所有内容,组织成列表,每个元素是文件的每行内容
    print(f.readable())  # 判断当前文件是否可读,返回布尔值
```

<!-- more -->

#### 1.2 写方法

```python
with open(r'a.txt', 'w', encoding='utf8') as f:
    f.write('这是写入的内容')  # 向文件写入内容
    f.write(123)  # 写入的内容必须是字符串,这句会报错
    f.writelines(['jason', 'tom', 'Bob'])  # 将列表中的每个字符串元素写入文件
    print(f.writable())  # 判断忘记拿是否可写
    f.flush()  # 将内存文件数据刷到硬盘
```

### 2. 文件优化操作

```python
with open(r'a.txt','r',encoding='utf8') as f:
    print(f.read())

read() 会一次性读取文件内所有的内容
弊端:
  1.一次性读完内容之后,光标停留在文件末尾,无法再次读取内容
  2.该方法在读取大文件的时候,可能会造成内存溢出的情况

解决:逐行读取文件内容
  with open(r'a.txt', 'r', encoding='utf8') as f:
    for line in f:
        print(line)

 # 涉及到文件多行内容读取的情况一般采用for循环读取
```

### 3. 文件操作模式

#### 3.1 文本模式

```python
t 文本模式
1.默认的模式, r w a 其实是rt wt at
2.该模式所有操作都是以字符串为基本单位(文本)
3.该模式必须要指定encoding参数
4.该模式只能操作文本文件
```

#### 3.2 二进制模式

```python
b 二进制模式
1.该模式可以操作任意类型的文件
2.该模式所有操作都是以bytes类型(二进制)基本单位
3.该模式不需要指定encoding参数
指定模式的时候就需要rb wb ab

eg:
  读:
    with open(r'a.txt', 'rb') as f:
        data = f.read()
        print(data, type(data))  # 英文正常显示,中文是一串字符, 类型是bytes

  写:
    with open(r'a.txt', 'ab') as f:
    data = '您好'
    res = data.encode('utf8')  # 将中文进行编码
    f.write(res)  # 编码之后再写入,文本中正常写入
```

### 4. 练习

#### 4.1 文件版登录注册功能

注册功能

```python
# 要求用户输入用户名和密码
username = input("请输入用户名>>>: ").strip()
password = input("请输入密码>>>: ").strip()
# 格式化输入的信息,添加分隔符和换行符
msg = "%s|%s\n" % (username, password)
with open(r'user.txt', 'r', encoding='utf8') as f_reg:
    # for循环读取文件内容
    for line in f_reg:
      # 取到用户名的值
      name = line.split("|")[0]
      # 如果用户输入的用户名与存在的用户名一致就退出
      if name == username:
          print("用户名已存在!")
          break
       else:
          # 否则就将用户输入的信息写入到文件
          with open(r'user.txt', 'a', encoding='utf8') as f_reg_insert:
              f_reg_insert.write(msg)
          print("%s注册成功" % username)
```

登录功能

```python
# 接收输入的用户名和密码
name = input("请输入用户名>>>: ").strip()
pwd = input("请输入密码>>>: ").strip()
with open(r'user.txt', 'r', encoding='utf8') as f_log:
    # for循环读取文件每行内容
    for line in f_log:
        # 将文件中的用户名与密码处理出来并解压赋值
        u_name, passwd = line.split("|")
        # 判断用户输入的用户名密码和文件存在的用户密码是否相等,密码后面有换行符也进行处理
        # 如果相等就登录成功,否则就显示登录失败
        if u_name == name and passwd.strip("\n") == pwd:
            print("登录成功!")
            break
        else:
            print("登录失败,用户名或密码错误!")
```

整体功能实现

```python
while True:
    print('''
        1.用户注册
        2.用户登录
        3.退出系统
    ''')
    # 接收用户想要的功能编号
    options = input("请输入选项>>>: ").strip()
    # 当编号为1的时候,实现用户注册功能
    if options == '1':
        # 要求用户输入用户名和密码
        username = input("请输入用户名>>>: ").strip()
        password = input("请输入密码>>>: ").strip()
        # 格式化输入的信息,添加分隔符和换行符
        msg = "%s|%s\n" % (username, password)
        with open(r'user.txt', 'r', encoding='utf8') as f_reg:
            # for循环读取文件内容
            for line in f_reg:
                # 取到用户名的值
                name = line.split("|")[0]
                # 如果用户输入的用户名与存在的用户名一致就退出
                if name == username:
                    print("用户名已存在!")
                    break
            else:
                # 否则就将用户输入的信息写入到文件
                with open(r'user.txt', 'a', encoding='utf8') as f_reg_insert:
                    f_reg_insert.write(msg)
                print("%s注册成功" % username)
    # 当用户输入编号为2时,就是登录功能
    elif options == '2':
        # 接收输入的用户名和密码
        name = input("请输入用户名>>>: ").strip()
        pwd = input("请输入密码>>>: ").strip()
        with open(r'user.txt', 'r', encoding='utf8') as f_log:
            # for循环读取文件每行内容
            for line in f_log:
                # 将文件中的用户名与密码处理出来并解压赋值
                u_name, passwd = line.split("|")
                # 判断用户输入的用户名密码和文件存在的用户密码是否相等,密码后面有换行符也进行处理
                # 如果相等就登录成功,否则就显示登录失败
                if u_name == name and passwd.strip("\n") == pwd:
                    print("登录成功!")
                    break
            else:
                print("登录失败,用户名或密码错误!")
    # 当用户输入3就退出整个循环
    elif options == '3':
        print("退出系统!!!")
        break
    else:
        print("输入的选项不正确,请重新输入!")
```

#### 4.2 简易拷贝功能

```python
src_path = input('源文件绝对路径: ').strip()
dst_path = input('目标文件绝对路径: ').strip()

with open(r'%s' % src_path, 'rb') as f:
    with open(r'%s' % dst_path, 'wb') as copy_f:
        for line in f:
            copy_f.write(line)
```
