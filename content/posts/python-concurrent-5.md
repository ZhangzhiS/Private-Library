---
title: "Python的并发编程（五）- 线程池"
date: 2020-01-25T16:17:59+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

## Python的并发编程（五）- 线程池

之前的文章学习了一些多线程的用法，在I/O密集型的程序中，多线程带来了显著的性能提升，那我们可以无限制的，大量的创建多线程任务吗？

一个线程创建、销毁都是需要消耗系统资源的，如果线程数大于一定的数量，线程的创建销毁就会占用大量的系统性能，就不能充分利用到系统资源了。这时候，可以选择使用线程池了，



### 线程池

创建一定数量的线程来执行任务，任务结束之后，该线程不进行销毁，而是继续从任务队列中获取新任务来执行，直到所有任务都执行结束之后，关闭所有线程。

`concurrent.futures`库中包含有一个`ThreadPoolExecutor`类可用来实现这个目的。下面来实现一个简单的TCP服务器，使用线程池来服务客户端：

```
from socket import AF_INET, socket, SOCK_STREAM
from concurrent.futures import ThreadPoolExecutor


def echo_client(sock, client_addr):
    print("{addr}已连接".format(addr=client_addr))
    while True:
        msg = sock.recv(40960)
        if not msg:
            break
        sock.sendall(msg)
    sock.close()
    print("关闭连接")


def echo_server(addr):
    pool = ThreadPoolExecutor(128)
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(addr)
    sock.listen(5)
    while True:
        client_sock, client_addr = sock.accept()
        pool.submit(echo_client, client_sock, client_addr)

if __name__ == '__main__':
    echo_server(("", 5000))
```

当然，我们也可以手动创建线程池，使用`Queue`来作为任务队列。

```
from socket import AF_INET, socket, SOCK_STREAM
from threading import Thread
from queue import Queue

def echo_client(q):
    sock, client_addr = q.get()
    print("{addr}已连接".format(addr=client_addr))
    while True:
        msg = sock.recv(40960)
        if not msg:
            break
        sock.sendall(msg)
    sock.close()
    print("关闭连接")

def echo_server(addr, nworkers):
    q = Queue()
    for n in range(nworkers):
        # 启动nworkers个线程
        t = Thread(target=echo_client, args=(q, ))
        t.daemon = True
        t.start()
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(addr)
    sock.listen(5)
    while True:
        client_sock, client_addr = sock.accept()
        q.put(client_sock, client_addr)

if __name__ == '__main__':
    echo_server(("", 5000), 10)
```

尽量使用`ThreadPoolExecutor`而不是手动实现线程池。这么做的优势在于使得任务的提交者能够更容易从调用函数中取得结果。

```
import urllib.request
from concurrent.futures import ThreadPoolExecutor


def fetch_url(url):
    u = urllib.request.urlopen(url)
    data = u.read()
    return data

pool = ThreadPoolExecutor(10)
a = pool.submit(fetch_url, "http://www.python.org")
b = pool.submit(fetch_url, "http://www.pypy.org")

x = a.result()
y = b.result()
```

示例中的结果对象（即a和b）负责处理所有需要完成的阻塞和同步任务，从工作者线程中取回数据。特别是，`a.result()`操作会阻塞，直到对应的函数已经由线程执行完幷返回了结果为止。

**注意**：避免写允许线程无限增长的程序。如果在web服务中这么做了，无法阻止恶意用户对服务器发起拒绝服务攻击，从而导致服务器上创建了大量的线程，耗尽了系统资源而崩溃。通过预先初始化好的线程池，就可以小心的为所有能支持的并发总数设定一个上限。

