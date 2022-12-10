# 面试复习笔记

> 这个repo主要就是记录一些个人复习总结的文档。

- [面试复习笔记](#面试复习笔记)
  - [语言](#语言)
    - [C++](#c)
      - [基础](#基础)
      - [容器](#容器)
      - [类 （封装性）](#类-封装性)
      - [继承和多态](#继承和多态)
      - [内存管理](#内存管理)
      - [拷贝控制](#拷贝控制)
      - [模板](#模板)
      - [多线程](#多线程)
      - [C++ 11新特性](#c-11新特性)
    - [Java](#java)
  - [基础——四大、八股文](#基础四大八股文)
    - [算法](#算法)
      - [回溯](#回溯)
      - [二分查找](#二分查找)
      - [树](#树)
      - [贪心](#贪心)
      - [双指针](#双指针)
      - [排序](#排序)
      - [数据结构](#数据结构)
      - [图](#图)
    - [操作系统](#操作系统)
    - [计算机网络](#计算机网络)
    - [数据库](#数据库)
    - [设计模式](#设计模式)
    - [编译原理](#编译原理)
  - [进阶——系统设计](#进阶系统设计)
    - [可用性与一致性](#可用性与一致性)
      - [CAP理论](#cap理论)
      - [特性 ACID \& BASE](#特性-acid--base)
      - [一致性模式](#一致性模式)
      - [可用性模式](#可用性模式)
    - [一致性算法](#一致性算法)
    - [数据库 DB](#数据库-db)
    - [缓存 Cache](#缓存-cache)
    - [异步 Asynchronism](#异步-asynchronism)
    - [通信 Communication](#通信-communication)
    - [安全 Security](#安全-security)
  - [资料](#资料)

## 语言

### C++

#### 基础

- 类型转换
- 指针和引用
  - 函数指针
  - 左右引用
- 结构体内存计算
- 关键字`const`、`constexptr`、`decltype`、`auto`、`final`、`override`和`explicit`

#### 容器

- 顺序容器
  - 数组底层
  - 栈、（有限队列）堆和队列
- 关联容器
  - 分类
  - 基本操作
  - 底层
    - 散列表
    - 红黑树
  - 冲突解决
- 模板算法

#### 类 （封装性）

- 构造函数和析构函数
  - 构造函数不能写成虚函数
  - 对有继承的情况，析构函数一般写为虚函数
- `this`指针
- 重载
- 访问修饰符
- 友类
- 静态成员和成员函数

#### 继承和多态

- 继承修饰符
- 虚函数实现多态（父类指针指向子类对象）
  - 虚表
  - 纯虚函数
  - 重写

#### 内存管理

- new和malloc
- 智能指针
  - shared_ptr & unique_ptr
    - shared_ptr自实现
  - RAII思想
- allocator类

#### 拷贝控制

- 拷贝构造函数 & 拷贝构造运算符
- 移动构造函数 & 移动构造运算符
  - 右值引用
- 析构函数

#### 模板

- 模板函数
- 模板类
- 萃取技术

#### 多线程

- 线程与进程
- 同步原语
  - 互斥量
  - 信号量
  - 管程
  - 条件变量
- C++ 标准库
  - thread
  - atomic
  - mutex
  - future
- 多线程最佳实践
  - 计数器
  - 线程池

#### C++ 11新特性

**智能指针 & 内存管理**:

- `shared_ptr`和`unique_ptr`
- 内存池

**lambda表达式**:

```c++
[capture list](parameter list) +> return type { function body }
```

### Java

## 基础——四大、八股文

> 有些公司的面试还会涉及一些设计模式和编译原理的知识，也一并放在这里了。

### 算法

#### 回溯

- 子集
- 组合
- 排序
- 搜索（一般会剪枝，比如leetcode 79题非常经典）

#### 二分查找

- 模板 + 变化

#### 树

- 遍历
  - 深度优先 DFS
    - 前序
    - 中序
    - 后序
  - 广度优先 BFS
    - 层序遍历
- 递归

#### 贪心

- 思路：排序+遍历
- 分类
  - 子序列
  - 区间问题
  - 股票问题

#### 双指针

#### 排序

- 快速排序 quick sort
- 合并排序 merge sort
- 堆排序 heap sort
- 其他

#### 数据结构

- 高频 & 初级
  - 栈
  - 堆（优先队列）
  - 队列
- 低频 & 高级
  - 字典树
  - 对顶堆
  - 并查集

#### 图

- 遍历
  - 深度优先 DFS
  - 广度优先 BFS
- 拓扑排序
- 最短路径——迪杰斯特拉 Dijkstra

### 操作系统

- 进程：进程与线程、进程的状态、进程调度算法、进程通信和一些特殊进程（守护、孤儿、僵尸）
- 同步与互斥：多线程、死锁、POSIX的pthread
- 内存管理：虚拟内存、分段和分页

### 计算机网络

- 应用层：
  - HTTP：HTTP方法、GET和POST的区别、HTTP状态码、cookie和session、HTTP1.x/2.0
  - HTTPS：加密方法
- 传输层：
  - TCP/UDP：区别、三次握手、四次挥手、优化（滑动窗口等等）

### 数据库

- SQL语法
- 索引
- 事务及事务隔离级别
- MVCC
- 数据库的锁
- 慢查询优化

### 设计模式

### 编译原理

## 进阶——系统设计

### 可用性与一致性

#### CAP理论

- Consistency
- Availability
- Partition Tolerance

#### 特性 ACID & BASE

#### 一致性模式

- 弱一致性
- 最终一致性
- 强一致性

#### 可用性模式

- 工作到备用切换
- 双工作切换

### 一致性算法

- Raft
- Paxos

### 数据库 DB

- SQL
- NoSQL

### 缓存 Cache

- 模式
  - cache aside
  - write through
  - write back
  - flush
- 问题
  - 缓存击穿
  - 缓存穿透
  - 缓存雪崩

### 异步 Asynchronism

- 消息队列
- 任务队列

### 通信 Communication

- TCP
- UDP
- RPC

### 安全 Security

- 加密与哈希
- 认证与授权
- 限流 & 降级 & 熔断
- 跨站脚本攻击XSS & 跨站请求伪造CSRF

## 资料

**综合**：

[github + CS+NOTES](https://github.com/Zhytou/CS+Notes)
[github system design](https://github.com/donnemartin/system-design-primer)
[phoenix framework](https://icyfenix.cn/)

**Java**：

[JavaGuide](https://javaguide.cn/)

**C++**：

[C++ Primer](https://github.com/bumzy/book/blob/master/C%2B%2B%20%20Primer%E4%B8%AD%E6%96%87%E7%89%88%EF%BC%88%E7%AC%AC%E4%BA%94%E7%89%88%EF%BC%89.pdf)
[STL 源码剖析](https://github.com/SilverMaple/STLSourceCodeNote/blob/master/STL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E5%AE%8C%E6%95%B4%E7%89%88(%E6%B8%85%E6%99%B0%E6%89%AB%E6%8F%8F%E5%B8%A6%E7%9B%AE%E5%BD%95).pdf)
[youtube + boqian](https://www.youtube.com/user/BoQianTheProgrammer)
[知乎 + C++](https://zhuanlan.zhihu.com/p/417640759)
[知乎 + 当面试官问C++ 11新特性时，应该怎么样回答？](https://zhuanlan.zhihu.com/p/420710578)
[帅地 + C++面试题指南](https://www.iamshuaidi.com/2309.html)

**JavaScript**：

[MDN](https://developer.mozilla.org/zh+CN/docs/Learn)

**刷题**：

[CodeTop](https://codetop.cc/home)
[Leetcode](https://leetcode+cn.com/)
