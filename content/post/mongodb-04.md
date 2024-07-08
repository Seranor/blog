---
title: MongoDB远程连接方式
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-04.html
toc: true
---

### 图形化工具

#### `compass`

<!-- more -->

```bash
# 下载地址，直安装即可
https://www.mongodb.com/try/download/compass

# 方式一的连接方法
mongodb://root:root123@db1:27017/admin

root          用户名
root123       密码
db1:27017     主机地址+端口
admin         验证库
```

方式一

![6gLJgx](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/6gLJgx.png)

方式二，在下图位置填入相应信息

![image-20211230160335813](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211230160335813.png)

![image-20211230160400469](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211230160400469.png)

#### `Navicat`

不做演示了

![Vd8uLU](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/Vd8uLU.png)

### 命令远程

```bash
mongo --host 192.168.0.11 --port 27017 -uroot -proot123 admin
```
