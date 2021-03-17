---
Category: ARTS
Author: qianlizhixing
Title: ARTS第三周
Date: 2020-05-04
Tags: python, pyinstaller, electron
---

## Algorithm

leetcode 20 有效的括号
- [原题](https://leetcode-cn.com/problems/valid-parentheses/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/20.c)

## Review

> https://pyinstaller.readthedocs.io/en/stable/

pyinstaller打包py脚本为系统可执行程序格式。

windows下支持打包为exe可执行文件。

### 安装

pip install pyinstaller

### 工作流程

1. 对脚本进行解析
2. 在脚本目录生成 xxx.spec 文件
3. 创建一个 build 目录，写入一些日志文件和中间流程文件到 build 目录
4. 创建 dist 目录，生成可执行文件到 dist 目录

### 使用

pyinstaller -F xxx.py

- -F 打包成单独的 .exe 文件，这时生成的 .exe 文件会比较大，而且运行速度较慢
- -i  指定可执行文件的图标
- -w 去掉控制台窗口

## Tip

> https://www.cnblogs.com/bytebee/p/8194677.html
>
> http://fuseyism.com/classpath/doc/java/util/ArrayDeque-source.html

取模运算转化成位运算 ，X % 2^n = X & (2^n – 1)

应用java源码ArrayDeque保持容量为2^n，便于头尾游标越界时取模计算

```java
  82:     /**
  83:      * The minimum capacity that we'll use for a newly created deque.
  84:      * Must be a power of 2.
  85:      */
  86:     private static final int MIN_INITIAL_CAPACITY = 8;
  87: 
  88:     // ******  Array allocation and resizing utilities ******
  89: 
  90:     /**
  91:      * Allocate empty array to hold the given number of elements.
  92:      *
  93:      * @param numElements  the number of elements to hold
  94:      */
  95:     private void allocateElements(int numElements) {
  96:         int initialCapacity = MIN_INITIAL_CAPACITY;
  97:         // Find the best power of two to hold elements.
  98:         // Tests "<=" because arrays aren't kept full.
  99:         if (numElements >= initialCapacity) {
 100:             initialCapacity = numElements;
 101:             initialCapacity |= (initialCapacity >>>  1);
 102:             initialCapacity |= (initialCapacity >>>  2);
 103:             initialCapacity |= (initialCapacity >>>  4);
 104:             initialCapacity |= (initialCapacity >>>  8);
 105:             initialCapacity |= (initialCapacity >>> 16);
 106:             initialCapacity++;
 107: 
 108:             if (initialCapacity < 0)   // Too many elements, must back off
 109:                 initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
 110:         }
 111:         elements = (E[]) new Object[initialCapacity];
 112:     }
 113: 
 114:     /**
 115:      * Double the capacity of this deque.  Call only when full, i.e.,
 116:      * when head and tail have wrapped around to become equal.
 117:      */
 118:     private void doubleCapacity() {
 119:         assert head == tail;
 120:         int p = head;
 121:         int n = elements.length;
 122:         int r = n - p; // number of elements to the right of p
 123:         int newCapacity = n << 1;
 124:         if (newCapacity < 0)
 125:             throw new IllegalStateException("Sorry, deque too big");
 126:         Object[] a = new Object[newCapacity];
 127:         System.arraycopy(elements, p, a, 0, r);
 128:         System.arraycopy(elements, 0, a, r, p);
 129:         elements = (E[])a;
 130:         head = 0;
 131:         tail = n;
 132:     }

 198:         elements[head = (head - 1) & (elements.length - 1)] = e;

 215:         if ( (tail = (tail + 1) & (elements.length - 1)) == head)
```

## Share

> https://www.electronjs.org/docs
>
> https://github.com/electron

Electron是GitHub开发的一个开源框架。它允许使用Node.js（后端）和Chromium（前端）完成桌面GUI应用程序的开发。著名项目包括GitHub的Atom和微软的Visual Studio Code。

### 依赖

js运行环境node.js

js包管理npm

推荐编辑器Atom，Visual Studio Code

### 安装

npm install (-g) electron

### 开始

```javascript
//初始化项目
npm init
//修改package执行命令
electron .

//main.js
const { app, BrowserWindow } = require('electron')

function createWindow() {
  // Create the browser window.
  let win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  // and load the index.html of the app.
  win.loadFile('index.html')

  // Open the DevTools.
  win.webContents.openDevTools()
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // On macOS it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow()
  }
})
```

如果报错Electron failed to install correctly, please delete node_modules/electron and try installing again，改用6.0版本