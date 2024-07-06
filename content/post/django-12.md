---
title: Cookie和Session
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
  - Cookie
  - Session
categories:
  - Django
url: post/django-12.html
toc: true
---

### Cookie、Session 和 Token

HTTP 协议是一种`无状态协议`，即每次服务端接收到客户端的请求时，都是一个全新的请求，服务器并不知道客户端的历史请求记录；Session 和 Cookie 的主要目的就是为了弥补 HTTP 的无状态特性

<!-- more -->

#### Cookie

```python
服务端发送给浏览器的一小段数据(键值对的形式)，浏览器会对这块数据存储，浏览器在下一次发送请求的时候会将这段数据一起发送给服务端，从而判断请求是否来自同一个浏览器

cookie的大小上限为4KB
一个服务器最多在客户端浏览器上保存20个Cookie
一个浏览器最多保存300个Cookie

cookie本身存在客户端，容易被拦截或窃取
```

#### Session

```python
服务端想客户端随机分配一个随机字符串(session id)，用户存到cookie中
客户端发送带cookie(session id)请求到服务端，由服务端进行校验

服务端就要存储session，涉及到集群反向代理时，nginx可以通过ip_hash简单处理会话保持
更多的做法是使用Redis进行存储，单点或者集群
```

![image-20200928230826583](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/007S8ZIlgy1gj6rmwmekdj315w0jkq6j.jpg)

#### Token

```python
三段式(jwt:json web token)(服务端不存了)  后续jwt中详细
```

### Django 中 Cookie 的使用

```python
# urls.py
url(r'^get_cookie/', views.get_cookie),

# views.py
# 设置cookie
def set_cookie(request):
    obj = HttpResponse('cookie set 成功')
    # 往客户端浏览器中写cookie
    obj.set_cookie('name', 'xxx')
    obj.set_cookie('is_login', True)
    return obj

# 获取cookie
def get_cookie(request):
    name = request.COOKIES.get('name')
    is_login = request.COOKIES.get('is_login')
    print(name, is_login, type(is_login))  # 此时的 is_login 值是 str 类型
    return HttpResponse('获取 cookie')

# 更新和删除cookie
def update_cookie(request):
    obj = HttpResponse('cookie update 成功')
    # 往客户端浏览器中写cookie
    obj.set_cookie('name', 'yyy')  # 更新cookie
    obj.delete_cookie('is_login')  # 删除cookie
    return obj

# 设置过期时间
  浏览器会管理cookie 到时间会自动删除
  obj.set_cookie('name', 'yyy', expires=10)  # 10s后浏览器会自动删除cookie
  如果不写该参数，关闭浏览器，cookie就会失效

# 设置加盐的cookie
  obj.set_signed_cookie('age', 18, '666', expires=1000)

# 获取加盐的cookie
  age = request.get_signed_cookie('age', salt='666')
```

![image-20220303180033188](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220303180033188.png)

### Django 中 Session 的使用

```python
# session 默认是存在数据库中: 必须进行数据迁移，存在 django_session 表中
# makemigrations  migrate

# 设置session
def set_session(request):
    request.session['name'] = 'sxxx'
    request.session['is_login'] = True
    '''
    1、生成随机字符串，一旦有，就是更新操作
    2、把随机字符串放到cookie中
    3、把 name 放入 django_session 表中
    session_key  session_data date
    过期时间是默认两周(可在全局配置文件中修改)
    '''
    return HttpResponse('session 设置成功')

# 获取session
def get_session(request):
    # 取出携带的cookie
    # 在进入视图函数之前，早已经吧session从django_session表中session_data取出来
    # 转到了session中
    name = request.session.get('name')
    is_login = request.session.get('is_login')
    print(name, is_login, type(is_login))  # 此时的is_login的类型是布尔值，因为是存在数据库中的
    return HttpResponse('获取 session')

# 更新session
def update_session(request):
    request.session['name'] = 'syyy'
    # 删除session
    del request.session['is_login']

    # request.session.delete() # 删除数据库
    # request.session.flush()  # cookie和数据库都删
    '''
    直接去django_session表中替换
    cookie还是原来的，没有动
    '''
    return HttpResponse('session 更新成功')
```

![image-20220303200432533](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220303200432533.png)

![image-20220303200451707](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220303200451707.png)

![image-20220303200706867](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220303200706867.png)

### 其他补充

#### Cookie 的其他参数

```python
key, 键
value='', 值
max_age=None, 超时时间 cookie需要延续的时间（以秒为单位）如果参数是\ None ，这个cookie会延续到浏览器关闭为止
expires=None, 超时时间(IE requires expires, so set it if hasn’t been already.)

path='/' Cookie生效的路径，/ 表示根路径，特殊的：根路径的cookie可以被任何url的页面访问，浏览器只会把cookie回传给带有该路径的页面，这样可以避免将cookie传给站点中的其他的应用

domain=None, Cookie生效的域名 你可用这个参数来构造一个跨站cookie
如: domain=”.example.com”所构造的cookie对下面这些站点都是可读的：www.example.com、www2.example.com 和an.other.sub.domain.example.com
如果该参数设置为 None ，cookie只能由设置它的站点读取

secure=False, 浏览器将通过HTTPS来回传cookie

httponly=False 只能http协议传输，无法被JavaScript获取（不是绝对，底层抓包可以获取到也可以被覆盖）
```

#### Session 的其他方法

```python
# 获取、设置、删除Session中数据
request.session['k1']
request.session.get('k1',None)
request.session['k1'] = 123
request.session.setdefault('k1',123) # 存在则不设置
del request.session['k1']


# 所有 键、值、键值对
request.session.keys()
request.session.values()
request.session.items()
request.session.iterkeys()
request.session.itervalues()
request.session.iteritems()

# 会话session的key(随机字符串)
request.session.session_key

# 将所有Session失效日期小于当前日期的数据删除
request.session.clear_expired()

# 检查会话session的key在数据库中是否存在
request.session.exists("session_key")

# 删除当前会话的所有Session数据(只删数据库)
request.session.delete()
　　
# 删除当前的会话数据并删除会话的Cookie（数据库和cookie都删）。
request.session.flush()
    这用于确保前面的会话数据不可以再次被用户的浏览器访问

# 设置会话Session和Cookie的超时时间
request.session.set_expiry(value)
    * 如果value是个整数，session会在些秒数后失效。
    * 如果value是个datatime或timedelta，session就会在这个时间后失效。
    * 如果value是0,用户关闭浏览器session就会失效。
    * 如果value是None,session会依赖全局session失效策略。
```

#### session 的其他配置(全局 setting)

```python
from django.conf import global_settings

1. 数据库Session
SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）

2. 缓存Session
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置

3. 文件Session
SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()

4. 缓存+数据库
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎

5. 加密Cookie Session
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎

其他公用设置项：
SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
SESSION_COOKIE_AGE = 60 * 60 * 24 * 7 * 2                             # Session的cookie失效日期（2周）（默认）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）
```

```python
登录功能，如果登录成功往cookie中写入用户名和登录成功的状态标志
访问order页面，如果登录了，可以正常显示，如果没登录，重定向到登陆页面
基于session的登陆认证装饰器

# views.py
def auth(func):
    def inner(request, *args, **kwargs):
        if request.session.get('is_login'):
            return func(request, *args, **kwargs)
        else:
            return redirect('/login/')

    return inner


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


@auth
def order(request):
    if request.method == 'GET':
        return render(request, 'order.html')
    # if request.session.get('is_login'):
    #     return render(request, 'order.html')
    # else:
    #     return redirect('/login/')
```
