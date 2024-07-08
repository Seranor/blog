---
title: Nginx代理
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Nginx
categories:
  - Linux
url: post/linux-20.html
toc: true
---

# Nginx 代理

代理一词往往并不陌生。可以理解为中介，比如在生活中我们处理法律问题、房产交易都会请专业人士代为处理。从网络角度讲，就是为事务参与双方提供连接通道的第三方网络服务器。在没有代理的模式的情况下，都是客户端直接请求的服务端，在实际的情况下，为了安全，客户端往往无法直接向服务端发起请求，就需要用到代理服务，来实现通信。

- nginx 是一款自由的、开源的、高性能的 HTTP 服务器和反向代理服务器。同时也是一个 IMAP、POP3、SMTP 代理服务器。nginx 可以作为一个 HTTP 服务器进行网站的发布处理，同时 nginx 可以作为反向代理进行负载均衡的实现。
<!-- more -->

## 代理的模式

- ### 1、正向代理（VPN）

  找完代理之后，还需要找服务器

  用于内部上网，**客户端 <----> 代理 ----> 服务端**

  - 正向代理是指对客户端提供的代理服务，在客户端无法直接访问原始服务器的情况下，通过配置代理服务器的方式，客户端向代理服务器发送一个请求并指定目标原始服务器，然后代理服务器向原始服务器转交请求并将获得的内容返回给客户端。

![bdd57](https://gitee.com/gengff/blogimage/raw/master/images/bdd57.png)

- #### 2、反向代理（负载均衡）

  只需要找代理，不需要服务器。

  用于公司的集群架构中，**客户端 ----> 代理 <----> 服务端**

  - 反向代理是指对服务端提供的代理服务，反向代理正好与正向代理相反，对于客户端而言代理服务器就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间(name-space)中的内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端。

![3099a7](https://gitee.com/gengff/blogimage/raw/master/images/3099a7.png)

区别:

```
正向代理代理客户端，服务端认为请求来自代理服务器；反向代理代理服务端，客户端认为提供服务的是代理服务器。
正向代理通常由客户端架设，与客户端同处一个局域网；反向代理由服务端架设，与服务端同处一个局域网。
正向代理通常解决访问限制的问题，反向代理通常解决对外服务和负载均衡的问题。
```

## Nginx 代理服务支持的协议

```bash
ngx_http_uwsgi_module		: Python
ngx_http_fastcgi_module		: PHP
ngx_http_scgi_module		: Java
ngx_http_v2_module			: Golang
ngx_http_proxy_module		: HTTP
```

| 反向代理模式                    | Nginx 配置模块          |
| :------------------------------ | :---------------------- |
| uwsgi（Python）                 | ngx_http_uwsgi_module   |
| fastcgi（PHP）                  | ngx_http_fastcgi_module |
| grpc（Golang）                  | ngx_http_v2_module      |
| scgi（Java）                    | ngx_http_scgi_module    |
| proxy（http，https，websocket） | ngx_http_proxy_module   |

## Nginx 反向代理实践

```bash
lb01  --->  web01
lb01：192.168.15.5
web01:192.168.15.8
以超级玛丽小游戏为例
```

### 1、部署 web01

```nginx
[root@web01 ~]#  vim /etc/nginx/conf.d/ game5.conf
server {
    listen 80;
    server_name 192.168.15.8;
    location / {
        root /opt/Super_Marie;
	index index.html;
    }
    location ~ /images {
        root /opt/image;
    }
}
```

### 2、部署 lb01

- 编译安装 nginx

```bash
# 下载Nginx源代码包

[root@lb01 ~]# wget https://nginx.org/download/nginx-1.20.2.tar.gz

# 解压
[root@lb01 ~]# tar -xf nginx-1.20.2.tar.gz

# 进入源代码目录
[root@lb01 ~]# cd nginx-1.20.2

# 安装依赖包
[root@lb01 nginx-1.20.2]# yum install openssl openssl-devel zlib zlib-devel -y

# 设置编译参数
[root@lb01 nginx-1.20.2]# ./configure  --with-http_gzip_static_module  --with-stream  --with-http_ssl_module  --with-http_sub_module

# 编译
[root@lb01 nginx-1.20.2]# make

# 安装
[root@lb01 nginx-1.20.2]# make install
```

- nginx 编译安装优化

```nginx
# 切换至安装路径
[root@lb01 nginx-1.20.2]# cd /usr/local/nginx/

# 创建/etc/nginx目录
[root@lb01 nginx]# mkdir /etc/nginx

# 移动nginx配置文件至/etc/nginx/
[root@lb01 nginx]# mv /usr/local/nginx/conf/* /etc/nginx/
[root@lb01 nginx]# cd /etc/nginx/

# 创建/etc/nginx/conf.d
[root@lb01 nginx]# mkdir /etc/nginx/conf.d

# 清空配置文件内容
[root@lb01 nginx]# >nginx.conf

# 编辑配置文件
[root@lb01 nginx]# vim /etc/nginx/nginx.conf

# 将下方内容复制粘贴进去
user  www;
worker_processes  10;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format json '{"@timestamp":"$time_iso8601",'
                    '"host":"$server_addr",'
                    '"service":"nginxTest",'
                    '"trace":"$upstream_http_ctx_transaction_id",'
                    '"log":"log",'
                    '"clientip":"$remote_addr",'
                    '"remote_user":"$remote_user",'
                    '"request":"$request",'
                    '"http_user_agent":"$http_user_agent",'
                    '"size":$body_bytes_sent,'
                    '"responsetime":$request_time,'
                    '"upstreamtime":"$upstream_response_time",'
                    '"upstreamhost":"$upstream_addr",'
                    '"http_host":"$host",'
                    '"url":"$uri",'
                    '"domain":"$host",'
                    '"xff":"$http_x_forwarded_for",'
                    '"referer":"$http_referer",'
                    '"status":"$status"}';
    access_log /var/log/nginx/access.log json;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
# ：wq! 保存退出

# 创建用户
[root@lb01 nginx]# groupadd www -g 666
[root@lb01 nginx]# useradd www -u 666 -g 666 -M -r -s /sbin/nologin

[root@lb01 nginx]# vim /usr/lib/systemd/system/nginx.service
# 下方内容复制粘贴进去
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)"

[Install]
WantedBy=multi-user.target

# 继续优化
[root@lb01 nginx]# cd /usr/local/nginx/sbin/
[root@lb01 sbin]# mv /usr/local/nginx/sbin/nginx /usr/sbin/
[root@lb01 sbin]# mkdir /var/log/nginx
[root@lb01 sbin]# systemctl start nginx

# 可以看出无法正常显示
[root@lb01 sbin]# nginx -t
 nginx: [emerg] open() "/usr/local/nginx/conf/nginx.conf" failed (2: No such file or directory)
 nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed

# 这样可以正常显示，但是比较麻烦
[root@lb01 sbin]# nginx -t -c /etc/nginx/nginx.conf

# 建立软链接进行优化
[root@lb01 sbin]# ln -s /etc/nginx/nginx.conf /usr/local/nginx/conf/nginx.conf

# 可以正常显示
[root@lb01 sbin]# nginx -t
 nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
 nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

```

- 部署反向代理
  - proxy_pass：设置代理服务器的地址，可以是主机名称、IP 地址加端口号等形式。

```bash
[root@lb01 conf.d]# cat /etc/nginx/conf.d/game.conf
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://192.168.15.8:80;
    }
}

#检查配置文件
[root@lb01 ~]# nginx -t
 nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
 nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

#重载配置文件
[root@lb01 ~]# nginx -s reload

# 测试
 浏览器访问：192.168.15.5
```

![image-20220107173331323](https://gitee.com/gengff/blogimage/raw/master/images/image-20220107173331323.png)

## Nginx 代理常用参数

#### 1、添加请求头信息发往后端服务器

```nginx
Syntax:    proxy_set_header field value;   #将value的值赋值给field字段
Default:    proxy_set_header Host $http_host;
            proxy_set_header Connection close;
Context:    http, server, location

示例配置：在lb01配置 /etc/nginx/conf.d/game.conf
# 传递域名给后端服务器，不设置此项，默认传递ip给后端服务器。
# 用户请求的时候HOST的值是linux.proxy.com, 那么代理服务会像后端传递请求的还是linux.proxy.com
proxy_set_header Host $http_host;

# 最后一层代理的IP地址。多层代理会覆盖，只显示最后一层代理IP地址。
# 将$remote_addr的值放进变量X-Real-IP中，$remote_addr的值为客户端的ip
proxy_set_header X-Real-IP $remote_addr;

# 客户端通过代理服务访问后端服务, 后端服务通过该变量会记录真实客户端地址。多层代理会追加。
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

测试：
# 在web01监控下nginx日志
[root@web01 ~]# tail -f /var/log/nginx/access.log

# 浏览器访问代理服务器地址
192.168.15.5

# 查看日志变化
```

#### 2、代理到后端的 TCP 连接、响应、返回等超时时间

```bash
#nginx代理服务器与后端服务器连接超时时间(代理连接超时)
Syntax: proxy_connect_timeout time;
Default: proxy_connect_timeout 60s;
Context: http, server, location

#nginx代理服务器等待后端服务器响应的超时时间
Syntax:    proxy_read_timeout time;
Default:    proxy_read_timeout 60s;
Context:    http, server, location

#后端服务器数据回传给nginx代理服务器超时时间
Syntax: proxy_send_timeout time;
Default: proxy_send_timeout 60s;
Context: http, server, location

# 配置示例：
proxy_connect_timeout 1s;
proxy_read_timeout 3s;
proxy_send_timeout 3s;
```

#### 3、proxy_buffer 代理缓冲区

```bash
#开启内容缓冲，nignx会把后端返回的内容先放到缓冲区当中，
#然后再返回给客户端，边收边传, 不是全部接收完再传给客户端
Syntax: proxy_buffering on | off;
Default: proxy_buffering on;
Context: http, server, location

#设置nginx代理保存用户头信息的缓冲区大小，
#这个参数并不受proxy_buffering开启或关闭的影响，它始终都是生效的。
Syntax: proxy_buffer_size size;
Default: proxy_buffer_size 4k|8k;
Context: http, server, location

#响应缓冲区的个数和大小，响应内容先写入缓冲区，写满或者写完，立即发送给客户端。
#这里设置的缓冲区大小是针对每个请求连接而言的。
Syntax: proxy_buffers number size;
Default: proxy_buffers 8 4k|8k;
Context: http, server, location

# 配置示例
proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 64k;
proxy_busy_buffers_size 96k;
```

#### 4、配置代理优化文件

```nginx
1.创建代理优化文件
[root@lb01 sbin]# cd /etc/nginx/
[root@lb01 nginx]# vim proxy_params
#将下方内容复制粘贴进去即可
proxy_set_header Host $http_host;  #传递ip给被代理(后端)服务器
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_connect_timeout 10s;  #nginx服务器与被代理(后端)服务器建立连接的超时时间(默认60秒)
proxy_read_timeout 10s;     #连接成功后，被代理(后端)服务器响应时间(代理接收超时)
proxy_send_timeout 10s;     #被代理(后端)服务器数据回传时间(代理发送超时)
proxy_buffering on;         #设置代理服务器（nginx）保存用户头信息的缓冲区大小
proxy_buffer_size 8k;       #高负荷下缓冲大小（proxy_buffers*2）
proxy_buffers 8 8k;         #缓冲区，网页平均在32k以下的话，这样设置

# 下方可以选择性配置
proxy_temp_file_write_size 8k;   #设定缓存文件夹大小，大于这个值，将从upstream服务器传
client_max_body_size       8k;   #允许客户端请求的最大单文件字节数
client_body_buffer_size    8k;   #缓冲区代理缓冲用户端请求的最大字节数


2.添加配置
[root@lb01 ~]# cat /etc/nginx/game.conf
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://172.16.1.7:80;
        include /etc/nginx/proxy_params;  #只需要添加代理优化文件路径即可
    }
}

3.重载nginx即可
[root@lb01 nginx]# nginx -t
[root@lb01 nginx]# nginx -s reload


4.测试（该小游戏无法测试到超时，后面可以用python项目测试）
# 在web01监控下nginx的日志
[root@web01 ~]# tail -f /var/log/nginx/access.log

# 浏览器访问代理服务器地址
192.168.15.5

# 在web01查看日志可以发现"xff":"192.168.15.1"记录到了真实的IP地址，说明设置生效了
{"@timestamp":"2022-01-07T19:51:00+08:00",
 "host":"192.168.15.8","service":"nginxTest","trace":"-","log":"log",
 "clientip":"192.168.15.5","remote_user":"-","request":"GET /sounds/stomp.mp3 HTTP/1.0",
  "http_user_agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36",
  "size":555,"responsetime":0.000,"upstreamtime":"-","upstreamhost":"-",
  "http_host":"192.168.15.5","url":"/sounds/stomp.mp3","domain":"192.168.15.5",
   "xff":"192.168.15.1","referer":"http://192.168.15.5/","status":"404"
}
```

# 负载均衡

- 上面我们介绍了 Nginx 一个很重要的功能——代理，包括正向代理和反向代理。这两个代理的核心区别是：正向代理代理的是客户端，而反向代理代理的是服务器。其中我们又重点介绍了反向代理，以及如何通过 Nginx 来实现反向代理。那么了解了 Nginx 的反向代理之后，我们要通过 Nginx 的反向代理实现另一个重要功能——负载均衡。

  为什么要用负载均衡？

- 负载均衡是将负载（工作任务，访问请求）进行平衡、分摊到多个操作单元（服务器，组件）上进行执行。是解决高性能，单点故障（高可用），扩展性（水平伸缩）的终极解决方案。

## 负载均衡的架构

通过代理将流量按照一定的比例，转发到后端。

![fuzaijunheng](https://gitee.com/gengff/blogimage/raw/master/images/fuzaijunheng.png)

## 负载均衡的实现

### 1、实现

```bash
将后端服务打包成一个IP连接池。

1、反向代理
server {
   	listen 80;
   	server_name _;
   	location / {
        proxy_pass http://[连接池];
   	}
}

2、IP连接池
upstream [连接池名称] {
    server [ip]:[port];
    server [ip]:[port];
    server [ip]:[port];
}


3、配置
[root@lb01 conf.d]# cat test.conf
upstream supermarie {
    server 172.16.1.7:80;
    server 172.16.1.8:80;
    server 172.16.1.9:80;
}

server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://supermarie;  #反向代理IP连接池
        include /etc/nginx/proxy_params;
    }
}

```

### 2、负载均衡的比例

#### 2.1、轮询

- 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
  - 客户端发送一条请求，第一条请求转发给了 web1 服务器处理，下一条请求则会转发给 web2 ，如果再有一条请求来，则发给 web3，还有一条请求来则又给 web1，最后的结果则是 web1、web2、web3、web1 实现了轮询访问

```bash
# 默认情况下，Nginx负载均衡的轮询状态。
upstream supermarie {
    server 172.16.1.7:80;
    server 172.16.1.8:80;
    server 172.16.1.9:80;
}
```

#### 2.2、权重

- Nginx 中的 weight 权重，默认为 1，weight 越大，负载的权重就越大，表示访问几率越大，用于后端服务器性能不均的情况

```bash
# 权重0-100，数字越大，权重越高，流量也就最多。
upstream supermarie {
    server 172.16.1.7:80 weight=9;
    server 172.16.1.8:80 weight=5;
    server 172.16.1.9:80 weight=1;
}
```

#### 2.3、ip_hash

- nginx 提供的 ip_hash 策略。既能满足每个用户请求到同一台服务器，又能满足不同用户之间负载均衡。
  - 通过客户端请求 ip 进行 hash，再通过 hash 值选择后端 server（每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题）

```bash
# 每一个IP固定访问某一个后端。
upstream supermarie {
    server 172.16.1.7:80;
    server 172.16.1.8:80;
    server 172.16.1.9:80;
    ip_hash;
}
```

### 3、负载均衡后端状态

| **状态**     | **概述**                                 |
| ------------ | ---------------------------------------- |
| down         | 表示当前的 Web Server 暂时不参与负载均衡 |
| backup       | 预留的备份服务器                         |
| max_fails    | 允许请求失败的次数                       |
| fail_timeout | 经过 max_fails 失败后， 服务暂停时间     |

#### 3.1、down

```bash
# 暂时不分配流量
upstream supermarie {
    server 172.16.1.7:80 down;
    server 172.16.1.8:80;
    server 172.16.1.9:80;
}

server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://supermarie;
        include /etc/nginx/proxy_params;
    }
}

```

#### 3.2、backup

```bash
# 只有当所有的机器全部宕机，才能启动。
upstream supermarie {
    server 172.16.1.7:80 backup;
    server 172.16.1.8:80;
    server 172.16.1.9:80;
}

server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://supermarie;
        include /etc/nginx/proxy_params;
    }
}
```

#### 3.3、max_fails、fail_timeout

```bash
server IP:端口 max_fails=3 fail_timeout=3s
# 允许请求失败3次，失败后服务暂停3秒。max_fails 和 fail_timeout 两个必须一起使用。
# proxy_next_upstream 后端错误标识

[root@lb01 ~]# cat /etc/nginx/conf.d/game.conf
upstream supermarie {
    server 172.16.1.7:80 max_fails=3 fail_timeout=3s;
    server 172.16.1.8:80 max_fails=3 fail_timeout=3s;
    server 172.16.1.9:80 max_fails=3 fail_timeout=3s;
}

server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://supermarie;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_404;
        include /etc/nginx/proxy_params;
    }
}

```

```bash
注意：proxy_next_upstream error timeout invalid_header http_500 http_503 http_404;

error             # 与服务器建立连接，向其传递请求或读取响应头时发生错误;
timeout           # 在与服务器建立连接，向其传递请求或读取响应头时发生超时;
invalid_header    # 服务器返回空的或无效的响应;
http_500          # 服务器返回代码为500的响应;
http_502          # 服务器返回代码为502的响应;
http_503          # 服务器返回代码为503的响应;
http_504          # 服务器返回代码504的响应;
http_403          # 服务器返回代码为403的响应;
http_404          # 服务器返回代码为404的响应;
http_429          # 服务器返回代码为429的响应（1.11.13）;
non_idempotent    # 通常，请求与 非幂等 方法（POST，LOCK，PATCH）不传递到请求是否已被发送到上游服务器（1.9.13）的下一个服务器; 启用此选项显式允许重试此类请求;
off               # 禁用将请求传递给下一个服务器。
```

## 负载均衡部署 BBS

### 1、部署后端服务

#### 1、部署 Python

```bash
1、创建用户
[root@web01 opt]# groupadd django -g 888
[root@web01 opt]# useradd django -u 888 -g 888 -r -M -s /bin/sh

2、安装依赖软件
[root@web01 opt]# yum install python3 libxml* python-devel gcc* pcre-devel openssl-devel python3-devel -y
```

#### 2、部署 Django 和 uwsgi

```bash
3、安装Django和uwsgi
[root@web01 opt]# pip3 install django==1.11
[root@web01 opt]# pip3 install uwsgi

# 根据需求安装相应缺少模块
[root@web01 opt]# pip3 install pymysql

4、创建项目
[root@web01 opt]# unzip bbs.zip
[root@web03 bbs]# pwd
/opt/bbs
[root@web03 bbs]# vim bbs/settings.py
ALLOWED_HOSTS = ['*']
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'bbs',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': '172.16.1.61',
        'PORT': 3306,
        'CHARSET': 'utf8'
    }
}

# 启动测试
[root@web01 bbs]# python3 manage.py runserver 0.0.0.0:8000
```

#### 3、配置并启动

```bash
5、编辑项目配置文件
[root@web01 ~]# cat /opt/bbs/myweb_uwsgi.ini
[uwsgi]
# 端口号
socket = :8000
# 指定项目的目录
chdir = /opt/bbs
# wsgi文件路径
wsgi-file = bbs/wsgi.py
# 模块wsgi路径
module = bbs.wsgi
# 是否开启master进程
master = true
# 工作进程的最大数目
processes = 4
# 结束后是否清理文件
vacuum = true

6、启动uwsgi
[root@web01 ~]# uwsgi -d --ini /opt/bbs/myweb_uwsgi.ini --uid 666

-d 	  : 以守护进程方式运行
--ini : 指定配置文件路径
--uid : 指定uid

7、编辑Nginx配置文件
[root@web01 ~]# cat /etc/nginx/conf.d/python.conf
server {
    listen 80;
    server_name py.test.com;
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8000;
        uwsgi_read_timeout 2;
        uwsgi_param UWSGI_SCRIPT bbs.wsgi;
        uwsgi_param UWSGI_CHDIR /opt/bbs;
        index  index.html index.htm;
        client_max_body_size 35m;
    }
}

8、重启Nginx配置
[root@web01 ~]# systemctl restart nginx
```

### 2、部署负载均衡

```bash
[root@lb01 ~]# cat /etc/nginx/conf.d/test.conf
upstream bbs {
    server 172.16.1.7:80 max_fails=3 fail_timeout=3s;
    server 172.16.1.8:80 max_fails=3 fail_timeout=3s;
    server 172.16.1.9:80 max_fails=3 fail_timeout=3s;
}

server {
    listen 80;
    server_name bbs.test.com;
    location / {
        proxy_pass http://bbs;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_404;
        include /etc/nginx/proxy_params;
    }
}

# 测试重载nginx
[root@lb01 ~]# nginx -t
[root@lb01 ~]# systemctl restart nginx


# 测试
# 如果访问不到，说明访问web的nginx配置为默认的配置可以在web内禁用掉默认配置
[root@web01 ~]# mv /etc/nginx/conf.d/default.conf default.conf.bak
[root@web01 ~]# nginx -t
[root@web01 ~]# systemctl restart nginx

# 在web01、web02监控下nginx的日志
[root@web01 ~]# tail -f /var/log/nginx/access.log

浏览器输入：bbs.test.com
```

![6671641645008_.pic_hd_gaitubao_461x523](https://gitee.com/gengff/blogimage/raw/master/images/6671641645008_.pic_hd_gaitubao_461x523.jpg)

# 四层负载均衡

- 1、四层 +七层来做负载均衡，四层可以保证七层的负载均衡的高可用性；如：nginx 就无法保证自己的服务高可用，需要依赖 LVS 或者 keepalived。
- 2、tcp 协议的负载均衡，有些请求是 TCP 协议的（mysql、ssh），或者说这些请求只需要使用四层进行端口的转发就可以了，所以使用四层负载均衡。
- 3、四层可以做：
  - mysql 读从库的负载均衡
  - 跳板机的端口映射

```bash
假设有三台MySQL数据库，请问怎样负载均衡？

在非HTTP协议的情况下，采用的四层负载均衡的方式负载服务。

注意：四层负载均衡中不支持域名。
```

## 四层负载均衡配置

- 配置四层负载均衡 nginx 必须有--with-stream 模块，之前已经装过了

```bash
# 四层负载均衡stream模块跟http模块同级别，不能配置在http里面
[root@lb01 ~]# vim /etc/nginx/nginx.conf

user  www;
worker_processes  10;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

# 配置四层负载均衡stream
stream {
    include /etc/nginx/stream/*.conf;
}

events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
# ......略



# 切换到nginx目录
[root@lb01 nginx]# pwd
 /etc/nginx

# 创建切换到stream目录
[root@lb01 nginx]# mkdir stream
[root@lb01 nginx]# cd stream

# 新建编辑stream配置文件
[root@lb01 stream]# vim mysql.conf

#注意：四层负载均衡中不支持域名。没有server_name
server {
    listen 3306;
    proxy_pass 192.168.15.61:3306;
}
# 重启nginx
[root@lb01 stream]# nginx -t
[root@lb01 stream]# systemctl restart nginx

# 测试:成功利用nginx负载均衡转发
[root@db01 ~]# mysql -h192.168.15.5 -uroot -p123456
 MariaDB [(none)]>
```

案例：使用四层负载均衡实现 SSH 的代理，端口为 1122

```bash
# 配置文件
[root@lb01 stream]# cat ssh.conf
server {
    listen 1122;
    proxy_pass 172.16.1.5:22;
}
# 查看端口
[root@lb01 stream]# netstart -nutlp

# 重启nginx
[root@lb01 stream]# nginx -t
[root@lb01 stream]# systemctl restart nginx

# 测试在web02登录
[root@web02 ~]# ssh root@192.168.15.5 -p1122

 root@192.168.15.5's password:

[root@lb01 ~]#
```
