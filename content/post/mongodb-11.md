---
title: MongoDB备份与恢复迁移
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-11.html
toc: true
---

### 备份工具

<!-- more -->

```bash
1.mongoexport/mongoimport
2.mongodump/mongorestore

# 区别
mongoexport/mongoimport  导入/导出的是JSON格式或者CSV格式
mongodump/mongorestore   导入/导出的是BSON格式

# 应用场景
mongoexport/mongoimport:json csv
1.异构平台迁移      mysql  <---> mongodb
2.同平台,跨大版本   mongodb2  ----> mongodb3

mongodump/mongorestore
日常备份恢复时使用
```

### `mongoexport`导出

```bash
# 参数说明
-h   指明数据库宿主机的IP
-u   指明数据库的用户名
-p   指明数据库的密码
-d   指明数据库的名字
-c   指明collection的名字
-f   指明要导出那些列
-o   指明到要导出的文件名
-q   指明导出数据的过滤条件
--authenticationDatabase admin

# 数据准备
use test
for(i=0;i<10000;i++){ db.log.insert({"uid":i,"name":"mongodb","age":6,"date":new Date()});}

# 单表备份至json格式
mongoexport -uroot -proot123 --port 27017 --authenticationDatabase admin -d test -c log -o /mongodb/log.json
## 备份文件的名字可以自定义，默认导出了JSON格式的数据

#  单表备份至csv格式
## 如果我们需要导出CSV格式的数据，则需要使用----type=csv参数
mongoexport -uroot -proot123 --port 27017 --authenticationDatabase admin -d test -c log --type=csv -f uid,name,age,date  -o /mongodb/log.csv
```

### `mongoimport` 导入

```bash
# 参数说明
-h  指明数据库宿主机的IP
-u  指明数据库的用户名
-p  指明数据库的密码
-d  指明数据库的名字
-c  指明collection的名字
-f  指明要导入那些列
-j  并发数

//并行

# 数据恢复
# 恢复json格式表数据到log1 ( --drop 就可以删除之前的表导入或者换个表，如导入到这边的 log1 )
mongoimport -uroot -proot123 --port 27017 --authenticationDatabase admin -d test -c log1 /mongodb/log.json

# 恢复csv格式的文件到log2
上面演示的是导入JSON格式的文件中的内容，如果要导入CSV格式文件中的内容，则需要通过--type参数指定导入格式，具体如下所示：
错误的恢复

# csv格式的文件头行，有列名字
mongoimport   -uroot -proot123 --port 27017 --authenticationDatabase admin -d test -c log2 --type=csv --headerline --file  /mongodb/log.csv
# --headerline 指明第一行是列名，不需要导入

# csv格式的文件头行，没有列名字
mongoimport   -uroot -proot123 --port 27017 --authenticationDatabase admin -d test -c log3 -j 4 --type=csv -f id,name,age,date --file  /mongodb/log.csv
```

### 异构平台迁移

#### `MySQL`到`MongoDB`

```bash
mysql ---> mongodb

# 准备mysql数据库，略
# 添加配置文件，导出文件存放位置
vim /etc/my.cnf
...
secure-file-priv=/data/bak
...

chown -R mysql.mysql /data/

# 导入一个world库，下载地址: https://dev.mysql.com/doc/index-other.html
mysql -uroot -p123 < world.sql

# sql导出city表为csv格式
use world;
select * from city into outfile '/data/bak/city.csv' fields terminated by ',' enclosed by '"';

# 导出的格式缺少mongodb的头信息，可以用desc 查看补齐
desc world.city;

# 或者用下列语句查看
select table_name,group_concat(column_name) from information_schema.columns where table_schema='world' group by table_name;
# 得到 Name,CountryCode,District,Population,ID

# mongodb 导入
mongoimport -uroot -proot123 --port 27017 --authenticationDatabase admin -d world -c city --type=csv -f Name,CountryCode,District,Population,ID --file /data/bak/city.csv

# 检查
mongo -uroot -proot123 --port 27017 admin
use world
show collections  // show tables
db.city.count()
db.city.find().pretty()


```

#### 大量数据批量迁移

```bash
# 模拟大量数据
wget https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/sql/100w.sql.zip
unzip 100w.sql.zip

# 进入到mysql
use world;
source /root/100w.sql;
create table t1 like t100w;
insert into t1 select * from t100w;
create table t2 like t100w;
insert into t2 select * from t100w;

# 将数据批量导出，world库的数据
mysqldump --fields-terminated-by ',' --fields-enclosed-by '"' world -T /data/bak/
rm -f /data/bak/*.sql
find ./ -name "*.txt"|awk -F "." '{print $2}' |xargs -i -t mv ./{}.txt ./{}.csv

# 使用 mysql 的SQL语句，拼接批量导入mongodb的语句
select concat("mongoimport -uroot -proot123 --port 27017 --authenticationDatabase admin -d ",table_schema," -c ",table_name," --type=csv -f ",group_concat(column_name)," --file /data/bak/", table_name,".csv") from information_schema.columns where table_schema='world' group by table_name into outfile '/data/bak/import-mongodb.sh';

# 查看批量导入语句
cat /data/bak/import-mongodb.sh
chmod +x /data/bak/import-mongodb.sh

# 删除之前的数据
su - mongod
 mongo -uroot -proot123 --port 27017 admin
 use world
 db.dropDatabase();

# 使用脚本批量导入数据
su - mongod
bash /data/bak/import-mongodb.sh

# 检查数据
```

### `mongodump`备份

#### 介绍

```bash
mongodump能够在Mongodb运行时进行备份，它的工作原理是对运行的Mongodb做查询，然后将所有查到的文档写入磁盘,
但是存在的问题时使用mongodump产生的备份不一定是数据库的实时快照，如果我们在备份时对数据库进行了写入操作,
则备份出来的文件可能不完全和Mongodb实时数据相等。另外在备份时可能会对其它客户端性能产生不利的影响,
```

#### 用法

```bash
# 参数说明
-h  指明数据库宿主机的IP
-u  指明数据库的用户名
-p  指明数据库的密码
-d  指明数据库的名字
-c  指明collection的名字
-o  指明到要导出的文件名
-q  指明导出数据的过滤条件
-j  指定并发数
--oplog  备份的同时备份oplog
```

#### 基本使用

```bash
# 全库备份
mkdir /mongodb/backup
mongodump -uroot -proot123 --port 27017 --authenticationDatabase admin -o /mongodb/backup

# 备份world库
mongodump -uroot -proot123 --port 27017 --authenticationDatabase admin -d world -o /mongodb/backup/

# 备份world库下的city集合
mongodump -uroot -proot123 --port 27017 --authenticationDatabase admin -d world -c city -o /mongodb/backup/

# 压缩备份
mongodump   -uroot -proot123 --port 27017 --authenticationDatabase admin -o /mongodb/backup/ --gzip
mongodump   -uroot -proot123 --port 27017 --authenticationDatabase admin -d world -o /mongodb/backup/ --gzip
mongodump   -uroot -proot123 --port 27017 --authenticationDatabase admin -d world -c city -o /mongodb/backup/ --gzip
```

### `mongorestore`恢复

```bash
# 全库恢复
mongorestore   -uroot -proot123 --port 27017 --authenticationDatabase admin /mongodb/backup/ --gzip

# 恢复world库
mongorestore   -uroot -proot123 --port 27017 --authenticationDatabase admin -d world1  /mongodb/backup/world

# 恢复world库下的t1集合
mongorestore   -uroot -proot123 --port 27017 --authenticationDatabase admin -d world -c t1  --gzip /mongodb/backup.bak/oldboy/log1.bson.gz

# --drop 表示恢复的时候把之前的集合删除再导入
mongorestore  -uroot -proot123 --port 27017 --authenticationDatabase admin -d world --drop  /mongodb/backup/world
```

### `oplog`

#### 介绍

```bash
这是replica set或者master/slave模式专用
在replica set中oplog是一个定容集合(capped collection),它的默认大小是磁盘空间的5%(可以通过--oplogSizeMB参数修改),位于local库的db.oplog.rs
其中记录的是整个mongod实例一段时间内数据库的所有变更（插入/更新/删除）操作,当空间用完时新记录自动覆盖最老的记录
其覆盖范围被称作oplog时间窗口。需要注意的是，因为oplog是一个定容集合，所以时间窗口能覆盖的范围会因为你单位时间内的更新次数不同而变化

想要查看当前的oplog时间窗口预计值，可以使用以下命令:
 use local
 db.oplog.rs.find().pretty()

rs.printReplicationInfo()
configured oplog size:   1561.5615234375MB      //<--集合大小
log length start to end: 423849secs (117.74hrs) //<--预计窗口覆盖时间
```

#### 应用

```bash
# 实现热备，在备份时使用 --oplog 选项
# 搭建复制集，略

# 测试数据
mongo --port 28018
use test
for(var i = 1 ;i < 100; i++) {
    db.foo.insert({a:i});
}

db.oplog.rs.find({"op":"i"}).pretty()

# oplog 配合mongodump实现热备
mongodump --port 28018 --oplog -o /mongodb/backup
# 作用介绍：--oplog 会记录备份过程中的数据变化。会以oplog.bson保存下来

# 恢复
mongorestore  --port 28018 --oplogReplay /mongodb/backup
```

#### 使用案例

```bash
背景：每天0点全备，oplog恢复窗口为48小时。某天，上午10点world.city 业务表被误删除
恢复思路：
    0、停应用
    2、找测试库
    3、恢复昨天晚上全备
    4、截取全备之后到world.city误删除时间点的oplog，并恢复到测试库
    5、将误删除表导出，恢复到生产库

# 恢复步骤如下
# 全备数据库
    # 模拟原始数据
    mongo --port 28017
    use wo
    for(var i = 1 ;i < 20; i++) {
        db.ci.insert({a: i});
    }

    # 全备
    rm -rf /mongodb/backup/*
    mongodump --port 28018 --oplog -o /mongodb/backup
   #  --oplog功能:在备份同时,将备份过程中产生的日志进行备份，文件必须存放在/mongodb/backup下,自动命令为oplog.bson

    # 再次模拟数据
    db.ci1.insert({id:1})
    db.ci2.insert({id:2})

# 上午10点：删除wo库下的ci表，10:00时刻,误删除
db.ci.drop()
show tables;

# 备份现有的oplog.rs表
mongodump --port 28018 -d local -c oplog.rs  -o /mongodb/backup

# 截取oplog并恢复到drop之前的位置，更合理的方法：登陆到原数据库
mongo --port 28018
use local
db.oplog.rs.find({op:"c"}).pretty();

{
    "ts" : Timestamp(1553659908, 1),
    "t" : NumberLong(2),
    "h" : NumberLong("-7439981700218302504"),
    "v" : 2,
    "op" : "c",
    "ns" : "wo.$cmd",
    "ui" : UUID("db70fa45-edde-4945-ade3-747224745725"),
    "wall" : ISODate("2019-03-27T04:11:48.890Z"),
    "o" : {
        "drop" : "ci"
    }
}

# 获取到oplog误删除时间点位置
"ts" : Timestamp(1553659908, 1)

# 恢复备份+应用oplog
cd /mongodb/backup/local/
ls
oplog.rs.bson  oplog.rs.metadata.json

cp oplog.rs.bson ../oplog.bson
rm -rf /mongodb/backup/local/

mongorestore --port 38021  --oplogReplay --oplogLimit "1553659908:1"  --drop   /mongodb/backup/
```
