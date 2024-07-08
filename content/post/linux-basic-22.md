---
title: Keepalived高可用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Keepalived
categories:
  - Linux
url: post/linux-22.html
toc: true
---

# keepalived 高可用

### 什么是高可用

- 一般是指 2 台机器启动着完全相同的业务系统，当有一台机器 down 机了，另外一台服务器就能快速的接管，对于访问的用户是无感知的。
<!-- more -->

```bash
比如公司的网络是通过网关进行上网的，那么如果该路由器故障了，网关无法转发报文了，此时所有人都无法上网了，
怎么办？

通常做法是给路由器增加一台备节点，但是问题是，如果我们的主网关master故障了，用户是需要手动指向backup的，
如果用户过多修改起来会非常麻烦。

问题一：假设用户将指向都修改为backup路由器，那么master路由器修好了怎么办？
问题二：假设Master网关故障，我们将backup网关配置为master网关的ip是否可以？

其实是不行的，因为PC第一次通过ARP广播寻找到Master网关的MAC地址与IP地址后，会将信息写到ARP的缓存表中，
那么PC之后连接都是通过那个缓存表的信息去连接，然后进行数据包的转发，即使我们修改了IP但是Mac地址是唯一的，
pc的数据包依然会发送给master。（除非是PC的ARP缓存表过期，再次发起ARP广播的时候才能获取新的backup对应
的Mac地址与IP地址）

如何才能做到出现故障自动转移，我们往下看
```

### 高可用常用的工具

```bash
1.硬件通常使用 F5
2.软件通常使用 keepalived
```

keepalived 是如何实现的？

### VRRP 协议

- VRRP 协议会在一个局域网中进行广播
- keepalived 软件是基于 VRRP 协议实现的，VRRP 是虚拟路由冗余协议，主要用于解决单点故障问题

```bash
如何才能做到出现故障自动转移，此时VRRP就出现了，我们的VRRP其实是通过软件或者硬件的形式在Master和
Backup外面增加一个虚拟的MAC地址（VMAC）与虚拟IP地址（VIP），那么在这种情况下，PC请求VIP的时候，
无论是Master处理还是Backup处理，PC仅会在ARP缓存表中记录VMAC与VIP的信息。
```

### 高可用 keepalived 的核心概念

```bash
1、如何确定谁是主节点谁是备节点（选举投票，优先级）
2、如果Master故障，Backup自动接管，那么Master恢复后会夺权吗（抢占试、非抢占式）
3、如果两台服务器都认为自己是Master会出现什么问题（脑裂）
```

# 部署 keepalived

### 1、环境准备

| 主机 | IP           | 身份   |
| :--- | :----------- | :----- |
| lb01 | 192.168.15.5 | master |
| lb02 | 192.168.15.6 | backup |
|      | 192.168.15.3 | VIP    |

### 2、保证 lb01 和 lb02 配置完全一致

```bash
[root@lb02 conf.d]# scp -r /etc/nginx/ssl_key 172.16.1.5:/etc/nginx/
[root@lb02 conf.d]# scp ./* 172.16.1.5:/etc/nginx/conf.d/
```

### 3、安装 Keepalived

```
[root@lb01 conf.d]# yum install keepalived -y
[root@lb02 conf.d]# yum install keepalived -y
```

### 4、Keepalived 配置

#### 主

```bash
[root@lb01 ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
# 全局配置
global_defs {
   # 身份验证，当前keepalived的唯一标识
   router_id lb01
}

# 检测脚本:每5秒执行一次脚本，脚本执行完成时间不能超过5秒，否则会重新执行脚本，死循环
vrrp_script check_nginx {
	# 指定脚本路径
    script "/etc/keepalived/checkNG.sh"
    # 执行间隔
    interval 5
}

# 配置VRRP协议
vrrp_instance VI_1 {
    # 状态，只有MASTER和BACKUP，MASTER是主，BACKUP是备
    state MASTER
    # 绑定网卡
    interface eth0
    # 虚拟路由标示，可以理解为分组id，把master和backup判断为一组
    virtual_router_id 50
    # 优先级（真正判断是主是从的条件）（值越大优先级越高）
    priority 100
    # 监测心跳间隔时间（单位是秒）
    advert_int 1
    # 配置认证
    authentication {
        # 认证类型
        auth_type PASS
        # 认证的密码
        auth_pass 1111
    }
    # 设置VIP
    virtual_ipaddress {
        # 虚拟的VIP地址
        192.168.15.3
    }
    # 调用检查
    track_script {
        check_nginx
    }
}

# 启动
[root@lb01 ~]# systemctl enable --now keepalived
```

#### 备

```bash
[root@lb02 ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
# 全局配置
global_defs {
   # 身份验证，当前keepalived的唯一标识
   router_id lb02
}

# 检测脚本
vrrp_script check_nginx {
	# 指定脚本路径
    script "/etc/keepalived/checkNG.sh"
    # 执行间隔
    interval 5
}

# 配置VRRP协议
vrrp_instance VI_1 {
    # 状态，只有MASTER和BACKUP，MASTER是主，BACKUP是备
    state BACKUP
    # 绑定网卡，主备必须一致
    interface eth0
    # 虚拟路由标示，可以理解为分组id，把master和backup判断为一组
    virtual_router_id 50
    # 优先级（真正判断是主是从的条件）（值越大优先级越高）
    priority 80
    # 监测心跳间隔时间（单位是秒）
    advert_int 1
    # 配置认证
    authentication {
        # 认证类型
        auth_type PASS
        # 认证的密码
        auth_pass 1111
    }
    # 设置VIP
    virtual_ipaddress {
        # 虚拟的VIP地址
        192.168.15.3
    }
    # 调用检查
    track_script {
        check_nginx
    }
}

# 启动
[root@lb02 ~]# systemctl enable --now keepalived
```

# 高可用 keepalived 的脑裂

- 两台高可用服务器在指定时间内，无法互相检查到对方的心跳而各自启动故障转移功能。

```bash
1、如果Nginx宕机怎么办？
想办法告诉keepalived，Nginx的情况。

2、局域网之内，keepalived无法相互广播，怎么办？
判断VIP是否可以ping的通


$?  : 上一条命令执行的结果。
```

1、Nginx 故障切换脚本

```bash
[root@lb01 ~]# cat checkNG.sh
#!/bin/bash

# 解决Nginx无法正常启动
ps -ef | grep -q [n]ginx

if [ $? -ne 0 ];then
	# 代表Nginx未正常启动
	systemctl start nginx &>/dev/null
	sleep 2
	ps -ef | grep -q [n]ginx
	if [ $? -ne 0 ];then
		systemctl stop keepalived
	fi
fi


# 局域网之内，keepalived无法相互广播，怎么办？
# VIP=192.168.15.3

# ping -c 1 $VIP &>/dev/null

# if [ $? -eq 0 ];then
	# 代表VIP还可以访问

# fi

# & : 正确的标准输出和错误的标准输出
```

2、测试

```bash
# 关闭lb01的nginx
[root@lb01 ~]# systemctl stop nginx

# 查看lb01的nginx启动状态可以发现已经自启动了
[root@lb01 ~]# systemctl status nginx

# 修改lb01的nginx配置造成故障
[root@lb01 ~]# vim /etc/nginx/nginx.conf

# 然后重启lb01的nginx发现无法启动
[root@lb01 ~]# systemctl stop nginx
[root@lb01 ~]# systemctl start nginx

# 然后再查看lb01的keepalived的启动状态发现也关掉了
[root@lb01 ~]# systemctl status keepalived

# 最后再浏览器访问192.168.15.3发现并不影响我们页面的正常访问，做到了真正的高可用
```

3、抢占式

```bash
# 查看主节点lb01的ip（窗口1）
[root@lb01 ~]# ip addr
inet 192.168.15.3/32 scope global eth0

# 循环执行，可以发现正常访问（窗口2）
[root@lb01 ~]# while ture; do curl -I 192.168.15.3; done


# 关闭主节点lb01的keepalived（窗口1）
[root@lb01 ~]# systemctl stop keepalived


# 查看备节点lb02的ip发现跳转到lb02这边来了
[root@lb02 ~]# ip addr
inet 192.168.15.3/32 scope global eth0


# 再次启动主节点lb01的keepalived（窗口2）
[root@lb01 ~]# systemctl start keepalived

# 此时查看主节点lb01的ip（窗口1）
[root@lb01 ~]# ip addr
inet 192.168.15.3/32 scope global eth0

# 查看备节点lb02的ip已经没有了192.168.15.3/32


# 再次启动主节点lb01的keepalived，我们可以发现lb01（窗口2）访问出现了卡顿现象
# 这是由于keepalived在切换ip的时候极其的不稳定会导致用户访问页面时出现报错极大影响客户的体验，
# 这就用到了非抢占式
```

# keepalived 的非抢占式

- 实现非抢占式，避免 keepalived 在切换 IP 的时候出现问题，我们一般配置的都是非抢占式的，因为宕机这种行为一次就够了
  - 1、状态全部都设置成 backup
  - 2、增加 nopreempt

## 主

```bash
[root@lb01 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id lb01
}

# 检测脚本
vrrp_script check_nginx {
    # 指定脚本路径
    script "/etc/keepalived/checkNG.sh"
    # 执行间隔
    interval 5
}

# 配置VRRP协议
vrrp_instance VI_1 {
    #状态，MASTER和BACKUP
    state BACKUP
    # 开启非抢占式
    nopreempt
    #绑定网卡
    interface eth0
    #虚拟路由标示，可以理解为分组
    virtual_router_id 50
    #优先级
    priority 100
    #监测心跳间隔时间
    advert_int 1
    #配置认证
    authentication {
        #认证类型
        auth_type PASS
        #认证的密码
        auth_pass 1111
    }
    #设置VIP
    virtual_ipaddress {
        #虚拟的VIP地址
        192.168.15.3
    }
    # 调用检查
    track_script {
        check_nginx
    }
}
```

## 备

```bash
[root@lb02 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id lb02
}

# 检测脚本
vrrp_script check_nginx {
    # 指定脚本路径
    script "/etc/keepalived/checkNG.sh"
    # 执行间隔
    interval 5
}

# 配置VRRP协议
vrrp_instance VI_1 {
    #状态，MASTER和BACKUP
    state BACKUP
    # 开启非抢占式
    nopreempt
    #绑定网卡
    interface eth0
    #虚拟路由标示，可以理解为分组
    virtual_router_id 50
    #优先级
    priority 80
    #监测心跳间隔时间
    advert_int 1
    #配置认证
    authentication {
        #认证类型
        auth_type PASS
        #认证的密码
        auth_pass 1111
    }
    #设置VIP
    virtual_ipaddress {
        #虚拟的VIP地址
        192.168.15.3
    }
    # 调用检查
    track_script {
        check_nginx
    }
}
```
