---
title: ORM常用查询操作(二)
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-07.html
toc: true
---

### 图书管理系统的表关系建立

<!-- more -->

```python
'''
5张表
书籍表 作者表 作者详情表 出版社表 书籍和作者表

一对一的关系，关联字段可以写在任意一方，推荐查询频率高的一方
一对多的关系，关联字段写在多的一方
多对多的关系，必须建立第三张表(orm中可以用一个字段表示，这个字段可以写在任意一方)
'''

class Book(models.Model):
    nid = models.AutoField(primary_key=True)  # 主键自增
    name = models.CharField(max_length=64)
    price = models.DecimalField(max_digits=5, decimal_places=2)  # 小数
    publish_data = models.DateField(auto_now_add=True)  # 日期类型

    # 阅读数
    # read_num=models.IntegerField(default=0)
    # 评论数
    # commit_num=models.IntegerField(default=0)

    # to关联的表不加引号引号时写在当前表上方  to_field 关联字段
    publish = models.ForeignKey(to='Publish', to_field='nid', on_delete=models.CASCADE)
    # models.CASCADE：级联删除，设为默认值，设为空，设为指定的值，不做处理 django2.x版本之后必须加

    authors = models.ManyToManyField(to='Author')

    def __str__(self):
        return self.name


class Publish(models.Model):
    nid = models.AutoField(primary_key=True)
    name = models.CharField(max_length=32)
    city = models.CharField(max_length=32)
    email = models.EmailField()


class Author(models.Model):
    nid = models.AutoField(primary_key=True)
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    author_detail = models.OneToOneField(to='AuthorDetail', to_field='nid', unique=True, on_delete=models.CASCADE)


class AuthorDetail(models.Model):
    nid = models.AutoField(primary_key=True)
    telephone = models.BigIntegerField()
    birthday = models.DateField
    addr = models.CharField(max_length=64)
```

### `Django admin`的使用

```python
1. 后台管理，方便快速录入数据
2. 使用方法:
   在 admin.py 中把要使用的表注册
    from app01 import models
    admin.site.register(models.Book)
    admin.site.register(models.Publish)
    admin.site.register(models.Author)
    admin.site.register(models.AuthorDetail)

   创建超级管理员
    python3 manage.py createsuperuser  # 后续输入用户名及密码

   访问页面 http://127.0.0.1:8000/admin  # 输入用户密码
```

### Django 查看原生 SQL

```python
1.queryset对象.query
2.通过日志，如下，配置到setting.py中
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```

### 基于双下划线的跨表查询

```python
1.基于对象的跨表查询      子查询，多次查询
2.基于双下划线的跨表查询   多表连接查询
```

```python
# 在django中使用测试脚本，创建任意名称的py文件

import os
#加载配置文件，跑django的项目，最开始就是把配置文件加载上
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "day53.settings")

if __name__ == '__main__':
    import django  # 安装了django模块，就可以import
    django.setup() # 使用环境变量中的配置文件，跑django

    # 查询主键为1的书籍的出版社所在的城市
    # book = models.Book.objects.filter(pk=1).first()
    # print(book.publish.city)
    # res = models.Book.objects.filter(pk=1).values('publish__city')
    # print(res)

    # 查询所有住址在北京的作者的姓名
    # res = models.Author.objects.filter(author_detail__addr='北京').values('name')
    # print(res)
    # res = models.AuthorDetail.objects.filter(addr='北京').values('author__name')
    # print(res)

    # 查询egon出过的所有书籍的名字
    # res = models.Book.objects.filter(authors__name='egon').values('name')
    # print(res)
    # res = models.Author.objects.filter(name='egon').values('book__name').all()
    # print(res)

    # 查询北京出版社出版过的所有书籍的名字以及作者的姓名和地址
    # res = models.Book.objects.filter(publish__name='北京出版社').values('name', 'authors__name','authors__author_detail__addr')
    # print(res)

    # res = models.Publish.objects.filter(name='北京出版社').values('name', 'book__name', 'book__authors__name','book__authors__author_detail__addr')
    # print(res)

    # res = models.AuthorDetail.objects.filter(author__book__publish__name='北京出版社').values('author__book__name','author__name', 'addr')
    # print(res)
```

### 聚合查询

```python
1.聚合函数 sum max min count avg
2.把聚合结果重命名
	res = modles.Book.objects.all().aggregate(aaa=Sum('price'))
3.使用时需要导入如下模块
	from django.db.models import Sum,Avg,Max,Min,Count
```

```python
# 计算所有图书的平均价格
    res = models.Book.objects.aggregate(aaa=Avg('price'))
    print(res)

# 计算所有图书的最高价格
	  res = models.Book.objects.aggregate(Max('price'))
    print(res)

# 计算所有图书的总价格
	  res = models.Book.objects.aggregate(Sum('price'))
    print(res)

# egon出版图书的总价格
	  res = models.Author.objects.filter(name='egon').aggregate(Sum('book__price'))
    print(res)

# 北京出版从出版社书的最高价格
    res = models.Publish.objects.filter(name='北京出版社').aggregate(Max('book__price'))
    print(res)

# 计算所有图书的总价格和平均价格
    res = models.Book.objects.all().aggregate(book_sum=Sum('price'), book_avg=Avg('price'))
    print(res)
```

### F 查询

```python
取出某个字段对应的值
from django.db.models import F

#查询评论数大于阅读数的书籍
  res = models.Book.objects.filter(commit_num__gt=F('read_num'))
  print(res)

# 把所有图书价格+1
  res = models.Book.objects.all().update(price=F('price')+1)
  print(res)
```

### Q 查询

```python
构造出 与& 或| 非~
  # 查询名字叫红楼梦或者价格大于80的书
    res = models.Book.objects.filter(Q(name='红楼梦') | Q(price__gt=80))
    print(res)

    # 下列是并且
    # res = models.Book.objects.filter(name='红楼梦',price__gt=80)
    # res = models.Book.objects.filter(Q(name='红楼梦') & Q(price__gt=80))

  # 查询名字不是红楼梦的书
    res = models.Book.objects.filter(~Q(name='红楼梦'))
    print(res)

  # 查询名字不是红楼梦，并且价格大于80的书
    res1 = models.Book.objects.filter(~Q(name='红楼梦'),price__gt=80)
    print(res1)
    res2 = models.Book.objects.filter(~Q(name='红楼梦') & Q(price__gt=80))
    print(res2)
```

### 分组查询

```python
'''
标准 annotate() 内写聚合函数
values在前 表示 group by 的字段
values在后 表示取字段
filter在前 表示where条件
filter在后 表示having
'''

# 查询每一个出版社id，以及出书平均价格(单表)
# select publish_id,avg(price) from book group by publish_id;
  res = models.Book.objects.values('publish_id').filter(publish_id__gt=1).annotate(price_ave=Avg('price')).values('publish_id', 'price_ave')
  print(res)

# 查询出版社id大于1的出版社id，以及出书平均价格大于60的
  res = models.Book.objects.values('publish_id').filter(publish_id__gt=1).annotate(price_avg=Avg('price')).filter(price_avg__gt=60).values('publish_id')
  print(res)

# 查询每一个出版社出版的名称和书籍个数(连表)
  res = models.Book.objects.values('publish_id').annotate(book_count=Count('nid')).values('publish__name','book_count')
  print(res)

## 联表的话最好以group by的表作为基表
  res = models.Publish.objects.values('nid').annotate(book_count=Count('book__nid')).values('nid','book_count')
  print(res)

## 简写成，如果基表是group by的表，就可以不写values
res = models.Publish.objects.annotate(book_count=Count('book')).values('name','book_count')

# 以book为基表
  res = models.Book.objects.values('publish__nid').annotate(book_count=Count('nid')).values('publish__name','book_count')
  print(res)

# 查询每个作者的名字，以及出版过书籍的最高价格(建议使用分组的表作为基表)
# 多对多如果不以分组表作为基表，可能会出数据问题
  res = models.Author.objects.values('name').annotate(max_price=Max('book__price')).values('name', 'max_price')
  print(res)

# 查询每一个书籍的名称，以及对应的作者个数
  res = models.Book.objects.values('name').annotate(author_count=Count('authors')).values('name','author_count')
  print(res)

# 统计不止一个作者的图书
  res = models.Book.objects.values('name').annotate(author_count=Count('authors')).filter(author_count__gt=1).values('name')
  print(res)

# 统计价格数大于10元，作者个数大于1的图书
  res = models.Book.objects.filter(price__gt=10).values('name').annotate(author_count=Count('authors')).filter(author_count__gt=1).values('name')
  print(res)

```

### 常用和非常用字段

```python
1.AutoField
  int自增，必须填入参数primary_key=True 当mode中如果没有自增列，则自动创建一个名为id的列

2.IntegerField
  一个整数类型，范围在 -2147483648 to 2147483647

3.CharField
	字符类型，必须提供max_length参数，表示字符长度

4.DateField
	日期时间字段，格式Y-M-D

5.DateField
	日期字段，格式Y-M-D H-M

FloatField(Field)    浮点型
DecimalField(Field)  10进制小数
BinaryField(Field)   二进制类型
```

### 字段参数

```python
null       用于表示字段可以为空
unique     设置为True 表示该字段在此表中唯一
db_index   设置为True 表示为该字段设置索引
default    为该字段设置默认值

DateField和DateTimeField
auto_now_add  创建数据记录的时候会把当前时间添加到数据库
auto_now      每次更新数据记录的时候都会更新该字段

choices  在model表模型定义的时候给某个字段指定choice
sex_choice = ((1,'男'),(2,'女'),(3,'其他'))
sex =models.IntegerField(default=1,choices=sex_choice)
在使用的时候直接取出中文， 对象.get_sex_display()

admin后台管理相关
    verbose_name        Admin中显示的字段名称
    blank               Admin中是否允许用户输入为空
    editable            Admin中是否可以编辑
    help_text           Admin中该字段的提示信息
    choices             Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作
```

### 字段关系

```python
一对一
一对多
多对多
```

#### `ForeignKey`

```python
to                   对哪张表
to_field             对表中某个字段
related_name         反向操作时，使用的字段名，用于代替原反向查询时的’表名_set’
related_query_name   反向查询操作时，使用的连接前缀，用于替换表名

on_delete						 当删除关联表中的数据时，当前表与其关联行的行为
  models.CASCADE     同时删除关联
  models.DO_NOTHING  什么都不做
  models.PROTECT     引发错误ProtectedError
  models.SET_NULL    将关联的值设置为null(前提FK字段需要设置为可空)
  models.SET_DEFAULT 将关联的值设置为默认值(前提FK字段需要设置默认值)
  models.SET         删除关联数据

db_constraint
  True    建立外键
  False   不建立外键

外键是否建立
	好处：不会出现脏数据
	坏处：插入的时候，效率低
	企业中：通常不建立，程序员控制
```

#### `ManyToManyField`

```python
db_table             指定第三张表的名字
to                   关联的表
related_name         同FireignKey字段
related_query_name   同FireignKey字段
through              手动创建第三张表，指定通过哪张表
through_fields       关联字段是什么
```

##### 手动创建第三张表

###### 第一种

```python
# 第一种：
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")

# 通过ORM自带的ManyToManyField自动创建第三张表
class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")
    books = models.ManyToManyField(to="Book", related_name="authors")
```

###### 第二种

```python
# 第二种
class Book1(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")

# 自己创建第三张表，并通过ManyToManyField指定关联
class Author1(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")
    books = models.ManyToManyField(to="Book1", through="Author2Book", through_fields=("author", "book"))

# through_fields 元组的第一个值是ManyToManyField所在的表去中间表通过那个字段，就写在第一个位置

class Author2Book(models.Model):
    author = models.ForeignKey(to="Author1")
    book = models.ForeignKey(to="Book1")

## 基于对象的跨表查，还能继续使用
## 基于双下划綫连表查
## 原来的多对多操作api用不了了，需要手动操作
```

###### 第三种

```python
# 第三种
class Book1(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")

class Author1(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")

# 自己创建第三张表，分别通过外键关联书和作者
class Author2Book1(models.Model):
    author = models.ForeignKey(to="Author")
    book = models.ForeignKey(to="Book")
```

### Meta 元信息

```python
# 在每一个模型类中都可以使用
class Publish(models.Model):
    title = models.CharField(verbose_name='出版社名称', max_length=64)
    addr = models.CharField(verbose_name='出版社地址', max_length=128)
    email = models.EmailField(verbose_name='邮箱')

    class Meta:
      db_table = 'publish'  # 重新定义表名 就不会是默认的 应用名_publish
      index_together=('title','addr')  # 多个字段联合索引
      unique_together=('title','addr')  # 联合唯一
      ordering('id')  # 默认以哪个字段排序
```

### 原生 SQL

```python
# raw 后直接使用原生SQL语句
from app01 import models
res = models.Author.object.raw('select * from app01_author where nid > 1')
for author in res:
  	print(author)

# 执行SQL语句时 和对象类型无光了，查出什么字段直接使用该字段
res = models.Author.object.raw('select * from app01_book where nid > 1')
for book in res:
		print(book)
```

### 其他补充

```python
1.普通函数以__开头: 当前函数只在当前模块下使用，尽量不在外部调用

2.mysql的字符编码
  utf8： 2个字节表示一个字符
  utf8mb4: 相当于文件编码中的 utf-8 (1-4个字节表示一个字符)

3.django中使用pymysql连接mysql
  import pymysql
  pymysql.install_as_MySQLdb()

  # 本质是猴子补丁，想要执行，写在任意位置都可以，__init__.py setting.py

4.settings.py以下配置一般如下配置:
  LANGUAGE_CODE = 'zh-hans'
  TIME_ZONE = 'Asia/Shanghai'
  USE_TZ = False
```
