# 二分查找

## 基础概念

### 简介

+ 二分查找是一种效率较高的查找方法。
+ 但是，它要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。

### 算法

+ 首先，假设表中元素是按升序排列，将表中间位置记录的关键字与查找关键字比较，如果两者相等，则查找成功；
+ 否则利用中间位置记录将表分成前、后两个子表，如果中间位置记录的关键字大于查找关键字，则进一步查找前一子表，否则进一步查找后一子表。
+ 重复以上过程，直到找到满足条件的记录，使查找成功，或直到子表不存在为止，此时查找不成功。

### 实现

#### 模板

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

#### ==注意==

+ `mid = left + (right - left)/2`是为了避免溢出
+ 由于整除的缘故，`mid`的实际取值范围是`[left,right)`，即：左闭右开
  - 因此，更新`left`时，`left = mid + 1`；更新`right`时，`right = mid`
  - 同时，循环条件也为`left < right`，而非小于等于
  - 同样的原因，也使得`right`的初始值必须为`nums.size()`（必须保证数组所有元素可以取得到）

#### 时间复杂度

+ 时间复杂度为`O(logn)`

## 经典题目

### 基础题

#### [34 在排序数组中查找元素的第一个位置和第二个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

+ 思路：

  - 使用模板找到目标元素出现
  - 再遍历或者使用`lower_bound`和`upper_bound`找到出现范围

+ 代码： 

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

#### [剑指offer04 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

+ 思路：
+ 代码：

#### [剑指offer11 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

+ 简介：与[153](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)题类似

+ 思路：

  - 对于旋转之后的数组的末尾数值与最小值有以下性质：
    * 末尾数值一定小于最小值左侧的数
    * 末尾数值一定大于最小值右侧的数
  - 我们应当意识到，==该性质对于包含最小值的区间也同样成立==
  - 根据上述性质，我们可以去实现一个二分查找算法
    * 当区间中值小于右边界时，说明该中值在最小值右边，更新右边界
    * 当区间中值大于右边界时，说明该中值在最小值左边，更新左边界
    * 由于该数组中可能允许存在重复数字，因此当区间中值与右边界相等时，移动右边界

+ 代码：

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

#### [33 搜索选择排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

+ 思路：

+ 代码：

#### [162 寻找峰值](https://leetcode-cn.com/problems/find-peak-element/)

+ 思路：

+ 代码：

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



