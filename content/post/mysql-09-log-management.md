---
title: MySQL日志管理
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-09.html
toc: true
---

### 日志管理

<!-- more -->

![f262Yf](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/f262Yf.jpg)

### 错误日志(`log_err`)

- #### 作用

  ```mysql
  记录启动、关闭、日常运行过程中的状态信、警告、错误
  ```

- #### 配置

  ```mysql
  默认开启的，默认位置在 /数据路径下/hostname.err

  # 自定义配置
  vim /etc/my.cnf
  [mysqld]
  ...
  log_error=/data/mysql/data/mysql.log
  ...

  # 重启生效

  # 数据库内查看错误日志文件位置
  select @@log_error;
  ```

- #### 查看日志

  ```mysql
  主要关注[ERROR],看上下文
  ```

### `binlog(binary logs):`二进制日志

#### 作用

```mysql
1.备份恢复必须依赖二进制日志
2.主从环境必须依赖二进制日志
```

#### 基础参数查看

```mysql
select @@log_bin;  # 开关
select @@log_bin_basename;  # 日志路径及名字
select @@server_id;  # 服务ID号
select @@binlog_format;  # 二进制日志格式
select @@sync_binlog;  # 双一标准之二
show variables like '%log_bin%'; # mysql内查看二进制日志情况
```

#### 配置开启二进制日志

```mysql
# MySQL默认是没有开启二进制日志的

# 配置
log_bin:    开关以及设置存放位置
server_id:  5.6中不需要，5.7中必须加

mkdir /data/binlog
vim /etc/my.cnf
...
server_id=6
log_bin=/data/binlog/mysql-bin

...

chown -R mysql.mysql /data/
systemctl restart mysqld

ls /data/binlog/
-rw-r----- 1 mysql mysql 154 12月 24 15:10 mysql-bin.000001
-rw-r----- 1 mysql mysql  30 12月 24 15:10 mysql-bin.index

# 注意: 日志和数据分开存放(物理磁盘)
```

#### 记录情况

```mysql
binlog是SQL层的功能。记录的是变更SQL语句，不记录查询语句

DDL ：原封不动的记录当前DDL(statement)。
DCL ：原封不动的记录当前DCL(statement)。
DML ：只记录已经提交的事务DML
三种记录方式：受binlog_format（binlog的记录格式）参数影响

1. STATEMENT (5.6默认)   SBR  ：语句模式原封不动的记录当前DML            ---> 行记录模式，记录行的变化
2. ROW       (5.7默认值)	RBR  ：记录数据行的变化(用户看不懂，需要工具分析)  ---> 语句记录模式，记录操作语句本身
3. MIXED     (混合)模式  MBR  ：以上两种模式的混合                      ---> 混合记录模式


SBR与RBR模式的对比
STATEMENT(SBR)：只记录语句本身，可读性较高，日志量少，但是不够严谨，对于函数类的操作，将来恢复时会造成错误
ROW(RBR)      ：逐行记录日志，可读性很低，日志量大，足够严谨，不会出现记录错误


5.7默认是RBR模式，企业建议模式

# 查看模式
mysql>select @@binlog_format;

# 配置文件中可更改
vim /etc/my.cnf
[mysqld]
...
binlog_format=row
...
```

#### 事件(`event`)

- ##### 简介

  ```mysql
  二进制日志的最小记录单元
  对于DDL,DCL,一个语句就是一个event
  对于DML语句来讲:只记录已提交的事务
  例如以下列子,就被分为了4个event

              position号码(字节偏移量，截取日志用)
  begin;      120  - 340
  DML1        340  - 460
  DML2        460  - 550
  commit;     550  - 760
  ```

- ##### 组成

  ```mysql
  三部分构成:
  (1) 事件的开始标识
  (2) 事件内容
  (3) 事件的结束标识

  Position:
  开始标识: at 194
  结束标识: end_log_pos 254

  194? 254?
  某个事件在binlog中的相对位置号
  位置号的作用是为了方便我们截取事件
  ```

#### 日志文件的查看

- ##### 简单查看

  ```mysql
  # 查看二进制日志位置
  show variables like '%log_bin%';

  ll /data/binlog/
  -rw-r----- 1 mysql mysql 154 12月 24 15:10 mysql-bin.000001
  -rw-r----- 1 mysql mysql  30 12月 24 15:10 mysql-bin.index

  # 查看正在使用的二进制
  show binary logs;
  +------------------+-----------+
  | Log_name         | File_size |
  +------------------+-----------+
  | mysql-bin.000001 |       154 |
  +------------------+-----------+
  1 row in set (0.00 sec)

  Log_name: 目前MySQL存在的二进制日志名字
  File_size: 目前MySQL用到哪个Position号，使用的是最后一个文件
  Position：最后一个事件的结束位置号

  # 二进制文件会增加一个
  flush logs;

  # 查看正在使用的二进制文件
  show master status;
  +------------------+----------+--------------+------------------+------------------------------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
  +------------------+----------+--------------+------------------+------------------------------------------+
  | mysql-bin.000003 |      154 |              |                  | 269f1ac3-8d95-11e9-8dc4-000c297969b7:1-7 |
  +------------------+----------+--------------+------------------+------------------------------------------+
  1 row in set (0.00 sec)
  ```

- #### 内容查看`binlog`文件内容

  ```mysql
  # 查看二进制日志文件
  # 先查看正在使用的二进制文件
  show master status;
  +------------------+----------+--------------+------------------+------------------------------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
  +------------------+----------+--------------+------------------+------------------------------------------+
  | mysql-bin.000003 |      154 |              |                  | 269f1ac3-8d95-11e9-8dc4-000c297969b7:1-7 |
  +------------------+----------+--------------+------------------+------------------------------------------+
  1 row in set (0.00 sec)

  # 根据当前使用的二进制日志再查看当前的event
  show binlog events in 'mysql-bin.000003';
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | mysql-bin.000003 |   4 | Format_desc    |         6 |         123 | Server ver: 5.7.31-log, Binlog ver: 4 |
  | mysql-bin.000003 | 123 | Previous_gtids |         6 |         154 |                                       |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  2 rows in set (0.00 sec)

  # 查看最开始的日志文件及说明
  show binlog events in 'mysql-bin.000001';
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | mysql-bin.000001 |   4 | Format_desc    |         6 |         123 | Server ver: 5.7.31-log, Binlog ver: 4 |
  | mysql-bin.000001 | 123 | Previous_gtids |         6 |         154 |                                       |
  | mysql-bin.000001 | 154 | Rotate         |         6 |         201 | mysql-bin.000002;pos=4                |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  3 rows in set (0.00 sec)

  Pos 4 是保留字段
  Pos 154 是日志文件头格式信息(5.7版本是154个字节，5.6版本是前120个字节)
  每一行都是一个事件

  # 说明
  Log_name    : 日志名
  Pos         : 事件开始的Position
  Event_type  : 事件类型
  Server_id   : 发生在哪台机器的事件
  End_log_pos : 事件结束的位置
  Info        : 事件内容

  # 录入信息查看事件信息
  # 没录入信息之前
  show binlog events in 'mysql-bin.000003';
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | mysql-bin.000003 |   4 | Format_desc    |         6 |         123 | Server ver: 5.7.31-log, Binlog ver: 4 |
  | mysql-bin.000003 | 123 | Previous_gtids |         6 |         154 |                                       |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  2 rows in set (0.00 sec)

  # 创建数据以及插入数据提交
  create database testdb;
  use testdb;
  create table t1(id int);
  insert into t1 values(1);
  commit;

  # 查看录入数据之后的情况
  show binlog events in 'mysql-bin.000003';
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  | mysql-bin.000003 |   4 | Format_desc    |         6 |         123 | Server ver: 5.7.31-log, Binlog ver: 4 |
  | mysql-bin.000003 | 123 | Previous_gtids |         6 |         154 |                                       |
  | mysql-bin.000003 | 154 | Anonymous_Gtid |         6 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
  | mysql-bin.000003 | 219 | Query          |         6 |         319 | create database testdb                |
  | mysql-bin.000003 | 319 | Anonymous_Gtid |         6 |         384 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
  | mysql-bin.000003 | 384 | Query          |         6 |         485 | use `testdb`; create table t1(id int) |
  | mysql-bin.000003 | 485 | Anonymous_Gtid |         6 |         550 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
  | mysql-bin.000003 | 550 | Query          |         6 |         624 | BEGIN                                 |
  | mysql-bin.000003 | 624 | Table_map      |         6 |         671 | table_id: 108 (testdb.t1)             |
  | mysql-bin.000003 | 671 | Write_rows     |         6 |         711 | table_id: 108 flags: STMT_END_F       |
  | mysql-bin.000003 | 711 | Xid            |         6 |         742 | COMMIT /* xid=38 */                   |
  +------------------+-----+----------------+-----------+-------------+---------------------------------------+
  11 rows in set (0.00 sec)
  # 这些可读性不强
  ```

- ##### `binlog`文件详细查看

  ```shell
  # 命令行上执行
  cd /data/binlog/
  mysqlbinlog mysql-bin.000003

  mysqlbinlog mysql-bin.000003  |grep -v 'SET'  # 这条命令得到也看不懂 有注释时间等

  mysqlbinlog --base64-output=decode-rows  -vvvv mysql-bin.000003
  ```

  ![wzf7nb](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/wzf7nb.png)

#### 基于二进制日志数据恢复

- ##### 按需截取日志

  > 截取二进制日志的核心在于找起点和终点

  - 基于`position`号截取

    ```mysql
    # mysqlbinlog 命令参数截取
    --start-position=
    --stop-position=

    # sql语句查到建库的position号
     show binlog events in 'mysql-bin.000003';
    +------------------+-----+----------------+-----------+-------------+---------------------------------------+
    | Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                  |
    +------------------+-----+----------------+-----------+-------------+---------------------------------------+
    | mysql-bin.000003 |   4 | Format_desc    |         6 |         123 | Server ver: 5.7.31-log, Binlog ver: 4 |
    | mysql-bin.000003 | 123 | Previous_gtids |         6 |         154 |                                       |
    | mysql-bin.000003 | 154 | Anonymous_Gtid |         6 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
    | mysql-bin.000003 | 219 | Query          |         6 |         319 | create database testdb                |
    | mysql-bin.000003 | 319 | Anonymous_Gtid |         6 |         384 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
    | mysql-bin.000003 | 384 | Query          |         6 |         485 | use `testdb`; create table t1(id int) |
    | mysql-bin.000003 | 485 | Anonymous_Gtid |         6 |         550 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
    | mysql-bin.000003 | 550 | Query          |         6 |         624 | BEGIN                                 |
    | mysql-bin.000003 | 624 | Table_map      |         6 |         671 | table_id: 108 (testdb.t1)             |
    | mysql-bin.000003 | 671 | Write_rows     |         6 |         711 | table_id: 108 flags: STMT_END_F       |
    | mysql-bin.000003 | 711 | Xid            |         6 |         742 | COMMIT /* xid=38 */                   |
    +------------------+-----+----------------+-----------+-------------+---------------------------------------+
    11 rows in set (0.00 sec)

    # 截取事件导出sql语句(命令行)
    mysqlbinlog --start-position=219 --stop-position=319 /data/binlog/mysql-bin.000003 > /tmp/bin.sql


    # 恢复(sql语句)
    # 模拟删除(不代表生产)
    show databases;
    drop database testdb;
    show databases;

    # 临时关闭当前查看二进制日志记录开关
    # 因为已经有了二进制日志做恢复，再次恢复的时候会再次记录恢复的二进制日志，所以要先短暂关闭二进制记录，避免IO损耗
    set sql_log_bin=0;

    # 恢复命令
    source /tmp/bin.sql;

    # 检查
    show databases;

    set sql_log_bin=1;
    # 恢复完成后需要退出当前窗口，当前窗口binlog二进制没有记录
    ```

    下图是根据`position`号截取的日志信息

    ![LBFNQB](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/LBFNQB.png)

  - 基于时间截取

    ```mysql
    --start-datetime
    --stop-datetime
    ```

- 案例恢复

  ```mysql
  # 使用binlog日志进行恢复
  #模拟删库
  create database binlog charset utf8mb4;

  use binlog;
  create table t1(id int);
  insert into t1 values(1);
  commit;
  insert into t1 values(2);
  commit;
  insert into t1 values(3);
  commit;

  drop database binlog;

  # 恢复
  # 查看当前的二进制日志文件是哪一个
  show master status;

  # 查看事件找到起点和终点
  show binlog events in 'mysql-bin.000003';
  ......
  | mysql-bin.000003 |  905 | Anonymous_Gtid |         6 |         970 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
  | mysql-bin.000003 |  970 | Query          |         6 |        1086 | create database binlog charset utf8mb4 |
  | mysql-bin.000003 | 1086 | Anonymous_Gtid |         6 |        1151 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
  | mysql-bin.000003 | 1151 | Query          |         6 |        1252 | use `binlog`; create table t1(id int)  |
  | mysql-bin.000003 | 1252 | Anonymous_Gtid |         6 |        1317 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
  | mysql-bin.000003 | 1317 | Query          |         6 |        1391 | BEGIN                                  |
  | mysql-bin.000003 | 1391 | Table_map      |         6 |        1438 | table_id: 110 (binlog.t1)              |
  | mysql-bin.000003 | 1438 | Write_rows     |         6 |        1478 | table_id: 110 flags: STMT_END_F        |
  | mysql-bin.000003 | 1478 | Xid            |         6 |        1509 | COMMIT /* xid=87 */                    |
  | mysql-bin.000003 | 1509 | Anonymous_Gtid |         6 |        1574 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
  | mysql-bin.000003 | 1574 | Query          |         6 |        1648 | BEGIN                                  |
  | mysql-bin.000003 | 1648 | Table_map      |         6 |        1695 | table_id: 110 (binlog.t1)              |
  | mysql-bin.000003 | 1695 | Write_rows     |         6 |        1735 | table_id: 110 flags: STMT_END_F        |
  | mysql-bin.000003 | 1735 | Xid            |         6 |        1766 | COMMIT /* xid=89 */                    |
  | mysql-bin.000003 | 1766 | Anonymous_Gtid |         6 |        1831 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
  | mysql-bin.000003 | 1831 | Query          |         6 |        1905 | BEGIN                                  |
  | mysql-bin.000003 | 1905 | Table_map      |         6 |        1952 | table_id: 110 (binlog.t1)              |
  | mysql-bin.000003 | 1952 | Write_rows     |         6 |        1992 | table_id: 110 flags: STMT_END_F        |
  | mysql-bin.000003 | 1992 | Xid            |         6 |        2023 | COMMIT /* xid=91 */                    |
  | mysql-bin.000003 | 2023 | Anonymous_Gtid |         6 |        2088 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'   |
  | mysql-bin.000003 | 2088 | Query          |         6 |        2186 | drop database binlog                   |
  +------------------+------+----------------+-----------+-------------+----------------------------------------+

  起点: 970
  终点: 2088(2186是删库语句，否则无意义)

  # 根据起点和终点截取需要的二进制日志(命令行操作)
  mysqlbinlog --start-position=970 --stop-position=2088 /data/binlog/mysql-bin.000003 > /tmp/databse-binlog.sql

  # 恢复
  set sql_log_bin=0;
  source /tmp/databse-binlog.sql;

  # 验证结果
  show databases;
  select * from t1;
  ```

- ##### `GTID`功能的二进制日志

  ```mysql
  create database binlog charset utf8mb4;
  use binlog;
  create table t1(id int);
  insert into t1 values(1);
  commit;
  insert into t1 values(2);
  commit;
  truncate table t1;  # 这条是误操作,不能恢复这条语句
  insert into t1 values(3);
  commit;
  drop database binlog;
  # 截取position号就比较麻烦 ，需要多次截取恢复，不方便
  ```

  - `GTID`介绍

    ```mysql
    GTID(全局事务编号)
    是对于一个已提交事务的编号，并且是一个全局唯一的编号。
    它的官方定义如下：
    GTID = source_id ：transaction_id
    7E11FA47-31CA-19E1-9E56-C43AA21293967:29

    说明:
    DDL DCL，一条语句(事件)就是一个事务，占一个GTID号
    DML，一个完整的事务(begin ---> commit)，是一个事务，占一个GTID号


    5.6 版本新加的特性,5.7中做了加强
    5.6 中不开启,没有这个功能.
    5.7 中的GTID,即使不开也会有自动生成
    SET @@SESSION.GTID_NEXT= 'ANONYMOUS'
    ```

  - `GTID`配置及截取使用

    ```mysql
    # 修改GTID相关的配置文件
    vim /etc/my.cnf
    [mysqld]
    ...
    gtid-mode=on
    enforce-gtid-consistency=true
    ...

    # 重启数据库
    systemctl restart mysql

    # GTID的前段
    cat /data/mysql/data/auto.cnf
    [auto]
    server-uuid=b25a4cc7-609b-11ec-afea-525400aaf73e

    # 查看GTID
     show master status;  # Executed_Gtid_Set信息

    # 创建数据查看GTID情况
     create database testdb2 charset utf8mb4;
     use testdb2;
     create table t1 (id int);
     insert into t1 values(1);
     commit;
     insert into t1 values(2);
     commit;
     insert into t1 values(3);
     commit;

     show master status;
     show binlog events in 'mysql-bin.000004';

     # 看到下图多了右边的编号，此时模拟执行数据库丢失
     drop database testdb2;

    show binlog events in 'mysql-bin.000004';  # 此时多了一条删库的编号

    # 基于GTID截取日志
    # 参数
    --include-gtids
    --exclude-gtids
    --skip-gtids

    # 错误的截取命令(命令行)
    mysqlbinlog --include-gtids='b25a4cc7-609b-11ec-afea-525400aaf73e:1-5' /data/binlog/mysql-bin.000004 > /tmp/testdb2.sql
    # 以上截取出来的日志不能直接恢复使用
    ```

    ![GzZJIW](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/GzZJIW.png)

  - `GTID`幂等性

    ```mysql
    开启GTID后,MySQL恢复Binlog时,重复GTID的事务不会再执行了
    show master status;  # 因为这条命令中出现的 Executed_Gtid_set信息 已经有了，表示不会再做了，检查幂等性

    # 恢复
    # 正确的截取日志命令(命令行)
    mysqlbinlog --skip-gtids --include-gtids='b25a4cc7-609b-11ec-afea-525400aaf73e:1-5' /data/binlog/mysql-bin.000004 > /tmp/testdb2.sql

    # sql语句恢复
    set sql_log_bin=0;
    source /tmp/testdb2.sql
    set sql_log_bin=1;

    # 检查
    show databases;
    select * from testdb2.t1;

    # 跳过某些GTID不截取
    mysqlbinlog --skip-gtids --include-gtids='b25a4cc7-609b-11ec-afea-525400aaf73e:1-5' \
    --exclude-gtids='b25a4cc7-609b-11ec-afea-525400aaf73e:6' \
    /data/binlog/mysql-bin.000004 > /tmp/testdb2.sql

    单个: 'b25a4cc7-609b-11ec-afea-525400aaf73e:6'
    连续个: 'b25a4cc7-609b-11ec-afea-525400aaf73e:1-6'
    多个不连续: 'b25a4cc7-609b-11ec-afea-525400aaf73e:2','b25a4cc7-609b-11ec-afea-525400aaf73e:3'
    ```

#### 二进制日志其他操作

- ##### 临时关闭

  ```mysql
  # 临时关闭二进制日志，退出mysql窗口就可以恢复，做数据恢复之前使用
  set sql_log_bin=0;
  ```

- ##### 自动清理

  ```mysql
  # 窗口自动清理周期
  select @@expire_logs_days;
  show variables like '%expire%';

  # 自动清理时间,是要按照全备周期+1
  # 企业建议,至少保留两个全备周期+1的binlog
  set global expire_logs_days=8;

  # 永久生效
  vim /etc/my.cnf
  [mysqld]
  ...
  expire_logs_days=15;
  ...
  ```

- ##### 手工清理

  ```mysql
  # 三天之前到现在的
  PURGE BINARY LOGS BEFORE now() - INTERVAL 3 day;
  PURGE BINARY LOGS TO 'mysql-bin.000010';

  # 查看binlog日志信息
  show binary logs;

  # 删除到哪个二进制日志为止
  PURGE BINARY LOGS TO 'mysql-bin.000004';

  # 注意:不要手工 rm binlog文件，如果 rm 删除了如下恢复
  1. my.cnf binlog关闭掉,启动数据库
  2.把数据库关闭,开启binlog,启动数据库
  删除所有binlog,并从000001开始重新记录日志

  # 重新计数二进制日志，主从关系中，主库执行此操作，主从环境必崩
  reset master;
  ```

- ##### 日志滚动

  ```mysql
  # 重启mysql也会自动滚动一个新的
  # 产生一个新的日志sql命令
  flush logs;

  # 默认日志文件大小1G，达到这个大小也回生成全新的，一般会设置小一点，避免分析的时候打不开
  show variables like '%max_binlog_size%';
  +-----------------+------------+
  | Variable_name   | Value      |
  +-----------------+------------+
  | max_binlog_size | 1073741824 |
  +-----------------+------------+
  ```

### `slow_log`慢日志

#### 作用

```mysql
记录慢SQL语句的日志,定位低效SQL语句的工具日志
```

#### 开启慢日志

```mysql
# 查看是否开启
select @@slow_query_log;

# 查看慢日志位置
select @@slow_query_log_file;

# 查看慢查询时间
select @@long_query_time;
select @@log_queries_not_using_indexes ;

# 设置慢日志配置文件
vim /etc/my.cnf
slow_query_log=1                               # 开关
slow_query_log_file=/data/mysql/data/slow.log  # 文件位置及名字
long_query_time=0.1                            # 设定慢查询时间
log_queries_not_using_indexes                  # 没走索引的语句也记录

# 重启mysql
systemctl restart mysqld
```

#### 慢日志分析

```mysql
# 模拟慢日志，多执行几次
select * from t100w where k1="aa" limit 1000,2000;

# mysqldumpslow 分析慢日志
mysqldumpslow -s c -t 10 /data/mysql/data/slow.log

# 命令擦参数说明
-s  # 以...排序
c   # 执行次数
-t  # top 10

# 第三方工具
# 下载地址 https://www.percona.com/downloads/percona-toolkit/LATEST/
# wget https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/sql/percona-toolkit-3.2.1-1.el7.x86_64.rpm
wget https://downloads.percona.com/downloads/percona-toolkit/3.3.1/binary/redhat/7/x86_64/percona-toolkit-3.3.1-1.el7.x86_64.rpm
yum -y install perl-DBI perl-DBD-MySQL perl-Time-HiRes perl-IO-Socket-SSL perl-Digest-MD5
yum -y localinstall percona-toolkit-3.3.1-1.el7.x86_64.rpm

# toolkit工具包中的命令
pt-query-diagest  /data/mysql/data/slow.log

# 图形化的第三方工具
Anemometer基于pt-query-digest将MySQL慢查询可视化
```
