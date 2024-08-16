# 图

- [图](#图)
  - [基础概念](#基础概念)
  - [重要算法](#重要算法)
    - [遍历](#遍历)
    - [拓扑排序](#拓扑排序)
    - [最短路径](#最短路径)
    - [连通性](#连通性)
    - [最小生成树](#最小生成树)
  - [经典题目](#经典题目)
    - [分类](#分类)
    - [基础题](#基础题)
    - [中等题](#中等题)
    - [难题](#难题)

## 基础概念

**图**:

图在数据结构中是中一对多的关系，一般分为无向图与无向图。一般用 **邻接矩阵** 或者 **邻接链表** 来表示图中结点的关系。

**度**:

在无向图中，一个顶点依附的边或弧的数目称为该顶点的度。而在有向图中，一个顶点依附的弧头数目称为该顶点的入度，一个顶点依附的弧尾数目称为该顶点的出度，这两者之和称为该顶点的度。

**权**:

在图的边或弧中给出相关的数，称为权。它可以代表一个顶点到另一个顶点的距离，耗费等。

**生成树**：

在图论中，生成树（Spanning Tree）是一个无向图的一棵包含了图中所有顶点且无环的子图。换句话说，生成树是一个无向连通图的一棵包含了所有顶点但没有回路的树。它的特点包括：

- 包含了图中所有的顶点。
- 没有包含任何环路（即无回路）。
- 是原图的一个子图。
- 是一个树结构，即是一个连通的无环图。

## 重要算法

### 遍历

**深度优先 Depth First Search**:

```c++
void dfs(const vector<vector<int>>& adj) {
    stack<int> stk;
    set<int> visited;

    stk.push(0);
    while (!stk.empty()) {
        int i = stk.top();
        visited.insert(i);
        int j = -1;
        for (auto ni : adj[i]) {
            if (visited.count(ni) == 0){
                j = ni;
                break;
            }
        }
        if (j == -1) {
            stk.pop();
        }
    }   
}
```

除了使用栈进行dfs遍历之外，更常见的方法是使用递归进行dfs遍历，比如：

```c++
void dfs(const vector<vector<int>>& adj, int x, int father) {
  for (auto y : adj[x]) {
    if (y != father) {
      dfs(adj, y, x);
    }
  }
}
```

**广度优先 Breadth First Search**:

``` c++
void bfs (const vector<vector<int>>& adj) {
    set<int> visited;
    queue<int> q;

    q.push(0);
    while(!q.empty()) {
        for (int k = q.size(); k > 0; k--) {
            int i = q.front();
            visited.insert(i);
            q.pop();
            for (int j : adj[i]) {
                if (visited.count(j) == 0) {
                    q.push(j);
                }
            }
        }
    }
}
```

**双向广度优先遍历Bidirectional Breadth First Search**：

当起点和终点已知时，使用BBFS可以提高搜索效率，避免算法超时，比如[127 单词接龙](https://leetcode.cn/problems/word-ladder/description)和[752 打开转盘锁](https://leetcode.cn/problems/open-the-lock/description/)都可以使用BBFS。值得注意的是，BBFS中使用那一头的进行搜索的条件很关键。即：`q1.size() <= q2.size()`时，搜索q1；反之搜索q2.如果不注意的话，可能会出现错误。

```c++
void bbfs(const vector<vector<int>>& adj, int s, int t) {
    queue<int> q1, q2;
    map<int, int> m1, m2;

    // 只有两个队列都不空，才有必要继续往下搜索
    // 如果其中一个队列空了，说明从某个方向搜到底都搜不到该方向的目标节点
    // 例如，如果q1为空，说明从起点搜索到底都搜索不到终点，那么反向搜索也没必要进行了
    q1.push(s);
    q2.push(t);
    m1[s] = 1;
    m2[t] = 1;
    while (!q1.empty() && q2.empty()) {
        int ret = -1;
        if (q1.size() <= q2.size()) {
            ret = update(adj, q1, m1, m2);
        } else {
            ret = update(adj, q2, m2, m1);
        }
        if (ret != -1) {
            return ret;
        }
    }
    return ret;
}

int update(const vector<vector<int>>& adj, queue<int>&q, map<int,int>& ms, const map<int,int>& mt) {
    for (int k = q.size(); k > 0; k--) {
        int i = q.front();
        q.pop();
        for (int j : adj[i]) {
            if (mt.count(j)) {
                return mt[j] + mt[i];
            }
            q.push(j);
            ms[j] = ms[i]+1;
        }
    }
    return -1;
}
```

### 拓扑排序

> 对一个有向无环图G进行拓扑排序，是将G中所有顶点排成一个线性序列，使得图中任意一对顶点u和v，若边<u,v>∈E(G)，则u在线性序列中出现在v之前。

**卡恩 Kahn**:

卡恩算法是一种基于**广度优先搜索**的拓扑排序算法。它的流程一般如下：

- 首先，在图中找出入度为0的节点，并把这些从图中删除；
- 接着，找出删除之后，入度为0的节点，并删除这些节点；
- 重复上述两个步骤直至遍历完图中所有节点，即可得到该图的拓扑排序。

``` c++
//Kahn算法
vector<int> toposort_kahn(const vector<vector<bool>>& adj) {
    int n = adj.size();
    vector<int> res, indegrees(n, 0);
    queue<int> q;
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (adj[i][j])
                indegrees[j] += 1;
        }
    }
    
    for (int i = 0; i < n; i++) {
        if (degrees[i] == 0)
            q.push(i);
    }
    
    while(!q.empty()) {
    int idx = q.front();
        q.pop();
        for (int i = 0; i < n; i++) {
            if (adj[idx][i]) {
                indegrees[i] -= 1;
                if (indegrees[i] == 0)
                    q.push(i);
            }
        }
        res.push_back(idx);
    }
    return res;
}
```

**基于深度优先搜索的拓扑排序算法**:

基于深度优先搜索的拓扑排序算法是将深度优先搜索的流程与拓扑排序的求解联系起来的一个方法。它用一个栈来存储所有**已经搜索完成的节点**，从栈顶到栈底的顺序即为一种拓扑排序。且其本质上是**Kahn算法的逆过程**。

在这个算法中，所有图中的节点只有以下三种状态：

- 「未搜索」：我们还没有搜索到这个节点。
- 「搜索中」：我们搜索过这个节点，但还没有回溯到该节点，即该节点还没有入栈，还有相邻的节点没有搜索完成）。
- 「已完成」：我们搜索过并且回溯过这个节点，即该节点已经入栈，并且所有该节点的相邻节点都出现在栈的更底部的位置，满足拓扑排序的要求。
  
``` c++
void toposort_dfs(vector<int>& res, const vector<vector<int>>& adj, vector<int>& visited, int idx, bool& valid) {
    //将当前点标记未搜索中
    visited[idx] = 1;
    for (int i = 0; i < adj[idx].size(); i++) {
        // 递归搜索
        if (adj[idx][i] && visited[i] == 0) {
            toposort_dfs(res, adj, visited, i, valid);
            if (!valid)
                return ;
        }
        // 再次遇到搜索中节点，图中成环
        else if (adj[idx][i] && visited[i] == 1) {
            valid = false;
            return ;
        }
    }
    visited[idx] = 2;
    res.push_back(idx);
} 
```  

### 最短路径

**迪杰斯特拉 Dijkstra**:

迪杰斯特拉算法是一种用于求解单源最短路径问题的算法。它能够找到从给定起点到所有其他节点的最短路径。它采用[贪心](https://baike.baidu.com/item/贪心算法/5411800)的思想，每次遍历到始点距离最近且未访问过的顶点的邻接节点，直到扩展到终点为止。

``` c++
#include<stdio.h>

#include<vector>
#include<queue>

using std::vector;
using std::priority_queue;
using std::pair;

//朴素算法，无堆优化
//参数依次为：点数、出发位置（根位置）、无向图带权重的邻接矩阵
vector<int> dijkstra_normal(int num,int start,const vector<vector<int>> &graph){
    vector<int> dist(num,__INT_MAX__);
    vector<bool> visited(num,false);
    int curr_pos=start;
    int visited_num=1;
    while(visited_num<num){
        //找出当前相邻顶点中到出发点最短的那个
        int next_pos;
        int min;
        min=__INT_MAX__;
        for(int i=0;i<graph[curr_pos].size();i++){
            if(!visited[i] && dist[i]>dist[curr_pos]+graph[curr_pos][i]){
                dist[i]=dist[curr_pos]+graph[curr_pos][i];
            }
            if(!visited[i] && min>dist[i]){
                min=dist[i];
                next_pos=i;
            }
        }

        curr_pos=next_pos;
        visited_num+=1;
        visited[curr_pos]=true;
    }
    return dist;
}

//堆优化
//参数依次为：点数、出发位置（根位置）、无向图带权重的邻接矩阵
vector<int> dijkstra_heap(int num,int start,const vector<vector<int>> &graph){
    vector<int> dist(num,__INT_MAX__);
    vector<bool> visited(num,false);
    //小顶堆
    auto cmp=[](pair<int,int> a,pair<int,int> b){
        return a.second<b.second;
    };
    priority_queue<pair<int,int>,vector<pair<int,int>>,decltype(cmp)> pq(cmp);
    
    pq.push(std::make_pair(start,0));
    while(!pq.empty()){
        auto pos=pq.top().first;
        auto len=pq.top().second;
        pq.pop();
        if(visited[pos])
            continue;
        
        visited[pos]=true;
        for(int i=0;i<dist.size();i++){
            if(dist[i]>graph[i][pos]+len){
                dist[i]=graph[i][pos]+len;
                pq.push(std::make_pair(i,dist[i]));
            }
        }
    }
    return dist;
}
```

**弗洛依达 Floyd**：

弗洛依达算法也是一种用于解决图中最短路径问题的经典算法。只不过，它通常用于计算任意两点之间的最短路径，而不是像迪杰斯特拉算法那样计算一个源点到其他所有点的最短路径。此外，弗洛依达算法采用的是动态规划的思想，而迪杰斯特拉算法则采用的是贪心的思想。

具体来说，弗洛依达算法首先使用邻接矩阵存储初始路径值。其中，如果没有直接相连的两点那么默认为一个很大的值。同时，自己和自己的距离要为0。接着，使用如下状态转移方程更新最短路径`f[i][j] = min(f[i][j], f[i][k]+f[k][j])`。最后，重复上述步骤直到遍历结束。示例代码如下：

```c++
void floyd(vector<vector<int>>& adj) {
    for (int k = 0; k < adj.size(); k++) {
        for (int i = 0; i < adj.size(); i++) {
            for (int j = 0; j < adj.size(); j++) {
                adj[i][j] = min(adj[i][i], adj[i][k]+adj[k][j]);
            }
        }
    }
}
```

**A星**：

A*算法同样是一种用于求解图中最短路径问题的算法。它综合了Dijkstra算法和启发式评估函数，通过评估当前节点到目标节点的估计代价来指导搜索过程，以找到最优的路径。为什么不直接使用Dijkstra算法而要额外添加一个启发式评估函数呢？这是因为如果起点和终点之间存在障碍物，则Dijkstra这类最佳优先算法找到的很可能不是最短路径，比如下图这种情况。

![A星算法的优势](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2019-02-05-a-star-algorithm/best_first_2.gif)

具体来说，A星算法通过计算g(n)+h(n)来确定每个节点的优先级。其中，g(n)是节点n距离起点的代价，而h(n)则是节点n距离终点的预计代价，这也就是A星算法的启发函数。换句话说，当h(n)始终为0时，A星算法也就退化成了迪杰斯克拉算法。一般来说，A星算法中常用的启发函数包括：

- 曼哈顿距离：也称为城市街区距离或L1距离，是指两点在坐标系上的横向和纵向的距离之和。一般用于仅支持上下左右四个方向搜索的A星算法。
- 对角距离：也称为切比雪夫距离或L∞距离，是指两点在坐标系上的横向和纵向的最大距离。一般用于仅支持八个方向搜索的A星算法。
- 欧几里得距离：也称为直线距离或L2距离，是指两点之间的直线距离，即两点之间的直角三角形的斜边长度。一般用于支持任意方向搜索的A星算法。

关于A星算法更详细的介绍可以查看这篇[博客](https://paul.pub/a-star-algorithm/)。

### 连通性

**并查集 Unionset**：

``` c++
#include<vector>

using std::vector;

class UnionSet{
    vector<int> parent;
    vector<int> rank;
public:
    int count;
    UnionSet();
    UnionSet(int n);
    ~UnionSet();

    int find(int index);
    bool merge(int index1,int index2);
};

UnionSet::UnionSet(){
    count=0;
}

UnionSet::~UnionSet(){
    
}

UnionSet::UnionSet(int n):count(n){
    for(int i=0;i<n;i++){
        rank.push_back(0);
        parent.push_back(i);
    }
}

int UnionSet::find(int index){
    if(parent[index]!=index){
        parent[index]=find(parent[index]);
    }
    return parent[index];
}

bool UnionSet::merge(int index1,int index2){
    auto root1=find(index1);
    auto root2=find(index2);
    if(root1==root2)
        return false;
    if(rank[root2]>rank[root1]){
        parent[root1]=root2;
    }
    else if(rank[root2]<rank[root1]){
        parent[root2]=root1;
    }
    else{
        parent[root2]=root1;
        rank[root1]+=1;
    }
    count--;
    return true;
}
```

**Tarjan**：

Tarjan是一种用于在图中查找强连通分量的算法。而所谓的强连通性分量则是指图中的一个子图，其中任意两个节点都可以互相到达。[1192 查找集群中的关键连接](https://leetcode.cn/problems/critical-connections-in-a-network/description/)就是该算法的一到典型例题。

Tarjan的本质是基于深度优先搜索的算法，不过它额外定义DFN和LOW两个数组，用于帮助判断每个节点属于哪个强连通分量。其中，DFN(u)为节点u搜索的次序编号(时间戳)；而LOW(u)则为u或u的子树能够追溯到的最早的栈中节点的次序号。由此可以得出，当DFN(u)=LOW(u)时，以u为根的搜索子树上所有节点是一个强连通分量。该算法伪代码如下：

```c++
tarjan(u){
  DFN[u]=LOW[u]=++Index                      // (1) 为节点u设定次序编号和Low初值
  Stack.push(u)                              // (2) 将节点u压入栈中
  for each (u, v) in E                       // (3) 枚举每一条边
    if (v is not visted)                     // (4) 如果节点v未被访问过
      tarjan(v)                              // (5) 继续向下找
      LOW[u] = min(LOW[u], LOW[v])           // (6) 
    else if (v in Stack)                     // (7) 如果节点v还在栈内
      LOW[u] = min(LOW[u], DFN[v])           // (8)

  if (DFN[u] == LOW[u])                      // (9) 如果节点u是强连通分量的根
    repeat                                   // (10) 循环出栈,直到u=v
      v = Stack.pop                          // (11) 将v退栈，为该强连通分量中一个顶点
      print v                                // (12) 输出v
    until (u == v)                           // (13) 循环终止条件u=v  
}
```

### 最小生成树

**普利姆 Prim**：

普里姆算法是图论中的一种贪心算法，可在一个加权连通图中找到其最小生成树。即由此算法搜索到的边子集所构成的树中，不但包括了连通图里的所有顶点，且其所有边的权值之和亦为最小。它和迪杰斯克拉算法有类似之处，即都使用贪心思想，但它每次选择与当前生成树最近的节点进行连接，而迪杰斯克拉算法则选择到初始节点最近的节点。此外，这两种算法的目标也不相同，Prim算法主要目的还是找出最小生成树。

## 经典题目

### 分类

| 类型                   | 算法                                         | 例题 |
| :--------------------- | -------------------------------------------- | ---- |
| 遍历                   | DFS、BFS、BBFS                                |[127 单词接龙](https://leetcode-cn.com/problems/word-ladder/)      |
| 最短路径               | 单点对单点：BFS、Floyd；单点对多点：Dijkstra |      |
| 最小生成树             | Prim                                         |      |
| 无向图连通性、成环个数 | Unionset                                     |      |
| 有向图连通性、成环个数 | Toposort                                     |      |

### 基础题

[210 课程表Ⅱ](https://leetcode-cn.com/problems/course-schedule-ii/)

- 简介：本题是一道经典的「拓扑排序」问题。
  - 我们将每一门课看成一个节点；
  - 如果想要学习课程 A 之前必须完成课程 B，那么我们从 B 到 A 连接一条有向边。这样以来，在拓扑排序中，B 一定出现在 A 的前面。

- 法一 卡恩算法 Kahn：

  ``` c++
  class Solution {
  public:
      vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
          vector<int> res, indegrees(numCourses, 0);
          vector<vector<int>> adj(numCourses);
          queue<int> q;
  
          for (auto& prerequisity : prerequisites) {
              indegrees[prerequisity[0]] += 1;
              adj[prerequisity[1]].push_back(prerequisity[0]);
          }
  
          for (int i = 0; i < numCourses; i++) {
              if (indegrees[i] == 0)
                  q.push(i);
          }
  
          while (!q.empty()) {
              int idx = q.front();
              q.pop();
              
              for (int i = 0; i < adj[idx].size(); i++) {
                  indegrees[adj[idx][i]] -= 1;
                  if (indegrees[adj[idx][i]] == 0)
                      q.push(adj[idx][i]);
              }
              res.push_back(idx);
          }
  
          if (res.size() == numCourses)
              return res;
          else
              return {};
      }
  };
  ```

- 法二 基于深度优先搜索的拓扑排序：

  ``` c++
  class Solution {
  private:
      // 存储有向图
      vector<vector<int>> edges;
      // 标记每个节点的状态：0=未搜索，1=搜索中，2=已完成
      vector<int> visited;
      // 用数组来模拟栈，下标 0 为栈底，n-1 为栈顶
      vector<int> result;
      // 判断有向图中是否有环
      bool valid = true;
  
  public:
      void dfs(int u) {
          // 将节点标记为「搜索中」
          visited[u] = 1;
          // 搜索其相邻节点
          // 只要发现有环，立刻停止搜索
          for (int v: edges[u]) {
              // 如果「未搜索」那么搜索相邻节点
              if (visited[v] == 0) {
                  dfs(v);
                  if (!valid) {
                      return;
                  }
              }
              // 如果「搜索中」说明找到了环
              else if (visited[v] == 1) {
                  valid = false;
                  return;
              }
          }
          // 将节点标记为「已完成」
          visited[u] = 2;
          // 将节点入栈
          result.push_back(u);
      }
  
      vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
          edges.resize(numCourses);
          visited.resize(numCourses);
          for (const auto& info: prerequisites) {
              edges[info[1]].push_back(info[0]);
          }
          // 每次挑选一个「未搜索」的节点，开始进行深度优先搜索
          for (int i = 0; i < numCourses && valid; ++i) {
              if (!visited[i]) {
                  dfs(i);
              }
          }
          if (!valid) {
              return {};
          }
          // 如果没有环，那么就有拓扑排序
          // 注意下标 0 为栈底，因此需要将数组反序输出
          reverse(result.begin(), result.end());
          return result;
      }
  };
  ```

[606 根据二叉树创建字符串](https://leetcode-cn.com/problems/construct-string-from-binary-tree/)

- 简介:

  - 虽然本题是关于二叉树这种数据结构的题目，即：可以采用树特有的递归深度优先遍历方法，但考虑到空间问题，想要刻意的使用**传统的深度优先遍历**实现先根节点、后左右子树的遍历

- 要点：
  - 题目要求用括号表示子树，且要达成一对一映射关系
  - 例如：输入二叉树 [1,2,3,4] 输出 "1(2(4))(3)"，输入二叉树 [1,2,3,null,4] 输出 "1(2()(4))(3)"
  - 要满足一对一映射关系，即必须保证 左子树为空且右子树不为空的情况需要用 空括号标记

- 代码：

  ``` c++
  class Solution {
  public:
      string tree2str(TreeNode* root) {
          if (root == nullptr)
              return "";
  
          stack<TreeNode*> path;
          set<TreeNode*> visited;
          string res;
  
          path.push(root);
          while(!path.empty()) {
              TreeNode* p = path.top();
  
              if (visited.find(p) == visited.end()) {
                  if (!res.empty())
                      res.push_back('(');
                  res += to_string(p->val);
                  visited.insert(p);
              }
              
              if (p->left != nullptr && visited.find(p->left) == visited.end()) 
                  path.push(p->left);
              else {
                  if (p->right != nullptr && visited.find(p->right) == visited.end()) {
                      if (p->left == nullptr) {
                          res.push_back('(');
                          res.push_back(')');
                      }
                      path.push(p->right);
                  }
                  else {
                      path.pop();
                      if (!path.empty())    
                          res.push_back(')');
                  }
              }
          }
          return res;
      }
  };
  ```

### 中等题

[133 克隆图](https://leetcode-cn.com/problems/clone-graph/)

- 思路：经典遍历题目，只不过需要在遍历的同时复制节点信息。
- 代码：

``` c++
class Solution {
public:
    Node* cloneGraph(Node* node) {
        if (node == nullptr) {
            return node;
        }

        unordered_map<Node*, Node*> visited;

        // 将题目给定的节点添加到队列
        queue<Node*> Q;
        Q.push(node);
        // 克隆第一个节点并存储到哈希表中
        visited[node] = new Node(node->val);

        // 广度优先搜索
        while (!Q.empty()) {
            // 取出队列的头节点
            auto n = Q.front();
            Q.pop();
            // 遍历该节点的邻居
            for (auto& neighbor: n->neighbors) {
                if (visited.find(neighbor) == visited.end()) {
                    // 如果没有被访问过，就克隆并存储在哈希表中
                    visited[neighbor] = new Node(neighbor->val);
                    // 将邻居节点加入队列中
                    Q.push(neighbor);
                }
                // 更新当前节点的邻居列表
                visited[n]->neighbors.emplace_back(visited[neighbor]);
            }
        }

        return visited[node];
    }
};
```

### 难题

[127 单词接龙](https://leetcode-cn.com/problems/word-ladder/)

- 简介：
  - 我们可以把每个单词都抽象为一个点，如果两个单词可以只改变一个字母进行转换，那么说明他们之间有一条双向边。因此我们只需要把满足转换条件的点相连，就形成了一张**图**。
  - 基于该图，寻找 `beginWord` 到 `endWord` 的最短路径。

- 法一 广度优先搜索 BFS：
  
  ``` C++
  class Solution {
  public:
      unordered_map<string, int> wordId;
      vector<vector<int>> edge;
      int nodeNum = 0;
  
      void addWord(string& word) {
          if (!wordId.count(word)) {
              wordId[word] = nodeNum++;
              edge.emplace_back();
          }
      }
  
      void addEdge(string& word) {
          addWord(word);
          int id1 = wordId[word];
          for (char& it : word) {
              char tmp = it;
              it = '*';
              addWord(word);
              int id2 = wordId[word];
              edge[id1].push_back(id2);
              edge[id2].push_back(id1);
              it = tmp;
          }
      }
  
      int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
          for (string& word : wordList) {
              addEdge(word);
          }
          addEdge(beginWord);
          if (!wordId.count(endWord)) {
              return 0;
          }
          vector<int> dis(nodeNum, INT_MAX);
          int beginId = wordId[beginWord], endId = wordId[endWord];
          dis[beginId] = 0;
  
          queue<int> que;
          que.push(beginId);
          while (!que.empty()) {
              int x = que.front();
              que.pop();
              if (x == endId) {
                  return dis[endId] / 2 + 1;
              }
              for (int& it : edge[x]) {
                  if (dis[it] == INT_MAX) {
                      dis[it] = dis[x] + 1;
                      que.push(it);
                  }
              }
          }
          return 0;
      }
  };
  ```

- 法二 双向广度优先遍历 BBFS
  
``` c++
class Solution {
public:
    unordered_map<string, int> wordId;
    vector<vector<int>> edge;
    int nodeNum = 0;

    void addWord(string& word) {
        if (!wordId.count(word)) {
            wordId[word] = nodeNum++;
            edge.emplace_back();
        }
    }

    void addEdge(string& word) {
        addWord(word);
        int id1 = wordId[word];
        for (char& it : word) {
            char tmp = it;
            it = '*';
            addWord(word);
            int id2 = wordId[word];
            edge[id1].push_back(id2);
            edge[id2].push_back(id1);
            it = tmp;
        }
    }

    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        for (string& word : wordList) {
            addEdge(word);
        }
        addEdge(beginWord);
        if (!wordId.count(endWord)) {
            return 0;
        }

        vector<int> disBegin(nodeNum, INT_MAX);
        int beginId = wordId[beginWord];
        disBegin[beginId] = 0;
        queue<int> queBegin;
        queBegin.push(beginId);

        vector<int> disEnd(nodeNum, INT_MAX);
        int endId = wordId[endWord];
        disEnd[endId] = 0;
        queue<int> queEnd;
        queEnd.push(endId);

        while (!queBegin.empty() && !queEnd.empty()) {
            int queBeginSize = queBegin.size();
            for (int i = 0; i < queBeginSize; ++i) {
                int nodeBegin = queBegin.front();
                queBegin.pop();
                if (disEnd[nodeBegin] != INT_MAX) {
                    return (disBegin[nodeBegin] + disEnd[nodeBegin]) / 2 + 1;
                }
                for (int& it : edge[nodeBegin]) {
                    if (disBegin[it] == INT_MAX) {
                        disBegin[it] = disBegin[nodeBegin] + 1;
                        queBegin.push(it);
                    }
                }
            }

            int queEndSize = queEnd.size();
            for (int i = 0; i < queEndSize; ++i) {
                int nodeEnd = queEnd.front();
                queEnd.pop();
                if (disBegin[nodeEnd] != INT_MAX) {
                    return (disBegin[nodeEnd] + disEnd[nodeEnd]) / 2 + 1;
                }
                for (int& it : edge[nodeEnd]) {
                    if (disEnd[it] == INT_MAX) {
                        disEnd[it] = disEnd[nodeEnd] + 1;
                        queEnd.push(it);
                    }
                }
            }
        }
        return 0;
    }
};
```

- 法三 迪杰斯科拉 Dijkstra：

  ``` c++
  class Solution {
      bool isConnected(string& first, string& second) {
          int n = first.size();
          bool flag = true;
          for (int i = 0; i < n; i++) {
              if (first[i] == second[i])
                  continue;
              else if(first[i] != second[i] && !flag)
                  return false;
              else    
                  flag = false;
          }
          return true;
      }
  public:
      int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
          int res = 0;
          int n = wordList.size();
          int end, begin;
          vector<int> distances(n, -1);
          vector<vector<int>> adj(n);
          //小顶堆
          auto cmp=[](pair<int,int> a,pair<int,int> b){
              return a.second<b.second;
          };
          priority_queue<pair<int,int>,vector<pair<int,int>>,decltype(cmp)> pq(cmp);
          
          for (end = 0; end < n; end++) {
              if (wordList[end] == endWord) {
                  res = 1;
                  break;
              }
          }
  
          if (res == 0)
              return 0;
  
          for (begin = 0; begin < n; begin++) {
              if (isConnected(wordList[begin], beginWord)) {
                  if (wordList[begin] == beginWord) 
                      distances[begin] = 0;
                  else 
                      distances[begin] = 1;
                  pq.emplace(begin, distances[begin]);
              }
          }
          
          for (int i = 0; i < n; i++) {
              for (int j = i + 1; j < n; j++) {
                  if (isConnected(wordList[i], wordList[j])) {
                      adj[i].emplace_back(j);
                      adj[j].emplace_back(i);
                  }
              }
          }
  
          while (!pq.empty()) {
              int curr = pq.top().first;
              pq.pop();
              for (int next : adj[curr]) {
                  if (distances[next] == -1 || distances[next] > distances[curr] + 1) {
                      distances[next] = distances[curr] + 1;
                      pq.emplace(next, distances[next]);
                  }
              }
          }
  
          return distances[end] + 1;
      }
  };
  ```

[126 单词接龙Ⅱ](https://leetcode.cn/problems/word-ladder-ii/description/)

- 思路：BFS + HashMap + Backtracking
- 代码：
  
``` c++
class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string> &wordList) {
        vector<vector<string>> res;
        // 因为需要快速判断扩展出的单词是否在 wordList 里，因此需要将 wordList 存入哈希表，这里命名为「字典」
        unordered_set<string> dict = {wordList.begin(), wordList.end()};
        // 修改以后看一下，如果根本就不在 dict 里面，跳过
        if (dict.find(endWord) == dict.end()) {
            return res;
        }
        // 特殊用例处理
        dict.erase(beginWord);

        // 第 1 步：广度优先搜索建图
        // 记录扩展出的单词是在第几次扩展的时候得到的，key：单词，value：在广度优先搜索的第几层
        unordered_map<string, int> steps = {{beginWord, 0}};
        // 记录了单词是从哪些单词扩展而来，key：单词，value：单词列表，这些单词可以变换到 key ，它们是一对多关系
        unordered_map<string, set<string>> from = {{beginWord, {}}};
        int step = 0;
        bool found = false;
        queue<string> q = queue<string>{{beginWord}};
        int wordLen = beginWord.length();
        while (!q.empty()) {
            step++;
            int size = q.size();
            for (int i = 0; i < size; i++) {
                const string currWord = move(q.front());
                string nextWord = currWord;
                q.pop();
                // 将每一位替换成 26 个小写英文字母
                for (int j = 0; j < wordLen; ++j) {
                    const char origin = nextWord[j];
                    for (char c = 'a'; c <= 'z'; ++c) {
                        nextWord[j] = c;
                        if (steps[nextWord] == step) {
                            from[nextWord].insert(currWord);
                        }
                        if (dict.find(nextWord) == dict.end()) {
                            continue;
                        }
                        // 如果从一个单词扩展出来的单词以前遍历过，距离一定更远，为了避免搜索到已经遍历到，且距离更远的单词，需要将它从 dict 中删除
                        dict.erase(nextWord);
                        // 这一层扩展出的单词进入队列
                        q.push(nextWord);
                        // 记录 nextWord 从 currWord 而来
                        from[nextWord].insert(currWord);
                        // 记录 nextWord 的 step
                        steps[nextWord] = step;
                        if (nextWord == endWord) {
                            found = true;
                        }
                    }
                    nextWord[j] = origin;
                }
            }
            if (found) {
                break;
            }
        }
        // 第 2 步：回溯找到所有解，从 endWord 恢复到 beginWord ，所以每次尝试操作 path 列表的头部
        if (found) {
            vector<string> Path = {endWord};
            backtrack(res, endWord, from, Path);
        }
        return res;
    }

    void backtrack(vector<vector<string>> &res, const string &Node, unordered_map<string, set<string>> &from,
             vector<string> &path) {
        if (from[Node].empty()) {
            res.push_back({path.rbegin(), path.rend()});
            return;
        }
        for (const string &Parent: from[Node]) {
            path.push_back(Parent);
            backtrack(res, Parent, from, path);
            path.pop_back();
        }
    }
};
```
