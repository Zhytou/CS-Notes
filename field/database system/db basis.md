# DB

- [DB](#db)
  - [Introduction](#introduction)
    - [History](#history)
    - [Notions](#notions)
    - [Classification](#classification)
    - [Architecture](#architecture)
  - [RDBMS](#rdbms)
    - [E-R Model](#e-r-model)
    - [Normalization](#normalization)
    - [Sort](#sort)
    - [Join](#join)
  - [MySQL Basis](#mysql-basis)
    - [Data Type](#data-type)
    - [SQL](#sql)
    - [MySQL Function](#mysql-function)
    - [MySQL Architecture](#mysql-architecture)
    - [Workflow](#workflow)
    - [Storage Engine](#storage-engine)

## Introduction

### History

个人认为想要更好的理解数据库这些年的发展变化，就要回顾历史从而体会所有变化背后的驱动因素。推荐阅读TiDB社区撰写的这篇[文章](https://tidb.net/blog/01f8c3ee)，可以很好的回顾数据库的发展历史。

![数据库发展历史](https://tidb-blog.oss-cn-beijing.aliyuncs.com/media/%E6%95%B0%E6%8D%AE%E5%BA%93%E5%8F%91%E5%B1%95%E5%8F%B2-1672663444988.jpeg)

**1996 关系数据库大火**：

MySQL和PostgreSQL诞生进一步促进了关系型数据库的发展，并奠定了关系型数据库的霸主地位。但与此同时，人们也提出了OLAP的概念，并指出关系型数据库并不适合这项工作，需要开发一款新数据库。

**2000年，数据爆炸，NoSQL和分布式数据库出现**：

由于 web2.0 时代的到来，互联网用户和数据量呈几何式上升，尽管技术人员已经使用了缓存、读写分离（主从复制）、分库分表和单机数据库集群等多种手段，但传统的关系型数据库还是很难应付大型网站的超大数据量和高并发。

面对这些问题，一部分技术人员开始探索多种数据模型和存储介质的数据库，NoSQL数据库也应运而生。它扩展了诸多数据模型，在不同场景下使用不同的数据模型来进行处理。其代表成果是2009推出的文档数据库Mongdb、2010年推出的键值数据库Redis和2010年推出的图数据库Neo4j。这类NoSQL数据库极大地扩展了人们存储、使用数据的方式。

另一部分技术人员则是寄希望于用多台机器形成集群来存储、处理数据进而解决超大量数据和高并发。大量分布式数据处理系统诞生，其中又以Hadoop的子项目HBase最为出名。值得一提的是，这时期谷歌发布的三篇论文，分别是Google File System、Google Big table和Google MapReduce，为这一技术方向提供了很多理论基础。后人也称这三篇论文为奠定了分布式数据系统基础的三驾马车。

**2015年，NewSQL时代**：

NoSQL数据库虽然解决了高并发读写、多结构化数据存储等问题，但其设计思路是牺牲事务处理、一致性以及牺牲SQL换来的。而SQL、事务的重要性让人们开始反思怎么样才能在解决前述问题的基础上保留SQL和事务的能力。Google 于2012年发布了Spanner的论文，这篇文章创新性地提出了TrueTime的概念，它在第一代 NoSQL 系统的基础之上引入了 SQL 和分布式事务，保证了强一致性。（也正是这篇论文，宣布了NoSQL时代的结束，数据库发展来到了NewSQL的阶段）

这篇文章在工业界和学术界都有着巨大的反响，截止2022年4月，对其开源实现最好的产品是于2015年诞生的CockroachDB和TiDB。和Spanner及它的追随者不同的是，Amazon在面对这一问题时，选择了完全不同的路径，Amazon 发布的Aurora 是一个存储计算分离的系统，运行在公有云之上，它的设计思想很巧妙，它把存储与计算分离使得可以非常简单得实现存储能力的可扩展。并于2017年在SIGMOD上发表了《Amazon Aurora: Design Considerations for High Throughput Cloud-NativeRelational Databases》这篇论文，披露了Aurora的一些技术实现细节。

### Notions

**DBMS vs OS File System**：

数据库管理系统(Database Management System, DBMS)和操作系统文件系统(Operating System File System, OSFS)都是用于存储和管理数据的系统，但它们在设计目标、数据组织和访问方式上存在显著差异。相比文件系统，数据库有很多优势包括：

- 支持事务，进而保证数据一致性；
- 可以非常方便的实现数据冗余和复制，提高可用性和容错性；
- 使用SQL查询语言，提供了强大的查询和分析能力；
- 支持多种数据结构，方便数据组织和查询。

**分布式数据库 vs 集中式数据库集群**：

分布式数据库通常由多个节点组成，每个节点都可以独立地处理数据和请求，数据可以分布在多个节点上，从而实现数据的水平分割和扩展。分布式数据库通常需要考虑数据分片、数据复制、数据一致性、故障恢复、负载均衡等问题，并且需要使用一些额外的组件或工具来管理分片，比如 MongoDB、Cassandra、HBase 等。

单机数据库集群则是将多个数据库实例部署在不同的物理节点上，通过一些技术来实现读写分离、负载均衡、故障转移等功能，从而提高系统的性能和可靠性。单机数据库集群通常需要考虑数据同步、事务处理、数据一致性等问题，并且需要使用一些额外的组件或工具来管理集群，比如 MySQL Cluster、PostgreSQL Cluster、Oracle RAC 等。

相比而言，分布式数据库通常可以实现更高的可扩展性和性能，因为数据可以在多个节点上并行处理，而且可以动态地调整节点数量和分片策略。但是，分布式数据库也可能存在数据一致性、跨分片查询、事务处理等问题，需要根据具体的应用场景和需求来选择和使用。

单机数据库集群则通常更容易管理和部署，因为所有的实例都运行在同一台机器或者在同一组机器上，而且可以使用一些标准的数据库技术和工具来管理和维护集群。但是，单机数据库集群也可能存在单点故障、性能瓶颈、资源浪费等问题，需要根据具体的应用需求和负载特点来选择和使用。

**SQL vs NoSQL vs NewSQL**：

SQL数据库：SQL数据库，也称为关系型数据库管理系统（RDBMS），是一种用于存储和操作历史数据的传统数据库类型。在这种系统中，信息按照结构化的方式使用表格或关系进行组织。

NoSQL数据库：NoSQL数据库也称为“非关系型数据库”，它使用键值对、文档、图形数据库或宽列存储等多种数据模型，没有固定的数据模式。NoSQL数据库可以水平扩展，可以在多个服务器上进行扩展，而传统的SQL数据库则通常只能进行纵向扩展，即增加单个服务器的资源。

NewSQL数据库：NewSQL数据库是一种结合了传统SQL数据库的关系数据模型和NoSQL数据库的可扩展性和性能的新型数据库类型。它提供了两种方法的优点，可以实现高性能和水平扩展的关系型数据库。

**BASE vs ACID**：

ACID是原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持久性（Durability）的缩写。这是一组保证数据库事务被可靠地处理的属性。

BASE是基本可用（Basically Available）、软状态（Soft state）和最终一致性（Eventual consistency）的缩写。这是一个数据一致性模型，它优先考虑可用性和分区容错性，而不是严格的一致性。

**OLAP vs OLTP vs HTAP**：

OLAP（Online Analytical Processing）用于支持在线分析处理应用程序，如商业智能、数据挖掘、数据分析等。OLAP通常需要处理大量的数据，而且需要进行复杂的查询和分析操作。OLAP系统通常使用列式存储和向量化查询技术来提高查询性能。

OLTP（Online Transaction Processing）用于支持在线事务处理应用程序，如电子商务、金融交易、库存管理等。OLTP系统需要处理大量的事务，需要保证数据的一致性和可靠性，并且需要支持高并发的访问。

HTAP（Hybrid Transactional and Analytical Processing）是一种将OLTP和OLAP结合起来的数据处理方式。在HTAP系统中，数据可以同时用于事务处理和分析处理。HTAP系统通常使用混合存储引擎，可以在行式存储和列式存储之间灵活切换，以支持不同的应用场景。

**Database vs Data Warehouse vs Data Lake**：

数据库存储应用程序所需的当前数据。

数据仓库存储来自一个或多个系统的当前和历史数据，并采用预定义和固定的模式，使业务分析师和数据科学家能够轻松地分析数据。

数据湖一般以数据的形式存储来自一个或多个系统的当前和历史数据，使业务分析师和数据科学家能够轻松地分析数据。

**Spark vs Flink vs Hadoop**：

Hadoop 是一个由 Apache 开发的大数据处理框架，它包括两个核心组件：分布式文件系统 HDFS 和分布式计算框架 MapReduce。Hadoop 基于分布式文件系统和分布式计算模型，可以处理大规模的结构化和半结构化数据，支持批量处理和离线处理等场景。Hadoop 使用 HDFS 存储数据，并使用 MapReduce 进行数据处理和计算。

Spark 是一个由 Apache 开发的大数据处理框架，它基于内存计算和分布式计算模型，可以处理大规模的结构化和半结构化数据，支持批量处理、流处理和机器学习等场景。Spark 支持使用多种编程语言进行开发，例如 Scala、Java、Python 和 R 等，提供了丰富的 API 和库，以支持各种数据处理和分析操作。

Flink 是一个由 Apache 开发的大数据处理框架，它基于流处理和批处理的统一计算模型，可以处理大规模的结构化和半结构化数据，支持实时处理、流式处理和批量处理等场景。Flink 支持使用多种编程语言进行开发，例如 Java、Scala 和 Python 等，提供了丰富的 API 和库，以支持各种数据处理和分析操作。

### Classification

![db_classification](https://zhytou.github.io/post/2023-6-11/db_classification.png)

我们可以按照数据库的存储模型将其分类，常见类型如下：

**关系型数据库**：关系型数据库是使用表格来存储数据的，其中每个表格包含多个行和列，每列代表一个属性。关系型数据库使用 SQL（Structured Query Language）来进行数据查询和操作，支持事务处理和 ACID 原则。常见的关系型数据库有 MySQL、Oracle、SQL Server 等。

**键值数据库**：键值数据库是一种类型简单的 NoSQL 数据库，数据以键值对的形式存储，其中键是唯一的标识符，值可以是任何类型的数据。键值数据库通常用于存储非结构化的数据，例如缓存、会话管理等场景。常见的键值数据库有 Redis、Memcached 等。

**文档数据库**：文档数据库是一种 NoSQL 数据库，数据以文档（Document）的形式存储，文档可以是 JSON、XML 等格式。文档数据库通常用于存储半结构化和非结构化的数据，例如 Web 应用程序、内容管理系统等场景。常见的文档数据库有 MongoDB、Couchbase 等。

**时序数据库**：时序数据库是一种专门用于存储和查询时间序列数据的数据库，例如传感器数据、网络日志等。时序数据库的特点是高效地存储和查询大量的时间序列数据。常见的时序数据库有 InfluxDB、OpenTSDB 等。

**图数据库**：图数据库是一种 NoSQL 数据库，数据以图的形式存储，其中节点表示实体，边表示实体之间的关系。图数据库通常用于存储和查询复杂的实体关系，例如社交网络、知识图谱等场景。常见的图数据库有 Neo4j、OrientDB 等。

另一方面，我们还可以将其按照功能分类，比如OLAP、OLTP和HTAP三种。一般来说，数据库的功能和特性能够和其存储模型对应，例如：关系型数据库往往是OLTP数据库。但由于一些复杂的应用场景，人们又推出了一些多模型数据库，例如：将SQL和NoSQL结合的NewSQL数据库，就能在保持数据强一致性的同时，通过水平扩展获得一定的高可用性，比如国内大火的TiDB。除此之外，时序数据库常常用于和消息存储队列+流处理框架的解决方案（也有厂商将其包装成数据库，称为流数据库，例如：Confluent的ksqldb）对比；流数据处理和OLAP的对比，可以阅读这篇[文章](https://zhuanlan.zhihu.com/p/600701331)。

### Architecture

常见的数据库系统架构包括：

- 单机：计算节点和存储节点一般在同一台机器上，通常存储节点是本地硬盘，如单机版的MySQL、Oracle。单机模式没有高可用保障，常用于临时开发测试或者个人学习场景，生产环境不建议使用。
- 主从：一个主数据库负责所有写操作和一部分读操作，而其他一个或多个从数据库则通过复制主数据库的数据来提供读操作。
- 主备：一个主数据库负责处理所有写操作和一部分读操作，而集群中一个备用数据库则通过复制数据库（不提供读操作）来保证主数据库挂之后的可用性。
- 主主：多个数据库都可以处理写操作，而其他数据库则通过复制彼此的数据来提供读操作。
- 分片：分片是指将数据库中的数据分为多个片段，每个片段可以单独存储在不同的服务器上。这种技术被广泛应用于分布式数据库中，以提高系统的可扩展性和性能。

## RDBMS

关系型数据库(Relational Database)是建立在关系模型基础上的数据库。它以表格的形式存储数据，并通过定义数据之间的关系来组织数据。关系模型的核心概念包括：

**Schema**：模式是表的结构，包括表的名称、属性及其数据类型，以及可能的约束。一个数据库可以包含多个模式。

**Attribute**：属性是表中的列，它定义了数据的某一特性或特征。每个属性都有一个名称和数据类型。

**Tuple**：元组是表中的行，它是一组有序的属性值，代表了关系中的一条记录。

**Key**：

- 超键(Super Key)：能够唯一标识表中元组的属性集。超键可以包含一个或多个属性，只要它们的组合能够唯一标识表中的每一行。
- 候选键(Candidate Key)：超键中不包含多余属性的最小集合，即没有冗余属性的超键。
- 主键(Primary Key)：数据库管理员指定的候选键，作为表的唯一标识，不允许有NULL值。
- 外键(Foreign Key)：引用另一个表的主键，用于建立表之间的关联，可以有NULL值。不过，实际开发中很少在数据库层使用外键来标识表之间的关系，更多的是在应用层解决这种关系。这是因为外键限制会增加数据库维护负担，尤其是在高并发场景下会导致数据库更新风暴。此外，在分库分表的情况下外键也是无法生效的。

### E-R Model

E-R模型是一种概念数据模型，用于描述现实世界中的实体、实体之间的关系以及实体的属性。它由以下元素组成：

- 实体(Entity)：现实世界中的对象，如人、地点或事物。
- 属性(Attribute)：描述实体的特性。
- 关系(Relationship)：连接两个或多个实体的联系。
- 实体集(Entity Set)：具有相同属性的实体集合。
- 联系集(Relationship Set)：具有相同关系的实体对或实体三元组的集合。

### Normalization

范式(Normalization)是指将关系型数据库设计成符合特定范式的过程。通过规范化，可以消除冗余数据，提高数据的一致性和可维护性。常见的范式包括：

- 第一范式(1NF)：确保每个表中的每个列都是不可分割的基本数据项，没有多值属性。
- 第二范式(2NF)：在1NF的基础上，消除部分函数依赖，确保非主属性完全依赖于主键。
- 第三范式(3NF)：在2NF的基础上，消除传递依赖，确保非主属性只依赖于主键。
- BC范式(Boyce-Codd范式)：在3NF的基础上，进一步消除对任何非候选键的依赖。

**Functional Dependency**：

其中，上述范式定义中涉及到函数依赖、部分函数依赖以及函数依赖传递的概念。

- 若在一张表中，在属性（或属性组）X 的值确定的情况下，必定能确定属性 Y 的值，那么就可以说 Y 函数依赖于 X，写作 X → Y。
- 如果 X→Y，并且存在 X 的一个真子集 X0，使得 X0→Y，则称 Y 对 X 部分函数依赖。比如学生基本信息表 R 中（学号，身份证号，姓名）当然学号属性取值是唯一的，在 R 关系中，（学号，身份证号）->（姓名），（学号）->（姓名），（身份证号）->（姓名）；所以姓名部分函数依赖于（学号，身份证号）；
- 在一个关系中，若某个非主属性数据项依赖于全部关键字称之为完全函数依赖。比如学生基本信息表 R（学号，班级，姓名）假设不同的班级学号有相同的，班级内学号不能相同，在 R 关系中，（学号，班级）->（姓名），但是（学号）->(姓名)不成立，（班级）->(姓名)不成立，所以姓名完全函数依赖与（学号，班级）；
- 在关系模式 R(U)中，设 X，Y，Z 是 U 的不同的属性子集，如果 X 确定 Y、Y 确定 Z，且有 X 不包含 Y，Y 不确定 X，（X∪Y）∩Z=空集合，则称 Z 传递函数依赖(transitive functional dependency) 于 X。传递函数依赖会导致数据冗余和异常。传递函数依赖的 Y 和 Z 子集往往同属于某一个事物，因此可将其合并放到一个表中。比如在关系 R(学号 , 姓名, 系名，系主任)中，学号 → 系名，系名 → 系主任，所以存在非主属性系主任对于学号的传递函数依赖。

**Closure**：

闭包(Closure)是指给定一组函数依赖后，通过应用这些依赖可以推导出的所有其他函数依赖的集合。换句话说，如果从一组函数依赖出发，通过推理可以得到的函数依赖的全部集合，就称为这组函数依赖的闭包。

### Sort

在数据库中，排序是一个非常常见的需求。对于那些足够小能够容纳到内存中的表来说，可以采用很多种方法，比如快排、堆排序等等。然后更常见的情况是表远大于可用内存大小。此时，就需要使用外排序进行处理。

**External Sorting**：

外排序(External Sorting)是指能够处理极大量数据的排序算法。通常来说，外排序处理的数据不能一次装入内存，只能放在读写较慢的外存储器（通常是硬盘）上。外排序通常采用的是一种“排序-归并”的策略。在排序阶段，先读入能放在内存中的数据量，将其排序输出到一个临时文件，依此进行，将待排序数据组织为多个有序的临时文件。而后在归并阶段将这些临时文件组合为一个大的有序文件，也即排序结果。

### Join

Join是数据库中重要的操作之一。其语法如下`... FROM table1 JOIN table ON condition`，用于将两张表按某种条件连接。比如，查询表A和B中id相同的数据。

```sql
SELECT * FROM A INNER JOIN B ON A.name = B.name;
```

**Classification**：

根据连接时行为不同，可将Join分为：

- Inner Join：只保留两个表中都有的数据。
- Right(Left) Outter Join：无论右(左)表中数据是否有满足条件的匹配，都保留。对于没有匹配的行，补NULL即可。
- Cross Join：得到两个表的笛卡尔积。

**Join Algorithms**：

常见用于实现Join的算法如下：

- Nested Loop Join
- Blocked Nested Loop Join
- Index Nested Loop Join
- Merge Join
- Hash Join

其中，嵌套循环连接适用于被连接的数据子集较小的情况。它通过扫描一个表，每读到一条记录，就根据索引去另一个表里面查找。如果被驱动表的连接字段有索引，且驱动表返回的记录不多，嵌套循环连接是一个较好的选择。

而哈希连接适用于大数据集连接的情况。它使用较小的数据表在内存中建立哈希表，然后扫描另一个表，每读到一条记录就来探测哈希表一次，找出与哈希表匹配的行。如果小表可以全部放入内存中，哈希连接的成本接近全表扫描两个表的成本之和。

最后，排序合并连接通常在没有索引的情况下使用。它的操作分为三步：对连接的每个表进行全表扫描，对扫描结果进行排序，然后进行合并连接。排序合并连接的性能开销主要在前两步。如果行源已经被排序过，执行排序合并连接时不需要再排序，这时排序合并连接的性能会优于哈希连接。

## MySQL Basis

MySQL是一个关系型数据库管理系统，它实现了SQL标准用于管理和操作数据库。

### Data Type

MySQL支持多种数据类型，大致可以分为数值、日期/时间和字符串三类。

**数值类型**：

MySQL支持所有标准SQL数字数据类型。这些类型包括：

- 精确数字类型（整型和定点数）：INTEGER、SMALLINT、MEDIUMINT、BIGINT和DECIMAL；
- 近似数字类型（浮点数）：REAL、FLOAT和DOUBLE PERCISION；
- 位置类型：BIT。

为了书写方便，MySQL给数值类型提供了一些同义词。比如，INT就是INTEGER的同义词；DEC和FIXED就是DECIMAL的同义词；DOUBLE则是DOUBLE PERCISION的同义词。

对于浮点和定点数据类型，可以提供参数M和N，用于指定存储的总位数和小数点后位数，比如：`DECIMAL(10, 2)`表示可以存储总位数为10，小数点后位数为2的小数；而对于整数来说，则只能提供一个参数M用于指定显示宽度，比如：`INT(2)`表示至少可以看到2个数字，若没满两位则前补零。此外，对于定点数来说，它占字节数受M和N影响，一般是min(M, N)+2字节；而对于浮点数和整数，其大小则是固定的，比如：FLOAT和INT都是4字节，DOUBLE则是8字节。值得一提的是，可以在整型后添加UNSIGNED关键字指定存储无符号型数，比如：`INT UNSIGNED`。

关于数值类型更多的语法介绍，可以参考MySQL官方手册中13.1.1小节[Numeric Data Type Syntax](https://dev.mysql.com/doc/refman/8.4/en/numeric-type-syntax.html)。

**日期/时间类型**：

在MySQL中，用于表示时间值的日期和时间数据类型为DATE、TIME、DATETIME、TIMESTAMP和YEAR。每种时间类型都有一系列有效值，以及一个用于表示无法表示的无效零值。

其中，DATE用于存储日期，格式为'YYYY-MM-DD'，范围是'1000-01-01'到'9999-12-31'；TIME用于存储时间，格式为'HH:MM:SS'，范围是'-838:59:59'到'838:59:59'；YEAR用于存储年份，可以是2位或4位格式，范围是'1901'到'2155'。

至于DATETIME和TIMESTAMP则均用于存储日期和时间，且格式均为'YYYY-MM-DD HH:MM:SS'，并且都能够自动初始化或更新。不过，二者能够表示的范围不同，前者范围是'1000-01-01 00:00:00'到'9999-12-31 23:59:59'；后者但范围更小，其存储年份不能早于1970或晚于2037。此外，DATETIME存储本地时间，占8个字节；而TIMESTAMP使用UTC时间因而可以快速切换时区，占4-8个字节。

关于TIMESTAMP和DATETIME自动初始化或更新的详细介绍，可以参考MySQL手册13.25小节[Automatic Initialization and Updating for TIMESTAMP and DATETIME](https://dev.mysql.com/doc/refman/8.4/en/timestamp-initialization.html)。比如：

```sql
/* 当新纪录插入时，ts列会自动设置为当前时间记录。当记录被更新时，ts列也会自动更新为当前时间。*/
CREATE TABLE t1 (
ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
);
```

**字符类型**：

至于字符类型，它们也可以按存储对象分成字符串、文本、二进制和字符串集合类型。其中，CHAR和VARCHAR都用于存储字符串。只不过前者的特点是有固定长度限制，在存储时会填充空格保持长度；而后者则是可变长度，可以节省空间。此外，二者在使用时都需要传入一个字符长度参数。比如，`CHAR(M)`和`VARCHAR(M)`。前者表示可存储最大M个字符的固定长度字符串，M范围0-255；而后者表示可存储最大M个字符的可变长度字符串，M范围0-65535。

此外，大段的文本文件如评论、文章等可用文本类型存储。它分为TINYTEXT、TEXT、MEDIUMTEXT和LONGTEXT四种，分别可以存储的最大长度为255个字符、65535个字符、16777215个字符和4294967295个字符

二进制类型数据如图片、视频、文件等可用TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB存储。它们分别可以存储的最大长度为255字节、65535字节、16777215字节和4294967295字节。

最后，MySQL还提供了ENUM和SET类型来存储字符串集合数据。两者都只能用于存储不同的元素，不过SET列可用存储集合中一个或多个值，而ENUM只能存储集合中一个值。此外，ENUM可用用元素序号指定值。比如：

```sql
CREATE TABLE t1 (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  keywords SET('tech', 'finance', 'health')
);

/* 可用插入('tech', 'finance', 'health')的组合('tech, finance') */
INSERT INTO t1 (keywords) VALUES ('tech, finance');

CREATE TABLE t2 (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `sex` enum('男','女') DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;

/* 可用序号指定插入的值，序号从1开始 */
INSERT INTO t2(sex) VALUES('男');  // sex = 男
INSERT INTO t2(sex) VALUES(2); // sex = 女
```

### SQL

SQL(Structured Query Language)是一种用于处理关系数据库的标准语言。它提供了查询、更新、插入和删除数据的功能。关于SQL的详细介绍请查看这篇[总结](https://github.com/Zhytou/CS-Notes/blob/main/field/database%20system/sql.md)。

### MySQL Function

为了进一步简化SQL编写复杂度，MySQL还提供了大量的函数。它们扩展了SQL的功能，提供了大量丰富的操作。

**COUNT()**：

此外，关于COUNT()函数的不同使用手段的性能差异，其结论如下：

![COUNT()函数结论](https://cdn.xiaolincoding.com//mysql/other/af711033aa3423330d3a4bc6baeb9532.png)

更详细的介绍可以参考这篇小林Coding的文章[count(*) 和 count(1)有什么区别？哪个性能最好？](https://xiaolincoding.com/mysql/index/count.html)。

**CONCAT()**：

```sql
SELECT CONCAT(Address, " ", PostalCode, " ", City) AS Address
FROM Customers;
```

### MySQL Architecture

MySQL通常由三层架构组成：客户端(Client Layer)、服务端(Server Layer)和存储引擎(Storage Layer)。

![MySQL三层架构](https://dev.mysql.com/doc/refman/8.0/en/images/mysql-architecture.png)

### Workflow

MySQL查询的执行流程可以分为以下几个步骤，从客户端开始：

- 客户端（如MySQL客户端、应用程序等）连接到MySQL服务器。接着，通过它发送查询语句（如SELECT语句）到服务器。
- MySQL服务器的协议层接收到查询请求，对请求进行解析和编码，然后传递给SQL解析器。
- 解析器将查询语句解析成抽象语法树（AST），检查语法是否正确。
- 优化器分析查询语句的执行计划，考虑不同的执行路径，如使用哪些索引、JOIN顺序、子查询优化等。它会选择最佳执行计划，并生成执行计划树。
- 执行引擎开始执行优化后的计划。
  - 对于JOIN操作，执行引擎会按照优化器选择的顺序处理JOIN，通常是从最小的表开始，逐步与更大的表进行JOIN。
  - 对于WHERE条件，执行引擎会在JOIN过程中尽可能早地应用条件过滤，以减少需要处理的数据量。这被称为“谓词下推”（Predicate Pushdown）。
- 执行引擎与存储引擎交互，从存储引擎中读取数据。
  - 对于SELECT语句，存储引擎会根据执行计划返回满足条件的行。
  - 存储引擎可能使用索引来加速数据的查找，但是否先返回所有结果再过滤，取决于查询的具体情况。通常，存储引擎会根据WHERE条件逐步返回匹配的行，而不是一次性返回所有数据。
- 执行引擎继续处理JOIN、聚合、排序等操作，直到完成整个查询。如果有GROUP BY或ORDER BY，这些操作通常在服务器层完成，而不是在存储引擎层。
- 执行引擎将最终结果集返回给优化器。优化器将结果集编码并发送回客户端。客户端接收到结果，显示给用户或进一步处理。

### Storage Engine

**InnoDB vs MyISAM**：

[InnoDB vs MyISAM](https://www.runoob.com/w3cnote/mysql-different-nnodb-myisam.html)

**InnoDB Architecture**：

![InnoDB结构](https://dev.mysql.com/doc/refman/8.4/en/images/innodb-architecture-8-0.png)

**Buffer Pool**：

为了提高存储效率并减少磁盘I/O次数，InnoDB存储引擎设计了缓存。其中，最重要的缓存就是Buffer Pool。它位于内存，默认大小为128MB，并将其分为了多个页。类似MySQL的磁盘存储单位，缓存页大小也是16KB。不过MySQL还为每个缓存页也开辟额外的一些空间，用来描述对应的缓存页的一些信息，例如：数据页所属的表空间，数据页号等。

和所有缓存一致，Buffer Pool也实现了其缓存更新策略。它使用Free List管理空闲缓存页，表示Buffer Pool中未分配的页；Flush List管理脏缓存页，后台线程从其中取出页号执行写入磁盘流程。当Buffer Pool中无空闲页时，它使用LRU List移除最不常访问的页。
