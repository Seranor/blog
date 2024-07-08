---
title: Linux系统优化及定时任务
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-09.html
toc: true
---

# 系统安全优化

## 关闭 selinux 安全服务

selinux（Security-Enhanced Linux）是美国国家安全局（NSA）对于强制访问控制的实现，这个功能让系统管理员又爱又恨，这里我们还是把它给关闭了吧，至于安全问题，后面通过其他手段来解决，这也是大多数生产环境的做法，如果非要开启也是可以的。

<!-- more -->

```bash
#临时关闭
[root@localhost ~]# setenforce  0

#永久关闭,修改完配置后重启主机
[root@localhost ~]# sed 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config

#检查结果
[root@localhost ~]# grep "disabled" /etc/selinux/config
```

## 关闭 firewalld 防火墙

关闭防火墙的目的是为了让初学者学习更方便，将来在学了 firewalld 技术后可再统一开启。 在企业环境中，
一般只有配置外网 IP 的 linux 服务器才需要开启防火墙，但即使是有外网 IP，对于高并发高流量的业务服务器
仍是不能开的，因为会有较大性能损失，导致网站访问很慢，这种情况下只能在前端加更好的硬件防火墙了。

```bash
# 临时关闭
[root@egon ~]# systemctl stop firewalld

# 设置开机不启动
[root@egon ~]# systemctl disable firewalld



系统优化脚本命令：
yum install wget -y && wget https://daima.baim0.xyz/init.sh && chmod +x init.sh && ./init.sh

脚本优化内容：

  删除初始源
  阿里centos7源
  阿里epel源
  nginx官方stable源
  清除缓存
  建立新缓存
  更新软件
  安装常用软件
  关闭selinux
  关闭防火墙
  系统参数优化
  ssh优化
  关闭不常用的东西
```

![image-20211221164241000](https://gitee.com/gengff/blogimage/raw/master/images/image-20211221164241000.png)

# 防止系统乱码优化

```bash
en_US.UTF-8		: 美式英文，utf-8
zh_CN.UTF-8
zh_HK.UTF-8
# 查看当前使用的系统语言
 [root@localhost ~]# echo $LANG
  zh_CN.UTF-8

# 查看安装的语言包
 [root@localhost ~]# locale
  LANG=zh_CN.UTF-8

# 临时优化
 [root@localhost ~]# export LANG=zh_CN.UTF-8  #设置编码

# 永久优化
 [root@localhost ~]# vim /etc/locale.conf
  LANG="zh_CN.UTF-8"  #将原来的配置内容修改，注销或重启后，中文的语言环境。
```

# 定时任务

## 计划任务基本概述

#### 1.什么是 crond

- crond 就是计划任务，类似于我们平时生活中的闹钟。定点执行。

#### 2.为什么要使用 crond

- crond 主要是做一些周期性的任务，比如: 凌晨 3 点定时备份数据。比如：11 点开启网站抢购接口，12 点关闭网站抢购接口。

#### 3.计划任务主要分为以下两种使用情况:

- 1.系统级别的定时任务： 临时文件清理、系统信息采集、日志文件切割
- 2.用户级别的定时任务： 定时向互联网同步时间、定时备份系统配置文件、定时备份数据库的数据

## 计划任务时间管理

#### 1.Crontab 配置文件记录了时间周期的含义

```bash
[root@localhost ~]# vim /etc/crontab
SHELL=/bin/bash                     #执行命令的解释器
PATH=/sbin:/bin:/usr/sbin:/usr/bin  #环境变量
MAILTO=root                         #邮件发给谁
# Example of job definition:
# .---------------- minute (0 - 59) #分钟
# |  .------------- hour (0 - 23)   #小时
# |  |  .---------- day of month (1 - 31)   #日期
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr #月份
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat  #星期
# |  |  |  |  |
# *  *  *  *  *   command to be executed

# *  表示任意的(分、时、日、月、周)时间都执行
# -  表示一个时间范围段, 如5-7点
# ,  表示分隔时段, 如6,0,4表示周六、日、四
# /1 表示每隔n单位时间, 如*/10 每10分钟
```

#### 2.了解 crontab 的时间编写规范

```bash
00 02 * * * ls      #每天的凌晨2点整执行
00 02 1 * * ls      #每月的1日的凌晨2点整执行
00 02 14 2 * ls     #每年的2月14日凌晨2点执行
00 02 * * 7 ls      #每周天的凌晨2点整执行
00 02 * 6 5 ls      #每年的6月周五凌晨2点执行
00 02 14 * 7 ls     #每月14日或每周日的凌晨2点都执行
00 02 14 2 7 ls     #每年的2月14日或每年2月的周天的凌晨2点执行
*/10  02 * * * ls   #每天凌晨2点，每隔10分钟执行一次
* * * * *  ls       #每分钟都执行
00 00 14 2 *  ls    #每年2月14日的凌晨执行命令
*/5 * * * *  ls     #每隔5分钟执行一次
00 02 * 1,5,8 * ls  #每年的1月5月8月凌晨2点执行
00 02 1-8 * *  ls    #每月1号到8号凌晨2点执行
0 21 * * * ls       #每天晚上21:00执行
45 4 1,10,22 * * ls #每月1、10、22日的4:45执行
45 4 1-10 * * l     #每月1到10日的4:45执行
3,15 8-11 */2 * * ls #每隔两天的上午8点到11点的第3和第15分钟执行
0 23-7/1 * * * ls   #晚上11点到早上7点之间，每隔一小时执行
15 21 * * 1-5 ls    #周一到周五每天晚上21:15执行
```

#### 3.使用 crontab 编写 cron 定时任务

| 参数 | 含义         |
| :--- | :----------- |
| -e   | 编辑定时任务 |
| -l   | 查看定时任务 |
| -r   | 删除定时任务 |
| -u   | 指定其他用户 |

## 计划任务编写实践

#### 1.使用 root 用户每 5 分钟执行一次时间同步

```bash
#1.如何同步时间
[root@localhost ~]# ntpdate time.windows.com &>/dev/null
#2.配置定时任务
[root@localhost ~]# crontab -e -u root
[root@localhost ~]#  crontab -l -u root
*/5 * * * * ntpdate time.windows.com &>/dev/null
```

#### 2.每天的下午 3,5 点，每隔半小时执行一次 sync 命令

```bash
[root@localhost ~]# crontab -l
*/30 15,17 * * * sync &>/dev/null
```

#### 3.案例：每天凌晨 3 点做一次备份？备份/etc/目录到/backup 下面

\1) 将备份命令写入一个脚本中

\2) 每天备份文件名要求格式: 2019-05-01_hostname_etc.tar.gz

\3) 在执行计划任务时，不要输出任务信息

\4) 存放备份内容的目录要求只保留三天的数据\*

```bash
#1.实现如上备份需求
[root@localhost ~]# mkdir /backup
[root@localhost ~]# tar zcf $(date +%F)_$(hostname)_etc.tar.gz /etc
[root@localhost ~]# find /backup -name “*.tar.gz” -mtime +3 -exec rm -f {}\;

#2.将命令写入至一个文件中
[root@localhost ~]# vim /root/back.sh
mkdir /backup
tar zcf $(date +%F)_$(hostname)_etc.tar.gz /etc
find /backup -name “*.tar.gz” -mtime +3 -exec rm -f {}\;

#3.配置定时任务
[root@localhost ~]# crontab -l
00 03 * * * bash /root/back.sh  &>/dev/null

#3.备份脚本
```

#### 4.crond 注意的事项

\1) 给定时任务注释

\2) 将需要定期执行的任务写入 Shell 脚本中，避免直接使用命令无法执行的情况 tar date

\3) 定时任务的结尾一定要有&>/dev/null 或者将结果追加重定向>>/tmp/date.log 文件

\4) 注意有些命令是无法成功执行的 echo "123" >>/tmp/test.log &>/dev/null

**如果一定要是用命令，命令必须使用绝对路径**

#### 5.crond 如何备份

\1) 通过查找/var/log/cron 中执行的记录，去推算任务执行的时间

\2) 定时的备份/var/spool/cron/{usernmae}\*

#### 6.crond 如何拒绝某个用户使用

```bash
#1.使用root将需要拒绝的用户加入/etc/cron.deny
[root@localhost ~]# echo "xuliangwei" >> /etc/cron.deny

#2.登陆该普通用户，测试是否能编写定时任务
[test@localhost ~]$ crontab -e
You (test) are not allowed to use this program (crontab)
See crontab(1) for more information
```

## 计划任务如何调试

#### 1.crond 调试

\1) 调整任务每分钟执行的频率, 以便做后续的调试。

\2) 如果使用 cron 运行脚本，请将脚本执行的结果写入指定日志文件, 观察日志内容是否正常。

\3) 命令使用绝对路径, 防止无法找到命令导致定时任务执行产生故障。

\4) 通过查看/var/log/cron 日志，以便检查我们执行的结果，方便进行调试。

#### 2.crond 编写思路

- 1.手动执行命令，然后保留执行成功的结果。
- 2.编写脚本
  - 脚本需要统一路径/scripts
  - 脚本内容复制执行成功的命令(减少每个环节出错几率)
  - 脚本内容尽可能的优化, 使用一些变量或使用简单的判断语句
  - 脚本执行的输出信息可以重定向至其他位置保留或写入/dev/null
- 3.执行脚本
  - 使用 bash 命令执行, 防止脚本没有增加执行权限(/usr/bin/bash)
  - 执行脚本成功后，复制该执行的命令，以便写入 cron
- 4.编写计划任务
  - 加上必要的注释信息, 人、时间、任务
  - 设定计划任务执行的周期
  - 粘贴执行脚本的命令(不要手敲)
- 5.调试计划任务
  - 增加任务频率测试
  - 检查环境变量问题
  - 检查 crond 服务日志
