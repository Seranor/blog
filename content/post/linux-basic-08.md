---
title: Linux包管理及压缩命令
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-08.html
toc: true
---

# Linux 中安装软件的三种方式

与 windows 类似，在 Linux 系统上也可以安装各种应用程序，或称之为软件包

<!-- more -->

```bash
1.rpm安装
   rpm安装预先编译打包，安装简单，下载下来之后直接安装。
   优点：已经制作好的安装程序
	 缺点：不能自己解决依赖


2.yum安装
   yum安装基于rpm安装
   优点：增加了自动解决依赖的功能。


3.源代码编译安装
   源代码安装通过编译源代码，得到软件包。
   优点：可以自定制软件包。
   缺点：比较复杂
```

## 镜像文件

```bash
# 挂载：
  mount /dev/sr0 /opt/
  或
  mount /dev/cdrom /munt/

# 卸载：
  umount /dev/sr0  #挂载源
  或
  umount /opt  #挂载点

# 强制卸载：
  umount -l  [挂载源或挂载点]

# 查看挂载信息
  df


# 查看/操作设备内容需要先挂载
[root@localhost dev]# mount /dev/sr0 /opt/
 mount: /dev/sr0 写保护，将以只读方式挂载

# 查看是否挂载成功
[root@localhost ~]# df
 文件系统     1K-块    已用     可用 已用% 挂载点
 /dev/sr0   4480476  4480476   0  100% /opt

# 浏览光盘内容
[root@localhost dev]# ls /opt/
 CentOS_BuildTag  EULA  images    LiveOS repodata     RPM-GPG-KEY-CentOS-Testing-7
 EFI              GPL   isolinux  Packages  RPM-GPG-KEY-CentOS-7  TRANS.TBL

# 查看光盘上的安装包。格式都是以.rpm结尾的
[root@localhost dev]# ls /opt/Packages/
 ......
 zlib-1.2.7-18.el7.x86_64.rpm
 zlib-devel-1.2.7-18.el7.x86_64.rpm
 zsh-5.0.2-31.el7.x86_64.rpm
 zziplib-0.13.62-9.el7.x86_64.rpm

# 查看自己当前平台
[root@localhost ~]# uname -m
 x86_64
# 查看系统内核信息
[root@localhost ~]# uname -r
 3.10.0-1160.49.1.el7.x86_64

```

## 1、RPM 安装

- rpm 包来源
  - 1、来源网络下载
  - 2、来源本地：自己的镜像自带的 rpm 包

```bash
# 安装：rpm -ivh [软件包名称]
   -v  #显示安装过程
   -i  #显示安装包的详细信息
   -h  #安装包哈希标记

# 卸载：rpm -e [软件包名称]
# 升级：rpm -Uvh [软件包名称]

 1、下载安装包

 2、安装
  [root@localhost ~]# rpm -qip /opt/Packages/zsh-5.0.2-34.el7_8.2.x86_64.rpm  #本地镜像
  或
  [root@localhost ~]# rpm -ivh zsh-5.0.2-34.el7_8.2.x86_64.rpm
  Preparing...                    ################################# [100%]
  Updating / installing...
  1:zsh-5.0.2-34.el7_8.2          ################################# [100%]


 3、卸载
	[root@localhost ~]# rpm -e zsh


 4、更新
  [root@localhost ~]# rpm -Uvh zsh-5.0.2-34.el7_8.2.x86_64.rpm
  Preparing...                   ################################# [100%]
  Updating / installing...
  1:zsh-5.0.2-34.el7_8.2         ################################# [100%]


 5、软件包名称
  zsh-5.0.2-34.el7_8.2.x86_64.rpm
   zsh    #软件包名称
   5.0.2  #版本号
   34     #第多少次编译
   el7_8（CentOS 7）#适用的平台
   x86_64 #适用的系统位数
   rpm    #扩展名


 6、查看已安装软件包的使用配置文件
   [root@localhost ~]# rpm -qc  zsh

 7、查看已安装包的描述信息
   [root@localhost ~]# rpm -qi zsh

 8、查看是否安装某软件
   [root@localhost ~]# rpm -q zsh

 9、查看当前系统安装了哪些rpm软件
   [root@localhost ~]# rpm -qa

 10、查看软件的安装路径，查看安装了哪些东西
   [root@localhost ~]# rpm -ql zsh

 上传与下载：yum install lrzsz -y


扩展：
 1、查看未安装包的软件信息
   [root@localhost ~]# rpm -qip /opt/Packages/snappy-1.1.0-3.el7.x86_64.rpm

```

## 2、yum 安装

- yum 是 CentOS 的软件包管理工具，自动为我们解决软件依赖问题。yum 包管理工具必须使用 yum 源指定软件下载地址去下载需要安装的软件包。
  - 配置的路径是：/etc/yum.repos.d
- 要成功的使用 YUM 工具安装更新软件或系统，就需要有一个包含各种 rpm 软件包的 repository（软件仓库），这个软件仓库我们习惯称为 yum 源。(可以是本地源、网络源)

```bash
基于rpm安装，自动解决依赖。
# yum源命令:
   # 查看yum配置文件
   [root@localhost ~]# ls /etc/yum.repos.d

   # 查看当前的有哪些仓库地址
   [root@localhost ~]# yum repolist

   # 查看包括启用或禁用的所有yum仓库
   [root@localhost ~]# yum repolist all

   # 清空yum缓存
   [root@localhost ~]# yum clean all

   # 生成yum缓存
	 [root@localhost ~]#yum makecache


# yum常用的基础命令：

 # 1、安装软件包的命令
   yum install [软件包的名称]
   参数：
    -y : 免交互安装
    --nogpgcheck : 忽略公钥认证


 # 2、卸载软件（直接将软件的依赖包一起删除）
   yum remove [软件包名称]
   参数：
    -y : 免交互移除


 # 3、更新软件
   yum update [软件包名称]
   参数：
    -y : 免交互更新

 ps：如果跟具体的软件包名称，就会更新指定软件包；如果没有指定，则更新系统所有的需要更新的软件包。


 # 4、查看当前系统需要更新软件
   yum check-update


 # 5、重装软件
   yum reinstall [软件包名称]


 # 6、搜索软件包
   yum search [软件包名称]


yum安装的生命周期：
 1、执行yum install zsh -y
 2、去 /etc/yum.repos.d/ 找以 .repo 结尾的文件
 3、通过 .repo 文件中的链接，找到对应的软件仓库
 4、在对应的软件仓库中下载指定的软件包
 5、缓存至 /var/cache/yum/
 6、根据缓存，安装软件包
 7、删除软件包（keepcache 是否保存缓存，0 代表不保存 ， 1 代表保存）
```

知识储备：

```bash
# wget:下载文件
如果系统中没有wget,执行如下命令：yum install wget -y
  wget url
   参数：
    -O  #指定下载文件的路径及名称


# curl:读取文件
  curl ：读取文件
   参数:
    -o  #指定下载文件的路径及名称
    -k  #免证书认证

curl命令是⼀个利⽤URL规则在命令⾏下⼯作的⽂件传输⼯具。它⽀持⽂件的上传和下载，所以是综合传输⼯具，
但按传统，习惯称curl为下载⼯具。作为⼀款强⼒⼯具，curl⽀持包括HTTP、HTTPS、[ftp]等众多协议，还⽀
持POST、cookies、认证、从指定偏移处下载部分⽂件、⽤户代理字符串、限速、⽂件⼤⼩、进度条等特征。做⽹
⻚处理流程和数据检索⾃动化，curl可以祝⼀臂之⼒。

 [root@localhost ~]# curl -o 123.png https://www.xxx.com/img/hello.png
 # ps: 如果遇到下载提示⽆法简历SSL链接，使⽤-k选项或者--insecure
  curl -k -o 123.png https://www.xxx.com/img/hello.png


# sz下载文件与rz上传文件
 ps:  yum install lrzsz -y

sz : 下载文件（从linux系统下载文件到windows）
系统默认没有该命令，需要下载:yum install lrzsz -y
将服务器上选定的⽂件下载/发送到本机，

# rz : 上传文件(将windows文件上传至Linux)

  rz [文件路径]
	# 系统默认没有该命令，需要下载：yum install lrzsz -y
  # 运⾏该命令会弹出⼀个⽂件选择窗⼝，从本地选择⽂件上传到服务器。
  [root@localhost opt]# rz # 如果⽂件已经存，则上传失败，可以⽤-E选项解决
  [root@localhost opt]# rz -E # -E如果⽬标⽂件名已经存在，则重命名传⼊⽂件。新⽂件名将添加⼀个点和⼀个数字(0..999）

 rz 回车即可选择上传文件
 也可以进入都某个路径下将文件直接拖入~ （人性化）
```

### 本地&远程仓库搭建

```bash
本地仓库：
  # 1、下载安装必须的软件包yum-utils，createrepo
    yum install yum-utils createrepo -y

  # 2、创建软件包目录，存放软件包的
    mkdir -p /opt/repos

  # 3、下载对应的软件
    mkdir -p /opt/repos/Packages

    把对应的软件包复制到 Packages 目录中

  # 4、初始化软件仓库
		 createrepo /opt/repos

  # 5、添加yum源，将软件包复制到yum仓库目录
     cd /etc/yum.repos.d/
     mkdir backup
     mv *.repo backup/
     [root@localhost ~]# yum-config-manager --add-repo=file:///opt/repos

     [root@localhost /etc/yum.repos.d]# cat opt_repos.repo

     [opt_repos]	#源的名称
      name=added from: file:///opt/repos	 #源的简介
      baseurl=file:///opt/repos	#源的下载地址
      enabled=1	#是否启用：1启用 ，0不启用

  # 6、清空yum缓存
     yum clean all

  # 7、生成yum缓存
     yum makecache

  # 8、测试
     yum install zsh

	远程仓库
		参考本地版前7步

    # 1、安装远程访问软件（Nginx）
      # ① 配备CentOS-7 源
      [root@localhost ~]# curl -o /etc/yum.repos.d/CentOS-Base.repohttps://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo
			# ② 配备EPEL源
      [root@localhost ~]# yum-config-manager --add-repo=https://repo.huaweicloud.com/epel/7/x86_64/
			# ③ 安装nginx
      [root@localhost ~]# yum install nginx --nogpgcheck

    # 2、修改nginx的配置文件
      https://nginx.org/en/docs/http/ngx_http_autoindex_module.html

      [root@localhost ~]# vim /etc/nginx/nginx.conf
      # include /etc/nginx/conf.d/*.conf;
      root         /opt/repos;
      autoindex on;

      # 测试更改是否成功
      [root@localhost ~]# nginx -t

      # 启动nginx
      [root@localhost ~]# systemctl start nginx

      # 关闭selinux和firewalld
      [root@localhost ~]# systemctl disable --now firewalld
      [root@localhost ~]# setenforce 0




    # 3、在测试机
      [root@localhost yum.repos.d]# yum install yum-utils -y

      # 备份源
      [root@localhost yum.repos.d]# mkdir backup
      [root@localhost yum.repos.d]# mv *.repo backup/

      # 添加源
      [root@localhost yum.repos.d]# yum-config-manager --add-repo=http://192.168.15.101/

      # 刷新缓存
      [root@localhost yum.repos.d]# yum clean all
      [root@localhost yum.repos.d]# yum makecache

    # 4、测试
      [root@localhost ~]# yum install zsh -y
```

## 3、源码包安装（编译安装）

- **1.源码包是什么**

  - 源码包指的是开发编写好的程序源代码，但并没有将其编译为一个能正常使用的工具。-

- **2.为什么要学习源码包**

  - 1、部分软件官网仅提供源码包，需要自行编译并安装。
  - 2、部分软件在新版本有一些特性还没来得及制作成 rpm 包时，可以自行编译软件使用其新特性。

- **3.源码包的优缺点**

  - 优点是：
    - 可以自行修改源代码
    - 可以定制需要的相关功能
    - 新版软件优先更新源码
  - 缺点是:
    - 相对 yum 安装软件会复杂很多。
    - 标准化实施困难，自动化就无法落地。

- **4.源码包如何获取**

  - 常见的软件包都可以在官网获取源码包，比如 apache、nginx、mysql 等等

- **5.将源码包编译为二进制可执行文件步骤如下，简称安装三步曲**

![编译安装过程](https://gitee.com/gengff/blogimage/raw/master/images/%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.png)

注意: 此方法不是百分百通用于所有源码包，建议拿到源码包解压后，进入到目录找相关的 README 帮助文档

```bash
编译安装

# 1、基础环境准备
  [root@localhost ~]# yum install -y gcc make wget

# 2、下载源码包
	[root@localhost ~]# wget https://nginx.org/download/nginx-1.20.2.tar.gz

# 2、解压源码包, 并进入相应目录
	[root@localhost ~]# tar -xf nginx-1.20.2.tar.gz
  [root@localhost ~]# cd nginx-1.20.2

# 3、配置相关的选项，并生成Makefile
  [root@localhost nginx-1.20.2]# ./configure

# 4、将Makefile文件编译可执行二进制程序
	[root@localhost nginx-1.20.2]# make

# 5、将二进制文件拷贝至对应的目录中
	[root@localhost nginx-1.20.2]# make install

# 6、启动
	[root@localhost ~]# /usr/local/nginx/sbin/nginx
  # 启动后浏览器访问ip地址即可查看是否安装成功

# 7、关闭
	[root@localhost ~]# /usr/local/nginx/sbin/nginx -s stop

知识储备：
	tar -xf [压缩包名称]


知识拓展：
  自定制安装，修改源代码包名以 nginx 为例
  # 1、关闭nginx服务后，删除源代码包，目录
  [root@localhost ~]# rm -rf nginx-1.20.2
  [root@localhost ~]# rm -rf /usr/local/nginx/
  # 2、重新解压源码包, 并进入相应目录
	[root@localhost ~]# tar -xf nginx-1.20.2.tar.gz
  [root@localhost ~]# cd nginx-1.20.2
  # 3、找到对应的版本，版本号
  [root@localhost nginx-1.20.2]# grep -R 'nginx' ./
  [root@localhost nginx-1.20.2]# grep -R '1.20.2' ./
    ./src/core/nginx.h:#define NGINX_VERSION   "1.20.2"  #可以看到版本的路径
  # 修改文件，修改完后:wq退出
  [root@localhost nginx-1.20.2]# vim ./src/core/nginx.h

    '''

   /*
    * Copyright (C) Igor Sysoev
    * Copyright (C) Nginx, Inc.
    */


    #ifndef _NGINX_H_INCLUDED_
    #define _NGINX_H_INCLUDED_


    #define nginx_version      1020002
    #define NGINX_VERSION      "1.0" # 修改版本
    #define NGINX_VER          "GengFeng/" NGINX_VERSION  #修改名称

    #ifdef NGX_BUILD
     '''
  # 配置相关的选项，并生成Makefile
  [root@localhost nginx-1.20.2]# ./configure

  # 将Makefile文件编译可执行二进制程序
  [root@localhost nginx-1.20.2]# make

  # 将二进制文件拷贝至对应的目录中
  [root@localhost nginx-1.20.2]# make install

  # 启动
  [root@localhost ~]# /usr/local/nginx/sbin/nginx
  # 启动后浏览器访问ip地址即可查看是否安装成功

  # 修改后看到的是已改过的名称和版本，以 http://192.168.15.100/sdasdas 为例
     404 Not Found
     GengFeng/1.0
```

# 压缩打包

#### **1、** **什么是打包压缩**

打包指的是将多个⽂件和⽬录合并为⼀个特殊⽂件，然后将该特殊⽂件进⾏压缩，最终得到⼀个压缩包

#### **2、为什么使⽤压缩包**

- 1.减少占⽤的体积

- 2.加快⽹络的传输

#### 3、Windows 的压缩和 Linux 的有什么不同

- **windows: zip rar(linux 不⽀持 rar)**
- **linux: zip tar.gz tar.bz2 .gz**

如果希望 windows 的软件能被 linux 解压，或者 linux 的软件包被 windows 能识别，选择 zip.

PS: 压缩包的后缀不重要，但⼀定要携带.

#### 4、Linux 下常⻅的压缩包类型

| 格式       | 压缩工具                                                              |
| ---------- | --------------------------------------------------------------------- |
| `.zip`     | **zip** 压缩工具                                                      |
| `.gz`      | **gzip** 压缩工具，只能压缩文件，会删除源文件（通常配合**tar**使用）  |
| `.bz2`     | **bzip2** 压缩工具，只能压缩文件，会删除源文件（通常配合**tar**使用） |
| `.tar.gz`  | 先使用**tar** 命令归档打包，然后使用 **gzip** 压缩                    |
| `.tar.bz2` | 先使用**tar** 命令归档打包，然后使用 **bzip** 压缩                    |

Linux 常见的压缩包有哪些？

- gzip
- bzip2

## gzip 打包与压缩

- 使用 gzip 方式进行压缩文件

```bash
# 压缩命令：gzip [压缩文件]
# 解压命令：gzip -d [压缩包]

[root@localhost ~]# gzip file       #对文件进行压缩
[root@localhost ~]# zcat file.gz    #查看gz压缩后的文件
[root@localhost ~]# gzip -d file.gz #解压gzip的压缩包

#使用场景:当需要让某个文件不生效时
[root@localhost ~]# gzip CentOS-Vault.repo --> CentOS-Vault.repo.gz
[root@localhost ~]# zcat CentOS-Vault.repo.gz --> 查看不想解压的压缩包文件内容
```

## bzip2 打包与压缩

```bash
# 压缩命令：bzip2 [压缩文件]
# 解压命令：bzip2 -d [压缩包]

[root@localhost ~]# bzip2 file        #对文件进行压缩
[root@localhost ~]# bzmore file.bz2   #查看bz2压缩后的文件
[root@localhost ~]# bzip2 -d file.bz2 #解压bzip2的压缩包
```

## tar 打包与压缩

- tar 是 linux 下最常用的压缩与解压缩, 支持文件和目录的压缩归档

```bash
tar : 打包的命令
  参数：
    -f  #指定包文件名称，多参数f写最后
    -c  #打包
    -v  #输出命令的打包或解包的过程
    -x  #解压（解压不需要指定压缩类型）
    -t  #查看压缩包内部的内容


    -z  #使用gzip压缩压缩包
    -j  #使用bzip2压缩压缩包
    -J  #使用xz压缩归档后的文件(tar.xz)
    -C  #指定解压目录位置
    -P  #忽略使用绝对路径时报出的错误
    -X  #排除多个文件(写入需要排除的文件名称)
    -h  #打包软链接



    --hard-dereference  #打包硬链接
    --exclude   #在打包的时候写入需要排除文件或目录

#常用打包与压缩组合
czf    #打包tar.gz格式
cjf    #打包tar.bz格式
cJf    #打包tar.xz格式

zxf    #解压tar.gz格式
jxf    #解压tar.bz格式
xf     #自动选择解压模式
tf     #查看压缩包内容

  注意：
     1、压缩时是什么路径，解压缩时就是什么路径，所以为了安全不要使用绝对路径压缩。
     2、-f参数后面永远跟压缩包名称



# tar命令练习
  #1.环境准备
  [root@localhost ~]# yum install mariadb-server
  [root@localhost ~]# systemctl start mariadb
  [root@localhost ~]# mkdir /backup

#案例1.mysql备份及恢复
  [root@localhost ~]# tar cJf /backup/mysql.tar.xz /var/lib/mysql
  [root@localhost ~]# tar xf /backup/mysql.tar.xz -C /

#案例2 mysql备份及恢复
  [root@localhost ~]# cd /var/lib/mysql
  [root@localhost mysql]# tar cJf /backup/mysql.tar.xz *
  [root@localhost mysql]# tar tf /backup/mysql.tar.xz
  [root@localhost mysql]# tar xf /backup/mysql.tar.xz -C /var/lib/mysql
```
