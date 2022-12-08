# 贪心算法

- [贪心算法](#贪心算法)
  - [基础概念](#基础概念)
    - [题型分类](#题型分类)
  - [经典例题](#经典例题)
    - [简单题](#简单题)
    - [中等题](#中等题)
    - [难题](#难题)
  - [参考](#参考)

## 基础概念

贪心思想，即找出局部最优解。

### 题型分类

- 区间问题
  - [435 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)
  - [452 用最少的箭射爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)
- 股票问题
- 子序列
- 其他

## 经典例题

### 简单题

[455 分发饼干](https://leetcode.cn/problems/assign-cookies/)

### 中等题

[300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)

思路：

- 此题同样也可以使用动态规划来做，具体参考[本目录动态规划总结](/basical/algorithm/dynamic%20programming.md)
- 贪心思想：
  - 贪心思想在此处可以体现为，尽可能的找到小的数字添加进递增子序列中；
  - 因此，我们可以维护一个数组`d` ，表示长度为`i`的上升子序列的末尾元素的最小值。
  - 每遍历到一个数字，找到数组`d`中大于它的最小值，并将其更新为该数字。
  - 由于数组`d`单调递增，因此可以使用二分法提高查找效率。

代码：

``` c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int len = 1, n = (int)nums.size();
        if (n == 0) {
            return 0;
        }
        vector<int> d(n + 1, 0);
        d[len] = nums[0];
        for (int i = 1; i < n; ++i) {
            if (nums[i] > d[len]) {
                d[++len] = nums[i];
            } else {
                int l = 1, r = len, pos = 0; // 如果找不到说明所有的数都比 nums[i] 大，此时要更新 d[1]，所以这里将 pos 设为 0
                while (l <= r) {
                    int mid = (l + r) >> 1;
                    if (d[mid] < nums[i]) {
                        pos = mid;
                        l = mid + 1;
                    } else {
                        r = mid - 1;
                    }
                }
                d[pos + 1] = nums[i];
            }
        }
        return len;
    }
};
```

[435 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)

[452 用最少的箭射爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)

思路：

一道非常经典的区间问题。只要理解了“为什么按右边界排序，从左往右遍历；而按左边界排序，从右往左遍历”，就能理解这道题的本质了。

代码：

``` c++
class Solution {
 public:
  int findMinArrowShots(vector<vector<int>>& points) {
    int ret = 0;
    sort(
        points.begin(), points.end(),
        [](const vector<int>& a, const vector<int>& b) { return a[0] < b[0]; });

    int i = points.size() - 1;
    while (i >= 0) {
      int arrow = points[i][0];
      int j = i - 1;
      while (j >= 0 && arrow <= points[j][1]) {
        j -= 1;
      }
      i = j;
      ret += 1;
    }

    return ret;
  }
};
```

[646 最长数对链](https://leetcode.cn/problems/maximum-length-of-pair-chain/description/)

思路：和[300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)思路类似，同样是维护一个数组`d` ，表示长度为`i`的上升子序列的末尾数对的最小右边界值。

代码：

``` c++
class Solution {
 public:
  int findLongestChain(vector<vector<int>>& pairs) {
    int ret = 0;
    int n = pairs.size();
    vector<int> d;

    sort(
        pairs.begin(), pairs.end(),
        [](const vector<int>& a, const vector<int>& b) { return a[0] < b[0]; });

    for (int i = 0; i < n; i++) {
      if (d.empty() || d[ret - 1] < pairs[i][0]) {
        d.push_back(pairs[i][1]);
        ret += 1;
        continue;
      }

      int low = 0, high = ret;
      while (low < high) {
        int mid = low + (high - low) / 2;
        if (d[mid] >= pairs[i][0]) {
          high = mid;
        } else {
          low = mid + 1;
        }
      }
      d[low] = min(d[low], pairs[i][1]);
    }
    return ret;
  }
};
```

[738 单调递增的数字](https://leetcode.cn/problems/monotone-increasing-digits/)

[1775 通过最少的次数使数组相等](https://leetcode.cn/problems/equal-sum-arrays-with-minimum-number-of-operations/description/)

### 难题

[354 俄罗斯套娃信封问题](https://leetcode.cn/problems/russian-doll-envelopes/)

## 参考

[代码随想录 贪心算法总结](https://github.com/youngyangyang04/leetcode-master/blob/master/problems/%E8%B4%AA%E5%BF%83%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93%E7%AF%87.md)
