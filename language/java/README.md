# Java Review Reminder

## 基础
  
### Java Collection & Map

### Java OOP

### Java Reflection

### Java Generics

### Java Proxy

### Java Compilation

### Java Annotation

### Java IO

### Java Concurrency

## JVM

## Framework

### Spring Framework

**IoC**：

- 控制反转&依赖注入异同；
- Bean：由Spring IoC容器创建、组装和管理的对象；
  - JavaBean vs EJB vs Spring Bean；
  - 生命周期；
  - 作用域；
- IoC容器：BeanFactory/ApplicationContext；
- 配置：XML/注解/完全注解；

**AOP**：

- AOP术语：连接点、切入点、通知、切面和织入等；
- Spring AOP与AspectJ的异同；
- Spring AOP的原理：JDK动态代理&CGLib动态代理；
- Spring AOP的注解配置：
  - 切点表达式`execution(* xxx.Xyz.*(..))`；
  - 拦截器：
    - @Before；
    - @After；
    - @AfterReturning；
    - @AfterThrowing；
    - @Around；
  - 切面定义；
  - 自定义注解；
  - @EnableAspectJAutoProxy或`<aop:aspectj-autoproxy/>`开启AOP自动代理。

**Transcational Management**：

- Spring事务管理的优势；
- Spring事务管理器的接口以及配置方法；
- Spring事务传播机制；
- @Transcational的作用；

**Spring MVC**：

- DispatcherServlet；
- WebApplicationContext；
- 工作流程；
- XML配置文件；
- 常用注解；

**SSM**:

- 启动流程；
- 配置方法；

### Spring Boot

**内嵌容器**：

**Stater依赖**：

**自动配置**：

**Spring Boot配置**：

### Spring Cloud

## Component

### MyBatis

**概述**：

- JDBC vs MyBatis；
- 原理：反射&动态代理；
- 核心组件：SqlSessionFactory；

**配置**：

- 属性
- 设置
- 别名
- 插件
- 配置环境
- 映射器注册

**映射器**：

- 查询/插入/更新/删除
- 重用语句
- 参数
- 缓存

**动态SQL**：

- if元素
- choose、when、otherwise元素
- trim、where、set元素
- foreach元素
- bind元素
- test属性

### Redis
