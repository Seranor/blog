---
title: python-并发编程
lastmod: 2021-06-21T16:43:23+08:00
date: 2021-06-21T11:52:03+08:00
tags:
  - Python
categories:
  - Python
url: post/python-25.html
toc: true
---

## 操作系统相关

> 操作系统就是一个协调、管理和控制计算机硬件资源和软件资源的控制程序

<!-- more -->

### 多道技术

```python
一 操作系统的作用：
    1：隐藏丑陋复杂的硬件接口，提供良好的抽象接口
    2：管理、调度进程，并且将多个进程对硬件的竞争变得有序

二 多道技术：
    1.产生背景：针对单核，实现并发
    ps：
    现在的主机一般是多核，那么每个核都会利用多道技术
    有4个cpu，运行于cpu1的某个程序遇到io阻塞，会等到io结束再重新调度，会被调度到4个
    cpu中的任意一个，具体由操作系统调度算法决定。

    2.空间上的复用：如内存中同时有多道程序
    3.时间上的复用：复用一个cpu的时间片

强调：CPU遇到I/O切，占用CPU时间过长也切，核心在于切之前将进程的状态保存下来，这样
     才能保证下次切换回来时，能基于上次切走的位置继续运行
```

### 进程相关

- 进程:程序运行的过程，是一个动态的概念
- 程序:是一系列的代码文件，是一个静态的概念

### 并发、并行和串行

- 并发:是伪并行，多个任务看起来同时运行，单个 CPU+多道技术就可以实现并发(并行也属于并发)
- 并行:多个任务真正意义上的同时运行，只有具备多个 CPU 才能实现并行
- 串行:一个任务运行完毕后才能开启下一个任务

![9Ihf9e](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/9Ihf9e.jpg)

### 提交任务的两种方式

- 同步:发出一个功能调用时，在没有得到结果之前，该调用就不会返回

- 异步:当一个异步功能调用发出之后,调用者不能立刻得到结果，当该异步功能完成后，通过状态、通知或回调来通知调用者

### 一个任务运行的三种状态

- 运行态:当前进程正在被 CPU 执行

- 阻塞态:正在执行的进程，由于等待某个事件而无法执行时，如遇到 I/O

- 就绪态:当前进程没有被 CPU 执行

![RqtvS8](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/RqtvS8.jpg)

## multiprocessing 模块

python 中的多线程无法利用多核优势(`os.cpu_count()`查看)，在 python 大部分情况使用多进程，python 提供了 multipprocessing 模块

multiprocess 模块功能众多，支持子进程、通信和共享数据、执行不同形式的同步，提供了 Process、Queue、Pipe、Lock 等组件

与线程不同，进程没有任何共享状态，进程修改的数据，改动仅限于该进程内

### Process 类

#### 介绍

##### 创建进程的类

```python
# 由改类实例化的对象，表示一个子进程中的任务，还没有启动
Process([group [,target [, name [, args [, kwargs]]]]])

# 1. 需要使用关键字的方式来指定参数
# 2. args指定的为传给target函数的位置参数，是一个元组形式，必须有逗号
```

##### 参数介绍

```python
group  参数未使用，值始终为None

target  表示调用对象，即子进程要执行的任务

agrs  表示调用对象位置参数，是一个元组，agrs=(1,)

kwargs  表示调用对象的字典，kwargs={'name': 'xxx'}

name  表示子进程的名称
```

##### 方法介绍

```python
p.start()  启动进程

p.run()  进程启动时的运行方法，正是它去调用target指定的函数，我们自定义类的类中一定要实现该方法

p.terminate()  强制终止进程p，不会进行任何清理操作，如果p创建了子进程，该子进程就成了僵尸进程，
    		  使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁

p.is_alive()  判断是否运行，值为True或False

p.join([timeout])  主线程等待p终止（强调：是主线程处于等的状态，而p是处于运行的状态）。timeout是
      可选的 超时时间，需要强调的是，p.join只能join住start开启的进程，而不能join住run开启的进程
```

##### 属性介绍

```python
p.daemon  默认值为False，如果设为True，代表p为后台运行的守护进程，当p的父进程终止时，p也随之终止，并且此时					p不能创建自己的新进程，必须在p.start()之前设置

p.name  进程名称

p.pid  进程的pid

p.exitcode  进程在运行时为None、如果为–N，表示被信号N结束

p.authkey  进程的身份验证键,默认是由os.urandom()随机生成的32字符的字符串。这个键的用途是为涉及网络连接的        底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功
```

#### 使用

> 在 Windows 中`Process()`必须放到`if name == 'main':`下

##### 开启进程方式一

```python
from multiprocessing import Process
import os
import time


def task(n):
    print('父进程: %s , 自己进程 %s 正在运行' % (os.getppid(), os.getpid()))
    time.sleep(n)
    print('父进程: %s , 自己进程 %s 正在运行' % (os.getppid(), os.getpid()))


if __name__ == '__main__':
    p = Process(target=task, args=(3,))
    p.start()
    print('主进程 %s ' % os.getpid())
```

##### 开启进程方式二

```python
from multiprocessing import Process
import os
import time

class MyProcess(Process):
    def __init__(self, n):
        super().__init__()
        self.n = n

    def run(self) -> None:
        print('父进程 %s , 自己 %s 正在运行' % (os.getppid(), os.getpid()))
        time.sleep(self.n)
        print('父进程 %s , 自己 %s 正在运行' % (os.getppid(), os.getpid()))


if __name__ == '__main__':
    p = MyProcess(3)
    p.start()
    print('主进程 %s ' % os.getpid())
```

> `os.getpid()`获取当前进程 pid
>
> `os.getppid()`获取当前进程的父进程 pid

##### 进程之间的内存空间是隔离的

```python
from multiprocessing import Process
import os
import time

count = 100
def task():
    global n
    count = 0
    print('自己', count)

if __name__ == '__main__':
    p = Process(target=task)
    p.start()
    time.sleep(3)
    print('主 %s' % count)

# 0
# 100
```

##### 进程对象的方法

- `join() `

  ```python
  from multiprocessing import Process
  import os
  import time
  import random


  class MyProcess(Process):
      def __init__(self, name):
          super().__init__()
          self.name = name

      def run(self) -> None:
          print('%s 正在运行, 进程号是 %s' % (self.name, os.getpid()))
          time.sleep(random.randint(1, 3))
          print('%s 运行结束, 进程号是 %s ' % (self.name, os.getpid()))


  if __name__ == '__main__':
      p = MyProcess('p1')
      p.start()
      p.join()  # 保证子进程结束后才会向下执行，当前主线程处于等的状态，而p是处于运行的状态
      # p.join(2)  # 指定等待p子进程的时间，如果子进程p运行完直接往下执行，如果等了2s之后还没执行完也会向下执行
      print('开始 主进程 %s ' % os.getpid())
  ```

  该方法并不是串行：

  ```python
  from multiprocessing import Process
  import os
  import time
  import random


  class MyProcess(Process):
      def __init__(self, name):
          super().__init__()
          self.name = name

      def run(self) -> None:
          print('%s 正在运行, 进程号是 %s' % (self.name, os.getpid()))
          time.sleep(random.randint(1, 3))
          print('%s 运行结束, 进程号是 %s ' % (self.name, os.getpid()))


  if __name__ == '__main__':
      p1 = MyProcess('p1')
      p2 = MyProcess('p2')
      p3 = MyProcess('p3')
      p4 = MyProcess('p4')
      p5 = MyProcess('p5')

      # 这几个进程是差不多一起一起的,并不是启动一个执行完之后再运行第二个进程,是让主进程等,而不是让后面的子进程等
      p1.start()
      p2.start()
      p3.start()
      p4.start()
      p5.start()

      # p_list = [p1, p2, p3, p4, p5]
      # for p in p_list:
      #     p.start()

      # 但是当 p1 执行完成后确实要等后面的 p2-p5 进程执行完成后才能继续往后
      p1.join()
      p2.join()
      p3.join()
      p4.join()
      p5.join()

  		#for p in p_list:
      #    p.join()

      print('主进程 %s ' % os.getpid())
  ```

- `terminate()和is_alive()`

  ```python
  from multiprocessing import Process
  import os
  import time
  import random

  def task(name):
      print('%s is run, task is %s ' % (name, os.getppid()))
      time.sleep(random.randint(1, 3))
      print('%s is end,task is %s ' % (name, os.getpid()))


  if __name__ == '__main__':
      p = Process(target=task, args=('test',))
      p.start()
      p.terminate()  # 关闭进程，不会立即关闭
      print(p.is_alive())  # 所以此时查看进程是否存活时为True
      print('main is start ')
      print(p.is_alive())  # 子进程已经关闭了，此时为False
  ```

- `name和pid`

  ```python
  class MyProcess(Process):
      def __init__(self, name):

          # self.name=name
          # super().__init__() #Process的__init__方法会执行self.name=Piao-1,
          #                    #所以加到这里,会覆盖我们的self.name=name

          #为我们开启的进程设置名字的做法
          super().__init__()
          self.name = name

      def run(self) -> None:
          print('%s is run' % self.name)
          time.sleep(random.randint(1, 3))
          print('%s is end' % self.name)


  if __name__ == '__main__':
      p = MyProcess('test')
      p.start()

      print('main is run')
      print(p.pid)  # 查看pid
  ```

## 进程相关

> 参考: https://www.cnblogs.com/Anker/p/3271773.html

### 僵尸进程

```python
僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程
僵尸进程虽然不会占用内存CPU等系统资源，但是PID号如果过多之后，操作系统也无法创建新PID号
```

- 产生僵尸进程

  ```python
  from multiprocessing import Process
  import os
  import time


  def run():
      print('子', os.getpid())


  if __name__ == '__main__':
      p = Process(target=run)
      p.start()
      print('主', os.getpid())
      time.sleep(1000)
  ```

- 查看僵尸进程
  ![sfHAIh](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/sfHAIh.png)

  ```bash
  ps aux|grep Z  # SATA 显示 Z 就是僵尸进程
  ```

- 解决办法

  ```python
  1. 杀死父进程
  			kill -CHLD 父进程的pid
    		kill -9 父进程的pid
  2. 对开启的子进程应该记得使用join，join会回收僵尸进程
  3. https://blog.csdn.net/u010571844/article/details/50419798
  ```

- 问题

  ```python
  from multiprocessing import Process
  import time,os

  def task():
      print('%s is running' %os.getpid())
      time.sleep(3)

  if __name__ == '__main__':
      p=Process(target=task)
      p.start()
      p.join() # 等待进程p结束后，join函数内部会发送系统调用wait，去告诉操作系统回收掉进程p的id号

      print(p.pid) #？？？此时能否看到子进程p的id号
      print('主')

  # p.join()是像操作系统发送请求，告知操作系统p的id号不需要再占用了，回收就可以，
  # 此时在父进程内还可以看到p.pid,但此时的p.pid是一个无意义的id号，因为操作系统已经将该编号回收
  ```

### 孤儿进程

```python
当父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程，由于进程不可能脱离进程树而独立存在，孤儿进程将被PID为1的init进程所收养，并由init进程对它们完成状态收集工作。孤儿进程被收养后进行正常的释放，没有危害
```

- 演示代码

  ```python
  from multiprocessing import Process
  import os
  import time


  def run():
      print('子', os.getpid())
      time.sleep(50)


  if __name__ == '__main__':
      p1 = Process(target=run)
      p2 = Process(target=run)
      p1.start()
      p2.start()
      print('主', os.getpid())
  ```

- 现象

  ![image-20211216191021094](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/image-20211216191021094.png)

  两个子进程并没有退出，此时两个子进程的父进程由 1 接管，当时间久了之后会被释放掉

### 守护进程

主进程创建守护进程

1. 守护进程会在主进程代码执行结束后就终止
2. 守护进程内无法再开启子进程，否则抛出异常: AssertionError: daemonic processes are not allowed to have children

注意：进程之间是互相独立的，主进程代码运行结束，守护进程随即终止

- 实例代码

  ```python
  import random
  from multiprocessing import Process
  import os
  import time


  class MyProcess(Process):
      def __init__(self, name):
          super().__init__()
          self.name = name

      def run(self) -> None:
          print('%s is run' % self.name)
          time.sleep(random.randint(1, 3))
          print('%s is end ' % self.name)


  p1 = MyProcess('p1')
  p1.daemon = True  # 一定要在p.start()前设置,设置p为守护进程,禁止p创建子进程,并且父进程代码执行结束,p即终止运行

  p1.start()
  print('main is run', os.getpid())

  # 结果：main is run 可以看到子线程没有执行
  ```

### 互斥锁

> 进程之间数据不共享,但是共享同一套文件系统,所以访问同一个文件,或同一个打印终端,是没有问题的
>
> 而共享带来的是竞争，竞争带来的结果就是错乱，如何控制，就是加锁处理

#### 代码一

没加锁的情况

```python
from multiprocessing import Process
import os, time


def work():
    print('%s is run ' % os.getpid())
    time.sleep(2)
    print('%s is end ' % os.getpid())


if __name__ == '__main__':
    for i in range(3):
        p = Process(target=work)
        p.start()
# 并发运行,效率高,但竞争同一打印终端,带来了打印错乱
```

加锁之后的情况

```python
#由并发变成了串行,牺牲了运行效率,但避免了竞争
from multiprocessing import Process, Lock
import os, time


def work(lock):
    lock.acquire()
    print('%s is run ' % os.getpid())
    time.sleep(2)
    print('%s is end ' % os.getpid())
    lock.release()


if __name__ == '__main__':
    lock = Lock()
    for i in range(3):
        p = Process(target=work, args=(lock,))
        p.start()
```

#### 代码二

文件当数据库,模拟抢票

不加锁的情况

```python
# 并发运行，效率高，但是在竞争一个文件，数据写入错乱
def search():
    with open('db.json', 'r', encoding='utf8') as f:
        dic = json.load(f)
    print('\033[43m剩余票数%s\033[0m' % dic['count'])


def get():
    dic = json.load(open('db.json'))
    time.sleep(0.1)
    if dic['count'] > 0:
        dic['count'] -= 1
        time.sleep(0.2)
        json.dump(dic, open('db.json', 'w'))
        print('\033[43m购票成功\033[0m')


def task(lock):
    search()
    get()


if __name__ == '__main__':
    lock = Lock()
    for i in range(10):
        p = Process(target=task, args=(lock,))
        p.start()
```

加锁之后

```python
# 查票还是并发，但是在购票的时候由并发变成了串行，牺牲了运行效率，但保证了数据安全
def search():
    with open('db.json', 'r', encoding='utf8') as f:
        dic = json.load(f)
    print('\033[43m剩余票数%s\033[0m' % dic['count'])


def get():
    dic = json.load(open('db.json'))
    time.sleep(0.1)
    if dic['count'] > 0:
        dic['count'] -= 1
        time.sleep(0.2)
        json.dump(dic, open('db.json', 'w'))
        print('\033[43m购票成功\033[0m')
    else:
        print('没票了')


def task(lock):
    search()
    lock.acquire()
    get()
    lock.release()


if __name__ == '__main__':
    lock = Lock()
    for i in range(5):
        p = Process(target=task, args=(lock,))
        p.start()
```

#### 总结

```python
#加锁可以保证多个进程修改同一块数据时，同一时间只能有一个任务可以进行修改，即串行的修改，没错，速度是慢了，但牺牲了速度却保证了数据安全。
虽然可以用文件共享数据实现进程间通信，但问题是：
1.效率低（共享数据基于文件，而文件是硬盘上的数据）
2.需要自己加锁处理


#因此我们最好找寻一种解决方案能够兼顾：1、效率高（多个进程共享一块内存的数据）2、帮我们处理好锁问题。这就是mutiprocessing模块为我们提供的基于消息的IPC通信机制：队列和管道。
1 队列和管道都是将数据存放于内存中
2 队列又是基于（管道+锁）实现的，可以让我们从复杂的锁问题中解脱出来，
我们应该尽量避免使用共享数据，尽可能使用消息传递和队列，避免处理复杂的同步和锁问题，而且在进程数目增多时，往往可以获得更好的可获展性。
```

### IPC 机制

进程彼此之间互相隔离，要实现进程之间通信（IPC），multiprocessing 模块支持两种形式：队列和管道，这两种方式都是使用消息传递的

- 管道

  ps -ef |grep xx 前面的进程产生的数据交给后面的进程

#### 队列

> 底层就是以管道和锁定的方式实现

创建队列的类

```
Queue([maxsize]):创建共享的进程队列，Queue是多进程安全的队列，可以使用Queue实现多进程之间的数据传递
maxsize是队列中允许最大项数，省略则无大小限制
```

主要方法

```python
q.put方法用以插入数据到队列中，put方法还有两个可选参数：blocked和timeout。blocked为True（默认值）如果
队列满了就锁住了并且timeout为正值，该方法会阻塞timeout指定的时间，直到该队列有剩余的空间。如果超时，会
抛出Queue.Full异常。如果blocked为False，但该Queue已满，会立即抛出Queue.Full异常。

q.get方法可以从队列读取并且删除一个元素。同样，get方法有两个可选参数：blocked和timeout。如果blocked
为True（默认值），并且timeout为正值，那么在等待时间内没有取到任何元素，会抛出Queue.Empty异常。如果
blocked为False有两种情况存在，如果Queue有一个值可用，则立即返回该值，否则，如果队列为空，则立即抛出
Queue.Empty异常.

q.get_nowait():同q.get(False)

q.put_nowait():同q.put(False)

q.empty():调用此方法时q为空则返回True，该结果不可靠，比如在返回True的过程中，如果队列中又加入了项目。

q.full()：调用此方法时q已满则返回True，该结果不可靠，比如在返回True的过程中，如果队列中的项目被取走。

q.qsize():返回队列中目前项目的正确数量，结果也不可靠，理由同q.empty()和q.full()一样
```

其他方法

```python
q.cancel_join_thread():不会在进程退出时自动连接后台线程。可以防止join_thread()方法阻塞

q.close():关闭队列，防止队列中加入更多数据。调用此方法，后台线程将继续写入那些已经入队列但尚未写入的
          数据，但将在此方法完成时马上关闭。如果q被垃圾收集，将调用此方法。关闭队列不会在队列使用者中
          产生任何类型的数据结束信号或异常。例如，如果某个使用者正在被阻塞在get()操作上，关闭生产者中
          的队列不会导致get()方法返回错误。

q.join_thread()：连接队列的后台线程。此方法用于在调用q.close()方法之后，等待所有队列项被消耗。默认
                 情况下，此方法由不是q的原始创建者的所有进程调用。调用q.cancel_join_thread方法可
                 以禁 止这种行为
```

应用

```python
from multiprocessing import Process, Queue
import time

q = Queue(3)  # 创建共享的进程队列，指定队列长度为3，最多放三个值，超过3个无法放入
q.put(1)
q.put(2)
q.put(3)
print(q.full())

print(q.get())
print(q.get())
print(q.get())
print(q.empty())
# print(q.get())  # 超值取不到q.get()默认为 q.get(block=True,timeout=None)
# print(q.get(block=True,timeout=3)) # 取不到三秒抛出异常
print(q.get(block=False))  # 取不到值立马抛异常
```

### 生产者消费者模型

> 在并发编程中使用生产者和消费者模式能够解决绝大多数并发问题。该模式通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度

- 为什么要使用生产者和消费者模式

  - 在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式

- 什么是生产者消费者模式

  - 生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力

- 总结

  ```python
      #程序中有两类角色
          一类负责生产数据（生产者）
          一类负责处理数据（消费者）

      #引入生产者消费者模型为了解决的问题是
          平衡生产者与消费者之间的工作能力，从而提高程序整体处理数据的速度

      #如何实现
          生产者<-->队列<——>消费者
      #生产者消费者模型实现类程序的解耦和
  ```

  基于队列实现生产者消费者模型

  ```python
  from multiprocessing import Process, Queue
  import time, os, random


  def producer(q, name, courier):
      for i in range(3):
          res = '%s  %s ' % (courier, i)
          time.sleep(random.randint(1, 3))
          q.put(res)
          print('%s 送来 %s ' % (name, res))
      q.put(None)  # 结束之后发送None信息到队里里面，有几个消费者就发几个None
      q.put(None)


  def consumer(q, name):
      while True:
          res = q.get()
          if res is None:
              break
          time.sleep(random.randint(1,3))
          print('%s 拿到了 %s' % (name, res))


  if __name__ == '__main__':
      q = Queue()

      p1 = Process(target=producer, args=(q, '快递员1', 'sf'))
      p2 = Process(target=producer, args=(q, '快递员2', 'yz'))
      p3 = Process(target=producer, args=(q, '快递员3', 'jd'))

      c1 = Process(target=consumer, args=(q, '拿货人1'))
      c2 = Process(target=consumer, args=(q, '拿货人2'))

      p1.start()
      p2.start()
      p3.start()

      c1.start()
      c2.start()

      print('%s is run ' % os.getpid())
  ```

**JoinableQueue([maxsize])`**

> 这就像是一个 Queue 对象，但队列允许项目的使用者通知生成者项目已经被成功处理。通知进程是使用共享的信号和条件变量来实现的。

介绍

```python
 #参数介绍：
    maxsize是队列中允许最大项数，省略则无大小限制。
　 #方法介绍：
   JoinableQueue的实例p除了与Queue对象相同的方法之外还具有：
   q.task_done()：使用者使用此方法发出信号，表示q.get()的返回项目已经被处理。如果调用此方法的次数大于从队列中删除项目的数量，将引发ValueError异常
   q.join():生产者调用此方法进行阻塞，直到队列中所有的项目均被处理。阻塞将持续到队列中的每个项目均调用q.task_done（）方法为止
```

优化上面队列代码

```python
from multiprocessing import Process, JoinableQueue
import time, os, random


def producer(q, name, courier):
    for i in range(3):
        res = '%s  %s ' % (courier, i)
        time.sleep(random.randint(1, 3))
        q.put(res)
        print('%s 送来 %s ' % (name, res))
    q.join()


def consumer(q, name):
    while True:
        res = q.get()
        time.sleep(random.randint(1, 3))
        print('%s 拿到了 %s' % (name, res))
        q.task_done()


if __name__ == '__main__':
    q = JoinableQueue()

    p1 = Process(target=producer, args=(q, '快递员1', 'sf'))
    p2 = Process(target=producer, args=(q, '快递员2', 'yz'))
    p3 = Process(target=producer, args=(q, '快递员3', 'jd'))

    c1 = Process(target=consumer, args=(q, '拿货人1'))
    c2 = Process(target=consumer, args=(q, '拿货人2'))
    c1.daemon = True  # 主进程结束顺便带走了守护进程
    c2.daemon = True

    p_l = [p1, p2, p3, c1, c2]
    for p in p_l:
        p.start()

    p1.join()
    p2.join()
    p3.join()  # p1、p2、p3都结束，代表队列一定被取空

    print('%s is run ' % os.getpid())


#主进程等--->p1,p2,p3等---->c1,c2
#p1,p2,p3结束了,证明c1,c2肯定全都收完了p1,p2,p3发到队列的数据
#因而c1,c2也没有存在的价值了,应该随着主进程的结束而结束,所以设置成守护进程
```

### 信号量

> 互斥锁 同时只允许一个线程更改数据，而 Semaphore 是同时允许一定数量的线程更改数据 ，比如厕所有 3 个坑，那最多只允许 3 个人上厕所，后面的人只能等里面有人出来了才能再进去，如果指定信号量为 3，那么来一个人获得一把锁，计数加 1，当计数等于 3 时，后面的人均需要等待。一旦释放，就有人可以获得一把锁
>
> 信号量与进程池的概念很像，但是要区分开，信号量涉及到加锁的概念

```python
from multiprocessing import Process, Semaphore
import os, time, random


def go_wc(sem, user):
    sem.acquire()  # 运行的时候都会抢这把锁
    print('%s 占到一个茅坑' % user)
    time.sleep(random.randint(1, 3))
    sem.release()


if __name__ == '__main__':
    sem = Semaphore(5)  # 创建信号量，自定义为5，相当于5把钥匙得到信号量对象
    p_l = []
    for i in range(10):
        p = Process(target=go_wc, args=(sem,'user%s' % i,))
        p.start()
        p_l.append(p)

    for i in p_l:
        i.join()

"""
ps：互斥锁只能acquire一次，再有人来执行acquire，如果没有释放，下一个来拿的人就只能阻在原地无法拿到acquire。而信号量一把锁可以acquire指定5次（Semaphore(5)），如果第6个来在
acquire的时候就没有了，相当于没有钥匙了，就只能在原地等着，只要5个人里面有人释放后面的人就
可以拿到钥匙

"""
```

## 线程相关

线程是进程内代码运行的过程，线程是一个执行单位，CPU 执行的就是线程。进程是一个资源单位

线程和进程的区别

1. 同一进程下的多个线程共享该进程的内存资源，线程之间可以互相通信
2. 开启子线程的开销要远远小于开启子线程

**线程相关的方法**

```python
Thread实例对象的方法
  # isAlive(): 返回线程是否活动的。
  # getName(): 返回线程名。
  # setName(): 设置线程名。

threading模块提供的一些方法：
  # threading.currentThread(): 返回当前的线程变量。
  # threading.enumerate(): 返回一个包含正在运行的线程的list。正在运行指线程启动后、结束前，不包括启动前和终止后的线程。
  # threading.activeCount(): 返回正在运行的线程数量，与len(threading.enumerate())有相同的结果。
```

### 开启线程的两种方式

- 方式一

  ```python
  from threading import Thread, current_thread
  def task():
      print('%s is running ' % current_thread().name)


  if __name__ == '__main__':
      t = Thread(target=task)
      t.start()
      print('主线程', current_thread().name)
  ```

- 方式二

  ```python
  from threading import Thread, current_thread
  class MyThread(Thread):
      def __init__(self):
          super().__init__()

      def run(self) -> None:
          print('%s is running ' % current_thread().name)  # 打印当前线程名


  if __name__ == '__main__':
      t = MyThread()
      t.start()
      print('主线程', current_thread().name)
  ```

![ggLKKR](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/ggLKKR.jpg)

- 线程之间数据相互影响

```python
from threading import Thread, current_thread
n = 100
def task():
    global n
    n = 0
if __name__ == '__main__':
    t = Thread(target=task)
    t.start()
    t.join()  # 让线程运行完成，避免有可能出现主线程先打印 n 的情况
    print(n)
```

### 守护线程

> **无论是进程还是线程，都遵循：守护 xxx 会等待主 xxx 运行完毕后被销毁**
>
> **需要强调的是：运行完毕并非终止运行**

```
1. 对主进程来说，运行完毕指的是主进程代码运行完毕
2. 对主线程来说，运行完毕指的是主线程所在的进程内所有非守护线程统统运行完毕，主线程才算运行完毕
```

```
1. 主进程在其代码结束后就已经算运行完毕了（守护进程在此时就被回收）,然后主进程会一直等非守护的子进程都运行完毕后回收子进程的资源(否则会产生僵尸进程)，才会结束
2. 主线程在其他非守护线程运行完毕后才算运行完毕（守护线程在此时就被回收）。因为主线程的结束意味着进程的结束，进程整体的资源都将被回收，而进程必须保证非守护线程都运行完毕后才能结束
```

代码案例

```python
from threading import Thread, current_thread
def task(n):
    print('%s is running' % current_thread().name)
    time.sleep(n)
    print('%s is end' % current_thread().name)


if __name__ == '__main__':
    t1 = Thread(target=task, args=(2,))
    t2 = Thread(target=task, args=(3,))
    t3 = Thread(target=task, args=(300,))
    t3.daemon = True  # t3最后的end并没有执行
    t1.start()
    t2.start()
    t3.start()
    print('主')  # 主线程要等子线程执行完后才结束
```

### 互斥锁

现象:

```python
from threading import Thread, current_thread
import time
n = 100


def task():
    global n
    temp = n
    time.sleep(0.1)  # 线程速度太快了，如果不加sleep能减完，但是处理速度如果慢的情况下就会数据错乱
    n = temp - 1


if __name__ == '__main__':
    thread_l = []
    for i in range(100):
        t = Thread(target=task)
        thread_l.append(t)
        t.start()
    for obj in thread_l:
        obj.join()
    print(n)  # 99

```

加锁

```python
from threading import Thread, current_thread,Lock
import time

n = 100
mutex = Lock()


def task():
    global n
    with mutex:
        temp = n
        time.sleep(0.1)
        n = temp - 1


if __name__ == '__main__':
    thread_l = []
    start_time = time.time()
    for i in range(100):
        t = Thread(target=task)
        thread_l.append(t)
        t.start()
    for obj in thread_l:
        obj.join()
    end_time = time.time()
    print('结果是 %s, 运行时间: %s ' % (n, end_time - start_time))  # 结果是 0, 运行时间: 10.33482813835144
```

### 信号量

```python
import random
import threading
from threading import Thread, Semaphore
import time

def func():
    sm.acquire()
    print('%s get sm' % threading.current_thread().getName())
    time.sleep(random.randint(1,3))
    sm.release()


if __name__ == '__main__':
    sm = Semaphore(5)
    for i in range(23):
        t = Thread(target=func)
        t.start()
```

### Event

同进程的一样

线程的一个关键特性是每个线程都是独立运行且状态不可预测。如果程序中的其 他线程需要通过判断某个线程的状态来确定自己下一步的操作,这时线程同步问题就会变得非常棘手。为了解决这些问题,我们需要使用 threading 库中的 Event 对象。 对象包含一个可由线程设置的信号标志,它允许线程等待某些事件的发生。在 初始情况下,Event 对象中的信号标志被设置为假。如果有线程等待一个 Event 对象, 而这个 Event 对象的标志为假,那么这个线程将会被一直阻塞直至该标志为真。一个线程如果将一个 Event 对象的信号标志设置为真,它将唤醒所有等待这个 Event 对象的线程。如果一个线程等待一个已经被设置为真的 Event 对象,那么它将忽略这个事件, 继续执行

```python
event.isSet()：返回event的状态值；

event.wait()：如果 event.isSet()==False将阻塞线程；

event.set()： 设置event的状态值为True，所有阻塞池的线程激活进入就绪状态， 等待操作系统调度；

event.clear()：恢复event的状态值为False
```

![yFh1Wi](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/yFh1Wi.jpg)

**案例代码一**

```python
from threading import Event, Thread, current_thread
import time

e = Event()  # 全局变量为False


def f1():
    print('%s is running ' % current_thread().name)
    time.sleep(3)
    e.set()  # 全局变量为True
    # e.clear()  # 全局变量 = False
    # e.is_set()  # 判断是否set过


def f2():
    e.wait()  # 等全局变量变为True
    print('%s is running ' % current_thread().name)


if __name__ == '__main__':
    t1 = Thread(target=f1)
    t2 = Thread(target=f2)

    t1.start()
    t2.start()
```

**模拟红绿灯**

```python
from threading import Event, Thread, current_thread
import time, random
e = Event()


def task1():
    while True:
        e.clear()
        print('红灯亮了')
        time.sleep(3)

        e.set()
        print('绿灯亮了')
        time.sleep(4)


def task2():
    while True:
        if e.is_set():
            print('可以走了 %s' % current_thread().name)
            break
        else:
            print('正在等待 %s' % current_thread().name)
            e.wait()


if __name__ == '__main__':
    Thread(target=task1).start()

    while True:
        time.sleep(random.randint(1, 2))
        Thread(target=task2).start()
```

### 定时器

> 定时器 Timer 类是 Thread 的派生类，用于在指定时间后调用一个方法。

指定 n 秒后执行某操作

```python
from threading import Timer

def hello(n):
    print('hello world',n)


t = Timer(3, hello, args=(1111,))  # 3秒之后执行
t.start()
```

### 线程 queue

> queue 队列 ：使用 import queue，用法与进程 Queue 一样
>
> 当信息必须在多个线程之间安全交换时，队列在线程编程中特别有用

- 基本方法

  ```python
  put 往线程队列里防止,超过队列长度,直接阻塞
  get 从队列中取值,如果获取不到,直接阻塞
  put_nowait: 如果放入的值超过队列长度,直接报错（linux）
  get_nowait: 如果获取的值已经没有了,直接报错
  ```

- 用法

  ```python
  import queue

  # 队列：先进先出
  q = queue.Queue(3) # 指定队列的大小
  q.put(111)  # 整型
  q.put("aaa") # 字符串
  q.put((1,2,3)) # 元组

  print(q.get())
  print(q.get())
  print(q.get())

  '''
  111
  aaa
  (1, 2, 3)

  '''

  # 堆栈：后进先出
  q = queue.LifoQueue(3)
  q.put(111)
  q.put("aaa")
  q.put((1,2,3))

  print(q.get())
  print(q.get())
  print(q.get())

  '''
  (1, 2, 3)
  aaa
  111

  '''

  # 优先级队列：
  # 1.默认按照数字大小排序,然后会按照ascii编码在从小到大排序
  # 2.先写先排,后写后排
  q = queue.PriorityQueue(3)
  q.put((10,111))  # 第一个值是优先级，第二值才是要放的元素
  q.put((11,"aaa"))
  q.put((-1,(1,2,3)))

  print(q.get())
  print(q.get())
  print(q.get())

  '''
  (-1, (1, 2, 3))  # 数越小优先级越高
  (10, 111)
  (11, 'aaa')

  '''
  ```

### 死锁和递归锁

> 死锁是指两个或两个以上的进程或线程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程

- 代码演示

  ```python
  from threading import Thread, Lock
  import time

  mutexA = Lock()
  mutexB = Lock()


  class MyThread(Thread):
      def __init__(self, name):
          super().__init__()
          self.name = name

      def f1(self):
          mutexA.acquire()
          print('%s 抢到了A锁 ' % self.name)

          mutexB.acquire()
          print('%s 抢到了B锁 ' % self.name)
          mutexB.release()

          mutexA.release()

      def f2(self):
          mutexB.acquire()
          print('%s 抢到了B锁 ' % self.name)
          time.sleep(0.1)

          mutexA.acquire()
          print('%s 抢到了A锁 ' % self.name)
          mutexA.release()

          mutexB.release()

      def run(self) -> None:
          self.f1()
          self.f2()


  if __name__ == '__main__':
      t1 = MyThread('线程1')
      t2 = MyThread('线程2')
      t3 = MyThread('线程3')
      t4 = MyThread('线程4')

      t1.start()
      t2.start()
      t3.start()
      t4.start()
      print('主线程')

  # 线程1 抢到了A锁
  # 线程1 抢到了B锁
  # 线程1 抢到了B锁
  # 线程2 抢到了A锁
  # 主线程
  # 此时卡在这了
  ```

- 解决方法

  > 递归锁，在 Python 中为了支持在同一线程中多次请求同一资源，python 提供了可重入锁 RLock
  >
  > 这个 RLock 内部维护着一个 Lock 和一个计数（counter）变量，计数记录了 acquire 的次数，从而使得资源可以被多次 require。直到一个线程所有的 acquire 都被 release，其他的线程才能获得资源。上面的例子如果使用 RLock 代替 Lock，则不会发生死锁

  ```python
  from threading import Thread, Lock, RLock
  import time

  mutexA = mutexB = RLock()


  class MyThread(Thread):
      def __init__(self, name):
          super().__init__()
          self.name = name

      def f1(self):
          mutexA.acquire()
          print('%s 抢到了A锁 ' % self.name)

          mutexB.acquire()
          print('%s 抢到了B锁 ' % self.name)
          mutexB.release()

          mutexA.release()

      def f2(self):
          mutexB.acquire()
          print('%s 抢到了B锁 ' % self.name)
          time.sleep(0.1)

          mutexA.acquire()
          print('%s 抢到了A锁 ' % self.name)
          mutexA.release()

          mutexB.release()

      def run(self) -> None:
          self.f1()
          self.f2()


  if __name__ == '__main__':
      t1 = MyThread('线程1')
      t2 = MyThread('线程2')
      t3 = MyThread('线程3')
      t4 = MyThread('线程4')

      t1.start()
      t2.start()
      t3.start()
      t4.start()
      print('主线程')
  ```

### 多线程实现 TCP 并发

```python
# 服务端
import socket
from multiprocessing import Process
from threading import Thread

s = socket.socket()
s.bind(('127.0.0.1', 8080))
s.listen(5)


def task(sock):
    while True:
        try:
            res = sock.recv(1024)
            if len(res) == 0: break
            data = res.upper()
            sock.send(data)
        except Exception:
            break
        sock.close()


while True:
    sock, address = s.accept()
    print(address)
    t = Thread(target=task, args=(sock,))
    t.start()

# 客户端
import socket

c = socket.socket()
c.connect(('127.0.0.1', 8080))

while True:
    cmd = input('>>>:').strip()
    if len(cmd) == 0: continue
    c.send(cmd.encode('utf8'))
    data = c.recv(1024)
    print(data.decode('utf8'))

```

## GIL 全局解释器锁

### 介绍

> **GIL 的全称是：Global Interpreter Lock,意思就是全局解释器锁**

```
'''
定义：
In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple
native threads from executing Python bytecodes at once. This lock is necessary mainly
because CPython’s memory management is not thread-safe. (However, since the GIL
exists, other features have grown to depend on the guarantees that it enforces.)
'''
结论：在Cpython解释器中，同一个进程下开启的多线程，同一时刻只能有一个线程执行，无法利用多核优势
```

首先需要明确的一点是`GIL`并不是 Python 的特性，它是在实现 Python 解析器(CPython)时所引入的一个概念。就好比 C++是一套语言（语法）标准，但是可以用不同的编译器来编译成可执行代码。有名的编译器例如 GCC，INTEL C++，Visual C++等。Python 也一样，同样一段代码可以通过 CPython，PyPy，Psyco 等不同的 Python 执行环境来执行。像其中的 JPython 就没有 GIL。然而因为 CPython 是大部分环境下默认的 Python 执行环境。所以在很多人的概念里 CPython 就是 Python，也就想当然的把`GIL`归结为 Python 语言的缺陷。所以这里要先明确一点：GIL 并不是 Python 的特性，Python 完全可以不依赖于 GIL

GIL 本质就是一把互斥锁，既然是互斥锁，所有互斥锁的本质都一样，都是将并发运行变成串行，以此来控制同一时间内共享数据只能被一个任务所修改，进而保证数据安全

综上：

如果多个线程的 target=work，那么执行流程是

多个线程先访问到解释器的代码，即拿到执行权限，然后将 target 的代码交给解释器的代码去执行

解释器的代码是所有线程共享的，所以垃圾回收线程也可能访问到解释器的代码而去执行，这就导致了一个问题:对于同一个数据 100，可能线程 1 执行 x=100 的同时，而垃圾回收执行的是回收 100 的操作，解决这种问题没有什么高明的方法，就是加锁处理，如下图的 GIL，保证 python 解释器同一时间只能执行一个任务的代码

![WK4tt1](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/WK4tt1.jpg)

### GIL 与 Lock

只要在一个进程里就一定有 GIL 锁的存在，GIL 锁不能保证 python 数据的安全，它保证的是解释器级别（内存管理）的安全，也可以说是背后存在的一种机制。可以肯定的一点是：保护不同的数据的安全，就应该加不同的锁。

GIL 保护的是解释器级的数据，保护用户自己的数据则需要自己加锁处理，如下图：

![dRjCYL](https://klcc-img-1251900471.cos.ap-chengdu.myqcloud.com/img/dRjCYL.jpg)

```python
from threading import Thread,Lock
import time

mutex = Lock()
n = 100

def task():
    global n

    mutex.acquire()
    temp = n
    time.sleep(0.1)
    n = temp - 1
    mutex.release()

if __name__ == '__main__':
    l = []
    for i in range(100):
        t = Thread(target=task)
        l.append(t)
        t.start()

    for obj in l:
        obj.join()

    print(n)  # 结果肯定为0，由原来的并发执行变成串行，牺牲了执行效率保证了数据安全
'''
分析：
1. 100个线程去抢GIL锁，即抢执行权限
2. 肯定有一个线程先抢到GIL（暂且称为线程1），然后开始执行，一旦执行就会mutex.acquire()
3. 极有可能线程1还未运行完毕，就有另外一个线程2抢到GIL，然后开始运行，但线程2发现互斥锁   	  lock还未被线程1释放，于是阻塞，被迫交出执行权限，即释放GIL
4. 直到线程1重新抢到GIL，开始从上次暂停的位置继续执行，直到正常释放互斥锁lock，然后其他的
   线程再重复2 3 4的过程

'''
```

### GIL 与多线程

对计算来说，cpu 越多越好，但是对于 I/O 来说，再多的 cpu 也没用

对运行一个程序来说，随着 cpu 的增多执行效率肯定会有所提高（不管提高幅度多大，总会有所提高），这是因为一个程序基本上不会是纯计算或者纯 I/O，所以我们只能相对的去看一个程序到底是计算密集型还是 I/O 密集型，从而进一步分析 python 的多线程到底有无用武之地

场景：

```python
分析：
我们有四个任务需要处理，处理方式肯定是要玩出并发的效果，解决方案可以是：
方案一：开启四个进程
方案二：一个进程下，开启四个线程

单核情况下，分析结果:
　　如果四个任务是计算密集型，没有多核来并行计算，方案一徒增了创建进程的开销，方案二胜
　　如果四个任务是I/O密集型，方案一创建进程的开销大，且进程的切换速度远不如线程，方案二胜

多核情况下，分析结果：
　　如果四个任务是计算密集型，多核意味着并行计算，在python中一个进程中同一时刻只有一个线
　　程执行用不上多核，方案一胜
　　如果四个任务是I/O密集型，再多的核也解决不了I/O问题，方案二胜


结论：现在的计算机基本上都是多核，python对于计算密集型的任务开多线程的效率并不能带来多
大性能上的提升，甚至不如串行(没有大量切换)，但是，对于IO密集型的任务效率还是有显著提升的。
```

### 多线程性能测试

- 计算密集型

  ```python
  from multiprocessing import Process
  from threading import Thread
  import os, time


  def work():
      res = 0
      for i in range(100000000):
          res *= 1


  if __name__ == '__main__':
      l = []
      print(os.cpu_count())  # 查看cpu核数
      start_time = time.time()
      for i in range(8):
          p = Process(target=work)  # 进程 7.7s多
          # p = Thread(target=work)  # 线程 28s多
          l.append(p)
          p.start()
      for p in l:
          p.join()
      stop_time = time.time()
      print('run time is %s ' % (stop_time - start_time))
  ```

- I/O 密集型

  ```python
  from multiprocessing import Process
  from threading import Thread
  import os, time

  def work():
      time.sleep(2)


  if __name__ == '__main__':
      l = []
      # print(os.cpu_count()) # 查看CPU核数
      start = time.time()
      for i in range(1000):
          p=Process(target=work)  # 使用进程
          # p = Thread(target=work)  # 使用线程比进程效率稍高
          l.append(p)
          p.start()
      for p in l:
          p.join()
      stop = time.time()
      print('run time is %s' % (stop - start))
  ```

- 结论

  多线程用于 IO 密集型，如 socket，爬虫，web

  多进程用于计算密集型，如金融分

## 进程池与线程池

​ 在刚开始接触多进程或多线程时，我们迫不及待地基于多进程或多线程实现并发的套接字通信，然而这种实现方式的致命缺陷是：**服务的开启的进程数或线程数都会随着并发的客户端数目地增多而增多，这会对服务端主机带来巨大的压力，甚至于不堪重负而瘫痪。**于是我们必须对服务端开启的进程数或线程数加以控制，让机器在一个自己可以承受的范围内运行，这就是进程池或线程池的用途，例如进程池，就是用来存放进程的池子，本质还是基于多进程，只不过是对开启进程的数目加上了限制

- Python 标准模块 concurrent.futures

  ```python
  # 1、介绍
  concurrent.futures模块是用来创建并行的任务，提供了高度封装的异步调用接口
  concurent.future这个模块用起来非常方便，它的接口也封装的非常简单，既可以实现进程池，也可以实现线程池
  ThreadPoolExecutor：线程池，提供异步调用
  ProcessPoolExecutor: 进程池，提供异步调用
  两者都实现了同一个接口，这个接口是由抽象Executor类定义的。

  # 2、基本方法
  submit(fn, *args, **kwargs)
  异步提交任务

  map(func, *iterables, timeout=None, chunksize=1)
  取代for循环submit的操作

  shutdown(wait=True)
  相当于进程池的pool.close()+pool.join()操作
  wait=True，等待池内所有任务执行完毕回收完资源后才继续
  wait=False，立即返回，并不会等待池内的任务执行完毕
  但不管wait参数为何值，整个程序都会等到所有任务执行完毕
  submit和map必须在shutdown之前

  result(timeout=None)
  取得结果

  add_done_callback(fn)
  回调函数
  ```

### 进程池

```python
"""
# 介绍：
ProcessPoolExecutor类是Executor的子类，它使用一个进程池来异步执行调用。ProcessPoolExecutor
使用多处理模块，这允许它避免全局解释器锁，但也意味着只能执行和返回可pickle的对象。
类concurrent.futures。ProcessPoolExecutor (max_workers = None, mp_context =没有)
使用最多max_workers进程池异步执行调用的Executor子类。如果max_workers为None或未给出，则默认值为
机器上的处理器数。如果max_workers小于或等于0，则会引发ValueError。

"""
# 用法：异步执行
from concurrent.futures import ProcessPoolExecutor
from threading import current_thread
import os,time,random

def task(n):  # 定一个任务
    print('%s is runing' %os.getpid()) # 任务启动先打印任务的进程pid
    # I/O密集型的，一般用线程，用进程开销大耗时长
    time.sleep(random.randint(1,3))  # 随机睡1-3秒
    return n**2   # 返回值

def handle(futrue): # 处理任务的函数，拿到futrue对象
    res = futrue.result() # 拿到返回结果，一个任务运行完就会触发回调函数，所以不会阻塞
    print("%s 正在处理结果：%s" %(os.getpid(),res))
    time.sleep(2)

if __name__ == '__main__':
    pool = ProcessPoolExecutor(max_workers=4) # 对于进程池如果不写max_works：默认的是cpu的数量是4个

    for i in range(19):  # 现在开了19个任务，如果是上百个任务，就不能无限开进程，就要考虑控制
        pool.submit(task,i).add_done_callback(handle) # 异步的方式提交任务

    pool.shutdown(wait=True)
'''
解析:
pool.submit(task,i)会返回一个futrue对象，这个任务对象可以调出add_done_callback()方法，
叫回调函数，里面就一个参数handle，也就是说每提交一个任务捆绑一个函数，一旦一个任务运行完就会立
马触发这个回调函数的运行,并且会自动的把任务对象当做第一个参数传给回调函数。
在回调函数里处理任务，先拿到结果，一个任务运行完就会触发这个回调函数，所以不会阻塞在原地。打印
一边在运行一边就会有人在处理结果，一边在运行着一边结果正在被处理，这个运行效率并不慢，一直都是
主进程在处理任务，这就是回调函数的概念。

'''
```

### 线程池

```python
"""
# 介绍：
ThreadPoolExecutor是Executor的子类，它使用一个线程池来异步执行调用。
类concurrent.futures。ThreadPoolExecutor (max_workers = None, thread_name_prefix = ")
一个Executor子类，使用最多max_workers线程池来异步执行调用。
3.5版本的变化:如果max_workers没有或没有,它将默认为处理器的机器上,乘以5,假设ThreadPoolExecutor通常
   用于重叠I / O而不是CPU工作和工人的数量应该为ProcessPoolExecutor高于工人的数量。
3.6新版功能:添加了thread_name_prefix参数，允许用户控制线程。由池创建的工作线程的线程名，以便于调试。

"""
# 用法：
from concurrent.futures import ThreadPoolExecutor
from threading import current_thread
import os,time,random

def task(n):
    print('%s is runing' %current_thread().name)
    time.sleep(random.randint(1,3))
    return n**2

def handle(futrue):
    res = futrue.result()
    print("%s 正在处理结果：%s" %(current_thread().name,res))
    time.sleep(2)

if __name__ == '__main__':
    pool = ThreadPoolExecutor(max_workers=10) # 对于线程池如果不写max_works：默认的是cpu的数目*5

    for i in range(19): # 同样是19个任务，线程池效率高了
        pool.submit(task,i).add_done_callback(handle)

    pool.shutdown(wait=True)
```

## 协程

### 介绍

​ 协程是单线程下实现的并发,协程是一种用户态的轻量级线程，即协程是由用户程序自己控制调度的

​ 对于单线程下，我们不可避免程序中出现 io 操作，但如果我们能在自己的程序中（即用户程序级别，而非操作系统级别）控制单线程下的多个任务能在一个任务遇到 io 阻塞时就切换到另外一个任务去计算，这样就保证了该线程能够最大限度地处于就绪态，即随时都可以被 cpu 执行的状态，相当于我们在用户程序级别将自己的 io 操作最大限度地隐藏起来，从而可以迷惑操作系统，让其看到：该线程好像是一直在计算，io 比较少，从而更多的将 cpu 的执行权限分配给我们的线程。

​ python 的线程属于内核级别的，即由操作系统控制调度（如单线程遇到 io 或执行时间过长就会被迫交出 cpu 执行权限，切换其他线程运行）

​ 单线程内开启协程，一旦遇到 io，就会从应用程序级别（而非操作系统）控制切换，以此来提升效率（！！！非 io 操作的切换与效率无关）

​ 对比操作系统控制线程的切换，用户在单线程内控制协程的切换

- 特点: 自己的应用程序实现多个人的调度

  遇到 I/O 切换，可以将单线程的 I/O 降到最低，因此可以将单线程的威力发挥到最大

- 缺点: 不能实现并行

  单线程下的多个任务一旦遇到 I/O，整个线程都会阻塞，所有的任务都停滞

- 总结

  - 必须在只有一个单线程里实现并发
  - 修改共享数据不需加锁
  - 用户程序里自己保存多个控制流的上下文栈
  - 附加：一个协程遇到 IO 操作自动切换到其它协程（如何实现检测 IO，yield、greenlet 都无法实现，就用到了 gevent 模块（select 机制））\*\*

  **yiled**可以保存状态，**yield**的状态保存与操作系统的保存线程状态很像，但是**yield 是代码级别控制**的，更轻量级 send 可以把一个函数的结果传给另外一个函数，以此实现**单线程内程序之间的切换**

### Gevent 模块

​ **Gevent**是一个第三方库，可以轻松通过 gevent 实现并发同步或异步编程，在 gevent 中用到的主要模式是 Greenlet, 它是以 C 扩展模块形式接入 Python 的轻量级协程。 Greenlet 全部运行在主程序操作系统进程的内部，但它们被协作式地调度。

Gevent 内部会用到 greenlet 这个模块，这个模块就是多个任务之间来回的切，切走之前把一个任务的状态保留下来，它们的底层都会用到 yield，其实就是层层帮我们封装好了。greenlet 内部会封装 yield，Gevent 就是对 greenlet 进行了进一步的封装，封装后 greenlet 会帮忙检测 I/O，实现遇到 I/O 切换，这个才是我们所追求的协程

- 使用方法

  ```python
    g1=gevent.spawn(func,1,,2,3,x=4,y=5)创建一个协程对象g1，spawn括号内第一个参数是函数名，
                    如eat，后面可以有多个参数，可以是位置实参或关键字实参，都是传给函数eat的

    g2=gevent.spawn(func2)

    g1.join()  等待g1结束

    g2.join()  等待g2结束

    或者上述两步合作一步：gevent.joinall([g1,g2])

    g1.value#拿到func1的返回值
  ```

  遇到 IO 阻塞时自动切换任务

  ```python
  import gevent
  def eat(name):
      print('%s eat 1' %name) # 1.吃了一口饭
      gevent.sleep(2)  # 2.原地睡了2秒，相当于模拟遇到I/O了
      print('%s eat 2' %name) # 6.接着打印又回来吃了一口饭

  def play(name):
      print('%s play 1' %name)  # 3.遇到I/O以后就切到了另外一个任务，玩了一下
      gevent.sleep(1)  # 4.又遇到I/O了，睡了1秒，它先睡完
      print('%s play 2' %name) # 5.接着又玩了一下，原本应该切到eat 2，但是仍在阻塞中


  g1=gevent.spawn(eat,'egon') # spawn提交eat任务，然后提交一个人名。协程1
  g2=gevent.spawn(play,name='egon')# spawn提交playt任务。协程2
  g1.join() # 等着协程对象g1结束
  g2.join() # 等着协程对象g2结束
  #或者gevent.joinall([g1,g2])
  print('主')

  '''
  上例gevent.sleep(2)模拟的是gevent可以识别的io阻塞,而time.sleep(2)或其他的阻塞,gevent是不能直接识别的需要用下面一行代码,打补丁,就可以识别了
  '''
  ```

- 打补丁

  ```python
  '''
  from gevent import monkey;monkey.patch_all()必须放到被打补丁者的前面，如time，socket模块之前或者我们干脆记忆成：要用gevent，需要将from gevent import monkey;monkey.patch_all()放到文件的开头
  '''
  from gevent import monkey;monkey.patch_all()

  import gevent
  import time
  def eat():
      print('eat food 1')
      time.sleep(2)
      print('eat food 2')

  def play():
      print('play 1')
      time.sleep(1)
      print('play 2')

  g1=gevent.spawn(eat)
  g2=gevent.spawn(play_phone)
  gevent.joinall([g1,g2])
  print('主')
  """
  单线程下能抗住的并发已经非常非常高了，因为现在接触的软件大部分都是I/O密集型的
  其实单线程下完全可以一个任务运行完以后（它真正运行完花的时间是非常短的，大量时间都在做I/O）
  可以利用运行一段时间遇到I/O操作了就快速切换另一个任务再运行，在多任务之间快速的切
  """
  ```

- 基于协程实现并发

  **通过 gevent 实现单线程下的 socket 并发（from gevent import monkey;monkey.patch_all()一定要放到导入 socket 模块之前，否则 gevent 无法识别 socket 的阻塞）**

  - 服务端

    ```python
    # 首先导了猴子补丁，打了补丁保证下面所有模块的I/O行为都能监测到
    from gevent import monkey;monkey.patch_all()
    from socket import *   # 然后导了socket模块，准备写套接字
    import gevent # 最后导入gevent模块， 用来单线程下实现并发


    def server(server_ip,port): # 套接字服务端任务1：建链接
        s=socket(AF_INET,SOCK_STREAM)
        s.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
        s.bind((server_ip,port)) # 绑定ip和端口
        s.listen(5)  # 监听
        while True:
            conn,addr=s.accept() # 等待链接请求
            # 每建成一个链接，就提交一个协程对象进行通信，异步提交
            gevent.spawn(talk,conn,addr)

    def talk(conn,addr):  # 套接字服务端任务2：建通信
        try:
            while True:
                res=conn.recv(1024) # 收消息
                print('client %s:%s msg: %s' %(addr[0],addr[1],res))
                conn.send(res.upper()) # 回消息，大写回
        except Exception as e:
            print(e)
        finally:
            conn.close()

    if __name__ == '__main__':
        server('127.0.0.1',8080) # 把ip和端口传进去

    # 注：没必要join在原地等了，因为服务端在启动运行起来后，服务端函数是一个死循环，
    # 不会结束，既然主进程不会结束那就不用再等了
    """
    整体逻辑：就一个线程server，没有多线程也没有多进程，这个线程每建成一个链接就提交
    一个协程对象，gevent会帮你在多个任务之间遇到I/O来回快速的切换，从而实现并发效果
    如何证明并发的效果？
    服务端启动起来后，同时多个客户端连接过去，如果多个客户端能同时得到结果，并发效果
    就实现了

    """
    ```

  - 客户端

    ```python
    # 可同时开多个客户端(客户端1、客户端2、客户端3)

    from socket import *

    client=socket(AF_INET,SOCK_STREAM)
    client.connect(('127.0.0.1',8080))

    while True:
        client.send("hello".encode('utf-8')) # 在不停的向服务端发送“hello”
        msg=client.recv(1024) # 收消息，在不停的收HELLO
        print(msg.decode('utf-8'))

    """
    解析：
    三个客户端都能同时不停的发消息和收消息，都有并发效果，但服务端没有开多线程，事实上
    就是服务端在多个任务之间来回的切换
    其实就是给第一个客户端执行一个seed来发送I/O请求，只要seed发出之后运行完就是操作
    系统的任务了，seed负责发消息，操作系统负责做I/O。gevent模块会利用你seed的过程
    直接切到下一个任务，再切到下下一个任务，一直往下切，给客户端的感觉就是每一个客户端
    都能被服务，并发就实现了

    """
    ```

## IO 模型

### 简介

IO 模型研究的主要是网络 IO(linux 系统)

- 同步（synchronous） 大部分情况下会采用缩写的形式 sync
- 异步（asynchronous） async
- 阻塞（blocking）
- 非阻塞（non-blocking）

五种 IO 模型：
_ blocking IO 阻塞 IO
_ nonblocking IO 非阻塞 IO
_ IO multiplexing IO 多路复用
_ signal driven IO 信号驱动 IO \* asynchronous IO 异步 IO
由 signal driven IO（信号驱动 IO）在实际中并不常用，所以主要介绍其余四种 IO Model

### 四种 IO 模型简介

#### 阻塞 IO

    最为常见的一种IO模型 有两个等待的阶段(wait for data、copy data)

#### 非阻塞 IO

    系统调用阶段变为了非阻塞(轮训) 有一个等待的阶段(copy data)

轮训的阶段是比较消耗资源的

#### 多路复用 IO

    利用select或者epoll来监管多个程序 一旦某个程序需要的数据存在于内存中了 那么立刻通知该程序去取即可

#### 异步 IO

    只需要发起一次系统调用 之后无需频繁发送 有结果并准备好之后会通过异步回调机制反馈给调用者
