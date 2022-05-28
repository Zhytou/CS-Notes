# 内存管理

## new 与 malloc 的区别

| Feature               | new/delete             | malloc/free                |
| --------------------- | ---------------------- | -------------------------- |
| Memory allocated from | 'Free Store'           | 'Heap'                     |
| Return                | Fully typed pointer    | void *                     |
| On failure            | Throws                 | Returns NULL               |
| Required size         | Calculated by compiler | Must be specified in bytes |

>  Technically, memory allocated by `new` comes from the 'Free Store' while memory allocated by `malloc` comes from the 'Heap'. Whether these two areas are the same is an implementation detail, which is another reason that `malloc` and `new` cannot be mixed.

## 智能指针 Smart Pointer

**特点——RAII**

+ 指针创建时，申请内存；指针销毁时，释放内存。

  > Smart pointers are used to make sure that an object is deleted if it is no longer used, which avoids the memory leakage in the process.

+ 此外，`shared_ptr`和`unique_ptr`的构造函数都是`explicit`的，必须直接使用（new返回的）普通指针直接赋值。

### shared_ptr

**描述**

+ The `shared_ptr` is a reference counting smart pointer that can be used to store and pass a reference beyond the scope of a function. 

**实现**

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
        this->_refCount = ++(*other._refCount);
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
       	
        *(other._refCount) += 1;
        
        if (--(*_refCount) == 0) {
            delete _pData;
            delete _refCount;
        }
        
        _pData = other._pData;
        _refCount = other._refCount;
    }
private:    
    T* _pData;
    int* _refCount;
};
```

+ `_refCount`应该使用`int*`而不是`int`或`static int`，因为使用`int`修改会很复杂，而使用`static int`则会出现这种情况：
  - 多个不同的类使用`SharedPtr`引用，只要不全部析构，则所有类都不会析构。
  - 即：所有类共享了一个`_refCount`。
+ 构造函数有两种：参数为普通指针的和参数为智能指针的。
+ 赋值运算符重载主要分两个步骤实现：
  - （*this）指针不再指向之前内存区域，所以赋值前`_refCount`要自减；
  - （*this）指针指向other指针，所以other指针的`_refCount`要自增。
+ 析构函数只有在`_refCount == 0`时，才执行。

### unique_ptr

**描述**

+ The `unique_ptr<>` template holds a pointer to an object and deletes this object when the `unique_ptr<>` object is deleted. 
+ 由于`unique_ptr`拥有它指向的对象，因此`unique_ptr`不支持普通的拷贝或赋值操作。
+ 但不能拷贝`unique_ptr`的规则有一个例外：我们可以拷贝或赋值一个将要被消耗的`unique_ptr`。

### weak_ptr

**描述**

+ `std::weak_ptr` is a smart pointer that holds a non-owning ("weak") reference to an object that is managed by `std::shared_ptr`. It must be converted to `std::shared_ptr` in order to access the referenced object.

## 内存池 Memory Pool

### 简介

+ **Memory pools**, also called fixed-size blocks allocation, is the use of pools for memory management that allows dynamic memory allocation comparable to malloc or C++'s operator new. 

## RAII

### 简介

**描述**

+ **RAII**，全称**资源获取即初始化**（英语：**R**esource **A**cquisition **I**s **I**nitialization），它是在一些面向对象语言中的一种惯用法。

**特点**

+ RAII guarantees that the resource is available to any function that may access the object.
+  It also guarantees that all resources are released when the lifetime of their controlling object ends, in reverse order of acquisition. 

**作用**

+ 有效避免内存泄露

### 实现

+ RAII在C++中的应用非常广泛，比如上面提到的智能指针。
+ 此外，C++标准库中的[lock_guard](http://en.cppreference.com/w/cpp/thread/lock_guard)也是用RAII方式来控制互斥量:

``` c++
template <class Mutex> class lock_guard {
private:
    Mutex& mutex_;

public:
    lock_guard(Mutex& mutex) : mutex_(mutex) { mutex_.lock(); }
    ~lock_guard() { mutex_.unlock(); }
	//= delete 禁用某函数
    lock_guard(lock_guard const&) = delete;
    lock_guard& operator=(lock_guard const&) = delete;
};
```

+ 程序员可以非常方便地使用lock_guard，而不用担心异常安全问题。

``` c++ 
using std::mutex;
using std::lock_guard;

mutex g_mutex;

void access_critical_section()
{
    lock_guard<mutex> lock(g_mutex);
    unsafe_code();
}
```

## std::allocator类



## 参考

**new 与 malloc 的区别**

+ [stackoverflow](https://stackoverflow.com/questions/240212/what-is-the-difference-between-new-delete-and-malloc-free)

**智能指针**

+ [cppreference](https://en.cppreference.com/book/intro/smart_pointers)

**RAII**

+ [wiki](https://zh.wikipedia.org/wiki/RAII)
+ [cppreference](https://en.cppreference.com/w/cpp/language/raii)

