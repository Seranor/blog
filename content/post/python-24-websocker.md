---
title: python-网络编程
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-24.html
toc: true
---

### OSI 七层模型

![g3kSJQ](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/g3kSJQ.jpg)

<!-- more -->

#### 每层运行常见物理设备

![bevfsC](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/bevfsC.jpg)

#### 七层协议数据传输的封包与解包过程

![1036857-20200415215541847-564448301](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/1036857-20200415215541847-564448301.gif)

#### TCP 三握四挥

![nj8EEE](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/nj8EEE.jpg)

### socket 层

![lrLyh3](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/lrLyh3.jpg)
![ek2QPk](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/ek2QPk.jpg)

**数据传输动图如下：**
![1036857-20200415220004538-1827984001](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/1036857-20200415220004538-1827984001.gif)

#### `socket`工作流程

![krzj1m](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/krzj1m.jpg)

#### `socker()`模块用法

**简单版本**

```python
# 服务端
import socket

# 创建一个socket
server_test = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 绑定地址
server_test.bind(('127.0.0.1', 8080))

# 开始监听, 半连接池数量
server_test.listen(5)

# 接收连接请求,得到客户端连接信息和客户端地址
conn, client_address = server_test.accept()
print(client_address)

# 收消息
data = conn.recv(1024)  # 最大接收字节
print(data)

# 发消息
conn.send(data.upper())

# 关闭连接
conn.close()

# 关闭服务
server_test.close()
```

```python
# 客户端
import socket

client_test = socket.socket(sockrt.AF_INET, socket.SOCK_STREAM)

client_test.connect(("127.0.0.1", 8080))

client_test.send('hello'.encode('utf8'))
data = client_test.recv(1024)
print(data.decode('utf8'))

client_test.close()
```

**处理服务端连接不断**

```python
# 服务端
import socket

server_test = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

server_test.bind(("127.0.0.1", 8080))

server_test.listen(5)

# 这个循环连接的
while True:
    conn, client_address = server_test.accept()
    print(client_address)
    # 这个循环处理连接内容
    while True:
        try:
            data = conn.recv(1024)
            if len(data) == 0:
                break
            print(data)
            conn.send(data.upper())
        except Exception:
            break
    conn.close()
server_test.close()
```

```python
# 客户端
import socket

client_test = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_test.connect(('127.0.0.1', 8080))

while True:
    msg = input('>>>:').strip()
    if len(msg) == 0:
        continue
    client_test.send(msg.encode('utf8'))
    data = client_test.recv(1024)
    print(data.decode('utf8'))
client_test.close()
```

```python
# 服务端套接字函数
s.bind()    绑定(主机,端口号)到套接字
s.listen()  开始TCP监听
s.accept()  被动接受TCP客户的连接,(阻塞式)等待连接的到来

# 客户端套接字函数
s.connect()     主动初始化TCP服务器连接
s.connect_ex()  connect()函数的扩展版本,出错时返回出错码,而不是抛出异常

# 公共用途的套接字函数
s.recv()            接收TCP数据
s.send()            发送TCP数据(send在待发送数据量大于己端缓存区剩余空间时,数据丢失,不会发完)
s.sendall()         发送完整的TCP数据(本质就是循环调用send,sendall在待发送数据量大于己端缓存区剩余空间时,数据不丢失,循环调用send直到发完)
s.recvfrom()        接收UDP数据
s.sendto()          发送UDP数据
s.getpeername()     连接到当前套接字的远端的地址
s.getsockname()     当前套接字的地址
s.getsockopt()      返回指定套接字的参数
s.setsockopt()      设置指定套接字的参数
s.close()           关闭套接字

# 面向锁的套接字方法
s.setblocking()     设置套接字的阻塞与非阻塞模式
s.settimeout()      设置阻塞套接字操作的超时时间
s.gettimeout()      得到阻塞套接字操作的超时时间

# 面向文件的套接字的函数
s.fileno()          套接字的文件描述符
s.makefile()        创建一个与该套接字相关的文件
```

### 粘包问题

远程执行 shell 命令

```python
# 服务端
import socket
import subprocess

server_test = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

server_test.bind(("127.0.0.1", 8080))

server_test.listen(5)

# 这个循环连接的
while True:
    conn, client_address = server_test.accept()
    print(client_address)
    # 这个循环处理连接内容
    while True:
        try:
            cmd = conn.recv(1024)
            if len(cmd) == 0:
                break
            res = subprocess.Popen(cmd,
                                   shell=True,
                                   stdout=subprocess.PIPE,
                                   stderr=subprocess.PIPE)
            res1 = res.stdout.read()
            res2 = res.stderr.read()
            conn.send(res1 + res2)
        except Exception:
            break
    conn.close()
server_test.close()
```

```python
# 客户端
import socket

client_test = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_test.connect(('127.0.0.1', 8080))

while True:
    msg = input('>>>:').strip()
    if len(msg) == 0:
        continue
    client_test.send(msg.encode('utf8'))
    data = client_test.recv(1024)
    print(data.decode('utf8'))  # windows编码是gbk
client_test.close()
```

客户端执行命令

```python
>>>:ps -ef

# 结果发现并没有接收全，还有残留
# 再次输入
>>>:ls

# 发现还是上次遗留的
```

#### 粘包现象

> 只有 TCP 有粘包现象，UDP 永远不会粘包
> 粘包问题主要还是因为接收方不知道消息之间的界限，不知道一次性提取多少字节的数据所造成的

![gXSqAs](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/gXSqAs.jpg)

**两种粘包情况**

1. 发送端需要等缓冲区满才发送出去，造成粘包（发送数据时间间隔很短，数据了很小，会合到一起，产生粘包）
2. 接收方不及时接收缓冲区的包，造成多个包接收（客户端发送了一段数据，服务端只收了一小部分，服务端下次再收的时候还是从缓冲区拿上次遗留的数据，产生粘包）

#### 解决粘包

> 为字节流加上自定义固定长度报头，报头中包含字节流长度，然后一次 send 到对端，对端在接收时，先从缓存中取出定长的报头，然后再取真实数据

**`struct`模块**

```python
import struct, json

header_dic = {'total_size': 10241321431312}  # 用字典存入服务端发送的头信息
header_json = json.dumps(header_dic)  # 将信息序列

print(header_json)  # {"total_size": 10241321431312}
print(len(header_json))  # 30  记录序列化信息后的长度

header_total_size = struct.pack('i', len(header_json))  # 将这个长度转化成固定为 4 的长度


print(len(header_total_size))  # 4 此时就是固定长度
# 客户端处就先接收4字节信息

header_upk = struct.unpack('i', header_total_size)
print(header_upk)  # (30,)  # 将接收到的4字节固定长度继续接收到了30字节信息,继续再拿30字节的信息
```

![QodwhX](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/QodwhX.jpg)

实现:

```python
# 服务端
import socket, struct, json
import subprocess

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('127.0.0.1', 8080))
s.listen(5)
while True:
    conn, address = s.accept()
    print(address)
    while True:
        try:
            cmd = conn.recv(1024)
            if len(cmd) == 0:
                break
            res = subprocess.Popen(cmd, shell=True,
                                   stdout=subprocess.PIPE,
                                   stderr=subprocess.PIPE)
            res1 = res.stdout.read()
            res2 = res.stderr.read()

            header_dic = {'total_size': len(res1) + len(res2)}  # 可以将内容记录到字典内,报头
            header_json = json.dumps(header_dic)  # 序列化字典
            header_bytes = header_json.encode('utf8')  # 转为bytes类型
            header = struct.pack('i', len(header_bytes))  # 得到字典的固定字节长度

            conn.send(header)
            conn.send(header_bytes)  # 发送bytes类型的序列化字典

            conn.send(res1)
            conn.send(res2)
        except Exception:
            break
    conn.close()
```

```python
# 客户端
import socket, struct, json

c = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
c.connect(('127.0.0.1', 8080))
while True:
    msg = input('>>>: ').strip()
    if len(msg) == 0:
        continue
    c.send(msg.encode('utf8'))

    header_bytes_len = struct.unpack('i', c.recv(4))[0]  # 获取序列化后的字典的长度

    header_bytes = c.recv(header_bytes_len)  # 按照序列化后字典的长度接收到序列化的字典
    header_json = header_bytes.decode('utf8')  # 将序列化后的字典解码
    header_dic = json.loads(header_json)  # 发序化得到记录了信息的字典
    total_size = header_dic['total_size']  # 此时就拿到了后面要具体接收多少字节的数

    recv_size = 0
    res = b''
    while recv_size < total_size:
        data = c.recv(1024)  # 每次接收1024字节
        recv_size += len(data)
        res += data  # 最后拼接到 res
    print(res.decode('utf8'))
c.close()
```

### UDP 协议套接字

```python
# 服务端
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

s.bind(('127.0.0.1', 9999))

while True:
    data, address = s.recvfrom(1024)
    print(data, address)
    s.sendto(data.upper(), address)

```

```python
# 客户端
import socket

c = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

while True:
    msg = input('>>>: ').strip()
    c.sendto(msg.encode('utf8'), ('127.0.0.1', 9999))
    res, s_address = c.recvfrom(1024)
    print(res.decode('utf8'), s_address)
```

### `socketserver模块`实现并发

> 基于 tcp 的套接字，关键就是两个循环，一个链接循环，一个通信循环
>
> socketserver 模块中分两大类：server 类（解决链接问题）和 request 类（解决通信问题）

server 类:
![phJIzv](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/phJIzv.jpg)

request 类：
![nFGdy6](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/nFGdy6.jpg)

TCP 并发

```python
# 服务端
import socketserver

class MyRequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            try:
                data = self.request.recv(1024)
                if len(data) == 0:
                    break
                print(data)
                self.request.send(data.upper())
            except Exception:
                break
        self.request.close()


server = socketserver.ThreadingTCPServer(('127.0.0.1', 8080), MyRequestHandler, bind_and_activate=True)
server.serve_forever()
```

```python
# 客户端
import socket

c = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

c.connect(('127.0.0.1', 8080))
while True:
    msg = input('>>>: ').strip()
    if len(msg) == 0:
        continue
    c.send(msg.encode('utf8'))
    data = c.recv(1024)
    print(data.decode('utf8'))

c.close()
```

UDP 并发

```python
# 服务端.py

import socketserver

class MyRequesthanlder(socketserver.BaseRequestHandler):
    # 必须要写一个函数，叫handle的方法，里面放通信循环
    def handle(self):
        # 收到消息，进行解压。第一个值是客户端发来的数据。第二个值是套接字对象，用它来回消息
        data,server = self.request
        # 将收到的消息转大写回复，所有套接字信息都封装进self里了
        server.sendto(data.upper(),self.client_address)


server = socketserver.ThreadingUDPServer(('127.0.0.1',9999),MyRequesthanlder)

server.serve_forever()

# 整体逻辑同上面TCP协议一样

# 客户端.py
from socket import *

client = socket(AF_INET,SOCK_DGRAM)

while True:
    msg = input(">>>>:").strip()
    client.sendto(msg.encode('utf-8'),('127.0.0.1',9999))
    res,server_addr = client.recvfrom(1024)
    print(res.decode('utf-8'))
```
