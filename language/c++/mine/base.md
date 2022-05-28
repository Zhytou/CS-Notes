# 基础

## 结构体字节数

### 原则

+ 结构体变量中成员的偏移量必须是成员大小的整数倍（数字零被认为是任意整数的公倍数）；
+ 结构体大小必须是所有成员的整数倍数，也即所有成员大小的公倍数。

### 例子

``` c++
struct A {
	char a;
    int b;
}
```

+ 成员a自身对齐值为1，存放的起始地址为0x0000；
+ 成员b自身对齐值为4，存放的起始地址为0x0004；
+ 整个结构体占据8个字节，为所有成员大小整数倍数。

### 作用

+ 按照4字节对齐，可以避免多次I/O

## 联合 Union

一个`union`可以拥有多个数据成员，但在任意时刻只有一个数据成员可以有值。

当我们给`union`的某个成员赋值之后，该`union`的其他成员就变成未定义状态了。

分配给一个`union`对象的存储空间至少要能容纳它的最大数据成员。

默认情况下，`union`的成员都是公有的，这一点与`struct`相同。

## 类型转换

### 隐式转换

#### 算数转换

#### 数值转指针

#### 转常量

### 显式转换

#### 命名的强制类型转换

一个命名的强制类型转换具有如下形式：

`cast-name<type>(expression)`

其中，`type`是类型转换的目标，而`expression`熬转换的值。如果`type`是引用类型，则结果是左值。

**static_cast**

任何具有明确定义的类型转换，只要不包含底层`const`，都可以使用`static_cast`。

``` c++
double slope = static_cast<double>(j) / i;
```

**const_cast**

`const_cast`只能改变运算对象的底层`const`（即：指针指向的对象为`const`）：

``` c++
const char *pc;
char *p = const_cast<char*>(pc);
```

**reinterpret_cast**

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

## const

### 引用与const

### 指针与const

**顶层const**

用名词顶层`const`表示指针本身是个常量。

``` c++
int i  = 1;
int *const p = &i;//p将一直指向i，即顶层const
```

**底层const**

用名词底层`const`表示指针所指对象是个常量。

``` c++
const int *p = 1;//p是一个指向常量的指针，即底层const
```

### 成员函数与const

可以将`const`放在参数列表之后来修饰`this`指针，从而保障`const`变量也能调用该成员函数。

## 引用与指针



## 位运算

### x ^ x = 0 x ^ 0 = x

#### 解释

**x ^ x = 0**

+ 一个数与其自身异或为零。

**x ^ 0 = x**

+ 一个数与零异或为其本身。

#### 例题 [数组中数字出现次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

### x & (x-1)

#### 解释

**x - 1**

+ If you subtract `1` from a number `x` in binary:
  - all the zeroes at the right of `x` become `1`;
  - the rightmost nonzero bit of `x` becomes `0`;
  - other bits are unaffected.

> What about negative numbers? Happily, representation using 2's complement is such that negative numbers behave exactly like positive numbers. In fact, when looking at the bits of `x`, you can subtract `1` from `x` without needing to know whether `x` is a signed int or an unsigned int.

**x & (x - 1) What happened?**

- all the zeroes at the right of `x` remain zero;
- the rightmost 1 of `x` becomes a 0 because of `x-1`;
- all other bits are unaffected, because they are the same in `x` and `x-1`.

Conclusion: we have zeroed the rightmost 1 of `x`, and left all other bits unaffected.

#### 例题 —— [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

**解释**

n & (n−1)，其预算结果恰为把 n 的二进制位中的最低位的 1 变为 0 之后的结果。

**代码**

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

## sizeof



## 参考

**结构体字节数**

+ [结构体字节对齐规则](https://www.cnblogs.com/heart-flying/p/9556401.html)

**类型转换**

+ [C++ Primer 4.11]()

**const**

+ [C++ Primer 2.4]()

**位运算**

+ [how does the formula x & (x - 1) works?](https://stackoverflow.com/questions/64605039/how-does-the-formula-x-x-1-works)

+ [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

**sizeof**

[stackoverflow - Is sizeof evaluated at compilation time or run time?](https://stackoverflow.com/questions/2615203/is-sizeof-in-c-evaluated-at-compilation-time-or-run-time)