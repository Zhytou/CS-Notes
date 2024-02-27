# 位运算

- [位运算](#位运算)
  - [异或](#异或)
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

## 参考

- [how does the formula x & (x - 1) works?](https://stackoverflow.com/questions/64605039/how-does-the-formula-x-x-1-works)
- [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)
