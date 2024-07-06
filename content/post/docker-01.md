---
title: Docker使用
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - Docker
categories:
  - Docker
url: post/docker-01.html
toc: true
---

## docker 简介

![moC1KB](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/moC1KB.jpg)

<!-- more -->

### 特点

- 灵活性：即使是最复杂的应用也可以集装箱化
- 轻量级：容器利用并共享主机内核
- 可互换：您可以即时部署更新和升级
- 便携式：您可以在本地构建，部署到云，并在任何地方运行
- 可扩展：您可以增加并自动分发容器副本
- 可堆叠：您可以垂直和即时堆叠服务

### 概念

镜像

```bash
是一个只读的模板，可以在模板上添加额外的功能
```

容器

```bash
是镜像的可运行实例，本质是进程，但是有独立的命名空间，有独立的网络配置，文件系统，进程空间等等... 相当于在一个独立的隔离环境
```

`registry `

```bash
存储镜像的一个仓库，Docker hub官网提供了一个公共的仓库，而且也可以单独的运行一个私有仓库
```

### 容器和虚拟机

```bash
虚拟机：是一个独立的操作系统，占用资源较多，完全隔离，所以比较安全
容器：与宿主机共享主机的内核，是一个独立的进程，不占用其他执行文件，比较轻量
```

![docker container vs vm](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/container--vs-vm.png)

### docker 的底层

[Namespace 和 CGroup](https://klcc.cc/4a318272.html)

UnionFS

```bash
docker镜像是由一系列的层组成，每层代表 Dockerfile 中的一条指令

镜像中每层都是只读的，在运行容器时，就可以在基础层上添加新的可写层，也就是通常说的容器层，对于运行中的容器所做的更改都会写入容器层

容器与镜像之间的主要区别就是在镜像之上有一个可写层，在容器中的所有操作都会存储在这个容器层中，删除容器后，容器层也会被删除，但是镜像不会变化
```

![docker filesystem](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/docker-filesystems.png)

![container layers](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/container-layers.jpg)

![sharing layers](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/sharing-layers.jpg)

### docker 架构

```bash
Docker 使用 C/S体系的架构，Docker客户端与Docker守护进程（Dockerd）通信，Docker守护进程负责构建，运行和分发 Docker 容器。Docker 客户端和守护进程可以在同一个系统上运行，也可以将 Docker 客户端连接到远程 Docker 守护进程。Docker 客户端和守护进程使用 REST API 通过 UNIX 套接字或网络接口进行通信
```

![docker structrue](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/docker-structrue.png)

## 安装

```bash
# 参考文档
# https://docs.docker.com/install/linux/docker-ce/centos/
# https://www.kuboard.cn/install/install-k8s.html

# 卸载旧版本
yum remove docker docker-client docker-client-latest docker-common \
docker-latest docker-latest-logrotate docker-logrotate docker-engine

# 安装最新docker-ce
yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io

# 安装指定版本
yum list docker-ce --showduplicates | sort -r
yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io -y

# 启动
systemctl start docker
systemctl enable docker

# 查看信息
docker info

# 镜像加速、更改Docker根目录
mkdir -p /data/docker  # 挂载到新的硬盘上
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {"max-size": "100m"},
  "registry-mirrors" : ["https://ot2k4d59.mirror.aliyuncs.com/"],
  "graph": "/data/docker"
}
EOF

systemctl daemon-reload
systemctl restart docker
docker info

# 测试
docker run hello-world

#卸载
yum remove docer-ce
rm -rf /var/lib/docker

# 修改内核参数，解决以下报错
# WARNING: bridge-nf-call-iptables is disabled
# WARNING: bridge-nf-call-ip6tables is disabled
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf

sysctl --system
sysctl -p

#内核调整出现错误
sysctl -p
net.ipv4.ip_forward = 1
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-ip6tables: No such file or directory
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables: No such file or directory

#解决办法
modprobe br_netfilter
```

## 基本操作

### 镜像管理

```bash
# 拉取镜像
docker pull ubuntu:18.04

# 推送镜像
docker push

# 查看本地镜像
docker images
docker image ls

# 删除指定镜像 可以指定镜像名字或者镜像ID
docker image rm xxx
docker rmi -f   xxx

# 打标签
docker tag nginx nginx:test

# 导出镜像
docker image save nginx > /tmp/nginx.tar.gz

# 导入镜像
docker load <  /tmp/nginx.tar.gz

#查看镜像的详情
docker image inspect nginx:latest
```

### 容器管理

```bash
# 启动容器
docker  run -itd --name bs busybox:latest

# 创建一个容器并进入并在退出容器是自动删除
docker run -it --rm  ubuntu:18.04  /bin/bash

# 进入 bs 容器内 屏幕共享的方式(两个终端都是用该命令进入就会同步操作)
docker  attach bs
ctrl + p ctrl + q  	# 退出容器后，容器不死

# 指定命令终端进入
docker exec -it bs sh

# 查看容器
docker ps        # 正在运行的
docker ps -a     # 查看所有容器(包含已退出的)
docker ps -a -q  # 查看所有容器id

# 停止所有容器
docker stop `docker ps -a -q`

# 删除所有容器
docker rm -f `docker ps -a -q`

# 根据容器ID强制删除容器
docker rm -f 562faccc3106

# 启动容器
## 这个容器不是后台运行
docker run --name test ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"

# 这个容器会在后台运行
docker run -d --name test ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"

# 查看容器日志
docker logs -f test

# 查看容器占用资源情况
docker stats

# docker run 参数
-it                     # 这是两个参数 -i 交互操作 -t 终端
-d                      # 后台运行
--name                  # 设置容器名
--restart=always        # 容器自启
-h x.x.x.x		          # 容器主机名
--dns x.x.x.x           # 容器dns
--dns-search
--add-host hostname:IP	# hostname和主机ip的关系 hosts文件
--rm		                # 服务停止自动删除
```

### 网络管理

#### 端口暴露

```bash
# 启动一个nginx容器
docker run --name webserver -d nginx

# 查看容器情况
docker ps

# 查看该容器的IP
docker inspect webserver |grep IPAddress
# 会得到 "IPAddress": "172.17.0.2"
# 访问这个地址的80端口 可以访问到nginx页面
curl 172.17.0.2

# 删除该容器
docker rm -f webserver

# 将宿主机的8080端口和容器的80端口进行绑定
docker run --name webserver -d -p 8080:80 nginx

# 此时就可以访问宿主机的8080端口访问到容器的80端口
curl localhost:8080
```

#### Bridge 模式

docker 启动会产生一个 docker0 的网桥，启动的容器会连接到这个网桥上，类似物理交换机

```bash
# 可以看到docker0这个网卡
ip address

# 可以使用 brctl 命令查看
yum install -y bridge-utils
brctl show
```

bridge 模式是 docker 的默认网络模式，使用`docker run -p`时，实际上是通过 iptables 做了`DNAT`规则，实现端口转发功能。可以使用 iptables -t nat -vnL 查看

![docker network bridge](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/docker-netework-bridge.jpeg)

```bash
brctl show
docker run -tid --net=bridge --name docker_bri busybox top
# interfaces
brctl show

# 查看容器的ip、路由
docker exec docker_bri ifconfig -a
docker exec docker_bri route -n

# 查看宿主机上的的iflink情况
ip link show

# 查看容器的iflink文件，由此可以看到是对应的
docker exec docker_bri cat /sys/class/net/eth0/iflink
```

![image-20220114171735473](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220114171735473.png)

#### 自定义网络模式

可以通过自定义的 Docker 网络来连接多个容器，而不是使用`--link`命令

##### `--link`方式

```bash
# 再创建一个容器加入到上面的 docker_bri容器建立连接
docker run -tid --link docker_bri --name docker_bri1 busybox top

# 在docker_bri1内可以ping通docker_bri
docker exec docker_bri1 ping docker_bri

# 但是反过来就不可以
docker exec docker_bri ping docker_bri1

# 其实 --link 操作就是在创建容器的时候将 ip 容器名称写入到了 /etc/hosts内，而最开始的那个容器并没有做解析
docker exec docker_bri cat /etc/hosts
docker exec docker_bri1 cat /etc/hosts
```

![image-20220114172747607](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220114172747607.png)

##### 自定义网络方式

```python
# 查看网络
docker network list

# 创建网络
docker network create -d bridge my-net

# 启动两个容器加入到 my-net 网络
docker run -itd --name busybox1 --network my-net busybox top
docker run -itd --name busybox2 --network my-net busybox top

# 测试互ping
docker exec busybox1 ping busybox2
docker exec busybox2 ping busybox1
```

#### Host 模式

使用 host 模式，这个容器不会获得一个独立的`Network Namespace`，而是和宿主机共用一个 Network Namespace，和宿主机共用端口。但是其他资源依旧是隔离的

```bash
# 在运行时使用下面参数即可
--net=host
```

![docker host network](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/docker-network-host.jpeg)

#### Container 模式

新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享

新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等

其他资源依旧隔离，文件系统、进程列表等

```bash
# 使用参数
--net=container:目标容器名

# Kubernetes 里面的 Pod 中容器之间就是通过 Container 模式链接到 pause 容器上面的，所以容器直接可以通过 localhost 来进行访问
```

![docker container network](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/docker-network-container.jpeg)

#### None 模式

None 模式不会为容器创建任何的网络配置，没有路由、IP 等信息

```bash
--network none
```

![docker none network](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/docker-network-none.jpeg)

### 数据持久化

#### 数据卷

```bash
# 创建数据卷
docker volume create my-vol

# 查看数据卷
docker volume ls
docker volume inspect my-vol

# 两种用法
docker run -d -p 8080:80 --name web -v my-vol:/usr/share/nginx/html nginx
docker run -d -p 8080:80 --name web --mount source=my-vol,target=/usr/share/nginx/html nginx

# 挂载之后查看宿主机内的该卷的内容
ls /data/docker/volumes/my-vol/_data/

# 访问容器的nginx服务
curl localhost:8080

# 替换掉默认nginx首页内容
echo "Hello Docker" > /data/docker/volumes/my-vol/_data/index.html

# 此时访问就是修改后的内容
curl localhost:8080

# 清理无主的数据卷
docker volume prune
```

#### 挂载主机目录

```bash
# 将宿主机的目录挂载到容器中,参数
-v
--mount

# 使用
# 模拟宿主机内容
echo 'hello' > /mnt/test.txt
# 运行容器并检查容器内挂载的内容
docker run -it -v /mnt/:/usr/mnt busybox /bin/sh
cat /usr/mnt/test.txt

# 注意: 冒号前为宿主机目录，必须为绝对路径，冒号后为容器内挂载的路径

# 挂载权限
默认挂载的路径权限为读写，如果指定为只读使用 ro ,例如 -v /mnt/:/usr/mnt:ro
容器目录不可以为相对路径
宿主机目录如果不存在，则会自动生成

挂载宿主机已存在目录后，在容器内对其进行操作，报“Permission denied”。可通过两种方式解决：
1.关闭selinux
    临时关闭：setenforce 0
    永久关闭：修改  /etc/sysconfig/selinux 文件，将 SELINUX 的值设置为disabled

2.以特权方式启动容器
    指定 --privileged 参数，如：
    docker run -it --privileged=true -v /test:/soft centos /bin/bash
```

## Dockerfile

镜像的定制实际上就是定制镜像的每一层所添加的配置、文件等信息

当在容器内添加或修改了一些文件后，可以通过`docker commit`命令来生成一个新的镜像(可用来还原场景等)，一般不使用该方式制作镜像，而是使用 Dockerfile 的方式，还可以作为版本记录的追踪

### FROM

```bash
指定基础镜像，而且必须是第一条指令
scratch镜像，这个镜像是一个虚拟的镜像，并不实际存在，表示一个空白的镜像，不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在

FROM centos:7
...

FROM scratch
```

### RUN

```bash
执行命令的指令，有两种格式
1.shell格式
...
RUN echo 'hello world' > /usr/share/nginx/html/index.html
...

2.exec格式
RUN ["可执行文件", "参数1", "参数2"]

在Dockerfile中RUN指令尽量使用 && 将命令拼接在一起，因为每一个指令就是一层，合并在一层优化层数，并且有 UnionFS 层数限制，不得超过127层
```

### WORKDIR

```bash
指定工作目录，后续的指令都会在当前工作目录，如果该目录不存在会创建该目录
格式: WORKDIR /path/
```

### ADD 和 COPY

```bash
这两个指令都可以将主机上的资源复制到容器镜像上
COPY
1.只能将宿主机上的资料复制到镜像里
格式: COPY <src>  <dest>

ADD
1.可以通过URL从远程服务器上复制到镜像上
2.可以将宿主机上的压缩包解压缩后复制到镜像上
格式:
ADD <src>  <dest>                         # 简单的将宿主机上的资源复制到镜像中
ADD http://test.com/nginx.tar.gz /tools/  # 下载资源到镜像中
ADD nginx.tar.gz /workdir/                # 将压缩包解压缩到镜像中

使用注意事项
1.源路径可以有多个
2.源路径是相对于执行 build 的相对路径
3.源路径如果是本地路径，必须是构建上下文中的路径
4.源路径如果是一个目录，则该目录下的所有内容都将被加入到容器，但是该目录本身不会
5.目标路径必须是绝对路径，或相对于 WORKDIR 的相对路径
6.目标路径如果不存在，则会创建相应的完整路径
7.目标路径如果不是一个文件，则必须使用/结束
8.路径中可以使用通配符
```

### 构建镜像

```bash
# Dockerfile所在目录，-t 指定镜像名称
docker build -t nginx:v1 .
```

### 构建上下文

```bash
上述中的 . 指的是上下文路径
```

### 推送镜像

### 私有仓库

客户端设置

```bash
# 对于自建的仓库荣如果不是https的会拒绝，因此在配置文件中写入以下内容
cat /etc/docker/daemon.json
{
"insecure-registries": ["127.0.0.1:5000"]
}

# 127.0.0.0.1:5000私有仓库地址，可设置为自己的
systemctl daemon-reload
systemctl restart docker
```

## Dockerfile 实践

## docker-compose

### 安装

```bash
# 国内下载http://get.daocloud.io/
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

### 管理参数

```bash
-f 			    # 指定使用的yaml文件
ps 			    # 显示所有的容器信息
restart 	  # 重新启动容器
logs 		    # 查看日志信息
config -q 	# 验证yaml配置文件
stop 		    # 停止容器
start 		  # 启动容器
up -d 		  # 启动容器项目
pause 		  # 暂停项目
UNpause 	  # 恢复暂停项目
rm 			    # 删除
```

### wordpress

```bash
# 编辑 docker-compose.yaml 文件
# 检查语法
docker-compose config -q

# 后台运行
docker-compose up -d
```

```yaml
# docker-compose.yaml
version: "2"

services:
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
```

### python-web

```bash
mkdir pythonweb && cd pythonweb
touch app.py
touch docker-compose.yaml
touch Dockerfile

docker pull python:3.6-alpine
docker pull redis:alpine

# app.py
import redis
import time
from flask import Flask
app = Flask(__name__)
cache = redis.Redis(host='redis',port=6379)
def get_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.3)
@app.route('/')
def hello():
    cnt = get_count()
    return 'Hello World! cnt={}\n'.format(cnt)

if __name__ == '__main__':
    app.run(host='0.0.0.0',debug=True)

# Dockerfile
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask -i https://pypi.douban.com/simple
CMD ["python","app.py"]


# docker-compose.yaml
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - ./code

  redis:
    image: redis:alpine


# 运行
docker-compos up
# 访问 http://0.0.0.0:5000
```
