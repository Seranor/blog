---
title: MySQL登录及密码管理
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-02.html
toc: true
---

### 已知密码情况下

<!-- more -->

```bash
mysqladmin -uroot -p password 123456
# Enter password: #因为我们现在没有密码，直接回车即可！
# 修改密码和上面一样，只是在要求输入密码时输入正确的密码即可
```

### 忘记密码修改

```bash
# 关闭数据库
systemctl stop mysqld.service

# 启动数据库到维护模式
mysqld_safe --skip-grant-tables --skip-networking &

# 参数解释
--skip-grant-tables  # 跳过授权表
--skip-networking    # 跳过远程登录

# 直接进入mysql
mysql

# mysql语句,重新修改密码
flush privileges;
alter user root@'localhost' identified by '1';
exit

# 重启数据库
pkill mysqld
systemctl restart mysqld.service

# 此时用密码 1 即可进入数据库
 mysql -uroot -p1
```

![eOzO9V](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/eOzO9V.png)
