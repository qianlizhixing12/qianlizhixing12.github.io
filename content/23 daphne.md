---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十三周
Date: 2021-04-12
Tags: ASGI, daphne, django, nginx, 数据采集
---

## Algorithm

leetcode 153 寻找旋转排序数组中的最小值

- [原题](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/153.py)
- 二分查找

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_53/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_54/

### 链接器不是魔法

- 链接器所做的就是将目标文件的代码和数据部分连接在一起，将符号的引用与它们的定义连接起来，从库中提取无法解析的符号，并写出一个可执行文件
- 映射文件是可执行文件中所有符号及其地址的列表。这告诉从库中链接了哪些模块，以及每个模块的大小

### 临时方案寿命

- 创建临时方案的原因多种多样，临时方案成功的关键很简单，它是有用的
- 临时解决方案会产生惯性，它们存在，最终是有用的并且被广泛接受的，所以不需要立即解决临时方案的弊端。每当必须决定什么行动最能增加价值时，就会有许多比适当集成临时解决方案更高的行动。为什么?因为它就在那里，它有效，它被接受。唯一的缺点是它不遵循所选择的标准和指导方针-除了少数利基市场，这并不被认为是一个重要的力量
- 如果临时解决方案出现问题，可能会创建新的临时方案解决这个临时方案的问题
- 当系统包含过多的临时解决方案时，其熵值或内部复杂性增加，可维护性降低；然而，首先问这个问题可能是错误的；记住，我们讨论的是一个解决方案，它可能不是你喜欢的解决方案——它不太可能是任何人喜欢的解决方案——但重做这个解决方案的动机很弱
- 应该怎么做
  1. 一开始就避免创建临时解决方案，回避在大多数地方都不起作用
  2. 改变影响项目经理决策的力量
  3. 让它保持原样
- 克服临时解决方案的最佳方法是使它们成为多余的，提供一个更优雅和有用的解决方案

## Tip

### 调试开关与静态文件

> https://stackoverflow.com/questions/5836674/why-does-debug-false-setting-make-my-django-static-files-access-fail

- 调试模式下(DEBUG=True)，默认提供静态文件服务

- 生产模式下(DEBUG=False)，默认不提供静态文件服务

  - 由nginx,apache提供文件服务

  - 改变运行方式

    ```bash
    python manage.py runserver --insecure
    ```

  - 添加静态文件路由

    ```python
    from django.views import static
    from django.conf import settings
    from django.conf.urls import url
    
    url(r'^static/(?P<path>.*)$',
            static.serve, {'document_root': settings.STATIC_ROOT},
            name='static'),
    ```

### nginx+daphne部署异步django项目

> https://www.upidev.com/tutoriels/set-up-a-django-async-project-with-daphne-nginx-on-ubuntu-18-04/

![django_with_daphne_nginx](.\img\django_with_daphne_nginx.png)

请求响应链路图

1. 启动django

   ```bash
   daphne --proxy-headers mydjango.asgi:application
   ```

2. 域名服务

   ```ini
   # C:\Windows\System32\drivers\etc\hosts文件加一行
   127.0.0.1       myspider.com
   ```

3. 配置nginx

   ```nginx
   # nginx.conf
   
   server {
          listen 80;
          server_name myspider.com;
          
          charset     utf-8;
          
          client_max_body_size 75M;   # adjust to taste
   
          # Django media
          location /static/ {
              alias D:\myspider\mydjango\static\\; 
          }
   
          location / {
             try_files $uri @proxy_to_app;
          }
   
          location @proxy_to_app {
              proxy_pass http://127.0.0.1:8000;
   
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection "upgrade";
   
              proxy_redirect off;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Host $server_name;
          }
       }
   ```

## Share

> 数据采集比较全面的介绍
>
> https://lovelybear.blog.csdn.net/article/details/73927750
>
> https://www.zhihu.com/question/28168585

### nginx反采集

```python
import requests
from fake_useragent import UserAgent
import json


def spider_run():
    headers = {
        'Content-Type':
            'application/json;charset=UTF-8',
        'Cookie':
            'csrftoken=IoItQ0o4Iuuej8EaMwihhCRVzLcCaNz7hxLOeGwOU3MsVI7GVQgrMgnjSRJ1F7gK; sessionid=k3hbungs0mpua2ensuey7kzfncwjg75o',
        'Accept':
            'application/json, text/javascript, */*; q=0.01',
        'Accept-Encoding':
            'gzip, deflate',
        'Accept-Language':
            'zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7',
        #'User-Agent':
        #    UserAgent(verify_ssl=False)['random'],
        #'Referer':
        #    'http://myspider.com/gaokao/schoollist.html'
    }
    data = json.dumps({
        "updatedt": "2021-04-05",
        "schoolkey": "",
        "length": 20,
        "start": 0
    })
    response = requests.post('http://myspider.com/api/gaokao/scoollist/get.do',
                             headers=headers,
                             data=data,
                             allow_redirects=False)
    print(response.status_code)


if __name__ == "__main__":
    spider_run()
```

- 未加任何反采集措施，返回200

#### 禁止某些user-agent

```nginx
#forbidden UA
if ($http_user_agent ~* (Python|Scrapy|Curl|HttpClient))
{
    return 403;
}
```

- 再次请求，返回403

- 加上user_agent，正常返回200

```python
from fake_useragent import UserAgent
headers = {'User-Agent': UserAgent(verify_ssl=False)['random']}
```

#### 禁止某些method

```nginx
#forbidden not GET|HEAD|POST method access
if ($request_method !~ ^(GET|HEAD|POST)$)
{
    return 403;
}
```

#### 禁止访问频率

- limit_req_zone 限制单位时间内的请求数，速率限制,采用的漏桶算法
- limit_req_conn 限制同一时间连接数，并发限制，根据源IP限制单用户并发访问的连接数

```nginx
http {
    limit_conn_log_level error;
    limit_conn_status 503;
    limit_conn_zone $binary_remote_addr zone=one:10m;
    limit_conn_zone $server_name zone=perserver:10m;
    limit_req_zone $binary_remote_addr zone=allips:10m   rate=20r/s;

    server {
        limit_conn  one  50;                                              
        limit_conn perserver 1000;
        limit_req   zone=allips  burst=5  nodelay;
    }
}
```

#### 限制下载速度

```nginx
location /file { 
       limit_rate_after 10m; 
       limit_rate 128k; 
} 
```

#### 限制referers

- valide_referers指令
  1. none允许缺失的头部访问(默认)
  2. blocked允许referer没有对应值
  3. server_names:表示一个或多个主机名称，myspider.com表示通过myspider.com访问referers，*表示任意host主机

```nginx
valid_referers server_names myspider.com;
if ($invalid_referer) {
    return 403;
}

if ($http_referer ~* "要拦截的域名"){
    return 301 要跳转的域名;
}
```

- 再次请求，返回403

- 加上referers，正常返回200

```python
headers = {'Referer': 'http://myspider.com/gaokao/schoollist.html'}
```

