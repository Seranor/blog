---
title: NFS搭建使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - NFS
categories:
  - Linux
url: post/linux-17.html
toc: true
---

# NFS 网络存储

## NFS 简介

<!-- more -->

```bash
NFS : 共享存储，文件服务器

基本概述：
   1.NFS是Network File System的缩写及网络文件系统。NFS主要功能是通过局域网络让不同的主机系统之间可以共享文件或目录。

   2.NFS系统和Windows网络共享、网络驱动器类似, 只不过windows用于局域网, NFS用于企业集群架构中, 如果是大型网站, 会用到更复杂的分布式文件系统FastDFS，glusterfs，HDFS，ceph


为何使用NFS：
  1.实现多台服务器之间数据共享
  2.实现多台服务器之间数据一致
```

![2251663-20210419213338770-1524770891](https://gitee.com/gengff/blogimage/raw/master/images/2251663-20210419213338770-1524770891.png)

## NFS 应用

```bash
1.用户访问NFS客户端，将请求转化为函数
2.NFS通过TCP/IP连接服务端
3.NFS服务端接收请求，会先调用portmap进程进行端口映射
4.Rpc.nfsd进程用于判断NFS客户端能否连接服务端；
5.Rpc.mount进程用于判断客户端对服务端的操作权限
6.如果通过权限验证，可以对服务端进行操作，修改或读取
```

nfs 运行原理

![nfs运行原理-1024x640](https://gitee.com/gengff/blogimage/raw/master/images/nfs%E8%BF%90%E8%A1%8C%E5%8E%9F%E7%90%86-1024x640.png)

## NFS 实践

- 为了实现文件共享
- 为了多台服务器之间数据一致

nfs 原理

![nfs原理-1024x754](https://gitee.com/gengff/blogimage/raw/master/images/nfs原理-1024x754.png)

#### 1、服务端

```bash
1、安装NFS和rpcbind
[root@nfs ~]# yum install nfs-utils rpcbind -y

2、创建挂载点
[root@nfs ~]# mkdir -p /web/nfs{1..9}

3、配置挂载点
[root@nfs ~]# vim /etc/exports
格式：
[挂载点] [可以访问的IP]([权限])#NFS配置详解参考下方
/web/nfs1  172.16.1.0/20(rw,sync,all_squash)

4、关闭selinux和防火墙
[root@nfs ~]# setenforce 0
[root@nfs ~]# systemctl disable --now firewalld

5、启动nfs和rpcbind服务
[root@nfs ~]# systemctl start nfs-server
[root@nfs ~]# systemctl start rpcbind

6、检查服务端是否正常
[root@nfs ~]# showmount -e [服务端的地址，默认是本机地址]

[root@nfs ~]# showmount -e
Export list for nfs:
/web/nfsv1 172.16.1.0/20
[root@nfs ~]# showmount -e 172.16.1.31
Export list for 172.16.1.31:
/web/nfsv1 172.16.1.0/20

[root@nfs ~]# cat /var/lib/nfs/etab

7、给挂载点授权
[root@nfs ~]# chown -R nfsnobody.nfsnobody /web
```

#### 2、客户端

```bash
1、安装NFS
[root@web01 opt]# yum install -y nfs-utils

2、创建目录
[root@web01 opt]# mkdir /opt/nfs/

3、挂载NFS
[root@web01 opt]# mount -t nfs 172.16.1.31:/web/nfs1  /opt/nfs/

4、测试NFS文件同步功能
[root@web opt]# touch nfs/{1..9}.txt
[root@web opt]# ll nfs/
 总用量 0
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 1.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 2.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 3.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 4.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 5.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 6.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 7.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 8.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 2021 9.txt

########################################################
 # 切换到服务端nfs服务器上查看
[root@nfs ~]# ll /web/nfs1 #发现已经同步过来了
 总用量 0
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 1.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 2.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 3.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 4.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 5.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 6.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 7.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 8.txt
 -rw-r--r-- 1 nfsnobody nfsnobody 0 12月 30 22:02 9.txt

```

## NFS 配置详解

| nfs 共享参数   | 参数作用                                                                      |
| -------------- | ----------------------------------------------------------------------------- |
| rw             | 读写权限 (常用)                                                               |
| ro             | 只读权限 (不常用)                                                             |
| root_squash    | 当 NFS 客户端以 root 管理员访问时，映射为 NFS 服务器的匿名用户 (不常用)       |
| no_root_squash | 当 NFS 客户端以 root 管理员访问时，映射为 NFS 服务器的 root 管理员 (不常用)   |
| all_squash     | 无论 NFS 客户端使用什么账户访问，均映射为 NFS 服务器的匿名用户 (常用)         |
| no_all_squash  | 无论 NFS 客户端使用什么账户访问，都不进行压缩 (不常用)                        |
| sync           | 同时将数据写入到内存与硬盘中，保证不丢失数据 (常用)                           |
| async          | 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据 (不常用) |
| anonuid        | 配置 all_squash 使用,指定 NFS 的用户 UID,必须存在系统 (常用)                  |
| anongid        | 配置 all_squash 使用,指定 NFS 的用户 GID,必须存在系统 (常用)                  |

```bash
1、控制读写
  rw  #可读可写
  ro  #只读

2、控制文件权限
root_squash
no_root_squash
all_squash
no_all_squash

3、控制写模式
sync  #同步写
async #异步写

4、控制用户
anonuid
anongid

统一用户：

1、创建用户
[root@nfs nfs1]# groupadd www -g 666
[root@nfs nfs1]# useradd www -u 666 -g 666 -M -r -s /sbin/nologin

2、修改配置挂载点
[root@nfs nfs1]# vim /etc/exports
 /web/nfs1  172.16.1.0/20(rw,sync,all_squash，anonuid=666，anongid=666)
# 修改后重启
[root@nfs nfs1]# systemctl restart nfs-server rpcbind

3、修改挂载点权限
[root@nfs nfs1]# chown -R www.www /web/

3、使用
# 先卸载再重新挂载
[root@nfs nfs1]#umount /opt/nfs/
[root@nfs nfs1]#mount -t nfs 172.16.1.31:/web/nfs1 /opt/nfs/
# 写入文件
[root@nfs nfs1]# touch nfs/10.txt
# 查看
[root@nfs nfs1]# ll nfs/
```

## 搭建考试系统

#### 1、搭建 WEB 服务

```bash
1、安装web软件
[root@web01 opt]# yum install httpd php php-devel -y

2、将代码放置于网站的根目录
[root@web01 opt]# cd /var/www/html/

# 上传代码

3、授权
[root@web01 html]# chown -R www.www /var/www/html

4、关闭selinux和防火墙
[root@nfs ~]# setenforce 0
[root@nfs ~]# systemctl disable --now firewalld

5、修改web软件的用户
[root@web01 html]# vim /etc/httpd/conf/httpd.conf
User www
Group www

6、启动web软件
[root@web01 html]# systemctl start httpd

7、测试

	1、上传

	2、访问
    http://172.16.1.7/upload/1_linux.jpg
```

#### 2、配合 NFS 实现文件共享

```bash
1、修改NFS配置文件
[root@nfs nfs1]# vim /etc/exports
/web/upload  172.16.1.0/20(rw,sync,all_squash,anonuid=666,anongid=666)

2、创建挂载点
[root@nfs nfs1]# mkdir /web/upload
[root@nfs nfs1]# chown www.www /web/upload

3、重启NFS
[root@nfs nfs1]# systemctl restart nfs-server rpcbind

4、客户端安装NFS软件
[root@web01 html]# yum install nfs-utils -y
[root@web02 html]# yum install nfs-utils -y
[root@web03 html]# yum install nfs-utils -y

5、挂载
[root@web01 html]# mount -t nfs 172.16.1.31:/web/upload /var/www/html/upload
[root@web02 html]# mount -t nfs 172.16.1.31:/web/upload /var/www/html/upload
[root@web03 html]# mount -t nfs 172.16.1.31:/web/upload /var/www/html/upload

6、测试
# 用web2上传，web3查看

#!/bin/bash
sed -i 's/.100/$1/g' /etc/sysconfig/network-scripts/ifcfg-eth[10]
hostnamectl set-hostname $2
systemctl restart network
```
