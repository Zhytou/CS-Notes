# [系统设计](https://github.com/donnemartin/system-design-primer)

## 系统

### 分类

#### 集中式系统 Centralized System

**描述**

+ Centralized systems are systems that use client/server architecture where one or more client nodes are directly connected to a central server. 

**实例**

+ 维基百科 Wikipedia

#### 去中心化系统 Decentralized System

**描述**

+ In decentralized systems, every node makes its own decision. The final behavior of the system is the aggregate of the decisions of the individual nodes. Note that there is no single entity that receives and responds to the request. 

**实例**

+ 区块链 Blockchain

#### 分布式系统 Distributed System

**描述**

+ In decentralized systems, every node makes its own decision. The final behaviour of the system is the aggregate of the decisions of the individual nodes. Note that there is no single entity that receives and responds to the request. 

**实例**

### 扩展

**水平扩展 Scaling Out**

+ [Understanding Database Sharding](https://www.digitalocean.com/community/tutorials/understanding-database-sharding#benefits-of-sharding) 

  > Horizontal scaling also konw as scaling out is the practice of adding more machines to an existing stack in order to spread out the load and allow for more traffic and faster processing.

**垂直扩展 Scaling Up**

+ [Understanding Database Sharding](https://www.digitalocean.com/community/tutorials/understanding-database-sharding#benefits-of-sharding)

  > Vertical scaling, otherwise known as scaling up, which involves upgrading the hardware of an existing server, usually by adding more RAM or CPU.

## 系统的设计原则

+ 我们需要对一般性原则有一个基本的认识，知道它们是什么，怎样使用以及权衡利弊。

### 性能与可扩展性

+ 如果服务**性能**的增长与资源的增加是成比例的，服务就是可扩展的。通常，提高性能意味着服务于更多的工作单元，另一方面，当数据集增长时，同样也可以处理更大的工作单位。
+ 另一个角度来看待性能与可扩展性:
  - 如果你的系统有**性能**问题，对于单个用户来说是缓慢的。
  - 如果你的系统有**可扩展性**问题，单个用户较快但在高负载下会变慢。

### 延迟与吞吐量

+ **延迟**是执行操作或运算结果所花费的时间。
+ **吞吐量**是单位时间内（执行）此类操作或运算的数量。
+ 通常，你应该以**可接受级延迟**下**最大化吞吐量**为目标。

### 可用性与一致性

#### CAP 理论

+ 在一个分布式计算系统中，只能同时满足下列的两点:
  - **一致性** ─ 每次访问都能获得最新数据但可能会收到错误响应
  - **可用性** ─ 每次访问都能收到非错响应，但不保证获取到最新数据
  - **分区容错性** ─ 在任意分区网络故障的情况下系统仍能继续运行
+ ==因为分布式系统依赖网络通信，但网络并不可靠，所以你应要支持分区容错性 P ，并需要在软件可用性 A 和一致性 C 间做出取舍==。

**CP 一致性和分区容错性**

+ 等待分区节点的响应可能会导致延时错误。如果你的业务需求需要原子读写，CP 是一个不错的选择。

**AP 可用性与分区容错性**

+ 响应节点上可用数据的最近版本可能并不是最新的。当分区解析完后，写入（操作）可能需要一些时间来传播。
+ 如果业务需求允许最终一致性，或当有外部故障时要求系统继续运行，AP 是一个不错的选择。

![](./img/cap_theoram.png)

### 一致性模式

+ 有同一份数据的多份副本，我们面临着怎样同步它们的选择，以便让客户端有一致的显示数据。

#### 弱一致性

+ 在写入之后，访问可能看到，也可能看不到（写入数据）。尽力优化之让其能访问最新数据。
+ 这种方式可以 memcached 等系统中看到。弱一致性在 VoIP，视频聊天和实时多人游戏等真实用例中表现不错。打个比方，如果你在通话中丢失信号几秒钟时间，当重新连接时你是听不到这几秒钟所说的话的。

#### 最终一致性

+ 在写入后，访问最终能看到写入数据（通常在数毫秒内）。数据被异步复制。
+ DNS 和 email 等系统使用的是此种方式。最终一致性在高可用性系统中效果不错。

#### 强一致性

+ 在写入后，访问立即可见。数据被同步复制。
+ 文件系统和关系型数据库（RDBMS）中使用的是此种方式。强一致性在需要记录的系统中运作良好。

### 可用模式

+ 两种支持高可用性的模式：`故障切换(fail-over)`和`复制(replication)`

#### 故障切换

**工作到备用切换**

+ 关于工作到备用的故障切换流程是，工作服务器发送周期信号给待机中的备用服务器。如果周期信号中断，备用服务器切换成工作服务器的 IP 地址并恢复服务。
+ 宕机时间取决于备用服务器处于“热”待机状态还是需要从“冷”待机状态进行启动。只有工作服务器处理流量。
+ 工作到备用的故障切换也被称为主从切换。

**双工作切换**

+ 在双工作切换中，双方都在管控流量，在它们之间分散负载。
+ 如果是外网服务器，DNS 将需要对两方都了解。如果是内网服务器，应用程序逻辑将需要对两方都了解。
+ 双工作切换也可以称为主主切换。

**缺陷**

+ 故障切换需要添加额外硬件并增加复杂性。
+ 如果新写入数据在能被复制到备用系统之前，工作系统出现了故障，则有可能会丢失数据。

#### 复制

+ 这一个话题可以参考数据库目录

**主—从复制**

**主—主复制**

## 重要组件的设计

+ 域名系统 DNS
+ 内容分发系统 CDN
+ 负载均衡器 Load Balance
+ 反向代理  Reverse Proxy
+ 应用
  - 微服务
  - 服务发现
+ 数据库
  - 关系数据库 RDBMS
    * 特征 ACID
    * 复制
    * 联合
    * 优化
  - 非关系数据库 NoSQL
    * 特征 BASE
    * 分类

+ 缓存
+ 异步
  - 消息队列
  - 任务队列
+ 通信
  - TCP
  - UDP
  - RPC
+ 安全

## 参考

+ [comparison-centrilized-decentrlized-and-distributed-systems](https://www.geeksforgeeks.org/comparison-centralized-decentralized-and-distributed-systems/)
+ [system-design-primer](https://github.com/donnemartin/system-design-primer)
+ [scalability-availability-stability-patterns](https://www.slideshare.net/jboner/scalability-availability-stability-patterns/)

