title: springmvc vs restlet
date: 2016-01-25 13:49:36
tags: springmvc restlet
categories: spring
---
本文将对springmvc和restlet进行对比。
<!--more-->
<h2>rest和restfull web services</h2>
REST是表述性状态转移（REpresentational State Transfer）的简称。REST是Roy T. Fielding在他的博士学位论文中定义的术语，REST是一种设计风格而不是标准。
REST关键原则为：
1.为所有“事物”定义ID
2.将所有事物链接在一起
3.使用标准方法
4.资源多重表述
5.无状态通信
RESTful Web Service 是一个使用HTTP并遵循REST原则的Web服务。它从三个方面资源进行定义：
1.URI，比如：http://example.com/resources/。
2.Web Service接受与返回的互联网媒体类型，比如：JSON，XML等。
3.Web Service在该资源上所支持的一系列请求方法（比如：POST，GET，PUT或DELETE）。
<h2>restlet</h2>
restlet宣称支持REST的轻量级Java框架。
特点：完全抛弃了Servlet API，作为替代，自己实现了一套API。能够支持复杂的REST架构设计。
缺点：
1). 虽然也可以运行于Web容器中，但是难以利用Servlet和JSP等资源。因为需要另外学习一套API和概念，学习成本比较高。 
2). 完全不支持服务器端的HTTP Session，强制完全基于无状态服务器模型来做开发。对于基于浏览器的应用来说，开发难度较高。 
3). 自身没有包括与Spring的集成，可以使用第三方代码与Spring集成，集成难度较大。 
4). 文档不是很丰富，大多比较简短，很多时候需要自己去读代码，或者到其wiki上去查找。 
5). 没有内建的国际化支持。
6). 一个Resource一般只能配置一个Get或者一个Post，若要配置多个Get或者Post可以分开多个文件或者进行复杂的配置。
7). 需要额外的配置文件配置URL映射：
例如
{% codeblock lang:java %}
<entry key="/admin/machine/maintainOp.json">
    <bean class="org.restlet.ext.spring.SpringFinder">
        <lookup-method name="create" bean="opMaintainLogResource"/>
    </bean>
</entry>
{% endcodeblock %}
映射路径统一在一个文件中易于管理但是却不易于维护，找到Resource中相应的实现代码相对麻烦。
优点： 
1). 有内建的HTTP认证机制，不需要另外开发安全机制。 
2). 灵活性较高，支持更多的REST概念，支持透明的内容协商，适合于开发更加强大的REST组件（不限于服务器端应用）。
3). 零配置文件，全部配置通过代码来完成。
<h2>springMVC</h2>
在SpringMVC 中，控制器Controller 负责处理由DispatcherServlet 分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。在SpringMVC 中提供了一个非常简便的定义Controller 的方法，你无需继承特定的类或实现特定的接口，只需使用@Controller 标记一个类是Controller ，然后使用@RequestMapping 和@RequestParam 等一些注解用以定义URL 请求和Controller 方法之间的映射，这样的Controller 就能被外界访问到。
springMVC对restfull的支持：将URI和HTTP请求方法映射到JAVA处理方法，并将JAVA方法处理结果返回给HTTP请求者（对应资源定义1和3）。@RequestMapping是最重要的一个注解，用于处理HTTP请求地址映射，可用于类或方法上，用于类上时表示类中的所有响应请求的方法都是以该地址作为父路径，在Spring中，一般一个Controller类处理一种资源，所以每个Controller类都会加@RequestMapping注解。
SpringMVC优点：
1.springMVC比较接近源生的servlet，灵活度高，controller是单例的。
2.文档丰富。
3.与spring天然集成配置简单。
4.Spring MVC 框架并不知道使用的视图，所以不会强迫您只使用 JSP 技术或者其他前端技术。Spring MVC 分离了控制器、模型对象、分派器以及处理程序对象的角色，这种分离让它们更容易进行定制。是典型的教科书式的MVC架构。
5.一个Controller可以处理一个功能模块的所有请求。配置@RequestMapping即可。