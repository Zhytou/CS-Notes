# 动态规划

- [动态规划](#动态规划)
  - [常用解法](#常用解法)
    - [01背包](#01背包)
    - [完全背包](#完全背包)
  - [题型分类](#题型分类)
    - [一维dp](#一维dp)
    - [二维dp](#二维dp)
    - [二维dp的优化——滚动数组](#二维dp的优化滚动数组)
    - [树形dp](#树形dp)
  - [经典例题](#经典例题)
    - [01背包问题](#01背包问题)
    - [完全背包问题](#完全背包问题)
    - [股票问题](#股票问题)
    - [树形DP](#树形dp-1)
    - [子序列问题](#子序列问题)
    - [博弈策略问题](#博弈策略问题)
    - [其他](#其他)

动态规划的题目分为两大类，一种是求最优解类，典型问题是背包问题，另一种就是计数类，比如典型的统计可行方案数量，它们都存在一定的递推性质。

前者的递推性质还有一个名字，叫做 「最优子结构」 ——即当前问题的最优解取决于子问题的最优解，后者类似，当前问题的方案数取决于子问题的方案数。所以在遇到求方案数的问题时，我们可以往动态规划的方向考虑。

## 常用解法

### 01背包

**问题描述**：

有N件物品和一个容量为M的背包。第i件物品的重量是w[i]，价值是v[i]。求解将哪些物品装入背包可使这些物品的重量总和不超过背包容量，且价值总和最大。

**朴素解法**：

将容量为`j`装入`i`个物品的背包最大价值记为`dp[i][j]`，则转移方程为：`dp[i][j] = max(dp[i-1][j], dp[i - 1][j - w[i]] + v[i])`，将该问题转换为如何在已经装入`i-1`个物品的背包是再装一个物品获得最大价值。

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

### 完全背包

完全背包和01背包问题唯一不同的地方就是，每种物品有无限件。

完全背包问题需要注意先遍历物品还是背包，例子可以参考[518 零钱兑换Ⅱ](https://leetcode.cn/problems/coin-change-ii/description/)。

## 题型分类

按**内容**，可以分为背包问题、子序列问题、股票问题、路线问题以及较为特殊的博弈和树形问题。

- 01 背包问题
  - [416 分割等和的子集](https://leetcode.cn/problems/partition-equal-subset-sum/description/)
  - [474 一和零](https://leetcode.cn/problems/ones-and-zeroes/description/)
  - [494 目标和](https://leetcode.cn/problems/target-sum/description/)
  - [1049 最后一个石头的重量Ⅱ](https://leetcode.cn/problems/last-stone-weight-ii/description/)
- 完全背包问题
  - [322 零钱兑换](https://leetcode.cn/problems/coin-change/description/)
  - [518 零钱兑换Ⅱ](https://leetcode.cn/problems/coin-change-ii/description/)
- 股票问题
  - [121 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)
  - [122 买卖股票的最佳时机Ⅱ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)
  - [123 买卖股票的最佳时机Ⅲ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)
  - [188 买卖股票的最佳时机Ⅳ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)
  - [309 买卖股票的最佳时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
  - [713 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
- 打家劫舍问题
- 树形dp/换根dp
  - [834 树中距离之和](https://leetcode.cn/problems/sum-of-distances-in-tree/description/)
  - [2581 统计可能的树根数目](https://leetcode.cn/problems/count-number-of-possible-root-nodes/description/)
- 路线
  - [62 不同的路径](https://leetcode.cn/problems/unique-paths/description/)
  - [63 不同的路径Ⅱ](https://leetcode.cn/problems/unique-paths-ii/)
  - [2684 矩阵中移动的最大次数](https://leetcode.cn/problems/maximum-number-of-moves-in-a-grid/)
- 子序列
  - 不连续
    - [300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)
    - [1035 不相交的线](https://leetcode.cn/problems/uncrossed-lines/)
    - [1143 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/description/)
  - 连续
    - [53 最大子序列和](https://leetcode.cn/problems/maximum-subarray/)
    - [152 最大子数组乘积](https://leetcode.cn/problems/maximum-product-subarray/description/)
    - [674 最长连续递增序列](https://leetcode.cn/problems/longest-continuous-increasing-subsequence/description/)
    - [718 最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/description/)
    - [918 环形子数组的最大和](https://leetcode.cn/problems/maximum-sum-circular-subarray/)
  - 其他
    - [72 编辑距离](https://leetcode.cn/problems/edit-distance/description/)
- 博弈策略问题
  - [486 预测赢家](https://leetcode.cn/problems/predict-the-winner/)
  - [877 石子游戏](https://leetcode.cn/problems/stone-game/description/)
- 其他
  - [96 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/description/)
  - [97 交错字符串](https://leetcode.cn/problems/interleaving-string/description/)
  - [1130 叶值的最小代价生成树](https://leetcode.cn/problems/minimum-cost-tree-from-leaf-values/description/)

按**解法**，可以分为一维dp、二维dp和树形dp等。

### 一维dp

最简单的一维dp，转移方程如下：

```c++
dp[i] = max(nums[i], dp[i-1]+nums[i]);
```

上述转移方程也是[53 最大子序列和](https://leetcode.cn/problems/maximum-subarray/)这道题的解答方法。其中最常见的变式，就是引入除了`dp[i-1]`之外的另一个状态，比如[152 最大子数组乘积](https://leetcode.cn/problems/maximum-product-subarray/description/)，就需要分别使用`dp[i][1]` 和`dp[i][0]`表示：以 nums[i] 结尾的连续子序列的乘积的最大值和最小值。

### 二维dp
  
最常见的二维dp转移方程如下，这也是1035和1143这两道题的解法：

```c++
if (text1[i] == text[j]) {
  dp[i][j] = dp[i - 1][j - 1] + 1;
} else {
  dp[i][j] = max(dp[i][j - 1], dp[i - 1][j]);
}
```

### 二维dp的优化——滚动数组

滚动数组思想是一种常见的动态规划空间优化方法。当动态规划的转移方程中只和某几个状态相关的时候，我们定义的状态在，就可以考虑这种优化方法，目的是给空间复杂度降维。比如：原状态转移方程为`dp[i][j] = dp[i-1][j]+dp[i][j-1]`，我们就可以使用滚动数组思想则可将其优化成`dp[j] += dp[j-1]`，因为`dp[j]`在修改前就存放着原本所需的`dp[i-1][j]`。

更多例子包括：

- [63 不同路径Ⅱ](https://leetcode.cn/problems/unique-paths-ii/description/)
- [120 最短三角形路径](https://leetcode.cn/problems/triangle/description/)
- [221 最大正方形](https://leetcode.cn/problems/maximal-square/description/)

### 树形dp

树形dp，也称为换根dp，其详细介绍可以参考[这篇文章](https://leetcode.cn/problems/sum-of-distances-in-tree/solutions/2345592/tu-jie-yi-zhang-tu-miao-dong-huan-gen-dp-6bgb/)。

## 经典例题

### 01背包问题

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

[1049 最后一个石头的重量Ⅱ](https://leetcode.cn/problems/last-stone-weight-ii/description/)

思路：

- 本题经过适当分析，可以转换为一个01背包问题。其分析方法类似[494 目标和](https://leetcode.cn/problems/target-sum/description/)
- 总的来说，就是将问题转换成如何将所有石头分成重量最接近的两份。
  - 因此，我们可以定义一个长度为所有石头重量之和一半的数组，作为保存背包状态的dp数组。
  - 最终，我们希望容量最大背包（dp数组尾部元素）中物品价值（石头重量）最接近所有物品价值的一半。
  - 整个思路类似[416 分割等和的子集](https://leetcode.cn/problems/partition-equal-subset-sum/description/)

代码：

``` c++
class Solution {
 public:
  int lastStoneWeightII(vector<int>& stones) {
    int sum = 0;
    for (auto& stone : stones) {
      sum += stone;
    }

    int n = sum / 2 + 1;
    vector<int> dp(n + 1, 0);
    for (auto& stone : stones) {
      for (int i = n; i >= 0; i--) {
        if (i > stone) {
          dp[i] = max(dp[i], dp[i - stone] + stone);
        }
      }
    }

    return sum - 2 * dp[n];
  }
};
```

### 完全背包问题

[322 零钱兑换](https://leetcode.cn/problems/coin-change/description/)

思路：

- 本题是一道完全背包问题，即：商品数量无限。
- 分析题意可以得到转移方程如下：

``` c++
// dp[i]表示面值为 i 的现金替换需要的最少硬币数量
dp[i] = min(dp[i - coin]) + 1;
```

代码：

``` c++
class Solution {
 public:
  int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, -1);

    dp[0] = 0;
    sort(coins.begin(), coins.end());
    for (int i = 1; i <= amount; i++) {
      for (auto& coin : coins) {
        if (coin > i) {
          break;
        }
        if (dp[i - coin] != -1 && (dp[i] == -1 || dp[i] > dp[i - coin] + 1)) {
          dp[i] = dp[i - coin] + 1;
        }
      }
    }

    return dp[amount];
  }
};
```

[518 零钱兑换Ⅱ](https://leetcode.cn/problems/coin-change-ii/description/)

思路：

- 遍历硬币数组，对于进行如下操作：
  - 从 coin 到 amount 遍历，将 dp[i−coin] 的值加到 dp[i]。
  - 最终得到 dp[amount]的值即为答案。
- 与[322 零钱兑换](https://leetcode.cn/problems/coin-change/description/)不同，此题需要注意如何避免重复计算兑换方案。具体来说，就是外层循环遍历硬币数组，内层循环是遍历不同的金额之和，在计算 dp[i] 的值时，可以确保金额之和等于 i 的硬币面额的顺序，由于顺序确定，因此不会重复计算不同的排列。

代码：

``` c++
class Solution {
 public:
  int change(int amount, vector<int>& coins) {
    vector<int> dp(amount + 1, 0);

    dp[0] = 1;
    sort(coins.begin(), coins.end());
    for (auto coin : coins) {
      for (int i = 1; i <= amount; i++) {
        if (i >= coin) {
          dp[i] += dp[i - coin];
        }
      }
    }

    return dp[amount];
  }
};
```

### 股票问题

[121 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

[122 买卖股票的最佳时机Ⅱ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

思路：

- 分别记录第i天交易完后手里没有股票的最大利润和手里持有一支股票的最大利润。
- 返回最后一天不持有股票的最大利润即可。
- 此外，本题还可以使用贪心+单调栈的方法解决。

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

[188 买卖股票的最佳时机Ⅳ](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)

### 树形DP

[834 树中距离之和](https://leetcode.cn/problems/sum-of-distances-in-tree/description/)

一道很经典的换根dp问题，除了要知道使用换根dp之外，还要能够找出转移方程，尤其是问题中答案和每个子树大小的关系。

```c++
class Solution {
public:
    vector<int> sumOfDistancesInTree(int n, vector<vector<int>> &edges) {
        vector<vector<int>> g(n); // g[x] 表示 x 的所有邻居
        for (auto &e: edges) {
            int x = e[0], y = e[1];
            g[x].push_back(y);
            g[y].push_back(x);
        }

        vector<int> ans(n);
        vector<int> size(n, 1); // 注意这里初始化成 1 了，下面只需要累加儿子的子树大小
        function<void(int, int, int)> dfs = [&](int x, int fa, int depth) {
            ans[0] += depth; // depth 为 0 到 x 的距离
            for (int y: g[x]) { // 遍历 x 的邻居 y
                if (y != fa) { // 避免访问父节点
                    dfs(y, x, depth + 1); // x 是 y 的父节点
                    size[x] += size[y]; // 累加 x 的儿子 y 的子树大小
                }
            }
        };
        dfs(0, -1, 0); // 0 没有父节点

        function<void(int, int)> reroot = [&](int x, int fa) {
            for (int y: g[x]) { // 遍历 x 的邻居 y
                if (y != fa) { // 避免访问父节点
                    ans[y] = ans[x] + n - 2 * size[y];
                    reroot(y, x); // x 是 y 的父节点
                }
            }
        };
        reroot(0, -1); // 0 没有父节点
        return ans;
    }
};
```

[2581 统计可能的树根数目](https://leetcode.cn/problems/count-number-of-possible-root-nodes/description/)

本题如果只求以0为根时的猜对次数cnt0，把guesses转成哈希表，DFS一次这棵树就可以算出来。

如果枚举0到n−1的每个点作为树根，就需要DFS n次，需要O(n2)的时间，怎么优化呢？

注意到，如果节点x和节点y相邻，那么从「以x为根的树」变成「以y为根的树」，就只有x和y的父子关系改变了，其余相邻节点的父子关系没有变化。所以只有 [x,y]和[y,x]这两个猜测的正确性变了，其余猜测的正确性不变。

因此，在计算出cnt0后，我们可以再次从0出发，DFS这棵树。从节点x递归到节点y时：

如果有猜测 [x,y]，那么猜对次数减一。
如果有猜测 [y,x]，那么猜对次数加一。

DFS 的同时，统计猜对次数≥k 的节点个数，即为答案。

```c++
class Solution {
public:
    using ll = long long;
    int rootCount(vector<vector<int>>& edges, vector<vector<int>>& guesses, int k) {
        int n = edges.size() + 1;
        vector<vector<int>> g(n);
        unordered_set<ll> st;
        for (auto &v : edges) {
            g[v[0]].push_back(v[1]);
            g[v[1]].push_back(v[0]);
        }
        auto h = [&](int x, int y) -> ll {
            return (ll) x << 20 | y;
        };
        for (auto &v : guesses) {
            st.insert(h(v[0], v[1]));
        }

        int cnt = 0, res = 0;
        function<void(int, int)> dfs = [&](int x, int fat) -> void {
            for (auto &y : g[x]) {
                if (y == fat) {
                    continue;
                }
                cnt += st.count(h(x, y));
                dfs(y, x);
            }
        };
        dfs(0, -1);

        function<void(int, int, int)> redfs = [&](int x, int fat, int cnt) {
            if (cnt >= k) {
                res++;
            }
            for (auto &y : g[x]) {
                if (y == fat) {
                    continue;
                }
                redfs(y, x, cnt - st.count(h(x, y)) + st.count(h(y, x)));
            }
        };
        redfs(0, -1, cnt);
        return res;
    }
};
```

### 子序列问题

[72 编辑距离](https://leetcode.cn/problems/edit-distance/description/)

又一道非常经典的二维dp，主要难点需要理解其转移方程：

- `dp[i][j]`表示word1 0到i的序列，转换到word2 0到j的序列需要的步数，比如：当word1 = "horse", word2 = "ros"，`dp[0][0]`就表示"h"到"r"的转移步数。
- 当`word1[i]==word2[j]`时，`dp[i][j]=min(dp[i-1][j]+1, dp[i][j-1]+1, dp[i-1][j-1])`，反之，`dp[i][j]=min(dp[i-1][j]+1, dp[i][j-1]+1, dp[i-1][j-1]+1)`；
- 其中，`dp[i-1][j]和dp[i][j-1]`始终需要加1是因为其和末尾是否相等无关。

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        if (word2.empty()) {
            return word1.size();
        }
        if (word1.empty()) {
            return word2.size();
        }
        vector<vector<int>> dp(word1.size(), vector<int>(word2.size()));
        for (int k = 0; k < word1.size() + word2.size() - 1; k++) {
            for (int i = 0; i <= min(k, (int)word1.size() - 1); i++) {
                int j = k - i;
                if (j > word2.size() - 1) {
                    continue;
                }
                if (i > 0 && j > 0) {
                    dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + 1;
                    dp[i][j] = min(dp[i][j], dp[i-1][j-1] + (word1[i] == word2[j] ? 0 : 1));
                } else if (i > 0) {
                    dp[i][j] = min(dp[i-1][j]+1, word1[i] == word2[j] ? i : i+1);
                } else if (j > 0) {
                    dp[i][j] = min(dp[i][j-1]+1, word1[i] == word2[j] ? j : j+1);
                } else {
                    dp[i][j] = word1[i] == word2[j] ? 0 : 1;
                }
            }
        }

        return dp[word1.size()-1][word2.size()-1];
    }
};
```

[152 最大子数组乘积](https://leetcode.cn/problems/maximum-product-subarray/description/)

[300 最长的递增序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)

这道题非常经典，除了可以使用常规的动态规划之外，还能使用贪心+二分查找的方法进一步降低时间复杂度。

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

[718 最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/description/)

思路：

- 典型的二维dp类型题目。
- 定义二维数组，其中元素`dp[i][j]`表示`nums1[0:i]`和`nums2[0:j]`中以`nums1[i]`和`nums2[j]`为结尾的最长重复子数组的长度。
- 由此可以得到转移方程：

``` c++
if (nums1[i] == nums2[j]) {
  if (i > 0 && j > 0) {
    dp[i][j] = max(dp[i - 1][j - 1] + 1, 1);
  } else {
    dp[i][j] = 1;
  }
}
```

代码：

``` c++
class Solution {
 public:
  int findLength(vector<int>& nums1, vector<int>& nums2) {
    int ret = 0;
    int m = nums1.size(), n = nums2.size();
    vector<vector<int>> dp(m, vector<int>(n, 0));
    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        if (nums1[i] == nums2[j]) {
          if (i > 0 && j > 0) {
            dp[i][j] = max(dp[i - 1][j - 1] + 1, 1);
          } else {
            dp[i][j] = 1;
          }
          ret = max(ret, dp[i][j]);
        }
      }
    }
    return ret;
  }
};
```

[1035 不相交的线](https://leetcode.cn/problems/uncrossed-lines/)

思路：本题思路与[1143 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/description/)极其相似，甚至连转移方程都一致。

代码：

``` c++
class Solution {
 public:
  int maxUncrossedLines(vector<int>& nums1, vector<int>& nums2) {
    int m = nums1.size(), n = nums2.size();
    vector<vector<int>> dp(m, vector<int>(n, 0));

    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        if (nums1[i] == nums2[j]) {
          dp[i][j] = (i > 0 && j > 0 ? dp[i - 1][j - 1] : 0) + 1;
        } else {
          dp[i][j] = max(i > 0 ? dp[i - 1][j] : 0, j > 0 ? dp[i][j - 1] : 0);
        }
      }
    }

    return dp[m - 1][n - 1];
  }
};
```

[1143 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/description/)

思路：

- 非常经典的一道二维dp例题。
- 只要能够写出转移方程即可解决本题。转移方程如下：

``` c++
if (text1[i] == text[j]) {
  dp[i][j] = dp[i - 1][j - 1] + 1;
} else {
  dp[i][j] = max(dp[i][j - 1], dp[i - 1][j]);
}
```

代码：

```  c++
class Solution {
 public:
  int longestCommonSubsequence(string text1, string text2) {
    int m = text1.length(), n = text2.length();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1));
    for (int i = 1; i <= m; i++) {
      char c1 = text1.at(i - 1);
      for (int j = 1; j <= n; j++) {
        char c2 = text2.at(j - 1);
        if (c1 == c2) {
          dp[i][j] = dp[i - 1][j - 1] + 1;
        } else {
          dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
      }
    }
    return dp[m][n];
  }
};
```

### 博弈策略问题

[486 预测赢家](https://leetcode.cn/problems/predict-the-winner/)

[877 石子游戏](https://leetcode.cn/problems/stone-game/description/)

``` c++
class Solution {
 public:
  bool stoneGame(vector<int>& piles) {
    int length = piles.size();
    auto dp = vector<int>(length);
    for (int i = 0; i < length; i++) {
      dp[i] = piles[i];
    }
    for (int i = length - 2; i >= 0; i--) {
      for (int j = i + 1; j < length; j++) {
        dp[j] = max(piles[i] - dp[j], piles[j] - dp[j - 1]);
      }
    }
    return dp[length - 1] > 0;
  }
};
```

### 其他

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

[1130 叶值的最小代价生成树](https://leetcode.cn/problems/minimum-cost-tree-from-leaf-values/description/)

``` c++
class Solution {
public:
    int mctFromLeafValues(vector<int>& arr) {
        int n = arr.size();
        vector<vector<int>> dp(n, vector<int>(n, INT_MAX / 4)), mval(n, vector<int>(n));
        for (int j = 0; j < n; j++) {
            mval[j][j] = arr[j];
            dp[j][j] = 0;
            for (int i = j - 1; i >= 0; i--) {
                mval[i][j] = max(arr[i], mval[i + 1][j]);
                for (int k = i; k < j; k++) {
                    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + mval[i][k] * mval[k + 1][j]);
                }
            }
        }
        return dp[0][n - 1];
    }
};
```
