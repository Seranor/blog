---
title: Linux磁盘挂载分区
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-13.html
toc: true
---

# 磁盘管理

<!-- more -->

```bash
Linux系统中磁盘管理就是将硬盘通过挂载的方式挂载到linux文件系统中。

1、挂载磁盘的步骤
	1、磁盘分区
	2、挂载

2、磁盘分区
	fdisk：分区2TB以下的磁盘，最多可以分4个分区
	gdisk：分区2TB以上的磁盘，最多可以分128个分区

3、添加一块磁盘
	lsblk   ： 查看本机的磁盘
	df -h   :  查看本机的分区

4、磁盘分区
	n : 新建分区
	p : 打印分区表
	w : 写入磁盘保存并退出
	q : 不保存退出
	d : 删除分区

5、挂载磁盘分区
	1、格式化文件系统
		mkfs.xfs /dev/sdb1

6、总结
	1、关机
	2、添加硬盘
	3、创建分区
		fdisk /dev/sdb
		或
		gdisk /dev/sdb
	4、格式化文件系统
		mkfs.xfs /dev/sdb1
	5、挂载
		mount /dev/sdb1 /mnt
```

## 磁盘分区

- 逻辑分区属于扩展分区，扩展分区属于主分区
- 主分区又叫做引导分区，是可以安装系统的分区

![mbr](https://gitee.com/gengff/blogimage/raw/master/images/mbr.jpeg)

目前常见的磁盘分区格式有两种，MBR 分区和 GPT 分区：

- MBR 分区，MBR 的意思是 "主引导记录"。MBR 最大支持 2TB 容量，在容量方面存在着极大的瓶颈。
- GPT 分区，GPT 意为 GUID 分区表，它支持的磁盘容量比 MBR 大得多。这是一个正逐渐取代 MBR 的新标准，它是由 UEFI 辅住而形成的，将来 UEFI 用于取代老旧的 BIOS，而 GPT 则取代老旧的 MBR。

```bash
# 磁盘分区工具

fdisk 工具用于 MBR 格式
gdisk 工具用于 GPT 格式
```

### 磁盘基本分区 fdisk

**1.添加一块小于 2TB 的磁盘进行使用，步骤如下:**

- 1.给虚拟机添加一块新的硬盘

- 2.使用 fdisk 进行分区

- 3.使用 mkfs 进行格式化

- 4.使用 mount 进行挂载

  PS: 生产分区建议，如无特殊需求直接使用整个磁盘即可，无需分区。

```bash
[root@localhost ~]# fdisk -l
[root@localhost ~]# fdisk  /dev/sdb
Command (m for help): m         #输入m列出常用的命令
Command action
   a   toggle a bootable flag               #切换分区启动标记
   b   edit bsd disklabel                   #编辑sdb磁盘标签
   c   toggle the dos compatibility flag    #切换dos兼容模式
   d   delete a partition                   #删除分区
   l   list known partition types           #显示分区类型
   m   print this menu                      #显示帮助菜单
   n   add a new partition                  #新建分区
   o   create a new empty DOS partition table   #创建新的空白分区表
   p   print the partition table            #显示分区表的信息
   q   quit without saving changes          #不保存退出
   s   create a new empty Sun disklabel     #创建新的Sun磁盘标签
   t   change a partitions system id        #修改分区ID,可以通过l查看id
   u   change display/entry units           #修改容量单位,磁柱或扇区
   v   verify the partition table           #检验分区表
   w   write table to disk and exit         #保存退出
   x   extra functionality (experts only)   #拓展功能
```

1). fdisk 创建主分区

```bash
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)  #主分区
   e   extended  #扩展分区
Select (default p): p   #选择创建主分区
Partition number (1-4, default 1):  #默认创建第一个主分区
First sector (2048-2097151, default 2048): #默认扇区回车
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-2097151, default 2097151): +50M #分配50MB
```

2). fdisk 创建扩展分区

```bash
Command (m for help): n  #新建分区
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): e   #创建扩展分区
Partition number (2-4, default 2):
First sector (104448-2097151, default 104448):
Using default value 104448
Last sector, +sectors or +size{K,M,G} (104448-2097151, default 2097151): #空间都给到扩展分区
```

3). fdisk 创建逻辑分区

```bash
Command (m for help): n  #新建分区
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): l   #创建逻辑分区
Adding logical partition 5
First sector (106496-2097151, default 106496):
Using default value 106496
Last sector, +sectors or +size{K,M,G} (106496-2097151, default 2097151): +100M  #分配100MB空间
```

4). fdisk 查看分区情况，并保存

```bash
Command (m for help): p #查看分区创建
Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048      104447       51200   83  Linux
/dev/sdb2          104448     2097151      996352    5  Extended
/dev/sdb5          106496      311295      102400   83  Linux

#保存分区
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.

#检查磁盘是否是MBR分区方式
[root@localhost ~]# fdisk /dev/sdb -l|grep type
Disk label type: dos

#安装parted, 刷新内核立即生效,无需重启
[root@localhost ~]# yum -y install parted
[root@localhost ~]# partprobe /dev/sdb
```

**2.格式化磁盘**

- mkfs 格式化磁盘，实质创建文件系统，文件系统类似于将房子装修成 3 室一厅，还是 2 室一厅。

```bash
#选项:
# -b  设定数据区块占用空间大小，目前支持1024、2048、4096 bytes每个块。
# -t  用来指定什么类型的文件系统，可以是ext4, xfs
# -i  设定inode的大小
# -N  设定inode数量，防止Inode数量不够导致磁盘不足

#1.格式化整个磁盘
[root@localhost ~]# mkfs.ext4  /dev/sdb

#2.格式化磁盘的某个分区
[root@localhost ~]# mkfs.xfs  /dev/sdb1
```

**3.使用 mount 挂载并使用**

- 如果需要使用该磁盘的空间，需要准备一个空的目录作为挂载点，与该设备进行关联。

```bash
[root@localhost ~]# mkdir /data
[root@localhost ~]# mount /dev/sdb1 /data
```

### 磁盘的基本分区 Gdisk

前面我们已经了解到 fdisk 分区，但 fdisk 不支持给高于 2TB 的磁盘进行分区。如果有单块盘高于 2TB，建议使用 Gdisk 进行分区。

**1.使用 gdisk 进行磁盘分区**

```bash
#1.安装gdisk分区工具
[root@localhost ~]# yum install gdisk -y

#2.创建一个新分区，500MB大小
[root@localhost ~]# gdisk /dev/sdb
Command (? for help): n     #创建新分区
Partition number (1-128, default 1):
First sector (34-2097118, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-2097118, default = 2097118) or {+-}size{KMGTP}: +500M #分配500M大小

Command (? for help): p #打印查看
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1026047   500.0 MiB   8300  Linux filesystem

Command (? for help): w #保存分区
Do you want to proceed? (Y/N): y    #确认
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.

#3.创建完成后，可以尝试检查磁盘是否为gpt格式
[root@localhost-node1 /]# fdisk /dev/sdb -l|grep type
Disk label type: gpt

#4.安装parted, 刷新内核立即生效,无需重启
[root@localhost ~]# yum -y install parted
[root@localhost ~]# partprobe /dev/sdb
```

**2.使用 mkfs 进行格式化磁盘。**

```bash
[root@localhost ~]# mkfs.xfs  /dev/sdb
```

**3.使用 mount 命令将某个目录挂载该分区，进行使用。**

```bash
[root@localhost ~]# mkdir /data_gdisk
[root@localhost ~]# mount /dev/sdb /data_gdisk
```

## 磁盘挂载

前面我们已经提到过，如果需要使用磁盘的空间，需要准备一个空的目录作为挂载点，与该设备进行关联。mount 主要是为文件系统指定一个访问入口。

**1.通过 mount 进行挂载，但重启将会失效。我们称为临时生效。**

```bash
# 选项：-t指定文件系统挂载分区 -a 挂载/etc/fstab中的配置文件 -o 指定挂载参数
# 挂载/dev/sdb1至db1目录
 [root@localhost ~]# mkdir /db1
 [root@localhost ~]# mount -t xfs /dev/sdb1  /db1/

ps：centos7选择xfs格式作为默认文件系统，而且不再使用以前的ext，仍然支持ext4，
xfs专为大数据产生，每个单个文件系统最大可以支持8eb，单个文件可以支持16tb，不仅数据量大，
而且扩展性高。还可以通过xfsdump，xfsrestore来备份和恢复。
```

**2.挂载的磁盘，如果不想使用可以使用 umount 进行卸载。**

```bash
#选项： -l 强制卸载

#1.卸载目录方式
 [root@localhost ~]# umount /db1

#2.卸载设备方式
 [root@localhost ~]# umount /dev/sdb1

#3.umount不能卸载的情况
 [root@localhost db1]# umount /db1
  umount: /db1: device is busy.
          (In some cases useful info about processes that use
          the device is found by lsof(8) or fuser(1)

#PS: 如上情况解决办法有两种, 1.切换至其他目录 2.使用'-l'选项强制卸载
 [root@student db1]# umount -l /db1
```

**3.如果需要实现永久挂载则需要将挂载信息写入/etc/fstab 配置文件中实现。**

```bash
#1.使用blkid命令获取各设备的UUID
 [root@localhost ~]# blkid |grep "sdb1"
  /dev/sdb1: UUID="e271b5b2-b1ba-4b18-bde5-66e394fb02d9" TYPE="xfs"

#2.使用UUID挂载磁盘sdb1分区至于db1， 测试挂载
 [root@localhost ~]# mount UUID="e271b5b2-b1ba-4b18-bde5-66e394fb02d9" /db1

#3.写入/etc/fstab中，实现开机自动挂载
 [root@localhost ~]# tail -1 /etc/fstab
  UUID=e271b5b2-b1ba-4b18-bde5-66e394fb02d9 /db1 xfs  defaults 0  0

#4.加载fstab配置文件, 同时检测语法是否有错误
 [root@localhost ~]# mount –a
```

**4./etc/fstab 配置文件编写格式**

| 要挂载的设备 | 挂载点(入口) | 文件系统类型 | 挂载参数 | 是否备份 | 是否检查 |
| :----------- | :----------- | :----------- | :------- | :------- | :------- |
| /dev/sdb1    | /db1         | xfs          | defaults | 0        | 0        |

第四列：挂载参数

| 参数        | 含义                                                      |
| :---------- | :-------------------------------------------------------- |
| async/sync  | 是否为同步方式运行。默认 async                            |
| user/nouser | 是否允许普通用户使用 mount 命令挂载。默认 nouser          |
| exec/noexe  | 是否允许可执行文件执行。默认 exec                         |
| suid/nosuid | 是否允许存在 suid 属性的文件。默认 suid                   |
| auto/noauto | 执行 mount -a 命令时，此文件系统是否被主动挂载。默认 auto |
| rw/ro       | 是否以只读或者读写模式进行挂载。默认 rw                   |
| default     | 具有 rw,suid,dev,exec,auto,nouser,async 等默认参数的设定  |

第五列：是否进行备份。参数的值为 0 或者 1

| 选项 | 含义                       |
| :--- | :------------------------- |
| 0    | 代表不做备份               |
| 1    | 代表要每天进行备份操作     |
| 2    | 代表不定日期的进行备份操作 |

第六列：是否检验扇区：开机的过程中，系统默认会以 fsck 检验我们系统是否为完整

| 选项 | 含义                                        |
| :--- | :------------------------------------------ |
| 0    | 不要检验磁盘是否有坏道                      |
| 1    | 检验                                        |
| 2    | 校验 (当 1 级别检验完成之后进行 2 级别检验) |
