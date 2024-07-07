# 质数

埃氏筛法是一种用于找出质数的经典算法。它的执行流程如下：

- 假设序列中所有数字默认为质数。
- 从2开始，将2的所有倍数标记为非质数(即可除以2的倍数)。
- 然后考虑3，将3的所有倍数标记为非质数。
- 然后是5,将5的所有倍数标记为非质数。
- 由于4、6、8等数字在之前必定被标记为非质数，所以从现在考虑的数字开始，只需要把它本身和倍数乘积大于序列界的倍数进行标记即可。

其实例代码如下：

```c++
vector<int> prime(int n) {
   // 假设n个数全是质数
   vector<bool> sequence(n+1, true);
   for(i = 2; i <= n; i++)
      if(sequence[i]==true)  
         for(j = i*2; j <= n; j += i)
            sequence[j] = false

   vector<int> nums;
   for (int i = 2; i <= n; i++) {
      if (sequence[i]) {
         nums.push_back(i);
      }
   }
   return nums;
}
```
