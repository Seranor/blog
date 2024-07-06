---
title: Bond4配置
lastmod: 2021-04-21T16:43:23+08:00
date: 2021-04-21T11:52:03+08:00
tags:
  - Linux
  - Bond
categories:
  - Linux
url: post/linux-06.html
toc: true
---

### Ubuntu 配置

<!-- more -->

```bash
cat /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    ens3f1: {}
    ens3f0: {}
  renderer: networkd
  bonds:
    bond4:
      addresses: [10.10.2.1/16]
      gateway4: 10.10.1.254
      nameservers:
        addresses: [114.114.114.114,202.96.128.86,8.8.8.8]
      interfaces:
        - ens3f1
        - ens3f0
      parameters:
        mode: 802.3ad
        mii-monitor-interval:
        lacp-rate: fast
        transmit-hash-policy: layer3+4

#生效
netplan apply
```

### Centos 配置

```bash
参考文档：https://support.huawei.com/enterprise/zh/knowledge/EKB1100053867

#双bond配置
#备份原本的网卡配置信息
mkdir /opt/net_bak
cd /etc/sysconfig/network-scripts
cp ifcfg-* /opt/net_bak

#生成bond网卡配置文件名称为bond4
nmcli connection add type bond ifname bond4 mode 4

#将ens1f0和ens6f0网卡绑定到bond4
#f0为两个网卡的第一口
nmcli connection add type bond-slave ifname ens1f0 master bond4
nmcli connection add type bond-slave ifname ens6f0 master bond4

#生成bond网卡配置文件名称为bond20
nmcli connection add type bond ifname bond20 mode 4

#将ens1f1和ens6f1网卡绑定到bond20
#f1为两个网卡的第一口
nmcli connection add type bond-slave ifname ens1f1 master bond20
nmcli connection add type bond-slave ifname ens6f1 master bond20


#查看生成的bond配置文件
ls ifcfg-bond-*
ifcfg-bond-bond0 ifcfg-bond-slave-enp125s0f0 ifcfg-bond-slave-enp125s0f1

#查看生成的bond配置信息
nmcli con show

#配置网卡，将IP、网关、掩码、DNS配置
vim ifcfg-bond-bond4
vim ifcfg-bond-bond20

#网卡模式选择：
#bond4配置：
BONDING_OPTS='mode=4 miimon=100 xmit_hash_policy=layer3+4'
#刻录系统需要将网卡内的UUID删除

#重启网卡
nmcli con reload
systemctl restart network.service

#检查配置情况
#查看是否配置成功
ip addr

#查看生成的bond是否正常
cat /proc/net/bond/bond0
ethtool bond0
```

交换机配置

```bash
#bond4需要交换机配置
interface Eth-Trunk10
mode lacp-static

interface GigabitEthernet0/0/1
eth-trunk 10

interface GigabitEthernet0/0/2
eth-trunk 10
```
