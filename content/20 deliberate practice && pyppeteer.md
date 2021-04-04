---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十周
Date: 2021-03-22
Tags: 刻意练习, pyppeteer, geetest, 数据采集
---

## Algorithm

leetcode 剑指Offer38 字符串的排列

- [原题](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/Offer38.py)

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_22/
>
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_26/

### 刻意练习

- 刻意练习是为了提高完成任务的能力，关于完成任务的技巧和技术
- 刻意练习意味着重复，为了提高对任务的一个或多个方面的掌握程度而执行任务
- 刻意练习慢慢地，一遍又一遍，直到达到你想要的精通水平
- 刻意练习为了掌握任务，而不是完成任务
- 伟大很大程度上是一个有意识的选择问题
- 获得专业知识的主要因素是花在有目的练习上的时间，天赋不是主要因素

### 不要忽略错误

- 无论错误在代码中出现的可能性有多大，都应该检查它，并处理它；如果不这样做，不是在节省时间，是在为将来储存潜在的问题
- 错误处理方式
  - 返回错误码太容易被忽略
  - errno全局错误信号容易被忽略，很难使用
  - 异常是一种更受结构化语言支持的错误信令和处理方式
- 不处理错误后果
- 代码脆弱，充满令人兴奋，难以发现的bug
- 代码安全，破解者经常利用糟糕的错误处理来入侵软件系统
- 代码结构丑陋，如果代码中有一些错误需要不断处理，那么界面可能很糟糕

## Tip

## Share(pyppeteer)

> https://github.com/pyppeteer/pyppeteer
>
> https://pyppeteer.github.io/pyppeteer/reference.html
>
> http://puppeteerjs.com/
>
> https://www.douban.com/note/193710154/

### 概念

- puppeteer是Google开发的一个Node库，它提供了一个高级API来通过DevTools协议控制Chromium，js代码调用这些API来操纵chromium浏览器，完成自动化任务。
- chromium是chrome的开源版本(实验版本)，和chorme内核相同，chromium包含新功能和实验功能，待稳定后同步到chrome新版本。
- pyppeteer是puppeteer的python版本。
- pyppeteer是基于asyncio构建的，它的所有属性和方法几乎都是coroutine (协程) 对象。

```bash
# 安装pyppeteer
pip install pyppeteer
# 安装chromium
Python\Scripts\pyppeteer-install.exe
```

### 简单demo

```python
import asyncio
from pyppeteer import launch


async def main():
    browser = await launch(headless=False)
    page = await browser.newPage()
    await page.goto('https://qianlizhixing12.github.io/')
    # 截屏
    await page.screenshot({'path': 'example.png'})
    # 执行js
    dimensions = await page.evaluate('''() => {
        return {
            width: document.documentElement.clientWidth,
            height: document.documentElement.clientHeight,
            deviceScaleFactor: window.devicePixelRatio,
        }
    }''')
    print(dimensions)
    await browser.close()


asyncio.get_event_loop().run_until_complete(main())
```

### 处理滑动验证

> 整理复用下面三篇博客
>
> https://blog.csdn.net/paololiu/article/details/52514504
>
> https://cloud.tencent.com/developer/article/1377218
>
> https://zhuanlan.zhihu.com/p/107786616

#### 逻辑步骤

1. 还原图片，包括包含滑动块和不含滑动块的背景图
2. 比较图片，比较两张图片，找出滑块通过时的位置
3. 模拟滑动，先加速接近最终位置减速滑动（三个步骤：鼠标点击滑块，移动滑块，放开鼠标）

#### 说明

- 不能百分百成功

感觉代码注释清楚，比文字有效，直接上代码

```python
import base64
import asyncio
from pyppeteer import launch
import PIL.Image as image
import PIL.ImageChops as imagechops
import requests
from io import BytesIO
import random


async def main():
    launch_args = {
        "headless": False,
        "args": [
            "--start-maximized",
            "--disable-infobars",
            "--log-level=3",
            "--enable-extensions",
            "--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36",
        ],
        "ignoreDefaultArgs": ['--enable-automation']  # 去除Chrome正受到自动测试软件的控制
    }
    browser = await launch(launch_args)
    try:
        [page] = await browser.pages()
        page.setDefaultNavigationTimeout(0)
        width, height = screen_size()
        # 最大化窗口
        await page.setViewport({"width": width, "height": height})
        await page.goto(url='https://****.html', options={'timeout': 0})
        await page.waitFor(1000)
        # 设置浏览器
        await page.setUserAgent(
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)'
            ' Chrome/74.0.3729.169 Safari/537.36')
        # 防止被识别，将webdriver设置为false
        await page.evaluate(
            '''() =>{ Object.defineProperties(navigator,{ webdriver:{ get: () => false } }) }'''
        )
        await validation(page)
    finally:
        await browser.close()


async def validation(page):
    # 点击登录，弹出验证
    elem = await page.xpath('//a[@class="js-login"]')
    await elem[0].click()
    # await page.click('a.js-login')
    await page.waitFor(1000)
    # 获取拖动起始位置
    x, y = await page.evaluate('''() => {
        let locat = document.getElementsByClassName('gt_slider_knob')[0].getBoundingClientRect();
        console.log(locat);
        return [locat.x, locat.y];
    } ''')
    print(x, y)
    # 获取验证图片
    full_img, fullbg_img, diff_img = await get_decode_image(page)
    # full_img.save('full.png')
    # fullbg_img.save('fullbg.png')
    # diff_img.save('diff.png')
    print(diff_img.getbbox())
    # 方法一
    # track_list = await get_track(diff_img.getbbox()[0])
    # 方法二
    track_list = await get_track_(diff_img.getbbox()[0])
    # 点击元素
    await page.hover('div.gt_slider_knob.gt_show')
    await page.waitFor(1000)
    await page.mouse.down()
    # 拖动元素
    x += 22
    y += 22
    for track in track_list:
        x += track
        await page.mouse.move(x, y)
    await asyncio.sleep(0.5)
    # 释放鼠标
    await page.mouse.up()
    print('ok')


def screen_size():
    """使用tkinter获取屏幕大小"""
    import tkinter
    tk = tkinter.Tk()
    width = tk.winfo_screenwidth()
    height = tk.winfo_screenheight()
    tk.quit()
    return width, height


async def get_decode_image(page):
    '''
    获取图片
    '''
    full_img_info = await page.evaluate('''() => {
        let items = document.getElementsByClassName("gt_cut_bg_slice");
        let len = items.length;
        let values = [];
        let url = items[0].style.backgroundImage;
        for (let i = 0; i< len; i++) {
            values.push({'x':parseInt(items[i].style.backgroundPositionX.replace('px', ''), 10),'y':parseInt(items[i].style.backgroundPositionY.replace('px', ''), 10)});
        }
        url = url.substring(5, url.length - 2);
        return {'url':url, 'postion':values};
    }''')
    #重新合并图片
    full_img = await get_merge_image(full_img_info)

    fullbg_img_info = await page.evaluate('''() => {
        let items = document.getElementsByClassName("gt_cut_fullbg_slice");
        let len = items.length;
        let values = [];
        let url = items[0].style.backgroundImage;
        for (let i = 0; i< len; i++) {
            values.push({'x':parseInt(items[i].style.backgroundPositionX.replace('px', ''), 10),'y':parseInt(items[i].style.backgroundPositionY.replace('px', ''), 10)});
        }
        url = url.substring(5, url.length - 2);
        return {'url':url, 'postion':values};
    }''')
    #重新合并图片
    fullbg_img = await get_merge_image(fullbg_img_info)
    #获取差异
    diff_img = await get_diff(full_img, fullbg_img)
    # x = await get_diff_location(full_img, fullbg_img)
    # print(x)
    return full_img, fullbg_img, diff_img


async def get_merge_image(img_info):
    '''
    根据位置对图片进行合并还原
    '''
    im = image.open(BytesIO(requests.get(img_info['url']).content))

    new_im = image.new('RGB', (260, 116))

    im_list_upper = []
    im_list_down = []

    for postion in img_info['postion']:
        if postion['y'] == -58:
            im_list_upper.append(
                im.crop((abs(postion['x']), 58, abs(postion['x']) + 10, 166)))
        if postion['y'] == 0:
            im_list_down.append(
                im.crop((abs(postion['x']), 0, abs(postion['x']) + 10, 58)))

    new_im = image.new('RGB', (260, 116))

    x_offset = 0
    for im in im_list_upper:
        new_im.paste(im, (x_offset, 0))
        x_offset += im.size[0]

    x_offset = 0
    for im in im_list_down:
        new_im.paste(im, (x_offset, 58))
        x_offset += im.size[0]

    return new_im


async def get_diff(img1, img2):
    return imagechops.difference(img1, img2)


async def get_diff_location(image1, image2):
    '''
    计算缺口的位置，difference太精确，用绝对值相差50比较
    '''

    def is_similar(image1, image2, x, y):
        '''
        对比RGB值
        '''
        pass

        pixel1 = image1.getpixel((x, y))
        pixel2 = image2.getpixel((x, y))

        for i in range(0, 3):
            if abs(pixel1[i] - pixel2[i]) >= 50:
                return False

        return True

    i = 0
    for i in range(0, 260):
        for j in range(0, 116):
            if is_similar(image1, image2, i, j) == False:
                return i


async def get_track(distance):
    '''
    根据缺口的位置模拟x轴移动的轨迹，完全随机
    '''
    pass

    list = []

    # 间隔通过随机范围函数来获得
    x = random.randint(1, 3)

    while distance - x >= 5:
        list.append(x)

        distance = distance - x
        x = random.randint(1, 3)

    for _ in range(distance):
        list.append(1)

    return list


async def get_track_(distance):
    """
    根据偏移量获取移动轨迹，模仿匀加速后匀减速
    """
    # 移动轨迹
    track = []
    # 当前位移
    current = 0
    # 减速阈值
    mid = distance * 4 // 5
    # 计算间隔
    t = 0.2
    # 初速度
    v = 0

    while current < distance:
        if current < mid:
            # 加速度为正2
            a = 2
        else:
            # 加速度为负3
            a = -3
        # 初速度v0
        v0 = v
        # 当前速度v = v0 + at
        v = v0 + a * t
        # 移动距离x = v0t + 1/2 * a * t^2
        move = v0 * t + 1 / 2 * a * t * t
        # 当前位移
        current += move
        # 加入轨迹
        track.append(round(move))
    return track


asyncio.get_event_loop().run_until_complete(main())
```

