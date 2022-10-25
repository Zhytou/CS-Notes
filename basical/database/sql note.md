# SQL

## 语法

### 创建

``` sql
//
create database
//
create table
```

### 修改

``` sql
```

### 插入

``` sql
INSERT INTO mytable(col1, col2)
VALUES(val1, val2);
```

### 更新

``` sql
UPDATE mytable
SET col1 = val1
WHERE col2 = val2;
```

### 删除

``` sql
DELETE FROM mytable
WHERE col = val;
```

### 查询

``` sql
SELECT col1
FROM mytable 
where col2 = val;
```

### 排序

``` sql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

### 连接

### 函数

## 优化

+ 尽量避免select *命令
  + 从表中读取越多的数据，查询会变得更慢。这是因为==不必要的数据增加了磁盘读取的时间==，考虑分布式数据库可能会因为网络延迟而更慢。

+ 预先知道只有一条返回结果，推荐使用`limit 1`
  + 通过加上`limit 1`，当一条相关的记录被查询到时，数据库==不会继续**扫表**==，而是返回结果

+ 优化查询语句避免索引失效
  + like用%开头
  + 不要在索引列上做函数计算和转换(隐式和显式都不要)
  + <>和!= 会使索引失效

+ 此外，还可以通过==慢查询日志==发现有效率问题的SQL，以及==explain命令==查看SQL的执行计划
