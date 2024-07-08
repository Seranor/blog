---
title: Linux-命令记录-01
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-cmd-01.html
toc: true
---

## Ubuntu 扩容 lvm

```
lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

<!-- more -->

## 测试硬盘灯

```shell
time dd if=/dev/sdb of=/dev/null bs=4k

for i in `lsblk |grep -w sd[a-z] |grep T  |awk '{print $1}'`
do
	nohup  dd if=/dev/$i of=/dev/null bs=4k > /tmp/${i}.log   &
done
```

## CPU 高压测试

```
for i in `seq 1 30`; do dd if=/dev/zero of=/dev/null & done
ps -ef |grep -v grep|grep 'dd if=/dev/zero of=/dev/null'|awk '{print $2}'|xargs kill -9


top
Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

id 前面数值是空闲率
```

## ip 排序

```bash
 sort -t . -k 4,4n tt
 sort -t'.' -k1,1n -k2,2n -k3,3n -k4,4n
```

## mdadm 软 raid

```shell
mdadm -C /dev/md1 -l raid0 -n 2 /dev/nvme2n2 /dev/nvme3n1
```

## dpkg-error

```bash

dpkg: error processing package

mv /var/lib/dpkg/info /var/lib/dpkg/info.bak
mkdir /var/lib/dpkg/info
apt update

apt install sl



mv /var/lib/dpkg/info       /var/lib/dpkg/info.ori
mv /var/lib/dpkg/info.bak   /var/lib/dpkg/info

apt install lrzsz

```

## CPU 温度查看

```shell

apt-get install lm-sensors sensors-applet -y
yes| sensors-detect

cat /sys/class/hwmon/hwmon0/device/hwmon/hwmon0/temp1_input

sensors
```

## 无交互修改密码

```bash
echo "root:test123."|chpasswd
```

## 开启 yum 缓存

```shell
vim /etc/yum.conf
#修改为1
keepcache=1

#默认存放目录在/var/cache
```

## find 找出的移动或删除

```shell
find ./ -name 'means'|xargs -i mv {}  /k-means/
find ./ -name 'means'|xargs -i cp {}  /k-means/

#找出小于90G的文件
find ./ -type f   -size -90G
```

##

## -bash 错误修复

```shell
-bash-4.2$
cp /etc/skel/.bashrc  /home/user/
cp /etc/skel/.bash_profile   /home/user
```

## useradd

```shell
#指定目录
-d
```

## 设置免密

```shell
ssh-keygen -f ~/.ssh/id_rsa  -P '' -q
sshpass -p123456

ssh-copy-id -f -i ~/.ssh/id_rsa.pub "-o StrictHostKeyChecking=no" 10.0.0.100
```

## rpm 管理命令

```shell
rpm -ql  nginx		#列出所有相关目录
rpm -qc  nginx		#列出配置目录
rpm -e	 nginx		#单独卸载
```

## 输出格式化整理

```bash
column -t    #格式化整理
```

## 自动补全命令

```shell
yum install bash-completion #自动补全命令
```

## VMware 相关

```shell
#VM安装linux在docker中装mysql挂起再启动后无法连接解决方法
vim /usr/lib/sysctl.d/00-system.conf
net.ipv4.ip_forward = 1

#重启网络服务
systemctl restart network

#查看IPv4转发状态
sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

## history 设置

### 历史命令显示时间

```shell
#写入/etc/bashrc或者/etc/profile
HISTFILESIZE=4000 #默认保存命令是1000条，这里修改为4000条
HISTSIZE=4000
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'` #取得登录客户端的IP
if [ -z $USER_IP ]
then
USER_IP=`hostname`
fi
HISTTIMEFORMAT="%F %T $USER_IP:`whoami` " #设置新的显示history的格式
export HISTTIMEFORMAT

. /etc/bashrc
```

### 记录用户 bash

```bash
#####记录用户bash######################################################
history
USER=`whoami`
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
if [ "$USER_IP" = "" ]; then
USER_IP=`hostname`
fi
if [ ! -d /var/log/history ]; then
mkdir /var/log/history
chmod 777 /var/log/history
fi
if [ ! -d /var/log/history/${LOGNAME} ]; then
mkdir /var/log/history/${LOGNAME}
chmod 300 /var/log/history/${LOGNAME}
fi
export HISTSIZE=4096
DT=`date +"%Y%m%d_%H:%M:%S"`
export HISTFILE="/var/log/history/${LOGNAME}/${USER}@${USER_IP}_$DT"
chmod 600 /var/log/history/${LOGNAME}/*history* 2>/dev/null
#######################################################################
```

## vim 设置

```shell
#TAB 键为四个空格 永久显示行号
vim /etc/vimrc
set ts=4
set sw=4
set number
```

## 分区扩容

```shell
umount /data/
fdisk /dev/vdb
d
n
p

w

e2fsck -f /dev/vdb1
resize2fs /dev/vdb1
mount /dev/vdb1 /data/
df -h
cd /data/
ls
```

## 随机数生成

```shell
openssl rand -base64 3
```

## tcpdump 使用

```shell
参数：
-i 指定网卡
-c 指定抓包数量
```

## 时间同步

```shell
#如果不是北京时间先改成北京时间
1.删除自带的localtime
  rm -rf /etc/localtime
2.创建软链接到localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#同步阿里云
ntpdate ntp.aliyun.com

#写入硬件主板
hwclock -w
```

## 终端颜色

```bash
#写入到环境变量内
RED:
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;31;40m\]\u\[\033[00;00;40m\]@\[\033[01;31;40m\]\h\[\033[00;31;40m\]:\[\033[00;00;40m\]\w \[\033[01;32;40m\]\$ \[\033[01;37;40m\]'


YELLOW:
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;33;40m\]\u\[\033[00;00;40m\]@\[\033[01;33;40m\]\h\[\033[00;33;40m\]:\[\033[00;00;40m\]\w \[\033[01;32;40m\]\$ \[\033[01;37;40m\]'
```

## linux 格式问题

```shell
#检查文件格式，如果带M即是Windows，需要使用dosunix转换
cat -v filename
apt install dosunix -y
yum install dosunix -y
dos2unix filename

参考：
https://www.cnblogs.com/chuyiwang/p/13823551.html
```

## 文件分割

```bash
split
```

## hexo 插件

```bash
npm install hexo-generator-searchdb --save
npm install hexo-deployer-git --save
npm install hexo-generator-cname
npm install hexo-generator-search --save
npm install hexo-abbrlink --save
```
