# Spring学习笔记

<div align="right"><font size="4.5">7月16日</div>

## 框架概述


+ Spring是一个生态系统，其中包含了Spring Framework、Spring Boot、Spring cloud等等，也即Spring全家桶。其中，Spring framework就是我们平时说的Spring框架，而Spring framework也是全家桶内其他框架的基础和核心。

+ Spring framework体系结构

  ![](../img/spring_framework_system.webp)

<div align="right"><font size="4.5">7月19日</div>

## IOC容器

### IOC容器原理

+ IOC的概念

  - 控制反转，通俗的说，就是把对象创建和对象调用过程，交给Spring进行管理。
  - 使用IOC的目的：为了降低耦合度

+ IOC底层原理

  - XML解析
  - 工厂模式
  - 反射

+ IOC创建对象的过程

  - XML解析，得到bean标签的class属性，即该对象所属类的包位置

  - 反射创建，生成对应类的class类，并加载至内存，等待调用

    > **反射的概念**
    >
    > 反射就是程序在运行的过程中，可以通过类名称创建对象，并获取类中申明的成员变量和方法。
    >
    > **Class 类概念**
    >
    > Class 也是一个 Java 类，保存的是与之对应 Java 类的 meta信息（元信息），用来描述这个类的结构，比如描述一个类有哪些成员，有哪些方法等，一般在反射中使用。
    >
    > **详细解释：**Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java  字节代码（.class 文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class类的一个实例（Class  对象）。也就是说，在 Java 中，每个 java 类都有一个相应的 Class 对象，用于表示这个 java 类的类型信息。
    >
    > **类加载概念**
    >
    > 当使用一个类的时候（比如 new 一个类的实例），会检查此类是否被加载到内存，如果没有，则会执行加载操作。
    >
    > 读取类对应的 class 文件数据，解析此数据，构造一个此类对应的 Class 类的实例。此时JVM就可以使用该类了，比如实例化此类，或者调用此类的静态方法。
    >
    > Java 也提供了手动加载类的接口，class.forName()方法就是其中之一。（说来说去，其实就是生成这个类的 Class）
    >
    > **类加载器的概念**
    >
    > 顾名思义，类加载器（class loader）用来加载 Java 类到 Java 虚拟机中。一般来说，Java 虚拟机使用 Java  类的方式如下：Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class  文件）。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class 类的一个实例。每个这样的实例用来表示一个 Java  类。通过此实例的 newInstance() 方法就可以创建出该类的一个对象。 基本上所有的类加载器都是 java.lang.ClassLoader 类的一个实例。
    >
    > **类初始化概念**
    >
    > 类被加载之后，jvm 已经获得了一个描述类结构的 Class 实例。但是还需要进行类初始化操作之后才能正常使用此类，类初始化操作就是执行一遍类的静态语句，包括静态变量的声明还有静态代码块。

### IOC接口（BeanFactory）

+ IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
+ Spring提供了IOC容器实现的两种方式（两个接口）：
  - BeanFactory：IOC容器基本实现，是Spring内部使用的接口
  - ApplicationContext：BeanFactory的子接口，提供了更多更强大的功能，一般由开发人员使用
+ ApplicationContext的两个实现类：
  - FileSystemXmlApplicationContext
  - ClassPathXmlApplicationContext

### IOC操作Bean管理

+ 什么是Bean管理

  - 创建对象
  - 属性注入

+ 两种Bean

  - 普通Bean：在Spring配置文件中，定义的类型就是返回的类型
  - 工厂Bean：在Spring配置文件中，定义的类型可以和返回的类型不同（具体实现方法：这个类要实现FactoryBean这个接口，其中有三个方法能够达到这个目的）

+ Bean的作用域

  - 在Spring中创建的对象是多实例还是单实例
  - 默认情况下，Spring是单实例
  - 可以修改配置中scope属性来更改Bean的作用域
    * 默认值，singleton，表示单实例，在加载配置文件时就创建对象
    * protytpe，表示多实例，在调用getBean函数获取对象时才创建对象
    * request
    * session

+ Bean的生命周期

  - 通过构造函数创建Bean实例（无参构造）

  - 为Bean属性设置值（调用set方法）

  - 调用Bean的初始化方法（配置文件中的init-method属性——类似触发器）

  - Bean可以使用了

  - 当容器关闭时，调用Bean的销毁方法（配置文件中的destroy-method属性——类似触发器）

    > 在第三步前后都还有传入后置处理器的步骤。具体来说，后置处理器就是Spring中定义的BeanPostProcessor 接口，通过该接口可以自定义调用初始化前后执行的操作方法。

+ Bean管理的XML方式

  ``` xml
  <bean id="helloWorld" class="net.biancheng.HelloWorld">
          <property name="message" value="Hello World!" />
  </bean>
  ```

  其中，属性id是标识符；属性class指明包所在位置；标签property指明被赋值的变量名。

  ![](../img/bean_attribute.png)

+ Bean管理的注解方式

  - 注解的概念

    * 注解是代码的特殊标记，格式：@注解名称
    * 特别注意：注解实际上是一类特殊的类
    * 使用注解的目的：简化XML配置

  - Spring针对Bean管理中对象创建提供的注解

    * @Component
    * @Service
    * @Controller
    * @Repository

  - 基于注解方式实现对象创建

    * 引入依赖（jar包）
    * 开启组件扫描（在配置文件中添加context标签）
    * 在类文件中加上注解

  - 扫描配置

    ``` xml
    <!--使用context命名空间，通知spring扫描指定目录，进行注解的解析 -->
    <context:component-scan base-package="net.biancheng" />
    ```

    扫描范围——net/biancheng/文件夹下的所有文件

    ``` xml
    <context:component-scan base-package="net.biancheng" user-default-filters="false">
    	<context:include-filter type="annotation" expression="org.springframework.sterotype.Controller">
    </context:component-scan>
    ```

    扫描范围——只扫描net/biancheng/文件夹下Controller注解

  - Spring针对Bean管理中属性注入提供的注解

    * @AutoWired：根据属性类型进行自动装配

      ``` java
      class UserService {
        @Autowired
      	private UserDao userDao;
      }
      ```

    * @Qualifier：根据属性名称进行注入（当一个接口有多个实现类时，必须用Qualifier指明用哪个子类实现）

      ``` java
      class UserService {
        @Autowired
      	@Qualifier(value = "userDaoImp1")
      	private UserDao userDao;
      }
      ```

    * @Resource：可以根据属性类型注入，也可以根据属性名称注入

      ``` java
      class UserService {
        @Resource
      	@Resource(value = "userDaoImp1")
      	private UserDao userDao;
      }
      ```

    * @Value：注入普通属性（上面三个都是注入对象的属性）

      ``` java
      @Value(value = "abc")
      private String str;
      ```

  - 完全注解开发

    * 创建配置类替代XML配置文件

      ``` java
      @Configuration
      @ComponentScan(basePackages = {"net.bianchen"})
      public class SpringConfig{
      }
      ```

    * 加载配置类

      ``` java
      ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
      ```


## AOP面向切面编程

###  AOP的概念 

+ 什么是AOP
  - 面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
  - 可以把业务逻辑各个部分相互隔离，降低耦合度
+ Spring AOP框架的底层原理
  - 动态代理
    * JDK动态代理（有接口情况，创建实现类代理对象）
    * CGLIB动态代理（无接口情况，创建子类代理对象）

<div align="right"><font size="4.5">7月20日</div>

### AOP的术语

+ 连接点：类里面可以被增强的方法称为增强点
+ 切入点：类里面实际被增强的方法称为切入点
+ 通知（增强）：实际增强的逻辑部分称为通知（增强）
  - 前置通知：在被增强方法之前添加逻辑
  - 后置通知：在被增强方法之后添加逻辑
  - 环绕通知：在被增强方法之前和之后添加逻辑
  - 异常通知：当被增强方法出现异常时执行增强逻辑
  - 最终通知：
+ 切面：把通知应用到切入点的过程

### Spring AOP（JDK动态代理）

+ 使用Proxy类里面的方法newProxyInstace创建代理对象

  - 参数1，类加载器
  - 参数2，增加方法的所在类实现的接口，支持多个接口
  - 参数3（InvocationHandler），创建代理对象写增强的部分

+ 实现JDK动态代理的代码

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

+ AspectJ 是一个基于 Java 语言的 AOP 框架，它扩展了 Java 语言，提供了强大的 AOP 功能。

  > AspectJ和Spring AOP是目前最流行的两种框架，但不同与Spring AOP，AspectJ本身不是Spring的一部分。直到 Spring 2.0 开始，Spring AOP才引入了对 AspectJ 的支持。
  >
  > 在新版本的 Spring 框架中，建议使用 AspectJ 方式开发 AOP。

+ 基于AspectJ实现AOP操作的两种方式

  - 基于XML配置文件实现
  - 基于注解方式实现

+ 切入点表达式

  - 目的：知道对哪个类型里面的哪个方法进行增强

  - 语法：execution([权限修饰符]\[返回类型][类全路径]\[方法名称]\[参数列表])  

    > 注意：1、参数列表常常用两个点表示；2、返回类型可以省略

+ 基于XML配置文件方式

  - 创建两个类，增强类和被增强类
  - 在Spring配置文件中创建两个类的对象
  - 在Spring配置文件中配置切入点

+ 基于AspectJ注解方式

  - 创建待增强类

  - 创建增强类

  - 进行通知的配置

    * 在Spring配置文件中，开启注解扫描（或用配置类，具体方法类似IOC控制）

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

      > ``` xml
      > xmlns="http://www.springframework.org/schema/beans"
      > xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      > ```
      >
      > 这个是每个配置文件必须的部分，也就是spring的根本。
      >
      > 第一句话声明xml文件默认的命名空间，表示未使用其他命名空间的所有标签的默认命名空间。
      >
      > 第二句话声明XML Schema 实例，声明后就可以使用 schemaLocation 属性了。

    * 使用注解生成待增强对象和增强对象

    * 在增强类上添加注解@Aspect

    * 在Spring配置文件中开启生成代理对象

      ``` xml
      <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
      ```

    * 配置不同类型的通知：在增强类中，在作为通知的方法上添加通知类型注解，使用切入点表达式进行配置

      ``` java
      @Before(value = "excution(*.com.atguigu.spring5.aopanno.User.add(..))")
      public void before(){
        //TODO
      }
      ```

      

  - 两点注意：

    * 相同切入点抽取（代码重用）

      ``` java
      @Pointcut(value = "excution(*.com.atguigu.spring5.aopanno.User.add(..))")
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

    * 设置优先级：多个增强类，可以设置增强类执行的优先级（具体方法：添加@order（数字）注解，order中数字小的先执行）

## JdbcTemplate

### JdbcTemplate的概念

+ 定义：Spring框架对JDBC进行了封装，也就是JdbcTemplate

### JdbcTemplate的配置

+ 引入jar包依赖

+ 添加数据源

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

  

+ 创建JdbcTemplate对象，注入DataSource

  ``` xml
  <!--配置JDBC模板-->
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  	<!--默认必须使用数据源-->
    <property name="dataSource" ref="dataSource"/>
  </bean>
  ```

+ 引用JdbcTemplate对象（一般在dao层使用）

### JdbcTemplate实现增删改查

+ 创建表（关系集和实体集）

+ 编写service和dao

  - 在dao层类中注入JdbcTemplate

  - 通过JdbcTemplate实现数据库的增删改查

    * update函数

      ``` java
      //dao层函数
      public void deleteBook(String id) {
        String sql = "delete from book where id = ?";
        //jdbcTemplate是dao层类内变量
        jdbcTemplate.update(sql, id)
      }
      ```

    * queryForObject函数

      查询返回值

      ``` java
      //dao层函数
      public int selectCount() {
        String sql = "select count(*) from book";
        //jdbcTemplate是dao层类内变量
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
        return count;
      }
      ```

      查询返回对象

      ``` java
      //dao层函数
      public Book findBook(String id) {
        String sql = "select * from book where id = ?";
        //jdbcTemplate是dao层类内变量
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
        return book;
      }
      ```

    * query函数

      查询返回集合

      ``` java
      //dao层函数
      public List<Book> findAllBook() {
        String sql = "select * from book";
        //jdbcTemplate是dao层类内变量
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
      }
      ```

  - JdbcTemplate实现数据库的批量操作

    * batchUpdate函数

      ``` java
      //dao层函数
      public void batchAddBook(List<Object[]> args) {
        String sql = "insert into book values(?,?,?)";
        //jdbcTemplate是dao层类内变量
      	jdbcTemplate.queryForObject(sql,args);
      }
      ```

<div align="right"><font size="4.5">7月21日</div>

## Spring事务管理

### 事务

+ 概念：数据库操作的基本单元
+ 特性（ACID）
  - 原子性
  - 一致性
  - 隔离性
  - 持久性

### 准备工作

+ 搭建service和dao层：将dao层注入service层，将JdbcTemplate注入dao层，将DataSource注入JdbTemplate
+ 事务逻辑常常加到JavaEE三层结构里的service层

### 事务管理

+ 有两种方式：编程式事务管理和声明式事务管理

+ 编程式事务管理

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

+ 声明式事物管理（底层原理：AOP）

  - 基于XML配置文件方式
  - 基于注解方式

### XML声明式事务管理

+ 在配置文件中创建事务管理器，并添加事务的命名空间

  ``` xml
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransationManager">
  	<property name="dataSource" ref="dataSource"/>
  </bean>
  ```

+ 配置通知

  ``` xml
  <tx:advice id="txadvice">
    <!-- 配置事务参数-->
    <tx:attributs>
      <!-- 指定在哪个方法上添加类-->
      <tx:method name="fun" propagation="REQUIRED"/>
    </tx:attributs>
  </tx:advice>
  ```

+ 配置切入点和切面

  ``` xml
  <aop:config>
    <!--配置切入点-->
    <aop:pointcut id="pt" expression="excution(* com.atguigu.spring5.service.Service)">
    <!--配置切入面-->
    <aop:advisor advice-ref="txadvice" point-ref="pt"/>
  </aop:config>
  ```

### 注解声明式事务管理

+ 在配置文件中创建事务管理器

  ``` xml
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransationManager">
  	<property name="dataSource" ref="dataSource"/>
  </bean>
  ```

+ 在配置文件中开启事务注解

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

+ 在service的类或方法中添加注解@Transactional

  - 添加到类上：这个类中所有方法都添加了事务
  - 添加到方法上：这个方法添加类事务

+ @Transactional的参数设置

  - propagation：事务传播行为
  - isolation：事务隔离级别
  - timeout：超时时间
    * 事务需要在一定时间内进行提交，超时则会回滚
    * 默认是-1
  - readOnly：是否只读
    * 读：查询操作
    * readOnly默认值为false
  - rollbackFor：回滚
    * 可以设置出现哪些异常进行事务回滚
  - noRollbackFor：不回滚
    * 可以设置出现哪些异常不进行事务回滚

### 完全注解声明式事务管理

+ 创建配置类

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
>
> 在 Spring 中使用 Log4J 是非常容易的。

### log4j日志框架整合

+ 引入jar包依赖

+ 创建配置文件（log4jx.xml）

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

  

