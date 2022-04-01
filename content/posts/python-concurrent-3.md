---
title: "Python的并发编程（三）-线程间通信"
date: 2020-01-25T16:15:05+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

在写多线程程序的时候，可能会有需求需要我们在线程之间交换数据

我们如何在线程之间实现安全的通信或者交换数据呢？

### Queue队列

也许将数据从一个线程发往另一个线程最安全的做法就是使用queue模块中的Queue（队列）了。



简单的流程：

1. 创建Queue实例，**Queue实例会被所有的线程共享。**
2. put()添加元素
3. get()获取元素

![img](https://i.loli.net/2019/05/08/5cd25f740a855.jpg)

```
import time
from queue import Queue
from threading import Thread


def producer(out_q):
    for i in range(10):
        out_q.put(i)
        time.sleep(2)


def consumer(in_q):
    while True:
        data = in_q.get()
        print(data)

q = Queue()
t1 = Thread(target=producer, args=(q, ))
t2 = Thread(target=consumer, args=(q, ))
t1.start()
t2.start()
```

Queue实例已经拥有了所有需要的锁，所以他们可以安全的在任意多线程之间共享。

如何对生产者和消费这的关闭过程进行同步协调？

我们可以用一个特殊的终止值，当我们把它放入队列，就使消费者退出。

```
import time
from queue import Queue
from threading import Thread

_sentinel = object()

def producer(out_q):
    for i in range(10):
        out_q.put(i)
        time.sleep(2)
    out_q.put(_sentinel)


def consumer(in_q):
    while True:
        data = in_q.get()
        if data is _sentinel:
            in_q.put(_sentinel)
            break
        print(data)

q = Queue()
t1 = Thread(target=producer, args=(q, ))
t2 = Thread(target=consumer, args=(q, ))
t1.start()
t2.start()
```

在示例中，，当一个消费者收到这个退出信号之后，会退出。将终止值放回队列是因为如果有多个消费者，这样可以使其他监听这个队列的其他消费者线程也能够接收到这个终止值。

尽管队列是线程之间通信 的最常见的机制，但是只要添加了所需要的锁和同步功能，就可以构建自己的线程安全结构，最常见的做法就是将你的数据结构和条件变量打包在一起。

下面我们构建一个线程安全的优先级队列。

```
import heapq
import threading

class PriorityQueue(object):
    def __init__(self):
        self._queue = []
        self._count = 0
        self._cv = threading.Condition()

    def put(self, item, priority):
        while self._cv:
            heapq.heappush(self._queue, (-priority, self._count, item))
            self._count += 1
            self._cv.notify()

    def get(self):
        with self._cv:
            while len(self._queue) == 0:
                self._cv.wait()
            return heapq.heappop(self._queue)[-1]
```

通过队列实现的线程之间通信是一种单向的且不确定的过程。一般来说，我们无法得知接收线程（消费者）何时会实际接收到消息并开始工作。但是，Queue对象提供了一些基本的事件完成功能。

`q.join()`会等待队列中所有的信息被消费。

Queue对象的`put()`和`get()`都支持非阻塞和超时机制。

```
import queue

q = queue.Queue()

try:
    data = q.get(block=False)
except queue.Empty:
    pass

try:
    q.put("item", block=False)
except queue.Full:
    pass

try:
    data = q.get(timeout=1)
except queue.Empty:
    pass
```

可以避免特定的队列操作上无限期的阻塞下去。用法比较灵活多变。

最后还有一些别的实用方法， 比如`q.qsize()`、`q.full()`、`q.empty()`，他们能够告诉你队列的当前大小和状态。但是，这些方法在多线程环境中是不可靠的。例如：对`q.empty()`的调用可能会告诉我们队列是空的，但是在完成这个调用的同时，另外的线程可能已经往队列中添加了一个元素。

所以，在编写代码的时候不要太过于依赖这些函数。

