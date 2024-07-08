---
title: MongoDB用户管理
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - MongoDB
categories:
  - MongoDB
url: post/mongo-03.html
toc: true
---

### 注意事项

<!-- more -->

```
验证库，建立用户时use到的库，在使用用户时，要加上验证库才能登陆
对于管理员用户,必须在admin下创建
1. 建用户时,use到的库,就是此用户的验证库
2. 登录时,必须明确指定验证库才能登录
3. 通常,管理员用的验证库是admin,普通用户的验证库一般是所管理的库设置为验证库
4. 如果直接登录到数据库,不进行use,默认的验证库是test,不是我们生产建议的
```

### 基本语法

```bash
mongo 10.0.0.151/admin
use admin

db.createUser
{
    user: "<name>",
    pwd: "<cleartext password>",
    roles: [
       { role: "<role>",
     db: "<database>" } | "<role>",
    ...
    ]
}

# 基本语法
user:用户名
pwd:密码
roles:
    role:角色名
    db:作用对象
role：root, readWrite,read

验证数据库：
mongo -u test -p 123 10.0.0.21/test
```

### 用法实例

#### 管理员用户

```bash
# 创建超级管理员 管理所有数据库（必须use admin再去创建）
$ mongo  # 命令行输入mongo进入数据库
use admin
db.createUser(
{
    user: "root",
    pwd: "root123",
    roles: [ { role: "root", db: "admin" } ]
}
)

# 验证用户
db.auth('root','root123')  # 结果为1代表成功

# 配置文件中，加入以下配置
vim /mongodb/conf/mongodb.conf
security:
  authorization: enabled

# 重启mongodb
mongod -f /mongodb/conf/mongodb.conf --shutdown
mongod -f /mongodb/conf/mongodb.conf

# 登录验证
mongo -uroot -proot123  admin
mongo -uroot -proot123  192.168.0.11/admin

  # 或者
  mongo
  use admin
  db.auth('root','root123')

# 查看用户
use admin
db.system.users.find().pretty()

# 创建库管理用户
mongo -uroot -proot123  admin

use app
db.createUser(
{
user: "admin",
pwd: "admin",
roles: [ { role: "dbAdmin", db: "app" } ]
}
)

db.auth('admin','admin')

# 登录测试
mongo -uadmin -padmin app
```

#### 普通用户

```bash
# 创建对app数据库，读、写权限的用户app01：
    # 超级管理员用户登陆
    mongo -uroot -proot123 admin

    # 选择一个验证库
    use app

    # 创建用户
    db.createUser(
        {
            user: "app01",
            pwd: "app01",
            roles: [ { role: "readWrite" , db: "app" } ]
        }
    )

    # 验证
    mongo  -uapp01 -papp01 app


# 创建app数据库读写权限的用户并对test数据库具有读权限
mongo -uroot -proot123 admin

use app
db.createUser(
{
user: "app03",
pwd: "app03",
roles: [ { role: "readWrite", db: "app" },
{ role: "read", db: "test" }
]
}
)

# 查询mongodb中的用户信息
mongo -uroot -proot123 admin
db.system.users.find().pretty()

# 删除用户（root身份登录，use到验证库）
mongo -uroot -proot123 admin
use app
db.dropUser("admin")
```

### 角色关系

![VX8Kie](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/VX8Kie.png)
