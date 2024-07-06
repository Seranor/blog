---
title: Linux文件权限管理
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-07.html
toc: true
---

# 文件权限

**把⼀个⽤户加⼊了⼀个组，该⽤户就拥有了该组的权限，当⼀个⽤户要操作某个⽂件时，系统会依次检索该⽤户是否是该⽂件的(属主)拥有者，其次是(数组)组成员，最后是其他⼈，如果扫描到是拥有者，则具备拥有者的权限，不必往后扫描，以此类推**

<!-- more -->

## 权限类型

每一个权限拥有一个数字编号

- **r：可读(read)---> 4**

- **w：可写(write)---> 2**

- **x：可执⾏(execute)---> 1**

- **-：没有对应权限**

  **执行脚本 == 运行脚本**

#### 权限的归属

- **属主：u**
- **属组：g**
- **其他人：o**

## 权限位

![image-20211218190929438](https://gitee.com/gengff/blogimage/raw/master/images/image-20211218190929438.png)

权限位主要分为三个部分，分别是属主、属组以及其他人

#### rwx ： 属主--->可读可写可执行

#### r-x ： 属组--->可读可执行

#### rw- ： 其他人--->可读可写

```bash
使用ll命令查看文件详情，在每个文件详情最前方，有10个字符来表示文件类型和权限
  [root@localhost ~]# ll
  -rw-r--r-- 1 root root    0 12月 14 19:11 1
  drwxr-xr-x 2 root root    6 12月 18 15:51 a
  -rwxr-xr-x 1 root root 4609 11月 21 11:43 init.sh

在Linux 系统中权限是区分用户的，即属主(用户)、属组(组用户)、其他用户，第一位表示文件的类型，-代表文件，d代表目录，其他每个用户占三个字符，这里-rwxr-xr-x对应如下关系
-    #文件
rwx  #属主：可读可写可执行
r-x  #属组：可读可执行
r-x  #其他：可读可执行

含义解释
第一位：-代表文件，d代表目录
用户、组用户、其他用户都未rwx形式，其中r表示读、w表示写、x表示可执行，-表示没有权限，拿用户组举例，r只能出现在第一个位置、w只能出现在第二个位置、x只能出现在第三位。
```

![1327924-20191106101057159-461417569](https://gitee.com/gengff/blogimage/raw/master/images/1327924-20191106101057159-461417569.png)

如果我们将出现字符（可以是 r、w、x）表示为 1，出现-表示为 0，那么对应二进制如下，r - - = 100、- w - = 010、- - x = 001、再转换成 10 进制，那么读=4、写=2、可执行=1，将转换为以下关系

![1327924-20191106101454753-1175152316](https://gitee.com/gengff/blogimage/raw/master/images/1327924-20191106101454753-1175152316.png)

也就是说这里的数字简写了用户权限，我们也可以用数字反推权限，比如数据 6，我们转换为为二进制：110，转换为：rw-，具有可读、可写权限。

现在我们已经明白了：-rwxr-xr-x 权限含义了，这里用数字简写就是-755，这里还需要改写成 0755，这里的 0 可以简单理解成 10 进制

## 设置权限

```bash
chmod
 	格式：
 		chmod [参数] [权限表达式] [操作对象]
 	参数：
 		-R  : 递归增加权限


 [root@localhost ~]# ll /root/1.txt
  -rw-r--r-- 1 root root 0 12月 18 16:42 /root/1.txt

# 1、加减法：+、- 在原来的权限基础上增减
 #给属主增加执行权限，属组增加执行权限，其他增加写权限
 [root@localhost ~]# chmod u+x,g+x,o+w /root/1.txt
 [root@localhost ~]# ll /root/1.txt
   -rwxr-xrw- 1 root root 0 12月 18 16:42 /root/1.txt

 #取消属主写权限，权限属组读权限，取消其他读权限
 [root@localhost ~]# chmod u-w,g-r,o-r /root/1.txt
 [root@localhost ~]# ll /root/1.txt
  -r-x--x-w- 1 root root 0 12月 18 16:42 /root/1.txt


# 2、赋值：= 不管原来是什么权限就直接覆盖了
 [root@localhost ~]# chmod u=rwx,g=rw,o=rx /root/1.txt
 [root@localhost ~]# ll /root/1.txt
  -rwxrw-r-x 1 root root 0 12月 18 16:42 /root/1.txt


# 3、数字：不能单独设置一个，必须全部设置
 #给属主设置读写执行权限，属组设置读执行权限，其他设置读写权限
 [root@localhost ~]# chmod 756 /root/1.txt
 [root@localhost ~]# ll /root/1.txt
  -rwxr-xrw- 1 root root 0 12月 18 16:42 /root/1.txt

 #全部设置读写执行权限
 [root@localhost ~]# chmod 777 /root/1.txt
 [root@localhost ~]# ll /root/1.txt
  -rwxrwxrwx 1 root root 0 12月 18 16:42 /root/1.txt


# 4、-R:递归设置权限
 [root@localhost ~]# mkdir -p /a/b/c
 [root@localhost ~]# touch /a/b/c/d.txt

 [root@localhost ~]# chomd -R 777 /a
 [root@localhost ~]# ll /a/b
  drwxrwxrwx 2 root root 19 12月 18 17:13 c

 [root@localhost ~]# ll /a/b/c/d.txt
  -rwxrwxrwx 1 root root 0 12月 18 17:13 /a/b/c/d.txt

# 置空权限：=-
[root@localhost ~]# chmod 000 /root/1.txt
或
[root@localhost ~]# chmod u=-,g=-,o=- /root/1.txt

[root@localhost ~]# ll /root/1.txt
 ---------- 1 root root 0 12月 18 16:42 /root/1.txt

```

## 权限的作用

```bash
1、权限对于用户的意义
  1、普通用户是严格遵守权限的
  2、root用户是高于权限
  3、权限需要重新登才生效（su和su - 都可以）


2、权限对于文件的意义
  1、r：读取文件内容
  2、w：修改文件内容
  3、x：可以把文件当成一个命令/程序运行


3、权限对于目录的意义
  1、r：可以浏览该目录下子目录名和子文件名（路径的最小权限是必须拥有可执行权限）
  2、w：创建、删除、移动（同上）
  3、x：可以进入该目录（同上）


# 1、在目录下，只有可读权限，无法查看文件内容（因为你无法操作目录）
# 2、在目录下，只有可读可写权限，无法查看文件内容（理由同上）
# 3、要想查看文件夹下的文件，文件夹必须至少拥有可执行权限；同时文件必须拥有可读权限
```

## 权限之特殊权限

### SUID

- 作用对象必须是二进制文件（cat 出来是乱码的文件）
- 该文件必须拥有可执行权限

```bash
chmod 4xxx [文件名]
or
chmod	u+s	[文件名]

 [root@localhost ~]# ll `which passwd`
  -rwsr-xr-x 1 root root 27856 4月   1 2020 /usr/bin/passwd

查看passwd命令的权限，我们可以看到一个不曾见到过的权限 s

而 S 权限是一个特殊权限
	SUID 权限仅对⼆进制可执⾏⽂件有效
    如果执⾏者对于该⼆进制可执⾏⽂件具有 s 的权限，执⾏者将具有该⽂件的所有者的权限
    本权限仅在执⾏该⼆进制可执⾏⽂件的过程中有效

# 示例：
[root@localhost ~]# su - user  # 切换到普通用户user10

[user@localhost ~]$ cat /etc/shadow  # 查看用户密码文件
cat: /etc/shadow: 权限不够   # 显示权限不够，没办法访问

[user@localhost ~]$ ll /etc/shadow  # 查看/etc/shadow 可以看出只有root用户才可以查看
----------. 1 root root 997 6月  20 18:02 /etc/shadow

[root@localhost ~]# ll `which cat`  # 登录root用户，查看cat命令
-rwxr-xr-x. 1 root root 54160 10⽉ 31 2018 /usr/bin/cat

[root@localhost ~]# chmod 4755 `which cat` # 或者 chmod u+s `which cat` 修改权限

[root@localhost ~]# ll `which cat`  # 现在cat命令的权限已经改成了s
-rwsr-xr-x. 1 root root 54160 10⽉ 31 2018 /usr/bin/cat

[root@localhost ~]# su - user		# 再次切换到user10用户
[user@localhost ~]$ cat /etc/shadow # 可以看到用户密码文件的内容了
```

### SBIT

SBIT 是 the restricted deletion flag or sticky bit 的简称，有时也称为 Sticky。

- 只对目录有效，用来阻止非文件的所有者删除文件，比较常见的就是/tmp 目录

```bash
chmod o+t [文件名]
or
chmod 1xxx [文件名]

[root@localhost ~]# ls -dl /tmp/ # -d 参数表示不查看目录下文件信息，就查看这个目录的信息
  drwxrwxrwt. 13 root root 4096 8⽉ 11 17:09 /tmp/

权限信息中最后⼀位 t 表明该⽬录被设置了 SBIT 权限。

SBIT 对⽬录的作⽤是：
  当⽤户在该⽬录下创建新⽂件或⽬录时，仅有⾃⼰和 root 才有权⼒删除，主要作⽤于⼀个共享的⽂件夹。
```

### SGID

- 用户对某一目录具有写和执行权限，该用户就可以在该目录下建立文件，如果该目录被 SGID 修饰，则该用户在这个目录下建立的文件都是属于这个目录所属的组

```bash
chmod g+s [文件名]
or
chmod 2xxx [文件名]

ps:当 SGID 作⽤于普通⽂件时，和 SUID 类似，在执⾏该⽂件时，⽤户将获得该⽂件所属组的权限。
```

## Umask

#### 新建的文件、目录的默认权限是由 umask 决定的

- uid > 199 并且属主与数组相等的⽤户下比如 test ，umask: 0002

  - 文件权限：664
  - 目录权限：775

- 除 1 之外的其他⽤户下，⽐如 root ⽤户，umask: 0022

  - 文件权限：644
  - 目录权限：755

  **在 Linux 中，常用的文件的权限是 666，目录的权限是 777**

```bash
1、文件权限计算方法：
    文件的权限是跟 umask 值相减，遇到奇数加一；遇到偶数则不变。
2、目录权限计算方法：
    目录的权限只要跟 umask 值相减即可。

总结：umask设置的越小，权限就越大，慎用

# 默认权限
  默认文件权限：644
	默认的目录权限：755

   [root@localhost ~]# touch a.txt  #创建文件
   [root@localhost ~]# ll a.txt
    -rw-r--r-- 1 root root 0 12月 18 20:55 a.txt  #默认权限644

	 [root@localhost ~]# mkdir b  #创建目录
   [root@localhost ~]# ll -d b
    drwxr-xr-x 2 root root 6 12月 18 20:57 b  #默认权限755
   [root@localhost ~]# umask
    0022


# 临时设置umask
[root@localhost ~]# umask 000   #设置umask权限


# 永久设置umask
#更改配置文件
[root@localhost tmp]# vim /etc/profile # 或者/etc/bashrc内容⼀样
......
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
 umask 002 #表示uid⼤于等于199的默认umask值，表示普通⽤户
else
 umask 022 #表示uid⼩于199的默认umask值，表示root
fi


需求：
		要求把1个月之前修改过的日志文件删除。

案例:
	案例1：将index文件添加属主 : 可读可写可执行、属组 ：可读可写、其他人：没有任何权限
		chmod 760 index

 	案例2：将baidu下的所有文件设置rwxr--r--
 		chmod -R  744  baidu/

 	案例3：将index这个文件的属组增加一个可执行权限。
 		chmod g+x index
 		chmod g+x,o-r index
```
