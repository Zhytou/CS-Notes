# 二分查找

- [二分查找](#二分查找)
  - [基础概念](#基础概念)
    - [简介](#简介)
    - [算法](#算法)
    - [实现](#实现)
    - [注意](#注意)
    - [常见问题](#常见问题)
  - [经典题目](#经典题目)
    - [基础题](#基础题)
    - [中等题](#中等题)
  - [参考](#参考)

## 基础概念

### 简介

- 二分查找是一种效率较高的查找方法。
- 但是，它要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。

### 算法

- 首先，假设表中元素是按升序排列，将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；
- 否则利用中间位置记录将表分成前、后两个子表，如果中间位置记录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。
- 重复以上过程，直到找到满足条件的记录，使查找成功，或直到子表不存在为止，此时查找不成功。

### 实现

``` c++
int binarySearch(const vector<int>& nums, int target) {
    int left = 0, right = nums.size();
    int mid;
    while (left < right) {
        mid = left + (right - left)/2
        if (nums[mid] == target) {
            return mid;
        }
        else if (nums[mid] < target) {
            left = mid + 1;
        }
        else {
            right = mid;
        }
    }
}
```

或

``` c++
int binarySearch(const vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    int mid;
    while (left <= right) {
        mid = left + (right - left)/2
        if (nums[mid] == target) {
            return mid;
        }
        else if (nums[mid] < target) {
            left = mid + 1;
        }
        else {
            right = mid - 1;
        }
    }
}
```

### 注意

- 取中值有两种避免溢出的写法：
  - `mid = left + (right - left)/2`，取值范围是`[left,right)`，查找左边界
  - `mid = right - (right - left)/2`，取值范围是`(left,right]`，查找右边界
- 跳出循环的条件同样也有两种：
  - `while (left < right)`，`right`初始化为`nums.size()`
  - `while (left <= right)`，`right`初始化为`nums.size()-1`
  - 此外，为了避免特殊情况，上述两种跳出边界的方法，还需要分别进行特例晒出。前者只需检验=0时，后者则需要检验0和`nums.size()-1`

时间复杂度为`O(logn)`

### 常见问题

- 最普通的：查找某个target是否存在于某个元素互不相同的有序数组中。
  - 比如：[74 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/description/)
- 重复变式：有序数组中元素可能相同。
  - 比如：[34 在排序数组中查找元素的第一个位置和第二个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
- 排序变式：有序数组发生旋转，出现山峰。
  - 比如：[33 搜索选择排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)、[153 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/description/)
  - 理解这一类型题目的关键在于：明白二分搜索必须发生在一个排序数组中，也就是说旋转排序中使用二分搜索，必须每次找出分割得到的两个区间，哪一个是完全升序的。
  - 换句话说，每次使用left和right计算出mid，并用`nums[mid]`和`nums[0]`进行比较，即下面代码

```c++
        if (nums[0] <= nums[mid]) {
            if (nums[0] <= target && target < nums[mid]) {
                r = mid - 1;
            } else {
                l = mid + 1;
            }
        } else {
            if (nums[mid] < target && target <= nums[n - 1]) {
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
```

## 经典题目

### 基础题

[34 在排序数组中查找元素的第一个位置和第二个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

- 思路：

  - 使用模板找到目标元素出现
  - 再遍历或者使用`lower_bound`和`upper_bound`找到出现范围

- 代码：

  ``` c++
  class Solution {
  public:
      vector<int> searchRange(vector<int>& nums, int target) {
          int left = 0, right = nums.size(), mid;
          bool flag = false;
          while (left < right) {
              mid = left + (right - left)/2;
              if (nums[mid] == target) {
                  flag = true;
                  break;
              }
              else if (nums[mid] < target) 
                  left = mid + 1;
              else 
                  right = mid;
          }
          if (!flag)
              return {-1, -1};
          left = mid;
          while (left >= 0 && nums[left] == target)
              left -= 1;
          right = mid;
          while (right < nums.size() && nums[right] == target)
              right += 1;
          return {left + 1, right - 1};
      }
  };
  ```

### 中等题

[240 搜索二维矩阵](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

- 思路：
  把矩阵逆时针选择45°，将原矩阵右上角顶点当作根节点，将其视为一个二叉搜索树结构。
- 代码：
  
  ``` c++
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int m = matrix.size(), n = matrix[0].size();
        int x = 0, y = n - 1;
        while (x < m && y >= 0) {
            if (matrix[x][y] == target) {
                return true;
            }
            if (matrix[x][y] > target) {
                --y;
            }
            else {
                ++x;
            }
        }
        return false;
    }
  ```

[剑指offer11 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

- 简介：与[153](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)题类似

- 思路：

  - 对于旋转之后的数组的末尾数值与最小值有以下性质：
    - 末尾数值一定小于最小值左侧的数
    - 末尾数值一定大于最小值右侧的数
  - 我们应当意识到，==该性质对于包含最小值的区间也同样成立==
  - 根据上述性质，我们可以去实现一个二分查找算法
    - 当区间中值小于右边界时，说明该中值在最小值右边，更新右边界
    - 当区间中值大于右边界时，说明该中值在最小值左边，更新左边界
    - 由于该数组中可能允许存在重复数字，因此当区间中值与右边界相等时，移动右边界

- 代码：

  ``` c++
  class Solution {
  public:
      int minArray(vector<int>& numbers) {
          int low = 0;
          int high = numbers.size() - 1;
          while (low < high) {
              int pivot = low + (high - low) / 2;
              if (numbers[pivot] < numbers[high]) {
                  high = pivot;
              }
              else if (numbers[pivot] > numbers[high]) {
                  low = pivot + 1;
              }
              else {
                  high -= 1;
              }
          }
          return numbers[low];
      }
  };
  ```

[33 搜索选择排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

理解这道题的关键就是明白这几点：

- 只有在顺序区间内才可以通过区间两端的数值判断target是否在其中。
- 判断顺序区间还是乱序区间，只需要对比 left 和 right 是否是顺序对即可，left <= right，顺序区间，否则乱序区间。
- 每次二分都会至少存在一个顺序区间。（感谢@Gifted VVilburgiX补充）

通过不断的用Mid二分，根据定理二，将整个数组划分成顺序区间和乱序区间，然后利用定理一判断target是否在顺序区间，如果在顺序区间，下次循环就直接取顺序区间，如果不在，那么下次循环就取乱序区间。

```c++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int n = (int)nums.size();
        if (!n) {
            return -1;
        }
        if (n == 1) {
            return nums[0] == target ? 0 : -1;
        }
        int l = 0, r = n - 1;
        while (l <= r) {
            int mid = (l + r) / 2;
            if (nums[mid] == target) return mid;
            if (nums[0] <= nums[mid]) {
                if (nums[0] <= target && target < nums[mid]) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else {
                if (nums[mid] < target && target <= nums[n - 1]) {
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
        }
        return -1;
    }
};
```

[162 寻找峰值](https://leetcode-cn.com/problems/find-peak-element/)

- 思路：
- 代码：

  ``` c++
  class Solution {
  public:
      int findPeakElement(vector<int>& nums) {
          int n = nums.size();
  
          // 辅助函数，输入下标 i，返回一个二元组 (0/1, nums[i])
          // 方便处理 nums[-1] 以及 nums[n] 的边界情况
          auto get = [&](int i) -> pair<int, int> {
              if (i == -1 || i == n) {
                  return {0, 0};
              }
              return {1, nums[i]};
          };
  
          int left = 0, right = n - 1, ans = -1;
          while (left <= right) {
              int mid = (left + right) / 2;
              if (get(mid - 1) < get(mid) && get(mid) > get(mid + 1)) {
                  ans = mid;
                  break;
              }
              if (get(mid) < get(mid + 1)) {
                  left = mid + 1;
              }
              else {
                  right = mid - 1;
              }
          }
          return ans;
      }
  };
  ```

## 参考

[二分查找有几种写法](https://www.zhihu.com/question/36132386)

[二分查找的细节](https://cloud.tencent.com/developer/article/1934759)
