---
title: MongoDB两地三中心部署
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-09.html
toc: true
---

### 容灾级别

<!-- more -->

| 级别 | 方式                                                                                                                                                             | RPO     | RTO            |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | -------------- |
| L0   | 无备源中心 : 没有灾难恢复能力，只在本地进行数据备份                                                                                                              | 24 小时 | 4 小时         |
| L1   | 本地备份+异地保存: 本地将关键数据备份，然后送到异地保存。 灾难发生后，按预定数据恢复程序恢复系统和数据                                                           | 24 小时 | 8 小时         |
| L2   | 双中心主备模式: 在异地建立一个热备份点，通过网络进行数据备份。 当出现灾难时，备份站点接替主站点的业务，维护业务连续性                                            | 秒      | 数分钟到半小时 |
| L3   | 双中心双活: 在相隔较远的地方分别建立两个数据中心，进行相互数据备份。 当某个数据中心发生灾难时，另一个数据中心接替其工作任务                                      | 秒      | 秒             |
| L4   | 双中心双活 + 异地热备 = 两地三中心: 在同城分别建立两个数据中心，进行相互数据备份。 当该城市的 2 个中心同时不可用（地震/大面积停电/网络等），快速切换到异地 L4 秒 | 秒      | 分钟           |

### 简单架构

![image-20220104155254690](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220104155254690.png)

### 规划

| 配置 | IP                | 角色                               |
| ---- | ----------------- | ---------------------------------- |
| 2c4g | 192.168.0.11(db1) | Primary(10011)、Secondary(10002)   |
| 2c4g | 192.168.0.12(db2) | Secondary(10003)、Secondary(10004) |
| 2c4g | 192.168.0.13(db3) | Secondary(10005)                   |

### 部署

#### db1 配置

```bash
mkdir -p /mongodb/{10001..10002}/{conf,data,log}
tree -L 2 /mongodb

# 配置文件
cat > /mongodb/10001/conf/mongodb.conf <<EOF
systemLog:
  destination: file
  path: /mongodb/10001/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/10001/data
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
  port: 10001
replication:
  oplogSizeMB: 2048
  replSetName: my_repl
EOF

cp /mongodb/10001/conf/mongodb.conf  /mongodb/10002/conf
sed -i 's#10001#10002#g' /mongodb/10002/conf/mongodb.conf

chown -R mongod:mongod /mongodb
su - mongod
mongod -f /mongodb/10001/conf/mongodb.conf
mongod -f /mongodb/10002/conf/mongodb.conf
```

#### db2 配置

```bash
mkdir -p /mongodb/{10003..10004}/{conf,data,log}
tree -L 2 /mongodb

# 配置文件
cat > /mongodb/10003/conf/mongodb.conf <<EOF
systemLog:
  destination: file
  path: /mongodb/10003/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/10003/data
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
  port: 10003
replication:
  oplogSizeMB: 2048
  replSetName: my_repl
EOF

cp /mongodb/10003/conf/mongodb.conf  /mongodb/10004/conf
sed -i 's#10003#10004#g' /mongodb/10004/conf/mongodb.conf

chown -R mongod:mongod /mongodb
su - mongod
mongod -f /mongodb/10003/conf/mongodb.conf
mongod -f /mongodb/10004/conf/mongodb.conf
```

#### db3 配置

```bash
mkdir -p /mongodb/10005/{conf,data,log}
tree -L 2 /mongodb

# 配置文件
cat > /mongodb/10005/conf/mongodb.conf <<EOF
systemLog:
  destination: file
  path: /mongodb/10005/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/10005/data
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
  port: 10005
replication:
  oplogSizeMB: 2048
  replSetName: my_repl
EOF

chown -R mongod:mongod /mongodb
su - mongod
mongod -f /mongodb/10005/conf/mongodb.conf
```

#### 配置复制集和权重

```bash
mongo --port 10001 admin
config = {_id: 'my_repl', members: [
                          {_id: 0, host: '192.168.0.11:10001'},
                          {_id: 1, host: '192.168.0.11:10002'},
                          {_id: 2, host: '192.168.0.12:10003'},
                          {_id: 3, host: '192.168.0.12:10004'},
                          {_id: 4, host: '192.168.0.13:10005'}]
          }
rs.initiate(config)
rs.status()

# 权重配置
cfg = rs.conf()
cfg.members[0].priority = 5
cfg.members[1].priority = 10
rs.reconfig(cfg)
```

#### 复制集安全加固

```bash
openssl rand -base64 756 > /mongodb/10001/conf/keyfile
chmod 600 /mongodb/10001/conf/keyfile
cp -a /mongodb/10001/conf/keyfile /mongodb/10002/conf
scp -rp /mongodb/10001/conf/keyfile db2:/mongodb/10003/conf
scp -rp /mongodb/10001/conf/keyfile db2:/mongodb/10004/conf
scp -rp /mongodb/10001/conf/keyfile db3:/mongodb/10005/conf


cat >> /mongodb/10001/conf/mongodb.conf <<EOF
security:
  keyFile: /mongodb/10001/conf/keyfile
EOF

cat >> /mongodb/10002/conf/mongodb.conf <<EOF
security:
  keyFile: /mongodb/10002/conf/keyfile
EOF

cat >> /mongodb/10003/conf/mongodb.conf <<EOF
security:
  keyFile: /mongodb/10003/conf/keyfile
EOF

cat >> /mongodb/10004/conf/mongodb.conf <<EOF
security:
  keyFile: /mongodb/10004/conf/keyfile
EOF

cat >> /mongodb/10005/conf/mongodb.conf <<EOF
security:
  keyFile: /mongodb/10005/conf/keyfile
EOF

# 关闭的时候最后关闭主节点
su - mongod
mongod -f /mongodb/10001/conf/mongodb.conf --shutdown
mongod -f /mongodb/10002/conf/mongodb.conf --shutdown
mongod -f /mongodb/10003/conf/mongodb.conf --shutdown
mongod -f /mongodb/10004/conf/mongodb.conf --shutdown
mongod -f /mongodb/10005/conf/mongodb.conf --shutdown

# 启动
mongod -f /mongodb/10001/conf/mongodb.conf
mongod -f /mongodb/10002/conf/mongodb.conf
mongod -f /mongodb/10003/conf/mongodb.conf
mongod -f /mongodb/10004/conf/mongodb.conf
mongod -f /mongodb/10005/conf/mongodb.conf

# 在主节点上添加用户密码
mongo --port 10001 admin
use admin
db.createUser(
{
    user: "root",
    pwd: "root123",
    roles: [ { role: "root", db: "admin" } ]
}
)

# 交互式输入密码
use admin
db.createUser(
{
    user: "root1",
    pwd: passwordPrompt(),
    roles: [ { role: "root", db: "admin" } ]
}
)

# 交互式验证
use admin
db.auth("root1",passwordPrompt())
```
