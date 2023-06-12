# Communication

- [Communication](#communication)
  - [RPC](#rpc)
  - [SOAP](#soap)
  - [REST](#rest)
  - [GraphQL](#graphql)
  - [References](#references)

![communication timeline](https://content.altexsoft.com/media/2020/05/word-image-52.png.webp)

## RPC

远程过程调用（Remote Procedure Call，RPC）是一种允许在不同上下文中远程执行函数的规范。

**优势**：

- 简单直接的交互。
- 易于添加函数，如果我们对API有新的需求，可以轻松地添加另一个函数来执行该需求。
- 高性能。

**缺点**：

- 与底层强耦合。
- 发现性差，没有办法自省API或发送请求并根据其请求开始理解调用哪个函数。
- 可能会添加过多冗余的函数，降低代码可读性。

**使用场景**：

内部微服务系统

## SOAP

SOAP（Simple Object Access Protocol）是一种基于XML的协议，用于在Web上交换结构化的和面向对象的信息。

## REST

表述性状态转移（Representational State Transfer，REST）是一种强制的客户端/服务端架构设计模型，客户端基于服务端管理的一系列资源操作。服务端提供修改或获取资源的接口。所有的通信必须是无状态和可缓存的。

**规范**：

- 统一接口：允许以统一的方式与给定服务器交互，而不考虑设备或应用程序类型。
- 无状态：处理请求所需的状态只包含在请求本身中，而不需要服务器存储与会话相关的任何内容。
- 缓存
- 客户端-服务器架构：允许任意一方独立演进。
- 分层系统应用
- 服务器向客户端提供可执行代码的能力。

**优势**：

- 解耦的客户端和服务器。
- 可缓存。
- 支持多种数据格式。

**缺点**：

- 没有统一的标准。
- 返回大量元数据，对性能有影响。

## GraphQL

与传统的RESTful API不同，GraphQL不是基于资源的，而是基于图形的。这意味着客户端可以指定需要的数据，而服务端只返回这些数据，避免了不必要的数据传输和处理。

**优势**：

- GraphQL API 有强类型 schema。
- 按需获取。
- GraphQL支持快速产品开发。
- 丰富的开源生态和社区。

## References

- [凤凰架构-访问远程服务](http://icyfenix.cn/architect-perspective/general-architecture/api-style/)
- [Comparing API Architectural Styles: SOAP vs REST vs GraphQL vs RPC](https://www.altexsoft.com/blog/soap-vs-rest-vs-graphql-vs-rpc/)
