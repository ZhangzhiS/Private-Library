---
title: "关于同源策略"
date: 2020-01-25T16:29:11+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

## 同源策略

> 同源策略是一种约定，是由[Netscape](https://baike.baidu.com/item/Netscape/2778944)提出的著名安全策略。他是浏览器最核心、最基本的安全功能。

### 同源的定义

如果两个页面的协议、端口、主机都相同，则两个页面具有相同的源。



下面做一些说明：

1. https://zz.zzs7.top/mysql-optimization.html
2. http://zz.zzs7.top/mysql-optimization.html
3. https://zz.zzs7.top:8000/mysql-optimization.html
4. https://s7.zzs7.top/mysql-optimization.html
5. https://104.125.11.39/mysql-optimization.html

上面的链接中：

- 1和2比较：不是同源，因为1用的https，2用的是http。属于协议不同。
- 1和3比较：不是同源，1未指定端口的时候，默认使用的是80,而3使用的是8000端口，属于端口不同。
- 1和4比较：不是同源，域名不同。
- 1和5比较：虽然1部署在了该ip地址对应的服务器，但是依然不是同源。

在不同源的情况下1站点里面的js脚本采用ajax读取其他三个站点中的数据是会报错的。

**注：IE浏览器未将不同端口加入同源策略的组成部分。**

### 跨域

受同源策略的影响，不是 同源下的脚本不能操作其他源下面的对象。想要操作其他源下的对象就需要跨域。

#### 为什么要跨域

比如某视频网站由于数据太多，分布在了不同的服务器上，所以需要突破同源策略，实现数据交互。

#### 跨域的实现

有以下几种实现方式：

- **降域（document.domain）**：同源策略认为域和子域属于不同的域。

  比如[https://zz.zzs7.top和https://s7.zzs7.top](https://zz.zzs7.xn--tophttps-1c2n//s7.zzs7.top)。虽然都属于zzs7.top域名的子域名，但是他们属于不同域。

  想让上述两个网页之间通信，可以通过设置 `document.damain='zzs7.top'`，来让浏览器认为他们在同样的域。两个页面都需设置。

  ```
  <script>
      document.domain = 'zzs7.top';
      // 获取父窗口中变量
  </script>
  ```

  **注：可以进行降域，不能升域。**

- **通过jsonp跨域**：不同源虽然不能读写，但可以引用js。我们可以让引用的js附带数据来调用我们预先定义的函数，而数据当作参数传入。

  ```
  <script>
     var script = document.createElement('script');
     script.type = 'text/javascript';
    
     // 传参一个回调函数名给后端，方便后端返回时执行这个在前端定义的回调函数
     script.src = 'http://zz.zzs7.top/login?user=admin&callback=handleCallback';
     document.head.appendChild(script);
    
     // 回调执行函数
     function handleCallback(res) {
         alert(JSON.stringify(res));
     }
  </script>
  ```

- **CORS：跨域资源共享**：一个W3C标准，它允许浏览器向跨源服务器发起`XMLHttpRequest请求`，和ajax同源的使用方法一致（区别在于对于跨域请求浏览器的请求会有附加的相关设置，用户看不到），分简单请求和非简单请求。

- 通过 **location.hash**：a欲与b跨域相互通信，通过中间页c来实现。 三个页面，不同域之间利用iframe的location.hash传值，相同域之间直接js访问来通信。

- **window.name**：window.name属性的独特之处：name值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 name 值（2MB）。

- **postMessage跨域**：postMessage是HTML5 XMLHttpRequest Level 2中的API，且是为数不多可以跨域操作的window属性之一，它可用于解决以下方面的问题：

  - 页面和其打开的新窗口的数据传递
  - 多窗口之间消息传递
  - 页面与嵌套的iframe消息传递
  - 上面三个场景的跨域数据传递

  用法：postMessage(data,origin)方法接受两个参数
  data： html5规范支持任意基本类型或可复制的对象，但部分浏览器只支持字符串，所以传参时最好用JSON.stringify()序列化。
  origin： 协议+主机+端口号，也可以设置为”*”，表示可以传递给任意窗口，如果要指定和当前窗口同源的话设置为”/“。

还有一些比的，比如可以通过nginx代理实现跨域等，就不在文章中细说了，后续有机会详细实践一下每种方式。

