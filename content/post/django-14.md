---
title: Auth组件
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-14.html
toc: true
---

### `auth`组件介绍

```python
django提供了用户认证，创建，修改密码等用户相关的操作
不需要创建用户表，默认自带了(auth_user)
创建用户: python3 manage.py createsuperuser
```

<!-- more -->

![image-20220306142948086](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220306142948086.png)

### `auth`组件常用方法

```python
from django.contrib import auth
```

#### `authenticate()`

```python
提供了用户认证功能，验证用户名以及密码是否正确，一般需要username、password两个关键字参数
如果认证成功，便会返回一个User对象

from django.contrib.auth import authenticate
user = authenticate(username='username',password='password')
```

#### `login(HttpRequest, user)`

```python
该函数接收一个HttpResqest对象，以及一个经过认证的User对象
该函数实现一个用户登录功能，本质上会在后端为该用户生成相关session数据

登录成功后调用
from django.contrib.auth import login
login(request, user)
```

#### `logout(request)`

```python
该函数接收一个HttpReqest对象，无返回值
当调用该函数是，当前请求的session信息会全部清除，该用户即使没有登录，使用该函数也不会报错

from django.contrib.auth import logout
def logout_view(request):
  logout(request)
```

#### `is_authenticated`

```python
用来判断当前请求是否通过了认证，这个方法property装饰过，不用加()

# 在视图中使用
if not request.user.is_authenticated:

# 模板中使用
{% if request.user.is_authenticated %}
    {{ request.user.username }} 欢迎你
    <a href="/auth_logout/">注销登录</a>
{% else %}
    <a href="/login_auth/">请去登录</a>
{% endif %}
```

#### `login_requierd()`

```python
auth 提供的一个装饰器工具，用来快捷的给某个视图添加登录校验

from django.contrib.auth.decorators import login_required
# login_url如果不填会有一个默认的地址 accounts/login 的位置
@login_required(login_url='/auth_login/')
def auth_order(request):
  ...

# 默认的地址可以在settings.py文件中配置
LOGIN_URL = '/login/'  # 这里配置成你项目登录页面的路由
```

#### `create_user()`

```python
auth 提供的一个创建新用户的方法，需要提供必要参数(username password)等

from django.contrib.auth.models import User
user = User.objects.create_user(username='用户名',password='密码',email='邮箱',...)
```

#### `create_superuser()`

```python
auth 提供的一个创建新的超级用户的方法，需要提供必要参数(username password)等

from django.contrib.auth.models import User
user = User.objects.create_superuser(username='用户名',password='密码',email='邮箱',...)
```

#### `check_password(password)`

```python
auth 提供的一个检查密码是否正确的方法，需要提供当前请求用户的密码
密码正确返回True，否则返回False

ok = user.check_password('密码')
```

#### `set_password(password)`

```python
auth 提供的一个修改密码的方法，接收要设置的新密码作为参数
注意：设置完一定要调用用户对象的save方法

user.set_password(password='')
user.save()
```

#### 后端

```python
from django.shortcuts import render, HttpResponse, redirect
from django.contrib.auth.models import User
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required


def auth_login(request):
    if request.method == 'GET':
        return render(request, 'login_auth.html')
    else:
        username = request.POST.get('username')
        password = request.POST.get('password')
        print(username, password)
        # password是加密的，不能直接认证
        # User.objects.filter(username=username, password=password)
        # 存在返回True 否则None 内部相当于对密码加密之后再比对， username  字段是auth_user里面的
        user = authenticate(username=username, password=password)
        if user:  # 登录成功
            # 登录成功需要要调用一下login，此时表示用户登录了
            login(request, user)  # 存到session里的
            next_url = request.GET.get('next')
            if not next_url:
                next_url = '/auth_home/'
            return redirect(next_url)
        else:
            return HttpResponse('用户名或密码错误')

def auth_home(request):
    # 直接拿到当前登录用户
    print(request.user)  # 如果没有登录，显示的是 AnonymousUser 如果用户登录了(调用了login)，直接拿到用户
    print(request.user.username)
    print(request.user.id)
    if request.user.is_authenticated:
        print('用户登录了')
    else:
        print('用户没有登录')
    return render(request, 'auth_home.html')

# login_url如果不填会有一个默认的地址 accounts/login 的位置
@login_required(login_url='/auth_login/')
def auth_order(request):
    # 必须登录才能访问
    return HttpResponse('我是order, 必须登录才可能进入')

def auth_logout(request):
    logout(request)  # 注销
    # return HttpResponse('注销成功')
    return redirect('/auth_login/')

def auth_register(request):
    if request.method == 'GET':
        return render(request, 'register.html')
    else:
        username = request.POST.get('username')
        password = request.POST.get('password')
        # password是加密的，不能这样创建
        # User.objects.create(username=username, password=password)

        # 创建超级用户 email 字段需要传入
        # User.objects.create_superuser(username=username, password=password, email=None)

        # 创建普通用户
        User.objects.create_user(username=username, password=password)
        return redirect('/auth_login/')

def auth_check_password(request):
    user = User.objects.filter(pk=1).first()
    res = user.check_password('admin12345')
    print(res)
    return HttpResponse('测试页面')

def auth_set_password(request):
    user = User.objects.filter(pk=2).first()
    user.set_password('123456')
    user.save()
    return HttpResponse('测试页面2')
```

#### 前端

```html
<!--auth_home.html-->
<h1>这是home页面</h1>
{% if request.user.is_authenticated %} {{ request.user.username }} 欢迎你
<a href="/auth_logout/">注销登录</a>
{% else %}
<a href="/login_auth/">请去登录</a>
{% endif %}

<!--register.html-->
<h1>注册功能</h1>
<form action="" method="post">
  {% csrf_token %}
  <p>用户名: <input type="text" name="username" /></p>
  <p>密码: <input type="password" name="password" /></p>
  <input type="submit" value="提交" />
</form>

<!--login_auth.html-->
<form action="" method="post">
  {% csrf_token %}
  <p>用户名: <input type="text" name="username" /></p>
  <p>密码: <input type="password" name="password" /></p>
  <input type="submit" value="提交" />
</form>
```

### `User`对象属性

```python
User对象属性: username password
is_staff: 用户是否拥有网站的管理权限，如果没有，后台admin登录不进去
is_active: 是否允许用户登录, 设置为 False，可以在不删除用户的前提下禁止用户登录
```

### 扩展默认的`auth_user`表

#### 方式一

```python
再创建一张表，与User表一对一关系

from django.contrib.auth.models import User
class user_detail(models.Model):
    user=models.OneToOneField(to=User)
    phone=models.CharField(max_length=32)
```

#### 方式二

```python
from django.contrib.auth.models import AbstractUser
class BaseModel(models.Model):
    create_time = models.DateTimeField(auto_now_add=True)
    last_update_time = models.DateTimeField(auto_add=True)

    class Meta:
        abstract = True  # 此时这张表是抽象表，只能用来继承，不会在数据库生成表

class Book(BaseModel):
    name = models.CharField(max_length=32)

继承 AbstractUser 类扩写(不是继承authd的User，因为这个也是继承的 AbstractUser 类)
使用步骤:
  大前提是auth_user表没有创建之前进行
  models.py中写一个类，继承 AbstractUser 在类中扩写字段
  class MyAuthUser(AbstractUser):
    username = models.CharFaield(max_length=12)
    phone = models.CharField(max_length=32)
  # 引用Django自带的User表，继承使用时需要设置，在settings.py中配置
  AUTH_USER_MODEL = "app01.MyAuthUser"

# 如果auth_user表已经有了，需要扩写，需要删除之前数据库内的数据(危)
  1.删除库
  2.清空项目中所有migration的记录
  3.清空源码中admin，auth俩app的migration的记录
```
