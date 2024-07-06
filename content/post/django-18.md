---
title: 认证、权限、频率
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
  - DRF
categories:
  - Django
url: post/django-18.html
toc: true
---

### 认证

#### 登录接口

<!-- more -->

```python
# modles.py
from django.db import models

class User(models.Model):
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=32)
    user_type = models.IntegerField(choices=((1, '超级用户'), (2, '普通用户'), (3, '二笔用户')))

class UserToken(models.Model):
    user = models.OneToOneField(to='User', on_delete=models.CASCADE)
    token = models.CharField(max_length=64)

# serializer.py
from rest_framework import serializers
from app01 import models
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.User
        fields = '__all__'

# views.py
import uuid
from app01 import models
from app01 import serializer
from rest_framework.viewsets import ViewSetMixin
from rest_framework.generics import CreateAPIView
from rest_framework.decorators import action
from rest_framework.response import Response

class UserView(ViewSetMixin, CreateAPIView):
    queryset = models.User.objects.all()
    serializer_class = serializer.UserSerializer

    @action(methods=['POST'], detail=False)
    def login(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        user = models.User.objects.filter(username=username, password=password).first()
        token = uuid.uuid4()
        # 根据user去查询，如果能查到，就修改token，如果查不到，就新增一条
        models.UserToken.objects.update_or_create(defaults={'token': token}, user=user)
        if user:
            return Response({'msg': '登录成功', "token": token})
        else:
            return Response({'status': 101, 'msg': '用户名或密码错误'})

# urls.py
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from app01 import views

router = SimpleRouter()
router.register('user', views.UserView)
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
]
```

#### 认证的使用

##### 认证类的编写

```python
# 认证类：用来校验用户是否登录，如果登录了，继续往下走，如果没有登录，直接返回
# 编写步骤：
  1.写一个类，继承BaseAuthentication,
  2.重写authenticate，在方法中做校验，校验是否登录，返回两个值，没有登录抛异常
  3.全局配置，局部配置
    全局配置：配置文件中
        REST_FRAMEWORK={
        "DEFAULT_AUTHENTICATION_CLASSES":["app01.auth.LoginAuth",]
        }
    局部配置：在视图类中
        class UserView(ViewSet):
          authentication_classes = [LoginAuth]

    局部禁用：
         class UserView(ViewSet):
            authentication_classes = []

# 认证类中返回的两个变量
   返回的第一个，给了request.user，就是当前登录用户
   返回的第二个，给了request.auth，就是token串
```

```python
# 在 app01 下创建 auth.py
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from app01 import models

class LoginAuth(BaseAuthentication):
    def authenticate(self, request):
        token = request.query_params.get('token')
        user_token = models.UserToken.objects.filter(token=token).first()
        if user_token:
            return user_token.user, token
        else:
            raise AuthenticationFailed('您没有登录')
```

##### 认证类的使用

```python
# 全局使用 在 settings.py 中配置 ---> 所有接口都需要认证
  REST_FRAMEWORK={
      "DEFAULT_AUTHENTICATION_CLASSES":["app01.auth.LoginAuth",]
  }

# 局部禁用 在视图类中添加(如登录的时候就不能要求认证)
  authentication_classes = []

# 局部使用
	from app01 import auth
  authentication_classes = [auth.LoginAuth,]
```

![image-20220405161826558](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220405161826558.png)

![image-20220405162315439](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220405162315439.png)

### 权限

#### 权限类的编写

```python
# 登录成功 ---> 所有必须登录能访问 ---> 每个视图类上加认证类
# 用户是普通用户 ---> 普通用户可以访问所有和单条
# 普通管理员和超级用户可以操作所有，除了访问单条和所有的那个视图类，加上认证类
# books：查看一条，和所有
# booksdetail 路由下有：删除，新增，修改 ---> 权限类加在这里

# book 5个接口，必须登录才能访问
# 5个接口分成了俩视图写:
    BookView：获取所有，获取单条
    BookDetailView：删除，修改，新增
    这俩视图都需要登录：authentication_classes = [LoginAuth, ]
    BookView只要登陆就可以操作
    BookDetailView必须有权限才能，加了一个权限，permission_classes = [UserPermission, ]

# 跟写认证类步骤差不多
第一步：写一个类，继承BasePermission，重写has_permission，判断如果有权限，返回True，如果没有权限，返回False
第二步：局部使用和全局使用
    局部使用
    class BookDetailView(GenericViewSet, CreateModelMixin, DestroyModelMixin, UpdateModelMixin):
      permission_classes = [UserPermission, ]

    全局使用
      REST_FRAMEWORK={
        "DEFAULT_PERMISSION_CLASSES":["app01.auth.UserPermission",]
      }
```

```python
# 在 app01 下创建 auth.py
from app01 import models
from rest_framework.permissions import BasePermission

class MyPermission(BasePermission):
    def has_permission(self, request, view):
        self.message = "您是: %s，权限不足" % request.user.get_user_type_display()  # 没有权限的提示信息
        user_type = request.user.user_type
				# 如果有权限，返回True,没有权限返回False
        # 权限类，在认证类之后，request.user有了当前登录用户
        if user_type > 2:  # 是 1 和 2 就没有权限
            return False
        else:
            return True
```

#### 权限类的使用

```python
# 局部使用（在视图类中加）
  from app01 import auth
	permission_classes = [auth.MyPermission,]
# 全局使用（在配置文件中配置）
  REST_FRAMEWORK={
      "DEFAULT_PERMISSION_CLASSES":["app01.auth.MyPermission",],
  }
```

使用一个普通用户登录的 token 访问需要权限的视图类就会失败
![image-20220405163147257](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220405163147257.png)

### 频率

#### 频率类的编写

```python
# 认证，权限都通过以后，现在某个接口的访问频率 ---> 一般根据ip或者用户限制

# 使用步骤
  第一步：写一个类，继承SimpleRateThrottle，重写类属性：scope，和get_cache_key方法
    get_cache_key返回什么，就以什么做现在，scope配置文件中要用

  第二步：在配置文件中配置
    'DEFAULT_THROTTLE_RATES': {
        'minute_3': '3/m'  # minute_3是scope的字符串，一分钟访问3次
        'minute_5'：'5/m'
    }

  局部使用 ---> 视图类中
  class BookView(GenericViewSet, ListModelMixin, RetrieveModelMixin):
    throttle_classes = [IPThrottle]
  全局使用--配置文件中
     'DEFAULT_THROTTLE_CLASSES': (  # 全局配置频率类
        'app01.auth.IPThrottle'
     ),
```

```python
# 在 app01 下创建 auth.py
# 以客户端IP限制
class IPThrottle(SimpleRateThrottle):
    scope = 'min_throttle'
    def get_cache_key(self, request, view):
        return self.get_ident(request)  # 真实的客户端IP
        # return request.META.get('REMOTE_ADDR')  # 有代理的IP地址


# 以用户限制
class UserThrottle(SimpleRateThrottle):
    scope = 'user_throttle'
    def get_cache_key(self, request, view):
        return request.user.id
```

#### 频率类的使用

```python
# 全局使用 settings.py
## 以IP地址限制
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": ["app01.auth.IPThrottle",],
    "DEFAULT_THROTTLE_RATES": {
        'min_throttle': '3/m',  # 每分钟三次的访问
    }
}

## 以用户限制
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": ["app01.auth.UserThrottle",],
    "DEFAULT_THROTTLE_RATES": {
        'user_throttle': '5/m',,  # 每分钟五次的访问
    }
}

# 局部使用 视图类中写 throttle_classes 属性
  from app01 import auth
  # 以IP限制
  throttle_classes = [auth.IPThrottle]
  # 以用户限制
  throttle_classes = [auth.UserThrottle]
  # settings.py 文件中的 "DEFAULT_THROTTLE_RATES" 还是需要设置
```

### 综合使用

#### `models.py`

```python
from django.db import models


class User(models.Model):
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=32)
    user_type = models.IntegerField(choices=((1, "超级用户"), (2, "管理用户"), (3, "普通用户")))


class UserToken(models.Model):
    user = models.OneToOneField(to="User", on_delete=models.CASCADE)
    token = models.CharField(max_length=64)


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


class Publish(models.Model):
    name = models.CharField(max_length=32)
    city = models.CharField(max_length=32)
    email = models.EmailField()

    def __str__(self):
        return self.name


class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    author_detail = models.OneToOneField(to="AuthorDetail", on_delete=models.CASCADE)

    def __str__(self):
        return self.name

    def tel(self):
        return self.author_detail.tel

    def addr(self):
        return self.author_detail.addr

    @property
    def detail_info(self):
        return {'phone': self.author_detail.tel, 'address': self.author_detail.addr}


class AuthorDetail(models.Model):
    tel = models.CharField(max_length=32)
    addr = models.CharField(max_length=32)

    def __str__(self):
        return self.addr
```

#### `serializer.py`

```python
from rest_framework import serializers
from app01 import models


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.User
        fields = "__all__"


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


class PublishSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Publish
        fields = "__all__"


class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Author
        fields = ["id", "name", "age", "tel", "addr"]

    tel = serializers.CharField()
    addr = serializers.CharField()

    # 重写了 create 和 update 方法 在插入作者的时候可以将详情以前插入
    def create(self, validated_data):
        detail = models.AuthorDetail.objects.create(tel=validated_data.get("tel"), addr=validated_data.get("addr"))
        author = models.Author.objects.create(author_detail=detail, name=validated_data.get('name'),
                                              age=validated_data.get('age'))
        return author

    def update(self, instance, validated_data):
        name = validated_data.get('name')
        age = validated_data.get('age')
        tel = validated_data.get('tel')
        addr = validated_data.get('addr')
        instance.author_detail.tel = tel
        instance.author_detail.addr = addr
        instance.name = name
        instance.age = age
        instance.save()
        return instance
```

#### `auth.py`

```python
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from app01 import models
from rest_framework.permissions import BasePermission
from rest_framework.throttling import BaseThrottle, SimpleRateThrottle


class LoginAuth(BaseAuthentication):
    def authenticate(self, request):
        token = request.query_params.get('token')
        user_token = models.UserToken.objects.filter(token=token).first()
        if user_token:
            return user_token.user, token
        else:
            raise AuthenticationFailed('您没有登录')


class MyPermission(BasePermission):
    def has_permission(self, request, view):
        self.message = "您是: %s，权限不足" % request.user.get_user_type_display()
        user_type = request.user.user_type
        if user_type > 2:  # 不是1 2 就是3 没有权限
            return False
        else:
            return True


# 以客户端IP
class IPThrottle(SimpleRateThrottle):
    scope = 'min_throttle'

    def get_cache_key(self, request, view):
        return self.get_ident(request)  # 真实的客户端IP
        # return request.META.get('REMOTE_ADDR')  # 客户端ip


# 以用户限制
class UserThrottle(SimpleRateThrottle):
    scope = 'user_throttle'

    def get_cache_key(self, request, view):
        return request.user.id
```

#### `views.py`

```python
import uuid
from rest_framework.response import Response
from app01 import models, serializer, auth
from rest_framework.viewsets import ViewSetMixin, ModelViewSet
from rest_framework.generics import CreateAPIView
from rest_framework.decorators import action


class UserView(ViewSetMixin, CreateAPIView):
    authentication_classes = []  # 登录接口禁用认证类
    queryset = models.User.objects.all()
    serializer_class = serializer.UserSerializer

    @action(methods=['POST'], detail=False)
    def login(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        user = models.User.objects.filter(username=username, password=password).first()
        token = uuid.uuid4()
        models.UserToken.objects.update_or_create(user=user, defaults={'token': token})
        if user:
            return Response({'msg': '登录成功', 'token': token})
        else:
            return Response({'status': 101, 'msg': '用户名或密码错误'})


class BookView(ModelViewSet):
    queryset = models.Books.objects.all()
    serializer_class = serializer.BookSerializer


class PublishView(ModelViewSet):
    permission_classes = [auth.MyPermission, ]  # 有管理员权限才可以操作该接口
    queryset = models.Publish.objects.all()
    serializer_class = serializer.PublishSerializer


class AuthorView(ModelViewSet):
    queryset = models.Author.objects.all()
    serializer_class = serializer.AuthorSerializer
```

#### `urls.py`

```python
from django.contrib import admin
from django.urls import path, include
from app01 import views
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register('user', views.UserView)
router.register('books', views.BookView)
router.register('publish', views.PublishView)
router.register('authors', views.AuthorView)
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
]
```

#### `settings.py`

```python
# drf配置
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": ["app01.auth.LoginAuth", ],
    "DEFAULT_THROTTLE_CLASSES": ["app01.auth.IPThrottle",],  # 使用的是以IP为限制
    # "DEFAULT_THROTTLE_CLASSES": ["app01.auth.UserThrottle",],
    "DEFAULT_THROTTLE_RATES": {
        'min_throttle': '3/m',  # 每分钟只允许访问三次
        # 'user_throttle': '5/m',
    }
}
```
