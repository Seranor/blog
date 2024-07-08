---
title: MongoDB全球多写
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-10.html
toc: true
---

### 全球多写(`Zone Sharding`)

<!-- more -->

![image-20220104181800463](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220104181800463.png)

### `db1`复制集配置

```bash
mkdir -p /mongodb/{20001..20003}/{conf,data,log}

# 配置文件
cat >  /mongodb/20001/conf/mongodb.conf  <<EOF
systemLog:
  destination: file
  path: /mongodb/20001/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/20001/data
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
net:
  bindIpAll: true
  port: 20001
replication:
  oplogSizeMB: 2048
  replSetName: CN_sh
sharding:
  clusterRole: shardsvr
processManagement:
  fork: true
EOF


cp /mongodb/20001/conf/mongodb.conf  /mongodb/20002/conf
cp /mongodb/20001/conf/mongodb.conf  /mongodb/20003/conf
sed -i 's#20001#20002#g' /mongodb/20002/conf/mongodb.conf
sed -i 's#20001#20003#g' /mongodb/20003/conf/mongodb.conf
sed -i 's#CN_sh#US_sh#g' /mongodb/20003/conf/mongodb.conf

chown -R mongod:mongod /mongodb
su - mongod
mongod -f /mongodb/20001/conf/mongodb.conf
mongod -f /mongodb/20002/conf/mongodb.conf
mongod -f /mongodb/20003/conf/mongodb.conf
```

### `db2`复制集配置

```bash
mkdir -p /mongodb/{20001..20003}/{conf,data,log}

# 配置文件
cat >  /mongodb/20001/conf/mongodb.conf  <<EOF
systemLog:
  destination: file
  path: /mongodb/20001/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/20001/data
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
net:
  bindIpAll: true
  port: 20001
replication:
  oplogSizeMB: 2048
  replSetName: US_sh
sharding:
  clusterRole: shardsvr
processManagement:
  fork: true
EOF


cp /mongodb/20001/conf/mongodb.conf  /mongodb/20002/conf
cp /mongodb/20001/conf/mongodb.conf  /mongodb/20003/conf
sed -i 's#20001#20002#g' /mongodb/20002/conf/mongodb.conf
sed -i 's#20001#20003#g' /mongodb/20003/conf/mongodb.conf
sed -i 's#US_sh#CN_sh#g' /mongodb/20003/conf/mongodb.conf

chown -R mongod:mongod /mongodb
su - mongod
mongod -f /mongodb/20001/conf/mongodb.conf
mongod -f /mongodb/20002/conf/mongodb.conf
mongod -f /mongodb/20003/conf/mongodb.conf
```

### `db1`上`config`节点

```bash
mkdir -p /mongodb/{20004..20006}/{conf,data,log}

cat > /mongodb/20004/conf/mongodb.conf  <<EOF
systemLog:
  destination: file
  path: /mongodb/20004/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/20004/data/
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
net:
  bindIpAll: true
  port: 20004
replication:
  oplogSizeMB: 2048
  replSetName: configReplSet
sharding:
  clusterRole: configsvr
processManagement:
  fork: true
EOF

cp /mongodb/20004/conf/mongodb.conf  /mongodb/20005/conf
cp /mongodb/20004/conf/mongodb.conf  /mongodb/20006/conf
sed -i 's#20004#20005#g' /mongodb/20005/conf/mongodb.conf
sed -i 's#20004#20006#g' /mongodb/20006/conf/mongodb.conf

chown -R mongod:mongod /mongodb
su - mongod
mongod -f /mongodb/20004/conf/mongodb.conf
mongod -f /mongodb/20005/conf/mongodb.conf
mongod -f /mongodb/20006/conf/mongodb.conf
```

### 复制集配置

```bash
# 设置复制集1
mongo 192.168.0.11:20001/admin
use  admin
config = {_id: 'CN_sh', members: [
                        {_id: 0, host: '192.168.0.11:20001'},
                        {_id: 1, host: '192.168.0.11:20002'},
                        {_id: 2, host: '192.168.0.12:20003'}]
         }
rs.initiate(config)

# 设置复制集2
mongo 192.168.0.12:20001/admin
use  admin
config = {_id: 'US_sh', members: [
                        {_id: 0, host: '192.168.0.12:20001'},
                        {_id: 1, host: '192.168.0.12:20002'},
                        {_id: 2, host: '192.168.0.11:20003'}]
         }
rs.initiate(config)

# 设置config节点，db1节点
mongo --port 20004 admin
use  admin
config = {_id: 'configReplSet', members: [
                        {_id: 0, host: '192.168.0.11:20004'},
                        {_id: 1, host: '192.168.0.11:20005'},
                        {_id: 2, host: '192.168.0.11:20006'}]
         }
rs.initiate(config)
```

### `mongos`配置

```bash
# db1上操作
mkdir -p /mongodb/20010/{conf,log}

cat > /mongodb/20010/conf/mongos.conf <<EOF
systemLog:
  destination: file
  path: /mongodb/20010/log/mongos.log
  logAppend: true
net:
  bindIpAll: true
  port: 20010
sharding:
  configDB: configReplSet/192.168.0.11:20004,192.168.0.11:20005,192.168.0.11:20006
processManagement:
  fork: true
EOF

mongos -f /mongodb/20010/conf/mongos.conf


# db2上操作
mkdir -p /mongodb/20011/{conf,log}

cat > /mongodb/20011/conf/mongos.conf <<EOF
systemLog:
  destination: file
  path: /mongodb/20011/log/mongos.log
  logAppend: true
net:
  bindIpAll: true
  port: 20011
sharding:
  configDB: configReplSet/192.168.0.11:20004,192.168.0.11:20005,192.168.0.11:20006
processManagement:
  fork: true
EOF


mongos -f /mongodb/20011/conf/mongos.conf


# 在db1上操作(随便一台都行)
# 连接到mongs的admin数据库
su - mongod
mongo 192.168.0.11:20010/admin

# 添加分片
db.runCommand( { addshard : "CN_sh/192.168.0.11:20001,192.168.0.11:20002,192.168.0.12:20003",name:"CN_sh"} )
db.runCommand( { addshard : "US_sh/192.168.0.12:20001,192.168.0.12:20002,192.168.0.11:20003",name:"US_sh"} )

# 列出分片
db.runCommand( { listshards : 1 } )

# 整体状态查看
sh.status()
```

### 使用

```bash
# 登录 mongos 操作
mongo 192.168.0.11:20010/admin

# 对shard添加标记
use config
sh.addShardTag("CN_sh","ASIA")
sh.addShardTag("US_sh","AMERICA")

# 开启crm库的功能
use admin
db.runCommand( { enablesharding : "crm" } )

# 开启路由规则
use config
sh.addTagRange( "crm.orders",
{ "locationCode" : "CN", "order_id" : MinKey },
{ "locationCode" : "CN", "order_id" : MaxKey },
"ASIA")

sh.addTagRange( "crm.orders",
{ "locationCode" : "US", "order_id" : MinKey },
{ "locationCode" : "US", "order_id" : MaxKey },
"AMERICA")

sh.addTagRange( "crm.orders",
{ "locationCode" : "CA", "order_id" : MinKey },
{ "locationCode" : "CA", "order_id" : MaxKey },
"AMERICA")

sh.status()
```
