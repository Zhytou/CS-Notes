# 数据库分片 Sharding

- [数据库分片 Sharding](#数据库分片-sharding)
  - [简介](#简介)
    - [概述](#概述)
    - [分类](#分类)
    - [优缺](#优缺)
      - [优势](#优势)
      - [缺陷](#缺陷)
  - [模式](#模式)
  - [其他](#其他)
    - [分区](#分区)
    - [分表](#分表)
    - [分库](#分库)
  - [参考](#参考)

## 简介

### 概述

**定义**：

- **Sharding** is the practice of optimizing database management systems by separating the rows or columns of a larger database table into multiple smaller tables.
- The new tables are called “shards” (or partitions), and each new table either has the same schema but unique rows (as is the case for “horizontal sharding”) .
- Or new table has a schema that is a proper subset of the original table’s schema (as is the case for “vertical sharding”).

**sharding vs partitioning**：

- 一种说法是二者几乎没有区别，但`sharding`强调多机器；
  > [sharding概述](https://hazelcast.com/glossary/sharding/): Sharding and partitioning are both about breaking up a large data set into smaller subsets. The difference is that sharding implies the data is spread across multiple computers while partitioning does not. Partitioning is about grouping subsets of data within a single database instance.

- 另一种说法是`sharding`仅特指`horizontal partitioning`。
  > [StackOverflow 回答](https://stackoverflow.com/questions/20771435/database-sharding-vs-partitioning): Partitioning is more a generic term for dividing data across tables or  databases. Sharding is one specific type of partitioning, part of what  is called horizontal partitioning.

### 分类

**水平分片**：

**垂直分片**：

### 优缺

#### 优势

- 分片的主要优势就是能帮助分布式系统进行水平扩展。

**水平扩展 Scaling Out**：

> Horizontal scaling also konw as scaling out is the practice of adding more machines to an existing stack in order to spread out the load and allow for more traffic and faster processing.

**垂直扩展 Scaling Up**：

> Vertical scaling, otherwise known as scaling up, which involves upgrading the hardware of an existing server, usually by adding more RAM or CPU.

#### 缺陷

- 你需要修改应用程序的逻辑来实现分片，这会带来复杂的 SQL 查询。
- 分片不合理可能导致数据负载不均衡。例如，被频繁访问的用户数据会导致其所在分片的负载相对其他分片高。
  - 再平衡会引入额外的复杂度。基于[一致性哈希](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html)的分片算法可以减少这种情况。
- 联结多个分片的数据操作更复杂。
- 分片需要更多的硬件和额外的复杂度。

## 模式

## 其他

数据库中有一系列类似分片的相关概念，具体解释如下：

### 分区

### 分表

### 分库

## 参考

- [Understanding Database Sharding](https://www.digitalocean.com/community/tutorials/understanding-database-sharding)
- [system-design-primer](https://github.com/donnemartin/system-design-primer#sharding)
- [An Overview of Sharding & Partitioning](https://hazelcast.com/glossary/sharding/)
