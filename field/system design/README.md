# [系统设计](https://github.com/donnemartin/system-design-primer)

- [系统设计](#系统设计)
  - [系统分类](#系统分类)
    - [扩展](#扩展)
  - [系统的设计原则](#系统的设计原则)
    - [性能与可扩展性](#性能与可扩展性)
    - [延迟与吞吐量](#延迟与吞吐量)
    - [可用性与一致性——CAP理论](#可用性与一致性cap理论)
    - [一致性模式](#一致性模式)
    - [可用模式](#可用模式)
      - [故障切换](#故障切换)
      - [复制](#复制)
  - [常见系统组件](#常见系统组件)
  - [参考](#参考)

## 系统分类

- **集中式系统 Centralized System**：Centralized systems are systems that use client/server architecture where one or more client nodes are directly connected to a central server.
- **去中心化系统 Decentralized System**：In decentralized systems, every node makes its own decision. The final behavior of the system is the aggregate of the decisions of the individual nodes. Note that there is no single entity that receives and responds to the request. For example, blockchain is a decentralized system.
- **分布式系统 Distributed System**：In decentralized systems, every node makes its own decision. The final behaviour of the system is the aggregate of the decisions of the individual nodes. Note that there is no single entity that receives and responds to the request.

### 扩展

> [Understanding Database Sharding](https://www.digitalocean.com/community/tutorials/understanding-database-sharding#benefits-of-sharding)

**水平扩展 Scaling Out**：Horizontal scaling also konw as scaling out is the practice of adding more machines to an existing stack in order to spread out the load and allow for more traffic and faster processing.

**垂直扩展 Scaling Up**：Vertical scaling, otherwise known as scaling up, which involves upgrading the hardware of an existing server, usually by adding more RAM or CPU.

## 系统的设计原则

### 性能与可扩展性

如果服务**性能**的增长与资源的增加是成比例的，服务就是可扩展的。通常，提高性能意味着服务于更多的工作单元，另一方面，当数据集增长时，同样也可以处理更大的工作单位。从另一个角度来看，如果系统有**性能**问题，对于单个用户来说是缓慢的；而如果系统有**可扩展性**问题，单个用户较快但在高负载下会变慢。

### 延迟与吞吐量

**延迟**是执行操作或运算结果所花费的时间，而**吞吐量**是单位时间内（执行）此类操作或运算的数量。通常，系统设计应该以**可接受级延迟**下**最大化吞吐量**为目标。

### 可用性与一致性——CAP理论

在一个分布式计算系统中，只能同时满足下列的两点:

- **一致性** ─ 每次访问都能获得最新数据但可能会收到错误响应
- **可用性** ─ 每次访问都能收到非错响应，但不保证获取到最新数据
- **分区容错性** ─ 在任意分区网络故障的情况下系统仍能继续运行

因为分布式系统依赖网络通信，但网络并不可靠，所以你应要支持分区容错性 P ，并需要在软件可用性 A 和一致性 C 间做出取舍==。

**CP 一致性和分区容错性**：

- 等待分区节点的响应可能会导致延时错误。如果你的业务需求需要原子读写，CP 是一个不错的选择。

**AP 可用性与分区容错性**：

- 响应节点上可用数据的最近版本可能并不是最新的。当分区解析完后，写入（操作）可能需要一些时间来传播。
- 如果业务需求允许最终一致性，或当有外部故障时要求系统继续运行，AP 是一个不错的选择。

![CAP理论](../img/cap_theoram.png)

### 一致性模式

**弱一致性**：在弱一致性模式下，读取数据可能看到刚写入的数据，也可能看不到。可见，这种模式会尽力优化让系统能访问最新数据。

**最终一致性**：在最终一致性模式下，写入数据后，访问最终能看到写入数据（通常在数毫秒内）。即，写入数据被异步复制。DNS和Email等系统均使用此种方式。可见，最终一致性在高可用性系统中效果不错。

**强一致性**：在强一致性，写入数据后，访问立即可见。即，数据被同步复制。文件系统和关系型数据库（RDBMS）中常使用这种模式。可见，强一致性在需要记录的系统中运作良好。

### 可用模式

- 两种支持高可用性的模式：`故障切换(fail-over)`和`复制(replication)`

#### 故障切换

**工作到备用切换**：

- 关于工作到备用的故障切换流程是，工作服务器发送周期信号给待机中的备用服务器。如果周期信号中断，备用服务器切换成工作服务器的 IP 地址并恢复服务。
- 宕机时间取决于备用服务器处于“热”待机状态还是需要从“冷”待机状态进行启动。只有工作服务器处理流量。
- 工作到备用的故障切换也被称为主从切换。

**双工作切换**：

- 在双工作切换中，双方都在管控流量，在它们之间分散负载。
- 如果是外网服务器，DNS 将需要对两方都了解。如果是内网服务器，应用程序逻辑将需要对两方都了解。
- 双工作切换也可以称为主主切换。

**缺陷**：

- 故障切换需要添加额外硬件并增加复杂性。
- 如果新写入数据在能被复制到备用系统之前，工作系统出现了故障，则有可能会丢失数据。

#### 复制

## 常见系统组件

- 域名系统 DNS
- 内容分发系统 CDN
- 负载均衡器 Load Balance
- 反向代理  Reverse Proxy
- 应用
  - 微服务
  - 服务发现
- 数据库
  - 关系数据库 RDBMS
  - 特征 ACID
  - 复制
  - 联合
  - 优化
  - 非关系数据库 NoSQL
    - 特征 BASE
    - 分类

- 缓存
- 异步
  - 消息队列
  - 任务队列
- 通信
  - TCP
  - UDP
  - RPC
- 安全

## 参考

- [comparison-centrilized-decentrlized-and-distributed-systems](https://www.geeksforgeeks.org/comparison-centralized-decentralized-and-distributed-systems/)
- [system-design-primer](https://github.com/donnemartin/system-design-primer)
- [scalability-availability-stability-patterns](https://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
- [凤凰架构](http://icyfenix.cn/)
