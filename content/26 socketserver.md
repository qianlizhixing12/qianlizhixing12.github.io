---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十六周
Date: 2021-05-03
Tags: 网络编程, python库, socketserver
---

## Algorithm

leetcode 377 组合总和Ⅳ

- [原题](https://leetcode-cn.com/problems/combination-sum-iv/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/377.py)
- 缓存中间结果，动态规划

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_88/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_90/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_92/

### Unix工具是你的朋友

- ide以特定的语言为目标，Unix工具可以处理任何以文本形式出现的内容
- 每年都会涌现出新的语言和符号，学习以Unix的方式工作是一项投资，而且会不断获得回报
- 通过组合小型但通用的Unix工具就可以创建自己的命令

### 冗长的日志记录

- 过多的日志记录可能就像没有日志一样毫无用处
- 对于许多系统，出现问题的第一个指示是将一条日志消息写入某个日志
- 杂乱的日志表明一旦系统进入生产状态，系统将难以控制

### 程序员和测试人员协作时

- 在缺陷跟踪系统中来回发送bug所花费的时间更少
- 更少的时间被浪费在试图找出某个东西到底是一个bug还是一个新特性上
- 更多的时间被花在开发好软件以满足客户的期望上
- 在开始编写代码之前，就有很多开始协作的机会

### 客户并不能总是理解自己所说的

- 我还从未见过一个顾客不太乐意告诉我他们想要什么——通常是非常详细的。问题是顾客并不总是告诉你全部的真相。他们通常不会撒谎，但他们是以客户的方式说话，而不是以开发人员的方式。他们使用自己的术语和上下文。他们遗漏了重要的细节。他们会假设你已经在他们公司工作了20年，就像他们一样。这是由于许多客户实际上不知道他们想要什么在第一个地方!有些人可能对“大局”有所了解，但他们很少能够有效地沟通愿景的细节。其他人可能对完整的愿景不太重视，但他们知道自己不想要什么。那么，您怎么可能将一个软件项目交付给那些没有告诉您他们想要什么的全部真相的人呢?这是相当简单的。多和他们互动。
- 尽早挑战你的客户，并经常挑战他们。不要简单地重申他们对你说的话。记住:他们的意思不是他们告诉你的。我经常这样做，在与他们交谈时，我和他们交换词语，判断他们的反应。你会惊讶地发现，“客户”这个术语有多少次与“客户”有完全不同的含义。然而，告诉你他想要在他的软件项目中做什么的人会交替使用这些术语，并希望你跟踪他说的是哪一个。你会感到困惑，你编写的软件也会受到影响。
- 在你决定了解客户的需求之前，和他们多次讨论一些话题。试着和他们重复两三次这个问题。和他们谈谈刚刚发生在你谈论的话题之前或之后的事情，以获得更好的背景。如果可能的话，让很多人在不同的对话中告诉你同一个话题。他们几乎总是会告诉你不同的故事，而这些故事将揭示独立但相关的事实。两个人告诉你相同的话题经常会互相矛盾。你成功的最好机会是在你开始你的超级复杂的软件制作之前消除差异。
- 在对话中使用视觉辅助。这可以像在会议中使用白板一样简单，也可以像在设计阶段早期创建视觉模型那样简单，也可以像制作功能原型那样复杂。众所周知，在谈话中使用视觉辅助可以延长我们的注意力持续时间，增加信息的留存率。利用这一事实，为你的项目成功做好准备。
- 在过去的生活中，我是一个团队的“多媒体程序员”，制作的项目都很耀眼。我们的一个客户详细描述了他们对项目外观和感觉的想法。在设计会议上讨论的一般配色方案表明了展示的黑色背景。我们以为已经搞定了。图形设计师团队开始制作数百个分层的图形文件。大量的时间都花在了最终产品的塑造上。在我们向客户展示我们的劳动成果的那一天，一个令人吃惊的发现出现了。当她看到这个产品时，她对背景颜色的准确描述是“当我说黑色时，我指的是白色。”所以，你看，它永远不会像黑与白那样清晰。

## Tip

### 为什么要进行 URI 编码

- 传递数据中，如果存在用作分隔符的保留字符怎么办？
- 对可能产生歧义性的数据编码
- 不在 ASCII 码范围内的字符
- ASCII 码中不可显示的字符
- URI 中规定的保留字符
- 不安全字符（传输环节中可能会被不正确处理），如空格、引号、尖括号等

### 保留字符与非保留字符

- 保留字符
  - reserved = gen-delims / sub-delims
    - gen-delims = ":" / "/" / "?" / "#" / "[" / "]" / "@"
    - sub-delims = "!" / "$" / "&" / "'" / "(" / ")" / "*" / "+" / "," / ";" / "="
- 非保留字符
  - unreserved = ALPHA / DIGIT / "-" / "." / "_" / "~"
    - ALPHA: %41-%5A and %61-%7A
    - DIGIT: %30-%39
    - -: %2D .: %2E _: %5F
    - ~: %7E，某些实现将其认为保留字符

### 百分号编码的方式

- pct-encoded = "%" HEXDIG HEXDIG
  - US-ASCII：128 个字符（95 个可显示字符，33 个不可显示字符）
- 非ASCII码字符（例如中文）：建议先 UTF8 编码，再 US-ASCII 编码
- 对URI合法字符，编码与不编码是等价的

## Share(socketserver源码分析)

socketserver用于建立服务端，讨论TCPServer，一步步简化代码，分析源码

#### 建立连接

- \_\_init_\_，server_bind，server_activate对应打开套接字，绑定ip和port，监听端口

#### 关闭连接

- handle_request处理一段请求请求后，关闭连接
- serve_forever启动监听，采用多路复用提高吞吐量，shutdown用来关闭连接；serve_forever阻断当前线程，监听selector，shutdown必须在另一个线程设置__shutdown_request，两个线程采用threading.Event通信

```python
import service as socketserver
from concurrent.futures import ThreadPoolExecutor
import time


def run(socket):
    print('run begin')
    socket.serve_forever(5)
    print('run end')


def shutdown(socket):
    print('shutdown begin')
    time.sleep(1 * 60)
    print('shutdown exec')
    socket.shutdown()
    print('shutdown end')


def main():
    socket = socketserver.TCPServer(('0.0.0.0', 8081), socketserver.BaseRequestHandler)
    with ThreadPoolExecutor(4) as pool:
        pool.submit(run, socket)
        pool.submit(shutdown, socket)


if __name__ == '__main__':
    main()

# 输出
# run begin
# shutdown begin
# shutdown exec
# run end
# shutdown end
```

#### 处理请求

- 从selector接受到信号后，调用_handle_request_noblock处理连接请求
- _handle_request_noblock调用get_request(连接可能是tcp或udp)获取请求数据
- _handle_request_noblock调用process_request实际处理请求，失败时调用handle_error，shutdown_request关闭请求

#### 处理请求详细

- process_request先调用finish_request正常处理请求后，再调用shutdown_request关闭请求
- finish_request调用创建时RequestHandlerClass类，创建实例处理请求，大多设计都将接受请求数据和处理请求数据分开到两个类，解耦
- process_request可以放在线程或进程中处理提高并发处理

#### RequestHandler

- BaseRequestHandler流程很清晰在构造中\_\_init\_\_中执行setup，handle，finish，执行完即销毁
- 默认提供BaseRequestHandler及子类的handle没有处理过程，自己必须实现
- 实现一个简单http请求返回demo

```python
# 服务端
import service as socketserver


class HttpRequestHandle(socketserver.StreamRequestHandler):

    def handle(self):
        data = self.request.recv(1024)
        if data:
            print(data)
        data += b'ok'
        resp = b'HTTP/1.1 200 OK\r\ncontent-type: application/json\r\ndate: Wed, 28 Apr 2021 16:20:04 GMT\r\nserver: socketserver\r\n\r\n{"code":"0000","message":"ok","data":{}}'
        self.request.send(resp)


def service_demo():
    print('service begin')
    socket = socketserver.TCPServer(('0.0.0.0', 8081), HttpRequestHandle)
    socket.serve_forever(1)
    print('service end')


if __name__ == '__main__':
    service_demo()

# 客户端
import http.client
import json


def client_demo():
    client = http.client.HTTPConnection('127.0.0.1', 8081)
    headers = {'test_head': 'nxx'}
    client.request('GET', '/index.html', headers=headers)
    respnese = client.getresponse()
    print(json.loads(respnese.read()))


if __name__ == '__main__':
    client_demo()


# 服务端输出
# service begin
# b'GET /index.html HTTP/1.1\r\nHost: 127.0.0.1:8081\r\nAccept-Encoding: identity\r\ntest_head: nxx\r\n\r\n'

# 客户端输出
# {'code': '0000', 'message': 'ok', 'data': {}}
```

