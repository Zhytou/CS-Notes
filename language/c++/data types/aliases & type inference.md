# 别名 & 类型推导

- [别名 \& 类型推导](#别名--类型推导)
  - [别名](#别名)
    - [typedef](#typedef)
    - [using](#using)
  - [auto](#auto)
  - [decltype](#decltype)

## 别名

类型别名（Type Alias）是一个名字，它是某种类型的同义词。使用它能使复杂类型变得简单明了、易于理解和使用。

### typedef

传统的定义别名方法是使用typedef关键字。比如：

```c++
typedef unsigned char byte;
typedef long long int ll;
typedef std::pair<int,int> pii;
```

**指针与typedef**：

```c++
typedef char* pchar;
// 定义了一个字符指针
pchar pa;
```

**数组与typedef**：

```c++
typedef int arr[10];
// 定义了一个长度为10的整数数组
arr a1;
```

**函数与typedef**：

```c++
typedef int (pfunc)(int, char*);
// 定义了一个长度为5的函数指针数组，原定义写法 int (f[5])(int, char*)
pfunc f[5];

typedef void (pfunc1)();
typedef void (pfunc2)(pfunc1);
// 定义了一个函数指针，该指针指向的函数需要接收void()()的函数指针参数
pfunc1 f;
```

### using

除了使用typedef关键字定义别名之外，C++还允许使用usign关键字达到同样的目的。比如：

```c++
using BYTE = unsigned char;
```

## auto

auto是C++11引入的关键字，用于自动推断变量的类型。准确来说，auto在C++98时就已经存在，只不过作用是声明变量拥有自动生命周期。C++11更改了它的作用。

当使用auto声明变量时，编译器会根据变量的初始化表达式来推断其类型。例如：

```c++
// 推断x为int
auto x = 5;
```

## decltype

decltype是C++11引入的关键字，用于获取表达式的类型。它可以在编译时获取表达式的静态类型，并将其作为变量的类型或函数返回类型。比如：

```c++
int x = 5;
decltype(x) y;  // y 的类型为 int
```
