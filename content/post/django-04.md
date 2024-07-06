---
title: 路由匹配和路由分发
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-04.html
toc: true
---

### Django 请求生命周期

![auznQ0](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/auznQ0.jpg)

<!-- more -->

### 路由匹配

```python
# urls.py
  url()方法
    1. 第一个参数是一个正则表达式
    2. 一旦第一个参数匹配到了内容直接结束匹配 执行对应的视图函数
```

#### 无名分组

```python
url(r'^test/\d+/$', views.test)

# 无名分组，将正则表达括起来为一组
url(r'^testadd/(\d+)/$', views.test)

'''无名分组会将括号内正则表达式匹配到的内容当做位置参数传递给后面的视图函数'''

# views.py
  def testadd(request, a):
      print(a)
      return HttpResponse('from testadd')
```

#### 有名分组

```python
url(r'^test1/(?P<id>\d+)/$', views.test1)

'''会将括号内正则表达式匹配到的内容当做关键字参数传递到后面的视图函数'''

# views.py
def test1(request,id):  # 这个关键字参数名称 id 与 正则表达式的分组名称一致 <id>
    print(id)
    return HttpResponse('from test1')

# 有名分组与无名分组不能混合使用，可以单个重复使用
url(r'^test2/(\d+)/(?P<id>\d+)/$', views.test2)  # 错误，两者不能混合使用
url(r'^test3/(?P<id1>\d+)/(?P<id2>\d+)/$', views.test3)  # 可以单独的可以重复使用
```

#### 反向解析

```python
当路由频繁变化的时候，html页面上的连接地址需要做到动态解析
<a href="/edit/?edit_id={{ user_obj.id }}" class="btn btn-warning">编辑</a>

1.给路由与视图函数对应关系添加一个别名(不冲突即可)
url(r'^edit/(?P<edit_id\d+>/$)', views.edit, name='edit')

2.根据该别名动态解析出一个结果，该结果可以直接访问到对应的路由
	前端: <a href="{% url 'index_name' %}">111</a>
	后端:
    from django.shortcuts import reverse
		reverse('index_name')  # 返回的是接口地址 还是需要用 redirect
		ps:redirect括号内也可以直接写别名
```

#### 无名有名反向解析

```python
url(r'^index/(\d+)/',views.index,name='index_name')
后端: reverse('index_name',args=(1,))  # 只要给个数字即可
前端: <a href="{% url 'index_name' 1 %}"></a>  # 只要给个数字即可


url(r'^index/(?P<id>\d+)/',views.index,name='index_name')
后端: reverse('index_name',kwargs={'id':123})
前端: <a href="{% url 'index_name' id=666 %}"></a>

eg:
  url(r'^edit/(?P<edit_id>\d+)/$', views.edit_user, name='edit')
  前端: <a href="{% url 'edit' edit_id=user_obj.id %}" class="btn btn-warning">编辑</a>
  后端: 在视图函数中定义 edit_id 接收到 id 值，根据id值操作对应的数据信息
总结
	无名有名都可以使用一种(无名)反向解析的形式
```

### 路由分发

```python
django专注于开发应用，当一个django项目大的时候，路由与视图函数映射关系全部写在总的urls.py就特别大

django中的每一个应用都可以有自己的urls.py  static文件 templates文件
因此每个人只需要写主机的应用即可，最后统一到总的urls.py路由中关联

1.复杂导入
from django.conf.urls import url, include
from app01 import urls as app01_urls
from app02 import urls as app02_urls
...
url(r'^app01/', include(app01_urls))
url(r'^app02/', include(app02_urls))
...


2.进阶导入
# 不用导入app的urls
url(r'app01/', include('app01.urls'))
url(r'app02/', include('app02.urls'))
```

### 名称空间

```python
'''当多个应用在反向解析的时候如果出现别名冲突的情况时，那么无法识别'''
# app01/urls.py
from django.conf.urls import url
from app01 import views
urlpatterns = [
    url(r'^index/', views.index, name='index_name'),
    url(r'^login/', views.login)
]

# app02/urls.py
from django.conf.urls import url
from app02 import views
urlpatterns = [
    url(r'^index/', views.index, name='index_name'),
    url(r'^login/', views.login)
]


# app01/views.py
from django.shortcuts import render, HttpResponse, redirect, reverse
def index(request):
    return HttpResponse('from app01 index')
def login(request):
    print(reverse('index_name'))
    return HttpResponse('from app01 login')


# app02/views.py
from django.shortcuts import render, HttpResponse, redirect, reverse
def index(request):
    return HttpResponse('from app02 index')
def login(request):
    print(reverse('index_name'))
    return HttpResponse('from app02 login')

'''针对上述问题有以下解决办法'''
方式1:名称空间(namespace)
  总路由:
    url(r'app01/', include('app01.urls', namespace='app01')),
    url(r'app02/', include('app02.urls', namespace='app02')),
  后端:
    reverse('app01:index_name')
    reverse('app02:index_name')
  前端:
    <a href="{% url 'app01:index_name' %}">app01</a>
		<a href="{% url 'app02:index_name' %}">app02</a>

方式2:起别名不要冲突即可(前缀加上应用名)
  # app01/urls.py
    url(r'^index/', views.index, name='app01_index_name')

  # app02/urls.py
    url(r'^index/', views.index, name='app02_index_name')

  # 前端
  <a href="{% url 'app01_index_name' %}">app01</a>
  <a href="{% url 'app02_index_name' %}">app01</a>
```
