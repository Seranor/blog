---
title: Containerd相关使用
lastmod: 2021-06-03T16:43:23+08:00
date: 2021-06-02T11:52:03+08:00
tags:
  - Containerd
categories:
  - Containerd
url: post/containerd.html
toc: true
---

## Containerd 安装

```bash
#依赖安装
rpm -qa |grep libseccomp
yum install wget -y
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libseccomp-2.3.1-4.el7.x86_64.rpm
yum install libseccomp-2.3.1-4.el7.x86_64.rpm -y
```

<!-- more -->

```bash
#下载
wget https://github.com/containerd/containerd/releases/download/v1.5.5/cri-containerd-cni-1.5.5-linux-amd64.tar.gz
# 如果有限制，也可以替换成下面的 URL 加速下载
# wget https://download.fastgit.org/containerd/containerd/releases/download/v1.5.5/cri-containerd-cni-1.5.5-linux-amd64.tar.gz

#解压到系统各个目录中去
tar -C / -xzf cri-containerd-cni-1.5.5-linux-amd64.tar.gz

#环境变量设置
echo 'export PATH=$PATH:/usr/local/bin:/usr/local/sbin' >> ~/.bashrc
source ~/.bashrc

#生成默认配置
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

#查看启动配置文件
cat /etc/systemd/system/containerd.service

#启动
systemctl enable containerd --now

#查看版本
ctr version

#查看插件列表
ctr plugin ls

#配置加速器
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://bqr1dr1n.mirror.aliyuncs.com"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
      endpoint = ["https://registry.aliyuncs.com/k8sxio"]
```

## Containerd 使用

### 镜像相关

```bash
#要注意的是镜像地址需要加上 docker.io Host 地址,  --platform 选项指定对应平台的镜像
ctr image pull docker.io/library/nginx:alpine

#推送镜像,如果是私有镜像则在推送的时候可以通过 --user 来自定义仓库的用户名和密码
ctr image push

#列出本地镜像  -q 参数只打印镜像名称
ctr image ls

#检测本地镜像,主要查看其中的 STATUS, complete 表示镜像是完整可用的状态
ctr image check

#重新打标签
ctr image tag docker.io/library/nginx:alpine harbor.k8s.local/course/nginx:alpine

#删除镜像 加上 --sync 选项可以同步删除镜像和所有相关的资源
ctr image rm harbor.k8s.local/course/nginx:alpine

#将镜像挂载到主机目录
ctr image mount docker.io/library/nginx:alpine /mnt

#将镜像从主机目录上卸载
ctr image unmount /mnt

#将镜像到处为压缩包
ctr image export  --all-platforms nginx.tar.gz docker.io/library/nginx:alpine

#将压缩包导入镜像
ctr image import nginx.tar.gz

#直接导入可能会出现类似于 ctr: content digest sha256:xxxxxx not found 的错误，要解决这个办法需要 pull 所有平台镜像：
ctr i pull --all-platforms docker.io/library/nginx:alpine
ctr i export --all-platforms nginx.tar.gz docker.io/library/nginx:alpine
ctr i rm docker.io/library/nginx:alpine
ctr i import nginx.tar.gz
```

### 容器相关

```bash
#创建容器
ctr container create docker.io/library/nginx:alpine nginx

#列出容器 -q 精简内容
ctr container ls

#查看容器详细配置,类似docker inspect
ctr container info nginx

#删除容器 也可以使用 delete 或者 del 删除容器
ctr container rm nginx
ctr container ls
```

### 任务相关

```bash
#container create 命令创建的容器，并没有处于运行状态，只是一个静态的容器
#一个 container 对象只是包含了运行一个容器所需的资源及相关配置数据，表示 namespaces、rootfs 和容器的配置都已经初始化成功了，只是用户进程还没有启动
#一个容器真正运行起来是由 Task 任务实现的，Task 可以为容器设置网卡，还可以配置工具来对容器进行监控等

#运行容器
ctr container create docker.io/library/nginx:alpine nginx
ctr task start -d nginx

#查看正在运行的容器
ctr task ls

#获取容器的cgroup相关信息,还有内存、CPU 和 PID 的限额与使用量
ctr task metrics nginx

#查看容器中所有进程在宿主机中的 PID
ctr task ps nginx

#进入容器操作 --exec-id 后的id可以随意,唯一即可
ctr task exex --exec-id 0 -t nginx sh

#暂停容器
ctr task pause nginx
ctr task ls  # STATUS 变成了PAUSED

#恢复容器
ctr task resume nginx
ctr task ls  # STATUS 变成了RUNING

#杀掉容器  没有stop,只有暂停或者杀死
ctr task kill nginx
ctr task ls  # STATUS 变成了STOPPED

#删除task
ctr task rm nginx
ctr task ls  # 此时没有了

```

### 命名空间

```bash
#containerd支持命名空间概念,不指定命名空间就是在default

#查看命名空间
ctr ns ls

#创建命名空间
ctr ns creat test

#删除命名空间
ctr ns rm test  # ctr ns remove test

#操作资源的时候就可以指定命名空间,使用 -n 参数指定即可
ctr -n test image ls

#Docker 使用的 containerd 下面的命名空间默认是 moby，而不是 default，所以假如我们有用 docker 启动容器，那么我们也可以通过 ctr -n moby 来定位下面的容器
ctr -n moby container ls

#同样 Kubernetes 下使用的 containerd 默认命名空间是 k8s.io，所以我们可以使用 ctr -n k8s.io 来查看 Kubernetes 下面创建的容器

```

## Containerd 高级工具 nerdctl

### 安装

```bash
# 如果没有安装 containerd，则可以下载 nerdctl-full-<VERSION>-linux-amd64.tar.gz 包进行安装
wget https://github.com/containerd/nerdctl/releases/download/v0.12.1/nerdctl-0.12.1-linux-amd64.tar.gz
# 如果有限制，也可以替换成下面的 URL 加速下载
# wget https://download.fastgit.org/containerd/nerdctl/releases/download/v0.12.1/nerdctl-0.12.1-linux-amd64.tar.gz
mkdir -p /usr/local/containerd/bin/ && tar -zxvf nerdctl-0.12.1-linux-amd64.tar.gz nerdctl && mv nerdctl /usr/local/containerd/bin/

ln -s /usr/local/containerd/bin/nerdctl /usr/local/bin/nerdctl

nerdctl version

```

### 使用命令

#### 容器相关

```bash
#运行容器
nerdctl run -d -p 80:80 --name=nginx --restart=always nginx:alpine

#进入容器
nerdctl exec -it nginx /bin/sh

#列出容器 -a -q
nerdctl ps

#获取容器详细信息
nerdctl inspect nginx

#查看容器日志
nerdctl logs -f nginx

#停止容器
nerdctl stop nginx

#删除容器 -f 或者 --force 强制删除
nerdctl rm nginx
```

#### 镜像相关

```bash
#查看所有镜像
nerdctl images

#拉取镜像
nerdctl pull docker.io/library/busybox:latest

#推送镜像
nerdctl login --username xxx --password xxx  #登录
nerdctl logout  #注销退出登录
nerdctl push

#重新给镜像打标签
nerdctl tag nginx:alpine harbor.k8s.local/course/nginx:alpine

#导出镜像
nerdctl save -o busybox.tar.gz busybox:lastest

#删除镜像
nerdctl rmi busybox

#导入镜像
nerdctl load -i busybox.tar.gz
```

#### 构建镜像

##### 安装 buildctl

```bash
nerdctl build 需要依赖 buildkit 工具

buildkit 项目也是 Docker 公司开源的一个构建工具包，支持 OCI 标准的镜像构建。它主要包含以下部分
		服务端 buildkitd：当前支持 runc 和 containerd 作为 worker，默认是 runc，我们这里使用 containerd
		客户端 buildctl：负责解析 Dockerfile，并向服务端 buildkitd 发出构建请求


buildkit 是典型的 C/S 架构，客户端和服务端是可以不在一台服务器上，而 nerdctl 在构建镜像的时候也作为 buildkitd 的客户端，所以需要我们安装并运行 buildkitd


wget https://github.com/moby/buildkit/releases/download/v0.9.1/buildkit-v0.9.1.linux-amd64.tar.gz
# 如果有限制，也可以替换成下面的 URL 加速下载
# wget https://download.fastgit.org/moby/buildkit/releases/download/v0.9.1/buildkit-v0.9.1.linux-amd64.tar.gz
tar -zxvf buildkit-v0.9.1.linux-amd64.tar.gz -C /usr/local/containerd/


ln -s /usr/local/containerd/bin/buildkitd /usr/local/bin/buildkitd
ln -s /usr/local/containerd/bin/buildctl /usr/local/bin/buildctl


cat /etc/systemd/system/buildkit.service
[Unit]
Description=BuildKit
Documentation=https://github.com/moby/buildkit

[Service]
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true

[Install]
WantedBy=multi-user.target


systemctl daemon-reload
systemctl enable buildkit --now
systemctl status buildkit

```

##### 构建

```bash
cat Dockerfile
FROM nginx
RUN echo 'Hello Nerdctl From Containerd' > /usr/share/nginx/html/index.html

#一定要关闭防火墙和selinux
nerdctl build -t nginx:nerdctl -f Dockerfile .

#查看是否构建成功
nerdctl images

#测试是否成功
nerdctl run -d -p 80:80 --name=nginx --restart=always nginx:nerdctl

crul 127.0.0.1
```

文档整理:https://www.qikqiak.com/k3s/
