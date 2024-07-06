---
title: 爬虫简介和requests模块的使用
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - crawler
categories:
  - crawler
url: post/crawler-01.html
toc: true
---

### 爬虫介绍

<!-- more -->

```python
# 写后台--->前端展示数据---》浏览器发送http请求，从后端服务器获取的--》只能从浏览器中看---》看到好看的东西---》保存到本地---》存到我们自己库中----》爬虫


# 百度本质就是一个大爬虫(搜索)，在输入框中输入搜索内容，实际是从百度的数据库搜索出来的---》
# 百度数据库的数据是从互联网爬下来的--》百度这个爬虫一刻不停的在互联网爬数据--》爬完就存到它的库里（seo优化--》优化我们的网站能够被搜索引擎先搜到，排的很靠前）--->尽可能被百度爬虫，并且容易搜索到---->seo(免费的)和sem(充钱，把你放前面)---》莆田系医院---->百度快照---》当时爬虫爬取这个网页这一刻，网页的样子---》保留这个网页地址---》当你点击标题--》跳转到这个网页--》完成了你的搜索   伪静态


# 爬虫的本质----》模拟发送http请求(浏览器携带什么，我们也要携带什么)---->服务器返回数据---》对数据进行清洗---》入库   后续操作是别的---》分析数据--》数据分析

# 爬虫协议：君子协议---》大家遵循这个协议，就不违法--》规定了我的网站，什么能爬，什么不能爬
https://www.baidu.com/robots.txt
https://www.cnblogs.com/robots.txt

# 爬虫本质跟用浏览器访问没什么区别
```

### `requests`模块

```python
# 安装
pip install requests
```

#### 发送 get 请求

```python
import requests

# 直接请求
res1 = requests.get('https://www.baidu.com')
print(res.text)

# 带请求参数
ip =  '127.0.0.1'
# https://www.svlik.com/t/ipapi/ip.php/?ip=127.0.0.1
res2 = requests.get('https://www.svlik.com/t/ipapi/ip.php',params={'ip': ip})
# params内可以多个，相当于拼接 ?name=lzj&age=18
print(res2.text)

```

#### 编码和解码

```python
# https://www.baidu.com/baidu?wd=python%E4%B8%ADurl%E7%BC%96%E7%A0%81%E5%92%8C%E8%A7%A3%E7%A0%81
# 这个 url 在浏览器中后面的字符就会变成中文的


# 把字典中的中文字段编码
from urllib.parse import urlencode
d = {"wd": 'python中url编码和解码'}
res = urlencode(d)  # wd=python%E4%B8%ADurl%E7%BC%96%E7%A0%81%E5%92%8C%E8%A7%A3%E7%A0%81
print(res)


# 只对中文编码和解码
from urllib.parse import quote, unquote
name1 = "编码和解码"
print(quote(name1))  # %E7%BC%96%E7%A0%81%E5%92%8C%E8%A7%A3%E7%A0%81

name2 = "wd=python%E4%B8%ADurl%E7%BC%96%E7%A0%81%E5%92%8C%E8%A7%A3%E7%A0%81"
print(unquote(name2))  # wd=python中url编码和解码
```

#### 携带请求头

```python
import requests

# 该请求被禁止 因为没有携带请求头
res = requests.get('https://dig.chouti.com/')
print(res.text)


# 携带请求头
# 请求头
header = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36",
    "Accept-Encoding": "gzip, deflate, br",
}

res = requests.get('https://dig.chouti.com/', headers=header)
# 可以写入到文件里
with open('chouti.html', 'wb') as f:
    f.write(res.content)
print(res.text)

'''
django 中从 request.META 中取出
参数说明
User-Agent  客户端浏览器类型版本信息等
Referer     上一次请求的地址 功能: 反爬 图片防盗链
'''
```

#### 携带`cookie`

```python
# cookie 本身是请求头里面的参数
# 就可以放在请求头中，但是 cookie 经常使用，也可以用单独的 cookies 参数

# 模拟点赞
# 注册登录 https://dig.chouti.com/ 在浏览器上拿到 cookie

header = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36",
    "Cookie": ""
}
data = {
    "linkId": "34946008",  # 点赞连接的Payload信息
}

# cookies={'key':'value'}  cookie 里面 xx=xxx 改
# res = requests.post('https://dig.chouti.com/link/vote',data=data,headers=header,cookies={'key':'value'})
res = requests.post('https://dig.chouti.com/link/vote',data=data,headers=header)
print(res.text)
```

#### 发送`POST`请求模拟登录

```python
# 使用 cookie
import requests

data = {
    'username': '616564099@qq.com',
    'password': 'lqz123',
    'captcha': 'qm4s',
    'remember': 1,
    'ref': 'http://www.aa7a.cn/',
    'act': 'act_login',
}

res = requests.post("http://www.aa7a.cn/user.php",data=data)
print(res.text)
print(res.cookies)  # 登录成功后返回的cookies 以后可以拿着这个cookie 登录

res2 = requests.get("http://www.aa7a.cn/",cookies=res.cookies)
print('616564099@qq.com' in res2.text)

# 每次都需要手动携带cookie  因此使用requests 提供的session

# 使用 session
import requests

session = requests.session()
data = {
    'username': '616564099@qq.com',
    'password': 'lqz123',
    'captcha': 'qm4s',
    'remember': 1,
    'ref': 'http://www.aa7a.cn/',
    'act': 'act_login',
}
res = session.post("http://www.aa7a.cn/user.php",data=data)
print(res.text)
res2 = session.get("http://www.aa7a.cn/")
print('616564099@qq.com' in res2.text)
```

#### 响应对象

```python
import requests
respone=requests.get('http://www.jianshu.com')

# respone属性
print(respone.text)      # 返回响应体的文本内容
print(respone.content)   # 返回响应体的二进制内容

print(respone.status_code)  # 响应状态码
print(respone.headers)      # 响应头
print(respone.cookies)      # 响应的cookie
print(respone.cookies.get_dict())  # 响应的cookie转成字典
print(respone.cookies.items())

print(respone.url)           # 请求地址
print(respone.history)       #  如果有重定向，列表，放着重定向之前的地址

print(respone.encoding)      # 页面的编码方式：utf-8   gbk
# response.iter_content()    # content迭代取出content二进制内容，一般用它存文件
```

#### 编码问题

```python
# 请求回来的数据，res.text 打印的时候乱码 ，因为没有指定编码，默认已utf-8编码
# 解决方法
response.encoding='gbk'
```

#### 获取二进制数据

```python
# 下载图片 视频
import requests

header = {
    'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36'
}

res = requests.get('https://tva1.sinaimg.cn/mw2000/9d52c073gy1h1v6lmny8nj20j60pjtdh.jpg', headers=header)
with open('mzt.jpg', 'wb') as f:
    # f.write(res.content)
    for line in res.iter_content(100):  # 每次获取多少字节
        f.write(line)
```

#### `json`格式解析

```python
import json
import requests

data = {
    'cname': '',
    'pid': '',
    'keyword': '上海',
    'pageIndex': 1,
    'pageSize': 10,
}

# res = requests.post('http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?op=keyword',data=data)
# res2 = json.loads(res.text)
# print(type(res2))  # 自动类型
# print(res2['Table'][0]['rowcount'])


res = requests.post('http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?op=keyword',data=data).json()
print(type(res))
print(res['Table'][0]['rowcount'])
```

#### `ssl`认证

```python
# 之前网站，有些没有认证过的ssl证书，我们访问需要手动携带证书
# 跳过证书直接访问

import requests
respone=requests.get('https://www.12306.cn',verify=False) # 不验证证书,报警告,返回200
print(respone.status_code)

# 手动携带
import requests
respone=requests.get('https://www.12306.cn',
                     cert=('/path/server.crt',
                           '/path/key'))
print(respone.status_code)
```

#### 使用代理

```python
# 爬虫，速度很快，超过频率限制，使用代理，切换ip，封ip封的也是代理的ip，自己的ip没问题

import requests
proxies = {
    'http': '39.103.217.44:59886',  # http或https代理
}
respone = requests.get('https://www.baidu.com', proxies=proxies)

print(respone.status_code)
```

##### 请求返回 IP 地址的`web`

```python
pip3 install django

django-admin startproject crawler_web
cd crawler_web
python3 manage.py startapp app01
```

```python
# crawler_web/app01/views.py
import json
import requests
from django.http import JsonResponse


# 根据 IP 返回 IP 归属地信息
def ipInfo(ip):
    res = requests.get('https://www.svlik.com/t/ipapi/ip.php', params={'ip': ip}).json()
    country = res.get("country")
    area = res.get("area")
    return country, area


def showIP(request):
    xff = request.META.get('HTTP_X_FORWARDED_FOR')  # 反向代理IP
    remote_addr = request.META.get('REMOTE_ADDR')   # 真实 IP
    if xff:
        country, area = ipInfo(xff)
        info = {"IP": xff, "country": country, "area": area}
        return JsonResponse(info, json_dumps_params={'ensure_ascii': False})
    country, area = ipInfo(remote_addr)
    info = {"IP": remote_addr, "country": country, "area": area}
    return JsonResponse(info, json_dumps_params={'ensure_ascii': False})


# crawler_web/crawler_web/urls.py
from django.contrib import admin
from django.urls import path
from app01 import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('ip/', views.showIP),
]
```

sqlite3 版本过低解决办法

```shell
 wget https://www.sqlite.org/2019/sqlite-autoconf-3300100.tar.gz
 tar xf sqlite-autoconf-3300100.tar.gz
 cd sqlite-autoconf-3300100/
 ./configure --prefix=/usr/local
 make && make install
 mv /usr/bin/sqlite3 /usr/bin/sqlite3_old
 ln -s /usr/local/bin/sqlite3 /usr/bin/sqlite3
 echo "/usr/local/lib" > /etc/ld.so.conf.d/sqlite3.conf
 ldconfig
 sqlite3 -version
```

```shell
# 启动
python3 manage.py runserver 0.0.0.0:8000
```

##### 搭建代理池

```python
# https://github.com/jhao104/proxy_pool
python的爬虫+flask写的
本质使用爬虫技术爬取免费的代理，requests模块验证代理是否可以，然后存到redis中
起一个web服务器，只要访问一个地址，他就随机给你一个ip地址

# 步骤
git clone git@github.com:jhao104/proxy_pool.git
pip install -r requirements.txt


'''
  # setting.py 为项目配置文件
  # 配置API服务

  HOST = "0.0.0.0"               # IP
  PORT = 5000                    # 监听端口

  # 配置数据库

  DB_CONN = 'redis://:pwd@127.0.0.1:8888/0'

  # 配置 ProxyFetcher

  PROXY_FETCHER = [
      "freeProxy01",      # 这里是启用的代理抓取方法名，所有fetch方法位于fetcher/proxyFetcher.py
      "freeProxy02",
      # ....
  ]

'''

# 启动爬虫
    python3 proxyPool.py schedule

# 启动webApi服务
    python proxyPool.py server

# 可以使用 docker-compose 直接启动
```

##### 使用代理池

```python
import json
import requests

# 将代理池部署到服务器上  请求参数参考 github 上
ip = requests.get('http://139.155.237.73:5010/get/').json()['proxy']

proxies = {
    "http": ip,
}

res = requests.get('http://42.194.173.230:8000/ip/', proxies=proxies)

print(res.text)
```

#### 超时设置

```python
import json
import requests

# 将代理池部署到服务器上  请求参数参考 github 上
ip = requests.get('http://139.155.237.73:5010/get/').json()['proxy']

proxies = {
    "http": ip,
}

res = requests.get('http://42.194.173.230:8000/ip/', proxies=proxies, timeout=3)  # 3s 后无反应抛出异常

print(res.text)
```

#### 异常处理

```python
from requests.exceptions import *  # 可以查看requests.exceptions获取异常类型

ip = requests.get('http://139.155.237.73:5010/get/').json()['proxy']

proxies = {
    "http": ip,
}
try:
    res = requests.get('http://42.194.173.230:8000/ip/', proxies=proxies, timeout=3)
    print(res.text)         # 内容
    print(res.status_code)  # 状态码
except ReadTimeout:         # timeout=3
    print("ReadTimeout")
except ConnectionError:     # 网络不通
    print("Network Error")
except Timeout:
    print('Timeout')
except Exception:
    print("Error")
```

#### 认证设置

```python
# 这种很少见，极个别公司内部可能还用这种

import requests
from requests.auth import HTTPBasicAuth
r=requests.get('xxx',auth=HTTPBasicAuth('user','password'))
print(r.status_code)

# HTTPBasicAuth 可以简写为如下格式
import requests
r=requests.get('xxx',auth=('user','password'))
print(r.status_code)
```

#### 上传文件

```python
# 上传文件,爬虫一般不会用，但是需要使用服务的时候会用 调用别人的

import requests
files={'file':open('a.jpg','rb')}
respone=requests.post('http://httpbin.org/post',files=files)
print(respone.status_code)

# django项目
  公司中项目，使用了第三方服务，第三放服务提供了api接口，没提供sdk
  就要使用request发送请求，获取数据

# 前端提交一个长链地址  www.cnblogs.com/liuqingzheng/p/233.html ---> 转成短链 ---> x.com/asdf--->存到自己数据库中
# 专门服务，处理长连转短链(go,java ---> 接口 ---> post，带着地址 ---> 返回json，短链地址
```
