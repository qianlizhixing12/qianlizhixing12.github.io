---
Category: ARTS
Author: qianlizhixing
Title: ARTS第二十一周
Date: 2021-03-29
Tags: bug报告, selenium, geetest
---

## Algorithm

leetcode 74 搜索二维矩阵

- [原题](https://leetcode-cn.com/problems/search-a-2d-matrix/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/74.py)
- 解题思路，数组元素具有单调性时，二分查找

## Review

> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_37/
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_38/
> https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_39/
>

### 努力工作是没有回报的

- 应该考虑减少工作量以提高效率，完成更多的工作
- 作为一名专业的程序员，你必须让自己在自己的专业领域保持更新
- 准备和教育是职业的重要组成部分
- 作为一名专业程序员，应该知道一周工作60个小时是不明智的。表现得像个专业人士:准备、影响、观察、反思和改变

### 如何使用Bug追踪器

- 一份好的bug报告需要以下三点
  - 如何尽可能精确地重现错误，以及这将使错误出现的频率
  - 应该发生什么事，至少在你看来是这样。
  - 到底发生了什么，或者至少是你记录的所有信息
- 花点时间解释为什么这个bug应该被关闭，可以节省以后向经理和客户证明它的冗长时间
- 更改bug的优先级也是公开声明，仅因为微不足道，并不意味着它不会阻止其他人使用该产品

### 通过删除代码来改进代码

- 为什么一开始就会出现不必要的代码
  - 这是一个有趣的额外的东西，程序员想要编写它(写代码是因为它能增加价值，而不是因为它能让你开心)
  - 有人认为将来可能会需要它，所以觉得最好现在就编写它(如果你现在不需要它，也不要马上写)
  - 它看起来并不是一个很大的“额外”，因此实现它比回到客户那里查看是否真的需要它更容易
  - 程序员发明了既没有记录也没有讨论的额外需求来证明额外的特性，这个要求实际上是假的

## Tip

## Share(selenium)

### 处理滑动验证

上篇用pyppeteer处理了geetest滑块验证，试着selenium处理

> 整理复用下面三篇博客
>
> https://blog.csdn.net/paololiu/article/details/52514504
>
> https://cloud.tencent.com/developer/article/1377218
>
> https://zhuanlan.zhihu.com/p/107786616
>
> https://www.jianshu.com/p/3f968958af5a

#### 逻辑步骤

1. 还原图片，包括包含滑动块和不含滑动块的背景图
2. 比较图片，比较两张图片，找出滑块通过时的位置
3. 模拟滑动，先加速接近最终位置减速滑动（三个步骤：鼠标点击滑块，移动滑块，放开鼠标）

#### 说明

- 不能百分百成功

感觉代码注释清楚，比文字有效，直接上代码

```python
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.action_chains import ActionChains
import PIL.Image as image
import PIL.ImageChops as imagechops
import requests
from io import BytesIO
import base64
from time import sleep
import numpy as np
import random


def main():
    options = webdriver.ChromeOptions()
    #隐藏Chrome正在受到自动软件的控制
    # options.add_argument('--disable-infobars')已弃用
    options.add_experimental_option('useAutomationExtension', False)
    options.add_experimental_option('excludeSwitches', ['enable-automation'])
    # options.add_argument('--headless')
    with webdriver.Chrome(options=options) as bs:
        bs.maximize_window()
        bs.get('https://****')
        try:
            # 防止被识别，将webdriver设置为false
            # bs.execute_script(
            #     'Object.defineProperties(navigator,{ webdriver:{ get: () => false } })'
            # )
            fill_info(bs, '15055495@qq.com', 'kdfjksdf')
            validation(bs)
            # print_info(bs)
        finally:
            bs.quit()


def fill_info(bs, user, passwd):
    bs.find_elements_by_class_name('login-button')[0].click()
    # bs.find_element_by_xpath(
    #     '//button[contains(@class,"login-button")]').click()
    email = WebDriverWait(
        bs, 0).until(lambda d: d.find_element_by_name('mobileOrEmail'))
    email.send_keys(user)
    # bs.find_element_by_name('mobileOrEmail').send_keys(user)
    # bs.find_element_by_xpath('//input[@name="mobileOrEmail"]').send_keys(user)
    bs.find_element_by_name('password').send_keys(passwd)
    # bs.find_element_by_xpath('//input[@name="password"]').send_keys(passwd)
    bs.find_element_by_class_name('submit').click()
    # bs.find_element_by_xpath(
    #     '//button[contains(@class,"sm-button")][1]').click()
    sleep(5)


def validation(bs):
    distance = get_slide_distance(bs)
    tracks = get_slide_track(distance - 8, 3)
    move_slide(bs, tracks)


def get_slide_distance(bs):
    '''
    # 获取验证图片相对距离
    '''
    bg_info = bs.execute_script(
        'return document.getElementsByClassName("geetest_canvas_bg")[0].toDataURL("image/png");'
    )
    bg = get_image(bg_info)
    fullbg_info = bs.execute_script(
        'return document.getElementsByClassName("geetest_canvas_fullbg")[0].toDataURL("image/png");'
    )
    fullbg = get_image(fullbg_info)
    # bg.save('bg.png')
    # fullbg.save('fullbg.png')
    #获取差异
    return get_diff_location(fullbg, bg)


def get_image(img_info):
    '''
    根据base64数据还原图片
    '''
    im_base64 = img_info.split(',')[1]
    im_bytes = base64.b64decode(im_base64)
    return image.open(BytesIO(im_bytes))


def get_diff_location(image1, image2):
    '''
    计算缺口的位置，difference太精确，用绝对值相差50比较
    '''

    def is_similar(image1, image2, x, y):
        '''
        对比RGB值
        '''
        pixel1 = image1.getpixel((x, y))
        pixel2 = image2.getpixel((x, y))

        for i in range(0, 3):
            if abs(pixel1[i] - pixel2[i]) >= 50:
                return False

        return True

    i = 0
    for i in range(0, 260):
        for j in range(0, 160):
            if is_similar(image1, image2, i, j) == False:
                return i


def get_slide_track(distance, seconds):
    """
    根据轨迹离散分布生成的数学生成
    :param distance: 缺口位置
    :param seconds:  时间
    :return: 轨迹数组
    """

    def ease_out_quad(x):
        return 1 - (1 - x) * (1 - x)

    def ease_out_quart(x):
        return 1 - pow(1 - x, 4)

    def ease_out_expo(x):
        if x == 1:
            return 1
        else:
            return 1 - pow(2, -10 * x)
        distance += 20

    def sigmoid(x):
        return (1.0 / (1 + np.exp(-x)))

    # ease: 生成函数
    ease = ease_out_quart
    # # ease = ease_out_quad
    # # ease = ease_out_expo
    # ease = sigmoid
    tracks = [0]
    offsets = [0]
    for t in np.arange(0.0, seconds, 0.1):
        offset = round(ease(t / seconds) * distance)
        tracks.append(offset - offsets[-1])
        offsets.append(offset)
    return tracks


def move_slide(bs, tracks):
    slide = WebDriverWait(bs, 0).until(
        lambda bs: bs.find_element_by_class_name('geetest_slider_button'))
    # slide = bs.find_element_by_class_name('geetest_slider_button')
    # 点击元素
    ActionChains(bs).click_and_hold(slide).perform()
    # 拖动元素
    for track in tracks:
        ActionChains(bs).move_by_offset(xoffset=track, yoffset=0).perform()
    # 释放鼠标
    sleep(0.02)
    ActionChains(bs).release().perform()
    print('ok')


def print_info(bs):
    # 获取cookies
    cookies = bs.get_cookies()
    sessionStorage = bs.execute_script('return sessionStorage;')
    localStorage = bs.execute_script('return localStorage;')
    print(cookies)
    print(sessionStorage)
    print(localStorage)


if __name__ == "__main__":
    main()
```

