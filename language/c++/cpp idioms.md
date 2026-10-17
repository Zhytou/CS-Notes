# C++习惯用法

- [C++习惯用法](#c习惯用法)
  - [RAII](#raii)
  - [SIFNAE](#sifnae)
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

## SIFNAE

SFINAE(Substitution Failure Is Not An Error)是指，C++编译器在实例化模板时发生了替换失败时，编译器不会报错，而是会继续尝试其他的重载函数或模板。这样可以在编译期间根据模板参数的特性来选择合适的函数重载或模板实例化。

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
#include <iostream>

// 1. 基类是一个模板类，接受一个模板参数 T（即派生类本身）
template <typename Derived>
class Base {
public:
    void interface() {
        // 在基类中，通过 static_cast 将自己强转为派生类指针，
        // 并直接调用派生类的方法。
        static_cast<Derived*>(this)->implementation();
    }

    // 也可以提供默认实现
    void implementation() {
        std::cout << "Base default implementation\n";
    }
};

// 2. 派生类在继承时，把“自己”作为模板参数传给基类
class Derived1 : public Base<Derived1> {
public:
    void implementation() {
        std::cout << "Derived1 implementation\n";
    }
};

class Derived2 : public Base<Derived2> {
public:
    void implementation() {
        std::cout << "Derived2 implementation\n";
    }
};

// 3. 泛型函数：接受任何继承自 Base 的类型
template <typename T>
void run(Base<T>& obj) {
    obj.interface(); // 静态绑定
}

int main() {
    Derived1 d1;
    Derived2 d2;

    run(d1); // 输出: Derived1 implementation
    run(d2); // 输出: Derived2 implementation
}
```

在这个例子中，`Base<Derived>::interface()`，虽然是在`struct Derived`之前就被声明了，但未被编译器实例化直至它被实际调用，这发生于`Derived`声明之后，此时`Derived::implementation()`的声明是已知的。

**CRTP原理**：

可见，CRTP 的核心在于利用了模板的类型参数，将基类 `Base<Derived>*` 类型的 this 指针向下转换为了派生类 `Derived*` 类型的指针。

换句话说，CRTP 通过这种方式，将基类和子类的同名函数联系了起来，使得在传入不同模板参数的派生类时，能够获得不同的实现。

**静态多态**：

总体而言，CRTP 这种技术获得了类似于虚函数的多态效果，同时彻底避免了动态多态的代价（如 vptr 开销、虚表跳转、内存膨胀），且能够在编译期间完成正确性检查。因此，它也常被形象地称为“模拟的动态绑定”，也即“静态多态”。

事实上，除了CRTP可以实现静态多态之外，C++中还可以函数重载和模板编程来实现。至于动态多态则主要依靠C++中虚函数机制实现。

**enable_shared_from_this**：

std::enable_shared_from_this 是 C++ 标准库中运用 CRTP（的经典实战范例。

它允许一个类安全地在其成员函数内部，通过 shared_from_this() 生成并返回一个指向自身的 std::shared_ptr，而不会引发由于直接用裸指针 this 构造多个独立控制块所导致的内存双重释放（Double Free）崩溃。

## References

**RAII**:

- [wiki](https://zh.wikipedia.org/wiki/RAII)
- [cppreference](https://en.cppreference.com/w/cpp/language/raii)
