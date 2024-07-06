---
title: DRF初探
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
  - DRF
categories:
  - Django
url: post/django-15.html
toc: true
---

### Web 应用模式

```python
前后端混合开发（前后端不分离）：返回的是html的内容，需要写模板
前后端分离：只专注于写后端接口，返回json，xml格式数据
```

<!-- more -->

#### 前后端不分离

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/002.jpg)

#### 前后端分离

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/001.jpg)

### `API`接口

```python
API接口: 规定了前后端信息交互规则的url链接，也就是前后台信息交互的媒介
接口文档: 手动编写或者自动生成(coreapi、swagger)
```

### `RESTful`规范

```python
1. 数据的安全保障，通常使用https进行传输
2. 域名(api的标识)
   https://api.example.com/      # 尽量部署在API专用域名下
   https://www.example.com/api/
3. 请求地址带版本，或者在请求头中
   https://api.example.com/v1/
   https://www.example.com/api/v1
4. 任何东西都是资源，均用名词表示(尽量不要用动词)
   https://api.example.com/v1/books/
5. 通过请求方式区分不同操作
   get: 获取数据
   post: 新增数据
   put/patch: put是全部更新，patch是局部更新
   delete: 删除
6. 在请求路径中带过滤
   https://api.example.com/v1/?name='三'&order=asc
7. 返回数据中带状态码
   http请求状态码(2xx 3xx 4xx 5xx)
   返回的JSON格式中带状态码(标志当次请求成功或失败)
8. 返回数据中带错误信息
   错误处理，应返回错误信息，error当做key
9. 对不同的操作，返回数据符合如下规范
   GET https://api.example.com/v1/books/       # 返回资源对象的列表(数组) [{},{}]
   GET https://api.example.com/v1/books/1/     # 返回单个资源对象 {}
   POST https://api.example.com/v1/books/      # 返回新生成的资源对象 {新增的数据}
   PUT  https://api.example.com/v1/books/1/    # 返回完整的资源对象  {返回修改后的数据}
   PATCH https://api.example.com/v1/books/1/   # 返回完整的资源对象 {返回修改后的数据}
   DELETE https://api.example.com/v1/books/1/  # 返回一个空文档
   # {status:100,msg:查询成功,data:null}
10 返回结果中带连接
```

### `postman`的使用

```python
后端写好之后需要测试，因此需要一个工具测试接口，常用的postman
下载地址: https://www.postman.com/
```

![image-20220314110921244](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220314110921244.png)

![image-20220314111001390](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220314111001390.png)

![image-20220314111111514](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220314111111514.png)

![image-20220314111310044](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220314111310044.png)

### 序列化与反序列化

```python
api接口开发，最核心最常见的一个过程就是序列化，所谓序列化就是把数据转换格式，序列化可以分两个阶段：
    1.序列化： 把我们语言识别的数据转换成指定的格式提供给别人。
              字典，列表，对象 ---> json/xml/prop,massagepack ---> 提供给别人(前端或其他服务)
    2.反序列化：把别人提供的数据转换/还原成我们需要的格式


我们在django中获取到的数据默认是模型对象（qs对象），但是模型对象数据无法直接提供给前端或别的平台使用，所以我们需要把数据进行序列化，变成字符串或者json数据，提供给别人 --->序列化过程

前端传入到后台的数据 --->json格式字符串 --->后端存到数据库中，需要转成python中的对象 ---> 把json格式字符串转成python对象存到数据库的过程称为反序列化
```

### `DRF`介绍和安装

#### 介绍和安装

```python
DRF可以更方便的使用django写出符合RESTful规范的接口(不用也可以写出符合规范的接口)
是一个app(需要在settings.py中配置)
安装: pip3 install djangorestframework
官网: https://www.django-rest-framework.org/
```

#### 简单使用

```python
# csrf已经禁用了
# 路由
path('test/', views.Test.as_view()),

# 视图
from rest_framework.views import APIView
from rest_framework.response import Response
class Test(APIView):
    def get(self, request):
        return Response({'name': 'xxx', 'age': 18})
    def post(self, request):
        return Response({'name': 'yyy', 'age': 20})

# 注册app(settings.py)
  INSTALLED_APPS = [
      ...
      'rest_framework',
      ...
  ]

# 请求地址
  http://127.0.0.1:8000/test/
  # 使用post和浏览器会有不同的结果
```

### `DRF`的快速使用

#### `models.py`

```python
from django.db import models
class Books(models.Model):
    name = models.CharField(max_length=32)
    publish = models.CharField(max_length=32)
    price = models.IntegerField()
```

#### `app01/serializer.py`

```python
from rest_framework.serializers import ModelSerializer
from app01 import models
class BookSerializer(ModelSerializer):
    class Meta:
        model = models.Books
        fields = '__all__'
```

#### `views.py`

```python
from rest_framework.viewsets import ModelViewSet
from app01 import models
from app01.serializer import BookSerializer

class BookView(ModelViewSet):
    serializer_class = BookSerializer
    queryset = models.Books.objects.all()
```

#### `urls.py`

```python
from django.contrib import admin
from django.urls import path
from rest_framework.routers import SimpleRouter
from app01 import views

router = SimpleRouter()
router.register('books', views.BookView)
urlpatterns = [
    path('admin/', admin.site.urls),
]
urlpatterns += router.urls
```

### `CBV`源码分析

```python
path('test/',views.TestView.as_view()),
    # path('test/',View类的as_view内部有个view闭包函数内存地址)
    '''
    1 path的第二个参数是：View类的as_view内部有个view闭包函数内存地址
    2 一旦有请求来了，匹配test路径成功
    3 执行第二个参数view函数内存地址(requset)
    4 本质执行了self.dispatch(request)
    5 通过反射去获得方法（如果是get请求，就是get方法）
     if request.method.lower() in self.http_method_names:
        handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
    6 执行get方法，传入参数
    handler(request, *args, **kwargs)
    '''

'''
装饰器函数 csrf_exempt
@csrf_exempt
def test():
    pass

本质就是
test=csrf_exempt(test)
'''
```

### `APIview`源码分析

```python
    # path('test/',APIView类的as_view内部是用了View的as_view内的view闭包函数),
    '''
    1 path的第二个参数是：APIView类的as_view内部是用了View的as_view内的view闭包函数
    2 一旦有请求来了，匹配test路径成功
    3 执行第二个参数view函数内存地址(requset)，还是执行View的as_view内的view闭包函数，但是加了个csrf_exempt装饰器
    4 所以，继承了APIView的所有接口，都没有csrf的校验了 （*****************）
    5 执行self.dispatch(request)----》APIView类的
        def dispatch(self, request, *args, **kwargs):
            # 以后所有的request对象，都是****新的request对象***，它是drf的Request类的对象
            request = self.initialize_request(request, *args, **kwargs)
            self.request = request
            try:
                #整个drf的执行流程内的权限，频率，认证
                self.initial(request, *args, **kwargs)
                if request.method.lower() in self.http_method_names:
                    handler = getattr(self, request.method.lower(),
                                      self.http_method_not_allowed)
                else:
                    handler = self.http_method_not_allowed

                response = handler(request, *args, **kwargs)

            except Exception as exc:
                # 全局异常
                response = self.handle_exception(exc)
            # 响应
            self.response = self.finalize_response(request, response, *args, **kwargs)
            return self.response
    '''

## request = self.initialize_request(request, *args, **kwargs)
## 返回的request对象是drf   Request类的request对象
def initialize_request(self, request, *args, **kwargs):
    return Request(
        request,
        parsers=self.get_parsers(),
        authenticators=self.get_authenticators(),
        negotiator=self.get_content_negotiator(),
        parser_context=parser_context
    )
## ******以后，在视图类中使用的request对象已经不是原来的request对象了，现在都是drf的request对象了

##### 需要记住的
  所有的csrf都不校验了
  request对象变成了新的request对象，drf的request对象
  执行了权限，频率，认证
  捕获了全局异常（统一处理异常）
  处理了response对象，如果浏览器访问是一个样，postman访问又一个样
  以后，在视图类中使用的request对象已经不是原来的request对象了，现在都是drf的request对象了
```

### `Request`对象分析

```python
1 django 原生的Request：django.core.handlers.wsgi.WSGIRequest

2 drf的Request：rest_framework.request.Request

3 drf的request对象内有原生的request
    request._request:原生的Request

4 在视图类中使用
    request.method  拿到的就是请求方式，
    正常拿，应该request._request.method

5 如何实现这种操作？
	-对象.属性会触发 类的__getattr__方法

6 drf的Request类重写了__getattr__
    def __getattr__(self, attr):
        try:
            # 去原生的request反射属性
            return getattr(self._request, attr)
        except AttributeError:
            return self.__getattribute__(attr)

 7 虽然视图类中request对象变成了drf的request，但是用起来，跟原来的一样，只不过它多了一些属性
   request.data  # post请求提交的数据，不论什么格式，都在它中
   requst.query_params  # get请求提交的数据（查询参数）

8 重点记住：
  drf的request对象用起来跟原来一样（重写了__getattr__）
  request.data  # post请求提交的数据，不论什么格式，都在它中
  requst.query_params  # get请求提交的数据（查询参数）
```
