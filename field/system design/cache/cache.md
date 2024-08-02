# 缓存 Cache

- [缓存 Cache](#缓存-cache)
  - [简介 Introduction](#简介-introduction)
    - [缓存概述](#缓存概述)
    - [缓存分类](#缓存分类)
    - [常见术语](#常见术语)
  - [映射分类 Cache Mapping](#映射分类-cache-mapping)
    - [直接关联 Direct Mapping](#直接关联-direct-mapping)
    - [全关联 Fully Associative Mapping](#全关联-fully-associative-mapping)
    - [N组关联 N-way Set Associative Mapping](#n组关联-n-way-set-associative-mapping)
  - [更新策略 Cache Replacement Policy](#更新策略-cache-replacement-policy)
    - [LRU](#lru)
  - [缓存模式 Cache Pattern](#缓存模式-cache-pattern)
    - [独立缓存模式 Cache-Aside](#独立缓存模式-cache-aside)
    - [直写模式 Write-Through](#直写模式-write-through)
    - [回写模式 Write-Back](#回写模式-write-back)
    - [刷新模式 Refresh-Ahead](#刷新模式-refresh-ahead)
  - [常见问题 Problem](#常见问题-problem)
    - [缓存穿透](#缓存穿透)
    - [缓存击穿](#缓存击穿)
    - [缓存雪崩](#缓存雪崩)
  - [参考](#参考)

## 简介 Introduction

### 缓存概述

- 缓存，最初是指CPU寄存器与主存之间的一种快速存储器。
- 如今，缓存的概念得到扩充，它可以泛指两种存储速度差异较大、用于协调两者数据传输差异的结构。
- 分布式系统中设计到的缓存，可以位于操作系统或者浏览器、客户端或者服务端、以及不同的缓存层。

### 缓存分类

**CDN 缓存**：

- CDN 也被是视作一种缓存。

**反向代理缓存**：

- 反向代理一般会缓存静态资源，从而降低对服务器的请求次数。

**数据库缓存**：

- 数据库的默认配置中通常包含缓存级别，针对一般用例进行了优化。

**应用缓存**：

- 基于内存的缓存比如 Redis 是应用程序和数据存储之间的一种键值存储。

### 常见术语

> 此处的术语主要是指计算机中高速缓存可能遇到的术语。

**block or line**：

The minimum unit of information that can be either present or not be present in a cache.

**hit rate**：

The fraction of memory accesses found in a level of the memory hierachy.

**miss penalty**：

The time required to fetch a block into a level of the memory hierachy from lower level.

**tag**：

A field in a table used for a memory hierachy that contains the address information required to identify whether the associated block in the hierachy corresponds to a requested word.

## 映射分类 Cache Mapping

> 这块内容主要是从计组中提取出来的

memory address = tag - index - byte offset

tag: identify if cache block hit

index:cache block index

### 直接关联 Direct Mapping

**描述**：

> Direct-mapped cache is a cache structure in which each memory location is mapped to exactly one location in the cache.

**实现**：

![缓存直接关联](../img/computer_architecture_cache_direct_mapping.png)

**优缺点**：

- 不用搜索，判断hit容易；
- 容易发生碰撞替换。

### 全关联 Fully Associative Mapping

**描述**：

> Fully associative cache is a cache structure in which a block can be placed in any location in the cache.

**实现**：

**优缺点**：

- 确认是否hit时，需要全文搜索；
- 发生碰撞替换的概率低。

### N组关联 N-way Set Associative Mapping

**描述**：

Set-associative cache is a cache that has a fixed number of locations(at least two) where each block can be placed.

An n-way set-associative cache consists of  a number of sets, each of which consists of n blocks. Each block in the memory maps to a unique set in the cache given by the index field, and a block can be placed any element of that set.

Set associative mapping is a middle range of design between direct mapping and fully associative mapping.

## 更新策略 Cache Replacement Policy

### LRU

## 缓存模式 Cache Pattern

- 由于你只能在缓存中存储有限的数据，所以你需要选择一个适用于你系统的缓存更新策略。
- 此处主要介绍的模式均为分布式缓存模式。

### 独立缓存模式 Cache-Aside

**定义**：

- 应用从存储器读写。缓存不和存储器直接交互，应用执行以下操作：
  - 在缓存中查找记录，如果所需数据不在缓存中
  - 从数据库中加载所需内容
  - 将查找到的结果存储到缓存中
  - 返回所需内容
- [Memcached](https://memcached.org/) 通常用这种方式使用。
- 添加到缓存中的数据读取速度很快。缓存模式也称为延迟加载。只缓存所请求的数据，这避免了没有被请求的数据占满了缓存空间。

**缺陷**：

- 请求的数据如果不在缓存中就需要经过三个步骤来获取数据，这会导致明显的延迟。
- 如果数据库中的数据更新了会导致缓存中的数据过时。这个问题需要通过设置 TTL 强制更新缓存或者直写模式来缓解这种情况。
- 当一个节点出现故障的时候，它将会被一个新的节点替代，这增加了延迟的时间。

### 直写模式 Write-Through

**定义**：

- 应用使用缓存作为主要的数据存储，将数据读写到缓存中，而缓存负责从数据库中读写数据。
  - 应用向缓存中添加/更新数据；
  - 缓存同步地写入数据存储；
  - 返回所需内容。

**缺陷**：

- 由于故障或者缩放而创建的新的节点，新的节点不会缓存，直到数据库更新为止。缓存应用直写模式可以缓解这个问题。
- 写入的大多数数据可能永远都不会被读取，用 TTL 可以最小化这种情况的出现。

### 回写模式 Write-Back

**定义**：

- 在回写模式中，应用执行以下操作：
  - 在缓存中增加或者更新条目；
  - 异步写入数据，提高写入性能。

**缺陷**：

- 缓存可能在其内容成功存储之前丢失数据。
- 执行直写模式比缓存或者回写模式更复杂。

### 刷新模式 Refresh-Ahead

**定义**：

- 你可以将缓存配置成在到期之前自动刷新最近访问过的内容。
- 如果缓存可以准确预测将来可能请求哪些数据，那么刷新可能会导致延迟与读取时间的降低。

**缺点**：

- 不能准确预测到未来需要用到的数据可能会导致性能不如不使用刷新。

## 常见问题 Problem

### 缓存穿透

**定义**：

- 缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

**解决**：

- 增加查询校验；
- 将这个不存在的键的值写为null，防止反复使用一个健攻击。

### 缓存击穿

**定义**：

- 缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。

**解决**：

- 设置热点数据不过期；
- 加锁，避免多个线程同时查询一个键的值。

### 缓存雪崩

**定义**：

- 缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。
- 和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

**解决**：

- 缓存过期时间设置随机，避免同一时间大量数据过期。

## 参考

**简介**：

- [wiki - cache](https://zh.wikipedia.org/wiki/%E7%BC%93%E5%AD%98)

**映射**：

- [Computer Organization and Design The Hardware Software Interface [RISC-V Edition] by David A. Patterson, John L. Hennessy (z-lib.org) 第五章](/)
