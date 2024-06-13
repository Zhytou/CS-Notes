# Smart Pointer

- [Smart Pointer](#smart-pointer)
  - [auto\_ptr](#auto_ptr)
  - [unique\_ptr](#unique_ptr)
  - [shared\_ptr](#shared_ptr)
  - [weak\_ptr](#weak_ptr)

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

**初始化**：

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

**make_shared**：

此外，shared_ptr也有一个名为make_shared的函数能够用于初始化，比如：

```c++
// 使用make_shared
auto p = make_shared<type>(args); 

// 等价于以下两步操作
type *raw = new type(args);
shared_ptr<type> p(raw); 
```

**shared_ptr实现**：

``` c++
using namespace std;

template <typename T>
class SharedPtr {
public:
    SharedPtr (T data) {
        _pData = new T(data);
        _refCount = new int(1);
    }
    
    SharedPtr (SharedPtr& other) {
        this->_pData = other._pdata;
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

## weak_ptr

std::weak_ptr是C++11引入的一种智能指针，主要与std::shared_ptr配合使用。它最主要的两个作用是：

- 解决循环引用问题：当两个或多个std::shared_ptr对象互相引用时，会导致循环引用。这种情况下，这些对象的引用计数永远不会变为0，从而导致内存泄漏。std::weak_ptr可以打破这种循环引用，因为它不会增加引用计数。只需要将其中一个对象的std::shared_ptr替换为std::weak_ptr，即可解决循环引用问题。
- 一切应该不具有对象所有权，又想安全访问对象的情况都应该使用std::weak_ptr进行访问。
