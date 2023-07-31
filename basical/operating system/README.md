# 操作系统

- [操作系统](#操作系统)
  - [并发](#并发)
  - [虚拟内存](#虚拟内存)
  - [同步](#同步)
  - [持久化](#持久化)
  - [其他](#其他)
  - [参考](#参考)

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

## 虚拟内存

> 虚拟化的思想在操作系统中有大量的体现，比如：进程就是一种虚拟化CPU的方法，而文件\目录则是一种虚拟化块设备的方法。此外，最常见的虚拟化则是虚拟内存技术。

- 存储器
  - RAID（Redundant Array of Inexpensive Disks）
  - 存储层次结构（Memory Hierarchy）
- 地址空间：虚拟化的内存（分段）
- 分页
  - （多级）页表 慢表
  - 快表 TLB
  - 交换空间
  - 缓存替换策略

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
