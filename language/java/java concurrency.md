# Java Concurrency

- [Java Concurrency](#java-concurrency)
  - [Concurency Basis](#concurency-basis)
    - [Concurrency Definition](#concurrency-definition)
    - [Concurrency Model](#concurrency-model)
    - [Synchronization Primitives](#synchronization-primitives)
      - [Mutex](#mutex)
      - [Semaphore](#semaphore)
      - [Condition Variable](#condition-variable)
      - [Monitor](#monitor)
    - [Synchronization Implementation](#synchronization-implementation)
      - [Memory Barrier](#memory-barrier)
      - [Lock Prefix Instructions](#lock-prefix-instructions)
      - [CAS](#cas)
  - [Thread](#thread)
    - [Java Thread API](#java-thread-api)
    - [Thread Pool](#thread-pool)
  - [Synchronization](#synchronization)
    - [Synchronized Key Word](#synchronized-key-word)
    - [Lock](#lock)
      - [ReentranLock](#reentranlock)
      - [ReentranReadWriteLock](#reentranreadwritelock)
      - [StampedLock](#stampedlock)
    - [Condition](#condition)
    - [Atomic](#atomic)
  - [Thread-Safe Data Structures](#thread-safe-data-structures)

## Concurency Basis

### Concurrency Definition

- 并发：在同一时间段内，多个任务交替执行，宏观上看起来是同时进行的，但微观上是交替进行的，如在单核CPU上。
- 并行：在同一时间段内，多个任务真正地同时执行，通常在多核或多处理器系统中实现。

### Concurrency Model

进程、线程和协程这些概念的提出其实都或多或少为了允许计算机在有限的计算资源下能够同时执行多个任务，即并发地运行程序。也就是说，进程、线程、协程都是操作系统中重要的并发模型。其中，进程是操作系统资源分配的基本单位。每个进程都有独立的内存空间，包括代码、数据和栈等。而线程则是执行程序的基本单位，是进程内的一个执行流。线程共享进程的内存空间，拥有自己的栈和程序计数器。至于协程则是一种轻量级线程。它一般在用户空间实现，并允许在一个程序中并发执行多个独立的控制流，且其调度由自身完成而不依赖操作系统。它们的关系如下图所示。

![进程 vs 线程](https://blog.kennycoder.io/2020/05/16/%E9%80%B2%E7%A8%8B-Process-%E3%80%81%E7%B7%9A%E7%A8%8B-Thread-%E3%80%81%E5%8D%94%E7%A8%8B-Coroutine-%E7%9A%84%E6%A6%82%E5%BF%B5%E8%AC%9B%E8%A7%A3/cover.png)

### Synchronization Primitives

同步原语是操作系统向其用户提供的简单软件机制，用于支持线程或进程同步。它们通常使用较低级别的机制（例如内存屏障、自旋锁、上下文切换等）和硬件原子指令（例如测试并设置、比较并交换等）构建。

#### Mutex

互斥量是一种简单的同步原语，用于保护共享资源，确保同一时刻只有一个线程访问。当一个线程持有互斥量时，其他试图获取该互斥量的线程将被阻塞，直到持有者释放。互斥量通常用于实现互斥访问，防止数据竞争。

#### Semaphore

信号量是一种更复杂的同步原语，它可以控制多个线程对共享资源的访问。信号量有两种类型：互斥信号量（类似于互斥量，但可以被多个线程持有）和计数信号量。计数信号量可以控制同时访问资源的线程数量，当计数值为0时，线程会被阻塞，直到其他线程释放信号量。

#### Condition Variable

条件变量允许线程在满足特定条件时等待，而不是简单地阻塞。线程可以释放锁并等待，直到其他线程改变条件并唤醒它。条件变量通常与互斥量一起使用，确保线程在等待和唤醒时的正确同步。

#### Monitor

管程(Monitor)是更高级的一种同步原语。它指的是使用一个互斥量以及一个或多个条件变量管理共享数据并发的方法，即结合了数据结构（共享数据）和控制结构（如互斥量和条件变量）来实现线程间的同步和通信。因此，和前面提到的其他同步原语相比，管程是一种更高层次的抽象。而Java中的`synchronized`关键字就可以看作是管程的一种实现。

关于同步原语更详细的介绍，可以参考stackoverflow的讨论[Definition of "synchronization primitive"](https://stackoverflow.com/questions/8017507/definition-of-synchronization-primitive)以及CMPSCI 377的[课件](https://lass.cs.umass.edu/~shenoy/courses/fall16/lectures/Lec09.pdf)

### Synchronization Implementation

#### Memory Barrier

内存栅栏是一种硬件指令，用于确保特定的内存操作按照指定的顺序执行，防止编译器和处理器的优化导致数据可见性问题。内存栅栏分为读屏障（Load Barrier）和写屏障（Store Barrier），可以防止指令重排序。例如，mfence指令在x86架构中用于实现全内存屏障，确保在屏障前后的读写操作按照正确的顺序执行。

#### Lock Prefix Instructions

#### CAS

CAS的全称是Compare And Swap。它指的是计算机的一种原子指令，可以实现无锁同步。该指令一般涉及三个操作数：

- V：要更新的变量值
- E：预期值
- N：拟写入的值

它允许原子地比较内存中的值并进行交换。如果内存中的值与预期值相等，CAS会将内存中的值更新为新值；如果不相等，它不会做任何修改。比如，`lock cmpxchg`指令就是x86体系下的CAS指令，所以我们可以使用Inline ASM实现如下效果。

```java

```

此外，GCC也提供了。

**ABA问题**：

**Atomic与乐观锁**：

由于CAS指令能够原子的更新某个变量，所以它可以很方便的实现原子类。那么CAS指令和乐观锁有什么关系呢？

## Thread

### Java Thread API

**Java线程模型**：

现在主流的后端开发语言中，Java和C++均是内核线程，而Go则是轻量级线程（通常称为协程）。换句话说，Java线程的创建、调度和销毁都需要操作系统内核的参与，而Go协程由Go Runtime库管理。它使用一种称为调度器的机制，在多个协程之间分配CPU时间，从而实现高效并发执行。

**Java线程状态**：

因此，Java线程的状态有以下几种：

- New：创建后尚未启动；
- Runnable：可能正在运行，也可能正在等待CPU时间片；
- Blocked：等待获取一个互斥锁或文件句柄等系统资源，如果其线程释放了该资源就会结束此状态；
- Waiting：无限期等待其他线程显示唤醒，否则就不会被分配CPU时间片，比如：Object.wait()、Thread.join()等方法；
- Timed Waiting：无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒，比如：Thread.sleep()等方法；
- Terminated：线程已终止。

注意区分阻塞和等待的区别，前者是被动的，而后者是主动的。

**创建和运行线程**：

在Java中，使用线程的方法包括：继承Runnable接口/Callable接口/Thread类并实现其中的run方法。其中，Callable与Runnable相比可以有返回值，返回值通过FutureTask进行封装。一般来说，更建议继承Runnable或Callable接口而不是Thread类去实现线程，因为Java不支持多继承，且继承Thread类开销太大。下面是一个使用Runnable接口创建并运行线程的例子：

```java
Runnable r = ()-> {
  // do work
}
Thread th = new Thread(r);
th.start();
```

由于Runnable是一个函数式接口，所以上面的例子使用了一个lambda表达式创建了一个实例。

**等待线程**：

在Java中，线程等待主要是其主动将自己挂起，它包括以下几种情况：

- Thread.sleep(long millis)：使当前线程休眠指定的毫秒数。
- Object.wait()：在synchronized代码块中使用，使当前线程休眠，直到被另外的线程唤醒。
- Thread.join()：等待指定线程执行完毕。

**终止线程**：

在Java中，线程会由于以下两个原因之一而终止：

- run方法正常退出，线程自热终止。
- run方法抛出异常，且没有对应catch语句，线程意外终止。

除此之外，Java还提供了Thread.interrupt()方法来请求终止一个线程。当一个线程调用Thread.interrupt()方法后，它并不会立即停止执行，而是设置一个中断标志。而检查是否设置中断状态，则可以使用静态方法Thread.currentThread().isInterrupted()。使用该方法我们就可以尝试在run方法中结束该进程，比如：

```java
Runnable r = () {
  while(!Thread.currentThread.isInterrupted()) {
    // ...
  }
}
```

但是，如果线程阻塞或等待时，就无法检查中断状态。此时就需要引入InterruptException异常。当一个正在sleep或wait的线程上调用interrupt方法时，它将被InterruptException异常中断。比如：

```java
Runnable r = () {
  try {
      while(!Thread.currentThread.isInterrupted()) {
      // ...
    }
  } catch(InterruptedException e) {
    // thread was interrupted during sleep or wait
  } finally {
    // cleanup
  }
}
```

**守护线程**：

通过调用Thread.setDaemon(true)方法，可以将线程设置为守护线程。但是守护线程并不会阻止程序的退出，即使还有守护线程在运行，只要非守护线程都结束，程序就会终止。

### Thread Pool

Java的java.util.concurrent包提供了ExecutorService和ThreadPoolExecutor，用于管理线程池，提高线程的复用性和系统资源利用率。线程池可以控制线程的创建、调度和销毁，避免频繁创建和销毁线程的开销。

## Synchronization

在Java中，同步主要依赖Java语言提供的`synchronized`关键字和JDK中提供的`java.util.concorrent`包。相比`synchronized`，JDK中提供的各种方法和接口更加灵活和强大，其内容主要如下。

![JUC](https://pdai.tech/images/thread/java-thread-x-juc-overview-1-u.png)

### Synchronized Key Word

Java中的每个对象都有一个内部锁，如果一个方法声明时使用了`synchronized`关键字，那么调用该方法时线程就必须获得内部对象锁。正因为有这个内部锁，Object中才有以下函数：

- `void notifyAll()`：唤醒所有等待内部锁的线程。
- `void notify()`：随机选择一个等待内部锁的线程。
- `void wait()`：使当前线程陷入等待状态，直到被唤醒。

比如，下面使用`synchronized`实现了一个Bank类的转账函数。

```java
class Bank {
  private double[] accounts;

  public synchronized void transfer(int from, int to, int amount) {
    while (accounts[from] < amount) 
      wait();
    accounts[from] -= amount;
    accounts[to] += amount;
    notifyAll();
  }
}
```

除了修饰普通函数之外，`synchronized`关键字还能修饰静态方法和代码块。

### Lock

#### ReentranLock

ReentrantLock实现了Lock接口，是一个可重入且独占式的锁，和`synchronized`关键字类似。不过，ReentrantLock更灵活、更强大，增加了轮询、超时、中断、公平锁和非公平锁等高级功能。

**和synchronized的区别**：

#### ReentranReadWriteLock

#### StampedLock

StampedLock是JDK1.8引入的性能更好的读写锁，不可重入且不支持条件变量Condition。

### Condition

### Atomic

## Thread-Safe Data Structures
