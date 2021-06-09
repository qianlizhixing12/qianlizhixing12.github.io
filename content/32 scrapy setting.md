---
Category: ARTS
Author: qianlizhixing
Title: ARTS第三十二周
Date: 2021-06-14
Tags: python库, 资源管理, os, sys, linux coding style, scrapy
---

## Algorithm

leetcode 494 目标和

- [原题](https://leetcode-cn.com/problems/target-sum/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/494.py)
- 动态规划转换为背包问题

## Review

> https://www.kernel.org/doc/html/latest/process/coding-style.html

#### 缩进

- 制表符是8个字符，缩进也是8个字符
- 不要把多个语句放在一行里，不要用逗号来避免使用大括号
- 除了注释、文档和Kconfig之外，空格从不用于缩进
- 不要在行尾留下空白

#### 断开长行长字符串

- 单行长度的首选限制是80列
- 超过80列的语句应该被分成合理的块，除非超过80列会显著增加可读性
- 一个非常常用的样式是将断开行对齐到函数开括号
- 这些规则适用于带有长参数列表的函数头

#### 括号和空格

- 首选的方式是把左括号放在行末，把右括号放在行首，独占一行，适用于所有非函数语句块(if, switch, for, while, do)
- 函数在下一行行首开始左括号，独占一行
- 不要在单个语句使用大括号
- 在保留词后面加一个空格
- 不要在圆括号周围添加空格
- 当声明指针数据或返回指针类型的函数时，*的首选用法是紧邻数据名或函数名，而不是紧邻类型名
- 在大多数二元和三元运算符周围(每边)使用一个空格
- 一元运算符后不能有空格
- 不要在行尾留下空格

#### 命名

- 全局变量需要有描述性的名称，就像全局函数一样。例如一个计算活动用户数量的函数，应该调用count_active_users()，不应该调用cntusr()
- 将函数的类型编码到名称中是愚蠢的
- 局部变量的名称应该简短且切中要害

#### 函数

- 函数应该简短亲切，只做一件事并且做得很好
- 函数局部变量的数量不应该超过5-10个
- 在函数原型中，包含参数名及其数据类型

#### 注释

- 将注释放在函数的开头，告诉人们它做什么，以及它为什么这样做

## Tip

- os模块负责程序与操作系统的交互，提供了访问操作系统底层的接口
- sys模块负责程序与python解释器的交互，提供了一系列的函数和变量，用于操控python的运行时环境

#### 运行信息

- 系统平台sys.platform
- 解释器版本信息sys.version sys.version_info
- 当前Unicode实现所使用的默认字符串编码名称sys.getdefaultencoding()
- 命令行传递参数sys.argv
- sys.getrefcount()对象引用计数，返回的计数通常比预期的多一，因为它包括了作为getrefcount()参数的这一次临时引用
- sys.modules将模块名称映射到已加载的模块。可以操作该字典来强制重新加载模块，或是实现其他技巧。但是，替换的字典不一定会按预期工作，并且从字典中删除必要的项目可能会导致Python崩溃
- sys.path字符串组成的列表，用于指定模块的搜索路径。初始化自环境变量PYTHONPATH，再加上一条与安装有关的默认路径
- 运行环境变量os.environ

#### 进程管理

- 模拟传统的UNIX函数发信号给进程(UNIX平台上有效)os.kill(os.getpid(), signal.SIGILL)
- os.fork()无法在Windows系统中使用，父进程中os.fork()返回子进程pid，子进程中os.fork()返回0
- os.system通过fork一个子进程,然后该子进程执行命令，阻塞等待子进程结束，返回命令执行结果的返回值
- os.popen创建一个管道，通过fork一个子进程,然后该子进程执行命令

#### 文件操作

- 拼接路径os.path.join(dir, path)

- 新建文件夹os.makedirs(os.path.dirname(file_full_path))
- 删除文件夹os.rmdir(os.path.dirname(file_full_path))
- 判断文件夹存在os.path.exists(os.path.dirname(file_full_path))
- 目录文件列表os.listdir(os.path.dirname(file_full_path))
- 判断资源类型(是否文件)os.path.isfile(file_full_path)
- 删除文件os.remove(file_full_path)
- 移动(重命名)os.rename(src_file_full_path, des_file_full_path)

#### ini文件解析

- configparser模块

```python
import configparser


def doini():
    filename = "test.ini"

    # 加载ini
    iniFile = configparser.ConfigParser()
    iniFile.read(filename)

    # 添加或更新节点
    section, key, value = "request", "timeout", "30"
    if not iniFile.has_section(section):
        iniFile.add_section(section)
    iniFile.set(section, key, value)

    # 删除节点
    section, key, value = "request", "retry", "5"
    if iniFile.has_section(section):
        if iniFile.has_option(section, key):
            iniFile.remove_option(section, key)
    if len(iniFile.options(section)) == 0:
        iniFile.remove_section(section)

    # 回写ini
    iniFile.write(open(filename, "w"))


if __name__ == "__main__":
    doini()
```

#### 注册表操作

- winreg模块

```python
import winreg


def doreg(regList):
    # root
    root, path = winreg.HKEY_CURRENT_USER, r"ttt\www"
    reg = winreg.CreateKey(root, path)

    try:
        # 添加或更新
        name, value = "host", "sss"
        winreg.SetValueEx(reg, name, 0, winreg.REG_SZ, value)

        # 删除
        name = "port"
        for i in range(0, winreg.QueryInfoKey(reg)[1]):
            if (winreg.EnumValue(reg, i)[0] == name):
                winreg.DeleteValue(reg, name)
                break
    except Exception as e:
        raise e
    finally:
        winreg.CloseKey(reg)
```

## Share(scrapy setting)

> https://docs.scrapy.org/en/latest/intro/tutorial.html

#### 项目结构

![scrapy_files](.\img\scrapy_files.png)

- scrapy.cfg部署配置文件
- settings.py项目设置文件
- items.py采集项定义文件
- pipelines.py采集管道文件
- middlewares.py中间件文件
- spiders/爬虫器目录

#### 源码结构

- 随着scrapy系列文章，会逐渐补充
- scrapy.cmdline  命令入口
- scrapy.commands 命令目录
- scrapy.templates 创建新项目或新爬取器模板
- scrapy.utils工具函数
  - scrapy.utils.project 项目设置，数据处理函数
  - scrapy.utils.conf 部署配置处理函数
  - scrapy.utils.versions 版本处理函数
  - scrapy.utils.template 模板变量字符串替换处理
- scrapy.settings运行设置
  - \_\_init\_\_ 类定义
  - default_settings默认设置

#### 命令

##### 全局命令

- startproject

- genspider

- settings

- runspider

- shell

- fetch

- view

- version


##### Project命令

- crawl

- check

- list

- edit

- parse

- bench

命令的入口都为scrapy.cmdline.execute，根据具体命令执行scrapy.commands下具体命令

#### 配置读取

- scrapy第一步先找部署配置文件，因为部署配置里面包含了运行设置项的路径配置和部署配置，如果本地运行execute时指定了settings运行配置，程序忽略部署配置文件和运行配置项的解析

- get_project_settings逻辑

  - 当环境变量没有SCRAPY_SETTINGS_MODULE，调用init_env从部署配置文件读取settings填充SCRAPY_SETTINGS_MODULE，init_env还会把部署配置文件的目录加入sys.path
  - 部署配置文件ini格式，可以在/etc/scrapy.cfg，c:\scrapy\scrapy.cfg，home/user/.config/scrapy.cfg，home/user/.scrapy.cfg和当前目录及父目录递层向上查找

- Settings类初始化加载default_settings默认配置(优先级default)，配置项SettingsAttribute的priority有5个优先级，高优先级覆盖低优先级

  ```python
  class SettingsAttribute:
  
      ...
  
      def set(self, value, priority):
          """Sets value if priority is higher or equal than current priority."""
          if priority >= self.priority:
              if isinstance(self.value, BaseSettings):
                  value = BaseSettings(value, priority=priority)
              self.value = value
              self.priority = priority
  ```

#### 执行命令

- 动态从scrapy.commands目录下加载所有模块，并从模块中找到继承自ScrapyCommand的类，文件名为命令名(exectue参数argv[1:]开始第一个不以-开始的值)，创建命令名到执行实例的映射
- 使用optparse解析命令参数
- 调用cmd.add_options将每个命令需要的参数定义到optparse
- 调用cmd.process_options将optparse加入或更新settings(优先级为cmdline)
- 调用cmd.run运行命令

```python
# 显示版本命令
from scrapy.cmdline import execute
import sys
import os
# 获取当前脚本路径
dirpath = os.path.dirname(os.path.abspath(__file__))
#切换工作目录
os.chdir(dirpath)
execute(['scrapy', 'version', '-v'])

# 输出
# Scrapy       : 2.5.0
# lxml         : 4.6.3.0
# libxml2      : 2.9.5
# cssselect    : 1.1.0
# parsel       : 1.6.0
# w3lib        : 1.22.0
# Twisted      : 21.2.0
# Python       : 3.8.10 (tags/v3.8.10:3d8993a, May  3 2021, 11:48:03) [MSC v.1928 64 bit (AMD64)]
# pyOpenSSL    : 20.0.1 (OpenSSL 1.1.1k  25 Mar 2021)
# cryptography : 3.4.7
# Platform     : Windows-10-10.0.19043-SP0
```

