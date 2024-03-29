# 关系型数据库管理系统 RDMS

- [关系型数据库管理系统 RDMS](#关系型数据库管理系统-rdms)
  - [简介](#简介)
    - [概述](#概述)
    - [特性 ACID](#特性-acid)
  - [技术](#技术)
    - [复制 Replication](#复制-replication)
    - [联合  Fedration](#联合--fedration)
    - [分片 Sharding](#分片-sharding)
    - [非规范化 Denormalization](#非规范化-denormalization)
    - [SQL调优  SQL Tuning](#sql调优--sql-tuning)

## 简介

### 概述

**描述**：

- 关系型数据库，是指采用了关系模型来组织数据的数据库。
- 其以行和列的形式存储数据，以便于用户理解，关系型数据库这一系列的行和列被称为表，一组表组成了数据库。

**概念**：

- 主键：主关键字(primary key)是表中的一个或多个字段，它的值用于唯一的标识表中的某一条记录。
- 外键：如果公共关键字在一个关系中是主关键字，那么这个公共关键字被称为另一个关系的外键。

### 特性 ACID

**原子性 Atomicity**：

- 事务的原子性指的是，事务中包含的程序作为数据库的逻辑工作单位，它所做的对数据修改操作要么全部执行，要么完全不执行。这种特性称为原子性。

**一致性 Consistency**：

- 事务的一致性指的是在一个事务执行之前和执行之后数据库都必须处于一致性状态。这种特性称为事务的一致性。假如数据库的状态满足所有的完整性约束，就说该数据库是一致的。

**隔离性 Isolation**：

- 分离性指并发的事务是相互隔离的。即一个事务内部的操作及正在操作的数据必须封锁起来，不被其它企图进行修改的事务看到。假如并发交叉执行的事务没有任何控制，操纵相同的共享对象的多个并发事务的执行可能引起异常情况。

**持久性 durability**：

- 持久性意味着当系统或介质发生故障时，确保已提交事务的更新不能丢失。即一旦一个事务提交，DBMS保证它对数据库中数据的改变应该是永久性的，即对已提交事务的更新能恢复。持久性通过数据库备份和恢复来保证。

## 技术

- 关系型数据库有许多扩展技术，包括：复制、联合、分片、非规范化和SQL调优。

### 复制 Replication

- 主从复制
- 主主复制

### 联合  Fedration

-

### 分片 Sharding

- 分片、分库、分表和分区

### 非规范化 Denormalization

- 非规范化要求适当地降低甚至抛弃关系模式的范式，不再要求一个表只描述一个实体或者实体间的一种联系。
- 非规范化试图以写入性能为代价来换取读取性能。在多个表中冗余数据副本，以避免高成本的联结操作。

### SQL调优  SQL Tuning
