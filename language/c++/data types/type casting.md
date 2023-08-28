# 类型转换

## 隐式转换

- 算数转换

- 数值转指针

- 转常量

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

### 旧的强制类型转换

早期C++语言中，显式进行强制类型转换主要有两种形式：

``` c++
type (expr);//函数形式的强制类型转换
(type) expr;//C语言风格的强制类型转换
```
