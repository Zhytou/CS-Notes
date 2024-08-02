# 数据结构

- [数据结构](#数据结构)
  - [单调栈/单调队列](#单调栈单调队列)
  - [字典树](#字典树)
  - [线段树](#线段树)
  - [对顶堆](#对顶堆)
  - [并查集](#并查集)

## 单调栈/单调队列

**描述**：

单调栈是一种特殊的栈数据结构。它满足单调性，即：栈内的元素按照一定的单调规律排列。通常，单调栈用于解决寻找元素左边（或右边）第一个比它大（或小）的元素的问题。通过维护一个递增或递减的栈，可以在线性时间内解决这类问题。

**例题**：

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

[239 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/description/)

简单来说，就是维护一个滑动窗口的单调递减队列。整个队列中的元素都是潜在的窗口最大值。当窗口向右滑动时，若队列头等于刚刚离开窗口的元素，则弹出。新进入区间的元素，若超过大于队尾，则需要重新维护，反之则区间最大值与之无关。

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

这个问题可以转化为：给定一个数组arr，不断地合并相邻的数，合并代价为两个数的乘积，合并之后的数为两个数的最大值，直到数组只剩一个数，求最小合并代价和。

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

字典树，也被称为前缀树，是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。

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

- [实现字典树](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)
- [字典中最长的单词](https://leetcode-cn.com/problems/longest-word-in-dictionary/)
- [字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/submissions/)
- [实现一个魔法字典](https://leetcode.cn/problems/implement-magic-dictionary/)

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

## 线段树

**描述**：

线段树一种用于处理区间查询的数据结构。它将一个区间划分为多个较小的子区间，并将每个子区间的信息存储在树节点中。线段树可以高效地进行区间查询和区间更新操作，例如计算区间和、区间最大值等。常见的应用包括区间最值查询、区间覆盖等。

事实上，线段树就是一个树状数组，类似于堆排序中用一个数组去表示一个完全二叉树。比如，数组[1,5,3,7,3,2,5,7]，由其所构建的最小值线段树为如下图所示。

![最小值线段树](https://img-blog.csdn.net/20180318130444215?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L1lhb2thaV9Bc3N1bHRNYXN0ZXI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可见，线段树中叶子节点存储输入数组的元素值，而其他节点则保存它所管理子节点的最小值，即区间信息。

**实现**：

```c++
void build(int s, int t, int p) {
  // 对[s,t]区间建立线段树，当前根的编号为 p
  if (s == t) {
    d[p] = a[s];
    return;
  }
  int m = s + ((t - s) >> 1);
  // 移位运算符的优先级小于加减法，所以加上括号
  // 如果写成 (s + t) >> 1 可能会超出 int 范围
  build(s, m, p * 2), build(m + 1, t, p * 2 + 1);
  // 递归对左右区间建树
  d[p] = d[p * 2] + d[(p * 2) + 1];
}

int query(int l, int r, int s, int t, int p) {
  // [l, r] 为查询区间, [s, t] 为当前节点包含的区间, p 为当前节点的编号
  if (l <= s && t <= r)
    return d[p];  // 当前区间为询问区间的子集时直接返回当前区间的和
  int m = s + ((t - s) >> 1), res = 0;
  if (l <= m) res += query(l, r, s, m, p * 2);
  // 如果左儿子代表的区间 [s, m] 与询问区间有交集, 则递归查询左儿子
  if (r > m) res = max(res, query(l, r, m + 1, t, p * 2 + 1));
  // 如果右儿子代表的区间 [m + 1, t] 与询问区间有交集, 则递归查询右儿子
  return res;
}
```

**例题**：

[2407 最长的递增子序列Ⅱ](https://leetcode.cn/problems/longest-increasing-subsequence-ii/description/)：线段树优化DP

## 对顶堆

**描述**：

对顶堆由一个大根堆与一个小根堆组成，小根堆维护大值即前k大的值，大根堆维护后n-k大的其他值。它的两个核心操作包括：

- 维护：当小根堆的大小小于k时，不断将大根堆堆顶元素取出并插入小根堆，直到小根堆的大小等于k；当小根堆的大小大于k时，不断将小根堆堆顶元素取出并插入大根堆，直到小根堆的大小等于k；
- 插入元素：若插入的元素大于等于小根堆堆顶元素，则将其插入小根堆，否则将其插入大根堆，然后维护对顶堆；

**应用**：

可见，这个数据结构能够很方便的获取中位数或第k大的数。这同样也是该数据结构的核心功能。

**例题**：

[259 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

[480 滑动窗口中位数](https://leetcode.cn/problems/sliding-window-median/)

## 并查集

**描述**：

并查集是一种用于解决元素分组及查询两个元素是否属于同一组的数据结构。并查集支持两种基本操作：查找（Find）和合并（Union）。通过路径压缩和按秩合并等优化策略，可以使得并查集的时间复杂度接近常数。

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
