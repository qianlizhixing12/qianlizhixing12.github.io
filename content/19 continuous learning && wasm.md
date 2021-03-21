---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十九周
Date: 2021-03-17
Tags: 持续学习, webassembly
---

## Algorithm

leetcode 138 复制带随机指针的链表

- [原题](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/138.cpp)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_18/

### 持续学习

- 随着编程越来越普遍，全球化越来越深入，开发者需要持续不断学习以保持竞争力
- 有些it公司会慷慨提供培训，以扩宽员工技能；有些公司不会抽出时间或金钱进行培训；开发者要对自己的生涯负责
- 持续学习的途径
  - 阅读博客，书籍
  - 要深入一种技术，要手动写些代码
  - 要和顶尖导师工作，如果找不到，考虑换个工作
  - 阅读顶尖博主的博客和书籍
  - 了解使用的框架和库的原理源码，试着调试源码
  - 遇到问题，理清问题本质，查看Google上其他人的回答
  - 学习的更好方法，复述或教授别人；当别人向你提问，你更有动力；尝试加入讨论
  - 加入或创建你感兴趣的技术学习小组
  - 参加大会，或者查看大会演讲
  - 在代码上运行静态分析工具或者查看ide警告，理解它们
  - 遵循实用程序员的建议，每年学习一种新技术或工具，尝试当前的技术栈中使用
  - 学习技术外的东西，工作领域，商业逻辑，效率管理

## Tip

- js是脚本语言，即使混淆过的代码，在控制台一点点跟踪，还是不能隐藏细节
- 使用webassembly可以隐藏细节，只暴漏接口方法，可用于前端加密

## Share(webassembly)

> https://webassembly.org/
>
> http://webassembly.org.cn/
>
> http://webassembly.org.cn/docs/security/
>
> https://cntofu.com/book/150/index.html
>
> https://www.cnblogs.com/detectiveHLH/p/9928915.html

### 诞生

- js代码执行过程：JavaScript引擎将代码分析为抽象语法数，中间生成字节码，最后将字节码转化为机器码执行
- 编译优化：参数类型确定情况下，执行频繁的函数，可以将对应机器码记住，直接执行
- js代码不能优化：js代码运行时可以改变变量数据类型，不能使用上述编译优化；因为机器码对应底层，对不同类型的数据即使相同操作，也可能对应不同cpu指令
- 于是对于js中频繁执行的函数，将其二进制字节码(.wasm)封装起来，暴露接口给js调用，增强执行效率

### 编译工具(Emscripten)安装

```bash
git clone https://github.com/juj/emsdk.git  #克隆库
emsdk.bat update             #更新emsdk
emsdk.bat install latest     #安装emsdk，外网访问不佳，可以直接下载zip放到zip目录
emsdk.bat activate latest    #激活emsdk
emsdk_env.bat                #添加环境变量
emcc –v                      #验证
```

### 测试demo

```c
// hello.c
#include <stdio.h>

int main() {
  printf("hello world!\n");
  return 0;
}
```

```bash
# 编译 -o 指定输出文件名
# hello.wasm为C源文件编译后形成的WebAssembly汇编文件
# hello.js是Emscripten生成的胶水代码，其中包含了Emscripten的运行环境和wasm的封装，导入hello.js即可自动完成.wasm载入/编译/实例化、运行时初始化等繁杂的工作
emcc hello.c -o hello.js

# 编译并发布测试网页，http://localhost:8080/hello.html
emcc hello.c -s WASM=1 -o hello.html
emrun --no_browser --port 8080 .
```

### 加载/运行.wasm

- 获取 `.wasm` 二进制文件，将它转换成类型数组或者 `ArrayBuffer`

  - 网络xhr请求文件
  - fetch文件
  - IndexedDB
  - JavaScript 合成

- 将二进制数据编译成一个 `WebAssembly.Module`

  ```javascript
  rst = WebAssembly.compile(bytes).then(m => {
      return new WebAssembly.Instance(m); #WebAssembly.Instance实例对象是无状态的
  });
  # 等价于
  rst = WebAssembly.instantiate(bytes)
  ```

- 使用 imports 实例化这个 `WebAssembly.Module`，获取 exports

  ```javascript
  rst.then(instance => {
  	funs = instance.exports; #其他地方调用funs相当于调用dll导出方法
  });
  ```

### 安全模型

- 每个WebAssembly模块都在使用与主机运行上下文隔离的沙箱环境中执行
- 每个WebAssembly模块都受制于其嵌入的环境的安全策略，例如web浏览器的同源策略
- 终止模块执行的操作
  - 无效索引
  - 函数签名与调用不匹配
  - 非法算术运算(除零等)
  - 超过最大调用堆栈最大值