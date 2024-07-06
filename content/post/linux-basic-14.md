---
title: Linux防火墙iptables
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
  - Iptables
categories:
  - Linux
url: post/linux-14.html
toc: true
---

# 架构图

<!-- more -->

![项目架构](https://gitee.com/gengff/blogimage/raw/master/images/%E9%A1%B9%E7%9B%AE%E6%9E%B6%E6%9E%84.png)

![6581640856451_.pic_副本](https://gitee.com/gengff/blogimage/raw/master/images/6581640856451_.pic_%E5%89%AF%E6%9C%AC.jpg)

# 防火墙种类及使用说明

![2615753-20211226235009781-222102770](https://gitee.com/gengff/blogimage/raw/master/images/2615753-20211226235009781-222102770.webp)

- 什么是防火墙

  - 防火墙一直被认为是保护敏感信息的第一道防线。他们在安全与受控内部网络之间建立了一道屏障，提供低级保护，以及重要的日志记录和审计功能。它能监视传入和传出的流量，并根据一组以定义的安全规则决定是允许还是阻止特定流量。

- 防火墙种类
  - 硬件防火墙（主机防火墙）
    - 三层路由： 华为 H3C(华三)
    - 深信服
  - 软件防火墙（网络防火墙）
    - iptables
    - firewalld
  - 云防火墙
    - 阿里云:安全组（默认的是白名单 防火墙默认规则是拒绝）

# Iptables

Netfilter/Iptables（以下简称 Iptables）是 unix/linux 自带的一款优秀且开放源代码的完全自由的基于包过滤的防火墙工具，它的功能十分强大，使用非常灵活，可以对流入和流出服务器的数据包进行很精细的控制。在早期的 Linux 系统中，默认使用的是 iptables 防火墙管理服务来配置防火墙。尽管新型的 firewalld 防火墙管理服务已经被投入使用多年，但是大量的企业在生产环境中依然出于各种原因而继续使用 iptables。

- iptables 是 linux2.4 及 2.6 内核中集成的服务。
- iptables 主要工作在 OSI 七层的二、三、四层，如果重新编译内核，iptables 也可以支持 7 层控制

![image-20210823223210395](https://gitee.com/gengff/blogimage/raw/master/images/image-20210823223210395.png)

```bash
用户  --->  调用iptables  --->  ip_tables内核模块  --->  Netfilter（系统安全框架） --->  过滤请求
```

### iptables 防火墙网路安全前言介绍

```bash
学好iptables的基础：
  1、OSI7层模型以及不同层对应哪些协议？
  2、TCP/IP三次握手，四次断开的过程，TCP HEADER，状态转换
  3、常用的服务端口要非常清楚了解。
  4、常用服务协议原理http协议，icmp协议。

企业中安全配置原则：
  1、尽可能不给服务器配置外网IP，可以通过代理转发或者通过防火墙映射。
  2、并发不是特别大情况有外网IP，可以开启防火墙服务。
  3、大并发的情况，不能开iptables，影响性能，利用硬件防火墙提升架构安全。Copy to clipboardErrorCopied
```

### 包过滤防火墙

```bash
1、什么是包
	在数据传输过程，并不是一次性传输完成的；而是将数据分成若干个数据包，一点一点的传输。

2、 什么是包过滤防火墙
	过滤数据包的防火墙。

4、包过滤防火墙如何实现
  通过系统安全框架，过滤数据包。

5、防火墙使用时名词概念理解
容器：装东西的器皿，docker容器技术，将镜像装在了一个系统中，这个系统就称为容器
iptables称为一个容器---装着防火墙的表
防火墙的表又是一个容器---装着防火墙的链
防火墙的链也是一个容器---装着防火墙的规则
iptables---表---链---规则

规则：防火墙一条一条安全策略
防火墙匹配规则流程：
  1. 防火墙是层层过滤的，实际是按照配置规则的顺序从上到下，从前到后进行过滤的。
  2. 如果匹配上规则，即明确表示是阻止还是通过，数据包就不再向下匹配新的规则。
  3. 如果规则中没有明确表明是阻止还是通过的，也就是没有匹配规则，向下进行匹配，直到匹配默认规则得到明确的阻止还是通过。
  4. 防火墙的默认规则是所有规则执行完才执行的。Copy to clipboardErrorCopied
```

## 4 表 5 链

```bash
1、四个表，有哪些作用
   具备某种功能的集合叫做表。

   filter：负责做过滤功能呢	  INPUT、OUTPUT、FORWARD
   nat：	  网络地址转换	     PREROUTING、INPUT、OUTPUT、POSTROUTING
   mangle：负责修改数据包内容  PREROUTING、INPUT、OUTPUT、POSTROUTING、FORWARD
   raw：   负责数据包跟踪      PREROUTING、OUTPUT

2、五条链，运行在那些地方

   PREROUTING、INPUT、OUTPUT、FORWARD、POSTROUTING

  1）PREROUTING: 主机外报文进入位置，允许的表mangle, nat（目标地址转换，把本机地址转换为真正的目标机地址，通常指响应报文）
  2）INPUT：报文进入本机用户空间位置，允许的表filter, mangle
  3）OUTPUT：报文从本机用户空间出去的位置，允许filter, mangle, nat
  4）FOWARD：报文经过路由并且发觉不是本机决定转发但还不知道从哪个网卡出去，允许filter, mangle
  5）POSTROUTING：报文经过路由被转发出去，允许mangle，nat（源地址转换，把原始地址转换为转发主机出口网卡地址）

   流入本机：PREROUTING  -->  INPUT  --> PROCESS(进程)
   流出本机：PROCESS(进程) -->  OUTPUT --> POSTROUTING
   经过本机：PREROUTING  --> FORWARD --> POSTROUTING
```

#### 流入本机

- 当外部的数据进入时通过网卡进入本机后，在网络层时会经过 PREROUTING（主机外报文进入位置）链继续前进到达用户层之前会经过 INPUT 链。

```
数据进入（通过网线）---> 链接网卡设备 ---> 网络接口层 ---> netfilter --->
在网络层时会经过PREROUTING（主机外报文进入的位置）链 ---> TCP UDP协议 --->
进入用户层之前（INPUT）---> 到达用户层
```

![2615753-20211227000852784-704163914](https://gitee.com/gengff/blogimage/raw/master/images/2615753-20211227000852784-704163914.png)

#### 流出本机

- 当用户从用户层发出数据之后，会先经过 OUTPUT 链，在经过了 OUTPUT 链到达 Netfilter 防火墙，在经过防火墙到达设备驱动之前，会经过 POSTROUTING 链，之后在发送出去。

```
用户操作命令工具(iptables) --> OUTPUT链 --> ip_tables内核模块 -->
Netfilter(防火墙) --> 网络层 --> 网络接口层 --> POSTROUTING链
-- 设备驱动 --> 网络传输出
```

![2615753-20211227002046380-1487109655](https://gitee.com/gengff/blogimage/raw/master/images/2615753-20211227002046380-1487109655.png)

#### 经过本机

- 报文经过路由并且发觉目的并不是本机,在经过 PREROUTING 链进入本机发现最终目的并不是本机时被转到 FORWORD 链后经过 POSTROUING 链转发出去。

```
数据进入 --> PREROUTING --> FORWARD --> POSTROUTING --> 出去
```

![2615753-20211227002203426-1208905019](https://gitee.com/gengff/blogimage/raw/master/images/2615753-20211227002203426-1208905019-20211230171628792.png)

## Iptables 流程图

![1](https://gitee.com/gengff/blogimage/raw/master/images/1.jpeg)

```bash
流入本机： A  --->  PREROUTING  --->  INPUT ---> B
流出本机：OUTPUT  --->  POSTROUTING  ---> B
经过本机： A ---> OUTPUT ---> POSTROUTING | ---> PREROUTING ---> FORWARD  ---> POSTROUTING ---> C ---> PREROUTING  ---> INPUT ---> B

filter :  INPUT 、FORWARD、 OUTPUT
nat : PREROUTING 、 INPUT、 OUTPUT、 POSTROUTING
raw : PREROUTING、 OUTPUT
mangle : PREROUTING INPUT FORWARD OUTPUT POSTROUTING
```

## Iptables 的使用

```bash
1、安装Iptables
	[root@m01 ~]# yum install iptables*

2、启动Iptables
	[root@m01 ~]# systemctl start iptables

3、关闭firewalld
	[root@m01 ~]# systemctl disable --now firewalld


格式：iptables -t 表名 选项 链名称 条件  动作

-t            #指定操作的表
-L, --list    #以列表形式显示当前的规则
-v            #显示数据包和数据包大小
-n            #不反解地址
-A, --append  #追加一条规则到指定链中
-I, --insert  #插入一条规则，插入到顶部
-F, --flush   #清除防火墙默认规则
-Z, --zero    #清空计数器（包数量 、包大小）
-D, --delete  #删除链中的规则
-R, --replace #修改
-S, --list-rules   #列出所有的规则
-N, --new-chain    #创建一个自定义 链
-X, --delete-chain #删除防火墙自定义链
-P, --policy       #指定链的默认策略
```

#### iptables 动作

```bash
ACCEPT   #将数据包放行，进行完此处理动作后，将不再比对其它规则，直接跳往下一个规则链。
REJECT   #拦阻该数据包，并传送数据包通知对方。
DROP     #丢弃包不予处理，进行完此处理动作后，将不再比对其它规则，直接中断过滤程序。
REDIRECT #将包重新导向到另一个端口，进行完此处理动作后，将会继续比对其它规则。
```

#### Iptables 基本的条件匹配

```bash
TCP(http)
UDP
ICMP(ping)
ALL
```

#### -s、-d 源地址、目标地址

```bash
-s  #源地址：发送请求的地址（指定匹配的源地址网段信息，或者匹配的主机信息）

-d  #目标地址：访问的地址（指定匹配的目标地址网段信息，或者匹配的主机信息）
```

#### --sport 源端口、--dport 目标端口

```bash
--sport  #源端口：发送请求的端口（表示指定源端口号信息）

--dport  #目标端口：访问的端口（表示指定目标端口信息）
```

#### -i、-o、-m、-j 动作

```bash
-i  #进来的网卡（指定匹配的进入流量接口信息 只能配置在INPUT链上）
-o  #出去的网卡（指定匹配的发出流量接口信息 只能配置在OUTPUT链上）
-m  #指定应用扩展模块参数
-p  #指定相应服务协议信息（tcp udp icmp all）
-j  #转发动作，指定对相应匹配规则执行什么操作（ACCEPT DROP REJECT REDIRECT）
    ACCEPT 允许通过
    DROP 直接拒绝
    REJECT 委婉拒绝
    REDIRECT 重定向
    MASQUERADE 地址伪装
    SNAT 如果内网主机访问外网而经过路由时，源IP会发生改变，这种变更行为就是SNAT
    DNAT 当外网的数据经过路由发往内网主机时，数据包中的目的IP (路由器上的公网IP) 将修改为内网IP，这种变更行为就是DNAT
```

## 案例

```
案例1：只允许22端口可以访问，其他端口全部无法访问。
iptables -t filter -A INPUT -p TCP --dport 22  -j ACCEPT
iptables -t filter -A INPUT -p TCP -j DROP

案例2：只允许22，80，443端口可以访问，其他端口全部无法访问。
iptables -t filter -A INPUT -p TCP --dport 22  -j ACCEPT
iptables -t filter -A INPUT -p TCP --dport 80  -j ACCEPT
iptables -t filter -A INPUT -p TCP --dport 443  -j ACCEPT
iptables -t filter -A INPUT -p TCP -j DROP

案例3：只允许22，80，443端口可以访问，其他端口全部无法访问，但是本机可以访问百度。

案例4：要求使用192.168.15.81能够通过22端口链接，但是其他的不行
iptables -t filter -A INPUT -p TCP -d 192.168.15.81 --dport 22  -j ACCEPT
iptables -t filter -A INPUT -p TCP -j DROP

案例5：只允许192.168.15.71能够通过22端口链接，其他的不行。
iptables -t filter -A INPUT -p  TCP -s 192.168.15.71  -d 192.168.15.81 --dport 22 -j ACCEPT
iptables -t filter -A INPUT -p TCP -j DROP

案例6：要求192.168.15.71对外部不可见
iptables -t filter -A INPUT -p TCP -d 192.168.15.71 -j DROP

案例7：要求使用eth0网卡的所有请求全部拒绝
iptables -t filter -A INPUT -p TCP -i etho -j DROP

使用172.16.1.71登录进来的窗口，不允许访问百度。
iptables -t filter -I OUTPUT -p TCP -o eth1 -j DROP

案例8：要求访问服务器的8080端口转发至80端口
iptables -t nat -A PREROUTING -p TCP --dport 8080 -j REDIRECT --to-port 80

案例9：要求只允许windows通过ssh连接192.168.15.81，其他的拒绝
iptables -t filter -I INPUT -p TCP -s 192.168.15.1 -d 192.168.15.81 --dport 22 -j ACCEPT
iptables -t filter -I INPUT -p TCP --dport 22 -j DROP

知识储备：
	查看本机端口占用的命令：
		netstat -nutlp
```

## 模块

```bash
拓展iptables的功能的。

-m : 指定模块

1、连续匹配多个端口（multiport）

	--dports  : 指定多个端口(不同端口之间以逗号分割，连续的端口使用冒号分割)。

2、指定一段连续的ip地址范围(iprange)
    --src-range from[-to]:	源地址范围
    --dst-range from[-to]	目标地址范围

3、匹配指定字符串(string)
    --string pattern	# 指定要匹配的字符串
    --algo {bm|kmp}		# 匹配的查询算法

4、根据时间段匹配报文(time)
    --timestart hh:mm[:ss]		# 开始时间
    --timestop hh:mm[:ss]		# 结束时间
    --monthdays day[,day...]	# 指定一个月的某一天
    --weekdays day[,day...]		# 指定周 还是  周天

5、禁ping, 默认本机无法ping别人 、别人无法ping自己
	--icmp-type {type[/code]|typename}
		echo-request  (8) 请求
		echo-reply    (0) 回应

6、限制链接数，并发连接数（connlimit）
    --connlimit-upto n		#  如果现有连接数小于或等于  n  则 匹配
    --connlimit-above n		#  如果现有连接数大于n 则匹配

7、针对 报文速率 进行限制。 秒、分钟、小时、天。

	--limit rate[/second|/minute|/hour|/day] # 报文数量
     --limit-burst number  # 报文数量（默认：5）

```

### 案例

```bash
1、要求将22,80,443以及30000-50000之间所有的端口向外暴露，其他端口拒绝

	iptables -t filter -A INPUT -p TCP -m multiport --dports 22,80,443,30000:50000 -j ACCEPT
	iptables -f filter -A INPUT -p TCP -j DROP

2、要求访问数据包中包含HelloWorld的数据不允许通过。
	iptables -t filter -A INPUT -p TCP -m string --string "HelloWorld" --algo kmp -j DROP

3、要求192.168.15.1 - 192.168.15.10之间的所有IP能够连接192.168.15.81，其他拒绝
	iptables -t filter -A INPUT -p TCP -m iprange --src-range 192.168.15.1-192.168.15.10 -j ACCEPT
	iptables -f filter -A INPUT -p TCP -j DROP

4、要求每天的12到13之间，不允许访问
	iptables -t filter -A INPUT -p TCP -m time  --timestart 4:00   --timestop 5:00 -j DROP

	必须使用UTC时间

5、要求别人不能ping本机，但是本机可以ping别人
	iptables -t filter -A INPUT -p TCP -m icmp --icmp-type "echo-request" -j DROP

6、要求主机连接最多有2个
	iptables -t filter -A INPUT -p TCP --dport 22 -m connlimit --connlimit-above 2 -j DROP

7、要求限制速率在500k/s左右
	iptables -t filter -A INPUT -p TCP -m limit 333/s -j ACCEPT
	iptables -t filter -A INPUT -p TCP -j DROP

8、只允许windows连接本机的iptables规则
iptables -t filter -A INPUT -p tcp -s 192.168.15.1 --dport 22 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 22 -j DROP

9、只允许192.168.15.0网段的IP连接本机，用两种方式实现。
iptables -t filter -A INPUT -p tcp -m iprange  --src-range 192.168.15.1-192.168.15.254 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 22 -j DROP

iptables -t filter -A INPUT -p tcp -i eth0 --dport 22 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 22 -j DROP

10、要求本机流出的数据中包含“元旦快乐”
iptables -t filter -A OUTPUT -p tcp --dport 80 -m string  --string "元旦快乐" --algo kmp -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 80 -j DROP

11、要求每天的九点到17点之间可以正常访问
iptables -t filetr -A INPUT -p tcp -m time  --timestart 1:00 --timestop 9:00 -j ACCEPT
iptables -t filter -A INPUT -p tcp -j DROP
```
