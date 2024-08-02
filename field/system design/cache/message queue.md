# 消息队列

- [消息队列](#消息队列)
  - [简介](#简介)
    - [概述](#概述)
    - [功能](#功能)
  - [模式](#模式)
    - [生产者到消息队列](#生产者到消息队列)
      - [队列 point to point queue](#队列-point-to-point-queue)
      - [订阅 publish/subscrib topic](#订阅-publishsubscrib-topic)
    - [消息队列到消费者](#消息队列到消费者)
  - [问题](#问题)
    - [重复消费](#重复消费)
    - [积压处理](#积压处理)
    - [消息丢失](#消息丢失)
  - [建议](#建议)
  - [参考](#参考)

## 简介

### 概述

**定义**：

消息队列就是在消息的传输过程中保存消息的容器。

**message broker v.s. message queue**：

消息代理更多的是在描述一种架构模式。这种模式被一些消息中间件实现了，比如Kafka等。这些消息中间件的服务器其实就是消息代理。

> A message broker (also know a service bus) is a piece of middleware responsible with persisting and routing of message while allowing you to decouple your system into smaller parts.

消息队列是一种消息中间件。

> A message broker is a separate component that manages queues.

### 功能

**解耦**：

降低系统间的耦合，降低协作成本，降低架构复杂度，提升系统扩展性。

**异步**：

将非核心逻辑/耗时逻辑异步处理，提高响应效率和系统吞吐。

**削峰**：

- 利用高吞吐特性，在不高的成本下承接巨大的流量。
- 将瞬时突发流量削峰，保护下游系统，提升系统可用性。

## 模式

### 生产者到消息队列

#### 队列 point to point queue

**描述**：

生产者生产消息发送到queue中，然后消费者从queue中取出并且消费消息。

**缺陷**：

#### 订阅 publish/subscrib topic

**描述**：

生产者将消息发布到topic中，同时有多个消费者订阅该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

**缺陷**：

当消息量比较大而订阅者又比较少时，会出现消息积压。

### 消息队列到消费者

- 推送 PUSH
- 拉取 PULL

## 问题

### 重复消费

**描述**：

由于网络传输等等故障，确认信息没有传送到消息队列，导致消息队列不知道自己已经消费过该消息了，再次将该消息分发给其他的消费者。

**解决**：

- 使用幂等（多次执行所产生影响与一次执行影响相同）
  - 表唯一键
  - 查询插入
  - 状态机
  - 去重表

### 积压处理

**描述**：

消费者宕机，信息积压。

**解决**：

扩容

### 消息丢失

**描述**：

- 消息可能在生产者与中转站之间、中转站本身以及消费者与中转站之间丢失。

**解决**：

- 消息确认：缓解生产者与中转站之间和消费者与中转站之间消息丢失的问题。
- 中转站数据持久化：缓解队列自身消息丢失的问题。

## 建议

- 需要理解获取运行结果，建议选择RPC而非MQ
- 消息队列是实现异步、解耦、削峰的充分非必要条件
- 将非核心、执行耗时的逻辑异步化，可以考虑使用线程池或者RPC框架提供的异步调用特性
- 依赖抽象而非具体实现，降低系统间解耦

## 参考

- [system-design-primer](https://github.com/donnemartin/system-design-primer#message-queues)
- [what is a message broker](https://www.tibco.com/reference-center/what-is-a-message-broker)
- [MessageQueue vs Message Broker](http://javaresolutions.blogspot.com/2014/08/messagequeue-vs-message-broker.html)
- [Message queue and Message broker differences](https://stackoverflow.com/questions/50061928/message-queue-and-message-broker-differences)
- [消息队列](https://baike.baidu.com/item/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97/4751675?fr=aladdin)
- [消息队列是什么](https://www.zhihu.com/question/54152397?sort=created)
- [meituan_note/messgae_queue.md](../../../meituan_note/messgae_queue.md)
