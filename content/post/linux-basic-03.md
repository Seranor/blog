---
title: vim的使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
  - Vim
categories:
  - Linux
url: post/linux-03.html
toc: true
---

# 系统快捷键

<!-- more -->

```bash
1、历史命令信息：上下键
2、清屏命令：ctrl + l | clear
3、中断取消命令执行过程 ： ctrl + c
4、快速移动光标到行首尾：ctrl + a/e
5、将光标到行首信息剪切：ctrl + u
6、将剪切的内容进行粘贴：ctrl + y
7、将光标到行尾信息剪切：ctrl + k
8、锁定系统窗口信息状态：ctrl + s
9、解锁系统窗口信息状态：ctrl + q
10、搜索最近执行过的指令：ctrl + r
11、命令行中快速移动光标：ctrl + 方向键
12、退出当前的登录，相当于logout：ctrl+d
13、删除当前光标向前一组字符串，以空格为分隔符：ctrl+w
14、杀死当前进程：ctrl+z
15、系统命令信息补全功能：tab
```

# 文件管理基础命令

## cp

复制文件：主要可以起到数据备份的作用

```bash
copy的缩写cp。主要作用就是复制、拷贝，没有-f选项，强制覆盖只能转义

格式：
	cp [参数] [被复制文件的路径] [复制到的新路径]

参数：
	-r : 递归复制，复制目录时所使用的
	-p : 保持属性（时间戳、大小等）
  -d : 复制的时候保证软连接
  -a : 保证某些属性不变。相当于-rpd，上面三种
  -t : 把源文件的位置与目标目录的位置进行交换，在批量拷贝文件时使用
  -i : 默认执行，当拷贝的文件在目标目录已经存在时，提示是否覆盖

案例：
   案例1：将/root目录下anaconda-ks.cfg复制到/tmp目录
    [root@localhost ~]# cp /root/anaconda-ks.cfg /tmp

   案例2：将/root目录下的test文件夹及其内部的文件复制到/tmp中
    [root@localhost ~]# cp -r /root/test /tmp

     补充：在linux中，文件夹是不可以直接复制。

   案例3：将/etc/hosts和/etc/resolv.conf 复制到/tmp目录中
    [root@localhost ~]# cp /etc/hosts /etc/resolv.conf  /tmp

注意：在linux没有提示就是做好的结果

补充：Esc + . : 上一条命令的最后一个元素
	 ls -l 等价于 ll

知识储备：
   linux中的链接相当于快捷方式。
   stat : 查看文件详细属性。
```

## mv

移动文件：移动文件相当于剪切

```bash
负责移动或者重命名，移动目录的时候最好是加/避免改名操作

格式：
	mv [移动文件的原路径] [移动文件的新路径]

案例：
	案例1：将/root目录下的1.txt移动到/opt目录中
    [root@localhost ~]# mv /root/1.txt /opt

		# 移动文件夹
    [root@localhost ~]# mv test/ /mnt/
```

## rm

删除文件：rm 是一个物理删除的命令，系统中的危险命令

删除文件有两种方式：

- 1、物理删除：直接删除文件。
- 2、逻辑删除：将文件隐藏，没有直接删除。

```bash
格式：
	rm [参数] [需要删除文件的路径]
参数：
	-f : 不提示强制删除
	-r : 递归删除目录及其内容
	-i : 每次删除前提示是否确认删除

案例：
	案例1：将/root目录下的1.txt删除
		[root@localhost ~]# rm 1.txt
		[root@localhost ~]# rm -f 1.txt

	案例2：删除/root目录下的test文件夹及其内部所有的文件
		[root@localhost ~]# rm -r /root/test/
		[root@localhost ~]# rm -rf /root/test/

补充：
   在linux系统中，不能够直接删除文件夹。
   linux系统中禁止使用：
      rm -rf /* # 表示删除目录下的所有文件

解决rm命令误操作
  将rm命令改一个名称。

知识储备：
  查看命令存放路径：which
```

## alias

系统别名

```bash
格式：
	alias xxx='命令'

	alias  ： 查看系统别名
	alias rm='xxx' ： 设置系统别名

不使用别名，就在命令之前增加\
	[root@localhost ~]# \rm 1.txt
```

## vi/vim 编辑器

**什么是 vim**

vi 和 vim 是 Linux 常用文本编辑工具，具有很强大的编辑功能，vim 是 vi 的升级版编辑器

**为什么要使用 VIM**
因为 Linux 系统一切皆为文件，而我们工作最多的就是修改某个服务的配置(其实就是修改文件内容)。
也就是说如果没有 vi/vim，我们很多工作都无法完成。PS: vim 是学习 linux 最重要的命令之一

**VI 与 VIM 有什么区别**
vi 和 vim 都是文本编辑器，只不过 vim 是 vi 的增强版，比 vi 多了语法高亮显示，其他编辑功能几乎无差，所以使用 vi 还是 vim 取决个人习惯。(相当于 windows 系统下的文本编辑软件“记事本”与"notepad++"的区别)

PS：因为前期最小化安装 CentOS 系统，所以默认情况下没有 vim 命令，但可以使用 yum install vim -y

**如何使用 VIM 编辑器**

- vim 编辑器中有三种模式
  - 命令模式：主要是使用各种快捷键，进入修改文件的第一个模式
  - 末行模式：主要用于保存或退出文本。
  - 编辑模式：主要进行文本内容编辑和修改

![vim三种模式](https://gitee.com/gengff/blogimage/raw/master/images/vim%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F.png)

小结: vim 编辑打开文件整体流程如下: 1.默认打开文件处于普通模式 2.从普通模式切换至编辑模式需要使用 a、i、o 3.编辑模式修改完毕后需要先使用 ECS 返回普通模式 4.在普通模式输入":"或"/"进入命令模式，可实现文件的保存与退出。
PS: 在 vim 中，无法直接从编辑模式切换到命令模式。

```bash
1、安装vim
	yum install vim -y

2、打开编辑文件
	[root@localhost ~]# vim 1.txt


3、普通模式：命令光标快速移动快捷方式

#1.命令光标跳转
G     #快速切换光标到底行
gg    #快速切换光标到首行
ngg   #光标跳转至当前文件内的N行
$     #快速跳转到行尾
^|0   #快速跳转到行首


#2.快速跳转到指定行
		1、进入末行模式
		2、输入跳转的行数
		3、回车


#3.快速复制文本内容信息
yy    #复制当前光标所在的行
nyy   #复制当前光标及光标向下的n行


#4.快速粘贴文本内容
p(小)	#在当前光标的下一行粘贴
P(大)  #在当前光标的上一行粘贴


#5. 删除文本内容
dd    #删除当前光标所在行
ndd   #删除当前光标所在行以及向下的n行


#6.回撤
u         #撤销上一次的操作
ctrl + r	#退回上一次回撤


4、进入编辑模式(从普通模式进入到编辑模式)
	i	   #在光标之前输入
	o	   #在光标下新创建一行空白内容
	a	   #在光标之后输入


5.文件保存与退出
#1、进入末行模式:
#2、操作
:w      #保存当前状态
:w!     #强制保存当前状态
:q      #退出当前文档(文档必须保存才能退出)
:q!     #强制退出文档不会修改当前内容
:wq     #先保存，在退出
:wq!    #强制保存并退出
:x      #先保存，在退出
ZZ      #保存退出, shfit+zz
:number #跳转至对应的行号


6.显示行号
#1、进入末行模式:
#2、输入:set nu
#3、回车


7.取消行号
#1、进入末行模式
#2、输入:set nonu
#3、回车


8.文件内容查找
#1、进入命令模式
#2、输入/
#3、输入搜索的内容
#4、回车

n    #下一个，按搜索到的内容依次往下进行查找
N    #上一个，按搜索到的内容依次往上进行查找

:set ic   #忽略大小写，在搜索的时候有用
:set ai   #自动缩进
:set list #显示制表符(空行、tab键)

9.可视化编辑
#1、ctrl + v
#2、编辑：Shift + i
#3、按 Esc键退出即可


10、解决vim编辑异常
	1、删除.1.txt.swp
	2、继续编辑（-r）
		[root@localhost ~]# vim -r 1.txt
	3、放弃编辑（-n）
		[root@localhost ~]# vim -n 1.txt

知识储备
  实时监控文件内容变化：
		tail -f [要监控的文件]

  演示vim编辑异常
		1、查看vim进程
      [root@localhost ~]# ps -ef | grep vim
		2、杀死vim进程
      [root@localhost ~]# kill -9 pid

  批量复制
		[root@localhost ~]# while true;do echo "Hello World" >> 1.txt; done

```

![vim生命周期](https://gitee.com/gengff/blogimage/raw/master/images/vim%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

## cp

复制文件：主要可以起到数据备份的作用

```bash
copy的缩写cp。主要作用就是复制、拷贝，没有-f选项，强制覆盖只能转义

格式：
	cp [参数] [被复制文件的路径] [复制到的新路径]

参数：
	-r : 递归复制，复制目录时所使用的
	-p : 保持属性（时间戳、大小等）
  -d : 复制的时候保证软连接
  -a : 保证某些属性不变。相当于-rpd，上面三种
  -t : 把源文件的位置与目标目录的位置进行交换，在批量拷贝文件时使用
  -i : 默认执行，当拷贝的文件在目标目录已经存在时，提示是否覆盖

案例：
   案例1：将/root目录下anaconda-ks.cfg复制到/tmp目录
    [root@localhost ~]# cp /root/anaconda-ks.cfg /tmp

   案例2：将/root目录下的test文件夹及其内部的文件复制到/tmp中
    [root@localhost ~]# cp -r /root/test /tmp

     补充：在linux中，文件夹是不可以直接复制。

   案例3：将/etc/hosts和/etc/resolv.conf 复制到/tmp目录中
    [root@localhost ~]# cp /etc/hosts /etc/resolv.conf  /tmp

注意：在linux没有提示就是做好的结果

补充：Esc + . : 上一条命令的最后一个元素
	 ls -l 等价于 ll

知识储备：
   linux中的链接相当于快捷方式。
   stat : 查看文件详细属性。
```

## mv

移动文件：移动文件相当于剪切

```bash
负责移动或者重命名，移动目录的时候最好是加/避免改名操作

格式：
	mv [移动文件的原路径] [移动文件的新路径]

案例：
	案例1：将/root目录下的1.txt移动到/opt目录中
    [root@localhost ~]# mv /root/1.txt /opt

		# 移动文件夹
    [root@localhost ~]# mv test/ /mnt/
```

## rm

删除文件：rm 是一个物理删除的命令，系统中的危险命令

删除文件有两种方式：

- 1、物理删除：直接删除文件。
- 2、逻辑删除：将文件隐藏，没有直接删除。

```bash
格式：
	rm [参数] [需要删除文件的路径]
参数：
	-f : 不提示强制删除
	-r : 递归删除目录及其内容
	-i : 每次删除前提示是否确认删除

案例：
	案例1：将/root目录下的1.txt删除
		[root@localhost ~]# rm 1.txt
		[root@localhost ~]# rm -f 1.txt

	案例2：删除/root目录下的test文件夹及其内部所有的文件
		[root@localhost ~]# rm -r /root/test/
		[root@localhost ~]# rm -rf /root/test/

补充：
   在linux系统中，不能够直接删除文件夹。
   linux系统中禁止使用：
      rm -rf /* # 表示删除目录下的所有文件

解决rm命令误操作
  将rm命令改一个名称。

知识储备：
  查看命令存放路径：which
```

## vi/vim 编辑器

**什么是 vim**

vi 和 vim 是 Linux 常用文本编辑工具，具有很强大的编辑功能，vim 是 vi 的升级版编辑器

**为什么要使用 VIM**
因为 Linux 系统一切皆为文件，而我们工作最多的就是修改某个服务的配置(其实就是修改文件内容)。
也就是说如果没有 vi/vim，我们很多工作都无法完成。PS: vim 是学习 linux 最重要的命令之一

**VI 与 VIM 有什么区别**
vi 和 vim 都是文本编辑器，只不过 vim 是 vi 的增强版，比 vi 多了语法高亮显示，其他编辑功能几乎无差，所以使用 vi 还是 vim 取决个人习惯。(相当于 windows 系统下的文本编辑软件“记事本”与"notepad++"的区别)

PS：因为前期最小化安装 CentOS 系统，所以默认情况下没有 vim 命令，但可以使用 yum install vim -y

**如何使用 VIM 编辑器**

- vim 编辑器中有三种模式
  - 命令模式：主要是使用各种快捷键，进入修改文件的第一个模式
  - 末行模式：主要用于保存或退出文本。
  - 编辑模式：主要进行文本内容编辑和修改

![vim三种模式](https://gitee.com/gengff/blogimage/raw/master/images/vim%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F.png)

小结: vim 编辑打开文件整体流程如下: 1.默认打开文件处于普通模式 2.从普通模式切换至编辑模式需要使用 a、i、o 3.编辑模式修改完毕后需要先使用 ECS 返回普通模式 4.在普通模式输入":"或"/"进入命令模式，可实现文件的保存与退出。
PS: 在 vim 中，无法直接从编辑模式切换到命令模式。

```bash
1、安装vim
	yum install vim -y

2、打开编辑文件
	[root@localhost ~]# vim 1.txt


3、普通模式：命令光标快速移动快捷方式

#1.命令光标跳转
G     #快速切换光标到底行
gg    #快速切换光标到首行
ngg   #光标跳转至当前文件内的N行
$     #快速跳转到行尾
^|0   #快速跳转到行首


#2.快速跳转到指定行
		1、进入末行模式
		2、输入跳转的行数
		3、回车


#3.快速复制文本内容信息
yy    #复制当前光标所在的行
nyy   #复制当前光标及光标向下的n行


#4.快速粘贴文本内容
p(小)	#在当前光标的下一行粘贴
P(大)  #在当前光标的上一行粘贴


#5. 删除文本内容
dd    #删除当前光标所在行
ndd   #删除当前光标所在行以及向下的n行


#6.回撤
u         #撤销上一次的操作
ctrl + r	#退回上一次回撤


4、进入编辑模式(从普通模式进入到编辑模式)
	i	   #在光标之前输入
	o	   #在光标下新创建一行空白内容
	a	   #在光标之后输入


5.文件保存与退出
#1、进入末行模式:
#2、操作
:w      #保存当前状态
:w!     #强制保存当前状态
:q      #退出当前文档(文档必须保存才能退出)
:q!     #强制退出文档不会修改当前内容
:wq     #先保存，在退出
:wq!    #强制保存并退出
:x      #先保存，在退出
ZZ      #保存退出, shfit+zz
:number #跳转至对应的行号


6.显示行号
#1、进入末行模式:
#2、输入:set nu
#3、回车


7.取消行号
#1、进入末行模式
#2、输入:set nonu
#3、回车


8.文件内容查找
#1、进入命令模式
#2、输入/
#3、输入搜索的内容
#4、回车

n    #下一个，按搜索到的内容依次往下进行查找
N    #上一个，按搜索到的内容依次往上进行查找

:set ic   #忽略大小写，在搜索的时候有用
:set ai   #自动缩进
:set list #显示制表符(空行、tab键)

9.可视化编辑
#1、ctrl + v
#2、编辑：Shift + i
#3、按 Esc键退出即可


10、解决vim编辑异常
	1、删除.1.txt.swp
	2、继续编辑（-r）
		[root@localhost ~]# vim -r 1.txt
	3、放弃编辑（-n）
		[root@localhost ~]# vim -n 1.txt

知识储备
  实时监控文件内容变化：
		tail -f [要监控的文件]

  演示vim编辑异常
		1、查看vim进程
      [root@localhost ~]# ps -ef | grep vim
		2、杀死vim进程
      [root@localhost ~]# kill -9 pid

  批量复制
		[root@localhost ~]# while true;do echo "Hello World" >> 1.txt; done

```

![vim生命周期](https://gitee.com/gengff/blogimage/raw/master/images/vim%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)
