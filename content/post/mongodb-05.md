---
title: MongoDB基本的CRUD
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-05.html
toc: true
---

### 通用方法和获取帮助

<!-- more -->

```bash
# 获取帮助
help
help
db.help()
db.t1.help()

# 常用操作
//查看当前db版本
db.version()

//显示当前数据库
db
db.getName()

// 查询所有数据库
show dbs

//切换数据库
use local

// 显示当前数据库状态
  //查看local数据
  use local
  db.stats()

//查看当前数据库的连接机器地址
db.getMongo()

// 指定数据库进行连接：（默认连接本机test数据库）
# mongo 192.168.0.11/admin
[mongod@mongodb ~]$ mongo 192.168.0.11/admin

# 库和表的操作
// 建库
use test

// 删除
db.dropDatabase()
{ "dropped" : "test", "ok" : 1 }

// 创建集合(表)
//方法1
use app
db.createCollection('a')
db.createCollection('b')

//方法2：当插入一个文档的时候，一个集合就会自动创建
use app
db.c.insert({username:"mongodb"})

//查看当前全部集合
show collections

//查看指定集合
db.c.find()
{ "_id" : ObjectId("5743c9a9bf72d9f7b524713d"), "username" : "mongodb" }

// 删除集合
use app
db.a.drop()

// 重命名集合
app> db.log.renameCollection("log1")
```

### 使用`insert`完成插入操作

```bash
# 操作格式
db.<集合>.insertOne(<JSON对象>)
db.<集合>.insertMany([<JSON 1>, <JSON 2>, …<JSON n>])

# 示例
db.fruit.insertOne({name: "apple"})
db.fruit.insertMany([
{name: "apple"},
{name: "pear"},
{name: "orange"}
])

# 批量插入数据
for(i=0;i<1000;i++){ db.test.insert({"uid":i,"name":"mongodb","age":6,"date":new Date()});}

# 时区问题，中国当前时间需要解决
```

### 使用`find`查询文档

```bash
# 关于 find:
find 是 MongoDB 中查询数据的基本指令，相当于 SQL 中的 SELECT
find 返回的是游标

# 示例
db.movies.find( { "year" : 1975 } ) //单条件查询
db.movies.find( { "year" : 1989, "title" : "Batman" } ) //多条件and查询
db.movies.find( { $and : [ {"title" : "Batman"}, { "category" : "action" }] } ) // and的另一种形式
db.movies.find( { $or: [{"year" : 1989}, {"title" : "Batman"}] } ) //多条件or查询
db.movies.find( { "title" : /^B/} ) //按正则表达式查找
```

### 查询条件对照表

| SQL    | MSQL         |
| ------ | ------------ |
| a = 1  | {a:1}        |
| a <> 1 | {a:{$ne:1}}  |
| a > 1  | {a:{$gt:1}}  |
| a >= 1 | {a:{$gte:1}} |
| a < 1  | {a:{$lt:1}}  |
| a <= 1 | {a:{$lte:1}} |

### 查询逻辑对照表

| SQL            | MSQL                              |
| -------------- | --------------------------------- |
| a = 1 AND b= 1 | {a:1,b:1}或者{$and:[{a:1},{b:1}]} |
| a = 1 OR b = 1 | {$or:[{$a:1},{$b:1}]}             |
| a IS NULL      | {a:{$exists:false}}               |
| a IN (1,2,3)   | {a:{$in:[1,2,3]}}                 |

### 逻辑查询运算符

```bash
$lt: 存在并小于
$lte: 存在并小于等于
$gt: 存在并大于
$gte: 存在并大于等于
$ne: 不存在或存在但不等于
$in: 存在并在指定数组中
$nin: 不存在或不在指定数组中
$or: 匹配两个或多个条件中的一个
$and: 匹配全部条件
```

### 使用`find`搜索子文档

```bash
# find 支持使用“field.sub_field”的形式查询子文档。假设有一个文档：
db.fruit.insertOne({
name: "apple",
from: {
country: "China",
province: "Guangdong" }
})

# 正确写法
db.fruit.find( { "from.country" : "China" } )
```

### 使用 find 搜索数组

```bash
# find 支持对数组中的元素进行搜索。假设有一个文档：
db.fruit.insert([
{ "name" : "Apple", color: ["red", "green" ] },
{ "name" : "Mango", color: ["yellow", "green"] }
])

# 查看单个条件
db.fruit.find({color: "red"})

# 查询多个条件
db.fruit.find({$or: [{color: "red"}, {color: "yellow"}]} )
```

### 使用 find 搜索数组中的对象

```bash
考虑以下文档，在其中搜索
db.movies.insertOne( {
"title" : "Raiders of the Lost Ark",
"filming_locations" : [
{ "city" : "Los Angeles", "state" : "CA", "country" : "USA" },
{ "city" : "Rome", "state" : "Lazio", "country" : "Italy" },
{ "city" : "Florence", "state" : "SC", "country" : "USA" }
]
})
// 查找城市是 Rome 的记录
db.movies.find({"filming_locations.city": "Rome"})
```

### 使用 find 搜索数组中的对象

```bash
# 在数组中搜索子对象的多个字段时，如果使用 $elemMatch，它表示必须是同一个子对象满足多个条件，考虑以下两个查询
db.getCollection('movies').find({
"filming_locations.city": "Rome",
"filming_locations.country": "USA"
})

db.movies.insertOne( {
"title" : "11111",
"filming_locations" : [
{ "city" : "bj", "state" : "CA", "country" : "CHN" },
{ "city" : "Rome", "state" : "Lazio", "country" : "Italy" },
{ "city" : "tlp", "state" : "SC", "country" : "USA" }
]
})


db.getCollection('movies').find({
"filming_locations": {
$elemMatch:{"city":"bj", "country": "CHN"}
}
})
```

### 控制 find 返回的字段

```bash
# find 可以指定只返回指定的字段
_id字段必须明确指明不返回，否则默认返回
在 MongoDB 中我们称这为投影(projection)

db.movies.find({"category": "action"},{"_id":0, title:1})
```

### 使用 remove 删除文档

```bash
remove 命令需要配合查询条件使用
匹配查询条件的的文档会被删除
指定一个空文档条件会删除所有文档

# 以下示例
db.testcol.remove( { a : 1 } ) // 删除a 等于1的记录
db.testcol.remove( { a : { $lt : 5 } } ) // 删除a 小于5的记录
db.testcol.remove( { } ) // 删除所有记录
```

### 使用 update 更新文档

```bash
Update 操作执行格式: db.<集合>.update(<查询条件>, <更新字段>)

# 以以下数据为例
# 插入数据
db.fruit.insertMany([
{name: "apple"},
{name: "pear"},
{name: "orange"}
])

# 更新数据
db.fruit.updateOne({name: "apple"}, {$set: {from: "China"}}) //有就更新，没有就行添加

# 使用规则
使用 updateOne 表示无论条件匹配多少条记录，始终只更新第一条
使用 updateMany 表示条件匹配多少条就更新多少条
updateOne/updateMany 方法要求更新条件部分必须具有以下之一，否则将报错
    $set/$unset
    $push/$pushAll/$pop
    $pull/$pullAll
    $addToSet
// 报错
db.fruit.updateOne({name: "apple"}, {from: "China"})
```

### 使用 update 更新数组

```bash
$push: 增加一个对象到数组底部
$pushAll: 增加多个对象到数组底部
$pop: 从数组底部删除一个对象
$pull: 如果匹配指定的值，从数组中删除相应的对象
$pullAll: 如果匹配任意的值，从数据中删除相应的对象
$addToSet: 如果不存在则增加一个值到数组
```

### 使用 drop 删除集合

```bash
使用 db.<集合>.drop() 来删除一个集合
集合中的全部文档都会被删除
集合相关的索引也会被删除

db.colToBeDropped.drop()
```

### 使用 dropDatabase 删除数据库

```bash
使用 db.dropDatabase() 来删除数据库
数据库相应文件也会被删除，磁盘空间将被释放

use tempDB
db.dropDatabase()
show collections // No collections
show dbs // The db is gone
```

### Python 操作 MongoDB

```bash
# 环境准备
# Centos安装python3
yum install python3
pip3 install pymongo

# 检查
在 python 交互模式下导入 pymongo，检查驱动是否已正确安装
import pymongo
pymongo.version


# 确定 MongoDB 连接串
使用驱动连接到 MongoDB 集群只需要指定 MongoDB 连接字符串即可。其基本格式可以参考文档: Connection String URI Format
最简单的形式是mongodb://数据库服务器主机地址：端口号
如：mongodb://127.0.0.1:27017

# 初始化数据库连接以下操作在python终端操作就行
from pymongo import MongoClient
uri = "mongodb://root:root123@db1:27017"
client = MongoClient(uri)
client

# 插入数据
# 初始化数据库和集合
db = client["eshop"]
user_coll = db["users"]

# 插入一条新的用户数据
new_user = {"username": "nina", "password": "xxxx", "email":
"123456@qq.com "}
result = user_coll.insert_one(new_user)  # 这条执行之后MongoDB才会有数据
result
```
