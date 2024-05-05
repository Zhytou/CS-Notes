# Spring

- [Spring](#spring)
  - [框架概述](#框架概述)
    - [Spring Framework](#spring-framework)
    - [Spring Boot](#spring-boot)
  - [控制反转 IoC](#控制反转-ioc)
    - [依赖注入 DI](#依赖注入-di)
    - [IoC容器](#ioc容器)
      - [Bean](#bean)
      - [IoC容器的原理](#ioc容器的原理)
      - [IoC容器的配置](#ioc容器的配置)
  - [面向切面编程 AOP](#面向切面编程-aop)
    - [AOP的术语](#aop的术语)
    - [Spring AOP（JDK动态代理）](#spring-aopjdk动态代理)
    - [AspectJ](#aspectj)
  - [JdbcTemplate](#jdbctemplate)
    - [JdbcTemplate的概念](#jdbctemplate的概念)
    - [JdbcTemplate的配置](#jdbctemplate的配置)
    - [JdbcTemplate实现增删改查](#jdbctemplate实现增删改查)
  - [Spring事务管理](#spring事务管理)
    - [事务](#事务)
    - [准备工作](#准备工作)
    - [事务管理](#事务管理)
    - [XML声明式事务管理](#xml声明式事务管理)
    - [注解声明式事务管理](#注解声明式事务管理)
    - [完全注解声明式事务管理](#完全注解声明式事务管理)
  - [Spring日志](#spring日志)
    - [log4j日志框架整合](#log4j日志框架整合)

## 框架概述

Spring是一个生态系统，其中包含了Spring Framework、Spring Boot、Spring Cloud等等，也即Spring全家桶。

### Spring Framework

其中，Spring Framework就是我们平时说的Spring框架。它Spring的基石，提供了许多核心特性和功能，比如：

- 依赖注入(DI)和控制反转(IoC)
- 面向切面编程(AOP)
- 数据访问抽象层
- 事务管理
- Web编程模块(如Spring MVC)
- 测试支持

其中，它提供的Spring MVC功能大大简化了Java企业级Web应用的开发，也是SSM框架（Spring+Spring MVC+MyBatis）的重要组成部分。

Spring Framework体系结构如下图：

![Spring Framework模块结构](https://docs.spring.io/spring-framework/docs/4.3.x/spring-framework-reference/html/images/spring-overview.png)

### Spring Boot

Spring Boot是Spring生态系统中的另一个重要项目，它建立在Spring Framework的基础之上。Spring Boot的目的是简化Spring应用的配置和部署。它提供的功能包括：提供自动配置的开箱即用体验、内嵌Tomcat/Jetty等Servlet容器、提供生产级监控和管理功能和集成大量开源中间件等等。通过Spring Boot，开发人员可以更快速地构建基于Spring的应用程序，省去大量样板配置代码。

此外，相比于SSM框架在DAO层限制只能使用MyBatis，Spring Boot更灵活。它没有和任何MVC框架绑定，也没有和任何持久层框架绑定，同样也没有和任何其他业务领域的框架绑定。

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

- 什么是AOP
  - 面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
  - 可以把业务逻辑各个部分相互隔离，降低耦合度
- Spring AOP框架的底层原理
  - 动态代理
    - JDK动态代理（有接口情况，创建实现类代理对象）
    - CGLIB动态代理（无接口情况，创建子类代理对象）

### AOP的术语

- 连接点：类里面可以被增强的方法称为增强点
- 切入点：类里面实际被增强的方法称为切入点
- 通知（增强）：实际增强的逻辑部分称为通知（增强）
  - 前置通知：在被增强方法之前添加逻辑
  - 后置通知：在被增强方法之后添加逻辑
  - 环绕通知：在被增强方法之前和之后添加逻辑
  - 异常通知：当被增强方法出现异常时执行增强逻辑
  - 最终通知：
- 切面：把通知应用到切入点的过程

### Spring AOP（JDK动态代理）

- 使用Proxy类里面的方法newProxyInstace创建代理对象

  - 参数1，类加载器
  - 参数2，增加方法的所在类实现的接口，支持多个接口
  - 参数3（InvocationHandler），创建代理对象写增强的部分

- 实现JDK动态代理的代码

  ``` java
  //需要被代理的对象
  Object targetobject;
  //调用newProxyInstance方法创建代理对象，其中Jdkproxy类实现了InvocationHandler，具体代码如下
  Object proxy = Proxy.newProxyInstance(targetObject.getClass().getClassLoader(),targetObject.getClass().getInterfaces(),new JdkProxy(targetobject));
  //通过代理调用被增强过的方法method
  proxy.method();
  ```

  ``` java
  // 实现InvocationHandler的类
  class JdkProxy implents InvocationHandler {
    //需要代理的对象
    private Object target;
    public JdkProxy (Object target) {
      this.target = target;
    }
    
    @override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      //需要被增强方法method执行之前的逻辑
      //TODO
      
      //需要被增强方法method执行
      Obejct result = method.invoke(target, args);
      
      //需要被增强方法method执行之后的逻辑
  		//TODO
  
    }
  }
  ```

### AspectJ

- AspectJ 是一个基于 Java 语言的 AOP 框架，它扩展了 Java 语言，提供了强大的 AOP 功能。

  > AspectJ和Spring AOP是目前最流行的两种框架，但不同与Spring AOP，AspectJ本身不是Spring的一部分。直到 Spring 2.0 开始，Spring AOP才引入了对 AspectJ 的支持。
  >
  > 在新版本的 Spring 框架中，建议使用 AspectJ 方式开发 AOP。

- 基于AspectJ实现AOP操作的两种方式

  - 基于XML配置文件实现
  - 基于注解方式实现

- 切入点表达式

  - 目的：知道对哪个类型里面的哪个方法进行增强

  - 语法：execution([权限修饰符]\[返回类型][类全路径]\[方法名称]\[参数列表])  

    > 注意：1、参数列表常常用两个点表示；2、返回类型可以省略

- 基于XML配置文件方式

  - 创建两个类，增强类和被增强类
  - 在Spring配置文件中创建两个类的对象
  - 在Spring配置文件中配置切入点

- 基于AspectJ注解方式

  - 创建待增强类

  - 创建增强类

  - 进行通知的配置

    - 在Spring配置文件中，开启注解扫描（或用配置类，具体方法类似IOC控制）

      ``` xml
      <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">
        
        <bean>
        	...
        </bean>
      </beans>
      ```



    - 使用注解生成待增强对象和增强对象

    - 在增强类上添加注解@Aspect

    - 在Spring配置文件中开启生成代理对象

      ``` xml
      <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
      ```

    - 配置不同类型的通知：在增强类中，在作为通知的方法上添加通知类型注解，使用切入点表达式进行配置

      ``` java
      @Before(value = "excution(-.com.atguigu.spring5.aopanno.User.add(..))")
      public void before(){
        //TODO
      }
      ```

      

  - 两点注意：

    - 相同切入点抽取（代码重用）

      ``` java
      @Pointcut(value = "excution(-.com.atguigu.spring5.aopanno.User.add(..))")
      public void pointFunc(){
        //TODO
      }
      
      //通过直接调用pointFunc而避免重写切入表达式
      @before(value = "pointFunc")
      public void before(){
        //TODO
      }
      
      //通过直接调用pointFunc而避免重写切入表达式
      @after(value = "pointFunc")
      public void after(){
        //TODO
      }
      ```

    - 设置优先级：多个增强类，可以设置增强类执行的优先级（具体方法：添加@order（数字）注解，order中数字小的先执行）

## JdbcTemplate

### JdbcTemplate的概念

- 定义：Spring框架对JDBC进行了封装，也就是JdbcTemplate

### JdbcTemplate的配置

- 引入jar包依赖

- 添加数据源

  ``` xml
  <!-- 配置数据源 --> 
  <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <!--数据库驱动-->
    <property name="driverClassName" value="com.mysql.jdbc.Driver" /> 
    <!--连接数据库的url-->
    <property name= "url" value="jdbc:mysql://localhost/xx" />
    <!--连接数据库的用户名-->
    <property name="username" value="root" />
    <!--连接数据库的密码-->
    <property name="password" value="root" />
  </bean>
  ```

  

- 创建JdbcTemplate对象，注入DataSource

  ``` xml
  <!--配置JDBC模板-->
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  	<!--默认必须使用数据源-->
    <property name="dataSource" ref="dataSource"/>
  </bean>
  ```

- 引用JdbcTemplate对象（一般在dao层使用）

### JdbcTemplate实现增删改查

- 创建表（关系集和实体集）

- 编写service和dao

  - 在dao层类中注入JdbcTemplate

  - 通过JdbcTemplate实现数据库的增删改查

    - update函数

      ``` java
      //dao层函数
      public void deleteBook(String id) {
        String sql = "delete from book where id = ?";
        //jdbcTemplate是dao层类内变量
        jdbcTemplate.update(sql, id)
      }
      ```

    - queryForObject函数

      查询返回值

      ``` java
      //dao层函数
      public int selectCount() {
        String sql = "select count(-) from book";
        //jdbcTemplate是dao层类内变量
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
        return count;
      }
      ```

      查询返回对象

      ``` java
      //dao层函数
      public Book findBook(String id) {
        String sql = "select - from book where id = ?";
        //jdbcTemplate是dao层类内变量
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
        return book;
      }
      ```

    - query函数

      查询返回集合

      ``` java
      //dao层函数
      public List<Book> findAllBook() {
        String sql = "select - from book";
        //jdbcTemplate是dao层类内变量
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
      }
      ```

  - JdbcTemplate实现数据库的批量操作

    - batchUpdate函数

      ``` java
      //dao层函数
      public void batchAddBook(List<Object[]> args) {
        String sql = "insert into book values(?,?,?)";
        //jdbcTemplate是dao层类内变量
      	jdbcTemplate.queryForObject(sql,args);
      }
      ```

## Spring事务管理

### 事务

- 概念：数据库操作的基本单元
- 特性（ACID）
  - 原子性
  - 一致性
  - 隔离性
  - 持久性

### 准备工作

- 搭建service和dao层：将dao层注入service层，将JdbcTemplate注入dao层，将DataSource注入JdbTemplate
- 事务逻辑常常加到JavaEE三层结构里的service层

### 事务管理

- 有两种方式：编程式事务管理和声明式事务管理

- 编程式事务管理

  ``` java
  try {
    //开启事务
    //TODO
    
    //业务逻辑
    //TODO
  
    //事务提交
    //TODO
  
  }
  catch(Exception e) {
    //事务回滚
    //TODO
  }
  ```

- 声明式事物管理（底层原理：AOP）

  - 基于XML配置文件方式
  - 基于注解方式

### XML声明式事务管理

- 在配置文件中创建事务管理器，并添加事务的命名空间

  ``` xml
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransationManager">
  	<property name="dataSource" ref="dataSource"/>
  </bean>
  ```

- 配置通知

  ``` xml
  <tx:advice id="txadvice">
    <!-- 配置事务参数-->
    <tx:attributs>
      <!-- 指定在哪个方法上添加类-->
      <tx:method name="fun" propagation="REQUIRED"/>
    </tx:attributs>
  </tx:advice>
  ```

- 配置切入点和切面

  ``` xml
  <aop:config>
    <!--配置切入点-->
    <aop:pointcut id="pt" expression="excution(- com.atguigu.spring5.service.Service)">
    <!--配置切入面-->
    <aop:advisor advice-ref="txadvice" point-ref="pt"/>
  </aop:config>
  ```

### 注解声明式事务管理

- 在配置文件中创建事务管理器

  ``` xml
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransationManager">
  	<property name="dataSource" ref="dataSource"/>
  </bean>
  ```

- 在配置文件中开启事务注解

  - 添加命名空间

    ``` xml
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
      <!-- 其他配置：数据源配置、JdbcTemplate类配置以及事务管理器配置 -->
      
    </beans>
    ```

  - 开启事务注解

    ``` xml
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
    ```

- 在service的类或方法中添加注解@Transactional

  - 添加到类上：这个类中所有方法都添加了事务
  - 添加到方法上：这个方法添加类事务

- @Transactional的参数设置

  - propagation：事务传播行为
  - isolation：事务隔离级别
  - timeout：超时时间
    - 事务需要在一定时间内进行提交，超时则会回滚
    - 默认是-1
  - readOnly：是否只读
    - 读：查询操作
    - readOnly默认值为false
  - rollbackFor：回滚
    - 可以设置出现哪些异常进行事务回滚
  - noRollbackFor：不回滚
    - 可以设置出现哪些异常不进行事务回滚

### 完全注解声明式事务管理

- 创建配置类

  ``` java
  @Configuration
  @ComponentScan(basePackages="")
  @EnableTransactionManger
  class TxConfig {
    //创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource() {
      DruidDataSource dataSource = new DruidDataSource();
      dataSource.setDriverClassName("xxx.xxx.xxx");
      dataSource.setUrl("jdbc:mysql://user_db");
      dataSource.setUsername("root");
      dataSource.setPassword("root");
      return dataSource;
    }
    
    //创建JdbcTemplate对象
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
    	JdbcTemplate jdbcTemplate = new JdbcTemplate();
      jdbcTemplate.setDataSource(dataSource);
      return jdbcTemplate;
    }
    
    //创建事务管理器
    public DataSourceTransactionManager getDataSource
  }
  ```

## Spring日志

> 日志是应用软件中不可缺少的部分，Apache 的开源项目 Log4J 是一个功能强大的日志组件。
在 Spring 中使用 Log4J 是非常容易的。

### log4j日志框架整合

- 引入jar包依赖

- 创建配置文件（log4jx.xml）

  ``` xml
  <?xml version="1.0" encoding="UTF-8"?>
   <Configuration status="WARN">
     <Appenders>
       <Console name="Console" target="SYSTEM_OUT">
         <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
       </Console>
     </Appenders>
     <Loggers>
       <Root level="error">
         <AppenderRef ref="Console"/>
       </Root>
     </Loggers>
   </Configuration>
  ```
