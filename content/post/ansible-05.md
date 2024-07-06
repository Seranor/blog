---
title: Task控制
lastmod: 2021-05-03T16:43:23+08:00
date: 2021-05-02T11:52:03+08:00
tags:
  - Ansible
categories:
  - Ansible
url: post/ansible-05.html
toc: true
---

## `Task Control`

### `when`条件语句

when 关键字主要针对 TASK 任务进行判断，对于此前我 们使用过的 yum 模块是可以自动检测软件包是否已被安 装，无需人为干涉；但对于有些任务则是需要进行判断 才可以实现的。

- web 节点都需要配置 nginx 仓库，但其他节 点并不需要，此时就会用到 when 判断

- Centos 与 Ubuntu 都需要安装 Apache，而 Centos 系统软件包为 httpd，而 Ubuntu 系统软件 包为 httpd2，那么此时就需要判断主机系统，然后 为不同的主机系统安装不同的软件包
<!-- more -->

```bash
# 为所有主机安装 Apache 软件
1.系统为CentOS：安装 httpd
2.系统为Ubuntu：安装 httpd2

cat facts4.yml
- hosts: web
  tasks:
    - name: Centos Install httpd
      yum:
        name: httpd
        state: present
      when: (ansible_distribution =="CentOS")
    - name: Ubuntu Install httpd
      yum:
        name: apache2
        state: present
      when: (ansible_distribution =="Ubuntu")
```

针对主机名为 web 的机器添加 nginx 仓库

```yaml
- hosts: all
  tasks:
    - name: Add Nginx Yum Repository
      yum_repository:
        name: ansible_web_nginx
        description: Nginx Repository
        baseurl: http://nginx.org/packages/centos/7/$basearch/
        gpgcheck: no
      when: (ansible_hostname is match("web*"))
#当然when也可以使用and与or方式
#when: (ansible_hostname is match("web*")) or
# (ansible_hostname is match("lb*"))
```

判断 httpd 服务是否处于运行状态

```yaml
# 判断 httpd 服务是否处于运行状态
# 已运行：则重启服务
# 未运行：则不做处理
# 通过 register 将命令执行结果保存至变量，然后通过 when 语句进行判断

- hosts: webservers
  tasks:
    - name: Get Httpd Server Status
      shell:
        cmd: systemctl status httpd &>/dev/null
      register: Httpd_Check

    - name: Debug
      debug:
        msg: "{{ Httpd_Check.rc }}"

    - name: Restart Httpd
      systemd:
        name: httpd
        state: restarted
      when: Httpd_Check.rc == 0
```

为特定的主机执行任务

```bash
# 有2台 server
# 第一台：172.16.1.7安装了 nginx
# 第二台：172.16.1.8没有安装 nginx
# 现在需要在没有安装 nginx的节点上做操作，需要通过 when 条件语句实现


- hosts: all
  tasks:
    - name: Get System Install Nginx
      shell:
        cmd: rpm -qa nginx | wc -l
      register: get_nginx

    - name: Create Nginx File
      file:
        path: /tmp/nginx_not_install.txt
        state: touch
      when: get_nginx.stdout == '0'
```

### `loop` 循环语句

在写 playbook 的时候发现了很多 task 都要重复引用 某个相同的模块，比如一次启动 10 个服务，或者一次拷 贝 10 个文件，如果按照传统的写法最少要写 10 次，这样 会显得 playbook 很臃肿。如果使用循环的方式来编写 playbook，这样可以减少重复编写 task 带来的臃肿

```bash
# 循环启动nginx和php-fpm
- hosts: webservers
  tasks:

    - name: Systemd Nginx php
      systemd:
        name: "{{ item }}"
        state: started
      loop:
        - nginx
        - php-fpm

# 循环安装httpd mariadb
- hosts: webservers
  tasks:

    - name: Installed Httpd Mariadb Package
      yum:
        name: "{{ item }}"
        state: latest
      loop:
         - httpd
         - mariadb-server

# 循环拷贝配置文件
- hosts: webservers
  tasks:

    - name: Rsync rsyncd.conf
      copy:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
        owner: root
        group: root
        mode: "{{ item.mode }}"
      loop:
        - { src: rsyncd.conf , dest: /tmp/rsyncd.conf , mode: "0644"  }
        - { src: rsyncd.pass , dest: /root/rsync.pass , mode: "0600" }
        - { src: rsynnd.test , dest: /mnt/rsync.test , mode: "0000" }


# 循环创建user
- hosts: webservers
  tasks:
    - name: Add Users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop:
        - { name: 'testuser1', groups: 'bin' }
        - { name: 'testuser2', groups: 'root' }

# 循环创建文件
- hosts: webservers
  tasks:

    - name: Create /data /backup
      file:
        path: "{{ item }}"
        state: directory
      loop:
        - /data
        - /backup
```

### `Handlers`和`Notify`

`Handlers` 是一个触发器，同时是一个特殊的 `tasks`， 它无法直接运行，它需要被 tasks 通知后才会运行

比如：httpd 服务配置文件发生变更，我们则可通过 Notify 通知给指定的 handlers 触发器，然后执行相 应重启服务的操作，如果配置文件不发生变更操作，则 不会触发 Handlers 任务的执行

handlers 注意事项

- 1.无论多少个 task 通知了相同的 `handlers`， `handlers`仅会在所有`tasks` 结束后运行一次

- 2.只有`task`发生改变了才会通知`handlers`，没 有改变则不会触发`handlers`

- 3.不能使用 handlers 替代 `tasks`、因为`handlers`是一个特殊的`tasks`

### `tags`任务标签

默认情况下，`Ansible` 在执行一个` playbook` 时，会执 行` playbook`中所有的任务。而标签功能是用来指定要 运行 `playbook`中的某个特定的任务；

- 为 `playbook` 添加标签的方式有如下几种

  1. 对一个`task`打一个标签
  2. 对一个 `task`打多个标签
  3. 对多个 `task`打一个标签

- `task`打完标签使用的几种方式
  1. `-t` 执行指定`tag`标签对应的任务
  2. `--skip-tags` 执行除` --skip-tags` 标签之外的所有任务

```bash
cat f5.yml
- hosts: nfs
  tasks:
    - name: Install Nfs Server
      yum: name=nfs-utils state=present
      tags:
        - install_nfs
        - install_nfs-server

    - name: Service Nfs Server
      service: name=nfs-server state=started enabled=yes
      tags: start_nfs-server


ansible-playbook f5.yml -t install_nfs
ansible-playbook f5.yml --skip-tags install_nfs
```

### `include`任务复用

有时，我们发现大量的 Playbook 内容需要重复编写， 各 Tasks 之间功能需相互调用才能完成各自功能， Playbook 庞大到维护困难，这时我们需要使用 include 比如：A 项目需要用到重启 httpd，B 项目需要用到，重 启 httpd，那么我们可以使用 Include 来减少重复编 写

```bash
cat main.yml
- hosts: web
  tasks:
  - name: Install Tomcat8 Server
    include: install_tomcat_8.yml
    tags: install_tomcat_8

  - name: Install Tomcat9 Server
    include: install_tomcat_9.yml
    tags: install_tomcat_9


install_tomcat_8.yml
install_tomcat_9.yml

ansible-playbook main.yml -t install_tomcat_8
```

## `Playbook`异常处理

在 playbook 执行的过程中，难免会遇到一些错误。由 于 playbook 遇到错误后，不会执行之后的任务，不便 于调试，此时，可以使用 ignore_errors 来暂时忽略 错误，使得 playbook 继续执行

```bash
- hosts: all
  remote_user: root
  tasks:
    - name: Ignore False
      command: /bin/false
      ignore_errors: yes

    - name: touch new file
      file: path=/tmp/oldxu_ignore state=touch
```
