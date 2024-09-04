# 类型限定符

- [类型限定符](#类型限定符)
  - [mutable](#mutable)
  - [volatile](#volatile)
    - [指针与volatile](#指针与volatile)
    - [多线程与volatile](#多线程与volatile)
  - [const](#const)
    - [指针与const](#指针与const)
    - [成员变量与const](#成员变量与const)
    - [成员函数与const](#成员函数与const)
  - [参考](#参考)

对于除了函数类型或引用类型以外的任何类型T（包括不完整类型），C++类型系统中有另外三个独立的类型：const-限定的T、volatile-限定的T及const-volatile-限定的T。

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

## 参考

[cppreference cv type qualifier](https://en.cppreference.com/w/cpp/language/cv)
