---
title: Pod基本使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Kuberneters
categories:
  - Kuberneters
url: post/kuberneters-03.html
toc: true
---

## Pod 原理

> Pod 是 Kubernetes 最基本的调度单元

一个 Pod 不等于一个容器

<!-- more -->

![image-20220213160259048](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220213160259048.png)

Pod 里面的容器都是共享同一个 Network Namespace，但是在文件系统上是完全隔离的

### Pod 网络

当新创建的容器和一个已经存在的容器共享一个 Network Namespace 时使用 Container 模式(--net=container:目标容器`)，缺点是有启动顺序，必须先启动一个容器后续容器才能加入

解决办法: 使用一个中间容器`Infra Container`，这个容器是 Pod 中第一个被创建的容器，这样后续容器加入到这个`Infra`容器中

![pod infra container](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/pod-infra-container.png)

### Pod 文件系统

Pod 中容器的文件系统默认是相互隔离的，要实现共享只需要在 Pod 的顶层声明一个 Volume，然后在需要共享这个 Volume 的容器中声明挂载即可

![pod containers share volumes](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/pod-volume-share.png)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  volumes:
    - name: varlog
      hostPath:
        path: /var/log/counter
  containers:
    - name: count
      image: busybox
      args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            i=$((i+1));
            sleep 1;
          done
      volumeMounts:
        - name: varlog
          mountPath: /var/log
    - name: count-log
      image: busybox
      args: [/bin/sh, -c, "tail -n+1 -f /opt/log/1.log"]
      volumeMounts:
        - name: varlog
          mountPath: /opt/log
```

## Pod 生命周期

![pod loap](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/pod-loap.jpg)

### Pod 状态

通过`kubectl explain pod.status.phase`命令可以看到 Pod 的几种状态

```
挂起（Pending）：Pod 信息已经提交给了集群，但是还没有被调度器调度到合适的节点或者 Pod 里的镜像正在下载
运行中（Running）：该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态
成功（Succeeded）：Pod 中的所有容器都被成功终止，并且不会再重启
失败（Failed）：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止
未知（Unknown）：因为某些原因无法取得 Pod 的状态，通常是因为与 Pod 所在主机通信失败导致的
```

创建 Pod 后可以通过`kubectl get pods {POD} -o yaml`

导出 yaml 的情况在 status-->conditons 属性有

```
astProbeTime：最后一次探测 Pod Condition 的时间戳。
lastTransitionTime：上次 Condition 从一种状态转换到另一种状态的时间。
message：上次 Condition 状态转换的详细描述。
reason：Condition 最后一次转换的原因。
status：Condition 状态类型，可以为 “True”, “False”, and “Unknown”.
type：Condition 类型，包括以下方面：
PodScheduled（Pod 已经被调度到其他 node 里）
Ready（Pod 能够提供服务请求，可以被添加到所有可匹配服务的负载平衡池中）
Initialized（所有的init containers已经启动成功）
Unschedulable（调度程序现在无法调度 Pod，例如由于缺乏资源或其他限制）
ContainersReady（Pod 里的所有容器都是 ready 状态）
```

### 重启策略

```
restartPolicy字段设置
Always     当容器失效时，由kubelet自动重启该容器，是默认值
OnFailure  当容器终止运行且退出码不为0时，由kubelet自动重启该容器
Never      不论容器运行状态如何，kubelet都不会重启该容器

控制器对Pod的重启策略
RC和DaemonSet：必须设置为Always，需要保证该容器持续运行。
Job和CronJob：OnFailure或Never，确保容器执行完成后不再重启。
kubelet：在Pod失效时自动重启它，不论将RestartPolicy设置为什么值，也不会对Pod进行健康检查。
```

### 初始化容器

```
Init Container 初始化容器，可以一个或多个
使用场景:
1.等待其他模块完成，例如WordPress先启动的数据库再启动后端等
2.初始化配置，chown权限设置等
3.将Pod注册到中央数据库、配置中心等
```

```yaml
# init-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  volumes:
    - name: workdir
      emptyDir: {}
  initContainers:
    - name: install
      image: busybox
      command:
        - wget
        - "-O"
        - "/work-dir/index.html"
        - http://www.baidu.com
      volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
  containers:
    - name: web
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: workdir
          mountPath: "/usr/share/nginx/html"
```

```bash
kubectl get pods -o wide  # 得到Pod的IP
curl PodIP  # 此时出现百度的页面
# install容器先启动完成任务后状态为Completed，然后启动主容器，启动完成后状态就为Running Man
# emptyDir{} 是一个临时目录。数据会保存在kubelet的工作目录下，生命周期与Pod生命周期一致
```

### `Pod Hook`

Pod Hook 是由 kubelet 发起的，当容器中的进程启动前或者容器中的进程终止之前运行，这是包含在容器的生命周期之中

Kubernetes 有以下两种钩子函数

PostStart: 容器创建后立即执行，主要用于资源部署，环境准备等会，钩子时间不能过长，否则容器不能达到 Running 状态

PreStop: 容器终止前立即被调用，主要用于优雅退出程序(如 nginx 的退出)，如果钩子在执行期间挂起，Pod 阶段将停留在 running 状态并且永不会达到 failed 状态

钩子函数应该尽量轻量，`PostStart` 或者 `PreStop` 钩子失败， 它会杀死容器，

实现钩子函数的方式

Exec:执行命令

HTTP:对容器上的特定的端点执行 HTTP 请求

```yml
# pod-poststart.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo1
spec:
  containers:
    - name: hook-demo1
      image: nginx
      lifecycle:
        postStart:
          exec:
            command:
              ["/bin/sh", "-c", "echo hello postStart heanlder > /opt/message"]
```

```bash
kubectl apply -f pod-poststart.yaml
kubectl get pods hook-demo1
kubectl   exec -it hook-demo1 -- cat /opt/message
```

```yaml
# pod-prestop.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo2
spec:
  containers:
    - name: hook-demo2
      image: nginx
      lifecycle:
        preStop:
          exec:
            command: ["/usr/sbin/nginx", "-s", "quit"] # 优雅退出

---
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo3
spec:
  volumes:
    - name: message
      hostPath:
        path: /tmp
  containers:
    - name: hook-demo2
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: message
          mountPath: /usr/share/
      lifecycle:
        preStop:
          exec:
            command:
              [
                "/bin/sh",
                "-c",
                "echo Hello from the preStop Handler > /usr/share/message",
              ]
```

```bash
kubectl apply -f pod-prestop.yaml
kubectl get pods
kubectl describe pod hook-demo3  # 去调度的节点上查看tmp目录中message文件
```

### `Pod健康检查`

`leveness probe`存活探针

```bash
检测程序是否存活，一旦检测到这个程序终止就会重启这个程序，例如检测到bug后就重启该容器，重启之后继续出现该bug，容易造成无限重启，因此在使用中可能会使用rediness probe，不让容器重启，保留当前状态进行排查问题
```

`readiness probe`可读性探针

```bash
确定容器是否已经就绪可以接收流量过来，只有当 Pod 中的容器都处于就绪状态的时候 kubelet 才会认定该 Pod 处于就绪状态，因为一个 Pod 下面可能会有多个容器。当然 Pod 如果处于非就绪状态，那么我们就会将他从 Service 的 Endpoints 列表中移除出来，这样我们的流量就不会被路由到这个 Pod 里面来了。
```

配置方式

exec

http

tcpSocket:类似端口检测，一般不推荐使用该方式

```yaml
# liveness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
spec:
  containers:
    - name: liveness
      image: busybox
      args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
spec:
  containers:
    - name: liveness
      image: cnych/liveness
      args:
        - /server
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
          httpHeaders:
            - name: X-Custom-Header
              value: Awesome
        initialDelaySeconds: 3
        periodSeconds: 3
```

```bash
periodSeconds：kubelet 每隔5秒执行一次存活探针，命令执行成功将返回0，当前这个容器是存活的，如果返回的是非0值，那么就会把该容器杀掉然后重启它。默认是10秒，最小1秒

initialDelaySeconds：表示在第一次执行探针的时候要等待5秒，这样能够确保我们的容器能够有足够的时间启动起来

timeoutSeconds：探测超时时间，默认1秒，最小1秒

successThreshold：探测失败后，最少连续探测成功多少次才被认定为成功，默认是 1，但是如果是 liveness 则必须是 1。最小值是 1

failureThreshold：探测成功后，最少连续探测失败多少次才被认定为失败，默认是 3，最小值是 1
```

## Pod 使用

### Pod 资源配置

```bash
 1. CGroup 里面对于 CPU 资源的单位换算
 1CPU = 1000millicpu (1 Core = 1000m)
 0.5CPU = 500millicpu (0.5 Core = 500m)

 2. CPU限制和请求设置
spec.containers[].resources.limits.cpu：CPU 上限值，可以短暂超过，容器也不会被停止
spec.containers[].resources.requests.cpu：CPU请求值，Kubernetes 调度算法里的依据值，可以超过
resources.requests.cpu的值如果设置大于集群内每个节点的最大CPU核心数，那么将没有节点满足，导致无法启动

3. 内存是不可压缩性资源，一旦达到上限就会OOM
1 Mib = 1024 Kib

4. 本质还是CGroup
   用下面的yaml创建pod

```

```yaml
# pod-resource-demo1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo1
spec:
  containers:
    - name: resource-demo1
      image: nginx
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: 50Mi
          cpu: 50m
        limits:
          memory: 50Mi
          cpu: 100m
```

```bash
kubectl get pods -o wide  # 查看pod调度在哪个节点上
crictl ps  # 查看容器ID
crictl inspect 容器ID  # 查看容器详细信息，一样可以查看到CPU的限制值等
crictl inspect 3f9d121e27999 |grep cgroupsPath  # 得到cgroupsPath信息
cd /sys/fs/cgroup/cpu/kubepods.slice/上面命令得到的ID信息
cat cpu.cfs_quota_us  # CPU的限制值
```

### 静态 Pod

```bash
Static Pod
直接由节点的kubelet进程管理和监控，因此命令行无法通过 kubnelet 管理
kubernetes的组件就是通过该方式创建的

创建静态Pod的方式有 配置文件和 HTTP

配置文件的方式:
cat /var/lib/kubelet/config.yaml |grep staticPodPath
# 默认位置是 /etc/kubernetes/manifests
ls /etc/kubernetes/manifests # 可以看到kubernetes的组件yaml文件

# 在该目录下创建一个yaml文件
cat <<EOF >/etc/kubernetes/manifests/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    app: static
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
EOF

kubectl get pods  # 此时有名称为 static-web-master 的容器，调度肯定是在本节点上
kubectl delete pods static-web-master #此时无法通过 kubectl 删除的
mv /etc/kubernetes/manifests/static-web.yaml /opt/ # 移走该文件之后此时pod也不存在了
kubectl get pods
```

### `Downward API`

```bash
作用:让Pod里的容器能够直接获取到这个Pod对象本身的一些信息

两种方式用于将 Pod 的信息注入到容器内部
	1.环境变量：用于单个变量，可以将 Pod 信息和容器信息直接注入容器内部
	2.Volume 挂载：将 Pod 信息生成为文件，直接挂载到容器内部中去
```

```yaml
#env-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
  namespace: kube-system
spec:
  containers:
    - name: env-pod
      image: busybox
      command: ["/bin/sh", "-c", "env"]
      env:
        - name: POD_XXX
          value: "xxx"
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
```

```bash
kubectl apply -f env-pod.yaml
kubectl logs env-pod -n kube-system |grep POD  # 可以看到 Pod 的 IP、NAME、NAMESPACE 都通过环境变量打印出来了
```

```yaml
# volume-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
  namespace: kube-system
  labels:
    k8s-app: test-volume
    node-env: test
  annotations:
    own: youdianzhishi
    build: test
spec:
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: labels
            fieldRef:
              fieldPath: metadata.labels
          - path: annotations
            fieldRef:
              fieldPath: metadata.annotations
  containers:
    - name: volume-pod
      image: busybox
      args:
        - sleep
        - "3600"
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
```

```bash
# 将元数据labels和annotaions以文件的形式挂载到了 /etc/podinfo 目录下
kubectl create -f volume-pod.yaml
kubectl exec -it volume-pod /bin/sh -n kube-system

cat /etc/podinfo/labels
cat /etc/podinfo/annotations
```

`DOwnward API`支持的字段

```bash
1. 使用 fieldRef 可以声明使用:
spec.nodeName - 宿主机名字
status.hostIP - 宿主机IP
metadata.name - Pod的名字
metadata.namespace - Pod的Namespace
status.podIP - Pod的IP
spec.serviceAccountName - Pod的Service Account的名字
metadata.uid - Pod的UID
metadata.labels['<KEY>'] - 指定<KEY>的Label值
metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
metadata.labels - Pod的所有Label
metadata.annotations - Pod的所有Annotation

2. 使用 resourceFieldRef 可以声明使用:
容器的 CPU limit
容器的 CPU request
容器的 memory limit
容器的 memory request
```
