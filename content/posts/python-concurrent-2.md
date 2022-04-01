---
title: "Python的并发编程（二）-多线程的执行状态"
date: 2020-01-25T16:15:03+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

[上一篇](https://zz.zzs7.top/python-concurrent-1.html)文章我们写了一个简单的多线程，我们使用`start()`来启动这个线程，但是如果我们想知道它实际会在什么时候开始运行呢？

## 获取线程状态

### 使用Event同步线程状态

线程的核心特点就是它们能够以非确定性的方式独立执行。即，什么时候开始执行、何时被打断、何时恢复执行等状态完全由操作系统来调度管理，这是用户和程序员都无法确定的。如果程序中有其他线程需要判断某个线程是否执行到了某个点，根据这个来判断后续的操作，那就产生了线程同步的问题。

要解决这类问题，我们可以使用threading中的Event对象。



```
import time

from threading import Thread, Event

def countdown(n, started_evt):
    print("开始")
    started_evt.set()
    while n > 0:
        print("n=", n)
        n-=1
        time.sleep(5)

# 创建一个Event对象
started_evt = Event()

print("启动 countdown")

t = Thread(target=countdown, args=(10, started_evt))
t.start()

# 等待线程开始

started_evt.wait()
print("countdown 正在执行")
```

`Event`对象和条件标记（sticky flag）类似，允许县城等待某个事件发生。初始状态时事件被设置为0。如果时间没有被设置而线程正在等待该事件，那么线程就会被阻塞（休眠状态），直到事件被设置为止。当有线程设置了这个事件时，这回唤醒所有正在等待该事件的线程（如果存在）。如果线程等待的事件已经设置了，那么线程会继续执行。

上面的代码执行结果如下：

```
启动 countdown
开始
n= 10
countdown 正在执行
n= 9
n= 8
n= 7
n= 6
n= 5
n= 4
n= 3
n= 2
n= 1
```

“countdown 正在执行”总是会在“启动 countdown”之后打印，这里使用了时间来同步线程，使主线程等待。

代码执行的顺序为：

1. t.start() 启动线程。
2. 顺序执行到`started_evt.wait()`的时候，会等待`started_evt.set()`的执行。

如果注释了`started_evt.set()`就会在`started_evt.wait()`处阻塞，“countdown 正在执行”也不会被打印，可以自己多修改代码调试，观察打印情况。

### 使用Condition通知事件代替Event

Event对象最好之用于一次性的事件，也就是说，我们创建一个事件，让线程等待时间被设置，然后一旦完成了设置，Event对象就被丢弃。尽管可以使用Event对象的`clean()`方法来清除事件，但是要安全地清楚事件并等待它被再次设置这个过程很难同步协调，可能会在成事件丢失，死锁或者其他问题（特别是，在设置完事件之后，我们无法保证发起的时间清除请求就一定会在线程再次等待该事件之前被执行）。

比如上面的代码：

```
started_evt.clear()
print("tag")
started_evt.wait()
print("countdown 正在执行11")
```

在后面填家上面的代码，每次运行结果中，tag的打印位置是不确定的。

如果线程打算一遍又一遍地重复通知某个事件，那最好使用Condition对象来处理。

下面的代码实现了一个周期性的定时器，每当定时器超时，其他的线程都可感知到超时事件。

```
import time

from threading import Thread, Condition

class PeriodicTimer:
    def __init__(self, interval):
        self._interval = interval
        self._flag = 0
        self._cv = Condition()

    def start(self):
        t = Thread(target=self.run)
        t.daemon = True
        t.start()

    def run(self):
        while True:
            time.sleep(self._interval)
            with self._cv:
                self._flag ^= 1
                self._cv.notify_all()

    def wait_for_tick(self):
        while self._cv:
            last_flag = self._flag
            while last_flag == self._flag:
                self._cv.wait()

ptimer = PeriodicTimer(5)
ptimer.start()

def countdown(nticks):
    while nticks > 0:
        ptimer.wait_for_tick()
        print("n=", nticks)
        nticks -= 1

def countup(last):
    n = 0
    while n < last:
        ptimer.wait_for_tick()
        print("n=", n)
        n += 1

Thread(target=countdown, args=(10, )).start()
Thread(target=countup, args=(5, )).start()
```

Event 对象的关键特性就是他会唤醒所有等待的线程。如果我们编写的程序只希望唤醒一个单独的等待线程，那么最好使用Semaphore或者Condition对象。

