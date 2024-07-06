---
title: 视图和路由
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
  - DRF
categories:
  - Django
url: post/django-17.html
toc: true
---

### 视图组件

#### 视图继承关系

<!-- more -->

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/dasdas-20220401164741865.jpg)

#### 2 个视图基类

##### APIView

```python
from rest_framework.views import APIView

属性:
	renderer_classes,parser_classes...
get方法，post方法，delete方法等和View的一样，只是request对象变成了新的request对象
比View多了三大认证和全局异常处理
```

##### GenericAPIView

```python
继承了APIView，多了一些属性和方法
from rest_framework.generics import GenericAPIView

属性:
  queryset
  serializer_class

方法:
  get_queryset: 获取qs数据
  get_object: 获取一条数据的对象
  get_serializer: 以后使用它来实例化得到ser对象
  get_serializer_class: 获取序列化类
```

#### 5 个视图扩展类

```python
  from rest_framework.mixins
  CreateModelMixin,
  ListModelMixin,
  DestroyModelMixin,
  RetrieveModelMixin,
  UpdateModelMixin

1 查所有：ListModelMixin
    列表视图扩展类，提供list(request, *args, **kwargs)方法快速实现列表视图，返回200状态码。
     该Mixin的list方法会对数据进行过滤和分页。

2 查一个：RetrieveModelMixin
    创建视图扩展类，提供create(request, *args, **kwargs)方法快速实现创建资源的视图，成功返回201状态码。
     如果序列化器对前端发送的数据验证失败，返回400错误。

3 增一个：CreateModelMixin
    详情视图扩展类，提供retrieve(request, *args, **kwargs)方法，可以快速实现返回一个存在的数据对象。
     如果存在，返回200， 否则返回404

4 改一个：UpdateModelMixin
    更新视图扩展类，提供update(request, *args, **kwargs)方法，可以快速实现更新一个存在的数据对象。
    同时也提供partial_update(request, *args, **kwargs)方法，可以实现局部更新。
     成功返回200，序列化器校验数据失败时，返回400错误。

5 删一个：DestroyModelMixin
    删除视图扩展类，提供destroy(request, *args, **kwargs)方法，可以快速实现删除一个存在的数据对象
    成功返回204，不存在返回404。
```

#### 9 个视图子类

```python
  from rest_framework.generics
  CreateAPIView,
  ListAPIView,
  DestroyAPIView,
  RetrieveAPIView,
  UpdateAPIView
  ListCreateAPIView,
  RetrieveUpdateAPIView,
  RetrieveUpdateDestroyAPIView,
  RetrieveDestroyAPIView

1）查所有：ListAPIView
提供 get 方法
继承自：GenericAPIView、ListModelMixin


2）增一个：CreateAPIView
提供 post 方法
继承自： GenericAPIView、CreateModelMixin


3）查所有+增一个：ListCreateAPIView
提供 get 和 post 方法
继承自： GenericAPIView、ListModelMixin、CreateModelMixin


4）查一个：RetrieveAPIView
提供 get 方法
继承自: GenericAPIView、RetrieveModelMixin


5）改一个：UpdateAPIView
提供 put 和 patch 方法
继承自：GenericAPIView、UpdateModelMixin


6）删一个：DestoryAPIView
提供 delete 方法
继承自：GenericAPIView、DestoryModelMixin


7）查一个+改一个：RetrieveUpdateAPIView
提供 get、put、patch方法
继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin


8）查一个+删一个：RetrieveDestroyAPIView
提供 get 和 delete 方法
继承自： GenericAPIView、RetrieveModelMixin、DestoryModelMixin


9） 查一个+改一个+删一个：RetrieveUpdateDestoryAPIView
提供 get、put、patch、delete方法
继承自：GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin
```

#### 视图类的使用

##### `views`

```python
## 第一层：继承APIView写视图类

## 第二层：继承GenericAPIView写视图类
from rest_framework.generics import GenericAPIView

class PublishView(GenericAPIView):
    queryset = Publish.objects.all()
    serializer_class = PublishSerialzier


    def get(self, request):
        # obj = self.queryset
        obj = self.get_queryset()  # 等同于上面，更好一些
        # ser = self.serializers(instance=obj,many=True)
        # ser=self.get_serializer_class()(instance=obj,many=True) # 等同于上面
        ser = self.get_serializer(instance=obj, many=True)  # 等同于上面
        return Response(ser.data)

    def post(self, request):
        # ser = BookSerializer(data=request.data)
        ser = self.get_serializer(data=request.data)  # 等同于上面
        if ser.is_valid():
            ser.save()
            return Response({"code": 100, 'msg': '新增成功', 'data': ser.data})
        return Response({"code": 101, 'msg': '新增出错', 'err': ser.errors})


# class PublishDetailView(GenericAPIView):
#     queryset = Publish.objects.all()
#     serializer_class = PublishSerialzier
#
#     # lookup_field = 'pk'
#     def get(self, request, *args, **kwargs):
#         # book = Book.objects.all().filter(pk=pk).first()
#         obj = self.get_object()  # 等同于上面
#         # ser = BookSerializer(instance=book)
#         ser = self.get_serializer(instance=obj)  # 等同于上面
#         return Response(ser.data)
#
#     def put(self, request, *args, **kwargs):
#         # book = Book.objects.all().filter(pk=pk).first()
#         obj = self.get_object()  # 等同于上面
#         # ser = BookSerializer(instance=book, data=request.data)
#         ser = self.get_serializer(instance=obj, data=request.data)  # 等同于上面
#         if ser.is_valid():
#             ser.save()
#             return Response({"code": 100, 'msg': '修改成功', 'data': ser.data})
#         return Response({"code": 101, 'msg': '修改出错', 'err': ser.errors})
#
#     def delete(self, request, *args, **kwargs):
#         # Book.objects.filter(pk=pk).delete()
#         self.get_object().delete()
#         return Response({"code": 100, 'msg': '删除成功'})


## 第三层：GenericAPIView+5个视图扩展类
# from rest_framework.generics import GenericAPIView
# from rest_framework.mixins import CreateModelMixin,ListModelMixin,DestroyModelMixin,RetrieveModelMixin,UpdateModelMixin
#
#
#
# class PublishView(GenericAPIView,ListModelMixin,CreateModelMixin):
#     queryset = Publish.objects.all()
#     serializer_class = PublishSerialzier
#
#     # def perform_create(self,serializer):
#     #     # 判断，验证失败了，抛异常，通过了再保存
#     #     serializer.save()
#     def get(self, request):
#         return super().list(request) # create(request)ListModelMixin的方法
#
#     def post(self, request):
#         return super().create(request)  # create(request)CreateModelMixin的方法
#
#
# class PublishDetailView(GenericAPIView,UpdateModelMixin,RetrieveModelMixin,DestroyModelMixin):
#     queryset = Publish.objects.all()
#     serializer_class = PublishSerialzier
#
#     # lookup_field = 'pk'
#     def get(self, request, *args, **kwargs):
#         return super().retrieve(request, *args, **kwargs)
#
#     def put(self, request, *args, **kwargs):
#         return super().update(request, *args, **kwargs)
#
#     def delete(self, request, *args, **kwargs):
#         return super().destroy(request, *args, **kwargs)


# 第四层：通过9个视图子类，编写视图函数
# from rest_framework.generics import CreateAPIView, ListAPIView, DestroyAPIView, RetrieveAPIView, UpdateAPIView
# from rest_framework.generics import ListCreateAPIView, RetrieveUpdateAPIView, RetrieveUpdateDestroyAPIView, \
#     RetrieveDestroyAPIView
#
#
# # class PublishView(ListCreateAPIView):  # 查询所有和新增接口就有了
# # class PublishView(CreateAPIView):  # 新增接口就有了
# class PublishView(ListAPIView):  # 查询所有接口就有了
#     queryset = Publish.objects.all()
#     serializer_class = PublishSerialzier
#
#     # 有可能要重写--》get_queryset--》get_serializer_class--》perform_create--》get，post方法
#
#
# class PublishDetailView(RetrieveUpdateDestroyAPIView): # 查询单条，删除，修改
# # class PublishDetailView(RetrieveAPIView): # 查询单条
# # class PublishDetailView(DestroyAPIView): # 删除
# # class PublishDetailView(UpdateAPIView): # 修改
# # class PublishDetailView(RetrieveDestroyAPIView): # 查询单条和删除
# # class PublishDetailView(RetrieveUpdateAPIView): # 查询单条和更新
# # class PublishDetailView(UpdateAPIView,DestroyAPIView): # 更新和删除
#     queryset = Publish.objects.all()
#     serializer_class = PublishSerialzier



# 第五层：通过ViewSet写视图类

# 5个接口，都用一个视图类----》路由 有两个get
from rest_framework.viewsets import ModelViewSet,ReadOnlyModelViewSet
# ViewSet=APIView+ViewSetMixin
# GenericViewSet=GenericAPIView+ViewSetMixin
# 以后只要想自动生成路由，必须继承ViewSetMixin及其子类
# 之前的写法可以沿用，只是如果要自动生成路由可以选择继承ViewSet，GenericViewSet
from rest_framework.viewsets import ViewSet,GenericViewSet
from rest_framework.viewsets import ViewSetMixin

# 继承了5个视图扩展类+GenericViewSet(ViewSetMixin, generics.GenericAPIView)
# ViewSetMixin-->控制了路由写法变了
# class PublishView(ModelViewSet):  # 修改路由,5个接口
# class PublishView(ReadOnlyModelViewSet):  # 修改路由,只读，查所有，查单个
#     queryset = Publish.objects.all()
#     serializer_class = PublishSerialzier
```

##### `models`

```python
class Publish(models.Model):
    name = models.CharField(max_length=32)
    city = models.CharField(max_length=32)
    email = models.EmailField()
```

##### `serializer`

```python
class PublishSerialzier(serializers.ModelSerializer):
    class Meta:
        model = Publish
        fields = '__all__'
```

##### `urls`

```python
path('publishs/', views.PublishView.as_view()),
path('publishs/<int:pk>', views.PublishDetailView.as_view()),
```

#### 视图集

```python
  from rest_framework.viewsets
  # 两个视图类
  ModelViewSet,ReadOnlyModelViewSet
  # 视图类
  ViewSet,GenericViewSet,
  # 魔法类
  ViewSetMixin

1） ViewSet
继承自APIView与ViewSetMixin，作用也与APIView基本类似，提供了身份认证、权限校验、流量管理等。

ViewSet主要通过继承ViewSetMixin来实现在调用as_view()时传入字典（如{‘get’:’list’}）的映射处理工作。

在ViewSet中，没有提供任何动作action方法，需要我们自己实现action方法。

2）GenericViewSet
使用ViewSet通常并不方便，因为list、retrieve、create、update、destory等方法都需要自己编写，而这些方法与前面讲过的Mixin扩展类提供的方法同名，所以我们可以通过继承Mixin扩展类来复用这些方法而无需自己编写。但是Mixin扩展类依赖与GenericAPIView，所以还需要继承GenericAPIView。

GenericViewSet就帮助我们完成了这样的继承工作，继承自GenericAPIView与ViewSetMixin，在实现了调用as_view()时传入字典（如{'get':'list'}）的映射处理工作的同时，还提供了GenericAPIView提供的基础方法，可以直接搭配Mixin扩展类使用。

3）ModelViewSet
继承自GenericViewSet，同时包括了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin。

4）ReadOnlyModelViewSet
继承自GenericViewSet，同时包括了ListModelMixin、RetrieveModelMixin。
```

#### 视图总结

```python
# 两个基类
APIView
GenericAPIView：有关数据库操作，queryset 和serializer_class


# 5个视图扩展类(rest_framework.mixins)
CreateModelMixin：create方法创建一条
DestroyModelMixin：destory方法删除一条
ListModelMixin：list方法获取所有
RetrieveModelMixin：retrieve获取一条
UpdateModelMixin：update修改一条

# 9个子类视图(rest_framework.generics)
CreateAPIView:继承CreateModelMixin,GenericAPIView，有post方法，新增数据
DestroyAPIView：继承DestroyModelMixin,GenericAPIView，有delete方法，删除数据
ListAPIView：继承ListModelMixin,GenericAPIView,有get方法获取所有
UpdateAPIView：继承UpdateModelMixin,GenericAPIView，有put和patch方法，修改数据
RetrieveAPIView：继承RetrieveModelMixin,GenericAPIView，有get方法，获取一条


ListCreateAPIView：继承ListModelMixin,CreateModelMixin,GenericAPIView，有get获取所有，post方法新增
RetrieveDestroyAPIView：继承RetrieveModelMixin,DestroyModelMixin,GenericAPIView，有get方法获取一条，delete方法删除
RetrieveUpdateAPIView：继承RetrieveModelMixin,UpdateModelMixin,GenericAPIView，有get获取一条，put，patch修改
RetrieveUpdateDestroyAPIView：继承RetrieveModelMixin,UpdateModelMixin,DestroyModelMixin,GenericAPIView，有get获取一条，put，patch修改，delete删除

# 视图集
ViewSetMixin：重写了as_view
ViewSet：     继承ViewSetMixin和APIView

GenericViewSet：继承ViewSetMixin, generics.GenericAPIView
ModelViewSet：继承mixins.CreateModelMixin,mixins.RetrieveModelMixin,mixins.UpdateModelMixin,mixins.DestroyModelMixin,mixins.ListModelMixin,GenericViewSet
ReadOnlyModelViewSet：继承mixins.RetrieveModelMixin,mixins.ListModelMixin,GenericViewSet
```

```python
1 序列化器源码
    -many参数控制，在__new__中控制了对象的生成
    -局部和全局钩子源码：is_valid--》找self.方法一定要从根上找
    -source参数是如何执行的：‘publish.name’,'方法'

2 视图：
    -2个视图基类
    -5个视图扩展类
    -9个视图子类
    -视图集
        -ViewSetMixin
        -ViewSet, GenericViewSet
        -ModelViewSet, ReadOnlyModelViewSet

3 视图类的继承原则
    -如果不涉及数据库操作：继承APIView
    -如果想让路由可以映射：继承ViewSetMixin
    -如果不涉及数据库操作，又要让路由可以映射：继承ViewSet
    -比如发邮件接口，发短信接口

    -如果涉及到数据库操作：继承GenericAPIView
    -如果想让路由可以映射：继承ViewSetMixin
    -如果涉及数据库操作，又要让路由可以映射：继承GenericViewSet
    -如果涉及数据库操作，又要让路由可以映射，还要能新增：继承GenericViewSet+CreateModelMixin或者继承ViewSetMixin+CreateAPIView

    -如果只涉及到数据库操作和新增：继承CreateAPIView

    -路由有映射，数据库操作，3个接口（查一个，删一个改一个）

4 ViewSetMixin：路由的写法就特殊了


5 类实例化，先执行了元类的__call__:调用了这个类的__new__,生成一个空对象，调用了类的__init__,完成了对象的初始化
6 对象()---->会触发类的 __call__
7 类()----->会触发类的类（元类）的__call__
```

![image-20220401204213872](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220401204213872.png)

### 路由组件

#### 三种路由写法

```python
# 以后路由写法有三种
	path('books/<int:pk>', views.BookDetailView.as_view()),
  # 继承了ViewSetMixin
  path('publish/', views.PublishView.as_view({'get':'list','post':'create'}))
  # 自动生成路由
  from rest_framework.routers import SimpleRouter,DefaultRouter
  router = SimpleRouter()
  router.register('publish', views.PublishView, 'publish')
	# 路由注册，两种方式
  path('api/v1/', include(router.urls)),
  urlpatterns+=router.urls
  # 路由类，有连个SimpleRouter，DefaultRouter，DefaultRouter自动生成的路由更多
```

#### 自动生成路由

```python
# 只要继承ViewSetMixin 及其子类，路由写法就变了
# 视图类：继承ViewSetMixin，路由写法变了--->而且视图类中的方法不一定写成get，post..,可以随意命名，只不过定义路由时写法变成了path('test/', views.TestView.as_view({'get': 'login'}))，get请求执行login方法

# 到底如何执行的
  # ViewSetMixin必须写在前面
  ViewSetMixin ---> as_view--->类的查找顺序 ---> actions就是传入的字典 ---> 》view闭包函数
  def view(request, *args, **kwargs):
     #    get      login
    	for method, action in actions.items():
        # 把login方法的内存地址给了handler
        handler = getattr(self, action)
        # 通过反射，设置给get ---> 对应login ---> get请求执行get方法，现在get方法变成了login方法
        setattr(self, method, handler)
      return self.dispatch(request, *args, **kwargs)# 跟之前一样了


# 视图类，继承ModelViewSet，路由写法
    path('publish/', views.PublishView.as_view({'get':'list','post':'create'})),
    path('publish/<int:pk>', views.PublishView.as_view({'get':'retrieve','put':'update','delete':'destroy'}))

    # 到现在，咱们有了一种方式，可以在一个视图类中写很多方法，通过路由的配置，实现一个视图函数对应很多路由

    # 上面的两个路由，可以自动生成--->生成上面那两条地址
    	第一步：导入drf提供的路由类
      from rest_framework.routers import SimpleRouter
      第二步：实例化得到对象
      router = SimpleRouter()
      第三步：注册路由
      router.register('地址', 视图类, '别名')
      router.register('publish', views.PublishView, 'publish')
      第四步：在urlpatterns加入（两种方式）
      	path('/api/v1', include(router.urls))
        urlpatterns+=router.urls


   # SimpleRouter和DefaultRouter 用法完全一样，只是生成的路由不一样DefaultRouter比SimpleRouter多一条根地址，一般咱么就用SimpleRouter就可以


  # 如果使用自动生成路由，必须继承谁及其子类？
    GenericViewSet+5个视图扩展类至少之一---》才能自动生成
    ModelViewSet
    ReadOnlyModelViewSet
```

#### `action`装饰器

```python
 from rest_framework.decorators import action
  # action装饰器---》只要继承ViewSetMixin都能自动生成路由
    # methods=请求方法，列表,
    # detail=是否带id,
    # url_path=地址如果不写，默认已方法名为地址,
    # url_name=别名
    # 写成如下，自动生成路由会生成一条路径：127.0.0.1:8000/test/login -->get，post都会触发
    @action(methods=['GET','POST'],detail=True)
    def login(self,request):

   # 如果写法如下，生成的路径是127.0.0.1:8000/test/数字/login
    @action(methods=['GET','POST'],detail=True)
    def login(self,request):

 # 通过action，怎么通过action判断，重写get_queryset，get_serializer_class
```

#### 简单案例

```python
# 视图及自动生成路由
# views.py
from rest_framework.viewsets import ModelViewSet
class BookView(ModelViewSet):
    queryset = models.Books.objects.all()
    serializer_class = serializer.BookSerializer

# models.py
from django.db import models
class Books(models.Model):
    name = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    publish = models.ForeignKey(to="Publish", on_delete=models.CASCADE)
    authors = models.ManyToManyField(to="Author")

    def __str__(self):
        return self.name

    @property
    def publish_list(self):
        return {'name': self.publish.name, 'city': self.publish.city, 'email': self.publish.email}

    @property
    def authors_list(self):
        return [{'name': author.name, 'age': author.age, 'phone': author.author_detail.tel,
                 'address': author.author_detail.addr} for author in self.authors.all()]

# serializer.py
from rest_framework import serializers
from app01 import models
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Books
        fields = "__all__"
        extra_kwargs = {
            "publish": {'write_only': True},
            "authors": {'write_only': True},
        }

    publish_list = serializers.DictField(read_only=True)
    authors_list = serializers.ListField(read_only=True)

# urls.py
from django.urls import path, include
from app01 import views
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register('books', views.BookView)

urlpatterns = [
    path('api/v1/', include(router.urls)),
]

```

```python
## action使用
# views.py
from rest_framework.viewsets import GenericViewSet
class TestView(GenericViewSet):
    @action(methods=['GET'], detail=False)
    def test(self, request):
        return Response("get-test")

# urls.py
    path('test/', views.TestView.as_view({'get': 'test'})),
```
