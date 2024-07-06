---
title: Linux命令总结
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-12.html
toc: true
---

## 帮助相关

<!-- more -->

- man 查看普通命令的帮助
- help 查看内置命令的帮助
- info 查看一个命令更多的信息

## 关机重启

- shutdown 关机重启
  - -r （reboot）重启
  - -h （halt）关机
- halt 关机 cpu 停止工作
- poweroff 关机断电
- init 切换运行级别
  - init 0 关机
  - init 6 重启
- reboot 重启
- systemctl
  - reboot 重启
  - poweroff 关机
- sync 把数据从 buffer 写回磁盘

## 目录相关

- pwd 打印工作目录

- cd 切换工作目录

  - -上一次所在目录

  - . 当前目录

  - .. 上级目录 d

  - ~ 用户家目录

- tree 以树形结构显示目录或文件

  - -L （level）指定层数

  - -d 只显示目录

- mkdir 创建目录

  - -p 递归创建目录

- ls 显示目录下的内容

  - -l （long）长格式显示文件属性\*
  - -a 显示所有文件，包含隐藏文件\*
  - -d 只显示目录本身内容\*
  - -i 显示 inode 节点\*
  - -h （human）人类可读形式显示
  - -p 目录结尾加斜线，区分目录和文件
  - -F 不同文件结尾加不同标识，目录结尾加斜线

  - --color=auto 显示不同文件的颜色

  - --time-style 显示修改时间的格式

    - long-iso 年月日时分

    - iso 月日时分

  - -r （reverse）反转排序

  - -t 按修改时间排序

- cp 拷贝 -copy files and directories
  拷贝 文件 和 目录

  - -r 复制目录
  - -p 保持文件或目录属性 (属主，属组，所属用户)
  - -a 相当于 dpr
  - -i 是否覆盖确认
  - -d 保持文件中软连接的属性

- rm 删除文件或目录

  - -f 强制删除
  - -r 递归删除

- mv 移动文件或目录或改名

## 文件内容

- touch 创建文件或更新文件时间戳
- vi/vim 文本编辑器
- echo 显示输出文本内容
  - -n 不换行输出
  - -e 支持转义字符输出
- printf 格式化打印字符串
- cat 查看文件内容
  - -n 显示行号
- tac 按行翻转文件内容
- rev 左右按字符翻转行的内容
- more 分页查看文件内容
- less 分页查看文件内容
  - -N 显示行号
- head 显示文件内容头部
  - -n 前 n 行 n 可省
- tail 显示文件内容尾部
  - -n 后 n 行 n 可省
  - -f 跟踪文件尾部的变化
- tr 替换或删除字符
- cut 取列
  - -d 指定分隔符
  - -f 指定哪列 连续多列可用- 多列可用，逗号
  - -c 按字符取内容
- diff 文本比较
- vimdiff 文本图形化比较

## 文件相关

- file 查看文件类型
- ln 创建
  - -s （soft）创建软链接
- which 查命令所在的路径
- whereis 查找命令，源码，帮助等路径
  - -b 查二进制命令
- locate 查找文件及帮助相关，从 updatedb 对应的数据库里查
- find 查找目录下的文件
  - -name 按文件名查找
  - -type 按类型查找
  - -exec 对查找的结果在处理
  - -mtime 按修改时间查找
  - -perm 查权限
  - -size
- xargs 从标准输入执行命令
  - -n 数字，几个东西在一组
  - -d 指定分隔符，不指定默认是空格
  - -i 把{}当做前面查找的结果
- stat 查看文件属性
  - -c 获取指定文件属性的一部分
    - %A 显示字符权限

## 用户管理

- id 查看用户身份

- whoami 查看当前登录的用户

- w 谁登陆了 干什么了

- last 显示登录过的用户信息列表

- lastlog 查看最近登录过的用户报告

- useradd 添加普通用户

  - -u 指定 UID
  - -s 指定登录的 SHELL 解释器
  - -M 不创建家目录
  - -g 指定所属的组
  - -c 添加用户说明
  - -d 指定家目录
  - -e 设定登录截止日期

- userdel 删除用户

  - -r 递归删除用户目录及下面内容

    ​ 备份或确认家目录下无有用内容

- usermod 修改用户属性

  - -u 指定 UID
  - -s 指定登录的 SHELL 解释器
  - -M 不创建家目录
  - -g 指定所属的组
  - -c 添加用户说明
  - -d 指定家目录
  - -e 设定登录截止日期

- passwd 修改密码

  - --stdin 从标准输入接收密码并设置

- chpasswd 从标准输入批量更改用户密码

- groupadd 添加用户组

  - -g 指定组 id

- groupdel 删除用户组

- chage 查看和修改密码属性

  - -l list 列表显示用户的密码信息
  - -E 修改账户过期时间

- su 用户身份切换

  - -携带环境变量登录
  - -c 以指定用户身份执行命令

- sudo 允许指定用户执行某命令期间拥有 root 角色权限

  - -l 查看获得的权限

- visudo 编辑 sudo 配置文件的命令

  - -c 检查配置文件语法

## 其他相关

- date 显示系统时间和日期
  - -s 修改时间
  - -d 指定过去或未来格式
- alias 查看或设置别名
- unalias 取消别名
- runlevel 查看运行级别
- init 切换运行级别
- getenforce 查看 selinux 状态
- setenforce 设置 selinux 状态
- md5sum 给文件设置指纹（计算和检查 MD5 数字信息）
- du 文件或目录大小
  - -s 显示总大小
  - -h 人类可读

## 打包压缩

- tar 打包压缩
  - -z 压缩
  - -c 创建
  - -v 输出打包过程
  - -f 文件
  - -t 查看文件
  - -C 指定解压的路径
  - -x 解压
  - -h 跟随软链接
  - --exclude 排除不打包的文件
  - -X 从文件中排除不打包的文件

## 磁盘管理

- df 查看文件系统
  - -i （inode）信息
  - -h 以人类可读形式查看 block 信息
  - -T 查看文件系统
- fdisk MBR 磁盘查看和分区工具（小于 2T）
  - -l 列表
- parted GPT 磁盘分区工具（常用于大于 2T）
- dd 创建一个虚拟文件系统
- partprobe 将分区信息通知内核，真正生效
- mkfs 格式化（本质创建文件系统）
  - -t 指定类型 -t ext4（mkfs.ext4）
  - -b 指定 block 大小
  - -i 指定 inode 大小
- mount 挂载文件系统
  - -t type 指定文件类型
  - -o 挂载的选项 mount -o rw，remount /
  - -a all 挂载所有磁盘
- umount 卸载文件系统
  - -lf 强制卸载
- blkid 查看块设备属性（UUID,FSTYPE）
- dumpe2fs 查看 ext 文件系统细节
- fsck 检查和修复 ext 文件系统（好的磁盘不能操作），类似 e2fsck
  - -a 修复磁盘
- xfs*info (xfs*一堆) 查看 xfs 文件系统细节
- xfs_repair 检查和修复 xfs 文件系统

## 三剑客

- grep 过滤
  - --color=auto 过滤的内容加色
  - -v （invert）取反
  - -i （ignore 忽略）不区分大小写
  - -n （number 数字）对输出的内容显示在源文件中的行号 显示行号
  - -w （word）按单词为单位过滤
  - -o 只输出匹配的内容
  - -E （extend）扩展的 grep，即 egrep
  - -A （after）显示过滤的字符串和它之后的多少行
  - -B （before）显示过滤的字符串和它之前的多少行
  - -C （context）显示过滤的字符串和它之前之后的多少行
  - -p 用于过滤 Perl 兼容正则表达式
- sed 流编辑器
  - 参数
    - -n 取消命令的默认输出
    - -i 直接修改文件内容，而不是输出到终端
    - -r 支持扩展正则
  - sed 的内置命令字符说明
    - s：替换
    - g：全局
    - p：打印
    - d：删除
    - a：追加
    - i：插入
- awk 是命令操作也可以作为编程语言，处理字符串
  - -F 指定分隔符
  - 列表示：$1 第一列 $2 第二列 以此类推……
  - $0 整行
  - $NF 最后一列
  - $（NF-1）倒数第二列
  - NR 行号

## 文件属性

- chmod 修改文件权限
  - -R 递归修改
- chown 改变文件用户和组
  - -R 递归修改
- chgrp 修改用户组
- chattr 设置文件属性
  - +i 锁定文件
  - -i 解锁文件
  - +a 只能追加不能删除文件和内容
  - -a 解锁
- lsattr 查看文件属性

## 网络服务命令

- 定时任务
  - crontab
    - -l 列表
    - -e 编辑定时任务
    - -u 查看特定用户下的定时任务

## 软件安装

- rpm 包管理器
  安装，卸载，升级，查询和验证
  - -i 安装 install
  - -v 显示安装过程
  - -h 用“#”显示安装进度条
  - -U 升级软件包
  - -e 卸载软件包
  - --nodeps 忽略依赖
  - -q 查询
  - -a 所有
  - -l 显示软件包中的所有文件列表
  - -f 查询文件或命令属于哪个软件包
- yum 安装 rpm 包自动解决依赖工具
  - install 安装软件包
  - list 获取软件包名
  - search 模糊查找软件包名
  - groupinstall 安装组包
  - grouplist 获取组包名称列表
  - list installed 查已安装软件
  - provides 根据命令配置查软件包
  - remove 移除软件包（禁止使用）
  - repolist 列出启用的 yum 源
  - repolist all 列出所有 yum，包括禁用的 yum 源也列出

## 网络命令

- ifconfig 查看设置 IP
- ip 查看和设置网络和 IP
- ping 检查网络是否通畅
- traceroute 查看到达主机的网络路由信息
  - -d 不做反向解析
- route 查看设置网关、路由
- -host 主机路由 -net 网络路由 默认网关 default gw
  - add 添加
  - del 删除
- telnet 检测远程端口是否通畅
- lsof
  - -d
  - -i 查看端口

## Bash 内置命令

- history
  - -c 清所有
  - -d 指定数字清
- ulimit
  - -n 查文件描述符大小

## 主机命令

- hostname 查看设置主机名
- hostnamectl 设置主机名（C7）
- hostname 修改主机名
- hostnamectl CentOS7 永久修改主机名
