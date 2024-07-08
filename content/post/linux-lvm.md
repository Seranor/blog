---
title: Linux系统下LVM扩容
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - LVM
categories:
  - Linux
url: post/linux-lvm.html
toc: true
---

### LVM 扩展

#### 剩余空间扩容

<!-- more -->

##### 当前磁盘的使用状态

![image-20220502173511764](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502173511764.png)

##### 将该机器扩展 10G

![image-20220502173611709](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502173611709.png)

##### 扩展盘后的状态

![image-20220502173653188](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502173653188.png)

##### 步骤

```shell
# 查看可用物理卷
lvmdiskscan

# 将剩余空间创建分区
lsblk           # 查看磁盘名称
fdisk /dev/sda  # 创建分区

p     # 查看当前分区情况
n     # 开始分区
p     # 分区的格式
回车  # 分区名称 回车使用默认的
回车  # 分区大小 回车使用默认的
w     # 保存退出
```

![image-20220502174037578](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502174037578.png)

```shell
# 让系统核心重新捕捉分区表
partprobe

# 查看新分区是否被系统识别
lsblk
```

![image-20220502174438514](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502174438514.png)

```shell
# 创建PV
pvcreate /dev/sda3

# 查看PV
pvdisplay   # 可以看到 VG Name
pvs
```

![image-20220502174538675](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502174538675.png)

```shell
# 扩容centos VG
vgextend centos /dev/sda3  # centos 就是上面一样的 VG Name

# 查看扩容的VG
vgdisplay

# 添加空间到根分区
lvextend -l +100%FREE /dev/centos/root

# 扩容生效
# xfs格式
xfs_growfs /dev/centos/root
# ext4格式
resize2fs /dev/centos/root
```

![image-20220502174712791](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502174712791.png)

#### 添加新盘扩容

##### 当前状态

![image-20220502175020769](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502175020769.png)

##### 添加新硬盘

![image-20220502175245257](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502175245257.png)

![image-20220502175519726](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502175519726.png)

```shell
# 查看可用物理卷
lvmdiskscan

# 创建物理卷pv 将整块sdb扩容到 / 下，如果不想就先分区
pvcreate /dev/sdb

# 查看PV
pvdisplay  # 可以看到 VG Name

# 扩展
vgextend centos /dev/sdb  # VG Name 设置一样的

# 查看PV
pvdisplay   # 可以看到 VG Name
pvs

# 扩容centos VG
vgextend centos /dev/sda3  # centos 就是上面一样的 VG Name

# 查看扩容的VG
vgdisplay

# 添加空间到根分区
lvextend -l +100%FREE /dev/centos/root

# 扩容生效
# xfs格式
xfs_growfs /dev/centos/root
# ext4格式
resize2fs /dev/centos/root
```

![image-20220502180947082](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502180947082.png)

![image-20220502181201227](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220502181201227.png)
