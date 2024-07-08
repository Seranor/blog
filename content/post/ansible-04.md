---
title: Variable和Register的使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Ansible
categories:
  - Ansible
url: post/ansible-04.html
toc: true
---

## `Ansible Variable`

### 介绍

```bash
变量提供了便捷的方式来管理 ansible 项目中的动态值。
比如 nginx-1.12，可能后期会反复的使用到这个版本的值，那么如果将此值设置为变量，后续使用和修改都将变得非常方便
```

<!-- more -->

### 定义方式

```bash
在 Ansible 中定义变量分为如下三种方式：
1.通过命令行传递变量参数定义
2.在play文件中进行定义变量
    2.1 通过vars定义变量
    2.2 通过vars_files定义变量
3.通过inventory在主机组或单个主机中设置变量
  3.1 通过host_vars对主机进行定义
  3.2 通过group_vars对主机组进行定义

问题：如果定义的变量出现重复，造成冲突，如何解决？
```

### 在 playbook 中定义变量

#### vars 方式

```bash
# 在 playbook 的文件中开头通过 vars 关键字进行变量定义

- hosts: web
  gather_facts: false  # 不获取被控集机器的fact数据，速度会变快
  vars:
    - web_packages: httpd
    - ftp_packages: vsftpd

  tasks:
    - name: Output Vaiables
      debug:
        msg:
          - "{{ web_packages }}"
          - "{{ ftp_packages }}"
```

#### `vars_file`方式

```bash
在 playbook 中使用 vars_files 指定文件作为变量文件，好处就是其他的 playbook 也可以调用

cat vars.yml
web_packages:
  - httpd
  - nginx
ftp_packages: vsftpd

cat f2.yml
- hosts: web
  gather_facts: false
  vars_files: ./vars.yml
  tasks:
    - name: Output Vaiables
      debug:
        msg:
          - "{{ web_packages }}"
          - "{{ ftp_packages }}"
```

### 在 inventory 中定义变量

定义主机变量和组变量

```bash
cat /etc/ansible/hosts
[webserver]
172.16.1.7 myid=1 state=master    # 定义的主机变量
172.16.1.8 myid=2 state=backup

[webserver:vars]
port=80  # 定义组变量

# 模拟配置文件
cat ./files/test.conf.j2
hostrole: {{ state  }}
myid: {{ myid }}
port: {{ port }}

# 根据变量替换公共配置文件，使不同的主机有不同的配置
cat f3.yml
- hosts: webserver
  gather_facts: false
  tasks:
    - name: Output Vaiables
      template:
        src: ./files/test.conf.j2
        dest: /tmp/test.conf

# 执行
ansible-playbook f3.yml

# 登录每一台主机进行检查配置文件查看是否根据变换配置
```

#### host_vars 定义变量

在项目目录中创建 host_vars 目录，然后在创建一个 文件，文件的文件名称要与 inventory 清单中的主机名 称要保持完全一致，如果是 ip 地址，则创建相同 ip 地址 的文件即可

```bash
# hosts文件改回去
cat /etc/ansible/hosts
[webserver]
172.16.1.7
172.16.1.8

# 在当前项目创建这个固定的目录
mkdir host_vars

# 创建对应的IP文件，写入变量内容
cat host_vars/172.16.1.7
state: MASTER
myid: 10

cat host_vars/172.16.1.8
state: BACKUP
myid: 20

# 执行上述 f3.yml
ansible-playbook f3.yml
```

#### `group_vars`定义变量

在项目目录中创建 group_vars 目录，然后在创建一 个文件，文件的文件名称要与 inventory 清单中的组名 称保持完全一致，但是系统提供了特殊的 all 组，也就说在 group_vars 目录下创建一个 all 文件，定义变量对所 有的主机组都生效

```bash
cat /etc/ansible/hosts
[webserver]
172.16.1.7
172.16.1.8

mkdir group_vars

cat group_vars/webserver
port: 8080

ansible-playbook f3.yml
```

### `Playbook`传递变量

在执行 Playbook 时，可以通过命令行 --extra-vars 或 - e 外置传参设定变量

```bash
rm -f group_vars/webserver

# 传递单个变量
ansible-playbook f3.yml  -e "port=9090"

# 可以使用多个 -e 拼接或者在一个 "" 内，可以使用这个方法运行不同的组定义不同的变量
cat group_vars/all
port: 8080

cat f4.yml
- hosts: "{{ host }}"
  gather_facts: false
  tasks:
    - name: Output Vaiables
      template:
        src: ./files/test.conf.j2
        dest: /tmp/test.conf

ansible-playbook f4.yml  -e "host=lb port=9090 state=MasTer myid=23"
```

### 变量优先级

```bash
定义相同的变量不同的值，来测试变量的优先级。操作步骤如下:
1）在plabook中定义vars变量
2）在playbook中定义vars_files变量
3）在host_vars中定义变量
4）在group_vars中定义变量
5）通过执行命令传递变量

结果(从高到低):
命令行传参
playbook中的vars_files
playbook中的vars
host_vars
group_vars
group_vars/all
```

### 使用变量改写`NFS`

变量文件已经配置文件

```bash
cat exports.j2
{{ nfs_share_data }} (rw,all_squash,anonuid={{ nfs_uid }},anongid={{ nfs_gid }})

cat nfs_variables.yml
nfs_share_data: /fff
nfs_uid: 5655
nfs_gid: 5655
nfs_group: dddd
nfs_user: dddd
```

playbook 文件

```yaml
- hosts: nfs-server
  gather_facts: false
  vars_files: ./nfs_variables.yml
  tasks:
    - name: 1.Install NFS Server
      yum:
        name: nfs-utils
        state: present

    - name: 2.Configure NFS Server
      template:
        src: ./exports.j2
        dest: /etc/exports
      notify: Restart NFS Server

    - name: 3.Created Group
      group:
        name: "{{ nfs_group }}"
        gid: "{{ nfs_gid }}"

    - name: 4.Created User
      user:
        name: "{{ nfs_user }}"
        uid: "{{ nfs_uid }}"
        group: "{{ nfs_group }}"
        shell: /sbin/nologin
        create_home: no

    - name: 5.Init Create  Directory
      file:
        path: "{{ nfs_share_data }}"
        state: directory
        owner: "{{ nfs_user }}"
        group: "{{ nfs_group }}"
        mode: "0755"

    - name: 6.Started NFS Server
      systemd:
        name: nfs
        state: started
        enabled: yes

  handlers:
    - name: Restart NFS Server
      systemd:
        name: nfs
        state: restarted

- hosts: localhost
  gather_facts: false
  vars_files: ./nfs_variables.yml
  tasks:
    - name: 7.Install NFS Cli
      yum:
        name: nfs-utils
        state: present

    - name: 8. Cli Mount
      mount:
        src: "192.168.100.7:{{ nfs_share_data }}"
        path: /mnt
        fstype: nfs
        opts: defaults
        state: mounted
```

## `Ansible Register`

register 可以将 task 执行的任务结果存储至某个变 量中，便于后续的引用

输出命令的结果

```yaml
- hosts: webservers
  gather_facts: false
  tasks:
    - name: Get Network Port
      shell: netstat -lntp
      register: System_Port

    - name: debug
      debug:
        msg:
          - "执行了 {{ System_Port.cmd }} 命令"
          - "{{ System_Port.stdout_lines  }}"
```

批量修改随机主机名称

```yaml
- hosts: webservers
  gather_facts: false
  tasks:
    - name: String
      shell: echo $RANDOM |md5sum | cut -c 2-10
      register: systemd_sj

    - name: debug
      debug:
        msg: "{{ systemd_sj.stdout }}"

    - name: Chanage Hostname
      hostname:
        name: "web_{{ systemd_sj.stdout }}"
```

## ` Facts Variables`

### 简介

```bash
Ansible facts 变量主要用来自动采集，“被控端主机”自身的状态信息。 比如：被动端的，主机名、IP地址、系统版本、CPU数 量、内存状态、磁盘状态等等。

使用场景:
1.通过facts变量检查被控端硬件CPU信息，从而生成不同的Nginx配置文件
2.通过facts变量检查被控端内存状态信息，从而生成不同的memcached的配置文件
3.通过facts变量检查被控端主机名称信息，从而生成不同的Zabbix配置文件
4.通过facts变量检查被控端主机IP地址信息，从而生成不同的redis配置文件
```

### 使用

```bash
# 可以获取被控端的主机信息
ansible localhost -m setup

# 查看被控端的hostname IP 系统类型，注：gather_facts 此时就不能为 false 了
- hosts: web
  tasks:
    - name: Print msg
      debug:
        msg: "{{ ansible_nodename }} {{ ansible_default_ipv4.address }} {{ ansible_distribution }}"
```

### 根据主机地址 IP 生成配置

```bash
cat ./files/tes2t.conf.j2
IP_Server: {{ ansible_default_ipv4.address }}

cat facts2.yml
- hosts: web
  tasks:
    - name: Output Vaiables
      template:
        src: ./files/test2.conf.j2
        dest: /tmp/test2.conf

ansible-playbook facts2.yml

# 检查web组内的每个主机查看 /tmp/test2.conf 是否按照IP生成了对应内容
```

### 批量修改主机名

```bash
cat facts3.yml
- hosts: web
  tasks:
    - name: Get msg
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Change hostname
      hostname:
        name: "web_{{ ansible_default_ipv4.address.split('.')[-1] }}"

ansible-playbook facts3.yml
```

### `Redis`缓存`facts`变量加速

当我们使用 gather_facts: no 关闭 facts，确实能加速 Ansible 执行，但是有时候又需要使用 facts 中的内容，还希望执行的速度快一点，这时候可以设置 facts 的缓存

```bash
[defaults]
# smart 表示默认收集 facts，但 facts 已有的情况下不会收集，即使用缓存 facts
# implicit 表示默认收集 facts，要禁止收集，必须使用 gather_facts: False；
# explicit 则表示默认不收集，要显式收集，必须使用gather_facts: Ture

#在使用 facts 缓存时设置为smart
gathering = smart
fact_caching_timeout = 86400
fact_caching = redis
fact_caching_connection = 172.16.1.41:6379:1

# 若 redis 设置了密码
# fact_caching_connection =172.16.1.41:6379:1:passwd  1 库
# 整体性能可以提升二到三倍以上
```
