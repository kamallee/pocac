title: DispatcherServlet
date: 2017-08-02 11:26:22
tags: DispatcherServlet spring srpingMVC
categories: spring
---
<h2>前端控制器</h2>
spring mvc也是依赖servlet，所以spring mvc的请求处理是从一个servlet开始，这个servlet就是DispatcherServlet。
前端控制器模式（Front Controller Pattern）是用来提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理。该处理程序可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。
前端控制器（Front Controller） - 处理应用程序所有类型请求的单个处理程序，应用程序可以是基于 web 的应用程序，也可以是基于桌面的应用程序。
调度器（Dispatcher） - 前端控制器可能使用一个调度器对象来调度请求到相应的具体处理程序。
视图（View） - 视图是为请求而创建的对象。

前端控制器也是表示层的设计模式，它的出现主要是由于表示层通常需要控制和协调来自不同用户的多个请求，而这种控制机制又根据不同的需要，可能会集 中式控制或分散式控制。换句话说，就是应用系统需要对于表示层的请求提供一个集中式控制模块，以提供各种系统服务，包括内容提取、视图管理和浏览，如果系 统中没有这种集中式控制模块或控制机制，每个不同的系统服务都需要进行单独的视图处理，这样代码的重复性就会提高，致使系统开发代价提高。同时，如果没有 一个固定模块管理视图之间的浏览机制，致使其浏览功能下放于每个不同的视图中，最终必将使得系统的可维护性受到破坏。本文中我们主要讨论的是集中式控制模 块，而不是分散式控制，因为前者更适合于大型的应用系统。
<!--more-->
<h2>什么是DispatcherServlet。它是</h2>
如果经常与 Spring MVC 打交道，那么很有必要了解什么是 DispatcherServlet。它是 Spring MVC 的核心，准确的说就是 MVC 设计模式中的 C 或 Controller。每个由 Spring MVC 处理的请求都要经过 DispatcherServlet。一般而言，它是前端控制器模式的实现，为应用提供一个统一入口。DispatcherServlet 是连接 Java 与 Spring 的桥梁,处理所有传入的请求。并且与其他声明在 web.xml 中的 Servlet 一样，也是通过一个 URL pattern 将每个请求映射到 DispatcherServlet。

DispatcherServlet 负责将请求委派给 Spring MVC 中其他的组件处理，比如注有 @Controller 或 @RestController 的 Controller类，Handler Mappers(处理器映射)，View Resolvers(视图解析器) 等等。

尽管，请求映射是由 @ResquestMapping 注解完成的，但实际上是由 DispatcherServlet 将请求委派给相应的 Controller 来处理的。

在 RESTFul 的 web 服务中， DispatcherServlet 还负责选择正确的信息转换器，以便将响应结果转换成客户端期望的格式(JSON, XML 或 TEXT)。比如，如果客户端期望 JSON 格式，那么会使用 MappingJacksonHttpMessageConverter 或 MappingJackson2HttpMessageConverter (取决于 classpath 中可用的是 Jackson 1 还是 Jackson 2) 来将响应结果转成 JSON 字符串的格式。

DispatcherServlet 如何处理请求
正如上面所说，DispatcherServlet 被用来处理所有传入的请求，并将它们路由到不同的 Controller 来进行进一步处理。它决定了由哪个 Controller 处理请求。

DispatcherServlet 使用处理器映射来将传入的请求路由到处理器。默认情况下，使用 BeanNameUrlHandlerMapping 和 由 @RequestMapping 注解驱动的DefaultAnnotationHandlerMapping。

为了找到正确的方法来处理请求，它会扫描所有声明了 @Controller 注解的类，并且通过 @RequestMapping 注解找到负责处理该请求的方法。@RequestMapping 注解可以通过路径来映射请求(比如: @RequestMapping(“path”)), 也可以通过 HTTP 方法(比如: @RequestMapping("path", method=RequestMethod.GET)), 也可以通过请求参数(比如: @RequestMapping("path"”, method=RequestMethod.POST, params="param1”)),还可以通过 HTTP 请求头(比如: @RequestMapping("path", header="content-type=text/*”))。我们也可以在类级别声明 @RequestMapping 注解来过滤传入的请求。

在请求处理之后，Controller 会将逻辑视图的名字和 model 返回给 DispatcherServlet。之后利用视图解析器定位到真正的 View 以便渲染结果。我们可以指定使用的视图解析器，默认情况下，DispatcherServlet 使用 InternalResourceViewResolver来将逻辑视图的名字转换成真正的视图，比如 JSP。

选定视图之后，DispatcherServlet 会将数据模型与视图相结合，并将结果返回给客户端。并不是任何时候都需要视图，比如一个 RESTful 的 web 服务就不需要，它们的处理方法会利用 @ResponseBody 注解直接将请求结果返回给客户端。可以看REST with Spring course了解更多关于如何使用 Spring MVC 开发和测试 RESTful 服务的知识。

总结
在这篇文章中，我分享了一些关于 DispatcherServlet 比较重要的一些知识点。这些不仅可以帮助大家更好的理解 DispatcherServlet，也可以鼓励大家进一步去学习相关的知识。

DispatcherServlet 是 Spring MVC 应用中主要的控制器。所有的请求都会先经由 DispatcherServlet 处理，再由 Controller (声明有 @Controller 注解的类) 处理。
DispatcherServlet 是前端控制器模式的实现。前端控制器就是个用来处理网站所有请求的控制器。
就像其他的 Servlet， DispatcherServlet 也是声明和配置在 web.xml 文件中的：

{% codeblock lang:java %}
<web-app> 
<servlet> 
<servlet-name>SpringMVC</servlet-name> 
<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
<load-on-startup>1</load-on-startup> 
</servlet> 
<servlet-mapping> 
<servlet-name>SpringMVC</servlet-name> 
<url-pattern>*</url-pattern> 
</servlet-mapping> 
</web-app>
{% endcodeblock %}
DispatcherServlet 继承自 HttpServlet 基类。Servlet 引擎(比如 Tomcat) 创建该类的实例，并且调用它不同的方法，比如：init(), service(), destroy()。

DispatcherServlet 为 Spring MVC 应用提供统一入口，处理所有的请求。
DispatcherServlet 也完全与 Spring IoC 容器集成，可以使用 Spring 框架的每一个特性，比如依赖注入。
当 DispatcherServlet 被配置为 load-on-startup = 1,意味着该 servlet 会在启动时由容器创建，而不是在请求到达时。这样做会降低第一次请求的响应时间，因为DispatcherServlet 会在启动时做大量工作，包括扫描和查找所有的 Controller 和 RequestMapping。
在 DispatcherServlet 初始化期间，Spring 框架会在 WEB-INF 文件夹中查找名为 [servlet-name]-servlet.xml 的文件，并创建相应的 bean。比如，如果 servlet 像上面 web.xml 文件中配置的一样，名为 “SpringMVC”，那么会查找 “SpringMVC-Servlet.xml”的文件。如果全局作用域中有相同名字的bean，会被覆盖。可以用 servlet 初始化参数 contextConfigLocation更改配置文件的位置。

{% img /img/dispatcherServlet.png 700 350 Dispatcher Servlet flow %} 

在 Spring MVC 框架中，每个 DispatcherServlet 都有它自己的 WebApplicationContext ，并且继承了根 WebApplicationContext 中定义的所有 bean。这些继承的 bean 在 servlet 指定的作用域中可以被重载，也可以在其指定作用域中定义新的 bean。
Spring MVC 中的 DispatcherServlet也允许返回 Servlet API 定义的 last-modification-date。为了决定请求最后修改时间，DispatcherServlet会先查找合适的 handler mapping，然后检测处理器是否实现了 LastModified 接口。如果实现了，就调用接口的 getLastModified(request) 方法，并将该值返回给客户端。
以上就是关于 DispatcherSerlvet 的内容。正如上面所讲，DispacherServlet 是 Spring MVC 的骨干，是主要的控制器，用来将不同的 HTTP 请求路由当相应的 Controller。它是前端控制器设计模式的实现，并且为应用提供单一入口。可以在 web.xml 中配置 DispatcherServlet，但建议将 load-on-startup 设置为 1。这样容器会在启动时加载该 Serlvet 而不是请求到达时。这样能减少第一个请求的响应时间。

参考：
https://yemengying.com/2017/10/07/spring-dispatcherServlet/