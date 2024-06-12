# 智能指针

- [智能指针](#智能指针)
  - [shared\_ptr](#shared_ptr)
    - [make\_shared](#make_shared)
    - [实现自己的shared\_ptr](#实现自己的shared_ptr)
  - [unique\_ptr](#unique_ptr)
  - [weak\_ptr](#weak_ptr)

在C++11标准中规定了四个智能指针:std::auto_ptr, std::unique_ptr, std::shared_ptr, 以及std::weak_ptr。它们都用来设计辅助管理动态分配对象的生命周期，即，确保这些对象在正确的时间(包括发生异常时)用正确的方式进行回收，以确保不会产生内存泄露。

此外，shared_ptr和unique_ptr的构造函数都是explicit的，即：必须直接使用（new返回的）普通指针直接赋值。

## shared_ptr

std::shared_ptr<>是一种智能指针，它能够记录多少个std::shared_ptr<>共同指向一个对象，从而消除显式的调用delete，当引用计数变为零的时候就会将对象自动删除。

### make_shared

std::make_shared能够用来消除显式的使用new初始化shared_ptr，比如。

```c++
// 使用make_shared
auto p = make_shared<type>(args); 

// 等价于以下两步操作
type *raw = new type(args);
shared_ptr<type> p(raw); 
```

### 实现自己的shared_ptr

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

## unique_ptr

- The unique_ptr<> template holds a pointer to an object and deletes this object when the unique_ptr<> object is deleted.
- 由于unique_ptr拥有它指向的对象，因此unique_ptr不支持普通的拷贝或赋值操作。
- 但不能拷贝unique_ptr的规则有一个例外：我们可以拷贝或赋值一个将要被消耗的unique_ptr。

## weak_ptr

std::weak_ptr是C++11引入的一种智能指针，主要与std::shared_ptr配合使用。它最主要的两个作用是：

- 解决循环引用问题：当两个或多个std::shared_ptr对象互相引用时，会导致循环引用。这种情况下，这些对象的引用计数永远不会变为0，从而导致内存泄漏。std::weak_ptr可以打破这种循环引用，因为它不会增加引用计数。只需要将其中一个对象的std::shared_ptr替换为std::weak_ptr，即可解决循环引用问题。
- 一切应该不具有对象所有权，又想安全访问对象的情况都应该使用std::weak_ptr进行访问。
