## MyBatis学习笔记
<div align="right"><font size="4.5">7月14日</div>

## MyBatis

### 简介

- MyBatis 是一个开源、轻量级的数据持久化框架，是 JDBC 和 Hibernate 的替代方案。
- MyBatis 内部==封装了 JDBC==，简化了加载驱动、创建连接、创建 statement 等繁杂的过程，开发者只需要关注 SQL 语句本身。
- MyBatis 支持定制化 SQL、存储过程以及高级映射，可以在实体类和 SQL 语句之间建立映射关系，是一种半自动化的 ORM 实现。

### ORM

> ORM框架是面向对象程序设计语言与关系数据库发展不同步时的中间解决方案，因为当使用一种面向对象的编程语言来进行应用开发时,从项目一开始就采用的是面向对象分析,面向对象设计,面向对象编程,但到了持久层数据库访问时,又必须重返关系数据库的访问方式,这是一种非常糟糕的感觉。

- ORM的全称是:Object Relational Mapping,即==对象/关系数据库映射==

- 采用ORM框架之后,应用程序不再直接访问底层数据库,而是以面向对象的方式来操作持久化对象(例如创建,修改,删除等),而ORM框架则将这些面向对象的操作转化成底层的SQL操作

![](../img/orm_system.png)

### MyBatis、Hibernate 和 JDBC

- 和 Hibernate 相比，MyBatis 封装性低于 Hibernate，但性能优秀、小巧、简单易学、应用广泛。
- 和 JDBC 相比，MyBatis 减少了 50% 以上的代码量，并且满足高并发和高响应的要求。

<div align="right"><font size="4.5">7月22日</div>

## MyBatis工作原理

![](../img/mybatis_system.png)

+ 根据xml配置文件（全局配置文件）创建一个Sql SessionFactory对象

  ``` xml-dtd
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

+ 写好映射器，并将其注册到全局配置文件中

  ``` xml-dtd
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="net.biancheng.mapper.Mapper">
      <!-- sql语句信息 id：唯一标识 returnType：返回类型 -->
      <select id="xxxx"
          resultType="xxx.xxx.xxx">
          select * from website
      </select>
  </mapper>
  ```
  
+ 通过SqlSessionFactory获取一个SqlSession实例，能够直接执行已经映射的sql语句

  - 通过SqlSession直接调用

    ```java
    //sqlSessionFactory是之前通过配置文件创建的
    SqlSession session = sqlSessionFactory.openSession();
    //selectItem函数是映射器中已注册函数
    //xxx.xxx.xxx是sql语句的唯一标识（一般是mapper文件位置+sql语句的id）， args是sql待输入参数
    Object selectedItem = session.selectOne("xxx.xxx.xxx", args); 
    ```

  - 通过SqlSession获取Mapper接口，接着通过Mapper接口实现SQL（与配置文件动态绑定）

    * 映射器命名空间指定为接口文件全类名

    * select标签改为接口类中方法名

      ``` java
      //Mabatis会为接口自动创建一个代理对象，代理对象去执行增删该查方法
      XXXMapper mapper = session.getMapper(XXXMapper.class)
      //使用mapper调用接口中的方法，实现增删改查操作
      //TODO
      ```

      

## MyBatis全局配置文件

+ MyBatis的配置文件包含了影响MyBatis行为甚深的设置（Settings）和属性（Properties）信息

+ dtd约束文件（定义了可使用的标签）

  ``` xml
  <!--config的约束文件-->
  <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <!--mapper的约束文件--->
  <!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  ```

### property标签

+ 作用：mybatis可以使用properties标签引入外部properties配置文件的内容

+ 使用方法：

  - 使用url属性或resource属性引入；
    * url属性：引入网络或磁盘路径下的资源
    * resource属性：引入类路径下的资源
  - 使用value属性取出外部配置文件具体值

+ e.g.

  ``` xml
  <datasource type="POOLED">
  	<property name="dirver" value="${jdbc.driver}"/>
  </datasource>
  ```

+ 注意：Mybatis和Spring整合时，这部分内容常常交给Spring处理

### setting标签

+ 作用：修改MyBatis运行时行为（很重要）

+ 使用方法 ：

  - 使用name属性设置项属性名

  - 使用value属性设置值

+ e.g.

  ``` xml
  <settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  ```

### plugin标签

+ 作用：为MyBatis添加拦截器

+ 使用方法 ：

  - 使用name属性设置项属性名
  - 使用value属性设置值

+ e.g.

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

+ 作用：MyBatis别名处理器设置

+ 使用方法：

  - 使用type属性指定要起别类名的类型全类名（默认别名是类名小写）
  - 使用alias属性指定别名

+ e.g.

  ``` xml
  <typeAlias type="xxx.xxx.xxx" alias="xxx"/>
  ```

### package标签

+ 作用：MyBatis批量别名处理器设置

+ 使用方法：

  - 使用name属性指定包名

+ e.g.

  ``` xml
  <package name="xxx.xxx.xxx">
  ```

### typeHandler标签

+ 作用：建立java对象和数据库对象的映射

+ e.g.

  ``` xml
  <typeHandler handler=""/>
  ```

### environments标签

+ 作用：配置MyBatis环境

+ 使用方法：

  - environments标签
    * default属性指定默认使用环境
  - environment标签
    * id属性是每种环境的唯一标识
    * transactionManager 标签
      - type属性指定事务管理器类型，例如：JDBC（JdbcTransactionManager），也可以自定义
    * dataSource标签
      * type属性指定数据源类型，例如：POOLED

+ e.g.

  ``` xml
  <environment id="">
  	<transactionManager type=""/>
    <dataSource type=""/>
  </environment>
  ```

### databaseIdProvider标签

+ 作用：支持多数据库厂商

+ e.g.

  ``` xml
  <databaseIdProvider type="DB_VENDOR">
  </databaseIdProvider>
  ```

### mapper标签

+ 作用：将写好的sql映射文件注册到全局配置中
+ 使用方法：

  - 注册配置文件
    * 使用resource属性引用类路径下的sql映射文件
    * 使用url属性引用网络路径或磁盘路径下的sql映射文件
  - 注册接口
    * 使用class属性引用注册接口接口
    * 使用条件：
      * 有sql映射文件，映射文件名必须和接口同名，且与接口放在同一目录
      * 没有sql映射文件，所有sql都是利用注解写在接口上

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

### ==特别注意==

+ 当Spring和MyBatis集成时，mapper的xml文件需要在Spring配置文件中注册，具体有两种方法：

  - 开启mapper xml==文件扫描==（mapper xml文件中会包含其对应的mapper接口位置）

    ``` xml
    <!--  配置sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 配置数据源 -->
    	<property name="dataSource" ref="multipleDataSource"/>
        <!-- 配置mybatis配置文件的位置 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!-- 配置扫描mapper xml文件的位置 -->
        <property name="mapperLocations">
            <list>       <value>classpath:mybatis_mapper/*Mapper.xml</value>
            </list>
        </property>
    </bean>
    ```

  - 如果不配置MapperScannerConfigurer，则需要单独定义每一个mapper接口的bean

    ```xml
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    	<!-- 配置扫描mapper接口的包路径 -->
        <property name="basePackage" value="com.mystudy.dao"/>
        <property name="sqlSessionTemplateBeanName" value="sqlSessionTemplate"/>
    </bean>
    ```

    单独定义每一个mapper接口（在接口多的情况下）代码太繁琐，冗杂

+ 除此之外，二者数据源的配置是类似的，可以写在Spring配置中也可以写在MyBatis配置中

<div align="right"><font size="4.5">7月29日</div>

## MyBatis映射文件

+ 定义：映射器是 MyBatis 中最重要的文件，文件中包含一组 SQL 语句（例如查询、添加、删除、修改），这些语句称为映射语句或映射 SQL 语句
+ 作用：
  - 定义参数类型
  - 配置缓存
  - 提供 SQL 语句和动态 SQL
  - 定义查询结果和 POJO 的映射关系

### XML实现映射器

#### 定义

+ XML 定义映射器分为两个部分：==接口和XML==

#### 流程

+ 首先定义接口

  ``` java
  package net.biancheng.mapper;
  import java.util.List;
  import net.biancheng.po.Website;
  public interface WebsiteMapper {
      public List<Website> selectAllWebsite();
    
    	public Website selectOneWebsite();
  }
  ```

+ 接着写对应的Mapper.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  
  <mapper namespace="net.biancheng.mapper.WebsiteMapper">
  
      <!-- 查询所有网站信息 -->
      <select id="selectAllWebsite"
          resultType="net.biancheng.po.Website">
          select * from website
      </select>
  	
    	<!-- 查询一个网站信息 -->
  </mapper>
  ```

  

+ 然后在MyBatis配置文件中注册上述Mapper，

  ```xml
  <mappers>
  <mapper resource="net/biancheng/mapper/WebsiteMapper.xml" />
  <!-- 可以用mapper标签继续添加其他映射器-->
  </mappers>
  ```

+ 接着可以用第一步定义的接口引用通过SqlSession获取的Mapper，调用该接口的函数

  ```java
  //用第一步的接口引用mapper
  WebsiteMapper websiteMapper = sqlSession.getMapper(WebsiteMapper.class);
  //通过上述mapper调用接口中的函数
  List<Website> websiteList = websiteMapper.selectAllWebsite();
  ```

#### 总结

+ 利用xxxMapper.xml中的sql语句实现了xxxMapper接口中的函数，最后直接通过接口类型调用实例化的函数实现增删改查的功能
+ 一般来说 ，不会通过反射用sqlsession去获取xxxMapper类，而是利用spring属性注入（bean管理）获取xxxMapper类

### 注解实现映射器

#### 定义

+ 使用注解的方式实现映射器，只需要在接口中使用 Java 注解，注入 SQL 即可

#### 流程

+ 首先定义接口

  ```java
  package net.biancheng.mapper;
  import java.util.List;
  import net.biancheng.po.Website;
  public interface WebsiteMapper {
      public List<Website> selectAllWebsite();
    
    	public Website selectOneWebsite();
  }
  ```

+ 接着为接口中方法添加注解

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

#### 总结

+ 如果使用注解和 XML 文件两种方式同时定义，那么 XML 方式将覆盖掉注解方式
+ 虽然这里注解的方式看起来比 XML 简单，但是现实中我们遇到的 SQL 会比该例子复杂得多，如果 SQL 语句中有多个表的关联、多个查询条件、级联、条件分支等，显然这条 SQL 就会复杂的多，所以并不建议读者使用这种方式

<div align="right"><font size="4.5">8月3日</div>

## MyBatis动态SQL

+ 动态SQL是MyBatis的强大特性之一，在JDBC或其他类似的框架中，开发人员需要手动拼接SQL语句

### 常用标签

#### if标签

- 使用方法：

  ```xml
  <if test = "判断条件">
  	SQL语句
  </if>
  ```

- 执行流程：当判断条件为true时，才会执行所包含的SQL语句

- e.g.

  ```xml
  <select id = "selectAllWebsite" resultMap = "myResult">
  	select * from website
    <if test = "name != null">
      where name like #{name}
    </if>
  </select>
  ```

+ choose、when和otherwise标签

  - 效果：由于 MyBatis 并没有为 if 提供对应的 else  标签，如果想要达到、\<if>...\<else>...\</else> \</if> 的效果，可以借助   \<choose>、\<when>、\<otherwise> 来实现

#### set标签

#### foreach标签

- 作用：用于循环语句，并且支持类List、Set接口等集合

- 语法：

  ```xml
  <foreach item = "item" index = "index" collection = "list|array|map key" open = "(" separator = "," close = ")">
  	参数
  </foreach>
  ```

  

- 属性说明

  * item：表示集合中每一个元素进行迭代时的别名
  * index：指定一个名字，表示在迭代过程中每次迭代到的位置
  * open：表示该语句可以以什么开始
  * separator：表示每次进行迭代之间以什么符号作为分隔符
  * close：表示该语句以什么结束

#### bind标签

- 作用：可以使用 OGNL 表达式创建一个变量井将其绑定到上下文中
- 属性说明：
  * value：对应传入实体类的某个字段，可以进行字符串拼接等特殊处理
  * name：给对应参数取的别名

#### trim标签

- 作用：
  * 实现多条件查询
  * 可以选择性地插入、更新、删除或条件查询
- 属性说明：
  * prefix
  * suffix
  * prefixOverrides
  * suffixOverrides

### 其他

+ @Param注解在mapper中的使用

  - 目的：为参数去别名，简化参数引用

  - 使用：方法（@Param（“别名”）Type  参数）

  - 例子：

    ``` java
     public int getUsersDetail(@Param("userid") int userid);
    ```

    ```sql
    <select id = "getUserDetail" resultMap = "baseMap">
    	select * from xxx
    </select>
    ```

+ 

