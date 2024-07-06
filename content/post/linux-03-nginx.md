---
title: Nginx安装配置使用
lastmod: 2021-04-21T16:43:23+08:00
date: 2021-04-21T11:52:03+08:00
tags:
  - Linux
  - Nginx
categories:
  - Linux
url: post/linux-03.html
toc: true
---

## 安装 nginx

```bash
#全一些的依赖
yum install -y libxml2 libxml2-devel openssl \
openssl-devel bzip2 bzip2-devel libcurl \
libcurl-devel libjpeg libjpeg-devel \
libpng libpng-devel freetype freetype-devel \
gmp gmp-devel libmcrypt libmcrypt-devel \
readline readline-devel libxslt libxslt-devel  \
libicu-devel  openldap  openldap-devel \
make zlib zlib-devel gcc-c++ libtool \
pcre pcre-devel  cmake gcc  ncurses ncurses-devel \
bison bison-devel libgcrypt perl wget
```

<!-- more -->

```bash
#最小依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

#创建www用户管理nginx并设置为不可登录
useradd www
usermod -s nologin www

#www用户一步到位#
useradd -s nologin www

#创建nginx工作目录
mkdir -p /usr/local/nginx

#创建存放nginx下载的目录
mkdir -p /data/tools && cd /data/tools

#版本选择
#http://nginx.org/en/download.html  #下载地址
#Stable version		 稳定版本
#Mainline version 	 新版本（不推荐，无特殊要求稳定版即可）

#下载安装包
wget http://nginx.org/download/nginx-1.20.1.tar.gz

#解压nginx
tar xf nginx-1.20.1.tar.gz

#隐藏版本，看需要是否隐藏nginx版本
sed -i 's/1.20.1//g' nginx-1.20.1/src/core/nginx.h

#编译参数开始编译
cd nginx-1.20.1
./configure --user=www --group=www \
--prefix=/usr/local/nginx  \
--with-http_stub_status_module  \
--with-http_ssl_module --with-pcre

#nginx -V 可查看安装的nginx的编译参数

#编译
make -j `cat /proc/cpuinfo |grep processor |wc -l` && make install

#添加环境变量
echo 'export PATH=/usr/local/nginx/sbin:$PATH' >> /etc/profile
source /etc/profile

#给nginx工作目录www权限
chown -R www.www /usr/local/nginx
```

## nginx 命令

```bash
#启动
nginx

#平滑重载
nginx -s reload

#停止
nginx -s stop

#查找nginx进程，杀死PID
ps -ef |grep nginx
kill nginxPID

#检查语法，修改配置文件后必做
nginx -t

#查看编译参数及版本
nginx -V
```

## nginx 配置文件

```bash
cd /usr/local/nginx/conf
cp nginx.conf{,.bak}

#得到最简单的配置文件
egrep -v "^$|#" nginx.conf.bak > nginx.conf
```

最简配置

```bash
cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

相对优化后的主配置文件

```bash
#相对优化后的配置文件
user  www www;					#用户
worker_processes 2;				#进程数，可根据自身配置调配  可选 auto
events
    {
        use epoll;
        worker_connections 8192;
        multi_accept on;
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
        include vhost/*.conf;#conf/vhost下存放虚拟主机配置文件，将每个域名配置文件写到此目录下以 .conf结尾即可，实现多域名
}
```

站点配置文件

```bash
cd /usr/local/nginx/conf/vhost
cat www.conf
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;  #配置访问日志文件位置

    location / {
        root   /usr/local/nginx/html;  #站点目录位置
        index  index.html index.htm;
    }
}
```

## 动态添加模块

**扩展模块 nginx-rtmp-module 为例**

```bash
#下载rtmp模块
cd /data/tools/
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip

#解压并查看模块
unzip master.zip
ls nginx-rtmp-module-master

#查看之前的编译参数
nginx -V

#进入到之前编译的nginx的目录
cd /data/tools/nginx-1.20.1/

#重新编译增加一个模块
./configure  --user=www --group=www \
--prefix=/usr/local/nginx \
--with-http_stub_status_module \
--with-http_ssl_module --with-pcre \
--add-module=/data/tools/nginx-rtmp-module-master

#不能make install 否则出问题
make

#备份之前的nginx软件
cp /usr/local/nginx/sbin/nginx{,.bak}

#替换旧版本nginx软件
cp ./objs/nginx{,.bak}
mv ./objs/nginx /usr/local/nginx/sbin/

#重新检查当前是否添加了rtmp模块
nginx -V
nginx -s reload
```

## 模块的使用

#### 下载服务器

编辑做下载的配置文件

```bash
vim /usr/local/nginx/conf/vhost/download.conf
server {
        listen       8000;  #端口
        server_name  localhost;
        location / {
            root /download;  #下载目录
            autoindex on;  #开启索引功能
            autoindex_exact_size off; #关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
            autoindex_localtime on; #显示本机时间而非 GMT 时间
        }
}

```

```bash
#创建存放文件的下载目录
 mkdir /download

 #改变属主属组
 chown -R www.www /download/

 #检查nginx语法配置
 nginx -t

 #平滑重启nginx
 nginx -s reload
```

测试

```bash
echo 1 > /download/test.txt
```

![vedTHQ](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/vedTHQ.png)

#### https 证书配置
