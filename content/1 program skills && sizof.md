---
Category: ARTS
Author: qianlizhixing
Title: ARTS第一周
Date: 2020-04-21
Tags: sizeof
---

## Algorithm

leetcode 206 反转链表
- [原题](https://leetcode-cn.com/problems/reverse-linked-list/)
- [解题](https://github.com/qianlizhixing12/leetcode/blob/master/c/206.c)

## Review

> https://medium.com/@maladdinsayed/advanced-techniques-and-ideas-for-better-coding-skills-d632e9f9675

#### 好的编程技巧和思想

- 不要重构没有单元测试的项目，除非你先补充单元测试。
- 从最深层开始重构，通常意味着功能单一，影响范围小，重构相对容易。
- 尽量使函数功能尽量单一，抽象为功能更小的子函数。
- 即使if只有一行，也尽量使用{}。
- 尽量不要使用"幻数"(直接使用常数)。
- 规范函数，变量，测试命名，使之有助理解逻辑。
- 用严格等号代替非严格等号。
- 给公共方法一个简短的名字。
- 移除没有被调用的方法。

这些应该都是耳熟能详的编程规范，软件是整体和局部的结合，都决定软件的质量。

程序员进阶法宝：模仿好的代码，养成好的习惯，想法经历大的项目。

## Tip

链表相关问题，可以适当借助“哨兵”，统一临界情况的处理。

## Share

> 引用：
>
> https://www.geeksforgeeks.org/sizeof-operator-c/
>
> https://en.wikibooks.org/wiki/C_Programming/Pointers_and_arrays#sizeof

```c
#include <stdio.h>

void demo(float arr[]) {
 printf("arr size: %d\n", sizeof(arr) / sizeof(float)); //arr size: 2
}

int main(int argc, char **argv) {
 float arr[] = {1.0, 2.0, 3.0, 4.0, 5.0};
 
 printf("arr size: %d\n", sizeof(arr) / sizeof(float)); //arr size: 5
 
 demo(arr);

 return 0;
}
```

c sizeof可用于数据类型或表达式(包含变量)，编译时用常数值替换sizeof，不是运行时计算的。

1. 当用于数据类型时，返回该类型在内存中占用的字节数。(sizeof(int ) === 2)
2. 当用于表达式时，返回表达式值占用的字节数

注意

1. 可用于计算数组大小，sizeof(arr) / sizeof(float)   //arr size: 5
2. 可用于动态申请内存块，int * ptr = (int*)malloc(10 * sizeof(int))
3. 不要用sizeof计算数组参数的size(数组参数被当作指针处理了)，即必须用在数组定义的函数里面