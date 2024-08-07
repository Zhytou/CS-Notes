# 索引

- [索引](#索引)
  - [简介](#简介)
    - [作用](#作用)
    - [分类](#分类)
  - [实现](#实现)
  - [使用](#使用)
    - [联合索引和最左匹配原则](#联合索引和最左匹配原则)
    - [索引下推](#索引下推)
    - [索引失效](#索引失效)
    - [优化](#优化)

## 简介

索引是对数据库表中一或多个列的值进行排序的结构，是帮助数据库高效获取数据的数据结构。

### 作用

索引就是加快检索表中数据的方法。数据库的索引类似于书籍的索引。在书籍中，索引允许用户不必翻阅完整个书就能迅速地找到所需要的信息。在数据库中，索引也允许数据库程序迅速地找到表中的数据，而不必扫描整个数据库。

### 分类

**按数据结构分类**：

从数据结构的角来看，常见的索引包括：

- B+树索引：查找、排序和分组。
- Hash索引：只能查找，且无法范围查找。
- 自适应哈希索引：InnoDB存储引擎提供了一种特殊的索引，它结合了B+树索引和Hash索引的优点。其原理就是当某个索引值被使用的非常频繁时，会在B+树索引之上再创建一个哈希索引。这样就让B+树索引具有哈希索引的一些优点，比如快速的哈希查找。

事实上，每一种存储引擎能够支持的索引都有所不同，MySQL存储引擎支持索引如下图所示。

![MySQL存储引擎支持索引](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/%E7%B4%A2%E5%BC%95/%E7%B4%A2%E5%BC%95%E5%88%86%E7%B1%BB.drawio.png)

其中，InnoDB是在MySQL 5.5之后成为默认的MySQL存储引擎，而B+Tree索引类型也是MySQL存储引擎采用最多的索引类型。在创建表时，InnoDB 存储引擎会根据不同的场景选择不同的列作为索引：

- 如果有主键，默认会使用主键作为聚簇索引的索引键；
- 如果没有主键，就选择第一个不包含NULL值的唯一列作为聚簇索引的索引键；
- 在上面两个都没有的情况下，InnoDB将自动生成一个隐式自增id列作为聚簇索引的索引键；

**按物理存储分类**：

从物理存储的角度来看，索引分为主索引/聚簇索引、辅助索引/二级索引/非聚簇索引两种。其中，主键索引的 B+Tree 的叶子节点存放的是实际数据，所有完整的用户记录都存放在主键索引的 B+Tree 的叶子节点里；二级索引的 B+Tree 的叶子节点存放的是主键值，而不是实际数据。

因此，在查询时使用了二级索引，如果查询的数据能在二级索引里查询的到，那么就不需要回表，这个过程就是覆盖索引（比如查询的值其实就只针对该索引列）。如果查询的数据不在二级索引里，就会先检索二级索引，找到对应的叶子节点，获取到主键值后，然后再检索主键索引，就能查询到数据了，这个过程就是回表。

**按字段特性分类**：

从字段特性的角度来看，索引分为主键索引、唯一索引、普通索引、前缀索引。其中，主键索引就是建立在主键字段上的索引，通常在创建表的时候一起创建，一张表最多只有一个主键索引，索引列的值不允许有空值。比如：

```sql
CREATE TABLE table_name  (
  ....
  PRIMARY KEY (index_column_1) USING BTREE
);
```

唯一索引建立在UNIQUE字段上的索引，一张表可以有多个唯一索引，索引列的值必须唯一，但是允许有空值。它的创建可以使用如下方式：

```sql
/* 创建表时，创建唯一索引 */
CREATE TABLE table_name  (
  ....
  UNIQUE KEY(index_column_1,index_column_2,...) 
);
/* 建表之后，创建唯一索引 */
CREATE UNIQUE INDEX index_name
ON table_name(index_column_1,index_column_2,...); 
```

前缀索引是指对字符类型字段的前几个字符建立的索引，而不是在整个字段上建立的索引，前缀索引可以建立在字段类型为char、 varchar、binary、varbinary的列上。相比其他索引，前缀索引能有效减少索引占用的存储空间，并提升查询效率。它的创建方法如下：

```sql
/* 创建表时，创建前缀索引 */
CREATE TABLE table_name(
    column_list,
    INDEX(column_name(length))
); 
/* 建表之后，创建前缀索引 */
CREATE INDEX index_name
ON table_name(column_name(length)); 
```

普通索引就是建立在普通字段上的索引，既不要求字段为主键，也不要求字段为UNIQUE。

**按字段个数分类**：

从字段个数的角度来看，索引可以分为单列索引、联合索引。其中，建立在多列上的索引就叫联合索引，也成为复合索引。它的创建方式如下：

```sql
/* 将商品表中的 product_no 和 name 字段组合成联合索引(product_no, name) */
CREATE INDEX index_product_no_name ON product(product_no, name);
```

值得一提的是，联合索引的匹配遵循最左匹配原则，即按照左边优先的方式进行索引匹配。因此，如果不遵守最左匹配原则，索引就会失效，查询效率也会大大降低。

## 实现

**为什么数据库需要实现一套独立的磁盘操作机制，而不直接使用操作系统提供的文件管理？**

- 和文件系统相比，数据库需要更频繁的读写数据。因此，数据库需要设计一套独立的磁盘读写机制，其中采用了缓存、预读取、多线程、锁和MVCC等方法来实现这种高并发和高性能的访问需求。
- 数据库需要支持事务进而保证数据的一致性和完整性，而操作系统的文件管理并不提供这些高级特性。

尽管二者是相互独立的系统，但其中也有类似的概念，比如，操作系统和数据库系统都提出了页(Page)的概念。

对于操作系统来说，页是其内存管理的基本单位，用于虚拟内存和物理内存之间的映射。操作系统页的大小通常由操作系统决定，例如，常见的页大小可能是4KB或8KB。操作系统使用页来实现虚拟内存的分页机制，进行内存的分配、交换和保护。

而对于数据库来说，页是其内部管理数据的基本单位，通常用于存储数据行、索引项等。数据库页的大小是数据库系统定义的，例如，InnoDB存储引擎的页大小通常是16KB。数据库系统使用页来组织数据，进行I/O操作，并且页通常与磁盘上的物理块对齐，以优化磁盘I/O。

**为什么采用B+树作为实现索引的数据结构？**：

数据库系统的设计需要权衡查询效率和存储空间的利用率。由于数据按页为基本单位进行读写操作，每次查询都可能触发页的I/O操作。也就是说，每当读取一条记录时，如果缓存中没有这条记录，就需要将其所在的页完整的从磁盘读取到主存中。所以，我们希望索引的数据结构能在尽可能少的磁盘的I/O操作中完成查询工作，同时还能支持范围查询等高效操作。

最容易想到的数据结构可能就是二叉搜索树(Binary Search Tree, BST)，它按照键值大小将节点排序，因此支持快速查找等操作。但BST有可能退化成链表，而导致查询效率下降。为了改善这一问题，我们可以将其改进成自平衡二叉搜索树(AVL Tree)。它的特点是任何节点的两个子树的高度差不超过1，因此能够保证查找、插入和删除操作的时间复杂度为O(log n)。

不过，AVL树的节点仅存储较少的键值对数据，无法有效减少磁盘I/O次数。更好的选择是多路搜索树(Mutiway Tree)，每个节点可以存储更多的键值对数据。著名的B树就是这种结构的一个例子，它是一种自平衡的多路搜索树，能够有效降低树的高度，从而降低查询过程中的磁盘I/O次数。

B+树是B树的一个变种，相比之下有更优秀的性能。B+树的数据都存储在叶子节点，叶子节点之间通过链表相连，便于范围查找;非叶子节点只存储键值，作为叶子节点的索引，从而较少磁盘访问次数。综合以上优点，B+树成为了数据库索引选用的最佳数据结构。

## 使用

### 联合索引和最左匹配原则

在实际工程开发中，我们通常会在表中建立多个联合索引而非单列索引。其原因有二，分别是：

- 建立联合索引相比建立多个普通索引能减少开销。建一个联合索引(col1,col2,col3)，实际相当于建了(col1),(col1,col2),(col1,col2,col3)三个索引。每多一个索引，都会增加写操作的开销和磁盘空间的开销。对于大量数据的表，使用联合索引会大大的减少开销！
- 联合索引能够大大减少回表查询的概率从而提高查询效率。对联合索引(col1,col2,col3)，如果有如下的sql: select col1,col2,col3 from test where col1=1 and col2=2。那么MySQL可以直接通过遍历索引取得数据，而无需回表，这减少了很多的随机io操作。减少io操作，特别的随机io其实是dba主要的优化策略。所以，在真正的实际应用中，覆盖索引是主要的提升性能的优化手段之一。

而使用联合索引最重要的规则就是遵循最左匹配原则。最左匹配原则指的是在查询时，可以只使用索引中的一部分，但只能是最左侧的部分。例如，如果索引是(a, b, c)，那么可以支持a、a,b、a,b,c三种组合进行查找，但不支持b,c进行查找。当使用最左侧的字段时，索引的效果最好。

### 索引下推

索引下推(Index Condition Pushdown, ICP)是MySQL 5.6版本中提供的一项索引优化功能，它允许存储引擎在索引遍历过程中，执行部分WHERE字句的判断条件，直接过滤掉不满足条件的记录，从而减少回表次数，提高查询效率。比如，有一个名为user的表，其中包含id, username,zipcode和birthdate 4个字段，创建了联合索引(zipcode, birthdate)。

```sql
CREATE TABLE `user` (
  `id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `zipcode` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `birthdate` date NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_username_birthdate` (`zipcode`,`birthdate`) ) ENGINE=InnoDB AUTO_INCREMENT=1001 DEFAULT CHARSET=utf8mb4;

/* 查询 zipcode 为 431200 且生日在 3 月的用户 */
/* birthdate 字段使用函数索引失效 */
SELECT * FROM user WHERE zipcode = '431200' AND MONTH(birthdate) = 3;
```

当在没有索引下推功能的情况下进行查询时，即使zipcode字段利用索引可以帮助我们快速定位到zipcode = '431200'的用户，但我们仍然需要对每一个找到的用户进行回表操作，获取完整的用户数据，再去判断。而有了索引下推之后进行查询时，存储引擎会在使用zipcode字段索引查找zipcode = '431200'的用户时，同时判断MONTH(birthdate) = 3。这样，只有同时满足条件的记录才会被返回，减少了回表次数。

### 索引失效

### 优化

除了使用用explain命令查看执行计划，然后针对性的修改sql语句这种常规优化手段之外，下面再介绍几种常见优化索引的方法：

前缀索引优化；
覆盖索引优化；
主键索引最好是自增的；
防止索引失效；
