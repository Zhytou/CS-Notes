# Pandas

- [Pandas](#pandas)
  - [Series](#series)
  - [DataFrame](#dataframe)

## Series

尽管从一般意义上来讲，Series可以简单地被认为是一维的数组，但其实它更像是拥有序号作起默认键值的哈希表。这也是它和一维数组最主要的区别，即Series类型具有索引。

## DataFrame

DataFrame是将数个Series按列合并而成的二维数据结构，每一列单独取出来是一个Series。所以，按列对一个DataFrame进行处理更为方便，用户在编程时注意培养按列构建数据的思维。
