---
title: "使用Hexo部署自己的博客"
date: 2019-04-25T14:13:26+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

## 为什么使用Hexo来搭建博客

当然作为一个后端程序员，还是python开发，一开始想着的是自己用Django框架可以快速的写一个简单的博客。确实很快，博客列表和详情一早上就搞定了，前端使用的一个简单的模板，修修改改，样式不是很喜欢，但是可选项不多，谁让我自己不会设计，不会写前端呢。没事，可以用就好了，博客最重要的是内容，又不是外表，只能这么安慰自己。

然后求职过程中接到了一个外包，博客就搁浅了，好多想要的功能还没开始写，还是要把接到的任务放在第一位的，毕竟要吃饭的嘛。

写完任务之后，心态有点变了，我在找工作，我还要吃饭，至少工作没稳定之前没有那么多时间，就决定使用现有的博客系统来完成，然后就查一些优劣，应该选择哪种方案，参考了以下文章。



[准备自己建一个个人博客，有什么好的框架推荐？](https://www.zhihu.com/question/24179143)

[个人博客平台的选择](http://git.bookislife.com/post/2015/personal-blog-choosen/)

结合自己目前的需求，最后选择了Hexo。

## Hexo

分为介绍和部署两部分。

***介绍部分内容在文档中都有，我只是按流程做一个简单的教程，以及遇到的问题，详细的建议仔细阅读官方文档。***

### 介绍

Hexo是由中国人写的，有着中文文档的支持。

[hexo中文官网](https://hexo.io/zh-cn/)是这么介绍的：

> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### 安装

#### 依赖

- node.js(需要6.9版本以上)

  根据自己的系统环境安装，官网有对应的安装包，Linux可以直接通过命令行安装。

- git

  Windows：下载并安装 [git](https://git-scm.com/download/win).

  Mac：使用 Homebrew, MacPorts ：brew install git;或下载 [安装程序](https://sourceforge.net/projects/git-osx-installer/) 安装。

  Linux：使用对应的Linux的发行版本的安装命令安装。

#### 安装Hexo

```
npm install -g hexo-cli
```

命令很简单，但是在我运行的时候报错了，报错两次。（讲道理这种情况比较少出现）

- 第一次是报`/usr/lib/node_modules/`的操作权限。

```
sudo chown -R `whoami` /usr/local/lib/node_modules
```

- 第二次是在创建`/usr/bin/hexo`软链接的时候报错了。

可以选择自己手动创建软链接，或者直接使用`sudo`命令安装，我选择`sudo`。（网上不建议这么做，怕安装的npm还是node环境多了有奇怪的异常，但是我是python开发啊，应该不会安装太多的这些环境）。

### 初始化

我要在`/home/zhi/hexo`下创建一个博客站点。

```
cd /home/zhi/hexo # hexo文件夹是我自己新建的，方便管理所有使用hexo创建的站点。目前是空的。
# 初始换
hexo init blog # blog是我需要创建的站点名字，会在当前目录下生成
ls
blog # 生成了对应名字的文件夹
```

blog目录里面生成了需要的文件

```
.
├── _config.yml # 我们主要会做修改的配置文件
├── db.json
├── node_modules
├── package.json
├── package-lock.json
├── public # 生成的静态页面会在这
├── scaffolds # 模板
├── source # 生成的md文章
└── themes # 主题
```

#### 新建文章

新建test的文章

```
hexo new test
```

在source下会生成名为test.md的文件，编写自己的博客内容。

#### 配置

hexo的配置文件为_config.yml，根据自己的需要进行配置。建议阅读hexo的文档，就不在这里赘述了。

### 部署

官方文档提供了几种不同的部署方式，较多的人部署在了github上面。由于github在国外，访问速度大多数情况下比较感人，如果有自己的服务器的话，可以选择部署在自己的服务器上。

我在这里选择使用nginx作为服务器，部署在了[阿里的云服务器中](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=4xhnc6a1)，学生机比较实惠。

### 方案

虽然可以将public中的内容复制到服务器中，使用nginx代理静态文件，但是过程肯定是比较繁琐的，我们需要本地修改了，然后可以同步到服务器中.

![flow](http://zz.zzs7.top/images/blog_flow.png)

### 环境准备

#### 个人PC

在之前的过程中，安装了hexo，还需要安装部署插件。

```
npm install hexo-deployer-git --save
```

#### 服务器环境

我的服务器系统环境是Centos7.4.

安装Git

```
yum install git -y
```

新建 Git 用户

```
useradd -m git
```

设置 gituser 的密码

```
passwd git
```

配置ssh免密登录服务器

1. 生成ssh密钥。

   ```
   ssh-keygen
   ```

2. 将公钥添加到服务器上。user为用户，host为服务器公网IP。

   ```
   ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
   ```

3. 验证是否成功

   ```
   ssh user@host
   ```

初始化一个Git仓库。

新建 /var/repo 目录（位置并不是必须在这），并在该目录下，使用 git init –bare 创建一个名为 blog.git（名字可以自己修改，将以下的对应的内容都做修改就好） 裸仓库，并改变该目录的所有者为 git 用户。

```
mkdir -p /var/repo
cd /var/repo
git init --bare blog.git
chown -R git:git blog.git
```

看看生成的blog.git内的内容

```
blog.git
├── branches
├── config
├── description
├── HEAD
├── hooks # 接下来需要在此处进行一些配置
├── index
├── info
├── objects
└── refs
```

配置Git hooks

进入上面生成的hooks目录，新建名为post-receive的文件，写入下面内容。

```
#!/bin/sh
git --work-tree=/home/www/hexo --git-dir=/var/repo/blog.git checkout -f
```

将post-receive设置为可执行。

```
chmod +x /var/repo/blog.git/hooks/post-receive
```

安装nginx

```
yum install nginx
```

创建需要代理的文件夹，就是你问博客内容需要放在哪。

```
mkdir -p /home/www/hexo      //创建目录
chown -R git:git /home/www/hexo   //更改目录所有者
```

修改配置文件`/etc/nginx/nginx.conf`中以下的内容。将root对应的文件夹改为你上面创建的文件夹。

```
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  _;
    root         /home/www/hexo;

    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

配置`_config.yml`文件。找到deploy选项进行修改。

```
# Deployment
deploy:
- type: git
  repo: your_user_name@HostIP:/var/repo/blog.git
  branch: master
  message:
```

发布自己的博客。

```
hexo clean && hexo d -g
```
