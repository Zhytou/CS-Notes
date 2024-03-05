# 贪心算法

- [贪心算法](#贪心算法)
  - [简介](#简介)
    - [引入](#引入)
    - [解释](#解释)
  - [要点](#要点)
    - [题型分类](#题型分类)
    - [常用技巧](#常用技巧)
  - [例题](#例题)
    - [简单题](#简单题)
    - [中等题](#中等题)
    - [难题](#难题)
  - [参考](#参考)

## 简介

### 引入

贪心算法在有最优子结构的问题中尤为有效。最优子结构的意思是问题能够分解成子问题来解决，子问题的最优解能递推到最终问题的最优解。

可以预见的是，子问题的最优解并不一定为最终解，所以使用贪心法时，一定要证明它，来确保其正确性。

### 解释

**证明**：

贪心算法有两种证明方法：反证法和归纳法。一般情况下，一道题只会用到其中的一种方法来证明。

- 反证法：如果交换方案中任意两个元素/相邻的两个元素后，答案不会变得更好，那么可以推定目前的解已经是最优解了。
- 归纳法：先算得出边界情况（例如 n = 1）的最优解 F_1，然后再证明：对于每个 n，F_{n+1} 都可以由 F_{n} 推导出结果。

**适用范围**：

贪心算法在有最优子结构的问题中尤为有效。最优子结构的意思是问题能够分解成子问题来解决，子问题的最优解能递推到最终问题的最优解。

## 要点

**与动态规划的区别**：

贪心算法与动态规划的不同在于它对每个子问题的解决方案都做出选择，不能回退。动态规划则会保存以前的运算结果，并根据以前的结果对当前进行选择，有回退功能。

### 题型分类

- 跳跃游戏
  - [45 跳跃游戏Ⅱ](https://leetcode.cn/problems/jump-game-ii/description/)
  - [55 跳跃游戏](https://leetcode.cn/problems/jump-game/)
- 区间问题
  - [435 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)
  - [452 用最少的箭射爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)
  - [763 划分字母区间](https://leetcode.cn/problems/partition-labels/description/)
  - [1326 灌溉花园的水龙头最少数目](https://leetcode.cn/problems/minimum-number-of-taps-to-open-to-water-a-garden/description/)
- 最长子序列
  - [300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)
  - [646 最长数对链](https://leetcode.cn/problems/maximum-length-of-pair-chain/description/)
  - [354 俄罗斯套娃信封问题](https://leetcode.cn/problems/russian-doll-envelopes/)
- 其他
  - [738 单调递增的数字](https://leetcode.cn/problems/monotone-increasing-digits/)

### 常用技巧

- 左边界排序+从右到左遍历（右边界排序+从左到右遍历）
  - 见[452 用最少的箭射爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)解法
- 维护数组
  - 见[300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)解法
- 单调栈代替遍历
  - 见[122 买卖股票的最佳时机Ⅱ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)解法二

## 例题

### 简单题

[455 分发饼干](https://leetcode.cn/problems/assign-cookies/)

### 中等题

[45 跳跃游戏Ⅱ](https://leetcode.cn/problems/jump-game-ii/description/)

思路：移动下标只要遇到当前覆盖最远距离的下标，直接步数加一，不考虑是不是终点的情况。

代码：

``` c++
class Solution {
public:
  int jump(vector<int> &nums) {
    int ans = 0;
    int start = 0;
    int end = 1;
    while (end < nums.size())
    {
        int maxPos = 0;
        for (int i = start; i < end; i++)
        {
            // 能跳到最远的距离
            maxPos = max(maxPos, i + nums[i]);
        }
        start = end;      // 下一次起跳点范围开始的格子
        end = maxPos + 1; // 下一次起跳点范围结束的格子
        ans++;            // 跳跃次数
    }
    return ans;
  }
};

class Solution {
 public:
  int jump(vector<int>& nums) {
    int maxPos = 0, n = nums.size(), end = 0, step = 0;
    for (int i = 0; i < n - 1; ++i) {
      if (maxPos >= i) {
        maxPos = max(maxPos, i + nums[i]);
        if (i == end) {
          end = maxPos;
          ++step;
        }
      }
    }
    return step;
  }
};
```

[55 跳跃游戏](https://leetcode.cn/problems/jump-game/)

从右至左遍历，如果能跳到target位置即代表能跳到末尾，不断更新target的值即可。

```c++
class Solution {
public:
    bool canJump(vector<int>& nums) {
        if (nums.empty() || nums.size() == 1) 
            return true;
        
        int target = nums.size() - 1;
        for (int i = nums.size()-2; i >= 0; i--) {
            if (i+nums[i] >= target) {
                target = i;
            }
        }

        return target == 0;
    }
};
```

[122 买卖股票的最佳时机Ⅱ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

思路：

- 解法一：遍历股票价格，只有有递增就继续持有，直到价格下滑，就抛出。
- 解法二：使用单调栈维护股票，减少遍历次数。

代码：

``` c++
// 解法一
class Solution {
 public:
  int maxProfit(vector<int>& prices) {
    int ret = 0;
    int i = 0;
    while (i < prices.size()) {
      int j = i + 1;
      while (j < prices.size() && prices[j - 1] <= prices[j]) {
        j += 1;
      }
      ret += prices[j - 1] - prices[i];
      i = j;
    }
    return ret;
  }
};
// 解法二
class Solution {
 public:
  int maxProfit(vector<int>& prices) {
    int ret = 0;
    stack<int> stk;
    for (auto price : prices) {
      int lastPrice = stk.empty() ? price : stk.top();
      while (!stk.empty() && price > stk.top()) {
        stk.pop();
      }
      if (lastPrice < price) {
        ret += price - lastPrice;
      }
      stk.push(price);
    }
    return ret;
  }
};
```

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
        int l = 1, r = len,
            pos = 0;  // 如果找不到说明所有的数都比 nums[i] 大，此时要更新
                      // d[1]，所以这里将 pos 设为 0
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

思路：类似[452 用最少的箭射爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)

代码：

``` c++
class Solution {
 public:
  int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    int ret = 0;
    sort(
        intervals.begin(), intervals.end(),
        [](const vector<int>& a, const vector<int>& b) { return a[0] < b[0]; });

    int i = intervals.size() - 1;
    while (i >= 0) {
      int j = i - 1;
      while (j >= 0 && intervals[j][1] > intervals[i][0]) {
        j -= 1;
      }
      ret += i - j - 1;
      i = j;
    }
    return ret;
  }
};
```

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

思路：

- 解法一：和[300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)思路类似，同样是维护一个数组`d` 。只不过该数组中`i`位置元素表示的是长度为`i`的上升子序列的末尾数对的最小右边界值。
- 解法二：要挑选最长数对链的第一个数对时，最优的选择是挑选第二个数字最小的，这样能给挑选后续的数对留下更多的空间。按照这样的思路，可以先将输入按照第二个数字排序，然后不停地判断第一个数字是否能满足大于前一个数对的第二个数字即可。

代码：

``` c++
// 解法一
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

// 解法二
class Solution {
public:
    int findLongestChain(vector<vector<int>>& pairs) {
        int curr = INT_MIN, res = 0;
        sort(pairs.begin(), pairs.end(), [](const vector<int> &a, const vector<int> &b) {
            return a[1] < b[1];
        });
        for (auto &p : pairs) {
            if (curr < p[0]) {
                curr = p[1];
                res++;
            }
        }
        return res;
    }
};
```

[738 单调递增的数字](https://leetcode.cn/problems/monotone-increasing-digits/)

思路：很直接明了的一道题。从左至右遍历每位数字，找到第一个开始下降的数字，将其减一，然后将其后续全部设置为9。需要注意的一点是，减一之后这个数之前的序列可能不满足递增了，需要递归减一并检查。

代码：

``` c++
class Solution {
 public:
  int monotoneIncreasingDigits(int n) {
    vector<int> num;
    string sn = to_string(n);
    for (int i = 0; i < sn.size(); i++) {
      if (i != 0 && sn[i - 1] > sn[i]) {
        break;
      }
      num.push_back(sn[i] - '0');
    }

    if (num.size() < sn.size()) {
      for (int i = num.size() - 1; i >= 0; i--) {
        num[i] -= 1;
        if (i > 0 && num[i] >= num[i - 1]) {
          break;
        }
        if (i > 0) {
          num.pop_back();
        }
      }
    }

    int ret = 0;
    for (int i = 0; i < sn.size(); i++) {
      ret *= 10;
      if (i < num.size()) {
        ret += num[i];
      } else {
        ret += 9;
      }
    }
    return ret;
  }
};
```

[1775 通过最少的次数使数组相等](https://leetcode.cn/problems/equal-sum-arrays-with-minimum-number-of-operations/description/)

### 难题

[354 俄罗斯套娃信封问题](https://leetcode.cn/problems/russian-doll-envelopes/)

思路：

- 和[646 最长数对链](https://leetcode.cn/problems/maximum-length-of-pair-chain/description/)思路相同。维护一个数组`d` 数组中`i`位置元素表示的是长度为`i`的上升子序列的末尾数对的最小右边界值。
- 只不过排序有新要求。第一个元素升序排列；在第一个元素相同时，第二个元素降序排列。这是为了避免`[[4,5],[4,6],[4,7]]`这种情况，若不对排序进行修改，则会得到三的错误答案。
- 简单来说，这题就是在限定一个维度的情况下，在另一个维度中寻找最长序列。

代码：

```c++
class Solution {
 public:
  int maxEnvelopes(vector<vector<int>>& envelopes) {
    int ret = 0;
    sort(envelopes.begin(), envelopes.end(),
         [](const vector<int>& a, const vector<int>& b) {
           return a[0] < b[0] || a[0] == b[0] && a[1] > b[1];
         });

    vector<int> d;

    for (auto& envelope : envelopes) {
      if (d.empty() || d[ret - 1] < envelope[1]) {
        d.push_back(envelope[1]);
        ret += 1;
        continue;
      }

      int low = 0, high = ret;
      while (low < high) {
        int mid = low + (high - low) / 2;
        if (d[mid] == envelope[1]) {
          high = mid;
          break;
        } else if (d[mid] > envelope[1]) {
          high = mid;
        } else {
          low = mid + 1;
        }
      }
      d[high] = min(d[high], envelope[1]);
    }

    return ret;
  }
};
```

## 参考

[代码随想录 贪心算法总结](https://github.com/youngyangyang04/leetcode-master/blob/master/problems/%E8%B4%AA%E5%BF%83%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93%E7%AF%87.md)
