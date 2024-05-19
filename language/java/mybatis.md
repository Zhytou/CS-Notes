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
    - [属性 properties](#属性-properties)
    - [设置 settings](#设置-settings)
    - [别名 typeAliases](#别名-typealiases)
    - [类型处理器 typeHandler](#类型处理器-typehandler)
    - [插件 plugins](#插件-plugins)
    - [配置环境 environments](#配置环境-environments)
    - [数据库厂商标识 databaseIdProvider](#数据库厂商标识-databaseidprovider)
    - [映射器注册 mappers](#映射器注册-mappers)
  - [映射器](#映射器)
    - [查询 select](#查询-select)
    - [插入 insert](#插入-insert)
    - [更新 updaet](#更新-updaet)
    - [删除 delete](#删除-delete)
    - [重用语句 sql](#重用语句-sql)
    - [参数 parameterType](#参数-parametertype)
    - [返回结果 resultMap](#返回结果-resultmap)
    - [缓存 cache](#缓存-cache)
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

- 通过XMLConfigBuilder解析全局配置文件，构建出全局Configuration对象；
- 根据配置文件指定的映射器位置，解析映射器XML并构建相应的MappedStatement对象；
- 向Configuration中注册MappedStatement对象；
- 使用Configuration初始化DefaultSqlSessionFactory对象。

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

### 属性 properties

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

### 设置 settings

### 别名 typeAliases

类型别名可以为Java类型设置一个缩写名字，但仅用于XML设置，意在降低冗余的全限定类名书写。值得注意的是，MyBatis里面分为系统定义和自定义别名两类，且MyBatis别名是不区分大小写。对于系统别名来说，它主要包含的就是一些基本类型的别名定义，比如：_byte、_long、_int等。而对于自定义别名来说，一个最简单的例子如下：

```xml
<typeAliases>
  <typeAlias alias="role" type="org.example.pojo.Role"/>
</typeAliases>
```

此外，也可以通过指定包名，开启扫描的方式定义别名。比如：

```xml
<typeAliases>
  <package name="org.example.pojo"/>
</typeAliases>
```

在配置之后，MyBatis就会将包中的所有类加载到配置上下文中。其中，默认的名字就是类名全小写，不过也可以使用注解指定别名，比如：

```java
@Alias("role")
class Role {
  // ... 
};
```

### 类型处理器 typeHandler

MyBatis在设置预处理语句（PreparedStatement）中的参数或从结果集（ResultSet）中取出一个值时，都会用注册了的类型处理器将获取到的值以合适的方式转换成Java类型。类型处理器和别名相同，也分为系统定义和用户定义两种。

### 插件 plugins

### 配置环境 environments

配置环境可以注册多个数据源，而每一个数据源的配置又分成两部分：一是数据库连接的配置，二是数据库事务的配置。比如：

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="autoCommit" value="false"/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="com.mysql.jdbc.driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/oa"/>
      <property name="username" value="root"/>
      <property name="password" value="1234"/>
    </dataSource>
  </environment>
</environments>
```

其中，environments标签中的属性default，表明在缺省的情况下，我们将启用哪个数据源配置。而environment元素是配置一个数据源的开始，属性id是这个数据源的标志，以便上下文使用。

transactionManager元素配置的数据库事务，其中的type属性有三种配置方式：

- JDBC
- MANAGED
- 自定义

### 数据库厂商标识 databaseIdProvider

### 映射器注册 mappers

映射器是MyBatis最复杂、最核心的组件，而mappers元素的作用正是将写好的映射器注册到全局配置中。它有三种引入方法。

1、用文件路径引入，比如：

```xml
<mappers>
  <mapper resource="org/example/mapper/userMapper.xml"/>
</mappers>
```

2、用包名引入该路径下的全部映射器，比如：

```xml
<mappers>
  <package name="org/example/mapper"/>
</mappers>
```

3、用映射器接口引入，比如：

```xml
<mappers>
  <mapper class="org.example.mapper.userMapper"/>
</mappers>
```

4、用资源限定符引入，比如：

```xml
<mappers>
  <mapper url="file:///var/mappers/org/example/mapper/userMapper.xml"/>
</mappers>
```

## 映射器

映射器是MyBatis中最重要、最核心的文件，它包含一组SQL语句，并且可以通过配置生成对应的JavaBean返回给调用者。此外，MaBatis还支持动态SQL来满足不同场景的需求。同时，它也支持动态绑定JavaBean，只需要要让SQL返回的字段和JavaBean的属性保持一直即可省去繁琐的映射配置。

### 查询 select

### 插入 insert

### 更新 updaet

### 删除 delete

### 重用语句 sql

### 参数 parameterType

### 返回结果 resultMap

resultMap是MyBatis中最复杂的元素，其作用是定义映射规则、级联的更新、定义转换器等。它通常包含以下元素：

constructor - 用于在实例化类时，注入结果到构造方法中
idArg - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能
arg - 将被注入到构造方法的一个普通结果
id – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
result – 注入到字段或 JavaBean 属性的普通结果
association – 一个复杂类型的关联；许多结果将包装成这种类型
嵌套结果映射 – 关联可以是 resultMap 元素，或是对其它结果映射的引用
collection – 一个复杂类型的集合
嵌套结果映射 – 集合可以是 resultMap 元素，或是对其它结果映射的引用
discriminator – 使用结果值来决定使用哪个 resultMap
case – 基于某些值的结果映射
嵌套结果映射 – case 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射

### 缓存 cache

**一级缓存**：

MyBatis提供缓存支持，但在没有配置的默认情况下，它只开启一级缓存。一级缓存中各个SqlSession的缓存是相互隔离的，即针对一个SqlSession的缓存。

**二级缓存**：

为了减少冗余查询，我们需要配置二级缓存。它是针对全局的，但同时也要求查询返回的POJO必须是可序列化的，即实现了Serializable接口。其配置方法很简单，需要在XML文件中添加`<cache/>`即可。这个简单的语句效果如下：

- 映射语句文件中的所有select 语句的结果将会被缓存。
- 映射语句文件中的所有insert、update和delete语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

此外，二级缓存还有一些高级属性可以设置，包括：

- eviction：缓存回收策略，可用值包括：
  - LRU：最近最少使用，移除最长时间不使用的对象。
  - FIFO：先进先出。
  - SOFT：软引用，基于垃圾回收器状态移除对象。
  - WEAK：弱引用，更积极的基于垃圾回收器状态移除对象。
- flushInterval：刷新间隔时间，单位为毫秒。
- size：缓存大小，代表可用存储多少个对象。
- readOnly：只读，意味着缓存只能读取不可修改。

**自定义缓存**：

在大型服务器上，往往需要定制缓存，比如使用Redis缓存。此时，就需要实现MyBatis提供的接口org.apache.ibatis.cache.Cache，然后再引入cache标签即可。比如：

```xml
<cache type="org.example.cache.RedisCache">
```

### 注解

除了使用XML配置文件的方式实现映射器之外，MyBatis还支持使用注解的方式实现映射器。开发者只需要在接口中使用Java注解，注入SQL即可。比如：

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

如果使用注解和XML文件两种方式同时定义，那么XML方式将覆盖掉注解方式。虽然这里注解的方式看起来比XML简单，但是现实中我们遇到的SQL会比该例子复杂得多，如果SQL语句中有多个表的关联、多个查询条件、级联、条件分支等，显然这条SQL就会复杂的多，所以并不建议使用这种方式。

## 动态SQL

动态SQL是MyBatis的强大特性之一，在JDBC或其他类似的框架中，开发人员需要手动拼接SQL语句，而MyBatis则通过动态SQL提供了SQL语句拼装能力。

## Spring集成
