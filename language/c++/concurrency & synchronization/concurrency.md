# 并发 & 同步

- [并发 \& 同步](#并发--同步)
  - [并发](#并发)
  - [同步](#同步)
    - [锁 lock](#锁-lock)
    - [信号量 semaphore](#信号量-semaphore)
    - [管程 monitor](#管程-monitor)
    - [条件变量 condition variable](#条件变量-condition-variable)
    - [原子操作 atomic operation](#原子操作-atomic-operation)
  - [底层](#底层)
  - [参考](#参考)

## 并发

> 对于并发的详细介绍可以阅读另一篇笔记[OS-Concurrency](https://github.com/Zhytou/CS-Notes/blob/main/field/operating%20system/concurrency.md)

**POSIX threads vs std::threads**：

POSIX threads是在Unix/Linux系统中广泛使用的线程API，它是由IEEE POSIX标准化组织定义的，因此可以在符合POSIX标准的系统上使用。而std::threads是C++11标准定义的线程库，只能在支持C++11标准的编译器上使用。

**volatile vs atomic**：

如果一个基本变量被volatile修饰，编译器将不会把它保存到寄存器中，而是每一次都去访问内存中实际保存该变量的位置上。

## 同步

**临界区 critical section**：

在同步的程序设计中，临界区段（Critical section）或称为关键区块 ，指的是一个访问共享资源（例如：共享设备或是共享存储器）的程序片段，而这些共享资源又无法同时被多个线程访问的特性。

**同步原语 synchronization primitive**：

同步原语是一种计算机编程中的基本构造，用于实现多个线程或进程之间的同步和互斥。

它们可以用于协调多个线程或进程的执行顺序和访问共享资源的方式，避免不同线程或进程之间的竞争和冲突。

### 锁 lock

> 关于锁更细致的介绍可以阅读我的这篇博客[南大OS学习笔记——5 并发控制：互斥](https://zhytou.top/post/2023-6-19/nju-os/)

**互斥量 mutex**:

互斥锁（mutex）是一种基于操作系统的同步原语，它使用了操作系统提供的系统调用来实现线程同步和互斥。互斥锁的实现通常是基于一种叫做“睡眠-唤醒”机制的方式，当一个线程需要获取锁时，如果锁已经被占用，那么这个线程会被阻塞，直到锁被释放。当锁被释放时，操作系统会唤醒等待的线程，让它们竞争锁。

**自旋锁 spin lock**:

自旋锁（spinlock）是一种基于忙等待的锁，它不涉及系统调用，而是使用了一种忙等待的方式来实现线程同步和互斥。当一个线程需要获取自旋锁时，如果锁已经被占用，那么这个线程会一直忙等待，直到锁被释放。自旋锁的实现通常是基于原子操作的方式，当一个线程获取锁时，它会使用原子操作来修改锁的状态，以避免并发访问锁时的数据竞争问题。

**快速用户空间锁 futex**:

快速用户空间互斥锁（Fast Userspace Mutex，Futex）是一种基于用户空间的同步原语。它综合了互斥锁和自旋锁优点，将阻塞和非阻塞的等待方式结合在一起，使用户空间互斥锁获得最高效率。

对于一个尝试获取一个被占用的futex锁的线程来说，短时间内，它自旋等待并不断检查锁的状态。如果足够快释放，则无需系统调用。若长时间锁未释放，则使用系统调用阻塞线程，避免占用过多CPU。

### 信号量 semaphore

信号量又称为信号标，是一个同步对象，用于保持在0至指定最大值之间的一个计数值。

在系统中，给予每一个进程一个信号量，代表每个进程目前的状态。其中，信号量为0时进程休眠；信号量大于0时进程就绪或运行。

### 管程 monitor

管程是由进程、变量及数据结构等组成的一个集合，它们组成一个特殊的模块或软件包。

管程提供了一种机制，线程可以临时放弃互斥访问，等待某些条件得到满足后，重新获得执行权恢复它的互斥访问。

### 条件变量 condition variable

管程中机制的实现方法。

> C++只提供了两种同步原语——互斥量和条件变量。

### 原子操作 atomic operation

原子操作：顾名思义就是不可分割的操作，该操作只存在未开始和已完成两种状态，不存在中间状态；

原子类型：原子库中定义的数据类型，对这些类型的所有操作都是原子的，包括通过原子类模板`std::atomic<T>`实例化的数据类型，也都是支持原子操作的。

**读-修改-写read-modify-write**是计算机科学中的一个原子操作（atomic operation，类似的还有test-and-set, fetch-and-add, compare-and-swap等），操作过程是读一个内存位置（或IO端口），修改其值，再写回原位置。

**比较并交换compare and swap, CAS**是原子操作的一种，可用于在多线程编程中实现不被打断的数据交换操作，从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题。 该操作通过将内存中的值与指定数据进行比较，当数值一样时将内存中的数据替换为新的值。

## 底层

锁实际上能通过底层、操作系统和软件实现。

- 硬件层面：CPU提供atomic的指令（lock前缀）
- 软件层面：[Peterson‘s Algorithm](https://en.wikipedia.org/wiki/Peterson's_algorithm)实现
- 操作系统：spin lock或者futex

## 参考

**同步原语**:

- [synchronization primitives](https://stackoverflow.com/questions/8017507/definition-of-synchronization-primitive)
- [monitor vs condition variable](https://stackoverflow.com/questions/31331724/monitor-and-conditional-variable-are-they-the-same)
- [现代操作系统 2.3 进程间的通信](file:///D:/ZhY/Workspace/Notes/repick_note/basical/operating%20system/%E7%8E%B0%E4%BB%A3%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%20%20%E5%8E%9F%E4%B9%A6%E7%AC%AC4%E7%89%88[%E9%AB%98%E6%B8%85%E6%89%AB%E6%8F%8F%E7%89%88].pdf)

**底层**:

- [stackoverflow-how are mutexes implemented](https://stackoverflow.com/questions/1485924/how-are-mutexes-implemented)
- [知乎-互斥锁的底层原理是什么？](https://www.zhihu.com/question/332113890)
