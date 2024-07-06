---
title: gRPC的使用
lastmod: 2021-10-20T16:43:23+08:00
date: 2021-10-20T11:52:03+08:00
tags:
  - gRPC
categories:
  - gRPC
url: post/grpc.html
toc: true
---

### `protobuf`在 python 中的使用

```python
pip3 install grpcio
pip3 install grpcio-tools

# protobuf_test 和 proto 都是定义的包
# 文件位置 protobuf_test/proto/hello.proto  该文件可以删除的 只是为了生成两个文件
```

<!-- more -->

```protobuf
syntax = "proto3";

message HelloRequest {
    string name = 1;  // name 表示名称  name 的编号是 1
}
```

```python
python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. hello.proto

此时会生成两个文件
  hello_pb2.py
  hello_pb2_grpc.py
```

` protobuf_test/client.py`

```python
from protobuf_test.proto import hello_pb2

# 生成的pb文件不要修改
request = hello_pb2.HelloRequest()
request.name = "lzj"
res_str = request.SerializeToString()
print(res_str)  # b'\n\x03lzj'

request2 = hello_pb2.HelloRequest()
request2.ParseFromString(res_str)
print(request2.name)


# json 和 protobuf 压缩对比
res_json = {
    "name": "lzj"
}
import json

print(len(json.dumps(res_json)))
print(len(res_str))
```

### gRPC 使用体验

#### python 下

##### `grpc_hello/proto/helloworld.proto`

```protobuf
syntax = "proto3";

service Greeter {
    rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

```shell
# 执行命令生成
cd grpc_hello/proto/
python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. helloworld.proto
```

##### `grpc_hello/server.py`

```python
import grpc
from concurrent import futures
from grpc_hello.proto import helloworld_pb2_grpc, helloworld_pb2

class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return helloworld_pb2.HelloReply(message=f'你好,{request.name}')


if __name__ == '__main__':
    # 实例化server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 注册逻辑到server中
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

##### `grpc_hello/client.py`

```python
from grpc_hello.proto import helloworld_pb2, helloworld_pb2_grpc
import grpc

if __name__ == '__main__':
    with grpc.insecure_channel("localhost:50051") as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        rsp: helloworld_pb2.HelloReply = stub.SayHello(helloworld_pb2.HelloRequest(name="lzj"))
        print(rsp.message)
```

```python
# grpc_hello/proto/helloworld_pb2_grpc.py 文件中如下修改导入模块路径
# import helloworld_pb2 as helloworld__pb2
from grpc_hello.proto import helloworld_pb2 as helloworld__pb2
```

#### golang 下

##### 下载软件

```
# 下载对应平台
https://github.com/protocolbuffers/protobuf/releases

# 设置环境变量
```

##### 下载依赖

```golang
go get github.com/golang/protobuf/protoc-gen-go

检查在 gopath 下的 bin 下是否有 protoc-gen-go 文件
```

##### `grpc/test/proto/helloworld.proto`

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
  rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

##### 执行命令生成

```
protoc -I . helloworld.proto --go_out=plugins=grpc:.
# 生成的就一个 go 文件

# 可能缺少依赖需要下载
go get google.golang.org/grpc
go get google.golang.org/grpc/codes
go get google.golang.org/grpc/status
go get google.golang.org/protobuf/reflect/protoreflect
go get google.golang.org/protobuf/runtime/protoimpl
```

##### server.go

```go
package main

import (
	"PackageTest/grpc_test/proto"
	"context"
	"google.golang.org/grpc"
	"net"
)

type Server struct{}

func (s Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	return &proto.HelloReply{
		Message: "Hello " + request.Name,
	}, nil
}
func main() {
	g := grpc.NewServer()
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:8080")
	if err != nil {
		panic("failed to listen " + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to grpc" + err.Error())
	}
}
```

##### client.go

```go
package main

import (
	"PackageTest/grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:50051", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	c := proto.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto.HelloRequest{Name: "lzj"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```

### 流模式

#### stream.proto

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
  rpc GetStream(StreamReqData) returns (stream StreamResData) {}  // 服务端流模式
  rpc PutStream(stream StreamReqData) returns (StreamResData) {}  // 客户端流模式
  rpc AllStream(stream StreamReqData) returns (stream StreamResData) {}  // 双向流模式
}

message StreamReqData{
  string data = 1;
}

message StreamResData{
  string data = 1;
}
```

#### server.go

```go
package main

import (
	"fmt"
	"google.golang.org/grpc"
	"net"
	"sync"
	"time"
)
import "PackageTest/stream_grpc_test/proto"

const PORT = ":4000"

type server struct{}

func (s *server) GetStream(req *proto.StreamReqData, res proto.Greeter_GetStreamServer) error {
	i := 0
	for {
		i++
		_ = res.Send(&proto.StreamResData{
			Data: fmt.Sprintf("%v", time.Now().Unix()),
		})
		time.Sleep(time.Second)
		if i > 10 {
			break
		}
	}
	return nil
}

func (s *server) PutStream(cliStr proto.Greeter_PutStreamServer) error {
	for {
		if a, err := cliStr.Recv(); err != nil {
			fmt.Println(err)
			break
		} else {
			fmt.Println(a.Data)
		}
	}
	return nil
}

func (s *server) AllStream(allStr proto.Greeter_AllStreamServer) error {
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		defer wg.Done()
		for {
			data, _ := allStr.Recv()
			fmt.Println("收到客户端信息：" + data.Data)
		}
	}()
	go func() {
		defer wg.Done()
		for {
			_ = allStr.Send(&proto.StreamResData{Data: "我是服务器"})
			time.Sleep(time.Second)
		}
	}()
	wg.Wait()
	return nil
}

func main() {
	s := grpc.NewServer()
	proto.RegisterGreeterServer(s, &server{})
	lis, err := net.Listen("tcp", PORT)
	if err != nil {
		panic(err)
	}
	err = s.Serve(lis)
	if err != nil {
		panic(err)
	}
}
```

#### client.go

```go
package main

import (
	"PackageTest/stream_grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"sync"
	"time"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:4000", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	// 服务端流模式
	c := proto.NewGreeterClient(conn)
	res, _ := c.GetStream(context.Background(), &proto.StreamReqData{Data: "慕课网"})
	for {
		a, err := res.Recv()
		if err != nil {
			fmt.Println(err)
			break
		}
		fmt.Println(a.Data)
	}
	// 客户端流模式
	putS, _ := c.PutStream(context.Background())
	i := 0
	for {
		i++
		_ = putS.Send(&proto.StreamReqData{
			Data: fmt.Sprintf("慕课网%d", i),
		})
		time.Sleep(time.Second)
		if i > 10 {
			break
		}
	}
	// 双向流模式
	allStr, _ := c.AllStream(context.Background())
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		defer wg.Done()
		for {
			data, _ := allStr.Recv()
			fmt.Println("收到服务端信息：" + data.Data)
		}
	}()
	go func() {
		defer wg.Done()
		for {
			_ = allStr.Send(&proto.StreamReqData{Data: "我是客户端"})
			time.Sleep(time.Second)
		}
	}()
	wg.Wait()
}
```

### protobuf 文件

```protobuf
message StreamReqData{
  string data = 1;
  repeated int32 id =2;
  // 数组 列表类型 在定义的时候提前定义  或者使用时 python使用 列表的方法操作
}
```

#### `option参数`

```go
option go_package = "../proto";
option go_package = "common/stream/proto/v1";

可以指定位置，将生成的包放在固定位置，公共使用
```

#### proto 文件不同步

```
PackageTest/gprc_proto_test/proto/hello.proto
PackageTest/gprc_proto_test/client.go
cd PackageTest/gprc_proto_test/proto/
protoc -I . helloworld.proto --go_out=plugins=grpc:.


grpc_proto_test/proto/hello.proto
grpc_proto_test/server.py
cd grpc_proto_test/proto/
python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. helloworld.proto
```

##### `hello.proto`

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply){}
}

message HelloRequest{
  string name = 1;
  string url = 2;
}

message HelloReply{
  string message = 1;
}
```

##### `server.py`

```python
import grpc
from concurrent import futures
from grpc_proto_test.proto import hello_pb2,hello_pb2_grpc


class Greeter(hello_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return hello_pb2.HelloReply(message=f'姓名:{request.name},url:{request.url}')


if __name__ == '__main__':
    # 实例化server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 注册逻辑到server中
    hello_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("0.0.0.0:50053")
    server.start()
    server.wait_for_termination()
```

##### `client.go`

```go
package main

import (
	"PackageTest/gprc_proto_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"log"
)

func main() {
	conn, err := grpc.Dial("localhost:50053", grpc.WithInsecure())
	if err != nil {
		log.Printf("连接失败：[%v]\n,err")
		return
	}
	defer conn.Close()
	client := proto.NewGreeterClient(conn)
	rsp, _ := client.SayHello(context.Background(), &proto.HelloRequest{
		Name: "lzj",
		Url:  "https://klcc.cc",
	})
	fmt.Println(rsp.Message)
}

```

##### hello.proto 出现不同步

```protobuf
// go里面
message HelloRequest{
  string name = 1;
  string url = 2;
}

// py里面
message HelloRequest{
  string url = 1;
  string name = 2;
}

// python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. helloworld.proto
```

```python
重新运行server.py

客户端连接出现 name 和 url 的值反了
姓名:https://klcc.cc,url:lzj

proto 文件一定要保持一致
```

#### 引入另一个 proto 文件

##### `base.proto`

```protobuf
syntax = "proto3";
option go_package = "../proto";
message Pong {
  string id = 1;
}
```

##### `hello.proto`

```protobuf
syntax = "proto3";
import "base.proto";
import "google/protobuf/empty.proto";
option go_package = "../proto";

service Greeter {
  rpc SayHello(HelloRequest) returns (HelloReply) {}
  rpc Ping(google.protobuf.Empty) returns (Pong) {}

}

message HelloRequest {
  string url = 2;
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

```
import "google/protobuf/empty.proto"; 如果是这个报错 在IDEA中下载protobuf buffer插件解决
mport "base.proto"; 这个是红色的应该没问题

在生成文件时两个文件都需要生成

python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. hello.proto
python3 -m grpc_tools.protoc --python_out=. --grpc_python_out=. -I. base.proto

protoc -I . base.proto --go_out=plugins=grpc:.
protoc -I . hello.proto --go_out=plugins=grpc:.
```

#### 嵌套`message`

```protobuf
syntax = "proto3";

message Pong {
    string id = 1;
}
```

```protobuf
syntax = "proto3";
import "base.proto";
import "google/protobuf/empty.proto";
option go_package = "../proto";

service Greeter {
  rpc SayHello(HelloRequest) returns (HelloReply) {}
  rpc Ping(google.protobuf.Empty) returns (Pong) {}

}

message HelloRequest {
  string url = 2;
  string name = 1;
}

message HelloReply {
  string message = 1;
  message Result{
    string name = 1;
    string url = 2 ;
  }
  repeated Result data = 2;
}
```

```
生成文件后
python
调用Result
from grpc_proto_test.proto.hello_pb2 import HelloReply
result = HelloReply.Result()
调用Pong
from grpc_proto_test.proto.base_pb2 import Pong
或者 pong = hello_pb2.base__pb2.Pong

golang
调用Result
	HelloReply_Result
调用Pong
生成的两个包都在 proto 包下  因此导入后就可以直接调用 Pong

```

#### 枚举类型

##### `hello.proto`

```protobuf
syntax = "proto3";
option go_package = "../proto_bak";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply){}
}

enum Gender {
  MALE = 0;
  FEMALE = 1;
}

message HelloRequest{
  string name = 1;
  string url = 2;
  Gender g = 3;
}

message HelloReply{
  string message = 1;
}
```

##### `client.go`

```go
package main

import (
	"PackageTest/gprc_proto_test/proto_bak"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"log"
)

func main() {
	conn, err := grpc.Dial("localhost:50053", grpc.WithInsecure())
	if err != nil {
		log.Printf("连接失败：[%v]\n,err")
		return
	}
	defer conn.Close()
	client := proto_bak.NewGreeterClient(conn)
	rsp, _ := client.SayHello(context.Background(), &proto_bak.HelloRequest{
		Url:  "https://klcc.cc",
		Name: "lzj",
		G:    proto_bak.Gender_FEMALE,
	})
	fmt.Println(rsp.Message)
}
```

##### `server.py`

```python
import grpc
from concurrent import futures
from grpc_proto_test2.proto import hello_pb2, hello_pb2_grpc

class Greeter(hello_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return hello_pb2.HelloReply(message=f'姓名:{request.name},url:{request.url},性别：{request.g}')


if __name__ == '__main__':
    # 实例化server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 注册逻辑到server中
    hello_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("0.0.0.0:50053")
    server.start()
    server.wait_for_termination()

```

#### `map类型`

```protobuf
syntax = "proto3";
option go_package = "../proto_bak";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply){}
}
enum Gender {
  MALE = 0;
  FEMALE = 1;
}
message HelloRequest{
  string name = 1;
  string url = 2;
  Gender g = 3;
  map<string, string> mp = 4;  // go --> map python --> dict
}

message HelloReply{
  string message = 1;
}
```

#### `timestamp`

```protobuf
syntax = "proto3";
import "google/protobuf/timestamp.proto";
option go_package = "../proto_bak";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply){}
}
enum Gender {
  MALE = 0;
  FEMALE = 1;
}
message HelloRequest{
  string name = 1;
  string url = 2;
  Gender g = 3;
  map<string, string> mp = 4;
  google.protobuf.Timestamp addTime = 5;
}

message HelloReply{
  string message = 1;
}
```

```go
package main

import (
	"PackageTest/gprc_proto_test/proto_bak"
	"context"
	"fmt"
	"google.golang.org/grpc"
	timestamppb "google.golang.org/protobuf/types/known/timestamppb"
	"log"
	"time"
)

func main() {
	conn, err := grpc.Dial("localhost:50053", grpc.WithInsecure())
	if err != nil {
		log.Printf("连接失败：[%v]\n,err")
		return
	}
	defer conn.Close()
	client := proto_bak.NewGreeterClient(conn)
	rsp, _ := client.SayHello(context.Background(), &proto_bak.HelloRequest{
		Url:     "https://klcc.cc",
		Name:    "lzj",
		G:       proto_bak.Gender_MALE,
		Mp:      map[string]string{"hobby": "Football"},
		AddTime: timestamppb.New(time.Now()),
	})
	fmt.Println(rsp.Message)
}
```

### `python`结合`asyncio`

```python
pip3 install grpclib

python3 -m grpc_tools.protoc --python_out=. --grpclib_python_out=. -I. hello.proto
```

### 控制 metadata

#### go

```go
MD 类型实际上是map key 是string value 是string 类型的slice
type MD map[string][]string

// 两种方式
md := metadata.New(map[string]string{"key":"value"})
md := metadata.Pairs(
  "key1","value1",
  "key2","value2",
)

// 发送
md := metadata.Pairs("key1","value1")
ctx := metadata.NewOutgoingContext(context.Background(),md)
response, err := client.SomeRPC(ctx,someRequest)

// 接收
func (s *server) SomeRPC(ctx context.Context, in *pb.SomeRequest) (*pb.SomeResponse,error){
  md, ok := metadata.FormIncomingContext(ctx)
}
```

`helloworld.proto`

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
  rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

`server.go`

```go
package main

import (
	"PackageTest/grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
	"net"
)

type Server struct{}

func (s Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	md, ok := metadata.FromIncomingContext(ctx)
	if ok {
		fmt.Println("get metadata error")
	}
	if nameSlice, ok := md["name"]; ok {
		fmt.Println(nameSlice)
		for i, e := range nameSlice {
			fmt.Println(i, e)
		}
	}
	//for key, val := range md {
	//	fmt.Println(key, val)
	//}

	return &proto.HelloReply{
		Message: "Hello " + request.Name,
	}, nil
}
func main() {
	g := grpc.NewServer()
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:8080")
	if err != nil {
		panic("failed to listen " + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to grpc" + err.Error())
	}
}
```

`client.go`

```go
package main

import (
	"PackageTest/grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:8080", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	c := proto.NewGreeterClient(conn)

	//md := metadata.Pairs("timestamp", time.Now().Format(timestampFormat))
	md := metadata.New(map[string]string{
		"name":     "lzj",
		"password": "123456",
	})
	ctx := metadata.NewOutgoingContext(context.Background(), md)

	r, err := c.SayHello(ctx, &proto.HelloRequest{Name: "lzj"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}

```

#### python

```protobuf
syntax = "proto3";

service Greeter {
    rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

`server.py`

```python
import grpc
from concurrent import futures
from grpc_metadata.proto import helloworld_pb2_grpc, helloworld_pb2


class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        for key, value in context.invocation_metadata():
            print(key, value)
        return helloworld_pb2.HelloReply(message=f'你好,{request.name}')


if __name__ == '__main__':
    # 实例化server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 注册逻辑到server中
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

`client.py`

```python
from grpc_metadata.proto import helloworld_pb2, helloworld_pb2_grpc
import grpc

if __name__ == '__main__':
    with grpc.insecure_channel("localhost:50051") as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        response, call = stub.SayHello.with_call(
            helloworld_pb2.HelloRequest(name="lzj"),
            metadata=(
                ("name", "lzj"),
                ("password", "123456")
            )
        )
        print(response.message)
```

### 拦截器

#### go

`helloworld.proto`

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
  rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

`server.go`

```go
package main

import (
	"PackageTest/grpc_interpretor/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"net"
)

type Server struct{}

func (s Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	return &proto.HelloReply{
		Message: "Hello " + request.Name,
	}, nil
}

func main() {

	interceptor := func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
		fmt.Println("接收到了一个新的请求")
		res, err := handler(ctx, req)
		fmt.Println("请求已经完成")
		return res, err
	}

	opt := grpc.UnaryInterceptor(interceptor)
	g := grpc.NewServer(opt)
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:8080")
	if err != nil {
		panic("failed to listen " + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to grpc" + err.Error())
	}
}
```

`client.go`

```go
package main

import (
	"PackageTest/grpc_interpretor/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"time"
)

func main() {
	interceptor := func(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
		strar := time.Now()
		err := invoker(ctx, method, req, reply, cc, opts...)
		fmt.Printf("耗时 %s\n", time.Since(strar))
		return err
	}
	var opts []grpc.DialOption
	opts = append(opts, grpc.WithInsecure())
	opts = append(opts, grpc.WithUnaryInterceptor(interceptor))
	conn, err := grpc.Dial("127.0.0.1:8080", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	c := proto.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto.HelloRequest{Name: "lzj"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```

#### python

`helloworld.ptoto`

```protobuf
syntax = "proto3";

service Greeter {
    rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

`server.py`

```python
import grpc
from concurrent import futures
from grpc_interceptor.proto import helloworld_pb2_grpc, helloworld_pb2


class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        for key, value in context.invocation_metadata():
            print(key, value)
        return helloworld_pb2.HelloReply(message=f'你好,{request.name}')


class LogInterceptor(grpc.ServerInterceptor):
    def intercept_service(self, continuation, handler_call_details):
        print("请求开始")
        rsp = continuation(handler_call_details)
        print("请求结束")
        return rsp


if __name__ == '__main__':
    # 实例化server
    interceptor = LogInterceptor()
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10), interceptors=(interceptor,))
    # 注册逻辑到server中
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

`client.py`

```python
from grpc_interceptor.proto import helloworld_pb2, helloworld_pb2_grpc
import grpc


class DefaultInterceptor(grpc.UnaryUnaryClientInterceptor):
    def intercept_unary_unary(self, continuation, client_call_details, request):
        from datetime import datetime
        start = datetime.now()
        rsp = continuation(client_call_details, request)
        print((datetime.now() - start).microseconds / 1000)
        return rsp


if __name__ == '__main__':
    default_interceptor = DefaultInterceptor()
    with grpc.insecure_channel("localhost:50051") as channel:
        interceptor_channel = grpc.intercept_channel(channel, default_interceptor)
        stub = helloworld_pb2_grpc.GreeterStub(interceptor_channel)
        response, call = stub.SayHello.with_call(
            helloworld_pb2.HelloRequest(name="lzj"),
            metadata=(
                ("name", "lzj"),
                ("password", "123456")
            )
        )

        print(response.message)
```

### `auth`认证

`helloworld.proto`

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
  rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

`server.go`

```go
package main

import (
	"PackageTest/grpc_interpretor/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"net"
)

type Server struct{}

func (s Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	return &proto.HelloReply{
		Message: "Hello " + request.Name,
	}, nil
}

func main() {

	interceptor := func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
		fmt.Println("接收到了一个新的请求")
		md, ok := metadata.FromIncomingContext(ctx)
		fmt.Println(md)
		if !ok {
			fmt.Println("get metadata error")
			return resp, status.Error(codes.Unauthenticated, "无token认证信息")
		}
		var (
			appid  string
			appkey string
		)
		if va1, ok := md["appid"]; ok {
			appid = va1[0]
		}
		if va1, ok := md["appkey"]; ok {
			appkey = va1[0]
		}

		if appid != "23421322" || appkey != "this is key" {
			return resp, status.Error(codes.Unauthenticated, "无token认证信息")
		}

		res, err := handler(ctx, req)
		fmt.Println("请求已经完成")
		return res, err
	}
	opt := grpc.UnaryInterceptor(interceptor)
	g := grpc.NewServer(opt)
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:8080")
	if err != nil {
		panic("failed to listen " + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to grpc" + err.Error())
	}
}
```

`client.go`

```go
package main

import (
	"PackageTest/grpc_interpretor/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
)

type customCredential struct{}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appid":  "23421322",
		"appkey": "this is key",
	}, nil
}

func (c customCredential) RequireTransportSecurity() bool {
	return false
}

func main() {
	// // 方式一
	//interceptor := func(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	//	strar := time.Now()
	//	md := metadata.New(map[string]string{
	//		"appid":  "23421322",
	//		"appkey": "this is key",
	//	})
	//	ctx = metadata.NewOutgoingContext(context.Background(), md)
	//	err := invoker(ctx, method, req, reply, cc, opts...)
	//	fmt.Printf("耗时 %s\n", time.Since(strar))
	//	return err
	//}

	var opts []grpc.DialOption
	opts = append(opts, grpc.WithInsecure())
	//opts = append(opts, grpc.WithUnaryInterceptor(interceptor))
	opts = append(opts, grpc.WithPerRPCCredentials(customCredential{}))
	conn, err := grpc.Dial("127.0.0.1:8080", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	c := proto.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto.HelloRequest{Name: "lzj"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```

### 验证器

```
protoc-gen-validate
```

### 错误处理

错误状态： `https://github.com/grpc/grpc/blob/master/doc/statuscodes.md`

```protobuf
syntax = "proto3";
option go_package = "../proto";
service Greeter {
    rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

#### python

`server`

```python
import grpc
from concurrent import futures
from grpc_error_test.proto import helloworld_pb2_grpc, helloworld_pb2


class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        context.set_code(grpc.StatusCode.NOT_FOUND)
        context.set_details("记录不存在")
        return helloworld_pb2.HelloReply(message=f'你好,{request.name}')


if __name__ == '__main__':
    # 实例化server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 注册逻辑到server中
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

`client`

```python
from grpc_error_test.proto import helloworld_pb2, helloworld_pb2_grpc
import grpc

if __name__ == '__main__':
    with grpc.insecure_channel("localhost:50051") as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        try:
            rsp: helloworld_pb2.HelloReply = stub.SayHello(helloworld_pb2.HelloRequest(name="lzj"))
            print(rsp.message)
        except grpc.RpcError as e:
            d = e.details()
            print(d)
            status_code = e.code()
            print(status_code.name)
            print(status_code.value)
```

#### go

`server`

```go
package main

import (
	"PackageTest/grpc_error_test/proto"
	"context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"net"
)

type Server struct{}

func (s Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	return nil, status.Errorf(codes.NotFound, "%s 记录不存在 ", request.Name)
	//return &proto.HelloReply{
	//	Message: "Hello " + request.Name,
	//}, nil
}
func main() {
	g := grpc.NewServer()
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:50051")
	if err != nil {
		panic("failed to listen " + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to grpc" + err.Error())
	}
}
```

`client`

```go
package main

import (
	"PackageTest/grpc_error_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/status"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:50051", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	c := proto.NewGreeterClient(conn)
	_, err = c.SayHello(context.Background(), &proto.HelloRequest{Name: "lzj"})
	if err != nil {
		st, ok := status.FromError(err)
		if !ok {
			panic("解析error 失败")
		}
		fmt.Println(st.Message())
		fmt.Println(st.Code())
	}
	//fmt.Println(r.Message)
}
```

```
grpc 以及处理了语言之间错误的不同方式
```

### 超时机制

`server.py`

```python
import grpc
from concurrent import futures
from grpc_error_test.proto import helloworld_pb2_grpc, helloworld_pb2


class Greeter(helloworld_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        # context.set_code(grpc.StatusCode.NOT_FOUND)
        # context.set_details("记录不存在")
        import time
        time.sleep(5)
        return helloworld_pb2.HelloReply(message=f'你好,{request.name}')


if __name__ == '__main__':
    # 实例化server
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 注册逻辑到server中
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    # 启动
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

`client.py`

```python
from grpc_error_test.proto import helloworld_pb2, helloworld_pb2_grpc
import grpc

if __name__ == '__main__':
    with grpc.insecure_channel("localhost:50051") as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        try:
            rsp: helloworld_pb2.HelloReply = stub.SayHello(helloworld_pb2.HelloRequest(name="lzj"), timeout=3)
            print(rsp.message)
        except grpc.RpcError as e:
            d = e.details()
            print(d)
            status_code = e.code()
            print(status_code.name)
            print(status_code.value)
```

`client.go`

```go
package main

import (
	"PackageTest/grpc_error_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/status"
	"time"
)

func main() {
	conn, err := grpc.Dial("127.0.0.1:50051", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()
	c := proto.NewGreeterClient(conn)
	ctx, _ := context.WithTimeout(context.Background(), time.Second*3)
	_, err = c.SayHello(ctx, &proto.HelloRequest{Name: "lzj"})
	if err != nil {
		st, ok := status.FromError(err)
		if !ok {
			panic("解析error 失败")
		}
		fmt.Println(st.Message())
		fmt.Println(st.Code())
	}
	//fmt.Println(r.Message)
}
```
