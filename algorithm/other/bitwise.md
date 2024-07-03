# 位运算

- [位运算](#位运算)
  - [异或](#异或)
  - [gcc内置位运算函数](#gcc内置位运算函数)
  - [bitset](#bitset)
  - [参考](#参考)

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

## gcc内置位运算函数

> 更详细的介绍可以查看GeeksforGeeks的文章[Builtin functions of GCC compiler](https://www.geeksforgeeks.org/builtin-functions-gcc-compiler/)或者GNU的GCC文档[6.63 Other Built-in Functions Provided by GCC](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)。

- **__builtin_popcount()**：计算传入数字中二进制表示法中1的数量。比如，`__builtin_popcount(4)`结果为1。
- **__builtin_clz()**：计算整型数字中前导0的数量，其中clz是count leading zero的缩写。比如，`__builtin_clz(16)`结果为27。
- **__builtin_ctz()**：计算整型数字中尾随0的数量，其中ctz是count trailing zero的缩写。比如，`__builtin_clz(16)`结果为4。

## bitset

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
```

## 参考

- [how does the formula x & (x - 1) works?](https://stackoverflow.com/questions/64605039/how-does-the-formula-x-x-1-works)
- [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)
