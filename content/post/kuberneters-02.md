---
title: YAML格式
lastmod: 2021-06-22T16:43:23+08:00
date: 2021-06-22T11:52:03+08:00
tags:
  - Kuberneters
  - YAML
categories:
  - Kuberneters
url: post/kuberneters-02.html
toc: true
---

## 资源清单

### 描述

在 Kubernetes 环境下可以使用 kubectl run 运行应用，但是不推荐，而是希望使用资源清单的东西来描述应用，资源清单可以使用 YAML 和 JSON 文件来编写，一般 YAML 更方便阅读

通过一个资源清单文件定义好一个应用后，可以使用 kubectl 工具直接运行

<!-- more -->

```bash
kuebctl create -f xxx.yaml
```

资源清单提交给了 APIServer，然后集群获取到清单描述的应用信息后存入到 etcd 数据库中，然后 `kube-scheduler` 组件发现这个时候有一个 Pod 还没有绑定到节点上，就会对这个 Pod 进行一系列的调度，把它调度到一个最合适的节点上，然后把这个节点和 Pod 绑定到一起（写回到 etcd），然后节点上的 kubelet 组件这个时候 watch 到有一个 Pod 被分配过来了，就去把这个 Pod 的信息拉取下来，然后根据描述通过容器运行时把容器创建出来，最后当然同样把 Pod 状态再写回到 etcd 中去

示例:

nginx-deployment.yaml

```yaml
apiVersion: apps/v1 # API版本
kind: Deployment # API对象类型
metadata:
  name: nginx-deploy
  labels:
    chapter: first-app
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # Pod 副本数量
  template: # Pod 模板
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
            - containerPort: 80
```

创建应用:

```bash
kubectl apply -f nginx-deployment.yaml
kubectl get pods
```

创建应用之后可以看到有两个副本，由定义的属性`replicas: 2` 决定的

可以使用`kubectl describe` 命令查看资源对象的详细信息

```bash
kubectl  describe pod nginx-deploy-75b69bd684-h4sc2  #后面编号随机的
```

我们可以看到看到很多这个 Pod 的详细信息，比如调度到的节点、状态、IP 等，一般我们比较关心的是下面的 `Events` 部分，可以看到这个 Pod 是如何创建的

在集群中删除这个应用

```bash
kubectl delete -f nginx-deployment.yaml
```

### YAML 文件

`YAML` 是专门用来写配置文件的语言，非常简洁和强大，远比 `JSON` 格式方便，为了方便人类读写，实质上是一种通用的数据串行化格式。

#### 语法规则

- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用`Tab`键，只允许使用空格
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- `#` 表示注释

#### 结构类型

在 kubernets 中只需要了解以下结构类型

- Lists(列表)
- Maps(字典)

#### Maps

字典，key:value 的键值对，例如:

```yaml
---
apiVersion: v1
Kind: Pod
```

转换为 JSON

```jso
{
    "apiVersion": "v1",
    "kind": "pod"
}
```

创建复杂一点的 Maps，

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ydzs-site
  labels:
    app: web
```

metadata 对应的值又可以是一个 Maps，转换为 JSON

```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "kube100-site",
    "labels": {
      "app": "web"
    }
  }
}
```

#### Lists

列表，也就是一个数组，定义：

```yaml
args
- Cat
- Dog
- Fish
```

对应的 JSON 格式:

```json
{
  "args": ["Cat", "Dog", "Fish"]
}
```

Lists 的子项也可以是 Maps，Maps 的子项也可以是 Lists 如下所示：

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ydzs-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: flaskapp-demo
      image: cnych/flaskapp
      ports:
        - containerPort: 5000
```

转换为 JSON 格式如下:

```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "ydzs-site",
    "labels": {
      "app": "web"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "front-end",
        "image": "nginx",
        "ports": [
          {
            "containerPort": "80"
          }
        ]
      },
      {
        "name": "flaskapp-demo",
        "image": "cnych/flaskapp",
        "ports": [
          {
            "containerPort": "5000"
          }
        ]
      }
    ]
  }
}
```
