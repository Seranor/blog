---
title: Mac上VMware的NAT网络设置
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MacOS
categories:
  - MacOS
url: post/mac-vm-nat.html
toc: true
---

### NAT 网络设置

#### 创建一个 NAT 网络

<!-- more -->

![4vNGH4](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/4vNGH4.png)

#### 修改子网段

![image-20211214144606331](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211214144606331.png)

#### Mac 上查看该子网段的网关

```bash
# mac上查自己的
cat /Library/Preferences/VMware\ Fusion/vmnet3/nat.conf | grep gateway -A 2

# NAT gateway address
ip = 192.168.100.1
netmask = 255.255.255.0
```

#### 设置虚拟机网络

![image-20211214144722670](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211214144722670.png)

![image-20211214144745643](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211214144745643.png)

![image-20211214144802455](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211214144802455.png)

#### Centos 镜像源设置

```bash
# centos更换阿里云源
mkdir -p /etc/yum.repos.d/bak
mv /etc/yum.repos.d/* /etc/yum.repos.d/bak
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum install vim net-tools gcc make -y
```
