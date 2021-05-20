---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十九周
Date: 2021-05-24
Tags: 网络编程, python库, https
---

## Algorithm

leetcode 421 数组中两个数的最大异或值

- [原题](https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/421.py)
- 前缀树

## Review

> https://www.keycdn.com/blog/http-security-headers
>
> https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy
>
> https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection
>
> https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security

### 加固http安全头

- HTTP安全头通过帮助减轻攻击和安全漏洞提供了另一层安全
- HTTP安全头，它告诉浏览器在处理网站内容时该如何操作

#### Content-Security-Policy

- Content-Security-Policy通过定义经过批准的内容源，从而允许浏览器加载这些内容源，从而帮助防止诸如跨站点脚本(Cross Site Scripting, XSS)和其他代码注入攻击

#### X-XSS-Protection

- X-XSS-Protection被设计用来支持内置在浏览器中的跨站点脚本(XSS)过滤器，即当浏览器检测到xss攻击时的行为

```nginx
# nginx配置
add_header X-XSS-Protection "1; mode=block" always;

# apache配置
header always set X-XSS-Protection "1; mode=block"
```

#### Strict Transport Security

- Strict-Transport-Security限制浏览器只能通过HTTPS访问服务器。这确保了连接不能通过不安全的HTTP连接建立，而不安全的HTTP连接容易受到攻击

#### X-Frame-Options

- X-Frame-Options提供了点击劫持保护，不允许在你的网站上加载iframes

```nginx
# nginx配置
add_header X-Frame-Options "SAMEORIGIN" always;

# apache配置
header always set X-Frame-Options "SAMEORIGIN"
```

#### Expect-CT

- Expect-CT允许网站报告并选择性地强制执行证书透明度要求，从而防止使用错误颁发的证书。当启用此头部时，网站请求浏览器验证证书是否出现在公共CT日志中。

#### X-Content-Type-Options

- X-Content-Type-Options阻止浏览器从声明的Content-Type中嗅探响应

#### Feature-Policy

- Feature-Policy授予允许或拒绝浏览器特性的能力，无论是在它自己的框架中还是在一个内联框架元

## Tip

- python3中OrderedDict是按插入顺序排序，不是按keys或values值大小关系排序

```python
from collections import OrderedDict


def test_ordereddict():
    words, counts = ["dd", "aa", "bb", "cc"], [5, 3, 4, 6]
    dic = OrderedDict(zip(counts, words))
    for k, v in dic.items():
        print(k, v)


if __name__ == "__main__":
    test_ordereddict()

# 输出
# 5 dd
# 3 aa
# 4 bb
# 6 cc
```

## Share(CGI)

### CGI请求流程

- CGI是指WEB server 与 WEB 应用程序之间的通用接口标准, 即: Common Gateway Interface
- 前端通过表单可以向服务器发送一个URL
- 服务器获取到很多参数（get或post请求的参数、服务器和客服端的信息，cookie等等）
- 服务器开启CGI模块将参数存到环境变量中，并调用一个CGI程序
- CGI程序从环境变量中解析出这样请求的具体参数，向标准输出输出内容，发送到了你的浏览器
- 浏览器收到http请求，解析html代码，然后渲染出了网页

### CGI请求缺点

- 大量消耗服务端资源(一般执行CGI脚本，要单独开启一个进程执行)
- 没有做到前后端分层，前端不但要知道任务做什么，还要知道任务怎么做调用相应CGI脚本，后端要返回组装的html内容
- 如果CGI脚本有bug被利用，容易对服务器造成攻击

### http中CGIHTTPRequestHandler处理

- GET请求非cgi程序按照SimpleHTTPRequestHandler处理(获取本地资源)，POST请求非cgi返回501
- cgi资源请求要求host后面跟'/cgi-bin', '/htbin'两个路径之一
- run_cgi处理url先判断服务器目录'/cgi-bin', '/htbin'目录下cgi脚本是否存在，url中?后面为参数
- run_cgi将服务端当前env加上服务信息http请求上下文(headers信息，参数信息)组装
- run_cgi启动子进程运行脚本p = subprocess.Popen(cmdline, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, env=env)

```python
# cgi服务端
import http.server


def open_server():
    server = http.server.HTTPServer(("0.0.0.0", 8088), http.server.CGIHTTPRequestHandler)
    try:
        server.serve_forever()
    except Exception as e:
        print(e)
    finally:
        server.server_close()


if __name__ == "__main__":
    open_server()

# cgi脚本 cgi-bin/math.py
import cgi


def add(*argv):
    return sum(*argv)


def do_exec():
    print("Content-type: text/html")
    print()
    form = cgi.FieldStorage()
    try:
        func_argv = []
        for field in form.list:
            if field.name == "func":
                func_name = field.value
                func_exec = register[func_name]
            else:
                func_argv.append(int(field.value))
        func_result = func_exec(func_argv)
        print(f"{func_name} result: {func_result}")
    except:
        cgi.print_exception()


register = {"add": add}

if __name__ == "__main__":
    do_exec()
```

```http
Request URL: http://127.0.0.1:8088/cgi-bin/math.py?func=add&a=1&b=2
Request Method: GET
Status Code: 200 Script output follows
Remote Address: 127.0.0.1:8088
Referrer Policy: strict-origin-when-cross-origin


HTTP/1.0 200 Script output follows
Server: SimpleHTTP/0.6 Python/3.7.8
Date: Tue, 18 May 2021 15:56:14 GMT
Content-type: text/html

add result: 3
```

