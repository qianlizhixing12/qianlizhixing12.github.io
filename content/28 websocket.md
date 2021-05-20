---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十八周
Date: 2021-05-17
Tags: 网络编程, python库, websocket
---

## Algorithm

leetcode 1734 解码异或后的排列

- [原题](https://leetcode-cn.com/problems/decode-xored-permutation/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/1734.py)
- 位运算

## Review(websocket)

> http://www.websocket.org/quantum.html

### 轮询、长轮询、流方式

- 通常，当浏览器访问一个网页时，一个HTTP请求被发送到承载该网页的web服务器,web服务器确认这个请求并发回响应。如果想获得最新的实时信息，可以不断手动刷新页面，但这显然不是一个很好的解决方案。
- 目前提供实时web应用程序的尝试主要围绕轮询和其他服务器端推技术，最值得注意的是Comet，它延迟了向客户机传递消息的HTTP响应的完成。基于comet的推送通常用JavaScript实现，并使用长轮询或流媒体等连接策略。
- 通过轮询，浏览器定期发送HTTP请求并立即接收响应，这种技术是浏览器提供实时信息的第一次尝试。如果知道确切的消息传递间隔，这是一个很好的解决方案。然而，实时数据往往不是那么可预测的，使得不必要的请求不可避免，因此，许多连接在低消息率的情况下被不必要地打开和关闭。
- 使用长轮询，浏览器向服务器发送请求，服务器将请求保持打开一段时间。如果在这段时间内收到通知，将向客户端发送包含消息的响应。如果在设置的时间内没有收到通知，服务器将发送一个响应来终止打开的请求。但是，重要的是要理解，当消息量很大时，长轮询与传统轮询相比并没有提供任何实质性的性能改进。事实上，情况可能会更糟，因为长期投票可能会失去控制，变成一个不受控制的、连续的即时投票循环。
- 使用流媒体，浏览器发送一个完整的请求，但服务器发送并维护一个打开的响应，该响应不断更新并无限期地(或在一段时间内)保持打开状态。然后，每当准备发送消息时，响应就会更新，但是服务器永远不会发出完成响应的信号，因此保持连接打开以交付未来的消息。但是，由于流媒体仍然封装在HTTP中，中间的防火墙和代理服务器可能会选择缓冲响应，从而增加了消息传递的延迟。因此，如果检测到缓冲代理服务器，许多流Comet解决方案会退回到长轮询。或者，可以使用TLS (SSL)连接来避免缓冲响应，但在这种情况下，每个连接的建立和拆除会增加可用服务器资源的负担。
- 最终，所有这些提供实时数据的方法都涉及到HTTP请求和响应头，这些头包含了大量额外的、不必要的头数据，并引入了延迟。最重要的是，全双工连接需要的不仅仅是从服务器到客户机的下行连接。为了在半双工HTTP上模拟全双工通信，今天的许多解决方案使用两个连接:一个用于下游，一个用于上游。这两个连接的维护和协调带来了巨大的资源消耗开销，并增加了很多复杂性。简单地说，HTTP不是为实时、全双工通信而设计的，如下图所示，这显示了构建使用半双工HTTP上的发布/订阅模型从后端数据源显示实时数据的Comet web应用程序的复杂性。

### HTML5 WebSocket

- HTML5 Web Sockets代表了Web通信的下一个进化——通过Web上的单个套接字操作的全双工、双向通信通道。HTML5 Web Sockets提供了一个真正的标准，你可以用它来构建可伸缩的实时Web应用程序。此外，由于它提供了浏览器本地的套接字，因此消除了Comet解决方案容易出现的许多问题。Web Sockets消除了开销并显著降低了复杂性。
- 目前，只有谷歌的Chrome浏览器原生支持HTML5 Web Sockets，但其他浏览器很快也会跟进。然而，为了解决这个限制，Kaazing WebSocket Gateway为所有老式浏览器(即5.5+，Firefox 1.5+， Safari 3.0+和Opera 9.5+)提供了完整的WebSocket模拟，所以你现在就可以开始使用HTML5 WebSocket api了。

## Tip(inspect.ismethod)

- python中函数和方法的不同，函数可以独立存在，方法和对象绑定在一起
- inspect.ismethod判断方法时，注意对象
- 方法带有\_\_self\_\_，即绑定的对象

## Share(websocket建立连接)

### http websocket对比

- http只能是客户端发起请求，服务端被动响应；因此如果要实时获取服务端某些数据，客户端只能轮询发请求
- websocket弱化了客户端服务端的区别，双方可以主动发信息
- 二者都是构建在tcp协议之上，实现在socket编程之上

### websocket建立连接

#### 客户端消息格式

- "ws:" "//" host [ ":" port ] path [ "?" query ] 默认 port 端口 80
- "wss:" "//" host [ ":" port ] path [ "?" query ] 默认 port 端口 443
- 客户端提供信息（放在http或https头headers里面）
  - host 与 port：主机名与端口
  - shema：是否基于 SSL
  - 访问资源：URI
  - 握手随机数：Sec-WebSocket-Key
  - 选择子协议： Sec-WebSocket-Protocol
  - 扩展协议： Sec-WebSocket-Extensions
  - CORS 跨域：Origin

#### 服务端消息格式

- HTTP/1.1 101 Web Socket Protocol Handshake // 101连接建立成功
- Connection: upgrade
- 握手随机数：Sec-WebSocket-Key
- Upgrade: websocket

```python
import os
import socket
from base64 import encodebytes as base64encode
import websocket


def do_request_():
    # ws = websocket.create_connection("wss://echo.websocket.org")
    ws = websocket.create_connection("ws://echo.websocket.org")
    ws.send("test aaa!")
    ws.send("test bbb!")
    print(ws.recv())
    print(ws.recv())


def _create_sec_websocket_key():
    randomness = os.urandom(16)
    return base64encode(randomness).decode('utf-8').strip()


def get_headers():
    websocket_key = _create_sec_websocket_key()
    print(websocket_key)
    print("----")
    headers = {"Host": "echo.websocket.org", "Sec-WebSocket-Version": "13",
               "Sec-WebSocket-Key": websocket_key, "Connection": "keep-alive, Upgrade",
               "Sec-WebSocket-Extensions": "permessage-deflate", "Upgrade": "websocket"}
    headers = (key + ": " + value for key, value in headers.items())
    return "GET / HTTP/1.1\r\n" + "\r\n".join(headers)


def do_request():
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        headers = get_headers()
        # client.connect(("echo.websocket.org", 80))
        client.connect(("121.40.165.18", 8800))
        client.send(headers.encode('utf-8'))
        data = client.recv(1024)
        print(data.decode())
    except Exception as e:
        print(f'error {e}!')
    finally:
        client.close()


if __name__ == "__main__":
    do_request()

   
# 输出
# Sec-WebSocket-Accept:  gk6AiBD48FKZC2RUFMH/ow==
# ----
# HTTP/1.1 101 Switching Protocols
# Upgrade: websocket
# Connection: Upgrade
# Sec-WebSocket-Accept: e5/mWaJy8r13Fsq0la9Rql8R27k=
```

