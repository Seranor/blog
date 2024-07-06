---
title: python-模块(三)
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-21.html
toc: true
---

### random 模块

- `random()`

  ```python
  import random
  print(random.random())  # 随机产生一个0-1之间的小数
  ```

  <!-- more -->

- `randint()`

  ```python
  import random
  print(random.radint(1, 6))  # 随机产生一个1-6之间的整数
  ```

- `uniform()`

  ```python
  import random
  print(random.uniform(1, 6)) # 随机产生一个1-6之间的小数
  ```

- `choice()`

  ```python
  import random
  print(random.choice(['一等奖', '二等奖', '三等奖', '谢谢惠顾']))  # 随机抽取其中一个
  ```

- `sample()`

  ```python
  import random
  print(random.sample(['安徽省', '江苏省', '山东省', '广东省'],2))  # 随机出去指定样本数量
  ```

- `shuffle()`

  ```python
  import random
  l = [2, 3, 4, 5, 6, 7, 8, 9, 10, 'J', 'Q', 'K', 'A']
  random.shuffle(l)  # 随机打乱容器类型中的元素
  print(l)
  ```

- `randrange()`

  ```python
  import random
  print(random.randrange(1, 10, 2))  # 随机产生1到10之间的奇数,2 步长
  ```

- 验证码生成
  ```python
  def get_code(n):
    # 提前定义一个存储验证码的变量
    code = ''
    # 由于需要产生五位 每一位的操作都是一样的 所以肯定需要使用循环
    for i in range(n):
        # 随机产生一个数字
        random_int = str(random.randint(0, 9))
        # 随机产生一个大写字母
        random_upper = chr(random.randint(65, 90))
        # 随机产生一个小写字母
        random_lower = chr(random.randint(97, 122))
        # 随机选取一个
        temp = random.choice([random_int, random_upper, random_lower])
        # 拼接到字符串中
        code += temp
    return code
  code1 = get_code(5)
  code2 = get_code(10)
  code3 = get_code(8)
  print(code1,code2,code3)
  ```

### os 模块

- `mkdir()`

  ```python
  import os
  os.mkdir('test')  # 只能创建单级目录
  ```

- `makedirs()`

  ```python
  import os
  os.makesdirs('test/tes1')  # 创建test目录和下级目录test1(当前斜线表示Linux系统内)
  # 和 linux命令 mkdir -p test/test1 结果一样
  ```

- `rmdir()`

  ```python
  import os
  os.rmdir('test')  # 只能删除空目录,否则会报错
  ```

- 获取当前文件所在路径

  ```python
  import os
  path = os.path.dirname(__file__)
  print(path)
  ```

- 路劲拼接

  ```python
  import os
  path = os.path.dirname(__file__)
  db_path = os.path.join(path, 'db')
  print(db_path)  # 返回当前路径加db
  ```

- `listdir()`

  ```python
  # 列举出指定路径下的文件名称(任意类型文件) 返回的是一个列表
  # 相当于Linux命令的 ls
  import os
  print(os.listdir())
  print(os.listdir('/tmp'))
  ```

- `remove()`

  ```python
  import os
  os.remove('a.txt')  # 删除当前a.txt
  # 删除文件不能是目录  相当于 rm -f
  ```

- `rename()`

  ```python
  import os
  os.rename('a.txt', 'b.txt')  # 将a.txt 改名为 b.txt
  os.rename('test', 'test1')  # 将test目录改为test1目录
  # 可以改文件或者目录名字 类似Linux命令的 mv
  ```

- `getcwd()`

  ```python
  import os
  print(os.getcwd())  # 获取当前工作路径  Linux --> pwd
  ```

- `chdir()`

  ```python
  import os
  os.chdir('/mnt')  # 切换工作路径到/mnt目录  Linux --> cd
  ```

- `exists()`

  ```python
  # 判断目录或者文件是否存在 存在返回Treu 不存在返回False
  import os
  print(os.path.exists('/tmp'))  # 存在目录/tmp返回True 不存在返回False
  print(os.path.exists('/tmp/test.txt'))  # 存在文件test.txt 返回True 不存在返回False
  print(os.path.exists('/tmp/test'))  # 存在目录/tmp/test返回True 不存在返回False
  ```

- `isfile()`

  ```python
  # 判断是否是文件,是文件返回True,是目录返回False
  import os
  print(os.path.isfile('/tmp/test.txt'))  # True
  print(os.path.isfile('/tmp/'))  # Fales
  ```

- `isdir()`

  ```python
  # 判断是否是目录,是目录返回True,是文件返回False
  print(os.path.exists('/tmp/test.txt'))  # False
  print(os.path.isfile('/tmp/'))  # True
  ```

- `getsize()`

  ```python
  # 获取文件大小(字节数)
  print(os.path.getsize(r'a.txt'))
  ```

- 获取目录文件内的文件并按选择读取

  ```python
  basic_path = os.path.dirname(__file__)  # 获取当前文件的路径
  log_path = os.path.join(basic_path, 'test1')  # 拼接路径,得到 当前路径/test1
  file_list = os.listdir(log_path)  # 得到 test1路径下的文件列表

  while True:
      for i, j in enumerate(file_list,1):  # 1 file_list[0], 2 file_list[1], 3 file_list[2]
          print(i,j)
      choice = input('请输入想查看的日志: ').strip()
      if choice.isdigit():
          choice = int(choice)
          if choice in range(len(file_list) + 1):  # range顾头不顾尾所以 加一
              file_name = file_list[choice - 1]  # choice是从1开始的,所以取列表索引时需要 减一
              file_path = os.path.join(log_path,file_name)  # 拼接选择的文件绝对路径
              with open(file_path, 'r', encoding='utf8') as f:
                  print(f.read())  # 打印文件内容
  ```

### sys 模块

```python
import sys

print(sys.path)  # 搜索模块的路径集
print(sys.version)  # 返回python解释器版本以及所处的平台
print(sys.platform)  # 返回当前操作系统类型

print(sys.argv)  # 获取当前执行文件的绝对路径

# sys.argv 第二种用法,类似shell脚本的外部传参
try:
    username = sys.argv[1]  # 相当于shell脚本中 $1
    password = sys.argv[2]  # 相当于shell脚本中 $2
    if username == 'jason' and password == '123':
        print('正常执行文件内容')
    else:
        print('用户名或密码错误')
except Exception:
    print('请输入用户名和密码')
    print('目前只能让你体验一下(游客模式)')


sys.exit(1)  #  执行python执行脚本后抛出的异常信息默认为0,在shell中可以使用  echo $?  命令可以捕获到 一般认为0是正常执行，如果抛出其他数值则出现异常,如果python脚本中找到确切的数字就可以找到指定位置
```

### 序列化模块

> json 格式化数据: 跨语言传输

```python
import json

d = {'username': 'jason', 'pwd': 123}
print(d,type(d))  # {'username': 'jason', 'pwd': 123} <class 'dict'>

# 将字典转为json格式的字符串(序列化), 此时是 str 类型, 但是是json的格式
res1 = json.dumps(d)
print(res1, type(res1))  # {"username": "jason", "pwd": 123} <class 'str'>

# 将json格式字符串转成当前语言对应的某个数据类型(反序列化)
res2 = json.loads(res1)
print(res2, type(res2))  # {'username': 'jason', 'pwd': 123} <class 'dict'>

# bytes的
bytes_data = b'{"username": "jason", "pwd": 123}'
bytes_str = bytes_data.decode('utf8')
bytes_dict = json.loads(bytes_str)
print(bytes_dict, type(bytes_dict))  # {'username': 'jason', 'pwd': 123} <class 'dict'>

# 将字典以json格式写入文件(序列化)
with open(r'a.json', 'w', encoding='utf8') as f:
    f.write(json.dumps(d))

# 将字典取出来(反序列化)
with open(r'a.json', 'r', encoding='utf8') as f:
    data = f.read()
res = json.loads(data)
print(res, type(res))  # {'username': 'jason', 'pwd': 123} <class 'dict'>

# 不用write 可以使用dump直接将 字典d 以json格式写入文件(序列化)
with open(r'b.json', 'w', encoding='utf8') as f:
    json.dump(d, f)

# 不用read 使用load 将字典d 从文件中去出来,直接转为字典(反序列化)
with open(r'b.json', 'r', encoding='utf8') as f:
    rest = json.load(f)
print(rest, type(rest))  # {'username': 'jason', 'pwd': 123} <class 'dict'>

"""
暂且可以简单的理解为
    序列化就是将其他数据类型转换成字符串过程
        json.dumps()
    反序列化就是将字符串转换成其他数据类型
        json.loads()
"""
```

### subprocess 模块

```python
res = subprocess.Popen('ps -ef',  # 在终端运行的命令
                       shell=True,  # 新开一个端口
                       stdout=subprocess.PIPE,  # 执行完命令, 将正确输出放到一个管道里
                       stderr=subprocess.PIPE  # 将错误输出放到一个管道里
                       )
print('stdout',res.stdout.read().decode('utf8'))  # 获取正确命令执行之后的结果
print('stderr',res.stderr.read().decode('utf8'))  # 获取错误命令执行之后的结果
```
