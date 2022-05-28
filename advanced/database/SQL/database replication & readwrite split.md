# 数据库复制与读写分离

## 简介

**描述**

+ 数据库复制是一种提高系统可用性的手段。

**缺陷**

+ 如果主库在将新写入的数据复制到其他节点前挂掉，则有数据丢失的可能。
+ 写入会被重放到负责读取操作的副本。副本可能因为过多写操作阻塞住，导致读取功能异常。
+ 读取从库越多，需要复制的写入数据就越多，导致更严重的复制延迟。
+ 在某些数据库系统中，写入主库的操作可以用多个线程并行写入，但读取副本只支持单线程顺序地写入。
+ 复制意味着更多的硬件和额外的复杂度。

## 分类

### 主从复制 Master-Slave Replication

![](../../img/database_replication_ms.png)

**描述**

+ 主库同时负责读取和写入操作，并复制写入到一个或多个从库中，从库只负责读操作。
+ 如果主库离线，系统可以以只读模式运行，直到某个从库被提升为主库或有新的主库出现。

**缺陷**

+ 将从库提升为主库需要额外的逻辑。

### 主主复制 Master-Master Replication

![](../../img/database_replication_mm.png)

**描述**

+ 两个主库都负责读操作和写操作，写入操作时互相协调。如果其中一个主库挂机，系统可以继续读取和写入。

**缺陷**

+ 你需要添加负载均衡器或者在应用逻辑中做改动，来确定写入哪一个数据库。
+ 多数主-主系统要么不能保证一致性（违反 ACID），要么因为同步产生了写入延迟。
+ 随着更多写入节点的加入和延迟的提高，如何解决冲突显得越发重要。

### 树型复制 Tree Replication

![](../../img/database_replication_tree.png)

**描述**

+ 在主从复制的基础上，增添了树型结构。

### 伙伴复制 Buddy Replication

![](../../img/database_replication_buddy.png)

## 参考

+ [system-design-primer](https://github.com/donnemartin/system-design-primer)
+ [scalability-availability-stability-patterns](https://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
+ [replication patterns](https://livebook.manning.com/book/reactive-design-patterns/chapter-13/63)

