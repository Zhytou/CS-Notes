# 内存管理

- [内存管理](#内存管理)
  - [动态内存申请](#动态内存申请)
    - [关键字new](#关键字new)
    - [标准库分配器std::allocator](#标准库分配器stdallocator)
  - [智能指针](#智能指针)
    - [共享式std::shared\_ptr](#共享式stdshared_ptr)
    - [观察式std::weak\_ptr](#观察式stdweak_ptr)
    - [独占式std::unique\_ptr](#独占式stdunique_ptr)
  - [类型擦除](#类型擦除)
  - [参考](#参考)

## 动态内存申请

值得注意的是，此处仅涉及 C++ 语言层面的动态内存管理机制。若需探究操作系统提供的用户态虚拟内存申请原语（如 mmap 与 brk 系统调用）以及 glibc malloc 的底层堆管理实现，则可以阅读操作系统笔记——[Linux中的内存管理](https://github.com/Zhytou/CS-Notes/blob/main/field/operating%20system/memory/memory%20management.md)。

### 关键字new

在 C++ 中，new 关键字常用于动态分配内存。它本质上是一个复合语法的语法糖：先分配一块原始、未初始化的内存，再在这片内存上构造（初始化）对象。

以 new T(...) 为例，这个 new 表达式会严格按顺序执行两步：

- 先调用 operator new(sizeof(T)) 分配原始内存，申请一块大小足够容纳 T 的原始内存；
- 在刚刚拿到的内存地址上，调用 T 的构造函数，完成对象初始化。

与之相对的 delete 表达式则先显式调用析构函数，随后调用 operator delete 释放内存。

**编译机制**：

从编译机制来看，new 关键字的背后是编译器与 C++ 标准库的共同配合：

- 编译器（如 g++ / clang++）的工作：new 不是普通函数调用，而是语言规范定义的表达式。编译器在语法解析阶段识别 new，并将其展开为等价逻辑：插入 operator new 调用、必要的判断分支与构造函数调用。
- 标准库（如 libstdc++ / libc++）的工作：编译器翻译出来的目标函数 operator new 并非语言内置的关键字，而是由 C++ 标准库提供的底层函数。

除了以 new 以及 delete 为代表的语法糖关键字之外，C++ 其他的关键字主要分为以下几类：

| 类别 | 代表关键字 | 编译后的产物 |
| --- | --- | --- |
| 1. 纯编译期概念 | `class`、`struct`、`namespace`、`public`、`template`、`typename`、`using`、`enum`、`constexpr` | 不直接生成 CPU 指令，主要进入编译器的符号表、类型系统、内存布局规划中；模板只有被实例化时才会产生对应代码 |
| 2. 直接映射为汇编指令 | `return`、`if`、`else`、`while`、`for`、`break`、`continue`、`goto`、`switch`、`case` | 通常直接变成机器指令，如 `ret`、`cmp`、`je`、`jmp`、`call`、条件跳转等 |
| 3. 语法糖 / 依赖运行时库 | `new`、`delete`、`throw`、`dynamic_cast`、`typeid`、`co_await`、`co_yield`、`co_return` | 被编译器展开为对运行时库函数的调用，如 `operator new`、`operator delete`、`__cxa_throw`、`__dynamic_cast`、协程状态机相关函数 |

**默认内存分配——operator new实现**：

operator new 函数负责分配一块原始、未初始化的裸内存，并返回该内存的指针。其默认的全局实现由 libstdc++ 或 libc++ 提供，本质就是 malloc 函数的上层封装：

```c++
void* operator new(std::size_t size) {
  void* p = malloc(size); // 内部直接转交给了 glibc 的 malloc！
  if (!p) {
    // 如果分配失败，抛出 std::bad_alloc 异常
    throw std::bad_alloc(); 
  }
  return p;
}
```

值得注意的是，g++ 无需采用 -l 编译指令手动链接 C++ 标准库。这是因为 g++ 是 C++ 的编译器驱动，它在链接阶段会自动把 C++ 标准库加入链接命令，等价于自动追加 -lstdc++ 或 -lc++。

可以用 g++ -v 查看完整的编译和链接过程，链接阶段会看到 collect2 / ld 实际接收到的参数中包含 -lstdc++。相比之下，如果用 gcc 编译 C++ 代码，默认不会自动链接 C++ 标准库，需要手动加 -lstdc++，否则会出现 undefined reference to std::cout 等链接错误

**自定义分配——operator new重载**：

对于高频创建与销毁的对象，通过重载 operator new 实现自定义内存分配可以显著提升性能。其核心思路是在 malloc 维护的用户态内存池之上再封装一层，例如构建类似 SLAB 的固定大小缓存槽，并通过链表管理空闲槽位。

值得注意的是，重载的 operator new 隐式为 static 静态成员函数，因为此时对象尚未诞生，this 指针并不存在。此外，当父类定义了自定义的 operator new 时，如果子类的内存占用大小与父类不同，子类必须显式重载自己的 operator new。

**对象构造——placement new语法**：

placement new 是 C++ 中在指定内存上执行对象构造的唯一语法。其基本形式为：

```c++
new (ptr) T(args...)
```

从底层内存布局的视角来看，它与 C 语言中将 malloc 返回的裸指针进行类型强转并直接对字段赋值的做法，在本质上都是在操作一块确定地址的内存。

```c++
void* p = malloc(sizeof(T));
// ...
p->x = 100;
p->y = 200;
```

然而，由于 C++ 中对象含有虚函数表指针（vptr）、虚基类偏移、多重继承产生的指针调整（thunk）等高级面向对象机制，仅仅靠 C 语言式的简单内存拷贝或字段赋值是远远不够的。编译器必须通过构造函数来正确初始化这些内部的运行时结构。

总而言之，placement new 的真正价值在于：它不会在堆上重新申请内存，而是直接指挥编译器在程序员指定的已有内存地址 ptr 上调用类型 T 的构造函数，从而确保所有复杂的对象初始化逻辑、虚表绑定以及不变式（Invariant）得以正确建立。

### 标准库分配器std::allocator

std::allocator 是 C++ 标准库提供的标准内存分配器模板类，其核心设计哲学在于将“内存的申请/释放（Allocation/Deallocation）”与“对象的构造/析构（Construction/Destruction）”彻底解耦。

```c++
template <typename T>
class allocator {
public:
  // 类型别名（STL 容器通过这些 traits 来识别指针类型、大小类型等）
  using value_type      = T;
  using pointer         = T*;
  using const_pointer   = const T*;
  using reference       = T&;
  using const_reference = const T&;
  using size_type       = std::size_t;
  using difference_type = std::ptrdiff_t;

  // 内存分配和释放函数：只管申请或回收原始内存，不调用构造和析构函数
  pointer allocate(std::size_t n);
  void deallocate(pointer p, std::size_t n) noexcept;
};
```

**std::allocator的分配流程**：

以 std::vector 为例，其类模板签名中便包含了分配器参数：

```c++
template <typename T, typename Allocator = std::allocator<T>>
class vector;
```

因此，在实际创建或扩容 vector 数组时，分配器会分两步执行该操作：

- 利用 allocator.allocate(n) 划出一块原始、未初始化的内存；
- 在该内存地址 ptr 上调用 placement new，构造对象。

这层抽象让容器可以高效地动态扩容：先申请一大块备用内存而不触发构造，等真正需要存数据时才逐个构造对象；缩容或清空时，先逐个析构对象，但底层内存不立即归还系统，留着下次复用。

**std::allocator与operator new的关系**：

std::allocator与operator new相似，它们都只负责划分未经构造的原始内存（Raw Memory），绝不涉及对象的构造与析构。但在设计定位与抽象层级上，二者存在明显的递进与封装关系：

- operator new 是底层基石：它是 C++ 语言级别提供的内存分配机制
- std::allocator 是面向容器的高层封装：它是标准库提供的模板类，面向的是类型与元素个数

实际上，当你调用 `std::allocator<T>::allocate(n)` 时，其标准的默认实现内部，通常就是直接执行：

```C++
return static_cast<T*>(::operator new(n * sizeof(T)));
```

**自定义Allocator的实现**：

## 智能指针

智能指针是 C++ 标准库中基于 RAII 机制设计的类模板，它把动态分配的资源封装到栈对象中，在构造时接管资源、在析构时自动释放，从而确保对象在正常退出或异常抛出时都能被正确回收，避免内存泄漏与重复释放。常见的智能指针包括三类：

- 共享式智能指针 std::shared_ptr
- 独占式智能指针 std::unique_ptr
- 配合共享智能指针使用的辅助智能指针 std::weak_ptr

**智能指针的区别**：

对于 std::shared_ptr 和 std::unique_ptr 而言，它们在所有权语义上存在本质区别：

- std::shared_ptr 允许多个智能指针指向同一块内存，只有当最后一个智能指针析构时才会释放资源；
- 而 std::unique_ptr 则严格独占所有权，每个指针独立管理一块内存，离开作用域时自动释放。

此外，二者在底层实现上，也存在明显差异：

- std::unique_ptr 将删除器作为模板参数，得益于 EBO（空基类优化），当删除器无状态时不占用任何额外空间，大小依然等同于一个裸指针；
- 相比之下，std::shared_ptr 的删除器则采用类型擦除，存放在堆上的控制块中，不影响智能指针自身的类型，但指针本身占用双指针大小，且控制块需要额外的堆内存分配。

**通用访问接口**：

通过对 * 和 -> 运算符的重载，智能指针完美模拟了原生指针的访问语义。除此之外，它们还提供了以下核心管理接口：

- get()：返回托管资源的裸指针。由于智能指针已重载了 * 和 ->，日常使用中很少需要它；它主要用于将裸指针传递给只接受原始指针的旧接口。
- reset()：重置托管的内存地址。无参调用时直接释放当前托管资源并置空；传入新地址时，若与当前地址不同，则先释放旧资源，再接管新地址。

**隐式转换**：

为了实现类似于 `std::shared_ptr<Base> p = std::shared_ptr<Derived>();` 的向上转型，智能指针需要支持接收其他兼容类型模板参数的构造函数。

为了防止类型不匹配导致模板实例化失败并报出晦涩的错误，现代 C++ 通常会利用 std::enable_if 结合 std::is_convertible 进行严格的 SFINAE 约束，确保只有当 U 可以隐式转换为 T 时，该转换构造函数才有效：

```c++

template <typename T>
class MySmartPtr {
// ...
template <typename U, 
              typename = std::enable_if_t<std::is_convertible_v<U*, T*>>>
    MySmartPtr(const MySmartPtr<U>& other) : _ptr(other._ptr) {
};
```

**动态数组与非内存资源管理**：

智能指针不仅能管理常规的单对象内存，还能完美适配动态数组与非内存系统资源。

其中，std::unique_ptr<T[]> 从 C++11 起就原生支持数组特化，会自动调用 delete[] 释放内存，并重载了 operator[] 以下标访问元素；std::shared_ptr<T[]> 则从 C++17 起正式支持数组特化，同样提供下标访问能力，并保证在析构时执行正确的数组销毁逻辑。

至于非内存资源（如文件句柄、数据库连接等），std::unique_ptr 是通过模板参数传入自定义删除器，std::shared_ptr 则是通过构造函数基于类型擦除机制传入自定义删除器。

### 共享式std::shared_ptr

shared_ptr 是 C++11 引入的共享智能指针。它允许多个实例同时指向同一对象，内部通过引用计数管理生命周期——当最后一个持有该对象的 shared_ptr 被销毁时，对象内存才会被自动释放。

**初始化std::shared_ptr**：

```c++
// 使用原生指针初始化
shared_ptr<int> sp1(new int(123));

// 复制
shared_ptr<int> sp2 = sp1;

// reset
shared_ptr<int> sp3;
sp3.reset(new int(456));
```

**使用std::make_shared初始化**：

除了上述初始化方式外，标准库还提供了一个名为 std::make_shared 的全局模板函数用于初始化，它的使用方法如下：

```c++
// 使用make_shared时，需要传入对象的类型和构造函数的参数，而非raw指针，比如make_share<int>(10)就创建了一个指向10的智能指针。
auto p = make_shared<type>(args); 

// 等价于以下两步操作
type *raw = new type(args);
shared_ptr<type> p(raw); 
```

和其他初始化方式相比，std::make_shared 最大的优势就是能够避免一块内存由多个引用计数无联系的 shared_ptr 指向，进而在析构时导致内存重复释放，比如：

```c++
int p = new int(5);

shared_ptr<int> sp1(p), sp2(p);
// 析构时，一定会报错
// free(): double free detected in tcache 2
```

此外，从性能的角度分析，std::make_shared 将对象本身和管理它的控制块（存放引用计数等信息）在内存中合并为一次分配，也即


而直接用 new 创建再交给 shared_ptr 通常需要两次独立分配。

**自定义共享智能指针的实现——引用计数**：

``` c++
template <typename T>
class SharedPtr {
public:
    // 1. 默认构造函数
    SharedPtr() : data(nullptr), refCount(nullptr) {}

    // 2. 接收原生指针的构造函数
    explicit SharedPtr(T* p) : data(p) {
        if (data) {
            refCount = new int(1);
        } else {
            refCount = nullptr;
        }
    }
    
    // 3. 拷贝构造函数
    SharedPtr(const SharedPtr& other) {
        data = other.data;
        refCount = other.refCount;
        if (refCount) {
            (*refCount)++;
        }
    }
    
    // 4. 析构函数
    ~SharedPtr() {
        release();
    }
    
    // 5. 赋值运算符重载
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) {
            // 先让新资源的引用计数自增（安全处理自赋值及同资源赋值）
            if (other.refCount) {
                (*(other.refCount))++;
            }
            
            // 释放当前对象的旧资源
            release();
            
            // 接管新资源
            data = other.data;
            refCount = other.refCount;
        }
        return *this; // 不要漏掉返回值
    }

    // 辅助函数：获取引用计数（方便测试）
    int use_count() const {
        return refCount ? *refCount : 0;
    }

    T* get() const noexcept { return data; }
    T& operator*() const noexcept { return *data; }
    T* operator->() const noexcept { return data; }

private: 
    void release() {
        if (refCount) {
            (*refCount)--;
            if (*refCount == 0) {
                delete data;
                delete refCount;
            }
            data = nullptr;
            refCount = nullptr;
        }
    }

    T* data;
    int* refCount; // 必须是指针，确保多个对象共享同一个计数器堆内存
};
```

可见，自定义 SharedPtr 的核心在于引用计数的实现。为了让多个对象安全地共享同一份资源，_refCount 必须使用 int* 指针类型：

- 若直接使用普通的 int，各个实例的计数无法关联；
- 若使用 static int，则会导致所有不同的对象共享同一个全局计数器，只要程序中还有一个对象未被析构，所有资源就都无法释放。

**标准库共享智能指针的实现——控制块**：

与自定义 SharedPtr 类似，标准库 std::shared_ptr 也采用了“对象指针 + 引用计数指针（控制块）”的结构实现。不同的是，为了实现弱引用观测、自定义删除器等更丰富的特性，标准库引入了设计更复杂的控制块（。具体而言，控制块由以下四项内容组成：

- 强引用计数（Strong Reference Count / use_count）：记录当前有多少个 shared_ptr 共同拥有该资源，归零时销毁托管对象。
- 弱引用计数（Weak Reference Count / weak_count）：记录当前有多少个 weak_ptr 正在观察该资源。只有当强弱引用计数同时归零时，控制块本身的内存才会被释放。
- 删除器（Deleter）：类型擦除的函数对象，用于记录如何正确销毁对象（默认是 delete）。
- 分配器（Allocator）：用于控制块自身的内存分配与释放。

**自定义删除器**：

**自定义分配器与std::allocate_shared**：

**std::shared_ptr 的线程安全性**：

std::shared_ptr 的线程安全隐患主要涉及以下几个方面：

- 引用计数的加减操作是否线程安全。
- 修改 shared_ptr 指向的对象是否线程安全。
- shared_ptr 管理对象的并发操作的安全性，也应该被讨论。

具体而言：

- 多个线程同时读取（拷贝/复制）同一个 shared_ptr 对象是线程安全的。虽然拷贝操作会使引用计数加一，但控制块中的引用计数修改是原子操作（Atomic），因此多个线程同时去复制或销毁不同的 shared_ptr 副本（它们指向同一资源）是完全安全的。
- 多个线程同时写入（修改）同一个 shared_ptr 对象是不安全的。如果多个线程同时去改变同一个 shared_ptr 变量本身（例如多个线程同时对同一个 shared_ptr 进行赋值操作，使其指向不同的对象），由于“底层数据指针”和“引用计数”的更新无法作为一个整体打包成原子操作，会导致数据竞争（Data Race）。例如下面这种并发修改场景：

![shared_ptr线程安全性例子](https://i-blog.csdnimg.cn/blog_migrate/f5c4b53e3d22d0023f7bed058b7156cb.png)

关于这一点更多的讨论，可以[查看这里](https://blog.csdn.net/weixin_42142630/article/details/121165649)。

**使用enable_shared_from_this**：

在某些场景下，我们希望实现一个类成员函数，能够直接返回调用对象自身的 std::shared_ptr。直觉上，可能会直接用该对象的 this 指针去初始化一个 std::shared_ptr 对象。比如：

```c++
struct A{
  shared_ptr<A> getPtr() {
    return make_shared<A>(*this);
  }
};

auto ptr1 = a.getPtr(), ptr2 = a.getPtr();
// 输出 1 1，也即ptr1和ptr2的use计数都是1，说明它们是独立的shared_ptr
cout << ptr1.use_count() << ' ' << ptr2.use_count() << endl;
// 析构报错，因为会double free
```

可见，即使 ptr1 和 ptr2 都由 this 初始化，但二者的 use_count 无法建立关系，最终会导致重复释放托管对象。

为了让对象能够安全地获取自身的共享指针，标准库提供了 std::enable_shared_from_this 模板，通过奇异递归模板模式（CRTP）来实现：

```c++
class A : public enable_shared_from_this<A> {
public:
  shared_ptr<A> getPtr() {
    return shared_from_this();
  }
};
```

其中，这里使用 CRTP 模板，不是为了实现静态多态，而是为了让基类能够精准获取派生类的类型信息，从而让 shared_from_this() 成员函数能够正确地返回指向派生类的 shared_ptr。

值得注意的是，仅当 A 类对象被一个 shared_ptr 管理时，才能调用该 getPtr() 函数，否则就会报错。比如：

```c++
A* a1 = new A();
auto p1 = a1->getPtr(); // wrong! Terminate called after throwing an instance of 'std::bad_weak_ptr'

shared_ptr<A> a2 = make_shared<A>();
shared_ptr<A> p2 = a2->getPtr(); // correct
```

至于为什么要求 A 类对象被一个 std::shared_ptr 管理时才能调用该 getPtr() 函数就涉及到 std::enable_shared_from_this 的实现原理了。

简单来说，std::enable_shared_from_this 内部维护了一个指向自身的 std::weak_ptr。shared_from_this() 正是通过这个弱引用来安全地生成并返回 std::shared_ptr 的：

```c++
template<class _Tp>
class enable_shared_from_this {
  mutable weak_ptr<_Tp> __weak_this_;
  //...

public:
  shared_ptr<_Tp> shared_from_this()
      {return shared_ptr<_Tp>(__weak_this_);}
  
  shared_ptr<_Tp const> shared_from_this() const
      {return shared_ptr<const _Tp>(__weak_this_);}

  // ...
}
```

正因为基类内部依赖这个 weak_ptr，对象必须首先由 shared_ptr 接管，才能完成该 weak_ptr 的初始化。也就是说，当使用 shared_ptr 构造对象时，标准库会检测该类是否继承自 std::enable_shared_from_this，并自动为其内部的 weak_ptr 赋值。若对象直接在栈上创建或通过裸指针分配而未交由智能指针托管，其内部的 weak_ptr 就会保持空置状态，此时调用 shared_from_this() 就会抛出异常。

### 观察式std::weak_ptr

std::weak_ptr 是 C++11 引入的一种辅助型智能指针，通常与 std::shared_ptr 配合使用。它的主要作用有两个：

- 解决循环引用问题：当多个 std::shared_ptr 互相引用时会导致引用计数无法归零，从而引发内存泄漏。将其中一方替换为 std::weak_ptr 即可打破循环，因为它不参与控制块中强引用计数的管理。
- 非拥有性安全访问：适用于所有“只希望观察、访问对象，但不拥有对象所有权”的场景。

**lock函数**：

当我们创建一个 weak_ptr 时，需要用一个 shared_ptr 实例来初始化 weak_ptr。但由于是弱共享，weak_ptr 的创建并不会影响 shared_ptr 的引用计数值。因此，就可能存在 weak_ptr 指向的对象被释放掉这种情况。为此，C++标准库给weak_ptr提供了 lock 函数用于帮助其访问。比如：

```c++
// lock函数返回一个shared_ptr
// 若该weak_ptr指向对象已释放，则返回空shared_ptr
shared_ptr<A> sp(new A());
weak_ptr<A> wp(sp);
if (shared_ptr<A> pa = wp.lock()) {
  cout << pa->a << endl;
}
else {
  cout << "wp指向对象为空" << endl;
}
```

值得注意的是，如果 lock 函数返回一个非空 shared_ptr，则其引用计数也会加1。

**expired函数**：

除此之外，weak_ptr还提供了 expired() 函数来判断所指对象是否已经被销毁。比如：

```c++
shared_ptr<A> sp(new A());
weak_ptr<A> wp(sp);
sp.reset(); // 此时sp被销毁
cout << wp.expired() << endl;  // true表示已被销毁，否则为false
```

**自定义辅助智能指针的实现**：

```c++
// 实现的前提包括：
// 1、WeakPtr需要被设置为SharedPtr的友元类；
// 2、SharedPtr中引用计数和数据指针均需改为protected；
// 3、SharedPtr中增加一个使用引用计数和数据指针作为入参的构造函数。
template <typename T>
class WeakPtr {
public:    
    WeakPtr (SharedPtr& other) {
        this->_pData = other._pData;
        this->_refCount = other._refCount;
    }
    
    ~WeakPtr () = default;
    
    SharedPtr<T> lock() {
        if (*_refCount == 0) {
          return SharedPtr<T>();
        }
        return SharedPtr<T>(_pData, _refCount)
    }
private:    
    T* _pData;
    int* _refCount;
};
```

### 独占式std::unique_ptr

std::unique_ptr 是 C++11 引入的独占式智能指针。它通过禁止拷贝构造与拷贝赋值，保证同一时刻只能有一个实例指向该对象——当持有该对象的 unique_ptr 离开作用域或被重置时，所管理的内存便会自动释放。

此外，它支持资源的“抢夺”语义，允许通过移动构造与移动赋值进行所有权交接，但必须由程序员显式调用 std::move 才能完成。

**初始化std::unique_ptr**：

```c++
// 1. 使用原生指针直接初始化（构造函数为 explicit，不支持隐式转换）
int* p = new int(42);
unique_ptr<int> sp1(p);

// 2. 使用 reset 函数为预先声明为空的 unique_ptr 赋值
unique_ptr<int> sp2;
sp2.reset(new int(43));

// 3. 使用移动语义实现所有权转移
unique_ptr<int> sp3(move(sp1)); // 移动构造：sp1 变为空指针
unique_ptr<int> sp4;
sp4 = move(sp2);                // 移动赋值：sp2 变为空指针

// 4. 使用 make_unique 函数（C++14 引入，推荐做法，异常安全且简洁）
unique_ptr<int> sp5 = make_unique<int>(100);

// 【避坑指南】
// - 绝对不能将同一个原生指针赋给多个 unique_ptr，否则会导致多次析构。
// - 资源被 move 转移后，旧的 unique_ptr 会失效置空，切勿再对其进行解引用或访问。
```

**自定义独占式智能指针的实现**：

```c++
template <typename T>
class UniquePtr {
public:
    explicit UniquePtr(T* p = nullptr) noexcept : data(p) {}
    UniquePtr (UniquePtr& other) = delete;
    UniquePtr& operator=(UniquePtr& other) = delete;
    UniquePtr (UniquePtr&& other): data(other.data) {
        other.data = nullptr;
    }
    UniquePtr& operator=(UniquePtr&& other) {
        if (this != &other) {
            if (data) {
                delete data;
            }
            data = other.data;
            other.data = nullptr;
        }
        return *this;
    }
    ~UniquePtr () { if (data) delete data; }

    T* get() const noexcept { return data; }
    T& operator*() const noexcept { return *data; }
    T* operator->() const noexcept { return data; }
private:
    T* data;
};
```

由上可见，自定义的 UniquePtr 实际上就是带有 RAII 资源自动释放与移动语义的裸指针包装器。其对象大小和普通指针相同，均为 sizeof(T*)。具体而言，其核心特点包括：

- 通过禁用拷贝构造与拷贝赋值，确保独占性；
- 通过正确的移动构造与移动赋值，支持所有权转移；
- 通过析构时自动销毁资源，保证 RAII；
- 通过重载解引用运算符与箭头运算符，提供与裸指针相同的语法。

**标准库独占式智能指针的实现**：

相较于基础的自定义版本，标准库的 std::unique_ptr 支持自定义删除器，进而允许其管理非内存资源，比如文件句柄、数据库连接等。

由于 std::unique_ptr 不需要引用计数，它没有控制块，因而其自定义删除器是直接作为模板参数存在的，其完整签名形式通常为 `std::unique_ptr<T, Deleter>`。它默认传入的是 `std::default_delete<T>`，也即标准库中对 delete 进行封装的默认删除器。

```c++
template <typename T>
struct default_delete {
  constexpr default_delete() noexcept = default;
  
  template <typename U, typename = std::enable_if_t<std::is_convertible_v<U*, T*>>>
  default_delete(const default_delete<U>&) noexcept {}

  void operator()(T* ptr) const {
    static_assert(sizeof(T) > 0, "Can't delete pointer to incomplete type");
    delete ptr;
  }
};
```

值得注意的是，动态数组特化的 std::unique_ptr<T[]>，则默认使用内部为 delete[] 的删除器。

**自定义删除器**：

```c++
// 使用lambda表达式作为删除器
auto fileDeleter = [](FILE* fp) {
  if (fp) fclose(fp);
};
std::unique_ptr<FILE, decltype(fileDeleter)> filePtr(fopen("test.txt", "r"), fileDeleter);

// 使用函数指针作为删除器
void my_fclose(FILE* fp) { if (fp) fclose(fp); }
std::unique_ptr<FILE, decltype(&my_fclose)> filePtr(fopen("test.txt", "r"), my_fclose);
```

**EBO优化**：

由于删除器类型属于模板参数的一部分，std::unique_ptr 能够针对不同的删除器进行底层优化。当传入无捕获的 Lambda、无状态函数对象，标准库会启用 EBO（Empty Base Optimization，空基类优化），通过继承机制避免类内额外存储状态。这使得 sizeof(std::unique_ptr) 完美保持为 sizeof(T*)（即单个裸指针的大小，无任何额外开销）。

具体而言，std::unique_ptr 内部只需要包含辅助类对象即可：

```c++
template <typename T, typename Deleter = std::default_delete<T>>
class unique_ptr {
private:
  // 唯一的成员变量，内部封装了指针和删除器
  __compressed_pair<T*, Deleter> m_packet; 
};
```

标准库在实现 __compressed_pair 时，通常会利用 C++ 的类型萃取（如 std::is_empty 和 std::is_final）结合模板偏特化（或 if constexpr），在编译期自动选择最优的存储策略：

- 无状态（空类）：触发 EBO，将删除器作为基类继承。
- 有状态（非空类）：无法使用 EBO，将其作为普通成员变量直接存储在内部。

```c++
// 简化的标准库内部实现逻辑

// 1. 偏特化版本一：当 Deleter 是空类且不是 final 时 -> 走继承路线（触发 EBO）
template <typename T, typename Deleter, bool = std::is_empty_v<Deleter> && !std::is_final_v<Deleter>>
class __compressed_pair : private Deleter {
private:
    T* ptr_; // 指针
public:
    // 通过继承拿到 Deleter 的实例来执行删除
    Deleter& get_deleter() { return *this; }
    T*& get_pointer() { return ptr_; }
};

// 2. 偏特化版本二：当 Deleter 是有状态（非空）时 -> 走组合路线（直接作为成员变量）
template <typename T, typename Deleter>
class __compressed_pair<T, Deleter, false> {
private:
  T* ptr_;         // 裸指针
  Deleter deleter_; // 有状态的删除器直接作为成员变量存储！
public:
  Deleter& get_deleter() { return deleter_; }
  T*& get_pointer() { return ptr_; }
};
```

## 类型擦除

## 参考

- [stackoverflow](https://stackoverflow.com/questions/240212/what-is-the-difference-between-new-delete-and-malloc-free)
- [C++ 模板编程](https://github.com/Walton1128/CPP-Templates-2nd--/tree/master)
