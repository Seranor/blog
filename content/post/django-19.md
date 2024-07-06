---
title: 过滤、排序、分页和异常处理
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
  - DRF
categories:
  - Django
url: post/django-19.html
toc: true
---

### 过滤

```python
# 针对于list, 获取所有
# 在请求路径中带过滤条件，对查询结果进行过滤
```

<!-- more -->

#### 内置过滤类

```python
# 导入
from rest_framework.filters import SearchFilter

# 视图类
  # 必须继承 GenericAPIView 才有这个类属性
  filter_backends = [SearchFilter,]
  # 需要配合一个类属性，按照models中的字段填写
  search_fields=['name','author']

# 搜索使用 支持模糊搜索 只能是 search
  http://127.0.0.1:8000/books/?search=红
  http://127.0.0.1:8000/books/?search=清  # 书名或者author中带清就能搜到
```

#### 第三方过滤类

```python
# 安装
  pip3 install django-filter

# 配置文件中注册
  INSTALLED_APPS = [
      'django_filters',
  ]

# 视图文件中导入过滤类
  from django_filters.rest_framework import DjangoFilterBackend

# 视图类中使用
  class BookView(GenericViewSet,ListModelMixin):
    # 必须继承GenericAPIView，才有这个类属性
    filter_backends = [DjangoFilterBackend,]
    # 需要配合一个类属性
    filter_fields=['name','author']

# 搜索使用
  http://127.0.0.1:8000/books/?name=红楼梦
  http://127.0.0.1:8000/books/?name=红楼梦&author=lxx  # and条件
  http://127.0.0.1:8000/books/?author=lxx
```

#### 自定义过滤类

```python
# 写一个过滤类 继承 BaseFilterBackend
  # app01/fulter.py
  from rest_framework.filters import BaseFilterBackend
  class BookNameFilter(BaseFilterBackend):
      def filter_queryset(self, request, queryset, view):
          query = request.query_params.get('name')
          # query = request.GET.get('name')
          if query:
              queryset = queryset.filter(name__contains=query)
          return queryset

# 视图类中使用
  from app01.filter import BookNameFilter
  class BookView(GenericViewSet,ListModelMixin):
      filter_backends = BookNameFilter

# 搜索使用
	http://127.0.0.1:8000/books/?name=红  # 模糊匹配 ，自己定义的
```

#### 源码

```python
# 源码分析 ----> GenericAPIView ----> 查询所有
# 调用了list ----> self.filter_queryset(self.get_queryset())
# 查看GenericAPIView的filter_queryset方法：

    def filter_queryset(self, queryset):
        for backend in list(self.filter_backends):
            queryset = backend().filter_queryset(self.request, queryset, self)
        return queryset
```

#### 视图类属性总结

```python
1 APIViwe中有哪些类属性
    renderer_classes = api_settings.DEFAULT_RENDERER_CLASSES
    parser_classes = api_settings.DEFAULT_PARSER_CLASSES
    authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
    throttle_classes = api_settings.DEFAULT_THROTTLE_CLASSES
    permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES

2 GenericAPIView中有哪些类属性
    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS
    pagination_class = api_settings.DEFAULT_PAGINATION_CLASS
```

### 排序

```python
# 按照id排序，按照年龄排序，按照价格排序
# 使用内置的即可

# 导入内置排序类
  from rest_framework.filters import OrderingFilter

# 在视图类中配置(必须继承GenericAPIView)
  class BookView(GenericViewSet,ListModelMixin):
      # 支持按price排序
      filter_backends = [OrderingFilter,]
      ordering_fields=['price']

# 查询
  http://127.0.0.1:8000/books/?ordering=-price  # 倒序排
  http://127.0.0.1:8000/books/?ordering=-price,-id # 先按价格倒序排，如果价格一样，再按id倒序排


# 过滤和排序可以同时用- --> 因为他们本质是for循环一个个执行，所有优先使用过滤，再使用排序
  filter_backends = [SearchFilter,OrderingFilter]
  ordering_fields=['price','id']
  search_fields=['name','author']
```

### 分页

```python
5个接口中，只有查询所有，涉及到分页
pc端是下一个页，下一页的形式，如果在app，小程序中展现形式是下拉加载下一个
```

#### 基本分页

```python
# PageNumberPagination: 基本分页 ---> 按照页码数，每页显示多少条
from rest_framework.pagination import PageNumberPagination

# 写一个类 继承 PageNumberPagination 重写四个类属性
  # app01/page.py
  class CommonPageNumberPagination(PageNumberPagination):
      # 重写四个类属性
      page_size = 3  # 每页显示条数，默认
      page_query_param = 'page'  # 查询条件叫page --> ?page=3
      page_size_query_param = 'size' # 每页显示的条数的查询条件 ?page=3&size=9 查询第三页，第三页显示9条
      max_page_size = 5  # 每页最大线上多少条，?page=3&size=9，最终还是显示5条

      # _就是一个函数的别名 ---> 通常约定俗成 ---> _ 翻译函数

# 视图类中配置 必须继承GenericAPIView才有
	from .page import CommonPageNumberPagination as PageNumberPagination
  # 视图类属性
	pagination_class = PageNumberPagination

# 查询方式
  http://127.0.0.1:8000/books/?page=2&size=8
```

#### 偏移分页

```python
# LimitOffsetPagination: 偏移分页
from rest_framework.pagination import LimitOffsetPagination

# 写一个类 继承 LimitOffsetPagination 重写四个类属性
  # app01/page.py
  class CommonLimitOffsetPagination(LimitOffsetPagination):
      default_limit = 2  # 默认一页获取条数 2  条
      limit_query_param = 'limit'  # ?limit=3  获取三条，如果不传，就用上面的默认两条
      offset_query_param = 'offset'  # ?limit=3&offset=2  从第2条开始，获取3条    ?offset=3：从第三条开始，获取2条
      max_limit = 5  # 最大显示条数 5 条

# 视图类中配置 必须继承GenericAPIView才有
	from .page import CommonLimitOffsetPagination
  # 视图类属性
	pagination_class = CommonLimitOffsetPagination

# 查询方式
  http://127.0.0.1:8000/books/?limit=2&offset=1
```

#### 游标分页

```python
# CursorPagination: 游标分页
# 跟上面两种的区别：上面两种，可以从中间位置获取某一页，Cursor方式只能上一页和下一页
# 上面这两种在获取某一页的时候，都需要从开始过滤到要取的页面数的数据
# 下面这种方式，先排序，内部维护了一个游标，游标只能选择往前走或往后走，在取某一页的时候，不需要过滤之前的数据
# 这种分页方式特殊，只能选择上一页和下一页，不能指定某一页，但是速度快，适合大数据量的分页
# 大数据量和app分页 ---> 下拉加载下一页，不需要指定跳转到第几页

from rest_framework.pagination import CursorPagination

# 写一个类 继承 CursorPagination 重写类属性
# app01/page.py
class CommonCursorPagination(CursorPagination):
    page_size = 2  # 每页显示2条
    cursor_query_param = 'cursor'  # 查询条件  ?cursor=sdafdase
    ordering = 'id'   # 排序规则，使用id排序

# 视图类中配置 必须继承GenericAPIView才有
  from .page import CommonCursorPagination
  # 视图类属性
  pagination_class = CommonCursorPagination

# 查询方式
  http://127.0.0.1:8000/books/?cursor=cD02
```

### 异常处理

```python
# 之前读APIViwe源码的时候，捕获了全局异常，在执行三大认证，视图类的方法时候，如果出了异常，会被全局异常捕获

# 统一返回格式，无论是否异常，返回的格式统一 记录日志（好排查）
  {code:999,msg:服务器异常，请联系系统管理员}
  {code:100,msg:成功,data:[{},{}]}


# 步骤：
# 写一个函数
  # app01/exception.py
  ## 源码
  # 'EXCEPTION_HANDLER': 'rest_framework.views.exception_handler',
  from rest_framework.views import exception_handler  # 默认没有配置，出了异常会走它
  from rest_framework.response import Response
  def common_exception_handler(exc, context):
      # 第一步，先执行原来的exception_handler
      # 第一种情况，返回Response对象，这表示已经处理了异常,它只处理APIExcepiton的异常，第二种情况，返回None，表示没有处理
      res = exception_handler(exc, context)
      if res:  # exception_handler 已经处理了，暂时先不处理了
          # 998:APIExcepiton
          # res=Response(data={'code':998,'msg':'服务器异常，请联系系统管理员'})
          res = Response(data={'code': 998, 'msg': res.data.get('detail', '服务器异常，请联系系统管理员')})
      else:
          # 999：出了APIExcepiton外的异常
          # res=Response(data={'code':999,'msg':'服务器异常，请联系系统管理员'})
          res = Response(data={'code': 999, 'msg': str(exc)})

      # 注意：咱们在这里，可以记录日志---》只要走到这，说明程序报错了，记录日志，以后查日志---》尽量详细
      # 出错时间，错误原因，哪个视图类出了错，什么请求地址，什么请求方式出了错
      request = context.get('request')  # 这个request是当次请求的request对象
      view = context.get('view')  # 这个viewt是当次执行的视图类对象
      print('错误原因：%s,错误视图类：%s,请求地址：%s,请求方式：%s' % (str(exc), str(view), request.path, request.method))
      return res


  第二步：把函数配置在配置文件中
    REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'app01.exception.common_exception_handler' # 再出异常，会执行这个函数
    }


# 其他
from rest_framework.response import  Response
def my_exception_handler(exc, context):
    response=exception_handler(exc, context)
    print(response)
    print(exc)
    print(context.get('view'))
    print(context.get('request').get_full_path())
    if not response: #更新粒度的对异常做一个区分
        if isinstance(exc,IndexError):
            response=Response({'status':5001,'msg':'越界异常'})
        elif isinstance(exc,ZeroDivisionError):
            response = Response({'status': 5002, 'msg': '越界异常'})
        else:
            response= Response({'status': 5000, 'msg': '没有权限'})
    # 记录日志
    return response

### 以后再出异常，都会走这个函数，后期需要记录日志，统一了返回格式
```
