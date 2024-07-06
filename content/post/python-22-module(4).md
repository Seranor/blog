---
title: python-模块(四)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-22.html
toc: true
---

### 1. `hashlib`模块

> 加密模块
>
> 加密: 将明文数据通过一些列算法变成密文，为了数据安全
>
> 加密算法: md5、sha、 base、hmac 等

<!-- more -->

#### 1.1 基本使用

```python
import hashlib

md51 = hashlib.md5()  # 先决定算法类型,md5普遍使用
md51.update('哈喽'.encode('utf8'))  # 将明文数据传递给md5算法,update只能接受bytes类型数据
# md51.update(b'hello word')  # 当数据是英文和数字的时候可以在前面加 b
rest1 = md51.hexdigest()  # 获取加密之后的密文数据
print(rest1)  # 492389292f5e200f6d1518055b0b1755  (一串随机的字符串)

"""
1.加密之后的密文数据是没有办法反解密成明文数据的
    市面上所谓的破解 其实就是提前算出一系列明文对应的密文
    之后比对密文再获取明文
"""
```

#### 1.2 特性一

```python
# 明文数据只要是相同的 那么无论如何传递加密结果肯定是一样的

import hashlib
md52 = hashlib.md5()
md52.update(b'admin123')
rest2 = md52.hexdigest()
print(rest2)  # 0192023a7bbd73250516f069df18b500

md53 = hashlib.md5()
md53.update(b'admin')
md53.update(b'123')
rest3 = md53.hexdigest()
print(rest3)  # 0192023a7bbd73250516f069df18b500
```

#### 1.3 特性二

```python
# 密文数据越长表示内部对应的算法越复杂 越难被正向破解

import hashlib
sha1 = hashlib.sha256()
sha1.update(b'admin123')
rest = sha1.hexdigest()
print(rest)  # 240be518fabd2724ddb6f04eeb1da5967448d7e831c08c8fa822809f74c720a9

"""
密文越长表示算法越复杂 对应的破解算法的难度越高
但是越复杂的算法所需要消耗的资源也就越多 密文越长基于网络发送需要占据的数据也就越大
具体使用什么算法取决于项目要求 一般情况下md5足够了
"""
```

#### 1.4 特性三

```python
# 涉及到用户密码存储,都是密文,只要用户自己知道明文是什么
# 内部程序员无法得知明文数据
# 数据泄露也无法得知明文数据
```

#### 1.5 加盐处理

```python
# 在对明文数据做加密处理过程前添加一些干扰项

import hashlib
md54 = hashlib.md5()
md54.update(b'Add salt')  # 此处是自己加的干扰项
md54.update(b'passwordadmin123')  # 此处就是用户传入的数据
rest4 = md54.hexdigest()
print(rest4)
```

#### 1.6 动态加盐处理

```python
# 在对明文数据做加密处理过程前添加一些变化的干扰项

import hashlib
password = 'admin123'
md55 = hashlib.md5()
# 动态加盐(干扰项)  当前时间 用户名的部分 uuid(随机字符串(永远不会重复))
md55.update(password[::-1].encode('utf8'))  # 此时的变化量为用户输入的密码字符串取反的值
md55.update(password.encode('utf8'))
rest5 = md55.hexdigest()
print(rest5)  # 658056dbefa3427fe4dddfbf28d4d54d
```

#### 1.7 校验文件一致性

```python
"""
文件不是很大的情况下,可以将所有文件内部全部加密处理
但是如果文件特别大,全部加密处理相当的耗时好资源,针对大文件可以使用切片读取的方式
"""
# 方式一: 分开读
import os
import hashlib

def file_md5(path):
    path_size = os.path.getsize(path)  # 获取文件的大小
    md5 = hashlib.md5()
    with open(path, 'rb') as f:
        while path_size >= 4096:  # 如果文件大于4096
            cont = f.read(4096)  # 每次读取文件4096个字节
            md5.update(cont)
            path_size -= 4096
        else:
            cont = f.read()  # 文件小于4096一次性读取
            if cont:
                md5.update(cont)
    return md5.hexdigest()  # 返回加密值


# print(file_md5('a.log'))

# 校验两个文件
def diff_file(path1, path2):
    return file_md5([path1]) == file_md5([path2])

# 方式二:
# 指定分片读取策略(读几段 每段几个字节)  10  f.seek()
```

### 2. `logging`模块

#### 2.1 基本使用

```python
import logging

# 日志有五个等级(从上往下重要程度不一样)
# logging.debug('debug级别')  # 10
# logging.info('info级别')  # 20
# logging.warning('warning级别')  # 30
# logging.error('error级别')  # 40
# logging.critical('critical级别')  # 50
'''默认记录的级别在30及以上'''

# 简单使用
import logging
file_handler = logging.FileHandler(filename='x1.log', mode='a', encoding='utf-8',)
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s -%(module)s:  %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S %p',
    handlers=[file_handler,],
    level=logging.ERROR
)
logging.error('日志模块很好学 不要自己吓自己')
"""
1.如何控制日志输入的位置
    想在文件和终端中同时打印
2.不同位置如何做到不同的日志格式
    文件详细一些 终端简单一些
"""
```

#### 2.2 详细介绍

```python
import logging

# 1.logger对象:负责产生日志
logger = logging.getLogger('转账记录')
# 2.filter对象:负责过滤日志(直接忽略)
# 3.handler对象:负责日志产生的位置
hd1 = logging.FileHandler('a1.log',encoding='utf8')  # 产生到文件的
hd2 = logging.FileHandler('a2.log',encoding='utf8')  # 产生到文件的
hd3 = logging.StreamHandler()  # 产生在终端的
# 4.formatter对象:负责日志的格式
fm1 = logging.Formatter(
    fmt='%(asctime)s - %(name)s - %(levelname)s -%(module)s:  %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S %p',
)
fm2 = logging.Formatter(
    fmt='%(asctime)s - %(name)s %(message)s',
    datefmt='%Y-%m-%d',
)
# 5.绑定handler对象
logger.addHandler(hd1)
logger.addHandler(hd2)
logger.addHandler(hd3)
# 6.绑定formatter对象
hd1.setFormatter(fm1)
hd2.setFormatter(fm2)
hd3.setFormatter(fm1)
# 7.设置日志等级
logger.setLevel(30)
# 8.记录日志
logger.debug('写了半天 好累啊')
```

#### 2.3 配置字典

```python
# 核心就在于CV
import logging
import logging.config

standard_format = '[%(asctime)s][%(threadName)s:%(thread)d][task_id:%(name)s][%(filename)s:%(lineno)d]' \
                  '[%(levelname)s][%(message)s]' #其中name为getlogger指定的名字

simple_format = '[%(levelname)s][%(asctime)s][%(filename)s:%(lineno)d]%(message)s'

logfile_path = 'a3.log'
# log配置字典
LOGGING_DIC = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': standard_format
        },
        'simple': {
            'format': simple_format
        },
    },
    'filters': {},  # 过滤日志
    'handlers': {
        #打印到终端的日志
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',  # 打印到屏幕
            'formatter': 'simple'
        },
        #打印到文件的日志,收集info及以上的日志
        'default': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件
            'formatter': 'standard',
            'filename': logfile_path,  # 日志文件
            'maxBytes': 1024*1024*5,  # 日志大小 5M
            'backupCount': 5,
            'encoding': 'utf-8',  # 日志文件的编码，再也不用担心中文log乱码了
        },
    },
    'loggers': {
        #logging.getLogger(__name__)拿到的logger配置  空字符串作为键 能够兼容所有的日志
        '': {
            'handlers': ['default', 'console'],  # 这里把上面定义的两个handler都加上，即log数据既写入文件又打印到屏幕
            'level': 'DEBUG',
            'propagate': True,  # 向上（更高level的logger）传递
        },  # 当键不存在的情况下 (key设为空字符串)默认都会使用该k:v配置
    },
}


# 使用配置字典
logging.config.dictConfig(LOGGING_DIC)  # 自动加载字典中的配置
logger1 = logging.getLogger('xxx')
logger1.debug('测试')
```

### 3. 第三方模块

```python
# 并不是python自带的 需要基于网络下载!!!

'''pip所在的路径添加环境变量'''
下载第三方模块的方式
    方式1:命令行借助于pip工具
        pip3 install 模块名  # 不知道版本默认是最新版
        pip3 install 模块名==版本号  # 指定版本下载
        pip3 install 模块名 -i 仓库地址  # 临时切换
        '''命令行形式永久修改需要修改python解释器源文件'''
    方式2:pycharm快捷方式
        settings
        	project
            	project interprter
                	双击或者加号
        点击右下方manage管理添加源地址即可
# 下载完第三方模块之后 还是使用import或from import句式导入使用
"""
pip命令默认下载的渠道是国外的python官网(有时候会非常的慢)
我们可以切换下载的源(仓库)
    (1) 阿里云 http://mirrors.aliyun.com/pypi/simple/
    (2) 豆瓣 http://pypi.douban.com/simple/
    (3) 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
    (4) 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/
    (5) 华中科技大学http://pypi.hustunique.com/

pip3 install openpyxl -i http://mirrors.aliyun.com/pypi/simple/
"""
"""
下载第三方模块可能报错的情况及解决措施
	1.报错的提示信息中含有关键字timeout
		原因:网络不稳定
		措施:再次尝试 或者切换更加稳定的网络
	2.找不到pip命令
		环境变量问题
	3.没有任何的关键字 不同的模块报不同的错
		原因:模块需要特定的计算机环境
		措施:拷贝报错信息 打开浏览器 百度搜索即可
			pip下载某个模块报错错误信息
"""
```

linux 全局配置 pip 源

```bash
mkdir ~/.pip

cat ~/.pip/pip.conf
[global]
timeout = 6000
index-url = https://pypi.douban.com/simple/
[install]
use-mirrors = true
mirrors = https://pypi.douban.com/simple/
trusted-host = pypi.douban.com
```
