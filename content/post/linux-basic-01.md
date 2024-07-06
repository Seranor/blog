---
title: Linux安装、远程连接
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-01.html
toc: true
---

# 基础介绍

主要介绍企业中常用的服务器操作系统

- 什么是 Linux？

  类似于 windows，是一个服务器上使用的操作系统，Linux 支持多用户，多进程，多 CPU，多任务等功能，而且 Linux 是开源的，支持嵌入式等。
  <!-- more -->

## Linux 发展史

1969 年，美国贝尔实验室开发，Unix

- 优点：性能好
- 缺点：消耗资源大

1987 年，谭宁邦开发微内核 unix，主要用来教学

1991 年，芬兰 林纳斯-托瓦丝 在大学期间基于 unix 微内核开发了第一款 Linux 内核，并且开源，并且很快加入 FSF 基金会，

## Linux 核心概念

FSF 基金会，GPL 通用公共协议：开源的公共协议

GNU

Linux 的组成：Linux 内核—>系统软件—>个人软件 GNU Linux

# 虚拟机介绍

- 网络类型

  - 仅主机

    只能跟宿主主机进行连接

  - 桥接

    共享宿主主机网卡，跟宿主主机处于同一个局域网

  - NAT

    使用自己的虚拟网卡，有自己的一套网络

## Linux 发现版本

- RedHat/CentOS
- Ubuntu
- Debian

## 虚拟机软件

一般用来虚拟化一台主机的

- 虚拟机软件分类
  - vmware workstation（个人使用，或者开发者使用）
  - KVM 一般用在云服务平台上
  - ESXI 部署在物理主机上

`![微信截图_20211208102719](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208102719.png)

# 网卡设置

<img src="https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103459.png" alt="微信截图_20211208103459" style="zoom: 50%;" />

![微信截图_20211208103520](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103520.png)

<img src="https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103635-8967853.png" style="zoom:50%;" />

![微信截图_20211208103657](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103657.png)

![微信截图_20211208103734](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103734.png)

![微信截图_20211208103839](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103839.png)

![微信截图_20211208103909](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208103909.png)

![微信截图_20211208104113](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104113.png)

![微信截图_20211208104202](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104202.png)

![微信截图_20211208104214](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104214.png)

![微信截图_20211208104400](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104400.png)

![微信截图_20211208104445](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104445.png)

# 安装 Linux 系统

![微信截图_20211208104743](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104743.png)

![微信截图_20211208104909](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104909.png)

![微信截图_20211208104924](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208104924.png)

![微信截图_20211208105046](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208105046.png)

![微信截图_20211208105219](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208105219.png)

![微信截图_20211208105400](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208105400.png)

![43C18B4F-1B0A-42E9-BB78-D3E5682D4106](https://gitee.com/gengff/blogimage/raw/master/images/43C18B4F-1B0A-42E9-BB78-D3E5682D4106.png)

![AE1A21DF-E10A-4841-9DBB-A85212501759](https://gitee.com/gengff/blogimage/raw/master/images/AE1A21DF-E10A-4841-9DBB-A85212501759.png)

![微信截图_20211208105638](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208105638.png)

<img src="https://gitee.com/gengff/blogimage/raw/master/images/WX20211208-170046@2x.png" alt="WX20211208-170046@2x" style="zoom: 50%;" />

![微信截图_20211208110204](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110204.png)

![微信截图_20211208110217](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110217.png)

![微信截图_20211208110341](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110341.png)

![20211208110532](https://gitee.com/gengff/blogimage/raw/master/images/20211208110532.png)

![微信截图_20211208110550](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110550.png)

![微信截图_20211208110602](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110602.png)

![微信截图_20211208110629](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110629.png)

![微信截图_20211208110745](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208110745.png)

![WeChatb1cc04bfcdf1fe355a7179e717e66de9](https://gitee.com/gengff/blogimage/raw/master/images/WeChatb1cc04bfcdf1fe355a7179e717e66de9.png)

![微信截图_20211208111408](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208111408.png)

![微信截图_20211208113903](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208113903.png)

![7590A4DC-6E12-4888-A23A-287833B9853D](https://gitee.com/gengff/blogimage/raw/master/images/7590A4DC-6E12-4888-A23A-287833B9853D.png)

![微信截图_20211208114033](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114033.png)

![微信截图_20211208114119](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114119.png)

![微信截图_20211208114253](https://gitee.com/gengff/blogimage/raw/master/images/微信截图_20211208114253.png)

![微信截图_20211208114459](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114459.png)

![4AE29186-308D-46BE-B840-0246191C2BF6](https://gitee.com/gengff/blogimage/raw/master/images/4AE29186-308D-46BE-B840-0246191C2BF6.png)

![70388F5F-6915-4105-B8E7-61000F4D8607](https://gitee.com/gengff/blogimage/raw/master/images/70388F5F-6915-4105-B8E7-61000F4D8607.png)

![C9FB0F45-D71B-4207-8631-C60196830281](https://gitee.com/gengff/blogimage/raw/master/images/C9FB0F45-D71B-4207-8631-C60196830281.png)

![微信截图_20211208114651](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114651.png)

![微信截图_20211208114833](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114833.png)

![微信截图_20211208114846](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114846.png)

![微信截图_20211208114953](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208114953.png)

![微信截图_20211208115212](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208115212.png)

![微信截图_20211208115358](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208115358.png)

![微信截图_20211208115413](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208115413.png)

![微信截图_20211208115438](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208115438.png)

![微信截图_20211208115855](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208115855.png)

![微信截图_20211208115921](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208115921.png)

![微信截图_20211208120050](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120050.png)

![](https://gitee.com/gengff/blogimage/raw/master/images/微信截图_20211208120113.png)

![微信截图_20211208120130](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120130.png)

![微信截图_20211208120201](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120201.png)

![微信截图_20211208120239](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120239.png)

![微信截图_20211208120311](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120311.png)

![微信截图_20211208120427](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120427.png)

![微信截图_20211208120907](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208120907.png)

![WX20211208-204050@2x](https://gitee.com/gengff/blogimage/raw/master/images/WX20211208-204050@2x.png)

![微信截图_20211208121606](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208121606.png)

# 连接 X-shell

![微信截图_20211208122027](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208122027.png)

![微信截图_20211208122101](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208122101.png)

![微信截图_20211208122135](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208122135.png)

![微信截图_20211208122151](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208122151.png)

![微信截图_20211208122205](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208122205.png)

![微信截图_20211208122218](https://gitee.com/gengff/blogimage/raw/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211208122218.png)

# mac 网卡配置

![wangka1](https://gitee.com/gengff/blogimage/raw/master/images/wangka1.png)

![wangka2](https://gitee.com/gengff/blogimage/raw/master/images/wangka2.png)

![](https://gitee.com/gengff/blogimage/raw/master/images/wangka3.png)

![wangka44](https://gitee.com/gengff/blogimage/raw/master/images/wangka44.png)

![wangka5](https://gitee.com/gengff/blogimage/raw/master/images/wangka5.png)

![wangka6](https://gitee.com/gengff/blogimage/raw/master/images/wangka6.png)

![wangka7](https://gitee.com/gengff/blogimage/raw/master/images/wangka7.png)

![wangka8](https://gitee.com/gengff/blogimage/raw/master/images/wangka8.png)

![wangka9](https://gitee.com/gengff/blogimage/raw/master/images/wangka9.png)

![wangka10](https://gitee.com/gengff/blogimage/raw/master/images/wangka10.png)

![wangka11](https://gitee.com/gengff/blogimage/raw/master/images/wangka11.png)
