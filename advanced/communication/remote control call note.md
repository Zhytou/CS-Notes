# 远程过程调用 RPC

# [RPC框架](http://www.hyhblog.cn/2018/01/29/rpc_arch_future/)

## 简介

### 概述

**IPC & LPC & RPC**

+ 进程间通信（IPC）是在多任务操作系统或联网的计算机之间运行的程序和进程所用的通信技术。
+ 其中，进程间通信（IPC）由可按照相互通信的进程是否在同一计算机中分为本地进程调用（LPC）和远场进程调用（RPC）。

**RPC & TCP**

### 目的

+ 让使用者像调用本地方法一样完成远程方法的调用

### 应用场景

#### RPC应用场景

+ 互联网系统整体通常是C/S或B/S的架构

  - 在这一类前后端分离的架构下，RPC技术是我们后端子系统之间交互的标准方式

    * 前端 $\longrightarrow$​后端：通常是HTTP API调用的方式进行交互

    * 后端子系统：通常是RPC调用的方式交互

      ![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_normal_process.png)

## 技术

### 序列化与反序列化

#### 定义

+ 序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。
+ 反序列化（Deserialization）是将可以存储或传输的形式转换为对象的状态信息的过程。

## 使用

### 服务提方

![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_work_process.png)

#### 定义接口契约

+ 内容：

  - 定义服务
  - 定义方法
  - 定义入参结构
  - 定义响应结构

+ 方法：

  - IDL方式

    ![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_idl.png)

  - Java注解方式

    ![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_java_annotation.png)

#### 实现接口逻辑

![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_interface.png)

+ 遵从接口契约，实现RPC方法的业务逻辑
+ 注意RPC接口不应该抛出业务异常（异常应在服务方模块内解决，若实在解决不了，可以使用状态码的方式告诉调用方）

#### 完成服务发布配置

![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_server_setting.png)

+ 方式：
  - XML
  - Java注解

#### 启动

### 调用方

#### 引入接口契约依赖

+ 用maven下载服务方提供的接口jar包

#### 配置调用Proxy

![](D:\ZhY\Workspace\Notes\meituan_note\img\rpc_proxy_setting.png)

+ 方式：
  - XML
  - Java注解（需要特定框架支持）

+ 配置参数：
  - remoteAppKey：被调用方服务的身份标识
  - remoteServerPort：RPC服务的端口
  - timeout：调用超时时间

#### 使用调用逻辑

+ 像调用本地方法一样调用远程方法

## 参考

+ [system-design-primer](https://github.com/donnemartin/system-design-primer)
+ [序列化与反序列化](https://kb.cnblogs.com/page/515982/)