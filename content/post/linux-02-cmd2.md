---
title: Linux-命令记录-02
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-cmd-02.html
toc: true
---

### 去除文件空行和#开头的行

```bash
grep ^[^#] file
grep -Ev "^$|[#;]"  file
egrep -v "^$|#"  file
```

<!-- more -->

### 修改文件最大打开数

```bash
ulimit -n 1048576
sed -i "/nofile/d" /etc/security/limits.conf
echo "* hard nofile 1048576" >> /etc/security/limits.conf
echo "* soft nofile 1048576" >> /etc/security/limits.conf
echo "root hard nofile 1048576" >> /etc/security/limits.conf
echo "root soft nofile 1048576" >> /etc/security/limits.conf
```

### 优化 ssh

```bash
vim /etc/ssh/sshd_config
Port 52113                 #10000以上的端口
PermitRootLogin no         #禁止root远程登录
PermitEmptyPasswords no    #禁止空密码登录
UseDNS no                  #不使用解析。
GSSAPIAuthentication no    #连接慢的解决配置。

#检查sshd语法
sshd -t

#重启ssh
service sshd restart
```

### Centos 常用包

```bash
yum install -y libxml2 libxml2-devel openssl \
openssl-devel bzip2 bzip2-devel libcurl \
libcurl-devel libjpeg libjpeg-devel \
libpng libpng-devel freetype freetype-devel \
gmp gmp-devel libmcrypt libmcrypt-devel \
readline readline-devel libxslt libxslt-devel  \
libicu-devel  openldap  openldap-devel \
make zlib zlib-devel gcc-c++ libtool \
pcre pcre-devel  cmake gcc  ncurses ncurses-devel \
bison bison-devel libgcrypt perl
```
