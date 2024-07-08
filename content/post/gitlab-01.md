---
title: GitLab安装使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - GitLab
categories:
  - GitLab
url: post/gitlab.html
toc: true
---

## `gitlab-ce`

### 安装

<!-- more -->

```bash
# 关闭防火墙和selinux
systemctl stop firewalld
systemctl disable firewalld

setenforce 0
sed -i  s#enforcing#disabled#g /etc/selinux/config

# 下载依赖
yum install -y curl wget postfix openssh-server

# 下载gitlab包
# 下载地址：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-12.9.9-ce.0.el7.x86_64.rpm

# 安装
yum localinstall -y gitlab-ce-12.9.9-ce.0.el7.x86_64.rpm

# gitlab相关配置
vim /etc/gitlab/gitlab.rb

# 域名配置
external_url 'http://gitlab.example.com'
# 如果要配置https，将url改成https，然后将证书拷贝到 /etc/gitlab/ssl/域名.{key,crt}

# 配置发送的邮箱
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = '2564334707@qq.com'
gitlab_rails['gitlab_email_display_name'] = 'gitlab-admin'

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "2564334707@qq.com"  # 发件人邮箱账户
gitlab_rails['smtp_password'] = ""  # 授权码
gitlab_rails['smtp_domain'] = "qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

# 关闭监控组件(按需求关闭或启用)
prometheus['enable'] = false
prometheus['monitor_kubernetes'] = false
alertmanager['enable'] = false
node_exporter['enable'] = false
redis_exporter['enable'] = false
postgres_exporter['enable'] = false
gitlab_exporter['enable'] = false
prometheus_monitoring['enable'] = false
grafana['enable'] = false

#  初始化gitlab组件
gitlab-ctl reconfigure  # 初始化
gitlab-ctl status       # 查看状态
gitlab-ctl stop         # 停止
gitlab-ctl start        # 启动

# 验证邮箱组件
gitlab-rails console Notify.test_email('接收者地址','标题','内容').deliver_now

# 网页访问，第一次进入需要修改密码

# 汉化包地址: https://gitlab.com/xhang/gitlab
wget https://gitlab.com/xhang/gitlab/-/archive/12-3-stable-zh/gitlab-12-3-stable-zh.tar.gz
tar xf gitlab-12-3-stable-zh.tar.gz

# 停止
gitlab-ctl stop
\cp -r gitlab-12-3-stable-zh/* /opt/gitlab/embedded/service/gitlab-rails/
gitlab-ctl reconfigure
gitlab-ctl restart

# 设置中文
头像位置 --> settings --> Preferences --> Localization --> 选择简体中文
```

### 使用

如果使用 user 创建一个仓库，那么这个用户就是这个仓库的 owner

如果使用 group 创建一个仓库，那么这个组下添加的所有用户就是这个仓库的 owner

主程序员角色能对 master 分支及其他分支操作

开发者只能对非 master 分支操作(默认 master 分支是受保护的，可以关闭)

开发者在其他分支上操作之后可以提交合并 master 请求

### 备份恢复迁移

```bash
# 备份配置
cat /etc/gitlab/gitlab.rb
...
gitlab_rails['manage_backup_path'] = true                  # 开启备份
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"    # 备份路径，可变更
gitlab_rails['backup_keep_time'] = 604800                  # 备份保留时间
...

# 如果变更了路径需要重新配置
gitlab-ctl stop
gitlab-ctl reconfigure
gitlab-ctl restart

#  备份命令(可以写入定时任务)
gitlab-rake gitlab:backup:create

# 恢复
# 停止写入数据
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 恢复(BACKUP 后面只需要备份文件的时间名称就行)
gitlab-rake gitlab:backup:restore BACKUP=1641691573_2022_01_09_12.3.5

# 重启组件
gitlab-ctl restart

# 迁移升级步骤
备份: /etc/gitlab/gitlab.rb 和 backup 备份文件
新节点安装对应版本的gitlab进行恢复数据
新节点此时可以选择升级或者不升级，升级不能跨版本升级，12.3 --> 12.9.9(当前版本的最后一个版本) --> 13
```

### 忘记`root`密码

```bash
gitlab-rails console -e production

User.where(username:"root").first
user.password = "test123456"
user.save!
quit
```
