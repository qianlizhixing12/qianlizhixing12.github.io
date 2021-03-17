---
Category: ARTS
Author: qianlizhixing
Title: ARTS第五周
Date: 2020-05-18
---

## Algorithm

leetcode 84 柱状图中最大的矩形

- [原题](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/84.best.c)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_03/
>

- 用户不像程序员那样思考，他们使用电脑的时间要少得多，他们既不知道也不在乎计算机是如何工作的。他们不能使用程序员所熟悉的解决问题的技术，它们不能识别程序员使用的模式和线索
- 观察用户怎么工作，怎么使用软件，提问工作流程相关问题，避免过于具体的任务，偏于软件逻辑的问题；不要打断和帮助别人，问自己 “他为什么要那样做”，“她为什么不那样做”
- 用户的核心工作是类似的，试图以同样的顺序完成任务，在同样的地方犯同样的错误；围绕核心进行设计，“如果用户想要……怎么办”，导致了复杂的特性和用户需求的混乱
- 用户陷入困境时，他们会缩小关注范围，帮助文本对于糟糕的用户界面设计来说是一个糟糕的解决方案；有说明或帮助文本，请确保将其放在问题区域旁边，有时工具提示比帮助菜单更有用
- Users tend to muddle through. They'll find a way that works and stick with it no matter how convoluted. It's better to provide one really obvious way of doing things than two or three shortcuts. You'll also find that there's a gap between what users say they want and what they actually do. That's worrying as the normal way of gathering user requirements is to ask them. It's why the best way to capture requirements is to watch users. Spending an hour watching users is more informative than spending a day guessing what they want.(原文太棒了)用户会探索自己的使用模式，他们会找到一种可行的方法，不管它有多复杂，他们都会坚持下去，提供一种显而易见的做事方式比提供两三个捷径要好。用户说他们想要什么和他们实际做什么之间存在差距，这就是为什么获取需求的最佳方法是观察用户，花一个小时观察用户比花一天时间猜测他们想要什么信息更有用

## Tip

算法题的缓存(可以是栈哈希表等)，有时存储数组索引，能简化处理

存储数组索引也等于存储数组索引对应值，比单纯存储数组最大最小值好

存储数组索引的栈，可以保持栈元素的单调性，且可以优雅的计算距离，leetcode 84

## Share

> Beauty of style and harmony and grace and good rhythm depends on simplicity. — Plato
>
> 风格之美、和谐之美、优雅之美、韵律之美，在于简洁。——柏拉图

##### 追求代码品质

- 可读性
- 可维护性
- 开发速度
- 难以衡量的质量

这些品质的促成因素是简单

##### 代码之美

- 这是比较主观的问题
- 答案依赖个人经历背景，经受艺术教育和科学教育，对美的思考方式不同，但简单些是双方争论的基础
- 引起共鸣，公认美的代码用共性：简单；无论整体多么复杂，各个部分都保持简单；单一职责，简单，集中的对象及方法
- 漂亮的代码就是简单的代码，单独的部分通过简单的职责和其他部分保持简单关系，这就是保持可维护性，使用干净，简单，可测试，高开发速度的方法。

##### 美是在简单中产生和发现的