# 系统架构

- [系统架构](#系统架构)
  - [单体架构](#单体架构)
  - [面向服务架构](#面向服务架构)
  - [微服务架构](#微服务架构)
    - [容器](#容器)
    - [云原生](#云原生)
  - [参考](#参考)

一个企业级的应用通常包含三个部分：

- 数据库：由许多的表组件构成一个通用的、相互关联的数据管理系统。
- 客户端界面：由HTML、Javascript组成，使用浏览器进行访问。
- 服务端应用：处理HTTP请求、执行领域逻辑、检索并更新数据库中的数据、使用适当的HTML视图发送给客户端。

基于这种背景，系统架构不断演变，从单体到面向服务、再到总线、到目前的微服务架构。

## 单体架构

单体架构指的是将一个应用的所有代码都放在同一个进程中运行。这种架构简单直观，开发和部署都相对容易，但随着业务规模和需求的增长，代码库会变得越来越庞大和复杂，从而导致可维护性和可扩展性变差。

## 面向服务架构

面向服务架构（Service Oriented Architecture，SOA）将整个应用按业务拆分为可重用的服务，服务之间通过标准协议(如SOAP)进行通信。这种架构提高了开发效率和服务复用，但服务界限模糊，会产生紧密耦合，且部署较为复杂。

## 微服务架构

微服务架构与SOA的核心思想相似，都是将应用拆分为独立的服务。但微服务架构的服务粒度更细、团队更小、自治性更强。每个微服务只关注于完成单一责任，服务之间通常采用轻量级通信机制，比如RPC。

具体来说，微服务架构的优点包括:

- 高内聚低耦合，每个服务只关注自身业务逻辑。
- 技术栈无关，不同服务可选用不同语言和框架。
- 独立部署，实现持续交付，提高开发效率。
- 良好的扩展性和容错性。

而其缺点是服务治理复杂，需要专门团队维护基础设施。

### 容器

容器（Container）是支撑微服务架构的核心技术。具体来说，它由以Docker为代表的容器化技术和以Kubernetes为代表的容器编排技术组成。

### 云原生

云原生（Cloud Native）是近年来兴起的一种IT基础设施和应用程序的设计和交付方法。它和微服务架构关系密切。

具体来说，云原生可以看作是一个更广阔的概念，微服务架构是云原生应用的核心部分之一。微服务作为云原生应用的组成单元，通过容器化、编排调度、动态扩缩容等手段，发挥了在云环境中的全部潜力。

## 参考

**综合**：

- [Microservices](https://martinfowler.com/articles/microservices.html)
- [微服务(Microservices)](https://blog.csdn.net/wurenhai/article/details/37659335)
- [程序员练级攻略 By 陈皓](https://www.pianshen.com/article/3847726071/)

**演变**：

- [the evolution of distributed system](https://medium.com/microservices-learning/the-evolution-of-distributed-systems-fec4d35beffd)
- [The evolution of distributed computing systems: from fundamental to new frontiers](https://iamssgill.files.wordpress.com/2021/01/lindsay2021.pdf) (refer中下载了原文)

**Microservice vs SOA**：

- [SOA vs. Microservices: What’s the Difference?](https://www.ibm.com/cloud/blog/soa-vs-microservices)

**容器部署**：

- [How to Deploy Microservices with Docker](https://www.linode.com/docs/guides/deploying-microservices-with-docker/)
