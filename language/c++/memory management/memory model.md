# 内存管理

- [内存管理](#内存管理)
  - [动态内存申请](#动态内存申请)
    - [关键字new](#关键字new)
    - [标准库std::allocator](#标准库stdallocator)
  - [类型擦除](#类型擦除)
  - [智能指针](#智能指针)
    - [std::unique\_ptr](#stdunique_ptr)
    - [std::shared\_ptr](#stdshared_ptr)
    - [make\_shared](#make_shared)
    - [shared\_ptr实现](#shared_ptr实现)
    - [enable\_shared\_from\_this](#enable_shared_from_this)
    - [shared\_ptr线程安全性](#shared_ptr线程安全性)
  - [weak\_ptr](#weak_ptr)
    - [weak\_ptr的使用：lock函数](#weak_ptr的使用lock函数)
    - [weak\_ptr的实现](#weak_ptr的实现)
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

### 标准库std::allocator

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

## 类型擦除

## 智能指针

在C++11标准中规定了四个智能指针std::auto_ptr，std::unique_ptr，std::shared_ptr以及std::weak_ptr。它们都用来设计辅助管理动态分配对象的生命周期，即，确保这些对象在正确的时间(包括发生异常时)用正确的方式进行回收，以确保不会产生内存泄露。其中，auto_ptr是C++98提出的，C++11已将其摒弃，并提出了unique_ptr替代auto_ptr。而shared_ptr和weak_ptr则是C+11从准标准库Boost中引入的两种智能指针。

总体来说，C++的智能指针能够像原生指针一样工作，因为它重载了解引用运算符和箭头运算符，即`*`和`->`运算符。此外，它还提供了get、release和reset三个函数丰富其功能。

其中，**get**函数用于获取托管资源地址，它会返回一个原生指针。由于智能指针重载了引用运算符和箭头运算符，所以一般情况下都很少使用该函数。而**release**函数则用于取消托管资源，即智能指针不再对该资源进行管理，改由管理员进行管理。

至于**reset**函数则用于重置智能指针托管的内存地址。当为传入地址时，它直接释放掉托管资源；而当传入地址不一致时，它则会先释放掉旧资源，再更新为传入地址。

### std::unique_ptr

unique_ptr 是 C++11 引入的新的独占式智能指针。相比auto_ptr，它改进了以下几点：

- 禁止了拷贝构造和拷贝赋值，避免了所有权转移的问题。
- 支持移动构造和移动赋值，即仍保留了auto_ptr抢夺资源所有权的特性，只不过需要程序员显式注明`std::move`，否则编译无法通过。
- 支持管理数组对象。

unique_ptr 提供了多种赋值方法，总体包括：

```c++
int* p = new int(42);
// 使用原生指针赋值
// 声明时赋值
unique_ptr<int> sp1(p);
// reset函数为声明为空的unique_ptr赋值
unique_ptr<int> sp2;
sp2.reset(new int(43));

// 使用移动语义赋值
// 声明时赋值
unique_ptr<int> sp3(move(sp1));
// 移动赋值运算符
unique_ptr<int> sp4;
sp4 = move(sp2);

// make_unique函数赋值
unique_ptr<int> sp5 = make_unique<int>(100);

// 值得一提的是，对于同一个原生指针，若赋值给多个unique_ptr则会出错
// 此外，被抢夺资源后的旧指针不应该在使用，否则就会报错。
```

- 二者都不支持隐式初始化（explicit）。
- unique_ptr因为限定管理对象所有权，所以不支持拷贝构造和拷贝赋值之外。
- shared_ptr在管理动态数组的时候，需要指定删除器删除器。（删除器不是shared_ptr类型的组成部分）

### std::shared_ptr

shared_ptr 是 C++11 引入的另一种智能指针。它和unique_ptr相比实现了共享所有权的概念。多个shared_ptr实例可以指向同一个对象，内部通过引用计数来控制对象的生命周期。当最后一个引用被销毁时，对象的内存也会被自动释放。它的初始化方式和unique_ptr以及auto_ptr类似，比如：

```c++
// 使用原生指针初始化
shared_ptr<int> sp1(new int(123));

// 复制
shared_ptr<int> sp2 = sp1;

// reset
shared_ptr<int> sp3;
sp3.reset(new int(456));
```

除了支持get、release和reset三个函数之外，shared_ptr还提供了一个use_count函数用于查看当前管理对象的引用计数。

### make_shared

shared_ptr还提供了一个名为make_shared的函数用于初始化，它的使用方法如下：

```c++
// 使用make_shared
// 注意是直接传参数，而非raw指针，比如make_share<int>(10)就创建了一个指向10的智能指针。
auto p = make_shared<type>(args); 

// 等价于以下两步操作
type *raw = new type(args);
shared_ptr<type> p(raw); 
```

和其他初始化方式相比，std::make_shared最大的优势就是能够避免一块内存由多个引用计数无联系的shared_ptr指向，比如：

```c++
int p = new int(5);

shared_ptr<int> sp1(p), sp2(p);
// 析构时，一定会报错
// free(): double free detected in tcache 2
```

### shared_ptr实现

``` c++
template <typename T>
class SharedPtr {
public:
    SharedPtr () : _pdata(nullptr), _refCount(nullptr) {}

    SharedPtr (T data) {
        _pData = new T(data);
        _refCount = new int(1);
    }
    
    SharedPtr (SharedPtr& other) {
        this->_pData = other._pData;
        this->_refCount = other._refCount;
        *(this->_refCount) += 1;
    }
    
    ~SharedPtr () {
        *(_refCount) -= 1;
        if (_pData && *(_refCount) == 0) {
          delete _pData;
          delete _refCount;
        }
    }
    
    SharedPtr &operator=(SharedPtr& other) {
        if (this == &other)
            return *this;

        // 如果原资源引用为0，则释放
        *_refCount -= 1;
        if (*_refCount == 0) {
            delete _pData;
            delete _refCount;
        }

        *(other._refCount) += 1;
        
        _pData = other._pData;
        _refCount = other._refCount;
    }
private:    
    T* _pData;
    int* _refCount;
};
```

- _refCount应该使用int*而不是int或static int，因为使用int修改会很复杂，而使用static int则会出现这种情况：
  - 多个不同的对象使用SharedPtr引用，只要不全部析构，则所有对象都不会析构，即：所有类共享了一个_refCount。
- 构造函数有两种：参数为普通指针的和参数为智能指针的。
- 赋值运算符重载主要分两个步骤实现：
  - （*this）指针不再指向之前内存区域，所以赋值前_refCount要自减；
  - （*this）指针指向other指针，所以other指针的_refCount要自增。
- 析构函数只有在_refCount == 0时，才执行。

### enable_shared_from_this

有些时候我们可能希望实现一个类函数直接返回调用对象的shared_ptr。常见的做法是：直接用该对象的this指针去初始化一个shared_ptr对象。不过，这样会返回多个独立的shared_ptr，即引用计数无关的指针指向同一块内存，从而引起析构时重复释放内存的错误。

**例子**：

比如，下面这个例子就展示了直接使用this指针初始化shared_ptr的错误做法。

```c++
#include <iostream>
#include <memory>

using namespace std;

class A{
public:
  string name;
  shared_ptr<A> getPtr() {
    return make_shared<A>(*this);
  }

  A() = default;

  A(const A& a) {
    name = a.name;
  }
};

int main() {
  A a;
  a.name = "a";
  auto ptr1 = a.getPtr();
  auto ptr2 = a.getPtr();

  cout << ptr1->name << ' ' << ptr2->name << endl;
  cout << ptr1.use_count() << ' ' << ptr2.use_count() << endl;
}
```

输出结果为：

``` txt
a a
1 1
```

可见，即使ptr1和ptr2都由this初始化，但二者的use_count确无法建立关系，因此会在后续释放时发生错误。

**使用**：

正确的做法是让该类继承std::enable_shared_from_this模板，通过CRTP的方式达成目的。比如：

```c++
class A : public enable_shared_from_this<A> {
public:
  shared_ptr<A> getPtr() {
    return shared_from_this();
  }
};
```

值得注意的是，仅当A类对象被一个shared_ptr管理时，才能调用该getPtr()函数，否则就会报错。比如：

```c++
A* a1 = new A();
auto p1 = a1->getPtr(); // wrong! Terminate called after throwing an instance of 'std::bad_weak_ptr'

shared_ptr<A> a2 = make_shared<A>();
shared_ptr<A> p2 = a2->getPtr(); // correct
```

**原理**：

至于，为什么要求A类对象被一个shared_ptr管理时才能调用该getPtr()函数就涉及到std::enable_shared_from_this的实现原理了。

简单来说，enable_shared_from_this的内部保存着一个对this的弱引用，通过该弱引用就可以获得指向this的shared_ptr了，即实现了shared_from_this函数。比如：

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

也正因为enable_shared_from_this包含着一个weak_ptr，所以该对象必须被shared_ptr管理，否则就无法正确初始化enable_shared_from_this中的weak_ptr。换句话说，构造shared_ptr指针时，判断管理类继承自enable_shared_from_this模板时就会初始化其中的weak_ptr。

另外，使用`class A : public enable_shared_from_this<A>`这样的方式则是涉及到奇异递归模板模式（Curiously Recurring Template Pattern，CRTP）的编程思想。不过，这里并没有用于获取静态多态的能力，而只是共享其成员函数shared_from_this罢了。

### shared_ptr线程安全性

shared_ptr的线程安全隐患主要涉及以下几个方面：

- 引用计数的加减操作是否线程安全。
- 修改shared_ptr指向是否线程安全。
- shared_ptr管理对象的并发操作的安全性，也应该被讨论。

以下是shared_ptr的线程安全性的结论。注意，不涉及它管理的对象的线程安全性。

- 一个shared_ptr对象可以被多个线程同时读取。因为读取操作只涉及到引用计数的增加和减少，而shared_ptr的引用计数又是原子操作，所以保证了在多线程环境下对引用计数的读取操作是线程安全的。
- 一个shared_ptr对象不可以被多个线程同时写入。因为写操作涉及到引用计数和实际数据指针的修改，这个两个操作是无法做到线程安全的。比如：下面这个例子。

![shared_ptr线程安全性例子](https://i-blog.csdnimg.cn/blog_migrate/f5c4b53e3d22d0023f7bed058b7156cb.png)

关于这一点更多的讨论，可以查看[帖子](https://blog.csdn.net/weixin_42142630/article/details/121165649)。

## weak_ptr

std::weak_ptr是C++11引入的一种智能指针，主要与std::shared_ptr配合使用。它最主要的两个作用是：

- 解决循环引用问题：当两个或多个std::shared_ptr对象互相引用时，会导致循环引用。这种情况下，这些对象的引用计数永远不会变为0，从而导致内存泄漏。std::weak_ptr可以打破这种循环引用，因为它不会增加引用计数。只需要将其中一个对象的std::shared_ptr替换为std::weak_ptr，即可解决循环引用问题。
- 一切应该不具有对象所有权，又想安全访问对象的情况都应该使用std::weak_ptr进行访问。

### weak_ptr的使用：lock函数

当我们创建一个weak_ptr时，需要用一个shared_ptr实例来初始化weak_ptr。但由于是弱共享，weak_ptr的创建并不会影响shared_ptr的引用计数值。因此，就可能存在weak_ptr指向的对象被释放掉这种情况。为此，C++标准库给weak_ptr提供了lock和expire函数用于帮助其访问。比如：

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

值得注意的是，如果lock函数返回一个非空shared_ptr，则其引用计数也会加1。除此之外，weak_ptr还提供了expired()函数来判断所指对象是否已经被销毁。比如：

```c++
shared_ptr<A> sp(new A());
weak_ptr<A> wp(sp);
sp.reset(); // 此时sp被销毁
cout << wp.expired() << endl;  // true表示已被销毁，否则为false
```

### weak_ptr的实现

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


## 参考

**new 与 malloc 的区别**:

- [stackoverflow](https://stackoverflow.com/questions/240212/what-is-the-difference-between-new-delete-and-malloc-free)
