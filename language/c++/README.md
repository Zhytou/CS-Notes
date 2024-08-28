# C++ Review Reminder

- [C++ Review Reminder](#c-review-reminder)
  - [基础](#基础)
    - [容器](#容器)
    - [类 （封装性）](#类-封装性)
    - [继承和多态](#继承和多态)
    - [拷贝控制](#拷贝控制)
  - [进阶](#进阶)
    - [内存管理](#内存管理)
    - [模板](#模板)
    - [多线程入门](#多线程入门)
  - [轮子实战](#轮子实战)
    - [仿写STL模板库](#仿写stl模板库)
    - [简单线程池实现](#简单线程池实现)

## 基础

### 容器

- 顺序容器
  - 数组
  - 双端队列
  - 链表
- 容器适配器
  - 栈
  - 堆（优先队列）
  - 队列
- 关联容器
  - 分类
  - 基本操作
  - 底层
    - 散列表
    - 红黑树
  - 冲突解决
    - 开放定址
    - 链地址
    - 再哈希
- 泛型算法

### 类 （封装性）

- 构造函数和析构函数
  - 构造函数不能写成虚函数
  - 对有继承的情况，析构函数一般写为虚函数
- `this`指针
- 重载
- 访问修饰符
- 友类
- 静态成员和成员函数

### 继承和多态

- 继承修饰符
- 虚函数实现多态（父类指针指向子类对象）
  - 虚表
  - 纯虚函数
  - 重写

### 拷贝控制

- 拷贝构造函数 & 拷贝构造运算符
- 移动构造函数 & 移动构造运算符
  - 右值引用
- 析构函数

## 进阶

### 内存管理

- new和malloc
- 智能指针
  - shared_ptr & unique_ptr
    - shared_ptr自实现
  - RAII思想
- allocator类

### 模板

- 模板函数/模板类
- 萃取技术/SFINAE
- constexpr
- concept

### 多线程入门

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

## 轮子实战

### 仿写STL模板库

- [SimpleSTL](https://github.com/Zhytou/SimpleSTL.git)
  - 容器底层
    - 数组
    - 优先队列（堆排序）
    - 哈希表（开链法）
    - 红黑树
  - 迭代器底层
    - 模板、泛型编程
    - 参数包
    - traits技术

### 简单线程池实现

- [MultiThreadPracticeInCPP](https://github.com/Zhytou/MultiThreadPracticeInCPP.git)
  - std::mutex
  - std::condition_variable
  - std::thread
  - std::future
