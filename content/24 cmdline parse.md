---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十四周
Date: 2021-04-19
Tags: 命令行解析, python库, argparse, getopt
---

## Algorithm

leetcode 179 最大数

- [原题](https://leetcode-cn.com/problems/largest-number/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/179.py)
- 字符串比较

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_57/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_60/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_61/

### 消息传递在并行系统中带来更好的可伸缩性

- 几乎所有并发问题都与共享可变内存的使用有关:竞争条件、死锁、活锁等。答案似乎很明显:要么放弃并发性，要么回避共享内存
- 可以使用进程和消息传递，而不是使用线程和共享内存作为编程模型

### 测试员是你的朋友

- 因为测试人员报告琐碎的问题，开发看起来很糟糕
- 因为测试人员的QC，客户没有让开发修复那些“小东西”所困扰而感到兴奋时，开发看起来很棒

### 部署二进制文件

- 构建或部署重写了部分代码，为每个目标环境生成定制的二进制文件，这是糟糕的
- 为生产环境从头开始重建，意味着无法证明生产环境中的版本是经过测试的版本
- 规则很简单:构建一个单独的二进制文件，在发行渠道的所有阶段识别并提升它。在每个目标环境中保留特定环境的细节。例如，将目标环境配置保存在组件容器、已知文件或路径中
- 设计上将应用程序的核心功能和特定于平台的功能分开
- 保持环境信息的版本化，环境信息应该与代码分开进行版本管理，因为它们将以不同的速率和不同的原因进行更改。一些团队使用分布式版本控制系统(如bazaar和git)来实现这一点，可以更容易地将生产环境中的更改(不可避免地会发生)推回存储库

## Tip

> https://www.w3school.com.cn/jsref/dom_obj_all.asp

- 刚好需要在某博客平台发布个文章，有图片，markdown格式

- Typora打草稿，图片稍大，生成img标签，在浏览器(chrome)中缩小25%，该博客平台未生效

  ```html
  <img src=".\img\aaa.png" style="zoom: 25%;" />
  ```

- 使用width属性，浏览器和该博客平台都生效

  ```html
  <img src=".\img\aaa.png" id="3" width="25%" />
  <--或者!-->
  <img src=".\img\aaa.png" id="4" style="width: 25%;" />   
  ```

- 使用transform属性，浏览器和该博客平台都生效

  ```html
  <--success!-->
  <img src=".\img\aaa.png" id="2" style="transform: scale(0.4, 0.4);" />
  <--fail!-->
  <img src=".\img\aaa.png" id="1" transform="scale(0.4, 0.4);" />
  ```

- transform为什么不能width使用，只能放在style中，继续测试

  ```javascript
  document.getElementById('1').style
  //输出 CSSStyleDeclaration {alignContent: "", alignItems: "", alignSelf: "", alignmentBaseline: "", all: "", …}
  document.getElementById('1').attributes
  //输出 NamedNodeMap {0: src, 1: id, 2: transform, src: src, id: id, transform: transform, length: 3}
  ```

- 总结，element.attributes，element.getAttribute()，element.getAttributeNode()都是HtmlDom节点的属性相关操作，element.attributes.style也是一个属性，属性属于节点；element.style样式，不是所有属性都能影响样式，哈哈自己这样推理

## Share(命令行解析)

### sys.argv变量

- sys.argv类型List[str]，就是普通的列表，sys.argv[0]运行的脚本
- 因为每个参数都是str，不能直接解析参数(例如--time=0)
- 每个参数用空白字符分割，参数包含空白符的用单引号或双引号分开

```python
# 测试脚本
import sys

if __name__ == "__main__":
    print(sys.argv)
    
#命令 python test.py --timeout=0  on=true  -p 80  'wait false' "save out.xml"
#输出 ['test.py', '--timeout=0', 'on=true', '-p', '80', 'wait false', 'save out.xml']
```

### argparse模块

- 基于optparse(不推荐使用)
- 帮助信息，附加信息更全面，推荐使用
- 使用ArgumentParser类解析参数，支持-p 80或-p=80格式，即用=号或空白符分割参数名和参数值
- 构造ArgumentParser对象后调用add_argument定义命令行参数，--完整参数名，-简写参数名，default默认值
- 调用parse_args返回Namespace类对象，用属性访问(.)获取参数

```python
# 测试脚本
from argparse import ArgumentParser
from multiprocessing import cpu_count


def parse_option():
    # 构造ArgumentParser
    parser = ArgumentParser(prog='argparse test',
                            usage='info',
                            description='get option from cmdline',
                            prefix_chars='-')
    # print(parser.prog, parser.usage, parser.description)
    # 定义命令行参数
    parser.add_argument('-n',
                        '--num',
                        type=int,
                        dest='number',
                        default=cpu_count())
    parser.add_argument('-f', '--fun', default='ping')
    parser.add_argument('-m', '--mode', default='proc')
    parser.add_argument('-p', '--port')
    parser.add_argument('-w', '--save')
    return parser.parse_args()


if __name__ == "__main__":
    options = parse_option()
    print(options)
    print(options.number, options.fun)

#命令 python test_argparse.py -n 300 -f tcp --mode thread -p 80
#输出 Namespace(fun='tcp', mode='thread', number=300, port='80', save=None)
#    300 tcp thread 80 None

#命令 python test_argparse.py -n 300 -f tcp --mode thread -p=80
#输出 Namespace(fun='tcp', mode='thread', number=300, port='80', save=None)
#    300 tcp thread 80 None

#命令 python test_argparse.py -n 300 -f tcp --mode thread -p 80 -timeout 120
#输出 usage: info
#    argparse test: error: unrecognized arguments: -timeout 120
```

### getopt模块

- 使用形式接近c语言getopt函数
- 使用getopt.getopt(args=args, shortopts, longopts=[])初始化
- args要解析的参数数组，通常为sys.argv[1:]
- shortopts短参数(-t)，有冒号(:)选项必须有值(-t 100或-t100都可，-t=100解析为=100)，没有冒号选项不能有值
- longopts长参数(--timeout)，有等号(=)选项必须有值(--t 100或--t100及--t=100都解析100)，没有等号选项不能有值
- 返回一个两元素元组，第一个元素(option, value)格式的列表，第二元素是没匹配的内容列表
- 遇到未匹配的立即返回，shortopts必须在longopts前，没有匹配的args
- 返回值没有属性访问(.)或字典访问([])方法，后期处理没有argparse方便

```python
import sys
import getopt


def parse_option(args):
    # 解析命令行
    option, argv = getopt.getopt(
        args=args,
        shortopts='n:t:f',
        longopts=['num', 'timeout', 'fun', 'mode', 'port=', 'save'])
    print(option)
    print(argv)


if __name__ == "__main__":
    parse_option(sys.argv[1:])
    
#命令 python .\test_getopt.py -n 10 -t 120 -f 200 --save 500
#输出 [('-n', '10'), ('-t', '120'), ('-f', '')]
#    ['200', '--save', '500']

#命令 python .\test_getopt.py -n 10 -t 120 -f --save --port 300
#输出 [('-n', '10'), ('-t', '120'), ('-f', ''), ('--save', ''), ('--port', '300')]
#    []
```

### 其他模块

- click
- fire
- 个人认为这两个库设计上，不是单纯为了解析命令行参数目的