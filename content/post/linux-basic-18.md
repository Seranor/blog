---
title: Nginx-localtion配置和LNMP架构
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Nginx
categories:
  - Linux
url: post/linux-18.html
toc: true
---

# Nginx 的 location 配置

<!-- more -->

```bash
使用Nginx location可以控制访问网站的路径, 但一个server可以有多个location配置, 多个location的优先级该如何区分。

location 的匹配顺序其实是“先匹配普通，再匹配正则”。
   nginx 是“先匹配正则 location 再匹配普通 location ”，其实这是一个误区， nginx 其实
   是“先匹配普通 location ，再匹配正则 location ”。
   但是普通 location 的匹配结果又分两种：
     一种是“严格精确匹配”，官方英文说法是“ 准确匹配 ”；
     一种是“最大前缀匹配”，官方英文说法是“ 字面值字符串匹配开始部分 查询的-将使用最具体的匹配 ”。

location 匹配的优先级(与location在配置文件中的顺序无关)
  = 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。
  普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有
  没有正则表达式匹配和更长的匹配。
  ^~ 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。最后匹配
  带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有
  正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用
```

#### location 语法

```bash
Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default:	—
Context:	server, location
```

#### location 匹配符号

| **匹配符** | **匹配规则**                       | **优先级** |
| ---------- | ---------------------------------- | ---------- |
| =          | 精确匹配                           | 1          |
| ^~         | 以某个字符串开头，一般用来匹配目录 | 2          |
| ~          | 区分大小写的正则匹配               | 3          |
| ~\*        | 不区分大小写的正则匹配             | 4          |
| /          | 通用匹配，任何请求都会匹配到       | 5          |

#### 优先级认证

```nginx
示例：
[root@web01 ~]# cat /etc/nginx/conf.d/text.conf
server {
    listen 80;
    server_name 192.168.15.8;

    location ~* /python {
        default_type text/html;
        return 200 "Location ~*";
    }

    location ~ /Python {
        default_type text/html;
        return 200 "Location ~";
    }

    location ^~ /python {
        default_type text/html;
        return 200 "Location ^~";
    }

    location = /python {
        default_type text/html;
        return 200 "Location =";
    }
}

案例：
[root@web01 ~]# cat /etc/nginx/conf.d/text1.conf
server {
   listen 80;
   server_name 192.168.15.8;
	# 访问站点根目录
	 location / {
		root /usr/share/nginx/html;
		index index.html;
	 }

	# 访问图片
	 location ~* \.(jpg|gif|png|jpeg) {
	 deny all;
	 }
	# 访问监控，需要输入用户名密码
	 location =/status {
		auth_basic           "closed site";
		auth_basic_user_file /etc/nginx/auth_basic;
		stub_status;
	 }
}
```

#### location 应用场景

```bash
#通用匹配，任何请求都会匹配到
location / {
    ...
}

#严格区分大小写，匹配以.py结尾的都走这个location
location ~ \.py$ {
    ...
}

#严格区分大小写，匹配以.html结尾的都走这个location
location ~ \.html$ {
    ...
}

#不区分大小写匹配，只要用户访问.jpg,gif,png,js,css结尾的都走这条location
location ~* .*\.(jpg|gif|png|js|css)$ {
    ...
}

#不区分大小写匹配
location ~* "\.(sql|bak|tgz|tar.gz|.git)$" {
    ...
}
```

# uWSGI

- 此次 LNMP 架构采用 Linux + Nginx + MySQL + Python，在学习之前我们需要先了解下 uWSGI 的知识。

- **uWSGI 是一个 Web 服务器，它实现了 WSGI 协议、uwsgi 协议 和 http 服务协议**

```bash
WebApp采用 Python 的web框架 Django 开发

因为Nginx不支持WSGI协议，无法直接调用 Python 开发的WebApp。所以需要借助uWSGI,
在 Nginx + uWSGI + Django 的框架里，nginx代理+WebServer，uWSGI是WSGI server，
Django是webApp。Nginx接收用户请求，并判定哪些转发到uWSGI，uWSGI再去调用pyWebApp。

由于apache、nginx等，它们自己都没有解析动态语言如php的功能，而是分派给其他模块来做，比如apache就
可以说是内置模块，让人感觉apache就支持php一样
```

- **这里一定要注意 WSGI , uwsgi , uWSGI 这三个概念的区分:**

  - **WSGI**: 是一种接口标准协议/规范，实现了 python web 程序与 web 服务器之间交互的通用性。利用它 django 等 python web 开发框架就可以部署不同的 web server 上了，目的是制定标准，以保证不同 Web 服务器可以和不同的 Python 程序之间相互通信

  - **uwsgi**: 是一种线路协议而不是通信协议，在此常用于在 uWSGI 服务器与其他网络服务器的数据通信。

  - **uWSGI**：是基于自有 uwsgi 协议、WSGI 协议和 http 服务协议的 web 网关或服务器。负责响应 python 的 web 请求。

    [参考文档](https://blog.csdn.net/weixin_44826484/article/details/108588997)

- uwsgi 协议是一个 uWSGI 服务器自有的协议，它用于定义传输信息的类型，每一个 uwsgi packet 前 4byte 为传输信息类型描述，它与 WSGI 相比是两样东西。uwsgi 是一种线路协议而不是通信协议，在此常用于在 uwsgi 服务器与其他网络服务器的数据通信。uwsgi 协议是一个 uwsgi 服务器自有的协议，它用于定义传输信息的类型。
- uwsgi 实现了 WSGI 的所有接口，是一个快速、自我修复、开发人员和系统管理员友好的服务器。uwsgi 代码完全
  用 C 编写，效率高、性能稳定。

![3173921](https://gitee.com/gengff/blogimage/raw/master/images/3173921.jpg)

# LNMP 架构

- LNMP 是一套技术的组合，L=Linux、N=Nginx、M=MySQL、P=Python

  - Linux 是一类 Unix 计算机操作系统的统称，是目前最流行的免费操作系统。

  - Nginx 是一个高性能的 HTTP 和反向代理服务器，也是一个 IMAP/POP3/SMTP 代理服务器。

  - Mysql 是一个小型关系型数据库管理系统。

  - Python-Django 是一个开源的 web 应用框架。采用了 MVC 的软件设计模式，即模型 M，视图 V 和控制器 C。

    这四种软件均为免费开源软件，组合到一起，成为一个免费、高效、扩展性强的网站服务系统。

## LNMP 工作流程

- 首先 Nginx 服务是不能处理动态请求，那么当用户发起动态请求时, Nginx 又是如何进行处理的。

  - 1.静态请求：请求的内容是静态文件就是静态请求
    - 1）静态文件：文件上传到服务器，永远不会改变的文件就是静态文件
    - 2）html 就是一个标准的静态文件
  - 2.动态请求：请求的内容是动态的就是动态请求
    - 1）不是真实存在服务器上的内容，是通过数据库或者其他服务拼凑成的数据

- 当用户发起 http 请求，请求会被 Nginx 处理，如果是静态资源请求 Nginx 则直接返回，如果是动态请求 Nginx 则通过 uwsgi 协议转交给后端的 Python 程序处理

## LNMP 访问流程

```bash
1、客户端(PC)向web服务器发起http请求服务资源

2、nginx作为直接对外的服务接口，接收到客户端发送过来的http请求，会解包、分析，如果是静态文件
   请求则根据nginx配置的静态文件目录，并返回请求的资源给客户端,不是则通过nginx就通过配置文件，
   将请求传递给uWSGI；uWSGI 将接收到的包进行处理，并转发给WSGI，WSGI协议将请求丢给web框架
   (django)代码处理

3、看web框架是否启动django中间件，如果启用，则依据中间件对请求进行修改，如果不启用，则进入下一步

4、web框架中的路由程序将根据请求中的url文件名将请求路由至相应py文件

5、相应py文件收到请求后根据用户提交的参数进行计算(期间可能会调用数据库)，然后返回计算后的结果和自
   定义头部信息以及状态码返回

6、web框架将返回的数据打上通用标识符(头部信息)后返回给WSGI

7、WSGI将返回数据进行打包，转发给uWSGI，uWSGI接收后转发给nginx，nginx最终将返回值返回给客户端(PC)。

8、客户端收到返回的数据
```

![720056](https://gitee.com/gengff/blogimage/raw/master/images/720056.png)

# LNMP 搭建

| 服务器 | IP           |
| :----- | :----------- |
| lb01   | 192.168.15.5 |
| db01   | 192.168.15.7 |
| web01  | 192.168.15.8 |

## uwsgi 服务部署

```bash
1、创建用户
[root@web01 ~]# cd /opt
[root@web01 opt]# groupadd django -g 888
[root@web01 opt]# useradd django -u 888 -g 888 -r -M -s /bin/sh

2、安装python依赖软件
[root@web01 opt]# yum install python3 libxml* python-devel gcc* pcre-devel openssl-devel python3-devel -y

3、安装Django和uwsgi
[root@web01 opt]# pip3 install django
[root@web01 opt]# pip3 install uwsgi

4、创建项目：linux
[root@web01 opt]# django-admin startproject linux

# 创建app
[root@web01 opt]# cd linux
[root@web01 linux]# django-admin startapp app01

# 修改配置文件
[root@web01 linux]# vim linux/settings.py
 ALLOWED_HOSTS = ['*']  #允许所有IP都可以访问
 DATABASES = {}  # 将数据库配置置空
# 启动测试
[root@web01 linux]# python3 manage.py runserver 0.0.0.0:8000


5、编辑项目配置文件
[root@web01 linux]# vim /opt/linux/myweb_uwsgi.ini
[uwsgi]
# 端口号
socket = :8000
# 指定项目的目录
chdir = /opt/linux
# wsgi文件路径
wsgi-file = linux/wsgi.py
# 模块wsgi路径
module = linux.wsgi
# 是否开启master进程
master = true
# 工作进程的最大数目
processes = 4
# 结束后是否清理文件
vacuum = true

6、后台启动uwsgi(不建议用root运行，所以指定给其他用户）
[root@web01 linux]# uwsgi -d --ini myweb_uwsgi.ini --uid 666

-d 	  : 以守护进程方式运行
--ini : 指定配置文件路径
--uid : 指定uid

# 用该方式查看uwsgi是否在后台运行
[root@web01 linux]# ps -ef | grep uwsgi

# 此时的 uwsgi 是一个TCP服务，包含http，但是http不包含TCP服务
# 需要用nginx 将uwsgi服务转换成http服务来实现通信

7、编辑Nginx配置文件
[root@web01 linux]# cat /etc/nginx/conf.d/python.conf
server {
    listen 80;
    server_name py.test.com;
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8000;
        uwsgi_read_timeout 2;
        uwsgi_param UWSGI_SCRIPT linux.wsgi;
        uwsgi_param UWSGI_CHDIR /opt/linux;
        index  index.html index.htm;
        client_max_body_size 35m;
    }
}

8、重启Nginx配置
[root@web01 linux]# systemctl restart nginx
```

## BBS 项目部署

### 部署数据库

```bash
1、安装数据库
[root@db01 ~]# yum install mariadb* -y

2、启动数据库
[root@db01 ~]# systemctl start mariadb

3、远程连接MySQL数据
# 数据库添加用户语句：创建用户，授权（全部数据库权限）
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

# 刷新权限
MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

# 创建数据库 bbs ，将该库的默认编码格式设置为utf8格式，数据库校对规则
MariaDB [(none)]> CREATE DATABASE `bbs` DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
Query OK, 1 row affected (0.00 sec)


# 数据库校对规则，utf8_bin将字符串中的每一个字符用二进制数据存储，区分大小写。
# utf8_genera_ci不区分大小写，ci为case insensitive的缩写，即大小写不敏感。
# utf8_general_cs区分大小写，cs为case sensitive的缩写，即大小写敏感。
```

### 部署 BBS

```bash
1、上传代码
[root@db01 ~]# unzip bbs.zip
# 将解压后的项目移动到opt目录
[root@db01 ~]# mv bbs /opt/


2、数据库迁移
# 切换到项目app的migrations目录
[root@db01 ~]# cd /opt/bbs/app01/migrations
[root@web01 migrations]# pwd
 /opt/bbs/app01/migrations

# 删除数据库迁移配置
[root@web01 migrations]# rm -rf 00*
# 删除数据库缓存
[root@web01 migrations]# rm -rf __pycache__/

[root@web01 migrations]# cd /opt/bbs/
[root@web01 bbs]# pwd
/opt/bbs

# 根据报错信息判断是否修改Django版本,如果版本对应无需卸载安装新版本
[root@web01 bbs]# pip3 uninstall django
[root@web01 bbs]# pip3 install django==1.11

# 安装MySQL数据库插件
[root@web01 bbs]# pip3 install pymysql

# 安装项目所需要的模块和依赖

# 修改数据连接
[root@web01 bbs]# vim bbs/settings.py
ALLOWED_HOSTS = ['*']
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', #数据库
        'NAME': 'bbs',          #数据库名
        'USER': 'root',         #数据库用户名
        'PASSWORD': '123456',   #数据库密码
        'HOST': '192.168.15.7',  #数据库ip地址
        'PORT': 3306,           #数据库端口号
        'CHARSET': 'utf8'       #数据编码格式
    }
}

# 创建数据库迁移文件
[root@web01 bbs]# python3 manage.py makemigrations

# 数据库迁移
[root@web01 bbs]# python3 manage.py migrate



3、配置并启动uwsgi
[root@web01 bbs]# vim /opt/bbs/myweb_uwsgi.ini
[root@web01 bbs]# cat /opt/bbs/myweb_uwsgi.ini
[uwsgi]
# 端口号
socket = :8002
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

# 启动uwsgi
[root@web01 bbs]# uwsgi -d --ini myweb_uwsgi.ini --uid 666

4、配置Nginx
[root@web01 bbs]# cat /etc/nginx/conf.d/python.conf
server {
    listen 80;
    server_name bbs.test.com;
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8002;
        uwsgi_read_timeout 2;
        uwsgi_param UWSGI_SCRIPT bbs.wsgi;
        uwsgi_param UWSGI_CHDIR /opt/bbs;
        index  index.html index.htm;
        client_max_body_size 35m;
    }
    location /s {
                alias /opt/bbs/static;

    }
}

[root@web01 bbs]# systemctl restart nginx  #重启nginx


5、测试访问BBS

浏览器输入 bbs.test.com
```
