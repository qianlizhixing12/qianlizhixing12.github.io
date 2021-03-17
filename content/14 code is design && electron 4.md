---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十四周
Date: 2020-07-27
Tags: electron
---

## Algorithm

leetcode 191 位1的个数

- [原题](https://leetcode-cn.com/problems/number-of-1-bits/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/191.c)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_12/

#### 代码设计

- 随着模型的过时，未完成的设计通过重复地构建和改进接近最终目标而发展。
- 一个偶然的观察者可能很难区分一个未完成的设计和一个完成的产品。
- 建造成本比设计成本更容易计算，当可预测的任务缩减为零时，不可预测的设计时间开始占主导地位。结果产生得更快，但可靠的时间线却会溜走。
- 随着建筑成本的减少，能够快速完成设计的公司在市场上获得优势。快速完成设计成为工程公司的核心推动力。
- 伟大的设计师致力于掌握他们的工艺而产生的。代码也不例外。


## Tip


## Share

### Electron(四)

#### 跨域发送请求

```javascript
// main
// node-adodb模块
  const req = require('request')
  console.log(request.config);
  req(request.config.uri, (err, response, body) => {
    console.log('http', err, response, body)
    event.sender.send('http.all', body);
  });

// render
ipcRenderer.on('http.all', (event, reponse) => {
      console.log(reponse);
    });

const requestHttp = {
      config: {
        uri: '*******',
        method: "GET",
        headers: {
          "Accept-Language": "zh-CN,zh;q=0.9",
          "Accept-Encoding": "gzip, deflate",
          "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
        }
      },
    }
    ipcRenderer.send('http.all', requestHttp);
```