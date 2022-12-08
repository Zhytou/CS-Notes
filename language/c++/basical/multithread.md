# C++多线程编程

- [C++多线程编程](#c多线程编程)
  - [简介](#简介)
    - [Unix process vs Unix thread](#unix-process-vs-unix-thread)
    - [并发 concurrency vs 并行 parallelism](#并发-concurrency-vs-并行-parallelism)
    - [Posix threads vs std::threads](#posix-threads-vs-stdthreads)
    - [Lock-free vs Lock-based Thread Synchronization](#lock-free-vs-lock-based-thread-synchronization)
    - [RMW: Atomic Read-Modify-Write Operations](#rmw-atomic-read-modify-write-operations)
    - [CAS:Compare-And-Swap Loop](#cascompare-and-swap-loop)
    - [volatile vs atomic](#volatile-vs-atomic)
    - [内存栅栏](#内存栅栏)
  - [概念](#概念)
    - [同步原语 synchronization primitive](#同步原语-synchronization-primitive)
      - [临界区 critical section](#临界区-critical-section)
      - [互斥量 mutex](#互斥量-mutex)
      - [自旋锁 spin lock](#自旋锁-spin-lock)
      - [信号量 semaphore](#信号量-semaphore)
      - [管程 monitor](#管程-monitor)
      - [条件变量 condition variable](#条件变量-condition-variable)
    - [原子操作 atomic](#原子操作-atomic)
  - [底层](#底层)
    - [mutex底层](#mutex底层)
    - [atomic底层](#atomic底层)
  - [基础](#基础)
    - [线程管理 Thread Management](#线程管理-thread-management)
      - [thread](#thread)
    - [有锁编程 Lock-based Coding](#有锁编程-lock-based-coding)
      - [mutex](#mutex)
      - [lazy initialization](#lazy-initialization)
      - [condition\_variable](#condition_variable)
    - [无锁编程 Lock-free Coding](#无锁编程-lock-free-coding)
      - [atomic](#atomic)
    - [异步编程 Async Coding](#异步编程-async-coding)
      - [future](#future)
  - [实战](#实战)
    - [锁](#锁)
      - [概述](#概述)
    - [条件变量](#条件变量)
    - [计数器](#计数器)
    - [线程池](#线程池)
    - [无锁队列](#无锁队列)
    - [无锁栈](#无锁栈)
    - [生产-消费问题](#生产-消费问题)
  - [参考](#参考)

## 简介

### Unix process vs Unix thread

**进程process**:

- 进程是资源分配的基本单位。

- 一个进程就是一个正在执行程序的实例。
- 一个进程包含了：
  - Process ID, process group ID, user ID, and group ID
  - Environment
  - Working directory
  - Program instructions
  - Registers
  - Stack
  - Heap
  - File descriptors
  - Signal actions
  - Shared libraries
  - Inter-process communication tools (such as message queues, pipes, semaphores, or shared memory).

**线程thread**:

- 线程是独立调度的基本单元。
- 一个线程包含了
  - Stack pointer
  - Registers
  - Scheduling properties (such as policy or priority)
  - Set of pending and blocked signals
  - Thread specific data.

### 并发 concurrency vs 并行 parallelism

Concurrency and parallelism often get mixed up, but it’s important to understand the difference. In parallelism, we run multiple copies of the same program simultaneously, but they are executed on different data.

For example, you could use parallelism to send requests to different websites but give each copy of the program a different set of URLs. These copies are not necessarily in communication with each other, but they are running at the same time in parallel.

As we explained above, concurrent programming involves a shared memory location, and the different threads actually “read” the information provided by the previous threads.

### Posix threads vs std::threads

### Lock-free vs Lock-based Thread Synchronization

**Lock-based**:

The lock-based synchronization is a historically popular way to avoid race conditions. The idea is simple. The segments of code where shared resources are accessed are identified as critical sections. A critical section is protected by a lock which ensures that only one thread can access the shared resource at a given time. To access the shared resource, a thread should acquire the lock first. The thread will release the lock when it no longer requires access to the shared resource. If another thread tries to acquire a lock that has already been acquired, the thread will be blocked (put to sleep) by the system.

**Lock-free**:

As the name implies, the idea is to synchronize threads without the use of locks. Lock-free synchronization relies on atomic instructions and hardware support.

**summary**:

有锁编程利用互斥量来实现，无锁编程利用原子操作来实现。

其中，C++中的std::atomic原子操作可能是由CPU提供的原子指令来支持，也可能是由操作系统中的spinlock来支持，具体可以根据std::atomic::is_lock_free()去判断。

### RMW: Atomic Read-Modify-Write Operations

**读-修改-写**（read-modify-write）是计算机科学中的一个原子操作（atomic operation，类似的还有test-and-set, fetch-and-add, compare-and-swap等），操作过程是读一个内存位置（或IO端口），修改其值，再写回原位置。

### CAS:Compare-And-Swap Loop

**比较并交换(compare and swap, CAS)**是原子操作的一种，可用于在多线程编程中实现不被打断的数据交换操作，从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题。 该操作通过将内存中的值与指定数据进行比较，当数值一样时将内存中的数据替换为新的值。

### volatile vs atomic

如果一个基本变量被volatile修饰，编译器将不会把它保存到寄存器中，而是每一次都去访问内存中实际保存该变量的位置上。

### 内存栅栏

## 概念

### 同步原语 synchronization primitive

synchronization primitives are simple software mechanisms provided by a platform (e.g. operating system) to its users for the purposes of supporting thread or process synchronization.

#### 临界区 critical section

在同步的程序设计中，临界区段（Critical section）或称为关键区块 ，指的是一个访问共享资源（例如：共享设备或是共享存储器）的程序片段，而这些共享资源又无法同时被多个线程访问的特性。

#### 互斥量 mutex

互斥量是一种用于多线程编程中，防止两条线程同时对同一公共资源（比如全局变量）进行读写的机制。一般来说，我们也将其称之为互斥锁。

在系统中，一个线程在没有竞争到它所需要的互斥量时，就会进入休眠状态，因此，我们也常常将这种锁称为`sleep lock`。

#### 自旋锁 spin lock

自旋锁其实是上面提到的互斥量中的一种，不同之处在于没竞争到自旋锁的线程会等待而非休眠。

> Spin mutexes are a simple spin lock. If the lock is held by another thread when a thread tries to acquire it, the second thread will spin waiting for the lock to be released.
> Due to this spinning nature, a context switch cannot be performed while holding a spin mutex to avoid deadlocking in the case of a thread owning a spin lock not being executed on a CPU and all other CPUs spinning on that lock.

#### 信号量 semaphore

信号量又称为信号标，是一个同步对象，用于保持在0至指定最大值之间的一个计数值。

在系统中，给予每一个进程一个信号量，代表每个进程目前的状态。其中，信号量为0时进程休眠；信号量大于0时进程就绪或运行。

#### 管程 monitor

管程是由进程、变量及数据结构等组成的一个集合，它们组成一个特殊 的模块或软件包。

管程提供了一种机制，线程可以临时放弃互斥访问，等待某些条件得到满足后，重新获得执行权恢复它的互斥访问。

#### 条件变量 condition variable

管程中机制的实现方法。

> C++只提供了两种同步原语——互斥量和条件变量。

### 原子操作 atomic

原子操作：顾名思义就是不可分割的操作，该操作只存在未开始和已完成两种状态，不存在中间状态；

原子类型：原子库中定义的数据类型，对这些类型的所有操作都是原子的，包括通过原子类模板std::atomic< T >实例化的数据类型，也都是支持原子操作的。

## 底层

- 锁实际上能通过底层、操作系统和软件实现。

- 硬件层面：CPU提供atomic的指令（lock前缀）

- 软件层面：[Peterson‘s Algorithm](https://en.wikipedia.org/wiki/Peterson's_algorithm)实现

- 操作系统：spin lock或者futex

### mutex底层

### atomic底层

## 基础

### 线程管理 Thread Management

- [Youtube - Bo Qian - C++ 11 Concurrent](https://www.youtube.com/watch?v=LL8wkskDlbs&list=PL5jc9xFGsL8E12so1wlMS0r0hTQoJL74M&index=2)

#### thread

> 特别注意由于pthread并非默认库，所以编译时要手动将pthread链接进去，具体指令为 g++  xxx.cpp -l pthread -o xxx

**std::thread**:

thread主要的构造函数如下：

``` c++
thread();
thread(thread&& other);
thread(const thread& other) = delete;
thread(Function f, Args args);
```

函数1）创建一个thread对象；

函数2）为thread的移动构造函数；

- Constructs the thread object to represent the thread of execution that was represented by `other`. After this call `other` no longer represents a thread of execution.

函数3）说明thread 的拷贝构造函数被禁用

- thread是不能拷贝的，只能用`std::move`将一个线程移动到另一个线程对象中去。

函数4）最常见的线程创建方式，利用函数和调用该函数的参数创建。

- 注意，这种创建方式值传参的，想要引用传参必须使用`std::ref`，具体如下：

  ``` c++
  void func(int i);
  int num;
  //create by value
  thread t1(func, num);
  //create by reference
  thread t2(func, ref(num))
  ```

**std::thread::join() vs std::thread::detach()**:

On the one hand, `std::thread::join()` allow parent thread to wait for the child thread to finish its execution;

On the other hand, `std::thread::detach()` allow child thread to excute independently.

**std::thread::swap(T&, T&) vs std::move()** ：

因为thread是不能复制构造的，所以只能使用右值进行转移，比如使用`std::swap`或`std::move`。

``` c++
void func1();
void func2();

thread t1(func1), t2(func2);

cout << "thread 1 id: " << t1.get_id() << '\n'
     << "thread 2 id: " << t2.get_id() << '\n';

swap(t1, t2);

cout << "after std::swap(t1, t2):" << '\n'
     << "thread 1 id: " << t1.get_id() << '\n'
     << "thread 2 id: " << t2.get_id() << '\n';
```

运行这段代码可以看到，交换后两者id也变了，即完成了thread交换。

``` c++
t1 = move(t2);
```

### 有锁编程 Lock-based Coding

#### mutex

**std::mutex::lock() vs std::lock()**:

`std::mutex::lock()`是`std::mutex`类中的一个成员函数，只能被`std::mutex`对象调用，当调用对象可以上锁，则成功锁住，否则发生死锁，所以一般不建议使用这个函数；

而`std::lock()`则是`<mutex>`库中的一个函数模板，可以一次锁多个`mutex`，当调用对象全部都可以上锁，则成功锁住，否则解锁全部调用对象，这样能有效避免死锁，不过可以用`scoped_lock`替代。具体例子如下：

``` c++
mutex mtx1, mtx2;
unique_lock<mutex> lk1(mtx1, defer_lock), lk2(mtx2, defer_lock);
//may have a deadlock problem
lock(lk1), lock(lk2);

//no deadlock problem
lock(lk1, lk2);
```

**std::mutex vs std::recursive_mutex**:

一个线程尝试多次获取`std::mutex`会导致死锁，但一个线程却可以多次获取`std::recursive_mutex`而不出现问题。

**std::lock_guard vs std::unique_lock**:

首先，上述二者都是基于RAII思想的管理互斥量的“锁”，都能够代替互斥量的`lock()`和`unlock()`，并且都能有效解决资源忘记释放的问题；

其次，二者的区别在于`std::lock_guard`只能上锁一次，即`std::lock_guard`在创建时上锁，析构时解锁;而对于`std::unique_lock`，它能够多次上锁解锁，这也是条件变量一般与它绑定的原因。

此外，`std::unique_lock`还可以使用`std::move`进行移动，而`std::lock_guard`则不行。

总的来说，`std::unique_lock`有更高的灵活性，但也要付出更性能损耗；与之相反的，`std::lock_guard`损耗更低，但灵活性也更差。

**std::scoped_lock**:

> C++17 also provides `scoped_lock` for the specific purpose of locking multiple mutexes that prevents deadlock in a RAII style, similar to `lock_guard`.

简单来说，`std::scoped_lock`就是`std::lock_guard`的升级版，它能够以RAII的方式管理多个`std::mutex`。

**std::adpot_lock & std::defer_lock**:

`std::defer_lock`一般和`std::unique_lock`一起使用，表示初始化`std::unique_lock`对象时，对应的`mutex`不用立刻上锁。

> We first create the lock without acquiring it (that's the point of `std::defer_lock`) 

``` c++
mutex mtx;
uniuqe_lock<mutex> locker(mtx, defer_lock);
```

`std::adopt_lock`则一般和`std::lock_guard`一起使用，表示初始化`std::lock_guard`对象时，对应的`mutex`已经上锁。

> Now, here we first acquire the locks (still avoiding deadlocks), and *then* we create the lockguards to make sure that they are properly released.
>
> Note that `std::adopt_lock` requires that the current thread owns the mutex (which is the case since we just locked them)

``` c++
mutex mtx1, mtx2;
std::lock(mtx1, mtx2);
lock_guard<mutex> lk1(mtx1, adopt_lock), lk2(mtx2, adopt_lock);
```

#### lazy initialization

**std::once_flag**

#### condition_variable

> 例子可以参考实战中的生产-消费问题

**std::condition_variable**

`std::condition_variable`的初始化并不需要`std::mutex`或者`mutex`RAII的包装类，而是无需传其他参数就可以直接使用。

**std::condition_variable::notify_one() vs std::condition_variable::notify_all()**

`std::condition_variable::notify_one()`和`std::condition_variable::notify_all()`的唯一区别就是：前者仅唤醒一个关联该条件变量的线程，而后者唤醒所有这种线程。

这两个函数都只能唤醒线程，释放锁仍然需要该线程的`std::unique_lock`对象手动调用`std::unique_lock::unlock()`函数释放；否则唤醒后的线程会因为无法获取所需的锁，而再次进入睡眠状态。

**std::condition_variable::wait(std::unique_lock lock, function predicate)**

`std::condition_variable::wait()`函数一定要接收第一个锁参数，而第二个预测函数则是可选参数，其作用是避免线程虚假苏醒`(suprious wakeup)`，仅当该参数值为真时，线程苏醒；否则，线程继续休眠。

> 虚假唤醒

### 无锁编程 Lock-free Coding

#### atomic

> 一般来说，atomic都是不可拷贝的。
>
> 这是因为有些不支持原子指令的硬件平台必须要通过锁来实现原子操作，而锁又是无法拷贝的。因此，atomic是无法拷贝的

**std::memory_order**

``` c++
typedef enum memory_order {
    memory_order_relaxed,    // 不对执行顺序做保证
    memory_order_acquire,    // 本线程中,所有后续的读操作必须在本条原子操作完成后执行
    memory_order_release,    // 本线程中,所有之前的写操作完成后才能执行本条原子操作
    memory_order_acq_rel,    // 同时包含 memory_order_acquire 和 memory_order_release
    memory_order_consume,    // 本线程中,所有后续的有关本原子类型的操作,必须在本条原子操作完成之后执行
    memory_order_seq_cst    // 全部存取都按顺序执行
    } memory_order;
```

设置指令的执行顺序。

**std::atomic::is_lock_free()**

返回该变量是否无锁。无锁变量不会导致其他线程访问阻塞。

### 异步编程 Async Coding

#### future

**std::async**

**std::future**

## 实战

参考 陈硕 muduo C++网络库的代码进行的一些总结。

> muduo库是一个基于POSIX thread的非阻塞事件驱动型C++网络库。

### 锁

#### 概述

*MutexLock* is a lockable object;

*MutexLockGuard* is a object that manages *MutexLock object* by keeping it locked. It guarantees the *MutexLock object* is properly unlocked in case an exception is thrown.

> 实际上，MutexLock就对应std::mutex；MutexLockGuard就对应std::lock_guard。

### 条件变量

### 计数器

### 线程池

### 无锁队列

``` c++
template<typename T>
class lock_free_queue
{
private:
    struct node
    {
        std::shared_ptr<T> data;
        std::atomic<node*> next;
        node(T const& data_):
            data(std::make_shared<T>(data_))
        {}
    };
    std::atomic<node*> head;
    std::atomic<node*> tail;
public:
    void push(T const& data)
    {
        std::atomic<node*> const new_node=new node(data);
        node* old_tail = tail.load();
        while(!old_tail->next.compare_exchange_weak(nullptr, new_node)){
          node* old_tail = tail.load();
        }
        tail.compare_exchange_weak(old_tail, new_node);
    }
    std::shared_ptr<T> pop()
    {
        node* old_head=head.load();
        while(old_head &&
            !head.compare_exchange_weak(old_head,old_head->next)){
            old_head=head.load();
        }
        return old_head ? old_head->data : std::shared_ptr<T>();
    }
};

```

### 无锁栈

### 生产-消费问题

``` c++
#include<iostream>
#include<queue>
#include<thread>
#include<mutex>
#include<condition_variable>

using namespace std;

queue<int> q;
mutex mu;
condition_variable cond;

void producer(int& num) {
    while (num > 0){
        int data = num--;
        unique_lock<mutex> locker(mu);
        q.push(data);
        cout << "producer sends " << data << endl;
        locker.unlock();
        cond.notify_one();
        this_thread::sleep_for(chrono::seconds(1));
    }
}

void consumer () {
    int data = 0;
    while (data != 1) {
        unique_lock<mutex> locker(mu);
        cond.wait(locker, []() {
            return !q.empty();
        });
        data = q.front();
        q.pop();
        cout << "consumer gets " << data << endl;
    }
}

int main() {
    int data_num = 5;
    cout << "before the process, data num = " << data_num << endl;
    thread t1(producer, ref(data_num)), t2(consumer);
    t1.join();
    t2.join();
    cout << "after the process, data num = " << data_num << endl;
    return 0;
}
```

![消费者](../img/producer_consumer_example.png)

## 参考

**thread pthread & process**:

- [A tutorial on mdern multithreadiing and concurrency in C++](https://www.educative.io/blog/modern-multithreading-and-concurrency-in-cpp)
- [POSIX Threads Programming](https://hpc-tutorials.llnl.gov/posix/)
- [What is a Thread?](https://hpc-tutorials.llnl.gov/posix/what_is_a_thread/)
- [c++11 std::threads vs posix threads](https://stackoverflow.com/questions/13134186/c11-stdthreads-vs-posix-threads)
- [陈硕关于thread or pthread的回答](https://www.zhihu.com/question/24109413/answer/26777678)

**Lock free vs Lock based**:

- [Lock-based vs Lock-free Thread Synchronization](https://medium.com/geekculture/lock-based-vs-lock-free-thread-synchronization-cbae710a8ab9)
- [原子操作与无锁编程](https://www.hiyu.space/2021/02/20/C-%E5%A4%9A%E7%BA%BF%E7%A8%8B%EF%BC%88%E4%BA%94%EF%BC%89-%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C%E4%B8%8E%E6%97%A0%E9%94%81%E7%BC%96%E7%A8%8B/)
- **[An introduction to Lock-Free Programming](https://preshing.com/20120612/an-introduction-to-lock-free-programming/)**

**RMW**:

- [read-modify-write](https://zh.wikipedia.org/wiki/Read-modify-write)

**CAS**:

- [比较并交换 cas](https://zh.wikipedia.org/wiki/%E6%AF%94%E8%BE%83%E5%B9%B6%E4%BA%A4%E6%8D%A2)

**TSL**:

- [Test and set lock](https://zh.wikipedia.org/wiki/%E6%A3%80%E6%9F%A5%E5%B9%B6%E8%AE%BE%E7%BD%AE)

**同步原语**:

- [synchronization primitives](https://stackoverflow.com/questions/8017507/definition-of-synchronization-primitive)
- [monitor vs condition variable](https://stackoverflow.com/questions/31331724/monitor-and-conditional-variable-are-they-the-same)
- [现代操作系统 2.3 进程间的通信](file:///D:/ZhY/Workspace/Notes/repick_note/basical/operating%20system/%E7%8E%B0%E4%BB%A3%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%20%20%E5%8E%9F%E4%B9%A6%E7%AC%AC4%E7%89%88[%E9%AB%98%E6%B8%85%E6%89%AB%E6%8F%8F%E7%89%88].pdf)

**spin lock vs sleep lock(mutex)**:

- [Synchronization Primitives](https://www.usenix.org/legacy/publications/library/proceedings/bsdcon02/full_papers/baldwin/baldwin_html/node5.html)

**mutex & atomic noncopyable**:

- [stackoverflow- why is mutex noncopyable or movable?](https://stackoverflow.com/questions/62369119/why-is-stdmutex-neither-copyable-nor-movable)
- [stackoverflow- why is atomic noncopybale?](https://stackoverflow.com/questions/15249998/why-are-stdatomic-objects-not-copyable)

**底层**:

- [stackoverflow-how are mutexes implemented](https://stackoverflow.com/questions/1485924/how-are-mutexes-implemented)
- [知乎-互斥锁的底层原理是什么？](https://www.zhihu.com/question/332113890)

**C++11 STL**:

- [Youtube - Bo Qian - C++ 11 Concurrent](https://www.youtube.com/watch?v=LL8wkskDlbs&list=PL5jc9xFGsL8E12so1wlMS0r0hTQoJL74M&index=2)
- [mutex vs recursive_mutex](https://stackoverflow.com/questions/14498892/stdmutex-vs-stdrecursive-mutex-as-class-member)
- [std::lock() vs std::mutex::lock()](http://www.cplusplus.com/reference/mutex/lock/)
- [unique_lock vs lock_guard](https://stackoverflow.com/questions/20516773/stdunique-lockstdmutex-or-stdlock-guardstdmutex)
- [stackoverlfow - locking multiple mutexes](https://stackoverflow.com/questions/13483767/locking-multiple-mutexes)
- [cppreference 17 scoped_lock](https://en.cppreference.com/w/cpp/thread/scoped_lock/scoped_lock)
- [stackoverflow - adopt_lock vs defer_lock](https://stackoverflow.com/questions/27089434/whats-the-difference-between-first-locking-and-creating-a-lock-guardadopt-lock)

**实战**

- [muduo](https://github.com/chenshuo/muduo)

**无锁队列**

- [Lock Free Queue](https://jbseg.medium.com/lock-free-queues-e48de693654b)
