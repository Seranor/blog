---
title: MySQL优化
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MySQL
categories:
  - MySQL
url: post/mysql-12.html
toc: true
---

### 优化风险

<!-- more -->

```mysql
优化不总是对一个单纯的环境进行！还很可能是一个复杂的已投产的系统。
优化手段本来就有很大的风险，只不过你没能力意识到和预见到！
任何的技术可以解决一个问题，但必然存在带来一个问题的风险！
对于优化来说解决问题而带来的问题控制在可接受的范围内才是有成果。
保持现状或出现更差的情况都是失败！

稳定性和业务可持续性通常比性能更重要！
优化不可避免涉及到变更，变更就有风险！
优化使性能变好，维持和变差是等概率事件！
优化不能只是数据库管理员担当风险，但会所有的人分享优化成果！
所以优化工作是由业务需要驱使的！！！

不应该一个人独自参与优化，风险要和参与人员一起承担
```

### 优化方向

```mysql
安全优化（业务持续性）
性能优化（业务高效性）
```

### 优化范围和思路

```mysql
存储、主机和操作系统:
    主机架构稳定性
    I/O规划及配置
    Swap
    OS内核参数
        网络问题

应用程序:（Index，lock，session）
        应用程序稳定性和性能
        SQL语句性能
    串行访问资源
    性能欠佳会话管理

数据库优化:（内存、数据库设计、参数）
    内存
    数据库结构(物理&逻辑)
    实例配置
```

![KqOaeJ](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/KqOaeJ.jpg)

### 数据库优化工具

```mysql
    show status
    show variables
    show index
    show processlist
    show slave status
    show engine innodb status
    desc/explain
         slowlog

    扩展类深度优化:
    pt系列
    mysqlslap
    sysbench
    information_schema
    performance_schema
    sys
```

### 参数优化

#### 参数优化前压力测试

```mysql
# 模拟数据库数据
create database test charset utf8mb4 collate utf8mb4_bin;
use test;
create table t_100w (id int,num int,k1 char(2),k2 char(4),dt timestamp);

delimiter //
create  procedure rand_data(in num int)
begin
declare str char(62) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
declare str2 char(2);
declare str4 char(4);
declare i int default 0;
while i<num do
set str2=concat(substring(str,1+floor(rand()*61),1),substring(str,1+floor(rand()*61),1));
set str4=concat(substring(str,1+floor(rand()*61),2),substring(str,1+floor(rand()*61),2));
set i=i+1;
insert into t_100w values (i,floor(rand()*num),str2,str4,now());
end while;
end;
//
delimiter ;

# 插入100w条数据
call rand_data(10000000);
commit;

# 压力测试
mysqlslap --defaults-file=/etc/my.cnf \
--concurrency=100 --iterations=1 --create-schema='test' \
--query="select * from test.t_100w where k2='FGCD'" engine=innodb \
--number-of-queries=200 -uroot -p123 -verbose
```

#### 优化参数说明

- `Max_connections(*)`

  ```mysql
  # 简介
  Mysql的最大连接数，如果服务器的并发请求量比较大，可以调高这个值，当然这是要建立在机器能够支撑的情况下，因为如果连接数越来越多，mysql会为每个连接提供缓冲区，就会开销的越多的内存，所以需要适当的调整该值，不能随便去提高设值

  # 判断依据
  show variables like 'max_connections';
      +-----------------+-------+
      | Variable_name   | Value |
      +-----------------+-------+
      | max_connections | 151   |
      +-----------------+-------+
  show status like 'Max_used_connections';
      +----------------------+-------+
      | Variable_name        | Value |
      +----------------------+-------+
      | Max_used_connections | 101   |
      +----------------------+-------+

  # 修改方式
  vim /etc/my.cnf
  Max_connections=1024

  # 补充
  1.开启数据库时,我们可以临时设置一个比较大的测试值
  2.观察show status like 'Max_used_connections';变化
  3.如果max_used_connections跟max_connections相同
      那么就是max_connections设置过低或者超过服务器的负载上限了
      低于10%则设置过大
  ```

- `back_log`

  ```mysql
  # 简介
  mysql能暂存的连接数量，当主要mysql线程在一个很短时间内得到非常多的连接请求时候它就会起作用，如果mysql的连接数据达到max_connections时候，新来的请求将会被存在堆栈中，等待某一连接释放资源，该推栈的数量及back_log,如果等待连接的数量超过back_log，将不被授予连接资源。
  back_log值指出在mysql暂时停止回答新请求之前的短时间内有多少个请求可以被存在推栈中，只有如果期望在一个短时间内有很多连接的时候需要增加它

  # 判断依据
  show full processlist
  发现大量的待连接进程时，就需要加大back_log或者加大max_connections的值

  # 修改方式举例
  vim /etc/my.cnf
  back_log=1024
  ```

- `wait_timeout和interactive_timeout`

  ```mysql
  # 简介
  wait_timeout：指的是mysql在关闭一个非交互的连接之前所要等待的秒数
  interactive_timeout：指的是mysql在关闭一个交互的连接之前所需要等待的秒数，比如我们在终端上进行mysql管理，使用的即使交互的连接，这时候，如果没有操作的时间超过了interactive_time设置的时间就会自动的断开，默认的是28800，可调优为7200。
  wait_timeout:如果设置太小，那么连接关闭的就很快，从而使一些持久的连接不起作用

  # 设置建议
  如果设置太大，容易造成连接打开时间过长，在show processlist时候，能看到很多的连接 ，一般希望wait_timeout尽可能低

  # 修改方式举例
  wait_timeout=60
  interactive_timeout=1200
  长连接的应用，为了不去反复的回收和分配资源，降低额外的开销
  一般我们会将wait_timeout设定比较小，interactive_timeout要和应用开发人员沟通长链接的应用是否很多。如果他需要长链接，那么这个值可以不需要调整。
  另外还可以使用类外的参数弥补。
  ```

- `key_buffer_size(*)`

  ```mysql
  # 简介
  key_buffer_size指定索引缓冲区的大小，它决定索引处理的速度，尤其是索引读的速度
  1.此参数与myisam表的索引有关
  2.临时表的创建有关（多表链接、子查询中、union）
       在有以上查询语句出现的时候，需要创建临时表，用完之后会被丢弃
       临时表有两种创建方式：
                          内存中------->key_buffer_size
                          磁盘上------->ibdata1(5.6)
                                        ibtmp1 (5.7）
  # 设置依据
  通过key_read_requests和key_reads可以直到key_baffer_size设置是否合理
  mysql> show variables like "key_buffer_size%";
  +-----------------+---------+
  | Variable_name   | Value   |
  +-----------------+---------+
  | key_buffer_size | 8388608 |
  +-----------------+---------+
  1 row in set (0.00 sec)

  mysql> show status like "key_read%";
  +-------------------+-------+
  | Variable_name     | Value |
  +-------------------+-------+
  | Key_read_requests | 10    |
  | Key_reads         | 2     |
  +-------------------+-------+
  2 rows in set (0.00 sec)

  一共有10个索引读取请求，有2个请求在内存中没有找到直接从硬盘中读取索引
  控制在 5%以内
  注：key_buffer_size只对myisam表起作用，即使不使用myisam表，但是内部的临时磁盘表是myisam表，也要使用该值。
  可以使用检查状态值created_tmp_disk_tables得知：

  mysql> show status like "created_tmp%";
  +-------------------------+-------+
  | Variable_name           | Value |
  +-------------------------+-------+
  | Created_tmp_disk_tables | 0     |
  | Created_tmp_files       | 6     |
  | Created_tmp_tables      | 1     |
  +-------------------------+-------+
  3 rows in set (0.00 sec)

  通常地，我们习惯以 Created_tmp_tables/(Created_tmp_disk_tables + Created_tmp_tables)
  Created_tmp_disk_tables/(Created_tmp_disk_tables + Created_tmp_tables)

  或者已各自的一个时段内的差额计算，来判断基于内存的临时表利用率。所以，我们会比较关注 Created_tmp_disk_tables 是否过多，从而认定当前服务器运行状况的优劣。
  Created_tmp_disk_tables/(Created_tmp_disk_tables + Created_tmp_tables)
  控制在5%-10%以内
  看以下例子：
  在调用mysqldump备份数据时，大概执行步骤如下：
  180322 17:39:33       7 Connect     root@localhost on
  7 Query       /*!40100 SET @@SQL_MODE='' */
  7 Init DB     guo
  7 Query       SHOW TABLES LIKE 'guo'
  7 Query       LOCK TABLES `guo` READ /*!32311 LOCAL */
  7 Query       SET OPTION SQL_QUOTE_SHOW_CREATE=1
  7 Query       show create table `guo`
  7 Query       show fields from `guo`
  7 Query       show table status like 'guo'
  7 Query       SELECT /*!40001 SQL_NO_CACHE */ * FROM `guo`
  7 Query       UNLOCK TABLES
  7 Quit

  其中，有一步是：show fields from `guo`。从slow query记录的执行计划中，可以知道它也产生了 Tmp_table_on_disk。
  所以说，以上公式并不能真正反映到mysql里临时表的利用率，有些情况下产生的 Tmp_table_on_disk 我们完全不用担心，因此没必要过分关注 Created_tmp_disk_tables，但如果它的值大的离谱的话，那就好好查一下，你的服务器到底都在执行什么查询了
  # 配置方法
  key_buffer_size=64M
  ```

- `query_cache_size`

  ```bash
  # 简介
  查询缓存简称QC，使用查询缓冲，mysql将查询结果存放在缓冲区中，今后对于同样的select语句（区分大小写）,将直接从缓冲区中读取结果。

  SQL层：
  select * from t1 where name=:NAME;
  select * from t1 where name=:NAME;

  1、查询完结果之后，会对SQL语句进行hash运算，得出hash值,我们把他称之为SQL_ID
  2、会将存储引擎返回的结果+SQL_ID存储到缓存中。

  存储方式：
  例子：select * from t1  where id=10;      100次

  1、将select * from t1  where id=10; 进行hash运算计算出一串hash值，我们把它称之为"SQL_ID"
  2、将存储引擎返回上来的表的内容+SQLID存储到查询缓存中

  使用方式：
  1、一条SQL执行时，进行hash运算，得出SQLID，去找query cache
  2、如果cache中有，则直接返回数据行，如果没有，就走原有的SQL执行流程

  一个sql查询如果以select开头，那么mysql服务器将尝试对其使用查询缓存。
  注：两个sql语句，只要想差哪怕是一个字符（列如大小写不一样；多一个空格等）,那么这两个sql将使用不同的一个cache。

  # 判断依据
  mysql> show status like "%Qcache%";
  +-------------------------+---------+
  | Variable_name           | Value   |
  +-------------------------+---------+
  | Qcache_free_blocks      | 1       |
  | Qcache_free_memory      | 1031360 |
  | Qcache_hits             | 0       |
  | Qcache_inserts          | 0       |
  | Qcache_lowmem_prunes    | 0       |
  | Qcache_not_cached       | 2002    |
  | Qcache_queries_in_cache | 0       |
  | Qcache_total_blocks     | 1       |
  +-------------------------+---------+
  8 rows in set (0.00 sec)

  ---------------------状态说明--------------------
  Qcache_free_blocks：缓存中相邻内存块的个数。
  如果该值显示较大，则说明Query Cache 中的内存碎片较多了，FLUSH QUERY CACHE会对缓存中的碎片进行整理，从而得到一个空闲块。
  注：当一个表被更新之后，和它相关的cache
  blocks将被free。但是这个block依然可能存在队列中，除非是在队列的尾部。可以用FLUSH QUERY CACHE语句来清空free blocks

  Qcache_free_memory：Query Cache 中目前剩余的内存大小。通过这个参数我们可以较为准确的观察出当前系统中的Query Cache 内存大小是否足够，是需要增加还是过多了。

  Qcache_hits：表示有多少次命中缓存。我们主要可以通过该值来验证我们的查询缓存的效果。数字越大，缓存效果越理想。

  Qcache_inserts：表示多少次未命中然后插入，意思是新来的SQL请求在缓存中未找到，不得不执行查询处理，执行查询处理后把结果insert到查询缓存中。这样的情况的次数越多，表示查询缓存应用到的比较少，效果也就不理想。当然系统刚启动后，查询缓存是空的，这很正常。

  Qcache_lowmem_prunes：
  多少条Query因为内存不足而被清除出QueryCache。通过“Qcache_lowmem_prunes”和“Qcache_free_memory”相互结合，能够更清楚的了解到我们系统中Query Cache 的内存大小是否真的足够，是否非常频繁的出现因为内存不足而有Query 被换出。这个数字最好长时间来看；如果这个数字在不断增长，就表示可能碎片非常严重，或者内存很少。（上面的free_blocks和free_memory可以告诉您属于哪种情况）

  Qcache_not_cached：不适合进行缓存的查询的数量，通常是由于这些查询不是 SELECT 语句或者用了now()之类的函数。

  Qcache_queries_in_cache：当前Query Cache 中cache 的Query 数量；
  Qcache_total_blocks：当前Query Cache 中的block 数量；。
  Qcache_hits / (Qcache_inserts+Qcache_not_cached+Qcache_hits)
      90/         10000             0             90

  如果出现hits比例过低，其实就可以关闭查询缓存了。使用redis专门缓存数据库

  Qcache_free_blocks    来判断碎片
  Qcache_free_memory   +   Qcache_lowmem_prunes  来判断内存够不够
  Qcache_hits 多少次命中  Qcache_hits / (Qcache_inserts+Qcache_not_cached+Qcache_hits)

  # 配置示例
  mysql> show variables like '%query_cache%' ;
  +------------------------------+---------+
  | Variable_name                | Value   |
  +------------------------------+---------+
  | have_query_cache             | YES     |
  | query_cache_limit            | 1048576 |
  | query_cache_min_res_unit     | 4096    |
  | query_cache_size             | 1048576 |
  | query_cache_type             | OFF     |
  | query_cache_wlock_invalidate | OFF     |
  +------------------------------+---------+
  6 rows in set (0.00 sec)

  mysql>
  -------------------配置说明-------------------------------
  以上信息可以看出query_cache_type为off表示不缓存任何查询

  各字段的解释：
  query_cache_limit：超过此大小的查询将不缓存
  query_cache_min_res_unit：缓存块的最小大小，query_cache_min_res_unit的配置是一柄”双刃剑”，默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数据查询，就容易造成内存碎片和浪费。
  query_cache_size：查询缓存大小 (注：QC存储的最小单位是1024byte，所以如果你设定了一个不是1024的倍数的值，这个值会被四舍五入到最接近当前值的等于1024的倍数的值。)

  query_cache_type：缓存类型，决定缓存什么样的查询，注意这个值不能随便设置，必须设置为数字，可选项目以及说明如下：
  如果设置为0，那么可以说，你的缓存根本就没有用，相当于禁用了。
  如果设置为1，将会缓存所有的结果，除非你的select语句使用SQL_NO_CACHE禁用了查询缓存。
  如果设置为2，则只缓存在select语句中通过SQL_CACHE指定需要缓存的查询。

  修改/etc/my.cnf,配置完后的部分文件如下：
  query_cache_size=128M
  query_cache_type=1
  ```

- `max_connect_errors`

  ```mysql
  # 简介
  max_connect_errors是一个mysql中与安全有关的计数器值，它负责阻止过多尝试失败的客户端以防止暴力破解密码等情况，当超过指定次数，mysql服务器将禁止host的连接请求，直到mysql服务器重启或通过flush hosts命令清空此host的相关信息 max_connect_errors的值与性能并无太大关系

  # 修改方式
  /etc/my.cnf
  max_connect_errors=2000
  ```

- `sort_buffer_size`

  ```mysql
  # 简介
  每个需要进行排序的线程分配该大小的一个缓冲区。增加这值加速
  ORDER BY
  GROUP BY
  distinct
  union

  # 配置依据
  Sort_Buffer_Size并不是越大越好，由于是connection级的参数，过大的设置+高并发可能会耗尽系统内存资源。
  列如：500个连接将会消耗500*sort_buffer_size（2M）=1G内存

  # 配置方法
  /etc/my.cnf
  sort_buffer_size=1M
  ```

- `max_allowed_packet(*)`

  ```mysql
  # 简介：
  mysql根据配置文件会限制，server接受的数据包大小。

  # 配置依据：
  有时候大的插入和更新会受max_allowed_packet参数限制，导致写入或者更新失败，更大值是1GB，必须设置1024的倍数

  # 配置方法：
  max_allowed_packet=32M
  ```

- `join_buffer_size`

  ```mysql
  select a.name,b.name from a join b on a.id=b.id where xxxx
  用于表间关联缓存的大小，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。
  尽量在SQL与方面进行优化，效果较为明显
  优化的方法：在on条件列加索引，至少应当是有MUL索引
  ```

- `thread_cache_size(*)`

  ```mysql
  # 简介
  服务器线程缓存，这个值表示可以重新利用保存在缓存中线程的数量,当断开连接时,那么客户端的线程将被放到缓存中以响应下一个客户而不是销毁(前提是缓存数未达上限),如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，增加这个值可以改善系统性能.

  # 配置依据
  通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用。
  设置规则如下：1GB 内存配置为8，2GB配置为16，3GB配置为32，4GB或更高内存，可配置更大。
  服务器处理此客户的线程将会缓存起来以响应下一个客户而不是销毁(前提是缓存数未达上限)

  试图连接到MySQL(不管是否连接成功)的连接数
  mysql>  show status like 'threads_%';
  +-------------------+-------+
  | Variable_name     | Value |
  +-------------------+-------+
  | Threads_cached    | 8     |
  | Threads_connected | 2     |
  | Threads_created   | 4783  |
  | Threads_running   | 1     |
  +-------------------+-------+
  4 rows in set (0.00 sec)

  Threads_cached :代表当前此时此刻线程缓存中有多少空闲线程。
  Threads_connected:代表当前已建立连接的数量，因为一个连接就需要一个线程，所以也可以看成当前被使用的线程数。
  Threads_created:代表从最近一次服务启动，已创建线程的数量，如果发现Threads_created值过大的话，表明MySQL服务器一直在创建线程，这也是比较耗cpu SYS资源，可以适当增加配置文件中thread_cache_size值。
  Threads_running :代表当前激活的（非睡眠状态）线程数。并不是代表正在使用的线程数，有时候连接已建立，但是连接处于sleep状态。

  # 配置方法：
  thread_cache_size=32

  整理：
  Threads_created  ：一般在架构设计阶段，会设置一个测试值，做压力测试。
  结合zabbix监控，看一段时间内此状态的变化。
  如果在一段时间内，Threads_created趋于平稳，说明对应参数设定是OK。
  如果一直陡峭的增长，或者出现大量峰值，那么继续增加此值的大小，在系统资源够用的情况下（内存）
  ```

- `innodb_buffer_pool_size(*)`

  ```mysql
  # 简介
  对于InnoDB表来说，innodb_buffer_pool_size的作用就相当于key_buffer_size对于MyISAM表的作用一样

  # 配置依据：
  InnoDB使用该参数指定大小的内存来缓冲数据和索引。
  对于单独的MySQL数据库服务器，最大可以把该值设置成物理内存的80%,一般我们建议不要超过物理内存的70%

  # 配置方法
  innodb_buffer_pool_size=2048M
  ```

- `innodb_flush_log_at_trx_commit(*)双一标准`

  ```mysql
  # 简介
  主要控制了innodb将log buffer中的数据写入日志文件并flush磁盘的时间点，取值分别为0、1、2三个。
  0，表示当事务提交时，不做日志写入操作，而是每秒钟将log buffer中的数据写入日志文件并flush磁盘一次；
  1，
  每次事务的提交都会引起redo日志文件写入、flush磁盘的操作，确保了事务的ACID；
  2，每次事务提交引起写入日志文件的动作,但每秒钟完成一次flush磁盘操作。

  # 配置依据
  实际测试发现，该值对插入数据的速度影响非常大，设置为2时插入10000条记录只需要2秒，设置为0时只需要1秒，而设置为1时则需要229秒。因此，MySQL手册也建议尽量将插入操作合并成一个事务，这样可以大幅提高速度。
  根据MySQL官方文档，在允许丢失最近部分事务的危险的前提下，可以把该值设为0或2。

  # 配置方法
  innodb_flush_log_at_trx_commit=1
  双1标准中的一个1
  ```

- `innodb_thread_concurrency`

  ```mysql
  # 简介
  此参数用来设置innodb线程的并发数量，默认值为0表示不限制。

  # 配置依据
  在官方doc上，对于innodb_thread_concurrency的使用，也给出了一些建议，如下：
  如果一个工作负载中，并发用户线程的数量小于64，建议设置innodb_thread_concurrency=0；
  如果工作负载一直较为严重甚至偶尔达到顶峰，建议先设置innodb_thread_concurrency=128，
  并通过不断的降低这个参数，96, 80, 64等等，直到发现能够提供最佳性能的线程数，
  例如，假设系统通常有40到50个用户，但定期的数量增加至60，70，甚至200。你会发现，
  性能在80个并发用户设置时表现稳定，如果高于这个数，性能反而下降。在这种情况下，
  建议设置innodb_thread_concurrency参数为80，以避免影响性能。
  如果你不希望InnoDB使用的虚拟CPU数量比用户线程使用的虚拟CPU更多（比如20个虚拟CPU），
  建议通过设置innodb_thread_concurrency 参数为这个值（也可能更低，这取决于性能体现），
  如果你的目标是将MySQL与其他应用隔离，你可以l考虑绑定mysqld进程到专有的虚拟CPU。
  但是需 要注意的是，这种绑定，在myslqd进程一直不是很忙的情况下，可能会导致非最优的硬件使用率。在这种情况下，
  你可能会设置mysqld进程绑定的虚拟 CPU，允许其他应用程序使用虚拟CPU的一部分或全部。
  在某些情况下，最佳的innodb_thread_concurrency参数设置可以比虚拟CPU的数量小。
  定期检测和分析系统，负载量、用户数或者工作环境的改变可能都需要对innodb_thread_concurrency参数的设置进行调整。

  128   -----> top  cpu
  设置标准：
  1、当前系统cpu使用情况，均不均匀
  top

  2、当前的连接数，有没有达到顶峰
  show status like 'threads_%';
  show processlist;

  # 配置方法：
  innodb_thread_concurrency=8
  方法:
      1. 看top ,观察每个cpu的各自的负载情况
      2. 发现不平均,先设置参数为cpu个数,然后不断增加(一倍)这个数值
      3. 一直观察top状态,直到达到比较均匀时,说明已经到位了.
  ```

- `innodb_log_buffer_size`

  ```mysql
  此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，对于较大的事务，可以增大缓存大小。
  innodb_log_buffer_size=128M

  设定依据：
  1、大事务： 存储过程调用 CALL
  2、多事务
  ```

- `innodb_log_file_size = 100M(*)`

  ```mysql
  设置 ib_logfile0  ib_logfile1
  此参数确定数据日志文件的大小，以M为单位，更大的设置可以提高性能

  innodb_log_file_size = 100M
  ```

- `innodb_log_files_in_group = 3(*)`

  ```mysql
  为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为3
  ```

- `read_buffer_size = 1M`

  ```mysql
  MySql读入缓冲区大小
  对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区
  如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能，和 sort_buffer_size一样，该参数对应的分配内存也是每个连接独享
  ```

- `read_rnd_buffer_size = 1M`

  ```mysql
  MySql的随机读（查询操作）缓冲区大小

  当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大

  注：顺序读是指根据索引的叶节点数据就能顺序地读取所需要的行数据。随机读是指一般需要根据辅助索引叶节点中的主键寻找实际行数据，而辅助索引和主键所在的数据段不同，因此访问方式是随机的。
  ```

- `bulk_insert_buffer_size = 8M`

  ```mysql
  批量插入数据缓存大小，可以有效提高插入效率，默认为8M
  tokuDB    percona
  myrocks
  RocksDB
  TiDB
  MongoDB
  ```

- `binary log(*)双一标准`

  ```mysql
  binlog_cache_size = 2M  # 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存, 提高记录bin-log的效率。没有什么大事务，dml也不是很频繁的情况下可以设置小一点，如果事务大而且多，dml操作也频繁，则可以适当的调大一点。前者建议是--1M，后者建议是：即 2--4M

  max_binlog_cache_size = 8M  # 表示的是binlog 能够使用的最大cache 内存大小

  max_binlog_size= 512M  # 指定binlog日志文件的大小，如果当前的日志大小达到max_binlog_size，还会自动创建新的二进制日志。你不能将该变量设置为大于1GB或小于4096字节。默认值是1GB。在导入大容量的sql文件时，建议关闭sql_log_bin，否则硬盘扛不住，而且建议定期做删除。

  expire_logs_days = 7  # 定义了mysql清除过期日志的时间 二进制日志自动删除的天数。默认值为0,表示“没有自动删除”。

  log-bin=/data/mysql-bin
  binlog_format=row
  sync_binlog=1

  双1标准(基于安全的控制)
  sync_binlog=1   # 时候刷新binlog到磁盘，每次事务commit
  innodb_flush_log_at_trx_commit=1
  set sql_log_bin=0;
  show status like 'com_%';
  ```

- 安全参数

  ```mysql
  Innodb_flush_method=(O_DIRECT, fsync)
  1、fsync    ：
  （1）在数据页需要持久化时，首先将数据写入OS buffer中，然后由os决定什么时候写入磁盘
  （2）在redo buffuer需要持久化时，首先将数据写入OS buffer中，然后由os决定什么时候写入磁盘
  但，如果innodb_flush_log_at_trx_commit=1的话，日志还是直接每次commit直接写入磁盘
  2、 Innodb_flush_method=O_DIRECT
  （1）在数据页需要持久化时，直接写入磁盘
  （2）在redo buffuer需要持久化时，首先将数据写入OS buffer中，然后由os决定什么时候写入磁盘
  但，如果innodb_flush_log_at_trx_commit=1的话，日志还是直接每次commit直接写入磁盘

  最安全模式：
  innodb_flush_log_at_trx_commit=1
  innodb_flush_method=O_DIRECT
  最高性能模式：
  innodb_flush_log_at_trx_commit=0
  innodb_flush_method=fsync

  一般情况下，我们更偏向于安全。
  “双一标准”
  innodb_flush_log_at_trx_commit=1
  sync_binlog=1
  innodb_flush_method=O_DIRECT
  ```

#### 参数优化结果

```mysql
[mysqld]
basedir=/usr/local/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
log-error=/var/log/mysql.log
log_bin=/data/binlog/mysql-bin
binlog_format=row
skip-name-resolve
server-id=7
autocommit=0
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
relay_log_purge=0
max_connections=1024
back_log=128
wait_timeout=60
interactive_timeout=7200
key_buffer_size=16M
query_cache_size=64M
query_cache_type=1
query_cache_limit=50M
max_connect_errors=20
sort_buffer_size=2M
max_allowed_packet=32M
join_buffer_size=2M
thread_cache_size=200
innodb_buffer_pool_size=1024M
innodb_flush_log_at_trx_commit=1
innodb_log_buffer_size=32M
innodb_log_file_size=128M
innodb_log_files_in_group=3
binlog_cache_size=2M
max_binlog_cache_size=8M
max_binlog_size=512M
expire_logs_days=7
read_buffer_size=2M
read_rnd_buffer_size=2M
bulk_insert_buffer_size=8M
[client]
socket=/tmp/mysql.sock

# 再次压力测试
mysqlslap --defaults-file=/etc/my.cnf --concurrency=100 --iterations=1 --create-schema='test' --query="select * from test.t_100w where k2='FGCD'" engine=innodb --number-of-queries=200000 -uroot -p123 -verbose
```

### 锁的监控及处理

![8SEDRO](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/8SEDRO.jpg)

#### 查看锁状态

```mysql
# 看有没有锁等待
show status like 'innodb_row_lock%';

# 查看哪个事务在等待(被阻塞了)
use information_schema
select * from information_schema.INNODB_TRX where trx_state='LOCK WAIT';
trx_id : 事务ID号
trx_state : 当前事务的状态
trx_mysql_thread_id:连接层的,连接线程ID(show processlist; ===>Id或trx_id )
trx_query : 当前被阻塞的操作(一般是要丢给开发的)
```

#### 查看锁源

```mysql
select * from sys.innodb_lock_waits;  # 被锁的和锁定它的之间关系

locked_table    : 哪张表出现的等待
waiting_trx_id  : 等待的事务(与上个视图trx_id 对应)
waiting_pid     : 等待的线程号(与上个视图trx_mysql_thread_id)
blocking_trx_id : 锁源的事务ID
blocking_pid    : 锁源的线程号
```

#### 找到锁源`thread_id`

```mysql
select * from performance_schema.threads where processlist_id=15;
```

#### 找到锁源的 SQL 语句

```mysql
-- 当前在执行的语句
SELECT * FROM performance_schema.`events_statements_current` WHERE thread_id=41;
-- 执行语句的历史
SELECT * FROM performance_schema.`events_statements_history` WHERE thread_id=41;

得出结果,丢给开发
表信息
被阻塞的
锁源SQL
```

#### 死锁监控

```mysql
show engine innodb status\G
show variables like '%deadlock%';

vim /etc/my.cnf
innodb_print_all_deadlocks = 1
```

### 主从优化

```mysql
## 5.7 从库多线程MTS
基本要求:
5.7以上的版本(忘记小版本)
必须开启GTID
binlog必须是row模式

gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
slave-parallel-type=LOGICAL_CLOCK
slave-parallel-workers=16
master_info_repository=TABLE
relay_log_info_repository=TABLE
relay_log_recovery=ON

5.7 :
slave-parallel-type=LOGICAL_CLOCK
slave-parallel-workers=8
cpu核心数作为标准

CHANGE MASTER TO
  MASTER_HOST='10.0.0.128',
  MASTER_USER='repl',
  MASTER_PASSWORD='123',
  MASTER_PORT=3307,
  MASTER_AUTO_POSITION=1;
start slave;
```
