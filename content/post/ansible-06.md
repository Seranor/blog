---
title: jinja2和Role
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Ansible
categories:
  - Ansible
url: post/ansible-05.html
toc: true
---

## template 模板

> 模板是一个文本文件，可以做为生成文件的模版，并且模板文件中还可嵌套 jinja 语法。

<!-- more -->

### jinja2 语言

> 官方网站：
>
> http://jinja.pocoo.org/
>
> https://jinja.palletsprojects.com/en/2.11.x/

![img](https://jinja.palletsprojects.com/en/3.0.x/_images/jinja-logo.png)

#### 数据类型

**jinja2 语言支持多种数据类型和操作:**

- 字符串：使用单引号或双引号,
- 数字：整数，浮点数
- 列表：[item1, item2, ...]
- 元组：(item1, item2, ...)
- 字典：{key1:value1, key2:value2, ...}
- 布尔型：true/false
- 算术运算：+, -, \*, /, //, %, \*\*
- 比较操作：==, !=, >, >=, <, <=
- 逻辑运算：and，or，not
- 流表达式：For，If，When

### template

template 功能：可以根据和参考模块文件，动态生成相类似的配置文件，template 文件必须存放于 templates 目录下，且命名为 .j2 结尾，yaml/yml 文件需和 templates 目录平级，目录结构如下示例：

```bash
 ./
├── temnginx.yml
└── templates
    └── nginx.conf.j2Copy to clipboardErrorCopied
```

范例：利用 template 同步 nginx 配置文件

```bash
#准备templates/nginx.conf.j2文件
[root@ansible ~]#vim temnginx.yml
---
- hosts: web
  remote_user: root
  tasks:
    - name: template config to remote hosts
     template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
 [root@ansible ~]#ansible-playbook temnginx.ymlCopy to clipboardErrorCopied
```

#### template 变更替换

```bash
#修改文件nginx.conf.j2
[root@ansible ~]#mkdir templates
[root@ansible ~]#vim templates/nginx.conf.j2
......
worker_processes {{ ansible_processor_vcpus }};
......
[root@ansible ~]#vim temnginx2.yml
---
- hosts: web
  remote_user: root

  tasks:
    - name: install nginx
      yum: name=nginx
    - name: template config to remote hosts
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
    - name: start service
      service: name=nginx state=started enabled=yes

[root@ansible ~]#ansible-playbook temnginx2.ymlCopy to clipboardErrorCopied
```

#### 常用的系统参数

```bash
ansible_all_ipv4_addresses：仅显示ipv4的信息
ansible_devices：仅显示磁盘设备信息
ansible_distribution：显示是什么系统，例：centos,suse等
ansible_distribution_version：仅显示系统版本
ansible_distribution_major_version：显示系统版本号（7）
ansible_machine：显示系统类型，例：32位，还是64位
ansible_eth0：仅显示eth0的信息
ansible_hostname：仅显示主机名
ansible_kernel：仅显示内核版本
ansible_lvm：显示lvm相关信息
ansible_memtotal_mb：显示系统总内存
ansible_memfree_mb：显示可用系统内存
ansible_memory_mb：详细显示内存情况
ansible_swaptotal_mb：显示总的swap内存
ansible_swapfree_mb：显示swap内存的可用内存
ansible_mounts：显示系统磁盘挂载情况
ansible_processor：显示cpu个数(具体显示每个cpu的型号)
ansible_processor_vcpus：显示cpu个数(只显示总的个数)
ansible_python_version：显示python版本
```

#### template 算术运算

```bash
[root@ansible ansible]#vim templates/nginx.conf.j2
worker_processes {{ ansible_processor_vcpus**3 }};
[root@ansible ansible]#cat templnginx.yml
---
- hosts: websrvs
  remote_user: root
  tasks:
    - name: install nginx
      yum: name=nginx
    - name: template config to remote hosts
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      notify: restart nginx
    - name: start service
      service: name=nginx state=started enabled=yes
 handlers:
    - name: restart nginx
      service: name=nginx state=restarted
[root@ansible ~]#-playbook templnginx.yml --limit 10.0.0.8Copy to clipboardErrorCopied
```

#### template 中使用流程控制 for 和 if

> template 中也可以使用流程控制 for 循环和 if 条件判断，实现动态生成文件功能

```bash
#temlnginx2.yml
---
- hosts: websrvs
 remote_user: root
 vars:
   nginx_vhosts:
     - 81
     - 82
     - 83
 tasks:
   - name: template config
     template: src=nginx.conf2.j2 dest=/data/nginx.conf
#templates/nginx.conf2.j2
{% for vhost in nginx_vhosts %}
server {
   listen {{ vhost }}
}
{% endfor %}
ansible-playbook -C templnginx2.yml --limit 192.168.15.8
#生成的结果：
server {
   listen 81
}
server {
   listen 82
}
server {
   listen 83
}




#templnginx4.yml
- hosts: websrvs
 remote_user: root
 vars:
   nginx_vhosts:
     - listen: 8080
       server_name: "web1.oldboy.com"
       root: "/var/www/nginx/web1/"
     - listen: 8081
       server_name: "web2.oldboy.com"
       root: "/var/www/nginx/web2/"
     - {listen: 8082, server_name: "web3.oldboy.com", root:
"/var/www/nginx/web3/"}
 tasks:
   - name: template config
     template: src=nginx.conf4.j2 dest=/data/nginx4.conf


# templates/nginx.conf4.j2
{% for vhost in nginx_vhosts %}
server {
   listen {{ vhost.listen }}
   server_name {{ vhost.server_name }}
   root {{ vhost.root }}
}{% endfor %}

[root@ansible ~]#ansible-playbook templnginx4.yml --limit 10.0.0.8
#生成结果：
server {
   listen 8080
   server_name web1.oldboy.com
   root /var/www/nginx/web1/
}
server {
   listen 8081
   server_name web2.oldboy.com
   root /var/www/nginx/web2/
}
server {
   listen 8082
   server_name web3.oldboy.com
   root /var/www/nginx/web3/
}Copy to clipboardErrorCopied
```

#### playbook 使用 when

when 语句，可以实现条件测试。如果需要根据变量、facts 或此前任务的执行结果来做为某 task 执行与否的前提时要用到条件测试,通过在 task 后添加 when 子句即可使用条件测试，jinja2 的语法格式。

```bash
tasks:
  - name: "shut down CentOS 6 and Debian 7 systems"
   command: /sbin/shutdown -t now
   when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")Copy to clipboardErrorCopied
```

#### playbook 使用迭代 with_items(loop)

迭代：当有需要重复性执行的任务时，可以使用迭代机制对迭代项的引用，固定内置变量名为"item"，要在 task 中使用 with_items 给定要迭代的元素列表

注意: ansible2.5 版本后,可以用 loop 代替 with_items

```bash
---
- hosts: websrvs
 remote_user: root

 tasks:
    - name: add several users
     user: name={{ item }} state=present groups=wheel
     with_items:
        - testuser1
        - testuser2
        - testuser3

#上面语句的功能等同于下面的语句
    - name: add several users
      user: name=testuser1 state=present groups=wheel
    - name: add several users
      user: name=testuser2 state=present groups=wheel
    - name: add several users
      user: name=testuser3 state=present groups=wheel



---
#remove mariadb server
- hosts: 172.16.1.7
  remote_user: root
  tasks:
    - name: stop service
      shell: /etc/init.d/mysqld stop
    - name: delete files and dir
      file: path={{item}} state=absent
      with_items:
        - /usr/local/mysql
        - /usr/local/mariadb-10.2.27-linux-x86_64
        - /etc/init.d/mysqld
        - /etc/profile.d/mysql.sh
        - /etc/my.cnf
        - /data/mysql
    - name: delete user
      user: name=mysql state=absent remove=yesCopy to clipboardErrorCopied
```

**迭代嵌套子变量：**在迭代中，还可以嵌套子变量，关联多个变量在一起使用

```bash
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: add some groups
      group: name={{ item }} state=present
      with_items:
        - nginx
        - mysql
        - apache
    - name: add some users
      user: name={{ item.name }} group={{ item.group }} state=present
      with_items:
        - { name: 'nginx', group: 'nginx' }
        - { name: 'mysql', group: 'mysql' }
        - { name: 'apache', group: 'apache' }Copy to clipboardErrorCopied
```

### 管理节点过多导致的超时问题解决方法

> 默认情况下，Ansible 将尝试并行管理 playbook 中所有的机器。对于滚动更新用例，可以使用 serial 关键字定义 Ansible 一次应管理多少主机，还可以将 serial 关键字指定为百分比，表示每次并行执行的主机数占总数的比例

```bash
#vim test_serial.yml
---
- hosts: all
  serial: 2  #每次只同时处理2个主机,将所有task执行完成后,再选下2个主机再执行所有task,直至所有主机
  gather_facts: False
  tasks:
    - name: task one
  comand: hostname
    - name: task two
      command: hostname


# 案例2：
- name: test serail
  hosts: all
  serial: "20%"   #每次只同时处理20%的主机
```

# [Role](http://192.168.13.73:3000/#/file/roles角色?id=role)

角色是 ansible 自 1.2 版本引入的新特性，用于层次性、结构化地组织 playbook。roles 能够根据层次型结构自动装载变量文件、tasks 以及 handlers 等。要使用 roles 只需要在 playbook 中使用 include 指令即可。简单来讲，roles 就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷地 include 它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中

运维复杂的场景：建议使用 roles，代码复用度高

roles：多个角色的集合目录， 可以将多个的 role，分别放至 roles 目录下的独立子目录中,如下示例

```bash
roles/
 mysql/
 nginx/
 tomcat/
 redis/Copy to clipboardErrorCopied
```

默认 roles 存放路径

```bash
/root/.ansible/roles
/usr/share/ansible/roles
/etc/ansible/rolesCopy to clipboardErrorCopied
```

## [Ansible Roles 目录编排](http://192.168.13.73:3000/#/file/roles角色?id=ansible-roles目录编排)

```bash
├── nginx -------------role1名称
│   ├── defaults  ---------必须存在的目录，存放默认的变量，模板文件中的变量就是引用自这里。defaults中的变量优先级最低，通常我们可以临时指定变量来进行覆盖
│   │   └── main.yml
│   ├── files -------------ansible中unarchive、copy等模块会自动来这里找文件，从而我们不必写绝对路径，只需写文件名
│   │   ├── mysql.tar.gz
│   │   └── nginx.tar.gz
│   ├── handlers -----------存放tasks中的notify指定的内容
│   │   └── main.yml
│   ├── meta
│   ├── tasks --------------存放playbook的目录，其中main.yml是主入口文件，在main.yml中导入其他yml文件，要采用import_tasks关键字，include要弃用了
│   │   ├── install.yml
│   │   └── main.yml -------主入口文件
│   ├── templates ----------存放模板文件。template模块会将模板文件中的变量替换为实际值，然后覆盖到客户机指定路径上
│   │   └── nginx.conf.j2
│   └── varsCopy to clipboardErrorCopied
```

## [Roles 各目录作用](http://192.168.13.73:3000/#/file/roles角色?id=roles各目录作用)

- files/ ：存放由 copy 或 script 模块等调用的文件

- templates/：template 模块查找所需要模板文件的目录

- tasks/：定义 task,role 的基本元素，至少应该包含一个名为 main.yml 的文件；其它的文件需要在此文件中通过 include 进行包含

- handlers/：至少应该包含一个名为 main.yml 的文件；此目录下的其它的文件需要在此文件中通过

- include 进行包含

- vars/：定义变量，至少应该包含一个名为 main.yml 的文件；此目录下的其它的变量文件需要在此文件中通过 include 进行包含

- meta/：定义当前角色的特殊设定及其依赖关系,至少应该包含一个名为 main.yml 的文件，其它文件需在此文件中通过 include 进行包含

- default/：设定默认变量时使用此目录中的 main.yml 文件，比 vars 的优先级低

  ## [创建 role](http://192.168.13.73:3000/#/file/roles角色?id=创建role)

> 创建 role 的步骤

```bash
1 创建以roles命名的目录
2 在roles目录中分别创建以各角色名称命名的目录，如mysql等
3 在每个角色命名的目录中分别创建files、handlers、tasks、templates和vars等目录；用不到的目录可以创建为空目录，也可以不创建
4 在每个角色相关的子目录中创建相应的文件,如 tasks/main.yml,templates/nginx.conf.j2
5 在playbook文件中，调用需要的角色

[root@m01 package]# mkdir -p /root/package/roles/nginx/{files,handlers,tasks,templates,vars,meta}
[root@m01 package]# tree
.
└── roles
    └── nginx
        ├── files
        ├── handlers
        ├── meta
        ├── tasks
        ├── templates
        └── vars
Copy to clipboardErrorCopied
```

### [针对大型项目使用 Roles 进行编排](http://192.168.13.73:3000/#/file/roles角色?id=针对大型项目使用roles进行编排)

```bash
# 范例
nginx-role.yml
roles/
└── nginx
     ├── files
     │   └── main.yml
     ├── tasks
     │   ├── groupadd.yml
     │   ├── install.yml
     │   ├── main.yml
     │   ├── restart.yml
     │   └── useradd.yml
     └── vars
         └── main.ymlCopy to clipboardErrorCopied
```

## [playbook 调用角色](http://192.168.13.73:3000/#/file/roles角色?id=playbook调用角色)

- 调用角色方法 1

```bash
---
- hosts: web
  remote_user: root
  roles:
    - mysql
    - memcached
    - nginx  Copy to clipboardErrorCopied
```

- 调用角色方法 2

```bash
---
- hosts: all
  remote_user: root
  roles:
    - mysql
    - { role: nginx, username: nginx }Copy to clipboardErrorCopied
```

- 调用角色方法 3

```bash
---
- hosts: all
  remote_user: root
  roles:
    - { role: nginx, username: nginx, when: ansible_distribution_major_version == '7' }Copy to clipboardErrorCopied
```

- 调用角色方法 4

```bash
---
- hosts: appsrvs
  remote_user: root
  roles:
    - { role: nginx ,tags: [ 'nginx', 'web' ] ,when: ansible_distribution_major_version == "6" }
    - { role: httpd ,tags: [ 'httpd', 'web' ] }
    - { role: mysql ,tags: [ 'mysql', 'db' ] }
    - { role: mariadb ,tags: [ 'mariadb', 'db' ] }Copy to clipboardErrorCopied
```

## [案例](http://192.168.13.73:3000/#/file/roles角色?id=案例)

### [httpd 角色](http://192.168.13.73:3000/#/file/roles角色?id=httpd角色)

```bash
#创建角色相关的目录
[root@ansible ~]#mkdir -pv /data/ansible/roles/httpd/{tasks,handlers,files}

#创建角色相关的文件
[root@ansible ~]#cd /data/ansible/roles/httpd/

#main.yml 是task的入口文件
[root@ansible ~]#vim tasks/main.yml
- include: group.yml
- include: user.yml
- include: install.yml
- include: config.yml
- include: index.yml
- include: service.yml

[root@ansible ~]#vim tasks/group.yml
- name: create apache group
  group: name=apache system=yes gid=80

[root@ansible ~]#vim tasks/user.yml
- name: create apache user
  user: name=apache system=yes shell=/sbin/nologin home=/var/www/ uid=80 group=apache

[root@ansible ~]#vim tasks/install.yml
- name: install httpd package
  yum: name=httpd
[root@ansible ~]#vim tasks/config.yml
- name: config file
  copy: src=httpd.conf dest=/etc/httpd/conf/ backup=yes
  notify: restart
[root@ansible ~]# tasks/index.yml
- name: index.html
  copy: src=index.html dest=/var/www/html/
[root@ansible ~]#vim tasks/service.yml
- name: start service
  service: name=httpd state=started enabled=yes
[root@ansible ~]#vim handlers/main.yml
- name: restart
  service: name=httpd state=restarted

#在files目录下准备两个文件
[root@ansible ~]#ls files/
httpd.conf index.html
[root@ansible ~]#tree /data/ansible/roles/httpd/
/data/ansible/roles/httpd/
├── files
│   ├── httpd.conf
│   └── index.html
├── handlers
│   └── main.yml
└── tasks
   ├── config.yml
   ├── group.yml
   ├── index.yml
   ├── install.yml
   ├── main.yml
   ├── service.yml
   └── user.yml
3 directories, 10 files
#在playbook中调用角色
[root@ansible ~]#vim /data/ansible/role_httpd.yml
---
# httpd role
- hosts: websrvs
  remote_user: root
  roles:
    - httpd

#运行playbook
[root@ansible ~]#ansible-playbook /data/ansible/role_httpd.ymlCopy to clipboardErrorCopied
```

### [NGINX 角色](http://192.168.13.73:3000/#/file/roles角色?id=nginx角色)

```bash
[root@ansible ~]#mkdir -pv /data/ansible/roles/nginx/{tasks,handlers,templates,vars}

#创建task文件
[root@ansible ~]#cd /data/ansible/roles/nginx/
[root@ansible nginx]#vim tasks/main.yml
- include: install.yml
- include: config.yml
- include: index.yml
- include: service.yml
[root@ansible nginx]#vim tasks/install.yml
- name: install
  yum: name=nginx

[root@ansible nginx]#vim tasks/config.yml
- name: config file for centos7
  template: src=nginx7.conf.j2 dest=/etc/nginx/nginx.conf
  when: ansible_distribution_major_version=="7"
  notify: restart
- name: config file for centos8
  template: src=nginx8.conf.j2 dest=/etc/nginx/nginx.conf
  when: ansible_distribution_major_version=="8"
  notify: restart

#跨角色调用文件
[root@ansible nginx]#vim tasks/index.yml
- name: index.html
  copy: src=roles/httpd/files/index.html dest=/usr/share/nginx/html/
[root@ansible nginx]#vim tasks/service.yml
- name: start service
  service: name=nginx state=started enabled=yes
#创建handler文件
[root@ansible nginx]#cat handlers/main.yml
- name: restart
  service: name=nginx state=restarted
#创建两个template文件
[root@ansible nginx]#cat templates/nginx7.conf.j2
...省略...
user {{user}};
worker_processes {{ansible_processor_vcpus+3}};   #修改此行
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
...省略...
[root@ansible nginx]#cat templates/nginx8.conf.j2
...省略...
user nginx;
worker_processes {{ansible_processor_vcpus**3}};  #修改此行
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
...省略...
#创建变量文件
[root@ansible nginx]#vim vars/main.yml
user: daemon
#目录结构如下
[root@ansible ~]#tree /data/ansible/roles/nginx/
/data/ansible/roles/nginx/
├── handlers
│   └── main.yml
├── tasks
│   ├── config.yml
│   ├── file.yml
│   ├── install.yml
│   ├── main.yml
│   └── service.yml
├── templates
│   ├── nginx7.conf.j2
│   └── nginx8.conf.j2
└── vars
   └── main.yml
4 directories, 9 files
#在playbook中调用角色
[root@ansible ~]#vim /data/ansible/role_nginx.yml
---
#nginx role
- hosts: web
 roles:
    - role: nginx

#运行playbook
[root@ansible ~]#ansible-playbook /data/ansible/role_nginx.yml
```

## 创建 Nginx 的 roles

```bash
# 创建初始文件
[root@m01 roles]# ansible-galaxy init nginx
- Role nginx was created successfully

# 查看目录层级结构
baim0/
└── roles
    └── nginx
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   └── main.yml
        ├── templates
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml

#
```
