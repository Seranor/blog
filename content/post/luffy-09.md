---
title: 项目上线
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-09.html
toc: true
---

### 上线架构

<!-- more -->

![image-20220505183254286](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220505183254286.png)

### 安装 mysql

```shell
# 下载 mysql5.7
wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

# 安装 mysql5.7
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum install mysql-community-server --nogpgcheck

# 启动 mysql5.7 并查看启动状态
systemctl start mysqld.service  # 启动mysql服务
systemctl enable mysqld.service # 开机自启动
systemctl status mysqld.service # 查看服务

# 查看默认密码并登录
grep "password" /var/log/mysqld.log
mysql -uroot -p

# 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Mysql12345?';
```

### 安装 redis

```shell
# 下载redis-5.0.5
wget http://download.redis.io/releases/redis-5.0.5.tar.gz

# 解压安装包
tar -xf redis-5.0.5.tar.gz

# 进入目标文件
cd redis-5.0.5

# 编译环境  src路径下就有可执行文件 redis-server redis-cli 等
make -j `cat /proc/cpuinfo |grep processor |wc -l`

# 复制环境到指定路径完成安装
cp -rp ~/redis-5.0.5 /usr/local/redis

# 配置redis可以后台启动：修改下方内容
vim /usr/local/redis/redis.conf
daemonize yes

# 建立软连接
ln -s /usr/local/redis/src/redis-server /usr/bin/redis-server
ln -s /usr/local/redis/src/redis-cli /usr/bin/redis-cli

# 后台运行redis
cd /usr/local/redis
redis-server ./redis.conf

# 查看服务是否启动
ps aux |grep redis

# 测试redis环境
redis-cli

# 关闭redis服务
pkill -f redis -9
```

### 安装 python3.8

```shell
# 下载依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel python-devel mariadb-devel gdbm-devel python-setuptools python-devel -y

# 下载源码安装
wget https://registry.npmmirror.com/-/binary/python/3.8.6/Python-3.8.6.tgz
tar xf Python-3.8.6.tgz
cd Python-3.8.6/
./configure --prefix=/usr/local/python38
make && make install
ln -s /usr/local/python38/bin/python3 /usr/bin/python3.8
ln -s /usr/local/python38/bin/pip3 /usr/bin/pip3.8

# 安装uwsgi
pip3.8 install uwsgi
ln -s /usr/local/python38/bin/uwsgi /usr/bin/uwsgi

# 配置虚拟环境
python3.8 -m pip install --upgrade pip
python3.8 -m pip install --upgrade setuptools
pip3.8 install virtualenv
pip3.8 install virtualenvwrapper

# 建立软连接
ln -s /usr/local/python38/bin/virtualenv /usr/bin/virtualenv

# 配置环境变量
cat >> ~/.bashrc <<EOF
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3.8
source /usr/local/python38/bin/virtualenvwrapper.sh
EOF
# 虚拟环境默认根目录：~/.virtualenvs

# 创建虚拟环境
mkvirtualenv luffy
```

### 安装 nginx

```shell
yum install gcc zlib zlib-devel pcre-devel openssl openssl-devel -y

wget http://nginx.org/download/nginx-1.20.1.tar.gz

tar xf nginx-1.20.1.tar.gz

cd nginx-1.20.1/

./configure --prefix=/usr/local/nginx  \
--with-http_stub_status_module  \
--with-http_ssl_module --with-pcre

make -j `cat /proc/cpuinfo |grep processor |wc -l` && make install

echo 'export PATH=/usr/local/nginx/sbin:$PATH' >> /etc/profile
source /etc/profile
```

### 上线前端

```shell
# 修改后端地址 luffycity/src/assets/js/settings.js  改为公网 IP
export default {base_url:'http://119.91.117.23:8080/api/v1/'}

# 将 vue 项目 编译为 html css js 等纯静态文件 项目目录下执行
npm run build

# 将 dist 文件夹压缩上传到服务器上 sftp 等工具
# 进入到上传所在的目录
mkdir /web/luffy -p
unzip dist.zip -d /web/luffy/

# 修改 nginx 配置文件
cd /usr/local/nginx/
# 修改配置文件，如下
```

```
nginx.conf
worker_processes  auto;
events {
    worker_connections  1024;
}
http
    {
        include       mime.types;
        default_type  application/octet-stream;
        charset UTF-8;
        server_names_hash_bucket_size 128;
        client_header_buffer_size 32k;
        large_client_header_buffers 4 32k;
        client_max_body_size 50m;
        #####################################
        sendfile   on;
        tcp_nopush on;
        keepalive_timeout 60;
        tcp_nodelay on;
        #####################################
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        fastcgi_buffer_size 64k;
        fastcgi_buffers 4 64k;
        fastcgi_busy_buffers_size 128k;
        fastcgi_temp_file_write_size 256k;
        #####################################
        gzip on;
        gzip_min_length  1k;
        gzip_buffers     4 16k;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_vary on;
        gzip_proxied   expired no-cache no-store private auth;
        server_tokens off;
        include vhost/*.conf;
}
```

```shell
# 创建目录
mkdir vhost
cat > vhost/web.conf <<EOF
server {
    listen 80;
    server_name  localhost;
    charset utf-8;
    location / {
        root /web/luffy/dist;
        index index.html;
        try_files $uri $uri/ /index.html; # 解决单页面应用刷新404问题
    }
}
EOF

# 检查
nginx -t

# 启动
nginx

# 检查是否成功启动
netstat -lntup
ps -ef |grep nginx

# 访问地址
```

### 上线后端

#### 上线前准备

##### `setting/pro.py`

将 luffy_api/setting/dev.py 拷贝过来修改

```python
import os
import sys
import datetime

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

sys.path.append(os.path.join(BASE_DIR, 'apps'))
sys.path.append(BASE_DIR)

SECRET_KEY = '-b4z=*^wa1rjc-5a+@+)f%t-%mppmx@vz%8zdps1$rr9r@y&$5'

DEBUG = False

ALLOWED_HOSTS = ['*']

INSTALLED_APPS = [
    'simpleui',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'user',
    'home',
    'course',
    'order',
    'django_filters',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'corsheaders.middleware.CorsMiddleware',
]

ROOT_URLCONF = 'luffy_api.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # 'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'luffy_api.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'luffy',
        'USER': 'luffy',
        'PASSWORD': 'Luffy123?',
        'HOST': 'localhost',
        'PORT': 3306,
    }
}

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_L10N = True
USE_TZ = False

STATIC_URL = '/static/'

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

REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (  # 默认响应渲染类
        'rest_framework.renderers.JSONRenderer',  # json渲染器
        # 'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览API渲染器
    ),
    'EXCEPTION_HANDLER': 'utils.exception.common_exception_handler'  # 再出异常，会执行这个函数
}

# 自定义User表
AUTH_USER_MODEL = 'user.User'

# 配置media
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

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

# 导入用户自定义配置
from .user_settings import *

JWT_AUTH = {
    # 过期时间1天
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7),
}

# redis 配置
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",  # 这句指的是django中的缓存也缓存到redis中了
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}  # 连接池的大小
            # "PASSWORD": "123",
        }
    }
}
```

##### `setting/user_settings.py`

```python
# 用户自己的配置，单独放到另一个py文件中
BANNER_COUNT = 4

# 用户自己别的配置
# 上线后必须换成公网地址
# 后台基URL
BASE_URL = 'http://119.91.117.23:8080'
# 前台基URL
LUFFY_URL = 'http://119.91.117.23'
# 支付宝同步异步回调接口配置
# 后台post异步回调接口
NOTIFY_URL = BASE_URL + "/api/v1/order/success/"
# 前台同步回调接口，没有 / 结尾
RETURN_URL = LUFFY_URL + "/pay/success"
```

##### `manage_pro.py`

将 `manage.py` 复制一份改为 `manage_pro.py`

```python
#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys


def main():
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffy_api.setting.pro')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()
```

##### 导出模块

```python
pip freeze > requirements.txt
```

#### 提交代码

```shell
git add .
git commit -m "发布版本"
git tag -a v1.5 -m "发布版本"
git push origin "master" --tags v1.5
```

#### 服务端上线

##### 拉取代码

```shell
cd /web/luffy
git clone https://gitee.com/liuzhijin1/luffy_api.git
```

##### 环境配置

```shell
# 进入虚拟环境
workon luffy

# 下载 mysqlclient 的依赖
yum install mysql-devel -y
yum install python-devel -y
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

# 安装所需模块
pip install -r requirements.txt
pip install uwsgi
```

##### `uwsgi`与`nginx`配置

```shell
# uwsgi
cat > /web/luffy/luffy_api/luffyapi.xml <<EOF
<uwsgi>
   <socket>127.0.0.1:8888</socket>
   <chdir>/web/luffy/luffy_api/</chdir>
   <module>luffy_api.wsgi</module>
   <processes>4</processes>
   <daemonize>uwsgi.log</daemonize>
</uwsgi>
EOF

# nginx 配置文件
cat > /usr/local/nginx/conf/vhost/luffapi.conf<< EOF
server {
    listen 8080;
    server_name  127.0.0.1;
    charset utf-8;
    location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:8888;
       uwsgi_param UWSGI_SCRIPT luffy_api.wsgi;
       uwsgi_param UWSGI_CHDIR /web/luffy/luffy_api/;
    }
}
EOF
```

##### 数据库创建用户

```shell
mysql -uroot -pMysql12345?

create database luffy default charset=utf8mb4;
grant all privileges on luffy.* to 'luffy'@'%' identified by 'Luffy123?';
grant all privileges on luffy.* to 'luffy'@'localhost' identified by 'Luffy123?';
flush privileges;

quit
```

##### 数据库迁移

```shell
# 只有库，没有表，迁移数据
# 必须在luffy环境下
# 数据库迁移
cd /web/luffy/luffy_api
python manage_pro.py migrate


# 导入测试数据
把原来本地的数据，导入到正式库中
navicat 中导出先 结构和数据  放到桌面
然后导入到线上库做测试用
无法导入可能是编码格式有问题，将sql文件中的编码格式修改一下即可
```

##### 启动 uwsgi

```shell
# 可能缺少的依赖
yum install libxml*

# 需要进入到虚拟环境中
workon luffy
uwsgi -x ./luffyapi.xml
nginx -s reload
```

##### 后台的静态样式收集和上线

```python
# 创建静态文件目录
mkdir /web/luffy/luffy_api/luffy_api/static -p

# 修改配置
vim /web/luffy/luffy_api/luffy_api/setting/pro.py
STATIC_URL = '/static/'
STATIC_ROOT = '/web/luffy/luffy_api/luffy_api/static'
STATICFILES_DIRS = (os.path.join(BASE_DIR, "static"),)

# 后台静态文件迁移 自动迁移到 上面创建的静态文件目录
python /home/project/luffy_api/manage_pro.py collectstatic

# 加入 nginx 的配置解析 后端是 8080端口的
cat  /usr/local/nginx/conf/vhost/luffapi.conf
server {
    listen 8080;
    server_name  127.0.0.1;
    charset utf-8;
    location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:8888;
       uwsgi_param UWSGI_SCRIPT luffy_api.wsgi;
       uwsgi_param UWSGI_CHDIR /web/luffy/luffy_api/;
    }
    location /static {
       alias /web/luffy/luffy_api/luffy_api/static;
    }

    location /media {
       alias /web/luffy/luffy_api/luffy_api/media;
    }
}
```
