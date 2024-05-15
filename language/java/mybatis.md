# MyBatis

- [MyBatis](#mybatis)
  - [概述](#概述)
    - [特点对比](#特点对比)
    - [简单例子](#简单例子)
  - [原理](#原理)
    - [核心组件](#核心组件)
    - [反射和动态代理](#反射和动态代理)
    - [会话工厂的构建](#会话工厂的构建)
    - [会话的执行](#会话的执行)
  - [配置](#配置)
    - [property标签](#property标签)
    - [setting标签](#setting标签)
    - [typeAliases标签](#typealiases标签)
    - [typeHandler标签](#typehandler标签)
    - [plugin标签](#plugin标签)
    - [typeAlias标签](#typealias标签)
    - [package标签](#package标签)
    - [environments标签](#environments标签)
    - [databaseIdProvider标签](#databaseidprovider标签)
    - [mapper标签](#mapper标签)
  - [映射器](#映射器)
    - [标签](#标签)
    - [注解](#注解)
  - [动态SQL](#动态sql)
  - [Spring集成](#spring集成)

## 概述

MyBatis是一个开源、轻量级的数据持久化框架，是JDBC和Hibernate的替代方案。

### 特点对比

**JDBC**：

**Hibernate**:

**MyBatis**:

### 简单例子

使用MyBatis时，我们需要向其提供以下文件：

1、全局配置文件（mybatis-config.xml），定义的数据源和注册的映射器：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="/UserMapper.xml" />
    </mappers>
</configuration>
```

2、映射器文件（UserMapper.xml），定义SQL语句和返回类型:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper">
  <select id="findAll" resultType="org.example.pojo.User">
      select * from User
  </select>
</mapper>
```

3、POJO类（User.java）和Mapper接口（UserMapper.java）

```java
package org.example.pojo;

public class User {
  private int id;
  private String username;

  // getter
  // ...
  
  // setter
  // ...
}

package org.example.mapper;

public interface UserMapper {
  public List<User> findAll();
}
```

通过这个简单的示例，我们可以看到MyBatis的使用过程非常简洁方便。开发者只需提供数据源、SQL语句和映射规则，并且定义相应的POJO类和Mapper接口，MyBatis就可以利用JDK动态代理自动生成对应的Mapper对象来根据我们提供的SQL语句操作POJO对象，从而实现Java对象到数据库表的映射。

对于UserMapper这个例子，如果在SSM技术栈下，我们可以使用IoC容器方便的管理它，并在在Service层中调用相关操作，比如：

```java
package org.example.service;

@Service
public class UserService {
  @Autowired
  UserMapper userMapper;

  public void xxx() {
    List<User> users = userMapper.findAll();
    // ...
  }
};
```

## 原理

上一章节中的UserMapper示例演示了在Spring框架下集成MyBatis的用法，而MyBatis本身也可以通过类似JDBC的方式，打开面向数据库的会话并执行已注册的SQL语句。这就涉及到了MyBatis的核心组件及其工作原理，同时也解释了为什么只需提供Mapper接口和映射器文件，而无需实现实体类。

![MyBatis工作流程](https://pic1.zhimg.com/v2-bc338a28008c9e0fa50deb6010ddf95c_r.jpg)

### 核心组件

MyBatis的核心组件包括：

- SqlSessionFactoryBuilder（构造器）：用于构建SqlSessionFactory实例
- SqlSessionFactory（会话工厂）：一个工厂接口，用于创建SqlSession实例
- SqlSession（会话）：线程安全的会话对象，用于执行持久化操作

当MyBatis运行时，它使用SqlSessionFactoryBuilder读取全局配置，并构建SqlSessionFactory对象。SqlSessionFactory实际上是一个接口，构建器创建的真正对象是继承于它的DefaultSqlSessionFactory类实例。

通过SqlSessionFactory的openSession方法，可以获取SqlSession对象。SqlSession类似于JDBC中的Connection，提供了面向数据库执行SQL命令所需的所有方法。，以通过它直接运行映射的SQL语句。比如，对于上一章节UserMapper的例子，使用SqlSession的调用方法如下：

```java
public static void main(String[] args) {
    //加载核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");

    // 获得sqlSession工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);

    //获得sqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    // 执行sql语句
    List<User> userList = sqlSession.selectList("userMapper.findAll");
    // 或使用Mapper执行sql语句
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> userList2 = mapper.findAll();

    // 打印结果
    for (User user : userList) {
        System.out.println(user);
    }

    // 释放资源
    sqlSession.close();
}
```

### 反射和动态代理

MyBatis的工作原理主要依赖于两种技术:反射和动态代理。

反射主要应用于MyBatis初始化时解析配置文件,构建Configuration对象。Configuration包含了数据源信息、映射器信息等核心配置数据。通过反射,MyBatis可以从XML中构建出对应的Java对象。

动态代理则发挥作用于执行映射语句时。当调用映射器接口的方法时,MyBatis会通过动态代理方式为其创建代理对象,代理对象负责拦截方法调用,并执行相应的数据库操作。从外部看,就像是直接调用了Mapper接口方法一样,实际上是由MyBatis框架自动完成了底层的数据库访问。

### 会话工厂的构建

SqlSessionFactoryBuilder的作用是根据指定的环境及配置信息,构建出SqlSessionFactory对象。构建过程包括以下几个步骤:

通过XMLConfigBuilder解析全局配置文件,构建出全局Configuration对象
根据配置文件指定的映射器位置,解析映射器XML并构建相应的MappedStatement对象
向Configuration中注册MappedStatement对象
使用Configuration初始化DefaultSqlSessionFactory对象

### 会话的执行

![SqlSession执行](https://pdai.tech/images/mybatis/mybatis-y-arch-4.png)

## 配置

一个常见的MyBatis项目配置如下。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration><!-- 配置 -->
    <properties /><!-- 属性 -->
    <settings /><!-- 设置 -->
    <typeAliases /><!-- 类型命名 -->
    <typeHandlers /><!-- 类型处理器 -->
    <objectFactory /><!-- 对象工厂 -->
    <plugins /><!-- 插件 -->
    <environments><!-- 配置环境 -->
        <environment><!-- 环境变量 -->
            <transactionManager /><!-- 事务管理器 -->
            <dataSource /><!-- 数据源 -->
        </environment>
    </environments>
    <databaseIdProvider /><!-- 数据库厂商标识 -->
    <mappers /><!-- 映射器 -->
</configuration>
```

其中，DOCTYPE声明包含对DTD文件的引用。而DTD（Document Type Definition，DTD）文件定义了XML文档的结构以及合法元素和属性。

另外，我们就能看出，在MyBatis的配置文件中：configuration节点为根节点。在configuration节点之下，我们可以配置10个子节点， 分别为：properties、typeAliases、plugins、objectFactory、objectWrapperFactory、settings、environments、databaseIdProvider、typeHandlers、mappers。

### property标签

properties是一个配置属性的元素，能让我们在配置文件的上下文中使用它。MyBatis提供了三种配置方式：

- property子元素；
- properties配置文件；
- 程序参数传递。

此外，当三种配置方式同时出现，MyBatis按照如下顺序来加载：

- properties元素内指定属性首先被读取；
- 接着根据resource属性或url属性，并覆盖已读取的同名属性；
- 读取作为方法参数传递的属性，并覆盖已读取的同名属性。

**property子元素**:

使用property子元素的配置方法如下：

```xml
<properties>
  <property name="driver" value="com.mysql.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
  <property name="username" value="root"/>
  <property name="password" value="learn"/>
</properties>
```

这样就可以在上下文中使用已经配置好的属性值了，比如配置数据库时就可以使用这些属性。

```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
</dataSource>
```

**properties配置文件**：

当我们使用properties配置文件（内容如下），可以直接使用resource属性将其引入，比如：

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis
username=root
password=1234
```

```xml
<properties resource="jdbc.properties"/>
```

### setting标签

### typeAliases标签

### typeHandler标签

### plugin标签

- 作用：为MyBatis添加拦截器
- 使用方法 ：
  - 使用name属性设置项属性名
  - 使用value属性设置值

``` xml
<!-- 拦截器 -->
<plugins>
  <plugin interceptor="cn.com.mybatis.plugins.MyInterceptor" />
  <plugin interceptor="cn.com.mybatis.plugins.MyInterceptor2">
    <property name="name" value="mybatis"/>
    <property name="age" value="10"/>
  </plugin>
</plugins>
```

### typeAlias标签

- 作用：MyBatis别名处理器设置
- 使用方法：
  - 使用type属性指定要起别类名的类型全类名（默认别名是类名小写）
  - 使用alias属性指定别名

``` xml
<typeAlias type="xxx.xxx.xxx" alias="xxx"/>
```

### package标签

- 作用：MyBatis批量别名处理器设置
- 使用方法：
  - 使用name属性指定包名

``` xml
<package name="xxx.xxx.xxx">
```

### environments标签

- 作用：配置MyBatis环境

- 使用方法：

  - environments标签
    - default属性指定默认使用环境
  - environment标签
    - id属性是每种环境的唯一标识
    - transactionManager 标签
      - type属性指定事务管理器类型，例如：JDBC（JdbcTransactionManager），也可以自定义
    - dataSource标签
      - type属性指定数据源类型，例如：POOLED

``` xml
<environment id="">
  <transactionManager type=""/>
  <dataSource type=""/>
</environment>
```

### databaseIdProvider标签

- 作用：支持多数据库厂商

``` xml
<databaseIdProvider type="DB_VENDOR">
</databaseIdProvider>
```

### mapper标签

- 作用：将写好的sql映射文件注册到全局配置中
- 使用方法：

  - 注册配置文件
    - 使用resource属性引用类路径下的sql映射文件
    - 使用url属性引用网络路径或磁盘路径下的sql映射文件
  - 注册接口
    - 使用class属性引用注册接口接口
    - 使用条件：
      - 有sql映射文件，映射文件名必须和接口同名，且与接口放在同一目录
      - 没有sql映射文件，所有sql都是利用注解写在接口上

- 注册配置文件

  ``` xml
  <mapper resource="xxx.xxx.xxx"/>
  ```

- 注册接口（注解方式）

  ``` xml
  <mapper class="xxx.xxx.EmployeeMapperAnnotation"/>
  ```

  ``` java
  public interface EmployeeMapperAnnotation{
    @Select("select * from employee where id=#{id}")
    public Employee getEmpById(Integer id);
  } 
  ```

## 映射器

- 定义：映射器是 MyBatis 中最重要的文件，文件中包含一组 SQL 语句（例如查询、添加、删除、修改），这些语句称为映射语句或映射 SQL 语句
- 作用：
  - 定义参数类型
  - 配置缓存
  - 提供 SQL 语句和动态 SQL
  - 定义查询结果和 POJO 的映射关系

### 标签

### 注解

使用注解的方式实现映射器，只需要在接口中使用Java注解，注入SQL即可。比如：

```java
package net.biancheng.mapper;
import java.util.List;
import net.biancheng.po.Website;
public interface WebsiteMapper {
    @Select(value = "select * from website")
    public List<Website> selectAllWebsite();
    
    @Select(value = "select * from website where id=#{id}")
    public Website selectOneWebsite(String id);
}
```

如果使用注解和 XML 文件两种方式同时定义，那么 XML 方式将覆盖掉注解方式。虽然这里注解的方式看起来比 XML 简单，但是现实中我们遇到的 SQL 会比该例子复杂得多，如果 SQL 语句中有多个表的关联、多个查询条件、级联、条件分支等，显然这条 SQL 就会复杂的多，所以并不建议读者使用这种方式

## 动态SQL

动态SQL是MyBatis的强大特性之一，在JDBC或其他类似的框架中，开发人员需要手动拼接SQL语句

## Spring集成
