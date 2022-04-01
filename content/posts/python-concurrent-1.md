---
title: "Python的并发编程（一）-了解并发以及简单的多线程实例"
date: 2020-01-25T16:14:57+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

求职过程中，好多公司的招聘信息都会写一条：**有构建大型互联网服务及高并发等经验**

那什么是高并发呢？

**对于服务端接口来说，就是我们的接口可以同时并行处理大量的请求。**

### 理解并发与并行

先了解一下并发和并行的区别。百度百科对并发和并行的解释如下：



> **并发**当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程，它只能把CPU运行时间划分成若干个时间段,再将时间 段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状。.这种方式我们称之为并发(Concurrent)。
>
> **并行：**当系统有一个以上CPU时,则线程的操作有可能非并发。当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)。

两者之间有什么区别呢，举个例子说明一下：

下面是ABC三座桥（画得比较丑），下面的例子只是为了帮助理解并行和并发的区别，不要太较真显示生活中怎么处理这样的情况。

- A桥：当有一辆车行驶在桥面上之后，如果对面来了车，只有等待桥上的车离开，对面的车才可以上桥并通过，这样可以理解为A桥不支持并发。
- B桥：当有一辆车行驶在桥面上之后，如果对面来了车，第一辆车可以进入那块小区域，对面的车过去第一辆车再接着行驶，可以说B桥支持并发。
- C桥：两条车道，自己走自己的，可以说C桥支持并行。

![img](https://i.loli.net/2019/05/08/5cd2711d34fca.jpg)

### Python中的并发

Python也在很早就开始支持多种不同的并发编程方法，包括多线程，加载子进程以及各种涉及生成器函数的技巧（协程）。

#### 多线程

Python3提供了threading库来实现多线程，可用来在单独的线程中执行任意的Python可调用对象。

##### 启动和停止

先写一个简单的函数：

```
import time

def countdown(n):
    while n > 0:
        print("n=", n)
        n-=1
        time.sleep(5)
```

使用多线程来执行上面的函数：

```
from threading import Thread

t = Thread(target=countdown, args=(10,))
t.start()
```

注意：当创建一个线程实例时，在调用它的`start()`方法之前，线程不会立即开始执行。**args需要的参数为元组。**

线程实例会在他们所属的系统级线程（POSIX线程或Windows线程）中执行，这些线程完全由操作系统来管理，一旦启动之后，线程就开始独立运行，直到目标函数返回，可以使用`t.is_alive()`来判断线程是否还在运行。

```
while True:
    if t.is_alive():
        print("执行中")
        time.sleep(10)
    else:
        print("完成")
        break
```

也可以使用：`t.join()`来连接到该线程，等待该线程执行结束。

```
t.join()
print("完成")
```

在线程执行的过程中，Python的解释器会一直保持运行，直到所有的线程都终结。对于需要长时间运行的线程或者一直不断运行的后台任务，应该考虑将这些线程设置为daemon（守护线程）,daemon是无法被连接的，但是在主线程结束之后他们会自动销毁。

```
t = Thread(target=countdown, args=(10,), daemon=True)
t.start()
```

除了以上展示的两种操作外，对于线程没有太多的操作可做了。比如终止线程，给线程发信号，调整线程调度属性以及执行任何其他的高级操作，如果需要，就得自己构建。

如果想要终止线程，这个线程必须要能够在某个点上轮询退出状态。我们可以试着实现一下：

```
class CountdownTask(object):
    def __init__(self):
        self.__running = True

    def terminate(self):
        self.__running = False

    def run(self, n):
        while self.__running and n > 0:
            print("n=", n)
            n -= 1
            time.sleep(5)

c = CountdownTask()

from threading import Thread

t = Thread(target=c.run, args=(10,))
t.start()
```

可以通过`c.terminate()`来终止线程。

#### 注意事项

由于Python存在大名鼎鼎的全局解释器锁（GIL），所以Python的多线程其实只是伪多线程，解释器限制在任意时刻只允许运行一个线程。由于这个原因，不应该使用Python线程来处理计算密集型的任务，因为在这种任务中我们希望在多个CPU和行上实现**并行**处理。Python线程更适合I/O处理以及设计阻塞操作的并发执行任务（即，等待I/O，等待从数据库中取出结果等）。

有时候我们会发现从Thread类中继承而来的线程类。比如：

```
from threading import Thread

class CountdownThread(Thread):
    def __init__(self, n):
        super().__init__()
        self.n = n
    def run(self):
        while self.n > 0:
            print("n=", n)
            n -= 1
            time.sleep(5)

c = CountdownThread(5)
c.start()
```

尽管这样也可以完成任务，但这个在代码和threading库之间引入了一层额外的依赖关系。意思就是说，上面的代码只能用在有关线程的上下文中，而我们之前展示的技术中编写的代码并不会依赖于threading库。

