---
Category: ARTS
Author: qianlizhixing
Title: ARTS第七周
Date: 2020-06-01
Tags: pull request
---

## Algorithm

leetcode 589 N叉树的前序遍历

- [原题](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/589.py)

leetcode 167 有序数组二数之和 左右指针解法

- [原题](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/167.c)

## Review

> http://blog.cleancoder.com/uncle-bob/2011/01/19/individuals-and-interactions.html

开发太关注编程本身，是否会忽略用户？

我们不仅重视客户的协作，而且重视富有成效的伙伴关系。

我们希望能够以专业和技能完成我们的工作，并与我们的客户合作，以实现最好的可能的结果。

我们沉浸在“客户沟通的关键作用”中，以至于我们将这种作用放大为合作关系。

软件工艺运动并没有夸大技术的作用;相反，它试图重新获得敏捷运动失去的平衡。

当你看到一些程序员谈论代码撤退、公审、katas、TDD或其他一些深入的技术话题时，恭喜他们在实践中付出了应有的努力。

当您看到敏捷教练谈论看板、精益、迭代长度或故事点时，请祝贺他们对规程的奉献。

## Tip

树遍历代码模板

> https://github.com/qianlizhixing12/leetcode/blob/master/doc/tree.md

## Share

> https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests

### GitHub协作流

1. 在个人仓库以源仓库为基础创建一个分支
2. 在个人仓库该分支上修改源码，提交个人仓库
3. 从个人仓库该分支向源仓库发送pull request请求，说明更改内容，发起讨论
4. 根据讨论内容，你的pull request请求通过或被拒绝，如果通过，会自动合并到源仓库
5. 清理个人仓该分支

### 适用pull request的两种开发模型

1. fork和pull模型，任何人都可以fork存在的源仓库到个人仓库，将更改push到fork的个人仓库；创建pull request请求，建议将fork的个人仓库的更改同步到上游源仓库上
2. 共享仓库模型，协作者被授予对单个共享存储库的推访问权，并且在需要进行更改时创建主题分支；发送pull request请求，在变更合并到主开发分支之前，开始对一组变更的代码审查和一般性讨论

### 在GitHub讨论

1. 通过创建或订阅issues，会得到讨论和评论

### fork

1. fork创建源仓库的副本，允许在不影响源仓库的情况下进行更改
2. 通过pull请求从源仓库获取更新或将更改提交到源仓库