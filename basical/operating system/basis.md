# 操作系统

- [操作系统](#操作系统)
  - [虚拟化](#虚拟化)
  - [并行](#并行)
  - [持久化](#持久化)
  - [其他](#其他)
  - [参考](#参考)

## 虚拟化

- 进程：虚拟化的CPU
  - 进程的定义
  - 进程API：fork/execve/exit
  - 进程间的通信
- CPU调度
- 地址空间：虚拟化的内存
- 分段&分页

## 并行

- 线程
  - 线程模型
  - POSIX线程API：pthread_create/pthread_join/pthread_exit
- 锁&条件变量
  - Mutex/Spin Lock/Futex
  - Dead Lock
  - Producer-Consumer
- 信号量

## 持久化

- I/O设备：块设备、字符设备和网络设备
  - 驱动：将操作系统给上层应用提供的文件操作（read/write/ioctl）翻译成设备控制协议
  - 持久化设备
    - RAID（）
    - 存储层次结构（Memory Hierarchy）
- 文件/目录：虚拟化的块设备
  - 文件/目录API
  - 文件系统（File System）

## 其他

- 虚拟机
- 管程
- 计算机启动流程
  - ROM Stage
  - RAM Stage
  - Find Something to boot Stage
- 编译
- libc/GNU

## 参考

- [NJU-OS](https://jyywiki.cn/OS/2022/)
- [Operating Systems: Three Easy Pieces](https://github.com/remzi-arpacidusseau/ostep-translations/tree/master/chinese)
