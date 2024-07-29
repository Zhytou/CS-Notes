# 同步

- [同步](#同步)
  - [锁](#锁)
    - [互斥锁 Mutex](#互斥锁-mutex)
    - [自旋锁 Spinlock](#自旋锁-spinlock)
    - [快速用户空间互斥锁 Futex](#快速用户空间互斥锁-futex)
    - [死锁 Deadlock](#死锁-deadlock)
    - [活锁 Livelock](#活锁-livelock)
  - [条件变量](#条件变量)
  - [信号量](#信号量)
  - [信号](#信号)
  - [参考](#参考)

## 锁

锁是一种计算机同步技术。它能通过限制共享资源的同时访问，来保证多个进程的同步执行。具体来说，在进入临界区段（Critical Section）之前，程序被要求获取锁，否则无法修改共享资源。在这样的机制下，任何时候只能有一个进程持有锁进入临界区。

> 在同步的程序设计中，临界区段或称为关键区块指的是一个访问共享资源的程序片段。

互斥锁和自旋锁是最常见的两种锁。其中，前者会使等待该锁的程序陷入内核而被阻塞，直到上一个持有锁的程序执行完毕之后，操作系统才会选择任意一个等待该锁的程序唤醒。相反的，后者不会使程序陷入内核态，而是让其不断询问锁是否可用，即：在用户态下自旋。换句话说，互斥锁会使程序将CPU的控制权交还给操作系统，而自旋锁则不会。

### 互斥锁 Mutex

互斥锁（Mutual Exclusion，Mutex）是一种基于操作系统的同步原语，它使用了操作系统提供的系统调用来实现线程同步和互斥。

**睡眠-唤醒**：

互斥锁的实现通常是基于一种叫做“睡眠-唤醒”机制的方式。当一个线程需要获取锁时，如果锁已经被占用，那么锁会通过系统调用如sleep等阻塞这个线程，直到上一个持有锁的程序执行完毕。当锁被释放时，操作系统会唤醒任意一个等待的线程，让它获得锁，直到该锁的等待队列为空时，释放掉锁。

**适用场景**：

这种会使等待锁的线程陷入内核的特点决定了互斥量在锁的所有权频繁切换场景下不会适用，因为频繁陷入内核会使线程（特别是跨进程的线程）上下文切换的成本过高。

### 自旋锁 Spinlock

自旋锁是一种基于忙等待和原子操作的锁。当一个程序需要获取自旋锁时，如果锁已经被占用，那么这个程序会一直忙等待，直到锁被释放。而当程序获取到锁时，它会使用原子操作来修改锁的状态，以避免并发访问锁时的数据竞争问题。

**x86 LOCK指令前缀**：

以x86架构的机器为例，它所支持的LOCK指令前缀就是用于实现原子操作（Atomic Operation）的机制，比如：

- 原子交换：例如在实现Spinlock中的锁获取和释放操作时，可以使用XCHG指令和LOCK前缀来保证锁的原子性。
- 原子比较-交换：例如在实现无锁算法中，可以使用CMPXCHG指令和LOCK前缀来实现原子比较-交换操作。

**实现自旋锁**：

``` c++
// atomic exchange
int xchg(volatile int *addr, int newval) {
  int result;
  asm volatile ("lock xchg %0, %1"
    : "+m"(*addr), "=a"(result) : "1"(newval));
  return result;
}

int locked = 0;
void lock() { while (xchg(&locked, 1)) ; }
void unlock() { xchg(&locked, 0); }
```

**原子操作与总线**：

早期原子操作的实现依靠总线上锁来实现。具体如下，当一个处理器执行带有LOCK前缀的指令时，它会锁住内存总线，防止其他处理器同时对同一内存地址进行操作，从而确保了原子操作的正确性。

**原子操作的现代实现**：

现代处理器中，原子操作通常是通过硬件支持来实现的，它们能够保证对内存的原子操作，而不需要锁住总线或者使用其他的软件实现方式，从而提高了系统的性能和并发能力。

**自旋锁的缺陷**：

- 自旋 (共享变量) 会触发处理器间的缓存同步，延迟增加；
- 除了进入临界区的线程，其他处理器上的线程都在空转，且争抢锁的处理器越多，利用率越低；
- 获得自旋锁的线程可能被操作系统切换出去。

**自旋锁适合的使用场景**：

- 临界区几乎不 “拥堵”；
- 持有自旋锁时禁止执行流切换。

换句话说，自旋锁适用于短临界区的场景，比如操作系统内核的并发数据结构。

### 快速用户空间互斥锁 Futex

快速用户空间互斥锁（Fast Userspace Mutex，Futex）是一种基于用户空间的同步原语。它综合了互斥锁和自旋锁优点，将阻塞和非阻塞的等待方式结合在一起，使用户空间互斥锁获得最高效率。

**Futex的上锁过程**：

具体来说，对于一个尝试获取一个当前被占用锁的线程来说，短时间内，它自旋等待并不断检查锁的状态。如果足够快释放，则无需系统调用。若长时间锁未释放，则使用系统调用阻塞线程，避免占用过多CPU。

### 死锁 Deadlock

死锁(Deadlock)是多个竞争资源的进程因为顺序问题而陷入等待的状态，形成循环依赖。它的产生条件如下：

- 互斥：一个资源每次只能被一个进程使用
- 请求与保持：一个进程请求资阻塞时，不释放已获得的资源
- 不剥夺：进程已获得的资源不能强行剥夺
- 循环等待：若干进程之间形成头尾相接的循环等待资源关系

### 活锁 Livelock

活锁（Livelock）是另一种多个进程因资源竞争导致的等待。与死锁不同的是，死锁是进程都在等待对方先释放资源；而活锁则是进程彼此释放资源又同时占用对方释放的资源。当此情况持续发生时，尽管资源的状态不断改变，但每个进程都无法取得所需资源，使得事情没有任何进展。

## 条件变量

在多线程程序中，一个线程等待某些条件是很常见的。简单的方案是该线程自旋等待，不断检查该共享变量，直到条件满足。这显然是极其低效的，因为CPU会浪费在不断检查共享变量上。

条件变量（Condition Variable）为我们提供了一种高效机制来解决这个问题。条件变量维护着一个队列，当线程在执行条件不满足时，该线程会被阻塞并加入该队列。直到另外某个线程执行，使得条件成立后，条件变量便可以唤醒其队列中等待的一个或多个线程。

根据上面的描述，条件变量主要有两种相关操作：阻塞线程使其等待和唤醒线程。以C++模板库中的条件变量为例，这两个操作分别对应着下面两个函数。

```c++
// 等待
void wait( std::unique_lock<std::mutex>& lock );

// 唤醒
void notify_one() noexcept;
```

**虚假唤醒**：

虚假唤醒(Spurious Wakeup)指的是条件变量在并没有满足其条件的情况下才唤醒等待线程。它主要是由于实现上的限制，难以避免。大多数操作系统实现的条件变量都是存在一定概率的虚假唤醒。因此在使用条件变量时，记得要在等待时添加一个While循环，具体如下：

```c++
while(condition) {
  cv.wait(mtx);
}
```

**生产者-消费者**：

```c++
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <queue>
#include <thread>

using namespace std;

queue<int> q;
mutex mu;
condition_variable cond;

void producer(int& num) {
  while (num > 0) {
    int data = num--;
    unique_lock<mutex> locker(mu);
    q.push(data);
    cout << "producer sends " << data << endl;
    locker.unlock();
    cond.notify_one();
    this_thread::sleep_for(chrono::seconds(1));
  }
}

void consumer() {
  int data = 0;
  while (data != 1) {
    unique_lock<mutex> locker(mu);
    cond.wait(locker, []() { return !q.empty(); });
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

## 信号量

信号量（Semaphore）是一种同步对象。曾经提出过图论中著名的最短路径算法的迪杰斯特拉（Edsger Dijkstra）首次在同步有关的工作中引入了信号量，并把它作为其中唯一的同步原语，实现了类似锁和条件变量的机制。最终信号量广泛运用于各种操作系统中。

**信号量的机制**：

简单来说，我们可以把信号量理解为一个整数计数器。在实际应用中，我们给予每个进程一个信号量，以标识进程当前的状态。信号量允许进程对它做两种原子操作:

- P操作：将信号量的值减一。如果值减为负数，则进程将被阻塞，等待信号量的值增加。
- V操作：将信号量的值加一。如果有进程被阻塞等待信号量值，则唤醒其中一个。

因此，如果将一个信号量初始化为10，那么最多有10个进程可以执行P操作（即调用sem_wait()函数）获取信号量以及被该信号量保护的资源，超过10个的进程会被阻塞等待。通过这种方法，共享资源可以一定限度的被并发访问，而不是像互斥量一样每个时刻仅一个进程能访问该资源。

**互斥锁和信号量**：

当我们对信号量的上限进行限制，使其成为一个只有0或1两种状态的二值信号量（Binary Semaphore）时，它的作用就和互斥锁非常类似了。换句话说，互斥锁就是一种特殊的信号量。

**条件变量和信号量**：

使用信号量也能达成条件变量的功能。以一个线程创建另外一线程，并且等待它结束为例。我们可以将在父线程中，初始化信号量并将其设置为0，接着创建子线程，最后执行信号量P操作。其中，子线程只需在完成自身工作后，执行该信号量的P操作即可。

因此，整个流程则是：父线程首先运行并创建子线程，但因为执行了P操作后信号量小于0而被阻塞。此时，子线程在完成自身工作后，会执行V操作使得信号量值大于0而唤醒父线程，从而最终达到一个线程创建另外一线程，并且等待它结束的目的。

**使用信号量**：

信号量的函数都以sem_开头，它们都声明在头文件semaphore.h中。其中，最常使用的信号量函数有4个，分别是：

- sem_init函数：用于创建信号量。具体来说，初始化由指向的信号对象，设置它的共享选项，并给它一个初始的整数值。其原型如下`int sem_init(sem_t *sem,int pshared,unsigned int value)`。其中，sem指向信号量对象；pshared控制信号量的类型，如果其值为0，就表示这个信号量是当前进程的局部信号量，否则信号量就可以在多个进程之间共享；value为sem的初始值。整个函数在调用成功时返回0，反之返回-1。
- sem_wait函数：用于以原子操作的方式将信号量的值减1。
- sem_post函数：该函数用于以原子操作的方式将信号量的值加1。
- sem_destroy函数：该函数用于对用完的信号量的清理。

```c++
int main() {
  sem_t semaphore;
  sem_init(&semaphore, 0, 1); // 初始化信号量

  pid_t pid = fork();

  if (pid == 0) {
      // 子进程
      sem_wait(&semaphore); // 等待获取信号量
      printf("Child process: I got the semaphore\n");
      sem_post(&semaphore); // 释放信号量
  } else {
      // 父进程
      sem_wait(&semaphore); // 等待获取信号量
      printf("Parent process: I got the semaphore\n");
      sem_post(&semaphore); // 释放信号量

      wait(NULL); // 等待子进程结束
  }

  sem_destroy(&semaphore); // 销毁信号量

  return 0;
}
```

## 信号

信号（Signals）是Unix/Linux操作系统中进程间通讯的一种有限制的方式。它是一种异步的通知机制，用来提醒进程一个事件已经发生。

## 参考

- [Operating Systems: Three Easy Pieces: Locks](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/28.pdf)
- [Operating Systems: Three Easy Pieces: Condition Variables](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/30.pdf)
- [Operating Systems: Three Easy Pieces: Semaphores](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/31.pdf)
