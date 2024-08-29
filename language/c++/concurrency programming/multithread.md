# Multithread Programming

- [Multithread Programming](#multithread-programming)
  - [Thread Management](#thread-management)
    - [Create a Thread](#create-a-thread)
    - [Pass Arguments to Thread](#pass-arguments-to-thread)
    - [Get Return Value](#get-return-value)
    - [Sub-thread](#sub-thread)
    - [POSIX Thread](#posix-thread)
  - [Lock-based Coding](#lock-based-coding)
    - [Mutex](#mutex)
    - [Mutex RAII Wrapper](#mutex-raii-wrapper)
    - [Condition Variable](#condition-variable)
  - [Atomic Operation](#atomic-operation)
  - [Async Programming](#async-programming)

## Thread Management

### Create a Thread

**创建即启动**：

在C++中，可以使用标准库`<thread>`提供的std::thread类来创建和启动新线程。通过创建一个std::thread对象并传入要执行的函数，即可启动一个新线程。比如：

```c++
void print() {
  cout << "hello world!" << endl;
}

int main() {
  thread t(print);
  t.join();
  return 0;
}
```

类似的，使用lambda表达式作为thread的入参也是OK的。

**懒启动**：

如果创建thread时，没有第一时间传入需要执行的函数，我们也可以通过一个移动赋值运算符来懒启动该线程，比如：

```c++
void print() {
  cout << "hello world!" << endl;
}

int main() {
  thread t; // 创建一个线程，什么也不做
  t = thread(print); // 第二时间传入一个需要执行的函数
  t.join();
  return 0;
}
```

**移动语义** ：

线程在创建后，它便拥有了一些资源，包括线程的执行状态、堆栈、ID等。因此，std::thread的拷贝构造和赋值操作符都是禁用的。如果需要将线程对象传递给其他函数或线程，可以使用右值进行转移，比如std::swap或std::move。

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

### Pass Arguments to Thread

**值传递**：

当想要使用线程执行一个带参数的函数时，我们可以在构造时直接将参数连同函数一起传入，比如：

```c++
void printNums(const vector<int>& nums) {
    for (auto num : nums) {
        cout << num << ' ';
    }
    cout << endl;
}

int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    thread t(printNums, v);
    t.join();
}
```

**引用传递**：

值得注意的是，默认情况下C++线程传参是按指传入。即子线程会复制一份参数副本到其内部，而无法影响主线程。那如果想要像普通函数那样传递引用呢？使用std::ref()即可，比如：

```c++
void sortNums(vector<int>& nums) {
  sort(nums.begin(), nums.end());
}

int main() {
  vector<int> v = {0, 3, 4, 2, 1, 5};
  thread t(sortNums, ref(v));
  t.join();
  for (auto i : v) {
    cout << i << ' ';
  }
  cout << endl;
}
```

另外，使用指针可以在传值的基础上修改其指向的内存，但需要注意内存可能提前释放的问题。比如：如果使用detach()，则当主线程崩溃或者正常结束后，该块内存被回收，若此时子线程没有结束，那么子线程中指针的访问将未定义，程序会出错。

总之，thread中参数传递有四条注意：

- 使用引用和指针是要注意；
- 对于内置简单类型，建议传值；
- 对于类对象，建议使用引用来接收，以为使用引用会只会构造两次，而传值会构造三次；
- 在detach下要避免隐式转换，因为此时子线程可能还来不及转换主线程就结束了，应该在构造线程时，用参数构造一个临时对象传入。

### Get Return Value

通过前两节，我们了解到可以通过构造std::thread对象启动一个线程来执行某个函数。有些时候，我们更关心执行完该函数的返回值。一个朴素的方法是依靠引用或者指针在参数中传递返回值。那是否还有更优雅的方式呢？

答案是有的，C++标准库`<future>`提供了future、promise、packaged_task以及async这几种手段来实现这一点。具体介绍可以查看异步编程小节。

### Sub-thread

在创建完子线程后，主线程可以选择等待或分离它，其相应函数如下：

- std::thread::join()：等待线程执行完毕，并将主线程阻塞，直到被调用的线程执行结束。
- std::thread::detach()：将线程分离，使得被调用的线程可以独立执行，不再与主线程有关联。

### POSIX Thread

除了标准库提供的线程管理功能外，还可以使用POSIX线程库（pthread）来进行线程管理。POSIX，即可移植操作系统接口（Portable Operating System Interface）是一种流行的操作系统应用程序接口、命令、工具和文件系统行为的规范。它的目标是定义一致和稳定的操作系统API，从而使程序在POSIX兼容的不同操作系统上运行。

POSIX提供的线程API包括：

- pthread_create()：创建一个新线程
- pthread_join()：等待线程终止
- pthread_exit()：终止当前线程

此外，它还提供了一些互斥量、条件变量等线程同步机制的支持。

## Lock-based Coding

### Mutex

**std::recursive_mutex**:

一个线程尝试多次获取std::mutex会导致死锁，但一个线程却可以多次获取std::recursive_mutex而不出现问题。

****

**std::lock()**:

std::mutex::lock()是std::mutex类中的一个成员函数，只能被std::mutex对象调用，当调用对象可以上锁，则成功锁住，否则发生死锁，所以一般不建议使用这个函数；而std::lock()则是STL库中的一个函数模板，可以一次锁多个mutex，当调用对象全部都可以上锁，则成功锁住，否则解锁全部调用对象，这样能有效避免死锁，不过可以用scoped_lock替代。具体例子如下：

``` c++
mutex mtx1, mtx2;
unique_lock<mutex> lk1(mtx1, defer_lock), lk2(mtx2, defer_lock);
//may have a deadlock problem
lock(lk1), lock(lk2);

//no deadlock problem
lock(lk1, lk2);
```

### Mutex RAII Wrapper

**std::lock_guard vs std::unique_lock**:

首先，上述二者都是基于RAII思想的管理互斥量的“锁”，都能够代替互斥量的lock()和unlock()，并且都能有效解决资源忘记释放的问题；

其次，二者的区别在于std::lock_guard只能上锁一次，即std::lock_guard在创建时上锁，析构时解锁;而对于std::unique_lock，它能够多次上锁解锁，这也是条件变量一般与它绑定的原因。

此外，std::unique_lock还可以使用std::move进行移动，而std::lock_guard则不行。

总的来说，std::unique_lock有更高的灵活性，但也要付出更性能损耗；与之相反的，std::lock_guard损耗更低，但灵活性也更差。

**std::scoped_lock**:

简单来说，std::scoped_lock就是std::lock_guard的升级版，它能够以RAII的方式管理多个std::mutex。注意：如果一次需要上锁多个互斥量，需要在一个操作内完成（比如使用std::scoped_lock），否则有可能发生死锁。例如：

**std::adpot_lock & std::defer_lock**:

std::defer_lock一般和std::unique_lock一起使用，表示初始化std::unique_lock对象时，对应的mutex不用立刻上锁。

``` c++
mutex mtx;
uniuqe_lock<mutex> locker(mtx, defer_lock);
```

std::adopt_lock则一般和std::lock_guard一起使用，表示初始化std::lock_guard对象时，对应的mutex已经上锁。

``` c++
mutex mtx1, mtx2;
std::lock(mtx1, mtx2);
lock_guard<mutex> lk1(mtx1, adopt_lock), lk2(mtx2, adopt_lock);
```

### Condition Variable

std::condition_variable的初始化并不需要std::mutex或者mutexRAII的包装类，而是无需传其他参数就可以直接使用。

**std::condition_variable::notify_one() vs std::condition_variable::notify_all()**：

std::condition_variable::notify_one()和std::condition_variable::notify_all()的唯一区别就是：前者仅唤醒一个关联该条件变量的线程，而后者唤醒所有这种线程。

这两个函数都只能唤醒线程，释放锁仍然需要该线程的std::unique_lock对象手动调用std::unique_lock::unlock()函数释放；否则唤醒后的线程会因为无法获取所需的锁，而再次进入睡眠状态。

**std::condition_variable::wait(std::unique_lock lock, function predicate)**：

std::condition_variable::wait()函数一定要接收第一个锁RAII管理类参数，且必须是能够多次解锁和上锁的std::unqiue_lock，因为std::condition_variable要保证等待结束后恢复成结束之前的状态。

std::condition_variable::wait()的第二个参数是可选参数。用户可以选择传入预测函数，其作用是避免线程虚假苏醒(suprious wakeup)。在C++中，程序员常用while循环代替if语句做等待判断，来避免条件变量被虚假唤醒。

因此，传入第二个参数，也就等价于：

 c++
// cond.wait(lock)
std::mutex mtx;
std::condition_variable cond;

std::unique_lock<std::mutex> lock;
// 第二个参数pred是个仿函数类
while(!pred()) {
  cond.wait(lock);
}

## Atomic Operation

> 一般来说，atomic都是不可拷贝的。这是因为有些不支持原子指令的硬件平台必须要通过锁来实现原子操作，而锁又是无法拷贝的。因此，atomic是无法拷贝的

**std::memory_order**：

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

**std::atomic::is_lock_free()**：

返回该变量是否无锁。无锁变量不会导致其他线程访问阻塞。

C++中的std::atomic原子操作可能是由CPU提供的原子指令来支持，也可能是由操作系统中的spinlock来支持，具体可以根据std::atomic::is_lock_free()去判断。

## Async Programming

**std::async**：

**std::future**：
