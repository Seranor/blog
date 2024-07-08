---
title: MySQL基础管理
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-04.html
toc: true
---

### 用户管理

作用: 登录、管理数据逻辑对象

<!-- more -->

定义:

```mysql
格式:
用户名@'白名单'

白名单支持的方式:
wordpress@'10.0.0.%'     # 10.0.0.0 段
wordpress@'%'            # 所有
wordpress@'10.0.0.200'   # 指定IP
wordpress@'localhost'    # 本地localhost
wordpress@'db02'         # 指定主机名,能解析该主机名
wordpress@'10.0.0.5%'    # 10.0.0.5x 段
wordpress@'10.0.0.0/255.255.254.0'  # 子网掩码方式
```

管理操作:

```mysql
# 创建用户
create user wordpress@'192.168.0.%' identified by '123';

# 查询用户
select user, host from mysql.user;

# 修改用户密码
 alter user wordpress@'192.168.0.%' identified by '456';

# 删除用户
drop user wordpress@'192.168.0.%';

# 创建用户密码以及授权操作
# 5.7版本 创建用户以及授权一步完成
grant all on *.* to wordpress@'192.168.0.%' identified by '123456';

# 8.0版本 必须先创建用户之后才能授权
create user wordpress@'%' identified by '123456';
grant all privileges on wordpress.* to wordpress@'%';
```

### 用户权限

#### 常用权限

```mysql
ALL:
SELECT,INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE
ALL : 以上所有权限，一般是普通管理员拥有的
```

#### 授权命令

```mysql
grant 权限 on 作用目标 to 用户 identified 密码 with grant option

作用目标：
*.*：代表授权所有库所有表权限
wordpress.*：代表授权 wordpress 库下的所有表权限
worpress.t1：代表授权 wordpress

with grant option：超级管理员才具备的，给别的用户授权的功能
```

#### 授权相关

```mysql
# 创建一个管理员用户root,可以通过192.168.0网段登录,管理所有数据库
grant all  on *.* to root@'192.168.0.%' identified by '123456';

# 创建一个应用用户app用户,可以远程登录mysql,并能操作app库
grant select,update,insert,delete on app.* to app@'%' identified by '123456';
```

#### 查看授权

```mysql
show grants for root@'%';
show grants for app@'%'';
```

#### 回收权限

```mysql
# 回收app用户对app库下所有表的删除的权限
revoke delete on app.* from app@'%';

# 回收app用户对app库下所有表查询 更新 插入权限
revoke select,update,insert  on app.* from app@'%';
```

### 连接管理

- 常用参数

```mysql
-u                   用户
-p                   密码
-h                   IP
-P                   端口
-S                   socket文件
-e                   免交互执行命令
<                    导入SQL脚本

# 远程连接
mysql -uroot -p -h 192.168.0.11 -P3306

# 免交互执行sql语句
mysql -uroot -p1 -e "select user,host from mysql.user;"

# 导入sql脚本
mysql -uroot -p1 <world.sql

# 查看socket
mysql> select @@socket;

# 查看连接情况
mysql> show processlist;

# 内置命令
help     打印 mysql 帮助
\c       ctrl+c 结束上个命令运行
\q       quit; exit; ctrl+d 退出 mysql
\G       将数据竖起来显示
source   恢复备份文件
```

### 启动方式

![AN1ZCg](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/AN1ZCg.jpg)

```mysql
以上多种方式，都可以单独启动MySQL服务
mysqld_safe和mysqld一般是在临时维护时使用。
另外，从Centos 7系统开始，支持systemd直接调用mysqld的方式进行启动数据库

维护性的启动方式
我们一般会将我们需要的参数临时加到命令行。也会读取/etc/my.cnf 的内容,但是如果冲突,命令行优先级最高

/usr/local/mysql/bin/mysqld_safe  --skip-grant-tables --skip-networking &
/usr/local/mysql/bin/mysqld &
```
