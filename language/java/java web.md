# Java Web

- [Java Web](#java-web)
  - [History](#history)
  - [Servlet](#servlet)
    - [HelloServlet：A simple servlet example](#helloservleta-simple-servlet-example)
    - [Servlet Basis](#servlet-basis)
    - [Filter](#filter)
    - [Listener](#listener)
  - [JSP](#jsp)
    - [Welcome.jsp: A simple JSP example](#welcomejsp-a-simple-jsp-example)
    - [JSP Workflow](#jsp-workflow)
  - [JPA](#jpa)
    - [ORM](#orm)
  - [MVC](#mvc)
    - [J2EE](#j2ee)
    - [Spring MVC](#spring-mvc)
  - [Tomcat](#tomcat)
    - [Standalone Tomcat](#standalone-tomcat)
    - [Embedded Tomcat](#embedded-tomcat)
    - [Maven Tomcat Plugin](#maven-tomcat-plugin)

## History

**CGI & Servlet**：

早期的动态网页技术主要包括CGI（通用网关接口，Common Gateway Interface）和Servlet。其中，CGI通过定义Web服务器与外部程序的标准接口，使得外部程序能够根据Web请求生成动态内容。CGI模式的具体工作流程如图1所示。

![图1 CGI工作流程](https://lushunjian.github.io/blog/2019/01/21/web%E5%BC%80%E5%8F%91%E6%8A%80%E6%9C%AF%E7%9A%84%E6%BC%94%E5%8F%98/CGI.png)

相比之下，尽管两者的整体工作架构类似，但Servlet模式把服务器中接收请求和返回响应的部分封装起来了，免去了开发者重复造轮子的工作。开发者只需要根据JDK中Servlet API实现业务逻辑即可。换句话说，开发者编写的Servlet程序其实就类似CGI模式下的外部程序。此外，Servlet拥有更好的可移植性和更高的运行效率，因为Servlet对象运行在JVM上且由Servlet容器管理，可以持久驻留在内存中为多个请求服务，而不需要像CGI一样每次请求都启动一个新进程来执行外部程序。

**JSP：HTML Template**：

在CGI和Servlet的模式下，服务代码往往和HTML大量杂糅在一起。为了处理更复杂的应用并且方便后期维护，人们开始尝试把HTML返回中固定的部分存起来（我们称之为模版），把动态的部分标记出来。当Web请求处理的时候，程序先获取动态数据，再把模版读入进来，把动态数据填充进去，形成最终返回。因此，各种服务器端脚本语言开始出现:

- ASP(1996) - 微软推出 Active Server Pages，使用 VBScript 等语言来渲染动态页面。
- PHP(1995 发布，2000 年代开始流行) - 简单的服务器端脚本语言，适用于构建动态网站。
- JSP/Servlet(1997) - Sun 公司发布 Java Server Pages，使用 Java 语言开发 Web 应用。

其中，JSP(JavaServer Pages)是一种基于Java的Web开发技术，它使HTML页面具备动态内容生成能力。JSP本质上也是一种Servlet，它将Java代码和HTML标签混合编写，然后由Web容器将其转译为一个Servlet类并执行。

**MVC: J2EE vs ASP.NET**:

随着网络应用复杂度不断提高，尤其是在构造大型应用是在可扩展性、容错性等方面的需求催生了J2EE和ASP.NET的平台。事实上，前文所提到的Java Servlet和JSP正是J2EE中的核心组件。

这个时期，人们开始考虑将Servlet和JSP的优势结合起来，分别使用Servlet和JSP实现各种复杂的业务逻辑和编写HTML，也就是后续在Web开发中广泛应用的MVC模式。MVC模式最早由Trygve Reenskaug在1978年提出。它是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：

- 模型（Model）：负责数据访问和算法实现；
- 视图（View）：负责界面显示；
- 控制器（Controller）：负责处理请求。

在J2EE Web应用的体系中，Servlet和JSP分别承担了Controller和View的职责，而Model则是由J2EE的另一个重要组件EJB（Enterprise Java Bean）负责。类似的，由微软推出的ASP.NET平台也同样采用了MVC架构模式。

**JavaScript Popularity And AJAX Occurrence ：Frontend Backend Split**：

随着JavaScript在浏览器中的广泛应用以及AJAX(Asynchronous JavaScript and XML)技术的出现，Web应用程序架构发生了重大转变。因为JavaScript能够直接操作DOM局部更新页面内容，而AJAX技术又使得其能与服务器进行异步通信获取所需数据。这种新模式逐渐取代了传统的JSP技术，改善了服务端大包大揽的状况，使得交互和显示的任务从服务端被剥离出来，进一步形成了前端负责界面显示和用户交互而后端专注于业务逻辑处理的现代Web应用架构。

**More Framework**:

此后，前后端分离的格局形成，大量Web框架涌现。在前端领域，Vue和React等现代JavaScript框架应运而生，提供了更高效、组件化的方式构建用户界面。与此同时，Node.js的出现使JavaScript可以运行在服务器端，开发人员能够使用同一种语言编写前后端代码，提高了开发效率，也催生了以MERN（MongoDB+Express+Node+React）等为代表的全栈工程师。

在后端领域，各种语言的框架百花齐放。其中，Python有Flask和Django，JavaScript有Express，而在Java阵营中，Spring框架则凭借其轻量、模块化的设计，成为了最主要和流行的企业级Web应用开发框架之一。

## Servlet

Java Servlet的全称是Java Server Applet，也就是Java服务器小程序。利用它开发者能根据业务逻辑动态生成内容返回给客户端。

Servlet技术是2000年代重要的动态网页技术之一，直至今天也仍然作为大型Java Web框架的中重要组成部分而存在（Spring的核心组件DispatcherServlet）。因此，了解Servlet工作原理是学习Java Web是绕不开的话题。

### HelloServlet：A simple servlet example

一个最简单的HelloServlet程序如下：

```java
public class HelloServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
        res.setContentType("text/html");
        //We need printwriter object to write html content
        PrintWriter pw = res.getWriter();

        // writing html in the stream
        pw.println("<html><body>");
        pw.println("Welcome to my first servlet");
        pw.println("</body></html>");

        pw.close();// close the stream
    }
}
```

将它打包并部署到Tomcat的webapps目录中，并修改Tomcat的web.xml配置文件如下。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <display-name>Java Creed | How To Run Embedded Tomcat with Maven</display-name>

    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>org.example.HelloServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

最终，我们可以得到图2所示效果。

![图2 HelloServlet效果](../img/hello_servlet.png)

### Servlet Basis

**Servlet API**:

在这个示例中，HelloServlet类继承于HttpServlet类，并实现了doGet方法来处理HTTP GET请求。而HttpServlet正是一个重要的Servlet接口类，可见Servlet程序就是使用JDK中Servlet API根据业务逻辑实现的一个类。

Java Servlet API是一套用于在服务器上实现响应请求的标准。它由javax.servlet和javax.servlet.http两个核心包组成，定义了Servlet组件的生命周期方法和客户端请求响应处理等规范。常用的Servlet接口和类包括:

- GenericServlet类
- HttpServlet类
- Request对象
- Response对象
- ServletConfig对象
- ServletContext对象

**Servlet Container**:

Servlet程序并没有main函数，需要部署在Servlet容器(如Tomcat)中运行。Servlet容器负责创建、管理和销毁Servlet对象，并根据HTTP请求调用相应的Servlet程序。整个调用过程如图3所示:

![图3 Tomcat调用](https://picx.zhimg.com/v2-ce6e39bb02e3c6a2f4eb1e5afaa6e4e6_r.jpg?source=2c26e567)

除了提供通信支持和资源管理之外，使用Servlet容器还提供多线程支持。容器会自动为它所接收的每个Servlet请求创建一个新的线程，并且当运行完相应的http服务方法，这个线程就会自动结束。当然，这并不是说开发者就不再不需要考虑线程安全性，只不过这样能减少做很多重复的工作。

**Servlet Lifecycle**:

查看JDK中提供的Servlet接口我们发现，其中最核心的就是三个函数init、service和destroy，这也正好对应了一个Servlet对象的生命周期。

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

以下是 Servlet 遵循的过程：

Servlet 初始化后调用 init () 方法。
Servlet 调用 service() 方法来处理客户端的请求。
Servlet 销毁前调用 destroy() 方法。
最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

### Filter

在一个比较复杂的Web应用程序中，通常都有很多URL映射，对应的，也会有多个Servlet来处理URL。假设现在有一个论坛Web 应用，它有5个Servlet。其中，ProfileServlet、PostServlet和ReplyServlet都需要用户登录后才能操作，否则，应当直接跳转到登录页面。我们可以直接把判断登录的逻辑写到这3个Servlet中，但是，同样的逻辑重复3次没有必要，并且，如果后续继续加Servlet并且也需要验证登录时，还需要继续重复这个检查逻辑。

为了把一些公用逻辑从各个Servlet中抽离出来，JavaEE的Servlet规范还提供了一种Filter组件，即过滤器，它的作用是，在HTTP请求到达Servlet之前，可以被一个或多个Filter预处理，类似打印日志、登录检查等逻辑，完全可以放到Filter中。

### Listener

除了Servlet和Filter外，JavaEE的Servlet规范还提供了第三种组件：Listener。Listener顾名思义就是监听器，用于监听Web应用中的事件，以实现一些全局操作。如监听Web应用的启动和停止，会话对象的创建和销毁等。比如，下面这个例子就实现了一个ServletContextListener接口的类。

```java
@WebListener
public class AppListener implements ServletContextListener {
    // 在此初始化WebApp,例如打开数据库连接池等:
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("WebApp initialized.");
    }

    // 在此清理WebApp,例如关闭数据库连接池等:
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("WebApp destroyed.");
    }
}
```

任何标注为@WebListener，且实现了特定接口的类会被Web服务器自动初始化。上述AppListener实现了ServletContextListener接口，它会在整个Web应用程序初始化完成后，以及Web应用程序关闭后获得回调通知。我们可以把初始化数据库连接池等工作放到contextInitialized()回调方法中，把清理资源的工作放到contextDestroyed()回调方法中，因为Web服务器保证在contextInitialized()执行后，才会接受用户的HTTP请求。

## JSP

正如前面历史介绍中所提到的那样，JSP（Java服务端网页，Java Server Page）是一种基于HTML模板的动态网页技术。早期Servlet程序是将HTML语句拆成一行一行嵌入Java源文件，并用ServletResponse中PrintWriter写入返回响应之中，而JSP则是在需要动态输出的地方使用JSP标签将Java代码嵌入HTML文件中。

### Welcome.jsp: A simple JSP example

有一个简单的Welcome.jsp内容如下，将其放置在Tomcat的webapps目录下。其中，out是JSP内置的变量，表示当前HttpServletResponse的PrintWriter。

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1><%
        out.println("Hello "+request.getParameter("name")+"!");
    %></h1>
</body>
</html>
```

接着，访问[http://localhost:9090/welcome.jsp?name=Zhytou](http://localhost:9090/welcome.jsp?name=Zhytou)这个链接，即可得到如图4所示效果。

![图4 Welcome.jsp](../img/welcome_jsp.png)

### JSP Workflow

JSP的工作流程是怎么样的呢？我们可以在Tomcat的工作目录下找到一个名为welcome_jsp.java的文件，其内容如下：

```java
/*
 * Generated by the Jasper component of Apache Tomcat
 * Version: Apache Tomcat/7.0.47
 * Generated at: 2024-05-07 17:05:35 UTC
 * Note: The last modified time of this file was set to
 *       the last modified time of the source file after
 *       generation to assist with modification tracking.
 */
package org.apache.jsp;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.jsp.*;

public final class welcome_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent {

  private static final javax.servlet.jsp.JspFactory _jspxFactory =
          javax.servlet.jsp.JspFactory.getDefaultFactory();

  private static java.util.Map<java.lang.String,java.lang.Long> _jspx_dependants;

  private javax.el.ExpressionFactory _el_expressionfactory;
  private org.apache.tomcat.InstanceManager _jsp_instancemanager;

  public java.util.Map<java.lang.String,java.lang.Long> getDependants() {
    return _jspx_dependants;
  }

  public void _jspInit() {
    _el_expressionfactory = _jspxFactory.getJspApplicationContext(getServletConfig().getServletContext()).getExpressionFactory();
    _jsp_instancemanager = org.apache.jasper.runtime.InstanceManagerFactory.getInstanceManager(getServletConfig());
  }

  public void _jspDestroy() {
  }

  public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
        throws java.io.IOException, javax.servlet.ServletException {

    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;


    try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response, null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write("\r\n");
      out.write("\r\n");
      out.write("<html>\r\n");
      out.write("<head>\r\n");
      out.write("    <title>Welcome</title>\r\n");
      out.write("</head>\r\n");
      out.write("<body>\r\n");
      out.write("    <h1>");

        out.println("Hello "+request.getParameter("name")+"!");
    
      out.write("</h1>\r\n");
      out.write("</body>\r\n");
      out.write("</html>\r\n");
    } catch (java.lang.Throwable t) {
      if (!(t instanceof javax.servlet.jsp.SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          try { out.clearBuffer(); } catch (java.io.IOException e) {}
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
        else throw new ServletException(t);
      }
    } finally {
      _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }
}

```

可见JSP文件会在执行前首先被编译成一个Servlet程序，所以我们访问welcome.jsp时其实是在访问welcome_jsp.java中的Servlet。

## JPA

JPA(Java Persistence API)是一种Java规范，旨在简化Java对象与关系数据库之间的持久化操作。它提供了一种对象关系映射(Object-Relational Mapping，ORM)的标准规范，使开发者能够以面向对象的方式操作关系数据库中的数据，而不必直接编写冗长且易出错的SQL语句。JPA定义了一组注解和接口，用于描述对象与数据库表之间的映射关系，以及执行持久化操作(如创建、查询、更新和删除)所需的API。它为Java应用程序提供了一层抽象，屏蔽了底层数据库系统的差异，提高了代码的可移植性和可维护性。

### ORM

对象关系映射（ORM）是一种通过将对象状态映射到数据库列来开发和维护对象与关系数据库之间的关系的功能。 它能够轻松处理各种数据库操作，例如插入、更新、删除等。

## MVC

MVC(模型-视图-控制器)是一种软件架构模式，将应用程序划分为模型(Model)、视图(View)和控制器(Controller)三个部分，各自负责不同的功能。Java Web开发中常采用这种模式，比如JavaBean充当模型，JSP充当视图，Servlet充当控制器。

### J2EE

**Java vs J2EE**:

![图5 J2EE架构](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS8yMjljZjlmZjViMTcyOWVhZjQwOGZhYzU2MjM4ZWViM19iLnBuZw)

### Spring MVC

基于Servlet/JSP规范，Java界出现了众多优秀的Web框架，提供更高层次的抽象，降低开发难度，如:

Struts - 老牌Web框架，支持MVC模式
WebWork - 经典的MVC框架
Spring MVC - 目前最流行的MVC框架
JSF - 基于组件的MVC框架
这些框架在Servlet/JSP技术之上，提供更多高级功能，扩展性更好，开发效率更高，架构更清晰。在企业级应用中被广泛采用。

## Tomcat

Tomcat是一种轻量级的Web应用服务器，也是Servlet/JSP规范的参考实现。它不只是一个静态资源服务器，更是一个动态资源处理容器，能够管理Servlet/JSP应用程序的生命周期。

### Standalone Tomcat

### Embedded Tomcat

Java Web应用程序总是需要一个可以运行这些应用程序的Tomcat，但是在每台开发机器上都安装和配置一个本地Tomcat会非常耗时。此外，开发人员如果想要运行 Web 应用程序，则需要手动管理依赖项。

Maven 有一个 tomcat 插件，可以让我们运行嵌入式 tomcat 实例，而无需安装本地 tomcat 服务器。

### Maven Tomcat Plugin

**Maven plugin config**：
