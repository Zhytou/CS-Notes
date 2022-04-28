# Cookie&Session

> 在Web发展史中，我们知道浏览器与服务器间采用的是http协议，而这种协议是**无状态**的，所以这就导致了服务器无法知道是谁在浏览网页，但很明显，一些网页需要知道用户的状态，例如登陆，购物车等。
>
> 所以为了解决这一问题，先后出现了四种技术，分别是隐藏表单域，URL重写，cookie，session。而用的最多也是比较重要的就是cookie和session了。

## Cookie

### 简介

#### 定义

+ cookie是浏览器保存在用户电脑上的一小段文本

#### 作用

+ cookie与特定的Web文档关联在一起, 保存了该客户机访问这个Web文档时的信息, 当客户机再次访问这个Web文档时这些信息可供该文档使用

#### 特点

+ 长度有限制
+ 数量有限制
+ 有时效性
+ 通常由key/value组成

### 实现

#### 流程

+ 服务器通过发送一个称为Set-Cookie的http消息来创建一个Cookie
+ 接着，Cookie保存在客户端
+ 再之后，当我们去请求一个URL时，浏览器会根据这个URL路径将符合条件的Cookie放在请求头中传给服务器

#### 分类

**会话级别Cookie**

+ 所谓会话级别Cookie，就是在浏览器关闭之后Cookie就会失效

**持久级别Cookie**

+ 保存在硬盘的Cookie，只要设置了过期时间就是硬盘级别Cookie

## Session

### 简介

#### 概述

+ Session是一种被成为“会话控制”的计算机网络技术

#### 目的

+ Session技术能够利用==Session对象==存储特点用户会话所需的属性及配置信息

## 实现

#### 概述

+ 简单来说，保存在服务端的Session对象通过客户端传输来的Session ID来区分不同客户端，并为其提供配置信息

#### 方式

+ 通过URL传递Session ID

+ 通过Cookie传递Session ID

+ 通过SSL传递Session ID

+ 通过隐藏表单传递Session ID

## 总结

### 相同点

+ Session和Cookie都是为了让http协议又状态而存在
+ Session通过Cookie工作，Cookie传输的SessionID让Session知道这个客户端到底是谁

### 不同点

+ Session将信息保存到服务器，Cookie将信息保存在客户端