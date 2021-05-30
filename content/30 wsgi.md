---
Category: ARTS
Author: qianlizhixing
Title: ARTS第三十周
Date: 2021-05-31
Tags: 网络编程, python库, wsgi, clang options
---

## Algorithm

leetcode 1707 与数组中元素的最大异或值

- [原题](https://leetcode-cn.com/problems/maximum-xor-with-an-element-from-array/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/1707.py)
- 字典树

## Review

- https://clang.llvm.org/docs/ClangFormatStyleOptions.html

- 样式定义-style=llve或-style=file(查找目录下.clang-format文件,YAML格式)
- 自动基于文件名检测编程语言，-assume-filename=强制指定编程语言
- 源码// clang-format off到// clang-format on之间代码不被格式化
- AlignAfterOpenBracket换行时对齐方式
- AccessModifierOffset访问修饰符缩进控制(private,public)
- AlignConsecutiveAssignments连续任务对齐方式
- AlignConsecutiveBitFields
- AlignConsecutiveDeclarations连续变量声明对齐方式
- AlignConsecutiveMacros连续宏定义对齐方式

## Tip

## Share(WSGI)

> https://www.python.org/dev/peps/pep-0333/
>
> https://www.python.org/dev/peps/pep-3333/

- WSGI服务器[网关](https://baike.baidu.com/item/网关)接口（Python Web Server Gateway Interface)是Python应用程序或框架和Web服务器之间的一种接口协议，类似于Java中的servlet API，让服务器和业务分离解耦
- WSGI接口有两个方面:服务器网关端，应用程序框架端。服务器端调用由应用程序端提供的可调用对象。提供该对象的具体方式取决于服务器或网关。假设一些服务器或网关需要应用程序的部署人员编写一个简短的脚本来创建服务器或网关的实例，并为其提供应用程序对象。其他服务器和网关可能使用配置文件或其他机制来指定应该从何处导入应用程序对象，或以其他方式获取应用程序对象
- 除了“纯”服务器/网关和应用程序/框架之外，还可以创建“中间件”组件来实现该规范的两个方面。这些组件充当包含它们的服务器的应用程序和包含它们的应用程序的服务器，可以用来提供扩展的api、内容转换、导航和其他有用的功能
- 实现可调用对象的服务器、网关或应用程序可以根据自己的需要选择适当的实现技术。相反，正在调用可调用对象的服务器、网关或应用程序必须不依赖于提供给它的可调用对象的类型。可调用对象只被调用，而不是被自省

### WSGI处理

- WSGIServer继承HTTPServer，WSGIRequestHandler继承BaseHTTPRequestHandler，继承了http库的风格，tcp层(socket)和http层分离
- WSGIRequestHandler仅仅使用了基类parse_request校验处理http协议版本，http头headers功能，就将流程转给ServerHandler.run。按道理ServerHandler和WSGIRequestHandler合二为一像CGIHTTPRequestHandler一样继承BaseHTTPRequestHandler处理http协议层应该是没问题的，ServerHandler也确实有大量处理http协议的代码，应该是为了处理WSGI细节和WSGI和CGI的转化
- 对业务开发来讲，application唯一入口WSGIServer，注入application启动WSGIServer，遵守协议处理业务需求。所以application从WSGIServer一直传递到ServerHandler.run中，待application处理完业务后，将处理结果状态通过回调start_response给ServerHandler.status，ServerHandler.headers，将处理结果数据通过返回结果给ServerHandler.result，ServerHandler负责http协议层返回
- ServerHandler传给application的env信息和start_response回调，env都包含那些内容?WSGIServer.setup_environ添加tcp层服务器和端口信息，WSGIRequestHandler.get_environ添加http层本次请求的方法资源路径头信息等，ServerHandler.setup_environ添加WSGI协议信息
- environ["PATH_INFO"]，environ["QUERY_STRING"]，environ["wsgi.input"]包含了业务数据

```python
# 服务端
from wsgiref.simple_server import WSGIServer, WSGIRequestHandler
import json


def demo_app(environ, start_response):
    body = {'method': environ["REQUEST_METHOD"], 'path': environ["PATH_INFO"], 'query': environ["QUERY_STRING"]}
    params = json.loads(environ["wsgi.input"].read(int(environ["CONTENT_LENGTH"])).decode('utf-8'))
    for k, v in params.items():
        body[k] = v

    from io import StringIO
    stdout = StringIO()
    print("request info:", file=stdout)
    print(file=stdout)
    print(json.dumps(body), file=stdout)

    start_response("200 OK", [('Content-Type', 'text/plain; charset=utf-8')])
    return [stdout.getvalue().encode("utf-8")]


def make_server(host, port, app, server_class=WSGIServer, handler_class=WSGIRequestHandler):
    """Create a new WSGI server listening on `host` and `port` for `app`"""
    server = server_class((host, port), handler_class)
    server.set_app(app)
    return server


if __name__ == '__main__':
    with make_server('', 8000, demo_app) as httpd:
        httpd.serve_forever()
        
# 客户端
from http.client import HTTPConnection
import json


def client_demo():
    client = HTTPConnection('127.0.0.1', 8000)
    body = json.dumps({'param1': 'xxx', 'param2': 'yyy'}).encode('utf-8')
    headers = {'trace': 'zzz', 'Content-Length': len(body)}
    client.request('POST', '/wsgi_text?user=go', body=body, headers=headers)
    respnese = client.getresponse()
    print(respnese.read().decode('utf-8'))


if __name__ == '__main__':
    client_demo()
    
# output
# request info:

# {"method": "POST", "path": "/wsgi_text", "query": "user=go", "param1": "xxx", "param2": "yyy"}
```

