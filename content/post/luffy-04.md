---
title: Python中Redis的使用
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - python
categories:
  - python
url: post/luffy-04.html
toc: true
---

### `Redis`介绍和安装

<!-- more -->

```python
# 1. redis 是一个非关系型数据库(区别于mysql关系型数据库，关联关系，外键，表)，nosql数据库(not only sql:不仅仅是SQL)，数据完全内存存储(速度非常快)
# 2. redis就是一个存数据的地方
# 3. redis是 key : value  存储形式 ---> value类型有5大数据类型: 字符串，列表，hash(字典)，集合，有序集合

# java:hashMap  存key-value形式
# go：maps      存key-value形式
# 4.redis的好处

(1) 速度快，因为数据存在内存中，类似于字典，字典的优势就是查找和操作的时间复杂度都是O(1)
(2) 支持丰富数据类型，支持string，list，set，sorted set，hash
(3) 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
(4) 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

# 5. redis 最适合的场景 ---> 主要做缓存 ---> 它又叫缓存数据库
（1）会话缓存（Session Cache）---》存session---》速度快
（2）接口，页面缓存---》把接口数据，存在redis中
（3）队列--->celery使用
（4）排行榜/计数器--->个人页面访问量
（5）发布/订阅


# 6. 安装 ---> c语言写的开源软件 ---> 官方提供源码 ---> 如果是在mac或linux上需要 编译，安装
    -redis最新稳定版版本6.x
    -win：作者不支持windwos，本质原因：redis很快，使用了io多路复用中的epoll的网络模型，这个模型不支持win，所以不支持（看到高性能的服务器基本上都是基于io多路复用中的epoll的网络模型，nginx），微软基于redis源码，自己做了个redis安装包，但是这个安装包最新只到3.x，又有第三方组织做到最新5.x的安装包
  		安装包---》编译完成的可执行文件---》下一步安装
    	linux--》make成可执行文件---》make install 安装
    -linux，mac平台安装


# 7. win下载地址
  // 最新5.x版本 https://github.com/tporadowski/redis/releases/
  // 最新3.x版本 https://github.com/microsoftarchive/redis/releases
  一路下一步安装

# mysql 有个图形化客户端-Navicat很好用
# redis 也有很多，推荐你用rdb，https://github.com/uglide/RedisDesktopManager/releases

# redis纯内存操作，有可能把内存占满了，这个配置是最多使用多少内存

# redis服务的启动与关闭
方式一：win上，就在服务中了，把服务开启即可，在服务中启动关闭
方式二：命令启动，等同于mysqld
  redis-server redis.windows-service.conf
  redis-server 配置文件


# 客户端连接
  命令行：redis-cli -p 端口 -h 地址
  客户端 ：rdb连接
```

### `Python`连接`Redis`

#### 普通连接

```python
# 安装模块
pip install redis

# 普通连接
from redis import Redis
conn = Redis(host='localhost', port=6379, password=None)  # 还有其他的参数，如 db 库
conn.set('name', 'lzj')
res = conn.get('name')
print(res)
```

#### 连接池连接

```python
# 连接池连接
import redis
POOL = redis.ConnectionPool(max_connections=10,host='localhost',port=6379)
conn = redis.Redis(connection_pool=POOL)
print(conn.get('name'))

'''
该方式有个问题，多线程时就会创建一个POOL，会产生多个连接数
需要使用单例的模式 使用模块导入方式
'''

# 使用模块导入方式的单例
  # redis_pool.py
  import redis
  POOL = redis.ConnectionPool(max_connections=10,host='localhost',port=6379)

  # conn_redis.py
  import redis
  from redis_pool import POOL
  conn = redis.Redis(connection_pool=POOL)
  print(conn.get('name'))

  # 多线程演示
  from threading import Thread
  import redis
  import time
  from redis_pool import POOL  # 真报错吗？不会报错,
  def get_name():
      conn=redis.Redis(connection_pool=POOL)
      print(conn.get('name'))
  for i in range(10):
      t=Thread(target=get_name)
      t.start()

  time.sleep(2)

# django中有没有连接池
  没有，django中一个请求就会创建一个mysql连接，django并发量不高，mysql能撑住
  想在django中使用连接池，有第三方：https://www.cnblogs.com/wangruixing/p/13030755.html

# 各种锁：https://zhuanlan.zhihu.com/p/489305763
```

#### 单例模式

```python
单例模式（Singleton Pattern）是一种常用的软件设计模式，该模式的主要目的是确保某一个类只有一个实例存在。当你希望在整个系统中，某个类只能出现一个实例时，单例对象就能派上用场

比如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个 AppConfig 的类来读取配置文件的信息。如果在程序运行期间，有很多地方都需要使用配置文件的内容，也就是说，很多地方都需要创建 AppConfig 对象的实例，这就导致系统中存在多个 AppConfig 的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似 AppConfig 这样的类，我们希望在程序运行期间只存在一个实例对象
```

#### 实现单例模式

```python
1.使用模块
其实，Python 的模块就是天然的单例模式，因为模块在第一次导入时，会生成 .pyc 文件，当第二次导入时，就会直接加载 .pyc 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做：

  mysingleton.py
  class Singleton(object):
      def foo(self):
          pass
  singleton = Singleton()

将上面的代码保存在文件 mysingleton.py 中，要使用时，直接在其他文件中导入此文件中的对象，这个对象即是单例模式的对象

  from a import singleton


2.使用装饰器
  def Singleton(cls):
      instance = None

      def _singleton(*args, **kargs):
          nonlocal instance
          if not instance:
              instance = cls(*args, **kargs)
          return instance

      return _singleton

  @Singleton
  class A(object):
      def __init__(self, x=0):
          self.x = x

  a1 = A(2)
  a2 = A(3)
  print(a1.x)
  print(a2.x)
  print(a1 is a2)

3.使用类方法
  class Singleton(object):
      _instance=None
      def __init__(self):
          pass
      @classmethod
      def instance(cls, *args, **kwargs):
          if not cls._instance:
              cls._instance=cls(*args, **kwargs)
          return cls._instance

  a1=Singleton.instance()
  a2=Singleton().instance()

  print(a1 is a2)

4.基于new方法实现
  class Singleton(object):
      _instance=None
      def __init__(self):
          pass

      def __new__(cls, *args, **kwargs):
          if not cls._instance:
              cls._instance = object.__new__(cls)
          return cls._instance

  obj1 = Singleton()
  obj2 = Singleton()
  print(obj1 is obj2)

5.基于metaclass方式实现
  class SingletonType(type):
      _instance=None
      def __call__(cls, *args, **kwargs):
          if not cls._instance:
              # cls._instance = super().__call__(*args, **kwargs)
              cls._instance = object.__new__(cls)
              cls._instance.__init__(*args, **kwargs)
          return cls._instance

  class Foo(metaclass=SingletonType):
      def __init__(self,name):
          self.name = name


  obj1 = Foo('name')
  obj2 = Foo('name')
  print(obj1.name)
  print(obj1 is obj2)
```

### `Redis`的`String`操作

```python
from redis import Redis
conn = Redis(host='localhost', port=6379, password=None)

# set(key, value) 设置 key 的值
'''
在Redis中设置值，默认，不存在则创建，存在则修改
参数:
   ex，过期时间（秒）
   px，过期时间（毫秒）
   nx，如果设置为True，则只有name不存在时，当前set操作才执行,值存在，就修改不了，执行没效果
   xx，如果设置为True，则只有name存在时，当前set操作才执行，值存在才能修改，值不存在，不会设置新值
'''
conn.set('name', 'lzj')
conn.set('age', 18)
conn.set('gender', 'male', ex=3)   # 3s 后自动销毁
conn.set('name', 'lxx', nx=True)  # name 存在 无法修改
conn.set('age', 20, xx=True)      # age 存在 才可以被修改


# get(key) 获取指定 key 的值
print(str(conn.get('name'),encoding='utf-8'))


# setnx(key, value) 只有在 key 不存在时设置 key 的值
conn.setnx('name', 'lxx')     # name 存在 无法修改设置
conn.setnx('gender', 'male')  # gender 不存在 添加值


# setex(key, seconds, value) 将值value关联到key 并将key的过期时间设为seconds(以秒为单位)
conn.setex('xx',5,'yy') # 设置key值xx，value值yy，5秒钟过期


# psetex(key, milliseconds, value) 这个命令和setex命令相似 但它以毫秒为单位设置 key 的生存时间
conn.psetex('xx',5000,'yy')


# mset( key1, value1 ,key2, value2) 同时设置一个或多个 key-value 对
conn.mset({'name':'lxx','age':19})


# mget（key1, key2) 获取所有(一个或多个)给定 key 的值。
print(conn.mget(['name','age']))
print(conn.mget('name','age')) # 本质跟上面是一样的


# getset(key, value) 将给定key的值设为 value 并返回 key 的旧值(old value)
res = conn.getset('name','lyy') # 先获取再更新
print(res) # 打印的是lxx，但已经更新为 lyy 了


# getrange(key, start, end) 返回 key 中字符串值的子字符
"""
gbk：  2个字节表示一个字符
utf-8  3个字节表示一个字符
"""
res=conn.getrange('name',0,0)  # 前闭后闭区间，取到的是 lyy 的 l
res=conn.getrange('name',0,2).decode('utf-8')  # 将key值手动改为中文了 打印显示一个字 3个就会报错
print(res)


# setrange(key, offset, value) 用value参数覆写给定key所储存的字符串值 从偏移量offset开始
conn.setrange('name',2,'zzz')  # lyzzz 将第三个 y 覆写成了 zzz


# strlen(key) 返回 key 所储存的字符串值的长度(一个汉字，是3个字节)
print(conn.strlen('name'))


# incr(key)：将 key 中储存的数字值增一
print(conn.incr('age'))


# decr(key)：将 key 中储存的数字值减一
print(conn.decr('age'))


# append( key, value) 如果 key 已经存在并且是一个字符串 APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾
conn.append('name','nb')       # 在原有的字符串后面加nb
conn.append('this', 'is key')  # key值不存在就新增

conn.close()
```

### `Redis`的`hash`操作

```python
from redis import Redis

conn = Redis()

# hset(key, field, value) 将hash表 key 中的字段 field 的值设为 value (不存在，则创建；否则，修改)
conn.hset('hash_test','name','lqz')
conn.hset('hash_test','age','19')


# hmset(key, {field1, value1, field2,value2}) 同时将多个 field-value (域-值)对设置到hash表 key 中
conn.hmset('hash_test1',{'name':'egon','age':18}) # 报错被弃用了，但是还能正常插入值，原生未被弃用


# hget(key, field) 获取存储在hash表中指定字段的值
print(conn.hget('hash_test1','name'))


# hmget(key, [field1, field2])：获取所有给定字段的值
print(conn.hmget('hash_test1',['name','age']))


# hgetall(key) 获取在hash表中指定key的所有字段和值
print(conn.hgetall('hash_test1'))  # 尽量少执行，慢长命令，可能会撑爆应用程序的内容


# hlen(key) 获取hash中key对应的字段的个数
print(conn.hlen('hash_test1'))


# hkeys(key)  获取所有hash表中的字段
print(conn.hkeys('hash_test1'))


# hvals(key) 获取hash中key对应所有的value值
print(conn.hvals('hash_test1'))


# hexists(key, field) 查看hash表 key 中，指定的字段是否存在
print(conn.hexists('hash_test1','hobby')) # False
print(conn.hexists('hash_test1','name')) # True


# hdel(key ,field1,field2) 删除一个或多个hash表key中指定的字段
conn.hdel('hash_test1','name','wife')

# 将key对应的hash中指定字段的键值对删除
print(re.hdel('xxx' ,'sex' ,'name'))


# hincrby(key, field, amount=1) 为hash表key中的指定字段field的整数值加上增量
conn.hincrby('hash_test1','age',amount=10)

"""
自增key对应的hash中的指定字段的值，不存在则创建key=amount参数：key， hash对应的key
name，hash中key对应的字段amount，自增数（整数）
"""


# hincrbyfloat(key, field, amount=1.0) 为hash表key中的指定字段的浮点数值加上增量
conn.hincrbyfloat('hash_test1','age',1.2)  # 存在精度损失问题


# hscan(key, cursor=0, match=None, count=None) 迭代hash表中的键值对
"""
curso：游标，表示从哪个位置开始取match：匹配的字段,不写表示所有count：要取的条数
"""

# 存数据进去，在hash表key中加1000行键值对数据
for i in range(1000):
    conn.hset('test','%s-%s'%('key',i),i)

res=conn.hscan('test',cursor=0,count=100)
print(res) # 从0开始取，取出的100条数据是无序的
print(len(res[1])) # 打印条数

# 下面取到的跟上面的不会有重复，会根据上次的游标位置开始取
res=conn.hscan('test',cursor=res[0],count=100)
print(res)
print(len(res[1]))


# hscan_iter(name, match=None, count=None) # 获取所有采取这种方法

# hgetall ---> 一次性全取出来 ---> 生成器用的位置
for item in conn.hscan_iter('test',count=10): # 取出全部的value值，但是每次取10个
    print(item)
"""
生成器用在了什么位置
在取redis的hash类型的时候，因为hash类型的值很多，所以我没有用hgetall，
自己写了一个生成器，来分片取
"""
conn.close()
```

### `Redis`的列表操作

```python
from redis import Redis

conn = Redis()
# lpush(key ,value) 将一个或多个值插入到列表头部
conn.lpush('list_test', 'egon')
conn.lpush('list_test', 'lqz')


# rpush(key ,value) 将一个或多个值添加到列表尾部
conn.rpush('list_test', 'ylf', 'szw')


# lpushx(key ,value) 在key对应的list中添加元素，只有key已经存在时，值添加到列表的最左边
conn.lpushx('yyyyy', 'egon')  # key不存在，无法添加
conn.lpushx('list_test', 'egon')  # 成功添加到列表头部，列表可重复


# llen(key) key对应的list元素的个数
print(conn.llen('list_test'))


# linsert(key, where, refvalue, value)) 在key对应的列表的某一个元素前或者后插入元素
conn.linsert('list_test','before','lqz','dlrb') # 在lqz前面插入dlrb
conn.linsert('list_test','after','lqz','jason') # 在lqz后面插入jason

"""
参数：
key，redis的key
where，BEFORE（前）或AFTER（后）(小写也可以)
refvalue，标杆值，即：在它前后插入数据（如果存在多个标杆值，以找到的第一个为准）
value，要插入的数据
"""


# lset(key, index, value) 将key对应的list中的某一个索引位置重新赋值
conn.lset('list_test',0,'xxx') # 将0这个位置的值改为xxx

"""
参数：
key，redis的key
index，list的索引位置
value，要设置的值
"""


# lrem(key,count, value)：在key对应的list中删除指定的值
conn.lrem('list_test', count=1, value='lqz')

"""
参数：
key，redis的key
value，要删除的值
count，  count=0，删除列表中所有的指定值；
count=2,从前到后，删除2个；
count=-2,从后向前，删除2个
"""


# lpop(key)：将key对应的list的左侧获取第一个元素并在列表中移除，返回值则是第一个元素
print(conn.lpop('list_test'))


# rpop(key)：将key对应的list的右侧获取最后一个元素并在列表中移除，返回值为移除的元素
print(conn.rpop('list_test'))


# lindex(key, index) 在key对应的列表中根据索引获取列表元素
print(conn.lindex('list_test',3)) # 从0开始取


# lrange(key, start, stop) 在key对应的列表分片获取元素（指定范围内）
print(conn.lrange('list_test', 0, 2))
"""
参数：
key，redis的key
start，索引的起始位置
stop，索引结束位置
"""


# ltrim(key, start, stop) 将key对应的列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
conn.ltrim('list_test',2,4)   # 保留2-4之间的

"""
参数：
key，redis的key
start，索引的起始位置
stop，索引结束位置（大于列表长度，则代表不移除任何）
"""


# rpoplpush(src, dst) 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它
# 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
"""
从一个列表取出最右边的元素，同时将其添加至另一个列表的最左边
参数：
src，要取数据的列表的name
dst，要添加数据的列表的name
"""

###记住
# blpop(keys, timeout) 移除并获取列表的第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
print(conn.blpop('list_test'))


# brpop(keys, timeout) 移除并获取列表的最后一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
print(conn.brpop('list_test'))
```

### `Redis`其他通用操作

```python
from redis import Redis

conn=Redis()

# delete(key) 该命令用于key存在时删除key，可以删除一个或多个
conn.delete('name','age','test')

# exists(key)：检查指定key是否存在
res=conn.exists('hash_test','hash_test1')
print(res) # 返回2，基本上是判断一个，1代表在，0代表不在

# keys(pattern='*') 查找所有符合给定模式(pattern)的 key
res=conn.keys('h?sh_test')  # 慎用
res=conn.keys('hash_*')  # 慎用
print(res)

"""
更多：
KEYS * 匹配数据库中所有 key 。
KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
KEYS h*llo 匹配 hllo 和 heeeeello 等。
KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo
"""

# expire(name, time) 指定某个key设置过期时间，以秒为单位
conn.expire('hash_test',5) # 五秒过后自动销毁


# rename(src, dst) 修改key的名称（对redis的key重命名）
conn.rename('hash_test1','xxx')


# move(key, db)) 将当前数据库的key移动到指定的数据库db当中
conn.move('xxx',1)  # 将db0库中的xxx移动到db1库下


# randomkey() 从当前数据库中随机返回一个key（不删除）
print(conn.randomkey())


# type(key)：返回key所储存的值的类型
print(conn.type('name'))  # b'string'
print(conn.type('list'))  # b'list'
```

### `Redis`的事务(管道)

```python
"""
1 非关系型数据库，本身不支持事务
2 redis中的管道可以实现事务的支持（要么都成功，要么都失败）
    -实现的原理：多条命令放到一个管道中，一次性执行
3 具体代码
4 如果是集群环境，不支持管道，只有单机才支持
redis当中如何实现事务？
  使用redis管道，具体实现：开启管道，把命令放进去
  调用execute依次执行管道中的所有命令，它的原理就
  是要么一次性都执行，要么都不执行，保证了事务
"""
import redis
pool = redis.ConnectionPool()
r = redis.Redis(connection_pool=pool)
pipe = r.pipeline(transaction=True)  # 开启事务
pipe.multi()    # 管道等待放入多条命令
pipe.set('name', 'xxx')
pipe.set('role', 'admin')

# 到此，命令都没有执行
pipe.execute()  # 执行管道中的所有命令
```

### `Django`使用`Redis`

#### 直接使用

```python
# 写一个pool连接池 放在 luffy_api/utils/redis_pool.py
from redis import ConnectionPool
POOL = ConnectionPool(max_connections=5, host='127.0.0.1', port=6379)


# 在使用的位置，获取连接，获取数据
import redis
from rest_framework.views import APIView
from rest_framework.response import Response
from utils.redis_pool import POOL

class TestView(APIView):
    def get(self, requeste):
        conn = redis.Redis(connection_pool=POOL)
        print(conn.get('name'))
        res = 'hello world'
        print(res)
        return Response('ok')
```

#### `Django`使用

```python
# 安装
pip install django-redis

# 在配置文件中配置 luffy_api/setting/dev.py
# redis 配置
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",  #这句指的是django中的缓存也缓存到redis中了
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}   #连接池的大小
            # "PASSWORD": "123",
        }
    }
}


# 在使用位置
from django_redis import get_redis_connection
conn=get_redis_connection()
print(conn.get('name'))


# 一旦这么配置了，以后django的缓存也缓存到reids中了
cache.set('asdfasd','asdfas')

# 以后在django中，不用使用redis拿连接操作了，直接用cache做就可以了
# 不需要关注设置的值类型是什么
cache.set('wife',['dlrb','lyf']) # value值可以放任意数据类型

'''
底层原理，把value通过pickle转成二进制，以redis字符串的形式存到了redis中
pickle是python独有的序列化和反序列化，只能python玩，把python中所有数据类型都能转成二进制
通过二进制可以在反序列化成功pyhton中的任意对象
'''
```
