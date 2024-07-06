---
title: 静态文件、request方法、连接MySQL、ORM
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-03.html
toc: true
---

### 静态文件配置

<!-- more -->

```python
1.什么是静态文件
静态文件，写好之后不改变的文件，如css文件、js文件、图片、第三方框架等
这类文件一般单独使用一个名为 static 的文件夹存放

2.使用静态文件
html文件调用
html在引入时，使用static文件夹的相对路径会在页面无法加载(404)
原因是在 urls.py 并没有设置该路由

3.配置
django框架针对这类文件有单独的设置，在settings.py中设置
settings.py最下方
...
STATIC_URL = '/static/'  # 接口前缀

STATICFILES_DIRS = [
  os.path.join(BASE_DIR,'static')  # static文件夹的路径
  ...                              # 文件夹可以有多个
]

html文件引入
<script src="/static/jQuery-3.6.0.js"></script>
# 这边的 /static/ 是接口而不是路径了

3.动态解析
在settings.py文件中的接口前缀是可以随意变换的，html文件中不可能一个个更改
因此就需要动态解析，配置在html文件中，如下配置
{% load static %}
<script src="{% static 'jQuery-3.6.0.js' %}"></script>
```

**关闭谷歌浏览器的缓存**

![image-20220220135919254](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220135919254.png)

### `request`对象方法

#### `form`表单

```python
<form action="" method="post" enctype="multipart/form-data"></form>

action:
  控制后端的提交路径
  不写: 默认朝当前页面地址提交数据
  后缀: /index/
  全写: https://127.0.0.1:8000/index

method提交方法
  get   # 默认的提交方法
  post

当有文件提交时需要增加属性: enctype="multipart/form-data"
```

前期使用 post 提交时，先在 django 中的 settings.py 先注释该行

![image-20220220141801183](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220141801183.png)

#### 相关文件

```python
# urls.py
  url(r'^login/', views.login),

# views.py
  def login(request):
    # res = request.method
    # print(res)
    if request.method == 'POST':
        # print(request.POST)
        username = request.POST.get('name')
        password = request.POST.get('password')
        hobby = request.POST.getlist('hobby')
        files = request.FILES
        print(username, password, hobby, files)
    return render(request, 'demo2.html')

# templates/demo2.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container">
    <div class="row">
        <div class="col-md-8 col-md-offset-2">
            <h1 class="text-center">登录界面</h1>
            <form action="" method="post" enctype="multipart/form-data">
                <p>username:<input type="text" class="form-control" name="name"></p>
                <p>password<input type="password" class="form-control" name="password"></p>
                <p>
                    <input type="checkbox" name="hobby" value="fb">足球
                    <input type="checkbox" name="hobby" value="bb">篮球
                    <input type="checkbox" name="hobby" value="cb">双色球
                </p>
                <input type="file" name="files" multiple>
                <input type="submit" class="btn btn-success btn-block" value="提交">
            </form>
        </div>
    </div>
</div>
</body>
</html>
```

#### `request.method`

```python
获取当前请求的请求方法，结果是纯大写的字符串类型，默认处理的也是GET请求

# 针对不同的请求方法返回不同的操作，默认的GET就直接返回页面
if request.method == 'POST'
		pass
return render(request, 'demo2.html')
```

#### `request.POST`

```python
获取用户通过POST请求过来的基本数据(不包含文件)，结果可以看成一个字典{'':[],'':[]}

get()
request.POST.get('name')  # 获取name对应列表里的最后一个元素

getlist()
request.POST.get('hobby')  # 获取hobby对应的整个列表
```

#### `request.GET`

```python
获取用户通过GET请求过来的基本数据(不包含文件)，结果可以看成一个字典{'':[],'':[]}
get()       # 获取列表的最后一个元素
getlist()   # 获取整个列表
```

#### `reqest.FILES`

```python
获取用户上传的数据
request.FILES  # 一个文件对象
```

### Pycharm 连接 MySQL

![image-20220220151141386](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220151141386.png)

![image-20220220151232719](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220151232719.png)

![image-20220220151548372](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220151548372.png)

![image-20220220151639520](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220151639520.png)

### Django 连接 MySQL

```python
Django默认使用自带的sqlite3，需要使用MySQL数据库需要配置

1.settings.py文件设置
# 将这段代码注释
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
#     }
# }

# 添加如下代码
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db01',
        'HOST': '10.0.0.60',
        'PORT': 3306,
        'USER': 'root',
        'PASSWORD': '123456',
        'CHARSET': 'utf8'

    }
}

2. 在项目文件夹或者应用文件夹内的__init__.py文件中书写固定的代码=
	 import pymysql
	 pymysql.install_as_MySQLdb()

	 # 如果不写上述两行代码会报如下错误
   '''
   django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: No module named 'MySQLdb'.
    Did you install mysqlclient or MySQL-python?
   '''
```

### ORM 简介

```python
ORM: 对象关系映射
优点: 不用直接操作SQL语句，更好的操作业务层面上的逻辑
缺点: 封装程度太高，有些时候效率低，就需要自己编写SQL语句

类          表
对象        字段
属性        字段对应的值
```

### ORM 操作

#### 编写模型类

```python
模型类需要写在应用下的 models.py 文件中
class User(models.Model):
    # id int primary key auto_increment
    id = models.AutoField(primary_key=True)
    # name varchar(32)
    name = models.CharField(max_length=32,verbose_name='用户名')  # 默认是varchar max_length参数必须添加
    # age int
    age = models.IntegerField('年龄')
    # 中文注释 使用 verbose_name 参数 和MySQL的 comment 功能一样
    # 或者直接写入注释 verbose_name位置参数在第一个
```

#### 数据库迁移

```python
1. 将数据库修改记录先记录到对应应用下的migrations文件夹内
	 python3 manage.py makemigrations

2. 执行数据库迁移操作
	 python3 manage.py migrate

# 只要操作了models.py中跟数据库相关的代码就必须要重新执行上述两条命令

3. 针对主键字段
   # 如果不指定主键，ORM会自动创建一个名为id的主键字段(MySQL创建的是隐藏的主键)
   # 主键字段名可以不交id 需要命名其他名称就手动指定
```

![image-20220220161526931](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220161526931.png)

![image-20220220161906012](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220161906012.png)

![image-20220220162439594](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220162439594.png)

![image-20220220163206695](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220163206695.png)

![image-20220220163348612](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220220163348612.png)

#### 数据库同步

```python
"""
数据库里面已经有一些表，我们如何通过django orm操作?
	1.照着数据库表字段自己在models.py
		数据需要自己二次同步
	2.django提供的反向同步
"""
1.先执行数据库迁移命令 完成链接
	python manage.py makemigrations
2.查看代码
	python manage.py inspectdb

    class Userinfo(models.Model):
        id = models.IntegerField(blank=True, null=True)
        name = models.CharField(max_length=32, blank=True, null=True)
        pwd = models.IntegerField(blank=True, null=True)

        class Meta:
            managed = False
            db_table = 'userinfo'
```

### 数据的 CRUD

#### 字段

```python
# 增
pwd = models.IntegerField('密码',null=True)  # 该字段可以为空
is_delete = models.IntegerField(default=0)  # 默认值
# 如果不写入上述参数 执行数据库迁移命令是会提示
# 原因是新添加字段，那么旧数据的新字段的数据填充什么

# 改
直接改代码然后直接数据库迁移命令即可
python3 manage.py makemigrations
python3 manage.py migrate

# 删
注释代码然后执行数据库迁移命令即可，删除后的数据也丢失
python3 manage.py makemigrations
python manage.py migrate
```

#### 数据

```python
# 查询数据
# views.py 文件中功能函数
username = request.POST.get('name')
password = request.POST.get('password')
from app01 import models
# select * from user where name=username
user_obj = models.User.objects.filter(name=username).first()
# 返回的结果是一个表名对象，打印时可以触发 __str__方法
# 然后结果是一个列表套数据对象
# models.py (上面的代码中需要添加如下代码)
def __str__(self):
  return self.name  # 必须要返回一个字符串类型

# 添加数据
username = request.POST.get('name')
password = request.POST.get('password')
# insert into user(name,pwd) values(username,password)
models.User.objects.create(name=username,pwd=password)

# 查询所有数据
models.User.object.all()

# 修改数据
  # models.User.objects.filter(id=edit_id).update(name=username, pwd=password)
  edit_obj.name = username
  edit_obj.pwd = password
  edit_obj.save()

# 删除数据
models.User.objects.filter(id=delete_id).delete()
```

### ORM 创建外键关系

```python
# ORM针对外键字段的创建位置
  一对多: 推荐在多的一方
  一对一: 在任意方都可以创建，推荐建在查询频率较高的一方
  多对多: 自己建表


class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)  # 总共8位 小数占2位
    publish = models.ForeignKey(to='Publish')  # 出版社外键 默认是主键 自动在外键字段后面加 _id 后缀
    authors = models.ManyToManyField(to='Author')  # 自动创建书籍和作者的第三张表
	  # 虚拟字段不会在表中实例化出来  而是告诉ORM创建第三张关系表

class Publish(models.Model):
    title = models.CharField(max_length=32)
    email = models.EmailField()

class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    author_detail = models.OneToOneField(to='AuthorDetail')  # 自动在外键字段后面加 _id 后缀

class AuthorDetail(models.Model):
    phone = models.BigIntegerField()
    addr = models.CharField(max_length=128)
```
