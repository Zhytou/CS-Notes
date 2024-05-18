# Spring Boot

- [Spring Boot](#spring-boot)
  - [概述](#概述)
  - [快速入门](#快速入门)
    - [项目结构](#项目结构)
    - [启动流程](#启动流程)
  - [核心功能](#核心功能)
    - [内嵌容器](#内嵌容器)
    - [Starter依赖](#starter依赖)
    - [自动配置](#自动配置)
    - [SpringBoot配置](#springboot配置)
  - [数据访问](#数据访问)
    - [MyBatis集成](#mybatis集成)
    - [事务管理](#事务管理)
  - [任务调度](#任务调度)
    - [@Scheduled注解](#scheduled注解)
    - [异步任务](#异步任务)
  - [日志管理](#日志管理)
  - [安全控制](#安全控制)
    - [Spring Security](#spring-security)
  - [监控管理](#监控管理)

## 概述

**简介**：

Spring Boot是Spring开源组织下的子项目，是Spring组件一站式解决方案，主要是简化了使用Spring的难度，简省了繁重的配置，提供了各种启动器，使开发者能快速上手。

**特性**：

第一，Spring Boot内置了多种Web容器（如Tomcat、Jetty和Undertow），使得Spring Boot可将项目打包成可执行jar并直接java -jar命令运行，无需额外部署到Web容器中；

第二，Spring Boot提供了一系列的Starter依赖，简化Maven配置。比如：如果要构建一个Web应用，只需添加 spring-boot-starter-web依赖，它就会自动将构建Web应用所需的Spring MVC、JSON支持等相关依赖全部引入。这样做不仅减少了手动添加依赖的工作量，同时还避免了依赖版本不一致的问题，更能方便维护只需升级Starter依赖的版本即可。

第三，Spring Boot根据项目中添加的依赖，自动配置相关的Spring组件和第三方库。例如，对比传统Spring MVC项目，使用Spring Boot时，开发者就无需配置DispatcherServlet、视图解析器等，Spring Boot会根据项目依赖自动配置这些组件。

第四，Spring Boot

第五，Spring Boot采用更灵活和简单的配置方式。它不再使用XML进行配置，而是使用Properties或YAML文件。

## 快速入门

Spring Boot旨在简化Spring应用程序的创建过程,让开发者能够快速构建生产级别的应用。让我们从一个简单的Hello World示例开始:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class HelloWorldApplication {

    @GetMapping("/")
    public String hello() {
        return "Hello, World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }
}
```

这个示例创建了一个简单的 Spring Boot 应用程序,包含一个 REST 端点,当访问根路径时,返回 "Hello, World!" 字符串。

### 项目结构

一个典型的 Spring Boot 项目结构如下:

```text
my-project
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com/example
│   │   │       └── MyProjectApplication.java
│   │   └── resources
│   │       ├── static
│   │       ├── templates
│   │       └── application.properties
│   └── test
│       └── java
│           └── com/example
│               └── MyProjectApplicationTests.java
├── pom.xml
```

- src/main/java 存放应用程序的 Java 源代码
- src/main/resources 存放配置文件和资源文件
- src/test/java 存放测试用例代码
- pom.xml Maven 构建配置文件

### 启动流程

Spring Boot 应用程序的启动过程如下:

启动入口是带有 @SpringBootApplication 注解的主类,该注解是一个组合注解,包含 @Configuration、@EnableAutoConfiguration 和 @ComponentScan。
@EnableAutoConfiguration 注解启用了 Spring Boot 的自动配置功能,根据项目依赖自动配置相关的 Bean。
@ComponentScan 注解启用组件扫描,Spring 会自动扫描当前包及其子包下的组件(@Component、@Service、@Repository、@Controller 等)并将其注册为 Bean。
在 main 方法中,调用 SpringApplication.run 方法启动 Spring 应用程序。
Spring Boot 会创建一个新的应用程序上下文(ApplicationContext),初始化自动配置的 Bean 并启动内嵌的 Web 服务器(如 Tomcat)。

## 核心功能

### 内嵌容器

### Starter依赖

### 自动配置

### SpringBoot配置

**properties**：

**YML**：

**Profile**：

## 数据访问

### MyBatis集成

### 事务管理

## 任务调度

### @Scheduled注解

### 异步任务

## 日志管理

## 安全控制

### Spring Security

用户认证
权限控制

## 监控管理

Actuator
端点(Endpoint)
健康检查
