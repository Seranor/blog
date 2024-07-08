---
title: MySQL备份恢复与迁移
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-10.html
toc: true
---

### 数据库备份恢复职责

<!-- more -->

```mysql
# 备份策略
全备、增量、时间、自动

# 日常备份检查
备份存在性
备份空间够用否

# 定期恢复演练(测试库)
一季度 或者 半年

# 故障恢复
通过现有备份,能够将数据库恢复到故障之前的时间点

# 迁移
1. 停机时间
2. 回退方案
```

### 备份类型

```mysql
# 热备
在数据库正常业务时,备份数据,并且能够一致性恢复（只能是innodb）
对业务影响非常小

# 温备
锁表备份,只能查询不能修改（myisam）
影响到写入操作

# 冷备
关闭数据库业务,数据库没有任何变更的情况下,进行备份数据.
业务停止
```

### 备份方式及工具

```mysql
# 逻辑备份
基于SQL语句进行备份
mysqldump
mysqlbinlog

# 物理备份
基于磁盘数据文件备份
xtrabackup(XBK) ：percona 第三方
MySQL Enterprise Backup（MEB）

# 逻辑备份与物理备份比较
    # mysqldump
    优点：
    1.不需要下载安装
    2.备份出来的是SQL，文本格式，可读性高,便于备份处理
    3.压缩比较高，节省备份的磁盘空间

    缺点：
    4.依赖于数据库引擎，需要从磁盘把数据读出
    然后转换成SQL进行转储，比较耗费资源，数据量大的话效率较低
    建议：
    100G以内的数据量级，可以使用mysqldump
    超过TB以上，我们也可能选择的是mysqldump，配合分布式的系统
    1EB  =1024 PB =1000000 TB

    # xtrabackup
    优点：
    1.类似于直接cp数据文件，不需要管逻辑结构，相对来说性能较高
    缺点：
    2.可读性差
    3.压缩比低，需要更多磁盘空间
    建议：
    >100G<TB

```

### 备份策略

```mysql
# 备份方式
全备:全库备份，备份所有数据
增量:备份变化的数据
逻辑备份=mysqldump+mysqlbinlog
物理备份=xtrabackup_full+xtrabackup_incr+binlog或者xtrabackup_full+binlog

# 备份周期
根据数据量设计备份周期
比如：周日全备，周1-周6增量

# 其他: 主从复制备份
```

### `mysqldump`使用

#### 客户端通用命令

```mysql
# 和mysql通用命令
-u  -p   -S   -h  -P

# 本地备份
mysqldump -uroot -p  -S /tmp/mysql.sock

# 远程备份
mysqldump -uroot -p  -h 10.0.0.51 -P3306
```

#### 基本备份参数

```mysql
# 全库备份 -A
    mkdir -p /data/backup
    mysqldump -uroot -p1 -A -S /tmp/mysql.sock > /data/backup/full.sql

# 备份多个单库 -B
    # 备份worldhe testdb库
    mysqldump -uroot -p1 -B -S /tmp/mysql.sock  world testdb >/data/backup/db.sql

# 备份单个或多个表
    # 备份world下的city表和country表
    mysqldump -uroot -p1 world city country  -S /tmp/mysql.sock >/backup/bak.sql
    # 此方法只会备份建表语句和插入语句，所以恢复前需要把库建好然后 use 到库中
```

#### 高级参数使用

```mysql
# 必加参数
-R                    # 在备份时，同时备份存储过程和函数，如果没有会自动忽略
-E                    # 在备份时，同时备份EVENT，如果没有会自动忽略
--triggers            # 在备份时，同时备份触发器，如果没有会自动忽略

--master-data=2       # 记录备份开始时记录position号，可以作为将来做日志截取的起点
    功能:
    (1)在备份时，会自动记录，二进制日志文件名和位置号
        参数有:
            0 默认值
            1  以change master to命令形式，可以用作主从复制
            2  以注释的形式记录，备份时刻的文件名+postion号
    (2)自动锁表
    (3)如果配合 --single-transaction ，只对非InnoDB表进行锁表备份，InnoDB表进行“热“”备，实际上是实现快照备份

--single-transaction  # 存储引擎开启热备(快照备份)功能
		功能:
		使用 master-data 参数时
    (1)在不加 --single-transaction ,启动所有表的温备份,所有表都锁定
    (2)加上 --single-transaction ,对innodb进行快照备份,对非innodb表可以实现自动锁表功能


# 其他参数 了解功能即可
-F  # 在备份开始时,刷新一个新binlog日志，刷新的个数较多，一个库一个

--set-gtid-purged=auto
# 参数:
    auto  # 默认参数
    on
    off
# 使用场景
1. --set-gtid-purged=OFF,可以使用在日常备份参数中
mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction --set-gtid-purged=OFF >/data/backup/full.sql
2. auto , on:在构建主从复制环境时需要的参数配置
mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction --set-gtid-purged=ON >/data/backup/full.sql

--max-allowed-packet=#  # 表特别大时设置参数避免超出大小
mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction --set-gtid-purged=OFF --max-allowed-packet=256M >/data/backup/full.sql
```

#### 备份及恢复

```mysql
# 备份语句
mysqldump -uroot -p1 -A -R -E --triggers --master-data=2 --single-transaction --set-gtid-purged=OFF >/data/backup/full.sql

# 恢复
mysqldump备份的恢复方式（在生产中恢复要谨慎，恢复会删除重复的表）
set sql_log_bin=0;
source /backup/full_2018-06-28.sql

# 压缩版
mysqldump -uroot -p1 -A -R -E --triggers --max-allowed-packet=128M --master-data=2 --single-transaction|gzip > /data/backup/full_$(date +%F-%T).sql.gz

gunzip full_2021-12-27-14:19:32.sql.gz

注意:
1、mysqldump在备份和恢复时都需要mysql实例启动为前提
2、一般数据量级100G以内，大约15-45分钟可以恢复，数据量级很大很大的时候（PB、EB）
3、mysqldump是覆盖形式恢复的方法

一般我们认为，在同数据量级，物理备份要比逻辑备份速度快
逻辑备份的优势:
1、可读性强
2、压缩比很高
```

#### 故障案例恢复

```bash
# 背景
正在运行的网站系统，mysql-5.7.20 数据库，数据量50G，日业务增量1-5M。
每天23:00点，计划任务调用mysqldump执行全备脚本
年底故障演练:模拟周三上午10点误删除数据库，并进行恢复.

# 恢复思路
1、停业务，避免数据的二次伤害
2、找一个临时库，恢复周三23：00全备
3、截取周二23：00  --- 周三10点误删除之间的binlog，恢复到临时库
4、测试可用性和完整性
5、
    5.1 方法一：直接使用临时库顶替原生产库，前端应用割接到新库
    5.2 方法二：将误删除的表导出，导入到原生产库
6、开启业务
处理结果：经过20分钟的处理，最终业务恢复正常

# 模拟全备之前的数据
create database mbp charset utf8mb4;
use mbp;
create table t1 (id int);
insert into t1 values(1),(2),(3);
commit;

# 做全备
mysqldump -uroot -p1 -A -R --triggers --set-gtid-purged=OFF --master-data=2 --single-transaction> /data/backup/full_$(date +%F).sql
ls /data/backup/

# 模拟增量的数据
use mbp;
insert into t1 values(11),(22),(33);
commit;
create table t2 (id int);
insert into t2 values(11),(22),(33);
commit;

# 模拟数据库损害
rm -rf /data/mysql/data/*
pkill mysql
rm -rf /data/mysql/data/*

# 恢复数据
# 重新初始化数据库
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data

# 启动数据库
service mysqld restart

# 全备恢复，此时没有增量的数据
set sql_log_bin=0;
source /data/backup/full_2021-12-27.sql;
set sql_log_bin=1;
flush privileges;  # 刷新权限，恢复账号权限等

# 恢复增量的数据---> 全备到故障时刻的数据
# 找日志起点和终点
vim /data/backup/full_2021-12-27.sql
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000005', MASTER_LOG_POS=949;  # 找打全备时的 binlog 日志文件，--master-data=2 参数实现的

show binlog events in 'mysql-bin.000005';
# 下图可以看到 GTID 号应该是11-13 就是后面增量的数据信息
# 截取日志
mysqlbinlog --skip-gtids --include-gtids='b25a4cc7-609b-11ec-afea-525400aaf73e:11-13' /data/binlog/mysql-bin.000005 > /data/backup/bin.sql

# 恢复增量数据
set sql_log_bin=0;
source /data/backup/bin.sql;
set sql_log_bin=1;

# 检查是否恢复
select * from mbp.t2;
```

![image-20211227124136331](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211227124136331.png)

#### 单表的恢复

```mysql
# 从全备中导出单表备份
# 获取表结构
sed -e '/./{H;$!d;}' -e 'x;/CREATE TABLE `city`/!d;q' full.sql > createtable.sql

# 获取INSERT INTO语句，用于数据的恢复
grep -i 'INSERT INTO `city`' full.sql > data.sql

# 获取单库的备份，可以直接创建库就行
sed -n '/^-- Current Database: `world`/,/^-- Current Database: `/p' full.sql > world.sql
```

### `xtrabackup`使用

#### 安装

```bash
# 下载依赖
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL libev

# 这个版本只支持8.0数据库以下的，最新版本支持8.0
# wget https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/sql/percona-xtrabackup-24-2.4.12-1.el7.x86_64.rpm
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.12/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.12-1.el7.x86_64.rpm

# 安装
yum -y install percona-xtrabackup-24-2.4.4-1.el7.x86_64.rpm

# 版本
innobackupex --version
```

#### 备份方式

```mysql
1.对于非Innodb表（比如 myisam）是，锁表cp数据文件，属于一种温备份。
2.对于Innodb的表（支持事务的），不锁表，拷贝数据页，最终以数据文件的方式保存下来，把一部分redo和undo一并备走，属于热备方式
```

#### InnoDB 表备份恢复流程

```mysql
1.xbk备份执行的瞬间,立即触发ckpt,已提交的数据脏页,从内存刷写到磁盘,并记录此时的LSN号
2.备份时，拷贝磁盘数据页，并且记录备份过程中产生的redo和undo一起拷贝走,也就是checkpoint LSN之后的日志
3.在恢复之前，模拟Innodb“自动故障恢复”的过程，将redo（前滚）与undo（回滚）进行应用
4.恢复过程是cp 备份到原来数据目录下
```

#### `innobackupex`使用

```mysql
# 会自动读取/etc/my.cnf 或者手动指定
    # 配置文件修改
    vim /etc/my.cnf
    ...
    [client]
    socket=/tmp/mysql.sock
    ...

    service mysqld restart

    # 指定配置文件
    innobackupex --defaults-file=/etc/my.cnf --user=root --password=1 /data/bak

# 默认是全备
mkdir -p /data/bak
innobackupex --user=root --password=1 /data/bak

innobackupex --defaults-file=/etc/my.cnf --user=root --password=1 --no-timestamp /data/bak/full_$(date +%F)
--no-timestap  # 自定义备份目录名

# 检查
ls /data/bak  # 会自动以当前时间戳创建目录



# 备份文件说明
    cat xtrabackup_binlog_info
    mysql-bin.000010	274	5bee84cd-66ce-11ec-b1fd-525400aaf73e:1,
    934ba059-66ca-11ec-8dce-525400aaf73e:1,
    b25a4cc7-609b-11ec-afea-525400aaf73e:1-13
    # 备份时刻的binlog位置
    # 记录的是备份时刻，binlog的文件名字和当时的结束的position，可以用来作为截取binlog时的起点


    cat xtrabackup_checkpoints
    backup_type = full-backuped
    from_lsn = 0          # 上次所到达的LSN号(对于全备就是从0开始,对于增量有别的显示方法)
    to_lsn = 155266303    #  备份开始时间(ckpt)点数据页的LSN
    last_lsn = 155266312  # 备份结束后，redo日志最终的LSN
    compact = 0
    recover_binlog_info = 0
    # 备份时刻，立即将已经commit过的，内存中的数据页刷新到磁盘(CKPT).开始备份数据，数据文件的LSN会停留在to_lsn位置
    # 备份时刻有可能会有其他的数据写入，已备走的数据文件就不会再发生变化了
    # 在备份过程中，备份软件会一直监控着redo的undo，如果一旦有变化会将日志也一并备走，并记录LSN到last_lsn
    # 从to_lsn ----> last_lsn 就是，备份过程中产生的数据变化.

    xtrabackup_info  # 整体统计
    xtrabackup_logfile  # 部分redo日志
```

#### 全备恢复

```bash
# 全备
innobackupex --defaults-file=/etc/my.cnf --user=root --password=1 /data/bak

# 模拟删除(不代表生成操作)
pkill mysql
rm -rf /data/mysql/data/*

# 恢复步骤
# 准备备份(Prepared)，将redo进行重做，已提交的写到数据文件，未提交的使用undo回滚掉，模拟了CSR的过程
innobackupex --apply-log  /data/bak/2021-12-27_14-51-12/

# 恢复启动
cp -a /data/bak/2021-12-27_14-51-12/* /data/mysql/data/
chown -R mysql:mysql /data/
service mysqld restart
```

#### 增量备份和恢复

```bash
# 说明
(1)增量备份的方式，是基于上一次备份进行增量
(2)增量备份无法单独恢复。必须基于全备进行恢复
(3)所有增量必须要按顺序合并到全备中

# 开始模拟
# 全备一次
innobackupex --user=root --password=1 --no-timestamp /data/bak/full_$(date +%F)

# 第一次数据变化
create database xbk charset utf8mb4;
use xbk;
create table t1(id int);
insert into t1 values (1),(2),(3);
commit;

# 第一次数据变化的增量备份，基于全备而备份
innobackupex --user=root --password=1 --no-timestamp --incremental --incremental-basedir=/data/bak/full_2021-12-27 /data/bak/inc1

# 第二次数据变化
use xbk;
create table t2(id int);
insert into t2 values (11),(22),(33);
commit;

# 第二次数据变化的增量备份，基于第一次增量备份进行备份
innobackupex --user=root --password=1 --no-timestamp --incremental --incremental-basedir=/data/bak/inc1 /data/bak/inc2

# 判断是否备份成功，可以查看每个xtrabackup_checkpoints信息是否接上
cd /data/bak/
cat full_2021-12-27/xtrabackup_checkpoints
cat inc1/xtrabackup_checkpoints
cat inc2/xtrabackup_checkpoints

# 增量恢复
合并所有的增备到全备
每个XBK备份都需要恢复准备(prepare)
参数:
--apply-log
--redo-noly

# 目录说明
/data/bak/full_2021-12-27  # 全备目录
/data/bak/inc1  # 第一次增备目录
/data/bak/inc2  # 第二次增备目录

# 整理全备
innobackupex --apply-log --redo-only /data/bak/full_2021-12-27

# 整理并合并第一次数据变化的增量到全备
innobackupex --apply-log --redo-only --incremental-dir=/data/bak/inc1 /data/bak/full_2021-12-27

# 整理并合并第二次数据变化的增量到全备中
innobackupex --apply-log --incremental-dir=/data/bak/inc2 /data/bak/full_2021-12-27

# 再次整理全备
innobackupex --apply-log /data/bak/full_2021-12-27

# 模拟故障
pkill mysql
rm -rf /data/mysql/data*

# 恢复
cp -a /data/bak/full_2021-12-27/* /data/mysql/data/
chown -R /data/
service mysqld start
```

检查是否增量备份成功：
![40j93z](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/40j93z.png)

#### 扩展

```bash
# xtrabackup的全备目录内有表空间结构等信息
ll /data/bak/full_2021-12-27/xbk/
总用量 220
-rw-r----- 1 mysql mysql    67 12月 27 17:10 db.opt
-rw-r----- 1 mysql mysql  8556 12月 27 17:10 t1.frm
-rw-r----- 1 mysql mysql 98304 12月 27 17:07 t1.ibd
-rw-r----- 1 mysql mysql  8556 12月 27 17:10 t2.frm
-rw-r----- 1 mysql mysql 98304 12月 27 17:10 t2.ibd
# 所以可以考虑使用迁移表空间恢复，参考 8、存储引擎 ---> 迁移表空间


# 生产中的备份命令
innobackupex --defaults-file=/etc/my.cnf --user=root --password=1 --no-timestamp --stream=tar --use-memory=256M --parallel=2 /data/bak/full | gzip | ssh root@192.168.0.12 "cat - > /data/bak/full.tgz"

# 参数说明
--stream=tar       # 压缩
--use-memory=256M  # 开辟内存空间
--parallel=8       # 并发，依据CPU核心调整
ssh root@192.168.0.12 "cat - > /data/bak/full.tgz"  # 推送到 192.168.0.12 服务器上
```

### MySQL 数据迁移

```mysql
1. 换主机
    在线使用mysqldump和xtrabackup备份，传输到目标主机
    追加所有备份后的日志
    申请停机
    剩余部分使用binlog继续恢复(使用主从的方式替代)
    校验数据进行业务割接

2. 换版本升级
    5.6 ---> 5.7
    5.7 mysqld_safe mysql_upgrade

		建议使用mysqldump 逻辑备份方式，按业务库进行分别备份，排除掉information_schema,performance_schema.sys
		恢复完成后，升级数据字典

3. 异构迁移 ---> 系统不一样
4. 异构迁移 ---> 数据库产品不同
```
