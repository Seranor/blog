---
title: MongoDB聚合框架
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-06.html
toc: true
---

### 简介

```bash
MongoDB 聚合框架（Aggregation Framework）是一个计算框架
1.作用在一个或几个集合上
2.对集合中的数据进行的一系列运算
3.将这些数据转化为期望的形式

从效果而言，聚合框架相当于 SQL 查询里的 GROUP BY，LEFT OUTER JOIN， AS等
```

<!-- more -->

### 管道和步骤

```bash
整个聚合运算过程称为管道（Pipeline），它是由多个步骤（Stage）组成的，每个管道：
接受一系列文档（原始数据）
每个步骤对这些文档进行一系列运算
结果文档输出给下一个步骤
```

![6CZpmp](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/6CZpmp.png)

### 聚合运算的基本格式

```bash
pipeline = [$stage1, $stage2, ...$stageN];

db.<COLLECTION>.aggregate(
pipeline,
{ options }
);
```

![qtz4v5](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/qtz4v5.png)

![OiAelr](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/OiAelr.png)

![hn9SKU](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/hn9SKU.png)

### MQL 常用步骤与 SQL 对比

![jOXO8p](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/jOXO8p.png)

![5Gmlwo](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/5Gmlwo.png)

![6yyGxc](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/6yyGxc.png)

![TEfb3m](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/TEfb3m.png)

![ozaeeI](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/ozaeeI.png)

### 运算实例

测试数据

```bash
# wget https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/sql/dump.tar.gz
curl -O -k https://raw.githubusercontent.com/tapdata/geektimemongodb-course/master/aggregation/dump.tar.gz
tar xf dump.tar.gz

# 如果按照上述修改了用户和密码如下导入
mongorestore -uroot -proot123 --port 27017 --authenticationDatabase admin

# 进入数据库查看
mongo -uroot -proot123 admin

show dbs
use mock
show collections  //查看集合
db.orders.findOne()  //查看数据
```

计算

```bash
# 计算到目前为止的所有订单总销售额
db.orders.aggregate([
	{$group:
		{
			_id:null,  //将整个表做汇总
			total: {$sum: "$total"}  //total自定义的，"$total"才是数据的字段
		}
	}
])
// 结果 { "_id" : null, "total" : NumberDecimal("44019609") }

# 查询2019年第一季度（1月1日~3月31日）已完成订单（completed）的订单总金额和订单总数

db.orders.aggregate([
// 步骤1：匹配条件
{ $match: { status: "completed", orderDate: {
		$gte: ISODate("2019-01-01"),
		$lt: ISODate("2019-04-01") } } },

// 步骤二：聚合订单总金额、总运费、总数量
{ $group: {
_id: null,
total: { $sum: "$total" },
shippingFee: { $sum: "$shippingFee" },
count: { $sum: 1 } } },

{ $project: {
// 计算总金额
grandTotal: { $add: ["$total", "$shippingFee"] },
count: 1,
_id: 0 } }
])

// 结果：
// { "count" : 5875, "grandTotal" : NumberDecimal("2636376.00") }

```

### compass 使用聚合

![I01uEx](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/I01uEx.png)

![ZuMRhx](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/ZuMRhx.png)

语句导出不同的开发语言

![XT5Uxr](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/XT5Uxr.png)

![UpX1kj](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/UpX1kj.png)
