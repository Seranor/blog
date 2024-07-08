---
title: Kubernetes安装
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Kuberneters
categories:
  - Kuberneters
url: post/kuberneters-01.html
toc: true
---

### 规划

| IP           | Hostname | 配置 | 系统      |
| ------------ | -------- | ---- | --------- |
| 192.168.0.11 | master1  | 4c8g | Centos7.6 |
| 192.168.0.12 | node1    | 4c8g | Centos7.6 |
| 192.168.0.13 | node2    | 4c8g | Centos7.6 |

<!-- more -->

### hosts 文件

```bash
cat /etc/hosts
192.168.0.11 master1
192.168.0.12 node1
192.168.0.13 node2
```

### 关闭防火墙及 selinux

```bash
systemctl stop firewalld
systemctl disable firewalld

setenforce 0
sed -i  s#enforcing#disabled#g /etc/selinux/config
```

### 加载内核模块

```bash
#由于开启内核 ipv4 转发需要加载 br_netfilter 模块，所以加载下该模块：

modprobe br_netfilter

echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf



sysctl --system
sysctl -p
```

### 安装 ipvs

```bash
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4

```

### 安装必要软件

```bash
yum install ipset ipvsadm wget vim net-tools curl chrony  -y

#同步服务器时间
systemctl enable chronyd
systemctl start chronyd
chronyc sources

#关闭swap

swapoff -a
#修改/etc/fstab文件，注释掉 SWAP 的自动挂载
```

### 安装 Containerd

```bash
#国内会被限制下载
#wget https://github.com/containerd/containerd/releases/download/v1.5.5/cri-containerd-cni-1.5.5-linux-amd64.tar.gz

# 如果有限制，也可以替换成下面的 URL 加速下载
wget https://download.fastgit.org/containerd/containerd/releases/download/v1.5.5/cri-containerd-cni-1.5.5-linux-amd64.tar.gz

#直接将压缩包解压到系统的各个目录中
tar -C / -xzf cri-containerd-cni-1.5.5-linux-amd64.tar.gz

#然后要将 /usr/local/bin 和 /usr/local/sbin 追加到 ~/.bashrc 文件的 PATH 环境变量中：

echo 'export PATH=$PATH:/usr/local/bin:/usr/local/sbin' >> /root/.bashrc
source ~/.bashrc

#命令生成一个默认的配置
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml


#修改一:
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
#修改二:配置加速器国内版本
 [plugins."io.containerd.grpc.v1.cri"]
  ...
  # sandbox_image = "k8s.gcr.io/pause:3.5"
  sandbox_image = "registry.aliyuncs.com/k8sxio/pause:3.5"
  ...
  [plugins."io.containerd.grpc.v1.cri".registry]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
        endpoint = ["https://bqr1dr1n.mirror.aliyuncs.com"]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
        endpoint = ["https://registry.aliyuncs.com/k8sxio"]


#启动
systemctl daemon-reload
systemctl enable containerd --now

#查看版本情况
ctr version
crictl version
```

### 下载 kubeadm、kubelet

```bash
#能上外网版本
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF


#国内版本
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

#下载
# --disableexcludes 禁掉除了kubernetes之外的别的仓库
yum makecache fast -y
yum install -y kubelet-1.22.2 kubeadm-1.22.2 kubectl-1.22.2 --disableexcludes=kubernetes

#查看版本
kubeadm version

#设置开机自启
systemctl enable --now kubelet
```

### 初始化集群

```bash
kubeadm config print init-defaults --component-configs KubeletConfiguration > kubeadm.yaml
```

kubeadm.yaml

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
  - groups:
      - system:bootstrappers:kubeadm:default-node-token
    token: abcdef.0123456789abcdef
    ttl: 24h0m0s
    usages:
      - signing
      - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.0.11 # 指定master节点内网IP
  bindPort: 6443
nodeRegistration:
  criSocket: /run/containerd/containerd.sock # 使用 containerd的Unix socket 地址
  imagePullPolicy: IfNotPresent
  name: master
  taints: # 给master添加污点，master节点不能调度应用
    - effect: "NoSchedule"
      key: "node-role.kubernetes.io/master"

---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs # kube-proxy 模式

---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/k8sxio
kind: ClusterConfiguration
kubernetesVersion: 1.22.2
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16 # 指定 pod 子网
scheduler: {}

---
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
clusterDNS:
  - 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
cgroupDriver: systemd # 配置 cgroup driver
logging: {}
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

### 下载镜像

```bash
#只下载镜像
kubeadm config images pull --config kubeadm.yaml

#上面coredns镜像有问题需要单独拉
ctr -n k8s.io i pull docker.io/coredns/coredns:1.8.4

#拉下来后进行改名
ctr -n k8s.io i tag docker.io/coredns/coredns:1.8.4 registry.aliyuncs.com/k8sxio/coredns:v1.8.4
```

### 初始化集群

```bash
kubeadm init --config kubeadm.yaml

 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

 #在初始化完成后会出现其他节点加入进来的命令
 kubeadm join 192.168.31.31:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ca0c87226c69309d7779096c15b6a41e14b077baf4650bfdb6f9d3178d4da645
```

### 查看是否初始化成功

```bash
kubectl get nodes
```

### 安装 flannel

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 如果有节点是多网卡，则需要在资源清单文件中指定内网网卡
# 搜索到名为 kube-flannel-ds 的 DaemonSet，在kube-flannel容器下面
vim kube-flannel.yml
......
containers:
- name: kube-flannel
  image: quay.io/coreos/flannel:v0.15.0
  command:
  - /opt/bin/flanneld
  args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=eth0  # 如果是多网卡的话，指定内网网卡的名称
......
kubectl apply -f kube-flannel.yml  # 安装 flannel 网络插件

#查看flannel情况
kubectl get pods -n kube-system
```

### Dashboard 安装

```bash
# 推荐使用下面这种方式
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
➜  ~ vi recommended.yaml
# 修改Service为NodePort类型
......
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort  # 加上type=NodePort变成NodePort类型的服务
......

kubectl apply -f recommended.yaml

kubectl get pods -n kubernetes-dashboard -o wide
```

### 更换 cni 网络

```bash
#每个节点都需要操作
mv /etc/cni/net.d/10-containerd-net.conflist /etc/cni/net.d/10-containerd-net.conflist.bak

ifconfig cni0 down && ip link delete cni0

systemctl daemon-reload

systemctl restart containerd kubelet

#删除coredns达到重启目的
kubectl  -n kube-system delete pod coredns-7568f67dbd-9wcv4

#重启dashboard
kubectl delete -f recommended.yaml
kubectl apply -f recommended.yaml
```

### 进入 Dashboard

```bash
#查看dashboard的端口
kubectl get svc -n kubernetes-dashboard

```

创建权限

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: admin
    namespace: kubernetes-dashboard
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kubernetes-dashboard
```

创建并生成 token

```bash
kubectl apply -f admin.yaml

kubectl get secret -n kubernetes-dashboard|grep admin-token
#得到一个以 admin-token-xxx 的一个

kubectl get secret {admin-token-xxx} -o jsonpath={.data.token} -n kubernetes-dashboard |base64 -d
# 会生成一串很长的base64后的字符串

#谷歌浏览器访问的时候会打不开
#解决方法一: 更换火狐浏览器
#解决方法二: 谷歌浏览器非安全页面,空白处输入thisisunsafe即可
```

### kubectl 命令补全

```bash
yum install -y bash-completion*
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### master 污点

```bash
#去除污点变成可调度
kubectl taint node master node-role.kubernetes.io/master-

#打上污点补课调度
kubectl taint node master node-role.kubernetes.io/master="":NoSchedule
```

### 剔除节点并重新加入

```bash
###简易版本###

##master节点上操作
#驱逐节点上的pod
kubectl drain node3 --delete-local-data --ignore-daemonsets --force

#主节点上删除node节点
kubectl  delete nodes node3

#查看加入集群命令
kubeadm token create --print-join-command

##node3节点上操作
#在node3上重置
kubeadm reset

#得到上面加入集群的命令重新加入
kubeadm join  xxx
```

### kubectl 远程

mac(zsh 下)操作

```bash

brew install kubectl

echo 'source <(kubectl completion zsh)' > ~/.zshrc

mkdir ~/.kube

#将k8s集群下的/etc/kubernetes/admin.conf拷贝到本机的~/.kube/config
#在k8s集群master上查看，DNS 区域就是包含的校验的域名，后面还有 IP
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text
...
DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:master1, IP Address:10.96.0.1, IP Address:192.168.0.21
    Signature Algorithm: sha256WithRSAEncryption
...

#本地，将k8s的master外网IP和上面得到的DNS后面的映射信息写入到/etc/hosts中
#将本地~/.kube/config中  server:6443 改成写入到/etc/hosts中的DNS映射信息
cat ~/.kube/config
...
server: https://master1:6443  # 这里如果直接用公网IP不行
...


cat /etc/hosts
139.155.237.70 master1


这个时候就可以愉快的本地操作k8s集群了
```

brew 安装

```bash
https://zhuanlan.zhihu.com/p/111014448

#全部国内源，下载速度快
#常规安装
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"

#急速安装
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)" speed

#卸载脚本
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"

#项目地址
https://gitee.com/cunkai/HomebrewCN/blob/master/error.md
```

文档整理:https://www.qikqiak.com/k3s/
