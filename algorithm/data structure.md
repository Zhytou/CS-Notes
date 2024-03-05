# 数据结构

> 栈、堆、队列以及哈希表的查找插入删除

## 基础概念

栈

堆

队列

哈希表

## 经典题目

### 基础题

[155 最小栈](https://leetcode.cn/problems/min-stack/description/)

思路：单调栈

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

[225 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

- 思路：一个队列实现，简单来说就是==循环入队==保证队首始终为最后入队的元素

  - 入栈操作时，首先获得入栈前的元素个数 n，然后将新元素入队到队列；
  - 再将队列中的前 n 个元素（即除了新入栈的元素之外的全部元素）依次出队并入队到队列；
  - 此时队列的前端的元素即为新入栈的元素，且队列的前端和后端分别对应栈顶和栈底。

``` c++
class MyStack {
public:
    queue<int> q;
  
    /** Initialize your data structure here. */
    MyStack() { }
  
    /** Push element x onto stack. */
    void push(int x) {
        int n = q.size();
        q.push(x);
        for (int i = 0; i < n; i++) {
            q.push(q.front());
            q.pop();
        }
    }
      
    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        int r = q.front();
        q.pop();
        return r;
    }
      
    /** Get the top element. */
    int top() {
        int r = q.front();
        return r;
    }
      
    /** Returns whether the stack is empty. */
    bool empty() {
        return q.empty();
    }
};
```

### 中等题

[49 字母异位分组](https://leetcode.cn/problems/group-anagrams/description/)

本题需要自己实习定长数组的哈希表，难点就在此，[StackOverflow解释](https://stackoverflow.com/questions/12438602/using-an-array-as-key-in-map-c)。

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // 自定义对 array<int, 26> 类型的哈希函数
        auto arrayHash = [fn = hash<int>{}] (const array<int, 26>& arr) -> size_t {
            return accumulate(arr.begin(), arr.end(), 0u, [&](size_t acc, int num) {
                return (acc << 1) ^ fn(num);
            });
        };

        unordered_map<array<int, 26>, vector<string>, decltype(arrayHash)> mp(0, arrayHash);
        for (string& str: strs) {
            array<int, 26> counts{};
            int length = str.length();
            for (int i = 0; i < length; ++i) {
                counts[str[i] - 'a'] ++;
            }
            mp[counts].emplace_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); ++it) {
            ans.emplace_back(it->second);
        }
        return ans;
    }
};
```

[71 简化路径](https://leetcode-cn.com/problems/simplify-path/)

[1996 游戏中的若角色数量](https://leetcode-cn.com/problems/the-number-of-weak-characters-in-the-game/)

- 思路：**单调栈**

  - 首先，把角色按攻击力从小到大排序，攻击力相同的按防御力从大到小排序
  - 接着，遍历角色数组，若栈顶角色攻击力和防御力均低于当前访问角色，则栈顶角色为弱角色，弹出并计数，否则将当前角色加入栈中
  - 重复上述步骤，直到遍历完成

``` c++
class Solution {
public:
    int numberOfWeakCharacters(vector<vector<int>>& properties) {
        int res = 0;
        stack<int> stk;
        sort(properties.begin(), properties.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0] || a[0] == b[0] && a[1] > b[1];
        });
  
        for (auto& p : properties) {
            while (!stk.empty() && stk.top() < p[1]) {
                res += 1;
                stk.pop();
            }
            stk.emplace(p[1]);
        }
        return res;
    }
};
```

[2048 股票价格波动](https://leetcode-cn.com/problems/stock-price-fluctuation/submissions/)

- 思路：哈希表 + 两个优先队列

- 代码：

 ``` c++
typedef pair<int,int> pii;
  
class StockPrice {
public:
    StockPrice() {
        this->maxTimestamp = 0;
    }
      
    void update(int timestamp, int price) {
        maxTimestamp = max(maxTimestamp, timestamp);
        timePriceMap[timestamp] = price;
        pqMax.emplace(price, timestamp);
        pqMin.emplace(price, timestamp);
    }
      
    int current() {
        return timePriceMap[maxTimestamp];
    }
      
    int maximum() {
        while (true) {
            int price = pqMax.top().first, timestamp = pqMax.top().second;
            if (timePriceMap[timestamp] == price) {
                return price;
            }
            pqMax.pop();
        }
    }
      
    int minimum() {
        while (true) {
            int price = pqMin.top().first, timestamp = pqMin.top().second;
            if (timePriceMap[timestamp] == price) {
                return price;
            }
            pqMin.pop();
        }
    }
private:
    int maxTimestamp;
    unordered_map<int, int> timePriceMap;
    priority_queue<pii, vector<pii>, less<pii>> pqMax;
    priority_queue<pii, vector<pii>, greater<pii>> pqMin;
};
```

### 难题

[32 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)

一道关于栈的难题，核心问题是如何通过栈中数据，获取当前遍历位置有效括号的长度。其解决方法就是使用栈保存idx:

- 当遍历到左括号时，直接入栈；
- 当遍历到右括号时，且栈顶为左括号时，即可通过序号-栈顶序号计算有效括号长度。

```c++
class Solution {
public:
    int longestValidParentheses(string s) {
        int maxans = 0;
        stack<int> stk;
        for (int i = 0; i < s.length(); i++) {
            if (s[i] == '(') {
                stk.push(i);
            } else if (stk.empty() || s[stk.top()] != '(') {
                stk.push(i);
            } else {
                stk.pop();
                if (stk.empty()) {
                    maxans = max(maxans, i + 1);
                } else {
                    maxans = max(maxans, i - stk.top());
                }
            }
        }
        return maxans;
    }
};

```
