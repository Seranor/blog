---
title: MongoDB复制集
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-07.html
toc: true
---

### 复制集的作用

```bash
MongoDB 复制集的主要意义在于实现服务高可用，它的现实依赖于两个方面的功能
1.数据写入时将数据迅速复制到另一个独立节点上
2.在接受写入的节点发生故障时自动选举出一个新的替代节点

在实现高可用的同时，复制集实现了其他几个附加作用
1.数据分发：将数据从一个区域复制到另一个区域，减少另一个区域的读延迟
2.读写分离：不同类型的压力分别在不同的节点上执行
3.异地容灾：在数据中心故障时候快速切换到异地
```

<!-- more -->

### 典型复制集结构

```bash
一个典型的复制集由3个以上具有投票权的节点组成
1.一个主节点（PRIMARY）：接受写入操作和选举时投票
2.两个（或多个）从节点（SECONDARY）：复制主节点上的新数据和选举时投票
3.不推荐使用 Arbiter（投票节点）
```

![B6n7ov](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/B6n7ov.png)

### 数据是如何复制的

```bash
1.当一个修改操作，无论是插入、更新或删除，到达主节点时，它对数据的操作将被记录下来（经过一些必要的转换），这些记录称为 oplog (是一张表)
2.从节点通过在主节点上打开一个 tailable 游标不断获取新进入主节点的 oplog，并在自己的数据上回放，以此保持跟主节点的数据一致
```

![cTeuoL](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/cTeuoL.png)

### 通过选举完成故障恢复

```bash
1.具有投票权的节点之间两两互相发送心跳
2.当5次心跳未收到时判断为节点失联
3.如果失联的是主节点，从节点会发起选举，选出新的主节点
4.如果失联的是从节点则不会产生新的选举
5.选举基于 RAFT一致性算法 实现，选举成功的必要条件是大多数投票节点存活
6.复制集中最多可以有50个节点，但具有投票权的节点最多7个
```

### 影响选举的因素

```bash
整个集群必须有大多数节点存活，被选举为主节点的节点必须
  1.能够与多数节点建立连接
  2.具有较新的 oplog
  3.具有较高的优先级（如果有配置）
```

### 常见选项

```bash
复制集节点有以下常见的选配项
1.是否具有投票权（v 参数）：有则参与投票
2.优先级（priority 参数）：优先级越高的节点越优先成为主节点。优先级为0的节点无法成为主节点
3.隐藏（hidden 参数）：复制数据，但对应用不可见。隐藏节点可以具有投票仅，但优先级必须为0
4.延迟（slaveDelay 参数）：复制 n 秒之前的数据，保持与主节点的时间差
```

![3QYZEh](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/3QYZEh.png)

### 复制集注意事项

```bash
关于硬件:
1.因为正常的复制集节点都有可能成为主节点，它们的地位是一样的，因此硬件配置上必须一致
2.为了保证节点不会同时宕机，各节点使用的硬件必须具有独立性

关于软件:
1.复制集各节点软件版本必须一致，以避免出现不可预知的问题。

增加节点不会增加系统写性能
```

### 搭建复制集

#### 多实例配置启动

```bash
# 三个以上的mongodb节点（或多实例）

# 采用多实例的端口规划
28017、28018、28019、28020

# 创建相关目录
mkdir -p /mongodb/{28017..28020}/{conf,data,log}
tree -L 2 /mongodb

# 配置文件
cat > /mongodb/28017/conf/mongod.conf <<EOF
systemLog:
  destination: file
  path: /mongodb/28017/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/28017/data
  directoryPerDB: true
  #engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 1
      directoryForIndexes: true
    collectionConfig:
      blockCompressor: zlib
    indexConfig:
      prefixCompression: true
processManagement:
  fork: true
net:
  bindIpAll: true
  port: 28017
replication:
  oplogSizeMB: 2048
  replSetName: my_repl
EOF

# 配置文件说明
engine: wiredTiger   # 存储引擎，默认就是wiredTiger
cacheSizeGB: 1       # 类似缓冲区大小
oplogSizeMB: 2048    # 设置  oplog 大小
replSetName: my_repl # 设置 oplog 名字，要和配置复制内的名称一样

# 多配置文件复制
cp  /mongodb/28017/conf/mongod.conf  /mongodb/28018/conf/
cp  /mongodb/28017/conf/mongod.conf  /mongodb/28019/conf/
cp  /mongodb/28017/conf/mongod.conf  /mongodb/28020/conf/

sed -i 's#28017#28018#g' /mongodb/28018/conf/mongod.conf
sed -i 's#28017#28019#g' /mongodb/28019/conf/mongod.conf
sed -i 's#28017#28020#g' /mongodb/28020/conf/mongod.conf

chown -R mongod:mongod /mongodb

# 启动
su - mongod
mongod -f /mongodb/28017/conf/mongod.conf
mongod -f /mongodb/28018/conf/mongod.conf
mongod -f /mongodb/28019/conf/mongod.conf
mongod -f /mongodb/28020/conf/mongod.conf

# 检查
netstat -lntp |grep 280
```

#### 配置复制集

```bash
# 1主2从，从库普通从库 PSS
mongo --port 28017 admin

config = {_id: 'my_repl', members: [
                          {_id: 0, host: '192.168.0.11:28017'},
                          {_id: 1, host: '192.168.0.11:28018'},
                          {_id: 2, host: '192.168.0.11:28019'}]
          }
rs.initiate(config)
rs.status();  //查询复制集状态


# 1主1从1个arbiter PSA
mongo -port 28017 admin
config = {_id: 'my_repl', members: [
                          {_id: 0, host: '192.168.0.11:28017'},
                          {_id: 1, host: '192.168.0.11:28018'},
                          {_id: 2, host: '192.168.0.11:28019',"arbiterOnly":true}]
          }
rs.initiate(config)
```

### 复制集测试

```bash
# 主库插入数据
mongo --port 28017 admin
use test
db.movies.insert([{ "title": "Jaws","year":1975 ,"imdb_rating":8.1},
                  { "title": "Batman","year":1989,"imdb_rating":7.6}]);

db.movies.find().pretty()

# 从库查看
mongo --port 28018 admin
use test
db.movies.find().pretty()  //默认情况下从库不允许读写
rs.slaveOk()               //当前版本4.4还可以用，提示后续会弃用，use secondaryOk() instead
db.movies.find().pretty()  //此时就可以查看到数据了

# 关闭主库查看从库接管情况
mongod -f /mongodb/28017/conf/mongod.conf --shutdown
```

### 复制集管理

```bash
# 查看状态
rs.status();    //查看整体复制集状态
rs.isMaster();  // 查看当前是否是主节点
rs.conf()；     //查看复制集配置信息

# 添加和删除节点命令
rs.add("ip:port"); // 新增从节点
rs.remove("ip:port"); // 删除一个节点
rs.addArb("ip:port"); // 新增仲裁节点

# 连接到主节点(shell命令)
mongo --port 28017 admin

# 新增从节点
rs.add("192.168.0.11:28020")
rs.status();

# 删除一个节点
rs.remove("192.168.0.11:28020");
rs.status();

# 添加 arbiter节点
rs.addArb("192.168.0.11:28020")
rs.status();

从其他集群踢出的节点不能直接加入新的集群，强行加入也会被踢出，需要先清空这个节点的数据再加入
```

### 特殊从节点

```bash
# 参数说明
priority (0-1000): 优先级越高的节点越优先成为主节点，优先级为0的节点无法成为主节点
hidden 参数: 复制数据，但对应用不可见，隐藏节点可以具有投票仅，但优先级必须为0
slaveDelay 参数: 复制 n 秒之前的数据，保持与主节点的时间差

# 节点种类
arbiter节点: 主要负责选主过程中的投票，但是不存储任何数据，也不提供任何服务
hidden节点: 隐藏节点，不参与选主，也不对外提供服务
delay节点: 延时节点，数据落后于主库一段时间，因为数据是延时的，也不应该提供服务或参与选主，所以通常会配合hidden（隐藏）
一般情况下会将delay+hidden一起配置使用

# 查看副本集的配置信息
rs.conf()
{
	"_id" : "my_repl",
	"version" : 2,
	"protocolVersion" : NumberLong(1),
	"writeConcernMajorityJournalDefault" : true,
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.0.11:28017",  // 主机和端口
			"arbiterOnly" : false,  //是不是arbiter节点
			"buildIndexes" : true,
			"hidden" : false,  //隐藏节点
			"priority" : 1,  //权重
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1  //参与节点投票
		}
......


# 配置延时节点（一般延时节点也配置成hidden）
# MongoDB数据库内配置
rs.conf()  //查看当前配置
cfg=rs.conf()
//members[num]中，num是从0开始依次往下数的节点数字，不是根据 _id 数字，因为后续添加删除之后这个数字容易混乱
cfg.members[3].priority=0  //不能成为主节点
cfg.members[3].hidden=true  //隐藏节点
cfg.members[3].slaveDelay=120  //延时2小时同步
cfg.members[3].votes=0  //不参与投票
rs.reconfig(cfg)  //生效
rs.conf()  //再次查看

# 取消以上配置,恢复
cfg=rs.conf()
cfg.members[3].priority=1
cfg.members[3].hidden=false
cfg.members[3].slaveDelay=0
cfg.members[3].votes=1
rs.reconfig(cfg)
rs.conf();
```

### 其他命令操作

```bash
# 查看副本集各成员的状态
rs.status()

# 副本集角色切换（不要人为随便操作）
rs.stepDown()

# 锁定从，使其不会转变成主库
rs.freeze(300)
//freeze()和stepDown单位都是秒

# 设置副本节点可读：在副本节点执行
rs.slaveOk()

# 查看副本节点（监控主从延时）
rs.printSlaveReplicationInfo()
```
