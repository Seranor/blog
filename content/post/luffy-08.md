---
title: 支付功能
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-08.html
toc: true
---

### 支付宝支付介绍

<!-- more -->

```python
# 目前线上付款方式有多种：支付宝，微信，银联
# 以支付宝为例
# 官方提供了API接口，原来没有提供python的sdk，使用第三方
# 现在官方提供了python的sdk可以尝试使用官方sdk

# 使用支付宝支付，需要是企业，要有营业执照才能申请，支付宝商家
# 测试环境，沙箱环境，即便咱们不是商家，也可以测试，以后换成真正的appid，公钥私钥--》付款就到真正的地址
# 沙箱环境：https://openhome.alipay.com/platform/appDaily.htm?tab=info
# 沙箱版支付宝客户端：一个app，连得环境是测试环境，里面的钱可以一直冲，支付付到测试环境里


# 支付宝支付的场景
	https://opendocs.alipay.com/open/270/105898

# 流程
	网站有个立即购买按钮
  用户点击
  向后端发送一个请求
  后端生成一个没有支付的订单和支付连接(支付宝的)
  返回给前端
  前端放在支付连接的地址
  显示出支付宝支付页面
  app扫码支付，账号密码支付
  支付成功
  支付宝收到了支付
  支付宝会回调回咱们项目支付成功的页面
  页面展示用户购买成功
  支付宝还有一个post回调
  咱们项目利用post回调，修改订单状态
```

![image-20220504154604470](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220504154604470.png)

### 支付宝支付 SDK

#### 测试使用

```python
# 非官方的
https://github.com/fzlee/alipay

# 安装
pip install python-alipay-sdk --upgrade

# 生成公钥私钥
	对称加密 ：加密和解密使用同一个秘钥
  非对称加密：加密用公钥，解密用私钥
  补充：支付宝为了更安全，需要使用自己的公钥配置在网站上，再生成一个支付宝公钥
  https://opendocs.alipay.com/common/02kipl
  下载软件，在软件中生成

  创建文件夹 scripts/t_alipay 存放
  软件生成的秘钥
  软件生成的公钥
  支付宝公钥

# 测试脚本 scripts/t_alipay/pay_test.py
  from alipay import AliPay

  app_private_key_string = open("./key.txt").read()
  app_public_key_string = open("./pub_alipay.txt").read()
  a = AliPay(
      appid='2021000119681771',
      app_notify_url=None,
      app_private_key_string=app_private_key_string,
      alipay_public_key_string=app_public_key_string,
      sign_type='RSA2',
      debug=True
  )

  res = a.api_alipay_trade_page_pay(
      subject="测试支付",
      out_trade_no="2112233",
      total_amount=float(999),  # 只有生成支付宝链接时，不能用Decimal
      return_url='http://192.168.0.198:8080/actual/detail/1',
      notify_url='http://192.168.0.198:8080/actual/detail/1',
  )

  print("https://openapi.alipaydev.com/gateway.do?" + res)


  '''
  公钥和私钥首尾格式
  -----BEGIN RSA PRIVATE KEY-----
  base64 encoded content
  -----END RSA PRIVATE KEY-----

  -----BEGIN PUBLIC KEY-----
  base64 encoded content
  -----END PUBLIC KEY-----

  会生成支付链接
  '''
```

![image-20220504160731166](/Users/zhijinliu/Library/Application Support/typora-user-images/image-20220504160731166.png)

#### 二次封装

目录结构

```python
luffy_api/libs/iPay            # aliapy二次封装包
├── __init__.py				         # 包文件
├── pay.py                     # 支付文件
├── pem                        # 公钥私钥文件夹
│   ├── alipay_public_key.pem  # 支付宝公钥文件
│   └── app_private_key.pem    # 应用私钥文件
└── settings.py                # 应用配置

'''
注意公钥私钥首尾格式
'''
```

`libs/iPay/settings.py`

```python
import os

# 应用私钥
APP_PRIVATE_KEY_STRING = open(
    os.path.join(os.path.dirname(os.path.abspath(__file__)), 'pem', 'app_private_key.pem')).read()
# 支付宝公钥
ALIPAY_PUBLIC_KEY_STRING = open(
    os.path.join(os.path.dirname(os.path.abspath(__file__)), 'pem', 'alipay_public_key.pem')).read()

# 应用ID
APP_ID = '2021000119681771'

# 加密方式
SIGN = 'RSA2'

# 是否是支付宝测试环境(沙箱环境)，如果采用真是支付宝环境，配置False
DEBUG = True

# 支付网关
GATEWAY = 'https://openapi.alipaydev.com/gateway.do' if DEBUG else 'https://openapi.alipay.com/gateway.do'
```

`libs/iPay/pay.py`

```python
from alipay import AliPay
from . import settings

# 支付对象
alipay = AliPay(
    appid=settings.APP_ID,
    app_notify_url=None,
    app_private_key_string=settings.APP_PRIVATE_KEY_STRING,
    alipay_public_key_string=settings.ALIPAY_PUBLIC_KEY_STRING,
    sign_type=settings.SIGN,
    debug=settings.DEBUG
)

# 支付网关
gateway = settings.GATEWAY
```

```python
# __init__.py
from .pay import gateway, alipay
```

### 支付接口

```python
登录之后才可以访问前端的支付按钮
点击支付按钮，向后端发送 post 请求 {courses:[1,2,3],total_amount:99}
生成订单(订单表)
生成支付链接
返回给前端

扩写 auth 的 user表 可以使用drf-jwt 提供的认证类、权限类、
```

#### 表设计

```python
# 订单表
# 订单详情表

"""
class Order(models.Model):
    # 主键、总金额、订单名、订单号、订单状态、创建时间、支付时间、流水号、支付方式、支付人(外键) - 优惠劵(外键，可为空)
    pass

class OrderDetail(models.Model):
    # 订单号(外键)、商品(外键)、实价、成交价 - 商品数量
    pass
"""

# 创建 app
cd luffy_api/apps
python3 ../../manage.py startapp order

# 复制一个 urls.py 过来

# 注册 app
INSTALLED_APPS = [
    'order',
]

# 总路由注册 order
path('api/v1/order/', include('order.urls')),  # http://127.0.0.1:8000/api/v1/order/
```

#### `apps/order/models.py`

```python
from django.db import models

from user.models import User
from course.models import Course


# 订单表
class Order(models.Model):
    """订单模型"""
    status_choices = (
        (0, '未支付'),
        (1, '已支付'),
        (2, '已取消'),
        (3, '超时取消'),
    )
    pay_choices = (
        (1, '支付宝'),
        (2, '微信支付'),
    )
    # 订单标题
    subject = models.CharField(max_length=150, verbose_name="订单标题")
    # 总价格
    total_amount = models.DecimalField(max_digits=10, decimal_places=2, verbose_name="订单总价", default=0)
    # 订单号--使用uuid生成
    out_trade_no = models.CharField(max_length=64, verbose_name="订单号", unique=True)
    # 流水号支付宝返回的
    trade_no = models.CharField(max_length=64, null=True, verbose_name="流水号")
    # 订单状态  待支付，已支付。。。
    order_status = models.SmallIntegerField(choices=status_choices, default=0, verbose_name="订单状态")
    # 微信，支付宝
    pay_type = models.SmallIntegerField(choices=pay_choices, default=1, verbose_name="支付方式")
    # 支付时间--》支付宝回调回来会有
    pay_time = models.DateTimeField(null=True, verbose_name="支付时间")
    # 用户表关联
    user = models.ForeignKey(User, related_name='order_user', on_delete=models.DO_NOTHING, db_constraint=False,
                             verbose_name="下单用户")
    # 订单创建时间  auto_now_add:新增这个时间可以不传，用当前时间   auto_now：修改时间不传，自动存入当前时间
    created_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')

    class Meta:
        db_table = "luffy_order"
        verbose_name = "订单记录"
        verbose_name_plural = "订单记录"

    def __str__(self):
        return "%s - ￥%s" % (self.subject, self.total_amount)

    @property
    def courses(self):
        data_list = []
        for item in self.order_courses.all():
            data_list.append({
                "id": item.id,
                "course_name": item.course.name,
                "real_price": item.real_price,
            })
        return data_list


# 订单详情表
class OrderDetail(models.Model):
    """订单详情"""
    # 跟订单一对多
    order = models.ForeignKey(Order, related_name='order_courses', on_delete=models.CASCADE, db_constraint=False,
                              verbose_name="订单")
    # 跟课程一对多
    course = models.ForeignKey(Course, related_name='course_orders', on_delete=models.CASCADE, db_constraint=False,
                               verbose_name="课程")
    # 价格
    price = models.DecimalField(max_digits=6, decimal_places=2, verbose_name="课程原价")
    # 真实价格
    real_price = models.DecimalField(max_digits=6, decimal_places=2, verbose_name="课程实价")

    class Meta:
        db_table = "luffy_order_detail"
        verbose_name = "订单详情"
        verbose_name_plural = "订单详情"

    def __str__(self):
        try:
            return "%s的订单：%s" % (self.course.name, self.order.out_trade_no)
        except:
            return super().__str__()

```

#### `apps/order/admin.py`

```python
from django.contrib import admin
from .models import *

admin.site.register(Order)
admin.site.register(OrderDetail)
```

```python
python3 manage.py makemigrations
python3 manage.py migrate
```

#### 实现接口

#### `apps/order/views.py`

```python
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import CreateModelMixin
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
from utils.APIResponse import APIResponse
from utils.luffy_logging import logger
from .models import Order
from .serializer import OrderSerializer


class OrderView(GenericViewSet, CreateModelMixin):
    # 登录之后才可以访问
    authentication_classes = [JSONWebTokenAuthentication, ]
    permission_classes = [IsAuthenticated, ]
    queryset = Order.objects.all()
    serializer_class = OrderSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        pay_url = serializer.context.get('pay_url')
        self.perform_create(serializer)

        return APIResponse(data={'pay_url': pay_url})


# 支付回调接口
class SuccessViewSet(APIView):
    # 认证取消
    authentication_classes = ()
    permission_classes = ()

    # 支付宝同步回调给前台，在同步通知给后台处理
    # 写不写都行---》给咱们前端做二次验证的
    def get(self, request, *args, **kwargs):
        # return Response('后台已知晓，Over！！！')
        # 不能在该接口完成订单修改操作
        # 但是可以在该接口中校验订单状态(已经收到支付宝post异步通知，订单已修改)，告诉前台
        # print(type(request.query_params))  # django.http.request.QueryDict
        # print(type(request.query_params.dict()))  # dict

        out_trade_no = request.query_params.get('out_trade_no')
        try:
            Order.objects.get(out_trade_no=out_trade_no, order_status=1)
            return APIResponse(status=100, msg='订单支付成功')
        except:
            return APIResponse(status=101, msg='订单还未支付')

    # 支付宝异步回调处理
    def post(self, request, *args, **kwargs):
        try:
            # request.data前端(支付宝)post传给咱们的数据--》request.data--》 QueyDic对象，不允许pop，把它转成字典
            result_data = request.data.dict()
            # 支付宝给我的订单号---》数据库有个订单号
            out_trade_no = result_data.get('out_trade_no')
            # 前面-->验证签名才信任支付宝，防止伪造
            signature = result_data.pop('sign')
            from libs import iPay
            result = iPay.alipay.verify(result_data, signature)
            if result and result_data["trade_status"] in ("TRADE_SUCCESS", "TRADE_FINISHED"):
                # 完成订单修改：订单状态、流水号、支付时间
                # 已支付
                Order.objects.filter(out_trade_no=out_trade_no).update(order_status=1)
                # 完成日志记录
                logger.warning('%s订单支付成功' % out_trade_no)
                return Response('success')
            else:
                logger.error('%s订单支付失败' % out_trade_no)
        except:
            pass
        return Response('failed')

```

#### `apps/order/serializer.py`

```python
from django.conf import settings
from rest_framework import serializers
from rest_framework.validators import ValidationError
from libs import iPay
from .models import Order, Course, OrderDetail


# 只用来做反序列化--->数据校验-->重写create方法--》存两个表
class OrderSerializer(serializers.ModelSerializer):
    # 前端传入的是课程列表[1,2,3,]---》转成[课程对象1，课程对象2，课程对象三]
    courses = serializers.PrimaryKeyRelatedField(queryset=Course.objects.all(), many=True)

    class Meta:
        model = Order
        fields = ['subject', 'total_amount', 'pay_type', 'courses']

    def _check_total_amount(self, attrs):
        courses = attrs.get('courses')  # 列表[课程1，课程2，课程3]
        total_amount = attrs.get('total_amount')  # 前端传入的总价格
        total_price = 0
        for course in courses:
            total_price += course.price
        if total_price != total_amount:  # 计算完的价格不等于传入的价格，抛异常
            raise ValidationError('total_amount error')
        return total_amount

    def _get_out_trade_no(self):
        import uuid
        code = str(uuid.uuid4())  # 分布式id的生成方案：订单号全局唯一，如何生成全局唯一的订单号：uuid，使用当前时间时间戳(重复概率)，雪花算法
        return code.replace('-', '')

    # 获取支付人
    def _get_user(self):
        return self.context.get('request').user

    # 获取支付链接
    def _get_pay_url(self, out_trade_no, total_amount, subject):
        order_string = iPay.alipay.api_alipay_trade_page_pay(
            out_trade_no=out_trade_no,
            total_amount=float(total_amount),  # 只有生成支付宝链接时，不能用Decimal
            subject=subject,
            return_url=settings.RETURN_URL,  # get回调地址，前台地址
            notify_url=settings.NOTIFY_URL,  # post回调地址,后台地址
        )
        pay_url = iPay.gateway + '?' + order_string
        # 将支付链接存入，传递给views
        self.context['pay_url'] = pay_url

    # 入库(两个表)的信息准备
    def _before_create(self, attrs, user, out_trade_no):
        attrs['user'] = user
        attrs['out_trade_no'] = out_trade_no

    def validate(self, attrs):
        # 1）订单总价校验--
        total_amount = self._check_total_amount(attrs)
        # 2）生成订单号--》唯一的
        out_trade_no = self._get_out_trade_no()
        # 3）支付用户：request.user
        user = self._get_user()
        # 4）支付链接生成
        self._get_pay_url(out_trade_no, total_amount, attrs.get('subject'))
        # 5）入库(两个表)的信息准备
        self._before_create(attrs, user, out_trade_no)

        # 代表该校验方法通过，进入入库操作
        return attrs

    def create(self, validated_data):
        courses = validated_data.pop('courses')
        # 订单表入库，不需要courses, 订单号，订单标题，订单价格，购买人，支付方式
        order = Order.objects.create(**validated_data)
        # 订单详情表入库：只需要订单对象，课程对象(courses要拆成一个个course)
        for course in courses:
            OrderDetail.objects.create(order=order, course=course, price=course.price, real_price=course.price)
        return order

```

#### `apps/order/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from .views import OrderView, SuccessViewSet

router = SimpleRouter()
router.register('pay', OrderView, 'pay')
urlpatterns = [
    path('', include(router.urls)),
    path('success/', SuccessViewSet.as_view()),
]
```

### 支付前端

在课程章节和详情的页面 `src/views/CourseDetail.vue` 中的 立即购买 功能( go_pay 方法)

内网中回调会出现问题，可以使用内网穿透

#### `src/views/PaySuccess.vue`

支付回调页面

```vue
<template>
  <div class="pay-success">
    <!--如果是单独的页面，就没必要展示导航栏(带有登录的用户)-->
    <Header />
    <div class="main">
      <div class="title">
        <div class="success-tips">
          <p class="tips">您已成功购买 1 门课程！</p>
        </div>
      </div>
      <div class="order-info">
        <p class="info">
          <b>订单号：</b><span>{{ result.out_trade_no }}</span>
        </p>
        <p class="info">
          <b>交易号：</b><span>{{ result.trade_no }}</span>
        </p>
        <p class="info">
          <b>付款时间：</b
          ><span
            ><span>{{ result.timestamp }}</span></span
          >
        </p>
      </div>
      <div class="study">
        <span>立即学习</span>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "@/components/Header";

export default {
  name: "Success",
  data() {
    return {
      result: {},
    };
  },
  created() {
    // url后拼接的参数：?及后面的所有参数 => ?a=1&b=2
    // console.log(location.search);

    // 解析支付宝回调的url参数
    let params = location.search.substring(1); // 去除? => a=1&b=2
    let items = params.length ? params.split("&") : []; // ['a=1', 'b=2']
    //逐个将每一项添加到args对象中
    for (let i = 0; i < items.length; i++) {
      // 第一次循环a=1，第二次b=2
      let k_v = items[i].split("="); // ['a', '1']
      //解码操作，因为查询字符串经过编码的
      if (k_v.length >= 2) {
        // url编码反解
        let k = decodeURIComponent(k_v[0]);
        this.result[k] = decodeURIComponent(k_v[1]);
        // 没有url编码反解
        // this.result[k_v[0]] = k_v[1];
      }
    }
    // 解析后的结果
    // console.log(this.result);

    // 把地址栏上面的支付结果，再get请求转发给后端
    this.$axios({
      url: this.$settings.base_url + "order/success/" + location.search,
      method: "get",
    })
      .then((response) => {
        console.log(response.data);
        if (response.data.status != 100) {
          alert("暂时还没收到您的支付，请稍后刷新再试");
        }
      })
      .catch(() => {
        console.log("支付结果同步失败");
      });
  },
  components: {
    Header,
  },
};
</script>

<style scoped>
.main {
  padding: 60px 0;
  margin: 0 auto;
  width: 1200px;
  background: #fff;
}

.main .title {
  display: flex;
  -ms-flex-align: center;
  align-items: center;
  padding: 25px 40px;
  border-bottom: 1px solid #f2f2f2;
}

.main .title .success-tips {
  box-sizing: border-box;
}

.title img {
  vertical-align: middle;
  width: 60px;
  height: 60px;
  margin-right: 40px;
}

.title .success-tips {
  box-sizing: border-box;
}

.title .tips {
  font-size: 26px;
  color: #000;
}

.info span {
  color: #ec6730;
}

.order-info {
  padding: 25px 48px;
  padding-bottom: 15px;
  border-bottom: 1px solid #f2f2f2;
}

.order-info p {
  display: -ms-flexbox;
  display: flex;
  margin-bottom: 10px;
  font-size: 16px;
}

.order-info p b {
  font-weight: 400;
  color: #9d9d9d;
  white-space: nowrap;
}

.study {
  padding: 25px 40px;
}

.study span {
  display: block;
  width: 140px;
  height: 42px;
  text-align: center;
  line-height: 42px;
  cursor: pointer;
  background: #ffc210;
  border-radius: 6px;
  font-size: 16px;
  color: #fff;
}
</style>
```

#### `src/router/index.js`

```javascript
...
import PaySuccess from "@/views/PaySuccess";
Vue.use(VueRouter)

const routes = [
    ...
    {
        path: '/pay/success',
        name: 'pay-success',
        component: PaySuccess
    },
]
```

### 文件托管

```python
# 图片，视频，一般不放在自己项目中--->只要访问一次图片，就向后端发一次请求--》消耗服务器资源
# 静态类型文件，托管到第三方平台---》花钱---》使用第三方平台的服务器资源和带宽
# 视频托管到云平台了
	-阿里 oss
  -七牛云存储

# 公司里多种方案---》托管到其他平台
	-云平台---》花钱---》七牛云
  -公司自己搭建文件服务器
  	-fastdfs：搭建笔记，python上传：https://zhuanlan.zhihu.com/p/372286804
    -Minio：自己搭建Minio，阿里云上--》搭建笔记--》上传下载比较简单
    	-图形化界面后台，登陆上看到文件
    -ceph
    。。。
  -放到项目中

# 七牛云上传
	-手动上传---》手动配置视频地址--（不采取）
  -前端上传，返回连接地址，提交到咱们自己的后端，存到数据库
  -前端把视频传到咱们后端，咱们后端通过python传到七牛云，生成连接，存到数据库
```
