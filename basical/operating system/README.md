# 操作系统

- [操作系统](#操作系统)
  - [并发](#并发)
  - [存储管理](#存储管理)
  - [同步](#同步)
  - [持久化](#持久化)
  - [其他](#其他)
  - [参考](#参考)

[FOLDOC](http://foldoc.org/)中关于操作系统的定义是`The low-level software which handles the interface to peripheral hardware, schedules tasks, allocates storage, and presents a default interface to the user when no application program is running.`简单点说，操作系统是一个管理和控制计算机硬件的底层软件。站在软件和用户的角度，操作系统提供了可操作的对象和可调用的API。比如：它将持久化设备抽象成文件，并提供了读写接口；它将CPU中的可执行流抽象成进程，并提供了执行、复制和终止接口。而站在硬件的角度，操作系统则仅仅是一些列C程序。

> FOLDOC（Free Online Dictionary Of Computing）是互联网上最知名也最古老的计算机科学与相关主题词典之一。它收录了大量计算机科学、电子工程、通信技术等相关领域的名词、术语和概念的定义。

尽管操作系统是一门更关注软件层面的学科，但学习它时，为了更好的理解其中部分设计原理，必然也会引入部分体系结构相关的知识，比如：指令集体系结构（ISA）、存储层级（Memory Hierachy）等。因此，本笔记参考[OSTEP](https://github.com/remzi-arpacidusseau/ostep-translations/tree/master/chinese)将操作系统的相关功能以及背后的原理分成并发、存储管理、同步、持久化和其他五部分。

## 并发

- 进程：虚拟化的CPU
  - 进程的定义
  - 进程API：fork/execve/exit
  - 进程间的通信
  - CPU调度
- 线程
  - 线程模型
  - POSIX线程API：pthread_create/pthread_join/pthread_exit
- 协程
- 事件驱动模型

## 存储管理

> 虚拟化的思想在操作系统中有大量的体现，比如：进程就是一种虚拟化CPU的方法，而文件\目录则是一种虚拟化块设备的方法。此外，最常见的虚拟化则是虚拟内存技术。

- 虚拟内存技术
  - 逻辑地址 & 物理地址
  - MMU
- 分段
  - 泛化的的基址/界限
  - 外部碎片
- 分页
  - 页表/多级页表
  - TLB
  - 内部碎片
- 存储层级
  - 交换空间
  - 缺页

## 同步

- 锁
  - Mutex/Spin Lock/Futex
  - Dead Lock
- 条件变量
  - 虚假唤醒
  - 生产者-消费者
- 信号量
- 信号

## 持久化

- I/O设备：块设备和字符设备
  - 总线
  - 轮询 & 中断
  - DMA
  - 设备控制器 & 驱动
    - 驱动：操作系统的一部分，将操作系统给上层应用提供的文件操作（read/write/ioctl）翻译成设备控制协议
  - I/O复用（并发处也同样涉及到）
- 文件/目录：虚拟化的块设备
  - 文件/目录API
  - 文件系统（File System）

## 其他

- 虚拟机 & 容器
- 计算机启动流程
  - ROM Stage
  - RAM Stage
  - Find Something to boot Stage
- 编译 & 链接
  - 编译流程
  - 编译器前端 & 后端
  - 动态编译 ELF
- libc/GNU

## 参考

- [NJU-OS](https://jyywiki.cn/OS/2022/)
- [Operating Systems: Three Easy Pieces](https://github.com/remzi-arpacidusseau/ostep-translations/tree/master/chinese)
- [CSAPP: Computer System A Programmer's Perspective](https://hansimov.gitbook.io/csapp/)
