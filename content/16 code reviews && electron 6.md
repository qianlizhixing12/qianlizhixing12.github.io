---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十六周
Date: 2020-08-10
Tags: 代码评审, electron
---

## Algorithm

leetcode 338 比特位计数

- [原题](https://leetcode-cn.com/problems/counting-bits/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/338.c)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_14/

#### 代码评审

- 提高了代码质量并降低了缺陷率
- 代码可能有很多细节涉及系统具体业务
- 代码审查的目的不应该是简单地纠正代码中的错误，而应该是分享知识和建立共同的编码指南
- 通过学习和理解代码来检查代码，而不是寻找错误
- 代码审查期间要温和，确保你的评论是有建设性的，而不是刻薄的
- 避免团队成员中的组织级别影响代码评审
- 如果团队有通过工具检查的编码约定，那么代码审查将更容易进行。这样，代码格式化就不会在代码评审会议上讨论
- 让代码审查变得有趣也许是成功的最重要因素

## Tip

## Share

### Electron(六)

#### 多个渲染进程间通信

```javascript
//cefmanage.js
const uuid = require('node-uuid');
const { ipcMain, ipcRenderer, remote } = require('electron');

function cefmanage() {

  this.cefmap = {};

  this.srvmap = {};

  this.cefreg = (uri, exec) => {
    this.cefmap[uri] = exec;
    if (ipcMain) {
      this.srvmap[uri] = 0;
    }
    if (ipcRenderer) {
      this.srvmap = ipcRenderer.sendSync('maincef', [uri, remote.getCurrentWebContents().id]);
    }
  }

  let ipc = ipcMain || ipcRenderer;
  ipc.on('cefquery', (event, request) => {
    request = JSON.parse(request);
    let queryid = request.queryid;
    let uri = request.uri;
    let param = request.param;
    this.cefexec(event, this.cefmap, queryid, uri, param);
  });

  this.cefexec = (event, cefmap, queryid, uri, param) => {
    let exec = cefmap[uri];
    if (exec) {
      exec(param).then((data) => {
        let response = {
          queryid: queryid,
          code: '000',
          data: data
        }
        event.sender.send('cefquery.callback', JSON.stringify(response));
      }, (err) => {
        let response = {
          queryid: queryid,
          code: err.code,
          msg: err.msg
        }
        event.sender.send('cefquery.callback', JSON.stringify(response));
      });
    }
  };

  if (ipcMain) {
    ipcMain.on('maincef', (event, request) => {
      if (request.length === 2) {
        this.srvmap[request[0]] = request[1];
      }
      event.returnValue = this.srvmap;
    });
  }

  if (ipcRenderer) {
    this.srvmap = ipcRenderer.sendSync('maincef', []);

    this.workmap = {};
    this.maincef = {};

    this.cefquery = (request) => {
      let queryid = uuid.v1();
      request['queryid'] = queryid;
      this.workmap[queryid] = request;
      let data = {
        queryid: request.queryid,
        uri: request.uri,
        param: request.param
      };
      debugger;
      //send 非str值没有传过去
      let srvid = this.srvmap[request.uri];
      // if (srvid == 'main') {
      //   ipcRenderer.send('cefquery', JSON.stringify(data));
      // } else
      if (typeof (srvid) === 'number') {
        ipcRenderer.sendTo(srvid, 'cefquery', JSON.stringify(data));
      } else {
        delete this.workmap[queryid];
        request.onFailure('001', '该操作未支持');
      }
    };

    ipcRenderer.on('cefquery.callback', (event, reponse) => {
      reponse = JSON.parse(reponse);
      let queryid = reponse.queryid;
      let request = this.workmap[queryid];
      delete this.workmap[queryid];
      if (request) {
        if (reponse.code === '000') {
          request.onSuccess(reponse.data);
        } else {
          request.onFailure(reponse.code, reponse.msg);
        }
      }
    });
  };
}

const cef = new cefmanage();
const cefreg = cef.cefreg;
const cefquery = cef.cefquery;

if (ipcMain) {
  module.exports = { cefreg };
}
if (ipcRenderer) {
  module.exports = { cefreg, cefquery };
}

//mainservice.js
// const mssql = require('mssql');
const { Connection, Request } = require('tedious')
const adodb = require('node-adodb');
const req = require('request')

function mssqlall(param) {
  // return mssql.connect(param.config).then(pool => {
  //   return pool.request().query(param.sql);
  // }).then(data => {
  //   return Promise.resolve(data);
  // }).catch(err => {
  //   return Promise.reject({
  //     code: '001',
  //     msg: err
  //   });
  // });

  return new Promise((resolve, reject) => {
    let conn = new Connection(param.config);

    conn.on('connect', function (err) {
      if (err) {
        reject({
          code: '001',
          msg: err
        });
      }

      request = new Request(param.sql, function (err, rowCount, rows) {
        if (err) {
          reject({
            code: '001',
            msg: err
          });
        } else {
          resolve(rows);
        }
      });

      // Execute SQL statement
      conn.execSql(request);
    });
  });
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

module.exports = { mssqlall, accessall, httpall };

//main.js
const { app, BrowserWindow } = require('electron')
// require('./main-cefQuery');
const { cefreg } = require('./cefmanage');
const { mssqlall, accessall, httpall } = require('./mainservice');

function createService() {
  cefreg('access.all', accessall);
  cefreg('http.all', httpall);
}

function createMainWindow() {
  // Create the browser window.
  let win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
      allowRunningInsecureContent: true
    }
  })

  console.log('indexid', win.webContents.id)

  // and load the index.html of the app.
  // win.loadFile('index.html')
  win.loadFile('index.html')

  // Open the DevTools.
  win.webContents.openDevTools()

  //任务栏图标进度条
  win.setProgressBar(0.5);
}

function createServiceWindow() {
  // Create the browser window.
  let win = new BrowserWindow({
    width: 800,
    height: 600,
    show: false,
    webPreferences: {
      nodeIntegration: true,
      allowRunningInsecureContent: true
    }
  })

  console.log('servid', win.webContents.id)

  // and load the index.html of the app.
  win.loadFile('service.html')

  // Open the DevTools.
  win.webContents.openDevTools()

  //任务栏图标进度条
  win.setProgressBar(0.5);
}

function createWindow() {
  createService();
  createServiceWindow();
  createMainWindow();
}

app.commandLine.appendSwitch('--disable-http-cache');

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(createWindow);

// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
});

//service.html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <title>service</title>
</head>

<body>
  <script>
    const {
      cefreg
    } = require('./cefmanage');

    function servicetest(param) {
      return new Promise((resolve, reject) => {
        resolve(`service.test ok!!! ${param}`);
      });
    }

    cefreg('service.test', servicetest);
  </script>
</body>

</html>

//index.html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <title>Hello World!</title>
  <!-- https://electronjs.org/docs/tutorial/security#csp-meta-tag -->
  <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />
</head>

<body>
  <h1>Hello World!</h1>
  <script>
    const {
      cefquery
    } = require('./cefmanage');


    new Promise((resolve, reject) => {
      cefquery({
        uri: 'mssql.all',
        param: {
          config: {
            userName: 'sa',
            password: '****',
            server: '****',
            options: {
              database: '****'
            }
          },
          sql: 'select * from UA_Account'
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
      cefquery({
        uri: 'service.test',
        param: {
          'e3e': 345
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
      console.log('service.test succ', data)
    }, (err) => {
      console.log('service.test fail', err.code, err.msg)
    });

    new Promise((resolve, reject) => {
      cefquery({
        uri: 'access.all',
        param: {
          config: {
            user: '****',
            password: '****',
            database: '****',
            systemdb: '****'
          },
          sql: 'select * from GLPref'
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
      cefquery({
        uri: 'http.all',
        param: {
          config: {
            uri: '****',
            method: "GET",
            headers: {
              "Accept-Language": "zh-CN,zh;q=0.9",
              "Accept-Encoding": "gzip, deflate",
              "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
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
  </script>
</body>

</html>
```