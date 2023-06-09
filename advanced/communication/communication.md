# Communication

- [Communication](#communication)
  - [RPC](#rpc)
  - [REST](#rest)
  - [GraphQL](#graphql)

## RPC

## REST

## GraphQL

表述性状态转移（REST）
REST 是一种强制的客户端/服务端架构设计模型，客户端基于服务端管理的一系列资源操作。服务端提供修改或获取资源的接口。所有的通信必须是无状态和可缓存的。

RESTful 接口有四条规则：

标志资源（HTTP 里的 URI） ── 无论什么操作都使用同一个 URI。
表示的改变（HTTP 的动作） ── 使用动作, headers 和 body。
可自我描述的错误信息（HTTP 中的 status code） ── 使用状态码，不要重新造轮子。
HATEOAS（HTTP 中的HTML 接口） ── 你的 web 服务器应该能够通过浏览器访问。

REST 关注于暴露数据。它减少了客户端／服务端的耦合程度，经常用于公共 HTTP API 接口设计。REST 使用更通常与规范化的方法来通过 URI 暴露资源，通过 header 来表述并通过 GET、POST、PUT、DELETE 和 PATCH 这些动作来进行操作。因为无状态的特性，REST 易于横向扩展和隔离。
