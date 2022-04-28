# 投票算法

## Boyer-Moore 投票算法

### 概述

摩尔投票算法的基本原理是：每次从序列里选择两个不相同的数字删除掉（或称为“抵消”），最后剩下一个数字或几个相同的数字，就是出现次数大于总数一半的那个。

### 例题

#### [数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

**算法**

+ 我们维护一个候选众数 `candidate` 和它出现的次数 `count`。初始时 `candidate` 可以为任意值，`count` 为 0；

+ 我们遍历数组 `nums` 中的所有元素，对于每个元素 x，在判断 x 之前，如果 `count` 的值为 0，我们先将 x 的值赋予 `candidate`，随后我们判断 x：
  - 如果 x 与 `candidate` 相等，那么计数器 `count` 的值增加 1；
  - 如果 x 与 `candidate` 不等，那么计数器 `count` 的值减少 1。
+ 在遍历完成后，`candidate` 即为整个数组的众数。

**代码**

``` c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int candidate = -1;
        int count = 0;
        for (int num : nums) {
            if (num == candidate)
                ++count;
            else if (--count < 0) {
                candidate = num;
                count = 1;
            }
        }
        return candidate;
    }
};
```

## 参考

+ [如何理解摩尔投票算法](https://www.zhihu.com/question/49973163)