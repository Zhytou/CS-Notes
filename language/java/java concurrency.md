# Java Concurrency

- [Java Concurrency](#java-concurrency)
  - [Concurency Basis](#concurency-basis)
    - [Concurrency Definition](#concurrency-definition)
    - [Concurrency Model](#concurrency-model)
    - [Synchronization Primitives](#synchronization-primitives)
  - [Thread](#thread)
    - [Java Thread API](#java-thread-api)
    - [Thread Pool](#thread-pool)
  - [Synchronization](#synchronization)
    - [Lock](#lock)
    - [Condition](#condition)
    - [Synchronized Key Word](#synchronized-key-word)
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

除此之外，Java还提供了Thread.interrupt()方法来请求终止一个线程。当一个线程调用Thread.interrupt()方法后，它并不会立即停止执行，而是设置一个中断标志。

**守护线程**：

通过调用Thread.setDaemon(true)方法，可以将线程设置为守护线程。但是守护线程并不会阻止程序的退出，即使还有守护线程在运行，只要非守护线程都结束，程序就会终止。

### Thread Pool

Java的java.util.concurrent包提供了ExecutorService和ThreadPoolExecutor，用于管理线程池，提高线程的复用性和系统资源利用率。线程池可以控制线程的创建、调度和销毁，避免频繁创建和销毁线程的开销。

## Synchronization

### Lock

### Condition

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

### Atomic

## Thread-Safe Data Structures
