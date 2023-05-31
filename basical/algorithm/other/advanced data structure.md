# 数据结构

- [数据结构](#数据结构)
  - [单调栈/单调队列](#单调栈单调队列)
  - [字典树](#字典树)
  - [对顶堆](#对顶堆)
  - [并查集](#并查集)

## 单调栈/单调队列

[42 接雨水](https://leetcode.cn/problems/trapping-rain-water/description/)

``` c++
class Solution {
public:
    int trap(vector<int>& height) {
        int ans = 0;
        stack<int> stk;
        int n = height.size();
        for (int i = 0; i < n; ++i) {
            while (!stk.empty() && height[i] > height[stk.top()]) {
                int top = stk.top();
                stk.pop();
                if (stk.empty()) {
                    break;
                }
                int left = stk.top();
                int currWidth = i - left - 1;
                int currHeight = min(height[left], height[i]) - height[top];
                ans += currWidth * currHeight;
            }
            stk.push(i);
        }
        return ans;
    }
};
```

[155 最小栈](https://leetcode.cn/problems/min-stack/description/)

代码：

``` c++
class MinStack {
 private:
  stack<int> nums, minNums;

 public:
  MinStack() {}

  void push(int val) {
    if (nums.empty() || val <= minNums.top()) {
      minNums.push(val);
    }
    nums.push(val);
  }

  void pop() {
    if (nums.top() == minNums.top()) {
      minNums.pop();
    }
    nums.pop();
  }

  int top() { return nums.top(); }

  int getMin() { return minNums.top(); }
};
```

[239 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/description/)

思路：
    简单来说，就是维护一个滑动窗口的单调递减队列。整个队列中的元素都是潜在的窗口最大值。
    当窗口向右滑动时，若队列头等于刚刚离开窗口的元素，则弹出。新进入区间的元素，若超过大于队尾，则需要重新维护，反之则区间最大值与之无关。

代码：

``` c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        deque<int> dq;
        
        for (int i = 0; i < k; i++) {
            while (!dq.empty() && dq.back() < nums[i]) {
                dq.pop_back();
            }

            dq.push_back(nums[i]);
        }

        vector<int> ret;
        for (int i = 0; i + k <= nums.size(); i++) {
            ret.push_back(dq.front());
            if (nums[i] == dq.front()) {
                dq.pop_front();
            }
            if (i + k < nums.size()) {
                while (!dq.empty() && dq.back() < nums[i + k]) {
                    dq.pop_back();
                }
                dq.push_back(nums[i + k]);
            }
        }
        return ret;
    }
};
```

[496 下一个元素更大](https://leetcode.cn/problems/next-greater-element-i/)

[1130 叶值的最小代价生成树](https://leetcode.cn/problems/minimum-cost-tree-from-leaf-values/description/)

- 问题可以转化为：给定一个数组 arr\textit{arr}arr，不断地合并相邻的数，合并代价为两个数的乘积，合并之后的数为两个数的最大值，直到数组只剩一个数，求最小合并代价和。

``` c++
class Solution {
public:
    int mctFromLeafValues(vector<int>& arr) {
        int res = 0;
        stack<int> stk;
        for (int x : arr) {
            while (!stk.empty() && stk.top() <= x) {
                int y = stk.top();
                stk.pop();
                if (stk.empty() || stk.top() > x) {
                    res += y * x;
                } else {
                    res += stk.top() * y;
                }
            }
            stk.push(x);
        }
        while (stk.size() >= 2) {
            int x = stk.top();
            stk.pop();
            res += stk.top() * x;
        }
        return res;
    }
};
```

## 字典树

**描述**：

- Trie（发音类似 "try"）或者说 前缀树 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。

**应用**：

- 字符串前缀搜索
- 自动补全
- 拼写检查

**图示**：

![字典树](../../img/algorithm_tire.png)

**代码**：

``` c++
class Trie {
private:
    vector<Trie*> children;
    bool isEnd;

    Trie* searchPrefix(string prefix) {
        Trie* node = this;
        for (char ch : prefix) {
            ch -= 'a';
            if (node->children[ch] == nullptr) {
                return nullptr;
            }
            node = node->children[ch];
        }
        return node;
    }

public:
    Trie() : children(26), isEnd(false) {}

    void insert(string word) {
        Trie* node = this;
        for (char ch : word) {
            ch -= 'a';
            if (node->children[ch] == nullptr) {
                node->children[ch] = new Trie();
            }
            node = node->children[ch];
        }
        node->isEnd = true;
    }

    bool search(string word) {
        Trie* node = this->searchPrefix(word);
        return node != nullptr && node->isEnd;
    }

    bool startsWith(string prefix) {
        return this->searchPrefix(prefix) != nullptr;
    }
};
```

**例题**：

[实现字典树](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

[字典中最长的单词](https://leetcode-cn.com/problems/longest-word-in-dictionary/)

[字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/submissions/)

[实现一个魔法字典](https://leetcode.cn/problems/implement-magic-dictionary/)

**描述**：

- 给定整数 `n` 和 `k`，返回 `[1, n]` 中字典序第 `k` 小的数字。

**解法**：

- 对`n`个数字建立字典树，对其进行前序遍历；
  - 实际不需要真的构造字典树，只需要确定目标节点位置即可；
- 具体做法是，计算当前节点子树节点数，进而确定目标节点是否在该子树中；
  - 如果在，则进入该子树搜索；
  - 否则，换到下一个节点（+ 1）。
- 确定子树节点数的方法使用层序遍历（step函数）。

**代码**：

``` c++
class Solution {
public:
    int getSteps(int curr, long n) {
        int steps = 0;
        long first = curr;
        long last = curr;
        while (first <= n) {
            steps += min(last, n) - first + 1;
            first = first * 10;
            last = last * 10 + 9;
        }
        return steps;
    }

    int findKthNumber(int n, int k) {
        int curr = 1;
        k--;
        while (k > 0) {
            int steps = getSteps(curr, n);
            if (steps <= k) {
                k -= steps;
                curr++;
            } else {
                curr = curr*10;
                k--;
            }
        }
        return curr;
    }
};
```

## 对顶堆

**描述**：

**应用**：

- 中位数
- 第k大的数

**例题**：

[259 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

[480 滑动窗口中位数](https://leetcode.cn/problems/sliding-window-median/)

## 并查集

**实现**：

``` c++
class UnionSet {
  vector<int> parent;
  vector<int> rank;

 public:
  int count;
  UnionSet();
  UnionSet(int n);
  ~UnionSet();

  int find(int index);
  bool merge(int index1, int index2);
};

UnionSet::UnionSet() { count = 0; }

UnionSet::~UnionSet() {}

UnionSet::UnionSet(int n) : count(n) {
  for (int i = 0; i < n; i++) {
    rank.push_back(0);
    parent.push_back(i);
  }
}

int UnionSet::find(int index) {
  if (parent[index] != index) {
    parent[index] = find(parent[index]);
  }
  return parent[index];
}

bool UnionSet::merge(int index1, int index2) {
  auto root1 = find(index1);
  auto root2 = find(index2);
  if (root1 == root2) return false;
  if (rank[root2] > rank[root1]) {
    parent[root1] = root2;
  } else if (rank[root2] < rank[root1]) {
    parent[root2] = root1;
  } else {
    parent[root2] = root1;
    rank[root1] += 1;
  }
  count--;
  return true;
}
```
