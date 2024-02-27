# 智能指针

- [智能指针](#智能指针)
  - [shared\_ptr](#shared_ptr)
    - [实现自己的shared\_ptr](#实现自己的shared_ptr)
  - [unique\_ptr](#unique_ptr)
  - [weak\_ptr](#weak_ptr)

**原生指针的问题**：

原生指针是一款很强大的工具，但是依据进数十年的经验，可以确定的一点是:稍有不慎，这
个工具就会反噬它的使用者。
终于，来解决上述难题的智能指针出现了，智能指针表现起来很像原生指针，它相当于是原
生指针的一层再包装(wrapper)，但是规避了许多使用原生指针带来的陷阱。你应该尽量使用
智能指针，它几乎能做到原生指针能做到的所有功能，却很少给你犯错的机会。

在C++11标准中规定了四个智能指针:std::auto_ptr, std::unique_ptr, std::shared_ptr, 以及std::weak_ptr.它们都用来设计辅助管理动态分配对象的生命周期，即，确保这些对象在正确的时间(包括发生异常时)用正确的方式进行回收，以确保不会产生内存泄露。

指针创建时，申请内存；指针销毁时，释放内存。此外，`shared_ptr`和`unique_ptr`的构造函数都是`explicit`的，即：必须直接使用（new返回的）普通指针直接赋值。

智能指针的API有着显著的区别，他们之间唯一共同的一点功能就是默认的构造方法。

## shared_ptr

`std::shared_ptr<T>`是一种智能指针，它能够记录多少个`std::shared_ptr<T>`共同指向一个对象，从而消除显式的调用delete，当引用计数变为零的时候就会将对象自动删除。

但还不够，因为使用`std::shared_ptr<T>`仍然需要使用new来调用，这使得代码出现了某种程度上的不对称。

`std::make_shared`就能够用来消除显式的使用new，所以`std::make_shared`会分配创建传入参数中的对象， 并返回这个对象类型的`std::shared_ptr<T>`指针。

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

- `_refCount`应该使用`int*`而不是`int`或`static int`，因为使用`int`修改会很复杂，而使用`static int`则会出现这种情况：
  - 多个不同的对象使用`SharedPtr`引用，只要不全部析构，则所有对象都不会析构，即：所有类共享了一个`_refCount`。
- 构造函数有两种：参数为普通指针的和参数为智能指针的。
- 赋值运算符重载主要分两个步骤实现：
  - （*this）指针不再指向之前内存区域，所以赋值前`_refCount`要自减；
  - （*this）指针指向other指针，所以other指针的`_refCount`要自增。
- 析构函数只有在`_refCount == 0`时，才执行。

## unique_ptr

- The `unique_ptr<>` template holds a pointer to an object and deletes this object when the `unique_ptr<>` object is deleted.
- 由于`unique_ptr`拥有它指向的对象，因此`unique_ptr`不支持普通的拷贝或赋值操作。
- 但不能拷贝`unique_ptr`的规则有一个例外：我们可以拷贝或赋值一个将要被消耗的`unique_ptr`。

## weak_ptr

`std::weak_ptr` is a smart pointer that holds a non-owning ("weak") reference to an object that is managed by `std::shared_ptr`. It must be converted to `std::shared_ptr` in order to access the referenced object.
