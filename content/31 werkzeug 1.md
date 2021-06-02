---
Category: ARTS
Author: qianlizhixing
Title: ARTS第三十一周
Date: 2021-06-07
Tags: 网络编程, python库, yaml, c++ style, werkzeug
---

## Algorithm

leetcode 1744 你能在你最喜欢的那天吃到你最喜欢的糖果吗

- [原题](https://leetcode-cn.com/problems/can-you-eat-your-favorite-candy-on-your-favorite-day/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/1744.py)
- 前缀和

## Review

> https://users.ece.cmu.edu/~eno/coding/CppCodingStandard.html

### 命名

- 名字是对生活的生态进行长期深入思考的结果
- 只有将系统作为一个整体来理解的程序员才能创建一个“适合”系统的名称
- 如果名称合适，那么一切都能自然地组合在一起，关系清晰，含义可以推导出来，从人类的共同期望中推理就能正常工作
- 如果变量表示时间、权重或其他单元，那么在名称中包含该单元，以便开发人员更容易发现问题，例如uint32 mTimeoutMsecs。更好的方法是将一个变量变成一个类，这样就可以捕捉到错误的转换

#### 类名

- 类名中每个单词首字母大写，不要使用下划线连接每个单词
- 超过三个单词的复合名称是一个线索，重新审视你的设计，看看你的对象是否比他们应该有更多的责任
- 避免将派生类的类名带入派生类名的诱惑，一个类应该是独立的，它来自于什么并不重要

#### 类库名

- 使用名称空间来防止来自不同供应商和组的库之间的类名冲突

#### 方法名

- 使用与类名相同的规则
- 每个方法和函数执行一个动作，名称应该清楚它做什么。CheckForErrors()而不是ErrorCheck()，这也将使函数和数据对象更容易区分，类通常是名词，函数名通常是动词
- 后缀有时是有用的
  - Max最大值
  - Cnt计数
  - Key键值
- 前缀有时是有用的
- Is问问题
- Get获取
- Set设置

#### 类属性名

- 属性名应该已小写a开始，再加上其他小写修饰符(p指针point)，后面代词使用与类名相同的规则

#### 方法参数名

- 第一个字符应该是小写，第一个字母之后的所有单词开头都应该是大写

#### 文件扩展名

- 头文件.hh
- 源文件.cc

#### 栈生存周期变量名

- 使用小写字母
- 使用下划线_连接单词

#### 指针变量

- 指针变量应该以前缀p开头
- 将*放在变量名附近，而不是指针类型

#### 引用变量和返回引用的函数

- 引用变量应该以前缀r开头
- 常量引用不应该以前缀r开头

#### 全局变量

- 全局变量应该以前缀g开头

#### 全局常量

- 全局常量应该是所有字母大写，以下划线_连接

#### 静态变量

- 静态变量应该以前缀s开头

#### 类必须方法

- 类应该实现以下方法
- 默认构造函数
- 虚拟析构函数，如果类打算由其他类派生，那么将析构函数设为虚函数
- 拷贝构造函数
- 赋值运算符

### 格式化

- 每行不应超过78个字符
- 在等式不等式比较中，总是把常数放在左边
- 每行应该只有一条语句，除非语句之间关系非常密切
- 每行定义一个变量
- 声明块应该对齐

### 类

- 初始化所有属性
- 不要在对象的构造函数中做任何实际工作。在构造函数内部，只初始化变量和只执行不会失败的操作

#### 类文件

- 每个类的定义应该在它自己的文件中，每个文件都直接以类的名字命名
- 每个类都应该在一个源文件中实现
- 如果源文件太大，或者想避免总是编译模板，按照ClassName_section.C添加额外的文件

### 文档

- 注释视为描述系统的故事，期望机器人提取注释并形成一个手册页，类注释是故事的一部分，方法签名注释是故事的另一部分，方法参数是另一部分，方法实现又是另一部分。所有这些部分应该交织在一起告诉他人到底做了什么，为什么做
- 注释应该记录决策，当需要选择做什么的时候，写下所做的选择和原因
- 显式注释关键词
  - @author模块作者
  - @version模块版本
  - @param函数参数
  - @return函数返回
  - @deprecated函数不再被使用
  - @see创建一个指向文件/函数/变量的链接，以便更好地理解当前代码块的功能
  - @todo还有什么事要做
  - @bug报告错误

## Tip

> https://www.runoob.com/w3cnote/yaml-intro.html

- yaml.YAMLObject中定义yaml_tag，YAMLObjectMetaclass元类在类创建时将类注册到loader，即yaml_tag与类一一对应，继承自YAMLObject的类定义了对象字典信息，实现序列化和反序列化

```python
import yaml


class People(yaml.YAMLObject):
    yaml_tag = u'!People'

    def __init__(self, name, age, sex, addr):
        self.name = name
        self.age = age
        self.sex = sex
        self.addr = addr

    def __repr__(self):
        return "%s(name=%r, age=%r, sex=%r, addr=%r)" % (self.__class__.__name__, self.name, self.age, self.sex, self.addr)


obj = yaml.load("""
!People
name: aaa
age: 16
sex: boy
addr: beijing
""", yaml.loader.FullLoader)
print(obj)
```

## Share(werkzeug源码解析)

### datastructures

- datastructures.py单元定义了基本类型
- 为了实现不可修改，定义了一系列Immutable类，通过覆盖默认的魔术方法(\_\_delitem\_\_,\_\_setitem\_\_等)使修改时抛异常

### Request

- 继承关系werkzeug.wrappers.request.Request-->werkzeug.sansio.request.Request

#### werkzeug.sansio.request.Request

- 方法基本是对wsgi中environ进行解析，使用时更方便（url，access_route，cookies，host，mimetype，user_agent，args等）

  ```python
  # werkzeug.wrappers.request.Request初始化__init__调用super(werkzeug.sansio.request.Request)时入参
  def __init__(
          self,
          environ: "WSGIEnvironment",
          populate_request: bool = True,
          shallow: bool = False,
      ) -> None:
          super().__init__(
              method=environ.get("REQUEST_METHOD", "GET"),
              scheme=environ.get("wsgi.url_scheme", "http"),
              server=_get_server(environ),
              root_path=_wsgi_decoding_dance(
                  environ.get("SCRIPT_NAME") or "", self.charset, self.encoding_errors
              ),
              path=_wsgi_decoding_dance(
                  environ.get("PATH_INFO") or "", self.charset, self.encoding_errors
              ),
              query_string=environ.get("QUERY_STRING", "").encode("latin1"),
              headers=EnvironHeaders(environ),
              remote_addr=environ.get("REMOTE_ADDR"),
          )
          ...
  ```

- 方法大多返回不可修改对象ImmutableMultiDict或ImmutableList

#### werkzeug.wrappers.request.Request

- 根据content-type解析http/https的body(environ["wsgi.input"])

  ```python
  from werkzeug.wrappers import Request, Response
  from werkzeug.test import create_environ
  
  environ = create_environ(path='/foo', base_url='http://localhost:8080/', query_string="id=22&name=aaa", content_type="application/json", json={'param1': '666', 'param2': 'kkk'})
  request = Request(environ)
  for key in list(request.__dict__.keys()):
      print(key, getattr(request, key))
  ```


### Response

- 继承关系werkzeug.wrappers.response.Response-->werkzeug.sansio.response.Response

#### werkzeug.sansio.response.Response

- 和werkzeug.sansio.request.Request对应，封装http协议响应控制，status，headers(主要处理mimetype，content_type)
- 属性status和status_code是同步的，存储在\_status, \_status\_code

#### werkzeug.sansio.response.Response

- 根据headers的content_type处理返回http内容body，存储在response
- Response是由业务绝对内容的，所以header和body必须在\_\_init\_\_中自己指定，不能像Request从environ解析且不可修改，至于get_wsgi_headers，get_app_iter等environ是用来确定Location等内容

### Request和Response配合封装wsgi

- werkzeug.wrappers.request.Request.application和werkzeug.wrappers.response.Response.\_\_call\_\_配合封装WSGI流程

```python
# 服务端
"""
探索werkzeug.wrappers.request.Request werkzeug.wrappers.response.Response使用
"""

import json
from wsgiref.simple_server import WSGIServer, WSGIRequestHandler
from werkzeug import Request, Response


@Request.application
def werkzeug_app(request):
    """
    werkzeug.Request.application将wsgi入口application参数environ转为werkzeug.Request实例request
    被装饰函数werkzeug_app处理业务请求和放回，return werkzeug.Response实例
    werkzeug.Request.application调用werkzeug.Response实例，实例调用__call__内部调用了wsgi回调start_response回写status和headers，本身返回body
    werkzeug.Request.application完成了wsgi原始入参(environ, start_response) -> body到request -> response的转换
    
    进一步，基于werkzeug的框架重点在于路由管理，即werkzeug.Request作为框架入参，根据path method等路由到具体业务，返回werkzeug.Response
    """
    body = {'method': request.method, 'path': request.path, 'query': request.query_string.decode('utf-8')}
    for k, v in request.json.items():
        body[k] = v

    return Response(response=json.dumps(body), status=200, content_type="application/json; charset=utf-8")


def make_server(host, port, app, server_class=WSGIServer, handler_class=WSGIRequestHandler):
    """Create a new WSGI server listening on `host` and `port` for `app`"""
    server = server_class((host, port), handler_class)
    server.set_app(app)
    return server


if __name__ == '__main__':
    with make_server('', 8000, werkzeug_app) as httpd:
        httpd.serve_forever()
        
        
# 客户端
from http.client import HTTPConnection
import json


def client_demo():
    client = HTTPConnection('127.0.0.1', 8000)
    body = json.dumps({'param1': 'xxx', 'param2': 'yyy'}).encode('utf-8')
    headers = {'Content-Type': 'application/json', 'Content-Length': len(body)}
    client.request('POST', '/wsgi_text?user=go', body=body, headers=headers)
    respnese = client.getresponse()
    print(respnese.read().decode('utf-8'))


if __name__ == '__main__':
    client_demo()
```

### 封装Application类

```python
# 官方demo，Shortly相当于Application类，dispatch_request处理路由
class Shortly(object):

    def __init__(self, config):
        self.redis = redis.Redis(config['redis_host'], config['redis_port'])

    def dispatch_request(self, request):
        return Response('Hello World!')

    def wsgi_app(self, environ, start_response):
        request = Request(environ)
        response = self.dispatch_request(request)
        return response(environ, start_response)

    def __call__(self, environ, start_response):
        return self. wsgi_app(environ, start_response)


def create_app(redis_host='localhost', redis_port=6379, with_static=True):
    app = Shortly({
        'redis_host':       redis_host,
        'redis_port':       redis_port
    })
    if with_static:
        app.wsgi_app = SharedDataMiddleware(app.wsgi_app, {
            '/static':  os.path.join(os.path.dirname(__file__), 'static')
        })
    return app
```

