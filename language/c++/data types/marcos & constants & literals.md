# 常量与字面量

- [常量与字面量](#常量与字面量)
  - [宏](#宏)
    - [宏函数](#宏函数)
    - [宏的缺陷](#宏的缺陷)
  - [宏的替换机制](#宏的替换机制)
    - [内联函数](#内联函数)
    - [枚举](#枚举)
    - [常量表达式](#常量表达式)
  - [常量](#常量)
    - [const](#const)
    - [constexpr](#constexpr)
  - [字面量](#字面量)
    - [整型字面量](#整型字面量)
    - [浮点型字面量](#浮点型字面量)
    - [字符字面量](#字符字面量)
    - [字符串字面量](#字符串字面量)
    - [自定义字面量](#自定义字面量)

## 宏

宏（Macro）是一种在编程中用来进行代码替换的预处理指令。宏定义了一段代码片段，可以在程序中多次使用，并且在编译阶段通过预处理器进行替换。

在实际开发中，我们常常使用宏定义一些常量、枚举量或者简单函数。比如：

```c++
#define PI 3.14159
#define WOKR_MODE_0 0
#define WORK_MODE_1 1
#define MAX(a, b) ((a) > (b) ? (a) : (b))
```

除此之外，宏还能用达成类似命名空间的功能，比如：

```c++
#ifndef COMDEF_H
#define COMDEF_H

//头文件内容 …
#endif
```

### 宏函数

在C/C++中，宏函数（Macro Function）是一种在预处理阶段展开的宏定义，可以用来简化代码、实现代码重用以及避免重复性的代码书写。它的格式一般如下：

```c++
#define MACRO_FUNCTION_NAME(parameter1, parameter2, ...) replacement_code
```

值得注意的是，宏函数通常是不包含返回值的。它和一般意义上的C++函数不同，它只是一个简单的替换。常见的宏函数实现包括：

```c++
#define  MAX( x, y )  ( ((x) > (y)) ? (x) : (y) )
#define  MIN( x, y )  ( ((x) < (y)) ? (x) : (y) )

#define ADD(x, y) ((x) + (y))

#define  HEXCHK( ch ) \
(((ch) >= ’0′ && (ch) <= ’9′) || \
((ch) >= ’A' && (ch) <= ’F') || \
((ch) >= ’a' && (ch) <= ’f') )

#define GETMID(nums, mid) {\
    sort(nums.begin(), nums.end());\
    mid = nums[nums.size()/2];\
}
```

此外，在编写宏函数时，有一些技巧能够避免错误，比如：

- 使用小括号包含；
- 用do{}while(0)语句包含多语句防止错误；
- 使用\定义多行宏函数；
- 使用#把宏参数变为一个字符串，用##把两个宏参数贴合在一起。

其中，需要特别强调的是第二条。可能你会觉得单独使用{}同样可以达到效果，为什么一定要使用do{}while(0)呢？下面就是一个例子。

```c++
#define switch(x,y) {int tmp; tmp=x;x=y;y=tmp;}
if(x>y)
  switch(x,y);
else        //error, parse error before else
  otheraction();
```

在把宏引入代码中，会多出一个分号，从而会报错。使用do{….}while(0) 把它包裹起来，成为一个独立的语法单元，从而不会与上下文发生混淆。同时因为绝大多数的编译器都能够识别do{…}while(0)这种无用的循环并进行优化，所以使用这种方法也不会导致程序的性能降低，

### 宏的缺陷

因为宏只进行替换，不能进行类型检查，同时也无法限制其作用域。

## 宏的替换机制

尽管宏定义常量和函数非常方便，但为了避免其带来的安全隐患，我们仍然倾向于使用C++提供的其他安全机制来来达成相同的目的，比如：内联函数、枚举和常量表达式等。

### 内联函数

内联函数是通过inline关键字定义的函数，一般来说类内定义的函数默认是内联的。它用于请求编译器将函数的定义内联展开。也就是说，编译完成后，内联函数的调用位置直接插入函数体机器码，而不是使用跳转指令。通过内联展开的方式，程序就能避免函数参数压栈/寄存器、CPU执行流切换、返回值存储等资源消耗的问题。值得一提的是，inline仅仅是对编译器的一种请求，编译器可以无视。

**和宏的区别**：

尽管从内联函数的本质就是替换上来看，它和宏非常相似，但二者还是有三点的区别：

- 第一，宏定义在预处理阶段进行代码替换，而内联函数是在编译阶段插入代码；
- 第二，宏定义没有类型检查，而内联函数有类型检查，这对于写出正确且鲁棒的程序是一个很大的优势；
- 最后，宏肯定会被展开，而用inline关键字修饰的函数不一定会被内联展开。

**内联的适用场景**：

一般来说，内联函数也通常是针对逻辑简单的小函数，而不适合逻辑复杂的大函数和递归函数。

因为小函数的的调用切换时间往往大于其执行时间，所以适合使用内联优化。而大函数使用内联将导致可执行代码膨胀过大，得不偿失。至于递归函数，则可能会引起部分编译器的无穷编译。

**inline和static**：

当多个源文件包含同一个inline函数，但是编译器又无视inline请求，如此一来，在每个目标模块中都有该函数的实现实例，造成重定义错误。把inline函数同时声明为static函数（限制其作用域为源文件内部），可以解决问题。

### 枚举

C++11引入了强类型枚举（Enum Class）。它可以定义一组具名的常量值，提供更好的类型安全性和代码可读性。比如：

```c++
enum class Color { black, white, red, blue };
Color c = Color::black; 
```

### 常量表达式

C++11引入了constexpr关键字，用于声明可以在编译时求值的函数或变量。const和constexpr都可以在许多场景中替代宏定义的常量，提供类型安全性和更好的编译时检查。

除此之外，constexpr的编译期求值特性使它广泛引用于元编程中，并逐步形成了"Constexpr all the things"的思想。关于这一点，可以查看constexpr metaprogramming笔记中的内容。

## 常量

在C/C++语言中，有两种简单的定义常量的方式：

- 使用#define预处理指令定义一个宏： #define可以在程序中定义一个常量，它在编译时会被替换为其对应的值。
- 使用const关键字：const 关键字用于声明一个只读变量，即该变量的值不能在程序运行时修改。

### const

**变量与const**：

在C/C++语言中，const变量是一种特殊的变量，因为它们的值不可以改变。

const变量在声明时可以选择进行初始化也可以选择不进行初始化。值得注意的是，如果不进行初始化，则无法在后续的代码中给它赋值。

const变量的初始化可以发生在程序运行时或编译时，比如下面两种情况都是正确的。

```c++
const int i = get_size(); // 运行时初始化
const int j = 42; // 编译时初始化
```

**指针与const**：

顶层`const`表示指针本身是个常量。

``` c++
int i  = 1;
int *const p = &i;//p将一直指向i，即顶层const
```

底层`const`表示指针所指对象是个常量。

``` c++
const int *p = 1;//p是一个指向常量的指针，即底层const
```

**成员变量与const**：

值得注意的是，初始化const成员变量只有一种方法，就是通过构造函数的初始化列表。而static成员变量则是在类外初始化。

**成员函数与const**：

可以将`const`放在参数列表之后来修饰`this`指针，从而保障`const`变量也能调用该成员函数。

### constexpr

constexpr用于声明可在编译时求值的常量表达式，其值在编译时确定。

**变量与constexpr**：

对于constexpr变量来说，必须用字面量或者constexpr函数初始化。

**函数与constexpr**：

constexpr函数可以用于计算常量表达式。但它也要求函数体中的操作必须是可求值的常量表达式，且不能包含具有副作用的操作（如 I/O 操作），并且参数类型和返回类型必须满足一定的要求。
此外，编译器通常会尝试对 constexpr 函数进行内联展开，将函数调用替换为函数体的内容，以减少函数调用的开销。

**const与constexpr**：

其实，const并不能代表“常量”，它仅仅是对变量的一个修饰，告诉编译器这个变量只能被初始化，且不能被直接修改（实际上可以通过堆栈溢出等方式修改）。而这个变量的值，可以在运行时也可以在编译时指定。

constexpr可以用来修饰变量、函数、构造函数。一旦以上任何元素被constexpr修饰，那么等于说是告诉编译器 “请大胆地将我看成编译时就能得出常量值的表达式去优化我”。

## 字面量

在编程语言中，字面量（Literals）是用于表示数据的固定值或常量。字面量可以直接在代码中使用，而不需要进行计算或求值。下面分别介绍一些常用的字面量。

### 整型字面量

通过添加`0b`、`0`、`0x`等前缀说明数字的进制：

- `0b`或`0B`表示二进制
- `0`表示八进制
- 无前缀表示十进制
- `0x`或`0X`表示十六进制

通过添加`U`、`L`等后缀说明数字的类型：

- int：无后缀
- unsigned int：`u`或`U`
- long int：`l`或`L`
- unsigned long int：`ul`或`UL`
- long long int：`ll`或`LL`
- unsigned long long int：`ull`或`ULL`

### 浮点型字面量

浮点型字面量分为两种表示方法：

- 小数点形式：必须包含整数部分、小数部分，或同时包含两者
- 指数形式：必须包含小数点、指数，或同时包含两者。

此外，浮点型字面量还可以使用后缀`f`或`F`表示常量为float类型，无后缀表示double类型。

### 字符字面量

字符字面量使用前缀区分存储类型：

- 无前缀：char类型
- 有`L`或`l`前缀：wchar_t类型
- 有`u`前缀：char16_t类型
- 有`U`前缀：char32_t类型

### 字符串字面量

字符串字面量与字符字面量类似，也是很通过前缀区分存储类型：

- 无前缀：char类型
- 有`L`或`l`前缀：wchar_t类型
- 有`u`前缀：char16_t类型
- 有`U`前缀：char32_t类型

### 自定义字面量

除了上面提到的几种字面量之外，C++11还引入了一种称为用户自定义字面量（User-Defined Literals）的机制。它允许程序员定义自己的字面量形式，以扩展语言的表达能力。

一般来说，用户自定义字面量由两部分组成：字面量运算符（Literal Operator）和字面量标识符（Literal Identifier）。

- 字面量运算符是一个重载的函数，用于处理自定义字面量的解释和计算。它以特殊的形式出现在字面量之后，可以是后缀形式或原生形式。
- 字面量标识符是一个标识符，用于标识自定义字面量的类型和含义。

下面是一个使用自定义字面量进行单位转换的例子。

``` c++
//User defined literals
constexpr long double operator"" _cm(long double x) {return x * 10; }
constexpr long double operator"" _m(long double x) {return x * 1000; }
constexpr long double operator"" _mm(long double x) {return x; }

int main() {
    long double height = 3.4_cm;
    cout << (height + 13.0_m) << endl;
    cout << (130.0_mm / 13.0_m) << endl;
}
```
