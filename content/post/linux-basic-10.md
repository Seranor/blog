---
title: Linux文本处理和find命令
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-10.html
toc: true
---

# find 命令

- 根据文件的名称或者属性查找文件。

- 为什么要有文件查找，因为很多时候我们可能会忘了某个文件所在的位置，此时就需要通过 find 来查找。
find 命令可以根据不同的条件来进行查找文件，例如：文件名称、文件大小、文件修改时间、属主属组、权限、等等方式。同时 find 命令是 Linux 下必须掌握的
<!-- more -->

## 语法格式

```bash
find [查找范围(路径)]  [参数]  [参数相关匹配值]  [指令（-print）]
```

## 参数及示例

### 1、find 名称查找

```bash
-name : 按照文件的名字查找文件
   *   #通配符
-iname : 按照文件的名字查找文件(-i忽略大小写)

# 1.创建文件
touch /etc/sysconfig/network-scripts/{ifcfg-eth1,IFCFG-ETH1}

# 2.查找/etc目录下包含ifcfg-eth0名称的文件
[root@localhost ~]# find /etc -name "ifcfg-eth1"

# 3.-i 忽略大小写
[root@localhost ~]# find /etc -iname "ifcfg-eth1"

# 4.查找/etc目录下包含ifcfg-eth名称所有文件
[root@localhost ~]# find /etc/ -name "ifcfg-eth*"
[root@localhost ~]# find /etc -iname "ifcfg-eth*"
```

### 2、find 大小查找

```bash
-size n(单位) : 按照文件的大小查询文件
      +n   #大于n个单位
      -n   #小于n个单位
       n   #等于n个单位

# 注：find查找同时打印出隐藏文件
# 注：n 必须是整数，不能是小数

# 1.查找大于5M的文件
[root@localhost ~]# find /etc -size +5M

# 2.查找等于5M的文件
[root@localhost ~]# find /etc -size 5M

# 3.查找小于5M的文件
[root@localhost ~]# find /etc -size -5M
```

### 3、find 时间查找

```bash
-atime : 访问时间（cat）
-ctime : 文件变更时间（修改了位置（mv）、所属组、所属用户）

-mtime : 按照修改时间去查询(包含创建时间)
     +n  #n 天以前
     -n  #n 天以内
      n  #

# 1.创建测试文件(后期shell会讲)
[root@localhost ~]# for i in {01..28};do date -s  201904$i && touch file-$i;done

# 2.查找3天以前的文件(不会打印当天的文件)
[root@localhost ~]# find ./ -iname "file-*" -mtime +7

# 3.查找3天以内的文件，不建议使用(会打印当天的文件)
[root@localhost ~]# find ./ -iname "file-*" -mtime -7

# 4.查找3天以前（一天之内）的文件(不会打印当天的文件)
[root@localhost ~]# find ./ -iname "file-*" -mtime 7


# 5.本地文件保留最近7天的备份文件, 备份服务器保留3个月的备份文件(实际使用方案)
find /backup/ -iname "*.bak" -mtime +7 -delete
find /backup/ -iname "*.bak" -mtime +90 -delete
```

### 4、find 用户查找

```bash
  -user : 按照用户的属主查询
  -group : 按照用户的属组查询

# 查找属主是tony
[root@localhost ~]# find /home -user tony

# 查找属组是admin
[root@localhost ~]# find /home -group admin

# 查找属主是tony, 属组是admin
[root@localhost ~]# find /home -user tony -group admin

# 查找属主是tony, 并且属组是admin
[root@localhost ~]# find /home -user tony -a -group admin

# 查找属主是tony, 或者属组是admin
[root@localhost ~]# find /home -user tony -o -group admin

# 查找没有属主
[root@localhost ~]# find /home -nouser

# 查找没有属组
[root@localhost ~]# find /home -nogroup

# 查找没有属主或属组
[root@localhost ~]# find /home -nouser -o -nogroup
```

### 5、find 类型查找

```bash
-type : 按照文件的类型查询
   f(-) #普通文件
      d #目录
      l #链接文件
      s #套接字文件
      p #管道文件
      c #字符文件
      b #磁盘文件

# f 普通文件
[root@localhost ~]# find /dev -type f
# d 目录
[root@localhost ~]# find /dev -type d
# l 链接文件
[root@localhost ~]# find /dev -type l
# s 套接字文件
[root@localhost ~]# find /dev -type s
# p 管道文件
[root@localhost ~]# find /dev -type p
# c 字符文件
[root@localhost ~]# find /dev -type c
# b 磁盘文件
[root@localhost ~]# find /dev -type b
```

### 6、find 权限查找

```bash
-perm ：按照文件的权限查询

#精切匹配644权限
[root@localhost ~]# find . -perm 644 -ls

#包含444权限即可
[root@localhost ~]# find . -perm -444  -ls

#查找全局可写(每位权限必须包含w)
[root@localhost ~]# find . -perm -222 -ls

#包含set uid
[root@localhost ~]# find  /usr/sbin -perm -4000 -ls

#包含set gid
[root@localhost ~]# find  /usr/sbin -perm -2000 -ls

#包含sticky
[root@localhost ~]# find  /usr/sbin -perm -1000 -ls
```

### 7、find 逻辑运算符

```bash
-a : 与（可以省略，默认时并且）
-o : 或
-not|! : 非

# 1.查找当前目录下，属主不是hdfs的所有文件
[root@localhost ~]# find . -not -user hdfs
[root@localhost ~]# find . ! -user hdfs

# 2.查找当前目录下，属主属于hdfs，且大小大于300字节的文件(c代表字节)
[root@localhost ~]# find . -type f -a -user hdfs -a -size +300c

# 3.查找当前目录下的属主为hdfs或者以xml结尾的普通文件
[root@localhost ~]# find . -type f -a \( -user hdfs -o -name '*.xml' \)


-inum : 根据index node号码查询
-maxdepth : 查询的目录深度（必须放置与第一个参数位）



知识储备：
 dd : 生成文件
    if   #从什么地方读
    of   #写入到什么文件
    bs   #每次写入多少内容
    count #写入多少次

案例：
  案例1：查询/etc目录下hosts文件
    [root@localhost ~]# find /etc/ -name 'hosts'
       /etc/hosts
  案例2：查询/etc目录下名称中包含hosts文件
    [root@localhost ~]# find /etc/ -name '*hosts*'

  案例3：要求把/etc目录下，所有的普通文件打包压缩到/tmp目录
    [root@localhost /tmp]# tar -czPf /tmp/etcv2.tar.gz `find /etc/ -type f | xargs`



知识储备
   |    #前面一个命令的结果交给后面一个命令处理
 xargs  #把处理的文本变成以空格分割的一行
   ``   #提前执行命令，然后将结果交给其他命令来处理

 #xargs将前者命令查找到的文件作为一个整体传递后者命令的输入
  [root@localhost ~]# touch file.txt
  [root@localhost ~]# find . -name "file.txt" |xargs rm -f
  [root@localhost ~]# find . -name "file.txt" |xargs -I {} cp -rvf {} /var/tmp
```

### 8、find 指令

- -print ：打印结果集
- -ls ： 打印结果集详情
- -delete : 删除结果集
- -exec : 将 find 处理好的结果集进行下一步处理
- -ok : 将 find 处理好对结果集进行下一步处理（交互）

```bash
# 打印结果集
[root@localhost dev]# find / -type s -ctime -3 -print

# 打印结果集详情
[root@localhost dev]# find / -type s -ctime -3 -ls

# 删除结果集
[root@localhost test]# ls
 Abc  abc1  Abc1  abc10  abc2  abc23  abc3  abc4  abc5
[root@localhost test]# find ./ -iname "abc?" -delete


# 对结果集进行下一步处理
# 格式
	find 路径 参数  参数表达式  -exec  命令 {} \;
 [root@localhost test]# find ./ -iname "abc"
  ./Abc
 [root@localhost test]# find ./ -iname "abc" -exec ls -l {} \;
  -rw-r--r-- 1 root root 0 Mar 10 09:44 ./Abc
 [root@localhost test]#
 [root@localhost test]# find ./ -iname "abc"
  ./Abc
 [root@localhost test]# find ./ -iname "abc" -exec rm -rf {} \;
 [root@localhost test]# ls
  abc10  abc23


# 对结果集进行下一步处理（交互）
 # 格式
  find 路径 参数  参数表达式  -ok   命令 {} \;

 [root@localhost test]# find ./ -iname "abc10"
  ./abc10
 [root@localhost test]# find ./ -iname "abc10" -ok rm {} \;
  < rm ... ./abc10 > ? n
 [root@localhost test]# ls
  abc10  abc23
 [root@localhost test]# find ./ -iname "abc10" -ok rm {} \;
  < rm ... ./abc10 > ? y
 [root@localhost test]# ls
  abc23
```

**find 相关练习题**

```bash
1.查找/tmp目录下，属主不是root，且文件名不以f开头的文件
2.查找/var目录下属主为root，且属组为mail的所有文件
3.查找/var目录下不属于root、lp、gdm的所有文件
4.查找/var目录下最近一周内其内容修改过，同时属主不为root，也不是postfix的文件
5.查找/etc目录下大于1M且类型为普通文件的所有文件
6.将/etc/中的所有目录(仅目录)复制到/tmp下，目录结构不变
7.将/etc目录复制到/var/tmp/,/var/tmp/etc的所有目录权限777/var/tmp/etc目录中所有文件权限666
8.保留/var/log/下最近7天的日志文件,其他全部删除
9.创建touch file{1..10}10个文件, 保留file9,其他一次全部删除
10.解释如下每条命令含义
mkdir /root/dir1
touch /root/dir1/file{1..10}
find /root/dir1 -type f -name "file5"
find /root/dir1 ! -name "file5"
find /root/dir1 -name "file5" -o -name "file9"
find /root/dir1 -name "file5" -o -name "file9" -ls
find /root/dir1 \( -name "file5" -o -name "file9" \) -ls
find /root/dir1 \( -name "file5" -o -name "file9" \) -exec rm -rvf {} \;
find /root/dir1  ! \( -name "file4" -o -name "file8" \) -exec rm -vf {}  \;
```

# 字符处理命令

## 1、sort 命令

- 用于将文本文件内容加以排序

```bash
sort [参数] 需要排序的对象
 参数：
  -n # 依照数值的大小排序
  -r # 以相反的顺序来排序
  -k # 以某列进行排序
  -t # 指定分割符，默认是以空格为分隔符

	cat 3.txt | sort -n -r -k3 -t '|'
# 1.首先创建一个文件，写入一写无序的内容
[root@localhost ~]# cat >> file.txt <<EOF
 b:3
 c:2
 a:4
 e:5
 d:1
 f:11
 EOF

# 2.使用sort下面对输出的内容进行排序
[root@localhost ~]# sort file.txt
 a:4
 b:3
 c:2
 d:1
 e:5
 f:11

#结果并不是按照数字排序，而是按字母排序。
#可以使用-t指定分隔符, 使用-k指定需要排序的列。
[root@localhost ~]# sort -t ":" -k2 sort.txt
 d:1
 f:11 #第二行为什么是11？不应该按照顺序排列？
 c:2
 b:3
 a:4
 e:5

#按照排序的方式, 只会看到第一个字符,11的第一个字符是1, 按照字符来排序确实比2小。
#如果想要按照数字的方式进行排序, 需要使用 -n参数。
[root@localhost ~]# sort -t ":" -n -k2 p.txt
 d:1
 c:2
 b:3
 a:4
 e:5
 f:11


#测试案例，下载文件http://fj.xuliangwei.com/public/ip.txt，对该文件进行排序
[root@localhost ~]# sort -t. -k3.1,3.1nr -k4.1,4.3nr ip.txt

```

## 2、uniq 命令

- 用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用

```bash
uniq [参数] 需要去重的对象
 参数：
  -c # 在每列旁边显示该行重复出现的次数
  -d # 仅显示重复出现的行列
  -u # 仅显示出一次的行列

# 1.创建一个file.txt文件:
[root@localhost ~]# cat file.txt
 abc
 123
 abc
 123

# 2.uniq需要和sort一起使用, 先使用sort排序, 让重复内容连续在一起
[root@localhost ~]# cat file.txt | sort
 123
 123
 abc
 abc

# 3.使用uniq去除相邻重复的行
[root@localhost ~]# cat file.txt |sort|uniq
 123
 abc

# 4.-c参数能统计出文件中每行内容重复的次数
[root@localhost ~]# cat file.txt |sort|uniq -c
  2 123
  2 abc
```

## 3、cut 命令

- cut 命令用来显示行中的指定部分，删除文件中指定字段

```bash
cut [参数] [操作对象]
 参数：
  -d # 指定分隔符
  -f # 指定显示的列；几列的内容，取第几列，-f3,6三列和六列
  -c # 按字符取（空格也算）

#echo "Im xlw, is QQ 552408925" >file.txt   #过滤出文件里 xlw以及552408925

#实现上述题目几种思路
# cut -d " " -f2,5 file.txt
# cut -d " " -f2,5 file.txt |sed 's#,##g'
# sed 's#,# #g' file.txt | awk -F " " '{print $2 " " $5}'
# awk  '{print $2,$5}' file.txt |awk -F ',' '{print $1,$2}'
# awk -F  "[, ]" '{print $2,$6}' file.txt
# awk -F '[, ]+' '{print $2,$5}' file.txt
```

## 4、tr 命令

- tr 可以用来删除一段信息当中的文字，或是进行文字信息的替换

```bash
tr  [参数]  [操作对象]
参数：
  -d # 删除字符
  -s # 替换重复的字符

[root@localhost ~]# cat /etc/passwd | tr "root" "ROOT"
 ROOT:x:0:0:ROOT:/ROOT:/bin/bash
```

## 5、wc 命令

- 统计，计算数字

```bash
wc  [参数]  [操作对象]
 参数：
  -c # 统计文件的Bytes数
  -l # 统计文件的行数
  -w # 统计文件中单词的个数，默认以空白字符做为分隔符

注：在Linux系统中，一段连续的数字或字母组合为一个词。

# wc -l /etc/fstab      #统计/etc/fstab文件有多少行
# wc -l /etc/services   #统计/etc/services 文件行号


#扩展方法
# grep -n ".*" /etc/services  | tail -1
# awk '{print NR $0}' /etc/services | tail -1
# cat -n /etc/services  | tail -1
```
