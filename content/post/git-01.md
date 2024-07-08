---
title: Git使用
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - git
categories:
  - git
url: post/git.html
toc: true
---

## git 安装

```bash
1.windows系统
https://git-scm.com/download/win

2.linux系统
yum install git -y

3.macos系统
https://git-scm.com/download/mac
```

<!-- more -->

## 本地仓库

### 基本使用

```bash
# 进入要管理的文件夹进行初始化
mkdir app
cd app
git init
ls -a

# 编辑文件
echo 'app01 20%' > index.html

# 管理目录下的文件状态
git status  # 当前可以看到提示 index.html 文件没有被追踪

# 管理指定文件或者当前目录 . 代表当前目录所有
git add index.html
git add .

# 配置个人信息
git config --global user.email "2564334707@qq.com"
git config --global user.name "klcc"

# 查看当前用户配置
 git config --global --list

# git颜色配置
git config --global color.ui true

# 生成版本 -m 后面是描述信息
git commit -m "app is 20% --v1"

# 查看记录
git log

# 编辑
echo 'app is complete' >> index.html

# 添加变化
git add .

# 提交
git commit -m "app complete --v2"

# 回退代码
git log  # 查看提交的历史记录
git reset --hard ID  # 根据ID回退(填写七位就行)

git reflog  # 查看所有的提交历史记录
```

### 基础命令总结

```bash
git init           # 就是将普通目录转为git的仓库(这个目录就支持版本管理了)
git add            # 将工作区的数据，拷贝到暂存区
git commit         # 将暂存区的数据同步到本地s仓库
git log            # 查看所有的提交记录
git reflog         # 查看所有的历史提交记录
git reset --hard   # 回退到指定的commitID
git status         # 查看提交状态
git checkout       # 把工作区变化撤销
git reset HEAD     # 把暂存区拉回到工作区（绿变红）

# 注意 ：
.git文件夹做了记录，不能删除，如果删除，版本的记录也就没了
空文件夹不会被版本管理
```

![image-20220108114451077](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220108114451077.png)

### 分支使用

分支可以给使用者提供多个环境的可以，意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线

```bash
git branch        # 查看分支
git branch dev    # 创建dev分支开发商城项目
git checkout dev  # 切换到dev分支

# 模拟商城代码
echo 'shopping system is 20%' >  shopping.py

# 第一次提交
git add .
git commit -m 'shopping system 20% --v3'

# 第二次提交
echo 'shopping system is 50%' >>  shopping.py
git add .
git commit -m 'shopping system 50% --v4'

# 发现 app --v2 完成后有bug需要切换回去修复
git checkout master
git branch

# 创建 bug 分支单独修复 基于app --v2的代码
git branch bug
git checkout bug
git branch

# 模拟修复
echo 'bug repair' >> index.html
# 可以进行上线测试bug是否修复

# 提交修复bug后的代码
git add .
git commit -m 'app bug repair --v5'

# 合并 bug 分支到 master 分支上
git checkout master
git branch
git merge bug -m 'app bug repair master merge bug --v6'


# 切回 dev 分支继续完成商城开发
git checkout dev
git branch

echo 'shopping system is 100%' >>  shopping.py
git add .
git commit -m 'shopping system 100% --v7'

# 合并主分支到dev分支测试 shop system 然后提交测试
git merge master  -m 'dev merge master test shopping --v8'

# 切换到master 合并dev分支功能
git checkout master
git branch
git merge dev -m 'master merge dev shop system --v9'
```

### 分支命令总结

```bash
git branch           # 查看所有分支
git branch name      # 创建 name 分支
git checkout -b name # 创建并切换到分支
git checkout name    # 切换到 name 分支
git branch -d name   # 删除 name 分支
git branch -a        # 查看远程分支
git merge dev        # 将dev分支最新代码合并到当前所在的分支
```

## 远程仓库

### 基本使用

![image-20220108124207557](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220108124207557.png)

![image-20220108124319492](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220108124319492.png)

```bash
# 给远程仓库起别名为 origin
git remote add origin git@gitee.com:liuzhijin1/gittest.git

# 查看当前远程仓库
git remote -v

# 删除origin远程仓库
git remote rm origin

# 查看当前仓库配置
git config --local  --list

# 向远程推送代码，需要添加公钥才能推送
git push origin master

# 可以切换分支推送代码
git checkout dev
git push origin dev

# 下载代码(git https都可以) 需要认证
git clone git@gitee.com:liuzhijin1/gittest.git

# 然后就可以正常的本地操作
...

# 同步远程仓库最新代码
git pull origin master


```

### 总结

```bash
# 下载代码
git clone 第一次必须要克隆项目

# 拉取代码
git pull origin master
git push origin master
等价于
git fetch origin
git merge origin/dev

# 添加远程连接(别名)
git remote add origin 地址
git remote -v

# remote源操作
git remote                # 查看远程仓库
git remote remove origin  # 删除远程仓库

# 推送代码
git push origin dev

# 记录图形展示
git log --graph --pretty=format:"%h %s"
```

![image-20220108131519462](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20220108131519462.png)

## 其他补充

### `tag`标签

git 标签就是对 commit 的一次快照，便于后续将特定时期 的代码快速取出，在代码发布时可以使用标签发布

```bash
# 对当前最新提交的代码创建标签，-a标签名称，-m标 签描述
git tag -a "v1.1" -m "描述信息"

# 创建标签，指定commitID
git tag -a v1.2 CommitID -m "Messages"

# 查看标签详情
git tag
git log -l

## 实例:
# 当前代码打上标签
git tag -a v2.0 -m "shopping up"

# 提交所有标签
git push origin --tags

# 找到你想打上标签的ID
git reflog   # f4ad9ef 是完成app时候的ID

# 指定ID创建标签
git tag -a v1.0 f4ad9ef -m "app complete"

# 提交指定标签
git push orgin --tag v1.0

# 根据版本查看commit ID信息
git show v1.0
```

### 免密登录

```bash
# HTTPS
# 原来的地址：https://gitee.com/xxx/treenb.git
# 修改的地址：https://用户名:密码@gitee.com/xxx/treenb.git

git remote add origin https://用户名:密码@gitee.com/xxx/treenb.git
git push origin maste

# SSH
# 生成公钥和私钥(默认放在 ~/.ssh目录下id_rsa.pub公钥、id_rsa私钥）
ssh-keygen

# 拷贝公钥的内容，并设置到github中
# 在git本地中配置ssh地址
git remote add origin git@github.com:/dbhot.git

# 以后使用
git push origin master
```

### `ignore`文件忽略

```bash
让 Git 不再管理当前目录下的某些文件 .gitignore 通常情况下有如下文件可能需要忽略
1.程序运行时产生的垃圾文件
2.程序运行时产生的缓存文件
3.程序本地开发使用的图片文件
4.程序连接数据一类的配置文件

例如以下文件类型:
.idea/
*.h
!a.h
files/
*.py[c|a|d]

# 实例
cd workdir
cat .gitignore
*.pyc

# 提交之后才会生效
git add .
git commit -m "ignore"
git push origin master

# 测试
touch a.txt
touch 1.pyc
touch 2.pyc

git status  # 此时 1.pyc 和 2.pyc 都被屏蔽了
```
