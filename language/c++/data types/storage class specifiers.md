# 存储类说明关键字

- [存储类说明关键字](#存储类说明关键字)
  - [mutable](#mutable)
  - [static](#static)
    - [局部变量与static](#局部变量与static)
    - [全局变量与static](#全局变量与static)
    - [普通函数与static](#普通函数与static)
    - [成员变量与static](#成员变量与static)
    - [成员函数与static](#成员函数与static)
    - [单例模式与static](#单例模式与static)
  - [参考](#参考)

## mutable

在C++中，mutable是一个关键字。它常用于在类中声明一个可变的成员变量，使其可以在const成员函数中被修改。比如：

```c++
class MyClass {
public:
  void incrementCounter() const {
      ++counter_;  // 在常量成员函数中修改被声明为 mutable 的成员变量
  }

private:
  mutable int counter_ = 0;
};
```

## static

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
