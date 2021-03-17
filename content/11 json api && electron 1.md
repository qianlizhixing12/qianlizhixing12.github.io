---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十一周
Date: 2020-07-06
Tags: json api, electron
---

## Algorithm

leetcode 433 最小基因变化

- [原题](https://leetcode-cn.com/problems/minimum-genetic-mutation/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/433.py)

## Review

> https://jsonapi.org/format/
>
> http://jsonapi.org.cn/format/
>
> https://github.com/kalasjocke/hyp
>
> https://www.kutu66.com/GitHub/article_108976

json api风格

### 介绍

- 数据交互规范，定义客户端如何获取修改资源，以及服务端如何响应
- 定义业界规范，有利于框架实现
- 常用的api风格，RPC，REST，GraphQL

### 约定

- 双方必须包含Content-Type: application/vnd.api+json请求头
- 不能包含媒体类型

#### top level

- json api 顶级对象
- 必须至少包含data(primary data),errors(错误列表),meta(非标准元信息)一个
- data，errors不能同时存在
- 可以包含jsonapi,links,include(primary data相关链接)
- 可以包含self,retated
- primary data代表请求单一资源(资源对象或null)或资源集合(资源对象列表或[])

#### 资源对象

- 必须包含id,type
- 可以包含attribute,relationshiops,links,meta

### demo

```python
#安装
#pip install hyp

from marshmallow import Serializer, fields
from hyp.responder import Responder


class CommentSerializer(Serializer):
  id = fields.Integer()
  content = fields.String()


class PersonSerializer(Serializer):
  id = fields.Integer()
  name = fields.String()


class PostSerializer(Serializer):
  id = fields.Integer()
  title = fields.String()


class CommentResponder(Responder):
  TYPE = 'comment'
  SERIALIZER = CommentSerializer


class PersonResponder(Responder):
  TYPE = 'person'
  SERIALIZER = PersonSerializer


class PostResponder(Responder):
  TYPE = 'post'
  SERIALIZER = PostSerializer
  LINKS = {
      'comments': {
          'responder': CommentResponder(),
          'href': 'http://example.com/comments/{posts.comments}',
      },
      'author': {
          'responder': PersonResponder(),
          'href': 'http://example.com/people/{posts.author}',
      },
  }


post = {
    'id':
        1,
    'title':
        'My post',
    'comments': [
        {
            'id': 1,
            'content': 'A comment'
        },
        {
            'id': 2,
            'content': 'Another comment'
        },
    ]
}

json = PostResponder.respond(post, linked={'comments': post['comments']})

print(json)

'''
{"posts": {"id": 1, "title": "My post", "links": {"comments": [1, 2]}}, "linked": {"comments": [{"content": "A comment", "id": 1}, {"content": "Another comment", "id": 2}]}, "links": {"posts.comments": {"href": "http://example.com/comments/{posts.comments}", "type": "comments"}}}
'''
```

## Tip

python模板字符串，使用f'...{var}...'格式化字符串，易读易用

## Share

### Electron(一)

#### 安装

```shell
#依赖
node -v
npm -v

#安装
npm install electron
```

#### 第一个程序(主流程)

```javascript
const { app, BrowserWindow } = require('electron')

function createWindow() {
  //打开窗体
  let win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  //加载起始页
  win.loadFile('index.html')

  //开启调试器
  win.webContents.openDevTools()
}

//Electron结束初始化后调用
app.whenReady().then(createWindow)

//window-all-closed窗体关闭回调事件
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow()
  }
})
```

