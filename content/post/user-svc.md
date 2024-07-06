---
title: 微服务开发用户服务流程
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - gRPC
  - Gin
  - Python
categories:
  - Python
url: post/python-grpc-user.html
toc: true
---

## 用户服务

### 微服务(python)

#### 目录结构

```python
mxshop_srvs
└── user_srv          # 用户服务
    ├── __init__.py
    ├── handler
    ├── logs
    ├── model
    ├── proto
    └── settings
```

<!-- more -->

#### 创建虚拟环境

```python
mkvirtualenv mxshop_srv
```

#### 用户表设计

```python
pip3 install peewee
pip3 install pymysql
```

##### `user_srv/settings/settings.py`

```python
from abc import ABC

from playhouse.pool import PooledMySQLDatabase  # peewee 连接池连接
from playhouse.shortcuts import ReconnectMixin  # 断开后重新连接数据库 否则会报错 mysql gone away


class ReconnectMysqlDatabase(PooledMySQLDatabase, ReconnectMixin, ABC):
    pass


MYSQL_DB = "mxshop_user_srv"
MYSQL_HOST = "localhost"
MYSQL_PORT = 3306
MYSQL_USER = "root"
MYSQL_PASSWORD = "asd123..."
DB = ReconnectMysqlDatabase(database=MYSQL_DB, host=MYSQL_HOST, port=MYSQL_PORT, user=MYSQL_USER,
                            password=MYSQL_PASSWORD)
```

##### `user_srv/model/models.py`

```python
from peewee import *
from user_srv.settings import settings


class BaseModel(Model):
    class Meta:
        database = settings.DB


class User(BaseModel):
    # 用户模型
    GENDER_CHOICES = (
        ("female", "女"),
        ("male", "男")
    )
    ROLE_CHOICES = (
        (1, "普通用户"),
        (2, "管理员")
    )
    mobile = CharField(max_length=11, index=True, unique=True, verbose_name="手机号")
    password = CharField(max_length=100, verbose_name="密码")  # 密码密文 密文不可反解
    nick_name = CharField(max_length=20, null=True, verbose_name="昵称")
    head_url = CharField(max_length=200, null=True, verbose_name="头像")
    birthday = DateField(null=True, verbose_name="生日")
    address = CharField(max_length=200, null=True, verbose_name="地址")
    desc = TextField(null=True, verbose_name="个人简介")
    gender = CharField(max_length=6, choices=GENDER_CHOICES, null=True, verbose_name="性别")
    role = IntegerField(default=1, choices=ROLE_CHOICES, verbose_name="用户角色")


if __name__ == '__main__':
    settings.DB.create_tables([User])  # 创建表

```

#### 密码处理

```python
# https://passlib.readthedocs.io/en/stable/

# 安装
pip install passwlib

# 使用
from passlib.hash import pbkdf2_sha256

hash1 = pbkdf2_sha256.hash("123456")
print(hash1)
print(pbkdf2_sha256.verify("123456", hash1))  # 校验 返回bool

# 存入用户
for i in range(10):
    user = User()
    user.nick_name = f"user{i}"
    user.mobile = f"1308822955{i}"
    user.password = pbkdf2_sha256.hash("admin123")
    user.save()

# 校验用户密码
for user in User.select():
    print(pbkdf2_sha256.verify("admin123", user.password))  # 返回 True
```

### `proto`接口定义

#### `user_srv/proto/user.proto`

```protobuf
syntax = "proto3";
import "google/protobuf/empty.proto";
option go_package = "../proto";

service User {
    rpc GetUserList(PageInfo) returns (UserListResonse); //用户列表
    rpc GetUserByMobile(MobileRequest) returns (UserInfoResponse); //通过mobile查询用户
    rpc GetUserById(IdRequest) returns (UserInfoResponse); //通过id查询用户
    rpc CreateUser(CreateUserInfo) returns (UserInfoResponse); //添加用户
    rpc UpdateUser(UpdateUserInfo) returns (google.protobuf.Empty); // 更新用户
    rpc CheckPassWord (PasswordCheckInfo) returns (CheckResponse); //检查密码
}

message PasswordCheckInfo {
    string password = 1;
    string encryptedPassword = 2;
}

message CheckResponse {
    bool success = 1;
}

message PageInfo {
    uint32 pn = 1;
    uint32 pSize = 2;
}

message MobileRequest {
    string mobile = 1;
}

message IdRequest {
    int32 id = 1;
}

message CreateUserInfo {
    string nickName = 1;
    string passWord = 2;
    string mobile = 3;
}

message UpdateUserInfo {
    int32 id = 1;
    string nickName = 2;
    string gender = 3;
    uint64 birthDay = 4;
}


message UserInfoResponse {
    int32 id = 1;
    string passWord = 2;
    string mobile = 3;
    string nickName = 4;
    uint64 birthDay = 5;
    string gender = 6;
    int32 role = 7;
}

message UserListResonse {
    int32 total = 1;
    repeated UserInfoResponse data = 2;
}
```

#### 安装模块生成文件

```python
# 安装模块
pip3 install grpcio
pip3 install grpcio-tools

# 生成文件
cd user_srv/proto/
python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. user.proto

# 修改导入
user_srv/proto/user_pb2_grpc.py 文件中
import user_pb2 as user__pb2 改为
from . import user_pb2 as user__pb2
```

#### 日志库`loguru`

```python
pip install loguru

# 日志文件 切割
logger.add("logs/user_srv_{time}.log", rotation="200 MB")

# 颜色
logger.debug("调试信息")
logger.info("普通信息")
logger.warning("警告信息")
logger.error("错误信息")
logger.critical("严重错误信息")

# 装饰器
@logger.catch
def my_func(x, y, z):
    return 1 / (x + y + z)
my_func(0, 0, 0)

# 上下文比较全
```

#### 查询所有用户接口

##### `user_srv/handler/user.py`

```python
import time
from datetime import date
import grpc
from loguru import logger
from user_srv.proto import user_pb2, user_pb2_grpc
from user_srv.model.models import User


class UserServicer(user_pb2_grpc.UserServicer):
    @logger.catch
    def GetUserList(self, request: user_pb2.PageInfo, context):
        # 获取用户的列表
        rsp = user_pb2.UserListResonse()

        users = User.select()
        rsp.total = users.count()

        start = 0
        per_page_nums = 10  # 每页数量
        if request.pSize:
            per_page_nums = request.pSize
        if request.pn:
            start = per_page_nums * (request.pn - 1)

        users = users.limit(per_page_nums).offset(start)

        for user in users:
            user_info_rsp = user_pb2.UserInfoResponse()

            user_info_rsp.id = user.id
            user_info_rsp.passWord = user.password
            user_info_rsp.mobile = user.mobile
            user_info_rsp.role = user.role

            if user_info_rsp.nickName:
                user_info_rsp.nickName = user.nick_name
            if user.gender:
                user_info_rsp.gender = user.gender
            if user.birthday:
                user_info_rsp.birthDay = int(time.mktime(user.birthday.timetuple()))

            rsp.data.append(user_info_rsp)
        return rsp
```

##### `user_srv/server.py`

```python
import grpc
import logging
from loguru import logger
from concurrent import futures
from user_srv.proto import user_pb2_grpc
from user_srv.handler.user import UserServicer


def serve():
    logger.add("logs/user_srv_{time}.log", rotation="200 MB")

    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServicer_to_server(UserServicer(), server)
    server.add_insecure_port('[::]:50051')
    logger.info("启动服务：127.0.0.1:50051")
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

##### `user_srv/tests/user.py`

```python
# 测试服务
import grpc

from user_srv.proto import user_pb2_grpc, user_pb2


class UserTest:
    def __init__(self):
        # 连接grpc服务器
        channel = grpc.insecure_channel("127.0.0.1:50051")
        self.stub = user_pb2_grpc.UserStub(channel)

    def user_list(self):
        rsp: user_pb2.UserInfoResponse = self.stub.GetUserList(user_pb2.PageInfo(pn=2, pSize=2))
        print(rsp.total)
        for user in rsp.data:
            print(user.mobile, user.birthDay)


if __name__ == '__main__':
    user = UserTest()
    user.user_list()

```

#### 优雅退出

##### `user_srv/server.py`

```python
import os
import sys
import grpc
import logging
import signal
from loguru import logger
from concurrent import futures

BASE_DIR = os.path.dirname(os.path.abspath(os.path.dirname(__file__)))
sys.path.insert(0, BASE_DIR)

from user_srv.proto import user_pb2_grpc
from user_srv.handler.user import UserServicer


def on_exit(signo, frame):
    logger.info("进程中断")
    sys.exit(0)


def serve():
    logger.add("logs/user_srv_{time}.log", rotation="200 MB")

    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServicer_to_server(UserServicer(), server)
    server.add_insecure_port('[::]:50051')

    # 主进程退出信号监听
    '''
        windows 支持的信号是有限的
            SIGINT   ctrl+c
            SIGTERM  kill的
    '''
    signal.signal(signal.SIGINT, on_exit)
    signal.signal(signal.SIGTERM, on_exit)

    logger.info("启动服务：127.0.0.1:50051")
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

#### 传入启动参数

##### `user_srv/server.py`

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--ip',
                    nargs="?",
                    type=str,
                    default="127.0.0.1",
                    help="binding ip")
parser.add_argument('--port',
                    nargs="?",
                    type=int,
                    default=50051,
                    help="the listening port")
args = parser.parse_args()
print(args)
print(args.ip, args.port)

'''
--ip     传入参数
default  默认值
help     帮助信息

运行脚本的时候可以输入 --help 查看到所需要传入的参数
'''
```

```python
import os
import sys
import grpc
import logging
import signal
import argparse
from loguru import logger
from concurrent import futures

BASE_DIR = os.path.dirname(os.path.abspath(os.path.dirname(__file__)))
sys.path.insert(0, BASE_DIR)

from user_srv.proto import user_pb2_grpc
from user_srv.handler.user import UserServicer


def on_exit(signo, frame):
    logger.info("进程中断")
    sys.exit(0)


def serve():
    parser = argparse.ArgumentParser()
    parser.add_argument('--ip',
                        nargs="?",
                        type=str,
                        default="127.0.0.1",
                        help="binding ip")
    parser.add_argument('--port',
                        nargs="?",
                        type=int,
                        default=50051,
                        help="the listening port")

    args = parser.parse_args()
    # print(args)
    # print(args.ip, args.port)

    logger.add("logs/user_srv_{time}.log", rotation="200 MB")

    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServicer_to_server(UserServicer(), server)
    server.add_insecure_port(f'{args.ip}:{args.port}')

    # 主进程退出信号监听
    '''
        windows 支持的信号是有限的
            SIGINT   ctrl+c
            SIGTERM  kill的
    '''
    signal.signal(signal.SIGINT, on_exit)
    signal.signal(signal.SIGTERM, on_exit)

    logger.info(f"启动服务: {args.ip} : {args.port}")
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

#### 基本实现

##### `user_srv/proto/user.proto`

```protobuf
syntax = "proto3";
import "google/protobuf/empty.proto";
option go_package = "../proto";

service User {
    rpc GetUserList(PageInfo) returns (UserListResonse); //用户列表
    rpc GetUserByMobile(MobileRequest) returns (UserInfoResponse); //通过mobile查询用户
    rpc GetUserById(IdRequest) returns (UserInfoResponse); //通过id查询用户
    rpc CreateUser(CreateUserInfo) returns (UserInfoResponse); //添加用户
    rpc UpdateUser(UpdateUserInfo) returns (google.protobuf.Empty); // 更新用户
    rpc CheckPassWord (PasswordCheckInfo) returns (CheckResponse); //检查密码
}

message PasswordCheckInfo {
    string password = 1;
    string encryptedPassword = 2;
}

message CheckResponse {
    bool success = 1;
}

message PageInfo {
    uint32 pn = 1;
    uint32 pSize = 2;
}

message MobileRequest {
    string mobile = 1;
}

message IdRequest {
    int32 id = 1;
}

message CreateUserInfo {
    string nickName = 1;
    string passWord = 2;
    string mobile = 3;
}

message UpdateUserInfo {
    int32 id = 1;
    string nickName = 2;
    string gender = 3;
    uint64 birthDay = 4;
}


message UserInfoResponse {
    int32 id = 1;
    string passWord = 2;
    string mobile = 3;
    string nickName = 4;
    uint64 birthDay = 5;
    string gender = 6;
    int32 role = 7;
}

message UserListResonse {
    int32 total = 1;
    repeated UserInfoResponse data = 2;
}
```

##### `user_srv/model/models.py`

```python
from peewee import *
from user_srv.settings import settings
from passlib.hash import pbkdf2_sha256

class BaseModel(Model):
    class Meta:
        database = settings.DB


class User(BaseModel):
    # 用户模型
    GENDER_CHOICES = (
        ("female", "女"),
        ("male", "男")
    )
    ROLE_CHOICES = (
        (1, "普通用户"),
        (2, "管理员")
    )
    mobile = CharField(max_length=11, index=True, unique=True, verbose_name="手机号")
    password = CharField(max_length=100, verbose_name="密码")  # 密码密文 密文不可反解
    nick_name = CharField(max_length=20, null=True, verbose_name="昵称")
    head_url = CharField(max_length=200, null=True, verbose_name="头像")
    birthday = DateField(null=True, verbose_name="生日")
    address = CharField(max_length=200, null=True, verbose_name="地址")
    desc = TextField(null=True, verbose_name="个人简介")
    gender = CharField(max_length=6, choices=GENDER_CHOICES, null=True, verbose_name="性别")
    role = IntegerField(default=1, choices=ROLE_CHOICES, verbose_name="用户角色")


if __name__ == '__main__':
    settings.DB.create_tables([User])
    # 对称加密 非对称加密 无法知道原始密码是什么
    # md5
    # import hashlib
    # m = hashlib.md5()
    # m.update(b"123456")
    # print(m.hexdigest())
    # from passlib.hash import pbkdf2_sha256
    #
    # hash1 = pbkdf2_sha256.hash("123456")
    # print(hash1)
    # print(pbkdf2_sha256.verify("123456", hash1))
    # for i in range(10):
    #     user = User()
    #     user.nick_name = f"user{i}"
    #     user.mobile = f"1308822955{i}"
    #     user.password = pbkdf2_sha256.hash("admin123")
    #     user.save()
    # for user in User.select():
    #     print(pbkdf2_sha256.verify("admin123", user.password))

    # # 时间转换
    # for user in User.select():
    #     if user.birthday:
    #         print(user.birthday)
    #         import time
    #         u_time = time.mktime(user.birthday.timetuple())
    #         print(int(u_time))
    #         from datetime import date
    #         print(date.fromtimestamp(u_time))
```

##### `user_srv/handler/user.py`

```python
import time
from datetime import date
import grpc
from loguru import logger
from peewee import DoesNotExist
from passlib.hash import pbkdf2_sha256
from google.protobuf import empty_pb2
from user_srv.proto import user_pb2, user_pb2_grpc
from user_srv.model.models import User


class UserServicer(user_pb2_grpc.UserServicer):
    @logger.catch
    def GetUserList(self, request: user_pb2.PageInfo, context):
        # 获取用户的列表
        rsp = user_pb2.UserListResonse()

        users = User.select()
        rsp.total = users.count()

        start = 0
        per_page_nums = 10  # 每页数量
        if request.pSize:
            per_page_nums = request.pSize
        if request.pn:
            start = per_page_nums * (request.pn - 1)

        users = users.limit(per_page_nums).offset(start)

        for user in users:
            rsp.data.append(self._convert_user_to_rsp(user))
        return rsp

    @logger.catch
    def GetUserById(self, request: user_pb2.IdRequest, context):
        # 通过ID查询用户
        try:
            user = User.get(User.id == request.id)
            return self._convert_user_to_rsp(user)
        except DoesNotExist as e:
            context.set_code(grpc.StatusCode.NOT_FOUND)
            context.set_details("用户不存在")
            return user_pb2.UserInfoResponse()

    @logger.catch
    def GetUserByMobile(self, request: user_pb2.MobileRequest, context):
        # 通过 mobile 查询用户
        try:
            user = User.get(User.mobile == request.mobile)
            return self._convert_user_to_rsp(user)
        except DoesNotExist:
            context.set_code(grpc.StatusCode.NOT_FOUND)
            context.set_details("用户不存在")
            return user_pb2.UserInfoResponse()

    @logger.catch
    def CreateUser(self, request: user_pb2.CreateUserInfo, context):
        # 新建用户
        try:
            User.get(User.mobile == request.mobile)
            context.set_code(grpc.StatusCode.ALREADY_EXISTS)
            context.set_details("用户已经存在")
            return user_pb2.UserInfoResponse()
        except DoesNotExist:
            pass
        user = User()
        user.nick_name = request.nickName
        user.mobile = request.mobile
        user.password = pbkdf2_sha256.hash(request.passWord)
        user.save()

        return self._convert_user_to_rsp(user)

    @logger.catch
    def UpdateUser(self, request: user_pb2.UpdateUserInfo, context):
        # 更新用户
        try:
            user = User.get(User.id == request.id)
            user.nick_name = request.nickName
            user.gender = request.gender
            user.birthday = date.fromisoformat(request.birthDay)  # proto 中是uint 类型 需要转换成时间
            user.save()
            return empty_pb2.Empty()
        except DoesNotExist:
            context.set_code(grpc.StatusCode.NOT_FOUND)
            context.set_details("用户不存在")
            return user_pb2.UserInfoResponse()

    def _convert_user_to_rsp(self, user):
        # 将 user 的model 转换为 message 对象
        user_info_rsp = user_pb2.UserInfoResponse()
        user_info_rsp.id = user.id
        user_info_rsp.passWord = user.password
        user_info_rsp.mobile = user.mobile
        user_info_rsp.role = user.role

        if user_info_rsp.nickName:
            user_info_rsp.nickName = user.nick_name
        if user.gender:
            user_info_rsp.gender = user.gender
        if user.birthday:
            user_info_rsp.birthDay = int(time.mktime(user.birthday.timetuple()))

        return user_info_rsp
```

##### `user_srv/settings/settings.py`

```python
from abc import ABC
from playhouse.pool import PooledMySQLDatabase  # 连接池连接
from playhouse.shortcuts import ReconnectMixin  # 断开后重新连接数据库 否则会报错 mysql gone away


class ReconnectMysqlDatabase(PooledMySQLDatabase, ReconnectMixin, ABC):
    pass


MYSQL_DB = "mxshop_user_srv"
MYSQL_HOST = "localhost"
MYSQL_PORT = 3306
MYSQL_USER = "root"
MYSQL_PASSWORD = "asd123..."
DB = ReconnectMysqlDatabase(database=MYSQL_DB, host=MYSQL_HOST, port=MYSQL_PORT, user=MYSQL_USER,
                            password=MYSQL_PASSWORD)
```

##### `user_srv/server.py`

```python
import os
import sys
import grpc
import logging
import signal
import argparse
from loguru import logger
from concurrent import futures

BASE_DIR = os.path.dirname(os.path.abspath(os.path.dirname(__file__)))
sys.path.insert(0, BASE_DIR)

from user_srv.proto import user_pb2_grpc
from user_srv.handler.user import UserServicer


def on_exit(signo, frame):
    logger.info("进程中断")
    sys.exit(0)


def serve():
    parser = argparse.ArgumentParser()
    parser.add_argument('--ip',
                        nargs="?",
                        type=str,
                        default="127.0.0.1",
                        help="binding ip")
    parser.add_argument('--port',
                        nargs="?",
                        type=int,
                        default=50051,
                        help="the listening port")

    args = parser.parse_args()
    # print(args)
    # print(args.ip, args.port)

    logger.add("logs/user_srv_{time}.log", rotation="200 MB")

    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServicer_to_server(UserServicer(), server)
    server.add_insecure_port(f'{args.ip}:{args.port}')

    # 主进程退出信号监听
    '''
        windows 支持的信号是有限的
            SIGINT   ctrl+c
            SIGTERM  kill的
    '''
    signal.signal(signal.SIGINT, on_exit)
    signal.signal(signal.SIGTERM, on_exit)

    logger.info(f"启动服务: {args.ip} : {args.port}")
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

##### `user_srv/tests/user.py`

```python
# 测试脚本
import grpc
from user_srv.proto import user_pb2_grpc, user_pb2

class UserTest:
    def __init__(self):
        # 连接grpc服务器
        channel = grpc.insecure_channel("127.0.0.1:50051")
        self.stub = user_pb2_grpc.UserStub(channel)

    def user_list(self):
        rsp: user_pb2.UserInfoResponse = self.stub.GetUserList(user_pb2.PageInfo(pn=2, pSize=2))
        print(rsp.total)
        for user in rsp.data:
            print(user.mobile, user.birthDay)

    def get_user_by_id(self, id):
        rsp: user_pb2.UserInfoResponse = self.stub.GetUserById(user_pb2.IdRequest(id=id))
        print(rsp.mobile)

    def get_user_by_mobile(self, mobile):
        rsp: user_pb2.UserInfoResponse = self.stub.GetUserByMobile(user_pb2.MobileRequest(mobile=mobile))
        print(rsp.id)

    def create_user(self, nick_name, mobile, password):
        rsp: user_pb2.UserInfoResponse = self.stub.CreateUser(user_pb2.CreateUserInfo(
            nickName=nick_name,
            passWord=password,
            mobile=mobile
        ))
        print(rsp.id)


if __name__ == '__main__':
    user = UserTest()
    # user.user_list()
    # user.get_user_by_id(10)
    # user.get_user_by_mobile('13088229550')
    user.create_user("lzj", "13088888888", "admin123")
```

### `web`服务(go)

#### 目录结构

```
mxshop-api
├── go.mod
└── user-web
    ├── api
    ├── config
    ├── forms
    ├── global
    ├── initialize
    ├── main.go
    ├── middlewares
    ├── proto
    ├── router
    ├── utils
    └── validator
```

#### 日志库`zap`

```go
// 地址 https://github.com/uber-go/zap

// 安装 go get go.uber.org/zap

// user-web/zap_test/main.go
package main

import "time"
import "go.uber.org/zap"

func main() {
	logger, _ := zap.NewProduction() // 生产环境 json
	//logger, _ := zap.NewDevelopment() // 测试环境

	defer logger.Sync() // flushes buffer, if any
	url := "https://klcc.cc"
	// 运用复杂些 性能稍微高
	logger.Info("failed to fetch URL",
		zap.String("url", url),
		zap.Int("nums", 3),
	)

	// 下面的这个方式运用简单
	sugar := logger.Sugar()
	sugar.Infow("failed to fetch URL",
		// Structured context as loosely typed key-value pairs.
		"url", url,
		"attempt", 3,
		"backoff", time.Second,
	)
	sugar.Infof("Failed to fetch URL: %s", url)
}
```

输出到文件中

```go
package main

import (
	"go.uber.org/zap"
	"time"
)

func NewLogger() (*zap.Logger, error) {
	cfg := zap.NewProductionConfig()
	cfg.OutputPaths = []string{
		"./myproject.log",
		"stderr",
		"stdout",
	}
	return cfg.Build()
}

func main() {
	logger, err := NewLogger()
	if err != nil {
		panic(err)
	}
	su := logger.Sugar()
	defer su.Sync()
	url := "https://klcc.cc"
	sugar := logger.Sugar()
	sugar.Infow("failed to fetch URL",
		"url", url,
		"attempt", 3,
		"backoff", time.Second,
	)
	sugar.Infof("Failed to fetch URL: %s", url)
}
```

#### 拆分`gin`和引入 zap

##### `user-web/main.go`

```go
package main

import (
	"fmt"
	"go.uber.org/zap"
	"mxshop-api/user-web/initialize"
)

func main() {
	port := 8021
	// 初始化logger
	initialize.InitLogger()

	// 初始化 routers
	Router := initialize.Routers()
	//S() 可以获取一个全局的 sugar 可以让我们自己设置一个全局的 logger S函数和L函数很有用 全局访问 logger
	zap.S().Infof("启动服务器，端口: %d", port)
	if err := Router.Run(fmt.Sprintf(":%d", port)); err != nil {
		zap.S().Panic("启动失败: ", err.Error())
	}
}
```

##### `user-web/router/user.go`

```go
package router

import (
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
	"mxshop-api/user-web/api"
)

func InitUserRouter(Router *gin.RouterGroup) {
	UserRouter := Router.Group("user")
	zap.S().Infof("配置用户相关的URL")
	{
		UserRouter.GET("list", api.GetUserList)
	}
}
```

##### `user-web/initialize/router.go`

```go
package initialize

import (
	"github.com/gin-gonic/gin"
	"mxshop-api/user-web/router"
)

func Routers() *gin.Engine {
	Router := gin.Default()
	ApiGroup := Router.Group("/u/v1")
	router.InitUserRouter(ApiGroup)

	return Router
}
```

##### `user-web/initialize/logger.go`

```go
package initialize

import "go.uber.org/zap"

func InitLogger() {
	// 其他地方就可以只用 S()
	//logger, _ := zap.NewProduction()
	logger, _ := zap.NewDevelopment()
	zap.ReplaceGlobals(logger)
}
```

##### `user-web/api/user.go`

```go
package api

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func GetUserList(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, gin.H{
		"message": "pong",
	})
}

```

#### 调用 grpc 服务

```go

```

#### 配置文件管理`viper`

``` 
使用go应用程序的完整配置解决方案
支持 JSON TOML YAML HCL 等
支持设置默认值
实时监控的重新读取配置文件
从环境中读取
从远程配置系统读取并监控配置变化
从命令行参数读取
从buffer读取
显式配置值

https://github.com/spf13/viper

go get github.com/spf13/viper
```

##### 简单配置文件

`main.go`

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

type ServerConfig struct {
	ServiceName string `mapstructure:"name"`
	Port        int    `mapstructure:"port"`
}

func main() {
	v := viper.New()
	// 文件的路径如何设置
	v.SetConfigFile("viper_test/ch01/config.yaml")
	if err := v.ReadInConfig(); err != nil {
		panic(err)
	}

	ServerConfig := ServerConfig{}
	if err := v.Unmarshal(&ServerConfig); err != nil {
		panic(err)
	}
	fmt.Println(v.Get("name"), v.Get("port"))
	fmt.Println(ServerConfig)

}
```

`config.yaml`

```yaml
name: "user-web"
port: 8022
```

##### 两层 yaml

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

type MysqlConfig struct {
	Host string `mapstructure:"host"`
	Port int    `mapstructure:"port"`
}
type ServerConfig struct {
	ServiceName string      `mapstructure:"name"`
	MysqlInfo   MysqlConfig `mapstructure:"mysql"`
}

func main() {
	v := viper.New()
	// 文件的路径如何设置
	v.SetConfigFile("viper_test/ch02/config.yaml")
	if err := v.ReadInConfig(); err != nil {
		panic(err)
	}
	ServerConfig := ServerConfig{}
	if err := v.Unmarshal(&ServerConfig); err != nil {
		panic(err)
	}
	fmt.Println(ServerConfig)
}
```

```yaml
name: "user-web"
mysql:
  host: "127.0.0.1"
  port: 3306
```

##### 隔离配置文件

`config-pro.yaml`

```yaml
name: "user-web"
mysql:
  host: "10.0.0.80"
  port: 3308
```

`cofig-debug.yaml`

```yaml
name: "user-web2"
mysql:
  host: "127.0.0.1"
  port: 3309
```

`main.go`

```go
package main

import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"github.com/spf13/viper"
	"time"
)

type MysqlConfig struct {
	Host string `mapstructure:"host"`
	Port int    `mapstructure:"port"`
}
type ServerConfig struct {
	ServiceName string      `mapstructure:"name"`
	MysqlInfo   MysqlConfig `mapstructure:"mysql"`
}

// 获取环境变量的值
func GetEnvInfo(env string) bool {
	viper.AutomaticEnv()
	return viper.GetBool(env)
}
func main() {
	debug := GetEnvInfo("MXSHOP_DEBUG")  // 调用 检查环境变量中 MXSHOP_DEBUG 的值
	configFilePrefix := "config"
	configFileName := fmt.Sprintf("viper_test/ch02/%s-pro.yaml", configFilePrefix)
	if debug {
		configFileName = fmt.Sprintf("viper_test/ch02/%s-debug.yaml", configFilePrefix)
	}

	v := viper.New()
	v.SetConfigFile(configFileName)
	if err := v.ReadInConfig(); err != nil {
		panic(err)
	}
	ServerConfig := ServerConfig{}
	if err := v.Unmarshal(&ServerConfig); err != nil {
		panic(err)
	}
	fmt.Println(ServerConfig)

	// viper 动态监控配置变化
	v.WatchConfig()
	v.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("config file changed: ", e.Name)
		_ = v.ReadInConfig()
		_ = v.Unmarshal(&ServerConfig)
		fmt.Println(ServerConfig)
	})
	time.Sleep(time.Second * 300)
}
```

#### 整体加入配置文件管理

```go

```

#### 表单验证

```go

```

#### 自定义`mobile`验证器

```go

```

#### 登录逻辑

```go

```

#### `jwt`集成到`gin`

```go
// https://github.com/dgrijalva/jwt-go

go get github.com/dgrijalva/jwt-go
```

#### 跨域问题

```go

```

#### 验证码

```go
// https://github.com/mojocn/base64Captcha
```

#### Redis 存储验证码

```go
// https://github.com/go-redis/redis
```

### 注册中心

#### 安装和基础使用

```shell
https://github.com/consul/consul

# 启动一个
docker run -itd --name consul-web \
-p 8500:8500 \
-p 8300:8300 -p 8301:8301 \
-p 8302:8302 -p 8600:8600/udp \
consul consul agent -dev -client=0.0.0.0 \
-enable-script-checks



web页面
http://10.0.0.70:8500/

dns
http://10.0.0.70:8600/


dig @10.0.0.70 -p 8600 consul.service.consul SRV

# 帮助文档
https://www.consul.io/api-docs/agent/service#register-service

# 注册服务
curl -X PUT \
  http://127.0.0.1:8500/v1/agent/service/register \
  -H 'content-type: application/json' \
  -d '{
        "Name": "mshop-web",
        "ID": "mshop-web",
        "Tags": ["mxshop", "klcc", "imooc", "web"],
        "Address": "127.0.0.1",
        "Port": 50051
    }'

# 注销 web 的服务
curl -X PUT   http://127.0.0.1:8500/v1/agent/service/deregister/mshop-web   -H 'content-type: application/json'

# 查看服务列表
curl http://localhost:8500/v1/agent/services

# 查询指定服务
curl http://localhost:8500/v1/agent/services?filter=Service==web
curl http://localhost:8500/v1/agent/services?filter=Service!=web

# 查看健康状态
curl http://localhost:8500/v1/agent/health/service/name/mshop-web
```

#### Python 操作 consul

```python
import requests

headers = {
    "contentType": "application/json"
}


# 注册服务
#  "HTTP": f"http://{address}:{port}/health",
# "GRPC": f"{address}:{port}",
# "GRPCUseTLS": False,
def register(name, id, address, port):
    url = "http://10.0.0.70:8500/v1/agent/service/register"
    rsp = requests.put(url, headers=headers, json={
        "Name": name,
        "ID": id,
        "Tags": ["mxshop", "klcc", "imooc", "web"],
        "Address": address,
        "Port": port,
        "Check": {
            "HTTP": f"http://{address}:{port}/health",
            "TimeOut": "5s",
            "InterVal": "5s",
            "DeregisterCriticalServiceAfter": "5s",
        }
    })
    if rsp.status_code == 200:
        print("注册成功")
    else:
        print("注册失败", rsp.status_code)


# 注销服务
def deregister(id):
    url = f"http://10.0.0.70:8500/v1/agent/service/deregister/{id}"
    rsp = requests.put(url, headers=headers)
    if rsp.status_code == 200:
        print("注销成功")
    else:
        print("注销失败", rsp.status_code)


# 过滤服务
def filter_service(name):
    url = "http://10.0.0.70:8500/v1/agent/services"
    params = {
        "filter": f'Service == "{name}"',
    }
    rsp = requests.get(url, params=params).json()
    for k, v in rsp.items():
        print(k, v)


if __name__ == '__main__':
    # register("mshop-web", "mshop-web", "192.168.0.100", 8022)
    # register("mshop-srv", "mshop-srv", "192.168.0.100", 50051)
    # deregister("mshop-web")
    filter_service("mshop-srv")

# python 第三方库 https://github.com/poppyred/python-consul2
# pip3 install python-consul2
```

#### go 操作 consul

```go
package main

import (
	"fmt"
	"github.com/hashicorp/consul/api"
)

func Register(address string, port int, name string, tags []string, id string) error {
	cfg := api.DefaultConfig()
	cfg.Address = "10.0.0.70:8500"

	client, err := api.NewClient(cfg)
	if err != nil {
		panic(err)
	}

	// 生成对应的检查对象
	check := &api.AgentServiceCheck{
		HTTP:                           "http://192.168.0.100:8022/health",
		Timeout:                        "5s",
		Interval:                       "5s",
		DeregisterCriticalServiceAfter: "10s",
	}

	// 生成注册对象
	registration := new(api.AgentServiceRegistration)
	registration.Name = name
	registration.ID = id
	registration.Port = port
	registration.Tags = tags
	registration.Address = address
	registration.Check = check

	err = client.Agent().ServiceRegister(registration)
	if err != nil {
		panic(err)
	}
	return nil
}

func AllServices() {
	cfg := api.DefaultConfig()
	cfg.Address = "10.0.0.70:8500"

	client, err := api.NewClient(cfg)
	if err != nil {
		panic(err)
	}
	data, err := client.Agent().Services()
	if err != nil {
		panic(err)
	}
	for key, _ := range data {
		fmt.Println(key)
	}

}

func FilterService() {
	cfg := api.DefaultConfig()
	cfg.Address = "10.0.0.70:8500"

	client, err := api.NewClient(cfg)
	if err != nil {
		panic(err)
	}
	data, err := client.Agent().ServicesWithFilter(`Service == "user-web"`)
	if err != nil {
		panic(err)
	}
	for key, _ := range data {
		fmt.Println(key)
	}
}
func main() {
	//_ = Register("192.168.0.100", 8022, "user-web", []string{"mxshop", "klcc"}, "user-web")
	//AllServices()
	FilterService()
}

```

#### 代码中配置监控检查接口

##### 配置`http`

```go
	Router.GET("/health", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"code":    http.StatusOK,
			"success": true,
		})
	})
```

##### 配置`grpc`

```python
# grpc的服务中 需要增加consul 的 proto 服务注册
```

#### 动态获取端口

```

```

#### grpc 负载均衡

```go
// https://github.com/mbobakov/grpc-consul-resolver

package main

import (
	"PackageTest/grpclb_test/proto"
	"context"
	"fmt"
	_ "github.com/mbobakov/grpc-consul-resolver" // It's important
	"google.golang.org/grpc"
	"log"
)

func main() {
	conn, err := grpc.Dial(
		"consul://10.0.0.70:8500/user-srv?wait=14s&tag=srv]",
		grpc.WithInsecure(),
		grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy": "round_robin"}`),
	)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	for i := 0; i < 10; i++ {
		userSrvClient := proto.NewUserClient(conn)
		rsp, err := userSrvClient.GetUserList(context.Background(), &proto.PageInfo{
			Pn:    1,
			PSize: 2,
		})
		if err != nil {
			panic(err)
		}
		for index, data := range rsp.Data {
			fmt.Println(index, data)
		}
	}
}
```

### nacos 配置中心

```
https://nacos.io/zh-cn/docs/what-is-nacos.html

docker run --name nacos-quick -e MODE=standalone -e JVM_XMS=512m -e JVM_XMX=512m -e JVM_XMN=256m  -p 8848:8848 -p 9848:9848 -d nacos/nacos-server:2.0.2

命名空间
组
配置集


https://github.com/nacos-group/nacos-sdk-python
https://github.com/nacos-group/nacos-sdk-go
```

#### python 操作

```python
import nacos

SERVER_ADDRESSES = "10.0.0.70:8848"
NAMESPACE = "7efaf717-319c-4637-9ae6-1a6c0931fc4c"  # 是 namespace id

client = nacos.NacosClient(SERVER_ADDRESSES, namespace=NAMESPACE, username="nacos", password="nacos")

data_id = "user-srv.json"
group = "dev"
res = client.get_config(data_id, group)
import json

data_json = json.loads(res)
print(data_json)


def test_cb(args):
    print("配置文件产生变化")
    print(args)


client.add_config_watcher(data_id, group, test_cb)
import time

time.sleep(3000)
```

#### go 操作 nacos

```go
package main

import (
	"fmt"
	"github.com/nacos-group/nacos-sdk-go/clients"
	"github.com/nacos-group/nacos-sdk-go/common/constant"
	"github.com/nacos-group/nacos-sdk-go/vo"
	"time"
)

func main() {
	sc := []constant.ServerConfig{
		{
			IpAddr: "10.0.0.70",
			Port:   8848,
		},
	}
	cc := constant.ClientConfig{
		NamespaceId:         "7efaf717-319c-4637-9ae6-1a6c0931fc4c", // 如果需要支持多namespace，我们可以场景多个client,它们有不同的NamespaceId。当namespace是public时，此处填空字符串。
		TimeoutMs:           5000,
		NotLoadCacheAtStart: true,
		LogDir:              "tmp/nacos/log",
		CacheDir:            "tmp/nacos/cache",
		LogLevel:            "debug",
	}
	configClient, err := clients.CreateConfigClient(map[string]interface{}{
		"serverConfigs": sc,
		"clientConfig":  cc,
	})
	if err != nil {
		panic(err)
	}

	content, err := configClient.GetConfig(vo.ConfigParam{
		DataId: "user-web.yaml",
		Group:  "dev"})
	if err != nil {
		panic(err)
	}
	fmt.Println(content)

	err = configClient.ListenConfig(vo.ConfigParam{
		DataId: "user-web.yaml",
		Group:  "dev",
		OnChange: func(namespace, group, dataId, data string) {
			fmt.Println("配置文件产生变化")
			fmt.Println("group:" + group + ", dataId:" + dataId + ", data:" + data)
		},
	})
	time.Sleep(3000 * time.Second)
}
```

## 商品服务
