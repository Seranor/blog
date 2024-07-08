---
title: MySQL体系结构
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-03.html
toc: true
---

### C/S 模型介绍

<!-- more -->

![AuYdO0](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/AuYdO0.jpg)

- 连接方式

  - TCP/IP 方式(远程、本地)

    `mysql -uroot -p1 -h 192.168.0.11 -P3306`

  - Socket 方式(仅本地)

    `mysql -uroot -p1 -S /tmp/mysql.sock`

- 实例介绍

  ```bash
  实例=mysqld后台守护进程+Master Thread +干活的Thread+预分配的内存
  公司=老板+经理+员工+办公室
  ```

### MySQL 程序运行原理

- MySQL 程序结构

  ![kfNBL7](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/kfNBL7.jpg)

  ​

- 一条 SQL 语句的执行过程

  - 连接层

    ```bash
    （1）提供连接协议：TCP/IP 、SOCKET
    （2）提供验证：用户、密码，IP，SOCKET
    （3）提供专用连接线程：接收用户SQL，返回结果
    通过以下语句可以查看到连接线程基本情况
    show processlist;
    ```

  - SQL 层

    ```bash
    （1）接收上层传送的SQL语句
    （2）语法验证模块：验证语句语法,是否满足SQL_MODE
    （3）语义检查：判断SQL语句的类型
        DDL ：数据定义语言
        DCL ：数据控制语言
        DML ：数据操作语言
        DQL： 数据查询语言
        ...

    （4）权限检查：用户对库表有没有权限
    （5）解析器：对语句执行前,进行预处理，生成解析树(执行计划),说白了就是生成多种执行方案.
    （6）优化器：根据解析器得出的多种执行计划，进行判断，选择最优的执行计划
            代价模型：资源（CPU IO MEM）的耗损评估性能好坏
    （7）执行器：根据最优执行计划，执行SQL语句，产生执行结果
    执行结果：在磁盘的xxxx位置上
    （8）提供查询缓存（默认是没开启的），会使用redis tair替代查询缓存功能
    （9）提供日志记录（日志管理章节）：binlog，默认是没开启的。
    ```

  - 存储引擎层(类似 LInux 的文件系统)

    ```bash
    负责根据SQL层执行的结果，从磁盘上拿数据。
    将16进制的磁盘数据，交由SQL结构化化成表，
    连接层的专用线程返回给用户
    ```

### 逻辑结构

![ZQGxzu](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/ZQGxzu.jpg)

### 物理存储结构

![aZsnpd](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/aZsnpd.jpg)

```bash
表的物理存储结构
MyISAM 引擎（相当于 Linux 的 ext2 文件系统）
user.frm：存储的表结构（列，列属性）
user.MYD：存储的数据记录
user.MYI：存储索引
# ps: mysql 8.0 版本之后不再支持 MyISAM 引擎


InnoDB 引擎（相当于 Linux 的 XFS 文件系统）
time_zone.frm：存储的表结构（列，列属性）
time_zone.ibd：存储的数据记录和索引
```

### InnoDB 段、区、页

InnoDB 存储引擎的逻辑存储结构中所有数据都被逻辑地存放在一个空间 中 ，我们称之为表空间（tablespace），空间又由段（segment）、区（extent）、 页(page）组成 。

页： InnoDB 管理存储空间的基本单位，数据页大小默认为 16KB，我们表中记录 都是存放在页中的，官方称这种存放记录的页为索引（INDEX）页。因为这种类 型的页是用来存放表数据的，也可以称为数据页。

区： 区是由连续的页组成的空间，无论页的大小怎么变，区的大小默认总是为 1MB。如连续的 64 个 16K 的页就是一个区，连续的 32 个 32K 的页也是一个区。 为了保证区中的页的连续性，InnoDB 存储引擎一次从磁盘申请 4-5 个区，在创 建一个段时就会创建一个默认的区。

段： MySQL 的表根据存储需求会分配多个区，多个区构成的表称为段，理论上一 个表就是一个段（非分区表）。 总结： 一般情况下（非分区表） 一个表就是一个段 一个段由多个区构成 一个区在（16k），64 个连续的页，1M 大小

总结：

```bash
页：最小的存储单元，默认16k
区：64个连续的页，共1M
段：一个表就是一个段，包含一个或多个区
```
