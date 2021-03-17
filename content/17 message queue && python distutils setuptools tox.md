---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十七周
Date: 2020-08-17
Tags: 消息队列, distutils, setuptools, tox
---

## Algorithm

leetcode 338 比特位计数

- [原题](https://leetcode-cn.com/problems/counting-bits/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/338.c)

## Review

> https://docs.oracle.com/cd/E19435-01/819-0069/intro.html

#### 消息队列架构

- 消息队列是一种可靠的异步消息传递服务
- 消息服务器由一个或多个代理组成，并执行消息路由和传递，它是消息队列服务的核心
- 消息服务器由一个或一组共同工作的代理(作为代理集群)组成，以执行消息路由和传递服务
  - 身份验证，操作授权
  - 设置和客户端通信协议
  - 接受客户端消息，并将其放置在各自的物理目的地
  - 将消息路由并交付给一个或多个消费客户端
  - 保证可靠的交付
  - 为监控系统性能提供数据
- 客户端运行时代理
  - 管理到消息服务器的消息传递
  - 建立连接，建立客户的身份
  - 实现客户端确认
  - 控制跨连接的消息流
  - 可以覆盖由生成客户端设置的消息头值

#### 消息传递步骤

- 持久、可靠地传递的消息步骤
  - 消息生产
    - 客户端运行时通过连接将消息从消息生成器传递到消息服务器
  - 消息处理和路由
    - 消息服务器从连接中读取消息并将其放置到适当的目的地
    - 消息服务器将(持久的)消息放在数据存储中
    - 消息服务器向消息生成器的客户端运行时确认收到消息
    - 消息服务器决定消息的路由
    - 消息服务器将消息从其目的地写到适当的连接
  - 消息消费
    - 消息使用者的客户端运行时将连接中的消息传递给消息使用者
    - 消息使用者的客户端运行时向消息服务器确认消息的使用
  - 消息临终
    - 消息服务器处理客户端确认，从其目的地和数据存储中删除(持久)消息
    - 消息服务器向使用者的客户端运行时确认，客户端确认已被处理，且消息无法再次传递
- 消息分类
  - 消息本体，由生产客户端发送到消费客户端的JMS消息
  - 消息控制确认，在消息服务器和客户端运行时之间传递确认和其他非有效负载消息，以确保有效负载消息成功传递，并控制连接之间的消息流

#### 消息传递模型

- 点到点
  - 从生产者传递到单个使用者
  - 消息首先传递到队列目的地(消息服务器)，然后从队列一次传递一个消息，传递到为队列注册的使用者之一
  - 任意数量的生产者都可以将消息发送到队列目的地，但是每个消息都保证只能传递给一个消费者并被一个消费者成功使用
  - 如果没有使用者为队列目的地注册，则队列保存它接收到的消息，并在使用者为队列注册时传递这些消息
- 发布/订阅
  - 从生产者传递到任意数量的使用者
  - 消息首先被传递到主题目的地，然后传递到订阅了主题的所有活动使用者
  - 任意数量的生产者都可以将消息发送到主题目的地，并且每个消息都可以发送到任意数量的订阅使用者

#### 消息推送模式

- Push模式，由消息中间件主动地将消息推送给消费者
  - consumer把轮询过程封装了，并注册MessageListener监听器，取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而言，感觉消息是被推送过来的
  - 需要一个中心节点，负责消息的分配情况（哪段消息分配给consumer1，哪段消息分配给consumer2），同时还要监听consumer的ack消息用于判断消息是否处理成功，如果在timeout时间内为收到响应可以认为该consumer挂掉，需要重新分配sonsumer上失败的消息
  - 容易冲垮consumer，虽然可以向amq那样使用prefetch limit  和预留缓冲区 ，但是还是不能解决由consumer的选择性消费
- Pull模式，由消费者主动向消息中间件拉取消息
  - 取消息的过程需要用户自己写，根据offset取得消息
  - consumer决定消息的消费情况，这种模式有一个好处是我们不需要返回ack消息，因为当consumer申请消费下一批消息时就可以认为上一批消息已经处理完毕，也不需要处理超时的问题，consumer可以根据自己的消费能力来消费消息
  - pull最大问题是实时性问题，对于现在的MQ系统，对latency的追求越来越高，这个时候纯粹的Pull模式将满足不了实时性方面的需求

## Tip

## Share

> https://mp.weixin.qq.com/s?__biz=MzUyOTk2MTcwNg==&mid=2247485029&idx=1&sn=a986bf7b7d7003f637cfccc6428e6310&scene=21#wechat_redirect
>
> https://mp.weixin.qq.com/s?__biz=MzUyOTk2MTcwNg==&mid=2247485034&idx=1&sn=b4a039013e3265c04700c48b1b338e06&scene=21#wechat_redirect
>
> https://mp.weixin.qq.com/s/Kmqhr9szPdvBI0KQPm9JsA

### distutils

> https://docs.python.org/zh-cn/3.7/distutils/introduction.html
>
> https://blog.csdn.net/weixin_36670529/article/details/102794001

distutils可以用来在Python环境中构建和安装额外的模块。新的模块可以是纯Python的，也可以是用C/C++写的扩展模块，或者可以是Python包，包中包含了由C和Python编写的模块。

#### 模块构建流程

- 编写setup脚本（一般是setup.py）
- 编写一个setup配置文件（可选）
- 创建一个源码发布
- 创建一个或多个构建（二进制）发布（可选）

##### setup脚本

```python
from distutils.core import setup

setup(
    name='aaa',
    version='1.0.0',
    include_package_data=True,
    packages=['aaa', 'aaa.bbb'],
    package_dir={'aaa': 'src'},
    package_data={
        'aaa': ['bbb/*.config'],
    },
    data_files=[
        ('config', ['src/bbb/ccc.config']),
    ],
)
```

- name 包名
- version 版本
- packages 导入包(包下所有py文件)
- package_dir 包所在目录
- package_data 静态文件(复制到相对目录)
- data_files 静态文件(安装到指定python系统目录)

##### 源码发布

```bash
python setup.py sdist
```

sdist命令会创建一个archive 文件（比如Unix上的tar文件，Windows上的zip文件），它包含setup.py， 源码文件。该archive文件命名为aaa-1.0.0.tar.gz(zip)

##### 安装模块

用户希望安装aaa模块，他只需要下载aaa-1.0.0.tar.gz，解压，进入aaa-1.0.0目录，然后运行

```bash
python setup.py install
```

该命令最终会将foo.py复制到Python环境存放第三方模块的目录中

##### 构建发布

```bash
python setup.py bdist
```

### setuptools

> https://www.jianshu.com/p/ea9973091fdf

setuptools是Python distutils增强版的集合，它可以帮助我们更简单的创建和分发Python包，尤其是拥有依赖关系的。

#### 模块构建流程

- 编写setup脚本（一般是setup.py）
- 编写一个setup配置文件（可选）
- 打包
- 安装

##### setup脚本

```python
from setuptools import setup, find_packages

setup(
    name='aaa',
    version='1.0.0',
    include_package_data=True,
    packages=find_packages('src'),
    install_requires=[
        'bbb>=0.10',
    ],
    # package_dir={'nxx': 'src'},
    # package_data={
    #     'nxx': ['log/*.config'],
    # },
    # data_files=[
    #     ('config', ['src/log/log.config']),
    # ],
)
```

- name 包名
- version 版本
- include_package_data = True，启用清单文件MANIFEST.in包含数据文件
- packages 导入包(包下所有py文件)，find_packages(src)自动找到src目录下所有包
- install_requires 自动安装依赖
- package_dir 包所在目录
- package_data 静态文件(复制到相对目录)
- data_files 静态文件(安装到指定python系统目录)

##### 打包

```bash
python setup.py bdist_egg
```

当前目录下的dist目录内创建一个egg文件，名为aaa-1.0-py3.7.egg（zip压缩文件）， 文件名格式就是”应用名-版本号-Python版本.egg”。

当前目录多了”build”和”MyApp.egg-info”子目录来存放打包的中间结果。

##### 安装

```bash
easy_install aaa-1.0-py3.7.egg
# 或者
easy_install aaa-1.0-py3.7.wheel
# 或者
pip install aaa-1.0-py3.7.wheel
# 或者
python setup.py install
```

### 通用命令

- build 构建
- sdist 源码分发
- install 源码安装
- bdist 可执行文件分发
- bdist_egg egg文件分发
- bdist_wheel wheel文件分发

### tox

> https://zhuanlan.zhihu.com/p/101330325

支持创建隔离的 Python 环境，在里面可以安装不同版本的 Python 解释器与各种依赖库，以此方便开发者做自动化测试、打包、持续集成等事情。

#### 配置文件

- pyproject.toml
- tox.ini
- setup.cfg

##### tox.ini

- [tox]下面是全局性的配置项，envlist 字段定义了 tox 操作的环境
- [xxx]下面是 xxx 虚拟环境的配置项，[xxx:yyy]继承 xxx 的配置，同时其自身配置项的优先级更高
- 常用的配置项有：description（描述信息）、basepython（Python解释器版本）、deps（环境依赖项）、commands（命令语句）等等
- 内置的基础变量{toxinidir}、{homedir}、{envname}、{envdir}
- 操作系统的环境变量：{env:KEY}，效果等同于`os.environ['KEY']`

#### 工作流程

- 配置（figuration）：加载配置文件（如 tox.ini），解析命令行参数，读取系统环境变量等
- 打包（packaging）：可选的，对于带有 setup.py 文件的项目，可以在这步去生成它的源发行版
- 创建虚拟环境：默认使用 virtualenv 来创建虚拟环境，并根据配置项中的“deps”安装所需的依赖项，然后执行配置好的命令（commands）
- 报告（report）：汇总所有虚拟环境的运行结果并罗列出来