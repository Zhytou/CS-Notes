# 类型转换

- [类型转换](#类型转换)
  - [隐式转换](#隐式转换)
    - [算数转换](#算数转换)
    - [指针转换](#指针转换)
    - [explicit](#explicit)
  - [显式转换](#显式转换)
    - [命名的强制类型转换](#命名的强制类型转换)
    - [旧的强制类型转换](#旧的强制类型转换)

## 隐式转换

### 算数转换

算数转换（Arthmetic Conversion）是指把一种算数类型转换为另一种算数类型。

从小整数类型(char、short)转换到int，或者从float转换到double，这种“提升型”的转换通常不会造成数值差异。但是下面的一些情形可能存在一些转换误差，使得编译器产生警告。

- 负数转化为无符号类型，通常会采用二进制补码表示。 (编译器不警告有符号和无符号整数类型之间的隐式转换)
- 无论是转换到bool类型或者是有bool类型进行转换： false等价于0(数值类型)或者空指针(指针类型)； true则等价于其它任何数值或者由true转化为1。
- 浮点数转化为整数会采取截断操作，即移除小数部分。如果转换时发生了数值溢出，可能出现未定义的行为。

### 指针转换

**数组转指针**：大多数用的数组的表达式中，数组都会自动转换成指向数组首元素的指针。比如：

```c++
int arr[10];
int* p = arr + 1;
```

**转常量**：允许指向非常量的指针转换成指向相应常量类型的指针，对于引用也是这样。

除了上述两种转换之外，还包括：

- 空指针可以转换到任意指针类型；
- 任意指针类型都可以转换到void*指针；
- 继承类指针可以转换到可访问的明确的基类指针，同时不改变const或者volatile属性。

### explicit

C++提供了关键字explicit，可以阻止不应该允许的经过转换构造函数进行的隐式转换的发生。即声明为explicit的构造函数不能在隐式转换中使用。

## 显式转换

### 命名的强制类型转换

一个命名的强制类型转换具有如下形式：

`cast-name<type>(expression)`

其中，`type`是类型转换的目标，而`expression`熬转换的值。如果`type`是引用类型，则结果是左值。

**static_cast**:

任何具有明确定义的类型转换，只要不包含底层`const`，都可以使用`static_cast`。

`static_cast`支持的类型转换：

- 原有的自动类型转换，例如`short`转`int`、`int`转`double`、`const`转非`const`、向上转型等；
- `void`指针和具体类型指针之间的转换，例如`void *`转`int *`、`char *`转`void *`等；
- 有转换构造函数或者类型转换函数的类与其它类型之间的转换，例如`double`转`Complex`（调用转换构造函数）、`Complex`转`double`（调用类型转换函数），具体如下代码所示；
- 派生类指针或引用转换为基类。

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

- 两个具体类型指针之间的转换，例如`int *`转`double *`、`Student *`转`int *`等。这种转换需要使用`reinterpret_cast`。
- `int`和指针之间的转换。
- 去掉表达式的 const 修饰和 volatile 修饰。换句话说，不能将 const/volatile 类型转换为非 const/volatile 类型。

**dynamic_cast**:

`dynamic_cast`专门用于将多态基类的指针或引用强制转换为派生类的指针或引用，而且能够检查转换的安全性。对于不安全的指针转换，转换结果返回NULL指针；而对于不安全的引用转换，则会抛出异常。此外，由于`dynamic_cast`会动用运行时信息（RTTI）来进行类型安全检查，因此它存在一定的效率损失。

值得注意的是，`dynamic_cast`只有在基类存在虚函数(虚函数表)的情况下才有可能将基类指针转化为子类。

**const_cast**:

`const_cast`比较好理解，它用来去掉指针或引用的`const`修饰或`volatile`修饰。比如：

``` c++
const char c = 'a'
const char *pc = &c;
char *p = const_cast<char*>(pc);
*p = 'b' // 编译通过，但未C++未定义行为
cout << c << ' ' << *p << endl; // a b 尽管*p = 'b'是允许的，但const c是不会改变的
```

从上面我们可以看到，尽管之后我们定义了一个普通的指针*q。将p指针通过const_cast去掉其常量性，并赋给q指针。之后我再修改q指针所指地址的值时，这是不会有问题的。

**reinterpret_cast**:

`reinterpret_cast`通常为运算对象的位模式提供较低层次上的重新解释，即：可以让编译器把这个变量当成比起原本类型小的类型看待。

``` c++
int ip;
char *pc = reinterpret_cast<char*>(ip);
```

我们必须牢记的是`pc`所指的真实对象是一个整型而非字符，如果强行将`pc`当作一个普通字符指针就可能发生错误。

因此，使用`reinterpret_cast`的风险是非常大的。

### 旧的强制类型转换

早期C++语言中，显式进行强制类型转换主要有两种形式：

``` c++
type (expr);//函数形式的强制类型转换
(type) expr;//C语言风格的强制类型转换
```
