# Maven

- [Maven](#maven)
  - [Introduction](#introduction)
  - [Dependency](#dependency)
    - [Scope](#scope)
    - [Unique Identification](#unique-identification)
  - [Lifecycle \& Phase](#lifecycle--phase)
  - [Plugin](#plugin)
  - [Module Management](#module-management)
  - [Maven with IDE](#maven-with-ide)

## Introduction

Maven是一个Java项目管理和构建工具，它可以定义项目结构、项目依赖，并使用统一的方式进行自动化构建，是Java项目不可缺少的工具。

**Maven Project Structure**：

一个使用Maven管理的普通的Java项目，它的目录结构默认如下：

```txt
a-maven-project
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       ├── java
│       └── resources
└── target
```

其中，pom.xml正是Maven的配置文件。它全称是Project Object Model，包含了项目的基本信息，依赖声明、插件和构建信息等等。

**A Simple Example**：

## Dependency

### Scope

Maven定义了几种依赖关系，分别是：

- compile: 编译时需要用到该jar包（默认）
- test: 编译Test时需要用到该jar包
- runtime: 编译时不需要，但运行时需要用到
- provided: 编译时需要用到，但运行时由JDK或某个服务器提供

比如：在使用java.servlet和tomcat插件开发web程序时，我们可能就需要在依赖中添加

### Unique Identification

对于某个依赖，Maven只需要3个变量即可唯一确定某个jar包：

- groupId：属于组织的名称，类似Java的包名；
- artifactId：该jar包自身的名称，类似Java的类名；
- version：该jar包的版本。

## Lifecycle & Phase

## Plugin

我们在前面介绍了Maven的lifecycle，phase和goal：使用Maven构建项目就是执行lifecycle，执行到指定的phase为止。每个phase会执行自己默认的一个或多个goal。goal是最小任务单元。

我们以compile这个phase为例，如果执行：

mvn compile
Maven将执行compile这个phase，这个phase会调用compiler插件执行关联的compiler:compile这个goal。

实际上，执行每个phase，都是通过某个插件（plugin）来执行的，Maven本身其实并不知道如何执行compile，它只是负责找到对应的compiler插件，然后执行默认的compiler:compile这个goal来完成编译。

所以，使用Maven，实际上就是配置好需要使用的插件，然后通过phase调用它们。

Maven已经内置了一些常用的标准插件：

|插件名称|对应Phase|
|-------|---------|
|clean| clean|
|compiler|compile|
|surefire|test|
|jar|package|

如果标准插件无法满足需求，我们还可以使用自定义插件。比如，我们希望使用Tomcat插件来简化Java Web的测试流程。此时，我们就需要在pom.xml声明这个插件。

```xml

```

## Module Management

Maven可以有效地管理多个模块，我们只需要把每个模块当作一个独立的Maven项目，它们有各自独立的pom.xml。

当某修模块的配置有大量相似之处时，我们可以提取出共同部分作为parent，比如：

例如，模块A的pom.xml：

## Maven with IDE
