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
  - [Multi Thread Promgramming](#multi-thread-promgramming)
    - [Java Thread Basis](#java-thread-basis)
    - [Java Thread API](#java-thread-api)
    - [Java Thread Pool](#java-thread-pool)
    - [Java ThreadLoacl Variable](#java-threadloacl-variable)
  - [Java Memory Model](#java-memory-model)
    - [Hardware Efficiency And Consistency](#hardware-efficiency-and-consistency)
    - [Main Memory And Working Memory](#main-memory-and-working-memory)
    - [JMM Operation](#jmm-operation)
    - [JMM Non-Atomic Treatment Of 64bit Variables](#jmm-non-atomic-treatment-of-64bit-variables)
    - [JMM Volatile](#jmm-volatile)
    - [JMM Feature](#jmm-feature)
    - [Happens-Before](#happens-before)
  - [Synchronization](#synchronization)
    - [Synchronized Key Word](#synchronized-key-word)
    - [Lock](#lock)
      - [ReentranLock](#reentranlock)
      - [ReentranReadWriteLock](#reentranreadwritelock)
      - [StampedLock](#stampedlock)
    - [Condition](#condition)
    - [AQS](#aqs)
    - [Atomic](#atomic)
    - [Volatile](#volatile)
    - [Final](#final)
  - [Java Thread-Safe Data Structures](#java-thread-safe-data-structures)

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

内存栅栏是一种硬件指令，用于确保特定的内存操作按照指定的顺序执行，防止编译器和处理器的优化导致数据可见性问题。内存栅栏分为读屏障(Load Barrier)和写屏障(Store Barrier)，可以防止指令重排序。例如，mfence指令在x86架构中用于实现全内存屏障，确保在屏障前后的读写操作按照正确的顺序执行。

#### Lock Prefix Instructions

[On multicore x86 systems, are mutexes implemented using a LOCK'd instruction?](https://stackoverflow.com/questions/6868007/on-multicore-x86-systems-are-mutexes-implemented-using-a-lockd-instruction)

#### CAS

[CAS](https://en.wikipedia.org/wiki/Compare-and-swap#Implementations)的全称是Compare And Swap。狭义上，它指的是计算机的一种原子指令，可以实现无锁同步；广义上，它则是指基于CAS指令实现的无锁同步算法。对于CAS指令来说，它一般涉及三个操作数：

- V：要更新的变量值
- E：预期值
- N：拟写入的值

如果内存中要更新的变量值与预期值相等，CAS会将内存中的值更新为新值；如果不相等，它不会做任何修改。比如，`lock cmpxchg`指令就是x86体系下的CAS指令，所以我们可以使用Inline ASM实现如下效果。

```c++
int cmpxchg(int* val, int expected, int newval) {
    __asm__ volatile(
        "lock cmpxchg %2, %0\n\t"
        : "+m" (*val), "+a" (expected)
        : "r" (newval)
        : "rax", "memory"
    );
    return expected;
}
```

此外，C语言头文件<stdatomic.h>和GCC扩展函数也直接支持CAS操作。

**ABA问题**：

如果一个值从A变为B，然后又变回A，CAS算法可能会错误地认为值没有发生改变，因为它只检查值是否与预期值相同。为了解决这个问题，可以使用版本号或时间戳来跟踪值的变化。

**Atomic与乐观锁**：

由于CAS指令能够原子的更新某个变量，所以它可以很方便的实现原子类，即将针对该类变量的更新操作都使用CAS指令完成。那么CAS指令和乐观锁有什么关系呢？

## Multi Thread Promgramming

### Java Thread Basis

**Java线程模型**：

现在主流的后端开发语言中，Java和C++均是内核线程，而Go则是轻量级线程（通常称为协程）。换句话说，Java线程的创建、调度和销毁都需要操作系统内核的参与，而Go协程由Go Runtime库管理。它使用一种称为调度器的机制，在多个协程之间分配CPU时间，从而实现高效并发执行。

**Java线程状态**：

因此，Java线程的状态有以下几种：

- New：创建后尚未启动；
- Runnable：可能正在运行，也可能正在等待CPU时间片；
- Blocked：等待获取一个互斥锁或文件句柄等系统资源，如果其线程释放了该资源就会结束此状态；
- Waiting：无限期等待其他线程显示唤醒，否则就不会被分配CPU时间片，比如：Object.wait()、Thread.join()等方法；
- Timed Waiting：无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒，比如：Thread.sleep(long)等方法；
- Terminated：线程已终止。

注意区分阻塞和等待的区别，前者是被动的，而后者是主动的。此外，等待和阻塞在资源释放一致，都会在线程挂起之前释放其获取到的资源；而有限时间等待则不会，比如Thread.sleep(long)等、方法只会让线程睡眠，而不会释放其持有的锁。

### Java Thread API

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

### Java Thread Pool

Java的java.util.concurrent包提供了ExecutorService和ThreadPoolExecutor等线程池。它可以控制线程的创建、调度和销毁，避免频繁创建和销毁线程的开销。

### Java ThreadLoacl Variable

ThreadLoacl是Java中的一个类，它允许每个线程都有自己的ThreadLocal变量副本。这些副本都存储在每个线程的本地存储区中，这涉及到JVM内存模型。它相较于基于锁或原子类的并发编程更简单易读，且变量相互在线程间隔离受JVM管理。

**ThreadLocal例子**：

通常来说，一个方法中的局部变量都是线程安全的。因此，ThreadLocal主要用于类的成员变量，尤其是当这些成员变量需要在多个方法中使用，并且需要在多线程环境下保持独立状态时。比如：用ThreaLocal类管理Session或数据库连接。

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionManager {
  private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>() {
    @Override
    protected Connection initialValue() {
      try {
        return DriverManager.getConnection("", "", "");
      } catch (SQLException e) {
        e.printStackTrace();
      }
      return null;
    }
  };

  public Connection getConnection() {
    return dbConnectionLocal.get();
  }
}
```

例子中的initialValue方法是Java中的匿名内部类和抽象方法实现的结合。匿名内部类用于创建`ThreadLocal<Connection>`的实例，同时重写了initialValue方法。这种语法在Java中很常见，用于简化代码并避免创建额外的类。或者也可以不重写initialValue函数，而将初始化工作放到getConnection函数中。

```java
public class ConnectionManager {
  private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>();

  public Connection getConnection() {
    Connection connection = dbConnectionLocal.get();
    if (connection == null) {
      try {
        connection = DriverManager.getConnection("", "", "");
        dbConnectionLocal.set(connection);
      } catch (SQLException e) {
        e.printStackTrace();
      }
    }
    return connection;
  }
}
```

**ThreadLocal函数**：

ThreadLocal类提供了以下函数：

- void set(T value)：为当前线程设置ThreadLocal变量的值。
- T get()：返回当前线程的ThreadLocal变量的值。
- void remove()：移除当前线程的ThreadLocal变量。
- T initialValue()：返回当前线程的ThreadLocal变量的初始值，通常通过重写此方法来设置默认值。

**ThreadLocal原理**：

ThreadLocal依靠其静态内部类ThreadLocalMap实现。它是一个定制的哈希表，用于高效地存储每个线程的ThreadLocal变量副本。

ThreadLocalMap的每个条目（Entry）也是一个内部类，它继承自`WeakReference<ThreadLocal<T>>`，而值则是需要ThreadLocal存储的变量类型T。这意味着ThreadLocalMap的键是弱引用，即当ThreadLocal对象不再被其他地方引用时，垃圾收集器可以回收它。

对于ThreadLocal类中get()方法获取当前线程变量副本的过程如下：

- ThreadLocal.get()方法首先获取当前线程（Thread.currentThread()）。
- 然后，它查找当前线程的ThreadLocalMap。每个线程都有一个ThreadLocalMap，存储了所有ThreadLocal变量的副本。
- 在ThreadLocalMap中，使用ThreadLocal实例作为键（key）查找对应的值（value）。如果找到了，就返回这个值；如果没找到，get()方法会返回null。

**ThreadLocal问题**：

尽管ThreadLocal在设计尽量保证不会出现内存泄漏，但使用不当仍会出现严重的问题。比如：在使用线程池（如ExecutorService）时，线程可能会被重用。如果一个线程在执行完一个任务后没有清除ThreadLocal变量，那么在执行下一个任务时，前一个任务的ThreadLocal变量可能会被意外访问，导致数据不一致。

## Java Memory Model

**JMM和跨平台性**：

和C/C++这类直接使用操作系统提供的内存模型不同，Java试图定义一种Java内存模型(Java Memory Model, JMM)来屏蔽各种硬件和操作系统的内存访问差异，进而保证Java程序的跨平台性。

**JMM和并发的关系**：

然而定义JMM并非一件易事，这个模型必须定义的足够严谨，才能让Java的并发内存访问操作不会出现歧义；但是也必须定义的足够宽松，使得JVM的实现能有足够的自由度去利用硬件的各种特性来获得更好的执行速度。即JMM必须完美支持Java并发，因为多任务并发处理在现代计算机操作系统中几乎已是一项必备的功能。

值得一提的是，在许多场景下，让计算机同时去做几件事情，不仅是因为计算机的运算能力强大了，还有一个很重要的原因是计算机的运算速度与它的存储和通信子系统的速度差距太大，大量的时间都花费在磁盘I/O、网络通信或者数据库访问上。因此，为了进一步榨干处理器的运算能力就需要支持并发。

### Hardware Efficiency And Consistency

**高速缓存**：

在现代中央处理器中，每个核心都有自己的计算单元、寄存器组以及一些列高速缓存。这些高速缓存用于存储主存中的数据副本，以提高访问速度。换句话说，每个处理器都有自己的高速缓存，但又共享同一主存，这种系统被称为共享内存多核系统(Shared Memory Multiprocess System)。然而，这也引入了缓存数据一致性问题，因为不同核心的缓存可能包含不同版本的数据。为了解决数据一致性问题，处理器访问缓存时都需要遵循一些协议来进行操作，这类协议包括MSI、MESI、MOSI等。

**乱序执行**：

除了增加高速缓存之外，为了进一步榨干处理器中计算单元的效能，处理器可能会对输入的代码进行乱序执行(Out-of-Order Execution)。常见的指令重排序包括：

- 编译器在不改变单线程程序语义的前提下，重新安排语句的执行顺序。
- 现代处理器采用了指令级并行技术(Instruction-Level Parallelism，ILP)来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

### Main Memory And Working Memory

JMM的主要目的其实就是定义程序中各种共享变量的访问规则。它并不涉及局部变量和方法参数等，因为这两类都是线程私有的。

JMM规定所有变量都存储在主内存中。每个线程还有自己的工作线程，其中保存了该线程使用共享变量的副本。线程对这些变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中数据。

这里所讲的主内存、工作内存与JVM运行时数据区域中的Java堆、栈、方法区等并不是同一个层次的对内存的划分，这两者基本上是没有任何关系的。如果两者一定要勉强对应起来，那么从变量、主内存、工作内存的定义来看，主内存主要对应于Java堆中的对象实例数据部分，而工作内存则对应于虚拟机栈中的部分区域。

### JMM Operation

关于主内存与工作内存之间具体的交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存这一类的实现细节，JMM中定义了以下8种操作来完成。JVM实现时必须保证下面提及的每一种操作都是原子的、不可再分的。

- lock：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
- unlock：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
- load：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- assign：：作用于工作内存的变量，它把一个从执行引擎接收的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store：：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
- write：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。

因此，如果要把一个变量从主内存拷贝到工作内存，那就要按顺序执行read和load操作，如果要把变量从工作内存同步回主内存，就要按顺序执行store和write操作。但是JMM只要求上述两个操作必须按顺序执行，但不要求是连续执行。也就是说read和load之间、store和write之间是可以插入其他指令的。

### JMM Non-Atomic Treatment Of 64bit Variables

JMM要求lock、unlock、read、load、assign、use、store、write这八种操作都具有原子性，但是对于64位的数据类型long和double，在模型中特别定义了一条宽松的规定：允许虚拟机将没有被volatile修饰的64位数据的读写操作划分为两次32位的操作来进行，即允许虚拟机实现自行选择是否要保证64位数据类型的load、store、read和write这四个操作的原子性。这也就是所谓的long和double的非原子性协定。

### JMM Volatile

关键字volatile是Java提供的最轻量级同步机制。当一个变量被定义成volatile之后，它将具备两项特性：

- 一是保证此变量对所有线程可见性；
- 二是禁止涉及该变量的指令重排序优化。

**可见性**：

关于volatile变量的可见性，经常会出现以下错误描述：因为volatile变量对所有线程是立即可见的，所以基于volatile变量的运算在并发下是线程安全的。这句话的论据部分并没有错，但是由其论据并不能得出“基于volatile变量的运算在并发下是线程安全的”这样的结论。volatile变量在各个线程的工作内存中是不存在一致性问题的（从物理存储的角度看，各个线程的工作内存中volatile变量也可以存在不一致的情况，但由于每次使用之前都要先刷新，执行引擎看不到不一致的情况，因此可以认为不存在一致性问题），但是Java里面的运算操作符并非原子操作，这导致volatile变量的运算在并发下一样是不安全的。比如，对volatile变量使用自增运算符。

因此，即便使用volatile修饰变量，但在其不符合下述两条规则的运算场景中，仍要通过加锁保证原子性：

- 运算结果并不依赖变量当前值，或者能够确保只有单一的线程修改变量的值。
- 变量不需要与其他状态变量共同参与不变约束。

### JMM Feature

JMM是为了应对CPU缓存和指令重排序带来的问题，它定义了线程如何访问和修改共享变量的规则，确保在多线程环境中的正确性。换言之，JMM是围绕着并发过程中处理原子性、可见性和有序性三个特征建立的。

**原子性**：

JMM可以通过read、load、assign、use、store和write这6个操作保证基本数据类型（除了某些情况下的long和double）的访问、读写都具备原子性。这是因为每个线程在其工作内存中都有共享变量的副本。对于基本类型其修改是直接发生在该副本上的，而对于引用类型其修改则是发生在其指向的实际内存。

然而，尽管基本变量本身是线程安全的，但如果代码中包含更复杂的逻辑，那么就仍然存在线程不安全问题。因为，JMM无法保证普通变量的指令执行顺序。此时，就需要使用基于JMM提供的lock和unlock操作的同步原语，比如：volatile和synchronized关键字等。的此外，如果应用场景需要一个更大范围的原子性保证，JMM还提供了lock和unlock来满足需求。

**可见性**：

可见性就是指当一个线程修改共享变量的值之后，其他线程能够立即得知这个修改。除了前面提到的volatile之外，Java还提供了两个关键字实现可见性，分别是synchronized和final。其中，synchronized的可见性是由“对一个变量执行unlock操作之前，必须先把此变量同步回主内存中（执行store、write操作）”这条规则获得的。而final关键字的可见性是指：被final修饰的字段在构造器中一旦被初始化完成(final变量无法修改)，那么在其他线程中就能看见final字段的值。

**有序性**：

Java语言提供了volatile和synchronized两个关键字来保证线程之间操作的有序性，volatile关键字本身就包含了禁止指令重排序的语义，而synchronized则是由“一个变量在同一个时刻只允许一条线程对其进行lock操作”这条规则获得的，这个规则决定了持有同一个锁的两个同步块只能串行地进入。

### Happens-Before

Happens-Before是JMM中的一个概念，它定义了两个操作之间的顺序关系，即使这些操作在不同的线程中。如果一个操作A happens-before 操作B，那么B可以观察到A的所有副作用。Happens-Before原则表达的意义其实并不是一个操作发生在另外一个操作的前面，虽然这从程序员的角度上来说也并无大碍。更准确地来说，它更想表达的意义是前一个操作的结果对于后一个操作是可见的，无论这两个操作是否在同一个线程里。

Happens-Before关系可以通过以下方式建立：

- 程序次序：在一个线程中，按照程序的顺序，前面的操作happens-before后面的操作。
- volatile变量规则：写操作happens-before后续的读操作。
- synchronized块/方法：一个线程释放锁（通过退出synchronized块或方法）happens-before另一个线程获取同一个锁（通过进入synchronized块或方法）。
- 线程启动规则：线程的启动（Thread.start()）happens-before该线程的任何操作。
- 线程中断规则：线程的中断请求（Thread.interrupt()）happens-before中断检查（Thread.isInterrupted()）。
- 线程终结规则：线程的终结happens-before后续对Thread.join()的返回。
- 终结器（Finalizer）规则：对象的构造函数完成happens-before该对象的终结器（finalize()）方法。

[JMM详解](https://javaguide.cn/java/concurrent/jmm.html#jmm-java-memory-model)

## Synchronization

在Java中，同步主要依赖Java语言提供的`synchronized`关键字和JDK中提供的`java.util.concorrent`包。相比`synchronized`，JDK中提供的各种方法和接口更加灵活和强大，其内容主要如下。

![JUC](https://pdai.tech/images/thread/java-thread-x-juc-overview-1-u.png)

### Synchronized Key Word

Java中的每个对象都有一个内部锁，如果一个方法声明时使用了synchronized关键字，那么调用该方法时线程就必须获得内部对象锁。正因为有这个内部锁，Object中才有以下函数：

- `void notifyAll()`：唤醒所有等待内部锁的线程。
- `void notify()`：随机选择一个等待内部锁的线程。
- `void wait()`：使当前线程陷入等待状态，直到被唤醒。

比如，下面使用synchronized实现了一个Bank类的转账函数。

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

除了修饰普通函数之外，synchronized关键字还能修饰静态方法和代码块。其中，使用synchronized关键字修饰的静态方法实际上是锁住了该类的Class对象，这表明一次只能有一个线程访问该类中任何静态同步方法。至于synchronized关键字修饰代码块，它需要指定锁定对象，可以指定this、类中其他成员变量或自定义的锁。比如：

```java
public class Counter {
  private int count = 0;
  private Object lock = new Object();

  public void increment() {
    synchronized (lock) {
      count++;
    }
  }
}
```

### Lock

Java中出现了很多锁的概念，包括：

- 乐观锁/悲观锁；
- 无锁/偏向锁/轻量级锁/重量级锁；
- 公平锁/非公平锁；
- 独享锁/共享锁；
- 可重入锁/非可重入锁。

关于这些概念的详细介绍可以参考这篇[文章](https://pdai.tech/md/java/thread/java-thread-x-lock-all.html)，下面主要介绍java.util.concurrent包中提供的锁。

#### ReentranLock

ReentrantLock实现了Lock接口，是一个可重入且独占式的锁，和`synchronized`关键字类似。不过，ReentrantLock更灵活、更强大，增加了轮询、超时、中断、公平锁和非公平锁等高级功能。

**可重入性**：

ReentrantLock是一个可重入锁，这意味着一个线程可以多次获取同一锁而不阻塞自己。因此，一个被该锁保护的方法可以调用另一个使用相同锁的方法。比如：

```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantExample {
  private final ReentrantLock lock = new ReentrantLock();

  public void outerMethod() {
    lock.lock();
    try {
      innerMethod();
    } finally {
      lock.unlock();
    }
  }

  public void innerMethod() {
    lock.lock();
    try {
      // 执行内部方法的逻辑
    } finally {
      lock.unlock();
    }
  }
}
```

如果该类中的成员变量lock是一个不可重入锁，那么在执行outerMethod方法时就会发生死锁。实际上，ReentrantLock通过维护一个持有计数来记录lock方法的嵌套调用，并以此保证unlock正确。

**和synchronized的异同**：

在Java中，synchronized和ReentrantLock都是可重入锁，且二者都可以使用条件变量，但ReentrantLock可以和多个条件变量关联。此外，ReentrantLock的lockInterruptibly()方法允许线程在等待锁时被中断，而synchronized无法中断等待锁的线程。比如：

```java
public class LockTest {
  private Lock lock = new ReentrantLock();

  public void doBussiness() {
    String name = Thread.currentThread().getName();

    try {
      System.out.println(name + " 开始获取锁");
      lock.lockInterruptibly();
      System.out.println(name + " 得到锁");
      System.out.println(name + " 开工干活");
      for (int i=0; i<5; i++) {
        Thread.sleep(1000);
        System.out.println(name + " : " + i);
      }
    } catch (InterruptedException e) {
      System.out.println(name + " 被中断");
      System.out.println(name + " 做些别的事情");
    } finally {
      try {
        lock.unlock();
        System.out.println(name + " 释放锁");
      } catch (Exception e) {
        System.out.println(name + " : 没有得到锁的线程运行结束");
      }
    }
  }


  public static void main(String[] args) throws InterruptedException {
    LockTest lockTest = new LockTest();
    Thread t0 = new Thread(
      new Runnable() {
        public void run() {
          lockTest.doBussiness();
        }
      }
    );

    Thread t1 = new Thread(
      new Runnable() {
        public void run() {
          lockTest.doBussiness();
        }
      }
    );

    // 启动线程t1
    t0.start();
    Thread.sleep(10);
    // 启动线程t2
    t1.start();
    Thread.sleep(100);
    // 线程t1没有得到锁，中断t1的等待
    t1.interrupt();
  }
}
```

其运行结果如下：

```txt
Thread-0 开始获取锁
Thread-0 得到锁
Thread-0 开工干活
Thread-1 开始获取锁
Thread-1 被中断
Thread-1 做些别的事情
Thread-1 : 没有得到锁的线程运行结束
Thread-0 : 0
Thread-0 : 1
Thread-0 : 2
Thread-0 : 3
Thread-0 : 4
Thread-0 释放锁
```

可见，使用lockInterruptibly()方法允许线程在等待锁时被中断，从而通过捕获InterruptedException异常的方式时线程转而去做一些其他事情。此外，ReentrantLock提供了tryLock()方法，可以尝试获取锁而不阻塞。tryLock()方法会立刻返回一个布尔值，表示获取成功与否。最后，ReentranLock可以在初始化时提供一个布尔值参数指定构建一个采用公平或非公平策略的锁。（公平锁总是倾向于等待时间最长的锁，但这有可能严重影响效率）

#### ReentranReadWriteLock

ReentranReadWriteLock是可重入读写锁，允许多个读线程同时访问，但写线程独占资源。

#### StampedLock

StampedLock是JDK1.8引入的乐观的读写锁。相比ReentranReadWriteLock，它性能更好但不可重入且不支持条件变量Condition。

### Condition

在Java中，条件变量由Condition接口实现。它总是由一个锁对象中的newCondition方法初始化，且一个锁可以有多个相关联的条件变量。比如：用一个条件变量表示资金充足。

```java
class Bank {
  private double[] accounts;
  private ReentranLock bankLock = new ReentranLock();
  private Condition sufficientFunds;

  public Bank() { 
    sufficientFunds = bankLock.newCondition();
  }

  public void transfer(int from, int to, int amount) {
    bankLock.lock();
    try {
      while (accounts[from] < amount) 
        sufficientFunds.await();
      accounts[from] -= amount;
      accounts[to] += amount;
      sufficientFunds.signalAll();  
    } finally {
      bankLock.unlock();
    }
  }
}
```

上面的transfer方法中，当资金少于转账金额时，就会调用sufficientFunds.await()使该线程阻塞。直到其他资金充足的线程调用sufficientFunds.signalAll()将其唤醒，再次检查条件，直到满足条件执行完毕。

### AQS

### Atomic

Java的java.util.concurrent包除了提供锁和条件变量之外，还提供了一组原子操作的封装类。以AtomicInteger为例，它提供的主要操作有：

- 增加值并返回新值：int addAndGet(int delta)
- 加1后返回新值：int incrementAndGet()
- 获取当前值：int get()
- 用CAS方式设置：int compareAndSet(int expect, int update)

### Volatile

volatile关键字用于修饰变量。它表示总是从内存而不是寄存器或其他缓存中读取该变量。它的特性包括：

- 对变量的修改对所有线程都是可见的。
- 禁止涉及到该变量的指令重排序。
- 只能保证单次读或写的原子性，所以不能用于多线程环境中的复合操作，如++或--。

这些特性意味着被volatile修饰的变量不需要额外的同步机制，因为该变量的修改所有线程可见。

### Final

final关键字用于声明一个不可变的变量，一旦赋值后就不能再改变。对于类的成员变量，final可以防止子类覆盖。对于方法，final可以防止子类重写。对于final变量，如果在多线程环境中，如果在初始化后被正确地赋值，那么所有线程都能看到一致的值。但是，final关键字本身并不提供线程安全性，它只是保证了不可变性。

## Java Thread-Safe Data Structures
