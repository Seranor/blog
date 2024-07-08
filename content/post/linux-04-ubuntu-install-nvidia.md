---
title: Ubuntu安装NVIDIA驱动
lastmod: 2020-11-21T16:43:23+08:00
date: 2020-11-21T11:52:03+08:00
tags:
  - Ubuntu
  - NVIDIA
  - Linux
categories:
  - Linux
url: post/linux-nvidia.html
toc: true
---

### 安装 NVIDIA 驱动

#### 方式一

```bash
#关闭桌面
systemctl disable gdm3

#禁用nouveau
sudo echo "blacklist nouveau" >>  /etc/modprobe.d/blacklist.conf
sudo echo "options nouveau modeset=0" >>  /etc/modprobe.d/blacklist.conf

# 官网下载Linux版驱动 https://www.nvidia.cn/Download/index.aspx

chmod +x NVIDIA-Linux-x86_64-440.100.run
apt install curl make gcc g++ wget pkg-config --reinstall -y

./NVIDIA-Linux-x86_64-440.100.run  -a -q -s -z -Z
```

<!-- more -->

#### 方式二

```bash
#关闭桌面
systemctl disable gdm3

#禁用nouveau
sudo echo "blacklist nouveau" >>  /etc/modprobe.d/blacklist.conf
sudo echo "options nouveau modeset=0" >>  /etc/modprobe.d/blacklist.conf

# 查找内核版本
cat /proc/driver/nvidia/version

# 卸载显卡残留依赖
sudo apt-get --purge remove nvidia* -y
sudo apt-get --purge remove "*nvidia*" -y
sudo apt-get --purge remove "*cublas*" "cuda*" -y
sudo apt autoremove -y


# 安装驱动460版本
apt install nvidia-headless-460 -y
apt install nvidia-utils-460


# 安装驱动455版本
apt install -y nvidia-driver-455 nvidia-utils-455 nvidia-cuda-dev nvidia-cuda-toolkit nvidia-opencl-dev

# 检查显卡驱动
nvidia-smi
```

### 禁止 Ubuntuz 自动更新软件包

- 防止自动更新依赖之后驱动无法正常使用

```bash
	sed -i 's#1#0#g' /etc/apt/apt.conf.d/20auto-upgrades 1>/dev/null 2>&1
	sed -i 's#1#0#g' /etc/apt/apt.conf.d/10periodic 1>/dev/null 2>&1
	systemctl stop apt-daily.service 1>/dev/null 2>&1
	systemctl stop apt-daily.timer 1>/dev/null 2>&1
	systemctl stop apt-daily-upgrade.service 1>/dev/null 2>&1
	systemctl stop apt-daily-upgrade.timer 1>/dev/null 2>&1
	systemctl disable apt-daily.service 1>/dev/null 2>&1
	systemctl disable apt-daily.timer 1>/dev/null 2>&1
	systemctl disable apt-daily-upgrade.service 1>/dev/null 2>&1
	systemctl disable apt-daily-upgrade.timer 1>/dev/null 2>&1
```
