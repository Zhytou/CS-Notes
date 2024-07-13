# 排序

- [排序](#排序)
  - [选择排序 Select Sort](#选择排序-select-sort)
  - [冒泡排序 Bubble Sort](#冒泡排序-bubble-sort)
  - [插入排序 Insertion Sort](#插入排序-insertion-sort)
  - [希尔排序 Shell Sort](#希尔排序-shell-sort)
  - [快速排序 Quick Sort](#快速排序-quick-sort)
  - [归并排序 Merge Sort](#归并排序-merge-sort)
  - [桶排序 Bucket Sort](#桶排序-bucket-sort)
  - [堆排序 Heap Sort](#堆排序-heap-sort)

**稳定性**：

排序算法的稳定性与其时间复杂度无直接关系。排序算法的稳定性是指在排序过程中，相等元素的相对顺序是否保持不变。十大排序算法中稳定算法包括：冒泡排序、插入排序和归并排序。

**使用场景**：

- 冒泡排序和选择排序：这两种排序算法比较简单，适用于小规模的数据集合。它们的时间复杂度都为O(n^2)，因此在数据量较大时效率较低。
- 插入排序：插入排序适用于部分有序的数据集合，或者数据量较小的情况。它的时间复杂度也为O(n^2)，但在数据基本有序的情况下，插入排序的效率会比较高。
- 希尔排序：希尔排序适用于大规模的数据集合和中等大小的数组。它通过将序列分组，逐渐减小步长，最后进行插入排序，可以在一开始就将较小的元素快速移动到序列的前部分，从而提高排序的效率。
- 归并排序：归并排序适用于任意规模的数据集合。它的时间复杂度为O(nlogn)，且具有稳定性，因此在需要稳定排序的场景中比较常用。
- 快速排序：快速排序适用于任意规模的数据集合。它的平均时间复杂度为O(nlogn)，但在最坏情况下可能达到O(n^2)。快速排序是一种高效的排序算法，通常比归并排序更快，因此在大规模数据集合的排序中经常使用。
- 堆排序：堆排序适用于任意规模的数据集合。它的时间复杂度为O(nlogn)，且具有稳定性。堆排序利用堆的性质进行排序，适用于对大量数据进行排序的场景。
- 桶排序：桶排序适用于数据范围较大，但分布均匀的情况。它的时间复杂度为O(n+k)，其中k是桶的数量。桶排序将数据分配到不同的桶中，然后对每个桶进行排序，最后将所有桶的元素合并起来得到有序序列。
- 基数排序：基数排序适用于数据范围较大，且每个元素都有多个关键字的情况。它的时间复杂度为O(kn)，其中k是关键字的数量。基数排序按照关键字的每个位进行排序，从低位到高位，最终得到有序序列。

## 选择排序 Select Sort

选择排序是一种简单的排序算法，它的基本思想是每次从未排序的部分选择最小（或最大）的元素，然后将其放到已排序部分的末尾。选择排序的时间复杂度为O(n^2)，适用于小规模的数据集合。由于每次只交换一次元素，因此选择排序是一种不稳定的排序算法。

```c++
void selectSort(vector<int>& nums) {
  for (int i = 0; i < nums.size(); i++) {
    for (int j = i+1; j < nums.size(); j++) {
      if (nums[i] > nums[j]) {
        swap(nums[i], nums[j]);
      }
    }
  }
}
```

## 冒泡排序 Bubble Sort

冒泡排序是一种简单的排序算法，它的基本思想是通过相邻元素的比较和交换来将最大（或最小）的元素逐渐“冒泡”到数组的末尾。冒泡排序的时间复杂度为O(n^2)，适用于小规模的数据集合。冒泡排序是一种稳定的排序算法，因为在元素比较和交换的过程中，相同元素的相对位置不会改变。

```c++
void bubbleSort(vector<int>& nums) {
  for (int i = 0; i < nums.size(); i++) {
    for (int j = nums.size()-1; j > i; j--) {
        if (nums[j] < nums[j-1]) {
            swap(nums[j], nums[j-1]);
        }
    }
  }
}
```

## 插入排序 Insertion Sort

插入排序是一种简单直观的排序算法，它的基本思想是将数组分为已排序部分和未排序部分，每次从未排序部分选择一个元素插入到已排序部分的正确位置。插入排序的时间复杂度为O(n^2)，适用于部分有序的数据集合或者数据量较小的情况。插入排序是一种稳定的排序算法，因为它只在相邻元素之间进行比较和交换。

```c++
void insertSort(vector<int>& nums) {
    for (int i = 1; i < nums.size(); i++) {
        int j = 0;
        // 在已排序的[0, i-1]中找到待排序nums[i]数的插入位置j，并不断更新nums[i]，即把nums[i]当作一个tmp
        while (j < i) {
            if (nums[j] > nums[i]) {
                swap(nums[j], nums[i]);
            }
            j += 1;
        }
    }
}
```

## 希尔排序 Shell Sort

希尔排序是一种改进的插入排序算法，也被称为缩小增量排序。它的基本思想是将待排序序列按照一定的步长分组，对每个分组进行插入排序。随着排序的进行，逐渐减小步长，直到步长为1，此时对整个序列进行最后一次插入排序。希尔排序的时间复杂度取决于步长序列的选择，不同的步长序列会导致不同的时间复杂度。一般来说，希尔排序的时间复杂度介于O(nlogn)和O(n^2)之间。希尔排序相对于插入排序的优势在于，它可以在一开始就将较小的元素快速移动到序列的前部分，从而减少后续的比较和交换操作，提高排序的效率。

```c++
void shellSort(vector<int>& nums) {
    for (int step = nums.size()/2+1; step >= 1; step /= 2) {
        // 对每个分组分别进行插入排序
        for (int group = 0; group < step; group++) {
            for (int i = group+step; i < nums.size(); i += step) {
                int j = group;
                while(j < i) {
                    if (nums[j] > nums[i]) {
                        swap(nums[j], nums[i]);
                    }
                    j += step;
                }
            }
        }    
    }
}
```

## 快速排序 Quick Sort

快速排序是一种基于分治思想的排序算法，也是最常用的排序算法之一。它的基本思想是选择一个基准元素，通过一系列的比较和交换操作，将序列分成两部分：小于基准元素的部分和大于基准元素的部分。然后对这两部分分别递归地进行快速排序，直到每个子序列只有一个元素。最后，将所有子序列合并起来，即可得到一个有序的序列。快速排序的平均时间复杂度为O(nlogn)，其中n为待排序序列的长度。然而，在最坏情况下，快速排序的时间复杂度可能达到O(n^2)。

``` c++
void quickSort(vector<int>& nums, int left, int right) {
  if(left >= right)
    return;
  int i, j, base;
  i = left, j = right;
  base = nums[left];  //取最左边的数为基准数
  while (i < j) {
    while (nums[j] >= base && i < j)
      j--;
    while (nums[i] <= base && i < j)
      i++;
    if(i < j) {
      swap(nums[i], nums[j]);
    }
  }
  //基准数归位
  nums[left] = nums[i];
  nums[i] = base;
  quickSort(nums, left, i - 1);//递归左边
  quickSort(nums, i + 1, right);//递归右边
}
```

## 归并排序 Merge Sort

归并排序是一种基于分治思想的排序算法。它将待排序序列不断拆分为更小的子序列，直到每个子序列只有一个元素。然后，通过将相邻的子序列合并成一个有序的序列，最终得到排序完成的序列。归并排序的时间复杂度为O(nlogn)，其中n为待排序序列的长度。

``` c++
void mergeSort(vector<int>& nums, int left, int right) {
  if (left >= right)
        return ;
    int mid = left + (right - left)/2;
    mergeSort(nums, left, mid);
    mergeSort(nums, mid + 1, right);
    // algorithm中函数
    merge(nums.begin() + left, nums.begin() + mid, nums.begin() + mid + 1, nums.begin() + right, nums.begin() + left);
    return ;
}
```

## 桶排序 Bucket Sort

桶排序是一种分布式排序算法，适用于元素均匀分布在某个范围内的情况。它的基本思想是将待排序序列分为一定数量的桶（通常是一个固定数量），然后将元素根据某种映射函数分配到对应的桶中。每个桶内的元素可以使用其他排序算法，比如插入排序，进行排序。最后，将各个桶中的元素按照顺序合并起来，即可得到一个有序的序列。

桶排序的时间复杂度取决于桶的数量和桶内排序算法的复杂度。它通常情况下是O(n+k)，其中n是待排序序列的长度，k是桶的数量。如果对每个桶内的元素使用快速排序等高效的排序算法，桶排序的时间复杂度可以接近O(n)。

桶排序适用于待排序序列分布均匀的情况，且元素值的范围不宜过大。当元素值范围过大时，可能会导致桶之间的差距较大，从而使得排序效率下降。此外，桶排序需要额外的空间来存储桶，因此对于内存有限的情况需要谨慎使用。

## 堆排序 Heap Sort

堆是一个近似完全二叉树的数据结构，其子节点总是大于或者小于其父节点。而堆排序正是利用这种数据结构所设计的一种排序算法。它的核心思想是将待排序的数据构建成一个二叉堆，然后逐步将堆顶元素与堆尾元素交换，并重新调整堆，使得剩余元素继续满足堆的性质。通过不断重复这个过程，最终可以得到一个有序的序列。堆排序的时间复杂度为O(nlogn)，其中n为待排序序列的长度。

具体来说，它的排序流程如下：

- 首先，在输入数组上建立大顶堆，即：`nums[i] >= nums[2*i + 1] && nums[i] >= nums[2*i + 2]`。
- 接着，将堆首与堆尾交换。因为堆首其实就是排序好的最大值，所以将最大的数放到队尾实际上就完成了这个数的排序。
- 此时，由于交换，新堆已经不符合堆定义，需要调整结构。
- 重复二三步直到完成排序。

``` c++
//此处堆化函数必须要有heapSize参数是因为排序时会把堆首与堆尾交换改变堆长度，因此堆化函数必须传入堆长度
void heapify(vector<int>& nums, int start, int heapSize) {
    //当用数组表示树时，若父节点序号为idx，则两个子节点分别为2*idx+1 和2*idx+2
    int dad = start, son = 2*dad + 1;
    while (son < heapSize) {
        //选出两个子节点中较大的
        if (son + 1 < heapSize && nums[son] < nums[son + 1])
            son += 1;
        //若此时父节点已经大于子节点，则已经符合堆规制；否则继续堆化。
        if (nums[dad] < nums[son]) {
            swap(nums[dad], nums[son]);
            dad = son;
            son = 2*dad + 1;
        }
        else
            break;
    }
}

void heapSort(vector<int>& nums) {
    //自下而上的建立堆
    for (int i = nums.size() / 2; i >= 0; i--)
        heapify(nums, i, nums.size());
    
    
    for (int i = nums.size() - 1; i > 0; i--) {
        //将堆顶最大值与堆尾交换，重新堆化
        //此处必须要传入heapSize参数，因为堆长度减少了1
        swap(nums[0], nums[i]);
        heapify(nums, 0, i);
    }
    
    return ;
}
```

**补充**：使用堆排序算法实现一个优先队列（大顶堆）

``` c++
template <typename T>
class priority_queue {
 private:
  std::vector<T> pq;

 public:
  void push(const T& val) {
    pq.push_back(val);
    int son = pq.size() - 1, dad = (son - 1) / 2;

    while (dad >= 0 && pq[dad] < pq[son]) {
      swap(pq[dad], pq[son]);
      son = dad;
      dad -= 1;
      dad /= 2;
    }
  }

  void pop() {
    swap(pq[0], pq[pq.size() - 1]);
    pq.pop_back();

    int dad = 0, son = 2 * dad + 1;
    while (son < pq.size() && pq[dad] < pq[son]) {
      if (son + 1 < pq.size() && pq[son] < pq[son + 1]) {
        son += 1;
      }
      swap(pq[dad], pq[son]);
      dad = son;
      son = 2 * dad + 1;
    }
  }

  T& top() { return pq[0]; }

  bool empty() { return pq.empty(); }
};
```
