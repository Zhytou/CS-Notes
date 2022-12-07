# 编译

- [编译](#编译)
  - [简介](#简介)
    - [静态库 vs 动态库](#静态库-vs-动态库)
    - [编译型 Complication VS 解释型 Interpreter](#编译型-complication-vs-解释型-interpreter)
    - [及时编译 JIT Just-In-Time Compiling VS 提前编译 AOT Ahead Of Time Compiling](#及时编译-jit-just-in-time-compiling-vs-提前编译-aot-ahead-of-time-compiling)
    - [RTTI Run Time Type Identification](#rtti-run-time-type-identification)
  - [GCC流程](#gcc流程)
    - [预编译](#预编译)
    - [编译](#编译-1)
    - [汇编](#汇编)
    - [链接](#链接)
  - [优化](#优化)
  - [参考](#参考)

## 简介

### 静态库 vs 动态库

**静态库**：

- 静态库对函数库的链接是放在编译时期完成的。
- 程序在运行时与函数库再无瓜葛，移植方便。
- 浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件。 

**动态库**：

- 动态库把对一些库函数的链接载入推迟到程序运行的时期。　
- 可以实现进程之间的资源共享。（因此动态库也称为共享库）
- 将一些程序升级变得简单。
- 甚至可以真正做到链接载入完全由程序员在程序代码中控制（**显示调用**）。

### 编译型 Complication VS 解释型 Interpreter

### 及时编译 JIT Just-In-Time Compiling VS 提前编译 AOT Ahead Of Time Compiling

JIT编译最显著的特点就是代码编译过程发生在程序执行期间，而非执行之前。

### RTTI Run Time Type Identification

通过运行时类型信息程序能够使用基类的指针或引用来检查这些指针或引用所指的对象的实际派生类型。

## GCC流程

### 预编译

- 将所有`#define`删除，并且展开所有的宏定义；
- 处理所有条件编译指令，比如`#if`、`#ifdef`等；
- 处理`#include`预编译指令，将被包含的文件插入到该预编译指令的位置；
  - 注意，这个过程可能是递归的，因为被包含的文件可能还包含着其他文件。

- 删除所有注释；
- 添加行号和文件名标识，以便于编译时产生调试信息标明行号和文件名；
- 保留所有`#pragma`编译器指令，因为编译器需要使用它们。

### 编译

编译过程就是把预处理完的文件进行一系列词法分析、语法分析、语义分析及优化后生成相应的汇编代码文件。

### 汇编

汇编是将汇编代码转换为机器可以执行的指令的流程。

每一个汇编语句几乎都对应着一条机器指令。

### 链接

链接是将所有的机器指令拼接组织成一个可执行文件的流程。

具体来讲，链接过程主要包括了地址和空间分配、符号决议和重定位等这些步骤。

## 优化

## 参考

- [stack over flow - JIT vs Interpreters](https://stackoverflow.com/questions/3718024/jit-vs-interpreters)

- [编译优化，编译器到底做了什么？](https://juejin.cn/post/6933180767656738824)