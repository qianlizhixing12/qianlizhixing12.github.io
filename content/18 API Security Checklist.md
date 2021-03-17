---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十八周
Date: 2021-01-25
Tags: 理性编码, 注释, 安全的API
---

## Algorithm

leetcode 430 扁平化多级双向链表

- [原题](https://leetcode-cn.com/problems/flatten-a-multilevel-doubly-linked-list/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c%2B%2B/430.cpp)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_15/

### 理性编码

- 推理软件正确性得基本方法是将所有代码分成小段(通常组织为精简函数)，并讨论小段代码的正确行
- 小块代码应该便于对其状态的解读，包括函数名，函数变量名，任务描述，解读简单，容易抽象成前提条件和后置条件
- 许多众所周知的编码实践使推理更容易，大多可以被静态代码分析器检查
  - 不要使用goto，会导致代码高度耦合
  - 不要使用可修改的全局变量，会导致代码相互依赖改变量状态(数值)
  - 变量尽量拥有最小的作用域
  - 让对象不可变
  - 通过使用水平和垂直的间距，例如对齐相关的结构，提高代码可读性
  - 通过为对象、类型、函数等选择描述性(但相对较短)的名称，使代码自文档化
  - 如果你需要一个嵌套的小块，让它成为一个函数
  - 函数尽量短而精，即简短并且专注于一个单一的任务，以前有24行的限制，现在也适用
  - 函数参数尽量少，最多4个。将相关参数组合成对象或结构体，这样的一致性有助于解读
  - 从块到库的每个代码单元都应该有一个*狭窄的接口*，更少的交流减少了推理的需要

### 注释建议

- 代码应该向后面的程序员解释清楚
- 代码中应该解释代码应该做什么，难写就难读
- 注释也可能走得太远，确保注释澄清了代码，而不是模糊了它
- 头注释应该能提供足够的信息，帮助其他程序员使用你的代码，而不需要读一遍
- 内联注释应该帮助下一个开发人员修复或扩展它

### 只注释晦涩难懂的代码

- 理论与实践之间的差异，在实践中比在理论中更大
- 当代码格式不佳时，编译器、解释器和其他工具肯定会反对，错误注释却无法借助工具发现
- 版本相关的注释，注释掉的代码试图解决版本控制和历史的问题，这些问题已经被版本控制工具回答了(更有效地)
- 代码库中普遍存在的嘈杂的注释和不正确的注释会鼓励程序员忽略所有的注释
- 为了避免代码库因程序员的聪明才智而被误用，并降低忽略任何真正有价值的注释的风险，注释应该被当作代码来对待
- 每条注释都应该为读者增加一些价值，否则它就是浪费，应该被删除或重写
- 注释应该说一些代码没有也不能说的东西
- 注释说明一段代码应该已经说了什么，这是在邀请更改代码结构或编码约定，以便代码自己说话
- 不要补偿不好的方法名或类名，而要重命名它们
- 不要在长函数中注释节，而要提取较小的函数，这些函数的名称能捕捉到前一节的意图
- 尽量通过代码来表达

## Tip

## Share

> https://github.com/shieldfy/API-Security-Checklist

### 开发安全的API所需要核对的清单

#### 身份认证
- 不要使用 `Basic Auth` ，请使用标准的认证协议（如 [JWT](https://jwt.io/)，[OAuth](https://oauth.net/)）。
- 不要重新实现 `Authentication`、`token generating` 和 `password storing`，请使用标准库。
- 限制密码错误尝试次数，并且增加账号冻结功能。
- 加密所有的敏感数据。

#### JWT（JSON Web Token）
- 使用随机复杂的密钥（`JWT Secret`）以增加暴力破解的难度。
- 不要在请求体中直接提取数据，要对数据进行加密（`HS256` 或 `RS256`）。
- 使 token 的过期时间尽量的短（`TTL`，`RTTL`）。
- 不要在 JWT 的请求体中存放敏感数据，因为它是[可解码的](https://jwt.io/#debugger-io)。

#### OAuth 授权或认证协议
- 始终在后台验证 `redirect_uri`，只允许白名单的 URL。
- 始终在授权时使用有效期较短的授权码（code）而不是令牌（access_token）（不允许 `response_type=token`）。
- 使用随机哈希数的 `state` 参数来防止跨站请求伪造（CSRF）。
- 对不同的应用分别定义默认的作用域和各自有效的作用域参数。

### 访问
- 限制流量来防止 DDoS 攻击和暴力攻击。
- 在服务端使用 HTTPS 协议来防止 MITM （中间人攻击）。
- 使用 `HSTS` 协议防止 SSL Strip 攻击。

### 输入
- 使用与操作相符的 HTTP 操作函数，`GET（读取)`，`POST（创建）`，`PUT（替换/更新）` 以及 `DELETE（删除记录）`，如果请求的方法不适用于请求的资源则返回 `405 Method Not Allowed`。
- 在请求头中的 `content-type` 字段使用内容验证来只允许支持的格式（如 `application/xml`，`application/json` 等等）并在不满足条件的时候返回 `406 Not Acceptable`。
- 验证 `content-type` 中申明的编码和你收到正文编码一致（如 `application/x-www-form-urlencoded`，`multipart/form-data`，`application/json` 等等）。
- 验证用户输入来避免一些普通的易受攻击缺陷（如 `XSS`，`SQL-注入`，`远程代码执行` 等等）。
- 不要在 URL 中使用任何敏感的数据（`credentials`，`Passwords`，`security tokens`，or `API keys`），而是使用标准的认证请求头。
- 使用一个 API Gateway 服务来启用缓存、限制访问速率（如 `Quota`，`Spike Arrest`，`Concurrent Rate Limit`）以及动态地部署 APIs resources。

### 处理
- 检查是否所有的接口都包含必要都身份认证，以避免被破坏了的认证体系。
- 避免使用特有的资源 id。使用 `/me/orders` 替代 `/user/654321/orders`。
- 使用 `UUID` 代替自增长的 id。
- 如果需要解析 XML 文件，确保实体解析（entity parsing）是关闭的以避免 `XXE` 攻击。
- 如果需要解析 XML 文件，确保实体扩展（entity expansion）是关闭的以避免通过指数实体扩展攻击实现的 `Billion Laughs/XML bomb`。
- 在文件上传中使用 CDN。
- 如果数据处理量很大，尽可能使用队列或者 Workers 在后台处理来避免阻塞请求，从而快速响应客户端。
- 不要忘了把 DEBUG 模式关掉。

### 输出
- 增加请求返回头 `X-Content-Type-Options: nosniff`。
- 增加请求返回头 `X-Frame-Options: deny`。
- 增加请求返回头 `Content-Security-Policy: default-src 'none'`。
- 删除请求返回中的指纹头 - `X-Powered-By`，`Server`，`X-AspNet-Version` 等等。
- 在响应中遵循请求的 `content-type`，如果你的请求类型是 `application/json` 那么你返回的 `content-type` 就是 `application/json`。
- 不要返回敏感的数据，如 `credentials`，`Passwords`，`security tokens`。
- 给请求返回使用合理的 HTTP 响应代码。（如 `200 OK`，`400 Bad Request`，`401 Unauthorized`，`405 Method Not Allowed` 等等）。

### 持续集成和持续部署
- 使用单元测试以及集成测试的覆盖率来保障你的设计和实现。
- 引入代码审查流程，禁止私自合并代码。
- 在推送到生产环境之前确保服务的所有组件都用杀毒软件静态地扫描过，包括第三方库和其它依赖。
- 为部署设计一个回滚方案。

### 参考
- [yosriady/api-development-tools](https://github.com/yosriady/api-development-tools) - 用于构建 RESTful HTTP + JSON API 的有用资源集合。