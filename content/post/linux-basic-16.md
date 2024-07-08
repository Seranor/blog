---
title: Rsync使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Rsync
categories:
  - Linux
url: post/linux-16.html
toc: true
---

# rsync 同步工具

- rsync (remote synchronizetion) 一款开源的快速的，多功能的，可实现全量及增量（差异化备份）的本地或远程数据备份的优秀工具
  <!-- more -->
  ![v2-6f26d816c7a3d517e75d8f4187d2c5b5_1440w](https://gitee.com/gengff/blogimage/raw/master/images/v2-6f26d816c7a3d517e75d8f4187d2c5b5_1440w.jpg)

## rsync 介绍

- rsync，从软件的名称就可以看出来，rsync 具有可使本地和远程两台主机之间的数据快速复制同步镜像、远程备份的功能，这个功能类似于 ssh 带的 scp 命令，但是又优于 scp 命令的功能，scp 每次都是全量拷贝，而 rsync 可以增量拷贝。当然，rsync 还可以在本地主机的不同分区或目录之间全量及增量的复制数据，这又类似 cp 命令。但是同样也优于 cp 命令，cp 每次都是全量拷贝，而 rsync 可以增量拷贝。

```bash
rsync官方地址：https://rsync.samba.org/
rsync监听端口：873
rsync运行模式：C/S   client/server

rsync简称叫做远程同步，可以实现不同主机之间的数据同步，还支持全量和增量
```

## rsync 特性

- 1)支持拷贝特殊文件，如链接文件、设备等。
- 2)可以有排除指定文件或目录同步的功能，相当于打包命令 tar 的排除功能。
- 3)可以做到保持原文件或目录的权限、时间、软硬链接、属主、组等所有属性均不改变 –p。
- 4)可以实现增量同步，既只同步发生变化的数据，因此数据传输效率很高（tar-N）。
- 5)可以使用 rcp、rsh、ssh 等方式来配合传输文件（rsync 本身不对数据加密）。
- 6)可以通过 socket（进程方式）传输文件和数据（服务端和客户端）。
- 7)支持匿名的活认证（无需系统用户）的进程模式传输，可以实现方便安全的进行数据备份和镜像。

## rsync 应用场景

```bash
全量备份
增量备份

两台服务器之间数据同步。
把所有客户服务器数据同步到备份服务器，生产场景集群架构服务器备份方案。
rsync结合inotify的功能做实时的数据同步。
```

## rsync 的传输方式

```bash
push 推：
  客户端将数据从本地推送至服务端(rsync服务器主动推送数据给其他主机。服务器开销大，适合后端服务器少的情况)

pull 拉：
  客户端将数据从服务端拉取到本地(客户端主动向rsync服务器拉取数据)
```

## rsync 传输模式

```bash
1.本地方式（类似于cp，不支持推送和拉取，只是单纯的复制）
2.远程方式（类似于scp，又不同于scp），scp只支持全量备份，rsync支持增量备份和差异备份
3.守护进程（socket）方式（客户端和服务端）
```

## rsync 使用

```bash
-a   #归档模式传输, 等于-tropgDl    -t -r -o -p -g -D -l
-v   #详细模式输出, 打印速率, 文件数量等
	[root@m01 ~]# rsync -v ./2.txt  root@172.16.1.41:/opt/
-z   #传输时进行压缩以提高效率
	[root@m01 ~]# rsync -vz ./2.txt  root@172.16.1.41:/opt/
-r   #递归传输目录及子目录，即目录下得所有目录都同样传输。
	[root@m01 ~]# rsync -vzr ./a  root@172.16.1.41:/opt/
-t   #保持文件时间信息
	[root@m01 ~]# rsync -vzrt ./a/b/c/2.txt  root@172.16.1.41:/opt/
-o   #保持文件属主信息
-g   #保持文件属组信息
	[root@m01 ~]# rsync -vzrtgo  ./a/b/c/2.txt  root@172.16.1.41:/opt/
-p   #保持文件权限
	[root@m01 ~]# rsync -vzrtgop  ./a/b/c/2.txt  root@172.16.1.41:/opt/
-l   #保留软连接
	[root@m01 ~]# rsync -vzrtgopl  ./*  root@172.16.1.41:/opt/
-P   #显示同步的过程及传输时的进度等信息
	[root@m01 ~]# rsync -vzrtgoplP  ./*  root@172.16.1.41:/opt/
-D   #保持设备文件信息
	[root@m01 dev]# rsync -vzrtgDopl /dev/tty1   root@172.16.1.41:/opt/
-L   #保留软连接指向的目标文件
-e   #使用的信道协议,指定替代rsh的shell程序

--append          # 指定文件接着上次传输中断处继续传输
--append-verify   # 使用参数续传（在断点续传之后，验证一下文件，如果不同，修复文件）

--exclude=PATTERN   # 指定排除不需要传输的文件
	[root@m01 ~]# rsync -avzP --append-verify --exclude=2.txt  ./* root@172.16.1.41:/opt/

--exclude-from=file # 按照文件指定内容排除
	[root@m01 ~]# rsync -avzP --append-verify --exclude-from=/tmp/exclude.txt  ./* root@172.16.1.41:/opt/

--bwlimit=100       # 限速传输（单位：MB）
	[root@m01 ~]# rsync -avzP --append-verify --bwlimit=10  ./* root@172.16.1.41:/opt/

--delete            # 让目标目录和源目录数据保持一致

--password-file=xxx # 使用密码文件

--port              # 指定端口传输
```

## rsync 守护进程模式

#### 1、服务端

```bash
1、安装
 [root@backup ~]# yum install -y rsync

2、修改配置文件
 [root@m01 ~]# vim /etc/rsyncd.conf
  uid = rsync
  gid = rsync
  port = 873
  fake super = yes
  use chroot = no
  max connections = 200
  timeout = 600
  ignore errors
  read only = false
  list = false
  auth users = rsync_backup
  secrets file = /etc/rsync.passwd
  log file = /var/log/rsyncd.log
  #####################################
  [backup]
  comment = welcome to backup!
  path = /backup
  [linux]
  comment = welcome to linux!
  path=/tmp/linux

3、创建系统用户
 [root@backup opt]# groupadd rsync -g 666
 [root@backup opt]# useradd rsync -u 666 -g 666 -M -s /sbin/nologin -r

4、创建密码文件
 [root@backup opt]# echo "rsync_backup:123456" > /etc/rsync.passwd

5、授权（必须授权为600）
 [root@backup opt]# chmod 600 /etc/rsync.passwd

6、创建备份目录
 [root@backup opt]# mkdir /backup
 [root@backup opt]# mkdir /tmp/linux

7、目录授权
 [root@backup opt]# chown rsync.rsync /backup/
 [root@backup opt]# chown rsync.rsync /tmp/linux/

8、关闭防火墙和selinux
 [root@backup opt]# systemctl disabel --now firewalld
 [root@backup opt]# setenforce 0

9、启动rsyncd服务
 [root@backup opt]# systemctl start rsyncd
```

#### 2、客户端

```bash
方法一：自己输入密码
  [root@m01 ~]# rsync -avzP ./* rsync_backup@172.16.1.41::backup

   rsync_backup ： 虚拟用户，只在数据传输时使用
   172.16.1.41  ： backup服务端的IP
   backup       ： 模块名称

方法二：设置密码文件，运行时读取

	1、编写密码文件
   [root@backup opt]# echo "123456" > /etc/rsync.passwd

	2、授权
   [root@m01 ~]# chmod 600 /etc/rsync.passwd

	3、连接
   [root@m01 ~]# rsync -avzP --password-file=/etc/rsync.passwd  ./* rsync_backup@172.16.1.41::linux


方法三：添加环境变量
	1、定义环境变量
   export RSYNC_PASSWORD=123456

	2、同步
   [root@m01 ~]# rsync -avzP  ./* rsync_backup@172.16.1.41::linux

```

## rsync 实时同步

```bash
rsync是不支持实时同步的，通常我们借助于inotify这个软件来实时监控文件变化，一旦inotify监控到文件变，则立即调用rsync进行同步。

1、安装inotify(装在客户端)
[root@web01 ~]# yum -y install inotify-tools

2、inotify参数介绍
-m 持续监控
-r 递归
-q 静默，仅打印时间信息
--timefmt 指定输出时间格式
--format 指定事件输出格式
    %Xe 事件
    %w 目录
    %f 文件
-e 指定监控的事件
    access 访问
    modify 内容修改
    attrib 属性修改
    close_write 修改真实文件内容
    open 打开
    create 创建
    delete 删除
    umount 卸载

3、开始监控
[root@m01 ~]# /usr/bin/inotifywait  -mrq  --format '%Xe  %w  %f' -e create,modify,delete,attrib,close_write  /root

4、实时监控并同步
[root@m01 ~]# /usr/bin/inotifywait  -mrq  --format '%Xe  %w  %f' -e create,modify,delete,attrib,close_write  /root | while read line;do
  cd  /root
  rsync -avzP --delete --password-file=/etc/rsyncd.passwd ./* rsync_backup@172.16.1.41::backup
done
```
