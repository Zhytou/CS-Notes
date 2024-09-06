# Smart Pointer

- [Smart Pointer](#smart-pointer)
  - [auto\_ptr](#auto_ptr)
  - [unique\_ptr](#unique_ptr)
    - [unique\_ptr初始化](#unique_ptr初始化)
    - [unique\_ptr和shared\_ptr的差别](#unique_ptr和shared_ptr的差别)
  - [shared\_ptr](#shared_ptr)
    - [make\_shared](#make_shared)
    - [shared\_ptr实现](#shared_ptr实现)
    - [enable\_shared\_from\_this](#enable_shared_from_this)
    - [shared\_ptr线程安全性](#shared_ptr线程安全性)
  - [weak\_ptr](#weak_ptr)
    - [weak\_ptr的使用：lock函数](#weak_ptr的使用lock函数)
    - [weak\_ptr的实现](#weak_ptr的实现)

在C++11标准中规定了四个智能指针std::auto_ptr，std::unique_ptr，std::shared_ptr以及std::weak_ptr。它们都用来设计辅助管理动态分配对象的生命周期，即，确保这些对象在正确的时间(包括发生异常时)用正确的方式进行回收，以确保不会产生内存泄露。其中，auto_ptr是C++98提出的，C++11已将其摒弃，并提出了unique_ptr替代auto_ptr。而shared_ptr和weak_ptr则是C+11从准标准库Boost中引入的两种智能指针。

总体来说，C++的智能指针能够像原生指针一样工作，因为它重载了解引用运算符和箭头运算符，即`*`和`->`运算符。此外，它还提供了get、release和reset三个函数丰富其功能。

其中，**get**函数用于获取托管资源地址，它会返回一个原生指针。由于智能指针重载了引用运算符和箭头运算符，所以一般情况下都很少使用该函数。而**release**函数则用于取消托管资源，即智能指针不再对该资源进行管理，改由管理员进行管理。

至于**reset**函数则用于重置智能指针托管的内存地址。当为传入地址时，它直接释放掉托管资源；而当传入地址不一致时，它则会先释放掉旧资源，再更新为传入地址。

## auto_ptr

> 尽管auto_ptr已被弃用，但了解其用法仍有意义。它能帮助我们认识到其不足之处以及后续改进手段，从而对整个智能指针技术有更好的认识。

auto_ptr是C++98中引入的第一个智能指针。它在构造时获取原始指针的所有权，在析构时自动释放所管理的对象。它支持拷贝构造和赋值操作，但这两个操作会发生"所有权转移"，即源对象会被置空。由于auto_ptr这种不常用的复制语义，它很难和STL容器配合使用。比如：

```c++
auto_ptr<int> sp1(new int(42));
auto_ptr<int> sp2(new int(100));
// 42 100
cout << *sp1 << ' ' << *sp2 << endl;
sp1 = sp2;
// Segmentation fault，p2为空
cout << *sp1 << ' ' << *sp2 << endl;
```

## unique_ptr

unique_ptr是C++11引入的新的独占式(单一所有权)智能指针。相比auto_ptr，它改进了以下几点：

- 禁止了拷贝构造和拷贝赋值，避免了所有权转移的问题。
- 支持移动构造和移动赋值，即仍保留了auto_ptr抢夺资源所有权的特性，只不过需要程序员显式注明`std::move`，否则编译无法通过。
- 支持管理数组对象。

### unique_ptr初始化

unique_ptr提供了多种赋值方法，总体包括：

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

### unique_ptr和shared_ptr的差别

- 二者都不支持隐式初始化（explicit）。
- unique_ptr因为限定管理对象所有权，所以不支持拷贝构造和拷贝赋值之外。
- shared_ptr在管理动态数组的时候，需要指定删除器删除器。（删除器不是shared_ptr类型的组成部分）

## shared_ptr

shared_ptr是C++11引入的另一种智能指针。它和unique_ptr相比实现了共享所有权的概念。多个shared_ptr实例可以指向同一个对象，内部通过引用计数来控制对象的生命周期。当最后一个引用被销毁时，对象的内存也会被自动释放。它的初始化方式和unique_ptr以及auto_ptr类似，比如：

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

类似shared_ptr的实现，weak_ptr需要