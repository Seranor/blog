---
title: Celery分布式的异步任务框架
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-05.html
toc: true
---

### `Celery`介绍

```python
# Celery: 分布式的异步任务框架
Celery是一个简单、灵活且可靠的，处理大量消息的分布式系统
Celery is a project with minimal funding, so we don’t support Microsoft Windows. Please don’t open any issues related to that platform.

# celery: 能做什么事，解决什么问题？
  异步任务: 项目中同步的操作，可以通过celery把它做成异步
  延迟任务: 隔一会再执行任务
  定时任务: 每隔多长时间干什么事
    如果你的项目仅仅想做定时任务，没有必要使用celery，使用apscheduler
    https://www.cnblogs.com/xiao-xue-di/p/14081790.html

'''
可以不依赖任何服务器，通过自身命令，启动服务
celery服务为为其他项目服务提供异步解决任务需求的
注：会有两个服务同时运行，一个是项目服务(django)，一个是celery服务，项目服务将需要异步处理的任务交给celery服务，celery就会在需要时异步完成项目的需求
'''
```

<!-- more -->

### `Celery`架构

```python
broker: 任务中间件，消息队列中间件，存储任务，celery本身不提供，需要借助第三方：redis，rabbitmq..
worker: 任务执行单元，真正指向任务的进程，celery提供的
backend:结果存储，任务执行结果存在某个地方，借助于第三方：redis
```

![image-20220427161427983](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220427161427983.png)

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/celery.png)

### `Celery`快速使用

```python
# 安装
pip install celery

# 创建 t_celery/celery_task.py 写任务
    from celery import Celery
    import time
    # 消息中间件
    broker = 'redis://127.0.0.1:6379/1'
    # 结果存储
    # backend = 'redis://:123456@127.0.0.1:6379/1'
    backend = 'redis://127.0.0.1:6379/2'
    app = Celery('test', broker=broker, backend=backend)

    # 写任务
    @app.task
    def add(a, b):
        time.sleep(1)
        return a + b

# 提交任务 t_celery/add_task.py
    from celery_task import add
    # res = add(1, 2)  # 同步调用
    # print(res)
    res = add.apply_async(args=[7, 8])  # 将任务提交到任务中间件中
    print(res)  # 得到任务ID: 090eb8b8-6a64-40f4-9177-c67e753cd447
    # 该任务ID可以在redis中查到

# 任务就被提交到redis中了，等待worker执行该任务，启动worker
    # 启动worker执行任务 ---> 使用命令启动 如果有虚拟环境需要进入到虚拟环境内
    # cd t_celery 目录下执行
    # 非windows
    celery -A celery_task worker -l info

    # windows
    pip3 install eventlet
    celery -A celery_task worker -l info -P eventlet

# 执行完成结果在redis中，可以直接查到
    # 通过代码取出 t_celery/get_result.py
    from celery_task import app  # 借助于app
    from celery.result import AsyncResult  # 导入一个类，来查询结果

    id = '090eb8b8-6a64-40f4-9177-c67e753cd447'
    if __name__ == '__main__':
        res = AsyncResult(id=id, app=app)  # 根据id，去哪个app中找哪个任务，
        if res.successful():  # 执行成功
            result = res.get()
            print('任务执行成功')
            print(result)  # 15
        elif res.failed():
            print('任务失败')
        elif res.status == 'PENDING':
            print('任务等待中被执行')
        elif res.status == 'RETRY':
            print('任务异常后正在重试')
        elif res.status == 'STARTED':
            print('任务已经开始被执行')
```

```python
# 借助于celery的异步秒杀场景分析
# 原始同步场景
100个人，秒杀3个商品 ---> 100个人在浏览器等着开始 ---> 一旦开始 ---> 瞬间100个人同时发送秒杀请求到后端 ----> 假设秒杀函数执行2s钟 ---> 100个请求在2s内，一直跟后端连着，假设我的并发量是100，这两秒钟，其他任何人都访问不了了
假设 150人来秒杀 ---> 最多能承受100个人，50个人就请求不了 ---> 不友好

# 异步场景
100个人，秒杀3个商品 ---> 100个人在浏览器等着开始 ---> 一旦开始---> 瞬间100个人同时发送秒杀请求到后端 ---> 假设秒杀函数执行2s钟 ---> 当前100个请求，过来，使用celery提交100个任务，请求立马返回 ---> 这样的话，2s内能提交特别多的任务，可以接收特别多人发的请求 ---> 后台使用worker慢慢的执行秒杀任务 ---> 多起几个worker ---> 过了一会，所有提交的任务都执行完了

提交完任务，返回前端 ---> 前端使用个动态图片盖住页面，显示您正在排队，每个2s钟，向后端发送一次ajax请求，带着id号，查询结果是否完成，如果没完成 ---> 再等2s钟 ---> 如果秒杀成功了，显示恭喜您，成了 ---> 如果没有成功，显示很遗憾，没有秒到
```

### `Celery`封装包使用

```python
.
├── celery_task        # 包
│   ├── __init__.py
│   ├── celery.py      # 写app的py文件
│   ├── home_task.py   # 任务一
│   ├── order_task.py  # 任务一
│   └── user_task.py   # 任务一
├── add_task.py        # 提交任务
└── get_result.py      # 查询结果
'''
提交任务和查询结果可以在不同的项目中
'''
```

#### `celery.py`

```python
from celery import Celery
# 消息中间件
# backend = 'redis://:123456@127.0.0.1:6379/1'
backend = 'redis://127.0.0.1:6379/2'
# 结果存储
broker = 'redis://127.0.0.1:6379/1'
app = Celery('test', broker=broker, backend=backend, include=[
    'celery_task.home_task',
    'celery_task.user_task',
    'celery_task.order_task',
])

# 在 celery_task 包同级下执行下面命令
# celery -A celery_task worker -l info
```

#### `user_task.py`

```python
from .celery import app

@app.task
def send_sms(phone):
    print('%s 短信发送成功' % phone)
    return 'sms send ok'
```

#### `add_task.py`

```python
from celery_task.user_task import send_sms

res = send_sms.apply_async(args=['13088888888', ])
print(res)
```

#### `get_result.py`

```python
from celery_task.celery import app
from celery.result import AsyncResult

id = '88f9b8cd-a09a-4aae-9f4a-023790dd5637'
if __name__ == '__main__':
    res = AsyncResult(id=id, app=app)  # 根据id，去哪个app中找哪个任务，
    if res.successful():  # 执行成功
        result = res.get()
        print('任务执行成功')
        print(result)  # 15
    elif res.failed():
        print('任务失败')
    elif res.status == 'PENDING':
        print('任务等待中被执行')
    elif res.status == 'RETRY':
        print('任务异常后正在重试')
    elif res.status == 'STARTED':
        print('任务已经开始被执行')
```

### `Celery`执行异步任务

```python
# 使用上述封装的包 add_task.py 中使用
from celery_task.user_task import send_sms

res = send_sms.delay('13088888888')  # 直接执行异步任务
print(res)
```

### `Celery`执行延迟任务

```python
from datetime import datetime, timedelta

# 构建时间对象
eta = datetime.utcnow() + timedelta(seconds=10)  # 10s后
# eta = datetime.utcnow() + timedelta(days=3)  # 3天后后时间
res = send_sms.apply_async(args=['13088888888', ], eta=eta)  # 10s后执行
print(res)
```

### `Celery`执行定时任务

```python
# 配置定时任务 在 celery_task/celery.py 中配置
    # celery的配置信息
    # 时区
    app.conf.timezone = 'Asia/Shanghai'
    # 是否使用UTC
    app.conf.enable_utc = False

    # 配置定时任务
    from datetime import timedelta
    from celery.schedules import crontab

    app.conf.beat_schedule = {
        'send_sms_5': {
            'task': 'celery_task.user_task.send_sms',  # 指定哪个任务
            'schedule': timedelta(seconds=5),  # 每5s执行一次
            # 'schedule': crontab(hour=8, day_of_week=1),  # 每周一早八点
            'args': ('18988377473',),
        },
    }

# 在 celery_task 同级下启动 worker
celery -A celery_task worker -l info

# 在 celery_task 同级下启动 beat
celery -A celery_task beat -l info

# 本质是 beat 5s 提交一次任务，worker 执行
```

### `Django`中`Celery`的使用

```python
# 将 celery_task 包放在项目目录下 celery.py 添加下面配置
# celery_task 的包也可以放在 apps 内 使用时注意 include 路径即可
  # 加载 django 配置环境
  import os
  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "luffy_api.setting.dev")

# celery_task/user_task.py
  from .celery import app

  @app.task
  def create_user(mobile, username, password):
      # 一旦使用了djang内的 就要加载配置
      from user.models import User
      User.objects.create_user(mobile=mobile, username=username, password=password)
      return True

# 使用 apps/user/views.py
from celery_task.user_task import create_user
class TestView(APIView):
    def get(self, requeste):
        create_user.delay('18066666666', 'lzjuser', 'asd123...')
        return Response('用户创建任务提交')

# 配置路由
  path('test/', TestView.as_view()),

# 此时就要在项目第一层下启动 worker
  celery -A celery_task worker -l info

# 访问就会创建 user
  http://127.0.0.1:8000/api/v1/user/test/
```

### 定时更新轮播图接口

```python
'''
首页轮播图现在是去mysql中查的，假设瞬间10w访问首页，数据库会查询10w次返回数据，但是实际上轮播图基本不变

优化思路：对轮播图接口做个缓存，第一次访问查询mysql，放到reids中，以后都从redis中取，如果redis中没有，再去数据库中查，好处在于，对mysql压力小，redis性能高

以后如果接口响应慢，第一想法先加缓存：把查出来的数据缓存到redis中，再来请求，先从redis中查，如果没有，再去mysql查，然后在redis缓存一份
'''
```

#### 视图中加入缓存

```python
class BannerView(GenericViewSet, ListModelMixin, UpdateModelMixin):
    queryset = Banner.objects.filter(is_delete=False, is_show=True).order_by('orders')[
               :settings.BANNER_COUNT]  # 自定义轮播图片数量
    serializer_class = BannerSerializer

    def list(self, request, *args, **kwargs):
        # 先去 redis 中查询 如果有就返回，如果没有就执行 super() 去数据库中查询
        banner_list = cache.get('banner_list')
        if banner_list:
            print('走了缓存')
            return APIResponse(data=banner_list)
        else:
            print('没走缓存')
            res = super().list(request, *args, **kwargs)
            cache.set('banner_list', res.data)  # 可以设置键的超时时间 timeout=300
            return APIResponse(data=res.data)
```

#### 加入缓存的问题

```python
'''
redis中有一份数据，mysql中有一份数据
存在问题:mysql更新了，reids更新了么？
双写一致性问题  写入mysql，redis是否同步
'''

# 解决方案 根据业务场景选择
  1.定时更新: 例如10分钟更新一次缓存
  2.写入mysql时删除缓存
  3.写入mysql时更新缓存

# 轮播图通过celery定时更新，解决双写一致性问题
```

```python
# celery_task/home_task.py
from .celery import app
from home import models, serializer
from django.conf import settings
from django.core.cache import cache

@app.task
def update_banner_list():
    queryset = models.Banner.objects.filter(is_delete=False, is_show=True).order_by('-orders')[:settings.BANNER_COUNT]
    banner_list = serializer.BannerSerializer(queryset, many=True).data
    # 拿不到request对象，所以头像的连接base_url要自己组装
    for banner in banner_list:
        banner['image'] = 'http://127.0.0.1:8000%s' % banner['image']

    cache.set('banner_list', banner_list, 86400)
    return True

```

```python
# celery_task/celery.py
from celery import Celery
import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "luffy_api.setting.dev")

backend = 'redis://127.0.0.1:6379/2'
broker = 'redis://127.0.0.1:6379/1'

app = Celery('test', broker=broker, backend=backend, include=[
    'celery_task.home_task',
    'celery_task.user_task',
    'celery_task.order_task',
])

app.conf.timezone = 'Asia/Shanghai'
app.conf.enable_utc = False
from datetime import timedelta
from celery.schedules import crontab

app.conf.beat_schedule = {
    'update_banner_5': {
        'task': 'celery_task.home_task.update_banner_list',
        'schedule': timedelta(seconds=5),
        'args': (),
    },
}
```

```python
# 在 celery_task 同级下启动 worker
celery -A celery_task worker -l info

# 在 celery_task 同级下启动 beat
celery -A celery_task beat -l info

# 此时在数据库中将 banner 表的数据 改为 is_delete 观察前后端的情况
```

```python

执行异步任务时报错：

django.db.utils.DatabaseError: DatabaseWrapper objects created in a thread can only be used in that same thread. The object with alias 'default' was created in thread id 45710528 and this is thread id 128109152.

原先启动命令

celery -A xxx worker -l info -P eventlet

修改后的启动命令（XXX为yourapp.celery）

# celery版本4.0.0

celery -A XXX worker --loglevel=info --pool=solo

# celery版本5.0.0

celery -A XXX worker --loglevel=INFO --pool=solo

```
