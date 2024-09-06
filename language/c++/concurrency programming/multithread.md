# Multithread Programming

- [Multithread Programming](#multithread-programming)
  - [Thread Management](#thread-management)
    - [Create a Thread](#create-a-thread)
    - [Pass Arguments to Thread](#pass-arguments-to-thread)
    - [Get Return Value](#get-return-value)
    - [Sub-thread](#sub-thread)
    - [POSIX Thread](#posix-thread)
  - [Mutex and Condition Variable](#mutex-and-condition-variable)
    - [Mutex](#mutex)
    - [Mutex RAII Wrapper](#mutex-raii-wrapper)
    - [Lockable and Lock Function](#lockable-and-lock-function)
    - [Condition Variable](#condition-variable)
  - [Memory Model and Atomic Operation](#memory-model-and-atomic-operation)
    - [Atomic Type](#atomic-type)
    - [Memory Order](#memory-order)
    - [Lock-Free Synchronization](#lock-free-synchronization)
  - [Async Programming](#async-programming)
    - [STL Asynchronous Facility](#stl-asynchronous-facility)
    - [Event Loop and Callback](#event-loop-and-callback)
  - [Multithread Programming Practice](#multithread-programming-practice)
    - [Producer-Consumer](#producer-consumer)
    - [Lock-Based Thread-Safe Data Structure](#lock-based-thread-safe-data-structure)
    - [Lock-Free Thread-Safe Data Structure](#lock-free-thread-safe-data-structure)
    - [Thread Pool](#thread-pool)

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

**指针**：

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

## Mutex and Condition Variable

### Mutex

**std::mutex**：

在C++中，标准库提供了std::mutex作为互斥量帮助管理共享数据。完成创建后，可通过成员函数lock()对互斥量上锁，unlock()进行解锁。不过，实践中不推荐直接去调用其成员函数去上锁和解锁。因为调用成员函数意味着必须在每个函数出口都要去调用unlock()，也包括异常的情况。

为此，C++标准库提供了一系列基于RAII思想的互斥量包装器。这些包装器在构造时就就锁上传入互斥量，并在析构的时候进行解锁，从而保证了一个已锁互斥量能被正确解锁。我们将在下一节详细介绍这些工具。

**std::recursive_mutex**:

除了最普通的std::mutex之外，标准库还提供了std::recursive_mutex和std::timed_mutex两种更为复杂的互斥量工具。

其中，std::recursive_mutex是一个递归互斥量，允许同一个线程多次对其进行锁定而不会导致死锁。当同一个线程多次对std::recursive_mutex进行加锁操作时，只有当对应的解锁操作次数与加锁次数相等时，才会真正释放互斥量。这对于涉及递归调用或者函数嵌套的情况非常有用。

**std::timed_mutex**:

至于，std::timed_mutex则是一种带有超时功能的互斥量。它允许线程在尝试获取锁时设置超时时间。如果在指定的超时时间内无法获取到锁，线程可以根据返回值来判断是否超时。

### Mutex RAII Wrapper

正如我前面所提到的，C++标准库提供了一系列基于RAII思想的互斥量包装器，包括：std::lock_guard、std::unique_lock和std::scoped_lock。

**std::lock_guard**:

总体来说，上述三者都是基于RAII思想的管理互斥量的“锁”，即在创建时上锁，析构时解锁，但它们相互之间也有区别。其中，std::lock_guard是最普通的，它只能上锁一次，且不支持移动和拷贝语义。

**std::unique_lock**:

相比之下，std::unique_lock能够多次上锁解锁，这也是条件变量一般与它绑定的原因。此外，std::unique_lock还可以使用std::move进行移动。

**std::scoped_lock**:

至于std::scoped_lock则是C++17时引入的一种新的锁包装器。它能够以RAII的方式管理多个std::mutex。注意：如果一次需要上锁多个互斥量，需要在一个操作内完成（比如使用std::scoped_lock或全局函数std::lock），否则有可能发生死锁。例如：

```c++
mutex mtx1, mtx2;
// 可能发生死锁
lock_guard lck1(mtx1), lck2(mtx2);
```

**std::adpot_lock & std::defer_lock**:

std::adpot_lock和std::defer_lock是`<mutex>`中定义的两个常量。它们主要用于创建std::unique_lock或std::lock_guard对象时，指明互斥量当前状态。

其中，当使用std::adopt_lock标记来构造std::unique_lock或std::lock_guard对象时，表示该对象将“接管”已经被锁定的互斥量。这意味着构造函数假定互斥量已经被另一个线程锁定，不再尝试再次锁定互斥量，而是直接将其管理起来。这在某些情况下非常有用，例如在函数调用中传递已锁定的互斥量。比如：在C++17之前（引入scoped_lock之前），使用std::lock_guard托管多个互斥量。

``` c++
mutex mtx1, mtx2;
std::lock(mtx1, mtx2);
// 使用lock_guard托管互斥量
lock_guard<mutex> lck1(mtx1, adopt_lock), lck2(mtx2, adopt_lock);
```

至于，使用std::defer_lock标记来构造std::unique_lock或std::lock_guard对象则表示该对象不会立即锁定互斥量。这样构造的对象需要在后续手动调用lock()函数来锁定互斥量。这种延迟锁定的方式可以在一些需要动态控制锁定时非常有用。

``` c++
mutex mtx;
uniuqe_lock<mutex> lck(mtx, defer_lock);
// 手动锁定互斥量
lck.lock();
```

### Lockable and Lock Function

**Lockable**:

在C++20之前，Lockable是标准库文档中的一个规范。它描述的是一个实现了lock()、unlock()以及try_lock()的接口。前一节提到的互斥量以及各种互斥量包装器都是Lockable。它主要是为了方便限制模板类入参，尤其是创建各种互斥量包装器和条件变量。

随着C++20引入concept语法，Lockable也改为使用概念做约束，其潜在实现如下：

```c++
/**
  * Checks whether given type T satisfies the requirements of
  * `BasicLockable` concept (a.k.a named requirement).
  * 
  * @see http://www.cplusplus.com/reference/concept/BasicLockable/
  * 
  * @tparam T Type to check
  */
template<typename T>
concept basic_lockable = requires(T a) {

    /**
      * @brief Instantiation of type T must have a public function named `lock` 
      * and it must have return type of `void`.
      */
    { a.lock()   } -> std::same_as<void>;

    /**
      * @brief Instantiation of type T must have a public function named `unlock` 
      * and it must have return type of `void`, and it must be noexcept.
      */
    { a.unlock() } -> std::same_as<void>;
};

/**
  * Checks whether given type T satisfies the requirements of
  * `Lockable` concept (a.k.a named requirement).
  * 
  * @see http://www.cplusplus.com/reference/concept/Lockable/
  * 
  * @tparam T Type to check
  */
template<typename T>
concept lockable = basic_lockable<T> && requires (T a){
    /**
      * @brief Instantiation of type T must have a public function named `try_lock` 
      * and it must have return type of `bool`.
      */
    { a.try_lock() } -> std::same_as<bool>;
};
```

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

### Condition Variable

**std::condition_variable**:

std::condition_variable的初始化并不需要std::mutex或者mutex的RAII的包装类，而是无需传其他参数就可以直接使用。

**std::condition_variable::notify_one() & std::condition_variable::notify_all()**：

std::condition_variable::notify_one()和std::condition_variable::notify_all()的唯一区别就是：前者仅唤醒一个关联该条件变量的线程，而后者唤醒所有这种线程。

这两个函数都只能唤醒线程，释放锁仍然需要该线程的std::unique_lock对象手动调用std::unique_lock::unlock()函数释放；否则唤醒后的线程会因为无法获取所需的锁，而再次进入睡眠状态。

**std::condition_variable::wait(std::unique_lock lock, function predicate)**：

std::condition_variable::wait()函数一定要接收第一个锁RAII管理类参数，且必须是能够多次解锁和上锁的std::unqiue_lock，因为std::condition_variable要保证等待结束后恢复成结束之前的状态。

std::condition_variable::wait()的第二个参数是可选参数。用户可以选择传入预测函数，其作用是避免线程虚假苏醒(suprious wakeup)。在C++中，程序员常用while循环代替if语句做等待判断，来避免条件变量被虚假唤醒。

因此，传入第二个参数，也就等价于：

```c++
// cond.wait(lock)
std::mutex mtx;
std::condition_variable cond;

std::unique_lock<std::mutex> lock;
// 第二个参数pred是个仿函数类
while(!pred()) {
  cond.wait(lock);
}
```

## Memory Model and Atomic Operation

### Atomic Type

原子操作是指在多线程环境下不可被中断或分割的操作，它能够保证操作的完整性和一致性。在C++中，它依靠标准库`<atomic>`提供的原子类型来实现。

**std::atomic**:

std::atomic是标准库中定义的一个原子类型模板，其主要操作包括：

- std::atomic::load()：读取原子对象值。
- std::atomic::store(T val)：写入原子对象值。
- std::atomic::exchange(T val)：交换原子对象值。
- std::atomic::compare_exchange_weak/strong(T& expected, T val)：比较原子对象值和expected，若相等则将原子对象值修改为val；反之将原子对象值修改为expected。weak和strong的区别在于内存顺序。

除此之外，针对模板参数为整型和指针类型的atomic，标准库还提供了下列函数：

- std::atomic::fetch_add(T val)：对当前值执行加法操作，并返回修改前的值。
- std::atomic::fetch_sub(T val)：对当前值执行减法操作，并返回修改前的值。
- std::atomic::fetch_and(T val)：执行按位与操作，并返回修改前的值。
- std::atomic::fetch_or(T val)：执行按位或操作，并返回修改前的值。
- std::atomic::fetch_xor(T val)：执行按位异或操作，并返回修改前的值。
- std::atomic::operater++：自增运算符。
- std::atomic::operater--：自减运算符。

**std::atomic::is_lock_free()**:

std::atomic::is_lock_free()是原子类型中一个特殊的函数，它能够确认该原子类型的实现方式。当其返回true时，该原子类型直接使用原子指令实现；反之，还是依靠内部锁实现。

换句话说，当该函数返回false时，使用这种原子类型的意义就不大了。（没有获得无锁所带来的性能提高）

**std::atomic_flag**:

std::atomic_flag是标准库定义的一种原子布尔类型。它只有set和unset两种状态，并且通过以下两个操作实现：

- std::atomic_flag::test_and_set()：将atomic_flag设置为set状态，若当前已经为set状态，则返回true；反之返回false。
- std::atomic_flag::clear()：将atomic_flag设置为unset状态，即下次调用test_and_set()会返回false。

和`std::atomic<bool>`不同，std::atomic_flag的操作必须使用机器指令来实现。也就是说，std::atomic_flag的 is_lock_free()成员函数始终返回true。

除此之外，std::atomic_flag创建后最好使用ATOMIC_FLAG_INIT宏进行初始化，否则就会处于一种不确定的状态。比如：

```c++
std::atomic_flag winner = ATOMIC_FLAG_INIT;
```

### Memory Order

**std::memory_order**：

为了能够便捷、统一地在各平台指定内存顺序，C++11引入了std::memory_order。它能够作为输入参数指定std::atomic对象执行操作的内存顺序，比如：

```c++
std::atomic<int> cnt(0);
cnt.store(1, std::memory_order_relaxed); 
```

编译器会根据CPU平台类型选用合适的手段来保证对应的同步，从而大大提升了C++多线程程序的可移植性。具体来说，std::memory_order是一个枚举类型，其定义如下：

``` c++
typedef enum memory_order {
  memory_order_relaxed,   // relaxed
  memory_order_consume,   // consume
  memory_order_acquire,   // acquire
  memory_order_release,   // release
  memory_order_acq_rel,   // acquire/release
  memory_order_seq_cst    // sequentially consistent
} memory_order;
```

其中，每个元素的含义如下：

- memory_order_relaxed：没有同步或顺序保障。适用于不需要跨线程通信的原子操作。
- memory_order_consume：消费者线程可看到生产者的修改，但不保证其他修改可见（较少使用）。
- memory_order_acquire：确保之后对共享变量的访问不会在当前操作之前执行。用于读取操作。
- memory_order_release：确保之前对共享变量的修改在当前操作之后对其他线程可见。用于写入操作。
- memory_order_acq_rel：结合了acquire和release，适用于读-改-写操作。
- memory_order_seq_cst：提供全局的顺序一致性，所有线程看到的修改顺序一致。这是最强的同步保障。

**std::atomic_thread_fence(memory_order sync)**：

除了直接在原子操作中指定内存顺序，还可以使用std::atomic_thread_fence来引入内存栅栏。内存栅栏是一种同步机制，它不对任何数据进行操作，但可以影响编译器和CPU的优化行为，确保在栅栏前的所有写操作在栅栏后的所有读操作之前完成。

### Lock-Free Synchronization

通过使用原子类型并设置合理的内存顺序，程序就可以达成同步，这也就是所谓的无锁编程。比如：有两个线程，一个向数据结构中填充数据，另一个读取数据结构中的数据。为了避免恶性条件竞争，第一个线程设置一个标志，用来表明数据已经准备就绪，并且第二个线程在这个标志设置前不能读取数据。

```c++
vector<int> arr;
atomic<bool> ready(false);

void reader() {
  while(!ready.load()) {
    this_thread::sleep_for(chrono::milliseconds(100));
  }
  cout << "The answer=" << arr[0] << endl;
}

void writer() {
  arr.push_back(42);
  ready = true;
}

int main() {
  std::thread t1(reader), t2(writer);
  t1.join();
  t2.join();
  return 0;
}
```

上述程序执行之后输出`The answer=42`。

## Async Programming

### STL Asynchronous Facility

**std::future**：

std::future是一个模板类，用于表示异步操作的结果。std::future对象通常不是直接创建的，而是与 std::async(launch policy, Fn&& fn, Args&&... args)，std::packaged_task和std::promise配合使用。

**std::async(launch policy, Fn&& fn, Args&&... args)**：

std::async是一个函数模板，用于启动一个异步任务。具体来说，它接受一个可调用的对象作为参数，并在一个单独的线程上异步执行它，同时返回一个std::future对象。比如：

```c++
bool is_prime (int x) {
  std::cout << "Calculating. Please, wait...\n";
  for (int i=2; i<x; ++i) if (x%i==0) return false;
  return true;
}

int main () {
  // call is_prime(313222313) asynchronously:
  std::future<bool> fut = std::async (is_prime,313222313);

  std::cout << "Checking whether 313222313 is prime.\n";
  // ...

  bool ret = fut.get();      // waits for is_prime to return

  if (ret) std::cout << "It is prime!\n";
  else std::cout << "It is not prime.\n";

  return 0;
}
```

此外，std::async还可以指定的执行策略，包括：

- std::launch::async强制在单独的线程上异步执行函数；
- std::launch::deferred延迟执行，直到调用返回结果的std::future::get()或std::future::wait()时才执行函数。

**std::packaged_task**:

除此之外，标准库还提供了std::packaged_task这个类模板，用于打包可调用对象，再将其传递给线程异步地执行。

类似的，使用std::packaged_task也可以获得一个std::future保存的执行结果。不过，需要调用std::packaged_task::get_future()。这个std::future对象可以通过.get()来获取任务完成后产生的结果。如果任务尚未完成，则获取结果的操作将阻塞调用线程，直到结果可用。

相比于std::async，std::packaged_task将异步任务的创建和执行分离开来，提供了更高的自由度。使用std::packaged_task，你可以显式地指定在哪个线程上执行任务，也可以更灵活地控制任务的生命周期。比如：

```c++
// count down taking a second for each value:
int countdown (int from, int to) {
  for (int i=from; i!=to; --i) {
    std::cout << i << '\n';
    std::this_thread::sleep_for(std::chrono::seconds(1));
  }
  std::cout << "Lift off!\n";
  return from-to;
}

int main () {
  std::packaged_task<int(int,int)> tsk (countdown);   // set up packaged_task
  std::future<int> ret = tsk.get_future();            // get future

  std::thread th (std::move(tsk),10,0);   // spawn thread to count down from 10 to 0

  // ...

  int value = ret.get();                  // wait for the task to finish and get result

  std::cout << "The countdown lasted for " << value << " seconds.\n";

  th.join();

  return 0;
}
```

**std::promise**:

std::promise是一个类模板，它提供了在线程间传递值或异常的机制。通过这个机制，一个线程可以在任务执行的任意时刻使用std::promise对象来设置值或异常，而另一个线程则可以通过与std::promise对象相关联的std::future对象来检索这个值或异常。

```c++
void print_int (std::future<int>& fut) {
  int x = fut.get();
  std::cout << "value: " << x << '\n';
}

int main () {
  std::promise<int> prom;                      // create promise

  std::future<int> fut = prom.get_future();    // engagement with future

  std::thread th1 (print_int, std::ref(fut));  // send future to new thread

  prom.set_value (10);                         // fulfill promise
                                               // (synchronizes with getting the future)
  th1.join();
  return 0;
}
```

### Event Loop and Callback

事件循环（Event Loop）是异步编程中的一个核心概念，它允许程序在处理I/O或其他耗时操作时保持响应。而回调机制（Callback Mechanism）则是一种常和事件循环配合的设计模式，它允许程序在某个任务完成时自动执行预定的函数。

具体来说，网络编程中Reactor设计模式就是其最好的示例。它使用epoll监听大量的文件描述符，并将它们就绪的事件通知给主线程。紧接着，主线程就会将该事件的回调处理逻辑交由线程池中空闲线程处理。

## Multithread Programming Practice

### Producer-Consumer

```c++
std::mutex mtx;
std::queue<int> data;

void consumer(int n) {
  int nn = 0;
  while (nn < n) {
    std::cout << "Consuming ";
    while(!data.empty()) {
      int num = data.front();
      data.pop();
      nn += 1;
      std::cout << num << ' ';
    }
    std::cout << std::endl;
  }
  std::cout << "Finish Consuming!" << std::endl;

  return ;
}

void producer(int n) {
  while (n > 0) {
    {
      std::lock_guard lck(mtx);
      int nn = rand()%n;
      std::cout << "Producing ";
      for (int num = n; num > nn; num--) {
        data.push(num);
        std::cout << num << ' ';
      }
      std::cout << std::endl;
      n = nn;
    }
  }
  std::cout << "Finish Producing!" << std::endl;
  return ;
}
```

完成一个最简单的生产-消费者问题，可能会按上面的实现。但其中有一个潜在的问题，即当消费者占有锁且data为空时，生产者无法展开生产的，因为消费者不会主动释放锁的，因此需要引入条件变量。同时，我们也能看出mutex和condition_variable的关系——前者用于保护共享内存安全，即同一时刻仅一个线程能操作该内存，但它无法保证这些线程的执行顺序。而后者能够通过条件来控制线程执行顺序。

```c++
std::mutex mtx;
std::condition_variable cv;
std::queue<int> data;

void consumer(int n) {
  int nn = 0;

  while (nn < n) {
    std::unique_lock lck(mtx);
    while(data.empty()) {
      cv.wait(lck);
    }

    std::cout << "Consuming ";
    while(!data.empty()) {
      int num = data.front();
      data.pop();
      nn += 1;
      std::cout << num << ' ';
    }
    std::cout << std::endl;
  }
  std::cout << "Finish Consuming!" << std::endl;

  return ;
}

void producer(int n) {
  while (n > 0) {
    {
      std::lock_guard lck(mtx);
      int nn = rand()%n;
      std::cout << "Producing ";
      for (int num = n; num > nn; num--) {
        data.push(num);
        std::cout << num << ' ';
      }
      std::cout << std::endl;
      n = nn;
    }
    cv.notify_one();
    if (n > 0) {
      std::this_thread::sleep_for(std::chrono::microseconds(100));
    }
  }
  std::cout << "Finish Producing!" << std::endl;
  return ;
}
```

### Lock-Based Thread-Safe Data Structure

```c++
template<typename T>
class LockBasedQueue {
private:
    std::mutex mtx;
    std::queue<T> data;
public:
    LockBasedQueue() {}
    ~LockBasedQueue() {}

    size_t size();
    void push(const T& val);
    std::optional<T> tryPop();
};

template<typename T>
size_t LockBasedQueue<T>::size() {
    std::lock_guard lck(mtx);
    return data.size();
}

template<typename T>
void LockBasedQueue<T>::push(const T& val) {
    std::lock_guard lck(mtx);
    data.push(val);
}

template<typename T>
std::optional<T> LockBasedQueue<T>::tryPop() {
    std::lock_guard lck(mtx);
    if (data.empty()) {
        return std::nullopt;
    }
    T val = std::move(data.front());
    data.pop();
    return val;
}
```

### Lock-Free Thread-Safe Data Structure

### Thread Pool

一个线程池的核心是任务队列和工作线程组，它本质是一个生产者-消费者模型。其中，用户通过接口向线程池添加任务；工作线程不断的处理任务，直到消费完成。其定义：

```c++
class ThreadPool {
 private:
  std::string name;
  std::mutex mtx;
  std::condition_variable cv;
  std::atomic<bool> running;
  std::atomic<size_t> maxTaskNum;
  std::queue<std::function<void()>> tasks;
  std::vector<std::thread> workers;

 public:  // 构造析构
  explicit ThreadPool(const std::string& _name);
  ~ThreadPool();

 public:  // 主要接口
  void start(size_t, size_t);
  void terminate();

  template <typename Func, typename... Args>
  std::future<typename std::invoke_result_t<Func, Args...>> run(Func&&, Args&&...);
};
```

其中，void ThreadPool::start(size_t, size_t)和void ThreadPool::terminate()分别用于启动和终止线程池。前者会初始化指定数目的工作线程，并定义一个taskWrapper函数来保证工作线程在任务队列为空时仅休眠而非终止；后者则会逐个关闭工作线程，从而终止线程池。

```c++
void ThreadPool::start(size_t workerNum, size_t taskNum) {
  maxTaskNum = taskNum;
  running = true;
  for (size_t i = 0; i < workerNum; i++) {
    auto taskWrapper = [this]() {
      // 线程池工作时
      while (running.load()) {
        // 取出任务
        std::function<void()> task;
        {
          std::unique_lock<std::mutex> lck(mtx);
          // 避免虚假唤醒
          while (running && tasks.empty()) {
            cv.wait(lck);
          }

          // 终止线程池
          if (!running || tasks.empty()) {
            return;
          }

          // 提取任务
          task = std::move(tasks.front());
          tasks.pop();
        }
        cv.notify_one();
        
        // 执行任务
        task();
      }
    };

    workers.emplace_back(taskWrapper);
  }
}

void ThreadPool::terminate() {
  if (running) {
    running = false;
    cv.notify_all();
    for (size_t i = 0; i < workers.size(); i++) {
      workers[i].join();
    }
  }
}
```

最后也是最关键的接口——用于指定任务的run函数，它的实现涉及到：

- 模板编程：可变参数模板、std::invoke_result_t返回值萃取器和std::forward完美转发；
- 函数式编程：std::bind绑定器；
- 异步编程：std::packaged_task异步任务和std::future异步任务结果；

```c++
template <typename Func, typename... Args>
std::future<typename std::invoke_result_t<Func, Args...>> ThreadPool::run(Func&& func, Args&&... args) {
  if (!running) {
    throw std::runtime_error("ThreadPool Exception: thread pool is not running!");
  }

  // 提取返回类型，并定义返回的future对象
  // 注意，C++17后禁用std::result_of_t<Func(Args...)>。
  using Res = typename std::invoke_result_t<Func, Args...>;
  std::future<Res> res;

  // 将任务加入任务队列
  {
    std::lock_guard<std::mutex> lck(mtx);
    if (tasks.size() > maxTaskNum) {
      throw std::runtime_error("ThreadPool Exception: the task queue is full!");
    }

    // 将参数绑定到函数对象上，并使用一个返回类型为Res的packaged_task管理该任务
    // 注意，此处packaged_task对象一定要放在堆区，并使用智能指针管理，否则传入tasks队列后会失效，出现segment fault错误。
    // 即，若task为普通变量，在离开该作用域后，task析构释放。此时tasks.emplace([&task]() { task(); });中的task已经失效。
    auto task = std::make_shared<std::packaged_task<Res()>>(std::bind(std::forward<Func>(func), std::forward<Args>(args)...));
    res = task->get_future();
    tasks.emplace([task]() { (*task)(); });
  }
  cv.notify_one();

  return res;
}
```
