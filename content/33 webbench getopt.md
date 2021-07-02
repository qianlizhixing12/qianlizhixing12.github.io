---
Category: ARTS
Author: qianlizhixing
Title: ARTS第三十三周
Date: 2021-06-21
Tags: c++学习, 魔术方法, llvm code style, webbench, getopt
---

## Algorithm

leetcode 279 完全平方数

- [原题](https://leetcode-cn.com/problems/perfect-squares/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/279.py)
- 转换为动态规划目标和问题

## Review

> https://llvm.org/docs/CodingStandards.html

- 黄金法则：如果正在扩展、增强或修复已经实现的代码，使用已经使用的样式，以便源代码是统一和易于遵循

#### 注释

- 描述代码试图做什么以及为什么要做，而不是在微观层面上它是如何做的
- 每个源文件都应该有一个描述文件基本用途的注释
  - 第一行的“\*- c++ -\*”字符串告诉Emacs源文件是c++文件，而不是C文件
  - 简要的说明，定义文件发布的许可。非常清楚源代码可以在什么条件下发布，并且不应该以任何方式进行修改
  - 描述文件用途的Doxygen注释(由注释标记///而不是通常的//标识)
- 头文件的保护应该包括该头文件的所有大写路径，使用_代替路径分隔符和扩展标记
- 类定义应该有一个注释块，解释类的用途和工作方式，每个重要的类都有一个doxygen注释块
- 方法和全局函数也应该有注释，简要说明一下它的作用，并描述一下边界情况
- 通常，更喜欢c++风格的注释(//用于普通注释，///用于doxygen文档注释)。但是，在一些情况下，使用c风格的(/* */)注释是有用的
- 注释掉大块代码是不被鼓励的，真的必须这样做，使用#if 0和#endif

#### #include

- 头文件注释之后，列出该文件所需的#include最小列表
  - 模块头文件
  - 本地/私有头文件
  - LLVM头文件
  - 系统头文件

#### 样式

- 行宽80个字符
- 在源文件中使用空格而不是制表符
- 不要在行尾加空格
- 格式化多行lambda时，将其格式化为代码块

#### 编译

- 应该像对待错误一样，对待编译器警告

- 有些编译器可以通过在代码上加冗余括号，抑制警告

- 不使用异常或RTTI运行时类型信息

- 不使用静态构造函数和全局变量

- c++中，class和struct关键字几乎可以互换使用。唯一的区别是class默认使所有成员为私有，而struct默认使所有成员为公共

- 不要使用带括号的初始化列表来调用构造函数

- 初始化变量时使用带括号的初始化列表，在左花括号前使用等号

- 使用auto类型推断使代码更具可读性

- 除非需要复制，否则使用auto &表示值，auto *表示指针

- 集合和映射等无序容器与指针键一起使用时，迭代顺序是未定义的

- 编写只计算一个布尔值的小循环是非常常见的，转化返回布尔值的函数

- 充分使用“assert”宏，检查您的所有先决条件和假设

- 尽可能使用基于范围的循环

  ```c++
  BasicBlock *BB = ...
  for (Instruction &I : *BB)
    ... use I ...
  ```

- 避免每次循环计算一次终结条件

  ```c++
  BasicBlock *BB = ...
  for (auto I = BB->begin(); I != BB->end(); ++I)
    ... use I ...
  
  // 替换为
  BasicBlock *BB = ...
  for (auto I = BB->begin(), E = BB->end(); I != E; ++I)
    ... use I ...
  ```

- 尽可能使用前递增

#### 命名

- 通常名称应该是驼峰式的
- 类型名称(包括类、结构体、枚举、类型定义等)应该是名词，并以大写字母开头
- 变量名应该是名词(因为它们代表状态)，并以大写字母开头
- 函数名应该是动词短语(因为它们代表动作)，并以小写字母开头
- Enum声明是类型，应该遵循类型的命名约定
- 枚举(例如enum {Foo, Bar})和公共成员变量应该以大写字母开头，就像类型一样

## Tip(魔术方法)

> https://docs.python.org/zh-cn/3/reference/datamodel.html
>
> https://docs.python.org/3/reference/datamodel.html

#### 基本定制

- \_\_doc\_\_类或函数文档字符串，没有则为None，不会被子类继承

- \_\_name\_\_类名或函数名

- \_\_module\_\_类或函数所属模块名称，没有则为None

- \_\_defaults\_\_ 函数具有默认值的参数的默认参数值组成的元组，如无任何参数具有默认值则为None

- \_\_code\_\_ 函数编译后的代码对象

- \_\_globals\_\_ 函数中全局变量的字典的引用(函数所属模块的全局命名空间)

- \_\_self\_\_ 类实例对象本身的引用

- \_\_bases\_\_ 基类的元组

- \_\_class\_\_ 实例对应的类

- \_\_dict\_\_ 属性命名空间的字典

- \_\_new\_\_ 静态方法，创建一个cls类新实例，如果在构造对象期间被发起调用并且它返回了一个实例，新实例的\_\_init\_\_方法将以\_\_init\_\_(self[, ...]) 的形式被发起调用；未返回cls实例，init不会被调用

```python
class C():
    ...

#x = C(23) 等价于
x = C.__new__(C, 23)
if isinstance(x, C):
    type(x).__init__(x, 23)
```

- \_\_init\_\_ 实例初始化，实例是由\_\_new\_\_()和\_\_init\_\_()协作构造完成的(由\_\_new\_\_()创建，并由\_\_init\_\_()定制)，所以\_\_init\_\_()返回的值只能是None，否则会在运行时引发TypeError

- \_\_del\_\_ 实例将被销毁时调用(内存回收)

- \_\_str__ 定义print()行为

- \_\_bool\_\_ 布尔测试


#### 属性访问定制

- \_\_getattr\_\_ 属性引用(属性未定义时)

- \_\_getattribute\_\_ 属性引用(无条件地被调用，不管属性是否定义)，同时定义了\_\_getattr\_\_，\_\_getattr\_\_不会被调用

```python
class Demo(object):

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __getattribute__(self, attr):
        try:
            # return self.__dict__[attr] 死循环
            return object.__getattribute__(self, attr)
        except:
            return "NOT FIND"


demo = Demo("aaa", 18)
print(demo.name, demo.age, demo.addr)
```

- \_\_setattr\_\_ 属性被尝试赋值时被调用

- \_\_delattr\_\_ 属性被尝试删除时被调用

#### 属性描述器定制

- \_\_get\_\_,\_\_set\_\_,\_\_delete\_\_属性描述器

```python
class Test:

    def __init__(self, filed):
        self.field = "_" + filed

    def __get__(self, instance, owner=None):
        return getattr(instance, self.field)

    def __set__(self, instance, value):
        setattr(instance, self.field, value)


class Demo(object):

    def __init__(self, name, age):
        self._name = name
        self._age = age

    name = Test("name")
    age = Test("age")


demo = Demo("aaa", 18)
demo.age = 16
print(demo.name, demo.age)
```

#### 其他控制

- \_\_slots\_\_显式地声明属性，并禁止创建\_\_dict\_\_和\_\_weakref\_\_(除非是在\_\_slots\_\_中显式地声明)，相比使用 *__dict__* 此方式可以显著地节省空间，属性查找速度也可得到显著的提升
  - 当继承自一个未定义\_\_slots\_\_的类时，实例的\_\_dict\_\_和\_\_weakref\_\_属性将总是可访问
  - 没有\_\_dict\_\_变量，实例就不能给未在\_\_slots\_\_定义中列出的新变量赋值

#### 模拟可调用对象

- \_\_call\_\_ 实例当作函数调用时调用

#### 模拟容器类型

- \_\_len\_\_ len()调用

- \_\_getitem\_\_(self, key) 实现self\[key\]取值

- \_\_setitem\_\_(self, key, value) 实现self\[key\]赋值

- \_\_iter\_\_(self) 支持迭代协议，返回迭代器

```python
from typing import Iterable


class Demo(object):

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __getitem__(self, key):
        value = getattr(self, key, None)
        return value if value is not None else "Not Found"

    def __setitem__(self, key, value):
        setattr(self, key, value)

    def __iter__(self):
        return iter(list(self.__dict__.keys()))


demo = Demo("aaa", 18)
demo["age"] = 16
for key in demo:
    print(demo[key])
print(issubclass(Demo, Iterable))
print(isinstance(demo, Iterable))
```

- \_\_next\_\_ 迭代器

```python
from typing import Iterable, Iterator


class MyRange(object):
    '''
    Iterable: 有迭代能力的对象，一个类，实现了__iter__，那么就认为它有迭代能力，通常此函数必须返回一个实现了__next__的对象，如果自己实现了，你可以返回self，当然这个返回值不是必须的
    Iterator: 迭代器(当然也是Iterable)，同时实现了__iter__和__next__的对象，缺少任何一个都不算是Iterator
    '''

    def __init__(self, end):
        self.start = 0
        self.end = end

    def __next__(self):
        if self.start < self.end:
            ret = self.start
            self.start += 1
            return ret
        else:
            raise StopIteration


b = MyRange(5)
print(next(b))
print(next(b))
print(next(b))
print(next(b))
print(next(b))
# print(next(b))
```

#### with上下文

- \_\_enter\_\_
- \_\_exit\_\_(self, exc_type, exc_value, exc_traceback) \_\_exit\_\_方法中异常，被顺利捕捉并进行了处理。如果方法\_\_exit\_\_没有返回 True，异常仍然会被抛出。如果确定异常已经被处理了，请\_\_exit\_\_的最后加上return True

## Share(getopt)

### getopt

- `int getopt(int ___argc, char *const *___argv, const char *__shortopts)`处理短格式命令选项
- argc，argv取自main函数argc，argv
- shortopts短格式命令选项定义，如`ht:v::`（:表示t后带参数，::表示v后可带可不带参数），-h -t 30 -v
- 返回命令项，如'h'，'t'，'v'带参数选项用optarg获取选项，未定义返回'?'并输出提示信息
- 在while循环中解析完参数返回-1(EOF)

### getopt_long

- `int getopt_long(int ___argc, char * const * ___argv, const char * __shortopts, const option * __longopts, int * __longind)`兼容处理短格式长格式命令选项
- argc，argv，shortopts同getopt 
- longopts为`struct option`数组，长格式命令选项定义`struct option{const char *name;   int has_arg;  int *flag;  int val;};`，如`{"head", no_argument, &method, METHOD_HEAD}`，name选项名称，has_arg选项后是否有参数，flag后面val存储位置(为NULL时getopt_long返回val，不为空val填充*flag同时getopt_long返回0)
- 返回命令项val，如'h'，'t'，'v'带参数选项用optarg获取选项，未定义返回'?'并输出提示信息
- 在while循环中解析完参数返回-1(EOF)

### demo(重构封装webbench)

> http://home.tiscali.cz/~cz210552/webbench.html

```c++
// option.h
#ifndef WEBBENCH_OPTION_H_
#define WEBBENCH_OPTION_H_

#define METHOD_GET 0
#define METHOD_HEAD 1
#define METHOD_OPTIONS 2
#define METHOD_TRACE 3

#define HTTP_09 0
#define HTTP_10 1
#define HTTP_11 2

class WebbenchOption {
public:
  WebbenchOption();
  ~WebbenchOption();
  static void usage(void);
  void print_option();
  void param_short_option(const int argc, char *argv[]);
  void param_long_option(const int argc, char *argv[]);
  static const char *const shortopts;

private:
  void param_option(const int opt);
  int force;
  int force_reload;
  int benchtime;
  char *proxyhost;
  int proxyport;
  int clients;
  int http10;
  int method;
};

#endif


// option.cc
#include "option.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <getopt.h>

const char *const WebbenchOption::shortopts = "frt:p:c:?h";

WebbenchOption::WebbenchOption() {
  force = 0;
  force_reload = 0;
  benchtime = 30;
  proxyhost = NULL;
  proxyport = 80;
  clients = 1;
  http10 = HTTP_09;
  method = METHOD_GET;
}

WebbenchOption::~WebbenchOption() {}

void WebbenchOption::usage(void) {
  // 原来的9，1，2代表http09，http10，http11短格式和长格式差异大
  // 去除短格式，强制使用长格式
  fprintf(stderr,
          "webbench [option]... URL\n"
          "  -f|--force               Don't wait for reply from server.\n"
          "  -r|--reload              Send reload request - Pragma: no-cache.\n"
          "  -t|--time <sec>          Run benchmark for <sec> seconds. Default "
          "30.\n"
          "  -p|--proxy <server:port> Use proxy server for request.\n"
          "  -c|--clients <n>         Run <n> HTTP clients at once. Default "
          "one.\n"
          "  --http09                 Use HTTP/0.9 style requests.\n"
          "  --http10                 Use HTTP/1.0 protocol.\n"
          "  --http11                 Use HTTP/1.1 protocol.\n"
          "  --get                    Use GET request method.\n"
          "  --head                   Use HEAD request method.\n"
          "  --options                Use OPTIONS request method.\n"
          "  --trace                  Use TRACE request method.\n"
          "  -h|--help                This information.\n");
}

void WebbenchOption::print_option() {
  printf("webbench option:\n");
  printf("\t--force\t%d\n", force);
  printf("\t--reload\t%d\n", force_reload);
  printf("\t--time\t%d\n", benchtime);
  printf("\t--proxy\t%s:%d\n", proxyhost, proxyport);
  printf("\t--clients\t%d\n", clients);
  printf("\t--http10\t%d\n", http10);
  printf("\t--method\t%d\n", method);
}

void WebbenchOption::param_option(const int opt) {
  char *tmp = NULL;

  switch (opt) {
  case 0:
    break;
  case '?':
    break;
  case 'f':
    force = 1;
    break;
  case 'r':
    force_reload = 1;
    break;
  case 't':
    benchtime = atoi(optarg);
    break;
  case 'p':
    // server:port
    proxyhost = optarg;
    tmp = strrchr(optarg, ':');
    if (tmp == NULL) {
      break;
    }
    if (tmp == optarg) {
      fprintf(stderr, "Error in option --proxy %s: Missing hostname.\n",
              optarg);
      exit(2);
    }
    // optarg + strlen(optarg) - 1指针操作
    if (tmp == optarg + strlen(optarg) - 1) {
      fprintf(stderr, "Error in option --proxy %s Port number is missing.\n",
              optarg);
      exit(2);
    }
    // *tmp = '\0'作用在于将proxyhost "test.com:8888" 改为"test.com\08888"
    // 意思稍隐晦，可以改为更明显些
    *tmp = '\0';
    proxyport = atoi(tmp + 1);
    break;
  case 'c':
    clients = atoi(optarg);
    break;
  case 'h':
    WebbenchOption::usage();
    exit(0);
    break;
  default:
    break;
  }
}

void WebbenchOption::param_short_option(const int argc, char *argv[]) {
  if (argc == 1) {
    WebbenchOption::usage();
    exit(2);
  }

  int opt = 0;
  while ((opt = getopt(argc, argv, WebbenchOption::shortopts)) != EOF) {
    param_option(opt);
  }
}

void WebbenchOption::param_long_option(const int argc, char *argv[]) {
  if (argc == 1) {
    WebbenchOption::usage();
    exit(2);
  }

  int opt = 0;
  const struct option longopts[] = {
      {"help", no_argument, NULL, 'h'},
      {"force", no_argument, NULL, 'f'},
      {"reload", no_argument, NULL, 'r'},
      {"time", required_argument, NULL, 't'},
      {"proxy", required_argument, NULL, 'p'},
      {"clients", required_argument, NULL, 'c'},
      {"http09", no_argument, &http10, HTTP_09},
      {"http10", no_argument, &http10, HTTP_10},
      {"http11", no_argument, &http10, HTTP_11},
      {"get", no_argument, &method, METHOD_GET},
      {"head", no_argument, &method, METHOD_HEAD},
      {"options", no_argument, &method, METHOD_OPTIONS},
      {"trace", no_argument, &method, METHOD_TRACE},
      {NULL, 0, NULL, 0}};
  while ((opt = getopt_long(argc, argv, "?h:m:t:p:c:", longopts, NULL)) !=
         EOF) {
    param_option(opt);
  }
}


// webbench.cc
/*
 * Simple forking WWW Server benchmark
 */
#include "option.h"
#include <stdio.h>

/*
 * Return codes:
 *    0 - sucess
 *    1 - benchmark failed (server is not on-line)
 *    2 - bad param
 *    3 - internal error, fork failed
 */
int main(int argc, char *argv[]) {
  // WebbenchOption options;
  // options.param_short_option(argc, argv);
  // options.print_option();

  WebbenchOption options1;
  options1.param_long_option(argc, argv);
  options1.print_option();

  return 0;
}


// 运行 ./webbench -p "test.com:8888" -t 60 --options -v --ver
```

