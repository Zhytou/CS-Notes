# 存储类说明关键字

- [存储类说明关键字](#存储类说明关键字)
  - [volatile](#volatile)
    - [指针与volatile](#指针与volatile)
    - [多线程与volatile](#多线程与volatile)
  - [const](#const)
    - [指针与const](#指针与const)
    - [成员变量与const](#成员变量与const)
    - [成员函数与const](#成员函数与const)
  - [static](#static)
    - [局部变量与static](#局部变量与static)
    - [全局变量与static](#全局变量与static)
    - [普通函数与static](#普通函数与static)
    - [成员变量与static](#成员变量与static)
    - [成员函数与static](#成员函数与static)
    - [单例模式与static](#单例模式与static)

## volatile

`volatile`关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改。`volatile`的意思是让编译器每次操作该变量时一定要从内存中真正取出，而不是使用已经存在寄存器中的值。

### 指针与volatile

和 const 修饰词类似，const 有常量指针和指针常量的说法，volatile 也有相应的概念：

``` c++
// 修饰由指针指向的对象、数据是 volatile 的：
volatile char* vpch;

// 指针自身是 volatile 的：
char* volatile pchv; 
```

**注意**：可以把一个非volatile int赋给volatile int，但是不能把非volatile对象赋给一个volatile对象。

### 多线程与volatile

- volatile 不能解决多线程中的问题。
- 按照 Hans Boehm & Nick Maclaren 的总结，volatile 只在三种场合下是合适的：
  - 和信号处理（signal handler）相关的场合；
  - 和内存映射硬件（memory mapped hardware）相关的场合；
  - 和非本地跳转（setjmp 和 longjmp）相关的场合。

## const

const变量在声明时可以选择进行初始化也可以选择不进行初始化。值得注意的是，如果不进行初始化，则无法在后续的代码中给它赋值。

### 指针与const

**顶层const**：

顶层`const`表示指针本身是个常量。

``` c++
int i  = 1;
int *const p = &i;//p将一直指向i，即顶层const
```

**底层const**：

底层`const`表示指针所指对象是个常量。

``` c++
const int *p = 1;//p是一个指向常量的指针，即底层const
```

### 成员变量与const

值得注意的是，初始化const成员变量只有一种方法，就是通过构造函数的初始化列表。而static成员变量则是在类外初始化。

### 成员函数与const

可以将`const`放在参数列表之后来修饰`this`指针，从而保障`const`变量也能调用该成员函数。

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
