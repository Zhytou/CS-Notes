# Java Web

- [Java Web](#java-web)
  - [Servlet](#servlet)
    - [Servlet Api](#servlet-api)
  - [Listener/Filter](#listenerfilter)
  - [JSP](#jsp)
    - [JSP: html template](#jsp-html-template)
  - [Tomcat](#tomcat)
    - [Standalone Tomcat](#standalone-tomcat)
    - [Embedded Tomcat](#embedded-tomcat)
    - [Maven Tomcat Plugin](#maven-tomcat-plugin)
  - [MVC](#mvc)
    - [Java Web Framework](#java-web-framework)

## Servlet

Servlet是Java提供的一种服务器端组件，运行在支持Java的应用服务器上， 用于扩展服务器功能。它可以响应客户端请求，处理业务逻辑，动态生成内容返回给客户端。

### Servlet Api

Servlet API由javax.servlet和javax.servlet.http两个核心包组成，定义了Servlet组件的生命周期方法和客户端请求响应处理等规范。常用的Servlet接口和类包括:

Servlet接口
GenericServlet类
HttpServlet类
Request对象
Response对象
ServletConfig对象
ServletContext对象

## Listener/Filter

Listener用于监听Web应用中的事件，以实现一些全局操作。如监听Web应用的启动和停止，会话对象的创建和销毁等。

Filter是一种实现了Filter接口的Web资源对象，可以对Web资源的请求和响应进行过滤和预处理。通常用于编码转换、数据压缩、日志记录等。

MVC(模型-视图-控制器)是一种软件架构模式，将应用程序划分为模型(Model)、视图(View)和控制器(Controller)三个部分，各自负责不同的功能。Java Web开发中常采用这种模式，比如JavaBean充当模型，JSP充当视图，Servlet充当控制器。

## JSP

JSP(JavaServer Pages)是一种基于Java的Web开发技术，它使HTML页面具备动态内容生成能力。JSP本质上是一种Servlet，它将Java代码和HTML标签混合编写，然后由Web容器将其转译为一个Servlet类并执行。

### JSP: html template

早期的动态网页技术主要包括CGI（通用网关接口，Common Gateway Interface）和Servlet。其中，CGI通过定义Web服务器与外部程序的标准接口，使得外部程序能够根据Web请求生成动态内容，而Servlet则把服务器中接收请求和返回响应的部分封装起来了，开发者免去了造轮子的工作，只需要继承Servelt Api的接口并根据业务逻辑实现其中的doGet和doPost函数。此时，两者架构类似，并且都需要在将HTML和服务代码杂糅在一起。比如，下面这个例子：

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

为了处理更复杂的应用并且方便后期维护，人们在想能不能把HTML返回中固定的部分存起来（我们称之为模版），把动态的部分标记出来，Web请求处理的时候，程序先获取动态数据，再把模版读入进来，把动态数据填充进去，形成最终返回。因此，各种服务器端脚本语言开始出现:

- ASP(1996) - 微软推出 Active Server Pages，使用 VBScript 等语言来渲染动态页面。
- PHP(1995 发布，2000 年代开始流行) - 简单的服务器端脚本语言，适用于构建动态网站。
- JSP/Servlet(1997) - Sun 公司发布 Java Server Pages 和 Servlet 技术，使用 Java 语言开发 Web 应用。

## Tomcat

Tomcat是一种轻量级的Web应用服务器，也是Servlet/JSP规范的参考实现。它不只是一个静态资源服务器，更是一个动态资源处理容器，能够管理Servlet/JSP应用程序的生命周期。

### Standalone Tomcat

### Embedded Tomcat

Java Web应用程序总是需要一个可以运行这些应用程序的Tomcat，但是在每台开发机器上都安装和配置一个本地Tomcat会非常耗时。此外，开发人员如果想要运行 Web 应用程序，则需要手动管理依赖项。

Maven 有一个 tomcat 插件，可以让我们运行嵌入式 tomcat 实例，而无需安装本地 tomcat 服务器。

### Maven Tomcat Plugin

**Maven plugin config**：

## MVC

### Java Web Framework

基于Servlet/JSP规范，Java界出现了众多优秀的Web框架，提供更高层次的抽象，降低开发难度，如:

Struts - 老牌Web框架，支持MVC模式
WebWork - 经典的MVC框架
Spring MVC - 目前最流行的MVC框架
JSF - 基于组件的MVC框架
这些框架在Servlet/JSP技术之上，提供更多高级功能，扩展性更好，开发效率更高，架构更清晰。在企业级应用中被广泛采用。
