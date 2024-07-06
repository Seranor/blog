---
title: 简单的小脚本
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
  - Linux
categories:
  - Linux
url: post/python-shell.html
toc: true
---

### 时间间隔执行命令

<!-- more -->

```python
import datetime

# 开始日期和结束日期填写
import subprocess

begin = datetime.datetime(2022, 2, 21, 00, 00, 00)
end = datetime.datetime(2022, 3, 7, 00, 00, 00)

d = begin
delta = datetime.timedelta(seconds=600)  # 间隔，按秒算，600s  10分钟
time_list = []
while d <= end:
    interval_time = d.strftime("%Y-%m-%d %H:%M:%S")
    time_list.append(interval_time)
    d += delta

time_list2 = time_list.copy()
time_list3 = []
for i in range(len(time_list2)):
    if i < len(time_list2) - 1:
        time_list3.append((time_list2[i], time_list2[i + 1]))

# 基础命令，密码自己修改
base_cmd = '''
    influx -username 'admin' -database location -password 'admin' -execute \
    "SELECT * INTO llocation FROM location_bak \
    where time>='%s' \
    AND time<='%s' \
    GROUP BY * TZ('Asia/Shanghai') && docker restart influxdb && sleep 60"
    '''

for j in time_list3:
    command = base_cmd % (j[0], j[1])
    # print(base_cmd % (j[0], j[1])) 在终端打印命令

    # 执行命令
    res = subprocess.Popen(
        command,
        shell=True,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE
    )

    print('stdout', res.stdout.read().decode('utf8'))  # 获取正确命令执行之后的结果 终端打印
    print('stderr', res.stderr.read().decode('utf8'))  # 获取错误命令执行之后的结果 终端打印
    # 写入文件操作  当前目录下的 command.sh
    with open(r'command.sh', 'a', encoding='utf8') as f:
        f.write(command)
```
