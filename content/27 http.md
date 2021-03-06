---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十七周
Date: 2021-05-10
Tags: 网络编程, python库, http
---

## Algorithm

leetcode 403 青蛙过河

- [原题](https://leetcode-cn.com/problems/frog-jump/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/403.py)
- 缓存中间结果，动态规划

## Review

> 转载：https://github.com/livoras/blog/issues/13
>
> 作者：戴嘉华

完整代码存放在 [Github](https://github.com/livoras/simple-virtual-dom)

### 对前端应用状态管理的思考

假如现在你需要写一个像下面一样的表格的应用程序，这个表格可以根据不同的字段进行升序或者降序的展示。

[![sort-table](https://camo.githubusercontent.com/1cd1a48c7e02ffee8f619d95fdec31fd05196940a98805770643d00b4c7da8be/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f736f72742d7461626c652e706e67)](https://camo.githubusercontent.com/1cd1a48c7e02ffee8f619d95fdec31fd05196940a98805770643d00b4c7da8be/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f736f72742d7461626c652e706e67)

这个应用程序看起来很简单，你可以想出好几种不同的方式来写。最容易想到的可能是，在你的 JavaScript 代码里面存储这样的数据：

```javascript
var sortKey = "new" // 排序的字段，新增（new）、取消（cancel）、净关注（gain）、累积（cumulate）人数
var sortType = 1 // 升序还是逆序
var data = [{...}, {...}, {..}, ..] // 表格数据
```

用三个字段分别存储当前排序的字段、排序方向、还有表格数据；然后给表格头部加点击事件：当用户点击特定的字段的时候，根据上面几个字段存储的内容来对内容进行排序，然后用 JS 或者 jQuery 操作 DOM，更新页面的排序状态（表头的那几个箭头表示当前排序状态，也需要更新）和表格内容。

这样做会导致的后果就是，随着应用程序越来越复杂，需要在JS里面维护的字段也越来越多，需要监听事件和在事件回调用更新页面的DOM操作也越来越多，应用程序会变得非常难维护。后来人们使用了 MVC、MVP 的架构模式，希望能从代码组织方式来降低维护这种复杂应用程序的难度。但是 MVC 架构没办法减少你所维护的状态，也没有降低状态更新你需要对页面的更新操作（前端来说就是DOM操作），你需要操作的DOM还是需要操作，只是换了个地方。

既然状态改变了要操作相应的DOM元素，为什么不做一个东西可以让视图和状态进行绑定，状态变更了视图自动变更，就不用手动更新页面了。这就是后来人们想出了 MVVM 模式，只要在模版中声明视图组件是和什么状态进行绑定的，双向绑定引擎就会在状态更新的时候自动更新视图（关于MV*模式的内容，可以看[这篇介绍](https://github.com/livoras/blog/issues/11)）。

MVVM 可以很好的降低我们维护状态 -> 视图的复杂程度（大大减少代码中的视图更新逻辑）。但是这不是唯一的办法，还有一个非常直观的方法，可以大大降低视图更新的操作：一旦状态发生了变化，就用模版引擎重新渲染**整个视图**，然后用新的视图更换掉旧的视图。就像上面的表格，当用户点击的时候，还是在JS里面更新状态，但是页面更新就不用手动操作 DOM 了，直接把整个表格用模版引擎重新渲染一遍，然后设置一下`innerHTML`就完事了。

听到这样的做法，经验丰富的你一定第一时间意识这样的做法会导致很多的问题。最大的问题就是这样做会很慢，因为即使一个小小的状态变更都要重新构造整棵 DOM，性价比太低；而且这样做的话，`input`和`textarea`的会失去原有的焦点。最后的结论会是：对于局部的小视图的更新，没有问题（Backbone就是这么干的）；但是对于大型视图，如全局应用状态变更的时候，需要更新页面较多局部视图的时候，这样的做法不可取。

**但是这里要明白和记住这种做法**，因为后面你会发现，**其实 Virtual DOM 就是这么做的，只是加了一些特别的步骤来避免了整棵 DOM 树变更**。

另外一点需要注意的就是，上面提供的几种方法，其实都在解决同一个问题：**维护状态，更新视图**。在一般的应用当中，如果能够很好方案来应对这个问题，那么就几乎降低了大部分复杂性。

### Virtual DOM算法

DOM是很慢的。如果我们把一个简单的`div`元素的属性都打印出来，你会看到：

[![dom-attr](https://camo.githubusercontent.com/031ccafc5859e6eb7c9d8dc528caeb5c239b1e86060e16197a5982341b8df363/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f646f6d2d617474726962757465732e706e67)](https://camo.githubusercontent.com/031ccafc5859e6eb7c9d8dc528caeb5c239b1e86060e16197a5982341b8df363/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f646f6d2d617474726962757465732e706e67)

而这仅仅是第一层。真正的 DOM 元素非常庞大，这是因为标准就是这么设计的。而且操作它们的时候你要小心翼翼，轻微的触碰可能就会导致页面重排，这可是杀死性能的罪魁祸首。

相对于 DOM 对象，原生的 JavaScript 对象处理起来更快，而且更简单。DOM 树上的结构、属性信息我们都可以很容易地用 JavaScript 对象表示出来：

```javascript
var element = {
  tagName: 'ul', // 节点标签名
  props: { // DOM的属性，用一个对象存储键值对
    id: 'list'
  },
  children: [ // 该节点的子节点
    {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
  ]
}
```

上面对应的HTML写法是：

```html
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
```

既然原来 DOM 树的信息都可以用 JavaScript 对象来表示，反过来，你就可以根据这个用 JavaScript 对象表示的树结构来构建一棵真正的DOM树。

之前的章节所说的，状态变更->重新渲染整个视图的方式可以稍微修改一下：用 JavaScript 对象表示 DOM 信息和结构，当状态变更的时候，重新渲染这个 JavaScript 的对象结构。当然这样做其实没什么卵用，因为真正的页面其实没有改变。

但是可以用新渲染的对象树去和旧的树进行对比，记录这两棵树差异。记录下来的不同就是我们需要对页面真正的 DOM 操作，然后把它们应用在真正的 DOM 树上，页面就变更了。这样就可以做到：视图的结构确实是整个全新渲染了，但是最后操作DOM的时候确实只变更有不同的地方。

这就是所谓的 Virtual DOM 算法。包括几个步骤：

1. 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
2. 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
3. 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了

Virtual DOM 本质上就是在 JS 和 DOM 之间做了一个缓存。可以类比 CPU 和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）。

### 算法实现

#### 步骤一：用JS对象模拟DOM树

用 JavaScript 来表示一个 DOM 节点是很简单的事情，你只需要记录它的节点类型、属性，还有子节点：

element.js

```javascript
function Element (tagName, props, children) {
  this.tagName = tagName
  this.props = props
  this.children = children
}

module.exports = function (tagName, props, children) {
  return new Element(tagName, props, children)
}
```

例如上面的 DOM 结构就可以简单的表示：

```javascript
var el = require('./element')

var ul = el('ul', {id: 'list'}, [
  el('li', {class: 'item'}, ['Item 1']),
  el('li', {class: 'item'}, ['Item 2']),
  el('li', {class: 'item'}, ['Item 3'])
])
```

现在`ul`只是一个 JavaScript 对象表示的 DOM 结构，页面上并没有这个结构。我们可以根据这个`ul`构建真正的`<ul>`：

```javascript
Element.prototype.render = function () {
  var el = document.createElement(this.tagName) // 根据tagName构建
  var props = this.props

  for (var propName in props) { // 设置节点的DOM属性
    var propValue = props[propName]
    el.setAttribute(propName, propValue)
  }

  var children = this.children || []

  children.forEach(function (child) {
    var childEl = (child instanceof Element)
      ? child.render() // 如果子节点也是虚拟DOM，递归构建DOM节点
      : document.createTextNode(child) // 如果字符串，只构建文本节点
    el.appendChild(childEl)
  })

  return el
}
```

`render`方法会根据`tagName`构建一个真正的DOM节点，然后设置这个节点的属性，最后递归地把自己的子节点也构建起来。所以只需要：

```javascript
var ulRoot = ul.render()
document.body.appendChild(ulRoot)
```

上面的`ulRoot`是真正的DOM节点，把它塞入文档中，这样`body`里面就有了真正的`<ul>`的DOM结构：

```html
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
```

完整代码可见 [element.js](https://github.com/livoras/simple-virtual-dom/blob/master/lib/element.js)。

#### 步骤二：比较两棵虚拟DOM树的差异

正如你所预料的，比较两棵DOM树的差异是 Virtual DOM 算法最核心的部分，这也是所谓的 Virtual DOM 的 diff 算法。两个树的完全的 diff 算法是一个时间复杂度为 O(n^3) 的问题。但是在前端当中，你很少会跨越层级地移动DOM元素。所以 Virtual DOM 只会对同一个层级的元素进行对比：

[![compare-in-level](https://camo.githubusercontent.com/8589323ee9f10643f1c1e0b98b7676ca3a1e4d6c5ef29e99ff028d1dcbf9c5da/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f636f6d706172652d696e2d6c6576656c2e706e67)](https://camo.githubusercontent.com/8589323ee9f10643f1c1e0b98b7676ca3a1e4d6c5ef29e99ff028d1dcbf9c5da/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f636f6d706172652d696e2d6c6576656c2e706e67)

上面的`div`只会和同一层级的`div`对比，第二层级的只会跟第二层级对比。这样算法复杂度就可以达到 O(n)。

##### 深度优先遍历，记录差异

在实际的代码中，会对新旧两棵树进行一个深度优先的遍历，这样每个节点都会有一个唯一的标记：

[![dfs-walk](https://camo.githubusercontent.com/b698f28c8fc76b4ee9a1c92ec9c29ab56fd246f60a1f207099e6b7e89c62a52c/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f6466732d77616c6b2e706e67)](https://camo.githubusercontent.com/b698f28c8fc76b4ee9a1c92ec9c29ab56fd246f60a1f207099e6b7e89c62a52c/687474703a2f2f6c69766f7261732e6769746875622e696f2f626c6f672f7669727475616c2d646f6d2f6466732d77616c6b2e706e67)

在深度优先遍历的时候，每遍历到一个节点就把该节点和新的的树进行对比。如果有差异的话就记录到一个对象里面。

```javascript
// diff 函数，对比两棵树
function diff (oldTree, newTree) {
  var index = 0 // 当前节点的标志
  var patches = {} // 用来记录每个节点差异的对象
  dfsWalk(oldTree, newTree, index, patches)
  return patches
}

// 对两棵树进行深度优先遍历
function dfsWalk (oldNode, newNode, index, patches) {
  // 对比oldNode和newNode的不同，记录下来
  patches[index] = [...]

  diffChildren(oldNode.children, newNode.children, index, patches)
}

// 遍历子节点
function diffChildren (oldChildren, newChildren, index, patches) {
  var leftNode = null
  var currentNodeIndex = index
  oldChildren.forEach(function (child, i) {
    var newChild = newChildren[i]
    currentNodeIndex = (leftNode && leftNode.count) // 计算节点的标识
      ? currentNodeIndex + leftNode.count + 1
      : currentNodeIndex + 1
    dfsWalk(child, newChild, currentNodeIndex, patches) // 深度遍历子节点
    leftNode = child
  })
}
```

例如，上面的`div`和新的`div`有差异，当前的标记是0，那么：

```javascript
patches[0] = [{difference}, {difference}, ...] // 用数组存储新旧节点的不同
```

同理`p`是`patches[1]`，`ul`是`patches[3]`，类推。

##### 差异类型

上面说的节点的差异指的是什么呢？对 DOM 操作可能会：

1. 替换掉原来的节点，例如把上面的`div`换成了`section`
2. 移动、删除、新增子节点，例如上面`div`的子节点，把`p`和`ul`顺序互换
3. 修改了节点的属性
4. 对于文本节点，文本内容可能会改变。例如修改上面的文本节点2内容为`Virtual DOM 2`。

所以我们定义了几种差异类型：

```javascript
var REPLACE = 0
var REORDER = 1
var PROPS = 2
var TEXT = 3
```

对于节点替换，很简单。判断新旧节点的`tagName`和是不是一样的，如果不一样的说明需要替换掉。如`div`换成`section`，就记录下：

```javascript
patches[0] = [{
  type: REPALCE,
  node: newNode // el('section', props, children)
}]
```

如果给`div`新增了属性`id`为`container`，就记录下：

```javascript
patches[0] = [{
  type: REPALCE,
  node: newNode // el('section', props, children)
}, {
  type: PROPS,
  props: {
    id: "container"
  }
}]
```

如果是文本节点，如上面的文本节点2，就记录下：

```javascript
patches[2] = [{
  type: TEXT,
  content: "Virtual DOM2"
}]
```

那如果把我`div`的子节点重新排序呢？例如`p, ul, div`的顺序换成了`div, p, ul`。这个该怎么对比？如果按照同层级进行顺序对比的话，它们都会被替换掉。如`p`和`div`的`tagName`不同，`p`会被`div`所替代。最终，三个节点都会被替换，这样DOM开销就非常大。而实际上是不需要替换节点，而只需要经过节点移动就可以达到，我们只需知道怎么进行移动。

这牵涉到两个列表的对比算法，需要另外起一个小节来讨论。

##### 列表对比算法

假设现在可以英文字母唯一地标识每一个子节点：

旧的节点顺序：

```
a b c d e f g h i
```

现在对节点进行了删除、插入、移动的操作。新增`j`节点，删除`e`节点，移动`h`节点：

新的节点顺序：

```
a b c h d f g i j
```

现在知道了新旧的顺序，求最小的插入、删除操作（移动可以看成是删除和插入操作的结合）。这个问题抽象出来其实是字符串的最小编辑距离问题（[Edition Distance](https://en.wikipedia.org/wiki/Edit_distance)），最常见的解决算法是 [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance)，通过动态规划求解，时间复杂度为 O(M * N)。但是我们并不需要真的达到最小的操作，我们只需要优化一些比较常见的移动情况，牺牲一定DOM操作，让算法时间复杂度达到线性的（O(max(M, N))。具体算法细节比较多，这里不累述，有兴趣可以参考[代码](https://github.com/livoras/list-diff/blob/master/lib/diff.js)。

我们能够获取到某个父节点的子节点的操作，就可以记录下来：

```
patches[0] = [{
  type: REORDER,
  moves: [{remove or insert}, {remove or insert}, ...]
}]
```

但是要注意的是，因为`tagName`是可重复的，不能用这个来进行对比。所以需要给子节点加上唯一标识`key`，列表对比的时候，使用`key`进行对比，这样才能复用老的 DOM 树上的节点。

这样，我们就可以通过深度优先遍历两棵树，每层的节点进行对比，记录下每个节点的差异了。完整 diff 算法代码可见 [diff.js](https://github.com/livoras/simple-virtual-dom/blob/master/lib/diff.js)。

#### 步骤三：把差异应用到真正的DOM树上

因为步骤一所构建的 JavaScript 对象树和`render`出来真正的DOM树的信息、结构是一样的。所以我们可以对那棵DOM树也进行深度优先的遍历，遍历的时候从步骤二生成的`patches`对象中找出当前遍历的节点差异，然后进行 DOM 操作。

```javascript
function patch (node, patches) {
  var walker = {index: 0}
  dfsWalk(node, walker, patches)
}

function dfsWalk (node, walker, patches) {
  var currentPatches = patches[walker.index] // 从patches拿出当前节点的差异

  var len = node.childNodes
    ? node.childNodes.length
    : 0
  for (var i = 0; i < len; i++) { // 深度遍历子节点
    var child = node.childNodes[i]
    walker.index++
    dfsWalk(child, walker, patches)
  }

  if (currentPatches) {
    applyPatches(node, currentPatches) // 对当前节点进行DOM操作
  }
}
```

applyPatches，根据不同类型的差异对当前节点进行 DOM 操作：

```javascript
function applyPatches (node, currentPatches) {
  currentPatches.forEach(function (currentPatch) {
    switch (currentPatch.type) {
      case REPLACE:
        node.parentNode.replaceChild(currentPatch.node.render(), node)
        break
      case REORDER:
        reorderChildren(node, currentPatch.moves)
        break
      case PROPS:
        setProps(node, currentPatch.props)
        break
      case TEXT:
        node.textContent = currentPatch.content
        break
      default:
        throw new Error('Unknown patch type ' + currentPatch.type)
    }
  })
}
```

完整代码可见 [patch.js](https://github.com/livoras/simple-virtual-dom/blob/master/lib/patch.js)。

### 结语

Virtual DOM 算法主要是实现上面步骤的三个函数：[element](https://github.com/livoras/simple-virtual-dom/blob/master/lib/element.js)，[diff](https://github.com/livoras/simple-virtual-dom/blob/master/lib/diff.js)，[patch](https://github.com/livoras/simple-virtual-dom/blob/master/lib/patch.js)。然后就可以实际的进行使用：

```javascript
// 1. 构建虚拟DOM
var tree = el('div', {'id': 'container'}, [
    el('h1', {style: 'color: blue'}, ['simple virtal dom']),
    el('p', ['Hello, virtual-dom']),
    el('ul', [el('li')])
])

// 2. 通过虚拟DOM构建真正的DOM
var root = tree.render()
document.body.appendChild(root)

// 3. 生成新的虚拟DOM
var newTree = el('div', {'id': 'container'}, [
    el('h1', {style: 'color: red'}, ['simple virtal dom']),
    el('p', ['Hello, virtual-dom']),
    el('ul', [el('li'), el('li')])
])

// 4. 比较两棵虚拟DOM树的不同
var patches = diff(tree, newTree)

// 5. 在真正的DOM元素上应用变更
patch(root, patches)
```

当然这是非常粗糙的实践，实际中还需要处理事件监听等；生成虚拟 DOM 的时候也可以加入 JSX 语法。这些事情都做了的话，就可以构造一个简单的ReactJS了。

本文所实现的完整代码存放在 [Github](https://github.com/livoras/simple-virtual-dom)，仅供学习。

### References

https://github.com/Matt-Esch/virtual-dom/blob/master/vtree/diff.js

## Tip

## Share(http源码分析)

### 服务管理

- HTTPServer继承socketserver.TCPServer，建立实例后serve_forever监听客户端请求

### 请求头校验

- http请求完整请求HTTP-message = start-line *( header-field CRLF ) CRLF [ message-body ]
- http请求头行格式request-start-line = method SP request-target SP HTTP-version CRLF(SP空格 CRLF \r\n)
- http请求头格式header-field = field-name ":" OWS field-value OWS(OWS空格或制表符)
- SimpleHTTPRequestHandler请求入口handle，handle_one_request处理具体请求
- handle_one_request先检测第一行长度，调用parse_request详细解析第一行
  - 请求方法不能忽略不传，调用请求方法处理请求内容
- parse_request中做的校验
  - 版本大于HTTP/1.1才支持多次请求，否则解析完本次请求后结束
  - 不支持HTTP/2.0及以上版本
  - 客户端不传版本默认HTTP/0.9，只支持GET请求
  - 请求头Connection代表客户端要求一次请求或多次请求
- http.client.parse_headers处理请求头
  - 独行'\r\n', '\n', ''代表请求头结束

### 请求方法处理请求返回(GET流程)

- http请求返回格式HTTP-message = start-line *( header-field CRLF ) CRLF [ message-body ]
- http请求返回头行格式status-line = HTTP-version SP status-code SP reason-phrase CRLF(SP空格 CRLF \r\n)
- http请求头格式header-field = field-name ":" OWS field-value OWS(OWS空格或制表符)

- send_head设置返回请求头，调用translate_path处理资源，
  - translate_path处理资源路径，以?或#结束资源路径，urllib.parse.unquote处理url编码，资源本地绝对路径为程序当前路径加请求资源路径
  - guess_type处理请求资源类型
  - 资源文件不存在返回404，资源文件存在返回200
  - send_response_only生成status-line
  - send_header生成header-field，HTTP/0.9不支持其他头字段
  - send_response生成Server信息和Date信息
  - 生成Content-type信息和Content-Length信息
  - end_headers以\r\n结束请求头

- SimpleHTTPRequestHandler没有实现do_POST方法，不支持post请求

```python
# 服务端
from server import HTTPServer, SimpleHTTPRequestHandler


def service_demo():
    print('service begin')
    http = HTTPServer(('0.0.0.0', 8081), SimpleHTTPRequestHandler)
    http.serve_forever(1)
    print('service end')


if __name__ == '__main__':
    service_demo()
    
# 客户端
from client import HTTPConnection
import json


def client_demo():
    client = HTTPConnection('127.0.0.1', 8081)
    # headers = {'test_head': 'nxx'}
    # client.request('GET', '/index.html', headers=headers)
    headers = {'Connection': 'keep-alive'}
    body = json.dumps({'test_head': 'nxx'}).encode('utf-8')
    client.request('POST', '/api/books', body=body, headers=headers)
    respnese = client.getresponse()
    print(respnese.read())
    # print(json.loads(respnese.read()))


if __name__ == '__main__':
    client_demo()
  
# 客户端https
from http.client import HTTPSConnection
import ssl
import json


def client_demo():
    client = HTTPSConnection('blog.csdn.net', context=ssl._create_unverified_context())
    client.request('GET', '/community/home-api/v1/get-business-list?page=1&size=20&businessType=blog&orderby=&noMore=false&username=weixin_37702639')
    respnese = client.getresponse()
    print(json.loads(respnese.read()))


if __name__ == '__main__':
    client_demo()
```

### 简化源码辅助理解主流程

#### 服务端

```python
__version__ = "0.6"

__all__ = ["HTTPServer", "BaseHTTPRequestHandler", "SimpleHTTPRequestHandler"]

import copy
import datetime
import email.utils
import html
import http.client
import io
import mimetypes
import os
import posixpath
import select
import shutil
import socket  # For gethostbyaddr()
import socketserver
import sys
import time
import urllib.parse

from http import HTTPStatus

# Default error message template
DEFAULT_ERROR_MESSAGE = """\
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
        "http://www.w3.org/TR/html4/strict.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
        <title>Error response</title>
    </head>
    <body>
        <h1>Error response</h1>
        <p>Error code: %(code)d</p>
        <p>Message: %(message)s.</p>
        <p>Error code explanation: %(code)s - %(explain)s.</p>
    </body>
</html>
"""

DEFAULT_ERROR_CONTENT_TYPE = "text/html;charset=utf-8"


class HTTPServer(socketserver.TCPServer):

    allow_reuse_address = 1  # Seems to make sense in testing environment

    def server_bind(self):
        """Override server_bind to store the server name."""
        socketserver.TCPServer.server_bind(self)
        host, port = self.server_address[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port


class BaseHTTPRequestHandler(socketserver.StreamRequestHandler):
    # The Python system version, truncated to its first component.
    sys_version = "Python/" + sys.version.split()[0]

    # The server software version.  You may want to override this.
    # The format is multiple whitespace-separated strings,
    # where each string is of the form name[/version].
    server_version = "BaseHTTP/" + __version__

    error_message_format = DEFAULT_ERROR_MESSAGE
    error_content_type = DEFAULT_ERROR_CONTENT_TYPE

    # The default request version.  This only affects responses up until
    # the point where the request line is parsed, so it mainly decides what
    # the client gets back when sending a malformed request line.
    # Most web servers default to HTTP 0.9, i.e. don't send a status line.
    default_request_version = "HTTP/0.9"

    def parse_request(self):
        self.command = None  # set in case of error on the first line
        self.request_version = version = self.default_request_version
        self.close_connection = True
        requestline = str(self.raw_requestline, 'iso-8859-1')
        requestline = requestline.rstrip('\r\n')
        self.requestline = requestline
        words = requestline.split()
        if len(words) == 0:
            return False

        if len(words) >= 3:  # Enough to determine protocol version
            version = words[-1]
            try:
                if not version.startswith('HTTP/'):
                    raise ValueError
                base_version_number = version.split('/', 1)[1]
                version_number = base_version_number.split(".")
                # RFC 2145 section 3.1 says there can be only one "." and
                #   - major and minor numbers MUST be treated as
                #      separate integers;
                #   - HTTP/2.4 is a lower version than HTTP/2.13, which in
                #      turn is lower than HTTP/12.3;
                #   - Leading zeros MUST be ignored by recipients.
                if len(version_number) != 2:
                    raise ValueError
                version_number = int(version_number[0]), int(version_number[1])
            except (ValueError, IndexError):
                self.send_error(HTTPStatus.BAD_REQUEST, "Bad request version (%r)" % version)
                return False
            if version_number >= (1, 1) and self.protocol_version >= "HTTP/1.1":
                self.close_connection = False
            if version_number >= (2, 0):
                self.send_error(
                    HTTPStatus.HTTP_VERSION_NOT_SUPPORTED, "Invalid HTTP version (%s)" % base_version_number
                )
                return False
            self.request_version = version

        if not 2 <= len(words) <= 3:
            self.send_error(HTTPStatus.BAD_REQUEST, "Bad request syntax (%r)" % requestline)
            return False
        command, path = words[:2]
        if len(words) == 2:
            self.close_connection = True
            if command != 'GET':
                self.send_error(HTTPStatus.BAD_REQUEST, "Bad HTTP/0.9 request type (%r)" % command)
                return False
        self.command, self.path = command, path

        # Examine the headers and look for a Connection directive.
        try:
            self.headers = http.client.parse_headers(self.rfile, _class=self.MessageClass)
        except http.client.LineTooLong as err:
            self.send_error(HTTPStatus.REQUEST_HEADER_FIELDS_TOO_LARGE, "Line too long", str(err))
            return False
        except http.client.HTTPException as err:
            self.send_error(HTTPStatus.REQUEST_HEADER_FIELDS_TOO_LARGE, "Too many headers", str(err))
            return False

        conntype = self.headers.get('Connection', "")
        if conntype.lower() == 'close':
            self.close_connection = True
        elif (conntype.lower() == 'keep-alive' and self.protocol_version >= "HTTP/1.1"):
            self.close_connection = False
        # Examine the headers and look for an Expect directive
        expect = self.headers.get('Expect', "")
        if (
            expect.lower() == "100-continue" and self.protocol_version >= "HTTP/1.1"
            and self.request_version >= "HTTP/1.1"
        ):
            if not self.handle_expect_100():
                return False
        return True

    def handle_expect_100(self):
        """Decide what to do with an "Expect: 100-continue" header.

        If the client is expecting a 100 Continue response, we must
        respond with either a 100 Continue or a final response before
        waiting for the request body. The default is to always respond
        with a 100 Continue. You can behave differently (for example,
        reject unauthorized requests) by overriding this method.

        This method should either return True (possibly after sending
        a 100 Continue response) or send an error response and return
        False.

        """
        self.send_response_only(HTTPStatus.CONTINUE)
        self.end_headers()
        return True

    def handle_one_request(self):
        try:
            self.raw_requestline = self.rfile.readline(65537)
            if len(self.raw_requestline) > 65536:
                self.requestline = ''
                self.request_version = ''
                self.command = ''
                self.send_error(HTTPStatus.REQUEST_URI_TOO_LONG)
                return
            if not self.raw_requestline:
                self.close_connection = True
                return
            if not self.parse_request():
                # An error code has been sent, just exit
                return
            mname = 'do_' + self.command
            if not hasattr(self, mname):
                self.send_error(HTTPStatus.NOT_IMPLEMENTED, "Unsupported method (%r)" % self.command)
                return
            method = getattr(self, mname)
            method()
            self.wfile.flush()  #actually send the response if not already done.
        except socket.timeout as e:
            #a read or a write timed out.  Discard this connection
            self.close_connection = True
            return

    def handle(self):
        """Handle multiple requests if necessary."""
        self.close_connection = True

        self.handle_one_request()
        while not self.close_connection:
            self.handle_one_request()

    def send_error(self, code, message=None, explain=None):
        try:
            shortmsg, longmsg = self.responses[code]
        except KeyError:
            shortmsg, longmsg = '???', '???'
        if message is None:
            message = shortmsg
        if explain is None:
            explain = longmsg
        self.send_response(code, message)
        self.send_header('Connection', 'close')

        # Message body is omitted for cases described in:
        #  - RFC7230: 3.3. 1xx, 204(No Content), 304(Not Modified)
        #  - RFC7231: 6.3.6. 205(Reset Content)
        body = None
        if (code >= 200 and code not in (HTTPStatus.NO_CONTENT, HTTPStatus.RESET_CONTENT, HTTPStatus.NOT_MODIFIED)):
            # HTML encode to prevent Cross Site Scripting attacks
            # (see bug #1100201)
            content = (
                self.error_message_format % {
                    'code': code,
                    'message': html.escape(message, quote=False),
                    'explain': html.escape(explain, quote=False)
                }
            )
            body = content.encode('UTF-8', 'replace')
            self.send_header("Content-Type", self.error_content_type)
            self.send_header('Content-Length', str(len(body)))
        self.end_headers()

        if self.command != 'HEAD' and body:
            self.wfile.write(body)

    def send_response(self, code, message=None):
        self.send_response_only(code, message)
        self.send_header('Server', self.version_string())
        self.send_header('Date', self.date_time_string())

    def send_response_only(self, code, message=None):
        """Send the response header only."""
        if self.request_version != 'HTTP/0.9':
            if message is None:
                if code in self.responses:
                    message = self.responses[code][0]
                else:
                    message = ''
            if not hasattr(self, '_headers_buffer'):
                self._headers_buffer = []
            self._headers_buffer.append(
                ("%s %d %s\r\n" % (self.protocol_version, code, message)).encode('latin-1', 'strict')
            )

    def send_header(self, keyword, value):
        """Send a MIME header to the headers buffer."""
        if self.request_version != 'HTTP/0.9':
            if not hasattr(self, '_headers_buffer'):
                self._headers_buffer = []
            self._headers_buffer.append(("%s: %s\r\n" % (keyword, value)).encode('latin-1', 'strict'))

        if keyword.lower() == 'connection':
            if value.lower() == 'close':
                self.close_connection = True
            elif value.lower() == 'keep-alive':
                self.close_connection = False

    def end_headers(self):
        """Send the blank line ending the MIME headers."""
        if self.request_version != 'HTTP/0.9':
            self._headers_buffer.append(b"\r\n")
            self.flush_headers()

    def flush_headers(self):
        if hasattr(self, '_headers_buffer'):
            self.wfile.write(b"".join(self._headers_buffer))
            self._headers_buffer = []

    def version_string(self):
        """Return the server software version string."""
        return self.server_version + ' ' + self.sys_version

    def date_time_string(self, timestamp=None):
        """Return the current date and time formatted for a message header."""
        if timestamp is None:
            timestamp = time.time()
        return email.utils.formatdate(timestamp, usegmt=True)

    weekdayname = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']

    monthname = [None, 'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

    # Essentially static class variables

    # The version of the HTTP protocol we support.
    # Set this to HTTP/1.1 to enable automatic keepalive
    protocol_version = "HTTP/1.0"

    # MessageClass used to parse headers
    MessageClass = http.client.HTTPMessage

    # hack to maintain backwards compatibility
    responses = {v: (v.phrase, v.description) for v in HTTPStatus.__members__.values()}


class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):

    server_version = "SimpleHTTP/" + __version__

    def __init__(self, *args, directory=None, **kwargs):
        if directory is None:
            directory = os.getcwd()
        self.directory = directory
        super().__init__(*args, **kwargs)

    def do_GET(self):
        """Serve a GET request."""
        f = self.send_head()
        if f:
            try:
                self.copyfile(f, self.wfile)
            finally:
                f.close()

    def do_HEAD(self):
        """Serve a HEAD request."""
        f = self.send_head()
        if f:
            f.close()

    def send_head(self):
        """Common code for GET and HEAD commands.

        This sends the response code and MIME headers.

        Return value is either a file object (which has to be copied
        to the outputfile by the caller unless the command was HEAD,
        and must be closed by the caller under all circumstances), or
        None, in which case the caller has nothing further to do.

        """
        path = self.translate_path(self.path)
        f = None
        if os.path.isdir(path):
            parts = urllib.parse.urlsplit(self.path)
            if not parts.path.endswith('/'):
                # redirect browser - doing basically what apache does
                self.send_response(HTTPStatus.MOVED_PERMANENTLY)
                new_parts = (parts[0], parts[1], parts[2] + '/', parts[3], parts[4])
                new_url = urllib.parse.urlunsplit(new_parts)
                self.send_header("Location", new_url)
                self.end_headers()
                return None
            for index in "index.html", "index.htm":
                index = os.path.join(path, index)
                if os.path.exists(index):
                    path = index
                    break
            else:
                return self.list_directory(path)
        ctype = self.guess_type(path)
        try:
            f = open(path, 'rb')
        except OSError:
            self.send_error(HTTPStatus.NOT_FOUND, "File not found")
            return None

        try:
            fs = os.fstat(f.fileno())
            # Use browser cache if possible
            if ("If-Modified-Since" in self.headers and "If-None-Match" not in self.headers):
                # compare If-Modified-Since and time of last file modification
                try:
                    ims = email.utils.parsedate_to_datetime(self.headers["If-Modified-Since"])
                except (TypeError, IndexError, OverflowError, ValueError):
                    # ignore ill-formed values
                    pass
                else:
                    if ims.tzinfo is None:
                        # obsolete format with no timezone, cf.
                        # https://tools.ietf.org/html/rfc7231#section-7.1.1.1
                        ims = ims.replace(tzinfo=datetime.timezone.utc)
                    if ims.tzinfo is datetime.timezone.utc:
                        # compare to UTC datetime of last modification
                        last_modif = datetime.datetime.fromtimestamp(fs.st_mtime, datetime.timezone.utc)
                        # remove microseconds, like in If-Modified-Since
                        last_modif = last_modif.replace(microsecond=0)

                        if last_modif <= ims:
                            self.send_response(HTTPStatus.NOT_MODIFIED)
                            self.end_headers()
                            f.close()
                            return None

            self.send_response(HTTPStatus.OK)
            self.send_header("Content-type", ctype)
            self.send_header("Content-Length", str(fs[6]))
            self.send_header("Last-Modified", self.date_time_string(fs.st_mtime))
            self.end_headers()
            return f
        except:
            f.close()
            raise

    def list_directory(self, path):
        """Helper to produce a directory listing (absent index.html).

        Return value is either a file object, or None (indicating an
        error).  In either case, the headers are sent, making the
        interface the same as for send_head().

        """
        try:
            list = os.listdir(path)
        except OSError:
            self.send_error(HTTPStatus.NOT_FOUND, "No permission to list directory")
            return None
        list.sort(key=lambda a: a.lower())
        r = []
        try:
            displaypath = urllib.parse.unquote(self.path, errors='surrogatepass')
        except UnicodeDecodeError:
            displaypath = urllib.parse.unquote(path)
        displaypath = html.escape(displaypath, quote=False)
        enc = sys.getfilesystemencoding()
        title = 'Directory listing for %s' % displaypath
        r.append('<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" ' '"http://www.w3.org/TR/html4/strict.dtd">')
        r.append('<html>\n<head>')
        r.append('<meta http-equiv="Content-Type" ' 'content="text/html; charset=%s">' % enc)
        r.append('<title>%s</title>\n</head>' % title)
        r.append('<body>\n<h1>%s</h1>' % title)
        r.append('<hr>\n<ul>')
        for name in list:
            fullname = os.path.join(path, name)
            displayname = linkname = name
            # Append / for directories or @ for symbolic links
            if os.path.isdir(fullname):
                displayname = name + "/"
                linkname = name + "/"
            if os.path.islink(fullname):
                displayname = name + "@"
                # Note: a link to a directory displays with @ and links with /
            r.append(
                '<li><a href="%s">%s</a></li>' %
                (urllib.parse.quote(linkname, errors='surrogatepass'), html.escape(displayname, quote=False))
            )
        r.append('</ul>\n<hr>\n</body>\n</html>\n')
        encoded = '\n'.join(r).encode(enc, 'surrogateescape')
        f = io.BytesIO()
        f.write(encoded)
        f.seek(0)
        self.send_response(HTTPStatus.OK)
        self.send_header("Content-type", "text/html; charset=%s" % enc)
        self.send_header("Content-Length", str(len(encoded)))
        self.end_headers()
        return f

    def translate_path(self, path):
        """Translate a /-separated PATH to the local filename syntax.

        Components that mean special things to the local file system
        (e.g. drive or directory names) are ignored.  (XXX They should
        probably be diagnosed.)

        """
        # abandon query parameters
        path = path.split('?', 1)[0]
        path = path.split('#', 1)[0]
        # Don't forget explicit trailing slash when normalizing. Issue17324
        trailing_slash = path.rstrip().endswith('/')
        try:
            path = urllib.parse.unquote(path, errors='surrogatepass')
        except UnicodeDecodeError:
            path = urllib.parse.unquote(path)
        path = posixpath.normpath(path)
        words = path.split('/')
        words = filter(None, words)
        path = self.directory
        for word in words:
            if os.path.dirname(word) or word in (os.curdir, os.pardir):
                # Ignore components that are not a simple file/directory name
                continue
            path = os.path.join(path, word)
        if trailing_slash:
            path += '/'
        return path

    def copyfile(self, source, outputfile):
        """Copy all data between two file objects.

        The SOURCE argument is a file object open for reading
        (or anything with a read() method) and the DESTINATION
        argument is a file object open for writing (or
        anything with a write() method).

        The only reason for overriding this would be to change
        the block size or perhaps to replace newlines by CRLF
        -- note however that this the default server uses this
        to copy binary data as well.

        """
        shutil.copyfileobj(source, outputfile)

    def guess_type(self, path):
        """Guess the type of a file.

        Argument is a PATH (a filename).

        Return value is a string of the form type/subtype,
        usable for a MIME Content-type header.

        The default implementation looks the file's extension
        up in the table self.extensions_map, using application/octet-stream
        as a default; however it would be permissible (if
        slow) to look inside the data to make a better guess.

        """

        base, ext = posixpath.splitext(path)
        if ext in self.extensions_map:
            return self.extensions_map[ext]
        ext = ext.lower()
        if ext in self.extensions_map:
            return self.extensions_map[ext]
        else:
            return self.extensions_map['']

    if not mimetypes.inited:
        mimetypes.init()  # try to read system mime.types
    extensions_map = mimetypes.types_map.copy()
    extensions_map.update(
        {
            '': 'application/octet-stream',  # Default
            '.py': 'text/plain',
            '.c': 'text/plain',
            '.h': 'text/plain',
        }
    )
```

#### 客户端

```python
import email.parser
import email.message
import http
import io
import re
import socket
import collections.abc
from urllib.parse import urlsplit
import ssl

__all__ = ["HTTPResponse", "HTTPConnection", "HTTPSConnection"]

HTTP_PORT = 80
HTTPS_PORT = 443

_UNKNOWN = 'UNKNOWN'

# hack to maintain backwards compatibility
globals().update(http.HTTPStatus.__members__)

# maximal line length when calling readline().
_MAXLINE = 65536
_MAXHEADERS = 100

_METHODS_EXPECTING_BODY = {'PATCH', 'POST', 'PUT'}


class HTTPMessage(email.message.Message):
    # XXX The only usage of this method is in
    # http.server.CGIHTTPRequestHandler.  Maybe move the code there so
    # that it doesn't need to be part of the public API.  The API has
    # never been defined so this could cause backwards compatibility
    # issues.

    def getallmatchingheaders(self, name):
        """Find all header lines matching a given header name.

        Look through the list of headers and find all lines matching a given
        header name (and their continuation lines).  A list of the lines is
        returned, without interpretation.  If the header does not occur, an
        empty list is returned.  If the header occurs multiple times, all
        occurrences are returned.  Case is not important in the header name.

        """
        name = name.lower() + ':'
        n = len(name)
        lst = []
        hit = 0
        for line in self.keys():
            if line[:n].lower() == name:
                hit = 1
            elif not line[:1].isspace():
                hit = 0
            if hit:
                lst.append(line)
        return lst


def parse_headers(fp, _class=HTTPMessage):
    """Parses only RFC2822 headers from a file pointer.

    email Parser wants to see strings rather than bytes.
    But a TextIOWrapper around self.rfile would buffer too many bytes
    from the stream, bytes which we later need to read as bytes.
    So we read the correct bytes here, as bytes, for email Parser
    to parse.

    """
    headers = []
    while True:
        line = fp.readline(_MAXLINE + 1)
        headers.append(line)
        if line in (b'\r\n', b'\n', b''):
            break
    hstring = b''.join(headers).decode('iso-8859-1')
    return email.parser.Parser(_class=_class).parsestr(hstring)


class HTTPResponse(io.BufferedIOBase):

    def __init__(self, sock, debuglevel=0, method=None, url=None):
        self.fp = sock.makefile("rb")
        self.debuglevel = debuglevel
        self._method = method

        self.headers = self.msg = None

        # from the Status-Line of the response
        self.version = _UNKNOWN  # HTTP-Version
        self.status = _UNKNOWN  # Status-Code
        self.reason = _UNKNOWN  # Reason-Phrase

        self.chunked = _UNKNOWN  # is "chunked" being used?
        self.chunk_left = _UNKNOWN  # bytes left to read in current chunk
        self.length = _UNKNOWN  # number of bytes left in response
        self.will_close = _UNKNOWN  # conn will close at end of response

    def _read_status(self):
        line = str(self.fp.readline(_MAXLINE + 1), "iso-8859-1")
        version, status, reason = line.split(None, 2)
        return version, int(status), reason

    def begin(self):
        if self.headers is not None:
            # we've already started reading the response
            return

        # read until we get a non-100 response
        while True:
            version, status, reason = self._read_status()
            if status != CONTINUE:
                break
            # skip the header from the 100 response
            while True:
                skip = self.fp.readline(_MAXLINE + 1)
                skip = skip.strip()
                if not skip:
                    break

        self.code = self.status = status
        self.reason = reason.strip()
        if version in ("HTTP/1.0", "HTTP/0.9"):
            # Some servers might still return "0.9", treat it as 1.0 anyway
            self.version = 10
        elif version.startswith("HTTP/1."):
            self.version = 11  # use HTTP/1.1 code for HTTP/1.x where x>=1

        self.headers = self.msg = parse_headers(self.fp)

        if self.debuglevel > 0:
            for hdr, val in self.headers.items():
                print("header:", hdr + ":", val)

        # are we using the chunked-style of transfer encoding?
        tr_enc = self.headers.get("transfer-encoding")
        if tr_enc and tr_enc.lower() == "chunked":
            self.chunked = True
            self.chunk_left = None
        else:
            self.chunked = False

        # will the connection close at the end of the response?
        self.will_close = self._check_close()

        # do we have a Content-Length?
        # NOTE: RFC 2616, S4.4, #3 says we ignore this if tr_enc is "chunked"
        self.length = None
        length = self.headers.get("content-length")
        if length and not self.chunked:
            try:
                self.length = int(length)
            except ValueError:
                self.length = None
            else:
                if self.length < 0:  # ignore nonsensical negative lengths
                    self.length = None
        else:
            self.length = None

        # does the body have a fixed length? (of zero)
        if (status == NO_CONTENT or status == NOT_MODIFIED or 100 <= status < 200 or  # 1xx codes
            self._method == "HEAD"):
            self.length = 0

        # if the connection remains open, and we aren't using chunked, and
        # a content-length was not provided, then assume that the connection
        # WILL close.
        if (not self.will_close and not self.chunked and self.length is None):
            self.will_close = True

    def _check_close(self):
        conn = self.headers.get("connection")
        if self.version == 11:
            # An HTTP/1.1 proxy is assumed to stay open unless
            # explicitly closed.
            if conn and "close" in conn.lower():
                return True
            return False

        # Some HTTP/1.0 implementations have support for persistent
        # connections, using rules different than HTTP/1.1.

        # For older HTTP, Keep-Alive indicates persistent connection.
        if self.headers.get("keep-alive"):
            return False

        # At least Akamai returns a "Connection: Keep-Alive" header,
        # which was supposed to be sent by the client.
        if conn and "keep-alive" in conn.lower():
            return False

        # Proxy-Connection is a netscape hack.
        pconn = self.headers.get("proxy-connection")
        if pconn and "keep-alive" in pconn.lower():
            return False

        # otherwise, assume it will close
        return True

    def _close_conn(self):
        fp = self.fp
        self.fp = None
        fp.close()

    def close(self):
        try:
            super().close()  # set "closed" flag
        finally:
            if self.fp:
                self._close_conn()

    # These implementations are for the benefit of io.BufferedReader.

    # XXX This class should probably be revised to act more like
    # the "raw stream" that BufferedReader expects.

    def flush(self):
        super().flush()
        if self.fp:
            self.fp.flush()

    def readable(self):
        """Always returns True"""
        return True

    # End of "raw stream" methods

    def isclosed(self):
        """True if the connection is closed."""
        # NOTE: it is possible that we will not ever call self.close(). This
        #       case occurs when will_close is TRUE, length is None, and we
        #       read up to the last byte, but NOT past it.
        #
        # IMPLIES: if will_close is FALSE, then self.close() will ALWAYS be
        #          called, meaning self.isclosed() is meaningful.
        return self.fp is None

    def read(self, amt=None):
        if self.fp is None:
            return b""

        if self._method == "HEAD":
            self._close_conn()
            return b""

        if amt is not None:
            # Amount is given, implement using readinto
            b = bytearray(amt)
            n = self.readinto(b)
            return memoryview(b)[:n].tobytes()
        else:
            # Amount is not given (unbounded read) so we must check self.length
            # and self.chunked

            if self.chunked:
                return self._readall_chunked()

            if self.length is None:
                s = self.fp.read()
            else:
                try:
                    s = self._safe_read(self.length)
                except IncompleteRead:
                    self._close_conn()
                    raise
                self.length = 0
            self._close_conn()  # we read everything
            return s

    def readinto(self, b):
        """Read up to len(b) bytes into bytearray b and return the number
        of bytes read.
        """

        if self.fp is None:
            return 0

        if self._method == "HEAD":
            self._close_conn()
            return 0

        if self.chunked:
            return self._readinto_chunked(b)

        if self.length is not None:
            if len(b) > self.length:
                # clip the read to the "end of response"
                b = memoryview(b)[0:self.length]

        # we do not use _safe_read() here because this may be a .will_close
        # connection, and the user is reading more bytes than will be provided
        # (for example, reading in 1k chunks)
        n = self.fp.readinto(b)
        if not n and b:
            self._close_conn()
        elif self.length is not None:
            self.length -= n
            if not self.length:
                self._close_conn()
        return n

    def _read_next_chunk_size(self):
        # Read the next chunk size from the file
        line = self.fp.readline(_MAXLINE + 1)
        i = line.find(b";")
        if i >= 0:
            line = line[:i]  # strip chunk-extensions
        try:
            return int(line, 16)
        except ValueError:
            # close the connection as protocol synchronisation is
            # probably lost
            self._close_conn()
            raise

    def _read_and_discard_trailer(self):
        # read and discard trailer up to the CRLF terminator
        ### note: we shouldn't have any trailers!
        while True:
            line = self.fp.readline(_MAXLINE + 1)
            if not line:
                # a vanishingly small number of sites EOF without
                # sending the trailer
                break
            if line in (b'\r\n', b'\n', b''):
                break

    def _get_chunk_left(self):
        # return self.chunk_left, reading a new chunk if necessary.
        # chunk_left == 0: at the end of the current chunk, need to close it
        # chunk_left == None: No current chunk, should read next.
        # This function returns non-zero or None if the last chunk has
        # been read.
        chunk_left = self.chunk_left
        if not chunk_left:  # Can be 0 or None
            if chunk_left is not None:
                # We are at the end of chunk, discard chunk end
                self._safe_read(2)  # toss the CRLF at the end of the chunk
            chunk_left = self._read_next_chunk_size()
            if chunk_left == 0:
                # last chunk: 1*("0") [ chunk-extension ] CRLF
                self._read_and_discard_trailer()
                # we read everything; close the "file"
                self._close_conn()
                chunk_left = None
            self.chunk_left = chunk_left
        return chunk_left

    def _readall_chunked(self):
        assert self.chunked != _UNKNOWN
        value = []
        while True:
            chunk_left = self._get_chunk_left()
            if chunk_left is None:
                break
            value.append(self._safe_read(chunk_left))
            self.chunk_left = 0
        return b''.join(value)

    def _readinto_chunked(self, b):
        assert self.chunked != _UNKNOWN
        total_bytes = 0
        mvb = memoryview(b)
        while True:
            chunk_left = self._get_chunk_left()
            if chunk_left is None:
                return total_bytes

            if len(mvb) <= chunk_left:
                n = self._safe_readinto(mvb)
                self.chunk_left = chunk_left - n
                return total_bytes + n

            temp_mvb = mvb[:chunk_left]
            n = self._safe_readinto(temp_mvb)
            mvb = mvb[n:]
            total_bytes += n
            self.chunk_left = 0

    def _safe_read(self, amt):
        data = self.fp.read(amt)
        return data

    def _safe_readinto(self, b):
        """Same as _safe_read, but for reading into a buffer."""
        amt = len(b)
        n = self.fp.readinto(b)
        return n

    def read1(self, n=-1):
        """Read with at most one underlying system call.  If at least one
        byte is buffered, return that instead.
        """
        if self.fp is None or self._method == "HEAD":
            return b""
        if self.chunked:
            return self._read1_chunked(n)
        if self.length is not None and (n < 0 or n > self.length):
            n = self.length
        result = self.fp.read1(n)
        if not result and n:
            self._close_conn()
        elif self.length is not None:
            self.length -= len(result)
        return result

    def peek(self, n=-1):
        # Having this enables IOBase.readline() to read more than one
        # byte at a time
        if self.fp is None or self._method == "HEAD":
            return b""
        if self.chunked:
            return self._peek_chunked(n)
        return self.fp.peek(n)

    def readline(self, limit=-1):
        if self.fp is None or self._method == "HEAD":
            return b""
        if self.chunked:
            # Fallback to IOBase readline which uses peek() and read()
            return super().readline(limit)
        if self.length is not None and (limit < 0 or limit > self.length):
            limit = self.length
        result = self.fp.readline(limit)
        if not result and limit:
            self._close_conn()
        elif self.length is not None:
            self.length -= len(result)
        return result

    def _read1_chunked(self, n):
        # Strictly speaking, _get_chunk_left() may cause more than one read,
        # but that is ok, since that is to satisfy the chunked protocol.
        chunk_left = self._get_chunk_left()
        if chunk_left is None or n == 0:
            return b''
        if not (0 <= n <= chunk_left):
            n = chunk_left  # if n is negative or larger than chunk_left
        read = self.fp.read1(n)
        self.chunk_left -= len(read)
        return read

    def _peek_chunked(self, n):
        # Strictly speaking, _get_chunk_left() may cause more than one read,
        # but that is ok, since that is to satisfy the chunked protocol.
        try:
            chunk_left = self._get_chunk_left()
        except IncompleteRead:
            return b''  # peek doesn't worry about protocol
        if chunk_left is None:
            return b''  # eof
        # peek is allowed to return more than requested.  Just request the
        # entire chunk, and truncate what we get.
        return self.fp.peek(chunk_left)[:chunk_left]

    def fileno(self):
        return self.fp.fileno()

    def getheader(self, name, default=None):
        headers = self.headers.get_all(name) or default
        if isinstance(headers, str) or not hasattr(headers, '__iter__'):
            return headers
        else:
            return ', '.join(headers)

    def getheaders(self):
        return list(self.headers.items())

    # We override IOBase.__iter__ so that it doesn't check for closed-ness

    def __iter__(self):
        return self

    # For compatibility with old-style urllib responses.

    def info(self):
        '''Returns an instance of the class mimetools.Message containing
        meta-information associated with the URL.

        When the method is HTTP, these headers are those returned by
        the server at the head of the retrieved HTML page (including
        Content-Length and Content-Type).

        When the method is FTP, a Content-Length header will be
        present if (as is now usual) the server passed back a file
        length in response to the FTP retrieval request. A
        Content-Type header will be present if the MIME type can be
        guessed.

        When the method is local-file, returned headers will include
        a Date representing the file's last-modified time, a
        Content-Length giving file size, and a Content-Type
        containing a guess at the file's type. See also the
        description of the mimetools module.

        '''
        return self.headers

    def geturl(self):
        '''Return the real URL of the page.

        In some cases, the HTTP server redirects a client to another
        URL. The urlopen() function handles this transparently, but in
        some cases the caller needs to know which URL the client was
        redirected to. The geturl() method can be used to get at this
        redirected URL.

        '''
        return self.url

    def getcode(self):
        '''Return the HTTP status code that was sent with the response,
        or None if the URL is not an HTTP URL.

        '''
        return self.status


class HTTPConnection:

    _http_vsn = 11
    _http_vsn_str = 'HTTP/1.1'

    response_class = HTTPResponse
    default_port = HTTP_PORT
    auto_open = 1
    debuglevel = 0

    @staticmethod
    def _is_textIO(stream):
        """Test whether a file-like object is a text or a binary stream.
        """
        return isinstance(stream, io.TextIOBase)

    @staticmethod
    def _get_content_length(body, method):
        """Get the content-length based on the body.

        If the body is None, we set Content-Length: 0 for methods that expect
        a body (RFC 7230, Section 3.3.2). We also set the Content-Length for
        any method if the body is a str or bytes-like object and not a file.
        """
        if body is None:
            # do an explicit check for not None here to distinguish
            # between unset and set but empty
            if method.upper() in _METHODS_EXPECTING_BODY:
                return 0
            else:
                return None

        if hasattr(body, 'read'):
            # file-like object.
            return None

        try:
            # does it implement the buffer protocol (bytes, bytearray, array)?
            mv = memoryview(body)
            return mv.nbytes
        except TypeError:
            pass

        if isinstance(body, str):
            return len(body)

        return None

    def __init__(self, host, port=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT, source_address=None, blocksize=8192):
        self.timeout = timeout
        self.source_address = source_address
        self.blocksize = blocksize
        self.sock = None
        self._buffer = []
        self.__response = None
        self._method = None

        (self.host, self.port) = self._get_hostport(host, port)

        # This is stored as an instance variable to allow unit
        # tests to replace it with a suitable mockup
        self._create_connection = socket.create_connection

    def _get_hostport(self, host, port):
        if port is None:
            i = host.rfind(':')
            j = host.rfind(']')  # ipv6 addresses have [...]
            if i > j:
                try:
                    port = int(host[i + 1:])
                except ValueError:
                    if host[i + 1:] == "":  # http://foo.com:/ == http://foo.com/
                        port = self.default_port
                host = host[:i]
            else:
                port = self.default_port
            if host and host[0] == '[' and host[-1] == ']':
                host = host[1:-1]

        return (host, port)

    def connect(self):
        """Connect to the host and port specified in __init__."""
        self.sock = self._create_connection((self.host, self.port), self.timeout, self.source_address)
        self.sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)

    def close(self):
        """Close the connection to the HTTP server."""
        self.__state = _CS_IDLE
        try:
            sock = self.sock
            if sock:
                self.sock = None
                sock.close()  # close it manually... there may be other refs
        finally:
            response = self.__response
            if response:
                self.__response = None
                response.close()

    def send(self, data):
        """Send `data' to the server.
        ``data`` can be a string object, a bytes object, an array object, a
        file-like object that supports a .read() method, or an iterable object.
        """

        if self.sock is None:
            if self.auto_open:
                self.connect()

        if self.debuglevel > 0:
            print("send:", repr(data))
        if hasattr(data, "read"):
            if self.debuglevel > 0:
                print("sendIng a read()able")
            encode = self._is_textIO(data)
            if encode and self.debuglevel > 0:
                print("encoding file using iso-8859-1")
            while 1:
                datablock = data.read(self.blocksize)
                if not datablock:
                    break
                if encode:
                    datablock = datablock.encode("iso-8859-1")
                self.sock.sendall(datablock)
            return
        try:
            self.sock.sendall(data)
        except TypeError:
            if isinstance(data, collections.abc.Iterable):
                for d in data:
                    self.sock.sendall(d)
            else:
                raise TypeError("data should be a bytes-like object " "or an iterable, got %r" % type(data))

    def _output(self, s):
        """Add a line of output to the current request buffer.

        Assumes that the line does *not* end with \\r\\n.
        """
        self._buffer.append(s)

    def _read_readable(self, readable):
        if self.debuglevel > 0:
            print("sendIng a read()able")
        encode = self._is_textIO(readable)
        if encode and self.debuglevel > 0:
            print("encoding file using iso-8859-1")
        while True:
            datablock = readable.read(self.blocksize)
            if not datablock:
                break
            if encode:
                datablock = datablock.encode("iso-8859-1")
            yield datablock

    def _send_output(self, message_body=None, encode_chunked=False):
        """Send the currently buffered request and clear the buffer.

        Appends an extra \\r\\n to the buffer.
        A message_body may be specified, to be appended to the request.
        """
        self._buffer.extend((b"", b""))
        msg = b"\r\n".join(self._buffer)
        del self._buffer[:]
        self.send(msg)

        if message_body is not None:

            # create a consistent interface to message_body
            if hasattr(message_body, 'read'):
                # Let file-like take precedence over byte-like.  This
                # is needed to allow the current position of mmap'ed
                # files to be taken into account.
                chunks = self._read_readable(message_body)
            else:
                try:
                    # this is solely to check to see if message_body
                    # implements the buffer API.  it /would/ be easier
                    # to capture if PyObject_CheckBuffer was exposed
                    # to Python.
                    memoryview(message_body)
                except TypeError:
                    try:
                        chunks = iter(message_body)
                    except TypeError:
                        raise TypeError("message_body should be a bytes-like " "object or an iterable, got %r" % type(message_body))
                else:
                    # the object implements the buffer interface and
                    # can be passed directly into socket methods
                    chunks = (message_body,)

            for chunk in chunks:
                if not chunk:
                    if self.debuglevel > 0:
                        print('Zero length chunk ignored')
                    continue

                if encode_chunked and self._http_vsn == 11:
                    # chunked encoding
                    chunk = f'{len(chunk):X}\r\n'.encode('ascii') + chunk \
                        + b'\r\n'
                self.send(chunk)

            if encode_chunked and self._http_vsn == 11:
                # end chunked transfer
                self.send(b'0\r\n\r\n')

    def putrequest(self, method, url, skip_host=False, skip_accept_encoding=False):
        """Send a request to the server.

        `method' specifies an HTTP request method, e.g. 'GET'.
        `url' specifies the object being requested, e.g. '/index.html'.
        `skip_host' if True does not add automatically a 'Host:' header
        `skip_accept_encoding' if True does not add automatically an
           'Accept-Encoding:' header
        """

        # if a prior response has been completed, then forget about it.
        if self.__response and self.__response.isclosed():
            self.__response = None

        # Save the method for use later in the response phase
        self._method = method

        url = url or '/'

        request = '%s %s %s' % (method, url, self._http_vsn_str)

        self._output(request.encode('ascii'))

        if self._http_vsn == 11:
            # Issue some standard headers for better HTTP/1.1 compliance

            if not skip_host:
                # this header is issued *only* for HTTP/1.1
                # connections. more specifically, this means it is
                # only issued when the client uses the new
                # HTTPConnection() class. backwards-compat clients
                # will be using HTTP/1.0 and those clients may be
                # issuing this header themselves. we should NOT issue
                # it twice; some web servers (such as Apache) barf
                # when they see two Host: headers

                # If we need a non-standard port,include it in the
                # header.  If the request is going through a proxy,
                # but the host of the actual URL, not the host of the
                # proxy.

                netloc = ''
                if url.startswith('http'):
                    nil, netloc, nil, nil, nil = urlsplit(url)

                if netloc:
                    try:
                        netloc_enc = netloc.encode("ascii")
                    except UnicodeEncodeError:
                        netloc_enc = netloc.encode("idna")
                    self.putheader('Host', netloc_enc)
                else:
                    host = self.host
                    port = self.port

                    try:
                        host_enc = host.encode("ascii")
                    except UnicodeEncodeError:
                        host_enc = host.encode("idna")

                    # As per RFC 273, IPv6 address should be wrapped with []
                    # when used as Host header

                    if host.find(':') >= 0:
                        host_enc = b'[' + host_enc + b']'

                    if port == self.default_port:
                        self.putheader('Host', host_enc)
                    else:
                        host_enc = host_enc.decode("ascii")
                        self.putheader('Host', "%s:%s" % (host_enc, port))

            # note: we are assuming that clients will not attempt to set these
            #       headers since *this* library must deal with the
            #       consequences. this also means that when the supporting
            #       libraries are updated to recognize other forms, then this
            #       code should be changed (removed or updated).

            # we only want a Content-Encoding of "identity" since we don't
            # support encodings such as x-gzip or x-deflate.
            if not skip_accept_encoding:
                self.putheader('Accept-Encoding', 'identity')

            # we can accept "chunked" Transfer-Encodings, but no others
            # NOTE: no TE header implies *only* "chunked"
            #self.putheader('TE', 'chunked')

            # if TE is supplied in the header, then it must appear in a
            # Connection header.
            #self.putheader('Connection', 'TE')

        else:
            # For HTTP/1.0, the server will assume "not chunked"
            pass

    def putheader(self, header, *values):
        if hasattr(header, 'encode'):
            header = header.encode('ascii')

        values = list(values)
        for i, one_value in enumerate(values):
            if hasattr(one_value, 'encode'):
                values[i] = one_value.encode('latin-1')
            elif isinstance(one_value, int):
                values[i] = str(one_value).encode('ascii')

        value = b'\r\n\t'.join(values)
        header = header + b': ' + value
        self._output(header)

    def endheaders(self, message_body=None, *, encode_chunked=False):
        self._send_output(message_body, encode_chunked=encode_chunked)

    def request(self, method, url, body=None, headers={}, *, encode_chunked=False):
        """Send a complete request to the server."""
        self._send_request(method, url, body, headers, encode_chunked)

    def _send_request(self, method, url, body, headers, encode_chunked):
        # Honor explicitly requested Host: and Accept-Encoding: headers.
        header_names = frozenset(k.lower() for k in headers)
        skips = {}
        if 'host' in header_names:
            skips['skip_host'] = 1
        if 'accept-encoding' in header_names:
            skips['skip_accept_encoding'] = 1

        self.putrequest(method, url, **skips)

        # chunked encoding will happen if HTTP/1.1 is used and either
        # the caller passes encode_chunked=True or the following
        # conditions hold:
        # 1. content-length has not been explicitly set
        # 2. the body is a file or iterable, but not a str or bytes-like
        # 3. Transfer-Encoding has NOT been explicitly set by the caller

        if 'content-length' not in header_names:
            # only chunk body if not explicitly set for backwards
            # compatibility, assuming the client code is already handling the
            # chunking
            if 'transfer-encoding' not in header_names:
                # if content-length cannot be automatically determined, fall
                # back to chunked encoding
                encode_chunked = False
                content_length = self._get_content_length(body, method)
                if content_length is None:
                    if body is not None:
                        if self.debuglevel > 0:
                            print('Unable to determine size of %r' % body)
                        encode_chunked = True
                        self.putheader('Transfer-Encoding', 'chunked')
                else:
                    self.putheader('Content-Length', str(content_length))
        else:
            encode_chunked = False

        for hdr, value in headers.items():
            self.putheader(hdr, value)
        if isinstance(body, str):
            # RFC 2616 Section 3.7.1 says that text default has a
            # default charset of iso-8859-1.
            body = body.encode("latin-1")
        self.endheaders(body, encode_chunked=encode_chunked)

    def getresponse(self):

        # if a prior response has been completed, then forget about it.
        if self.__response and self.__response.isclosed():
            self.__response = None

        response = self.response_class(self.sock, method=self._method)

        try:
            try:
                response.begin()
            except ConnectionError:
                self.close()
                raise
            assert response.will_close != _UNKNOWN

            if response.will_close:
                # this effectively passes the connection to the response
                self.close()
            else:
                # remember this, so we can tell when it is complete
                self.__response = response

            return response
        except:
            response.close()
            raise


class HTTPSConnection(HTTPConnection):
    "This class allows communication via SSL."

    default_port = HTTPS_PORT

    # XXX Should key_file and cert_file be deprecated in favour of context?

    def __init__(
        self, host, port=None, key_file=None, cert_file=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT, source_address=None, *, context=None, check_hostname=None, blocksize=8192
    ):
        super(HTTPSConnection, self).__init__(host, port, timeout, source_address, blocksize=blocksize)
        if (key_file is not None or cert_file is not None or check_hostname is not None):
            import warnings
            warnings.warn("key_file, cert_file and check_hostname are " "deprecated, use a custom context instead.", DeprecationWarning, 2)
        self.key_file = key_file
        self.cert_file = cert_file
        if context is None:
            context = ssl._create_default_https_context()
            # enable PHA for TLS 1.3 connections if available
            if context.post_handshake_auth is not None:
                context.post_handshake_auth = True
        will_verify = context.verify_mode != ssl.CERT_NONE
        if check_hostname is None:
            check_hostname = context.check_hostname
        if check_hostname and not will_verify:
            raise ValueError("check_hostname needs a SSL context with " "either CERT_OPTIONAL or CERT_REQUIRED")
        if key_file or cert_file:
            context.load_cert_chain(cert_file, key_file)
            # cert and key file means the user wants to authenticate.
            # enable TLS 1.3 PHA implicitly even for custom contexts.
            if context.post_handshake_auth is not None:
                context.post_handshake_auth = True
        self._context = context
        if check_hostname is not None:
            self._context.check_hostname = check_hostname

    def connect(self):
        "Connect to a host on a given (SSL) port."

        super().connect()

        if self._tunnel_host:
            server_hostname = self._tunnel_host
        else:
            server_hostname = self.host

        self.sock = self._context.wrap_socket(self.sock, server_hostname=server_hostname)
```

