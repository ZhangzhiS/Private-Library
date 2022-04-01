---
title: "Python的并发编程（四）- 多线程中的锁"
date: 2020-01-25T16:17:56+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

如何在多线程中安全的使用可变对象呢？比如给文件中写入内容的时候，我们希望当前线程写入完毕之后，其他线程才能够继续操作这个文件，避免文件内容错乱。

### Lock对象

可以利用threading库中的Lock对象来解决问题。



```
import threading

class SharedCounter(object):

    def __init__(self, initial_value=0):
        self._value = initial_value
        self._value_lock = threading.Lock()

    def incr(self, delta=1):
        """增加"""
        with self._value_lock:
            self._value += delta

    def decr(self, delta=1):
        """减少"""
        with self._value_lock:
            self._value -= delta
```

当线程中使用with 语句时，Lock对象可以确保产生互斥的行为–也就是说，统一时间只允许一个线程with语句代码块中的逻辑。with 语句会在执行缩进的代码块时获取到锁，当控制流离开缩进语句时释放这个锁。

线程的调度从本质上来说是非确定性的。正因为如此，在多线程程序中如果不用好锁就会使得数据被随机的破坏掉，以及产生我们称之为竟态条件（race condition）的奇怪行为。要避免这些问题，只要共享的可变状态需要被多个线程访问，那么就需要使用锁。

当然还有以下这种写法：

```
import threading

class SharedCounter(object):
    # ....
    def incr(self, delta=1):
        self._value_lock.acquire()
        self._value += delta
        self._value_lock.release()

    # ....
```

不过with不是更优雅吗？而且with也不容易出错————尤其是我们在持有锁的时候抛出了异常，我们可能会忘记调用`release()`，而with语句总是会确保释放锁。

### 避免死锁

在多线程程序中，出现死锁的常见原因就是线程一次尝试获取了多个锁，列如有一个线程获取到第一个锁，但是他在尝试获取第二个锁的时候阻塞了，那么这个线程可能会阻塞住其他线程的执行，进而使得整个程序僵死。

避免出现死锁的的一种解决方案就是给程序的每个锁分配一个唯一的数字编号，并且在获取多个锁时按照编号的升序方式来获取。看看示例：

```
import threading
from contextlib import contextmanager

_local = threading.local()

@contextmanager
def acquire(locks):
    locks = sorted(locks, key=lambda x: id(x))

    acquired = getattr(_local, "acquired", [])

    if acquired and max(id(lock) for lock in acquired) > id(locks[0]):
        raise RuntimeError("Lock Order Violation")

    acquired.extend(locks)
    _local.acquired = acquired
    try:
        for lock in locks:
            lock.acquire()
        yield
    finally:
        for lock in reversed(locks):
            lock.release()
        del acquired[-len(locks):]
```

要使用这个上下文管理器，只要按照正常的方式来分配锁对象，但是当相同一个或者多个锁打交道时就是用`acquire()`这个函数。例如：

```
import threading

x_lock = threading.Lock()
y_lock = threading.Lock()

def thread_1():
    while True:
        with acquire(x_lock, y_lock):
            print("Thread-1")

def thread_2():
    while True:
        with acquire(x_lock, y_lock):
            print("Thread-2")

t1 = threading.Thread(target=thread_1)
t1.daemon = True
t1.start()

t2 = threading.Thread(target=thread_1)
t2.daemon = True
t2.start()
```

如果运行这个程序，就会发现这个程序运行的很好，永远不会死锁。

本文只是提供简单的讲解，编码的时候灵活运用。

明天我们来说说线程池。

