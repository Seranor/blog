---
title: Django中间件
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-13.html
toc: true
---

### `CBV`加装饰器

```python
方式一:
  装饰器加在类上
  from django.utils.decorators import method_decorator
  @method_decorator(auth, name='get')

方式二:
  装饰器直接加在方法上
```

<!-- more -->

```python
from django.shortcuts import render, HttpResponse, redirect
from app01 import models
from django.views import View

# 登录认证(判断session)功能的装饰器
def auth(func):
    def inner(request, *args, **kwargs):
        if request.session.get('is_login'):
            return func(request, *args, **kwargs)
        else:
            return redirect('/login/')
    return inner

# 登录成功创建session
def login(request):
    if request.method == 'GET':
        return render(request, 'login.html')
    else:
        name = request.POST.get('name')
        password = request.POST.get('password')
        res = models.User.objects.filter(name=name, password=password).first()
        if res:
            request.session['name'] = name
            request.session['password'] = password
            request.session['is_login'] = True
            return redirect('/order/')
        else:
            return HttpResponse('用户名或密码不正确')

# 需要引入该模块
from django.utils.decorators import method_decorator

# 给 get 请求加 auth 装饰器
@method_decorator(auth, name='get')
class Index(View):
    # 装饰器也可以直接加在该方法上，不需要使用说明的模块
    # @auth
    def get(self, request, *args, **kwargs):
        return HttpResponse('index')
```

### `Djanog`中间件

```python
中间件: 数据库中间件（mycat，分库分表），服务器中间件（tomcat，nginx），消息队列中间件（rabbitmq）
django中间件(Middleware): 介于request与response处理之间的一道处理过程，在全局上改变django的输入与输出
```

#### 内置的中间件

```python
# settings.py文件中
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    # 处理session的中间件
    'django.contrib.sessions.middleware.SessionMiddleware',
    # 处理 访问地址尾部会自带加/
    'django.middleware.common.CommonMiddleware',
    # 跨站请求伪造处理
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]


# 写入中间件就是 django.contrib.sessions.middleware.SessionMiddleware
from django.middleware.csrf import  CsrfViewMiddleware
# django.contrib.sessions.middleware.SessionMiddleware
from django.contrib.sessions.middleware import SessionMiddleware


process_request(self, request)  # 请求来了触发
process_response(self, request, response)  # 请求走了触发
```

#### 自定义中间件

```python
参考settings.py中的处理方式，因此可以新创建一个py文件中写
app01目录(其他目录也可以) --> middleware.py

# app01/middleware.py
# 其他中间件也继承，参考其他的中间件源码编写方式
from django.utils.deprecation import MiddlewareMixin
class MyMiddleware(MiddlewareMixin):
    def process_request(self, request):
        print('请求来了')
       	return

    def process_response(self, request, response):
        print('请求走了')
        return response  # 一定要返回 response

# settings.py
MIDDLEWARE =[
	...
  # 注册中间件
  'app01.middleware.MyMiddleware',
  ...
]
# from app01.middleware import MyMiddleware --> app01.middleware.MyMiddleware
```

##### 中间件的执行顺序

```python
# app01/middleware.py
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import render, HttpResponse, redirect

class MyMiddleware(MiddlewareMixin):
    def process_request(self, request):
        print('请求来了')
        # return HttpResponse('截胡了')

    def process_response(self, request, response):
        print('请求走了')
        return response

class MyMiddleware2(MiddlewareMixin):
    def process_request(self, request):
        print('请求来了222')

    def process_response(self, request, response):
        print('请求走了222')
        return response

# settings.py
MIDDLEWARE =[
	...
  # 注册中间件
  'app01.middleware.MyMiddleware',
  'app01.middleware.MyMiddleware2',
  ...
]

# 中间件的执行顺序: 请求来从上往下，请求走从下往上，当在process_request中return走了，后续的中间件就不会再执行了
```

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/1342004-20180626145355317-56223999.png)

##### `process_request`

```python
请求来了会触发，从上往下依次执行(settings.py中间件注册的位置)
如果返回None就继续往下走
如果返回四件套之一，就直接回去了

在这里面写请求来了的一些判断: 如判断ID地址以及客户端地址
	request.META.REMOTE_ADDR      #客户端地址
	request.META.HTTP_USER_AGENT  # 客户端类型
  request.META.get('HTTP_X_FORWARDED_FOR')  # 获取到是否存在代理IP
  print(request.META)  # 查看更多的字段信息
```

```python
HttpRequest.META
   一个标准的Python 字典，包含所有的HTTP 首部。具体的头部信息取决于客户端和服务器，下面是一些示例：
　　取值：
    CONTENT_LENGTH        —— 请求的正文的长度(是一个字符串)
    CONTENT_TYPE          —— 请求的正文的 MIME 类型
    HTTP_ACCEPT           —— 响应可接收的Content-Type
    HTTP_ACCEPT_ENCODING  —— 响应可接收的编码
    HTTP_ACCEPT_LANGUAGE  —— 响应可接收的语言
    HTTP_HOST             —— 客服端发送的HTTP Host 头部
    HTTP_REFERER          —— Referring 页面
    HTTP_USER_AGENT       —— 客户端的user-agent 字符串
    QUERY_STRING          —— 单个字符串形式的查询字符串（未解析过的形式）
    REMOTE_ADDR           —— 客户端的IP 地址
    REMOTE_HOST           —— 客户端的主机名
    REMOTE_USER           —— 服务器认证后的用户
    REQUEST_METHOD        —— 一个字符串，例如"GET" 或"POST"
    SERVER_NAME           —— 服务器的主机名
    SERVER_PORT           —— 服务器的端口（是一个字符串）

 　　从上面可以看到，除 CONTENT_LENGTH 和 CONTENT_TYPE 之外，请求中的任何 HTTP 首部转换为 META 的键时，都会将所有字母大写并将连接符替换为下划线最后加上 HTTP_  前缀，所以，一个叫做 X-Bender 的头部将转换成 META 中的 HTTP_X_BENDER 键
```

##### `process_response`

```python
请求走了会触发，从下往上执行
在最后一定要  return response

需求:
  在所有的响应中都写入cookie    response.set_cookie('name', 'xxx')
  在所有的响应头中都写入xxx     response['x-head']='xxx'
```

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/007S8ZIlgy1gj6ropdwwmj31xw0tik2x.jpg)

##### `provess_view`

```python
Django会在调用视图函数之前调用process_viwe方法

它应该返回None或者一个HttpResponse对象
如果返回None，Django讲继续处理这个请求，执行任何其他中间件的process_view方法，然后执行响应的视图
如果返回一个HttpResponse对象，Django不会调用适当的视图函数，将执行中间件的process_response方法并将应用到该HttpResponse并返回结果
```

```python
def process_view(self, request, view_func, view_args, view_kwargs):
        # view_func 视图函数
        # view_args,  位置参数
        # view_kwargs 关键字参数
        print('我是process view')

        # 如果return None，会执行视图函数
        #手动执行了视图函数
        # response=view_func(request,view_args, view_kwargs)
        # 返回response，视图函数就不执行了
        return HttpResponse('ddddd')
```

##### `process_exception`

```python
这个方法只有在视图函数中出现异常了才执行

def process_exception(self, request, exception):
    #记录错误日志
    print(exception)
    print('出错了')
```

![image-20200928231435070](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/007S8ZIlgy1gj6rt8vwvjj30ue0k2wh6.jpg)

##### `porcess_template_response`

```python
该方法对视图函数返回值有要求，必须是一个含有render方法类的对象，才会执行此方法

def process_template_response(self,request,response):
    print('我执行了')
    return response
class Test:
    def __init__(self,status,msg):
        self.status=status
        self.msg=msg
    def render(self):
        import json
        dic={'status':self.status,'msg':self.msg}

        return HttpResponse(json.dumps(dic))
def index(response):
    return Test(True,'测试')
```

### `CSRF_TOKEN`跨站请求伪造

```python
https://www.cnblogs.com/liuqingzheng/p/9505044.html

CSRF或者XSRF：跨站请求伪造
攻击者盗用了你的身份，以你的名义发送恶意请求，对服务器来说这个请求是完全合法的但是却完成了攻击者所期望的一个操作，比如以你的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。 如下：其中Web A为存在CSRF漏洞的网站，Web B为攻击者构建的恶意网站，User C为Web A网站的合法用户

防范：CSRF攻击防范
    Referer：上一次访问的地址（图片防盗链）
    - 在请求地址中添加 token 并验证
    - 在 HTTP 头中自定义属性并验证
    - 把随机字符串放在请求体中
```

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/mmm.png)

### `Django`处理`CSRF`

```python
settings.py中不要注释csrf的中间件
```

`form`表单中处理

```python
<form action="" method="post">
    {% csrf_token %}
    <p>给谁转：<input type="text" name="to_user" id="id_name"></p>
    <p>转多少：<input type="text" name="money" id="id_money"></p>
    <input type="submit" value="转账">

</form>
```

`Ajax`处理

```javascript
$.ajax({
  url: "/transfer/",
  method: "post",
  //data: {to_user: $('#id_name').val(), money: $('#id_money').val(),csrfmiddlewaretoken:$('[name="csrfmiddlewaretoken"]').val()},
  data: {
    to_user: $("#id_name").val(),
    money: $("#id_money").val(),
    csrfmiddlewaretoken: "{{csrf_token}}",
  },
  success: function (data) {
    console.log(data);
  },
});

// 两种方式放入data中
// csrfmiddlewaretoken:$('[name="csrfmiddlewaretoken"]').val()
// csrfmiddlewaretoken:'{{csrf_token}}'
```

#### `cookie`的处理

```python
# 需要引入这个js文件
<script src="/static/jquery.cookie.js"></script>
var token=$.cookie('csrftoken')
```

#### `csrf`放到请求头中

```python
 $.ajax({
            url: '',
            headers:{'X-CSRFToken':token},
            type: 'post',
            data: {
                'name': $('[name="name"]').val(),
                'password': $("#pwd").val(),
            },
            success: function (data) {
                console.log(data)
            }

        })
```

#### `csrf`的控制使用

```python
在视图函数上加装饰器

from django.views.decorators.csrf import csrf_exempt,csrf_protect
# 不再检测，局部禁用（前提是全站使用）
# @csrf_exempt
# 检测，局部使用（前提是全站禁用）
@csrf_protect
def csrf_token(request):
```

### `Django`启动流程

```python
配置文件中指定WSGI_APPLICATION = 'mysite.wsgi.application'
被wsgi服务器管理，一旦有请求进来，会触发application()
实际触发WSGIHandler类的__call__传入environ, start_response
把environ包装成request对象，执行中间件，执行路由，执行视图函数，返回response
最终结束django
```
