# Pandas

- [Pandas](#pandas)
  - [Series](#series)
  - [Index](#index)
  - [DataFrame](#dataframe)
    - [index](#index-1)
    - [inspect](#inspect)
    - [sort](#sort)
    - [column \& row](#column--row)
    - [str](#str)
    - [datetime](#datetime)
    - [type](#type)
    - [loc \& iloc](#loc--iloc)
    - [subset](#subset)
    - [groupby](#groupby)
    - [merge](#merge)
  - [I/O](#io)

Pandas是基于NumPy的数学分析工具，是为了解决数据分析任务而创建的。它的名称来自面板数据（Panel Data）和数据分析（Data Analysis）两个名称的缩写。

## Series

尽管从一般意义上来讲，Series可以简单地被认为是一维的数组，但其实它更像是拥有序号作起默认键值的哈希表。这也是它和一维数组最主要的区别，即Series类型具有索引。

## Index

事实上，Series和DataFrame中的索引对象就是Index类型，它负责管理轴标签和其他元数据。

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
# 按index排序
df.sort_index(ascending=True, inplace=True)
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

### datetime

### type

```python
# 将a列类型视作float处理
df['diff'] = df['a'].astype(float)
# 选择df中bool类型的列
df.select_dtypes(include='bool')
```

### loc & iloc

```python
# loc 使用索引值和列名来定位
price_ap401 = df.loc['ap401', 'price']
# loc 使用行序号和列序号来定位
last_row = df.iloc[-1, :]
```

### subset

```python
# 选出price大于100的列，df['price'] > 100返回的实际是一个长度为len(df)的bool Series
df1 = df[df['price'] > 100]
# 选出tickers属于ap401, jm2401的行
df2 = df[df['ticker'].isin(['ap401', 'jm2401'])]
```

### groupby

```python
# 统计df内订单的数量、撤单率和交易笔数
def statOrd(group: pd.DataFrame):
  order = len(group)
  cancelledOrd = round(len(group[group['status'] == 'cancelled']) * 100.0 / order, 2)
  tradedLot = group[group['status'] == 'filled']['volume'].abs().sum()
  return pd.Series([order, cancelledOrd, tradedLot])
df = ordDF.groupby(['ticker', 'account', 'acctNo'], as_index=False).apply(statOrd)
```

### merge

```python
# 合并持仓和合约乘数表
volDF = pd.merge(volDF, volmulDF, on='ticker', how='left')
```

## I/O

Pandas提供了丰富的读取和导入函数，涉及各种文件类型和使用场景，常见的包括：csv、html、json和db等
