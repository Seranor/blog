---
title: ORM常用查询操作(一)
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-06.html
toc: true
---

### Django 创建测试环境

```python
在Pycharm中运行django项目不能单独运行单个文件

方式1: 在当前项目任意位置创建一个py文件，内容如下即可，后续的操作都要在下方
import os
if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite6.settings")
    import django
    django.setup()

方式2: 直接使用pycharm提供的python console
```

<!-- more -->

### 单表查询关键字

#### 数据准备

```python
# 创建表
# models.py
from django.db import models
class Books(models.Model):
  title = models.CharField(verbose_name='书名', max_lenth=64)
  price = models.DecimalField(verbose_name='价格', max_digits=8, decimal_places=2)
  pubilsh_time = models.DateField(auto_now_add=True)
  '''
  auto_now: 每次修改数据时都会自动更新当前时间
  auto_now_add: 只记录数据创建的时间，后续不更新
  '''
  def __str__(self):
    return self.name

# 迁移数据库和创建数据
python3 manage.py makemigrations
python3 manage.py migrate


# 增加数据
# res = models.Books.objects.create(title='西游记',price=687.90)
# print(res.title)
'''create返回值就是当前被创建的数据对象'''

# 修改数据
# 方式1
res = models.Books.objects.filter(pk=1).update(price=666.66)
print(res)  # 返回值是受影响的行数

# 方式2
book_obj = models.Books.objects.filter(pk=1).first()
book_obj.price = 999.66
book_obj.save()  # 效率低(所有字段重新写一遍)

# 删除数据
models.Books.objects.filter(pk=1).delete()
```

#### `all()`

```python
res = models.Books.objects.all()
print(res.query)  # query 查看 orm 内部对应的SQL语句
```

#### `filter()`和`get()`

```python
res = models.Books.objects.filter()  # 不加条件默认查询所有
print(res.query)

res = models.Books.objects.filter(title='西游记',price=687.90)
print(res)
''' filter括号内可以放多个参数 默认是 and 关系 推荐使用 条件不符合不会报错 '''

res = models.Books.object.get(title='西游记')
print(res)
''' get 括号内可以放多个参数 默认是 and 关系 不推荐使用 条件不符合直接报错 '''
```

#### `first()`

```python
# 取第一个数据对象
res = models.Books.object.all().first()
# 等价于           res = models.Books.object.all()[0]
# 同时支持切片操作   res = models.Books.object.all()[0:2]
```

#### `last()`

```python
# 取最后一个数据对象
res = models.Books.object.all().last()
```

#### `values()`

```python
# 获取指定字段的值
res = models.Books.object.all().values('title','publish_time')
"""all()加不加都表示所有数据  values获取的结果 类似于列表套字典"""
```

#### `values_list()`

```python
# 获取数据指定字段的值
res = models.Books.object.values_list('title','publish_time')
"""values_list获取的结果 类似于列表套元组"""
```

#### `order_by()`

```python
# 排序
res = models.Books.object.order_by('price')  # 默认是升序
res = models.Books.object.order_by('-price')  # 降序
```

#### `count()`

```python
# 计数
res = models.Books.object.count()  # 统计数据条数
```

#### `distinct()`

```python
# 去重
res = models.Books.object.all().destinct()
res = models.Books.object.values('title').destinct()
"""去重的前提是数据必须是一模一样  一定不能忽略主键"""
```

#### `exclude()`

```python
# 排除...在外  取反
res = models.Books.object.exclude(title='西游记')
```

#### `reverse()`

```python
# 反转
res = models.Books.objects.order_by('price').reverse()
"""reverse需要先排序之后才能反转"""
```

#### `exists()`

```python
# 判断是否有数据 返回布尔值
res = models.Books.object.all.exists()
```

#### 针对返回值总结

```python
# 返回QuerySet对象的方法有
all()
filter()
exclude()
order_by()
reverse()
distinct()

# 特殊的QuerySet
values()       # 返回一个可迭代的字典序列
values_list()  # 返回一个可迭代的元祖序列

# 返回具体对象的
get()
first()
last()

# 返回布尔值的方法有
exists()

# 返回数字的方法有
count()
```

### 双下划线查询 (范围查询)

```python
1. 查询价格大于700的书籍
res = models.Books.objects.filter(price__gt=700)

2. 查询价格小于700的书籍
res = models.Books.objects.filter(price__lt=700)

3. 查询价格是 212.34, 210.99, 599.78 的书籍
res = models.Books.objects.filter(price__in=[212.34, 210.99, 599.78])
''' python对数字不敏感 精确度不高 很多时候会采取字符串类型 '''

4. 查询价格在500到800之间的
res = models.Books.objects.filter(price__range=(500, 800))

5. 查询书名中包含字母 s 的书
res = models.Books.objects.filter(title__contains='s')   # 区分大小写
res = models.Books.objects.filter(title__icontains='s')  # 不区分大小写

6. 查询出版日期是2022的书
res = models.Books.objects.filter(publish_time__year=2022)

7. 查询出版日期是3月的书
res = models.Books.objects.filter(publish_time__month=3)
```

### 图书管理系统表设计

```python
class Book(models.Model):
    title = models.CharField(verbose_name='书名', max_length=64)
    price = models.DecimalField(verbose_name='价格', max_digits=8, decimal_places=2)
    publish_time = models.DateField(auto_now_add=True)

    # 一对多 外键字段建在多的一方
    publish = models.ForeignKey(to='Publish')

    # 多对多 外键字段推荐建在查询频率较高的表中
    authors = models.ManyToManyField(to='Author')


class Publish(models.Model):
    title = models.CharField(verbose_name='出版社名称', max_length=64)
    addr = models.CharField(verbose_name='出版社地址', max_length=128)
    email = models.EmailField(verbose_name='邮箱')


class Author(models.Model):
    name = models.CharField(verbose_name='作者名称', max_length=64)
    age = models.IntegerField(verbose_name='年龄')

    # 一对一 外键字段推荐建立在查询频率较高的表中
    author_detail = models.OneToOneField(to='AuthorDetail')


class AuthorDetail(models.Model):
    phone = models.CharField(verbose_name='电话', max_length=64)
    addr = models.CharField(verbose_name='地址', max_length=128)
```

### 外键字段的增删改查

```python
# 增
# book_obj = models.Book.objects.filter(pk=1).first()
# 主键值
# book_obj.authors.add(1)  # 去第三张关系表中 与作者主键为1的绑定关系
# 作者对象
# author_obj = models.Author.objects.filter(pk=2).first()
# book_obj.authors.add(author_obj)
# 括号内支持传多个参数
# book_obj.authors.add(1,2)
# author_obj1 = models.Author.objects.filter(pk=1).first()
# author_obj2 = models.Author.objects.filter(pk=2).first()
# book_obj.authors.add(author_obj1, author_obj2)

# 改
# book_boj = models.Book.objects.filter(pk=1).first()
# book_boj.authors.set([1, 2])  # 传入的是可迭代对象 列表 元组等
# 可以传入数据对象
# author_obj1 = models.Author.objects.filter(pk=1).first()
# author_obj2 = models.Author.objects.filter(pk=2).first()
# book_boj.authors.set([author_obj1, author_obj2])

# 删
# book_obj = models.Book.objects.filter(pk=1).first()
# book_obj.authors.remove(1,2)  # 同时也可以传入数据对象

# 清空
# book_obj = models.Book.objects.filter(pk=1).first()
# book_obj.authors.clear()  # 去第三张关系表中删除所有该书籍对应的记录

'''
# 括号内既可以传数字也可以传对象 逗号隔开
add()
remove()

# 括号内必须传递可迭代对象 可迭代对象内既可以传数字也可以传对象 支持多个
set()

# 无需传值
clear()
'''
```

### 跨表查询

```python
正向查询  外键字段所在表，再去查其他表内的数据

反向查询  没有外键字段，再去查其他的表

# 判断是否有关联的外键字段
正向查询按 外键字段
反向查询按 表名小写_set
```

#### 基于对象的跨表查询 (子查询)

```python
1. 查询聊斋书籍对应的出版社名称
    book_obj = models.Book.objects.filter(title='聊斋').first()
    res = book_obj.publish
    print(res.title)

2. 查询神雕侠侣对应的作者
    book_obj = models.Book.objects.filter(title='神雕侠侣').first()
    res = book_obj.authors.all()
    print(res)

3. 查询Jason的地址
    author_obj = models.Author.objects.filter(name='jason').first()
    res = author_obj.author_detail
    print(res.addr)

4. 查询东方出版社出过的书籍
    publish_obj = models.Publish.objects.filter(title='东方出版社').first()
    res = publish_obj.book_set.all()
    print(res)

5. 查询jason写过的书
    author_obj = models.Author.objects.filter(name='jason').first()
    res = author_obj.book_set.all()
    print(res)

6. 查询电话是 122222 的作者姓名
    author_detail_obj = models.AuthorDetail.objects.filter(phone=122222).first()
    res = author_detail_obj.author
    print(res)
```

#### 基于下划线的跨表查询 (连表查询)

```python
1. 查询聊斋书籍对应的出版社名称
    # 方式一:
    res = models.Book.objects.filter(title='聊斋').values('publish__title').first()
    print(res)

    # 方式二:
    res = models.Publish.objects.filter(book__title='聊斋').values('title')
    print(res)

2. 查询神雕侠侣对应的作者名字和年龄
    # 方式一:
    res = models.Book.objects.filter(title='神雕侠侣').values('authors__name', 'authors__age')
    print(res)

    # 方式二:
    res = models.Author.objects.filter(book__title='神雕侠侣').values('name')
    print(res)

3. 查询jason的地址
    # 方式一:
    res = models.Author.objects.filter(name='jason').values('author_detail__addr')
    print(res)

    # 方式二:
    res = models.AuthorDetail.objects.filter(author__name='jason').values('addr')
    print(res)

4. 查询神雕侠侣对应的作者的电话和地址
    # 方式一:
    res = models.Book.objects.filter(title='神雕侠侣').values('authors__author_detail__phone','authors__author_detail__addr')
    print(res)

    # 方式二:
    res = models.AuthorDetail.objects.filter(author__book__title='神雕侠侣').values('phone', 'addr')
    print(res)
```
