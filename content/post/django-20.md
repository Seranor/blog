---
title: RBAC、JWT和SimpleUI
lastmod: 2021-09-03T16:43:23+08:00
date: 2021-09-02T11:52:03+08:00
tags:
  - Django
  - DRF
categories:
  - Django
url: post/django-20.html
toc: true
---

### RBAC

<!-- more -->

```python

# python用来做公司内部项目居多，人事系统，进销存，报销审批，自动化运维
  公司内部项目对执行效率要求不高（人少）
  对开发效率要求高（越快开发出越好，成本越低越好）
  知乎，豆瓣用python写的 ---> 随着用户量增大 ---> 切换语言
# 对外的权限比较简单：普通注册用户，VIP用户，超级VIP ---> 优酷，网易云音乐，百度网盘
# 公司内部系统：通常使用RBAC的权限控制
  公司内有部门（开发部，运维部，市场部，总裁办，人力资源部门）
  权限和角色(部门)绑定
  例子：发工资权限，招人权限，开发代码权限 ---> 人权限，发工资权限给人力资源--》开发代码权限给开发部门



# RBAC  是基于角色的访问控制（Role-Based Access Control ）在 RBAC  中，权限与角色相关联，用户通过成为适当角色的成员而得到这些角色的权限。这就极大地简化了权限的管理。这样管理都是层级相互依赖的，权限赋予给角色，而把角色又赋予用户，这样的权限设计很清楚，管理起来很方便
# 权限赋予给角色(部门)，而把角色(部门)又赋予用户

# 针对于公司内部项目，后台管理居多(运营在使用),使用rbac居多
# django-vue-admin：后端用drf，前端用vue，权限管理的 脚手架 ---> 后端分离
# django的admin ---> 合的后台管理用的多 ---> 于django的admin二次开发
# simpleui：对django admin的美化
# django的admin自带rbac权限管理（表设计完成权限管理）---> 张表
  -用户表
  -角色表(组表，部门表)
  -权限表
  ----------
  -角色和权限多对多中间表
  -用户和角色多对多中间表
  -----django-admin中多了一张表-----
  用户对权限多对多中间表

# 于django的admin做二次开发，开发出公司内部的管理系统
  纯基于原生
  使用第三方美化：xadmin(早就不维护了，弃坑了)，simpleui(国内的，主流)，国外也有很多

# 在关系型数据库的关系中：只有三种---》本质只有一种：外键关系
  一对多
  多对多
  一对一

# 前后端分离，验证码如何实现？写验证码接口
```

### 自动生成接口文档

```python
# 顺利的写接口 ---> 在公司里，前端和后端是两拨人写 ---> 咱们后端接口写好了，我们知道接口怎么用
127.0.0.1:8080/user/login ---> post ---> {"username":'lxx','pwd':123} ---> {status:100,msg:登陆成功，token:dafasdf}

# 后端，需要写出接口文档，给前端用，前端按照接口文档去开发
# 具体格式可以参照：https://open.weibo.com/wiki/2/comments/show

# 如何写
  第一种：使用word或者md文档编写 ---> 纯手写 ---> 好的公司这么用
  第二种：第三方平台录入 ---> 半手写 ---> https://blog.csdn.net/weixin_44337261/article/details/121005675 ---> 部分公司
  第三种：公司自己开发接口平台，搭建接口平台 ---> 数据放在公司自己
         https://zhuanlan.zhihu.com/p/366025001

  第四种：自动生成接口文档 ---> 用的少 ---> 自动生成+导出 ---> 录入到yapi
         coreapi，swagger

  # 如何写好接口文档
  http://www.liuqingzheng.top/article/1/02-%E5%A6%82%E4%BD%95%E5%86%99%E5%A5%BD%E6%8E%A5%E5%8F%A3%E6%96%87%E6%A1%A3/
```

#### `coreapi`

```python
# 安装
  pip3 install coreapi

# 使用步骤
    ## 在路由中
      from rest_framework.documentation import include_docs_urls
      urlpatterns = [
      path('doc/', include_docs_urls(title='路飞项目接口文档')),
      ]
  ## 在配置文件中
      REST_FRAMEWORK = {
        'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.coreapi.AutoSchema',
      }
  ## 在视图类中方法上加注释即可
  ## 如果是ModelViewSet
    	"""
       list:
       返回图书列表数据，通过Ordering字段排序

       retrieve:
       返回图书详情数据

       latest:
       返回最新的图书数据

       read:
       查询单个图书接口
       """
  ## 字段描述，写在models的help_text上
  ## 访问 http://127.0.0.1:8000/docs/
```

#### `swagger`

```python
# 安装
pip install django-rest-swagger

# 使用步骤
## 配置
    INSTALLED_APPS = [
        'rest_framework_swagger',
    ]
## 路由
    # 导入辅助函数get_schema_view
    from rest_framework.schemas import get_schema_view
    from rest_framework_swagger.renderers import SwaggerUIRenderer, OpenAPIRenderer
    schema_view = get_schema_view(title='API接口文档', renderer_classes=[SwaggerUIRenderer, OpenAPIRenderer])

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('docs/', schema_view),
    ]
## 在视图类中方法上加注释即可
## 访问 http://127.0.0.1:8000/docs/
```

### `JWT`

#### 介绍

```python
# cookie，session，token的区别？
https://www.cnblogs.com/liuqingzheng/articles/8990027.html

# 认证：session机制:需要在后端存储数据 ---> 之前使用的django-session
# 如果登录用户很多，需要在后端存很多数据，频繁查询数据库，导致效率低---》能不能想一种方案，不在服务端存数据 ---> 客户端存数据(数据安全) ---> token认证机制

# Json web token (JWT)，token是一种认证机制，用在web开发方向，叫jwt

# JWT的构成 ---> 三段式 ---> 每一段都使用base64编码
  典型的jwt串样子，通过. 分隔成三段：

 eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

  第一段：头：声明类型，这里是jwt，声明加密的算法，公司信息等等 ---> 目前作用不大
  第二段：荷载(payload): 有效信息
         用户名，用户id，登陆时间，token失效时间
  第三段：签名(signature): 通过 头+荷载 使用某种加密方式加密后得到的
```

#### `base64`

```python
# base64编码和解码---》只是编码和解码，不能叫加密
import base64

# 编码
# s = b'''{"name":"lqz","age":19}'''
# res = base64.b64encode(s)
# print(res)  # eyJuYW1lIjoibHF6IiwiYWdlIjoxOX0=

# 解码
# s=b'eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9'
# res=base64.b64decode(s)
# print(res)
```

#### `jwt`签发和认证

```python
## jwt的签发和认证 ---> 保证安全
# 签发 ---> 登陆过程 ---> 如果没有第三方模块帮助我们做，我们就自己做
"""
1）用基本信息公司信息存储json字典，采用base64算法得到 头字符串
2）用关键信息存储json字典，采用base64算法得到 荷载字符串，过期时间，用户id，用户名
3）用头、体加密字符串通过加密算法+秘钥加密得到 签名字符串
拼接成token返回给前台
"""


# 认证 ---> 访问需要登陆的接口
"""
1）将token按 . 拆分为三段字符串，第一段 头加密字符串 一般不需要做任何处理
2）第二段 体加密字符串，要反解出用户主键，通过主键从User表中就能得到登录用户，过期时间是安全信息，确保token没过期
3）再用 第一段 + 第二段 + 加密方式和秘钥得到一个加密串，与第三段 签名字符串 进行比较，通过后才能代表第二段校验得到的user对象就是合法的登录用户
"""

# 大部分的web框架都会有第三方模块支持 ---> 如果没有需要自己写
django中有一个django-rest-framework-jwt，咱们讲的：
# https://github.com/jpadilla/django-rest-framework-jwt
django中有一个，django-rest-framework-simplejwt，咱们不讲，公司可能会用：
# https://github.com/jazzband/djangorestframework-simplejwt

# 区别
# https://blog.csdn.net/lady_killer9/article/details/103075076
```

#### 快速使用

##### 签发

```python
# 第一步：pip3 install djangorestframework-jwt
# 第二步：在路由中配置
from rest_framework_jwt.views import obtain_jwt_token
urlpatterns = [
    path('login/', obtain_jwt_token),
]

# 第三步：使用接口测试工具发送post请求到后端，就能基于auth的user表签发token
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InB5eSIsImV4cCI6MTY0OTMxNjc3MiwiZW1haWwiOiIzMDYzMzQ2NzhAcXEuY29tIn0.YOUc9gcFIQf9FBibZJANaI3Rmpw4zfMy1e8ez1roKSI"
}


# 比session优势在不需要在后端存数据了，jwt不会在后端存数据，保证了数据安全

### 会有被别人窃取的风险----》只能拿到和使用---》避免不了窃取后使用，只能避免别人篡改不了---》爬虫就是干这事，扣除token串，模拟发请求----》如果你能想一种方式，避免别人获取了token串，不能发请求---》你整个就把爬虫行业干掉了
```

##### 认证

```python
# 视图类的某个方法，访问时候，需要认证通过才能访问---》写认证类-->我们用了第三方模块---》第三方模块写了一个认证类

# 在视图类中配置：认证类+权限类
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
from rest_framework.permissions import IsAuthenticated
class BookView(GenericViewSet,ListModelMixin):
    # JSONWebTokenAuthentication :rest_framework_jwt模块写的认证类
    authentication_classes = [JSONWebTokenAuthentication,]
    # 需要配合一个权限类
    permission_classes = [IsAuthenticated,]


### 在前端使用的时候，要携带token，token携带方式：---》为什么要按这个格式？人家的认证类已经写完了，就是去请求头中取的，按固定规则取的，所以咱们需要按照这个格式
## 后期咱么要自己基于自定义的User表，签发token，和自定义认证类
在请求头中使用  Authorization : jwt token串
```

![image-20220410155707528](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220410155707528.png)

#### `JWT`定制返回格式

```python
# 签发token，其实就是jwt模块基于auth的user表，帮咱们写了一个登陆功能，但是一般请求，我们登陆成功后，返回的数据更多 {code:100,msg:登陆成功，token:adfadf,username:pyy}

# 定义签发token(登陆接口)返回格式
	# 第一步：写一个函数 ---> 返回什么格式，前端就能看到什么格式
  	# app01/utils.py
  	def jwt_response_payload_handler(token, user=None, request=None):
    return {
        'code': 100,
        'msg': "登陆成功",
        'token': token,
        'username': user.username

    }
  # 第二步：在配置文件中配置
  # jwt模块的配置文件，统一放在JWT_AUTH
    import datetime
    JWT_AUTH = {
        # 过期时间1天
        'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
        # 自定义认证结果：见下方序列化user和自定义response
        # 如果不自定义，返回的格式是固定的，只有token字段
        'JWT_RESPONSE_PAYLOAD_HANDLER': 'users.utils.jwt_response_payload_handler',
    }
```

#### 自定义签发`token`

```python
# 如果项目中的User表使用auth的user表，使用快速签发token即可
# 如果自定义User表，签发token，需要手动签发 ---> 自己写
```

##### 普通写法

```python
# models.py
  class UserInfo(models.Model):
      username = models.CharField(max_length=32)
      password = models.CharField(max_length=32)
      email = models.CharField(max_length=32, null=True)

# views.py
  from .models import UserInfo
  from rest_framework.viewsets import ViewSetMixin
  from rest_framework.decorators import action
  from rest_framework.views import APIView
  from rest_framework.response import Response
  from rest_framework_jwt.settings import api_settings

  jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
  jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER


  class UserView(ViewSetMixin, APIView):
      # /user/login/--->post请求
      @action(methods=['POST'], detail=False)
      def login(self, request):
          res_dic = {'code': 100, 'msg': 'success'}
          username = request.data.get('username')
          password = request.data.get('password')
          user = UserInfo.objects.filter(username=username, password=password).first()
          if user:
              payload = jwt_payload_handler(user)
              print(payload)
              token = jwt_encode_handler(payload)
              res_dic['token'] = token
              res_dic['username'] = user.username
              return Response(res_dic)
          else:
              res_dic['code'] = 101
              res_dic['msg'] = '用户名或密码错误'
              return Response(res_dic)

# urls.py
  from django.contrib import admin
  from django.urls import path, include
  from app01 import views
  from rest_framework.routers import SimpleRouter

  router = SimpleRouter()
  router.register('user', views.UserView, 'user')
  urlpatterns = [
      path('admin/', admin.site.urls),
      path('', include(router.urls)),
  ]
```

##### 写在序列化类中

```python
# models.py
  class UserInfo(models.Model):
      username = models.CharField(max_length=32)
      password = models.CharField(max_length=32)
      email = models.CharField(max_length=32, null=True)

# serializer.py
  from rest_framework import serializers
  from .models import UserInfo
  from rest_framework_jwt.settings import api_settings
  from rest_framework.exceptions import ValidationError

  jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
  jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

  class UserInfoSerializer(serializers.ModelSerializer):
      class Meta:
          model = UserInfo
          fields = ['username', 'password']  # 根据表模型中写的，字段自己有校验规则

      def validate(self, attrs):
          # 打印请求方式
          print(self.context.get('request').method)
          # 签发token逻辑，签发生成token，放到ser.context字典中
          username = attrs.get('username')
          password = attrs.get('password')
          user = UserInfo.objects.filter(username=username, password=password).first()
          if user:
              payload = jwt_payload_handler(user)  # 得到荷载 --> 字典
              token = jwt_encode_handler(payload)  # 通过荷载得到token串
              # serializer和视图类沟通的桥梁
              self.context['token'] = token
              self.context['username'] = username
          else:
              raise ValidationError('用户名或密码错误')
          return attrs

# views.py
  from .serializer import UserInfoSerializer
  from rest_framework.viewsets import ViewSetMixin
  from rest_framework.decorators import action
  from rest_framework.views import APIView
  from rest_framework.response import Response

  class UserView(ViewSetMixin, APIView):
      @action(methods=['POST'], detail=False)
      def login(self, request):
          res_dic = {'code': 100, 'msg': 'success'}
          ser = UserInfoSerializer(data=request.data, context={'request': request})
          if ser.is_valid():  # 这句话会走：字段自己的校验规则，局部钩子，全局钩子
              token = ser.context.get('token')
              username = ser.context.get('username')
              res_dic['token'] = token
              res_dic['username'] = username
          else:
              res_dic['code'] = 101
              res_dic['msg'] = ser.errors
          return Response(res_dic)

# urls.py
  from django.contrib import admin
  from django.urls import path, include
  from app01 import views
  from rest_framework.routers import SimpleRouter

  router = SimpleRouter()
  router.register('user', views.UserView, 'user')
  urlpatterns = [
      path('admin/', admin.site.urls),
      path('', include(router.urls)),
  ]
```

##### 自定义认证类

###### `app01/models.py`

```python
from django.db import models

class UserInfo(models.Model):
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=32)
    email = models.CharField(max_length=32, null=True)

class Books(models.Model):
    name = models.CharField(max_length=32)
    price = models.IntegerField()
    publish = models.CharField(max_length=32)

    class Meta:
        verbose_name = '图书'
        verbose_name_plural = '图书'

    def __str__(self):
        return self.name
```

###### `app01/serializer.py`

```python
from rest_framework import serializers
from .models import Books, UserInfo
from rest_framework_jwt.settings import api_settings
from rest_framework.exceptions import ValidationError

jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER


class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Books
        fields = '__all__'

class UserInfoSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserInfo
        fields = ['username', 'password']

    def validate(self, attrs):
        print(self.context.get('request').method)
        username = attrs.get('username')
        password = attrs.get('password')
        user = UserInfo.objects.filter(username=username, password=password).first()
        if user:
            payload = jwt_payload_handler(user)
            token = jwt_encode_handler(payload)
            self.context['token'] = token
            self.context['username'] = username
        else:
            raise ValidationError('用户名或密码错误')
        return attrs
```

###### `app01/auth.py`

```python
import jwt
# from django.utils.translation import ugettext as _
from rest_framework.authentication import BaseAuthentication
from rest_framework_jwt.settings import api_settings
from rest_framework.exceptions import AuthenticationFailed
from .models import UserInfo

jwt_decode_handler = api_settings.JWT_DECODE_HANDLER


class JWTAuthentication(BaseAuthentication):
    def authenticate(self, request):
        jwt_value = request.META.get('HTTP_TOKEN')
        # 第一步: 取出传入的token --> 从哪去--> 咱们定的：请求地址？请求头？
        # token=request.query_params.get('token') # 请求地址？
        # http请求头中的数据，在META中, 统一变成 HTTP_请求头的key大写
        if jwt_value:
            try:
                # 得到荷载
                payload = jwt_decode_handler(jwt_value)
            except jwt.ExpiredSignature:
                msg = '签名过期'
                raise AuthenticationFailed(msg)
            except jwt.DecodeError:
                msg = '签名被篡改'
                raise AuthenticationFailed(msg)
            except jwt.InvalidTokenError:
                raise AuthenticationFailed('未知错误')
            # 通过payload获得当前登录用户，效率更高一写，不需要查数据库了
            user = UserInfo.objects.filter(pk=payload['user_id']).first()
            return (user, jwt_value)
        else:
            raise AuthenticationFailed('您没有携带token')
```

###### `app01/views.py`

```python
from .models import Books
from .serializer import BookSerializer,UserInfoSerializer
from .auth import JWTAuthentication
from rest_framework.mixins import ListModelMixin
from rest_framework.viewsets import GenericViewSet, ViewSetMixin
from rest_framework.decorators import action
from rest_framework.views import APIView
from rest_framework.response import Response

class UserView(ViewSetMixin, APIView):
    @action(methods=['POST'], detail=False)
    def login(self, request):
        res_dic = {'code': 100, 'msg': 'success'}
        ser = UserInfoSerializer(data=request.data, context={'request': request})
        if ser.is_valid():
            token = ser.context.get('token')
            username = ser.context.get('username')
            res_dic['token'] = token
            res_dic['username'] = username
        else:
            res_dic['code'] = 101
            res_dic['msg'] = ser.errors
        return Response(res_dic)


class BookView(GenericViewSet, ListModelMixin):
    authentication_classes = [JWTAuthentication, ]
    queryset = Books.objects.all()
    serializer_class = BookSerializer
```

###### `urls.py`

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework_jwt.views import obtain_jwt_token
from app01 import views
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register('books', views.BookView, 'books')
router.register('user', views.UserView, 'user')
urlpatterns = [
    path('admin/', admin.site.urls),
    path('login/', obtain_jwt_token),
    path('', include(router.urls)),

]
```

#### `JWT`源码分析

```python
# 签发源码
obtain_jwt_token ---> ObtainJSONWebToken.as_view()：视图类.as_view ---> ObtainJSONWebToken视图类 ---> 登陆post请求，携带用户名密码 ---> 视图类中有post方法
# 视图类中的序列化类： serializer_class = JSONWebTokenSerializer ---> 全局钩子中获取当前登录用户和签发token
# post方法中：serializer = self.get_serializer(data=request.data)


### 你会的：obtain_jwt_token ---> ObtainJSONWebToken视图类 ---> post方法 ---> 通过前端传入的用户名和密码拿到了当前用户，通过当前用户签发了token ---> 返回给了前端


# 认证源码
  -认证类 ---> authenticate方法 ---> 验证
  def authenticate(self, request):
        jwt_value = self.get_jwt_value(request) # 获取真正的token，三段式
        if jwt_value is None: # 如果没传token，就不认证了，直接通过，所以需要配合权限类一起用
            return None

        try:
            payload = jwt_decode_handler(jwt_value) # 验证签名
        except jwt.ExpiredSignature:
            msg = _('Signature has expired.') # 过期了
            raise exceptions.AuthenticationFailed(msg)
        except jwt.DecodeError:
            msg = _('Error decoding signature.') # 被篡改了
            raise exceptions.AuthenticationFailed(msg)
        except jwt.InvalidTokenError:
            raise exceptions.AuthenticationFailed() # 不知名的错误

        user = self.authenticate_credentials(payload)

        return (user, jwt_value)

# jwt配置入口位置 api_settings
from rest_framework_jwt.settings import api_settings

# token的过期时间设置
# settings.py中添加配置
# JWT_AUTH = {
#     'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300),
# }
```

### `simpleui`使用

```python
# django-admin混合开发 --> 后台管理 --> 美化 --> simpleui

# 使用步骤：
  # 安装
  	pip3 install django-simpleui
  # 注册app  放在最上面
    INSTALLED_APPS = [
      'simpleui',
    ]
  # settings.py 定制左侧菜单
  	SIMPLEUI_CONFIG = {
    'system_keep': False,
    'menu_display': ['监控大屏','应用1', '权限认证', '测试', '动态菜单测试'],  # 开启排序和过滤功能, 不填此字段为默认排序和全部显示, 空列表[] 为全部不显示.
    'dynamic': True,  # 设置是否开启动态菜单, 默认为False. 如果开启, 则会在每次用户登陆时动态展示菜单内容
    'menus': [
        {
            'name': '监控大屏',
            'icon': 'fas fa-code',
            'url': '/index/'
        },
        {
            'app': 'app01',
            'name': '应用1',
            'icon': 'fas fa-user-shield',
            'models': [
                {
                    'name': '图书',
                    'icon': 'fa fa-user',
                    'url': 'app01/book/'
                },
                {
                    'name': '用户',
                    'icon': 'fa fa-user',
                    'url': 'app01/userinfo/'
                }
            ]
        },
        {
            'app': 'auth',
            'name': '权限认证',
            'icon': 'fas fa-user-shield',
            'models': [
                {
                    'name': '用户',
                    'icon': 'fa fa-user',
                    'url': 'auth/user/'
                },
                {
                    'name': '用户组',
                    'icon': 'fa fa-user',
                    'url': 'auth/group/'
                }
            ]
        },
        {
            # 自2021.02.01+ 支持多级菜单，models 为子菜单名
            'name': '测试',
            'icon': 'fa fa-file',
            # 二级菜单
            'models': [{
                'name': 'Baidu',
                'icon': 'far fa-surprise',
                # 第三级菜单 ，
                'models': [
                    {
                        'name': '爱奇艺',
                        'url': 'https://www.iqiyi.com/dianshiju/'
                        # 第四级就不支持了，element只支持了3级
                    }, {
                        'name': '百度问答',
                        'icon': 'far fa-surprise',
                        'url': 'https://zhidao.baidu.com/'
                    }
                ]
            }, {
                'name': '内网穿透',
                'url': 'https://www.wezoz.com',
                'icon': 'fab fa-github'
            }]
        },
        {
            'name': '动态菜单测试',
            'icon': 'fa fa-desktop',
            'models': [{
                'name': time.time(),
                'url': 'http://baidu.com',
                'icon': 'far fa-surprise'
            }]
        }
    ]
}


"""
自带权限
自定义左侧菜单的页面显示
通过混合开发，编写路径，配置到上面即可
更多操作见官方
"""
```
