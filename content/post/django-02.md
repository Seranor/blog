---
title: Django的安装及三板斧
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-02.html
toc: true
---

### django 框架

版本

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/release-roadmap.688d8d65db0b.png)

<!-- more -->

```python
# 注意事项
1. 计算机名称不能有中文
2. 项目名和py文件名也不要使用中文
3. django版本选择LTS长期支持版本
```

### 安装

```python
# 命令行下载 如果没配置加速 -i https://pypi.douban.com/simple/
pip3 install django==1.11.11

# 测试是否完成
django-admin
```

### 创建项目

#### 命令创建 django 项目

```python
# 创建django项目
django-admin startproject 项目名

# 启动django项目
cd 项目名
python3 manage.py runserver ip:port

 # python版本原因可能会造成的问题
ps:如果报错需要修改py文件源码
D:\Python38\lib\site-packages\django\contrib\admin\widgets.py
152行后面的逗号去掉即可!!!
'%s=%s' % (k, v) for k, v in params.items()

# 创建app
python3 manage.py startapp app名字

'''
django是一款专门开发app(应用)的软件
我们创建一个django项目之后类似于创建了一所大学
而app就类似于大学里面的各个学院，每个学院都可以有自己独立的各项功能职责
django相当于是一个空壳子用来给各个学院提供资源!!!

"""我们创建的app一定要去settings文件中注册才能生效"""
'''
```

#### pycharm 创建 django 项目

##### 创建项目

![image-20220218215414177](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218215414177.png)
![image-20220218215651361](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218215651361.png)
![image-20220218215724571](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218215724571.png)
![image-20220218215808164](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218215808164.png)

![image-20220219144033575](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220219144033575.png)

![image-20220219143929900](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220219143929900.png)

```python
'DIRS': [os.path.join(BASE_DIR, 'templates')]
```

##### 修改启动端口

![image-20220218220250581](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218220250581.png)
![image-20220218220333420](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218220333420.png)

![image-20220218220350510](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220218220350510.png)

##### 创建新的 app

![image-20220219134528313](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220219134528313.png)

![image-20220219134621861](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220219134621861.png)

![image-20220219135113273](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220219135113273.png)

#### 区别

```python
命令行与pycharm创建不同点
1.命令行不会自动创建templates模板文件夹
2.命令行也不会自动在配置文件中配置模板文件夹路径
```

### 目录结构

```python
mysite1
├── app01             # 应用
│   ├── admin.py      # django后台管理
│   ├── apps.py       # 注册app
│   ├── migrations    # 存储数据库记录相关(类似操作日志)
│   ├── models.py     # 数据库相关(模型层)
│   ├── tests.py      # 测试文件
│   └── views.py      # 视图函数(视图层)
├── db.sqlite3        # django自带的小型数据库
├── manage.py         # django入口文件
└── mysite1           # 项目同名文件夹
    ├── settings.py   # django暴露给用户可以配置的配置文件
    ├── urls.py       # 路由与视图函数(函数或类)对应关系(路由层)
    └── wsgi.py
```

### 必会三板斧

```python
1. HttpResponse
   返回字符串

2. render
   返回html页面，还可以使用模板语法

3. redirect
   重定向
```

```python
# urls.py
from django.conf.urls import url
from django.contrib import admin
from app01 import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index),
]

# app01/views.py
from django.shortcuts import render, HttpResponse, redirect

def index(request):
    # return HttpResponse('你好呀')
    # return render(request, 'index.html')
    # l1 = [111, 222, 333, 444]
    # return render(request, 'index.html', {'xxx': l1})
    return redirect('https://www.baidu.com')


# templates/index.html
<body>
<h1>这是html页面</h1>
{{ xxx }}
{% for l in xxx %}
    {{ l }}
{% endfor %}
</body>

# 联动
# urls.py增加login路由对应函数
url(r'^login/', views.login),

# views.py增加login函数功能
def login(request):
    return HttpResponse('login')

# index的html页面上a标签跳转到当前 127.0.0.1:8000/login  上
<a href="/login/">百度一下</a>
```
