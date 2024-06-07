# SQL

- [SQL](#sql)
  - [简介](#简介)
  - [语法](#语法)
    - [创建](#创建)
    - [修改](#修改)
    - [插入](#插入)
    - [更新](#更新)
    - [删除](#删除)
    - [查询](#查询)
    - [排序](#排序)
    - [like关键字](#like关键字)
    - [连接](#连接)
  - [优化](#优化)
    - [MySQL的优化策略](#mysql的优化策略)
    - [常见的优化手段](#常见的优化手段)

## 简介

SQL(Structured Query Language)是用于管理和操作关系数据库的标准语言，它包括了多种不同的子集，用于执行不同的任务。以下是SQL的五个主要子集及其定义：

- DML(Data Manipulation Language)：DML用于处理数据库中的数据，包括插入、更新、删除和查询数据。比如：INSERT, UPDATE, DROP。
- DDL(Data Definition Language)：DDL用于定义数据库的结构，包括创建、修改和删除数据库对象，如表、视图、索引、存储过程等。比如：CREATE, ALTER, DROP, TRUNCATE。
- DCL(Data Control Language)：DCL用于控制数据库的访问权限和安全性，包括授权和撤销权限。比如：GRANT, REVOKE。
- DQL(Data Query Language)：DQL是SQL的一个子集，专门用于查询数据。虽然DQL不是SQL的官方子集，但通常用来区分查询操作。比如：SELECT。
- TCL(Transaction Control Language)：TCL用于管理数据库事务，包括开始、提交、回滚和保存点。比如：BEGIN, COMMIT, ROLLBACK, SAVEPOINT。

## 语法

### 创建

``` sql
/* 创建数据库 */
CREATE DATABASE mydatabase;
/* 使用该数据库，直到下一个USE指令 */
USE DATABASE mydatabase;
/* 创建表 */
CREATE TABLE t_info (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50),
  age INT
);
```

### 修改

``` sql
/* 修改表结构 */
ALTER TABLE mytable
ADD COLUMN address VARCHAR(100),
MODIFY COLUMN age INT NOT NULL,
DROP COLUMN name;
```

### 插入

``` sql
INSERT INTO mytable(name, age)
VALUES('John', 30);
```

### 更新

``` sql
UPDATE mytable
SET name = 'John Doe'
WHERE id = 1;
```

### 删除

``` sql
DELETE FROM mytable
WHERE id = 1;
```

### 查询

``` sql
SELECT name
FROM mytable 
where id = 1;
```

### 排序

``` sql
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

### like关键字

### 连接

## 优化

### MySQL的优化策略

### 常见的优化手段

- 尽量避免select *命令
  - 从表中读取越多的数据，查询会变得更慢。这是因为不必要的数据增加了磁盘读取的时间，考虑分布式数据库可能会因为网络延迟而更慢。

- 预先知道只有一条返回结果，推荐使用`limit 1`
  - 通过加上`limit 1`，当一条相关的记录被查询到时，数据库不会继续**扫表**，而是返回结果

- 优化查询语句避免索引失效
  - like用%开头
  - 不要在索引列上做函数计算和转换(隐式和显式都不要)
  - <>和!= 会使索引失效

- 此外，还可以通过慢查询日志发现有效率问题的SQL或使用explain命令查看SQL的执行计划。
