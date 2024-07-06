---
title: Linux用户管理
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-06.html
toc: true
---

# 用户和用户组

![1583571865_ab05a7218d58cc85bbfc02fbec2d730f](https://gitee.com/gengff/blogimage/raw/master/images/1583571865_ab05a7218d58cc85bbfc02fbec2d730f.png)

<!-- more -->

- 用户

  - 用户对硬件资源的操作都需要通过操作系统，比如用户要读取硬盘中的一份关键数据。出于安全考虑，操作系统的开发者们都专门开发了安全机制，要使用操作系统必须事先输入正确的用户名与密码，根据用户给相应权限，这便是用户的由来
  - 用户是权限的化身，通常在公司是使用普通用户管理服务器，因为 root 权限过大，容易出问题
  - 每启动一个进程都会与一个用户关联
    - 进程===》用户===》权限（作用在文件身上）

- 用户组

  - 主组：用户本身所在的组
  - 附属组：为用户添加的组

  用户与组的关系

  - 一对一：一个用户可以属于一个组，用户默认就在自己的主组下
  - 一对多：一个用户可以属于多个组，用户只有一个主组，但可以为用户添加多个附加组
  - 多对多：多个用户可以属于多个组

```bash
# linux系统中用户角色划分
  系统用户：uid在 0-999 之间的是系统用户
		系统用户一般用在启动应用程序上，一般不需要登录系统。

  普通用户：uid在 1000及以上的是普通用户
		一般用在登录上。
```

## 用户与组相关文件

```bash
# 用户详情的文件
# 文件信息：/etc/passwd
[root@localhost ~]# cat /etc/passwd
 tony:x:1004:1004::/home/tony:/bin/bash
  tony    #用户名
  x       #密码占位符
  1001    #用户uid
  1001		#用户组gid
  /home/tony #家目录
  /bin/bash	 #默认的解析器


# 用户密码的文件
# 文件信息：/etc/shadow
[root@localhost /]# cat /etc/shadow
 tony:!!:18978:0:99999:7:::  #用户密码
  tony  #用户名
  !!    #密码是一长串的字符串，!!表示没有密码
  18978 #最近一次变更密码，距离1970年到现在过了多少天
  0     #密码最少使用天数，0表示无限制
  99999 #密码最长使用天数，99999表示无限制
  7     #密码过期预警天数


# 用户组配置文件
# 文件信息：/etc/group
[root@localhost /]# cat /etc/group
 tony:x:1004:
  tony  # 用户组名
  x     # 用户组密码占位符
  1004  # 用户组id


# 用户组密码文件
# 文件信息：/etc/gshadow
[root@localhost /]# cat /etc/gshadow
 tony:!::
  tony  #用户组名
  !     #用户组密码，!或空表示没有密码
  :     #用户组管理者，可以为空，多个管理用，分割
  :     #显示用户组为哪个用户的附加组，多个用，分割
```

## 用户管理命令

```bash
用户：相当于账号
  root  #root用户拥有最高权限
  test

# 创建用户的命令：
	useradd [用户名]

  参数：
	 -g  #指定用户组（用户必须存在）
	 -r  #创建系统用户
	 -M  #不创建家目录
	 -u  #指定创建用户的ID的
	 -s  #指定解析器

  [root@localhost /]# useradd tony  #默认将用户写进了/etc/passwd 这个文件
	ps：每创建一个用户都会创建一个该用户的主组，组名与用户名相同


# 切换用户的命令：
	su - [用户名]
	su [用户名]


# 查看用户相关信息
  [root@localhost /]# id  #查看当前用户
   uid=0(root) gid=0(root) 组=0(root)

  [root@localhost /]# id test  #id [用户名] 查看指定用户信息
   uid=1001(test) gid=1001(test) 组=1001(test)

  [root@localhost /]# whoami  # 查看当前用户是谁
   root

  [root@localhost /]# who  #查看所有登录的用户
   root   pts/0    2021-12-17 19:37 (192.168.15.1)
   root   pts/1    2021-12-17 19:37 (192.168.15.1)



# 修改用户的命令：
  usermod [用户名]

  参数：
   -u  #修改用户的UID
	 –g  #指定⽤户所属的GID

 # 修改用户的UID
	[root@localhost ~]# tail -1 /etc/passwd
   user:x:1005:1005::/home/user:/bin/bash
  [root@localhost /]# usermod -u 1006 user  #显示无法修改
   usermod: user user is currently used by process 9001
  [root@localhost /]# kill -9 9001  #需要杀死正在使用用户的进程
  [root@localhost /]# 已杀死
  #这时候可以修改了
  [root@localhost ~]# usermod -u 1006 user
  [root@localhost ~]# tail -1 /etc/passwd  #修改成功
   user:x:1006:1005::/home/user:/bin/bash


 # 用户组必须是已经存在的，否则无法修改
  [root@localhost /]# usermod user -g 1001
  [root@localhost /]# tail -1 /etc/passwd
    user:x:1006:1001::/home/user:/bin/bash


# 删除用户的命令：
 userdel [用户名]
   参数：-r
  #删除⽤户test1，但不删除⽤户家⽬录和mail
  [root@localhost /]# userdel test1

  [root@localhost /]# ll /home   #查看并没有删掉test1的家目录
   drwx------ 3 1003 1003 92 12月 17 21:02 test1
  [root@localhost /]# ll /var/spool/mail/  #查看并没有删掉test1的mail
   -rw-rw---- 1 1003 mail 0 12月 17 21:02 test1
  [root@localhost /]# rm -rf /home/test1  #手动删除家目录
  [root@localhost /]# rm -rf /var/spool/mail/test1 #手动删除mail


  #要想删彻底，加-r选项，其实是做了上面所有的操作
  [root@localhost /]# userdel -r test1

```

### 手动创建用户

- 不使用 useradd 创建用户

```bash
# 创建用户家目录
[root@localhost opt]# mkdir -p /home/test

# 查看用户信息配置文件
[root@localhost opt]# cat /etc/passwd

# 创建test用户并写入用户信息配置文件
[root@localhost ~]# echo 'test:x:1001:1001::/home/test/:/bin/bash' >> /etc/passwd

# 再次查看用户信息配置文件
[root@localhost ~]# cat /etc/passwd
test:x:1001:1001::/home/test/:/bin/bash  #可以发现新增的用户

# 给用户添加属组
[root@localhost ~]# echo 'test:x:1001' >> /etc/group

# 给用户添加密码
[root@localhost /]# vim /etc/shadow

# 给用户家目录配置环境变量
[root@localhost ~]# cp /etc/skel/.bashrc  /home/test
[root@localhost ~]# cp /etc/skel/.bash_profile   /home/test
或
[root@localhost ~]# cp /etc/skel/.bash*  /home/test

# 修改用户家目录权限
[root@localhost home]# chown test.test test/  # 用户.用户组 用户家路径

# 切换用户
[root@localhost ~]# su - test

上一次登录：二 12月 14 19:05:50 CST 2021pts/0 上

# 成功切换用户
[test@localhost ~]$
```

### 密码

- 修改或添加 Linux 普通用户的密码。直接影响的文件是/etc/shado

```bash
 参数
  -d	#删除密码
  -l	#锁定用户密码，无法被用户自行修改
  -u	#解开已锁定用户密码，允许用户自行修改
  -e	#密码立即过期，下次登陆强制修改密码
  -k	#保留即将过期的用户在期满后能仍能使用
  -S	#查询密码状态

# 增加或修改密码
当用户密码不存在的时候即为增加密码，当用户密码存在时即为修改密码。

 [root@localhost home]# useradd user
 [root@localhost home]# tail -1 /etc/passwd
   user:x:1005:1005::/home/user:/bin/bash
 [root@localhost home]# tail -1 /etc/shadow
   user:!!:18701:0:99999:7:::

 [root@localhost home]# passwd user
 [root@localhost ~]# passwd user
   更改用户 user 的密码 。
   新的 密码：
   重新输入新的 密码：
   passwd：所有的身份验证令牌已经成功更新。
 [root@localhost home]# tail -1 /etc/passwd
   user:x:1005:1005::/home/user:/bin/bash
 [root@localhost home]# tail -1 /etc/shadow
 user:$6$RApJMwf1$jytlisorvavpdDmuZ4RGyuFLZaHd5C0uMqXJU0dFt/Vn7Oj8tSiN7/RswvXc3LIBh6JuDOq73u2K1Uf4up476/:18979:0:99999:7:::



# 免交互修改密码
[root@localhost /]# echo '123' | passwd --stdin user
Changing password for user user.
passwd: all authentication tokens updated successfully.
```

![yhz](https://gitee.com/gengff/blogimage/raw/master/images/yhz.png)

## 用户组管理命令

每个用户都有一个用户组，这样系统可以对一个用户组中的所有用户进行集中管理。不同 Linux 系统对用户组的规定有所不同，如 Linux 下的用户属于与它同名的用户组，这个用户组在创建用户时同时创建。

```bash
用户组：某些具有相同属性的账号的集合
	root
# 创建用户组的命令：
	groupadd

	参数：
		-g  #指定用户组的GID
	  -r  #创建系统工作组，系统工作组的组ID小于1000
    -K  #覆盖配置文件“/ect/login.defs”
    -o  #允许添加组ID号不唯一的工作组

  #创建组
   [root@localhost /]# groupadd group1
   [root@localhost /]# cat /etc/group   #查看组信息
    group1:x:1005:

  #指定GID创建组
   [root@localhost /]# groupadd -g 2001 group2
   [root@localhost /]# tail -1 /etc/group  #查看文件最后一行的内容
    group2:x:2001:

  #创建系统组
   [root@localhost /]# groupadd -r group4
   [root@localhost /]# tail -1 /etc/group
    group4:x:996:

# 修改组的命令：
  groupmod

  参数：
   -g  #设置欲使用的组GID
   -n  #设置欲使用的群组名称

  #修改组GID
   [root@localhost /]# groupmod -g 2222 group1  #将group1的组GID改为2222
   [root@localhost /]# tail -1 /etc/group
    group1:x:2222:

  #修改组名
   [root@localhost /]# groupmod -n new_group group1 #将组group1改名为new_group
   [root@localhost /]# tail -1 /etc/group
    new_group:x:2222:


# 删除组的命令：
 groupdel

  [root@localhost /]# groupdel group2
  [root@localhost /]# tail -5 /etc/group #已经删除了
   mysql:x:27:
   tony:x:1004:
   group3:x:2005:
   group4:x:996:
   new_group:x:2222:

ps：用户组在系统中删除，如果一个组被用户占用则不能删除。用户被删除，用户基本组也会被删除

知识储备：
  tail  #tail 命令可用于查看文件的内容
	-f  #循环读取
  -q  #不显示处理信息
  -v  #显示详细的处理信息
  -c<数目>  #显示的字节数
  -n<行数>  #显示文件的尾部 n 行内容
```

![WechatIMG1411](https://gitee.com/gengff/blogimage/raw/master/images/WechatIMG1411-20211218004252145.png)

## SUDO 提权

> 用于普通用提升权限的。

- 相关的文件：`/etc/sudoers`

- 检查`/etc/sudoers`是否修改正确：visudo -c

- sudoers 文件格式

  ```
  tom       ALL=           (ALL)          ALL
  用户名称   所有机器可登陆    所有IP或主机名   所有的指令
  ```

- 指令编写格式

  ```
  # 必须写全路径：which查看命令全路径

  # 只支持vim命令提权
  xianchen ALL=(ALL)  /usr/bin/vim

  # 支持所有的命令提权
  tom ALL=(ALL)  ALL

  # 不支持某个命令提权
  tom ALL=(ALL) ALL, !/usr/bin/vim

  # 不支持某个命令的部分功能
  xiaochen ALL=(ALL)   ALL, !/usr/bin/vim /root/123.txt
  ```

[![img](https://gitee.com/gengff/blogimage/raw/master/images/2251663-20210316184607539-1301650386.png)](https://img2020.cnblogs.com/blog/2251663/202103/2251663-20210316184607539-1301650386.png)

[![img](https://img2020.cnblogs.com/blog/2251663/202103/2251663-20210316184619104-256801431.png)](https://img2020.cnblogs.com/blog/2251663/202103/2251663-20210316184619104-256801431.png)

**\*4\***|**\*0\*\*\***su\*\*

- su - xxx 和 su xxx 之间区别

  ```
  1、su - xxx ：相当于切换一个窗口，su xxx 仅仅切换了用户

  2、su - xxx ： 切换用户执行的系统文件要多于 su xxx

  3、su - xxx 是登录
     su  xxx  切换用户
  ```

- Linux 中的 shell 可以分为两类

```
登陆shell，需要输⼊⽤户名和密码才能进⼊Shell，⽇常接触的最多的⼀种
⾮登陆shell，不需要输⼊⽤户和密码就能进⼊Shell,⽐如运⾏bash会开启⼀个新的会话窗⼝
```

**\*4\***|**\*1\*\*\***bash shell 配置文件介绍\*\*

```
全局配置⽂件：
 /etc/profile
 /etc/profile.d/*.sh
 /etc/bashrc
个⼈配置⽂件：
 ~/.bash_profile
 ~/.bashrc
profile类⽂件, 设定环境变量, 登陆前运⾏的脚本和命令。
bashrc类⽂件, 设定本地变量, 定义命令别名
PS: 如果全局配置和个⼈配置产⽣冲突，以个⼈配置为准。
```

**\*4\***|**\*2\*\*\***配置文件的执行顺序\*\*

```
如果执⾏的是登录式shell，那么配置⽂件执⾏顺序是:
/etc/profile -> /etc/profile.d/*.sh -> ~/.bash_profile -> ~/.bashrc -> /etc/bashrc
如果执⾏的是⾮登录式shell，那么配置⽂件执⾏顺序是:
~/.bashrc -> /etc/bashrc -> /etc/profile.d/*.sh
PS: 验证使⽤echo在每⾏添加⼀个输出即可，注意，要把输出放在⽂件的第⼀⾏。如果说要写登录执行脚本，可以配置在/.bashrc当中。
```
