---
title: 完整创建一个项目
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-01.html
toc: true
---

### 企业中的项目类型

```python
1 商城类的
2 门户网站[企业站和门户站]
3 社交网络
4 咨询论坛
5 内部系统
6 个人博客
7 内容收费网站
```

<!-- more -->

### 企业项目开发流程

```python
# 公司项目来源
	-公司需要用
  -给客户定制
  -互联网项目

# 立项---》需求分析(产品经理，技术人员)---》产品原型--->前端后端
# 前端：根据原型图：ui+前端---》ui切图---》前端实现---》mock数据(自己造的假数据)
# 后端：确立项目架构，技术选型----》需求说明书+原型图---》开发接口，自己测试--》接口文档
# 前后端代码整合---》前后端联调---》集成测试(测试部，质量控制部)
# 上线

# 你们的工作流程：在公司项目管理平台(禅道)---》看自己任务----》确定需求(测试,问领导,问同事)---》写代码--》提交到git仓库---》管理平台把需求设置成完成


# 软件开发模式
  -瀑布模式：早期被广泛采用的软件开发模型---》bbs项目
  -敏捷开发：不停的开发--》测试---》上线转圈
    -scrum---》Sprint周期(小功能从开发到完成的时间)---》1周

 -不做整体数据库的设计---》写到哪个版块，再去设计相关表

# 路飞项目需求
  -首页轮播图
  -登陆注册
    -多方式登陆，手机号登陆
    -手机号注册
  -课程列表
    -过滤，排序
  -课程详情
    -视频播放，课程介绍
  -下单--》支付
   -支付宝支付
  -回调修改订单

  -上线
```

![开发流程](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/007S8ZIlgy1ggplfqw1vlj30o10oldij.jpg)

![img](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/007S8ZIlgy1ggplfvihpqj30z50oun17.jpg)

### `pip`源设置

```python
pip install package # 默认走国外
pip install package  -i https://pypi.douban.com/simple  # 为豆瓣源下载

# windowns环境
1、文件管理器文件路径地址栏敲：%APPDATA% 回车，快速进入 C:\Users\电脑用户\AppData\Roaming 文件夹中
2、新建 pip 文件夹并在文件夹中新建 pip.ini 配置文件
3、新增 pip.ini 配置文件内容
  [list]
  format=columns
  [global]
  timeout = 6000
  index-url = http://pypi.douban.com/simple
  trusted-host = pypi.douban.com
```

![image-20220418173532416](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220418173532416.png)

```shell
# mac/linux环境
# 创建隐藏目录
  mkdir ~/.pip

# 配置文件
  cat > ~/.pip/pip.conf <<EOF
  [global]
  timeout = 6000
  index-url = https://pypi.douban.com/simple/
  [install]
  use-mirrors = true
  mirrors = https://pypi.douban.com/simple/
  trusted-host = pypi.douban.com
  EOF
```

### 虚拟环境的设置

#### `pycharm`设置虚拟环境

```python
# 为什么会出现虚拟环境？
  -假设有个项目django 1.11.8
  -又有个项目django 2.2.2
  -由于在系统的解释器上只能装一个django，导致同时只能跑一个项目
  -每个项目使用自己的一个解释器---》虚拟环境--》通过系统解释器创造出一个解释器环境，他俩相互不干扰
  -系统有解释器，第一个项目有个虚拟环境 django1.11.8 ,第二个项目有个虚拟环境 django 2.2.2
  -以后变成，一个项目一个解释器
# 其他语言相关方案
  -vue 项目路径下--》node moduls---》这个项目依赖的环境--》删除---》npm install
  -go：go mod解决多版本共存问题
  -java：maven解决多版本jar包问题
  -python：虚拟环境

# python中虚拟环境解决方案有好几个(virtualenv,pipenv。。。。)
# 如果不使用virtualenv，可以直接在pycharm中做
```

![image-20220418173956576](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220418173956576.png)

#### 命令行设置虚拟环境

##### `Windows`平台

```python
# centos没有图形化界面的话--->没法装pycharm---》没法点点点创建，只能使用命令

# 虚拟环境命令的配置方案
# win环境
## 第一步：安装
pip3 install virtualenv   # 虚拟环境模块--》创建虚拟环境麻烦
pip3 install virtualenvwrapper-win # 虚拟环境辅助模块---》更快捷方便的操作和管理虚拟环境
# 安装完，在script文件夹下就会有virtualenv.exe和virtualenvwrapper.bat批处理文件
# 而你的script文件夹又在环境变量里--》这俩命令可以在任意路径下执行
## 第二步：配置环境
# 控制面板 => 系统和安全 => 系统 => 高级系统设置 => 环境变量 => 系统变量 => 点击新建 => 填入变量名与值
变量名：WORKON_HOME  变量值：自定义存放虚拟环境的绝对路径
eg: WORKON_HOME: D:\Virtualenvs

# 同步配置信息：
# 去向Python3的安装目录 => Scripts文件夹 => virtualenvwrapper.bat => 双击
```

##### `Mac/Linux`平台

```python
# 相当于创建一个新的python环境

## Linux使用virtualenv创建虚拟环境
# 安装扩展
pip3 install virtualenv -i https://pypi.douban.com/simple
pip3 install virtualenvwrapper -i https://pypi.douban.com/simple

# pip使用
pip3 list         # 列出所有的扩展
pip3 freeze       # 查看所有的扩展及版本，便于导出
pip3 show 扩展名	 # 查看扩展详细信息

# 查找路径
which virtualenvwrapper.sh
which python3
which virtualenv

'''
先找到virtualenvwrapper的工作文件 virtualenvwrapper.sh该文件可以刷新自定义配置，但需要找到它
MacOS可能存在的位置 /Library/Frameworks/Python.framework/Versions/版本号文件夹/bin
Linux可能所在的位置 /usr/local/bin  |  ~/.local/bin  |  /usr/bin
建议不管virtualenvwrapper.sh在哪个目录，保证在 /usr/local/bin 目录下有一份
如果不在 /usr/local/bin 目录，如在 ~/.local/bin 目录，则复制一份到 /usr/local/bin 目录
	-- sudo cp -rf ~/.local/bin/virtualenvwrapper.sh /usr/local/bin
'''

# 创建工作目录
mkdir ~/virtualenvs

# 编辑全局环境，当前用户的
vi ~/.bashrc  # linux平台
vi ~/.zshrc   # mac平台 可以看当前的shell是哪种的 echo $SHELL

# 添加的内容
# linux 平台
  export WORKON_HOME=/home/lzj/virtualenvs
  export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
  export VIRTUALENVWRAPPER_VIRTUALENV=/home/lzj/.local/bin/virtualenv
  source /home/lzj/.local/bin/virtualenvwrapper.sh

# mac平台
  export WORKON_HOME=/Users/zhijinliu/virtualenvs
  export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3
  export VIRTUALENVWRAPPER_VIRTUALENV=/Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenv
  source /Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh

# 生效
source ~/.bashrc  # linux
source ~/.zshrc   # mac

# 使用
mkvirtualenv test-env      # 创建test-env 的虚拟环境,默认进入到虚拟环境中
mkvirtualenv --python==    # 指定python版本路径创建虚拟环境
lsvirtualenv               # 查看所有的虚拟环境
cdvirtualenv               # 进入虚拟环境的目录，需要进入到虚拟环境
workon test-env            # 进入test-env虚拟环境中
deactivate                 # 退出当前虚拟环境
rmvirtualenv               # 删除
```

### 创建`luffy`项目

```shell
# 创建虚拟环境
mkvirtualenv luffy
workon luffy
pip install django==2.2.2
pip install djangorestframework
```

`Pycharm`创建项目

![image-20220418175407440](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220418175407440.png)

![image-20220418175437982](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220418175437982.png)

![image-20220418175527379](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220418175527379.png)

### 调整目录结构

#### 配置调整

```python
# 删除 templates

# setting调整
1.项目根目录下创建配置目录 luffy_api/setting
2.将 settings.py 移动到上面的配置目录中并重命名为 dev.py 作为开发中的配置文件
  'DIRS': [BASE_DIR / 'templates'], # 该行可以注释掉了
3.再创建 pro.py 到该目录 作为线上的配置文件(可以先复制dev.py中的内容到该文件内)
4.调整启动文件
  此时无法启动项目的，需要调整manage.py的内容
  os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffy_api.settings')改为
  os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffy_api.setting.dev')
```

#### `app`应用调整

```python
# 调整app路径 为了整洁
# python manage.py startapp user
# 创建 app 此时会在项目根目录创建 app 需要调整
# 调整创建 app 时，放在 luffy_api/luffy_api/apps/ 目录下
cd luffy_api/apps/
python ../../manage.py startapp user  # 此时就会在 apps 目录下创建了一个 user 应用

'''
使用时会报错，提示 No module named 'user'
原来直接写app名字不报错，原因是app就在项目根路径下(模块的查找)
由于项目的根路径在环境变量中，app就在根路径下，它能直接找到
现在的问题是apps路径不在环境变量中，它就找不到
'''

# 解决
  方式一
    注册时使用相对路径   'luffy_api.apps.user',

  方式二
    把apps的路径加入到环境变量中，要在项目启动时加，加到启动入口配置文件
    pro.py中添加两行代码
    import sys
    sys.path.append(os.path.join(BASE_DIR, 'apps'))

    此时就可以在 INSTALLED_APPS 直接写 app 名称
```

#### 其他调整

```python
# 上线时会使用uwsgi 运行wsgi.py 将配置改为上线的配置
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffy_api.setting.pro')

# 国际化
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_L10N = True
USE_TZ = False

# 把项目内的 luffy_api 也就是 BASE_DIR 也加入到环境变量
# 在 luffy_api/setting/dev.py和pro.py 文件中添加下面内容
  sys.path.append(BASE_DIR)

此时导入模块的时候，只要从环境变量的路径开始导就可以了
从小luffy_api路径开始导入即可
但是pycharm爆红，但是没有错，点右键，把该路径(在环境变量中的)，做成Sources root即可
以后再从这个路径下导包，不会报错了

# 注意：以后导入包
  尽量用最短路径导入，如果从长路径导入 ---> 路径经过的py文件都会去执行 ---> 可能会导致循环导入的问题
  推荐用相对导入
  # from apps.user import models
  from . import models

  py文件中有相对导入，这个py文件不能作为脚本运行
  django项目中，由于没有右键运行的脚本，所以都可以用相对导入
```

![image-20220418184932481](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220418184932481.png)

#### 目录结构

```python
luffy_api           # 项目名称
├── luffy_api       # 项目主应用 开发时的代码保存 - 包
│   ├── apps        # 开发者的代码保存目录，以模块[子应用]为目录保存 - 包
│   ├── libs        # 第三方类库的保存目录[第三方组件、模块] - 包
│   ├── setting     # 配置目录 - 包
│   │   ├── dev.py  # 项目开发时的本地配置
│   │   └── pro.py  # 项目上线时的运行配置
│   ├── urls.py     # 总路由
│   ├── utils       # 多个模块[子应用]的公共函数类库[自己开发的组件]
│   └── wsgi.py     # uwsgi运行的配置文件 上线时用
├── manage.py       # 脚本文件
└── scripts         # 保存项目运营时的脚本文件 - 文件夹
├── logs            # 项目运行时/开发时日志目录 - 包
```

### `luffy`后台配置

#### 日志配置

```python
## 配置
# 以下内容添加在  luffy_api/setting/dev.py 中
# 日志相关
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
        },
    },
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            # 实际开发建议使用WARNING
            'level': 'DEBUG',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {
            # 实际开发建议使用ERROR
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            # 日志位置,日志文件名,日志保存目录必须手动创建，注：这里的文件路径要注意BASE_DIR代表的是小luffyapi
            'filename': os.path.join(os.path.dirname(BASE_DIR), "logs", "luffy.log"),
            # 日志文件的最大值,这里我们设置300M
            'maxBytes': 300 * 1024 * 1024,
            # 日志文件的数量,设置最大日志数量为10
            'backupCount': 10,
            # 日志格式:详细格式
            'formatter': 'verbose',
            # 文件内容编码
            'encoding': 'utf-8'
        },
    },
    # 日志对象
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'propagate': True,  # 是否让日志信息继续冒泡给其他的日志处理系统
        },
    }
}

## 使用
  # 在utils目录下创建luffy_logging.py 内容如下
    import logging
    # 创造一个logger对象，使用的是配置文件中的django这个
    logger = logging.getLogger('django')

  # 需要打印日志的地方直接导入使用 打印到控制台和记录到文件中
    from utils.luffy_logging import logger

    logger.info("日志执行了")
```

#### 处理全局异常

```python
## utils/exception.py 内容如下
# 全局异常捕获
from rest_framework.views import exception_handler  # 默认没有配置，出了异常会走它
from rest_framework.response import Response
from utils.luffy_logging import logger


def common_exception_handler(exc, context):
    res = exception_handler(exc, context)
    if res:
        res = Response(data={'code': 998, 'msg': res.data.get('detail', '服务器异常，请联系系统管理员')})
    else:
        res = Response(data={'code': 999, 'msg': str(exc)})
    request = context.get('request')
    view = context.get('view')
    logger.error('错误原因：%s,错误视图类：%s,请求地址：%s,请求方式：%s' % (str(exc), str(view), request.path, request.method))
    return res


# setting/dev.py 添加如下内容
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'utils.exception.common_exception_handler'  # 再出异常，会执行这个函数
}
```

#### 二次封装`Response`

```python
# 新建py文件 luffy_api/utils/APIResponse.py
from rest_framework.response import Response

class APIResponse(Response):
    def __init__(self, code=100, msg='successfully', data=None, status=None,
                 template_name=None, headers=None,
                 exception=False, content_type=None, **kwargs):
        dic = {'status': code, 'msg': msg}
        if data:  # 如果data有值，说明要往里面放东西
            dic['data'] = data
        if kwargs:  # 自定义传的参数会添加到字典里
            dic.update(kwargs)

        super().__init__(data=dic, status=status,
                         template_name=template_name, headers=headers,
                         exception=exception, content_type=content_type)


# 直接导入使用即可
from utils.APIResponse import APIResponse
...
  return APIResponse(msg='成功了',data={'ok'})

# 结果
{
    "status": 100,
    "msg": "成功了",
    "data": [
        "ok"
    ]
}
```

#### 后台数据库配置

##### 数据库的创建

```python
# 使用 mysql 需要先创建数据库 库名 luffy
create database luffy default charset=utf8;

# 创建一个数据库普通用户管理 luffy数据库
##  查看用户
    # 5.7之前版本
    select user,host,password from mysql.user;
    # 5.7往后的版本
    select user,host,authentication_string from mysql.user;

## 创建用户并授权
    # 8.0版本需要先创建用户再授权
    create user luffy@'%' identified by '123456';
    grant all privileges on luffy.* to luffy@'%';

## 刷新权限
flush privileges;



# 关于pymysql和mysqlclient的选择
## 使用 pymysql
  import pymysql
  pymysql.install_as_MySQLdb()
  # 这两句话，只要执行即可，放在那里都行，只要django执行，所有py文件中顶格写的代码都会执行
  # 作用是？猴子补丁，动态替换 ---> python一切皆对象，可以动态替换对象
  # 如果该源码，后期只要使用django，都要改它的源码 讲deconde



### 关于pymysql和mysqlclient的选择
  # 这两句话，只要执行即可，放在那里都行 ---> 只要django执行，所有py文件中顶格写的代码都会执行
  # 作用是？猴子补丁，动态替换 ---> python一切皆对象，可以动态替换对象
  # 如果该源码，后期只要使用django，都要改它的源码
  # 所以咱们换另一个操作mysql的模块，mysqlclient ---> MysqlDB的3版本 ---> 有可能装不上 ---> win上看人品，实在装不上用whl文件装 linux上有不同解决方案
  # import pymysql
  # pymysql.install_as_MySQLdb()

#### 使用mysqlclient不需要写两句话，不用改源码
```

##### 配置连接

```python
# dev.py 配置文件配置 mysql 连接
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'luffy',
        'USER': 'luffy',
        'PASSWORD': '123456',
        'HOST': 'localhost',
        'PORT': 3306,
    }
}

# 选择不同的第三方模块连接
```

###### `pymysql`方式

```python
# 下载
pip install pymysql

# 使用
# 这两句话，只要执行即可，放在那里都行，只要django执行，所有py文件中顶格写的代码都会执行
import pymysql
pymysql.install_as_MySQLdb()
'''
作用是？猴子补丁，动态替换 ---> python一切皆对象，可以动态替换对象
如果该源码，后期只要使用django，都要改它的源码
出现报错需要将报错文件的   decode  改为  encode
'''
```

![image-20220419181539251](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220419181539251.png)

###### `mysqlclient`

```python
# 安装
pip install mysqlclient

# 直接使用，不用修改源码 和 添加内容


# 可能会出现报错
'''
Mac环境下的一些报错及解决
1.需要下载安装 mysql 之后才可以安装mysqlclient
2.启动会项目可能报 NameError: name ‘_mysql‘ is not defined 的错误
    解决: 环境变量中添加如下内容 .zshrc 或者 .bashrc

    export PATH=$PATH:/usr/local/mysql/bin
    export LD_LIBRARY_PATH="/usr/local/mysql/lib:${LD_LIBRARY_PATH}"
    export DYLD_LIBRARY_PATH=/usr/local/mysql/lib/
'''


# CentOS有Python、Mysql的开发工具包，安装后使用pip安装mysqlclient即可
  yum install mysql-devel
  yum install python-devel
  # yum install python36-devel
  pip install mysqlclient

# Ubuntu 也是安装一些Mysql的依赖或开发库
apt-get install libmysql-dev
apt-get install libmysqlclient-dev
apt-get install python-dev # python3 要装 python3-dev
pip install mysqlclient
#此时如果遇到如下错误
'''
Collecting mysqlclient
  Using cached  https://files.pythonhosted.org/packages/f4/f1/3bb6f64ca7a429729413e6556b7ba5976df06019a5245a43d36032f1061e/mysqlclient-1.4.2.post1.tar.gz
   Complete output from command python setup.py egg_info:
   Traceback (most recent call last):
    File "<string>", line 1, in <module>
   ModuleNotFoundError: No module named 'setuptools'

   \----------------------------------------
 Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-p8wpb1kf/mysqlclient/
'''
# 需要安装一下setuptools包
# pip install setuptools==33.1.1
```

#### `user` 模块和 `user`表设计

##### 创建

```python
# 用户板块 做成app
  cd luffy_api/apps/
  python ../../manage.py startapp

# 创建用户表，基于auth表的user表扩写
'''
注意：在写好这个之前，不要先迁移数据，如果迁移了数据库，就不行了
如果你已经迁移了，删除数据库，删除所有的migrations文件，包含你自己的app，和auth和admin这两个app
'''

# luffy_api/apps/user/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    mobile = models.CharField(max_length=11, unique=True)  # 唯一，长度11
    # 需要pillow包的支持 ImageField继承自FileField
    icon = models.ImageField(upload_to='icon', default='icon/default.png')

    class Meta:
        db_table = 'luffy_user'
        verbose_name = '用户表'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.username

'''
db_table 数据库中别名
verbose_name指定在admin管理界面中显示中文
verbose_name表示单数形式的显示
verbose_name_plural表示复数形式的显示
中文的单数和复数一般不作区别
'''
```

##### 配置

```python
# luffy_api/setting/dev.py 配置文件中注册应用
INSTALLED_APPS = [
    # ...
    'user',
]

# luffy_api/setting/dev.py 配置文件中配置自定义表
  AUTH_USER_MODEL = 'user.User'

# luffy_api/setting/dev.py 配置文件中配置media
  MEDIA_URL = '/media/'
	MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# 创建media文件夹在 项目名称/luffy_api 下 和seting同级

# 总路由配置 luffy_api/urls.py 中添加
from django.conf import settings
from django.views.static import serve

urlpatterns = [
		# ...
    path('media/<path:path>', serve, {'document_root': settings.MEDIA_ROOT})
]

# 安装pillow
  pip instasll pillow

# 迁移数据
  python manage.py makemigrations
  python manage.py migrate
```

#### `vue`创建前端

##### 创建项目

```python
# 创建vue项目
vue create luffycity

# pycharm打开项目 配置运行
```

##### 文件配置

```python
# 删除多余的文件
src/components/HelloWorld.vue
src/views/AboutView.vue

# 删除多余的内容
  ## src/views/HomeView.vue
    <template>
      <div class="home">
          <h1>首页</h1>
      </div>
    </template>

    <script>

    export default {
      name: 'HomeView',
      components: {
      }
    }
    </script>

  ## src/App.vue
    <template>
      <div id="app">
        <router-view/>
      </div>
 	  </template>

  ## src/router/index.js 该内容下删除 about 相关的内容
    const routes = [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    ]
```

##### 安装插件

```python
# elementui
  ## 安装
  npm install element-ui -S

  ## main.js
    import ElementUI from 'element-ui';
    import 'element-ui/lib/theme-chalk/index.css';
    Vue.use(ElementUI);


# bootstrap jQuery
  ## 安装
  npm install bootstrap@3 -S
  npm install jquery -S

  ## 配置
  ### main.js
    import 'bootstrap'
    import 'bootstrap/dist/css/bootstrap.min.css'

	### vue.congig.js
      const {defineConfig} = require('@vue/cli-service')
      const webpack = require("webpack");
      module.exports = defineConfig({
          transpileDependencies: true,
          configureWebpack: {
              plugins: [
                  new webpack.ProvidePlugin({
                      $: "jquery",
                      jQuery: "jquery",
                      "window.jQuery": "jquery",
                      "window.$": "jquery",
                      Popper: ["popper.js", "default"]
                  })
              ]
          },
        })

# axios
  ## 安装
  npm install axios  -S

  ## main.js
  import axios from 'axios'
  Vue.prototype.$axios = axios;
```

##### 全局样式

```python
# src/assets/css/global.css
/* 声明全局样式和项目的初始化样式 */
body, h1, h2, h3, h4, h5, h6, p, table, tr, td, ul, li, a, form, input, select, option, textarea {
    margin: 0;
    padding: 0;
    font-size: 15px;
}

a {
    text-decoration: none;
    color: #333;
}

ul {
    list-style: none;
}

table {
    border-collapse: collapse; /* 合并边框 */
}


# main.js 添加
// 把自己定义的global.css 引入
import '@/assets/css/global.css'
```

##### 全局配置

```python
# 创建 assets/js/settings.js 文件
// 定义全局后端 url ，取值方便
export default {
    base_url: "http://127.0.0.1:8000"
}

# main.js 添加 后面需要 url 使用 this.$settings.url 即可取值
// 导入自定义配置
import settings from './assets/js/settings'
Vue.prototype.$settings = settings;
```
