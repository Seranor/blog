---
title: Docker上线项目
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-10.html
toc: true
---

### 前端代码

```
# src/assets/js/settings.js 改为自己的公网 IP
export default {base_url:'http://42.194.173.230:8080/api/v1/'}

# 编译
npm run build

# 将 dist 文件夹打包为 dist.zip 上传到服务器中
```

### 后端代码

#### `luffy_api/luffy_api/setting/pro.py`

```shell
1. DEBUG = False
2. ALLOWED_HOSTS = ['*']
3. MySQL连接地址和密码
	 这里写的是 luffy_mysql 是在docker中的 --link 参数 指定docker容器名称可以在 /etc/hosts 文件中做映射
4. Redis连接地址和密码
   "redis://:admin@luffy_redis:6379",
   admin 是设置的密码
   luffy_redis 也是docker的 --link 参数如上
```

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
        'HOST': 'luffy_mysql',
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
        "LOCATION": "redis://:admin@luffy_redis:6379",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",  # 这句指的是django中的缓存也缓存到redis中了
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}  # 连接池的大小
            #"PASSWORD": "admin",
        }
    }
}
```

#### `luffy_api/luffy_api/setting/user_settings.py`

```python
# 用户自己的配置，单独放到另一个py文件中
BANNER_COUNT = 4

# 用户自己别的配置
# 上线后必须换成公网地址
# 后台基URL
BASE_URL = 'http://42.194.173.230:8080'
# 前台基URL
LUFFY_URL = 'http://42.194.173.230'
# 支付宝同步异步回调接口配置
# 后台post异步回调接口
NOTIFY_URL = BASE_URL + "/api/v1/order/success/"
# 前台同步回调接口，没有 / 结尾
RETURN_URL = LUFFY_URL + "/pay/success"
```

### 服务器上操作

#### 安装 docker

```shell
# 安装
yum install yum-utils device-mapper-persistent-data lvm2 -y
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y

# 启动
systemctl start docker
systemctl enable docker

# 镜像加速、更改Docker根目录
mkdir -p /data/docker   # 挂载到新的硬盘上
cat >/etc/docker/daemon.json<<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {"max-size": "100m"},
  "registry-mirrors" : ["https://ot2k4d59.mirror.aliyuncs.com/"],
  "graph": "/data/docker"
}
EOF

# 重启docker
systemctl daemon-reload
systemctl restart docker
docker info
```

#### 相关配置文件

##### 后端配置

```shell
# 创建操作目录
mkdir ~/docker-luffy
cd ~/docker-luffy

# 克隆代码 检查是否如上修改了相应的配置 也可以自定义配置
yum install git -y
git clone https://gitee.com/liuzhijin1/luffy_api.git


# 准备 uwsgi文件
cat  > ~/docker-luffy/luffy_api/luffyapi.xml <<EOF
<uwsgi>
   <socket>0.0.0.0:8888</socket>
   <chdir>/soft/</chdir>
   <module>luffy_api.wsgi</module>
   <processes>4</processes>
   <daemonize>uwsgi.log</daemonize>
</uwsgi>
EOF


# 创建日志文件
touch ~/docker-luffy/luffy_api/uwsgi.log


# 准备 Dockerfile 文件
cat > ~/docker-luffy/Dockerfile << EOF
FROM python:3.8

WORKDIR /soft
COPY luffy_api /soft
RUN  pip install --upgrade pip -i https://pypi.douban.com/simple/ && \
     pip install uwsgi  -i https://pypi.douban.com/simple/ && \
     pip install -r requirements.txt -i https://pypi.douban.com/simple/

EXPOSE 8888
CMD uwsgi -x /soft/luffyapi.xml && tail -f /soft/uwsgi.log
EOF
```

##### 制作镜像

```shell
# 制作镜像前一定要检查好 django 的相关配置
cd ~/docker-luffy
docker build -t luffy:v1 .

docker images
```

##### `nginx相关配置文件`

```shell
# nginx的配置文件
cat  ~/docker-luffy/nginx.conf
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
        server {
            listen 8080;
            server_name  0.0.0.0;
            charset utf-8;
            location / {
               include uwsgi_params;
               uwsgi_pass luffy_api:8888;
            }
            location /static {
               alias /usr/share/nginx/html/static;
            }

            location /media {
               alias /usr/share/nginx/html/media;
            }
         }
        server {
            listen 80;
            server_name  0.0.0.0;
            charset utf-8;
            location / {
                root /usr/share/nginx/html/dist;
                index index.html;
                try_files $uri $uri/ /index.html; # 解决单页面应用刷新404问题
            }
        }
}

# 静态文件准备
mkdir ~/docker-luffy/webstatic -p

# 将前端代码放入
unzip dist.zip -d ~/docker-luffy/webstatic
cp -rp ~/docker-luffy/luffy_api/luffy_api/media ~/docker-luffy/webstatic

# 还有一个django 后台的页面导出放到下面文件夹内
mkdir ~/docker-luffy/webstatic/static

#### 在其他位置打包好再拿过来也可以用
    # 修改配置
    vim luffy_api/luffy_api/setting/pro.py
    STATIC_URL = '/static/'
    STATIC_ROOT = '/web/luffy/luffy_api/luffy_api/static'
    STATICFILES_DIRS = (os.path.join(BASE_DIR, "static"),)

    # 后台静态文件迁移 自动迁移到 上面创建的静态文件目录
    python luffy_api/manage_pro.py collectstatic
```

#### 启动项目

```shell
# 创建持久化文件夹
mkdir /data/luffy -p
cd ~/docker-luffy

# 启动顺序一般都是从后往前的
# 启动 mysql
docker run -itd --name luffy_mysql \
-p 3306:3306 \
-v /data/luffy/mysql_data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=Mysql123456? \
-e MYSQL_DATABASE=luffy \
-e MYSQL_USER=luffy \
-e MYSQL_PASSWORD=Luffy123? \
mysql:5.7  \
--character-set-server=utf8mb4

# 启动 redis
docker run -itd --name luffy_redis \
-p 6379:6379 \
-v /data/luffy/redis_data:/data \
redis:6.0  \
--requirepass "admin"

# 启动luffy
docker run -itd --name luffy_api \
-p 8888:8888 \
--link luffy_redis \
--link luffy_mysql \
luffy:v1

# 启动 nginx
docker run -itd --name luffy_web \
--link luffy_api \
-p 80:80 \
-p 8080:8080 \
-v /root/docker-luffy/nginx.conf:/etc/nginx/nginx.conf \
-v /root/docker-luffy/webstatic:/usr/share/nginx/html \
nginx:1.20

docker run -itd --name luffy_web \
--link luffy_api \
-p 80:80 \
-p 8080:8080 \
-v $PWD/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/webstatic:/usr/share/nginx/html \
nginx:1.20

# 运行完成后进入容器执行数据迁移命令
docker exec -it luffy_api /bin/sh
python /soft/manage_pro.py migrate
```
