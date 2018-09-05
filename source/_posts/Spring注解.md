title: Spring注解
date: 2016-05-19 16:34:02
tags: Annotation Spring
categories: Spring
---

Spring注解的引入可以简化Spring的开发，使用Spring注解需要在XML配置文件中启用Bean的自动扫描功能（<context:component-scan base-package=”com.crazyfish.controller />），Spring在容器初始化的时候将会自动扫描base-package包及其子包下的所有class文件，所有被注解的类将被注册为Spring Bean。
<!--more-->
<h2>常用注解</h2>
(1).Controller，通常作用在控制层，但是目前该功能与 @Component 相同
(2).Service，通常作用在业务层，但是目前该功能与 @Component 相同
(3).Repository
(4).Component，是一个泛化的概念，仅仅表示一个组件 (Bean) ，可以用在任何层次
(5).RestController，Spring4.0引入，继承自Controller
(6).Autowired
(7).Qualifier
(8).Resource，非Spring注解，来自javax.annotation
(9).RequestMapping
(10).ResponseBody
(11).transactional
(12).RequestParam
<h2>Controller&RestController&RequestMapping&RequestParam</h2>
Controller用来表示当前类是控制器Servlet，控制器负责处理DispatchServlet分发的请求，将用户请求的数据经过业务处理层处理之后封装成一个Model，返回给View进行展示。只需要使用RequestMapping和RequestParam等一些注解用以定义URL请求和Controller方法支架的关系，这样控制层中的方法就能被外部访问到。Controller 不会直接依赖于HttpServletRequest 和HttpServletResponse 等HttpServlet 对象，它们可以通过Controller 的方法参数灵活的获取到。

{% codeblock lang:java %}
@Controller
@ResponseBody
public class HistoryBoxController {
	@RequestMapping(value = "/book/list.json")
    public BookDO getBookList (
            @RequestParam(value = "date", required=true) String  date
            @RequestParam(value = "cnt", required=false, defaultValue="20") int cnt) throws Exception {
        return null;
    }
}
{% endcodeblock %}
RequestParam用于绑定HttpServletRequest请求参数到控制器方法参数。

使用RestController，可以开发REST服务的时候不需要使用@Controller而专门的@RestController。当你实现一个RESTful web services的时候，response将一直通过response body发送。为了简化开发，Spring 4.0提供了一个专门版本的controller。如上代码，如果使用Controller则需要增加注解ResponseBody。

<h2>Autowired&Qualifier&Resource</h2>
网上找了个例子http://stackoverflow.com/questions/27937249/why-autowire-bytype-will-also-inject-beans-by-name/27937875#27937875
{% codeblock lang:java %}
public class User {
    private String name;
    public User(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
@Component
public class UserService {
    @Autowired
    private User user1;
    @Autowired
    private User user2;

    public String getNames() {
        return user1.getName() + " & " + user2.getName();
    }
}

import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService service = context.getBean(UserService.class);
        System.out.println(service.getNames());
    }
}
{% endcodeblock %}
{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.1.xsd">
    <context:component-scan base-package="my"/>
    <context:annotation-config/>
    <bean id="user1" class="my.User" autowire="byType">
        <constructor-arg value="Freewind"/>
    </bean>
    <bean id="user2" class="my.User" autowire="byType">
        <constructor-arg value="Lily"/>
    </bean>
</beans>
{% endcodeblock %}
执行并打印：Freewind & Lily
这里的<bean id="user1" class="my.User" autowire="byType">byType表示，如果这个User里引用了别的bean，将会以byType的方式去寻找它们。

UserService里同时@Autowired了两个相同类型的bean，而没有指定名字为什么不会出错？
其实@Autowired执行流程如下：
1.如果某个字段是@Autowired，将会去寻找有没有跟它类型匹配的唯一bean。如果有，直接用它
2.如果发现多个满足条件的，则要看该字段是有没有使用@Qualifier("user1")这样的注解说明到底使用哪个名字的bean。如果有，则使用它
3.如果没有@Qualifier，则以字段名为bean的id，去寻找合适的bean。这是最后一种尝试的手段，也叫fallback
所以为了减少误解，我们还是使用Qualifier进行显示指定需要注入的Bean。

javax.annotation.Resource注解是java标准化后提供的注解，让我们不必依赖于spring。它同时拥有@Autowired和@Qualifier的作用。

<h2>Service</h2>
一般用于业务层

<h2>Repository&Transactional</h2>
@Transactional注解可以标注在类和方法上，也可以标注在定义的接口和接口方法上。
每一个业务方法开始时都会打开一个事务。 Spring默认情况下会对运行期例外(RunTimeException)进行事务回滚。这个例外是unchecked，如果遇到checked意外就不回滚。 
如何改变默认规则： 
1 让checked例外也回滚：在整个方法前加上 @Transactional(rollbackFor=Exception.class) 
2 让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)

Repository都标注在DAO层的类上。
为什么@Repository只能标注在DAO类上呢？这是因为该注解的作用不只是将类识别为 Bean，同时它还能将所标注的类中抛出的数据访问异常封装为 Spring 的数据访问异常类型。 Spring 本身提供了一个丰富的并且是与具体的数据访问技术无关的数据访问异常结构，用于封装不同的持久层框架抛出的异常，使得异常独立于底层的框架。