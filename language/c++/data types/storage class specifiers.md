# 存储类说明关键字

- [存储类说明关键字](#存储类说明关键字)
  - [存储期](#存储期)
  - [链接性](#链接性)
  - [static关键字](#static关键字)
    - [局部变量与static](#局部变量与static)
    - [全局变量与static](#全局变量与static)
    - [普通函数与static](#普通函数与static)
    - [成员变量与static](#成员变量与static)
    - [成员函数与static](#成员函数与static)
    - [单例模式与static](#单例模式与static)
  - [参考](#参考)

存储类说明关键字常用在变量或函数的声明中，用于描述其存储周期已经是否可链接的性质。常见的存储类说明符包括：

- auto：自动存储期（C++11前）；
- register：自动存储期，同时提示编译器将此对象置于处理器内存（C++17前）；
- static：静态或线程存储区，同时内部链接；
- extern：静态或线程存储期，同时外部链接；
- thread_local：线程存储期，且可与static或extern结合使用；
- mutable - 不影响存储期或连接（详见[type qualifier](https://en.cppreference.com/w/cpp/language/cv)）。

## 存储期

在C++程序中，所有对象都具有下列存储期之一：

- 自动（Automatic）存储期：对象的存储在外围代码块开始时分配，而在结束时解分配。未声明为 static、extern 或 thread_local 的所有局部对象均拥有此存储期。
- 静态（Static）存储期：对象的存储在程序开始时分配，而在程序结束时解分配。只存在对象的一个实例。所有声明于命名空间作用域（包含全局命名空间）的对象，加上声明带有 static 或 extern 的对象均拥有此存储期。
- 线程（Thread）存储期。对象的存储在线程开始时分配，而在线程结束时解分配。每个线程拥有其自身的对象实例。唯有声明为 thread_local 的对象拥有此存储期。thread_local 能与 static 或 extern 一同出现，以调整该对象的链接特性。
- 动态（Dynamic）存储期。对象的存储是通过使用动态内存分配函数（比如：new或malloc）来按请求进行分配和解分配的。

## 链接性

链接性是指对象、引用、函数、类型、模板、命名空间或值的名字，可在另一个作用域中使用。若某个名字可链接，则其所指代的实体与另一作用域中的声明所引入的同一名字指代相同的实体。若变量、函数或另一实体声明于数个作用域，但不可链接，则将生成该实体的多个实例。

## static关键字

### 局部变量与static

- 静态局部变量在程序执行到该对象的声明处时被首次初始化，即以后的函数调用不再进行初始化；
- 静态局部变量一般在声明处初始化，如果没有显式初始化，会被程序自动初始化为 0；

### 全局变量与static

- 全局变量是不显式用`static`修饰的全局变量，全局变量默认是有外部链接性的，作用域是整个工程，在一个文件内定义的全局变量，在另一个文件中，通过`extern`全局变量名的声明，就可以使用全局变量。
- 全局静态变量是显式用`static`修饰的全局变量，作用域是声明此变量所在的文件，其他的文件即使用`extern`声明也不能使用。

### 普通函数与static

- 使用`static`修饰函数可以将该函数的作用域限定在当前文件中，从而隐藏该函数，使其对其他文件不可见。

### 成员变量与static

- 被`static`修饰的变量属于类变量，可以通过类名.变量名直接引用，而不需要创建一个对象。
- 类的非静态成员函数可以调用用静态成员。
- 一般来说，不在类内初始化静态成员变量（仅常量可以在内部初始化）。

### 成员函数与static

- 静态成员函数只能定义在类外部。
- 被 static 修饰的方法属于类方法，可以通过类名.方法名直接引用，而不需要创建一个对象。
- 静态成员函数中不能调用非静态成员。
- 静态成员函数没有 this 指针。

### 单例模式与static

``` c++
template<typename T>
class Singleton{
private:
  static Singleton* singleton_;
  T val_;
  Singleton(T val) : val_(val) {
    if (singleton_ == nullptr) {
      singleton_ = this;
    }
  }
public:
  Singleton(const Singleton& other) = delete;
  Singleton& operator=(const Singleton& other) = delete;

  static Singleton* getInstance() {
    return singleton_;
  }
}

Singleton::singleton_ = nullptr;

Singleton* Singleton::getInstance() {
  if (singleton_ == nullptr) {
    singleton = new Singleton(val)
  }
  return singleton_;
}
```

## 参考

[cppreference storage class specifier](https://en.cppreference.com/w/cpp/language/storage_duration)
