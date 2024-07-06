---
title: CGroups和Namespace
lastmod: 2021-04-21T16:43:23+08:00
date: 2021-04-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-05.html
toc: true
---

## CGroup

### CGroups 概述

`CGroups` 全称为 `Linux Control Group`，其作用是限制一组进程使用的资源（CPU、内存等）上限，`CGroups` 也是 Containerd 容器技术的核心实现原理之一

<!-- more -->

- Task: 在 cgroup 中，task 可以理解为一个进程，但这里的进程和一般意义上的操作系统进程不太一样，实际上是进程 ID 和线程 ID 列表。
- CGroup: 即控制组，一个控制组就是一组按照某种标准划分的 Tasks，可以理解为资源限制是以进程组为单位实现的，一个进程加入到某个控制组后，就会受到相应配置的资源限制。
- Hierarchy: cgroup 的层级组织关系，cgroup 以树形层级组织，每个 cgroup 子节点默认继承其父 cgroup 节点的配置属性，这样每个 Hierarchy 在初始化会有 root cgroup。
- Subsystem: 即子系统，子系统表示具体的资源配置，如 CPU 使用，内存占用等，Subsystem 附加到 Hierarchy 上后可用。

* 查看当前系统支持的 CGroups 子系统

![n6YGX3](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/n6YGX3.png)

查看 cgroup

```bash
df -h |grep cgroup
```

查看当前系统挂载了哪些 cgroup

![fARleD](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/fARleD.png)

`/sys/fs/cgroup` 目录下的每个子目录就对应着一个子系统，cgroup 是以目录形式组织的，`/` 是 cgroup 的根目录，但是这个根目录可以被挂载到任意目录，例如 CGroups 的 memory 子系统的挂载点是 `/sys/fs/cgroup/memory`，那么 `/sys/fs/cgroup/memory/` 对应 memory 子系统的根目录

### CGroups 测试

```bash
mkdir -p /sys/fsc
ls /sys/fs/cgroup/cpu/klcc.test
cat /sys/fs/cgroup/cpu/klcc.test/cpu.cfs_period_us
cat /sys/fs/cgroup/cpu/klcc.test/cpu.cfs_quota_us
```

![hdIO6e](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/hdIO6e.png)

目录创建完成后，下面就会已经自动创建 cgroup 的相关文件

`cpu.cfs_period_us` 文件，用来配置 CPU 时间周期长度的，默认为 `100000us`

cpu.cfs_quota_us 文件，用来设置在此时间周期长度内所能使用的 CPU 时间数，默认值为-1，表示不受时间限制。

编写一个简单的 python 脚本消耗 cpu

```python
while True:
    pass
```

直接运行

```bash
python cgroup.py &
[1] 8288
```

此时用 top 命令查看到 8288 进程已经达到了 100%

![hv8Mr8](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/hv8Mr8.png)

现在我们将这个进程 ID 写入到 `/sys/fs/cgroup/cpu/klcc.test/tasks` 文件下面去，然后设置 `/sys/fs/cgroup/cpu/ydzs.test/cpu.cfs_quota_us` 为 `10000us`，因为 `cpu.cfs_period_us` 默认值为 `100000us`，所以这表示我们要限制 CPU 使用率为 10%：

```bash
echo 8288 >  /sys/fs/cgroup/cpu/klcc.test/tasks
echo 10000 > /sys/fs/cgroup/cpu/klcc.test/cpu.cfs_quota_us
```

此时使用`top`命令查看是就是被限制在 10%左右了

![Cw8YLg](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/Cw8YLg.png)

如果要限制内存等其他资源的话，同样去对应的子系统下面设置资源，并将进程 ID 加入 tasks 中即可。如果要删除这个 cgroup，直接删除文件夹是不行的，需要使用 `libcgroup` 工具：

```bash
yum install libcgroup libcgroup-tools
cgdelete cpu:klcc.test
```

### 容器中 CGroup 的使用

创建一个加`-m`参数限制容器内存

此时启动完成后容器的 cgroup 会出现在`/sys/fs/cgroup/memory/docker`下

![1Sc8kM](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/1Sc8kM.png)

可以看到很多和内存相关的配置文件

![Xdpq7i](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/Xdpq7i.png)

查看`memory.limit_in_bytes` 结果是创建时设置的内存大小

```bash
cat  /sys/fs/cgroup/memory/docker/3f1a79a1ef6d613d36d32a5b7216068ef008d59cce879475b3ce5ad7ee131263/memory.limit_in_bytes
52428800
```

容器的进程 ID 也会在 task 文件中

![image-20211124165322290](/Users/zhijinliu/Library/Application Support/typora-user-images/image-20211124165322290.png)

当删除这个容器之后，相应的容器 ID 也会被删除

## Namespace

`namespace` 也称命名空间，是 Linux 为我们提供的用于隔离进程树、网络接口、挂载点以及进程间通信等资源的方法。在日常使用个人 PC 时，我们并没有运行多个完全分离的服务器的需求，但是如果我们在服务器上启动了多个服务，这些服务其实会相互影响的，每一个服务都能看到其他服务的进程，也可以访问宿主机器上的任意文件，一旦服务器上的某一个服务被入侵，那么入侵者就能够访问当前机器上的所有服务和文件，这是我们不愿意看到的，我们更希望运行在同一台机器上的不同服务能做到完全隔离，就像运行在多台不同的机器上一样。而我们这里的容器其实就通过 Linux 的 Namespaces 技术来实现的对不同的容器进行隔离。

linux 共有 6(7)种命名空间:

- `ipc namespace`: 管理对 IPC 资源（进程间通信（信号量、消息队列和共享内存）的访问
- `net namespace`: 网络设备、网络栈、端口等隔离
- `mnt namespace`: 文件系统挂载点隔离
- `pid namespace`: 用于进程隔离
- `user namespace`: 用户和用户组隔离（3.8 以后的内核才支持）
- `uts namespace`: 主机和域名隔离
- `cgroup namespace`：用于 cgroup 根目录隔离（4.6 以后版本的内核才支持）

通过 lsns 查看当前系统已经创建的名称空间

![jxos29](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/jxos29.png)

要查看一个进程所属的命名空间信息，可以到 `/proc/<pid>/ns` 目录下查看，

这些 namespace 都是链接文件, 格式为 `namespaceType:[inode number]`，`inode number` 用来标识一个 namespace，可以理解为 namespace id，如果两个进程的某个命名空间的链接文件指向同一个，那么其相关资源在同一个命名空间中，也就没有隔离了

![snZVnb](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/snZVnb.png)

可以看出 nginx 容器启动后，已经为该容器自动创建了单独的 `mtn`、`uts`、`ipc`、`pid`、`net` 命名空间，也就是这个容器在这些方面是独立隔离的，其他容器想要和该容器共享某一个命名空间，那么就需要指向同一个命名空间

白嫖地址: https://www.qikqiak.com/k3s/runtime/cgroups-namespaces/
