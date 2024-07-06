---
title: python-编码和文件操作
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-09.html
toc: true
---

### 1. 字符编码

只跟文本和字符串有关
由于计算机内部只是别二进制,但是用户在使用计算机的时候却可以看到各种语言字符,字符编码就是内部记录了人类字符与数字对应关系的数据

<!-- more -->

#### 1.1 字符编码史

1. 一家独大

```python
计算机由美国发明,因此美国人为了能让计算机识别英文字符诞生了ASCII码表
特点:
  只有英文字符与数字的一一对应关系
  一个英文字符对应1Bytes,1Bytes=8bit,8bit最多包含256个数字,可以对应256个字符,足够表示所有的英文字符,目前只用到127个,剩下的为了后续发现新的语言
需要记住的是:
  A-Z: 65-90
  a-z: 97-122
```

![L7G4SM](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/L7G4SM.jpg)

1. 群雄割据

```python
中国:
  GBK码:记录了英文中文与数字的对应关系
        对于英文还是使用一个字节
        中文使用了两个字节甚至更多字节,两个字节也不能够全部表示出所有的中文,需要生僻字需要更多位

日本:
  shift_JIS码:记录了日文英文与数字的对应关系
韩国
  Euc_kr码:记录了韩文英文与数字的对应关系
```

1. 分久必合

```python
为了能够实现不同国家之间的文本数据能够彼此无障碍交流需要对编码统一
unicode(万国码)出现了
  特点:统一使用两个及以上字符记录字符与数字的对应关系

utf8(万国码的优化版)
  英文还是用一个字节存储,中文使用三个字节或更多字节存储

现在默认使用的编码是uft8
```

#### 1.2 编码操作

1. 如何解决文件乱码

```python
文件当初以什么编码编的,打开的时候就以什么编码解
```

2. python 解释器不同版本的编码差异

```python
python2.x内部使用的编码默认是ASCII
  1.文件头
  # coding:utf8

  2.在python2中定义字符串前面需要加一个u
    s = u'你'

python3.x内部使用utf8
```

3. Pycharm 定义文件模板内容

![uJTsDP](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/uJTsDP.png)

4. 编码与解码

```python
编码
  将人类能够读懂的字符按照指定的编码转换成数字
解码
  将数字按照指定的编码转换成人类能够读懂的字符

eg:
 # 编码
   s = '这是一段文字'
   res = s.encode('utf8')
   print(res, type(res))  # b'\xe8\xbf\x99\xe6\x98\xaf\xe4\xb8\x80\xe6\xae\xb5\xe6\x96\x87\xe5\xad\x97' <class 'bytes'>

 # 解码
  res1 = res.decode('utf8')
  print(res1)  # 这是一段文字
```

### 2. 文件

```python
文件其实是操作系统暴露给用户操作硬盘的接口
```

#### 2.1 文件操作

##### 2.1.1 如何操作文件

```python
关键字open()
    1.open()打开文件
    2.其他方法操作文件
    3.关闭文件
```

##### 2.1.2 路径斜杠

```python
在路径中出现字母与斜杠的组合产生了特殊含义如何取消
在路径字符串前面加一个r
  r'D:\py20\day08\a.txt'
```

##### 2.1.3 操作文件

```python
格式:
  open(文件路径,读写模式,字符编码)
       文件路径与读写模式是必须的
       字符编码是可选的(有些模式需要编码)

  eg:
    res = open('a.txt', 'r', encoding='utf8')
    print(res.read())
    res.close()
```

##### 2.1.4 with 上下文管理

```python
可以自动close()
eg:
   with open(r'a.txt', 'r', encoding='utf8') as f1:
       print(f1.read())
```

#### 2.2 读写模式

##### 2.2.1 只读模式 r

```python
只能查看不能修改
# 当路径不存在时,直接报错
with open(r'b.txt','r',encoding='utf8') as f1:
    pass  # 运行代码报错

# 当路劲存在时,读取没有问题,写操作时报错
with open(r'a.txt','r',encoding='utf8') as f1:
    print(f1.read())  # 能读取文件内容
    f1.write('123')  # 报错,无法写入
```

##### 2.2.2 只写模式 w

```python
# 当路劲不存在时,不会报错,会创建该文件
with open(r'b.txt', 'w', encoding='utf8') as f2:
    pass

# 路劲存在时,写入会先清空文件内容,再写入内容
with open(r'a.txt','w',encoding='utf8') as f1:
    print(f1.read())  # 读取会报错
    f1.write('123')  # 写入的都会在一行,不会自动换行
    f1.write('\n123\n')  # 需要加入换行符
```

##### 2.2.3 只追加模式 a

```python
# 当路劲不存在时,不会报错,同样会创建该文件
  with open(r'c.txt', 'a', encoding='utf8') as f3:
      pass

# 当路劲存在时,写入不会清空文件
  with open(r'a.txt', 'a', encoding='utf8') as f1:
      f1.write('\nwoooooo')  # 需要加入换行符,否则都会在一行
      f1.write('\nwoooooo')
      print(f1.read())  # 读取会报错

# r w a读写模式都只能操作文本文件
```

### 3. debug 代码调试

1. 在代码右侧使用右键标记，空白处右键出现在 Run 下面有 Debug 运行

![JSfh9B](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/JSfh9B.png)

2. debug 运行的时候会一步步执行,并给出每一步的结果

![AMmZCZ](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/AMmZCZ.png)

3. 停止 debug

![BrFJ9l](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/BrFJ9l.png)

取消小点

![xfnoQa](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/xfnoQa.png)
