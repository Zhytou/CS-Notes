# NoSQL

## 简介

### 概述

+ NoSQL = Not Only SQL，意即“不仅仅是SQL”
+ NoSQL泛指非关系型的数据库

### 目的

+ NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，特别是大数据应用难题

### 优缺点

**优点**

+ 高可扩展性
+ 分布式计算
+ 低成本
+ 架构灵活、半结构化数据
+ 没有复杂的关系

**缺点**

+ 没有标准化
+ 有限的查询功能
+ 最终一致是不直观的

## 特性 BASE

+ CAP理论指出，一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。
+ 传统的RDBMS强调一致性和可用性，因此有ACID特性；

+ 而不同于RDBMS，NoSQL更强调可用性和容错性，牺牲一致性，因此有BASE特性。
  - 说起来很有趣，BASE的英文意义是碱，而ACID是酸。真的是水火不容啊。

### 基本可用 Basically Available

+ NoSQL允许分布式系统中某些部分出现故障，那么系统的其余部分依然可用。它不会像ACID那样，在系统出现故障时，进行强制拒绝，允许继续部分访问。

### 软状态 Soft State

+ NoSQL在数据处理过程中，允许这个过程，存在数据状态暂时不一致的情况。但经过纠错处理，最终会一致的。

### 最终一致性 Eventually Consistency

+ NoSQL的软状态允许数据处理过程的暂时不一致，但是最终处理结果将是一致的，说明NoSQL对数据处理过程可以有短暂的时间间隔，也允许分更细的步骤一个一个地处理，最好数据达到一致即可。这在互联网上进行分布式应用具有其明显的优势。

## 分类

### 键值存储数据库



### 列存储数据库



### 文档型数据库



### 图像数据库
