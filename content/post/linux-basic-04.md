---
title: Linux系统目录结构
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-04.html
toc: true
---

# 重要目录文件

## 网卡配置文件

<!-- more -->

```bash
文件信息：/etc/sysconfig/network-scripts/ifcfg-eth0
ip a
作用：
1、查看网卡配置
  [root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0

  [root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens32

  或
  ip address show 或 nmtui
2、重载网卡信息
  # 方式一
  [root@localhost ~]# systemctl restart network
  # 方式二
	ifdown [网卡名称] && ifup [网卡名称]

  关闭网络管理器（因为已经有了network）
		systemctl  stop NetworkManager
		systemctl  disable NetworkManager
		或
		systemctl  disable --now  NetworkManager

3、判断SSH服务是否开启
   [root@localhost ~]# systemctl status sshd
```

![1113510-20170614182256790-1747672277](https://gitee.com/gengff/blogimage/raw/master/images/1113510-20170614182256790-1747672277-20211214152247352-20211214171846597.png)

## 解析配置文件

```bash
文件信息：/etc/resolv.conf
作用：用于设置DNS解析地址，网卡中配置优于此文件配置

#查看DNS信息
  [root@test1 data]# cat /etc/reslov.conf  #临时dns配置文件
  nameserver 114.114.114.114  #中国电信

   223.5.5.5/223.6.6.6	 #中国阿里云
   8.8.8.8 谷歌
```

## 解析映射文件

```bash
文件信息：/etc/hosts
作用：用于设置DNS域名与IP地址对应关系

#查看解析映射文件（dns解析）
   [root@localhost ~]# cat /etc/hosts
   127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

#查看系统版本
   [root@localhost ~]# cat /etc/redhat-release
   CentOS Linux release 7.6.1810 (Core)
```

## 主机名称文件

```bash
文件信息：/etc/sysconfig/network
作用：配置主机名称信息

  修改主机名
   #查看主机名：
    [root@localhost ~]# ehco $HOSTNAME

   #临时修改
    [root@localhost ~]# hostname baidu

   #永久修改
    [root@baidu ~]# vim /etc/hostname	  #需要重启生效
    [root@baidu ~]# hostnamectl set-hostname admin	#立即生效
```

## 磁盘挂载文件

```bash
文件信息：/etc/fstab
作用：实现指定设备文件信息，进行开机自动挂载

#查看磁盘挂载文件
[root@localhost ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Dec 13 11:38:54 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /        xfs     defaults    0 0  #挂载在根(/)目录
UUID=9f8a98b0-805c-4adf-b9ef-517a2b527f89 /boot      xfs     defaults   0 0 #挂载在/boot目录

```

## 开机加载脚本

```bash
文件信息：/etc/rc.local
作用：开启开机自启动脚本

1、编辑开机自启动脚本
	vim /etc/rc.local
	 #写入
	 echo 'hello worl'
2、设置开机自启动权限
	chmod +x /etc/rc.d/rc.local
3、重启系统
```

## 系统启动级别文件

```bash
文件信息：/etc/inittab

作用：设置系统启动级别
 0、关机
 1、单用户模式(无法通过xshell的方式使用)
 2、多用户无网络模式
 3、完全多用户模式
 4、待定
 5、桌面模式
 6、reboot (Do NOT set initdefault to this) 重启

设置系统级别：
 init [编号]	#临时设置
 systemctl set-default [系统启动级别]

通过单用户模式修改密码
 1、重启
 2、在启动选择系统内核界面，按 e 键进入单用户模式
 3、找到 linux16 开头行，删除 ro ，并且在 ro 处添加 rw init=/sysroot/bin/sh
 4、按 ctrl + x 进行系统重新引导
 5、执行 chroot /sysroot
 6、执行 passwd root
 7、执行 touch /.autorelabel
 8、执行 Ctrl + D 重启系统
```

## 变量加载文件

```bash
# 在Linux中添加环境变量怎么添加呢？

文件信息：/etc/profile
作用：配置环境变量和别名文件
  文件
    /etc/profile
    /etc/bashrc
    ~/.bash_profile
    ~/.bash_rc
  文件夹
   /etc/profile.d/

增加环境变量有两种方式：
  1、临时添加
  2、永久添加

增加环境变量的格式：
 export PYTHON_HOME='D:/python'

查看本机的环境变量：
  echo $PYTHON_HOME  #查看某一个环境变量
  printenv           #查看所有的环境变量

读取环境变量的几种情况，并且测试出使用文件的先后顺序
 1、重启
  /etc/profile.d --> /etc/profile --> /etc/bashrc --> ~/.bashrc --> ~/.bash_profile
 2、切换用户
  /etc/profile.d --> /etc/bashrc --> ~/.bashrc
  知识储备
    创建用户：
      useradd [用户名]
    切换用户：
      su [用户名]
 3、重新登录用户
  1、su - [用户名]
   /etc/profile.d --> /etc/profile --> /etc/bashrc --> ~/.bashrc --> ~/.bash_profile

  2、ssh root@192.168.15.101
   /etc/profile.d --> /etc/profile --> /etc/bashrc --> ~/.bashrc --> ~/.bash_profile
```

## 登录提示文件

```bash
文件信息：/etc/motd

作用：登录成功之后显示的信息。


文件信息：/etc/issue

作用：登录系统之前显示的信息。
```

## 编译安装目录

```bash
文件信息：/usr/local  #安装第三方软件的目录

作用：编译安装软件的默认目录


下载rpm安装包
  # yum安装python
    [root@localhost ~]# yum install python3
  # 查看软件安装路径
    [root@localhost ~]# which python3


知识储备：当前为DVD镜像需要设置阿里云的源
 #第一步：把之前的源备份换个位置
 [root@localhost ~]# mv /etc/yum.repos.d/* /etc/yum.repos.d/bak

 #第二步：下载阿里云源repo文件
 [root@localhost ~]# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

 #第三步：下载阿里云源的epel文件
 [root@localhost ~]# curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

![image-20211214173804157](https://gitee.com/gengff/blogimage/raw/master/images/image-20211214173804157.png)

## 重要日志文件

- 系统日志目录：**/var**
  - 绝大部分的日志都存放在该目录下

```bash
系统日志文件
文件信息：/var/log/messages
作用：软件安装运行以及系统运行异常日志文件

  #查看日志
  [root@localhost /]# cat /var/log/messages
```

## 重要信息文件

- 保存系统运行状态的目录：**/proc**

```bash
保存CPU信息情况的文件
文件信息：：/proc/cpuinfo
相关命令：lscpu


保存内存信息情况的文件
文件信息：/proc/meminfo
相关命令：free


保存系统负载信息情况的文件，用于衡量系统繁忙程度
文件信息：/proc/loadavg
相关命令：w
  [root@localhost /]# cat /proc/loadavg  #查看CPU负载
    0.00  #1分钟内的CPU负载
    0.01  #5分钟内的CPU负载
    0.05  #15分钟内的CPU负载

 负载：当前系统的所有进程占用CPU的时间比


保存系统挂载信息文件
文件信息：/proc/mounts
  mount
  umount
```
