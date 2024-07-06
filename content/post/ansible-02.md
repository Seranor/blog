---
title: Ansible常用模块
lastmod: 2021-05-03T16:43:23+08:00
date: 2021-05-02T11:52:03+08:00
tags:
  - Ansible
categories:
  - Ansible
url: post/ansible-02.html
toc: true
---

## `ad-hoc`与常用模块

`ad-hoc`就是临时命令，执行完成即结束，并不会保存

可以查看多台节点是的进程是否存在

拷贝指定文件至本地

<!-- more -->

### 使用范例

```
ansible 'groups' -m command -a "df -h"
```

![image-20220103175305795](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220103175305795.png)

### `ad-hoc`执行过程

```
1.加载配置文件，默认 /etc/ansible/ansible.cfg
2.读取inventory
3.操作对应的目标主机组；如果组不存在则报错
4.构建对应的py文件，推送到远程目标主机
5.远程主机执行该文件
6.执行完成后，删除对应的py文件
7.像服务端返回最终执行结果
```

### 执行状态

```bash
返回结果的颜色说明
绿色: 代表被管理端主机没有被修改
黄色: 代表被管理端主机发现变更
红色: 代表出现了故障，注意查看提示
```

### 单独项目使用`ansible`

```bash
# 创建项目
cd ~
mkdir project1
cd project1

# 项目配置文件和主机文件
cp /etc/ansible/ansible.cfg .
cp /etc/ansible/hosts  hosts_group

# 修改配置文件
vim ansible.cfg
...
inventory      = ./hosts_group
...

# 查看读取的配置文件
ansible --version

# 执行
ansible all -m ping
```

## `Ansible`常用模块

常用模块较多，可以使用`ansible-doc  模块名`查看帮助

### command 模块

功能：在远程主机执行 shell 命令，是一个默认模块，可以忽略参数 `-m`，但是不支持管道命令 `|`

| 参数    | 选项               | 含义                              |
| ------- | ------------------ | --------------------------------- |
| chdir   | chdir /opt         | 执行 ansible 时，切换到指定的目录 |
| creates | creates /data/file | 如果文件在，则跳过执行            |
| removes | removes /data/file | 如果文件存在，则执行              |

```bash
# 当 /data/opt 不存在时，就会执行 ifconfig eth0 命令
ansible all -m command -a "creates=/data/opt ifconfig eth0"
# 备份: 如果备份的文件存在，则不执行备份的命令；如果文件不存在，则执行备份的命令


# 当 /data/opt 存在时，就会执行 ifconfig eth0 命令
ansible all -m command -a "removes=/data/opt ifconfig eth0"
```

### `shell`模块

command 支持在这个模块一样能执行，被控端已有的`shell`命令都可以执行，且支持管道

### `yum`模块

```bash
name:  # 软件包名称
  state: # 状态
		present # 安装
		absent # 删除
		latest # 最新版

enablerepo  # 通过哪个仓库获取
disablerepo  # 不使用哪些仓库的包
excludekernel  # kernel排除
```

使用

```bash
# 1.安装vsftpd软件包
ansible all -m yum -a 'name=vsftpd state=present'

# 2.删除vsftpd
ansible all -m yum -a 'name=vsftpd state=absent'

# 3.安装httpd服务，必须从epel仓库中安装(所有的被控都有这个epel仓库)
ansible all -m yum -a 'name=httpd state=present enablerepo=epel'

# 4.更新所有的软件包，唯独kernel程序不更新
ansible all -m yum -a 'name=* state=present exclude="kernel*"'
```

### `copy`模块

控制端的文件，拷贝到被控端，实现替换

```bash
src:      控制端的源文件路径
dest:     被控端的文件路径
owner:    属主
group:    属组
mode:     权限
backup：   备份
validate： 验证
content:   往一个文件写入内容


# 1.更新nfs配置，将控制端的 exports.j2 文件同步到被控端的 /etc/exports
ansible all -m copy -a 'src=./exports.j2 dest=/etc/exports owner=root group=root mode=0644 backup=yes'

# 2.往一个文件中写入内容，如果文件不存在则创建
ansible all -m copy -a 'content="123" dest=/data/test.txt owner=root group=root mode="0600" backup=yes'


# 3.验证sudo配置是否正确(adhoc测试失败)：
cat tt.yaml
- hosts: all
  tasks:
  - name: Copy a "sudoers" file on the remote machine for editing
    copy:
      src: ./sudoers
      dest: /etc/sudoers
      validate: /usr/sbin/visudo -csf %s

ansible-ploybook tt.yamy
```

### `systemd`模块

```bash
name                # 服务名称
state               # 服务状态
  started           # 启动
  stopped           # 停止
  restarted         # 重启
  reloaded          # 重载
enabled              # 开启自启动
daemon_reload: yes  # 重新刷新daemon-reload

# 启动nfs并设置开机自启
ansible all -m systemd -a "name=nfs state=started enabled=yes"

# 停止nfs并关闭开机自启动
ansible all -m systemd -a "name=nfs state=stopped enabled=no"
```

### `file`模块

```bash
# 创建文件、创建目录、授权
  file:
    path: 		在被控端创建的路径
    owner: 		属主
    group: 		属组
    mode: 		权限
	state: 		  类型
		touch:    文件
		directory： 目录
		link: 软链接
		hard：硬链接
	recurse： yes 递归授权


# 被控端需要有 www 用户
gropuadd -g 888 www
useradd -u 888 -g 888 www


# 创建一个/data/www 目录，授权为 www 身份
ansible webservers -m file -a 'path=/data/www owner=www group=www mode="0755" state=directory recurse=yes'

# 在 /data/www/ 目录中创建一个文件
ansible webservers -m file -a 'path=/data/www/books.html owner=www group=www mode="0644" state=touch'
```

### `group`模块

```bash
name: 指定组名称
  gid： 指定gid
	state:
		present：创建  默认
		absent：删除


# 指定 gid 为 666 创建 www 组
ansible webservers -m group -a 'name=www gid=666 state=present'

# 创建 mysqldb 组为系统的 gid
ansible webservers -m group -a 'name=mysqldb system=yes state=present'
```

### `user`模块

```bash
user:
  name: 				创建的名称
  uid: 				  指定uid
  group: 				指定基本组
  shell: 				登录shell类型默认/bin/bash
  create_home		是否创建家目录
	password			设定对应的密码，必须是加密后的字符串才行，否则不生效；
	system				系统用户

	groups: admins,dev  附加组
	append: yes			追加

  state: absent		删除
  remove: yes			家目录一起结束



# 创建 www 用户，指定 uid 666，基本组 www ，不创建家目录
ansible webservers -m user -a 'name=www uid=666 group=www shell=/sbin/nologin create_home=no'

# 创建db用户，基本组是root，附加组，adm，sys
ansible webservers -m user -a 'name=db group=root groups=adm,sys append=yes shell=/bin/bash create_home=yes'

# 创建一个ddd用户，密码123，需要正常登录系统
# 生成密码
ansible localhost -m debug -a "msg={{ '123' | password_hash('sha512','salt')}}"

ansible webservers -m user -a 'name=ddd password=$6$salt$jkHSO0tOjmLW0S1NFlw5veSIDRAVsiQQMTrkOKy4xdCCLPNIsHhZkIRlzfzIvKyXeGdOfCBoW1wJZPLyQ9Qx/1 shell=/bin/bash create_home=yes'

# 创建一个dev用户，并为其生成对应的秘钥
ansible webservers -m user -a 'name=dev generate_ssh_key=yes ssh_key_bits=2048 ssh_key_file=.ssh/id_rsa'
```

### `mount`模块

```bash
src: 源设备路径，或网络地址；
  path: 挂载至本地哪个路径下；
  fstype: 设备类型； nfs
  opts:   挂载的选项
  state:  挂载还是卸载
    present			永久挂载，但没有立即生效
    absent			卸载，临时挂载+永久挂载
    mounted			临时(fstab)挂载
    unmounted		临时卸载

# 将 172.16.1.7 的 /data 目录挂载到 172.16.1.8 /mnt 目录
ansible 172.16.1.8 -m mount -a 'src=172.16.1.7:/data path=/opt fstype=nfs opts=defaults state=mounted'
```

### `cron`模块

```bash
name: 		 描述信息，描述脚本的作用
minute: 	 分钟
hour:		   小时
weekday:	 周
user:		   该任务由哪个用户取运行；默认root
job: 		   任务


# 每天凌晨3点执行 /bin/bash /scripts/client_push_data_server.sh &>/dev/null
ansible webservers -m cron -a 'name="backups app data scripts" hour=03 minute=00 job="/bin/bash /scripts/client_push_data_server.sh &>/dev/null"'

# 删除 backups app data script  执行
ansible webservers -m cron -a 'name="backups app data scripts" hour=03 minute=00 job="/bin/bash /scripts/client_push_data_server.sh &>/dev/null" state=absent'

# 注释 backups app data script  执行
ansible webservers -m cron -a 'name="backups app data scripts" hour=03 minute=00 job="/bin/bash /scripts/client_push_data_server.sh &>/dev/null" disabled=yes'

```

### `get_url`

```bash
get_url:
  url: 下载地址
  dest: 下载到本地的路径；
  mode: 权限；
  checksum：对资源做校验；

# 下载一个资源到本地/tmp目录；
ansible webservers -m get_url -a 'url=http://nginx.org/download/nginx-1.20.2.tar.gz dest=/tmp mode=0666'

# 对下载的资源做验证：
ansible webservers -m get_url -a 'url=http://nginx.org/download/nginx-1.20.2.tar.gz dest=/opt mode=0666 checksum=md5:3bcc5ccdc052c35d0d3c5557cf56c7d2'
```

### `unarchive`模块

```bash
# unarchive解压
unarchive:
  src: 			  控制端的源文件
  dest: 			解压到被控端的路径
remote_src: yes	源文本是否在被控端，yes则在，no则不在

# 将控制端的压缩包，解压到被控端 remote_src: no
ansible webservers -m unarchive -a 'src=./test.tar.gz dest=/mnt'

# 将被控端的压缩包解压到被控端  remote_src: yes   config_vpn_new.zip
ansible webservers -m unarchive -a 'src=/tmp/test.tar.gz dest=/mnt remote_src=yes'

# archive 压缩
# 将被控端的/opt 打包到 /mnt 目录下，并命名为 opt.tar.gz
ansible webservers -m archive -a 'path=/opt dest=/mnt/opt.tar.gz format=gz'
```

### `selinux`模块

```bash
selinux 防火墙模块：
ansible webservers -m selinux -a 'state=disabled'
```

### `firewalld`模块

```bash
zone：		要操作的区域  默认public
	source：	来源地址
    service: 	 服务名称 http,https,sshd,......
	port:		端口
    permanent:	永久生效，但不会立即生效
	immediate：	临时生效；
    state: 		启用和关闭；
		disabled
		enabled

# 让被控端都放行80端口；
ansible webservers -m systemd -a 'name=firewalld state=started'
ansible webservers -m firewalld -a 'port=80/tcp immediate=yes state=present'

# 让被控端都放行https端口；
ansible webservers -m systemd -a 'name=firewalld state=started'
ansible webservers -m firewalld -a 'service=https immediate=yes state=present'
```

### `iptables`

```bash
iptables:
	table: 					  表
    chain: 					链
    source: 				来源IP
	destination				目标IP
	destination_port  目标端口
	protocol	协议
    jump: DROP	动作
	action		如何添加规则
		insert：插入
		append：追加


# 来源IP是192.168.1.1 目标地址 1.1.1.1 目标端口 80  协议 tcp  则拒绝； 规则要写入第一行；
ansible webservers -m iptables -a 'table=filter chain=INPUT source=192.168.1.1/32 destination=1.1.1.1 destination_port=80 protocol=tcp jump=DROP action=insert'


# NAT：SNAT和DNAT：
	DNAT： 如果请求1.1.1:80端口，则DNAT到2.2.2.2:8800
	ansible webservers -m iptables -a 'table=nat chain=PREROUTING protocol=tcp destination=1.1.1.1 destination_port=80 jump=DNAT to_destination="2.2.2.2:8800"'

	DNAT： 如果请求1.1.1:81端口，则DNAT到3.3.3.3:8800
	ansible webservers -m iptables -a 'table=nat chain=PREROUTING protocol=tcp destination=1.1.1.1 destination_port=81 jump=DNAT to_destination="3.3.3.3:8800"'

SNAT:
	POSTROUTING
	iptables -t nat -I POSTROUTING -s 172.16.1.0/24 -j SNAT --to-source 5.5.5.5

ansible webservers -m iptables -a 'table=nat chain=POSTROUTING source=172.16.2.0/24 jump=SNAT to_source=6.6.6.6'
ansible webservers -m iptables -a 'table=nat chain=POSTROUTING source=172.16.3.0/24 jump=SNAT to_source=7.7.7.7 action=insert'
```

### `yum_repo`

```bash
yum_repository
	name				  名称,文件名称
	description	  描述，必填
	baseurl				仓库的地址
	gpgcheck			验证开启
	gpgkey


ansible webservers -m yum_repository -a 'name=ansible_nginx description=xxx baseurl="http://nginx.org/packages/centos/$releasever/$basearch/" gpgcheck=yes gpgkey="https://nginx.org/keys/nginx_signing.key"'
```

### `hostname`

```bash
hostname 修改主机名称：
    name: 	修改后的主机名称；

ansible webservers -m hostname -a 'name=web_cluster'
```

### `sysctl`

```bash
sysctl	修改内核参数模块
- sysctl:
    name: vm.swappiness
    value: '5'
    state: present

ansible webservers -m sysctl -a 'name=net.ipv4.ip_forward value=1 state=present'
```

### `lineinfile`

```bash
lineinfile	替换|追加|删除
    path: 					      被控端的路径
    regexp: '^Listen '		正则匹配语法格式
    line: Listen 8080			填充的内容
	state: absent					  删除
	insertafter: '^#Listen '
	insertbefore: '^www.*80/tcp'

# 替换httpd.conf文件中， ^Listen   为  Linsten 8080
ansible webservers -m lineinfile -a 'path=/etc/httpd/conf/httpd.conf regexp="^Listen" line="Listen 8080"'

# 给主机增加一个网关
ansible webservers -m lineinfile -a 'path=/etc/sysconfig/network-scripts/ifcfg-eth1 line="GATEWAY=172.16.1.200"'

# 删除主机的网关
ansible webservers -m lineinfile -a 'path=/etc/sysconfig/network-scripts/ifcfg-eth1 regexp="^GATEWAY" state=absent'

# 给主机增加一个网关，但需要增加到ONBOOT下面
ansible webservers -m lineinfile -a 'path=/etc/sysconfig/network-scripts/ifcfg-eth1 insertafter="ONBOOT=yes" line="GATEWAY=172.16.1.200"'

# 给主机增加一个网关，但需要增加到ONBOOT上面
ansible webservers -m lineinfile -a 'path=/etc/sysconfig/network-scripts/ifcfg-eth1 insertbefore="ONBOOT=yes" line="test=172.16.1.200"'
```
