---
title: VPN搭建使用
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
  - VPN
categories:
  - Linux
url: post/linux-15.html
toc: true
---

# 虚拟专用网络

VPN(全称：Virtual Private Network)虚拟专用网络，是依靠 ISP 和其他的 NSP，在公共网络中建立专用的数据通信的网络技术，可以为企业之间或者个人与企业之间提供安全的数据传输隧道服务。在 VPN 中任意两点之间的链接并没有传统专网所需的端到端的物理链路，而是利用公共网络资源动态组成的，可以理解为通过私有的隧道技术在公共数据网络上模拟出来的和专网有同样功能的点到点的专线技术，所谓虚拟是指不需要去拉实际的长途物理线路，而是借用了公共 Internet 网络实现的。

<!-- more -->

## VPN 的作用

VPN 的功能是帮助公司里的远程用户（出差，在家）、公司的分支机构、商业合作伙伴及供应商等公司和自己的公司内部网络之间建立可信的安全连接或者是局域网连接，确保数据的加密安全传输和业务访问，对于运维工程师来说，还可以连接不同的机房为局域网来处理相关事宜。

## VPN 的分类

- 根据常用的使用场景来分类。

#### 远程访问 VPN 服务

- 通过个人电脑远程拨号到企业办公网络。

  1、一般为企业内部员工出差、休假或特殊情况下载原理办公室的时候，又有需求访问公司的内部网络获取相关资源，就可以通过 VPN 拨号到公司内部。此时远程拨号的员工和办公室内的员工以及其他拨号的员工之间都相当于在一个局域网内。例如：访问内部的域控、文件服务器、OA 系统等局域网应用。

  2、对于运维人员来说就是需要个人电脑远程拨号到企业网站的服务器机房，远程维护机房中的（无外网 IP 的）服务器。

  这种形式的 VPN 一般在运维人员在工作中会经常遇到。

#### 企业内部网络互联

- 一般是建立一个网络隧道，多个机房之间互联。

在公司的分支机构的局域网和公司总部 LAN 之间的 VPN 链接。通过公网 Internet 建立 VPN 将公司在各地的分支机构的 LAN 链接到公司总部的 LAN。例如：各大银行之间的资金结算业务。

这是由于各个地域的原因产生的 VPN 的需求，通过 VPN 让不同地域的机器可以实现内网互联。例如远程协同办公，机房互联数据同步及业务访问。

#### 互联网各地域机房之间的互联

- 主要是用于不同机房之间的内网互通。

#### 企业外部的 VPN 服务

- 主要是提供给合作伙伴共享企业内网数据的 VPN 服务。

## 常见的隧道协议

- 这里列举常见的隧道协议

#### PPTP

点对点协议（PPTP）是由包括微软和 3Com 等公司组成的 PPTP 论坛开发的一种点对点隧道协议，基于拨号使用的 PPP 协议，使用 PAP 或 CHAP 之类的加密算法，或者使用 Microsoft 的点对点加密算法 MPPE。其通过跨越基于 TCP/IP 的数据网络创建 VPN 实现了从远程客户端到专用企业服务器之间数据的安全传输。PPTP 支持通过公共网络建立按需的、多协议的、虚拟专用网络。PPTP 允许加密 IP 通讯，然后在跨域公司 IP 网络或公共 IP 网络发送的 IP 头中对其进行封装。典型的 Linux 平台的开源软件为 PPTP。PPTP 属于点对点应用，比较合适远程的企业用户拨号到企业进行办公等应用。

#### L2TP

L2TP 第 2 等隧道协议（L2TP）是 IETF 基于 L2F（Cisco 的第二层转发协议）开的的 PPTP 的后续版本。是一种工业标准 Internet 隧道协议，其可以为跨越面向数据包的媒体发送点到点的协议（PPP）框架提供封装。PPTP 和 L2TP 都使用 PPP 协议对数据进行封装，然后添加附加爆头用于数据在互联网上传输。PPTP 只能在两端点间建立单一隧道。L2TP 支持在两端点间使用多隧道，用户可以针对不同的服务质量创建不同隧道。L2TP 可以提供隧道验证，而 PPTP 则不支持隧道验证。但是当 L2TP 或 PPTP 与 IPSEC 共同使用时，可以由 IPSEC 提供隧道验证，不需要在第二层协议上验证隧道使用 L2TP。PPTP 要求互联网络为 IP 网络。L2TP 只要求隧道媒介提供面向数据包的点对点的链接，L2TP 可以在 IP（使用 UDP），祯中继续永久虚拟电路（PVCs）,X.25 虚拟电路（VCs）或 ATM VCS 网络上使用。

#### IPSec

IP 安全协议实际上是一套协议包而不是一个单独的协议。从 1995 年开始 IPSec 的研究以来，IETF IPSec 工作组在它的主页上发布了几十个 Internet 草案文献和 12 个 RFC 文件。其中比较重要的有 RFC2409IKE（互联网秘钥交换）、RFC2401 IPSec 协议、RFC2402AH 验证包头、RFC2406ESP 加密数据等文件。

IPSec 隧道模式隧道是封装、路由与解封的整个过程。隧道将原始数据包隐藏（或封装）在新的数据包内部。该新的数据包可能会有新的寻址与路由信息，从而使其能够通过网络传输。隧道与数据保密性结合使用时，在网络上窃听通讯的人将无法获取原始数据包数据（以及原始的源和目标）。封装的数据包到达目的地后，会删除封装，原始数据包头用于将数据包路由到最终目的地。

隧道本身是封装数据经过的逻辑数据路径，对原始的源和目的的端，隧道是不可见的，而只能看到网络路径中的点对点连接。将隧道和数据保密性结合使用时，可用于提供 VPN。

封装的数据包在网络中的隧道内部传输。再次示例中，该网络是 Internet。网关可以是外部 Internet 与专用网络之间的周边网关。周界网关可以是路由器、防火墙、代理服务器或其他安全网关。另外，在专用网络内部可以使用两个网关来保护网络中不信任的通讯。

当以隧道模式使用 IPSEC 时，其只为 IP 通讯提供封装。使用 IPSec 隧道模式主要是为了与其他不支持 IPSec 上的 L2TP 或 PPTP VPN 隧道技术的路由器、网关或终端系统之间的相互操作。

#### SSL VPN

SSL 协议提供了数据私密性、端点验证、信息完整性等特性。SSL 协议由许多子协议组成，其中两个主要的子协议是握手协议和记录协议。握手协议允许服务器和客户端在应用协议传输第一个数据字节以前，彼此确认，协商一种加密算法和密码钥匙。在数据传输期间，记录协议利用握手协议生成的秘钥加密和解密后来交换的数据。

SSL 独立应用，因此任何一个应用程序都可以享受它的安全性而不必理会执行细节。SSL 置身于网络结构体系的传输层和应用层之间。此外，SSL 本身就被几乎所有的 WEB 浏览器支持。这意味着客户端不需要为了支持 SSL 链接安装额外的软件。这两个特征就是 SSL 能应用于 VPN 的关键点。

典型的 SSL VPN 应用：Open VPN，这是一个比较好的开源软件。Open VPN 允许参与建立 VPN 的单点使用预设的私钥，第三方证书，或者用户名/密码来进行身份验证。它大量使用了 OpenSSL 加密库，以及 SSLv3/TLSv1 协议。OpenVPN 能在 Linux、xBSD、MacOS 上运行。它并不是一个基于 Web 的 VPN 软件，也不能与 IPSec 及其他 VPN 软件包兼容。

## 常见的 VPN 软件

#### PPTP VPN

使用 PPTP VPN 的最大优势在于，无需在 Windows 客户端独立安装客户端软件，Windows 默认就知道 PPTP VPN 拨号连接功能。另外，PPTP VPN 属于点对点方式的应用，比较适合远程的企业用户拨号到企业进行办公等应用。

#### SSL VPN

PPTP 主要为那些经常外出移动办公或家庭办公的用户考虑，而 OpenVPN 不但使用于 PPTP 的应用场景，还适合针对企业异地两地总分公司之间的 VPN 不间断按需链接，例如：OA，及时通讯工具等在企业中的应用。

#### IPSec VPN

IPSecVPN 也适用针对企业异地办公或多个 IDC 机房之间 VPN 不间断按需链接，并且在部署使用上更简单方便。

# OpenVPN

## OpenVPN 介绍

专用网：专用网就是在两个网络（例如，北京和广州）之间架设一条专用线路，但是它并不需要真正地去铺设光缆之类的物理线路。虽然没有亲自去铺设，但是需要向电信运营商申请租用专线，在这条专用的线路上只传输自己的信息,所以安全稳定,同时也费用高昂

在众多的 VPN 产品中，OpenVPN 无疑是 Linux 下开源 VPN 的经典产品，他提供了良好的访问性能和友好的用户 GUI。

Open VPN 是一个用于创建虚拟专用网络加密通道的软件包，最早由 James Yonan 编写。一个实现 VPN 的开源软件，OpenVPN 是一个健壮的、高度灵活的 VPN 守护进程。它支持 SSL/TLS 安全、Ethernet bridging、经由代理的 TCP 或 UDP 隧道和 NAT。另外，它也支持动态 IP 地址以及 DHCP，可伸缩性足以支持数百或数千用户的使用场景，同时可移植至大多数主流操作系统平台上。

官网：https://openvpn.net

GitHub 地址：https://github.com/OpenVPN/openvpn

#### OpenVPN 依赖的 SSL 与 TLS 协议介绍

众所周知，真正的通信实际上是两台主机之间的进程在交换数据，而运输层作为整个网络最关键的从层次之一，扮演沿着向上层（应用层）提供通信服务的角色。想要剖析运输层的数据安全传输策略就一定无法绕开三个至关重要的协议，它们分别是 HTTPS 协议、SSL 协议、TSL 协议。SSL（Secure Sockets Layer）协议既安全套接字层协议，TLS（Transport Layer Security）协议即`安全传输层协议`。

![](https://gitee.com/gengff/blogimage/raw/master/images/dasdasdas.jpg)

## 部署 OpenVPN

```bash
OpenVPN 分为客户端和服务端
```

### 环境规划

| 公网 IP        | 内网 IP    | 主机名  |
| -------------- | ---------- | ------- |
| 192.168.15.110 | 172.16.1.0 | openvpn |

### 服务端

#### 1、安装 OpenVPN 和证书工具，准备相关配置文件

```bash
# 安装openvpn和证书工具
[root@m01 ~]# yum -y install openvpn easy-rsa

# 生成服务器配置文件
[root@m01 ~]# cp /usr/share/doc/openvpn-2.4.11/sample/sample-config-files/server.conf /etc/openvpn/

# 准备证书签发相关文件
[root@m01 ~]# cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa-server

# 准备签发证书相关变量的配置文件
[root@m01 ~]# cp /usr/share/doc/easy-rsa-3.0.8/vars.example /etc/openvpn/easy-rsa-server/3/vars


#建议修改给CA和OpenVPN服务器颁发的证书的有效期,可适当加长（可忽略）
[root@m01 ~]# vim /etc/openvpn/easy-rsa-server/3/vars

#CA的证书有效期默为为10年,可以适当延长,比如:36500天
#set_var EASYRSA_CA_EXPIRE     3650
set_var EASYRSA_CA_EXPIRE      36500

#服务器证书默为为825天,可适当加长,比如:3650天
#set_var EASYRSA_CERT_EXPIRE   825
#将上面行修改为下面
set_var EASYRSA_CERT_EXPIRE    3650


# 查看配置文件目录结构
[root@m01 ~]# tree /etc/openvpn/
/etc/openvpn/
├── client
├── easy-rsa-server
│   ├── 3 -> 3.0.8
│   ├── 3.0 -> 3.0.8
│   └── 3.0.8
│       ├── easyrsa
│       ├── openssl-easyrsa.cnf
│       ├── vars
│       └── x509-types
│           ├── ca
│           ├── client
│           ├── code-signing
│           ├── COMMON
│           ├── email
│           ├── kdc
│           ├── server
│           └── serverClient
├── server
└── server.conf

7 directories, 12 files
```

#### 2、初始化 PKI 和 CA 签发机构环境

```bash
# 初始化PKI生成PKI相关目录和文件
[root@m01 3]# cd /etc/openvpn/easy-rsa-server/3

# 初始化数据,在当前目录下生成pki目录及相关文件
[root@m01 3]# ./easyrsa init-pki

# 创建CA机构必须在该目录下
[root@m01 3]# pwd
/etc/openvpn/easy-rsa-server/3

# 创建CA机构,回车
[root@m01 3]# ./easyrsa build-ca nopass

# 验证CA证书
[root@m01 3]# openssl x509 -in pki/ca.crt -noout -text

# 创建服务端证书申请
[root@m01 3]# ./easyrsa gen-req server nopass

# 签发证书，输入yes回车
[root@m01 3]# ./easyrsa sign server server

# 验证证书，删除 BF97ECFB46EFB13E5A684AEF07DE4CD8.pem 该内容按tab键进行验证
[root@m01 3]# diff pki/certs_by_serial/BF97ECFB46EFB13E5A684AEF07DE4CD8.pem  pki/issued/server.crt
# 没有结果则为验证成功
```

#### 3、创建 Diffie-Hellman 密钥

Diffie-Hellman 密钥交换方法，由惠特菲尔德·迪菲（Bailey Whitfield Diffie）、马丁·赫尔曼（Martin Edward Hellman）于 1976 年发表。它是一种安全协议，让双方在完全没有对方任何预先信息的条件下通过不安全信道建立起一个密钥，这个密钥一般作为“对称加密”的密钥而被双方在后续数据传输中使用。DH 数学原理是 base 离散对数问题。做类似功能的还有非对称加密类算法，如：RSA。其应用非常广泛，在 SSH、VPN、Https 等都有应用。

wiki 参考链接: https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange

```bash
方式一：
 [root@m01 3]# ./easyrsa gen-dh
 #查看生成的文件
 [root@m01 3]# ll pki/dh.pem
 [root@m01 3]# cat pki/dh.pem

方式二：
 [root@m01 3]# openssl dhparam -out /etc/openvpn/dh2048.pem 2048
```

### 客户端

#### 1、客户端证书环境

```bash
[root@m01 3]# cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa-client
[root@m01 3]# cp /usr/share/doc/easy-rsa-3.0.8/vars.example /etc/openvpn/easy-rsa-client/3/vars

# 切换到客户端目录
[root@m01 3]# cd /etc/openvpn/easy-rsa-client/3

# 初始化证书目录
[root@m01 3]# ./easyrsa init-pki

[root@m01 3]# pwd
/etc/openvpn/easy-rsa-client/3
[root@m01 3]# ls
easyrsa  openssl-easyrsa.cnf  varsa  x509-types

# 生成客户端证书，回车
[root@m01 3]# ./easyrsa gen-req gengfeng nopass

# 将客户端证书请求文件复制到CA的工作目录
[root@m01 3]# cd /etc/openvpn/easy-rsa-server/3
[root@m01 3]# ./easyrsa import-req /etc/openvpn/easy-rsa-client/3/pki/reqs/gengfeng.req gengfeng
# 签发客户端证书，yes
[root@m01 3]# pwd
 /etc/openvpn/easy-rsa-server/3
[root@m01 3]# ./easyrsa sign client gengfeng

# 验证，对比一下
[root@m01 3]# cat pki/index.txt
 V 240401022739Z  ADBFFB9F45E5CEF861E7F642BA6C447E  unknown /CN=server
 V 240401023724Z  47765AD8225E12A13FB1EEBAC769B999  unknown /CN=gengfeng
# 查看是否和上面一致
[root@m01 3]# ll pki/certs_by_serial/
total 16
 -rw------- 1 root root 4438 Dec 28 10:37 47765AD8225E12A13FB1EEBAC769B999.pem
 -rw------- 1 root root 4552 Dec 28 10:27 ADBFFB9F45E5CEF861E7F642BA6C447E.pem
```

#### 2、创建链接配置文件

```bash
1、修改openvpn配置文件
[root@m01 3]# >/etc/openvpn/server.conf  #清空文件
[root@m01 3]# vim /etc/openvpn/server.conf

cat > /etc/openvpn/server.conf <<EOF
port 1194
proto tcp
dev tun
ca  /etc/openvpn/certs/ca.crt
cert  /etc/openvpn/certs/server.crt
key  /etc/openvpn/certs/server.key
dh  /etc/openvpn/certs/dh.pem
server 10.8.0.0 255.255.255.0
push "route 172.16.1.0 255.255.255.0"  #必须更改为自己的网卡地址
keepalive 10 120
cipher AES-256-CBC
compress lz4-v2
push "compress lz4-v2"
max-clients 2048
user openvpn
group openvpn
status  /var/log/openvpn/openvpn-status.log
log-append   /var/log/openvpn/openvpn.log
verb 3
mute 20
EOF

# 创建日志文件目录
[root@m01 3]# mkdir -p /var/log/openvpn
# 创建权限
[root@m01 ~]# chown openvpn.openvpn /var/log/openvpn
# 创建存放证书目录
[root@m01 ~]# mkdir -p /etc/openvpn/certs
# 复制证书
[root@m01 ~]# cp /etc/openvpn/easy-rsa-server/3/pki/dh.pem /etc/openvpn/certs/
[root@m01 ~]# cp /etc/openvpn/easy-rsa-server/3/pki/ca.crt /etc/openvpn/certs/
[root@m01 ~]# cp /etc/openvpn/easy-rsa-server/3/pki/private/server.key /etc/openvpn/certs/
[root@m01 ~]# cp /etc/openvpn/easy-rsa-server/3/pki/issued/server.crt /etc/openvpn/certs/
[root@m01 ~]# ll /etc/openvpn/certs/
total 20
 -rw------- 1 root root 1172 Dec 28 10:54 ca.crt
 -rw------- 1 root root  424 Dec 28 10:54 dh.pem
 -rw------- 1 root root 4552 Dec 28 10:54 server.crt
 -rw------- 1 root root 1704 Dec 28 10:54 server.key

2、启动OpenVPN
# 开启系统内核网络转发功能(该功能默认为关闭状态)
[root@m01 ~]# echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf
[root@m01 ~]# sysctl -p

# 安装防火墙
[root@m01 ~]# yum install iptables-services -y
[root@m01 ~]# systemctl disable --now firewalld
[root@m01 ~]# systemctl start iptables
[root@m01 ~]# iptables -F
[root@m01 ~]# iptables -F -t nat

# 添加iptables规则
[root@m01 ~]# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE

# 永久保存Iptables规则
[root@m01 ~]# service iptables save

# 启动OpenVPN
[root@m01 ~]# systemctl enable --now openvpn@server

3、创建链接文件

[root@m01 ~]# mkdir -p /etc/openvpn/client/gengfeng/

# 准备证书
[root@m01 gengfeng]# cp /etc/openvpn/easy-rsa-server/3/pki/ca.crt /etc/openvpn/client/gengfeng/
[root@m01 gengfeng]# cp /etc/openvpn/easy-rsa-server/3/pki/issued/gengfeng.crt /etc/openvpn/client/gengfeng/
[root@m01 gengfeng]# cp /etc/openvpn/easy-rsa-client/3/pki/private/gengfeng.key /etc/openvpn/client/gengfeng/

# 准备链接文件
[root@m01 ~]# vim /etc/openvpn/client/gengfeng/client.ovpn
client
dev tun
proto tcp
remote 192.168.15.110 1194
resolv-retry infinite
nobind
ca ca.crt
cert gengfeng.crt
key gengfeng.key
remote-cert-tls server
cipher AES-256-CBC
verb 3
compress lz4-v2
```

## 服务器端配置文件说明

```bash
#server.conf文件中以#或;开头的行都为注释
[root@instance-gvpb80ao ~]# grep -Ev "^#|^$" /etc/openvpn/server.conf
;local a.b.c.d  #本机监听IP,默认为本机所有IP
port 1194   #端口
;proto tcp  #协议,生产推荐使用TCP
proto udp #默认协议
;dev tap  #创建一个以太网隧道，以太网使用tap,一个tap设备允许完整的以太网帧通过Openvpn隧道，可提供非ip协议的支持，比如IPX协议和AppleTalk协议,tap等同于一个以太网设备，它操作第二层数据包如以太网数据帧。
dev tun  #创建一个路由IP隧道，生产推存使用tun.互联网使用tun,一个tun设备大多时候，被用于基于IP协议的通讯。tun模拟了网络层设备，操作第三层数据包比如IP数据封包。
;dev-node MyTap  #TAP-Win32适配器。非windows不需要配置
ca ca.crt   #ca证书文件
cert server.crt  #服务器证书文件
key server.key  #服务器私钥文件
dh dh2048.pem   #dh参数文件
;topology subnet
server 10.8.0.0 255.255.255.0 #客户端连接后分配IP的地址池，服务器默认会占用第一个IP 10.8.0.1将做为客户端的网关
ifconfig-pool-persist ipp.txt #为客户端分配固定IP，不需要配置,建议注释
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100 #配置网桥模式，不需要配置,建议注释
;server-bridge
;push "route 192.168.10.0 255.255.255.0" #给客户端生成的到达服务器后面网段的静态路由，下一跳为openvpn服务器的10.8.0.1
;push "route 192.168.20.0 255.255.255.0" #推送路由信息到客户端，以允许客户端能够连接到服务器背后的其它私有子网
;client-config-dir ccd #为指定的客户端添加路由，此路由通常是客户端后面的内网网段而不是服务端的，也不需要设置
;route 192.168.40.128 255.255.255.248
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252
;learn-address ./script   #运行外部脚本，创建不同组的iptables规则，无需配置
;push "redirect-gateway def1 bypass-dhcp" #启用后，客户端所有流量都将通过VPN服务器，因此生产一般无需配置此项
;push "dhcp-option DNS 208.67.222.222" #推送DNS服务器，不需要配置
;push "dhcp-option DNS 208.67.220.220"
;client-to-client  #允许不同的client直接通信,不安全,生产环境一般无需要配置
;duplicate-cn     #多个用户共用一个证书，一般用于测试环境，生产环境都是一个用户一个证书,无需开启
keepalive 10 120  #设置服务端检测的间隔和超时时间，默认为每10秒ping一次，如果 120 秒没有回应则认为对方已经down
tls-auth ta.key 0 #访止DoS等攻击的安全增强配置,可以使用以下命令来生成：openvpn --
genkey --secret ta.key #服务器和每个客户端都需要拥有该密钥的一个拷贝。第二个参数在服务器端应该为’0’，在客户端应该为’1’
cipher AES-256-CBC #加密算法
;compress lz4-v2  #启用Openvpn2.4.X新版压缩算法
;push "compress lz4-v2" #推送客户端使用新版压缩算法,和下面的comp-lzo不要同时使用
;comp-lzo  #旧户端兼容的压缩配置，需要客户端配置开启压缩,openvpn2.4.X等新版可以不用开启
;max-clients 100  #最大客户端数
;user nobody  #运行openvpn服务的用户和组
;group nobody
persist-key  #重启VPN服务时默认会重新读取key文件，开启此配置后保留使用第一次的key文件,生产环境无需开启
persist-tun  #启用此配置后,当重启vpn服务时，一直保持tun或者tap设备是up的，否则会先down然后再up,生产环境无需开启
status openvpn-status.log #openVPN状态记录文件，每分钟会记录一次
;log  openvpn.log  #第一种日志记录方式,并指定日志路径，log会在openvpn启动的时候清空日志文件,不建议使用
;log-append openvpn.log #第二种日志记录方式,并指定日志路径，重启openvpn后在之前的日志后面追加新的日志,生产环境建议使用
verb 3   #设置日志级别，0-9，级别越高记录的内容越详细,0 表示静默运行，只记录致命错误,4 表示合理的常规用法,5 和 6 可以帮助调试连接错误。9 表示极度冗余，输出非常详细的日志信息
;mute 20  #相同类别的信息只有前20条会输出到日志文件中
explicit-exit-notify 1  #通知客户端，在服务端重启后自动重新连接，仅能用于udp模式，tcp模式不需要配置即可实现断开重新连接,且开启此项后tcp配置后将导致openvpn服务无法启动,所以tcp时必须不能开启此项
```

## mac 连接 OpenVPN

安装 OpenVPN 软件包

Windows 下载：OpenVPN https://openvpn.net

Mac 下载：Tunnelblick https://tunnelblick.en.softonic.com/mac

打开 FinalShell 找到/etc/openvpn/client/目标文件夹

![image-20211230150939829](https://gitee.com/gengff/blogimage/raw/master/images/image-20211230150939829.png)

![image-20211230151130162](https://gitee.com/gengff/blogimage/raw/master/images/image-20211230151130162.png)

必须将文件夹重新命名，以.tblk 结尾，文件名字必须和之前的配置一样

![image-20211230151403052](https://gitee.com/gengff/blogimage/raw/master/images/image-20211230151403052.png)

最后打开 Tunnelblick 连接即可

![image-20211230151730831](https://gitee.com/gengff/blogimage/raw/master/images/image-20211230151730831.png)

![image-20211230151600943](https://gitee.com/gengff/blogimage/raw/master/images/image-20211230151600943.png)
