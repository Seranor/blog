---
title: MySQL二进制安装
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-01.html
toc: true
---

## Centos7 下二进制安装

<!-- more -->

```shell
# 下载地址
# https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/

# 关闭防火墙及selinux
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i  s#enforcing#disabled#g /etc/selinux/config

# 卸载mariadb相关软件包
yum remove mariadb* -y

# 下载依赖包
yum install  libaio-devel -y

# 下载tar包
wget https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/mysql-5.7.34-el7-x86_64.tar.gz

# 解压
tar xf mysql-5.7.31-el7-x86_64.tar.gz
mv mysql-5.7.31-el7-x86_64 /usr/local/mysql

# 创建mysql用户
useradd -s /sbin/nologin mysql

#添加环境变量
echo 'export PATH=/usr/local/mysql/bin:$PATH' >>/etc/profile
source /etc/profile

# mysql软件目录授权mysql用户
chown -R mysql:mysql /usr/local/mysql

# 查看版本
mysql -V
```

数据目录

```shell
# 数据库数据目录应该单独用一块盘挂载
# mkfs.xfs /dev/sdb
# vim /etc/fstab
# mount -a

# 这边测试就直接创建目录了
mkdir /data/mysql/data -p
chown -R mysql:mysql /data

# 初始化
/usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data

# 参数说明
--initialize-insecure  # 使用空密码,后续手动添加密码即可
--initialize           # 会产生一个临时密码
--user                 # 指定用户
--basedir              # 指定软件路径
--datadir              # 指定数据路径


# 最简配置文件
cat >/etc/my.cnf <<EOF
[mysqld]
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=6
port=3306
[mysql]
socket=/tmp/mysql.sock
EOF

# 启动配置
# 方法一: mysql的脚本启动文件
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
service mysqld start  # 该方法启动命令
service mysqld stop  # 该方法停止命令

# 方法二: systemd启动配置
cat >/etc/systemd/system/mysqld.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
EOF

# 使用systemd启动和检查
systemctl start mysqld.service
systemctl status mysqld.service
netstat -lntup |grep 3306

# 数据库启动错误分析
 without updating PID 类似错误
查看日志：
	/data/mysql/data/主机名.err
	[ERROR] 上下文
可能情况：
	/etc/my.cnf 路径不对等
	/tmp/mysql.sock文件修改过 或 删除过
	数据目录权限不是mysql
	参数改错了

# 调试命令
mysqld --default-file=/etc/my.cnf
```
