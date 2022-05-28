# 数据结构

> 栈、堆、队列已经哈希表的查找插入删除

## 基础概念

### 栈



### 堆



### 队列



### 哈希表



## 经典题目

### 基础题

#### [225 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

+ 简介：略

+ 思路：一个队列实现，简单来说就是==循环入队==保证队首始终为最后入队的元素

  - 入栈操作时，首先获得入栈前的元素个数 n，然后将新元素入队到队列；
  - 再将队列中的前 n 个元素（即除了新入栈的元素之外的全部元素）依次出队并入队到队列；
  - 此时队列的前端的元素即为新入栈的元素，且队列的前端和后端分别对应栈顶和栈底。

+ 代码：

  ``` c++
  class MyStack {
  public:
      queue<int> q;
  
      /** Initialize your data structure here. */
      MyStack() {
  
      }
  
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

#### [71 简化路径](https://leetcode-cn.com/problems/simplify-path/)

#### ==[1996 游戏中的若角色数量](https://leetcode-cn.com/problems/the-number-of-weak-characters-in-the-game/)==

+ 思路：**单调栈**

  - 首先，把角色按攻击力从小到大排序，攻击力相同的按防御力从大到小排序
  - 接着，遍历角色数组，若栈顶角色攻击力和防御力均低于当前访问角色，则栈顶角色为弱角色，弹出并计数，否则将当前角色加入栈中
  - 重复上述步骤，直到遍历完成

+ 代码：

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

#### [2048 股票价格波动](https://leetcode-cn.com/problems/stock-price-fluctuation/submissions/)

+ 思路：哈希表 + 两个优先队列

+ 代码：

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



