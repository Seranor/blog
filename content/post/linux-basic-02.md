---
title: Linux目录结构
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-02.html
toc: true
---

# 虚拟机快照

### 1、拍摄快照

![image-20211209204224924](https://gitee.com/gengff/blogimage/raw/master/images/image-20211209204224924.png)

<!-- more -->

![image-20211209204345586](https://gitee.com/gengff/blogimage/raw/master/images/image-20211209204345586.png)

### 2、恢复快照

![image-20211209204613798](https://gitee.com/gengff/blogimage/raw/master/images/image-20211209204613798.png)

```python

1、克隆主机
		 管理 ---> 克隆

2、改ip

	最后一位3 - 254
    # 查看网卡名称，或者查看本机IP
    [root@localhost ~]# ip a

    # 修改网卡
    [root@localhost ~]#

    [root@localhost ~]# sed -i 's#.100#.101#g' /etc/sysconfig/network-scripts/ifcfg-eth[01]-

    # 重启系统网络
    [root@localhost ~]# systemctl restart network
```

# bash 概述

bash（壳）是一个命令解释器，负责跟系统的内核进行交互，在操作系统的最外层

bash 可以干什么？针对于操作系统做了一些操作

- 文件管理

- 目录管理

- 权限管理

- 用户管理

- 应用管理

- 软件管理

- 磁盘管理

- 等等

  执行方式 操作简单 针对简单的管理操作

  脚本 script 操作复杂 操作一些复杂性较大的操作

# 系统命令行介绍

```bash
1、ping
	格式：
    	ping [网址]

2、主机登录用户信息
	[root@localhost ~]#   :  #表示超级用户管理员命令提示符，注释
	[test@localhost ~]		:  $普通用户命令提示符

    root  		:  登录当前系统的用户名
    @     		:  表示分隔符，没有特殊含义
    localhost :  表示当前系统的主机名
    ~     		:  表示当前所在的目录（~ 代表的是当前的家目录， /root）
    []	 	    :  表示括号，没有其他的作用
    #			    :  没有实际含义

3、自定义系统登录用户信息
	PS1 环境变量

	[root@localhost ~]# echo $PS1
  [\u@\h \W]\$
  [root@localhost ~]# PS1='[\u@\h --- \W]\$'


知识储备：
	print('Hello World')
	echo "Hello World!"
```

# 系统命令语法格式

```bash
通常系统命令语法格式：
一条完整命令
	命令      [参数] 				[选项] 			[路径]

command		[arguments]  [options]

1、中括号内的内容是可有可无的，选项和参数不是必须的
2、命令是指令的主体，是必须存在的
3、选项是用于调节命令的某个功能
		引导短格式（单个字符）	以短横杠表示‘-’	例如	-l
		引导长格式（多个字符）	多个字符表示一定的含义	以‘--’表示		--all
		多个短格式（多个字符）	每个字符都有一定的功能，‘-’	-al
4、参数是命令操作的对象，文件或者目录
5、指令、选项、参数两两之间必须要有一个空格
6、完整的命令、选项、参数之间不能有空格
7、命令的位置是在最前面的，是不能改变位置的
8、选项和参数的位置是可以发生改变的
```

# 系统运行命令

## 1、关机

```bash
同步时间：
yun install -y ntpdate
ntpdate ntp.aliyun.com


shutdown	：关机或重启
			参数：-h : 指定关机的延时时间
			   	 -c : 取消关机


  关机/取消：
    shutdown -h 10	# 10是以分钟为节点的
    shutdown -h 11:00	# 定时关机
    shutdown -c	# 取消你的关机操作


  立即关机:
    shutdown -h now  # 立刻关机
    shutdown -h 0  # 立刻关机



halt		:  禁用CPU资源
halt -p # 立刻关机，不加-p只关闭系统

poweroff	： 立即关闭电源


init	：设置系统启动模式
	参数： 0 ： 立刻关机
        1 ： 单用户模式
        2 ： 多用户无网络模式
        3 ： 多用户模式
        4 ： 待定
        5 ： 桌面模式
        6 ： 重启
```

## 2、重启

```bash
shutdown
		参数：
			-r : 指定重启的延时时间

    shutdown -r 10 # 10分钟后立刻重启
    shutdown -r 0	 # 立即重启
    shutdown -r now	# 立即重启
    shutdown -r 11:00	# 11:00重启

reboot
		reboot	# 系统推荐的重启操作
```

## 3、注销

```bash
logout	: 退出当前登录的用户	只能退出登录式shell，不能退出非登陆式shell

ctrl+d	: 快捷键	退出当前登录的用户

exit	  :  退出当前登录的用户	能退出登录式shell，也能退出非登陆式shell，主要用于脚本退出
```

# 查看系统命令帮助

```bash
格式：
	man [需要查看帮助的命令]    ：详细的显示一个命令的使用方法

		命令解释说明信息：NAME
		命令语法说明信息：SYNOPSIS
		命令描述详细说明：DESCRIPTION
		命令参数详细说明：OPTIONS

	q : 退出
	/[搜索内容] ： 搜索内容

	推荐网址：https://www.linuxcool.com/
```

# 设置别名

```bash
alias
# 格式：
[root@localhost ~]# alias alias net_test = 'ping baidu.com'   #设置别名

[root@localhost ~]# alias  #查看系统别名是否设置成功

[root@localhost ~]# net_test  #测试别名

[root@localhost ~]# unalias net_test  #取消别名

[root@localhost ~]# alias rm='xxx'   #设置系统别名

# 不使用别名，就在命令之前增加\，\代表转义
	[root@localhost ~]# \rm 1.txt
```

# 系统路径的类型

- 绝对路径：参照物是根（/）路径，凡是以/开始的路径就是绝对路径 或者以~为开头的路径也是绝对路径
- 相对路径：参照物是当前路径，不是以/开头的路径就是相对路径 针对当前路径而言的

```bash
# 包含整个文件名称及文件的位置	这样的定位称之为路径
# 路径就是对于文件的定位的一种方式
# 每个目录下都有一个.和..

.	    # 表示的是当前所在的目录
..	  # 当前目录的上一级目录
./	  # 用于表示当前目录
../	  # 从当前目录的上一级目录开始
~     # 家目录
```

# 系统目录结构

**在 Linux 中，所有的文件或者目录的起点或者顶点都是以(/)开始。**

![](https://gitee.com/gengff/blogimage/raw/master/images/image-20211213184825727.png)

**Linux 的目录结构拥有层次，就像是一个倒挂的树形结构**

![xitongmulu](https://gitee.com/gengff/blogimage/raw/master/images/xitongmulu.jpg)

Linux**系统中的目录需要挂载使用**

#### 目录挂载初识

```bash
挂载的命令：mount
   mount [磁盘路径] [挂载的路径]

查看本机挂载的命令
   [root@localhost dev]# df -h

卸载挂载的磁盘
   [root@localhost dev]# umount /mnt/
```

**必知必会的目录及文件**

- **/bin**
  bin 是 Binaries (二进制文件) 的缩写, 这个目录存放着最经常使用的命令。

- **/sbin：**存放系统命令的目录 需要管理员权限才可以执行的命令

- **/boot**
  这里存放的是启动 Linux 时使用的一些核心文件，包括一些连接文件以及镜像文件。

- **/dev**
  dev 是 Device(设备) 的缩写, 该目录下存放的是 Linux 的外部设备，在 Linux 中访问设备的方式和访问文件的方式是相同的。

  - ```
    /dev/cdrom	#光盘镜像
    /dev/null	#黑洞设备	将一些不用的数据导入到黑洞设备
    /dev/zero	#字符设备	会源源不断的产生数据，字符
    /dev/random	#产生随机数的设备

    #磁盘设备及分区
    /dev/sda
    /dev/sda1
    /dev/sda2
    /dev/sda3
    ```

- **/etc**
  etc 是 Etcetera(等等) 的缩写,这个目录用来存放所有的系统管理所需要的配置文件和子目录。

  - ```
    /etc/sysconfig/network-scripts/ifcfg-*	#查看网卡配置文件
    /etc/hosts#	本地域名解析文件	#记录ip地址与主机名的对应映射关系
    /etc/resolv.conf	#本地DNS配置文件
    /etc/fstab	#挂载设备目录配置文件	开机自启动挂载列表
    /etc/hostname	#主机名字配置文件
    ```

- **/home**
  用户的主目录，在 Linux 中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的，如上图中的 alice、bob 和 eve。

- **/lib**
  lib 是 Library(库) 的缩写这个目录里存放着系统最基本的动态连接共享库，其作用类似于 Windows 里的 DLL 文件。几乎所有的应用程序都需要用到这些共享库。

  - /lib #库文件目录 32 位库文件
    /lib64 #库文件目录 64 位库文件

- **/lost+found**
  这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。

- **/media**
  linux 系统会自动识别一些设备，例如 U 盘、光驱等等，当识别后，Linux 会把识别的设备挂载到这个目录下。

- **/mnt**
  系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在 /mnt/ 上，然后进入该目录就可以查看光驱里的内容了。

- **/opt**
  opt 是 optional(可选) 的缩写，这是给主机额外安装软件所摆放的目录。比如你安装一个 ORACLE 数据库则就可以放到这个目录下。默认是空的。

- **/proc**

  虚拟可变的目录 记录了系统的实时状态-->类似于汽车的仪表盘

- **/root**
  该目录为系统管理员，也称作超级权限者的用户主目录。

- **/sbin**
  s 就是 Super User 的意思，是 Superuser Binaries (超级用户的二进制文件) 的缩写，这里存放的是系统管理员使用的系统管理程序。

- **/selinux**
  这个目录是 Redhat/CentOS 所特有的目录，Selinux 是一个安全机制，类似于 windows 的防火墙，但是这套机制比较复杂，这个目录就是存放 selinux 相关的文件的。

```bash
关闭selinux
   临时关闭
    [root@localhost ~]# setenforce 0
   永久关闭
    [root@localhost ~]# vim /etc/selinux/config
    SELINUX=disabled  #编辑改成永久关闭
```

- **/srv**
  物理设备所产生的一些文件

- **/sys**

  物理设备的驱动信息文件

  这是 Linux2.6 内核的一个很大的变化。该目录下安装了 2.6 内核中新出现的一个文件系统 sysfs 。

  sysfs 文件系统集成了下面 3 种文件系统的信息：针对进程信息的 proc 文件系统、针对设备的 devfs 文件系统以及针对伪终端的 devpts 文件系统。

  该文件系统是内核设备树的一个直观反映。

  当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。

- **/tmp**
  公共临时目录 公共场所 只能针对自己的文件进行操作 系统会定时的删除这个目录下长时间没有访问的文件

- **/usr**

  /usr #系统目录 系统文件目录 跟 windows 目录一样
  /userlocal #系统软件安装目录 跟 windows 的一样

- \*\*/usr/bin
  系统用户使用的应用程序。

- \*\*/usr/sbin
  超级用户使用的比较高级的管理程序和系统守护程序。

- \*\*/usr/src
  内核源代码默认的放置目录。

- **/var**

  这是一个非常重要的目录，系统上跑了很多程序，那么每个程序都会有相应的日志产生，而这些日志就被记录到这个目录下，具体在 /var/log 目录下，另外 mail 的预设放置也是在这里。

  ```
  /var/log	#系统日志存放目录
  /var/log/messages	#系统级别日志
  /var/log/secure	#用户登录日志
  /var/tmp	#程序运行时所产生的一些进程文件
  ```

- **/run**
  是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。

在 Linux 系统中，有几个目录是比较重要的，平时需要注意不要误删除或者随意更改内部文件。

**/etc**： 上边也提到了，这个是系统中的配置文件，如果你更改了该目录下的某个文件可能会导致系统不能启动。

**/bin, /sbin, /usr/bin, /usr/sbin**: 这是系统预设的执行文件的放置目录，比如 ls 就是在 /bin/ls 目录下的。

值得提出的是，/bin, /usr/bin 是给系统用户使用的指令（除 root 外的通用户），而/sbin, /usr/sbin 则是给 root 使用的指令。

- **stderr -> /proc/self/fd/2 #错误输出 2>**
  **stdin -> /proc/self/fd/0 #标准输入<**
  **stdout -> /proc/self/fd/1 #标准输出>**
