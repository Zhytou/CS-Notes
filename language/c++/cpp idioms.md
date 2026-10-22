# C++习惯用法

- [C++习惯用法](#c习惯用法)
  - [RAII](#raii)
  - [SIFNAE](#sifnae)
  - [PIMPL](#pimpl)
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

## PIMPL

PIMPL 是“具体实现的指针”（Pointer to Implementation）的缩写，它是隐藏实现，降低耦合性、减少编译依赖和分离接口实现的一个现代C++技术，并有着编译防火墙(Compilation Firewall)的名头。具体来说，它通过一个私有的成员指针，将指针所指向的类的内部实现数据进行隐藏。比如：

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

在这个例子中，`Base<Derived>::interface()`，虽然是在`Derived`完整定义之前就被声明了，但由于 C++ 模板的两阶段名字查找与延迟实例化特性，它直到被实际调用时才会被编译器实例化。此时，`Derived::implementation()` 的声明已经完全可见，因此调用能够顺利展开。

**CRTP原理**：

可见，CRTP 的核心在于利用了模板的类型参数，将基类 `Base<Derived>*` 类型的 this 指针向下转换为了派生类 `Derived*` 类型的指针。

换句话说，CRTP 通过这种方式，将基类和子类的同名函数联系了起来，使得在传入不同模板参数的派生类时，能够获得不同的实现。

**静态多态**：

总体而言，CRTP 这种技术获得了类似于虚函数的多态效果，同时彻底避免了动态多态的代价（如 vptr 开销、虚表跳转、内存膨胀），且能够在编译期间完成正确性检查。因此，它也常被形象地称为“模拟的动态绑定”，也即“静态多态”。

事实上，在 C++ 中，多态的实现路径主要分为两类：

- 静态多态（编译期）：主要通过 CRTP、函数重载 以及 模板元编程 实现。
- 动态多态（运行期）：主要依靠传统的 虚函数机制 实现。

**std::variant**：

回顾上述实现会发现，CRTP 并没有继承虚函数多态的“灵魂”——即无法通过一个统一的基类指针去多路引用并管理不同的子类对象。CRTP 的各个派生类在类型上是彼此独立的，仅仅是碰巧复用了相同的函数接口。

若想在容器中将这些独立的类型强行捏合并统一管理，通常需要借助 std::variant 或带标签的联合（Toggled Union），通过 std::visit 在编译期进行分发，从而在享受静态多态高性能的同时，兼顾容器化管理的需求。

假设使用 CRTP 实现一个渲染命令基类 `CommandBase`，并定义了三个具体的渲染命令类 `DrawCmd`、`DispatchCmd`、`BarrierCmd`。

```c++
template <typename Derived>
struct CommandBase {
    // 静态多态接口：在编译期直接绑定并调用派生类的方法
    void common_validate() const {
        std::cout << "[Validator] Checking command: " << Derived::command_name() << "\n";
        static_cast<const Derived*>(this)->validate_impl();
    }

    // 默认的校验实现，子类可以按需重写
    void validate_impl() const {
        // 默认通过
    }
};

struct DrawCmd : public CommandBase<DrawCmd> {
    // ...
};

struct DispatchCmd : public CommandBase<DispatchCmd> {
    // ...
};

struct BarrierCmd : public CommandBase<BarrierCmd> {
    // ...
};
```

此时，我们可以使用联合体来将这三个独立的类统一管理起来，并使用一个枚举标签来表示当前联合体中实际持有的类型。

```c++
struct RenderCommand {
    // 1. 显式定义一个标签（Discriminant），用来标记当前 union 里装的是谁
    enum Tag { DRAW, DISPATCH, BARRIER } type;

    // 2. 原生联合体：所有的成员共享同一块内存
    union {
        DrawCmd draw_cmd;
        DispatchCmd dispatch_cmd;
        BarrierCmd barrier_cmd;
    };
};
```

而在现代 C++ 的标准库中，std::variant 正是对这种模式的标准化封装。它能够将这三个类型安全地聚合在一个对象中，并自动管理内部的类型标签与内存布局。

相比于手动维护联合体和枚举，std::variant 不仅提供了类型安全的访问接口（如 std::visit 或 std::get），还能在编译期规避许多因类型不匹配导致的未定义行为，非常适合用于构建类似渲染命令这样的异构数据结构。

```c++
// 将 DrawCmd、DispatchCmd、BarrierCmd 这三个类型“强行捏合”到一个对象中
using RenderCommand = std::variant<DrawCmd, DispatchCmd, BarrierCmd>;

int main() {
    // 创建一个可以装载多种不同 CRTP 命令的连续容器
    std::vector<RenderCommand> command_buffer;

    // 往里面塞入各种类型互不相关的命令对象
    command_buffer.push_back(DrawCmd{1024});
    command_buffer.push_back(BarrierCmd{"VertexBuffer"});
    command_buffer.push_back(DispatchCmd{16, 16});
    command_buffer.push_back(DrawCmd{512});

    std::cout << "=== Starting Command Processing ===\n";

    // 遍历容器并用 std::visit 统一分发
    for (const auto& cmd_variant : command_buffer) {
        
        // std::visit 会自动检查 variant 内部的标签（Tag），
        // 并把实际持有的对象推导为对应类型（c），交由 lambda 处理。
        std::visit([](const auto& c) {
            // 1. 享受 CRTP 带来的编译期静态复用能力
            c.common_validate(); 

            // 2. 执行各自特有的动作
            c.execute();
            
            std::cout << "---------------------------\n";
        }, cmd_variant);
    }

    return 0;
}
```

**std::enable_shared_from_this**：

std::enable_shared_from_this 是 C++ 标准库中运用 CRTP 的经典实战范例。

它允许一个类安全地在其成员函数内部，通过 shared_from_this() 生成并返回一个指向自身的 std::shared_ptr，而不会引发由于直接用裸指针 this 构造多个独立控制块所导致的内存双重释放（Double Free）崩溃。

## References

**RAII**:

- [wiki](https://zh.wikipedia.org/wiki/RAII)
- [cppreference](https://en.cppreference.com/w/cpp/language/raii)
