---
title: "了解TOX以及简单使用"
date: 2020-01-25T16:33:48+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

本文是我学习知识的一个记录，只是简单了解使用TOX，，英文阅读能力不错的建议看看[官方文档](https://tox.readthedocs.io/en/latest/)进行深入的学习。

## 什么是TOX

简单来说，TOX是一个虚拟环境管理和命令行测试工具，目标是为了自动化，标准化的进行Python程序的测试。作用如下：

- 使用不同的Python版本和解释器检查项目的包安装是否正确。
- 在每个环境中执行项目的测试工具，比如pytest等。
- 处于CI（持续集成）之前，能大幅减少测试工具所需要的时间。



## 安装

支持目前主流的python版本，以及其他解释器，如： CPython 2.7 and 3.4 or later, Jython-2.5.1, pypy-1.9ff

TOX可以直接安装在虚拟环境中。

- 可以使用pip直接安装(推荐)

```
pip install tox
```

- 使用源码安装

```
cd py_packages # 是你需要将包源码存放的位置，建议和其他使用源码安装的包放在一起统一管理
git clone https://github.com/tox-dev/tox
cd tox
pip install .
tox --version # 安装成功之后可以查看安装的版本
```

## 配置文件

常用的是`tox.ini`格式的配置文件。下面简单介绍一下配置文件：

```
[tox]
# tox部分是全局配置
minversion = 3.4.0  # 定义运行所需的tox的最小版本
envlist = py27,py36  # 需要测试的python环境，会按顺序测试，使用","隔开，下面会说如何定义环境
skipsdist = true  # tox默认会使用sdist构建包，对于测试来说没有必要，而且构建还会要求存在README、setup.py等文件，并且保证setup.py的格式符合要求等，所以跳过此步

[testenv:py27]
commands = python -c 'print "Python2 out! Enjoy Python3 :D!"'

[testenv:py36]
deps =
    -rrequirements.txt
commands =
    pytest

# 如上面的两个"testenv"，:后面的为测试环境的名字
# deps 是安装依赖的命令
# commands 是环境配好之后需要执行的命令
```

## 执行

```
tox
```

或者也可以在命令行中指定测试的环境

```
tox -e py36
```

## 举个例子

写一个简单的例子试试

app.py

```
#!/usr/bin/python3.6
# -*- coding: UTF-8 -*-

def foo_1(a, b):
    return a+b

def foo_2(a, b):
    return f"两个参数分别是{a},{b}"
```

test.py

```
#!/usr/bin/python3.6
# -*- coding: UTF-8 -*-

from app import foo_1, foo_2


def test_foo_1():
    assert foo_1(1, 2) == 3
    assert foo_1("1", "2") == "12"


def test_foo_2():
    assert foo_2(1, 2) == "两个参数分别是1,2"
```

tox.ini

```
[tox]
skipsdist = True
envlist = py36

[testenv:py36]
deps =
    -rrequirements.txt
commands =
    pytest test.py

[pytest]
addopts = -p no:warnings  # 屏蔽测试中的WARNING警告
```

看看执行结果

```
~/PycharmProjects/local » tox                                                                                                                                                                       zhi@bogon
py36 recreate: /Users/zhi/PycharmProjects/local/.tox/py36
py36 installdeps: -rrequirements.txt
py36 installed: atomicwrites==1.3.0,attrs==19.1.0,importlib-metadata==0.17,more-itertools==7.0.0,packaging==19.0,pluggy==0.12.0,py==1.8.0,pyparsing==2.4.0,pytest==4.6.2,six==1.12.0,wcwidth==0.1.7,zipp==0.5.1
py36 run-test-pre: PYTHONHASHSEED='546851202'
py36 run-test: commands[0] | pytest test.py
============================================================================================ test session starts =============================================================================================
platform darwin -- Python 3.6.8, pytest-4.6.2, py-1.8.0, pluggy-0.12.0
cachedir: .tox/py36/.pytest_cache
rootdir: /Users/zhi/PycharmProjects/local, inifile: tox.ini
collected 2 items

# 这是测试进度，后面的"."是测试结果，如果出错，是显示的是F
test.py ..                                                                                                                                                                                             [100%]

========================================================================================== 2 passed in 0.02 seconds ==========================================================================================
__________________________________________________________________________________________________ summary ___________________________________________________________________________________________________
  py36: commands succeeded
  congratulations :)
```

**注意：requirements.txt文件没有的话，可以不写，但是会有WARNING警告**。

