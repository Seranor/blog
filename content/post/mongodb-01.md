---
title: MongoDB简介
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-01.html
toc: true
---

### 简单认识

| Q              | A                                                             |
| -------------- | ------------------------------------------------------------- |
| 什么是 MongoDB | 一个以 JSON 为数据模型的文档数据库                            |
| 文档数据库     | 文档来自于'JSON Document'，并非 PDF，Word 等                  |
| 主要用途       | OLTP/OLAP 数据库，类似于 Oracle，MySQL 海量数据处理，数据平台 |
| 主要特点       | 无模式或可选，友好的 JSON 数据模型，开发方便                  |
| 版本           | 企业版和社区版                                                |

<!-- more -->

### 版本变迁

![image-20211229213159423](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211229213159423.png)

### 与 RDBMS 比较

| 功能         | MongoDB                                  | RDBMS           |
| ------------ | ---------------------------------------- | --------------- |
| 数据模型     | JSON                                     | Relational      |
| 数据库类型   | OLTP/OLAP                                | OLTP/OLAP       |
| CRUD 操作    | MQL/SQL                                  | SQL/SQLX        |
| 高可用       | 原生 Replica-Set                         | Cluster、中间件 |
| 横向扩展能力 | 原生 MSC                                 | 分片、中间件    |
| 索引支持     | B-Tree、F-text、GIS、multikey、HASH、TTL | B-Tree          |
| 开发难度     | 简单                                     | 难              |
| 数据容量     | 无理论上限                               | 千万、亿        |
| 扩展方式     | 垂直扩展+水平扩展                        | 垂直扩展        |

### 逻辑结构对比

![image-20211229213216469](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211229213216469.png)

### MongoDB 优势

- 简单直观

  以自然的方式来建模，以直观的方式来与数据库交互

- 结构灵活

  弹性模式从容响应需求的频繁变化

- 快速开发

  做更多的事情，写更少的代码

![image-20211229213046722](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211229213046722.png)
