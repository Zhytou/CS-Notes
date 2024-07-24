# 位运算

- [位运算](#位运算)
  - [整型](#整型)
    - [整型的表示——补码](#整型的表示补码)
    - [整型加减](#整型加减)
    - [二进制乘除](#二进制乘除)
  - [浮点数](#浮点数)
  - [异或](#异或)
  - [C/C++语言常用的位运算手段](#cc语言常用的位运算手段)
    - [gcc内置位运算函数](#gcc内置位运算函数)
    - [bitset](#bitset)
  - [参考](#参考)

## 整型

整型是计算机中用于表示整数的数据类型，通常分为有符号整型和无符号整型两种。有符号整型可以表示正数、负数和零，而无符号整型只能表示非负数（即正数和零）。

### 整型的表示——补码

补码是一种表示有符号整数的方法，也是计算机中常用的表示负数的方式。在补码表示法中，正数的补码等于其原码，负数的补码是其对应正数的按位取反再加1的结果。例如，对于8位二进制数，-3的原码为10000011，其补码为11111101。

补码的优点是可以简化负数的加减运算，将减法运算转换为加法运算。在计算机中，通常使用补码表示负数，便于进行数值运算。

### 整型加减

[溢出](https://blog.csdn.net/Baoli1008/article/details/46687327)

### 二进制乘除

二进制乘法和除法是在计算机中进行整数运算的基本操作，与十进制乘法和除法类似，但是操作对象是二进制数。

例如：

```text
110
x 101
------
  110    (110 * 1)
 000     (110 * 0，向左移位)
110      (110 * 1，向左移位)
------
11110   (结果为30)

1010
-------
  11 | 1010
      -11
      -----
       10
```

## 浮点数

浮点数是计算机中用于表示实数（包括小数）的数据类型。浮点数通常使用IEEE 754标准来表示，通过指数和尾数的组合来表示实数的近似值。IEEE 754标准定义了单精度浮点数（32位）和双精度浮点数（64位），分别用于表示较小范围和较大范围的实数值。浮点数的表示形式包括符号位、指数位和尾数位。符号位表示正负号，指数位表示指数部分，尾数位表示有效数字部分。

## 异或

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

## C/C++语言常用的位运算手段

除了最基本的位运算符之外，C/C++语言还提供一些简单实用的工具帮助程序员处理位运算问题。

### gcc内置位运算函数

> 更详细的介绍可以查看GeeksforGeeks的文章[Builtin functions of GCC compiler](https://www.geeksforgeeks.org/builtin-functions-gcc-compiler/)或者GNU的GCC文档[6.63 Other Built-in Functions Provided by GCC](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)。

- **__builtin_popcount()**：计算传入数字中二进制表示法中1的数量。比如，`__builtin_popcount(4)`结果为1。
- **__builtin_clz()**：计算整型数字中前导0的数量，其中clz是count leading zero的缩写。比如，`__builtin_clz(16)`结果为27。
- **__builtin_ctz()**：计算整型数字中尾随0的数量，其中ctz是count trailing zero的缩写。比如，`__builtin_clz(16)`结果为4。

### bitset

位数组(bitset)是C++标准库中的位集合容器，它提供了一种方便操作和存储位级数据的机制。它类似布尔类型的数组，但它每个元素只占一个bit，且大小在编译期确定不能修改。常见操作包括：

```c++
// 创建bitset
std::bitset<16> bs;
// 将所有位设置位1
bs.set();
// 重置所有位
bs.resst();
// 设置指定位
bs[2] = 1; //  or bs.set(2);
// 清除位
bs.reset(2);
// 获取位
bool b = bs[2];
// 获取位数量
int cnt = bs.count();
// 直接进行位运算
std::bitset<16> bs2, bs3;
bs3 = bs&bs2;
```

## 参考

- [how does the formula x & (x - 1) works?](https://stackoverflow.com/questions/64605039/how-does-the-formula-x-x-1-works)
- [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)
