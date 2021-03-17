---
Category: ARTS
Author: qianlizhixing
Title: ARTS第十周
Date: 2020-06-29
Tags: 单元测试, pyunit
---

## Algorithm

leetcode 529 扫雷游戏

- [原题](https://leetcode-cn.com/problems/minesweeper/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/python/529.py)

## Review

> https://medium.com/@amymaraisane/6-tips-to-success-as-a-self-taught-software-developer-ba5eebfb7a98

### 自学编程的6个技巧

- 写下你的问题
  - 搜索答案更容易，迫使你有意识地清晰地表达你的问题
  - 避免陷入浩瀚的文档中，意识主要方向
  - 有时会发现更好的答案，见证自己的成长
- 记录花在编码上的时间
- 展示自己学习成果
- 向别人展示你的成果，推销自己
- 为代码编写单元测试
- 为解决现实需求编码

## Tip

- hash存储value-index，改变list的indexof循环判断value查找index

```python
# 客户是分级的，一级二级客户的code是不同的，现实中可能是层层代理商，找出代理商的级次
custom = ['001', '00101', '002', '00201', '00202', '0020201']

# 可能存在数据库中，或其他配置文件中，-分割每级长度
level = '3-2-2'


# 用数组索引当作级次，存放对应长度
def solution1(level, custom):
  print('solution1')
  help = list(map(int, level.split('-')))
  for i in range(1, len(help)):
    help[i] += help[i - 1]
  for cur in custom:
    print(f'custom:{cur} level:{help.index(len(cur)) + 1}')


#使用hash存储value-index，改变list的indexof循环判断value查找index
def solution2(level, custom):
  print('solution2')
  help = {}
  cur = 0
  curLen = 0
  for i in list(map(int, level.split('-'))):
    curLen += i
    cur += 1
    help[curLen] = cur
  for cur in custom:
    print(f'custom:{cur} level:{help[len(cur)]}')


solution1(level, custom)
solution2(level, custom)
```

## Share

> https://en.wikipedia.org/wiki/Unit_testing
>
> https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks
>
> https://docs.python-guide.org/writing/tests/
>
> https://docs.python.org/zh-cn/3/library/unittest.html#module-unittest
>
> https://docs.python.org/zh-cn/3/using/cmdline.html#cmdoption-m
>
> https://docs.python.org/3/library/__main__.html?highlight=main#module-__main__
>
> https://docs.pytest.org/en/latest/
>
> https://github.com/scrapy/scrapy/tree/master/tests


### 单元测试定义

- 在不运行最终应用程序前，先为最小的可测试单元编写测试，然后为它们之间的复合行为编写测试，以确保某个部分符合其设计并按预期运行，可以为复杂的应用程序构建全面的测试。
- 最小的可测试单元可以是一个的函数，一个类，一个接口，一个模块等。

### 单元测试意义

- 编写单元测试，需要一定的‘代价’，因为质量不高的最小单元，是写不出好的单元测试的。
- 付出‘代价’的同时，带来的优势(相互影响包含)。
  - 提高质量，强迫向大师看齐
    - 逼迫遵守单一职责原则，保证单一功能
    - 逼迫遵守低耦合原则，毕竟职责单一
    - 逼迫遵守输入输出无状态，即函数式编程理念，不要滥用全局变量等特技
    - 逼迫遵守编码规范，框架规范(不想自己造轮子，就要使用开源测试框架，就要按套路组织文件，组织命名)
  - 提高可维护性，保证重构质量
    - 新老交接，单元测试是活的文档，有助于新人熟悉功能和业务
    - 重构过程，保证之前的测试用例可以通过
  - 提高自动化程度
    - 单元测试注重的粒度更小，可以和接口测试，UI测试，集成测试有机结合
    - 可以渗透到自动化，持续集成，持续构建，持续交付，静态分析中(如Jenkins代码测试覆盖率，通过率)
  - 传承开源文化(分享，互助，团队)
    - 单元测试和代码评审(code review)，已成为软件工程标准化流程
    - 讨论交流中取长补短
    - 大佬呼吁，没有代码评审和单元测试文化的公司，请离开

### 单元测试框架

- 几乎所有的编程语言都有单元测试框架

- 先来感受下，一个c语言的单元测试框架Cmocka

  ```c
  /**
   * https://github.com/clibs/cmocka/blob/master/example/assert_module.h
   * 头文件
   */
  //将value指向的值加一
  extern void increment_value(int * const value);
  //将value指向的值减一
  extern void decrement_value(int * const value);
  
  
  /**
   * https://github.com/clibs/cmocka/blob/master/example/assert_module.c
   * 源文件
   */
  #include <assert.h>
  #include "assert_module.h"
  
  void increment_value(int * const value) {
      assert(value);
      (*value) ++;
  }
  
  void decrement_value(int * const value) {
      if (value) {
        (*value) --;
      }
  }
  
  
  /**
   * https://github.com/clibs/cmocka/blob/master/example/assert_module_test.c
   * 测试入口文件
   */
  #include <stdarg.h>
  #include <stddef.h>
  #include <setjmp.h>
  #include <cmocka.h>
  #include "assert_module.h"
  
  /**
   *increment_value执行过程，assert(value)为false，抛出异常
   */
  static void increment_value_fail(void **state) {
      (void) state;
  
      increment_value(NULL);
  }
  
  /**
   *increment_value执行过程，assert(value)为false，抛出异常
   *expect_assert_failure期望异常出现，用例通过
   */
  /* This test case succeeds since increment_value() asserts on the NULL
   * pointer. */
  static void increment_value_assert(void **state) {
      (void) state;
  
      expect_assert_failure(increment_value(NULL));
  }
  
  /**
   *decrement_value执行过程，if (value)为false，没有运行(*value) --，也没有异常
   *expect_assert_failure期望异常未出现，用例失败
   */
  static void decrement_value_fail(void **state) {
      (void) state;
  
      expect_assert_failure(decrement_value(NULL));
  }
  
  int main(void) {
      const struct CMUnitTest tests[] = {
          cmocka_unit_test(increment_value_fail),
          cmocka_unit_test(increment_value_assert),
          cmocka_unit_test(decrement_value_fail),
      };
      return cmocka_run_group_tests(tests, NULL, NULL);
  }
  
  /**
   * main.c
   * 应用入口文件
   */
  #include <stdio.h>
  #include "assert_module.h"
  
  int main(int argc, char **argv) {
      int x = 5;
      increment_value(&x);
      printf(“5加1后：%d\n”, x);
      
      int y = 9;
      decrement_value(&y);
      printf(“9减1后：%d\n”, y);
      
      return 0;
  }
  ```

  ```c
  /*
   * 头文件tool.h
   */
  #ifndef __TOOL_H__
  #define __TOOL_H__
  
  #include <stdbool.h>
  
  extern void readline(char *value);
  extern bool isValidNum(char *value);
  
  #endif
  
  /*
   * 源文件tool.c
   */
  #include <stdio.h>
  #include <string.h>
  #include "tool.h"
  
  void readline(char *value) {
    int position = 0;
    char c;
    while ((c = getchar()) != '\n') {
      value[position++] = c;
    }
    value[position] = '\0';
  }
  
  bool isValidNum(char *value) {
    if (!value) {
      return false;
    }
  
    //正负标志在第一位跳过
    if (*value == '-' || *value == '+') {
      value += 1;
    }
  
    int position = 0;
    while (*value) {
      char c = *value;
      //数字
      if (c < '0' || c > '9') {
        return false;
      }
      position++;
      value += 1;
    }
  
    return position > 0;
  }
  
  /*
   * 单元测试入口test.c
   */
  #include <setjmp.h>
  #include <stdarg.h>
  #include <stddef.h>
  #include <stdint.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <cmocka.h>
  #include "tool.h"
  
  static void test_isValidNum_1(void **state) {
    (void)state; /* unused */
    char value[] = "tst";
    bool want = false;
    bool target = isValidNum(value);
    assert_int_equal(want, target);
  }
  
  static void test_isValidNum_2(void **state) {
    (void)state; /* unused */
    char value[] = "123";
    bool want = true;
    bool target = isValidNum(value);
    assert_int_equal(want, target);
  }
  
  static void test_isValidNum_3(void **state) {
    (void)state; /* unused */
    char value[] = "+123";
    bool want = true;
    bool target = isValidNum(value);
    assert_int_equal(want, target);
  }
  
  static void isValidNum_4(void **state) {
    (void)state; /* unused */
    char value[] = "1-23";
    bool want = true;
    bool target = isValidNum(value);
    assert_int_equal(want, target);
  }
  
  int main(int argc, char **argv) {
    const struct CMUnitTest tests[] = {
        cmocka_unit_test(test_isValidNum_1), cmocka_unit_test(test_isValidNum_2),
        cmocka_unit_test(test_isValidNum_3), cmocka_unit_test(isValidNum_4)};
  
    return cmocka_run_group_tests(tests, NULL, NULL);
  }
  
  /*
   * 应用入口main.c
   */
  #include <stdio.h>
  #include "tool.h"
  
  #include <stdio.h>
  #include "tool.h"
  
  int main(int argc, char **argv) {
    char tmp[255];
  
    while (true) {
      readline(tmp);
      if (isValidNum(tmp)) {
        printf("%s is number!\n", tmp);
      } else {
        printf("%s is not number!\n", tmp);
      }
    }
  
    return 0;
  }
  ```

- 用例通过：期望值==运行值；用例失败：期望值!=运行值。

- 测试assert_module_test.c和assert_module.h assert_module.c是隔离的，互不影响。

- 编写一个main.c文件，作为正式的程序入口；assert_module_test.c作为单元测试的入口。

- 正式编译指定main.c，单元测试指定assert_module_test.c。

- 结合编程语言本身特点，测试框架可以隐藏入口文件，通过@Test，文件Test_等魔法分离发布代码和测试代码，前提条件，遵守约定(类似Servlet规范等)；指定运行入口(如pytest)。

### unittest

- unittest是python标准模块，风格类似JUnit。

- demo文件运行模式

  ```python
  #launch.json
  {
    "version": "0.2.0",
    "configurations": [{
      "name": "mytest",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "args": ["-v"]
    }]
  }
  
  #widget.py
  class Widget():
  
    def __init__(self, name):
      self.name = name
      self.width = 50
      self.height = 50
  
    def size(self):
      return (self.width, self.height)
  
    def resize(self, width, height):
      self.width = width
      self.height = height
  
  #demo.py
  import unittest
  from widget import Widget
  
  
  # unittest.TestCase提供了测试样例运行的基本框架
  class Demo(unittest.TestCase):
  
    # 每一个测试用例必须以test开头，assert开头方法比较预期值和运行值
    def test_upper(self):
      self.assertEqual('foo'.upper(), 'FOO')
  
    def testadd(self):
      self.assertTrue(2 + 3 == 6)
  
    def tstadd(self):
      self.assertTrue(2 + 3 == 5)
  
  
  class Tst(unittest.TestCase):
  
    def testin(self):
      self.assertNotIn('n', ['h', 'e', 'l', 'l', 'o'])
  
  
  class WidgetTest(unittest.TestCase):
  
    #前置操作，创建资源等
    def setUp(self):
      self.widget = Widget('The widget')
  
    def test_default_widget_size(self):
      value = self.widget.size()
      self.assertEqual(value, (50, 50), 'incorrect default size')
  
    def test_widget_resize(self):
      self.widget.resize(100, 150)
      value = self.widget.size()
      self.assertEqual(value, (100, 150), 'wrong size after resize')
  
    #后置操作，清理资源等
    def tearDown(self):
      pass
  
  
  if __name__ == '__main__':
    #launch配置-v显示更详细信息
    unittest.main()
      
      
  #运行结果
  '''
  test_upper (__main__.Demo) ... ok
  testadd (__main__.Demo) ... FAIL
  testin (__main__.Tst) ... ok
  test_default_widget_size (__main__.WidgetTest) ... ok
  test_widget_resize (__main__.WidgetTest) ... ok
  
  ======================================================================
  FAIL: testadd (__main__.Demo)
  ----------------------------------------------------------------------
  Traceback (most recent call last):
    File "d:\testdemo\demo.py", line 13, in testadd
      self.assertTrue(2 + 3 == 6)
  AssertionError: False is not true
  
  ----------------------------------------------------------------------
  Ran 5 tests in 0.006s
  
  FAILED (failures=1)
  '''
  ```

- demo命令行运行模式，让框架自己找到测试单元，或指定测试单元

  ```cmd
  python -m unittest -v demo.py            #指定demo.py文件
  python -m unittest -v                    #将demo.py改名为test*.py
  python -m unittest -v demo.Demo          #指定测试类
  python -m unittest -v demo.Demo.tstadd   #指定测试函数
  ```

- python -m参数：在 sys.path 中搜索指定名称的模块并将其内容作为 \_\_main\_\_ 模块来执行，运行的是该模块下\_\_mian\_\_.py。

  ```python
  #unittest模块下__main__.py
  from .main import main
  
  main(module=None) #unittest.main()
  ```

- unittest.main()即TestProgram()是unittest框架执行的入口，TestLoader(默认加载器)识别和加载了测试用例。

- 将测试代码和源代码分开到不同文件很有必要。

### pytest

- 第三方模块，需要安装

- 框架封装的更智能，用例发现更简单(`test_` 前缀函数， `Test` 前缀类)，用户只需要按约定命名并import pytest即可

  ```python
  #luanch.json
  {
    "version": "0.2.0",
    "configurations": [{
      "name": "mytest",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "module": "pytest",
      "cwd": "${fileDirname}"
    }]
  }
  
  #demo.py
  def inc(x):
    return x + 1
  
  #test_demo.py
  import pytest
  from demo import inc
  
  
  def test_answer():
    assert inc(3) == 5
  
  
  class TestDemo():
  
    def testinc(self):
      assert inc(8) == 9
   
  
  '''
  =========================================================== test session starts ============================================================
  platform win32 -- Python 3.7.5, pytest-5.3.5, py-1.8.1, pluggy-0.13.1
  rootdir: D:\testdemo
  collected 2 items                                                                                                                            
  
  test_demo.py F.                                                                                                                       [100%] 
  
  ================================================================= FAILURES ================================================================= 
  _______________________________________________________________ test_answer ________________________________________________________________ 
  
      def test_answer():
  >     assert inc(3) == 5
  E     assert 4 == 5
  E      +  where 4 = inc(3)
  
  test_demo.py:6: AssertionError
  ======================================================= 1 failed, 1 passed in 0.14s ========================================================
  '''
  ```

#### 指定测试用例

- 在模块中运行测试 pytest test_mod.py
- 在目录中运行测试 pytest testing/
- 按节点ID运行测试 pytest test_mod.py::test_func或者pytest test_mod.py::TestClass::test_method
- 从包运行测试 pytest --pyargs pkg.testing

#### 跟踪回溯打印

- --showlocals # show local variables in tracebacks
- -l           # show local variables (shortcut)
- --tb=auto    # (default) 'long' tracebacks for the first and last # entry, but 'short' style for the other entries
- --tb=long    # exhaustive, informative traceback formatting
- --tb=short   # shorter traceback format
- --tb=line    # only one line per failure
- --tb=native  # Python standard library formatting
- --tb=no      # no traceback at all

#### 详细总结报告-r

默认fE

- f -失败

- E -误差

- s 跳过

- x -失败

- X -XPASS

- p 通过

- P -通过输出

#### 保存结果

- JUnitXML格式文件 pytest --junitxml=path
- 结果日志格式文件  pytest --resultlog=path

#### 代码调用pytest

- pytest.main()

#### fixtures

- 初始化测试功能，它们提供了一个固定的基线，以便测试可靠地执行并产生一致的、可重复的结果。初始化可以设置服务、状态或其他操作环境。在fixture函数中，每个函数的参数通常在test之后被命名为fixture
- pytest fixtures相对于传统的xUnit风格的setup/teardown函数提供了显著的改进
- conftest.py共享初始化
- @pytest.fixtures(params=)，params可循环序列，request.param . 无需更改测试功能代码。让我们再跑一次

### 总结

- 单元测试框架上手快
- 多写多练多看
- 异步多线程场景，编写单元测试比较困难