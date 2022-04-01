---
title: "Python的并发编程（六）- 多进程"
date: 2020-01-25T16:20:47+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

之前学习了多线程以及线程池，他们在执行I/O密集的程序的时候，性能是很高的，但是如果我们有大量的CPU密集型工作的程序，现在想利用多个CPU的优势运行的更快，应该怎么解决呢？

这时候，就不能使用多线程了，而是需要真正的并行来解决问题。

在`concurrent.futures`库中提供了一个`ProcessPoolExecutor`类，可用来在单独运行的python解释器实例中执行计算密集的函数。

`ProcessPoolExecutor`的典型用法是下面这样的：

```
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as pool:
    """
    在进程池pool中并行执行任务
    """
```



在底层，`ProcessPoolExecutor`创建了N个独立运行的Python解释器，这里的N就是系统上检测到的可用的CPU个数。可以创建和修改Python的进程数，只要给`ProcessPoolExecutor(N)`提供一个可选的参数。进程池会一直运行，直到with语句块中的最后一条语句执行完毕为止，此时进程池就会关闭。但是程序会一直等待所有已经提交的任务都处理完毕为止。

提交到进程池中的任务必须定义为函数形式。有两种方法可以提交任务。如果想并行处理一个列表推导式或者map()操作，可以使用`pool.map()`：

```
from concurrent.futures import ProcessPoolExecutor

def work(x):
    """任务逻辑"""
    return x

data = [1, 2, 3, 4]

with ProcessPoolExecutor() as pool:
    """
    在进程池pool中并行执行任务
    """
    results = pool.map(work, data)
```

另一种方式是通过`pool.submit()`来手动提交单独的任务：

```
from concurrent.futures import ProcessPoolExecutor

def work(x):
    """任务逻辑"""
    return x

data = 1

with ProcessPoolExecutor() as pool:
    """
    在进程池pool中并行执行任务
    """
    future_result = pool.submit(work, data)
    result = future_result.result()
```

如果手动提交任务，得到的结果就是一个Future实例。要获取到结果还需要手动调用`result()`方法。这么做会阻塞进程，直到结果返回为止。所以与其让进程阻塞，不如提供一个回调函数，让他执行任务完成时出发执行。示例如下：

```
from concurrent.futures import ProcessPoolExecutor

def work(x):
    """任务逻辑"""
    return x

def when_done(r):
    print("result：", r.result())

data = 1

with ProcessPoolExecutor() as pool:
    """
    在进程池pool中并行执行任务
    """
    future_result = pool.submit(work, data)
    future_result.add_done_callback(when_done)
```

用户提供的回调函数需要接受一个Future实例，必须用他才能获取实际的结果。

尽管进程池看起来很简单，但是在设计规模更大的程序时有下面几个重要的因素需要考虑。

- 这种并行处理的技术只适用于可以将问题分解成各个独立部分的情况。
- 任务必须定义成普通函数来提交。实例方法，比包或者其他类型的可调用对象都是不支持并行处理的。
- 函数的参数和返回值必须可兼容`pickle`编码。任务的执行是在单独的解释器进程中完成的。这中间需要进程间通信。因此，在不同的解释器之间交换数据必须要进行序列化处理。
- 提交的工作函数都不应该维护持久的状态或者带有副作用。除了简单的日志功能，一旦子进程启动，将无法控制他的行为。因此，为了让思路保持清晰，最好让每件事情都保持简单，让任务在不会修改执行环境的纯函数中执行。
- 进程池是通过调用UNIX上的fork()系统调用来创建的。这么做会克隆一个Python解释器，在fork()时会包含所有的程序状态。在windows上，这么做会加载一个独立的解释器拷贝，但不包含状态。克隆出来的进程在首次调用`pool.map()`或者`pool.submit()`之前不会实际运行。
- 既然是克隆一个独立的解释器，那每个进程都可以再执行线程。当进程池和多线程技术结合在一起使用的时候需要格外的小心。特别是，很可能我们应该在创建任何线程之前优先创建并加载进程池（例如，当程序启动时在主线程中创建进程池）。

