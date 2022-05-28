# 动态规划

## 基础概念



## 经典例题

### 中等题

#### [64 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

+ 思路

+ 代码

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


### 难题

#### [688 骑士在棋盘上的概率](https://leetcode-cn.com/problems/knight-probability-in-chessboard/)

+ 思路：

  - 一个骑士有 8 种可能的走法，骑士会从中以等概率随机选择一种。部分走法可能会让骑士离开棋盘，另外的走法则会让骑士移动到棋盘的其他位置，并且剩余的移动次数会减少 1。
  - 因此，定义dp\[step]\[i][j]表示骑士从棋盘上的点(i, j)出发，走了step步时的概率
    * 特别地，当点(i, j)不在棋盘上时，dp\[step]\[i][j] = 0；
    * 当step = 0时， dp\[step]\[i][j] = 1；
    * 对于其他情况，dp\[step]\[i][j] 为 dp\[step - 1]\[i][j] 之和除以 8 。
  - 其中， dp\[step - 1]有八种坐标偏移量，具体见题

+ 代码：

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

  