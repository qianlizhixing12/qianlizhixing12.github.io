---
Category: ARTS
Author: qianlizhixing
Title: ARTS第三十四周
Date: 2021-06-28
Tags: python库, google code style, webbench
---

## Algorithm

leetcode 剑指Offer20 表示数值的字符串

- [原题](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)
- [解题](https://github.com/qianlizhixing12/coding-training/blob/main/leetcode/Offer20.py)
- 状态模式，状态较多容易写错

## Review

> https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/#
>
> https://google.github.io/styleguide/cppguide.html#Header_Files

### 头文件

- 插入文本的文件应以.inc结尾

- 头文件要能够自给自足，即头文件本身的依赖要全部包含，调用者不用考虑引用的头文件本身的依赖

  ```c++
  // 反例
  // b.h依赖a.h，调用者c.cpp使用b.h
  
  // file: b.h b不是自给自足的
  #include ...
  ...
  
  // file: c.cpp
  #include "a.h"
  #include "b.h"
  ```

- 头文件都应该使用#define来防止头文件被多重包含，命名格式`<PROJECT>_<PATH>_<FILE>_H_`

- 尽可能地避免使用前置声明，使用 `#include` 包含需要的头文件即可

- 只有当函数只有10行甚至更少时才将其定义为内联函数

- 标准的头文件包含顺序可增强可读性, 避免隐藏依赖: 相关头文件, C库, C++库, 其他库的.h, 本项目内的.h

  - `dir/foo.cc` 或 `dir/foo_test.cc` 的主要作用是实现或测试 `dir2/foo2.h` 的功能, `foo.cc` 中包含头文件的次序
    - `dir2/foo2.h` (`dir/foo.cc` 和 `dir2/foo2.h` 通常位于同一目录下 (如 `base/basictypes_unittest.cc` 和 `base/basictypes.h`), 但也可以放在不同目录下)
    - C 系统文件
    - C++ 系统文件
    - 其他库的 `.h` 文件
    - 本项目内 `.h` 文件
  - 这种优先顺序排序保证当 `dir2/foo2.h` 遗漏某些必要的库时， `dir/foo.cc` 或 `dir/foo_test.cc` 的构建会立刻中止

### 作用域

- 不要在命名空间 `std` 内声明任何东西, 包括标准库的类前置声明. 在 `std` 命名空间声明实体是未定义的行为, 会导致如不可移植. 声明标准库下的实体, 需要包含对应的头文件
- 不应该使用using指示引入整个命名空间的标识符号，例如using namespace std
- 禁止使用内联命名空间（inline namespace）
- 在 `.cc` 文件中定义一个不需要被外部引用的变量时，可以将它们放在匿名命名空间或声明为 `static` 。但是不要在 `.h` 文件中这么做
- 使用静态成员函数或命名空间内的非成员函数, 尽量不要用裸的全局函数. 将一系列函数直接置于命名空间中，不要用类的静态方法模拟出命名空间的效果，类的静态方法应当和类的实例或静态数据紧密相关
- 将函数变量尽可能置于最小作用域内, 离第一次使用越近越好,并在变量声明时进行初始化
- 属于 `if`, `while` 和 `for` 语句的变量应当在这些语句中正常地声明，这些变量的作用域就被限制在这些语句中
- 禁止定义静态储存周期非POD变量，禁止使用含有副作用的函数初始化POD全局变量，因为多编译单元中的静态变量执行时的构造和析构顺序是未明确的，这将导致代码的不可移植

### 类

- 在基类构造函数内调用了自身的虚函数, 调用是不会重定向到子类的虚函数实现
- 不要在无法报出错误时进行可能失败的初始化
- 不要定义隐式类型转换，对于转换运算符和单参数构造函数, 请使用 `explicit` 关键字
- 如果类型需要, 就让它们支持拷贝 / 移动. 否则, 就把隐式产生的拷贝和移动函数禁用
- 仅当只有数据成员时使用 `struct`, 其它一概使用 `class`
- 使用组合常常比使用继承更合理. 如果使用继承的话, 定义为 `public` 继承
- 只有当所有父类除第一个外都是纯接口类时, 才允许使用多重继承. 为确保它们是纯接口, 这些类必须以 `Interface` 为后缀
- 接口是指满足特定条件的类, 这些类以 `Interface` 为后缀 (不强制)
- 只有在意义明显, 不会出现奇怪的行为并且与对应的内建运算符的行为一致时才定义重载运算符
- 将所有数据成员声明为 `private`, 除非是 `static const` 类型成员
- 将相似的声明放在一起, 将 `public` 部分放在最前，具体顺序: 类型 (包括 `typedef`, `using` 和嵌套的结构体与类), 常量, 工厂函数, 构造函数, 赋值运算符, 析构函数, 其它函数, 数据成员

### 函数

- 输入参数在先, 后跟输出参数.
- 编写简短, 凝练的函数
- 所有按引用传递的参数必须加上 `const`
- 输入参数是值参或 `const` 引用, 输出参数为指针. 输入参数可以是 `const` 指针, 但决不能是非 `const` 的引用参数, 除非特殊要求, 比如 `swap()`
- 如果打算重载一个函数, 可以试试改在函数名里加上参数信息. 例如, 用 `AppendString()` 和 `AppendInt()` 等, 而不是一口气重载多个 `Append()`. 如果重载函数的目的是为了支持不同数量的同一类型参数, 则优先考虑使用 `std::vector` 以便使用者可以用列表初始化指定参数
- 虚函数不允许使用缺省参数, 因为在虚函数中缺省参数不一定能正常工作. 如果在每个调用点缺省参数的值都有可能不同, 在这种情况下缺省函数也不允许使用. (例如, 不要写像 `void f(int n = counter++);` 这样的代码.)
- 只有在常规写法 (返回类型前置) 不便于书写或不便于阅读时使用返回类型后置语法

### Google奇技

- 动态分配出的对象最好有单一且固定的所有主, 并通过智能指针传递所有权，不要使用 `std::auto_ptr`, 使用 `std::unique_ptr` 代替它
- 使用 `cpplint.py` 检查风格错误

### 其他特性

- 所有按引用传递的参数必须加上 `const`
- 只在定义移动构造函数与移动赋值操作时使用右值引用, 不要使用 `std::forward` 功能函数，可能会使用 `std::move` 来表示将值从一个对象移动而不是复制到另一个对象
- 不允许使用变长数组和 `alloca()`，改用更安全的分配器（allocator），就像 `std::vector` 或 `std::unique_ptr<T[]>`
- 允许合理的使用友元类及友元函数
- 禁止使用 RTTI
- 不要使用 C 风格类型转换. 而应该使用 C++ 风格
  - 用 `static_cast` 替代 C 风格的值转换, 或某个类指针需要明确的向上转换为父类指针时
  - 用 `const_cast` 去掉 `const` 限定符
  - 用 `reinterpret_cast` 指针类型和整型或其它指针之间进行不安全的相互转换. 仅在你对所做一切了然于心时使用
- 只在记录日志时使用流
- 对于迭代器和其他模板对象使用前缀形式 (`++i`) 的自增, 自减运算符
- 在C++11里，用 constexpr 来定义真正的常量，或实现常量初始化
- 使用宏时要非常谨慎, 尽量以内联函数, 枚举和常量代替之
- 整数用 `0`, 实数用 `0.0`, 指针用 `nullptr` 或 `NULL`, 字符 (串) 用 `'\0'`
- 尽可能用 `sizeof(varname)` 代替 `sizeof(type)`
- 用 `auto` 绕过烦琐的类型名，只要可读性好就继续用，别用在局部变量之外的地方
- 可以用列表初始化
- 适当使用 lambda 表达式。别用默认 lambda 捕获，所有捕获都要显式写出来
- 不要使用复杂的模板编程
- 只使用Boost中被认可的库

### 命名约定

- 函数命名, 变量命名, 文件命名要有描述性，少用缩写；一些特定的广为人知的缩写是允许的
- 文件名要全部小写, 可以包含下划线 (`_`) 或连字符 (`-`), 依照项目的约定. 如果没有约定, 那么 “`_`” 更好
- C++文件要以 `.cc` 结尾, 头文件以 `.h` 结尾. 专门插入文本的文件则以 `.inc` 结尾
- 类型(类, 结构体, 类型定义`typedef`, 枚举, 类型模板参数)名称的每个单词首字母均大写, 不包含下划线
- 变量 (包括函数参数) 和数据成员名一律小写, 单词之间用下划线连接. 类的成员变量以下划线结尾, 但结构体的就不用
- 声明为 `constexpr` 或 `const` 的变量, 或在程序运行期间其值始终保持不变的, 命名时以 “k” 开头, 大小写混合
- 常规函数使用大小写混合, 取值和设值函数则要求与变量名匹配，函数名的每个单词首字母大写, 没有下划线. 对于首字母缩写的单词, 更倾向于将它们视作一个单词进行首字母大写
- 枚举的命名应当和 [常量](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/#constant-names) 或 [宏](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/#macro-names) 一致: `kEnumName` 或是 `ENUM_NAME`
- 通常不应该使用宏，如果不得不用, 其命名像枚举命名一样全部大写, 使用下划线

### 注释

- `//` 或 `/* */` 都可以; 但 `//` *更* 常用. 要在如何注释及注释风格上确保统一
- 在每一个文件开头加入版权公告
- 每个类的定义都要附带一份注释, 描述类的功能和用法, 除非它的功能相当明显
- 函数声明处的注释描述函数功能; 定义处的注释描述函数实现
- 通常变量名本身足以很好说明变量用途. 某些情况下, 也需要额外的注释说明
- 对于代码中巧妙的, 晦涩的, 有趣的, 重要的地方加以注释
- 如果函数参数的意义不明显, 考虑进行弥补
- 不要描述显而易见的现象
- 注意标点, 拼写和语法; 写的好的注释比差的要易读的多
- 那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 `TODO` 注释
- 通过弃用注释（`DEPRECATED` comments）以标记某接口点已弃用.

### 格式

- 每一行代码字符数不超过80
- 尽量不使用非 ASCII 字符, 使用时必须使用 UTF-8 编码
- 只使用空格, 每次缩进 2 个空格，不要在代码中使用制表符，应该设置编辑器将制表符转为空格
- 返回类型和函数名在同一行, 参数也尽量放在同一行, 如果放不下就对形参分行, 分行方式与函数调用一致
- 倾向于不在圆括号内使用空格，关键字 `if` 和 `else` 另起一行
- 句点或箭头前后不要有空格，指针/地址操作符 (`*, &`) 之后不能有空格
- 在声明指针变量或参数时, 星号与类型或变量名紧挨都可以
- 如果一个布尔表达式超过标准行宽, 断行方式要统一
- 不要在 `return` 表达式里加上非必须的圆括号
- 变量及数组初始化，用 `=`, `()` 和 `{}` 均可
- 预处理指令不要缩进, 从行首开始
- 访问控制块的声明依次序是 `public:`, `protected:`, `private:`, 每个都缩进 1 个空格
- 构造函数初始化列表放在同一行或按四格缩进并排多行
- 水平留白的使用根据在代码中的位置决定. 永远不要在行尾添加没意义的留白
- 垂直留白越少越好

## Tip

> https://coolshell.cn/articles/12199.html

- Copy-On-Write技术在于节省内存，用一个引用计数实现内存共享，在最新的STL中已经不支持

## Share(http proxy)

### http proxy代理请求实现方式

> https://datatracker.ietf.org/doc/html/rfc7230
>
> https://datatracker.ietf.org/doc/html/rfc7231
>
> https://datatracker.ietf.org/doc/html/draft-luotonen-web-proxy-tunneling-01

- 普通代理，代理服务器充当中间人，客户端--》代理--》服务端
- 隧道代理，基于http的connect方法(2014HTTP/1.1修订版)

### 普通代理

- 非代理http请求
  - 向host建立tcp连接
  - start-line:method SP request-target(相对url资源路径) SP HTTP-version CRLF
  - host_header:host:hostname
  - 例如目标请求url:http://test.com/books.html，request-target为books.html，host header为test.com
- 普通代理
  - start-line:method SP request-target(绝对url资源路径) SP HTTP-version CRLF
  - host_header:不发送host
  - 代理服务器根据绝对url资源路径，转化为相对url资源路径和host，代理完成中间人角色

### demo(c++重构封装webbench)

> http://home.tiscali.cz/~cz210552/webbench.html

- 普通代理处理demo
- 注意重构的param_http_path，param_http_host方法

```c++
// option.h
#ifndef WEBBENCH_OPTION_H_
#define WEBBENCH_OPTION_H_

#include <string>

using std::string;

class WebbenchOption {
public:
  WebbenchOption();
  ~WebbenchOption();
  static void usage(void);
  void print_option() const;
  void param_option(const int argc, char *argv[]);
  const string &param_http_method() const;
  const string &param_http_version() const;
  const string param_http_path() const;
  const string param_http_host() const;
  const string build_http_headers() const;
  static const string shortopts;

private:
  void fix_option();
  bool force;
  bool force_reload;
  int benchtime;
  string proxyhost;
  int proxyport;
  int clients;
  int http10;
  int method;
  string url;
};

#endif

// options.cc
#include "option.h"
#include <iostream>
#include <unordered_map>
#include <getopt.h>

using std::cout;
using std::endl;
using std::ios_base;
// using std::npos;
using std::unordered_map;

#define METHOD_GET 0
#define METHOD_HEAD 1
#define METHOD_OPTIONS 2
#define METHOD_TRACE 3

#define HTTP_09 0
#define HTTP_10 1
#define HTTP_11 2

static unordered_map<int, string> method_map = {{METHOD_GET, "GET"},
                                                {METHOD_HEAD, "HEAD"},
                                                {METHOD_OPTIONS, "OPTIONS"},
                                                {METHOD_TRACE, "TRACE"}};

static unordered_map<int, string> http_version_map = {
    {HTTP_09, "HTTP/0.9"}, {HTTP_10, "HTTP/1.0"}, {HTTP_11, "HTTP/1.1"}};

const string WebbenchOption::shortopts = "frt:p:c:h";

WebbenchOption::WebbenchOption() {
  force = false;
  force_reload = false;
  benchtime = 30;
  proxyhost = "";
  proxyport = 80;
  clients = 1;
  http10 = HTTP_09;
  method = METHOD_GET;
  url = "";
}

WebbenchOption::~WebbenchOption() {}

void WebbenchOption::usage(void) {
  // 原来的9，1，2代表http09，http10，http11短格式和长格式差异大，强制使用长格式
  cout << "webbench [option]... URL" << endl;
  cout << "  -f|--force               Don't wait for reply from server."
       << endl;
  cout << "  -r|--reload              Send reload request - Pragma: no-cache."
       << endl;
  cout << "  -t|--time <sec>          Run benchmark for <sec> seconds. Default "
          "30."
       << endl;
  cout << "  -p|--proxy <server:port> Use proxy server for request." << endl;
  cout << "  -c|--clients <n>         Run <n> HTTP clients at once. Default "
          "one."
       << endl;
  cout << "  --http09                 Use HTTP/0.9 style requests." << endl;
  cout << "  --http10                 Use HTTP/1.0 protocol." << endl;
  cout << "  --http11                 Use HTTP/1.1 protocol." << endl;
  cout << "  --get                    Use GET request method." << endl;
  cout << "  --head                   Use HEAD request method." << endl;
  cout << "  --options                Use OPTIONS request method." << endl;
  cout << "  --trace                  Use TRACE request method." << endl;
  cout << "  -h|--help                This information." << endl;
}

void WebbenchOption::print_option() const {
  cout.setf(ios_base::boolalpha);
  cout << "webbench option:" << endl;
  // proxy
  cout << "\t--proxy      ";
  if (proxyhost != "") {
    cout << proxyhost << ":" << proxyport;
  } else {
    cout << "no";
  }
  cout << endl;
  // force
  cout << "\t--force      early socket close " << force << endl;
  // reload
  cout << "\t--reload     forcing reload " << force_reload << endl;
  // http version
  cout << "\t--http       using " << param_http_version() << endl;
  // method
  cout << "\t--method     " << param_http_method() << endl;
  // timeout
  cout << "\t--time       " << benchtime << endl;
  // clients
  cout << "\t--clients    " << clients << endl;
  // url
  cout << "\t--url        " << url << endl;
}

void WebbenchOption::param_option(const int argc, char *argv[]) {
  if (argc == 1) {
    usage();
    exit(2);
  }

  // 长命令选项
  int opt = 0;
  string tmp;
  int i;
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
  while ((opt = getopt_long(argc, argv, WebbenchOption::shortopts.c_str(),
                            longopts, NULL)) != EOF) {
    switch (opt) {
    case 0:
      break;
    case '?':
      break;
    case 'f':
      force = true;
      break;
    case 'r':
      force_reload = true;
      break;
    case 't':
      benchtime = atoi(optarg);
      break;
    case 'p':
      // server:port
      tmp = optarg;
      // tmp = strrchr(optarg, ':');
      if (tmp == "") {
        break;
      }
      i = tmp.rfind(":");
      if (i < 0) {
        cout << "Error in option --proxy " << tmp << ": Not valid proxy."
             << endl;
        exit(2);
      }
      if (i == 0) {
        cout << "Error in option --proxy " << tmp << ": Missing hostname."
             << endl;
        exit(2);
      }
      if (i == tmp.size() - 1) {
        cout << "Error in option --proxy " << tmp << " Port number is missing."
             << endl;
        exit(2);
      }
      proxyhost = tmp.substr(0, i);
      proxyport = atoi(tmp.substr(i + 1).c_str());
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

  // url
  if (optind == argc) {
    cout << "webbench: Missing URL!" << endl;
    usage();
    exit(2);
  }
  url = argv[optind];
  i = url.find("://");
  if (i < 0) {
    cout << url << ": is not a valid URL." << endl;
    exit(2);
  }
  i = url.find("/", url.find("://") + 3);
  if (i < 0) {
    cout << "Invalid URL syntax - hostname don't ends with '/'." << endl;
    exit(2);
  }
  if (url.size() > 1500) {
    cout << "URL is too long." << endl;
    exit(2);
  }
  if (proxyhost == "") {
    // 只有http协议可以直连，其他协议要代理支持
    if (0 != url.find_first_of("http://")) {
      cout << "Only HTTP protocol is directly supported, set --proxy for "
              "others."
           << endl;
      exit(2);
    }
  }

  // option
  cout << "----------cmdline option----------" << endl;
  print_option();

  fix_option();

  // fix option
  cout << "--------fix cmdline option--------" << endl;
  print_option();

  cout << "----------------ok----------------" << endl;
}

void WebbenchOption::fix_option() {
  if (clients == 0) {
    clients = 1;
  }

  if (benchtime == 0) {
    benchtime = 30;
  }

  // http/0.9不支持缓冲控制
  if (force_reload && proxyhost != "" && http10 < HTTP_10) {
    http10 = HTTP_10;
  }
  // http/0.9只支持get请求
  if (method == METHOD_HEAD && http10 < HTTP_10) {
    http10 = HTTP_10;
  }
  if (method == METHOD_OPTIONS && http10 < HTTP_11) {
    http10 = HTTP_11;
  }
  if (method == METHOD_TRACE && http10 < HTTP_11) {
    http10 = HTTP_11;
  }
  if (proxyhost != "") {
    http10 = HTTP_11;
  }
}

const string &WebbenchOption::param_http_method() const {
  return method_map[method];
}

const string &WebbenchOption::param_http_version() const {
  return http_version_map[http10];
}

const string WebbenchOption::param_http_path() const {
  if (proxyhost == "") {
    return url.substr(url.find("/", url.find("://") + 3));
  } else {
    return url;
  }
}

const string WebbenchOption::param_http_host() const {
  /* protocol/host delimiter */
  if (proxyhost == "") {
    // 是否包含端口号
    int i = url.find("://") + 3;
    int j = url.find(":", i);
    if (j >= 0) {
      return url.substr(i, j - i);
    } else {
      int k = url.find("/", i);
      return url.substr(i, k - i);
    }
  } else {
    return "";
  }
}

// http_headers与options强耦合
const string WebbenchOption::build_http_headers() const {
  string request;

  // http start-line
  request += param_http_method() + " " + param_http_path() + " " +
             param_http_version() + "\r\n";
  // http headers
  if (http10 > 0) {
    request += "User-Agent: WebBench\r\n";
  }
  if (proxyhost == "" && http10 > 0) {
    request += "Host: " + param_http_host() + "\r\n";
  }
  if (force_reload && proxyhost != "") {
    request += "Pragma: no-cache\r\n";
  }
  if (http10 > 1) {
    request += "Connection: close\r\n";
  }
  /* add empty line at end */
  if (http10 > 0) {
    request += "\r\n";
  }

  return request;
}

// webbench.cc
/*
 * Simple forking WWW Server benchmark
 */
#include "option.h"
#include <iostream>

using std::cout;
using std::endl;

/*
 * Return codes:
 *    0 - sucess
 *    1 - benchmark failed (server is not on-line)
 *    2 - bad param
 *    3 - internal error, fork failed
 */
int main(int argc, char *argv[]) {

  WebbenchOption options;
  options.param_option(argc, argv);
  cout << options.build_http_headers() << endl;

  return 0;
}


// 运行 ./webbench --http11 -p "test.com:8888" -t 60 --trace "https://httpbin.org/get"
// TRACE https://httpbin.org/get HTTP/1.1\r\nUser-Agent: WebBench\r\nConnection: close\r\n\r\n
// 运行 ./webbench --http11 -t 60 --trace "https://httpbin.org/get"
// TRACE /get HTTP/1.1\r\nUser-Agent: WebBench\r\nHost: httpbin.org\r\nConnection: close\r\n\r\n"
```
