---
title: MongoDB安装部署
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-02.html
toc: true
---

### 获取安装包

<!-- more -->

```bash
# 下载地址
https://www.mongodb.com/try/download/community

# centos7下载
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.17.tgz
```

### 安装 MongoDB

#### 关闭 THP

```bash
# 关闭THP
# root用户下
# 在 /etc/rc.local 最后添加如下代码
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

#其他系统关闭参照官方文档
https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/

# 为什么要关闭
Transparent Huge Pages (THP) is a Linux memory management system
that reduces the overhead of Translation Lookaside Buffer (TLB)
lookups on machines with large amounts of memory by using larger memory pages.
However, database workloads often perform poorly with THP,
because they tend to have sparse rather than contiguous memory access patterns.
You should disable THP on Linux machines to ensure best performance with MongoDB.
```

#### 环境准备

```bash
tar xf mongodb-linux-x86_64-rhel70-4.2.17.tgz
mv mongodb-linux-x86_64-rhel70-4.2.17 /usr/local/mongodb

# 创建用户
useradd mongod
passwd mongod

# 创建相关目录
mkdir -p /mongodb/{data,conf,log}
chown -R mongod:mongod /mongodb
chown -R mongod:mongod /usr/local/mongodb

# 环境变量配置
echo 'export PATH=/usr/local/mongodb/bin:$PATH' >> /etc/profile
source /etc/profile
```

#### 初始化数据库

```bash
# 切换用户
su - mongod

# 初始化mongodb
mongod --dbpath=/mongodb/data --logpath=/mongodb/log/mongodb.log --port=27017 --logappend --fork

# 登录mongodb数据库
mongo
```

### 普通配置文件

```bash
# 普通配置文件
cat > /mongodb/conf/mongodb.conf <<EOF
logpath=/mongodb/log/mongodb.log
dbpath=/mongodb/data
port=27017
logappend=true
fork=true
EOF

# 关闭mongodb
mongod -f /mongodb/conf/mongodb.conf --shutdown

# 启动mongodb
mongod -f /mongodb/conf/mongodb.conf
```

### YAML 格式配置文件

```bash
# 配置文件解释
--系统日志有关
systemLog:
   destination: file
   path: "/mongodb/log/mongodb.log"    --日志位置
   logAppend: true                     --日志以追加模式记录

--数据存储有关
storage:
   journal:
      enabled: true
   dbPath: "/mongodb/data"            --数据路径的位置

-- 进程控制
processManagement:
   fork: true                         --后台守护进程
   pidFilePath: <string>              --pid文件的位置，一般不用配置，可以去掉这行，自动生成到data中

--网络配置有关
net:
   bindIp: <ip>                       -- 监听地址
   bindIpAll: true                    -- 开启所有端口
   port: <port>                       -- 端口号,默认不配置端口号，是27017

-- 安全验证有关配置
security:
  authorization: enabled              --是否打开用户名密码验证，授权完成之后再开启

------------------以下是复制集与分片集群有关----------------------

replication:
 oplogSizeMB: <NUM>
 replSetName: "<REPSETNAME>"
 secondaryIndexPrefetch: "all"

sharding:
   clusterRole: <string>
   archiveMovedChunks: <boolean>

---for mongos only
replication:
   localPingThresholdMs: <int>

sharding:
   configDB: <string>
---
------------------------------------


# YAML例子
cat >  /mongodb/conf/mongodb.conf <<EOF
systemLog:
  destination: file
  path: "/mongodb/log/mongodb.log"
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: "/mongodb/data/"
processManagement:
  fork: true
net:
  port: 27017
  bindIpAll: true
EOF

# 关闭和启动
mongod -f /mongodb/conf/mongodb.conf --shutdown
mongod -f /mongodb/conf/mongodb.conf
```

### 优化启动警告

```bash
** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
建议使用xfs文件系统，但不影响使用

** WARNING: Access control is not enabled for the database.
配置密码之后开启安全

** WARNING: This server is bound to localhost.
配置文件中绑定IP就可以解决

** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
这两条参考上面关闭THP即可


** WARNING: soft rlimits too low. rlimits set to 31192 processes, 100001 files. Number of processes....
vim /etc/security/limits.conf
# end of file
mongod soft nofile 64000
mongod hard nofile 64000
mongod soft nproc 32000
mongod hard nproc 32000
```
