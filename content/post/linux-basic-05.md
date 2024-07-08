---
title: Linux文件管理
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-05.html
toc: true
---

# 文件管理

#### **Linux 系统的单根⽬录结构**

linux 与 windows 的⽬录结构对⽐

<!-- more -->

![ ](https://gitee.com/gengff/blogimage/raw/master/images/2321466-20210620230259931-1963226798.png)

#### **⽂件的时间**

```d
ls -l ⽂件名 仅看的是⽂件的修改时间
Linux⽂件有三种时间,⽤stat查看

例如：stat anaconda-ks.cfg

访问时间：atime，查看内容，⽤cat检测
修改时间：mtime，修改内容
改变时间：ctime，修改内容，修改权限等属性，凡是有改动都会变
```

## 文件类型概念说明

**1、文件详细信息详解**

```bash
[root@localhost ~]# ls -lhi
 总用量 8K
 134319695 -rw-------. 1 root root 1.7K Dec 8 12:08 anaconda-ks.cfg
 134319707 -rw-r--r--  1 root root  12 Dec 13 11:48 index
```

文件属性信息详述图

![image-20211216191817853](https://gitee.com/gengff/blogimage/raw/master/images/image-20211216191817853.png)

**2、inode 编号**

- Linux 系统中文件的唯一编号，就相当于身份证号。

```bash
Linux系统内部不使用文件名，而使用inode编号来识别文件
  对于系统来说，文件名只是inode编号便于识别的别称或者绰号，表面上用户通过文件名打开文件
  实际上系统内部这个过程分成立三步：
  		首先：系统找到这个文件名对应的inode编号
  		其次：通过ionde编号获取inode信息
  		最后：根据ionde信息找到文件数据所在的block，读出数据

  使用ls -i命令可以看到文件对应额inode编号
   [root@localhost data]# ls -i
    16781387 test
```

## 硬链接和软链接

**1、什么是硬链接**

- 硬链接：不具有完整的文件结构，它的文件名直接指向文件节点，它和源文件节点一致。
  - 硬链接用来指向(保存)inode 编号。

**2、什么是软链接**

- 软链接：具有完整的文件结构，最后指向的是目标文件名，它和源文件节点不一致。
  - 相当于 Windows 中的快捷方式，主要用来指向(保存)对应文件的路径。

**3、创建命令**

- **ln** 默认创建的就是硬链接
  - 参数： **-s** 创建的就是软链接

```bash
# 硬链接示例：ln [源文件] [链接文件名]
 [root@localhost test]# echo 'hello world' >> a.txt  #创建源文件添加数据

 [root@localhost test]# ls -i 1.txt  #查看的inode编号
  16781390 1.txt

 [root@localhost test]# ln 1.txt 2.txt   #创建硬链接
 [root@localhost test]# ls -i 1.txt
  16781390 1.txt
 [root@localhost test]# ls -i 2.txt  #两个文件的inode编号一模一样，数据也一样
  16781390 2.txt

# 软链接示例：ln -s [源文件] [链接文件名]
 [root@localhost haha]# echo '123' >> a.txt  #创建源文件添加数据

 [root@localhost haha]# ls -i a.txt  #查看的inode编号
  33712451 a.txt

 [root@localhost haha]# ln -s a.txt b.txt  #创建软链接
 [root@localhost haha]# ls -i a.txt
  33712451 a.txt
 [root@localhost haha]# ls -i b.txt  #两个文件的inode编号不一样，数据也一样
  33712452 b.txt


# 删除文件的底层逻辑
   1、删除的是硬链接
   2、判断该文件硬链接数是否为0
   3、如果为0，则在磁盘中将其删除
   4、如果不为0，则只删除一个硬链接


# 删除源文件软链接和硬链接的影响
查看软链接文件，查看的文件不存在。和windows一样，删除源文件，快捷方式也用不了。但是删除源文件，为什么硬链接文件还可以查看呢？
这里要简单说下i节点了。i节点是文件和目录的唯一标识，每个文件和目录必有i节点，不然操作系统就无法识别该文件或系统，就像没有上户口的黑户。linux操作系统是不识别些字母的。

通俗理解：
硬链接文件相当于文件硬链接数+1，在windows里没这个概念，删除文件删除的是硬链接数，硬链接数为0时，数据就没了
软连接就是指向文件的路径，文件删除了，路径就不存在了，所以软连接找不到了
```

## 文件类型

```bash
Linux⽂件没有扩展名！！！

#⽅法⼀：
 ls -l ⽂件名  #看第⼀个字符
- #普通⽂件（⽂本⽂件，⼆进制，压缩⽂件，电影，图⽚。。。）,例如：/bin/ls
   [root@localhost ~]# ls -l
   -rw-r--r-- 1 root root   0 12月 14 19:11 1

d #⽬录⽂件，例如/home/
   [root@localhost home]# ls -l
   drwx------ 2 test test 63 12月 14 19:13 test


b #设备⽂件（块设备）存储设备硬盘，U盘，例如：/dev/sda
   [root@localhost dev]# ll
   brw-rw---- 1 root disk    8,   0 12月 16 19:39 sda


c #设备⽂件（字符设备）打印机，例如：/dev/ttycc
   [root@localhost dev]# ll
   crw-rw-rw- 1 root tty     5,   0 12月 16 19:39 tty


s #套接字⽂件（socket），例如： /var/lib/mysql/
   [root@localhost mysql]# ll
   srwxrwxrwx 1 mysql mysql   0 12月 16 21:52 mysql.sock


p #管道⽂件，例如：/run/systemd/initctl/fifo
   [root@localhost initctl]# ll
   prw------- 1 root root 0 12月 16 19:39 fifo

l #链接⽂件，例如：/bin
   [root@localhost bin]# ll
   lrwxrwxrwx.   1 root root   6 12月 13 11:40 apropos -> whatis


ps:通过颜⾊判断⽂件的类型是错误的！！！

# 准备套接字文件
  #安装mysql数据库
	 [root@localhost run]# yum install mariadb* -y
	#启动
   [root@localhost run]# systemctl start mariadb



# ⽅法⼆：大致判断文件的类型
[root@xxx ~]# file /etc/krb5.conf
  /etc/krb5.conf: ASCII text

.conf #配置文件
.log  #日志文件
.sh   #脚本文件
.py   #脚本文件
```
