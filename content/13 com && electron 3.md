---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十三周
Date: 2020-07-20
Tags: COM, electron
---

## Algorithm

leetcode 77 组合

- [原题](https://leetcode-cn.com/problems/combinations/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/77.py)

## Review

> https://docs.microsoft.com/zh-cn/windows/win32/com/component-object-model--com--portal

### Component Object Model

- 用于组件之间函数调用的二进制标准
- 微软OLE(复合文档)和ActiveX(支持internet的组件)技术的基础技术
- COM指定了一个对象模型和编程要求，使COM对象(也称为COM组件)能够与其他对象交互
  - 提供符合COM规范的运行时环境的主机系统
  - 定义特性契约的接口，以及实现接口的组件
  - 向系统提供组件的服务器和使用组件提供的特性的客户机
  - 跟踪组件在本地和远程主机上部署的位置的注册表
  - 一种服务控制管理器，用于定位本地和远程主机上的组件，并将服务器连接到客户机
  - 一种结构化的存储协议，定义了如何在主机文件系统上导航文件的内容
- COM定义了一些基本接口，这些接口提供所有基于COM的技术通用的功能
  - 将函数的强类型分组为接口的规定
  - 提供多态性、特性发现和对象生命周期跟踪的基本接口
  - 一种唯一标识组件及其接口的机制
    - 每个接口都有自己唯一的接口标识符，是一个全局唯一标识符(GUID)
    - 在创建新接口时，必须为该接口创建新的标识符
    - 当调用者使用接口时，它必须使用唯一标识符
  - 从部署中创建组件实例的组件加载器

- 所有COM接口都继承IUnknown接口
  - 所有的COM对象都需要实现IUnknown接口
  - IUnknown接口包含用于多态性和实例生命周期管理的基本COM操作
  - IUnknown接口有三个成员函数，分别是QueryInterface、AddRef和Release
  - 调用QueryInterface在运行时确定COM对象是否支持特定接口；COM对象如果实现了请求的接口，则返回ppvObject ' ' out参数中的接口指针，否则返回NULL
  - COM对象实例的生存期由其引用计数控制；IUnknown成员函数AddRef和释放控制计数；AddRef增加计数，Release减少计数。当引用计数达到零时，Release成员函数可以释放实例，因为没有调用者在使用它
- 具体实现细节
  - COM允许在不同的应用程序中使用类，在Windows平台上，类存在于动态链接库(DLL)或另一个应用程序(EXE)中
  - 系统上安装的COM对象维护一个所有clsid的注册数据库，注册数据库是每个CLSID和存放相应类的DLL或EXE位置之间的映射。每当调用者想要创建COM类的实例时，COM查询该数据库。调用者只需要知道CLSID就可以请求类的新实例
  - 创建实例最简单的方法是调用COM函数CoCreateInstance
  - 要基于单个CLSID创建多个对象，调用CoGetClassObject函数
  - 要连接到已经创建并运行的对象，调用GetActiveObject函数

## Tip

### node.js中解析返回的html

- 正则表达式
- xpath
- cheerio类似jQuery操作

## Share

### Electron(三)

#### 访问mssql

```javascript
// main
// mssql模块 依赖msnodesql或着tedious
ipcMain.on('mssql.all', (event, request) => {
  console.log(request.config, request.sql);
  mssql.connect(request.config).then((conn) => {
    return conn.request().query(request.sql);
  }).then(result => {
    event.sender.send('mssql.all', result);
  }).catch(function (err) {
    console.log(err);
  });
});

// render
	ipcRenderer.on('mssql.all', (event, reponse) => {
      console.log(reponse);
    });

    const request = {
      config: {
        user: 'sa',
        password: '******',
        server: '******',
        database: '******',
        options: {
          encrypt: false //使用windows azure，需要设置配置。
        }
      },
      sql: 'select * from ******'
    }
    ipcRenderer.send('mssql.all', request);
```

#### 访问access

```javascript
// main
// node-adodb模块
ipcMain.on('access.all', (event, request) => {
  console.log(request.config, request.sql);
  const config = `Provider=Microsoft.Jet.OLEDB.4.0;Data Source=${request.config.database};Jet OLEDB:System Database=${request.config.systemdb};User ID=${request.config.user};Password=${request.config.password}`;
  try {
    let conn = adodb.open(config);
    conn.query(request.sql).then(result => {
      event.sender.send('access.all', result);
    }).catch(err => {
      console.log(err);
    });
  } catch (err) {
    event.sender.send('access.all', err);
  }
});

// render
ipcRenderer.on('access.all', (event, reponse) => {
      console.log(reponse);
    });

    const requestAccess = {
      config: {
        user: '******',
        password: '******',
        database: '******',
        systemdb: '******'
      },
      sql: 'select * from ******'
    }
    ipcRenderer.send('access.all', requestAccess);
```

