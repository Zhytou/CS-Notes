# C++习惯用法

- [C++习惯用法](#c习惯用法)
  - [RAII](#raii)
  - [Pimpl](#pimpl)
  - [CRTP](#crtp)
  - [References](#references)

## RAII

资源获取即初始化（Resource Acquisition Is Initialization，RAII）是在C++语言中的一种惯用法。它的具体做法是，在类的构造函数申请资源，在类的析构函数释放资源。这样做能够有效避免内存泄露等问题。

RAII在C++中的应用非常广泛，比如智能指针、互斥锁包装类等。比如，我们可以非常方便地使用lock_guard，而不用担心异常安全问题。

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

此外，我们还可以实现自己的互斥锁包装类，比如:

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

## Pimpl

Pimpl是“具体实现的指针”（Pointer to Implementation）的缩写，它是隐藏实现，降低耦合性、减少编译依赖和分离接口实现的一个现代C++技术，并有着编译防火墙(Compilation Firewall)的名头。具体来说，它通过一个私有的成员指针，将指针所指向的类的内部实现数据进行隐藏。比如：

```c++
class MyClass_Impl; // forward declaration

class MyClass
{
public:
    MyClass();
    ~MyClass();
    void some_method();

private:
    MyClass_Impl *pimpl; // pointer to the implementation
};
```

## CRTP

奇异递归模板模式（Curiously Recurring Template Pattern，CRTP）是C++模板编程时的一种惯用法。它的具体做法是把派生类作为基类的模板参数。比如：

```c++
template <class T> 
struct Base
{
    void interface()
    {
        // ...
        static_cast<T*>(this)->implementation();
        // ...
    }

    static void static_func()
    {
        // ...
        T::static_sub_func();
        // ...
    }
};

struct Derived : Base<Derived>
{
    void implementation();
    static void static_sub_func();
};
```

在这个例子中，`Base<Derived>::interface()`，虽然是在`struct Derived`之前就被声明了，但未被编译器实例化直至它被实际调用，这发生于`Derived`声明之后，此时`Derived::implementation()`的声明是已知的。

这种技术获得了类似于虚函数的效果，并避免了动态多态的代价。也有人把CRTP称为“模拟的动态绑定”。

**静态多态**：

事实上，C++语言中的动态分成静态和动态两种。其中，静态多态是指编译期间确定调用函数的多态，除了CRTP可以实现静态多态之外，C++中还可以函数重载和模板编程来实现。至于动态多态则主要依靠C++中虚函数机制实现。

## References

**RAII**:

- [wiki](https://zh.wikipedia.org/wiki/RAII)
- [cppreference](https://en.cppreference.com/w/cpp/language/raii)
