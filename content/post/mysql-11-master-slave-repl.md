---
title: MySQL主从复制
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-11.html
toc: true
---

### 简单介绍

<!-- more -->

```mysql
# 简介
1.基于二进制日志复制的
2.主库的修改操作会记录二进制日志
3.从库会请求新的二进制日志并回放,最终达到主从数据同步
4.主从复制核心功能:
  辅助备份,处理物理损坏
  扩展新型的架构:高可用,高性能,分布式架构等

# 前提
1.两台以上mysql实例 ,server_id,server_uuid不同
2.主库开启二进制日志
3.专用的复制用户
4.保证主从开启之前的某个时间点,从库数据是和主库一致
5.告知从库,复制user,passwd,IP port,以及复制起点(change master to)
6.线程(三个):Dump thread  IO thread  SQL thread 开启(start slave)
```

### 搭建主从复制

```bash
# 环境准备
cat /etc/hosts
192.168.0.12  db2
192.168.0.13  db3

# 主机角色分配
db2做主库
db3做从库

# 搭建数据库，两台机器参考最开始的二进制搭建，注意目录的权限
# 配置文件 server_id 不能相同 db2 是7 db3是8
cat /etc/my.cnf
[mysqld]
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=7
port=3306
secure-file-priv=/tmp
autocommit=0
innodb_flush_method=O_DIRECT
innodb_flush_log_at_trx_commit=1
log_error=/data/mysql/data/mysql.log
log_bin=/data/binlog/mysql-bin
gtid-mode=on
enforce-gtid-consistency=true
slow_query_log=1
slow_query_log_file=/data/mysql/data/slow.log
long_query_time=0.1
log_queries_not_using_indexes
[client]
socket=/tmp/mysql.sock

# 两台都启动数据库
service mysqld start

# 两台检查server_id
select @@server_id;

# 两台检查二进制日志开启情况
show variables like '%log_bin%';

# 主库db2创建复制用户
grant replication slave on *.* to repl@'192.168.0.%' identified by '1';
select user, host from mysql.user;

# 主库db2模拟一点数据
create database test charset utf8mb4;
use test;
create table t1 (id int);
insert into t1 values (11),(22),(33);
commit;
select * from t1;

# db2主库进行全备
mysqldump -A -R -E --master-data=2 --triggers --single-transaction > /tmp/full.sql
scp -rp /tmp/full.sql  db3:/tmp  # 推送到db3从库的 /tmp 目录中

# db3恢复主库的全备数据
set sql_log_bin=0;
source /tmp/full.sql;

# db2需要告诉从库的复制信息
# db2主库上查看帮助命令
mysql> help change master to
# 得到相应模板，修改即可
# 需要的信息在 mysqldump 备份文件中能查看到
vim /tmp/full.sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1048;

# 在db3从库中执行
CHANGE MASTER TO
  MASTER_HOST='192.168.0.12',
  MASTER_USER='repl',
  MASTER_PASSWORD='1',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=1048,
  MASTER_CONNECT_RETRY=10;

# db3从库启动主从线程
start slave;

# 如果上面信息错误而且执行了使用以下办法
stop slve;
reset slave all;
change master to...
start slave;

# 检查主从情况 db3从库执行
show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.12
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 1048
               Relay_Log_File: db3-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes  # 基本状态
            Slave_SQL_Running: Yes  # 基本状态
                         ......
```

### 主从复制原理

##### 涉及到的文件及线程

```mysql
# 文件
主库: binlog
从库:
  relay-log          # 中继日志
  master.info        # 主库信息记录日志
  relay-log.info     # 记录中继应用情况信息

# 线程
主库:
  binlog_dump_thread  # 二进制日志投递线程
  show processlist;  # 可以查看到 Binlog Dump
从库:
  IO_Thread  # 从库IO线程，请求和接收binlog
  SQL_Thread  # 从库SQL线程，回放日志
```

##### 主从复制原理

```mysql
1. change master to 时，ip pot user password binlog position写入到master.info进行记录
2. start slave 时，从库会启动IO线程和SQL线程
3. IO_T，读取master.info信息，获取主库信息连接主库
4. 主库会生成一个准备binlog DUMP线程，来响应从库
5. IO_T根据master.info记录的binlog文件名和position号，请求主库DUMP最新日志
6. DUMP线程检查主库的binlog日志，如果有新的，TP(传送)给从从库的IO_T
7. IO_T将收到的日志存储到了TCP/IP 缓存，立即返回ACK给主库 ，主库工作完成
8. IO_T将缓存中的数据，存储到relay-log日志文件,更新master.info文件binlog 文件名和postion，IO_T工作完成
9. SQL_T读取relay-log.info文件，获取到上次执行到的relay-log的位置，作为起点，回放relay-log
10.SQL_T回放完成之后，会更新relay-log.info文件。
11.relay-log会有自动清理的功能。

细节：
1.主库一旦有新的日志生成，会发送“信号”给binlog dump ，IO线程再请求
```

![ApDem6](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/ApDem6.jpg)

### 主从复制监控信息

```mysql
# 主从监控
    # 主库
    # 查看连接情况，每个从库都会有一行dump相关的信息
    show processlist;

		# 从库
		show slave status \G

	  # 详细信息说明:
        # 主库的信息(master.info)
        Master_Host: 192.168.0.12          # 主库的IP
        Master_User: repl                  # 复制用户名
        Master_Port: 3306                  # 主库的端口
        Connect_Retry: 10                  # 断连之后的重试次数
        Master_Log_File: mysql-bin.000001  # 已经获取到的binlog的文件名
        Read_Master_Log_Pos: 1301          # 已经获取到的binlog的位置号

        # 从库 relay-log.info 信息
        Relay_Log_File: db3-relay-bin.000002  # 从库已经运行过的relaylog的文件名
        Relay_Log_Pos: 573                    # 从库已经运行过的relaylog的位置点

        # 从库复制线程工作状态
        Slave_IO_Running: Yes
        Slave_SQL_Running: Yes

        # 过滤复制相关的状态
        Replicate_Do_DB:
        Replicate_Ignore_DB:
        Replicate_Do_Table:
        Replicate_Ignore_Table:
        Replicate_Wild_Do_Table:
        Replicate_Wild_Ignore_Table:

        # 从库延时主库的时间(非人为)
        Seconds_Behind_Master: 0  # 从库延时主库的时间(单位为秒)

        # 延时从库有关状态
        SQL_Delay: 0              # 延时从库设定的时间
        SQL_Remaining_Delay: NULL # 延时操作剩余时间

        # 从库线程报错详细信息
        Last_IO_Errno: 0         # IO报错的号码
        Last_IO_Error:           # IO报错的具体信息
        Last_SQL_Errno: 0        # SQL报错的号码
        Last_SQL_Error:          # SQL报错具体信息

        # GTID复制信息
        Retrieved_Gtid_Set:      # 接收的GTID的个数
        Executed_Gtid_Set:       # 执行的GTID的个数
        Auto_Position: 0
```

### 主从故障分析处理

```bash
# 连接主库
(1) 用户 密码  IP  port
	Last_IO_Error: error reconnecting to master 'repl@10.0.0.51:3307' - retry-time: 10  retries: 7
	ERROR 1045 (28000): Access denied for user 'repl'@'db01' (using password: YES)
  原因:密码错误、用户错误、地址错误、端口
		  skip_name_resolve
		  ERROR 2003 (HY000): Can't connect to MySQL server on '10.0.0.52' (113)
	处理方法:
		stop  slave
		reset slave all
		change master to
		start slave

(2) 主库连接数上线,或者是主库太繁忙
	show slave  staus \G
	Last_IO_Errno: 1040
  Last_IO_Error: error reconnecting to master 'repl@10.0.0.51:3307' - retry-time: 10  retries: 7

	处理思路:
		拿复制用户,手工连接一下: mysql -urepl -p123 -h 10.0.0.51 -P 3307
		ERROR 1040 (HY000): Too many connections
	处理方法:
			set global max_connections=300;

(3)防火墙,网络不通

# 请求二进制日志
	主库缺失日志
	从库方面,二进制日志位置点不对

Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'could not find next log; the first event 'mysql-bin.000001' at 154, the last event read from '/data/3307/data/mysql-bin.000002' at 154, the last byte read from '/data/3307/data/mysql-bin.000002' at 154.'

注意: 在主从复制环境中,严令禁止主库中reset master; 可以选择expire 进行定期清理主库二进制日志
解决方案: 重新构建主从

# SQL 线程故障
(1)读写relay-log.info
(2)relay-log损坏,断节,找不到
(3)接收到的SQL无法执行
	0. SQL_MODE影响
	1.要创建的数据库对象,已经存在
	2.要删除或修改的对象不存在
	3.DML语句不符合表定义及约束时.

归根揭底的原因都是由于从库发生了写入操作
Last_SQL_Error: Error 'Can't create database 'db'; database exists' on query. Default database: 'db'. Query: 'create database db'

以下是有风险的操作:处理方法(以从库为核心的处理方案)：
stop slave;
set global sql_slave_skip_counter = 1;

# 将同步指针向下移动一个，如果多次不同步，可以重复操作
start slave;

/etc/my.cnf
slave-skip-errors = 1032,1062,1007

1007:对象已存在
1032:无法执行DML
1062:主键冲突,或约束冲突

但是，以上操作有时是有风险的，最安全的做法就是重新构建主从，把握一个原则,一切以主库为主
一劳永逸的方法:可以设置从库只读
show variables like '%read_only%';
```

### 主从延时原因分析

```mysql
# 从库延时主库的时间
Seconds_Behind_Master: 0   # 从库延时主库的时间（秒为单位）

# 主库方面：
  1.日志写入不及时，解决参数(双一标准): sync_binlog=1;
  2.主库并发业务较高，使用分布式架构
  4.从库太多，使用级联主从

对于Classic Replication: 主库是有能力并发运行事务的，但是在Dump_T在传输日志的时候，是以事件为单元传输日志的，所以导致事务的传输工作是串行方式的，这时在主库TPS很高时，会产生比较大的主从延时
解决办法: group commit
从5.6开始加入了GTID，在复制时，可以将原来串行的传输模式变成并行的，除了GTID支持，还需要双一保证

# 从库方面
Classic Replication
SQL 线程只有一个，所以说只能串行执行relay的事务
可以多加几个SQL线程

在5.6中出现了database级别的多线程SQL
只能针对不同库下的事务，才能并发
到5.7版本加入了MTS ，真正实现了事务级别的并发SQL
```

### 延时从库

```mysql
# 数据损坏
  物理损坏
  逻辑损坏
  对于传统的主从复制，比较擅长处理物理损坏

# 设计理念
  对SQL线程进行延时设置

# 延时多久合适
  一般企业，延时3-6小时

# 如何设置
stop slave;
CHANGE MASTER TO MASTER_DELAY = 300;
start slave;
show slave status \G
  ......
  SQL_Delay: 300
  SQL_Remaining_Delay: NULL
  ......

# 如何使用延时从库
  # 模拟故障
    create database  delay charset utf8mb4;
    use delay;
    create table t1(id int);
    insert into t1 values(1),(2),(3);
    commit;
    drop database delay;

  # 问题
  1. 停止SQL线程，停止主库业务
  2. 模拟SQL手工恢复relaylog到drop之前的位置点
  3. 截取relaylog日志，找到起点（relay-log.info）和终点(drop 操作)
  4. 恢复截取的日志，验证数据可用性

  # 开始处理
  1. 停从库的SQL线程
  		stop slave sql_thread;

  2. 找relaylog的起点和终点
      # 起点
      Relay_Log_File: db01-relay-bin.000002
      Relay_Log_Pos: 476

      # 终点
      show relaylog events in 'db01-relay-bin.000002'
      | db01-relay-bin.000002 | 1149 | Query          |         7 |        2036 | drop database delay

  3. 截取日志
  mysqlbinlog --start-position=476 --stop-position=1149 /data/3308/data/db01-relay-bin.000002 >/tmp/relay.sql

  4. 恢复
  set sql_log_bin=0;
  source /tmp/relay.sql
```

### 过滤复制

```mysql
# 主库(有弊端)
show master status;
binlog_do_db
binlog_ignore_db

# 从库
show slave status\G
  # 库级别
  Replicate_Do_DB:
  Replicate_Ignore_DB:

  # 表级别
  Replicate_Do_Table:
  Replicate_Ignore_Table:

  # 模糊匹配
  Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:

# 实现: 只需要复制xyz库的数据到从库
  vim /etc/my.cnf
  [mysqld]
  ...
  replicate_do_db=xyz
  ...

  #  检查
  show slave status\G
  Replicate_Do_DB: xyz
```

### 半同步复制

```mysql
解决主从数据一致性问题

# 原理
1. 主库执行新的事务,commit时,更新 show master  status\G ,触发一个信号给
2. binlog dump 接收到主库的 show master status\G信息,通知从库日志更新了
3. 从库IO线程请求新的二进制日志事件
4. 主库会通过dump线程传送新的日志事件,给从库IO线程
5. 从库IO线程接收到binlog日志,当日志写入到磁盘上的relaylog文件时,给主库ACK_receiver线程
6. ACK_receiver线程触发一个事件,告诉主库commit可以成功了
7. 如果ACK达到了我们预设值的超时时间,半同步复制会切换为原始的异步复制

# 配置
# 加载插件
  主:
  INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
  从:
  INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

# 查看是否加载成功
  show plugins;

# 启动
  主:
  SET GLOBAL rpl_semi_sync_master_enabled = 1;
  从:
  SET GLOBAL rpl_semi_sync_slave_enabled = 1;

# 重启从库上的IO线程
  STOP SLAVE IO_THREAD;
  START SLAVE IO_THREAD;

# 查看是否在运行
  主:
  show status like 'Rpl_semi_sync_master_status';
  从:
  show status like 'Rpl_semi_sync_slave_status';
```

### `GTID`复制

```mysql
# GTID介绍
GTID(Global Transaction ID)是对于一个已提交事务的唯一编号，并且是一个全局(主从复制)唯一的编号
它的官方定义如下：
GTID = source_id ：transaction_id
7E11FA47-31CA-19E1-9E56-C43AA21293967:29
核心特性: 全局唯一,具备幂等性

# 核心参数
gtid-mode=on                        # 启用gtid类型，否则就是普通的复制架构
enforce-gtid-consistency=true       # 强制GTID的一致性
log-slave-updates=1                 # slave更新是否记入日志

# 配置GTID复制过程
# 主库db01配置文件
cat > /etc/my.cnf <<EOF
[mysqld]
basedir=/application/mysql/
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=51
port=3306
secure-file-priv=/tmp
autocommit=0
log_bin=/data/binlog/mysql-bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
[mysql]
prompt=db01 [\\d]>
EOF

# 从库db02配置文件
cat > /etc/my.cnf <<EOF
[mysqld]
basedir=/application/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=52
port=3306
secure-file-priv=/tmp
autocommit=0
log_bin=/data/binlog/mysql-bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
[mysql]
prompt=db02 [\\d]>
EOF

# 从库db03配置文件
cat > /etc/my.cnf <<EOF
[mysqld]
basedir=/application/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=53
port=3306
secure-file-priv=/tmp
autocommit=0
log_bin=/data/binlog/mysql-bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
[mysql]
prompt=db03 [\\d]>
EOF

# 初始化数据(所有节点)
mysqld --initialize-insecure --user=mysql --basedir=/application/mysql  --datadir=/data/mysql/data

# 启动数据库
/etc/init.d/mysqld start

# 主库创建用户
grant replication slave on *.* to repl@'10.0.0.%' identified by '123';

# 两个从库开启主从
mysql -e "change master to master_host='10.0.0.51',master_user='repl',master_password='123',MASTER_AUTO_POSITION=1;start slave;"
mysql -e "show slave status \G"|grep Y
```
