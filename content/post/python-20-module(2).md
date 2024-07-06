---
title: python-模块(二)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-20.html
toc: true
---

### re 模块

> 在 python 中使用正则必须借助于模块，re 是其中之一

- `re.findall()`
  <!-- more -->

  ```python
  # 根据正则匹配所有符合条件的内容

  res = re.findall('t', 'test adsa dcxzawqd ')
  print(res)  # ['t', 't'] 匹配到有元素时结果是一个列表，没有匹配到时是一个空列表
  ```

- `re.search()`

  ```python
  # 根据正则匹配到一个符合条件的结束

  res = re.search('d', 'test adsa dcxzawqd ')
  print(res)  # <_sre.SRE_Match object; span=(6, 7), match='d'>
  print(res.group())  # d
  # 返回的是一个结果对象,想要获取值需要通过 group()

  res = re.search('o', 'test adsa dcxzawqd ')
  print(res)  # None
  print(res.group())  # 当没有匹配到值时用 group() 取值会报错

  # 可以使用判断是否取到值
  if res:
    	print('res.group()')
  else:
    	print('没匹配到值')
  ```

- `re.match()`

  ```python
  # 根据正则从头开始匹配,开头匹配上了就停止匹配

  res = re.match('a', 'abacad')
  print(res)  # <_sre.SRE_Match object; span=(0, 1), match='a'>
  print(res.group())  # a
  # 返回的也是一个结果对象,想获取值需要 group()


  res = re.match('a', 'bbacad')
  print(res)  # None
  print(res.group())  # 报错
  # 当没有匹配到时也会报错
  ```

- `re.split()`

  ```python
  # 先用 a 分割得到 '' 和 'bbcdd'
  # 再用 b 分割得到 '' '' 和 'bcdd'
  # 再用 b 分割得到 '' '' '' 'cdd'
  res = re.split('[ab]','abbcdd')
  print(res)  # ['', '', '', 'cdd']
  ```

- `re.sub()`

  ```python
  #类似于字符串类型的replace方法
  res1 = re.sub('\d','H','eva3jason4yuan4',1)  # 替换正则匹配到的内容
  res2 = re.sub('\d','H','eva3jason4yuan4')  # 不写默认替换所有
  print(res1)  # evaHjason4yuan4
  print(res2)  # evaHjasonHyuanH
  ```

- `re.subn()`

  ```python
  # 返回元组 并提示替换了几处
  res = re.subn('\d','H','eva3jason4yuan4',1)
  print(res)  # ('evaHjason4yuan4', 1)
  res = re.subn('\d','H','eva3jason4yuan4')
  print(res)  # ('evaHjasonHyuanH', 3)
  ```

- `re.compile()`

  ```python
  # 将正则表达式生成一个Pattern对象
  regexp_obj = re.compile('\d+')
  res1 = regexp_obj.search('absd213j1hjj213jk')
  res2 = regexp_obj.match('123hhkj2h1j3123')
  res3 = regexp_obj.findall('1213k1j2jhj21j3123hh')
  print(res1, res2, res3)  # <_sre.SRE_Match object; span=(4, 7), match='213'> <_sre.SRE_Match object; span=(0, 3), match='123'> ['1213', '1', '2', '21', '3123']
  print(res1.group(), res2.group())  # 213 123
  ```

- `re.finditer()`

  ```python
  # 将匹配到的内容存为一个迭代对象
  res = re.finditer('\d+', 'ashdklah21h23kj12jk3klj112312121kl131')
  print([i.group() for i in res]) # ['21', '23', '12', '3', '112312121', '131']
  ```

- 分组优先展示

  ```python
  # 无名分组
  # findall针对分组优先展示
  res = re.findall("^[1-9]\d{14}(\d{2}[0-9x])?$",'110105199812067023')
  print(res)  # ['023']
  # 取消分组优先展示
  res1 = re.findall("^[1-9](?:\d{14})(?:\d{2}[0-9x])?$",'110105199812067023')
  print(res1)  # ['110105199812067023']

  # 有名分组
  res = re.search('^[1-9](?P<xxx>\d{14})(?P<ooo>\d{2}[0-9x])?$','110105199812067023')
  print(res)
  print(res.group())  # 110105199812067023
  print(res.group(1))  # 10105199812067  无名分组的取值方式(索引取)
  print(res.group('xxx'))  # 10105199812067
  print(res.group('ooo'))  # 023
  ```

- 通过正则获取网页信息

  ```python
  import re
  import requests

  res = requests.get('http://www.redbull.com.cn/about/branch')
  if res.status_code == 200:
      with open(r'index.html', 'wb') as f:
          f.write(res.content)

  with open('index.html', 'r', encoding='utf8') as f:
      data = f.read()

  title_list = re.findall('<h2>(.*?)</h2>', data)
  address_list = re.findall("<p class='mapIco'>(.*?)</p>", data)
  zip_code_list = re.findall("<p class='mailIco'>(.*?)</p>", data)
  phone_list = re.findall("<p class='telIco'>(.*?)</p>", data)

  res = zip(title_list, address_list, zip_code_list, phone_list)
  # print(list(res))
  for data in res:
      print('''
          公司名称: %s
          公司地址: %s
          公司邮编: %s
          公司电话: %s
      ''' % (data[0], data[1], data[2], data[3]))
  ```

### collections 模块

> 该模块内部提供了一些高阶的数据类型

- ` namedtuple()`

  ```python
  # 具名元组
  from collections import namedtuple

  point = namedtuple('坐标', ['x', 'y'])
  res = point(11, 22)
  print(res)  # 坐标(x=11, y=22)
  print(res.x)  # 11
  print(res.y)  # 12

  card = namedtuple('扑克', '花色 点数')
  card1 = card('♠', 'A')
  card2 = card('♥', 'K')
  print(card1)  # 扑克(花色='♠', 点数='A')
  print(card2)  # 扑克(花色='♥', 点数='K')
  ```

- `deque()`

  ```python
  # 双端队列

  # 队列模块
      import queue  # 内置队列模块:FIFO
      # 初始化队列
      # q = queue.Queue()
      # 往队列中添加元素
      q.put('first')
      q.put('second')
      q.put('third')
      # 从队列中获取元素
      print(q.get())
      print(q.get())
      print(q.get())
      print(q.get())  # 值去没了就会原地等待

  # deque()
  		from collections import deque
      q = deque([11,22,33])
      q.append(44)  # 从右边添加
      q.appendleft(55)  # 从左边添加
      print(q.pop())  # 从右边取值
      print(q.popleft())  # 从做边取值
  ```

- `OrderedDict()`

  ```python
  # 有序字典
      # 无序的字典
      normal_dict = dict([('name', 'jason'), ('pwd', 123), ('hobby', 'study')])
      print(normal_dict)  # {'hobby': 'study', 'pwd': 123, 'name': 'jason'} 每次打印出来顺序都不一样

      #
      order_dict = OrderedDict([('name', 'jason'), ('pwd', 123), ('hobby', 'study')])
      print(order_dict)  # 打印结果顺序不变

      OrderedDict([('name', 'jason'), ('pwd', 123), ('hobby', 'study')])
      order_dict['xxx'] = 111
      print(order_dict)  # 添加的值在最后面

  ```

- `defaultdict()`

  ```python
  # 默认字典
  from collections import defaultdict

  values = [11, 22, 33,44,55,66,77,88,99,90]
  d = defaultdict(list)
  for i in values:
      if i > 60:
          d['k1'].append(i)
      else:
          d['k2'].append(i)
  print(d)  # defaultdict(<class 'list'>, {'k2': [11, 22, 33, 44, 55], 'k1': [66, 77, 88, 99, 90]})
  ```

- `Counter()`

  ```python
  # 统计字符出现的次数
  res = 'abcdeabcdabcaba'
  new_dict = {}
  for i in res:
      if i not in new_dict:
          new_dict[i] = 1
      else:
          new_dict[i] += 1
  print(new_dict)  # {'a': 5, 'b': 4, 'c': 3, 'd': 2, 'e': 1}

  # 使用Counter()
  from collections import Counter

  ret = Counter(res)
  print(ret)  # Counter({'a': 5, 'b': 4, 'c': 3, 'd': 2, 'e': 1})
  ```

### time 模块

时间的三种表现形式:

- 时间戳: 时间戳表示的是从 1970 年 1 月 1 日 00:00:00 开始按秒计算的偏移量
- 结构化时间: 元组(struct_time) 共九个元素:(年，月，日，时，分，秒，一年中第几周，一年中第几天等）
- 格式化时间: 格式化的时间字符串(Format String)： ‘1999-12-06’

```python
# 常用方法
time.sleep()  # 原地阻塞指定秒数
time.time()  # 获取当前时间戳
```

**python 中时间日期格式化符号**

```python
python中时间日期格式化符号：
%y 两位数的年份表示（00-99）
%Y 四位数的年份表示（000-9999）
%m 月份（01-12）
%d 月内中的一天（0-31）
%H 24小时制小时数（0-23）
%I 12小时制小时数（01-12）
%M 分钟数（00=59）
%S 秒（00-59）
%a 本地简化星期名称
%A 本地完整星期名称
%b 本地简化的月份名称
%B 本地完整的月份名称
%c 本地相应的日期表示和时间表示
%j 年内的一天（001-366）
%p 本地A.M.或P.M.的等价符
%U 一年中的星期数（00-53）星期天为星期的开始
%w 星期（0-6），星期天为星期的开始
%W 一年中的星期数（00-53）星期一为星期的开始
%x 本地相应的日期表示
%X 本地相应的时间表示
%Z 当前时区的名称
%% %号本身
```

**python 中结构化时间**

![cI9gOu](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/cI9gOu.png)

```python
>>> import time
>>> time.time()
1637840852.839533
>>> time.strftime('%Y-%m-%d %X')
'2021-11-25 19:47:34'
>>> time.strftime('%Y-%m-%d %H:%M:%S')
'2021-11-25 19:47:56'
>>> time.localtime()
time.struct_time(tm_year=2021, tm_mon=11, tm_mday=25, tm_hour=19, tm_min=50, tm_sec=35, tm_wday=3, tm_yday=329, tm_isdst=0)
```

几种格式之间的转换

![bEbO8P](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/bEbO8P.jpg)

### datetime 模块

```python
import datetime

print(datetime.date.today())  # 当前年月日
print(datetime.datetime.today())  # 当前年月日时分秒

res = datetime.datetime.today()
print(res.year)  # 返回当前年
print(res.month)  # 返回当前月
print(res.day)  # 返回当前日
print(res.weekday())  # 返回星期(0-6) 0代表周一
print(res.isoweekday())   # 返回星期(1-7) 1代表周一

# 时间差
ctime = datetime.datetime.today()
time_tel = datetime.timedelta(days=3)
print(ctime)  # 返回当前年月日时分秒
print(ctime - time_tel)  # 当前年月日时分秒往后推三天
print(ctime + time_tel)  # 当前年月日时分秒往前推三天

"""
日期对象 = 日期对象 +/- timedelta对象
timedelta对象 = 日期对象 +/- 日期对象
"""
ret = ctime + time_tel
print(ret - ctime)  # 3 days, 0:00:00
print(ctime - ret)  # -3 days, 0:00:00
```

python 标准库：https://docs.python.org/zh-cn/3.6/library/
