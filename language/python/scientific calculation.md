# 科学计算

- [科学计算](#科学计算)
  - [NumPy](#numpy)
    - [Array](#array)
    - [Array Index](#array-index)
    - [Array Math](#array-math)
  - [SciPy](#scipy)

## NumPy

NumPy（Numerical Python）是Python中科学计算的核心库。和MatLab类似，它提供了高性能的多维数组对象以及用于处理这些数组的工具。

### Array

Array与Python List的区别：NumPy数组Array中的每一个元素都必须是同一种数据类型，而Python List中的元素可以是不同的数据类型。

ndim查看维度；shape查看各维度的大小；size查看全部的元素个数，等于各维度大小的乘积；dtype可查看元素类型；dsize查看元素占位（bytes）大小。

### Array Index

```python
import numpy as np

# [[ 1  2  3  4]
#  [ 5  6  7  8]
#  [ 9 10 11 12]]
a = np.array([[1,2,3,4], [5,6,7,8], [9,10,11,12]])

# slicing
# [[2 3]
#  [6 7]]
b = a[:2, 1:3]

# integer indexing
# A slice of an array is a view into the same data, so modifying it
# will modify the original array.
print(a[0, 1])   # Prints "2"
b[0, 0] = 77     # b[0, 0] is the same piece of data as a[0, 1]
print(a[0, 1])   # Prints "77"

# boolean indexing
bool_idx = (a > 2)   # Find the elements of a that are bigger than 2;
                     # this returns a numpy array of Booleans of the same
                     # shape as a, where each slot of bool_idx tells
                     # whether that element of a is > 2.
```

### Array Math

``` python
import numpy as np

x = np.array([[1,2],[3,4]], dtype=np.float64)
y = np.array([[5,6],[7,8]], dtype=np.float64)

# Elementwise sum; both produce the array
# [[ 6.0  8.0]
#  [10.0 12.0]]
print(x + y)
print(np.add(x, y))

# Elementwise difference; both produce the array
# [[-4.0 -4.0]
#  [-4.0 -4.0]]
print(x - y)
print(np.subtract(x, y))

# Elementwise product; both produce the array
# [[ 5.0 12.0]
#  [21.0 32.0]]
print(x * y)
print(np.multiply(x, y))

# Elementwise division; both produce the array
# [[ 0.2         0.33333333]
#  [ 0.42857143  0.5       ]]
print(x / y)
print(np.divide(x, y))

# Elementwise square root; produces the array
# [[ 1.          1.41421356]
#  [ 1.73205081  2.        ]]
print(np.sqrt(x))
```

## SciPy

SciPy（Scientific Python）是用于高级科学计算的另一个核心库。它基于 NumPy 构建，并扩展了其功能，提供了许多友好且高效的模块，用于科学和数值计算。
