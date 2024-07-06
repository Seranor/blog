---
title: MySQL初始化配置
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-05.html
toc: true
---

### 作用

```mysql
控制MySQL的启动
影响到客户端的连接
```

<!-- more -->

### 初始化配置的方法

```mysql
初始化配置文件(例如/etc/my.cnf)
启动命令行上进行设置(例如:mysqld_safe mysqld)
预编译时设置(仅限于编译安装时设置)
```

### 初始化配置文件

初始化配置文件的默认读取路径

```mysql
# 默认配置文件读取顺序
mysqld --help --verbose |grep my.cnf
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf

默认情况下，MySQL启动时，会依次读取以上配置文件，如果有重复选项，会以最后一个文件设置的为准
如果启动时加入了--defaults-file=xxxx时，以上的所有文件都不会读取
```

配置文件的书写方式

```mysql
格式:
    [标签]
    配置项=xxxx

    标签类型：服务端、客户端
    服务器端标签：
    [mysqld]
    [mysqld_safe]
    [server]

    客户端标签：
    [mysql]
    [mysqldump]
    [client]

配置文件的示例：
    cat /etc/my.cnf
    [mysqld]
    user=mysql  # 用户
    basedir=/usr/local/mysql  # 软件目录
    datadir=/data/mysql/data  # 数据目录
    socket=/tmp/mysql.sock  # socket位置
    server_id=6  # 服务器ID号
    port=3306  # 端口
    log_error=/data/mysql/mysql.log  # 日志位置
    [mysql]
    socket=/tmp/mysql.sock
```

### 多实例配置

```bash
# 数据目录准备
mkdir /data/330{7,8,9}/data -p
chown -R mysql.mysql /data/*
tree -L 2 /data/
```

```bash
# 配置文件
cat > /data/3307/my.cnf <<EOF
[mysqld]
basedir=/usr/local/mysql
datadir=/data/3307/data
socket=/data/3307/mysql.sock
log_error=/data/3307/mysql.log
port=3307
server_id=7
log_bin=/data/3307/mysql-bin
EOF

cat > /data/3308/my.cnf <<EOF
[mysqld]
basedir=/usr/local/mysql
datadir=/data/3308/data
socket=/data/3308/mysql.sock
log_error=/data/3308/mysql.log
port=3308
server_id=8
log_bin=/data/3308/mysql-bin
EOF

cat > /data/3309/my.cnf <<EOF
[mysqld]
basedir=/usr/local/mysql
datadir=/data/3309/data
socket=/data/3309/mysql.sock
log_error=/data/3309/mysql.log
port=3309
server_id=9
log_bin=/data/3309/mysql-bin
EOF
```

```bash
# 移动原来的配置文件
mv /etc/my.cnf{,.bak}

# 初始化三套数据库
mysqld --initialize-insecure  --user=mysql --datadir=/data/3307/data --basedir=/usr/local/mysql
mysqld --initialize-insecure  --user=mysql --datadir=/data/3308/data --basedir=/usr/local/mysql
mysqld --initialize-insecure  --user=mysql --datadir=/data/3309/data --basedir=/usr/local/mysql

# 启动文件配置
cp /etc/systemd/system/mysqld.service  /etc/systemd/system/mysqld3307.service
cp /etc/systemd/system/mysqld.service  /etc/systemd/system/mysqld3308.service
cp /etc/systemd/system/mysqld.service  /etc/systemd/system/mysqld3309.service

# 修改相关启动文件
vim /etc/systemd/system/mysqld3307.service
ExecStart=/application/mysql/bin/mysqld  --defaults-file=/data/3307/my.cnf

vim /etc/systemd/system/mysqld3308.service
ExecStart=/application/mysql/bin/mysqld  --defaults-file=/data/3308/my.cnf

vim /etc/systemd/system/mysqld3309.service
ExecStart=/application/mysql/bin/mysqld  --defaults-file=/data/3309/my.cnf

# 启动
systemctl start mysqld3307.service
systemctl start mysqld3308.service
systemctl start mysqld3309.service

# 检查验证多实例
ps -ef |grep mysql
netstat -lntup|grep 330

mysql -S /data/3307/mysql.sock  -e 'select @@server_id'
mysql -S /data/3308/mysql.sock  -e 'select @@server_id'
mysql -S /data/3309/mysql.sock  -e 'select @@server_id'
```
