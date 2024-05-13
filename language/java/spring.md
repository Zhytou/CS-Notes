# Spring

- [Spring](#spring)
  - [框架概述](#框架概述)
    - [Spring Framework](#spring-framework)
    - [Spring Boot](#spring-boot)
    - [Spring Cloud](#spring-cloud)
  - [控制反转 IoC](#控制反转-ioc)
    - [依赖注入 DI](#依赖注入-di)
    - [IoC容器](#ioc容器)
      - [Bean](#bean)
      - [IoC容器的原理](#ioc容器的原理)
      - [IoC容器的配置](#ioc容器的配置)
  - [面向切面编程 AOP](#面向切面编程-aop)
    - [Spring AOP](#spring-aop)
    - [AspectJ](#aspectj)
  - [Web开发 Spring MVC](#web开发-spring-mvc)
  - [数据访问 Data Access](#数据访问-data-access)
  - [参考 Reference](#参考-reference)

## 框架概述

Spring是一款开源的轻量级Java开发框架，旨在提高开发人员的开发效率以及系统的可维护性。它包含了Spring Framework、Spring Boot、Spring Cloud等等模块。

### Spring Framework

其中，Spring Framework就是我们平时说的Spring框架。它Spring的基石，提供了许多核心特性和功能，比如：

- 核心机制
  - 依赖注入(DI)和控制反转(IoC)
  - 面向切面编程(AOP)
- 数据访问
  - DAO
  - ORM
  - 事务管理
- Web编程模块(如Spring MVC)
- 测试支持

其中，它提供的Spring MVC功能大大简化了Java企业级Web应用的开发，也是SSM框架（Spring+Spring MVC+MyBatis）的重要组成部分。

Spring Framework体系结构如下图：

![Spring Framework模块结构](https://docs.spring.io/spring-framework/docs/4.3.x/spring-framework-reference/html/images/spring-overview.png)

### Spring Boot

Spring Boot是Spring生态系统中的另一个重要项目，它建立在Spring Framework的基础之上。Spring Boot的目的是简化Spring应用的配置和部署。它提供的功能包括：提供自动配置的开箱即用体验、内嵌Tomcat/Jetty等Servlet容器、提供生产级监控和管理功能和集成大量开源中间件等等。通过Spring Boot，开发人员可以更快速地构建基于Spring的应用程序，省去大量样板配置代码。

此外，相比于SSM框架在DAO层限制只能使用MyBatis，Spring Boot更灵活。它没有和任何MVC框架绑定，也没有和任何持久层框架绑定，同样也没有和任何其他业务领域的框架绑定。

### Spring Cloud

Spring Cloud是Spring官方支持的微服务全家桶，主要目标是简化分布式系统的开发。它基于Spring Boot，并整合了众多开源组件:

- 服务发现和注册(如Eureka)
- 智能路由(如Zuul)
- 客户端负载均衡(如Ribbon)
- 断路器模式(如Hystrix)
- 配置中心
- 分布式跟踪

Spring Cloud提供了一整套微服务架构的解决方案，帮助开发人员快速构建分布式、高可用、高扩展的微服务系统。

## 控制反转 IoC

IoC(Inversion of Control)，即控制反转是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。IoC的思想是将原本在程序中手动创建对象的控制权，交由调度框架或程序组件来管理。程序本身只需定义好具体的行为即可，至于行为的执行时机由管理外部对象施加控制。

Spring通过提供一种名为IoC容器的类来帮助管理对象，使用者可以通过配置文件和注解的方式告诉容器要创建哪些对象以及如何配置对象之间的依赖关系。配置完成之后，所有实例不再由应用程序自己创建和配置，而是由IoC容器负责。

### 依赖注入 DI

**IoC vs DI**：

IoC是一种降低模块耦合、提高对象复用的思想，在Spring中它依靠依赖注入(Dependency Injection, DI)的方式实现，但并非所有支持IoC的框架都使用DI实现。因此，我们不能将二者等同。总之，控制反转是一种思想，而依赖注入则是一种设计模式。

**什么是依赖**：

如果在 Class A 中，有 Class B 的实例，则称 Class A 对 Class B 有一个依赖。因此，依赖注入也就是将实例变量传入到一个对象中去。

### IoC容器

IoC容器也就是Spring框架中提供给开发者用于实现反转控制的工具，而被管理的类则被称为Bean。具体来说，Spring提供了IOC容器实现的两种方式（两个接口）：BeanFactory和ApplicationContext。

其中，BeanFactory是IoC容器所具有的最基本形式，也被称为IoC容器的最底层实现。它由org.springframework.beans.factory.BeanFactory接口定义,主要的功能是对Bean的实例化、配置、管理等。而ApplicationContext则是BeanFactory的子接口，它不仅提供BeanFactory所具有的功能，还提供了更多企业级功能，如解析自定义配置、启动和关闭回调、国际化消息、事件传播等。ApplicationContext有多个具体的实现类，比较常用的有：ClassPathXmlApplicationContext、FileSystemXmlApplicationContext和AnnotationConfigApplicationContext等。

一般来说，开发人员更多地使用ApplicationContext，因为它支持更多功能和特性。而BeanFactory则被认为是底层容器，更多地在框架内部使用。

**BeanFactory和ApplicationContext的特点**：

BeanFactory的特点是每次使用时都会创建一个新的Bean实例。因此，多次请求注入同一个Bean时实际得到的都是不同的实例。而ApplicationContext在初始化时，默认会创建并初始化所有的单例Bean。因此，我们可以认为ApplicationContext是以单例模式运行的容器。

**一个基于XML配置的IoC容器使用例子**：

首先定义两个简单的类，一个作为被注入的依赖对象，另一个作为需要依赖注入的类:

``` java
// 依赖对象
public class MessageService {
    public String getMessage() {
        return "Hello from MessageService";
    }
}

// 需要依赖注入的类
public class MessageRenderer {
    private MessageService messageService;

    // 需要通过构造器注入依赖对象
    public MessageRenderer(MessageService messageService) {
        this.messageService = messageService;
    }

    public void renderMessage() {
        System.out.println(messageService.getMessage());
    }
}
```

接下来创建Spring配置文件applicationContext.xml:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 定义 MessageService Bean -->
    <bean id="messageService" class="com.example.MessageService"/>

    <!-- 定义 MessageRenderer Bean, 并注入 MessageService 依赖 -->
    <bean id="renderer" class="com.example.MessageRenderer">
        <constructor-arg ref="messageService"/>
    </bean>

</beans>
```

最后在主程序中使用ApplicationContext获取Bean实例:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        // 创建 ApplicationContext 容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        // 从容器中获取 MessageRenderer 实例
        MessageRenderer renderer = context.getBean("renderer", MessageRenderer.class);

        // 调用渲染方法
        renderer.renderMessage();
    }
}
```

#### Bean

在Spring中，Bean是指一个由IoC容器创建、组装和管理的对象。

**Bean的作用域**是指Bean在Spring整个框架中的某种行为模式。常见的作用域类型包括：

- singleton：单例作用域
- prototype：原型作用域（多例作用域）
- request：请求作用域
- session：会话作用域
- application：全局作用域

**Bean的生命周期**：

- 实例化 Instantiation
- 属性赋值 Populate
- 初始化 Initialization
- 销毁 Destruction

#### IoC容器的原理

IoC容器可以动态地创建和管理对象，而其底层原理则涉及到反射和工厂模式。具体来说，使用它创建对象的过程如下：

- XML配置/注解配置解析，得到需要管理类的class属性，即该对象所属类的包位置；
- 反射创建，生成对应类的class类，并加载至内存，等待调用。

**反射**：

反射就是程序在运行的过程中，可以通过类名称创建对象，并获取类中申明的成员变量和方法。

**Class类**：

Class 也是一个 Java 类，保存的是与之对应 Java 类的元信息，用来描述这个类的结构，比如描述一个类有哪些成员，有哪些方法等，一般在反射中使用。实际上，Java 源程序在经过 Java 编译器编译之后就被转换成 Java  字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class类的一个实例。也就是说，在 Java 中，每个类都有一个相应的 Class 对象，用于表示这个类的类型信息。

**工厂模式**：

能够创建和管理Bean对象的IoC容器本质上就是一个Bean Factory。

#### IoC容器的配置

Spring框架支持两种IoC容器配置方式:XML配置和注解配置。

**XML方式**：

XML配置是Spring早期使用的配置方式，也是最传统的方式。开发人员需要在一个或多个XML文件中定义Bean及其依赖关系。下面是一个简单的XML配置示例:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="messageService" class="com.example.MessageService"/>

    <bean id="renderer" class="com.example.MessageRenderer">
        <property name="messageService" ref="messageService"/>
    </bean>
</beans>
```

其中，配置文件的含义解释如下：

- xmlns是XML NameSpace的缩写，因为XML文件的标签名称都是自定义的，自己写的和其他人定义的标签很有可能会重复命名，而功能却不一样，所以需要加上一个namespace来区分这个xml文件和其他的xml文件，类似于java中的package。
- xmlns:xsi指出了XML文件遵守XML规范，其中xsi全称是xml schema instance。
- xsi:schemaLocation是指本文档里的XML元素所遵守的规范，这些规范都是由Spring官方制定的。
- bean则是IoC容器的具体配置。其中，属性id是标识符；属性class指明包所在位置；标签property指明被赋值的变量名。

**注解方式**：

注解是代码的特殊标记，其格式格式一般如下@注解名称。注意：注解实际上是一类特殊的类

随着Spring版本的不断迭代，注解配置已经成为了主流的配置方式。使用注解可以很大程度上简化XML配置。下面是一个使用注解配置的示例:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public MessageService messageService() {
        return new MessageService();
    }

    @Bean
    public MessageRenderer renderer() {
        MessageRenderer renderer = new MessageRenderer();
        renderer.setMessageService(messageService());
        return renderer;
    }
}
```

事实上，使用注解的方式配置IoC容器除了在类文件中加上注解之外，还需要引入依赖和开启组件扫描。其中，前者引入org.springframework.context.annotation.Bean等jar包使注解可用；后者启用自动扫描机制，并为符合条件的类创建 Bean 对象。这样就避免了在配置文件中一个一个的使用`<bean>`标签显式定义Bean。下面就是两个组件扫描配置的例子，它们有不同的扫描范围。

``` xml
<!--扫描范围——net/biancheng/文件夹下的所有文件 -->
<context:component-scan base-package="net.biancheng" />


<!--扫描范围——只扫描net/biancheng/文件夹下Controller注解 -->
<context:component-scan base-package="net.biancheng" user-default-filters="false">
  <context:include-filter type="annotation" expression="org.springframework.sterotype.Controller">
</context:component-scan>
```

最后来具体介绍一下，Spring提供的注解。它们可以按对象创建和属性注入分两类。以下这些注解都是用于标记Spring管理的Bean组件，只是分别对应不同的应用层:

- @Component 通用组件
- @Service 服务层组件
- @Controller 表现层组件
- @Repository 持久层组件

换句话说，一旦某个类被上面这些注解标注之后，那么它的对象创建和管理都将交由Spring框架负责。此外，还有另一类注解用于说明依赖注入关系，它们往往用在类的成员变量、构造方法和方法上。比如：

- @Autowired 自动装配Bean：根据Bean的类型自动注入，可注入到构造器、方法、属性上
- @Resource 注入Bean：@Resource与@Autowired功能相似，支持根据类型和名称自动装配
- @Qualifier 限定描述Bean：配合@Autowired使用，当有多个同类型Bean时，指定注入哪个Bean
- @Value 注入常量：常用于注入基本类型/String的属性值

下面是一个使用这些注解的一个例子。

``` java
@Component("fooFormatter")
public class FooFormatter implements Formatter {
    public String format() {
        return "foo";
    }
}

@Component("barFormatter")
public class BarFormatter implements Formatter {
    public String format() {
        return "bar";
    }
}

@Component
public class FooService {
    @Autowired
    @Qualifier("barFormatter")
    private Formatter formatter;
    
    //todo 
}
```

**完全注解方式**：

除了XML和注解这两种方式之外，Spring还支持的另一种配置方式，即：只使用注解配置。

通过创建一个配置类 SpringConfig并使用 @Configuration 注解标记，该类就成为了 Spring 的配置类，相当于以前的 XML 配置文件。@ComponentScan 注解指定需要扫描的包路径，Spring 将自动发现这些包及子包下标注了 @Component、@Service、@Repository、@Controller 等注解的类，并自动将它们注册为 Bean。这个功能类似于 XML 配置中的 <context:component-scan> 元素。

``` java
@Configuration
@ComponentScan(basePackages = {"net.bianchen"})
public class SpringConfig{
}
```

通过使用 AnnotationConfigApplicationContext 并将配置类 SpringConfig.class 作为参数传入,即可创建一个基于注解配置的 Spring 应用上下文环境。

``` java
ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
```

## 面向切面编程 AOP

AOP(Aspect Oriented Programming) 面向切面编程是一种编程范式，可以将遵循单一职责原则的代码进行解耦，把分布于多个模块中的与业务无关的代码重新集中到一个共同维护的模块中，从而提高代码复用性。这种与业务无关的横切代码被称为横切关注点，比如日志、事务管理、安全检查等。

传统的面向对象编程只关注对象，而AOP则关注横切多个对象的行为，将对象内的散乱完成多个特定行为的语句进行封装，从而让编程逻辑更加清晰简洁。

**术语**：

在AOP编程中，我们经常会遇到下面的概念：

- Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；
- Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；
- Pointcut：切入点，即一组连接点的集合；
- Advice：增强，指特定连接点上执行的动作；
- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；
- Weaving：织入，指将切面整合到程序的执行流程中；
- Interceptor：拦截器，是一种实现增强的方式；
- Target Object：目标对象，即真正执行业务的核心逻辑对象；
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。

### Spring AOP

Spring框架对AOP的概念提供了支持，称为Spring AOP。Spring AOP为程序提供了面向切面增强的编程能力，允许程序员定义方法执行的拦截规则，甚至可以新增增强处理，从而实现功能上的织入。Spring AOP使用动态代理机制实现，有两种实现方式:

- 基于 JDK 动态代理，为接口创建代理对象
- 基于 CGLib 动态代理，为指定类创建子类的代理对象

### AspectJ

AspectJ是一个基于Java语言的 AOP 实现，它扩展了 Java 语言本身，使其包含 AOP 的概念构造，如切面、切入点、通知等。AspectJ 定义了 AOP 语法，可直接编写代码进行AOP开发。

但不同与Spring AOP，AspectJ本身不是Spring的一部分。直到 Spring 2.0 开始，Spring AOP才引入了对 AspectJ 的支持。在新版本的 Spring 框架中，建议使用 AspectJ 方式开发 AOP。类似IoC容器，基于AspectJ实现AOP操作也可以通过XML和注解的两种方式。

## Web开发 Spring MVC

## 数据访问 Data Access

## 参考 Reference

[Spring Framework文档](https://docs.spring.io/spring-framework/reference/)
