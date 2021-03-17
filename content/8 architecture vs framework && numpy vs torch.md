---
Category: ARTS
Author: qianlizhixing
Title: ARTS第八周
Date: 2020-06-08
Tags: numpy, torch
---

## Algorithm

leetcode 589 N叉树的前序遍历

- [原题](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/589.py)

leetcode 590 N叉树的后序遍历

- [原题](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/590.py)

leetcode 429 N叉树的层序遍历

- [原题](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/429.py)

leetcode 144 二叉树的前序遍历

- [原题](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/144.py)

leetcode 94 二叉树的中序遍历

- [原题](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/94.py)

leetcode 102 二叉树的层序遍历

- [原题](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/102.py)

## Review

> http://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html

- 就像住宅或图书馆的计划需要描述这些建筑的用例一样，软件应用程序的架构也需要描述应用程序的用例。

- 架构不应该与框架有关，架构不应该由框架提供；框架是要使用的工具，而不是要遵守的架构；如果你的架构基于框架，那么它就不能基于你的用例。

- 好的架构以用例为中心的原因是，架构师可以安全地描述支持那些用例的结构，而不需要使用框架、工具和环境。

- 一个好的软件架构允许关于框架、数据库、web服务器和其他环境问题和工具的决策被推迟或延迟。好的体系结构使得在项目的后期才有必要决定使用Rails、Spring、Hibernate、Tomcat或MySql。一个好的架构也可以使您很容易地改变对这些决策的想法。好的体系结构强调用例，并将它们与外围关注点解耦。

- Web是一种交付机制，不应该主导您的系统结构。

- 让框架做所有事情的立场。这不是你想要的立场。

- 防止框架接管架构。

- 体系结构应该告诉读者关于系统的信息，而不是在系统中使用的框架。


## Tip

leetcode 429 N叉树的层序遍历 递归写法

```python
# Definition for a Node.
class Node:

  def __init__(self, val=None, children=None):
    self.val = val
    self.children = children


#迭代写法
class Solution:

  def levelOrder(self, root: 'Node') -> List[List[int]]:
    if not root:
      return []

    ans, nodes = [], [root]
    while nodes:
      cur = []
      visit = []
      for node in nodes:
        if node == None:
          continue
        visit.append(node.val)
        cur.extend(node.children)
      nodes = cur
      ans.append(visit)
    return ans


#递归写法
class Solution:

  def levelOrder(self, root: 'Node') -> List[List[int]]:

    def help(node, ans, level):
      if len(ans) == level:
        ans.append([])
      ans[level].append(node.val)

      for children in node.children:
        help(children, ans, level + 1)

    ans = []
    if root:
      help(root, ans, 0)
    return ans
```

- 常规递归写法：一般会把每层所有的节点作为集合为数组，作为递归参数调用，对本层处理时将本层所有子节点集合为数组；递归此过程。

- 评论区所示：不收集每层所有节点，每个元素及深度作为递归参数调用，这样每个节点在最终结果数组的索引明确了。


## Share

> https://www.numpy.org.cn/about/

NumPy是使用Python进行科学计算的基础包，包含一个强大的N维数组对象，有用的线性代数，傅里叶变换和随机数功能。广泛应用于科学计算和深度学习。

NumPy最基础结构是封装的N维数组，基本操作：

- 可用numpy.array，numpy.zeros，numpy.ones，numpy.random.random构造一维或n维数组
- 可以像普通数组一样通过索引赋值和访问
- 可以通过arr[:col]快速访问某列元素
- 可以逐元素执行加减乘除(+-*/)和矩阵乘法(dot)

```python
import numpy

arr = numpy.array([1, 2, 3])
print(arr)
print(arr.shape)

arr1 = numpy.zeros((5))
print(arr1)

arr2 = numpy.ones((3))
print(arr2)

arr3 = numpy.random.random((5))
print(arr3)

arr4 = numpy.zeros((2, 4))
print(arr4)

a = numpy.array([[1.0, 2.0], [3.0, 4.0]])
b = numpy.array([[5.0, 6.0], [7.0, 8.0]])
print(a + b)
print(a - b)
print(a * b)
print(a / b)
print(a.dot(b))
```

Torch最基础结构是封装好的tensor(张量)，类比操作，与numpy转换：

```python
import torch
import numpy

data = [1, 3, -1, -3]
tensor = torch.FloatTensor(data)
print('numpy abs', numpy.abs(data))
print('torch abs', torch.abs(tensor))
print('numpy sin', numpy.sin(data))
print('torch sin', torch.sin(tensor))
print('numpy mean', numpy.mean(data))
print('torch mean', torch.mean(tensor))

data = [[1, 2], [3, 4]]
tensor = torch.FloatTensor(data)
print('numpy multiplication ', numpy.matmul(data, data))
print('torch multiplication ', torch.mm(tensor, tensor))
print('numpy multiplication ', numpy.array(data).dot(data))
# print('torch multiplication ', tensor.dot(tensor))

np_data = numpy.arange(6).reshape((2, 3))
torch_data = torch.from_numpy(np_data)
tensor2array = torch_data.numpy()
print('numpy array:', np_data)
print('torch tensor:', torch_data)
print('tensor to array:', tensor2array)
```

