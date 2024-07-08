---
title: Ansible简介、安装
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Ansible
categories:
  - Ansible
url: post/ansible-01.html
toc: true
---

## `Ansible`介绍

### 简 介

ansible 是自动化运维工具，基于 Python 开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等

<!-- more -->

### 特性

- 优势

  ```
  1.ansible不需要单独安装客户端，也不需要启动任何服务
  2.ansible是python中的一套完整的自动化执行任务模块
  3.ansible playbook，采用yaml配置，对于自动化任务执行一目了然
  4.ansible 模块较多，对于自动化的场景支持较丰富
  ```

- 劣势

  ```
  1.幂等性，每次的描述一种状态后，服务器会按照你所期望的状态去运行；出了问题无法回退，需要重新在描述一次状态，然后执行，以实现回退的效果

  2.效率，如果连接的主机较多，执行的速度会比较的慢，速度相对比saltstack慢
  ```

### 基础架构

```
1.连接插件connectior plugins用于连接主机 用来连接被管理端
2.核心模块 core modules 连接主机实现操作， 它依赖于具体的模块来做具体的事情
3.自定义模块 custom modules，根据自己的需求编写具体的模块
4.插件 plugins，完成模块功能的补充
5.剧本 playbooks，ansible的配置文件,将多个任务定义在剧本中，由ansible自动执行
6.主机清单 inventor，定义ansible需要操作主机的范围
最重要的一点是 ansible是模块化的 它所有的操作都依赖于模块
```

![image-20220103130441938](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220103130441938.png)

## 安装部署

### 包管理方式(`yum`)

```bash
# centos需要有epel源
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# 直接安装
yum install ansible -y

# 查看版本
ansible --version

# 简单使用
ansible localhost -m ping
```

### `pip`安装

```bash
# 安装python3
yum -y install python3 python3-devel python3-pip

# 升级pip3
pip3 install --upgrade pip -i https://pypi.douban.com/simple/

# 安装ansible
pip3 install ansible -i https://pypi.douban.com/simple/

# 查看ansible安装位置
which ansible

# 查看ansible版本
/usr/local/bin/ansible --version
```

## 配置文件说明

### 默认配置文件

```bash
# 查看默认配置文件路径
rpm -qc ansible
/etc/ansible/ansible.cfg  # 主配置文件
/etc/ansible/hosts				# 主机清单文件 inventory

# /etc/ansible/ansible.cfg 部分说明
#inventory      = /etc/ansible/hosts			    # 默认主机清单路径
#library        = /usr/share/my_modules/      # 模块存放路径
#module_utils   = /usr/share/my_module_utils/ # 模块存放路径
#remote_tmp     = ~/.ansible/tmp              # 将模块推送到被控端的临时位置
#local_tmp      = ~/.ansible/tmp              # 本地模块临时存放位置
#forks          = 5											      #	默认并行的主机数量  可以使用参数 -f 单独指定
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
#remote_port    = 22
#host_key_checking = False								    # 第一次连接主机的 yes 需要打开注释
#roles_path    = /etc/ansible/roles           # 默认角色路径
```

### 配置文件优先级

```bash
# 配置文件加载顺序
第一步读取：ANSIBLE_CONFIG
第二步读取：当前项目目录下的ansible.cfg
第三步读取：当前用户家目录下的 .ansible.cfg
第四步读取: /etc/ansible/ansible.cfg

# 环境变量
export ANSIBLE_CONFIG=/tmp/ansible.cfg  # 定义
touch /tmp/ansible.cfg
ansible --version     # 可以看到配置文件位置是 /tmp/ansible.cfg
unset ANSIBLE_CONFIG  # 取消定义

# 为项目单独定义配置文件，非常的重要
mkdir project1
cd project1/
touch ansible.cfg
ansible --version  # 此时加载的配置文件位置就是 /root/project1/ansible.cfg
# 退出当前目录就变成默认的配置文件了

# 为当前执行的用户家目录植入一个配置文件；
touch ~/.ansible.cfg
ansible --version  # 此时加载的配置文件位置就是 ~/.ansible.cfg
rm -f ~/.ansible.cfg

# 默认的配置文件加载路径，优先级是最低的
ansible --version
```

## `Inventory`

### 简介

```bash
主要用来填写被管理主机及主机组信息(逻辑定义)
默认Inventory文件为/etc/ansible/hosts
也可以自定义一个我看，使用ansible命令 -i 参数指定Inventory文件位置
```

### 主机清单定义方式

#### 用户密码

```bash
# 直接写主机IP
[server1]
192.168.0.11 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'
192.168.0.12 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'

# 将密码改成变量，等同于上面
[server1]
192.168.0.11
192.168.0.11

[server1:vars]
ansible_ssh_port=22
ansible_ssh_user=root
ansible_ssh_pass='123456'

# 多台机器时
[server1]
192.168.0.[11:100] ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'

# 可以通过域名简写(前提在 hosts 文件中解析)
[server1]
server[1:2].test.com ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'
```

#### 免密

```bash
# 免交互生成秘钥对
ssh-keygen -f ~/.ssh/id_rsa  -P '' -q

yum install sshpass

sshpass -p123456 ssh-copy-id -f -i ~/.ssh/id_rsa.pub "-o StrictHostKeyChecking=no" root@10.0.0.100 >/dev/null 2>&1
# 可以依据上述编写批量免密推送脚本

# 主机清单
# 方式一
[server1]
192.168.0.11
192.168.0.12

# 方式二
[db]
192.168.0.[100:110]

# 方式三
[server1]
server[1:100]

# 查看 server 主机组内有多少机器，两种方式通用
ansible server  --list-hosts
```

### 匹配主机组的方式

`ansible`命令格式

```bash
ansible <host-pattern> [-m module_name] [-a args]
```

`host-pattern`的使用：

```bash
# 指定所有组
ansible all -m ping

# 通配符
ansible "server*" -m ping
ansible 192.168.0.* -m ping

# 与: 在server1组，并且在db1组中的主机
ansible "server1:&db1" -m ping

# 或: 在server1组，或者在db1组中的主机
ansible "server1:db1" -m ping

# 非: 在server1组，或者在db1组中的主机
ansible "server1:!db1" -m ping

# 正则表达式
ansible "~(web|db).*" -m ping
```

### 使用普通用户管理被控端

`ansible`使用`test`普通用户统一管理所有被控端节点

```bash
# 所有主机都创建 test 普通用户
useradd test
echo '123' |passwd --stdin test

# 管理机推送秘钥到被控端
su - test
ssh-keygen -f ~/.ssh/id_rsa  -P '' -q
sshpass -p123 ssh-copy-id -f -i ~/.ssh/id_rsa.pub "-o StrictHostKeyChecking=no" test@192.168.0.12 >/dev/null 2>&1

# 开启所有主机的sudo权限
visudo
test ALL=(ALL) NOPASSWD: ALL

# 控制端配置文件中配置普通用户提权
[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

# 不配置上述参数可以使用 -b -K 输入密码即可
```
