# 负载均衡

## 简介

### 概述

- 负载均衡，英文名称为Load Balance，其含义就是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行。

### 算法

#### 轮询 Round Robin

**描述**：

- 这种方法会将收到的请求循环分配到服务器集群中的每台机器，即有效服务器。

#### 加权轮询 Weighted Round Robin

**描述**：

- 这种算法解决了简单轮循调度算法的缺点：传入的请求按顺序被分配到集群中服务器，但是会考虑提前为每台服务器分配的权重。
- 管理员只是简单的通过服务器的处理能力来定义各台服务器的权重。

#### 最少连接 Least Connection

**描述**：

- 传入的请求是根据每台服务器当前所打开的连接数来分配的。
- 活跃连接数最少的服务器会自动接收下一个传入的请求。

#### 加权最少连接 Weighted Least Connection

**描述**：

- 由管理员根据服务器情况定制的权重所决定的活跃连接数一般提供了一种对服务器非常平衡的利用。
- 它借鉴了最少连接和权重两者的优势。

### 分类

#### Layer 4 Load Balancing

**描述**：

- Layer 4 load balancers look at info at the transport layer to decide how to distribute requests.  

**例子**：

- NAT负载均衡

#### Layer 7 Load Balancing

- Layer 7 load balancers look at the application layer to decide how to distribute requests.  

### 功能

- 负载均衡构建在原有网络结构之上，它提供了一种透明且廉价有效的方法扩展服务器和网络设备的带宽、加强网络数据处理能力、增加吞吐量、提高网络的可用性和灵活性。

## 应用

### DNS负载均衡

**描述**：

- 在DNS中为多个地址配置同一个名字，因而查询这个名字的客户机将得到其中一个地址，从而使得不同的客户访问不同的服务器，达到负载均衡的目的。

**缺陷**：

- DNS负载均衡是一种简单而有效的方法，但是它不能区分服务器的差异，也不能反映服务器的当前运行状态。

### NAT负载均衡

**描述**:

- 简单地说就是将一个IP地址转换为另一个IP地址，一般用于未经注册的内部地址与合法的、已获注册的Internet IP地址间进行转换。

### 反向代理负载均衡

**描述**:

- 反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求；
- 然后，将请求转发给内部网络上的服务器；
- 最后，将从服务器上得到的结果返回给internet上请求连接的客户端。
- 此时代理服务器对外就表现为一个服务器。

## 参考

- [system-design-primer](https://github.com/donnemartin/system-design-primer#sharding)
- [How does a load balancer distribute client traffic across severs?](https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques/)
- [负载均衡](https://baike.baidu.com/item/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/932451?fr=aladdin)
- [什么是负载均衡原理](https://www.zhihu.com/question/61783920)
- [负载均衡调度算法大全](https://www.runoob.com/w3cnote/balanced-algorithm.html)
