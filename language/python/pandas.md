# Pandas

- [Pandas](#pandas)
  - [Series](#series)
  - [DataFrame](#dataframe)
    - [index](#index)
    - [inspect](#inspect)
    - [sort](#sort)
    - [column \& row](#column--row)
    - [str](#str)
    - [type](#type)

Pandas是基于NumPy的数学分析工具，是为了解决数据分析任务而创建的。它的名称来自面板数据（Panel Data）和数据分析（Data Analysis）两个名称的缩写。

## Series

尽管从一般意义上来讲，Series可以简单地被认为是一维的数组，但其实它更像是拥有序号作起默认键值的哈希表。这也是它和一维数组最主要的区别，即Series类型具有索引。

## DataFrame

DataFrame是将数个Series按列合并而成的二维数据结构，每一列单独取出来是一个Series。所以，按列对一个DataFrame进行处理更为方便，用户在编程时注意培养按列构建数据的思维。

### index

index是dataframe中特殊的一列，默认是从0开始的一串数字，用于定位行。

```python
# 将department列设置为index
df.set_index("department", inplace=True)
# 将index重置为默认index
df.reset_index(inplace=True)
# 将列重新排序
df.reset_index(columns=[], inplace=True)
```

### inspect

```python
# 展示df的行列数
df.shape
# 展示表头信息
df.info()
# 展示表的统计信息
df.describe()
# 展示df表的前20行
df.head(20)
# 展示df表的后10行
df.tail(10)
```

### sort

```python
# 按diff列倒序排序
df.sort_values(by=['diff'], ascending=False, inplace=True)
```

### column & row

```python
# 添加一列
df['c'] = df['a']
# 删除一列
df.drop(columns=['a', 'b'], inplace=True)
# 删除行
df.dropna(inplace=True)
df.drop(2, axis='index')
```

### str

### type

```python

df['diff'] = df['a'].astype(float)
```
