# 基础

- [基础](#基础)
  - [结构体 Struct 字节大小](#结构体-struct-字节大小)
    - [原则](#原则)
    - [例子](#例子)
    - [作用](#作用)
  - [联合 Union](#联合-union)
  - [类型转换](#类型转换)
    - [隐式转换](#隐式转换)
    - [显式转换](#显式转换)
      - [命名的强制类型转换](#命名的强制类型转换)
      - [旧的强制类型转换](#旧的强制类型转换)
  - [引用](#引用)
    - [std::ref](#stdref)
  - [volatile](#volatile)
    - [指针与volatile](#指针与volatile)
    - [多线程与volatile](#多线程与volatile)
  - [const](#const)
    - [指针与const](#指针与const)
    - [成员函数与const](#成员函数与const)
  - [static](#static)
    - [局部变量与static](#局部变量与static)
    - [全局变量与static](#全局变量与static)
    - [成员变量与static](#成员变量与static)
    - [成员函数与static](#成员函数与static)
    - [单例模式与static](#单例模式与static)
  - [位运算](#位运算)
  - [sizeof](#sizeof)
  - [参考](#参考)

## 结构体 Struct 字节大小

### 原则

- 结构体变量中成员的偏移量必须是成员大小的整数倍（数字零被认为是任意整数的公倍数）；
- 结构体大小必须是所有成员的整数倍数，也即所有成员大小的公倍数。

### 例子

``` c++
struct A {
    char a;
    int b;
}
```

- 成员a自身对齐值为1，存放的起始地址为0x0000；
- 成员b自身对齐值为4，存放的起始地址为0x0004；
- 整个结构体占据8个字节，为所有成员大小整数倍数。

### 作用

- 按照4字节对齐，可以避免多次I/O

## 联合 Union

一个`union`可以拥有多个数据成员，但在任意时刻只有一个数据成员可以有值。

当我们给`union`的某个成员赋值之后，该`union`的其他成员就变成未定义状态了。

分配给一个`union`对象的存储空间至少要能容纳它的最大数据成员。

默认情况下，`union`的成员都是公有的，这一点与`struct`相同。

## 类型转换

### 隐式转换

- 算数转换

- 数值转指针

- 转常量

### 显式转换

#### 命名的强制类型转换

一个命名的强制类型转换具有如下形式：

`cast-name<type>(expression)`

其中，`type`是类型转换的目标，而`expression`熬转换的值。如果`type`是引用类型，则结果是左值。

**static_cast**:

任何具有明确定义的类型转换，只要不包含底层`const`，都可以使用`static_cast`。

`static_cast`支持的类型转换：

- 原有的自动类型转换，例如`short`转`int`、`int`转`double`、`const`转非`const`、向上转型等；
- `void`指针和具体类型指针之间的转换，例如`void *`转`int *`、`char *`转`void *`等；
- 有转换构造函数或者类型转换函数的类与其它类型之间的转换，例如`double`转`Complex`（调用转换构造函数）、`Complex`转`double`（调用类型转换函数）。

``` c++
class Complex{
public:
    Complex(double real = 0.0, double imag = 0.0): m_real(real), m_imag(imag){ }
public:
    operator double() const { return m_real; }  //类型转换函数
private:
    double m_real;
    double m_imag;
};

Complex c;
double slope = static_cast<double>(c);
```

`static_cast`不支持的类型转换：

- 两个具体类型指针之间的转换，例如`int *`转`double *`、`Student *`转`int *`等。
- `int`和指针之间的转换。
- 去掉表达式的 const 修饰和 volatile 修饰。换句话说，不能将 const/volatile 类型转换为非 const/volatile 类型。

**const_cast**:

`const_cast`比较好理解，它用来去掉表达式的`const`修饰或`volatile`修饰。换句话说，`const_cast`就是用来将`const/volatile`类型转换为非 `const/volatile`类型。

``` c++
const char *pc;
char *p = const_cast<char*>(pc);
```

**reinterpret_cast**:

`reinterpret_cast`通常为运算对象的位模式提供较低层次上的重新解释，即：可以让编译器把这个变量当成比起原本类型小的类型看待。

``` c++
int ip;
char *pc = reinterpret_cast<char*>(ip);
```

我们必须牢记的是`pc`所指的真实对象是一个整型而非字符，如果强行将`pc`当作一个普通字符指针就可能发生错误。

因此，使用`reinterpret_cast`的风险是非常大的。

#### 旧的强制类型转换

早期C++语言中，显式进行强制类型转换主要有两种形式：

``` c++
type (expr);//函数形式的强制类型转换
(type) expr;//C语言风格的强制类型转换
```

## 引用

### std::ref

C++11 中引入`std::ref`用于取某个变量的引用，这个引入是为了解决一些传参问题。

我们知道 C++ 中本来就有引用的存在，为何 C++11 中还要引入一个`std::ref`了？主要是考虑函数式编程（如`std::bind`）在使用时，是对参数直接拷贝，而不是引用。

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

### 指针与const

**顶层const**：

用名词顶层`const`表示指针本身是个常量。

``` c++
int i  = 1;
int *const p = &i;//p将一直指向i，即顶层const
```

**底层const**：

用名词底层`const`表示指针所指对象是个常量。

``` c++
const int *p = 1;//p是一个指向常量的指针，即底层const
```

### 成员函数与const

可以将`const`放在参数列表之后来修饰`this`指针，从而保障`const`变量也能调用该成员函数。

## static

### 局部变量与static

- 静态局部变量在程序执行到该对象的声明处时被首次初始化，即以后的函数调用不再进行初始化；

- 静态局部变量一般在声明处初始化，如果没有显式初始化，会被程序自动初始化为 0；

### 全局变量与static

- 全局变量是不显式用`static`修饰的全局变量，全局变量默认是有外部链接性的，作用域是整个工程，在一个文件内定义的全局变量，在另一个文件中，通过`extern`全局变量名的声明，就可以使用全局变量。

- 全局静态变量是显式用`static`修饰的全局变量，作用域是声明此变量所在的文件，其他的文件即使用`extern`声明也不能使用。

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
public:
  Singleton(T val) : val_(val) {
    if (singleton_ == nullptr) {
      singleton_ = this;
    }
  }
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

## 位运算

**x ^ x = 0**：一个数与其自身异或为零。

**x ^ 0 = x**：一个数与零异或为其本身。

[数组中数字出现次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

**x & (x-1)**：

- If you subtract `1` from a number `x` in binary:
  - all the zeroes at the right of `x` become `1`;
  - the rightmost nonzero bit of `x` becomes `0`;
  - other bits are unaffected.

> What about negative numbers? Happily, representation using 2's complement is such that negative numbers behave exactly like positive numbers. In fact, when looking at the bits of `x`, you can subtract `1` from `x` without needing to know whether `x` is a signed int or an unsigned int.

**x & (x - 1) What happened?**：

- all the zeroes at the right of `x` remain zero;
- the rightmost 1 of `x` becomes a 0 because of `x-1`;
- all other bits are unaffected, because they are the same in `x` and `x-1`.

Conclusion: we have zeroed the rightmost 1 of `x`, and left all other bits unaffected.

[二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

``` c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int ret = 0;
        while (n) {
            n &= n - 1;
            ret++;
        }
        return ret;
    }
};
```

n & (n−1)，其预算结果恰为把 n 的二进制位中的最低位的 1 变为 0 之后的结果。

## sizeof

## 参考

- [结构体字节对齐规则](https://www.cnblogs.com/heart-flying/p/9556401.html)

- [volatile在C++中的作用](https://zhuanlan.zhihu.com/p/62060524)

- [how does the formula x & (x - 1) works?](https://stackoverflow.com/questions/64605039/how-does-the-formula-x-x-1-works)

- [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

- [stackoverflow - Is sizeof evaluated at compilation time or run time?](https://stackoverflow.com/questions/2615203/is-sizeof-in-c-evaluated-at-compilation-time-or-run-time)
