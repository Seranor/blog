---
title: 首页主体、轮播功能和跨域问题
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-02.html
toc: true
---

### 前台主页

#### `Banner.vue`

<!-- more -->

```vue
<template>
  <div class="banner">
    <el-carousel :interval="5000" arrow="always" height="400px">
      <el-carousel-item v-for="item in 4" :key="item">
        <img src="../assets/img/banner1.png" alt="" />
      </el-carousel-item>
    </el-carousel>
  </div>
</template>

<script>
export default {
  name: "Banner",
};
</script>

<style scoped>
el-carousel-item {
  height: 400px;
  min-width: 1200px;
}

.el-carousel__item img {
  height: 400px;
  margin-left: calc(50% - 1920px / 2);
}
</style>
```

#### `Header.vue`

```vue
// 需要在 assets目录创建 img 目录放入相关图片 否则会报错
<template>
  <div class="header">
    <div class="slogan">
      <p>老男孩IT教育 | 帮助有志向的年轻人通过努力学习获得体面的工作和生活</p>
    </div>
    <div class="nav">
      <ul class="left-part">
        <li class="logo">
          <router-link to="/">
            <img src="../assets/img/head-logo.svg" alt="" />
          </router-link>
        </li>
        <li class="ele">
          <span
            @click="goPage('/free-course')"
            :class="{ active: url_path === '/free-course' }"
            >免费课</span
          >
        </li>
        <li class="ele">
          <span
            @click="goPage('/actual-course')"
            :class="{ active: url_path === '/actual-course' }"
            >实战课</span
          >
        </li>
        <li class="ele">
          <span
            @click="goPage('/light-course')"
            :class="{ active: url_path === '/light-course' }"
            >轻课</span
          >
        </li>
      </ul>

      <div class="right-part">
        <div>
          <span>登录</span>
          <span class="line">|</span>
          <span>注册</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Header",
  data() {
    return {
      url_path: sessionStorage.url_path || "/",
    };
  },
  methods: {
    goPage(url_path) {
      // 已经是当前路由就没有必要重新跳转
      if (this.url_path !== url_path) {
        this.$router.push(url_path);
      }
      sessionStorage.url_path = url_path;
    },
  },
  created() {
    sessionStorage.url_path = this.$route.path;
    this.url_path = this.$route.path;
  },
};
</script>

<style scoped>
.header {
  background-color: white;
  box-shadow: 0 0 5px 0 #aaa;
}

.header:after {
  content: "";
  display: block;
  clear: both;
}

.slogan {
  background-color: #eee;
  height: 40px;
}

.slogan p {
  width: 1200px;
  margin: 0 auto;
  color: #aaa;
  font-size: 13px;
  line-height: 40px;
}

.nav {
  background-color: white;
  user-select: none;
  width: 1200px;
  margin: 0 auto;
}

.nav ul {
  padding: 15px 0;
  float: left;
}

.nav ul:after {
  clear: both;
  content: "";
  display: block;
}

.nav ul li {
  float: left;
}

.logo {
  margin-right: 20px;
}

.ele {
  margin: 0 20px;
}

.ele span {
  display: block;
  font: 15px/36px "微软雅黑";
  border-bottom: 2px solid transparent;
  cursor: pointer;
}

.ele span:hover {
  border-bottom-color: orange;
}

.ele span.active {
  color: orange;
  border-bottom-color: orange;
}

.right-part {
  float: right;
}

.right-part .line {
  margin: 0 10px;
}

.right-part span {
  line-height: 68px;
  cursor: pointer;
}
</style>
```

#### `Footer.vue`

```vue
<template>
  <div class="footer">
    <ul>
      <li>关于我们</li>
      <li>联系我们</li>
      <li>商务合作</li>
      <li>帮助中心</li>
      <li>意见反馈</li>
      <li>新手指南</li>
    </ul>
    <p>Copyright © luffycity.com版权所有 | 京ICP备17072161号-1</p>
  </div>
</template>

<script>
export default {
  name: "Footer",
};
</script>

<style scoped>
.footer {
  width: 100%;
  height: 128px;
  background: #25292e;
  color: #fff;
}

.footer ul {
  margin: 0 auto 16px;
  padding-top: 38px;
  width: 810px;
}

.footer ul li {
  float: left;
  width: 112px;
  margin: 0 10px;
  text-align: center;
  font-size: 14px;
}

.footer ul::after {
  content: "";
  display: block;
  clear: both;
}

.footer p {
  text-align: center;
  font-size: 12px;
}
</style>
```

#### `Homeview.vue`

```vue
<template>
  <div class="home">
    <Header></Header>
    <Banner></Banner>
    <div class="course">
      <el-row>
        <el-col :span="6" v-for="(o, index) in 8" :key="o">
          <el-card :body-style="{ padding: '0px' }" class="course_card">
            <img
              src="https://tva1.sinaimg.cn/large/e6c9d24egy1h1g0zd133mj20l20a875i.jpg"
              class="image"
            />
            <div style="padding: 14px;">
              <span>推荐的课程</span>
              <div class="bottom clearfix">
                <time class="time">价格：100元</time>
                <el-button type="text" class="button">查看详情</el-button>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>
    <img
      src="https://tva1.sinaimg.cn/large/e6c9d24egy1h1g112oiclj224l0u0jxl.jpg"
      alt=""
      height="500px"
      width="100%"
    />
    <Footer></Footer>
  </div>
</template>

<script>
import Footer from "@/components/Footer";
import Header from "@/components/Header";
import Banner from "@/components/Banner";

export default {
  name: "HomeView",
  data() {
    return {};
  },
  components: {
    Footer,
    Header,
    Banner,
  },
};
</script>

<style>
.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}

.course {
  margin-left: 20px;
  margin-right: 20px;
}

.course_card {
  margin: 50px;
}
</style>
```

### 公共基表

```python
# utils/model.py
from django.db import models


# 5个公共字段
class BaseModel(models.Model):
    created_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_time = models.DateTimeField(auto_now=True, verbose_name='最后更新时间')
    is_delete = models.BooleanField(default=False, verbose_name='是否删除')
    is_show = models.BooleanField(default=True, verbose_name='是否上架')
    orders = models.IntegerField(verbose_name='优先级')

    class Meta:
        abstract = True  # 表示它是虚拟的，不在数据库中生成表，它只用来做继承
```

### 轮播图接口

```python
# 创建 home app
cd luffy_api/apps
python ../../manage.py startapp home

# 配置文件中注册
INSTALLED_APPS = [
    # ...
    'home',
]
```

#### 表设计

```python
# 轮播图表设计  luffy_api/apps/home/models.py
from django.db import models
from utils.model import BaseModel

# 继承基表之后只需要写 title，image，info，link 字段
class Banner(BaseModel):
    title = models.CharField(max_length=16, unique=True, verbose_name='名称')
    image = models.ImageField(upload_to='banner', verbose_name='图片')
    # 在前端点击图片，会跳转到某个地址
    link = models.CharField(max_length=64, verbose_name='跳转链接')
    info = models.TextField(verbose_name='详情')  # 也可以用详情表，宽高出处

    class Meta:
        db_table = 'luffy_banner'
        verbose_name_plural = '轮播图表'

    def __str__(self):
        return self.title

```

#### 迁移数据

```python
python manage.py makemigrations  # 如果没有变化，是app没注册
python manage.py migrate
python manage.py createsuperuser  # 创建个用户
```

#### 后台引入 simpleui

```python
# 下载
pip install django-simpleui

# setting/dev.py 注册app
INSTALLED_APPS = [
      'simpleui',
      ...
  ]

# 在 apps/home/admin.py 中添加内容
from django.contrib import admin
from .models import Banner

@admin.register(Banner)
class BannerAdmin(admin.ModelAdmin):
    list_display = ('id', 'title', 'link','is_show', 'is_delete')

    # 增加自定义按钮
    actions = ['make_copy']
    def make_copy(self, request, queryset):
        # 选中一些数据，点击 【自定义按钮】  触发方法执行，传入你选中 queryset
        # 保存，删除
        print(queryset)
    make_copy.short_description = '自定义按钮'

# 访问后台录入数据
```

![image-20220420193319631](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220420193319631.png)

![image-20220420193332208](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220420193332208.png)

#### 接口代码

##### 接口格式

```python
{
  code:100,
  msg:成功，
  result:[
    {
      img:地址，
      link:跳转地址，
      orders:顺序，
      title:名字
    }
  ]
}
```

##### 总路由

```python
# luffy_api/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.views.static import serve

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/home/', include('home.urls')),  # http://127.0.0.1:8000/api/v1/home/banner/
    path('media/<path:path>', serve, {'document_root': settings.MEDIA_ROOT})
]

```

##### home 路由

```python
# luffy_api/apps/home/urls.py
from django.contrib import admin
from django.urls import path, include
from .views import BannerView
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register('banner', BannerView, 'banner')
urlpatterns = [
    path('', include(router.urls))
]
```

##### 视图类

```python
# luffy_api/apps/home/views.py
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import ListModelMixin
from utils.APIResponse import APIResponse
from .models import Banner
from .serializer import BannerSerializer

class BannerView(GenericViewSet, ListModelMixin):
    # 获取所有接口 list，自动生成路由
    queryset = Banner.objects.filter(is_delete=False, is_show=True).order_by('orders')
    serializer_class = BannerSerializer

    def list(self, request, *args, **kwargs)  # 重写list 可以将这段提取出来做公共方法
        res = super().list(request, *args, **kwargs)
        return APIResponse(data=res.data)
```

##### 序列化类

```python
# luffy_api/apps/home/serializer.py
from rest_framework import serializers
from .models import Banner

class BannerSerializer(serializers.ModelSerializer):
    class Meta:
        model = Banner
        fields = ['title', 'image', 'link', 'orders']
```

### 跨域问题

#### 跨域错误和同源策略

```python
# 现在写好了后端接口，前端加载数据时会报如下错误
'''
localhost/:1 Access to XMLHttpRequest at
'http://127.0.0.1:8000/api/v1/home/banner/'
from origin 'http://localhost:8080'
has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header
is present on the requested resource.
'''

该错误是跨域的错误

# 浏览器的同源策略
  请求的url地址,必须与浏览器上的url地址处于同域上,也就是域名,端口,协议相同，否则，加载回来的数据就会禁止
  前端：http://127.0.0.1:8080
  后端：http://127.0.0.1:8000
  这俩属于不同源，协议，地址一样，但是端口不一样，所以请求成功，但是到了浏览器被禁止掉了，因为浏览器的同源策略
  前后端分离，就会遇到这个问题，解决这个问题
```

#### 解决方式

```python
1. jsonp 通过 img，script，link 这些标签回调(已经不常用了)
   https://www.zhihu.com/question/19966531

2. 通过CORS(跨域资源共享)
   CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能
   实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信
   只需要在响应头中指定，允许跨域即可
```

#### `CORS`

```python
# CORS请求种类
  浏览器将CORS请求分成两类：
  简单请求（simple request）
  非简单请求（not-so-simple request）

# 种类的划分
只要同时满足以下两大条件，就属于简单请求，否则就是非简单请求
1.请求方法是以下三种方法之一
  HEAD
  GET
  POST
2.HTTP的头信息不超出以下几种字段
  Accept
  Accept-Language
  Content-Language
  Last-Event-ID
  Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

问：post，josn格式是什么请求？ 非简单

# 简单请求和非简单请求的区别
简单请求：一次请求，直接发真正的请求，如果允许，数据拿回来，如果不允许，浏览器拦截
非简单请求：两次请求，在发送数据之前会先发一次请求用于做“预检”，只有“预检”通过后才再发送一次请求用于数据传输。非简单请求发两次，第一次是OPTIONS请求，如果允许跨域，再发真正的请求
```

#### 解决跨域问题

```python
简单请求，在响应头中加入 "Access-Control-Allow-Origin":"*"
  headers={"Access-Control-Allow-Origin": "*"}

非简单请求，需要增加判断，如果是OPTIONS请求，在请求头中加入允许

# 放到中间件处理复杂和简单请求
from django.utils.deprecation import MiddlewareMixin
class CorsMiddleWare(MiddlewareMixin):
    def process_response(self,request,response):
        if request.method=="OPTIONS":
            #可以加*
            response["Access-Control-Allow-Headers"]="Content-Type"
        response["Access-Control-Allow-Origin"] = "*"
        return response

# 使用第三方模块
## 下载
pip install django-cors-headers

## 配置文件中注册app
INSTALLED_APPS = [
    ...
    'corsheaders',
    ...
]

## 中间件注册
MIDDLEWARE = [  # Or MIDDLEWARE_CLASSES on Django < 1.10
    ...
    'corsheaders.middleware.CorsMiddleware',
    ...
]

## 配置文件中配置如下参数
    # 跨域问题处理
    # 允许简单请求，所有地址 相当于CORS_ORIGIN_ALLOW_ALL="*"
    CORS_ALLOW_ALL_ORIGINS = True
    # 运行的请求
    CORS_ALLOW_METHODS = (
        'DELETE',
        'GET',
        'OPTIONS',
        'PATCH',
        'POST',
        'PUT',
        'VIEW',
    )

    # 允许的请求头
    CORS_ALLOW_HEADERS = (
      'XMLHttpRequest',
      'X_FILENAME',
      'accept-encoding',
      'authorization',  # jwt
      'content-type',  # json
      'dnt',
      'origin',
      'user-agent',
      'x-csrftoken',
      'x-requested-with',
      'Pragma',
    )
```

简单请求

![image-20220420213246456](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220420213246456.png)

![image-20220420213221975](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220420213221975.png)

非简单请求

![image-20220420213100218](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220420213100218.png)

![image-20220420213028715](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220420213028715.png)

### 前后端打通

#### `Banner.vue`

```vue
<template>
  <div class="banner">
    <el-carousel :interval="5000" arrow="always" height="400px">
      <el-carousel-item v-for="item in banner_list" :key="item.image">
        // 不能直接使用item对象
        <img :src="item.image" alt="" />
      </el-carousel-item>
    </el-carousel>
  </div>
</template>

<script>
export default {
  name: "Banner",
  data() {
    return {
      banner_list: [],
    };
  },
  created() {
    this.$axios.get(this.$settings.base_url + "home/banner/").then((res) => {
      if (res.data.status == 100) {
        this.banner_list = res.data.data;
      }
    });
  },
};
</script>

<style scoped>
el-carousel-item {
  height: 400px;
  min-width: 1200px;
}

.el-carousel__item img {
  height: 400px;
  margin-left: calc(50% - 1920px / 2);
}
</style>
```

### 后端自定义配置

```python
# 创建luffy_api/setting/user_settings.py
	BANNER_COUNT=3

# 在 luffy_api/setting/dev.py 中导入
# 导入用户自定义的配置
from .user_settings import *

# 使用，例如在views.py中
from django.conf import settings
...
    queryset = Banner.objects.filter(is_delete=False, is_show=True).order_by('orders')[
               :settings.BANNER_COUNT]  # 自定义轮播图片数量
...
```
