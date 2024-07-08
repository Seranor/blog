---
title: MySQL存储引擎(InnoDB)
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-08.html
toc: true
---

### 存储引擎介绍

> 类似于 Linux 系统中的文件系统

### 功能

<!-- more -->

```mysql
数据读写
数据安全和一致性
提高性能
热备份
自动故障恢复
高可用方面支持
```

### 种类

```mysql
InnoDB
MyISAM
MEMORY
ARCHIVE
FEDERATED
EXAMPLE
BLACKHOLE
MERGE
NDBCLUSTER
CSV

# 引擎种类查看
show engines;

存储引擎是作用在表上的，也就意味着，不同的表可以有不同的存储引擎类型。
PerconaDB:默认是XtraDB
MariaDB:默认是InnoDB
其他的存储引擎支持:
TokuDB
RocksDB
MyRocks
以上三种存储引擎的共同点:压缩比较高,数据插入性能极高
现在很多的NewSQL,使用比较多的功能特性.
```

### InnoDB 存储引擎介绍

> 在 MySQL5.5 版本之后，默认的存储引擎，提供高可用性和高性能

![KssDvX](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/KssDvX.jpg)

#### 优点

```php
1、事务（Transaction）
2、MVCC（Multi-Version Concurrency Control多版本并发控制）
3、行级锁(Row-level Lock)
4、ACSR（Auto Crash Safey Recovery）自动的故障安全恢复
5、支持热备份(Hot Backup)
6、Replication: Group Commit , GTID (Global Transaction ID) ,多线程(Multi-Threads-SQL )
```

#### 存储引擎查看

- ##### 使用 select 确认会话存储引擎

  ```mysql
  select @@default_storage_engine;
  show variables like '%engine%';
  ```

- ##### 默认存储引擎设置

  ```mysql
  # 不会在生产中操作
  会话级别
  set default_storage_engine=myisam;

  全局级别(仅影响新会话)
  set global default_storage_engine=myisam;
  重启之后，所有参数均失效

  # 放入配置文件永久生效
  vim /etc/my.cnf
  [mysqld]
  default_storage_engine=myisam

  存储引擎是表级别的，每个表创建时可以指定不同的存储引擎，但是建议统一为Innodb
  ```

- ##### 查看表的存储引擎

  ```mysql
  # 查看单表的存储引擎
  show create table world.city\G

  use world;
  show table status like 'countrylanguage'\G

  # 查看每个表的存储引擎
  select table_schema,table_name ,engine from information_schema.tables where table_schema not in ('sys','mysql','information_schema','performance_schema');
  ```

- ##### 修改一个表的存储引擎(碎片整理)

  ```mysql
  alter table world.city engine innodb;
  # 这条命令还可以进行innodb表的碎片化整理

  # 将test数据库下的所有1000表，存储引擎从MyISAM替换为innodb
  select concat("alter table ",table_name," engine innodb;")
  from information_schema.tables
  where table_schema='test'
  into outfile '/tmp/alter.sql';
  ```

### InnoDB 物理存储结构

#### 最直观的存储方式

```mysql
# 数据目录 /data/mysql/data
ibdata1：系统数据字典信息(统计信息)，UNDO（回滚）表空间等数据
ib_logfile0 ~ ib_logfile1: REDO（重做日志）日志文件，事务日志文件
ib_buffer_pool: 缓冲池(5.7) 上次关机之前的热数据会保存在这，下次启动后会加载这些数据
ibtmp1： 临时表空间磁盘位置，存储临时表
frm：存储表的列信息
ibd：表的数据行和索引
```

#### 表空间

```mysql
需要将所有数据存储到同一个表空间中 ，管理比较混乱
5.5版本出现的管理模式，也是默认的管理模式。（数据字典，undo，临时表，索引，表数据）
5.6版本，共享表空间保留，只用来存储:数据字典信息,undo,临时表。
5.7版本,临时表被独立出来了
8.0版本,undo也被独立出去了

具体变化参考官方文档:
https://dev.mysql.com/doc/refman/5.6/en/innodb-architecture.html
https://dev.mysql.com/doc/refman/5.7/en/innodb-architecture.html
https://dev.mysql.com/doc/refman/5.8/en/innodb-architecture.html
```

##### 共享表空间

```mysql
# 共享表空间 ibdata1
select @@innodb_file_per_table;  # 默认是 1 ，代表当前独立表空间模式，代表每个表一个idb文件
    # 当前 innodb_file_per_table 参数为 1 时创建表查看数据目录情况
    mysql>use world;
    mysql>create table tab(id int);

		# 此时在 /data/mysql/data/world 数据目录下会存在下面两个文件，记录表数据和索引
    [root@db1 world]# ll
    -rw-r----- 1 mysql mysql   8556 12月 23 09:52 tab.frm
    -rw-r----- 1 mysql mysql  98304 12月 23 09:52 tab.ibd

    # 临时设置 innodb_file_per_table 参数为 0 时创建表查看数据目录情况
    set global innodb_file_per_table=0;
    select @@innodb_file_per_table;
    create table bb(id int);

    [root@db1 world]# ll
    ...  # 此时没有bb.frm和bb.ibd 两个文件了，此时bb表数据和索引存放在了/data/mysql/ibdata1 文件里了


# 共享表空间设置(在搭建MySQL时，初始化数据之前设置到参数文件中)
select @@innodb_data_file_path;  # 查看ibdata1的信息
+-------------------------+
| @@innodb_data_file_path |
+-------------------------+
| ibdata1:12M:autoextend  |
+-------------------------+
# 默认设置为12M ，autoextend自增长，不够之后每次自增长64兆

show variables like '%extend%';  # 查看自增长大小，可设置

# 初始化数据之前设置到参数文件中
innodb_data_file_path=ibdata1:512M:ibdata2:512M:autoextend
innodb_autoextend_increment=64
```

##### 独立表空间

```mysql
从5.6，默认表空间不再使用共享表空间，替换为独立表空间。
主要存储的是用户数据
存储特点为：一个表一个ibd文件，存储数据行和索引信息

基本表结构元数据存储：
xxx.frm
最终结论：
      元数据            数据行+索引
mysql表数据    =（ibdataX+frm）+ibd(段、区、页)
        DDL             DML+DQL

MySQL的存储引擎日志：
Redo Log: ib_logfile0  ib_logfile1，重做日志
Undo Log: ibdata1 ibdata2(存储在共享表空间中)，回滚日志
临时表:ibtmp1，在做join union操作产生临时数据，用完就自动

# 独立表空间设置(在上面例子中)
```

##### 迁移表空间

```mysql
# 迁移表空间功能，导入和导出表空间
alter table city discard tablespace;  # 删除city表空间
alter table city import tablespace;  # 导入city表空间

# 小测试
# 先备份ibd文件
cd /data/mysql/data/school/
cp teacher.ibd /opt/

# 删除表空间
use school;
alter table teacher discard tablespace;

ll /data/mysql/data/school/  # 该目录下teacher.ibd文件就mysql删除了

# 表还在但是无法正常读取,ibdata不识别,统计信息等不存在
show tables;
select * from teacher;  # ERROR 1814 (HY000): Tablespace has been discarded for table 'teacher'

# 将备份的ibd文件恢复回位置
cp /opt/teacher.ibd /data/mysql/data/school/
chown -R mysql.mysql /data/mysql/data/school/

select * from teacher;  # 拷贝回ibd文件之后还是不能查询,ibdata不识别,统计信息等不存在
alter table teacher import tablespace;
select * from teacher;  # 导入表空间之后正常了

# 批量导入表空间
select concat("alter table ",table_schema,".",table_name," import tablespace;") from information_schema.tables where table_schema='world' into outfile '/tmp/import.sql';

# 批量删除表空间
select concat("alter table ",table_schema,".",table_name," discard tablespace;") from information_schema.tables where table_schema='world' into outfile '/tmp/discad.sql';

# 导入表空间时可能会报错,可以跳过外键检查
set foreign_key_checks=0
```

### 事务

#### 事务`ACID`特性

> 影响 DML 语句(`insert`、`update`、`delete`和一部分`select`)

`Atomic`原子性

> 所有语句作为一个单元全部成功执行或全部取消，不能出现中间状态

`Consistent`一致性

> 如果数据库在事务开始时处于一致状态，则在执行该事务期间将保留一致状态

`Isolated`隔离性

> 事务之间不相互影响

`Durable`持久性

> 事务成功完成后，所做的所有更改都会准确地记录在数据库中，所做的更改不会丢失。

#### 事务的生命周期

- 事务的开始

  ```mysql
  begin;
  start transaction;
  # 在5.5 以上的版本，不需要手工begin，只要你执行的是一个DML，会自动在前面加一个begin命令。
  ```

- 事务的结束

  ```mysql
  commit;
  完成一个事务，一旦事务提交成功，，就说明具备ACID特性了

  rollback;
  回滚事务
  将内存中已执行过的操作回滚回去
  ```

- 例子

  ```mysql
  # mysql 窗口一
  user world;
  begin;
  delete from city where id > 3000;
  delete from city where id > 2000;


  # mysql 窗口二
  use world;
  select * from city;  # 此时查询的数据还是4079条数据，并没有被删除
  begin;
  delete from city where id > 1800;
  # 此时会卡主，因为在等上一个事务，隔离性
  # 操作相同的语句会等上一个事务结束之后才会继续
  # 过了超时时间就会报 ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

  # mysql 窗口一
  commit;  # 此时提交

  # mysql 窗口二
  delete from city where id > 1800;  # 这个窗口就可以执行操作了
  commit;

  # 回滚事务
  begin;
  delete from city where id=100;
  select * from city where id=100;
  rollback;  # 对上面的sql操作语句进行回滚，如果commit了，就不能回滚
  select * from city where id=100;
  commit;
  ```

- 自动提交策略(`autocommit`)

  ```mysql
  默认执行DML语句的时候会自动的在语句前加 begin 和 commit，针对需要使用事务功能的语句 begin 或者直接全部使用事务功能

  select @@autocommit;  # 默认开启的 1 为开启

  set autocommit=0;  # 关闭自动提交策略
  set global autocommit=0;  # 全局关闭自动提交策略

  # 永久关闭自动提交策略
  vim /etc/my.cnf
  [mysqld]
  ...
  autocommit=0
  ...


  自动提交是否打开，一般在有事务需求的MySQL中，将其关闭
  不管有没有事务需求，我们一般也都建议设置为0，可以很大程度上提高数据库性能
  ```

- 事务的隐式控制

  ```mysql
  # 隐式回滚
    1. 关闭窗口的时候会自动回滚
      # 已经永久关闭了自动提交策略

    2. 删除会话ID
      show processlist;
      kill 3;

  # 隐式提交
    begin;
    a
    b
    begin;  # 此时上面的事务自动提交了

    begin;
    a
    b
    set autocommit=1;  # 此时上面的事务也自动提交了

  # 导致提交的非事务语句
  DDL语句: (ALTER、CREATE 和 DROP)
  DCL语句: (GRANT、REVOKE 和 SET PASSWORD)
  锁定语句: (LOCK TABLES 和 UNLOCK TABLES)
  导致隐式提交的语句示例：
  TRUNCATE TABLE
  LOAD DATA INFILE
  SELECT FOR UPDATE
  ```

#### InnoDB 事务的 ACID 是如何保证的

- ##### 基础概念

  ```mysql
  redo log          ---> 重做日志 ib_logfile0~1  50M ,轮询使用
  redo log buffer   ---> redo内存区域
  t1.ibd            ---> 存储 数据行和索引
  buffer pool       ---> 数据缓冲区池,数据和索引的缓冲

  LSN : 日志序列号
  磁盘数据页,redo文件,buffer pool,redo buffer
  MySQL 每次数据库启动,都会比较磁盘数据页和redolog的LSN,必须要求两者LSN一致数据库才能正常启动

  WAL : write ahead log 日志优先写的方式实现持久化
  脏页:  内存脏页,内存中发生了修改,没写入到磁盘之前,我们把内存页称之为脏页
  CKPT: Checkpoint,检查点,就是将脏页刷写到磁盘的动作
  TXID: 事务号,InnoDB会为每一个事务生成一个事务号,伴随着整个事务
  ```

  ![jYSnqG](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/jYSnqG.jpg)

- ##### `redo log`

  ```mysql
  # Redo是什么
  redo,顾名思义“重做日志”，是事务日志的一种。

  # 作用是什么
  在事务ACID过程中，实现的是“D”持久化的作用。对于AC也有相应的作用

  # redo日志位置
  redo的日志文件：iblogfile0 iblogfile1

  # redo buffer
  redo的buffer:数据页的变化信息+数据页当时的LSN号
  LSN：日志序列号  磁盘数据页、内存数据页、redo buffer、redolog

  # redo的刷新策略
  commit;
  刷新当前事务的redo buffer到磁盘
  还会顺便将一部分redo buffer中没有提交的事务日志也刷新到磁盘
  ```

- ##### CSR 前滚

  ```mysql
  MySQL : 在启动时,必须保证redo日志文件和数据文件LSN必须一致, 如果不一致就会触发CSR,最终保证一致
  情况一:
  我们做了一个事务,begin;update;commit.
  1.在begin ,会立即分配一个TXID=tx_01.
  2.update时,会将需要修改的数据页(dp_01,LSN=101),加载到data buffer中
  3.DBWR线程,会进行dp_01数据页修改更新,并更新LSN=102
  4.LOGBWR日志写线程,会将dp_01数据页的变化+LSN+TXID存储到redobuffer
  5. 执行commit时,LGWR日志写线程会将redobuffer信息写入redolog日志文件中,基于WAL原则,
  在日志完全写入磁盘后,commit命令才执行成功,(会将此日志打上commit标记)
  6.假如此时宕机,内存脏页没有来得及写入磁盘,内存数据全部丢失
  7.MySQL再次重启时,必须要redolog和磁盘数据页的LSN是一致的.但是,此时dp_01,TXID=tx_01磁盘是LSN=101,dp_01,TXID=tx_01,redolog中LSN=102
  MySQL此时无法正常启动,MySQL触发CSR.在内存追平LSN号,触发ckpt,将内存数据页更新到磁盘,从而保证磁盘数据页和redolog LSN一值.这时MySQL正长启动
  以上的工作过程,我们把它称之为基于REDO的"前滚操作"
  ```

- ##### `undo`回滚日志

  ```mysql
  # undo是什么
  undo,顾名思义“回滚日志”

  # 作用是什么
  在事务ACID过程中，实现的是“A” 原子性的作用
  另外CI也依赖于Undo
  在rolback时,将数据恢复到修改之前的状态
  在CSR实现的是,将redo当中记录的未提交的时候进行回滚.
  undo提供快照技术,保存事务修改之前的数据状态.保证了MVCC,隔离性,mysqldump的热备

  # 概念性的东西
  redo怎么应用的
  undo怎么应用的
  CSR(自动故障恢复)过程
  LSN :日志序列号
  TXID:事务ID
  CKPT(Checkpoint)
  ```

- ##### 锁

  ```mysql
  # 锁介绍
  锁顾名思义就是锁定的意思，提供的是隔离的方面的功能，需要配合 undo+隔离级别一起来实现

  # InnoDB锁级别
  行级锁，修改这一行就会持有这行的锁，默认情况是排他锁(悲观锁)
  悲观锁:行级锁定(行锁)
  谁先操作某个数据行,就会持有<这行>的(X)锁.
  乐观锁: 没有锁

  # 死锁
    # mysql窗口一
    begin;
    update city set countrycode='CHN' where id=1;
    update city set countrycode='CHN' where id=2;
    # mysql窗口二
    begin;
    update city set countrycode='USA' where id=2;
    update city set countrycode='USA' where id=1;

  	# 业务逻辑有问题，开发中不能出现
  	# ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction

  # 锁的作用
  在事务ACID过程中，“锁”和“隔离级别”一起来实现“I”隔离性和"C" 一致性 (redo也有参与)
  ```

- ##### 事务的隔离级别

  ```mysql
  作用: 影响到数据的读取
  默认的级别是 RR模式
  transaction_isolation   隔离级别(参数)
    # 查看隔离级别
    select @@tx_isolation;
    show variables like '%iso%';

    # RR模式
      # 创建表环境
      use world;
      begin;
      create table t1 (id int not null , ticker int null);
      desc t1;
      insert into t1 values (1,1);
      commit;
      select * from t1;

      # mysql窗口一
      use world;
      begin;
      select * from t1;  # 这一步窗口二也开始查询
      update t1 set ticker=0 where id=1;
      select * from t1;
      commit;

      # mysql窗口二
      use world;
      select * from t1;  # 和窗口一的第一次查询时间同步
      select * from t1;  # 当窗口一已经提交了事务之后再查询发现并没有改变
      # 这个现象就是可重复读现象，如果想在窗口二看到改变后的情况，先commit一下


  # 修改隔离级别
      set global transaction_isolation='read-committed';  # 此时是RC模式  需要退出窗口后重新进入生效
      # 测试，设置之后重新打开mysql窗口

      # mysql窗口一
      select @@tx_isolation;  # 查看隔离级别
      use world;
      begin;
      select * from t1;  # # 这一步窗口二也开始查询
      update t1 set ticker=1 where id=1;
      select * from t1;
      commit;
      select * from t1;

      # mysql窗口二
      selet @@tx_isolation;  # 查看隔离级别
      use world;
      select * from t1;  # 和窗口一的第一次查询时间同步
      select * from t1;  # 当窗口一提交了事务之后再查询发现此时已经改变了


  隔离级别负责的是,MVCC,读一致性问题
  RU  : 读未提交,可脏读,一般部议叙出现
  RC  : 读已提交,可能出现幻读,可以防止脏读.
  RR  : 解决了不可重复读,功能是防止"幻读"现象 ,利用的是undo的快照技术+GAP(间隙锁)+NextLock(下键锁)
  SR  : 可串行化,可以防止死锁,但是并发事务性能较差

  幻读现象是由MVCC+GAP+Next-Lock解决

  补充: 在RC级别下,可以减轻GAP+NextLock锁的问题,但是会出现幻读现象,一般在为了读一致性会在正常select后添加for update语句.但是,请记住执行完一定要commit 否则容易出现所等待比较严重.
  例如:
  [world]>select * from city where id=999 for update;
  [world]>commit;
  ```

- ##### 幻读

  ```mysql
  # 环境准备
  select @@tx_isolation;  # RC模式
  select @@autocommit;  # 0

  # 建库建表
  create database test charset utf8mb4;
  use test;
  create table t1(id int not null primary key auto_increment, num int not null);
  insert into t1(num) values (1),(3),(5);
  commit;

  # mysql 窗口一
  begin;
  update t1 set num=10 where num>=3;
  commit;
  # 上面的更新语句和窗口二的插入语句两个事务同时进行

  select * from t1;  # 此时发现会多一条 num 为 7 的列，这种现象就是幻读

  # mysql 窗口二
  begin;
  insert into t1(num) values(7);
  commit;


  # RR 模式下
      # 环境准备
      set global transaction_isolation='repeatable-read';  # 需要退出窗口后重新进入
      select @@tx_isolation;  # RR级别
      select @@autocommit;  # 0

      # 创建表
      use test;
      create table t2(id int not null primary key auto_increment, num int not null);
      insert into t2(num) values (1),(3),(5);
      alter table t2 add index idx(num);
      commit;

  		# mysql 窗口一
      begin;
      update t2 set num=10 where num>=3;  # 更新表之后开始窗口二

      # mysql 窗口二
      begin;
      insert into t1(num) values(7);
      # 当窗口一更新之后此时再插入发现夯住
      # ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
      # 防止幻读
      # 辅助索引+Next-Lock
  ```

  幻读
  ![yMjqBO](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/yMjqBO.png)

  不可幻读
  ![GhQN3e](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/GhQN3e.png)

#### InnoDB 核心参数

##### 双一标准之一(`innodb_flush_log_at_trx_commit`)

```mysql
# 参数
innodb_flush_log_at_trx_commit=1

# 作用
控制了redo buffer 刷写策略，是一个安全参数，是一个5.6版本的以上默认的参数
redo buffer ---> ib_logfileo~N

# 查看参数
select @@innodb_flush_log_at_trx_commit;

# 参数说明
1: 每次事务提交，都会立即刷下redo到磁盘(redo buffer --每事务-->os buffer --每事务--> 磁盘)
0: 表示当事务提交时，不立即做日志写入操作(redo buffer --每秒-->os buffer --每秒--> 磁盘)
2: 每次事务提交时写入文件缓存(redo buffer --每事务-->os buffer --每秒--> 磁盘)
```

##### `Innodb_flush_method`

```mysql
# 作用
	控制了redo buffer 和 data buffer 刷写磁盘的时候是否经过文件系统缓存

# 查看
  show variables like '%innodb_flush%';

# 参数值说明
  O_DIRECT  :数据缓冲区写磁盘,不走OS buffer
  fsync     :日志和数据缓冲区写磁盘,都走OS buffer
  O_DSYNC   :日志缓冲区写磁盘,不走 OS buffer

# 使用建议
  最高安全模式
  innodb_flush_log_at_trx_commit=1
  innodb_flush_method=O_DIRECT
  最高性能:
  innodb_flush_log_at_trx_commit=0
  innodb_flush_method=fsync
```

![TsmCEP](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/TsmCEP.jpg)

##### `redo`日志参数设置

```mysql
innodb_log_buffer_size=16777216  # 调大并发的数量会越多，128M起，结合业务调整，这边单位是字节
innodb_log_file_size=50331648  # 一般是log_buffer 1-2倍
innodb_log_files_in_group = 3  # 3-4组
```

##### `innodb_buffer_pool_size`

```mysql
# 一般调整为物理内存的50%-80%(系统中只有一个mysql实例)
```
