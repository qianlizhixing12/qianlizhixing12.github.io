---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十五周
Date: 2021-04-26
Tags: 网络编程, python库, socket
---

## Algorithm

leetcode 220 存在重复元素III

- [原题](https://leetcode-cn.com/problems/contains-duplicate-iii/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/220.py)
- 保持有序滑动窗口，二分查找

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_62/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_64/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_68/

### 只有代码代表真相

- 查看源代码，程序的意义应该是显而易见的；要知道一个程序做什么，源代码最终是您可以确定的
- 最准确的需求文档也不能说出全部真相，不包含程序实际的详细故事，只包含需求分析人员的高层次意图
- 使代码尽可能简单，便于阅读和理解
- 精心设计代码要表达的内容，尽可能直接地传达代码正在做的事

### 结对编程

- 作为一名团队成员，要对没有比你有经验的开发者有耐心
- 直面你对被更熟练的开发人员吓倒的恐惧，要意识到每个人都是不同的，并重视这一点
- 了解他团队成员的长处和短处，能从同事身上学到东西

### 版本控制

- 版本控制之下，可以跟踪历史，查看谁编写了什么代码
- 可以毫无顾虑地进行大胆的代码更改
- 将项目的所有资产置于版本控制之下，除了源代码，还包括文档、工具、构建脚本、测试用例、图片，库等
- 在单独的操作中提交每个逻辑更改，将许多更改集中在一次提交中，以后很难将它们分开
- 每个提交都附带一条解释信息，至少要简洁地描述您所更改的内容
- 避免提交会破坏项目构建的代码，否则您将不受项目其他开发人员的欢迎

## Tip(dataclass)

- python3去掉了\_\_cmp\_\_魔术方法，使用\_\_eq\_\_和\_\_lt\_\_替代
- dataclass marshmallow pydantic都实现了ORM功能

```python
from dataclasses import dataclass
from queue import PriorityQueue


class Node(object):

    def __init__(self, index, value):
        self.index = index
        self.value = value

    def __cmp__(self, other):
        return self.value >= other.value

    def __str__(self):
        return f'{self.value} {self.index}'


class NodeCmp(object):

    def __init__(self, index, value):
        self.index = index
        self.value = value

    def __eq__(self, other):
        return self.value == other.value

    def __lt__(self, other):
        return self.value < other.value

    def __str__(self):
        return f'{self.value} {self.index}'


@dataclass(order=True)
class NodeDate(object):
    value: int
    index: int


def main(nums, cls):
    q = PriorityQueue()
    for i, num in enumerate(nums):
        # q.put(cls(i, num))
        q.put(cls(num, i))
    while not q.empty():
        print(q.get())
    obj = cls('sss', 2)
    print(obj)


if __name__ == '__main__':
    nums = [3, 6, 7, 1, 5, 9, 8]
    # main(nums, Node)
    # TypeError: '<' not supported between instances of 'Node' and 'Node'
    # main(nums, NodeCmp)
    main(nums, NodeDate)
```

## Share(socket编程)

- socket比较底层，在网络通信中属于网络层(IP/TCP协议)
- 套接字ip:port标识会话双方
- Unix一切皆文件，打开套接字类似打开文件

### 服务端

- 打开套接字，指定协议簇(相当约定使用哪种语言)
- 绑定ip和port，指定身份
- 准备监听port端口
- 接受来自port端口的连接请求(阻断)
- 在连接上接受发送数据

```python
import socket
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def init_server(host, port):
    # pool = ThreadPoolExecutor(max_workers=4)
    pool = ProcessPoolExecutor(max_workers=4)
    # socket.AF_INET, socket.SOCK_STREAM 协议
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        # 绑定套接字
        server.bind((host, port))
        # 监听
        server.listen(5)
        while True:
            conn, addr = server.accept()
            # todo 单独放在一个地方处理该对话(线程等)，不阻断其他连接
            pool.submit(handle_session, conn, addr)
    except Exception as e:
        print(f'error {e}!')
    finally:
        server.close()
        pool.shutdown()


def handle_session(conn, addr):
    while True:
        #接收数据
        data = conn.recv(1024)
        # ?判断客户端是否关闭连接
        if data:
            print(f'recive {addr}: {data.decode()}')
            #发送数据
            conn.send(data.upper())
        else:
            break


if __name__ == "__main__":
    init_server(socket.gethostname(), 8888)
```

### 客户端

- 打开套接字，指定服务端相同协议簇(使用语言)
- 客户端主动发消息，服务端被动应答，没必要指定客户端绑定的接口，系统随机分配
- 主动建立连接，指定服务端ip和port
- 在连接上发送接受数据

```python
import socket
from concurrent.futures import ProcessPoolExecutor


def init_client(host, port, id):
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        client.connect((host, port))
        client.send(f'id:{id} msg:hello'.encode('utf-8'))
        data = client.recv(1024)
        print('recv:', data.decode())
        client.send(f'id:{id} msg:world'.encode('utf-8'))
        data = client.recv(1024)
        print('recv:', data.decode())
    except Exception as e:
        print(f'error {e}!')
    finally:
        client.close()


if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=2) as pool:
        for id in range(1, 4):
            pool.submit(init_client, socket.gethostname(), 8888, id)
```

### demo输出

```shell
# 客户端输出
recv: ID:1 MSG:HELLO
recv: ID:2 MSG:HELLO
recv: ID:1 MSG:WORLD
recv: ID:2 MSG:WORLD
recv: ID:3 MSG:HELLO
recv: ID:3 MSG:WORLD

#服务端输出
recive ('192.168.3.86', 18384): id:1 msg:hello
recive ('192.168.3.86', 18385): id:2 msg:hellorecive ('192.168.3.86', 18384): id:1 msg:world

recive ('192.168.3.86', 18385): id:2 msg:world
recive ('192.168.3.86', 18398): id:3 msg:hello
recive ('192.168.3.86', 18398): id:3 msg:world
```

