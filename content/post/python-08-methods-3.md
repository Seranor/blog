---
title: python-内置方法(三)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-08.html
toc: true
---

### 1. 列表其他方法

#### 1.1 排序

```python
l1 = [33, 22, 77, 99, 11, 88, 44, 55]
1.sort()  # 默认是升序
  l1.sort()
  print(l1)  # [11, 22, 33, 44, 55, 77, 88, 99]
```

<!-- more -->

```python
2.sort(reverse=True)  # 降序
  l1.sort(reverse=True)
  print(l1)  # [99, 88, 77, 55, 44, 33, 22, 11]

3.revers()  # 顺序颠倒
  l1.reverse()
	print(l1)  # [55, 44, 88, 11, 99, 77, 22, 33]
```

#### 1.2 切片

```python
l1 = [33, 22, 77, 99, 11, 88, 44, 55]

print(l1[1:3])  # [22, 77]
print(l1[:])  # [33, 22, 77, 99, 11, 88, 44, 55]
print(l1[3:])  # [99, 11, 88, 44, 55]
print(l1[:3])  # [33, 22, 77]
```

#### 1.3 比较

```python
l1 = [99, 22]
l2 = [88, 44, 33]
print(l1 > l2)  # True
# 列表比较运算采用相同索引的元素进行比较,只要有一个比出了结果就直接得出结论

s1 = 'hello'
s2 = 'world'
print(s1 > s2)  # False
# 字符串比较也会根据索引位置内部转成ASCII对应的数字进行比较
```

### 2. 字典内置方法

#### 2.1 取值

```python
dic = {
    'name': 'jason',
    'age': 18,
    'hobbies': ['play game', 'basketball']
}

1.按K取值,K不存在会直接报错,不太推荐此方式
  print(dic['name'])  #	jason
  print(dic['pwd'])  # 报错

2.get() 键值不存在返回None,不会报错
  print(dic.get('xxx'))  # None  键不存在 不会报错返回None
  print(dic.get('name', '哈哈哈'))  # 第二个参数 可以在k不存在的时候自定义返回信息
  print(dic.get('xxx', '哈哈哈'))  # 第二个参数 可以在k不存在的时候自定义返回信息
```

#### 2.2 修改值

```python
1.键值存在则修改
dic['name'] = 'jasonxx'
print(dic)  # {'name': 'jasonxx', 'age': 18, 'hobbies': ['play game', 'basketball']}

2.键不存在就新增键值对
dic['pwd'] = 123
print(dic)  # {'name': 'jason', 'age': 18, 'hobbies': ['play game', 'basketball', 'read'], 'pwd': 123}

3.hobbies的 V 是一个列表,可以用append()为列表增加一个值
dic['hobbies'].append('read')
print(dic)  # {'name': 'jason', 'age': 18, 'hobbies': ['play game', 'basketball', 'read']}
```

#### 2.3 统计

```python
统计字典内部键值对的个数
print(len(dic))  # 3
```

#### 2.4 成员运算

```python
print('name' in dic)  # True
print('jason' in dic)  # False
# 默认只暴露K
```

#### 2.5 删除元素

```python
1.方式1 del 根据K删除键值对
  del dic['name']
  print(dic)  # {'age': 18, 'hobbies': ['play game', 'basketball']}

2.方式2 弹出指定K的键值对pop()
  print(dic.pop('age'))  # 18
  print(dic)  # {'name': 'jason', 'hobbies': ['play game', 'basketball']}

3.方式3 直接弹出键值对,组织成元组的形式,第一个元素K,第二个元素是V
  print(dic.popitem())  # ('hobbies', ['play game', 'basketball'])
  print(dic)  # {'name': 'jason', 'age': 18}
```

#### 2.6 取值

```python
1.keys()
print(dic.keys())  # dict_keys(['name', 'age', 'hobbies'])  获取字典所有的键 看成列表即可

2.values()
print(dic.values())  # dict_values(['jason', 18, ['play game', 'basketball']])  获取字典所有的值 看成列表即可

3.items()
print(dic.items())  # dict_items([('name', 'jason'), ('age', 18), ('hobbies', ['play game', 'basketball'])])
# 获取字典里面所有的键值对 组织成列表套元组的形式 元组内有两个元素 第一个是k第二个是v
```

#### 2.7 更新字典

```python
update()  键存在则修改 不存在则创建
dic.update({'name': 'jasonNB', 'pwd': 123})
print(dic)  # {'name': 'jasonNB', 'age': 18, 'hobbies': ['play game', 'basketball'], 'pwd': 123}
```

#### 2.8 初始化字典

```python
print(dict.fromkeys(['k1', 'k2', 'k3'], []))
'''笔试题'''
res = dict.fromkeys(['k1', 'k2', 'k3'], [])
res['k1'].append(111)
res['k2'].append(222)
res['k3'].append(333)
# V 是相同的一个列表,对该列表操作,V的值是一样的

# 当对k1对应的V重新赋值后,就会被单独出来了
res['k1'] = [111,222,333]
res['k1'].append(444)
print(res)
```

#### 2.9 setdefault()

```python
当键存在的情况下 不修改而是获取该键对应的值
# print(dic.setdefault('name', 'jasonNB'))
# print(dic)
# 当键不存在的情况下 新增一组键值对 并且该方法的结果是新增的值
print(dic.setdefault('pwd', '123'))
print(dic)
```

### 3. 元组内置方法

#### 3.1 类型转换

```python
1.能够支持for循环的数据都可以转换成元组
print(tuple('hello'))  # ('h', 'e', 'l', 'l', 'o')
print(tuple([11,22,33]))  # (11, 22, 33)
print(tuple({'name':'jason','pwd':123}))  # ('name', 'pwd')

2.元组类型的定义
t1 = (111)  # 整型
t2 = (11.11)  # 浮点型
t3 = ('hello')  # 字符串
t1 = (11, )  # 第一个元素后一定需要加逗号才会被定义为元组,否则就不是元组类型
```

#### 3.2 取值

```python
t = (111, 222, 333, 444, 555)
print(t[1])  # 222
print(t[-1])  # 555
```

#### 3.3 切片

```python
print(t[1:3])  # (222, 333)
print(t[1:5:2])  # (222, 444)
```

#### 3.4 统计元素个数

```python
print(len(t))  # 5
```

#### 3.5 for 循环取值

```python
for i in t:
		print(i)
```

#### 3.6 计数

```python
count()
print(t.cont(111))  # 1  111只出现一次
```

### 4. 集合操作

#### 4.1 类型转换

```python
能够支持for循环的数据类型都可以转成集合(元素要是不可变类型)
集合内元素是无序的

s1 = set()
```

#### 4.2 去重

```python
s1 = {1, 2, 2, 2, 3, 4, 3, 4, 3, 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3}
print(s1)  # {1, 2, 3, 4}

1.去重练习1
name_list = ['kevin', 'jason', 'jason', 'jason', 'kevin', 'kevin']
s2 = set(name_list)
l1 = list(s2)
print(l1)  # ['jason', 'kevin']

2.练习2
  ll = [33, 22, 11, 22, 11, 44, 33, 22, 55, 66, 77, 77, 66, 55, 44]
  # 基本要求:去重即可
  s3 = set(ll)
  ll1 = list(s3)
  print(ll1)  # [33, 66, 11, 44, 77, 22, 55]

  # 拔高要求:去重并保留原来的顺序
    l2 = []  # 定义一个新列表
    for i in ll:  # 循环取值列表ll
        if i not in l2:  # 判断取到的值是不是在新列表l2里,如果在说明重复,则不操作
            l2.append(i)  # 如果值不在新列表里就追加进去,达到去重且按顺序
    print(l2)  # [33, 22, 11, 44, 55, 66, 77]
```

#### 4.3 关系运算

```python
两个群体之间做差异比较

friends1 = {"zero", "kevin", "jason", "eg"}  # 用户1的好友们
friends2 = {"Jy", "ricky", "jason", "eg"}  # 用户2的好友们
```

##### 4.3.1 交集

```python
# 共同的好友
print(friends1 & friends2)  #{'jason', 'eg'}
```

##### 4.3.2 并集

```python
# 求两个用户所有的好友
print(friends1 | friends2)  # {'kevin', 'ricky', 'jason', 'zero', 'Jy', 'eg'}
```

##### 4.3.3 差集

```python
# 求用户1独有的好友
  print(friends1 - friends2)  # {'zero', 'kevin'}

# 求用户2独有的好友
  print(friends2 - friends1)  # {'ricky', 'Jy'}
```

##### 4.3.4 对称差集

```python
# 求用户1和用户2各自的好友
  print(friends1 ^ friends2)  # {'Jy', 'zero', 'kevin', 'ricky'}
```

##### 4.3.5 父集与子集

```python
s1 = {11, 22, 33, 44}
s2 = {11, 33}
print(s1 > s2)  # 判断s1是否是s2的父集   True
print(s2 < s1)  # 判断s2是否是s1的子集   True
```
