---
title: Linux三剑客
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-11.html
toc: true
---

Linux 中最重要的三个命令在业界被称为“**三剑客**”，它们是 grep，sed，awk。

我们知道 Linux 下一切皆文件，对 Linux 的操作就是对文件的处理，那么怎么能更好的处理文件呢？这就要用到我们的三剑客命令。

<!-- more -->

- grep ：过滤文本
- sed : 修改文本
- awk : 处理文本

使用这三个工具可以提升运维效率，熟练掌握好正则表达式是使用`Linux三剑客`的前提，在说三剑客前我们要插入一个小插曲就是“正则表达式”。在掌握好正则表达式后，将具体讲解三剑客的用法。

# 正则表达式

正则表达式：REGular EXPression, REGEXP。我们通过特定的字符串匹配模板，来获取到所需的内容。

Linux 三剑客以正则表达式作为基础，而在 Linux 系统中，支持两种正则表达式：

- 标准正则表达式
- 扩展正则表达式

```bash
标准正则表达式：

 ^    #以某字符开头
 $    #以某字符结尾
 .    #匹配除换行符之外的任意单个字符
 *    #匹配前导字符的任意个数
 []   #某组字符串的任意一个字符
 [^]  #取反
 [a-z]    #匹配小写字母
 [A-Z]    #匹配大写字母
 [a-zA-Z] #匹配字母
 [0-9]    #匹配数字
 \      #取消转义
 ()     #分组
 \n     #代表第n个分组


扩展正则表达式：
 {}     #匹配的次数
 {n}    #匹配n次
 {n,}   #至少匹配n次
 {n,m}  #匹配 n 到 m 次
 {,m}   #最多匹配m次
  +     #匹配至少有一个前导字符
  ?     #匹配一个或零个前导字符
  |     #或
```

# linux 三剑客之 grep

- 文本过滤器（根据文本内容过滤文件）
- grep 命令家族由 grep, egrep, fgrep 三个子命令组成，适用于不同的场景。具体如下：
  - 命令描述
    - grep 原生的 grep 命令，使用“标准正则表达式”作为匹配标准。
    - egrep 扩展的 grep 命令，相当于`$(grep -E)`，使用“扩展正则表达式”作为匹配标准。
    - fgrep 简化版的 grep 命令，不支持正则表达式，但搜索速度快，系统资源使用率低。

```bash
语法格式：
  grep [参数] [匹配规则] [操作对象]

 参数：
   -n  #过滤文本时，将过滤出来的内容在文件内的行号显示出来
   -A  #匹配成功之后，将匹配行的后n行显示出来
   -B  #匹配成功之后，将匹配行的前n行显示出来
   -C  #匹配成功之后，将匹配行的前后各n行显示出来
   -c  #只显示匹配成功的行数
   -o  #只显示匹配成功的内容
   -v  #显示不包含匹配文本的所有行（反向过滤）
   -q  #静默输出(禁止输出任何结果，已退出状态表示搜索是否成功)
   -i  #搜索时，忽略大小写
   -l  #匹配成功之后，将文本的名称打印出来
   -h	 #查询多文件时不显示文件名
   -b  #打印匹配行距文件头部的偏移量，以字节为单位
   -s	 #不显示不存在、没有匹配文本的错误信息
   -w  #匹配整词
   -x  #匹配整行
   -R|-r #递归匹配
   -E    #使用扩展正则,等价于 egrep


知识储备：
  $?  #上一行命令执行的结果，0代表执行成功，其他数字代表执行失败。
  wc  #匹配行数
   参数：
		-l  #打印匹配行数
		-c  #打印匹配的字节数

案例：
 # 在/etc目录下，有多少个文件包含root。
  [root@localhost ~]# grep -rl 'root' /etc/ | wc -l
   130


 # 在/etc/passwd文件中，匹配以ftp开头的行
  [root@localhost ~]# grep '^root' /etc/passwd
   root:x:0:0:root:/root:/bin/bash


 # 在/etc/passwd文件中，匹配以bash结尾的行,-n显示行号
  [root@localhost ~]# grep -n 'bash$' /etc/passwd
   1:root:x:0:0:root:/root:/bin/bash
   21:test:x:1001:1001::/home/test/:/bin/bash
   22:gf:x:1002:1002::/home/gf:/bin/bash
   28:tony:x:1004:1004::/home/tony:/bin/bash
   29:user:x:1006:1001::/home/user:/bin/bash


 # 匹配本机中有哪些ip
  [root@localhost ~]# ip a | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'
   127.0.0.1
   192.168.15.100
   192.168.15.255


 # 要求将/etc/fstab中的去掉包含 # 开头的行，且要求 # 后至少有一个空格
  [root@localhost ~]# grep -vE '^#\ +' /etc/fstab

   #
   #
   #
   /dev/mapper/centos-root /      xfs     defaults    0 0
   UUID=9f8a98b0-805c-4adf-b9ef-517a2b527f89 /boot   xfs   defaults    0 0


 # 找出文件中至少有一个空格的行
  [root@localhost ~]# grep -E '\ +' 1.txt
   11 11
   22 22
   5 5 5 5


 # 将 nginx.conf 文件中以#开头的行和空行，全部删除
  [root@localhost ~]# grep -vE '^\ *#|^$' /etc/nginx/nginx.conf
```

# linux 三剑客之 sed

- sed 是一个流式编辑器，在处理行内容时功能十分强大。

  - 定位到某一行，将某一行的某一部分给替换掉

  - 定位到某一行，然后删除

  - 定位到某一行，在该行后添加新的配置

```bash
语法格式
  sed [参数] '处理规则' [操作对象]

 参数
   -n  #取消默认输出
   -e  #允许多项编辑
   -i  #直接编辑源文件（把流向屏幕的内容写到文件中）
   -r  #支持扩展正则
   -f  #指定sed匹配规则脚本文件

 定位
# 1、行号定位法，指定行号定位
  # 不写代表定位所有行
  [root@localhost ~]# sed '' 1.txt
   1111
   2222
   3333
   4444
   5555

  # 3定位到第三行
  [root@localhost ~]# sed '3d' 1.txt
   1111
   2222
   4444
   5555

  # 2，3从第二行到第三行
  [root@localhost ~]# sed '2,3d' 1.txt
   1111
   4444
   5555


# 2、正则定位法，指定正则定位。
  [root@localhost ~]# sed '' 2.txt
   gen111
   222gen
   333gen333
   444xxx444
   555gen555gen


  # 删除包含gen的行
  [root@localhost ~]# sed '/gen/d' 2.txt
   444xxx444


  # 删除以gen开头的行
  [root@localhost ~]# sed '/^gen/d' 2.txt
   222gen
   333gen333
   444xxx444
   555gen555gen

  # 删除以gen结尾的行
	[root@localhost ~]# sed '/gen$/d' 2.txt
   gen111
   333gen333
   444xxx444


# 3、数字+正则定位法
  # 把1-3行的gen换成GEN
  [root@localhost ~]# sed '1,3s/gen/GEN/' 2.txt
   GEN111
   222GEN
   333GEN333
   444xxx444
   555gen555gen

  # 把所有的gen换成GEN
  [root@localhost ~]# sed 's/gen/GEN/g' 2.txt
   GEN111
   222GEN
   333GEN333
   444xxx444
   555GEN555GEN

  # 把所有的gen换成fen执行到文件中
  [root@localhost ~]# sed -i 's/gen/fen/g' 2.txt
  [root@localhost ~]# cat 2.txt
   fen111
   222fen
   333fen333
   444xxx444
   555fen555fen


sed的编辑模式：
  d ：删除
  p ：打印
  a : 在当前行后添加一行
  c ：用新文本修改（替换）当前行
  i : 在当前行之前，插入文本（单独使用时）
  r : 在文件中读内容
  w : 将指定行写入文件
  y : 将字符转换成另一个字符
  s : 将字符串转换成另一个字符串（每一行只替换一次）
  g : 全部执行
  i : 忽略大小写（跟 s 模式一起使用时）
  & ：代表前面匹配到的内容


示例：
	# a模式：在第二行后面添加一行xxx
  [root@localhost ~]# sed '2axxx' 2.txt
   fen111
   222fen
   xxx
   333fen333
   444xxx444
   555fen555fen

  # c模式：将第一行的内容替换为xxx
  [root@localhost ~]# sed '1cxxx' 2.txt
   xxx
   222fen
   333fen333
   444xxx444
   555fen555fen

  # i模式：在第五行之前插入xxx
  [root@localhost ~]# sed '5ixxx' 2.txt
   fen111
   222fen
   333fen333
   444xxx444
   xxx
   555fen555fen



  # r模式：将3.txt内容读到2.txt中第二行后显示
  [root@localhost ~]# sed '2r w.txt' 2.txt
   fen111
   222fen
   hahaha
   333fen333
   444xxx444
   555fen555fen


  # w模式：将第二行的内容写入w.txt 文件，文件不存在自动创建
  [root@localhost ~]# sed '2w w.txt' 2.txt
  [root@localhost ~]# cat w.txt
   222fen

  # y模式：将第二行内容替换
  [root@localhost ~]# sed '2y/fe/FE/' 2.txt
   fen111
   222FEn
   333fen333
   444xxx444
   555fen555fen

```

### 案例

```bash
1、将nginx.conf中的注释行全部去掉
   [root@localhost ~]# sed '/^ *#/d' /etc/nginx/nginx.conf

2、将nginx.conf中每一行之前增加注释
   [root@localhost ~]# sed 's/.*/# &/g' /etc/nginx/nginx.conf

3、要求一键修改本机的ip，
   192.168.15.100 ---> 192.168.15.101
   172.16.1.100   ---> 172.16.1.101
   sed -i 's#.100#.101#g' /etc/sysconfig/network-scripts/ifcfg-eth[01]

4、将/etc/passwd中的root修改成ROOT
   sed -i 's#root#ROOT#g' /etc/passwd
```

# linux 三剑客之 awk

![awk](https://gitee.com/gengff/blogimage/raw/master/images/awk.jpg)

- awk 是一个强大的 Linux 命令，有强大的文本格式化的能力。相对于 grep 的查找，sed 的编辑，awk 在其对数据分析并生成报告时，显得尤为强大。简单来说 awk 就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

  awk 其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。实际上 AWK 的确拥有自己的语言： AWK 程序设计语言 ， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能

```bash
语法格式：
  awk [选项]  '模式{动作}' [文件信息]
	awk [参数]  [处理规则] [操作对象]

  参数
   -F  #指定文本分隔符（默认是以空格作为分隔符）

案例：
 # 准备文本文件
 [root@localhost ~]# cat 3.txt
  test1d test2f test3y
  test4d test5f test6y
  test7d test7f test9y
 # 以字符d为分割符
 [root@localhost ~]# awk -F'd' '{print $NF}' 3.txt
  test2f test3y
  test5f test6y
  test7f test9y


案例：打印系统所有用户的解析器
   [root@localhost ~]# awk -F: '{print $NF}' /etc/passwd


# awk中的内置变量
	$0 : 代表当前行
		[root@localhost ~]# awk -F: '{print $0, "---"}' /etc/passwd

	$n : 代表第n列
		[root@localhost ~]# awk -F: '{print $1}' /etc/passwd

	NF : 记录当前行的字段数
		[root@localhost ~]# awk -F: '{print NF}' /etc/passwd

	$NF : 代表最后一列
		[root@localhost ~]# awk -F: '{print $NF}' /etc/passwd

	NR : 用来记录行号
		[root@localhost ~]# awk -F: '{print NR}' /etc/passwd

	FS : 指定文本内容分隔符（默认是空格）# FS 的优先级要高于 -F
		[root@localhost ~]# awk 'BEGIN{FS=":"}{print $NF, $1}' /etc/passwd

	OFS : 指定打印分隔符（默认空格）
		[root@localhost ~]# awk -F: 'BEGIN{OFS=" >>> "}{print $NF, $1}' /etc/passwd

```

## awk 的生命周期和处理规则的执行流程

```bash
# awk的生命周期和
  grep、sed 和 awk 都是读一行处理一行，直至处理完成。

  ① 接收一行作为输入
  ② 把刚刚读入进来得到文本进行分解
  ③ 使用处理规则处理文本
  ④ 输入一行，赋值给$0，直至处理完成
  ⑤ 把处理完成之后的所有的数据交给END{}来再次处理


# awk处理规则的执行流程
 awk [参数][分隔符] '{BEGIN{开始初需要的处理}/定位/{循环}END{结束前需要的处理}}' [操作对象]
   BEGIN{} #开始语句：定义变量在BEGIN块里面
    //     #定位：正则匹配
    {}     #循环语句：循环处理文本
   END{}   #结束语句：打印之前统一处理
```

![21-1](https://gitee.com/gengff/blogimage/raw/master/images/21-1.png)

## awk 中的函数

```bash
# 只能用于循环语句和结束语句
  print : 打印
  printf : 格式化打印
   参数：
      %s  #字符串
      %d  #数字
       -  #左对齐
       +  #右对齐
      15  #至少占用15字符
 [root@localhost ~]# awk -F: 'BEGIN{OFS=" | "}{printf "|%+15s|%-15s|\n", $NF,$1}' /etc/passwd
  |      /bin/bash|root           |
  |  /sbin/nologin|bin            |
  |  /sbin/nologin|daemon         |
  |  /sbin/nologin|adm            |
  |  /sbin/nologin|lp             |
  |      /bin/sync|sync           |
  | /sbin/shutdown|shutdown       |
  |     /sbin/halt|halt           |
  |  /sbin/nologin|mail           |
  |  /sbin/nologin|operator       |
  |  /sbin/nologin|games          |
......

```

## awk 中的定位和流程控制

```bash
# awk中的定位

 # 1、正则表达式
   # 打印含root所在行的所有内容
   [root@localhost ~]# awk -F: '/root/{print $0}' /etc/passwd

   # 打印以root开头所在行的所有内容
   [root@localhost ~]# awk -F: '/^root/{print $0}' /etc/passwd


 # 2、比较表达式
    >   #大于
    <   #小于
    >=  #大于等于
    <=  #小于等于
    ~	  #表示匹配后面的正则表达式
    !~  #表示匹配后面的正则表达式

  案例：要求打印属组ID大于属主ID的行
   [root@localhost ~]# awk -F: '$4 > $3{print $0}' /etc/passwd

  案例：结尾包含bash
   [root@localhost ~]# awk -F: '$NF ~ /bash/{print $0}' /etc/passwd

  案例：结尾不包含bash
   [root@localhost ~]# awk -F: '$NF !~ /bash/{print $0}' /etc/passwd

  # 3、逻辑表达式
   &&	#与
    [root@localhost ~]# awk -F: '$3 + $4 > 2000 && $3 * $4 > 2000{print $0}' /etc/passwd

   || #或

    [root@localhost ~]# awk -F: '$3 + $4 > 2000 || $3 * $4 > 2000{print $0}' /etc/passwd
   !  #非
    [root@localhost ~]# awk -F: '!($3 + $4 > 2000){print $0}' /etc/passwd


  # 4、算术表达式
    +  #加
    -  #减
    *  #乘
    /  #除
    %  #取余

   案例：要求属组 + 属主的ID 大于 2000
    [root@localhost ~]# awk -F: '$3 + $4 > 2000{print $0}' /etc/passwd

   案例：要求属组 * 属主的ID 大于 2000
    [root@localhost ~]# awk -F: '$3 * $4 > 2000{print $0}' /etc/passwd

   案例：要求打印偶数行
    [root@localhost ~]# awk -F: 'NR % 2 == 0{print $0}' /etc/passwd

   案例：要求打印奇数行
    [root@localhost ~]# awk -F: 'NR % 2 == 1{print $0}' /etc/passwd

  # 5、条件表达式
    ==  #等于
    >   #大于
    <   #小于
    >=  #大于等于
    <=  #小于等于

   案例：要求打印第三行
    [root@localhost ~]# awk -F: 'NR == 3{print $0}' /etc/passwd

  # 6、范围表达式：一个选定条件到另一个选定条件之间的数据
		案例：
		[root@localhost ~]# awk -F: '/^root/,/^ftp/{print $0}' /etc/passwd




# 流程控制
  只存在循环之中。
  if
    [root@localhost ~]# awk -F: '{if($3>$4){print "大于"}else{print "小于或等于"}}' /etc/passwd

      if(){}
      if(){}else{}
      if(){}else if(){}else{}
  for

    [root@localhost ~]# awk -F: '{for(i=10;i>0;i--){print $0}}' /etc/passwd

    for(i="初始值";条件判断;游标){}

  while

    [root@localhost ~]# awk -F: '{i=1; while(i<10){print $0, i++}}' /etc/passwd

    while(条件判断){}


    每隔5行，打印一行横线
    -------------------------------------------------------------------------

    [root@localhost ~]# awk -F: '{if(NR%5==0){print "----------------"}print $0}' /etc/passwd


```
