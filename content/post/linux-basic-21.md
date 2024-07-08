---
title: Nginx动静分离
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Nginx
categories:
  - Linux
url: post/linux-21.html
toc: true
---

# 动静分离

- 在弄清动静分离之前，我们要先明白什么是动，什么是静。

  在 Web 开发中，通常来说，动态资源其实就是指那些后台资源，而静态资源就是指 HTML，JavaScript，CSS，img 等文件。

  一般来说，都需要将动态资源和静态资源分开，将静态资源部署在 Nginx 上，当一个请求来的时候，如果是静态资源的请求，就直接到 nginx 配置的静态资源目录下面获取资源，如果是动态资源的请求，nginx 利用反向代理的原理，把请求转发给后台应用去处理，从而实现动静分离。

  在使用前后端分离之后，可以很大程度的提升静态资源的访问速度，同时在开过程中也可以让前后端开发并行可以有效的提高开发时间，也可以有些的减少联调时间 。
  <!-- more -->

```nginx
1、创建NFS挂载点（在nfs服务器上进行）
  ①.创建并授权static目录
   [root@nfs ~]# mkdir /static
   [root@nfs ~]# chown -R www.www /static/

  ②.增加挂载点（编辑/etc/exports配置文件）
   [root@nfs ~]# vim /etc/exports
    /static      172.16.1.0/20(rw,sync,all_squash,anonuid=666,anongid=666)

  ③.重启nfs服务端
   [root@nfs ~]# systemctl restart nfs-server

  ④.检查服务端是否正常
   [root@nfs ~]# showmount -e
    /static 172.16.1.0/20



2、将web01服务器的静态资源放置于挂载点内（在web01服务器上进行）
  ①.创建目录
   [root@web01 ~]# mkdir /opt/static

  ②.挂载
   [root@web01 ~]# mount -t nfs 172.16.1.31:/static /opt/static/

  ③.将项目的静态资源放置于挂载点内
   [root@web01 ~]# cp -r /opt/bbs/static/* /opt/static/


3、挂载到lb01(在代理服务器上进行操作)
  ①.安装nfs-utils
   [root@lb01 ~]# yum install nfs-utils -y
  ②.创建挂载点
   [root@lb01 ~]# mkdir /opt/static/
  ③.挂载
   [root@lb01 ~]# mount -t nfs 172.16.1.31:/static /opt/static/
   [root@lb01 ~]# df -h

  ④.修改nginx配置文件
   [root@lb01 ~]# vim /etc/nginx/conf.d/test.conf
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
    location /static {
        alias /opt/static;
    }

}

  ⑤.测试重载nginx
   [root@lb01 ~]# nginx -t
   [root@lb01 ~]# systemctl restart nginx


4、测试
 浏览器访问：bbs.test.com
```

# Rewrite

- Rewrite 主要实现 url 地址重写，以及重定向，就是把传入 web 的请求重定向到其他 url 的过程。

## Rewrite 基本概述

```bash
1.地址跳转，用户访问www.linux.com这个URL是，将其定向至一个新的域名www.baidu.com。
2.协议跳转，用户通过http协议请求网站时，将其重新跳转至https协议方式。
3.伪静态，将动态页面显示为静态页面方式的一种技术，便于搜索引擎的录入，同时建上动态URL地址对外暴露过多的参数，提升更高的安全性。
4.搜索引擎，SEO优化依赖于url路径，好记的url便于搜索引擎录入。
```

## rewrite 语法

```bash
Syntax: rewrite regex replacement [flag];
Default:    —
Context:    server, location, if

rewrite         # 模块命令
regex           # 请求的链接（支持正则表达式）
replacement     # 跳转的链接
[flag];         # 标签

location /download/ {
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 break;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
}
```

## Rewrite 标记 Flag

- rewrite 指令根据表达式来重定向 URL，或者修改字符串，可以应用于 server，location，if 环境下，每行 rewrite 指令最后跟一个 flag 标记，支持的 flag 标记有如下表格所示：

| flag      | 作用                                             |
| --------- | ------------------------------------------------ |
| last      | 本条规则匹配完成后，停止匹配，不再匹配后面的规则 |
| break     | 本条规则匹配完成后，停止匹配，不再匹配后面的规则 |
| redirect  | 返回 302 临时重定向，地址栏会显示跳转后的地址    |
| permanent | 返回 301 永久重定向，地址栏会显示跳转后的地址    |

### 1、last 和 break 的区别

```nginx
[root@lb01 ~]# cat /etc/nginx/conf.d/rewrite.conf

server {
    server_name _;
    listen 80;
    location ~ ^/break {
        rewrite (.*) /test break;
    }

    location ~ ^/last {
        rewrite (.*) /test last;
    }

    location /test {
        default_type text/html;
        return 200 "test";
    }
}

break 只要匹配到规则，则会去本地配置路径的目录中寻找请求的文件；
而last只要匹配到规则，会对其所在的server(...)标签重新发起请求。


break请求：
1.请求linux.rewrite.com/break
2.匹配 location ~ ^/break 会跳转到 linux.rewrite.com/test
3.请求跳转后，回去查找本地站点目录下的 /test
4.如果找到了，则返回/code/test/index.html的内容；
5.如果没找到该目录则报错404，如果找到该目录没找到对应的文件则403

last请求:
1.请求linux.rewrite.com/last
2.匹配 location ~ ^/last 会跳转到 linux.rewrite.com/test
3.如果找到了，则返回/code/test/index.html的内容；
4.如果没有找到，会重新对当前server发起请求，这个时候访问地址就变成 linux.rewrite.com/test
5.重新请求server会匹配到 location /test/ 直接返回该location的内容
6.如果也没有location匹配，再返回404;
```

### 2、redirect 和 permanent 的区别

```nginx
重定向
location /redirect {
	rewrite (.*) http://www.baidu.com redirect;
}
location /permanent {
    rewrite (.*) http://www.baidu.com permanent;
}
redirect: 每次请求都会询问服务器，如果当服务器不可用时，则会跳转失败。
permanent: 第一次请求会询问，浏览器会记录跳转的地址，第二次则不再询问服务器，直接通过浏览器缓存的地址跳转。
```

# HTTPS

- 为什么需要使用 HTTPS，因为 HTTP 不安全，当我们使用 http 网站时，会遭到劫持和篡改，如果采用 https 协议，那么数据在传输过程中是加密的，所以黑客无法窃取或者篡改数据报文信息，同时也避免网站传输时信息泄露。

- 那么我们在实现 https 时，需要了解 ssl 协议，但我们现在使用的更多的是 TLS 加密协议。

- 那么 TLS 是怎么保证明文消息被加密的呢？在 OSI 七层模型中，应用层是 http 协议，那么在应用层协议之下，我们的表示层，是 ssl 协议所发挥作用的一层，他通过（握手、交换秘钥、告警、加密）等方式，是应用层 http 协议没有感知的情况下做到了数据的安全加密

## 模拟网站劫持

### 1、正常的页面

```bash
# 1.新建html文件
[root@web01 ~]# mkdir /opt/code
[root@web01 ~]# cd /opt/code
[root@web01 ~]# vim index.html
#复制粘贴下方内容
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>学生信息注册页面</title>
</head>
<body>
<h3>学生信息注册</h3>
<form  name="stu"action="">
<table>
  <tr><td>姓名:</td><td><input type="text"name="stuName"/></td></tr>
  <tr><td>性别:</td>
      <td><input type="radio"name="stuSex"checked="checked">男
          <input type="radio"name="stuSex">女
          </td>
          </tr>
   <tr><td>出生日期</td>
       <td><input type="text"name="stuBirthday"></td>
       <td>按格式yyyy-mm-dd</td>
       </tr>
       <tr><td>学校:</td><td><input type="text"name="stuSchool"></td></tr>
       <tr><td>专业:</td>
           <td><select name="stuSelect2">
               <option selected>计算机科学与技术</option>
               <option>网络工程</option>
               <option>物联网工程</option>
               <option>应用数学</option>
               </select>
               </td>
               </tr>
               <tr><td>体育特长:</td>
                   <td colspan="2">
                      <input type="checkbox"name="stuCheck" >篮球
                      <input type="checkbox"name="stuCheck" >足球
                      <input type="checkbox"name="stuCheck" >排球
                      <input type="checkbox"name="stuCheck" >游泳
                   </td>
               </tr>
               <tr><td>上传照片:</td><td colspan="2"><input type="file" ></td></tr>
               <tr><td>密码:</td><td><input type="password"name="stuPwd" ></td></tr>
               <tr><td>个人介绍:</td>
                   <td colspan="2"><textarea name="Letter"rows="4"cols="40"></textarea></td>
               </tr>
               <tr>
                 <td><input type="submit"value="提交" ><input type="reset"value="取消" ></td>
                 </tr>
                 </table>
                 </form>
</body>
</html>


# 2.压缩下之前web01的配置，没有可以忽略
[root@web01 ~]# gzip /etc/nginx/conf.d/game.conf

# 3.创建新的配置文件
[root@web01 ~]# vim /etc/nginx/conf.d/http.conf

server {
    listen 80;
    server_name _;
    location / {
        root /opt/code;
        index index.html;
    }
}

# 4.测试重载nginx
[root@web01 ~]# nginx -t
[root@web01 ~]# systemctl restart nginx

# 5.浏览器访问：http://192.168.15.8/
```

![image-20220111040645557](https://gitee.com/gengff/blogimage/raw/master/images/image-20220111040645557.png)

### 2、网站劫持

```bash
# 将之前的nginx代理负载均衡配置压缩（没有可忽略该操作）
[root@lb01 ~]# gzip /etc/nginx/conf.d/test.conf
# 创建新的nginx配置文件
[root@lb01 ~]# vim /etc/nginx/conf.d/http.conf
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://192.168.15.8;

        sub_filter '<title>学生信息注册页面</title>' '<title>澳门首家线上赌场</title>';
        sub_filter '<h3 align="center">学生信息注册</h3>' '<h3 align="center">VIP用户信息注册</h3>';
        sub_filter '<tr><td>性别:</td>' '<tr><td>爱好:</td>';
        sub_filter '<option selected>计算机科学与技术</option>' '<option selected>按摩</option>';
        sub_filter '<option>网络工程</option>' '<option>抽烟</option>';
        sub_filter '<option>物联网工程</option>' '<option>喝酒</option>';
        sub_filter '<option>应用数学</option>' '<option>烫头</option>';
        sub_filter '<tr><td>上传照片:</td><td colspan="2"><input type="file" ></td></tr>' '<img src="https://blog.driverzeng.com/zenglaoshi/xingganheguan.gif">';
    }
}

# 重启nginx服务
[root@lb01 ~]# systemctl restart nginx

# 测试
浏览器访问：http://192.168.15.5/
```

![image-20220111044404939](https://gitee.com/gengff/blogimage/raw/master/images/image-20220111044404939.png)

## 加密流程

- 1、浏览器发起往服务器的 443 端口发起请求，请求携带了浏览器支持的加密算法和哈希算法。
- 2、服务器收到请求，选择浏览器支持的加密算法和哈希算法。
- 3、服务器下将数字证书返回给浏览器，这里的数字证书可以是向某个可靠机构申请的，也可以是自制的。
- 4、浏览器进入数字证书认证环节，这一部分是浏览器内置的 TLS 完成的：
  - 4.1 首先浏览器会从内置的证书列表中索引，找到服务器下发证书对应的机构，如果没有找到，此时就会提示用户该证书是不是由权威机构颁发，是不可信任的。如果查到了对应的机构，则取出该机构颁发的公钥。
  - 4.2 用机构的证书公钥解密得到证书的内容和证书签名，内容包括网站的网址、网站的公钥、证书的有效期等。浏览器会先验证证书签名的合法性（验证过程类似上面 Bob 和 Susan 的通信）。签名通过后，浏览器验证证书记录的网址是否和当前网址是一致的，不一致会提示用户。如果网址一致会检查证书有效期，证书过期了也会提示用户。这些都通过认证时，浏览器就可以安全使用证书中的网站公钥了。
  - 4.3 浏览器生成一个随机数 R，并使用网站公钥对 R 进行加密。
- 5、浏览器将加密的 R 传送给服务器。
- 6、服务器用自己的私钥解密得到 R。
- 7、服务器以 R 为密钥使用了对称加密算法加密网页内容并传输给浏览器。
- 8、浏览器以 R 为密钥使用之前约定好的解密算法获取网页内容。

## 证书对比

| 对比         | 域名型 DV                                                           | 企业型 OV                                                           | 增强型 EV                                                                                  |
| ------------ | ------------------------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 绿色地址栏   | ![img](https://blog.driverzeng.com/zenglaoshi/02.jpg)小锁标记+https | ![img](https://blog.driverzeng.com/zenglaoshi/03.jpg)小锁标记+https | ![img](https://gitee.com/gengff/blogimage/raw/master/images/04.jpg)小锁标记+企业名称+https |
| 一般用途     | 个人站点和应用； 简单的 https 加密需求                              | 电子商务站点和应用； 中小型企业站点                                 | 大型金融平台； 大型企业和政府机构站点                                                      |
| 审核内容     | 域名所有权验证                                                      | 全面的企业身份验证； 域名所有权验证                                 | 最高等级的企业身份验证； 域名所有权验证                                                    |
| 颁发时长     | 10 分钟-24 小时                                                     | 3-5 个工作日                                                        | 5-7 个工作日                                                                               |
| 单次申请年限 | 1 年                                                                | 1-2 年                                                              | 1-2 年                                                                                     |
| 赔付保障金   | ——                                                                  | 125-175 万美金                                                      | 150-175 万美金                                                                             |

## 自签证书

```bash

[root@web01 ~]# cd /etc/nginx/
[root@web01 nginx]# mkdir ssl_key
[root@web01 nginx]# cd ssl_key

#使用openssl命令充当CA权威机构创建证书（生产不使用此方式生成证书，不被互联网认可的黑户证书）
[root@web01 ssl_key]# openssl genrsa -idea -out server.key 2048
Generating RSA private key, 2048 bit long modulus
..............+++
..................................+++

e is 65537 (0x10001)
Enter pass phrase for server.key: 123456
Verifying - Enter pass phrase for server.key: 123456

[root@web01 ssl_key]# ll
total 4
-rw-r--r--. 1 root root 1739 Dec  9 11:27 server.key

#生成自签证书(公钥)，同时去掉私钥的密码
[root@web01 ssl_key]# openssl req -days 36500 -x509 -sha256 -nodes -newkey rsa:2048 -keyout server.key -out server.crt
Generating a 2048 bit RSA private key
.....................................+++
............+++
writing new private key to 'server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:china
string is too long, it needs to be less than  2 bytes long
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:meiguo
Locality Name (eg, city) [Default City]:riben
Organization Name (eg, company) [Default Company Ltd]:heishoudang
Organizational Unit Name (eg, section) []:oldboy
Common Name (eg, your name or your server's hostname) []:oldboy
Email Address []:123@qq.com

# req  --> 用于创建新的证书
# new  --> 表示创建的是新证书
# x509 --> 表示定义证书的格式为标准格式
# key  --> 表示调用的私钥文件信息
# out  --> 表示输出证书文件信息
# days --> 表示证书的有效期
# sha256 --> 加密方式

#1.开启证书
Syntax: ssl on | off;
Default:    ssl off;
Context:    http, server

#2.指定证书文件
Syntax: ssl_certificate file;
Default:    —
Context:    http, server

#3.指定私钥文件
Syntax: ssl_certificate_key file;
Default:    —
Context:    http, server

#4.修改nginx配置文件(如果负载均衡配置了就不需要下面的配置了)
[root@web01 ~]# cat /etc/nginx/conf.d/https.conf
server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/nginx/ssl_key/server.crt;
    ssl_certificate_key /etc/nginx/ssl_key/server.key;

    location / {
        root /opt/code;
        index index.html;
    }
}

#5.重启nignx服务
[root@lb01 ~]# systemctl restart nginx

#6.测试
浏览器访问：https://192.168.15.8/
```

#### 证书加载到代理

```bash
# 将web01的ssl配置同步到代理lb01
[root@web01 conf.d]# scp https.conf 192.168.15.5:/etc/nginx/conf.d/https.conf
[root@web01 conf.d]# scp /etc/nginx/ssl_key 192.168.15.5:/etc/nginx/conf.d/https.conf

# 修改lb01的nginx配置文件
[root@lb01 conf.d]# cat /etc/nginx/conf.d/https.conf
upstream ssl {
    server 172.16.1.7;
    server 172.16.1.8;
    server 172.16.1.9;
}

server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/nginx/ssl_key/server.crt;
    ssl_certificate_key /etc/nginx/ssl_key/server.key;

    location / {
        proxy_pass http://ssl;
        include /etc/nginx/proxy_params;
    }
}

server {
    listen 80;
    server_name _;
    rewrite (.*) https://192.168.15.5 permanent;
}

# 测试重启nignx服务
[root@lb01 ~]# nginx -t
[root@lb01 ~]# systemctl restart nginx

# 测试
浏览器访问：https://192.168.15.5/
```
