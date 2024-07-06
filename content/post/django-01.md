---
title: 手撸简易的web框架
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
categories:
  - Django
url: post/django-01.html
toc: true
---

### HTTP 协议

> 规定了浏览器与服务端之间的数据交互格式

<!-- more -->

#### 四大特性

```python
1.基于TCP/IP作用于应用层之上的协议
2.基于请求响应
3.无状态
  保存状态的的技术有cookie、session、token
4.无(短)连接
  长连接:websocket
```

#### 数据格式

```python
1.请求数据格式:
  请求首行(包含有请求方法...)
  请求头(K:V键值对信息)
  \r\n
  请求体(主要用来携带敏感数据)

2.响应数据格式:
  响应首行(响应状态码)
  响应头(K:V键值对)
  \r\n
  响应体(展示给用户的数据)
```

#### 响应状态码

```python
用一串数字表示信息
1xx:服务端已经接收到数据并正在处理，可以继续提交
2xx:200请求成功
3xx:重定向
4xx:403 404
5xx:服务器内部错误

响应状态码可以自定义
```

### 请求方法

```python
1.get请求
	从指定资源请求数据

2.post请求
  将数据发送到服务器

'''
get请求: url?username=xxx&password=123456
post请求:数据是放在请求体内
'''
```

### 手写 web 框架

```python
import socket

server = socket.socket()
server.bind(('127.0.0.1', 8000))
server.listen(5)

while True:
    conn, add = server.accept()
    data = conn.recv(1024)
    # print(data)
    res = data.decode('utf8')
    path = res.split(' ')[1]
    # 发送响应头
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')
    if path == '/index':
        # conn.send(b'index')
        # 发送html页面
        with open(r'demo1.html', 'rb') as f:
            data = f.read()
            conn.send(data)
    elif path == '/login':
        conn.send(b'login')
    else:
        conn.send(b'404 error')
    # conn.send(b'hello world')
    conn.close()
```

### 基于`wsgiref`模块

```python
from wsgiref.simple_server import make_server


def run(request, response):
    response('200 OK', [])
    current_path = request.get("PATH_INFO")
    if current_path == '/index':
        return [b'index']
    elif current_path == '/login':
        return [b'login']
    else:
        return [b'404 error']


if __name__ == '__main__':
    server = make_server('127.0.0.1', 8000, run)
    server.serve_forever()
```

### 封装处理

```python
1.定义一个网址与函数的对应关系 urls.py
from views import *

urls = [
    ('/index', index_func),
    ('/login', login_func),
    ('/reg', reg_func),
]

2.定义函数功能 views.py
def index_func(request):
    return 'index'
def login_func(request):
    return 'login'
def reg_func(request):
    return 'reg'
def errors(request):
    return '404 error'

3.服务端文件 server.py
from wsgiref.simple_server import make_server
import urls
from views import *

def run(request, response):
    response('200 OK', [])
    current_path = request.get("PATH_INFO")
    func = None
    for url_tuple in urls:
        if current_path == url_tuple[0]:
            func = url_tuple[1]
            break
    if func:
        res = func(request)
    else:
        res = errors(request)
    return [bytes(res, encoding='utf8')]
if __name__ == '__main__':
    server = make_server('127.0.0.1', 8000, run)
    server.serve_forever()
```

### 动静态网页

```python
# 静态网页
	数据全是写死的
# 动态网页
  数据来源于后端(代码、数据库)
```

- 访问展示当前时间(时间是后端模块生成)

  ```python
  # urls.py 添加
    ('/get_time', get_time_func)

  # views.py中对应的功能函数
    def get_time_func(request):
        import time
        current_time = time.strftime('%Y-%m-%d %X')
        with open(r'demo1.html', 'r', encoding='utf8') as f:
            data = f.read()
        data = data.replace('asdasdasd', current_time)
        return data

  # demo1.html
    <h1>asdasdasd</h1>
  ```

- 后端字典展示

  ```python
  pip3 install jinja2

  # urls.py 添加
    ('/get_dict', get_dict_func)

  # views.py中对应的功能函数
    def get_dict_func(request):
        user_dict = {'name': 'jason', 'age': 18, 'password': 123456}

        with open(r'Template/dem02.html', 'r', encoding='utf8') as f:
            data = f.read()
        temp = Template(data)
        # 将user_dict 传递给demo2.html页面 在该页面上使用变量名user_data
        res = temp.render(user_data=user_dict)
        return res

  # Template/dem02.html
    {{ user_data }} <br>
    {{ user_data.get('name') }} <br>
    {{ user_data['age'] }} <br>
    {{ user_data.password }}
  ```

- 获取 MySQL 数据库数据

  ```python
  # urls.py 添加
    ('/get_sql', get_sql_func)

  # views.py 中对应的功能函数
    def get_sql_func(request):
        import pymysql
        conn = pymysql.connect(
            host='10.0.0.60',
            port=3306,
            user='root',
            password='123456',
            db='db1',
            charset='utf8',
            autocommit=True
        )
        cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
        sql = 'select * from student'
        affect_rows = cursor.execute(sql)
        res1 = cursor.fetchall()  # [{},{},{}]
        with open(r'Template/db.html', 'r', encoding='utf8') as f:
            data = f.read()  # 字符串
        temp = Template(data)
        # 将user_dict传递给get_dict.html页面 在该页面上使用变量名user_data调用
        res = temp.render(data_list=res1)
        return res

  # Template/db.html
  <div class="container">
      <div class="row">
          <h1 class="text-center">学生数据</h1>
          <div class="col-md-8 col-md-offset-2">
              <table class="table table-hover table-striped table-bordered">
                  <thead>
                  <tr>
                      <td>学号</td>
                      <td>姓名</td>
                      <td>年龄</td>
                      <td>性别</td>
                  </tr>
                  </thead>
                  <tbody>
                  {%for user_dict in data_list%}
                      <tr>
                          <td>{{user_dict.id}}</td>
                          <td>{{user_dict.name}}</td>
                          <td>{{user_dict.age}}</td>
                          <td>{{user_dict.gender}}</td>
                      </tr>
                  {%endfor%}
                  </tbody>
              </table>
          </div>
      </div>
  </div>
  ```

### 总结

```python
wsgiref模块
  1. 封装了socket代码
  2. 处理了http数据格式

根据功能的不同拆分成不同的文件
urls.py    路由与视图函数对应关系
views.py   视图函数
templates  模板文件夹
# 1.第一步添加路由与视图函数的对应关系
# 2.去views中书写功能代码
# 3.如果需要使用到html则去模板文件夹中操作

jinja2模板语法
{{}}
{%%}
```

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/2240626-20210312230334847-1372533552.png)

### 主流的 web 框架

```python
Django框架
  大而全，自带的功能组件非常多，但是缺点就是太重了

Flask框架
  小而精，自带组件少，第三方模块非常多，但是第三方模块比较难管理

Tornado框架
  异步非阻塞 速度非常快

A:socket部分
B:路由与视图匹配
C:模板语法

django
   A:用的是wsgiref模块
   B:自己写的
   C:自己写的
flask
   A:用的是wsgiref模块封装之后werkzeug
   B:自己写的
   C:jinja2模块
tornado
	A、B、C都是自己写的
```
