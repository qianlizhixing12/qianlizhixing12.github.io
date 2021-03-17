---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十五周
Date: 2020-08-03
Tags: 代码布局, electron
---

## Algorithm

leetcode 190 颠倒二进制位

- [原题](https://leetcode-cn.com/problems/reverse-bits/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/190.c)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_13/

#### 代码布局

- 编程时花在浏览和阅读代码上的时间要比实际输入多得多
- 如果行为相同的代码看起来也一样，那么感知系统将帮助找出其中的区别；要遵守关于如何在编译单元中布置类的各个部分的约定:常量、字段、公共方法和私有方法
- 让团队就基础的自动格式化程序达成一致
- 除非有积极的分歧，一个团队将很快汇聚到一个共同的“手工完成”的风格
- 格式化程序无法理解意图，重要的是换行和分组反映了代码的意图，而不仅仅是语言的语法
- 紧凑的格式，在屏幕上看到的内容越多，在不破坏上下文滚动或切换文件的情况下，能看到的内容就越多；布局能帮助理解代码

## Tip

## Share

### Electron(五)

#### 批量管理进程间通信

```javascript
// main.js
const { ipcMain } = require('electron');
const mssql = require('mssql');
const adodb = require('node-adodb');
const req = require('request')

function mssqlall(param) {
  return mssql.connect(param.config).then((conn) => {
    return conn.request().query(param.sql);
  }).then((data) => {
    return Promise.resolve(data);
  }, (err) => {
    return Promise.reject({
      code: '001',
      msg: err
    });
  })
}

// node-adodb模块
function accessall(param) {
  const config = `Provider=Microsoft.Jet.OLEDB.4.0;Data Source=${param.config.database};Jet OLEDB:System Database=${param.config.systemdb};User ID=${param.config.user};Password=${param.config.password}`;
  return adodb.open(config).query(param.sql).then((data) => {
    return Promise.resolve(data);
  }, (err) => {
    return Promise.reject({
      code: '001',
      msg: err
    });
  });
}

// node-request模块
function httpall(param) {
  return new Promise((resolve, reject) => {
    req(param.config.uri, (err, response, body) => {
      if (err) {
        reject({
          code: '001',
          msg: err
        })
      } else {
        resolve(response);
      }
    });
  });
}

let urimap = {
  'mssql.all': mssqlall,
  'access.all': accessall,
  'http.all': httpall
}

ipcMain.on('cefQuery', (event, request) => {
  request = JSON.parse(request);
  let queryid = request.queryid;
  let uri = request.uri;
  let exec = urimap[uri];
  if (exec) {
    let param = request.param;
    exec(param).then((data) => {
      let response = {
        queryid: queryid,
        code: '000',
        data: data
      }
      event.sender.send('cefQuery.callback', JSON.stringify(response));
    }, (err) => {
      let response = {
        queryid: queryid,
        code: err.code,
        msg: err.msg
      }
      event.sender.send('cefQuery.callback', JSON.stringify(response));
    });
  }
});

//render-cefQuery.js
const uuid = require('node-uuid');
const { ipcRenderer } = require('electron');

let workmap = {}

ipcRenderer.on('cefQuery.callback', (event, reponse) => {
  debugger;
  reponse = JSON.parse(reponse);
  let queryid = reponse.queryid;
  let request = workmap[queryid];
  if (request) {
    if (reponse.code === '000') {
      request.onSuccess(reponse.data);
    } else {
      request.onFailure(reponse.code, reponse.msg);
    }
  }
});

function cefQuery(request) {
  let queryid = uuid.v1();
  request['queryid'] = queryid;
  workmap[queryid] = request;
  let data = {
    queryid: request.queryid,
    uri: request.uri,
    param: request.param
  };
  //send 非str值没有传过去
  ipcRenderer.send('cefQuery', JSON.stringify(data));
}


module.exports = cefQuery;

// render.js
    const {
      remote
    } = require('electron');
    const cefQuery = require('./render-cefQuery');

    console.log(remote.app.getVersion());

    new Promise((resolve, reject) => {
      cefQuery({
        uri: 'mssql.all',
        param: {
          config: {
            user: "*",
            password: "*",
            server: "*",
            database: "*",
            options: {
              encrypt: false //使用windows azure，需要设置配置。
            }
          },
          sql: "*"
        },
        onSuccess: (data) => {
          resolve(data);
        },
        onFailure: (code, msg) => {
          reject({
            code: code,
            msg: msg,
          });
        }
      });
    }).then((data) => {
      console.log('mssql.all succ', data)
    }, (err) => {
      console.log('mssql.all fail', err.code, err.msg)
    });

    new Promise((resolve, reject) => {
      cefQuery({
        uri: 'access.all',
        param: {
          config: {
            user: "*"
            password: "*"
            database: "*"
            systemdb: "*"
          },
          sql: "*"
        },
        onSuccess: (data) => {
          resolve(data);
        },
        onFailure: (code, msg) => {
          reject({
            code: code,
            msg: msg,
          });
        }
      });
    }).then((data) => {
      console.log('access.all succ', data)
    }, (err) => {
      console.log('access.all fail', err.code, err.msg)
    });

    new Promise((resolve, reject) => {
      cefQuery({
        uri: 'http.all',
        param: {
          config: {
            uri: "*",
            method: "GET",
            headers: {
              "Accept-Language": "*",
              "Accept-Encoding": "*",
              "Accept": "*",
            }
          }
        },
        onSuccess: (data) => {
          resolve(data);
        },
        onFailure: (code, msg) => {
          reject({
            code: code,
            msg: msg,
          });
        }
      });
    }).then((data) => {
      console.log('http.all succ', data)
    }, (err) => {
      console.log('http.all fail', err.code, err.msg)
    });
```