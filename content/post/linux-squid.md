---
title: "Squid实现代理上网"
date: 2024-07-18T15:14:25+08:00
lastmod: 2024-07-18T15:14:25+08:00

categories:
  - Squid
tags:
  - Squid
  - Linux

url: post/linux-squid.html
toc: true
---

### Squid
说明
squid是一款高效的http代理服务器程序，而且更经常被用来做缓存服务器。
官网：http://www.squid-cache.org

这里是给纯内网机器实现上网功能
<!--more-->

### 环境
系统: Ubuntu20.0.4

上网机器

ens33 可以上网

ens34 纯内网
```bash
root@out-test:~# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:79:d3:4e brd ff:ff:ff:ff:ff:ff
    inet 172.0.0.100/24 brd 172.0.0.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe79:d34e/64 scope link 
       valid_lft forever preferred_lft forever
3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:79:d3:58 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.100/24 brd 192.168.0.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe79:d358/64 scope link 
       valid_lft forever preferred_lft forever
```
没有做NAT转发
```bash
root@out-test:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination  
```


内网机器
```bash
root@in-test:~# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:73:bd:da brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.200/24 brd 192.168.0.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe73:bdda/64 scope link 
       valid_lft forever preferred_lft forever
```
网卡配置
```bash
root@in-test:~# cat /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    ens34:
      dhcp4: no
      addresses:
        - 192.168.0.200/24
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
  version: 2
```
### 安装squid
上网机器
```bash
# 安装
apt-get install squid3

# 查看版本
squid3 -v
```

### 配置文件
```bash
cat /etc/squid/squid.conf
# 定义本地网络范围为192.168.0.0/24
acl localnet src 192.168.0.0/24

# 允许本地网络和 localhost 访问代理
http_access allow localnet
http_access allow localhost

# 禁止所有其他访问
http_access deny all

# Squid 监听的端口
http_port 3128

# 允许 SSL/HTTPS
acl SSL_ports port 443
acl Safe_ports port 80      # http
acl Safe_ports port 443     # https
acl Safe_ports port 1025-65535  # unregistered ports
acl CONNECT method CONNECT
http_access allow CONNECT SSL_ports
```

重启
```bash
systemctl restart  squid.service
```
### 使用
内网机器
```bash
# 写入到 /etc/profile
export http_proxy="http://192.168.0.100:3128"
export https_proxy="http://192.168.0.100:3128"
```

### 效果
内网机器
```bash
root@in-test:~# curl https://www.baidu.com
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta conss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.cden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input t=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 clahttp://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baij_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponenz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=f <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a 30173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```
外网机器
```bash
root@out-test:~# tail -f /var/log/squid/access.log 
1721285751.863    257 192.168.0.200 TCP_REFRESH_UNMODIFIED/304 426 GET http://archive.ubuntu.com/ubuntu/dists/focal-updates/InRelease - HIER_DIRECT/91.189.91.81 -
1721285752.124    260 192.168.0.200 TCP_MISS/304 420 GET http://archive.ubuntu.com/ubuntu/dists/focal-backports/InRelease - HIER_DIRECT/91.189.91.81 -
1721285752.428    302 192.168.0.200 TCP_MISS/304 420 GET http://archive.ubuntu.com/ubuntu/dists/focal-security/InRelease - HIER_DIRECT/91.189.91.81 -
1721285773.438   2393 192.168.0.200 TCP_MISS/200 1243740 GET http://archive.ubuntu.com/ubuntu/pool/main/v/vim/vim_8.1.2269-1ubuntu5.23_amd64.deb - HIER_DIRECT/91.189.91.81 application/vnd.debian.binary-package
1721285775.823   2383 192.168.0.200 TCP_MISS/200 582202 GET http://archive.ubuntu.com/ubuntu/pool/main/v/vim/vim-tiny_8.1.2269-1ubuntu5.23_amd64.deb - HIER_DIRECT/91.189.91.81 application/vnd.debian.binary-package
1721285778.603   2779 192.168.0.200 TCP_MISS/200 5880456 GET http://archive.ubuntu.com/ubuntu/pool/main/v/vim/vim-runtime_8.1.2269-1ubuntu5.23_all.deb - HIER_DIRECT/91.189.91.81 application/vnd.debian.binary-package
1721285778.895    292 192.168.0.200 TCP_MISS/200 88349 GET http://archive.ubuntu.com/ubuntu/pool/main/v/vim/vim-common_8.1.2269-1ubuntu5.23_all.deb - HIER_DIRECT/91.189.91.81 application/vnd.debian.binary-package
1721286116.269   1362 192.168.0.200 TCP_TUNNEL/200 8247 CONNECT www.baidu.com:443 - HIER_DIRECT/103.235.47.188 -
1721286549.203    356 192.168.0.200 TCP_TUNNEL/200 8218 CONNECT www.baidu.com:443 - HIER_DIRECT/183.2.172.42 -
1721287417.978    370 192.168.0.200 TCP_TUNNEL/200 8247 CONNECT www.baidu.com:443 - HIER_DIRECT/183.2.172.42 -
```
还能使用 apt 进行安装软件等
