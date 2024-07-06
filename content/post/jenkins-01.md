---
title: Jenkins安装
lastmod: 2021-11-21T16:43:23+08:00
date: 2021-11-21T11:52:03+08:00
tags:
  - Jenkins
categories:
  - Jenkins
url: post/jenkins.html
toc: true
---

## Jenkins

Jenkins 是一个开源提供友好操作界面的持续集成的工 具，是由 JAVA 开发而成

Jenkins 是一个调度平台，本身不处理任何事情，调用 插件来完成所有的工作

<!-- more -->

### 安装

```bash
# 官方文档: https://www.jenkins.io/zh/doc/book/installing/
# 安装方式有多种

# 关闭防火墙和selinux
systemctl stop firewalld
systemctl disable firewalld

setenforce 0
sed -i  s#enforcing#disabled#g /etc/selinux/config

# 设置语言环境
localectl set-locale LANG=en_US.UTF-8
localectl status

# 安装jdk11
yum install java-11-openjdk-devel -y
java -version

# 安装
yum localinstall -y https://mirror.tuna.tsinghua.edu.cn/jenkins/redhat/jenkins-2.303-1.1.noarch.rpm

# 查看目录结构
rpm -ql jenkins

# 配置文件
vim /etc/sysconfig/jenkins
JENKINS_USER="root" # 运行Jenkins的用户身份，避免后期权限不足的情况
JENKINS_PORT="80"  # 如果jenkins监听在80端口，运行身份必须为root

# 启动
systemctl start jenkins
systemctl enable jenkins
```

### 配置

访问的时候需要解锁

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
# 这个密码也是默认的admin密码
```

![image-20220109104122460](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109104122460.png)

跳过插件安装

![image-20220109104325063](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109104325063.png)

直接进入 Jenkins

![image-20220109104343502](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109104343502.png)

先配置管理密码

![image-20220109104456161](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109104456161.png)

![image-20220109104541671](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109104541671.png)

### 插件管理

Jenkins 系统管理中的插件管理非常重要，因为的工作全部是由插件来完成，但插件默认从国外下载，速度会很慢，所以需要在安装插件前将下载地址改为国内的下载地址

1.修改 jenkins "下载插件" 地址为国内源

```bash
# jenkins检测地址
sed -i 's#http://www.google.com/#https://www.baidu.com/#g' /var/lib/jenkins/updates/default.json

# jenkins插件下载地址
sed -i 's#updates.jenkins.io/download#mirror.tuna.tsinghua.edu.cn/jenkins#g' /var/lib/jenkins/updates/default.json


https://mirror.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

![image-20220109105629652](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109105629652.png)

```bash
# 下面地址填入该地址
https://mirror.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

![image-20220109105730451](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109105730451.png)

插件可以选择上传 hpi 文件安装

![image-20220109111909979](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109111909979.png)

安装中文插件

![image-20220109105902878](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109105902878.png)

离线安装插件

将之前 jenkins 服务器的插件保存下来，然后导入到服务器中，(离线安装)，最后重启 Jenkins

```bash
wget plugins.tar.gz
tar xf jenkins_plugin.tar.gz -C /var/lib/jenkins/plugins/
chown -R jenkins.jenkins /var/lib/jenkins/plugins/
systemctl restart jenkins
```

### 升级

这个版本有告警漏洞提示

![image-20220109110751446](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220109110751446.png)

根据提示下载对应的 war 包

```bash
# 下载高版本war包
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/2.314/jenkins.war

# 停止Jenkins
systemctl stop jenkins


# 备份war包
mv /usr/lib/jenkins/jenkins.war /usr/lib/jenkins/jenkins.war.bak

# 将新版本的war包替换过来
mv jenkins.war /usr/lib/jenkins/
```
