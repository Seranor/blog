---
title: Web服务器-Nginx介绍
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Nginx
categories:
  - Linux
url: post/linux-19.html
toc: true
---

# Web 服务器

### 什么是 web 服务

​ web 就是 B/S 架构

- Web 服务器一般指网站服务器，是指驻留于因特网上某种类型计算机的程序，可以处理浏览器等 Web 客户端的请求并返回相应响应，也可以放置网站文件，让全世界浏览；可以放置数据文件，让全世界下载。

- WEB 服务器也称为 WWW(WORLD WIDE WEB)服务器，主要功能是提供网上信息浏览服务。WWW 是 Internet 的多媒体信息查询工具，是 Internet 上近年才发展起来的服务，也是发展最快和目前用的最广泛的服务。正是因为有了 WWW 工具，才使得近年来 Internet 迅速发展，且用户数量飞速增长。
<!-- more -->

### 网络模型（IO 多路复用之 select、poll、epoll）

epoll 跟 select 都能提供多路 I/O 复用的解决方案。在现在的 Linux 内核里有都能够支持，`其中epoll是Linux所特有，而select则应该是POSIX所规定`，一般操作系统均有实现。

```bash
select
  - 基本原理：select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

  - select目前几乎在所有的平台上支持，`其良好跨平台支持也是它的一个优点`。`select的一个缺点在于单个进程能够监视的文件描述符的数量存在最大限制`，在Linux上一般为1024，`可以通过修改宏定义甚至重新编译内核的方式提升这一限制`，但是这样也会造成效率的降低。

poll
  - *基本原理：`poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间`，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。

epoll
  - 基本原理：`epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次`。还有一个特点是，`epoll使用“事件”的就绪通知方式`，通过epoll_ctl注册fd，`一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd`，epoll_wait便可以收到通知。

  - epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。`epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次`。
```

参考链接：https://www.cnblogs.com/Anker/p/3265058.html

select，poll，epoll 都是 IO 多路复用的机制。I/O 多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。`但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的`，而异步 I/O 则无需自己负责进行读写，异步 I/O 的实现会负责把数据从内核拷贝到用户空间。关于这三种 IO 多路复用的用法，链接如下所示：

select：http://www.cnblogs.com/Anker/archive/2013/08/14/3258674.html

poll：http://www.cnblogs.com/Anker/archive/2013/08/15/3261006.html

epoll：http://www.cnblogs.com/Anker/archive/2013/08/17/3263780.html

# Web 服务器软件

## 1、apache

- Apache(音译为阿帕奇)是世界使用排名第一的 Web 服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的 Web 服务器端软件之一。它快速、可靠并且可通过简单的 API 扩充，将 Perl/Python 等解释器编译到服务器中。

```bash
# Apache 和 Nginx 功能对比
   Nginx和Apache一样，都是HTTP服务器软件，在功能实现上都采用模块化结构设计，都支持通用的语言接口，如PHP、Perl、Python等，同时还支持正向和反向代理、虚拟主机、URL重写、压缩传输、SSL加密传输等。

1.在功能实现上，Apache的所有模块都支持动、静态编译，而Nginx模块都是静态编译的，
2.对FastCGI的支持，Apache对Fcgi的支持不好，而Nginx对Fcgi的支持非常好；
3.在处理连接方式上，Nginx支持epoll，而Apache却不支持；
4.在空间使用上，Nginx安装包仅仅只有几百K，和Nginx比起来Apache绝对是庞然大物。
```

## 2、Nginx

- Nginx (engine x) 是一个很强大的、高性能的 HTTP 和反向代理 web 服务器，同时也提供 IMAP/POP3/SMTP 服务。Nginx 是一款轻量级的 Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在 BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上 nginx 的并发能力在同类型的网页服务器中表现较好，中国大陆使用 nginx 网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。
- Nginx 代码完全用 C 语言从头写成，已经移植到许多体系结构和操作系统，包括：Linux、FreeBSD、Solaris、Mac OS X、AIX 以及 Microsoft Windows。Nginx 有自己的函数库，并且除了 zlib、PCRE 和 OpenSSL 之外，标准模块只使用系统 C 库函数。而且，如果不需要或者考虑到潜在的授权冲突，可以不使用这些第三方库。

```bash
# 选择Nginx的优势所在
1. 作为Web服务器: Nginx处理静态文件、索引文件，自动索引的效率非常高。
2. 作为代理服务器，Nginx可以实现无缓存的反向代理加速，提高网站运行速度。
3. 作为负载均衡服务器，Nginx既可以在内部直接支持Rails和PHP，也可以支持HTTP代理服务器对外进行服务，同时还支持简单的容错和利用算法进行负载均衡。
4. 在性能方面，Nginx是专门为性能优化而开发的，在实现上非常注重效率。它采用内核Poll模型(epoll and kqueue )，可以支持更多的并发连接，最大可以支持对50 000个并发连接数的响应，而且只占用很低的内存资源。
5. 在稳定性方面，Nginx采取了分阶段资源分配技术，使得CPU与内存的占用率非常低。Nginx官方表示，Nginx保持10 000个没有活动的连接，而这些连接只占用2.5MB内存，因此，类似DOS这样的攻击对Nginx来说基本上是没有任何作用的。
6. 在高可用性方面，Nginx支持热部署，启动速度特别迅速，因此可以在不间断服务的情况下，对软件版本或者配置进行升级，即使运行数月也无需重新启动，几乎可以做到7×24小时不间断地运行。
```

# Nginx

![d21fa](https://gitee.com/gengff/blogimage/raw/master/images/d21fa.jpg)

## 安装 Nginx

官网：https://nginx.org/
软件：https://nginx.org/download/

```nginx
1、yum安装
 [root@web01 ~]# vim /etc/yum.repos.d/nginx.repo
   [nginx-stable]
   name=nginx stable repo
   baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=1
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true

   [nginx-mainline]
   name=nginx mainline repo
   baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true
 [root@web01 ~]# yum install nginx -y
 [root@web01 ~]# systemctl stop httpd
 [root@web01 ~]# systemctl start nginx

2、二进制安装
    略.....

3、编译安装
 [root@web01 ~]#  wget https://nginx.org/download/nginx-1.20.2.tar.gz
 [root@web01 ~]# tar -xf nginx-1.20.2.tar.gz
 [root@web01 nginx-1.20.2]# ./configure
 [root@web01 nginx-1.20.2]# make
 [root@web01 nginx-1.20.2]# make install
```

## 平滑增加 Nginx 模块

- 众所周知 Nginx 是分成一个个模块的，比如 core 模块，gzip 模块，proxy 模块，每个模块负责不同的功能，除了基本的模块，有些模块可以选择编译或不编译进 Nginx。官网文档中的 Modules reference 部分列出了 nginx 源码包的所有模块。我们可以按照自己服务器的需要来定制出一个最适合自己的 Nginx 服务器。

- 除了 Nginx 官网源码包提供了各种模块，Nginx 还有各种各样的第三方模块。官方文档 NGINX 3rd Party Modules 也列出了 Nginx 的很多第三方模块，除此官网列出的之外，还有很多很有用的模块也能在 Github 等网站上找到。

- 这些模块提供着各种各样意想不到的功能，灵活使用 Nginx 的第三方模块，可能会有非常大的意外收获。

```nginx
# 增加模块必须重新编译。
 [root@web01 ~]#  wget https://nginx.org/download/nginx-1.20.2.tar.gz
 [root@web02 ~]# tar -xf nginx-1.20.2.tar.gz
 [root@web02 ~]# cd nginx-1.20.2
 [root@web02 nginx-1.20.2]#./configure  --with-http_ssl_module
 [root@web02 nginx-1.20.2]# make
 [root@web02 nginx-1.20.2]# make install

# 编译安装无法自动添加到环境变量，需要带上路径查看版本
 [root@web01 nginx-1.20.2]# /usr/local/nginx/sbin/nginx -v

# 查看配置参数
 [root@web02 nginx-1.20.2]# ./configure --help

# 增加模块，报错直接yum增加依赖即可
 [root@web02 nginx-1.20.2]# ./configure --with-http_ssl_module
 [root@web02 nginx-1.20.2]# make
 [root@web02 nginx-1.20.2]# make install

# 此时可以看见模块已经增加进去了
 [root@web01 nginx-1.20.2]# /usr/local/nginx/sbin/nginx -V
```

## Nginx 的命令

```nginx
# -?,-h : 帮助
 [root@web01 ~]# nginx -h
  nginx version: nginx/1.20.2
  Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
               [-e filename] [-c filename] [-g directives]

  Options:

 1、-v : 版本信息
  [root@web01 ~]# nginx -v
   nginx version: nginx/1.20.2

 2、-V : 版本和配置选项信息
  [root@web01 ~]# nginx -V
   nginx version: nginx/1.20.2
   built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
   built with OpenSSL 1.0.2k-fips  26 Jan 2017
   TLS SNI support enabled
   configure arguments: --prefix=/etc/nginx
   ......

 3、-t : 检查配置文件是否有语法错误
  [root@web01 ~]# nginx -t
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful

 4、-T : 测试配置文件并启动
  [root@web01 ~]# nginx -T
   nginx: the configuration file /etc/nginx/nginx.conf syntax is onginx: the configuration           file /etc/nginx/nginx.conf syntax is oTk
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   # configuration file /etc/nginx/nginx.conf:

   user  nginx;
   worker_processes  auto;

   error_log  /var/log/nginx/error.log notice;
   pid        /var/run/nginx.pid;
   ......

 5、-q : 打印错误日志
  [root@web01 ~]# nginx -q
   nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
   nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
   nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)

 6、-s : 指定Nginx启动配置（给nginx主进程发送信号）
    stop : 停止（快速关闭）
     [root@web01 ~]# nginx -s stop
    quit : 退出（正常关闭）
     [root@web01 ~]# nginx -s quit
    reopen : 重启
     [root@web01 ~]# nginx -s reopen
    reload : 重载
     [root@web01 ~]# nginx -s reload

 7、-p : 指定nginx的安装路径（默认是：/etc/nginx/）

 8、-e : 指定错误日志路径

 9、-c : 指定配置文件的路径（默认是：/etc/nginx/nginx.conf）

 10、-g : 设置一个全局的Nginx配置项
  [root@web01 ~]# nginx -g 'daemon off;'  #前端启动
```

## Nginx 配置文件

- nginx 分为全局配置和模块配置

```nginx
#/etc/nginx/nginx.conf
-------------------- 全局配置(全局生效) -----------------
user  www;	#启动nginx work进程的用户名
worker_processes  auto;  #启动的worker进程数，auto === CPU数量

error_log  /var/log/nginx/error.log notice;  #错误日志路径
pid        /var/run/nginx.pid;  #pid的存放文件文件路径

-------------- 系统事件配置模块(全局生效) ---------------
events {	#事件配置模块
    worker_connections  1024;  #最大连接数（每一个worker进程最多同时接入多少个请求）
    use epool;  #指定Nginx的网络模型
}

-------------- Http请求模块(处理Http请求的模块) --------------
http {  #web服务的模块
    include       /etc/nginx/mime.types;  #加载外部的配置项（Nginx可以处理的文件类型）
    default_type  application/octet-stream;  #如果找不到文件的类型，则按照指定默认类型处理
    #设置的日志格式，下方详解
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;	#访问日志

    sendfile        on;	 #tcp连接配置（高效读取文件）
    #tcp_nopush     on;

    keepalive_timeout  65;  #长链接配置（HTTP 1.0 短链接，HTTP 1.1 长连接）

    #gzip  on;  #压缩

    include /etc/nginx/conf.d/*.conf;  #包含其他文件
}

server {  #网址模块
    listen 80;  #监听的端口
    server_name 127.0.0.1;	#定义域名

    location / {  #访问路径
	root /blog;  #指定的站点目录（网址路径）
	index index.php;  #指定网址的索引文件
	}
```

## 超级玛丽和象棋

```nginx
1、上传代码
 # 定义目录
 [root@web01 ~]# cd /opt
 [root@web01 opt]# mkdir Super_Marie
  # 此时需要手动上传文件
 [root@web01 opt]# ll Super_Marie/
  总用量 176
  drwxr-xr-x 2 root root   329 1月   3 21:44 images
  -rw-r--r-- 1 root root  1703 1月   3 21:44 index.html
  -rw-r--r-- 1 root root 72326 1月   3 21:44 jquery.js
  -rw-r--r-- 1 root root 78982 1月   3 21:44 QAuIByrkL.js
  -rw-r--r-- 1 root root  4777 1月   3 21:44 VNkyVaVxUV.css
  -rw-r--r-- 1 root root  9539 1月   3 21:44 wNGu2CtEMx.js



2、编辑配置文件
[root@web01 opt]# vim /etc/nginx/conf.d/game.conf
server {
    listen 80;
    server_name game.test.com;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
}

3、测试配置文件是否正常
[root@web01 opt]# nginx -t
 nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
 nginx: configuration file /etc/nginx/nginx.conf test is successful

4、重启Nginx
[root@web01 opt]# systemctl restart nginx

5、Windows域名解析
C:\Windows\System32\drivers\etc\hosts
172.16.1.7 game.test.com


6、Mac域名解析
# 切换到root权限
  sudo -i

# 编辑/etc/hosts文件，在后面追加内容即可，:wq保存退出
  vim /etc/hosts
  172.16.1.7 game.test.com


```

# Nginx 配置虚拟主机

**可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。**

虚拟主机技术是互联网服务器采用的节省服务器硬件成本的技术，虚拟主机技术主要应用于 HTTP 服务，将一台服务器的某项或者全部服务内容逻辑划分为多个服务单位，对外表现为多个服务器，从而充分利用服务器硬件资源。

虚拟主机是使用特殊的软硬件技术，把一台真实的物理服务器主机分割成多个逻辑存储单元。每个逻辑单元都没有物理实体，但是每一个逻辑单元都能像真实的物理主机一样在网络上工作，具有单独的 IP 地址（或共享的 IP 地址）、独立的域名以及完整的 Internet 服务器（支持 WWW、FTP、E-mail 等）功能。

虚拟主机的关键技术在于，即使在同一台硬件、同一个操作系统上，运行着为多个用户打开的不同的服务器程式，也互不干扰。而各个用户拥有自己的一部分系统资源（IP 地址、文档存储空间、内存、CPU 等）。各个虚拟主机之间完全独立，在外界看来，每一台虚拟主机和一台单独的主机的表现完全相同。所以这种被虚拟化的逻辑主机被形象地称为“虚拟主机”。

- 基于多 IP 的方式
- 基于多端口的方式
- 基于多域名的方式

```nginx
1、基于多IP的方式
[root@web01 conf.d]# vim game.conf
[root@web01 conf.d]# cat game.conf
server {
    listen 80;
    server_name 192.168.15.7;
    location / {
	root /opt/Super_Marie;
        index index.html;
    }
}
server {
    listen 80;
    server_name 172.16.1.7;
    location / {
        root /opt/tank;
        index index.html;
    }
}
# 测试：浏览器访问
192.168.15.7
172.16.1.7

2、基于多端口的方式
[root@web01 conf.d]# vim game1.conf
[root@web01 conf.d]# cat game1.conf
server {
    listen 80;
    server_name 192.168.15.7;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
}
server {
    listen 81;
    server_name 192.168.15.7;
    location / {
        root /opt/tank;
        index index.html;
    }
}

# 测试：浏览器访问
192.168.15.7
192.168.15.7:81

3、基于多域名的方式
[root@web01 conf.d]# vim game2.conf
[root@web01 conf.d]# cat game2.conf
server {
    listen 80;
    server_name www.game.com;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
}
server {
    listen 80;
    server_name www.game1.com;
    location / {
        root /opt/tank;
        index index.html;
    }
}

# 测试：浏览器访问
www.game.com
www.game1.com
```

# Nginx 日志

- nginx 的 log 日志分为 access log 和 error log

  - 其中 access log 记录了哪些用户，哪些页面以及用户浏览器、ip 和其他的访问信息

  - error log 则是记录服务器错误日志

- Nginx 有非常灵活的日志记录模式，每个级别的配置可以有各自独立的访问日志。日志格式通过 log_format**命令定义格式**

```nginx
access_log  /var/log/nginx/access.log  main;

access_log  	#日志的模块
/var/log/nginx/access.log  	#日志的路径
main;	#日志的格式


日志的格式
 log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

$remote_addr        # 记录客户端IP地址
$remote_user        # 记录客户端用户名
$time_local         # 记录通用的本地时间
$time_iso8601       # 记录ISO8601标准格式下的本地时间
$request            # 记录请求的方法以及请求的http协议
$status             # 记录请求状态码(用于定位错误信息)
$body_bytes_sent    # 发送给客户端的资源字节数，不包括响应头的大小
$bytes_sent         # 发送给客户端的总字节数
$msec               # 日志写入时间。单位为秒，精度是毫秒。
$http_referer       # 记录从哪个页面链接访问过来的
$http_user_agent    # 记录客户端浏览器相关信息
$http_x_forwarded_for #记录经过的所有服务器的IP地址（在反向代理中生效）
$X-Real-IP		   #记录起始的客户端IP地址和上一层客户端的IP地址
$request_length     # 请求的长度（包括请求行， 请求头和请求正文）。
$request_time       # 请求花费的时间，单位为秒，精度毫秒

# 注:如果Nginx位于负载均衡器，nginx反向代理之后， web服务器无法直接获取到客户端真实的IP地址。
# $remote_addr获取的是反向代理的IP地址。 反向代理服务器在转发请求的http头信息中，
# 增加X-Forwarded-For信息，用来记录客户端IP地址和客户端请求的服务器地址。

ps:出现报错等问题首先应该查看报错日志
排错：
 [root@web01 ~]# systemctl status nginx.service -l
查看报错日志：
 [root@web01 ~]# cat /var/log/nginx/error.log
```

### json 日志模板

```nginx
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
```

# Nginx 模块介绍

[Nginx 模块官方文档 TP](http://nginx.org/en/docs/)

## Nginx 访问控制模块

- ngx_http_access_module

```nginx
允许或者拒绝某些客户端地址的访问
deny  : 拒绝
allow : 允许

案例1：允许192.168.15.1访问，不允许其他IP访问
server {
    listen 80;
    server_name www.game.com;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
    allow 192.168.15.1;
    deny all;
}


案例2：允许192.168.15.0这个网段访问，不允许其他网段访问
server {
    listen 80;
    server_name www.game.com;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
    allow 192.168.15.0/24;
    deny all;
}


案例3：只允许通过VPN来访问
server {
    listen 80;
    server_name www.game.com;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
    allow 172.16.1.81;
    deny all;
}

ps:每次修改完配置需要重启nginx
[root@web01 ~]# systemctl restart nginx
```

## Nginx 访问认证模块

- ngx_http_auth_basic_module

```nginx
访问之前需要登录

# 注釋
auth_basic "Welcome To Login";
# 指定认证的文件
auth_basic_user_file /etc/nginx/auth;


1、安装httpd-tools
[root@web01 ~]# yum install httpd-tools -y

2、生成用户名密码文件
[root@web01 ~]# htpasswd -c /etc/nginx/auth gengfeng
New password:
Re-type new password:
Adding password for user gengfeng

3、查看密码文件内容
[root@web01 ~]# cat /etc/nginx/auth
gengfeng:$apr1$b5wYYZ9b$PzcNn9gq.fgYUR4l0M3OL/


4、将文件路径加入Nginx配置
[root@web01 ~]# vim /etc/nginx/conf.d/game.conf
server {
        listen 80;
        server_name game.test.com;
        location / {
                root /opt/Super_Marie;
                index index.html;
                }
        auth_basic "Welcome To Login";
        auth_basic_user_file /etc/nginx/auth;
        }


5、重启Nginx
[root@web01 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@web01 ~]# systemctl restart nginx

6、测试
 浏览器访问 http://game.test.com
```

![image-20220104174024762](https://gitee.com/gengff/blogimage/raw/master/images/image-20220104174024762.png)

## Nginx 目录索引模块

- ngx_http_autoindex_module

```nginx
展示目录索引

#启用或禁用目录列表输出。(默认off)
autoindex on | off;

#显示具体大小 off 显示K/M/G单位   on 单位就是 bytes (默认on)
autoindex_exact_size on | off;

#显示文件最后修改时间  如果是 off 需要加8小时（默认off）
autoindex_localtime on | off;

#设置目录列表的格式（默认html）
autoindex_format html | xml | json | jsonp;


[root@web01 ~]# vim /etc/nginx/conf.d/game.conf
server {
        listen 80;
        server_name game.test.com;
        location / {
                root /opt/Super_Marie;
                }
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
        autoindex_format html;
        }
[root@web01 ~]# systemctl restart nginx

#测试：
 浏览器访问 http://game.test.com
```

![image-20220104185038699](https://gitee.com/gengff/blogimage/raw/master/images/image-20220104185038699.png)

## Nginx 状态监控模块

- ngx_http_stub_status_module

```nginx
监控Nginx运行状态

[root@web01 ~]# vim /etc/nginx/conf.d/game.conf
[root@web01 ~]# cat /etc/nginx/conf.d/game.conf
server {
        listen 80;
        server_name game.test.com;
        location / {
                root /opt/Super_Marie;
                index index.html;
        }
        location /status {
                stub_status;
        }
}

[root@web01 down]# nginx -s reload  #重载


#测试：浏览器访问 http://game.test.com/status
```

![image-20220104194138460](https://gitee.com/gengff/blogimage/raw/master/images/image-20220104194138460.png)

### Nginx 的七种状态

```nginx
Active connections
   当前活动客户端连接数，包括Waiting连接数。
accepts
   接受的客户端连接总数。
handled
   处理的连接总数。通常，accepts 除非达到某些资源
   限制（例如，worker_connections限制），否则 该参数值是相同的。
requests
   客户端请求的总数。
Reading
   nginx 正在读取请求头的当前连接数。
Writing
   nginx 将响应写回客户端的当前连接数。
Waiting
   当前等待请求的空闲客户端连接数。

#注意：一次tcp连接，可以发起多次请求；
keepalive_timeout  0;   #类似于关闭长连接
keepalive_timeout  65;	#最长65秒没有活动则断开连接
```

## Nginx 连接限制模块

- ngx_http_limit_conn_module

```nginx
用于限制每个定义的键的连接数，特别是来自单个IP地址的连接数。

limit_conn : 该指令指定每个给定键值的最大同时连接数，当超过这个数字时返回503错误
limit_conn_log_level : 当达到最大限制连接数后，记录日志的等级，默认error级别
limit_conn_status : 指定当超过限制时，返回的状态码。默认是503。
limit_conn_zone : 主要用来定义变量、zone名称、共享内存大小
  $remote_addr
      变量的长度为7字节到15字节，而存储状态在32位平台中占用32字节或64字节，
      在64位平台中占用64字节。
  $binary_remote_addr
      变量的长度是固定的4字节，存储状态在32位平台中占用32字节或64字节，
      在64位平台中占用64字节。

ps:1M共享空间可以保存3.2万个32位的状态，1.6万个64位的状态。如果共享内存空间被耗尽，
   服务器将会对后续所有的请求返回 503 (Service Temporarily Unavailable) 错误。


控制Nginx连接数
[root@web01 ~]# vim /etc/nginx/conf.d/game.conf
[root@web01 ~]# cat /etc/nginx/conf.d/game.conf
limit_conn_zone $remote_addr zone=addr:1m;
limit_conn_log_level error;
limit_conn_status 503;
server {
    listen 80;
    server_name 192.168.15.8;
    limit_conn addr 1;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
}

[root@web01 down]# nginx -s reload

测试
  1、安装ab压力测试命令
  [root@web01 ~]# yum install httpd-tools -y

  2、ab 参数
     -n : 总共需要访问多少次
     -c : 每次访问多少个

测试结果:
[root@web01 ~]# ab -n 10000 -c 1000 http://192.168.15.8/
.....

Benchmarking 192.168.15.8 (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.20.2
Server Hostname:        192.168.15.8
Server Port:            80

Document Path:          /
Document Length:        1703 bytes

Concurrency Level:      1000
Time taken for tests:   6.416 seconds
Complete requests:      10000  #总共访问1万次
Failed requests:        128    #失败次数128次
   (Connect: 0, Receive: 0, Length: 64, Exceptions: 64)
Write errors:           0
Total transferred:      19246032 bytes
HTML transferred:       16921008 bytes
Requests per second:    1558.69 [#/sec] (mean)
Time per request:       641.562 [ms] (mean)
Time per request:       0.642 [ms] (mean, across all concurrent requests)
Transfer rate:          2929.56 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0  243 253.4    203    1310
Processing:    75  371 111.0    365     656
Waiting:        0  311 106.6    310     612
Total:        173  613 270.2    569    1752

Percentage of the requests served within a certain time (ms)
  50%    569
  66%    599
  75%    654
  80%    667
  90%    719
  95%   1479
  98%   1637
  99%   1718
 100%   1752 (longest request)
```

## Nginx 请求限制模块

- ngx_http_limit_req_module

```nginx
基于漏桶算法实现的请求限流模块，用于对指定KEY对应的请求进行限流，比如按照IP维度限制请求速率

limit_req : 配置限流区域、桶容量（突发容量，默认0）、是否延迟模式（默认延迟）
limit_req_log_level : 配置记录被限流后的日志级别，默认error级别
limit_req_status : 配置被限流后返回的状态码，默认返回503
limit_req_zone : 配置限流KEY、及存放KEY对应信息的共享内存区域大小、固定请求速率
    $binary_remote_addr
       变量，可以将每条状态记录的大小减少到64个字节，这样1M的内存可以保存大约1万6千个64字节的记录。
    zone=one:10m
       区域名称为one，大小为10m
    固定请求速率
       使用rate参数配置，支持10r/s和60r/m，即每秒10个请求和每分钟60个请求，不过最终都会转换
       为每秒的固定请求速率（10r/s为每100毫秒处理一个请求；60r/m，即每1000毫秒处理一个请求）


控制Nginx访问量

	1、连接池
		limit_req_zone $remote_addr zone=one:10m rate=1r/s;
		声明连接池       变量          名称 连接池的大小  速率

	2、限制数

案例：要求每秒只能有一个访问。
[root@web01 ~]# vim /etc/nginx/conf.d/game.conf
[root@web01 ~]# cat /etc/nginx/conf.d/game.conf
limit_req_zone $remote_addr zone=one:10m rate=1r/s;
limit_req_log_level error;
limit_req_status 503;
server {
    listen 80;
    server_name game.test.com;
    limit_req zone=one burst=5;
    location / {
        root /opt/Super_Marie;
        index index.html;
    }
}

[root@web01 down]# nginx -s reload
```

![image-20220104210224831](https://gitee.com/gengff/blogimage/raw/master/images/image-20220104210224831.png)
