---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十二周
Date: 2021-04-05
Tags: fontTools, woff, 字体加密, 数据采集
---

## Algorithm

leetcode 面试题17.21 直方图的水量

- [原题](https://leetcode-cn.com/problems/volume-of-histogram-lcci/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/17.21.py)
- 单调栈
- 双指针

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_41/
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_42/
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_43/

### 进程间通信影响应用程序响应时间

- 数据结构和算法，在某些情况下可能会对性能产生影响，但在现代多层企业应用程序中不太可能主导性能
- 远程进程间通信的数量通常对性能占主导地位，每个远程进程间通信都会对总体响应时间造成一些不可忽略的延迟，这些影响会累积起来，特别是当它们依次发生时
- 应用节俭原则，优化流程之间的接口，以便以最小的交互量交换手边正确的数据
- 在可能的情况下并行处理进程间通信，这样总体响应时间主要由延迟时间最长的IPC驱动
- 缓存以前的ipc的结果，这样将来的ipc就可以通过使用本地缓存来避免
- 无论是通过缓存、并行或其他技术减少远程ipc响应时间，都比改变数据结构选择或调整排序算法带来的回报要大得多

### 保持构建整洁

- 在成百上千个不关心的编译器警告中，找到真正关注的编译器警告困难得多
- 对构建警告使用零容忍策略，即使知道警告内容永远不会在生产环境中出现
- 某些构建警告真的不关心，也不重要(例如版本升级，老的代码在新版本)，改变警告政策
- 构建警告是有用的，不要等到大扫除后再做，马上修复警告源、抑制此警告或修复工具的警告策略
- 保持构建的整洁不仅仅是避免编译错误或测试失败，警告也是代码整洁的重要和关键部分

### 使用命令行工具

- IDE有很多值得喜欢的地方，不仅易于使用，还使程序员不必考虑构建过程中涉及的大量小细节
- 当一个工具易于使用时，是因为这个工具为你做决定，并在幕后自动做很多事情
- 编写make文件帮助理解构建可执行文件的所有步骤(编译、汇编、链接等)
- 使用许多命令行选项进行试验也是一种宝贵的教育经验
- 一个设计良好的IDE只是一组命令行工具的图形前端
- 有些任务使用命令行工具比使用IDE更容易或更有效地执行
- 命令行工具本质上支持脚本，它允许任务的自动化

## Tip

### unicode_escape

- 用unicode_escape转义字符串普通字符相互转化

```python
def convert_unicode_escape():
    '''
    转义字符串普通字符相互转化
    '''
    #转义字符串
    s1 = u'\u597d\u4eba'
    #普通字符串
    s2 = r'\u597d\u4eba'
    print(s1, s2)
    #out ----  好人 \u597d\u4eba
    print(
        s1.encode('unicode_escape').decode('utf8'),
        s2.encode('utf8').decode('unicode_escape'))
    #out ----  \u597d\u4eba 好人


if __name__ == "__main__":
    convert_unicode_escape()
```

## Share

### 字体加密

> 整理了链接博客
>
> https://zhuanlan.zhihu.com/p/33112359

#### 逻辑步骤

1. 人工得到一个基本映射(如\ue456-->3)，作为对照
2. 实时获取字体和隐藏的源码
3. 根据对比基本映射，得到实时映射
4. 根据实时映射和源码，得到最终结果

```python
import requests
from fake_useragent import UserAgent
from lxml import etree
from fontTools.ttLib import TTFont
import re
from io import BytesIO


def main():
    info, woff_href = get_info()
    woff = get_woff(woff_href)
    get_word(info, woff)


def get_info():
    '''
    获取信息
    获取字体链接
    '''
    headers = {'User-Agent': UserAgent(verify_ssl=False)['random']}
    response = requests.get(r'https://***.com/',
                            headers=headers,
                            verify=False)
    # with open(r'test_font.txt', mode='wb') as f:
    #     f.write(response.content)
    info = []
    selector = etree.HTML(response.text)
    for item in selector.xpath('//ul[contains(@class, "ranking-box")]/li'):
        name = item.xpath(
            './/span[@class="ranking-top-moive-name" or @class="ranking-movie-name"]/text()'
        )[0]
        num = item.xpath('.//span[@class="stonefont"]/text()')[0]
        # print(name, num)
        info.append({'name': name, 'num': num})
    # "url('//4858115f54fc92296.woff')"
    woff_info = etree.HTML(response.text).xpath('/html/head/style/text()')[0]
    woff_href = re.search(r"url\('(\S*\.woff)'\)", woff_info).groups()[0]
    return info, woff_href


def get_woff(woff_href):
    '''
    下载字体
    '''
    headers = {'User-Agent': UserAgent(verify_ssl=False)['random']}
    response = requests.get("https:" + woff_href, headers=headers, verify=False)
    # with open(r'test_font.woff', mode='wb') as f:
    #     f.write(response.content)
    woff = TTFont(BytesIO(response.content))
    # woff.saveXML('test_font.xml')
    return woff


def get_word(info, woff):
    # test_font_base.otf是某一次访问获取的字体文件，然后人工识别内容，作为与后面获取字体的比对标本，从而让电脑自动获得后面获取字体的实际内容
    #FontCreator 得到一个基本映射
    base = TTFont('test_font_base.woff')
    base_dic = {
        'uniF19B': '2',
        'uniF4EF': '6',
        'uniE137': '8',
        'uniE343': '1',
        'uniE8CD': '5',
        'uniE7A1': '9',
        'uniF848': '3',
        'uniF489': '0',
        'uniE5E2': '4',
        'uniF88A': '7',
    }
    unis = woff.getGlyphOrder()[2:]
    woff_dic = {}
    for uni in unis:
        woff_glyf = woff['glyf'][uni]
        for uni_ in base_dic:
            # 同一字体的glyf相同，不深究
            if woff_glyf == base['glyf'][uni_]:
                key = eval(r"u'\u" + uni[3:] + "'")
                woff_dic[key] = base_dic[uni_]
                break
    for item in info:
        num = item['num'].upper()
        for uni in woff_dic:
            num = num.replace(uni, woff_dic[uni])
        print(item['name'], num)
    pass


if __name__ == "__main__":
    main()
```

