---
title: 常用软件部署安装
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Linux
categories:
  - Linux
url: post/linux-software.html
toc: true
---

### `Ubuntu`下载源设置

<!-- more -->

```shell
# ubuntu 20.04
# 备份
mv /etc/apt/sources.list{,.bak}

# 源设置
cat >/etc/apt/sources.list<<EOF
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
EOF
```

### `Centos7`的`yum`源设置

```shell
# 备份
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/* /etc/yum.repos.d/bak

# 下载源
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# 缓存
yum makecache
```

### 常用软件下载

```shell
yum install wget net-tools vim git curl dstat htop -y
```

### `Docker`安装

```shell
# 安装最新docker
yum install yum-utils device-mapper-persistent-data lvm2 -y
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y

# 安装指定版本
yum list docker-ce --showduplicates | sort -r
yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io -y

# 启动
systemctl start docker
systemctl enable docker

# 查看信息
docker info

# 镜像加速、更改Docker根目录
mkdir -p /data/docker   # 挂载到新的硬盘上
cat >/etc/docker/daemon.json<<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {"max-size": "100m"},
  "registry-mirrors" : ["https://ot2k4d59.mirror.aliyuncs.com/"],
  "graph": "/data/docker"
}
EOF

# 重启docker
systemctl daemon-reload
systemctl restart docker
docker info

# 测试
docker run hello-world

# 卸载
yum remove docer-ce
rm -rf /var/lib/docker


# 修改内核参数，解决以下报错
# WARNING: bridge-nf-call-iptables is disabled
# WARNING: bridge-nf-call-ip6tables is disabled
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf

sysctl --system
sysctl -p

#内核调整出现错误
sysctl -p
net.ipv4.ip_forward = 1
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-ip6tables: No such file or directory
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables: No such file or directory

#解决办法
modprobe br_netfilter
```

### `docker-compose`安装

```shell
# 国内下载http://get.daocloud.io/
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

### `Python3.8`相关配置

#### 安装`python3.8`

```shell
# 下载依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel python-devel mariadb-devel gdbm-devel python-setuptools python-devel -y

# 下载源码安装
wget https://registry.npmmirror.com/-/binary/python/3.8.6/Python-3.8.6.tgz
tar xf Python-3.8.6.tgz
cd Python-3.8.6/
./configure --prefix=/usr/local/python38
make && make install
ln -s /usr/local/python38/bin/python3 /usr/bin/python3.8
ln -s /usr/local/python38/bin/pip3 /usr/bin/pip3.8
```

#### `pip`源设置

```shell
cat > ~/.pip/pip.conf<<EOF
[global]
timeout = 6000
index-url = https://pypi.douban.com/simple/
[install]
use-mirrors = true
mirrors = https://pypi.douban.com/simple/
trusted-host = pypi.douban.com
EOF
```

#### 虚拟环境安装

```shell
# python虚拟环境安装
pip3.8 install virtualenv
pip3.8 install virtualenvwrapper

# 如果有错误执行以下可能解决
python3.8 -m pip install --upgrade pip
python3.8 -m pip install --upgrade setuptools
pip3.8 install pbr

# 建立软连接
ln -s /usr/local/python38/bin/virtualenv /usr/bin/virtualenv


# 环境变量设置  >> file << 追加的模式
cat >> ~/.bashrc <<EOF
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3.8
source /usr/local/python38/bin/virtualenvwrapper.sh
EOF

source ~/.bashrc
```

#### 虚拟环境创建

```shell
# 使用virtualenvwrapper 创建
mkvirtualenv -p python3.8 test-env
lsvirtualenv               # 查看所有的虚拟环境
cdvirtualenv               # 进入虚拟环境的目录，需要进入到虚拟环境
workon test-env            # 进入test-env虚拟环境中
deactivate                 # 退出当前虚拟环境
rmvirtualenv               # 删除

# 不使用virtualenvwrapper创建和使用虚拟环境
mkdir virtual
cd virtual
ln -s /usr/local/python38/bin/virtualenv /usr/bin/virtualenv
virtualenv  test-env

# 进入到虚拟环境
source /root/virtual/test-env/bin/activate

# 退出虚拟环境
deactivate
```

#### `uwsgi`安装

```shell
pip3.8 install uwsgi
ln -s /usr/local/python38/bin/uwsgi /usr/bin/uwsgi
```

### `Golang`安装

```shell
# 下载软件
wget https://studygolang.com/dl/golang/go1.16.14.linux-amd64.tar.gz

# 解压
tar xf go1.16.14.linux-amd64.tar.gz
mv go /usr/local/
mkdir /code/goproject -p

# 环境变量
cat >> ~/.bashrc <<EOF
export GOROOT=/usr/local/go
export GOPATH=/code/goproject
export PATH=$PATH:$GOROOT/bin/:$GOPATH/bin
EOF

source ~/.bashrc

# 设置代理和开启 module
go env -w GOPROXY="https://goproxy.cn,direct"
go env -w GO111MODULE="on"
```

### `Node`安装

```shell
# 下载软件
wget https://npmmirror.com/mirrors/node/v14.19.1/node-v14.19.1-linux-x64.tar.xz

# 解压移动
tar xf node-v14.19.1-linux-x64.tar.xz
mv node-v14.19.1-linux-x64 /usr/local/node

# 环境变量设置
cat 'export PATH=$PATH:/usr/local/node/bin/' >> ~/.bashrc
source ~/.bashrc

# npm 源设置
npm config set registry https://registry.npmmirror.com
npm config get registry
npm install -g cnpm --registry=https://registry.npmmirror.com
```

### `MySQL`安装

#### 二进制方式安装

```shell
# 下载地址
# https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/

# 关闭防火墙及selinux
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i  s#enforcing#disabled#g /etc/selinux/config

# 卸载mariadb相关软件包
yum remove mariadb* -y

# 下载依赖包
yum install  libaio-devel -y

# 下载tar包
wget https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/mysql-5.7.34-el7-x86_64.tar.gz

# 解压
tar xf mysql-5.7.31-el7-x86_64.tar.gz
mv mysql-5.7.31-el7-x86_64 /usr/local/mysql

# 创建mysql用户
useradd -s /sbin/nologin mysql

#添加环境变量
echo 'export PATH=/usr/local/mysql/bin:$PATH' >>/etc/profile
source /etc/profile

# mysql软件目录授权mysql用户
chown -R mysql:mysql /usr/local/mysql

# 查看版本
mysql -V


# 数据库数据目录应该单独用一块盘挂载
# mkfs.xfs /dev/sdb
# vim /etc/fstab
# mount -a

# 这边测试就直接创建目录了
mkdir /data/mysql/data -p
chown -R mysql:mysql /data

# 初始化
/usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data

# 参数说明
--initialize-insecure  # 使用空密码,后续手动添加密码即可
--initialize           # 会产生一个临时密码
--user                 # 指定用户
--basedir              # 指定软件路径
--datadir              # 指定数据路径


# 最简配置文件
cat >/etc/my.cnf <<EOF
[mysqld]
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
server_id=6
port=3306
[mysql]
socket=/tmp/mysql.sock
EOF

# 启动配置
# 方法一: mysql的脚本启动文件
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
service mysqld start  # 该方法启动命令
service mysqld stop  # 该方法停止命令

# 方法二: systemd启动配置
cat >/etc/systemd/system/mysqld.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
EOF

# 使用systemd启动和检查
systemctl start mysqld.service
systemctl status mysqld.service
netstat -lntup |grep 3306

# 数据库启动错误分析
 without updating PID 类似错误
查看日志：
	/data/mysql/data/主机名.err
	[ERROR] 上下文
可能情况：
	/etc/my.cnf 路径不对等
	/tmp/mysql.sock文件修改过 或 删除过
	数据目录权限不是mysql
	参数改错了

# 调试命令
mysqld --default-file=/etc/my.cnf
```

#### `yum`方式安装

```shell
# 下载 mysql5.7
wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

# 安装 mysql5.7
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum install mysql-community-server --nogpgcheck

# 启动 mysql5.7 并查看启动状态
systemctl start mysqld.service  # 启动mysql服务
systemctl enable mysqld.service # 开机自启动
systemctl status mysqld.service # 查看服务

# 查看默认密码并登录
grep "password" /var/log/mysqld.log
mysql -uroot -p

# 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Mysql12345?';
```

### `Redis`安装

```shell
# 下载redis-5.0.5
wget http://download.redis.io/releases/redis-5.0.5.tar.gz

# 解压安装包
tar -xf redis-5.0.5.tar.gz

# 进入目标文件
cd redis-5.0.5

# 编译环境  src路径下就有可执行文件 redis-server redis-cli 等
make -j `cat /proc/cpuinfo |grep processor |wc -l`

# 复制环境到指定路径完成安装
cp -rp ~/redis-5.0.5 /usr/local/redis

# 配置redis可以后台启动：修改下方内容
vim /usr/local/redis/redis.conf
daemonize yes

# 建立软连接
ln -s /usr/local/redis/src/redis-server /usr/bin/redis-server
ln -s /usr/local/redis/src/redis-cli /usr/bin/redis-cli

# 后台运行redis
cd /usr/local/redis
redis-server ./redis.conf

# 查看服务是否启动
ps aux |grep redis

# 测试redis环境
redis-cli

# 关闭redis服务
pkill -f redis -9
```

### `Nginx`安装

```shell
yum install gcc zlib zlib-devel pcre-devel openssl openssl-devel -y

wget http://nginx.org/download/nginx-1.20.1.tar.gz

tar xf nginx-1.20.1.tar.gz

cd nginx-1.20.1/

./configure --prefix=/usr/local/nginx  \
--with-http_stub_status_module  \
--with-http_ssl_module --with-pcre

make -j `cat /proc/cpuinfo |grep processor |wc -l` && make install

echo 'export PATH=/usr/local/nginx/sbin:$PATH' >> /etc/profile
source /etc/profile
```
