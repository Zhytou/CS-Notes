# 回溯算法

- [回溯算法](#回溯算法)
  - [基础概念](#基础概念)
    - [回溯](#回溯)
    - [剪枝](#剪枝)
    - [记忆化搜索](#记忆化搜索)
  - [题型分类](#题型分类)
    - [子集](#子集)
    - [组合](#组合)
    - [排列](#排列)
  - [经典题目](#经典题目)
    - [基础题](#基础题)
    - [中等题](#中等题)
    - [难题](#难题)

## 基础概念

### 回溯

回溯算法是一种基于深度优先搜索的算法，用于解决组合优化问题、排列问题、选择问题等。其思想是通过逐步地构建解，当发现当前构建的解不满足问题的约束条件时，就回溯到上一步进行调整，继续尝试其他的可能解，直到找到合适的解或所有可能的解都被尝试。

### 剪枝

剪枝(Pruning)是回溯算法中常用的优化技巧，通过提前排除不符合条件的选择，减少搜索的分支，从而提高算法的效率。

### 记忆化搜索

记忆化搜索(Memoization)是回溯算法中的另一种常用技巧，用于避免重复计算，提高算法的效率。记忆化搜索通过保存中间结果，避免重复的计算过程，当需要再次使用相同的结果时，直接从缓存中获取，避免了重复的计算。

## 题型分类

| 题目类型 | 经典题目                                                     |
| -------- | ------------------------------------------------------------ |
| 子集     | [子集](https://leetcode-cn.com/problems/subsets/)、[子集Ⅱ](https://leetcode-cn.com/problems/subsets-ii/) |
| 组合     | [组合](https://leetcode-cn.com/problems/combinations/)、[组合总和](https://leetcode-cn.com/problems/combination-sum/)、[组合的总和Ⅱ](https://leetcode-cn.com/problems/combination-sum-ii/) |
| 排列     | [全排列](https://leetcode-cn.com/problems/permutations/)、[全排列Ⅱ](https://leetcode-cn.com/problems/permutations-ii/)、[字符串的全排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)、[字母大小写全排列](https://leetcode-cn.com/problems/letter-case-permutation/)、[活字印刷](https://leetcode.cn/problems/letter-tile-possibilities/description/)|
| 搜索     | [二进制手表](https://leetcode-cn.com/problems/binary-watch/)、[解数独](https://leetcode-cn.com/problems/sudoku-solver/)、[单词搜索](https://leetcode-cn.com/problems/word-search/)、[N皇后](https://leetcode-cn.com/problems/eight-queens-lcci/)、[分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/) |

### 子集

![子集](./img/code_backtracking_subset.png)

``` c++
void backtracking(vector<vector<int>>& res, const vector<int>& nums, vector<int>& subset, int start) {
  res.push_back(subset);
  for (int i = start; i < nums.size(); i++) {
    subset.push_back(nums[i]);
    backtracking(res, nums, subset, i + 1);
    subset.pop_back();
  }
}
```

### 组合

![组合](./img/code_backtracking_combine.png)

``` C++
void backtracking(vector<vector<int>>& res, vector<int>& combination, const int& n, const int& k, int start) {
  if (combination.size() == k) {
    res.push_back(combination);
    return ;
  }

  for (int i = start; i <= n; i++) {
    combination.push_back(i);
    backtracking(res, combination, n, k, i + 1);
    combination.pop_back();
  }
  return ;
}
```

### 排列

![排列](../img/code_backtracking_permute.png)

``` c++
  void backtracking(vector<vector<int>>& res, vector<int>& permutation, int idx) {
  if (idx == permutation.size()) {
    res.push_back(permutation);
    return ;
  }

  for (int i = idx; i < permutation.size(); i++) {
    swap(permutation[idx], permutation[i]);
    backtracking(res, permutation, idx + 1);
    swap(permutation[idx], permutation[i]);
  }
  return ;
}
```

## 经典题目

### 基础题

[39 组合总和](https://leetcode-cn.com/problems/combination-sum/_)

与常规求组合或子集不同的时，本题增加了数组中 **同一个** 数字可以 **无限制重复被选取**的条件。因此，只需要在求组合或子集的模板上，将递归调用函数的参数start由i + 1改为i即可。这样就既保证了同一数字可以重复选取，又保证了排在后面的数字不能选排在前面的数字进行组合（避免重复）。

``` c++
class Solution {
    void backtracking(vector<vector<int>>& res, vector<int>& combination, const vector<int>& nums, const int& targetSum, int& currentSum, int start) {
        if (currentSum >= targetSum) {
            if (currentSum == targetSum)
                res.push_back(combination);
            return ;
        }
        for (int i = start; i < nums.size(); i++) {
            combination.push_back(nums[i]);
            currentSum += nums[i];
            backtracking(res, combination, nums, targetSum, currentSum, i); 
            currentSum -= nums[i];
            combination.pop_back();
        }
        return ;
    }
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> res;
        vector<int> combination;
        int sum = 0;
        backtracking(res, combination, candidates, target, sum, 0);
        return res;
    }
};
```

### 中等题

[40 组合总和Ⅱ](https://leetcode-cn.com/problems/combination-sum-ii/)

与[39 组合总和](https://leetcode-cn.com/problems/combination-sum/_) 相比，本题要求所有元素**只能用一次** ，同时候选元素中也**允许重复元素**。

- 组合：要求所有元素只用一次，即：每次start都要更新为i + 1。
- 剪枝：本题的剪枝逻辑与[47 全排列Ⅱ](https://leetcode-cn.com/problems/permutations-ii/ ) 类似， 即：保证每次选择重复数字的第一个。
  - 剪枝前提：对排序数组进行排序，保证重复数字相邻排序
  - 访问记录：用visited数组记录是否访问过该位置的数字
  - 剪枝条件：当前数字与前一个数字相同、且钱一个数字未被访问（放入排列中）

    即：nums[i] == nums[i - 1] && !visited[i - 1]

``` c++
class Solution {
    vector<bool> visited;

    void backtracking(vector<vector<int>>& res, vector<int>& combination, const vector<int>& nums, const int& targetSum, int& currentSum, int start) {
        if (currentSum >= targetSum) {
            if (currentSum == targetSum)
                res.push_back(combination);
            return ;
        }
        for (int i = start; i < nums.size(); i++) {
            if (i > 0 && nums[i] == nums[i - 1] && !visited[i - 1])
                continue;
            
            if (!visited[i]) {
                combination.push_back(nums[i]);
                visited[i] = true;
                currentSum += nums[i];
                backtracking(res, combination, nums, targetSum, currentSum, i + 1);
                currentSum -= nums[i];
                visited[i] = false;
                combination.pop_back();
            }
        }
        return ;
    }
public:
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        vector<vector<int>> res;
        vector<int> combination;
        int sum = 0;

        visited.resize(candidates.size(), false);
        sort(candidates.begin(), candidates.end());
        backtracking(res, combination, candidates, target, sum, 0);
        return res;
    }
};
```

[1079 活字印刷](https://leetcode.cn/problems/letter-tile-possibilities/description/)

首先，统计给定字符串中出现字符数量；接着，每次递归回溯时确定一个字符并将该字符数量减一。

``` c++
class Solution {
public:
    int numTilePossibilities(string tiles) {
        unordered_map<char, int> count;
        set<char> tile;
        int n = tiles.length();
        for (char c : tiles) {
            count[c]++;
            tile.insert(c);
        }
        return dfs(count, tile, n) - 1;
    }

    int dfs(unordered_map<char, int>& count, set<char>& tile, int i) {
        if (i == 0) {
            return 1;
        }
        int res = 1;
        for (char t : tile) {
            if (count[t] > 0) {
                count[t]--;
                res += dfs(count, tile, i - 1);
                count[t]++;
            }
        }
        return res;
    }
};
```

### 难题

[79 单词搜索](https://leetcode-cn.com/problems/word-search/)

- 简介：

  - 题目就是要求在二维字符网格中找出一个字符串。
    - 首先想到的肯定是用深度优先搜索去做，但深度优先搜索却始终迈不过 *[["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]] "ABCB"* 的情况，因为深度优先搜索的visited数组是针对全局搜索的，而回溯搜索的visited数组确实针对当前这种策略，它会在**策略失败返回重置visited数组**而深度优先搜索却不可以
    - 在明确使用回溯搜索解决本问题之后，难度就大大减小了

- 思路：

  - 要点：
    - 用方向数组 directions 是一种简化搜索代码常规手段，需要记住
    - visited 数组和整数 x、y 需要在当前回溯策略失败后重置
  - 流程：略

- 代码：

``` c++
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        int m = board.size();
        int n = board[0].size();

        visited.resize(m, vector<bool>(n, false));
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (backtracking(i, j, 0, board, word))
                    return true;
            }
        }
        return false;
    }
private:
    bool backtracking(int& x, int& y, int idx, const vector<vector<char>>& board, const string& word) {
        if (idx == word.size())
            return true;
        
        if (0 <= x && x < board.size() && 0 <= y && y < board[0].size() && !visited[x][y] &&board[x][y] == word[idx]) {
            visited[x][y] = true;
            for (auto& [dx, dy] : directions) {
                x += dx;
                y += dy;
                if (backtracking(x, y, idx + 1, board, word))
                    return true;
                x -= dx;
                y -= dy;
            }
            visited[x][y] =false;
        }
        return false;
    }
    const vector<pair<int, int>> directions = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    vector<vector<bool>> visited;
};
```
