# 动态规划

- [动态规划](#动态规划)
  - [基础概念](#基础概念)
    - [常用解法](#常用解法)
      - [01背包问题](#01背包问题)
      - [完全背包问题](#完全背包问题)
    - [题型分类](#题型分类)
  - [经典例题](#经典例题)
    - [简单题](#简单题)
    - [中等题](#中等题)
    - [难题](#难题)

## 基础概念

### 常用解法

#### 01背包问题

**问题描述**：

有N件物品和一个容量为M的背包。第i件物品的重量是w[i]，价值是v[i]。求解将哪些物品装入背包可使这些物品的重量总和不超过背包容量，且价值总和最大。

**朴素解法**：

将容量为`j`装入`i`个物品的背包最大价值记为`dp[i][j]`，则转移方程为：`dp[i] = max(dp[i], dp[i - 1][j - w[i]] + v[i])`，将该问题转换为如何在已经装入`i-1`个物品的背包是再装一个物品获得最大价值。

``` c++
// weight数组的大小 就是物品个数
for(int i = 1; i < weight.size(); i++) { // 遍历物品
  for(int j = 0; j <= bagweight; j++) { // 遍历背包容量
    if (j < weight[i]){
      dp[i][j] = dp[i - 1][j];
    } 
    else {
      dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
    }
  }
}
```

**压缩解法**：

对于背包问题其实状态都是可以压缩的。我们可以使用一维数组，也可以理解是一个滚动数组来代替原本的二维数组。

``` c++
for (int i = 0; i < weight.size(); i++) {         // 遍历物品
  for (int j = bagWeight; j >= weight[i]; j--) {  // 遍历背包容量
    dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
  }
}
```

注意：使用压缩解法的时候，需要倒序遍历，否则小容量背包的dp值会被先覆盖。

#### 完全背包问题

完全背包和01背包问题唯一不同的地方就是，每种物品有无限件。

### 题型分类

按内容

- 01 背包问题
  - [416 分割等和的子集](https://leetcode.cn/problems/partition-equal-subset-sum/description/)
  - [474 一和零](https://leetcode.cn/problems/ones-and-zeroes/description/)
  - [494 目标和](https://leetcode.cn/problems/target-sum/description/)
- 完全背包问题
- 股票问题
  - [121 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)
  - [122 买卖股票的最佳时机Ⅱ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)
  - [123 买卖股票的最佳时机Ⅲ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)
  - [188 买卖股票的最佳时机Ⅳ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)
  - [309 买卖股票的最佳时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
  - [713 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
- 子序列
  - 不连续
    - [300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)
  - 连续
  - 编辑距离
- 其他
  - [96 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/description/)

按解法

- 一维dp
- 二维dp

## 经典例题

### 简单题

[121 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

### 中等题

[64 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

代码

  ``` c++
  class Solution {
  public:
      int minPathSum(vector<vector<int>>& grid) {
          if (grid.size() == 0 || grid[0].size() == 0) {
              return 0;
          }
          int rows = grid.size(), columns = grid[0].size();
          auto dp = vector < vector <int> > (rows, vector <int> (columns));
          dp[0][0] = grid[0][0];
          for (int i = 1; i < rows; i++) {
              dp[i][0] = dp[i - 1][0] + grid[i][0];
          }
          for (int j = 1; j < columns; j++) {
              dp[0][j] = dp[0][j - 1] + grid[0][j];
          }
          for (int i = 1; i < rows; i++) {
              for (int j = 1; j < columns; j++) {
                  dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
              }
          }
          return dp[rows - 1][columns - 1];
      }
  };
  ```

[96 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/description/)

代码：

``` c++
class Solution {
  vector<int> dp;

 public:
  int numTrees(int n) {
    if (n >= dp.size()) {
      for (int i = dp.size(); i <= n; i++) {
        if (i == 0 || i == 1) {
          dp.push_back(i);
          continue;
        }
        int dpi = 0;
        for (int root = 1; root <= i; root++) {
          int left = root - 1 == 0 ? 1 : dp[root - 1];
          int right = i - root == 0 ? 1 : dp[i - root];
          dpi += left * right;
        }
        dp.push_back(dpi);
      }
    }

    return dp[n];
  }
};
```

[122 买卖股票的最佳时机Ⅱ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

思路：

- 分别记录第i天交易完后手里没有股票的最大利润和手里持有一支股票的最大利润。
- 返回最后一天不持有股票的最大利润即可。

代码：

``` c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int dp[n][2];
        dp[0][0] = 0, dp[0][1] = -prices[0];
        for (int i = 1; i < n; ++i) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[n - 1][0];
    }
};
```

[300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)

代码：

``` c++
class Solution {
 public:
  int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, 1);

    for (int i = nums.size() - 1; i >= 0; i--) {
      int dpj = 0;
      for (int j = nums.size() - 1; j > i; j--) {
        if (nums[i] < nums[j]) {
          dpj = max(dpj, dp[j]);
        }
      }
      dp[i] += dpj;
    }

    int ret = 1;
    for (int i = 0; i < dp.size(); i++) {
      ret = max(ret, dp[i]);
    }
    return ret;
  }
};
```

[416 分割等和的子集](https://leetcode.cn/problems/partition-equal-subset-sum/description/)

思路：

- 背包的体积为sum / 2。
- 背包要放入的商品（集合里的元素）重量为元素的数值，价值也为元素的数值。
- 背包如果正好装满，说明找到了总和为 sum / 2 的子集。
- 背包中每一个元素是不可重复放入。

注意：需要注意的是第二层的循环我们需要从大到小计算，因为如果我们从小到大更新 dp 值，那么在计算较大序号dp值的时候，较小序号dp值已经是被更新过的状态，不再是上一行的dp值。

代码：

``` c++
class Solution {
 public:
  bool canPartition(vector<int>& nums) {
    int sum = 0;
    for (int i = 0; i < nums.size(); i++) {
      sum += nums[i];
    }

    if (sum % 2 == 1) return false;
    int target = sum / 2;

    vector<int> dp(target + 1, 0);
    // 开始 01背包
    for (int i = 0; i < nums.size(); i++) {
      for (int j = target; j >= nums[i]; j--) {
        dp[j] = max(dp[j], dp[j - nums[i]] + nums[i]);
      }
    }
    // 集合中的元素正好可以凑成总和target
    if (dp[target] == target) return true;
    return false;
  }
};
```

[474 一和零](https://leetcode.cn/problems/ones-and-zeroes/description/)

思路：十分类似经典的背包问题，只不过背包容量是两个维度（1的数量和0的数量）上统计，且所有物品（字符串）的价值均为一。因此，只需将原本的一维数组转换成二维数组，即可得到新的状态转移方程如下：

``` c++
// dp[i][j]表示数字0容量为i、数字1容量为j时，背包能存放字符串的最大数量（价值）
dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1);
```

代码：

``` c++
class Solution {
 public:
  int findMaxForm(vector<string>& strs, int m, int n) {
    int len = strs.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int k = 0; k < len; k++) {
      int ones = 0, zeros = 0;
      for (auto& ch : strs[k]) {
        if (ch == '1') {
          ones += 1;
        } else {
          zeros += 1;
        }
      }

      for (int i = m; i >= zeros; i--) {
        for (int j = n; j >= ones; j--) {
          dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1);
        }
      }
    }

    return dp[m][n];
  }
};
```

[494 目标和](https://leetcode.cn/problems/target-sum/description/)

思路：

- 本题直接思路应该是回溯，此处略过；
- 想要使用动态规划解决本题，首先需要找到转移方程。
  - 根据数学推导得出，加负号的数字和可以表达为：`negative = (sum - target)/2`；
  - 因此，整个问题可以转换成，如何找出所有价值刚刚好为`(sum - target)/2`的物品组合；

代码：

``` c++
class Solution {
 public:
  int findTargetSumWays(vector<int>& nums, int target) {
    int sum = 0;
    for (int& num : nums) {
      sum += num;
    }
    int diff = sum - target;
    if (diff < 0 || diff % 2 != 0) {
      return 0;
    }
    int neg = diff / 2;
    vector<int> dp(neg + 1);
    dp[0] = 1;
    for (int& num : nums) {
      for (int j = neg; j >= num; j--) {
        dp[j] += dp[j - num];
      }
    }
    return dp[neg];
  }
};
```

### 难题

[123 买卖股票的最佳时机Ⅲ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

代码：

``` c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int buy1 = -prices[0], sell1 = 0;
        int buy2 = -prices[0], sell2 = 0;
        for (int i = 1; i < n; ++i) {
            buy1 = max(buy1, -prices[i]);
            sell1 = max(sell1, buy1 + prices[i]);
            buy2 = max(buy2, sell1 - prices[i]);
            sell2 = max(sell2, buy2 + prices[i]);
        }
        return sell2;
    }
};
```

[688 骑士在棋盘上的概率](https://leetcode-cn.com/problems/knight-probability-in-chessboard/)

- 思路：
  - 一个骑士有 8 种可能的走法，骑士会从中以等概率随机选择一种。部分走法可能会让骑士离开棋盘，另外的走法则会让骑士移动到棋盘的其他位置，并且剩余的移动次数会减少 1。
  - 因此，定义`dp\[step]\[i][j]`表示骑士从棋盘上的点(i, j)出发，走了step步时的概率
    - 特别地，当点(i, j)不在棋盘上时，`dp\[step]\[i][j]`= 0；
    - 当step = 0时， `dp\[step]\[i][j] = 1`；
    - 对于其他情况，`dp\[step]\[i][j]`为`dp\[step - 1]\[i][j]`之和除以 8 。
  - 其中， dp\[step - 1]有八种坐标偏移量，具体见题

- 代码：

  ``` c++
  // 递归，超时
  class Solution {
  public:
      inline bool ifInZone(const int& n, const int& x, const int& y) {
          return 0 <= x && x < n && 0 <= y && y < n;
      }
  
      double knightProbability(const int& n, int k, int& x, int& y) {
          if (k == 0)
              return ifInZone(n, x, y) ? 1 : 0;
          if (!ifInZone(n, x, y))
              return 0;
          double p = 0;
          for (int i = 0; i < 8; i++) {
              int dx = directions[i][0], dy = directions[i][1];
              x += dx;
              y += dy;
              p += knightProbability(n, k - 1, x, y);
              x -= dx;
              y -= dy;
          }
          return p / 8;
      }
  private:
      int directions[8][2] = {{1, 2}, {2, 1}, {2, -1}, {1, -2}, {-1, -2}, {-2, -1}, {-2, 1}, {-1, 2}};
  };
  
  
  // 动态规划解法，不超时
  class Solution {
  public:
      vector<vector<int>> dirs = {{-2, -1}, {-2, 1}, {2, -1}, {2, 1}, {-1, -2}, {-1, 2}, {1, -2}, {1, 2}};
  
      double knightProbability(int n, int k, int row, int column) {
          vector<vector<vector<double>>> dp(k + 1, vector<vector<double>>(n, vector<double>(n)));
          for (int step = 0; step <= k; step++) {
              for (int i = 0; i < n; i++) {
                  for (int j = 0; j < n; j++) {
                      if (step == 0) {
                          dp[step][i][j] = 1;
                      } else {
                          for (auto & dir : dirs) {
                              int ni = i + dir[0], nj = j + dir[1];
                              if (ni >= 0 && ni < n && nj >= 0 && nj < n) {
                                  dp[step][i][j] += dp[step - 1][ni][nj] / 8;
                              }
                          }
                      }
                  }
              }
          }
          return dp[k][row][column];
      }
  };
  
  ```
