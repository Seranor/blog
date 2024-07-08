---
title: Playbook使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Ansible
categories:
  - Ansible
url: post/ansible-03.html
toc: true
---

## `Playbook`入门

### Playbook 简介

<!-- more -->

```bash
playbook 是一个由 yml 语法编写的文本文件，它由 play 和 task 两部分组成
play:主要定义要操作主机或者主机组
task:主要定义对主机或主机组具体执行的任务，可以是一个任务，也可以是多个任务（模块）

总结: playbook 是由一个或多个 play 组成，一个play 可以包含多个 task 任务，
可以理解为: 使用多个不同的模块来共同完成一件事情
```

![image-20220106100622359](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220106100622359.png)

### `Playbook`与`Ad-hoc`

```bash
1) playbook 是对 AD-Hoc 的一种编排方式
2) playbook 可以持久运行，而 Ad-Hoc 只能临时运行
3) playbook 适合复杂的任务，而 Ad-Hoc 适合做快速简单的任务
4) playbook 能控制任务执行的先后顺序
```

### `Playbook`格式

playbook 是由 yml 语法书写，结构清晰，可读性强， 所以必须掌握 yml 语法

| 语法     | 描述                                                                       |
| -------- | -------------------------------------------------------------------------- |
| 缩 进    | YAML 使用固定的缩进风格表示层级结构,每个缩进由两个空格组成, 不能使用 tab   |
| 冒 号    | 以冒号结尾的除外，其他所有冒号后面所有必须 有空格                          |
| 短 横 线 | 表示列表项，使用一个短横杠加一个空格，多个项使用同样的缩进级别作为同一列表 |

```bash
host: 对哪些主机进行操作
remote_user: 我要使用什么用户执行
tasks: 具体执行什么任务

# 显示playbook执行时间
ansible2.0以上的版本需要在ansible.cfg中加入
callback_whitelist = profile_tasks
```

安装 nginx 的 playbook，install-nginx.yml

```yaml
- hosts: all
  tasks:
    - name: Install Nginx Server
      yum:
        name: nginx
        state: present

    - name: Systemd Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes
```

### 执行方式

```bash
ansible-playbook --syntax-check install-nginx.yml   # 检查语法
ansible-playbook -C install-nginx.yml         # 模拟执行
ansible-playbook install-nginx.yml            # 真实执行

执行playbook，注意观察执行返回的状态颜色:
红色：表示有task执行失败，通常都会提示错误信息
黄色：表示远程主机按照编排的任务执行且进行了改变
绿色：表示该主机已经是描述后的状态，无需在次运行
```

### 案例

#### 部署`NFS`

```bash
cat exports.j2
/ansible_test (rw,all_squash,anonuid=7777,anongid=7777)
```

`install-yaml`文件内容:

```yaml
- hosts: nfs-server
  tasks:
    - name: 1.Install NFS Server
      yum:
        name: nfs-utils
        state: present

    - name: 2.Configure NFS Server
      copy:
        src: ./exports.j2
        dest: /etc/exports
      # 定义触发器
      notify: Restart NFS Server

    - name: 3.Created Group
      group:
        name: bbb
        gid: 7777

    - name: 4.Created User
      user:
        name: bbb
        uid: 7777
        group: bbb
        shell: /sbin/nologin
        create_home: no

    - name: 5.Init Create  Directory
      file:
        path: /ansible_test
        state: directory
        owner: bbb
        group: bbb
        mode: "0755"

    - name: 6.Started NFS Server
      systemd:
        name: nfs
        state: started
        enabled: yes

  handlers:
    # 激活触发器,进行重启nfs，因为nfs服务如果在启动状态的话就不会再启动
    - name: Restart NFS Server
      systemd:
        name: nfs
        state: restarted
```

#### 部署`Rsync`

`rsyncd.conf.j2`文件内容

```bash
uid = ansible_www
gid = ansible_www
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
#ignore errors
read only = false
list = false
auth users = rsync_backup
secrets file = /etc/rsync.passwd
log file = /var/log/rsyncd.log
#####################################
[backup]
path = /backup=
```

`install-rsync.yml`文件内容

```yaml
- hosts: rsync-server
  tasks:
    - name: Install Rsync Server
      yum:
        name: rsync
        state: present

    - name: Configure Rsync Server
      copy:
        src: ./rsyncd.conf.j2
        dest: /etc/rsyncd.conf
        owner: root
        group: root
        mode: 0644
      notify: Restart Rsync Server

    - name: Init Group
      group:
        name: ansible_www
        gid: 8888

    - name: Init User
      user:
        name: ansible_www
        uid: 8888
        shell: /sbin/nologin
        create_home: no

    - name: Init Create Directory
      file:
        path: /backup
        owner: ansible_www
        group: ansible_www
        mode: 0755
        recurse: yes

    - name: Init Rsync Server Virtual User Passwd file
      copy:
        content: "rsync_backup:123456"
        dest: /etc/rsync.passwd
        owner: root
        group: root
        mode: 0600
      notify: Restart Rsync Server

    - name: Started Rsync Server
      systemd:
        name: rsyncd
        state: started
        enabled: yes

  handlers:
    - name: Restart Rsync Server
      systemd:
        name: rsyncd
        state: restarted
```

测试是否成功

```bash
rsync -avz exports.j2 rsync_backup@192.168.0.12::backup
```

#### 部署`Redis`

```yaml
- hosts: redis
  tasks:
    - name: Install Redis Server
      yum:
        name: redis
        state: present

    - name: Configure Redis Server
      copy:
        src: ./files/redis.conf.j2
        dest: /etc/redis.conf
        owner: redis
        group: root
        mode: 0640
      notify: Restart Redis Server

    - name: Systemd Redis Server
      systemd:
        name: redis
        state: started
        enabled: yes

  handlers:
    - name: Restart Redis Server
      systemd:
        name: Redis
        state: restarted
```

#### 部署`Nginx+PHP`

`PHP`源

```bash
ansible web -m shell -a "rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm"
ansible web -m shell -a "rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm"
ansible web -m shell -a "yum makecache"
```

`nginx+php`

```yaml
- hosts: webservers
  tasks:
    - name: Installed Nginx Server
      yum:
        name: nginx
        state: present

    - name: Installed PHP Server
      yum:
        name: "{{ pack }}"
      vars:
        pack:
          - php71w-fpm
          - php71w-gd
          - php71w-mbstring
          - php71w-mcrypt
          - php71w-mysqlnd
          - php71w-opcache
          - php71w-pdo
          - php71w-pear
          - php71w-pecl-igbinary
          - php71w-pecl-memcached
          - mod_php71w
          - php71w-pecl-mongodb
          - php71w-pecl-redis
          - php71w-cli
          - php71w-process
          - php71w-common
          - php71w-xml
          - php71w-devel
          - php71w-embedded

    # nginx
    - name: Configure Nginx nginx.conf
      copy:
        src: files/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: 0644
      notify: Restart Nginx Server

    - name: Create Group www
      group:
        name: www
        gid: 666

    - name: Create User www
      user:
        name: www
        uid: 666
        group: www
        create_home: no
        shell: /sbin/nologin

    - name: Started Nginx Server
      systemd:
        name: nginx
        state: started
        enabled: yes

    # php
    - name: Confgiure PHP Server php.ini
      copy:
        src: ./files/php.ini.j2
        dest: /etc/php.ini
        owner: root
        group: root
        mode: 0644
      notify: Restart PHP Server

    - name: Confgiure PHP Server php-fpm.d/www.conf
      copy:
        src: ./files/php-fpm.www.conf.j2
        dest: /etc/php-fpm.d/www.conf
        owner: root
        group: root
        mode: 0644
      notify: Restart PHP Server

    - name: Started PHP Server
      systemd:
        name: php-fpm
        state: started
        enabled: yes

    # code
    #
    - name: Copy Nginx Virtual Site
      copy:
        src: ./files/ansible.oldxu.net.conf.j2
        dest: /etc/nginx/conf.d/ansible.oldxu.net.conf
      notify: Restart Nginx Server

    - name: Create Ansible Directory
      file:
        path: /ansible
        owner: www
        group: www
        mode: 0755
        recurse: yes

    - name: Unarchive PHP Code
      unarchive:
        src: files/phpMyAdmin-5.1.1-all-languages.zip
        dest: /ansible/
        creates: /ansible/phpMyAdmin-5.1.1-all-languages/config.inc.php

    - name: Create Link
      file:
        src: /ansible/phpMyAdmin-5.1.1-all-languages/
        dest: /ansible/phpmyadmin
        state: link

    - name: Change phpmyadmin Configure
      copy:
        src: ./files/config.inc.php.j2
        dest: /ansible/phpmyadmin/config.inc.php

  handlers:
    - name: Restart Nginx Server
      systemd:
        name: nginx
        state: restarted

    - name: Restart PHP Server
      systemd:
        name: php-fpm
        state: restarted
```
