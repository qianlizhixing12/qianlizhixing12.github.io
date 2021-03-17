---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十二周
Date: 2020-07-13
Tags: git规范, electron
---

## Algorithm

leetcode 98 验证二叉搜索树

- [原题](https://leetcode-cn.com/problems/validate-binary-search-tree/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/98.py)

## Review

> https://github.com/agis/git-style-guide
>
> https://github.com/aseaday/git-style-guide

git规范

#### 分支

- 选择简短和具有描述性的名字来命名分支
- 来自外部的标识符也适合用作分支的名字，例如来自 Github 的 Issue 序号
- 用破折号分割单词
- 当不同的人围绕同一个特性开发时，维护整个团队的特性分支与每个人的独立分支是比较方便的做法，branch/dever
- 合并之后，除非有特殊原因，从上游仓库中删除你的分支

#### 提交

- 每个提交应当只包含一个简单的逻辑改动，不要在一个提交里包含多个逻辑改动。比如，如果一个补丁修复了一个 Bug，又优化了一个特性的性能，就将其拆分
- 不要将一个逻辑改动拆分提交。例如一个功能的实现及其对应的测试应当一并提交
- 尽早、尽快提交。出问题时，短小、完整的提交更容易发现并修正
- 提交应当依逻辑排序。例如，如果 X 提交依赖于 Y，那么 Y 提交应该在 X 前面

#### 消息

- 使用编辑器编写提交信息，而非命令行
- 概要行（即第一行）应当简明扼要。它最好不超过 50 个字符，首字母大写，使用现在时祈使语气。不要以句号结尾, 因为它相当于标题
- 在那之后空一行，然后填写详细描述。每行不超过 72 字符，解释为什么需要改动, 如何解决了这个 issue 以及它有什么副作用
- 如果 提交 A 依赖于另一个 提交 B ，在前者的 commit message 中应当指明。援引对应提交的 Hash
- 如果将一个提交 squash 到另一个提交，分别使用 --squash 和 --fixup 来强调目的

#### 合并

- 不要篡改提交历史，对任何参与项目的人来说，修改历史是万恶之源
- 保持你的提交历史干净、简单
- 如果你的分支包含多个 commmit , 不要使用快进模式

## Tip

#### 尝试不擅长的事

- 第一次通过zoom分享单元测试，分享的视频会看了三四次，总结如下
- 总体上讲明白了自己要表达的内容，得益于早早准备了文档
- 语言过于生涩，虽然准备了文档，没有练习直播分享彩排

## Share

### Electron(二)

#### 主进程渲染进程通信

```javascript
//main
ipcMain.on('asynchronous-message', function (event, arg) {
  console.log(arg);
  event.sender.send('asynchronous-reply', 'asyn pong');
});

ipcMain.on('synchronous-message', function (event, arg) {
  console.log(arg);
  event.returnValue = 'sync pong' + dialog.showOpenDialog(event.sender);
});

//render
console.log(ipcRenderer.sendSync('synchronous-message', 'sync ping'));

ipcRenderer.on('asynchronous-reply', function (event, arg) {
  console.log(arg);
});

ipcRenderer.send('asynchronous-message', 'asyn ping');
```

