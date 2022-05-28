# 微服务架构

> 微服务是分布式系统中最近比较流行的架构模型，也是 SOA 架构的一个进化。微服务架构并不是 银弹，所以，也不要寄希望于微服务架构能够解决所有的问题。微服务架构主要解决的是如何快速 地开发和部署我们的服务，这对于一个能够适应快速开发和成长的公司是非常必要的。同时我也觉 得，微服务中有很多很不错的想法和理念，所以学习微服务是每一个技术人员迈向卓越的架构师的 必经之路。
>
> ——陈皓 左耳朵耗子

## 简介

### 概述

**通常的系统结构**

+ 企业应用通常包含三个部分：
  - 客户端界面：由HTML、Javascript组成，使用浏览器进行访问。
  - 数据库：由许多的表组件构成一个通用的、相互关联的数据管理系统。
  - 服务端应用：处理HTTP请求、执行领域逻辑、检索并更新数据库中的数据、使用适当的HTML视图发送给客户端。
+ 基于这种背景，系统架构不断演变，从单体到面向服务、再到总线、到目前的微服务架构（具体见第二章）。

**微服务架构定义**

+ 微服务架构，是一种软件开发技术- 面向服务的体系结构（SOA）架构样式的一种变体。
+ 它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值。

**Microservice vs SOA**

+ 不同之处：

  ![1](../img/microservice_vs_soa.png)

  

+ 最大的不同：

  > The main distinction between the two approaches comes down to **scope**. To put it simply, service-oriented architecture (SOA) has an enterprise scope, while the microservices architecture has an application scope.

![Graphic relating soa to microservices](https://1.cms.s81c.com/sites/default/files/2020-09-02/SOA_microservices%20%281%29.png)

### 特性

+ 组件与服务
+ 围绕业务功能进行组织
+ 产品而非项目
+ 弱化通道，强化终端
+ 分散治理
+ 分散数据管理
+ 基础设施自动化

## 演变

![](../img/distributed_system_history.png)

+ 此处主要关注的是万维网出现之后，分布式系统的架构变化。

### 点对点集成架构 Point To Point Integration Architecture P2P

>  Point to point architecture, also known as peer to peer architecture.

#### 概述

+ 单体结构，即客户端和服务端是两个独立节点，且客户端和服务端都完全集成运行在一台电脑上。

#### 优劣

**优势**

+ 结构简单，编写快

**缺陷**

+ 功能扩展及运维困难

### 面向服务架构 Service Oriented Architecture SOA

#### 概述

+ 面向服务架构（SOA）是一个组件模型，它将应用程序的不同功能单元（称为服务）进行拆分，并通过这些服务之间定义良好的接口和协议联系起来。
+ 接口是采用中立的方式进行定义的，它应该独立于实现服务的硬件平台、操作系统和编程语言。

#### 优劣

**优势**

+ 更易维护
+ 更高的可用性
+ 更好的伸缩性

### 数据总线架构 Enterprise Service Bus ESB

#### 概述

+ 和SOA类似，一般来说SOA架构中的服务通过ESB通信，即也采用了ESB架构。

### 微服务架构 Microservice Architecture

## 技术

### 虚拟机和容器

## 参考

**综合**

+ [Microservices](https://martinfowler.com/articles/microservices.html)
+ [微服务(Microservices)](https://blog.csdn.net/wurenhai/article/details/37659335)
+ [程序员练级攻略 By 陈皓](https://www.pianshen.com/article/3847726071/) (原文件下载在notes中)

**演变**

+ [the evolution of distributed system](https://medium.com/microservices-learning/the-evolution-of-distributed-systems-fec4d35beffd)
+ [The evolution of distributed computing systems: from fundamental to new frontiers](https://iamssgill.files.wordpress.com/2021/01/lindsay2021.pdf) (refer中下载了原文)
+ [分布式服务架构的演变](https://blog.csdn.net/qq_33223299/article/details/86715371)
+ [分布式架构演变历史](https://blog.csdn.net/yuhaiyang_1/article/details/80862914)

**Microservice vs SOA**

+ [SOA vs. Microservices: What’s the Difference?](https://www.ibm.com/cloud/blog/soa-vs-microservices)

**容器部署**

+ [How to Deploy Microservices with Docker](https://www.linode.com/docs/guides/deploying-microservices-with-docker/)

