title: AOP技术
date: 2015-08-27 10:16:51
tags: spring 
categories: spring
---
<h2>什么事AOP</h2>
AOP（Aspect-Oriented Programming，面向方面编程），可以说是OOP的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当需要为分散的对象引入公共行为时，OOP则显得无能为力。例如日志功能，日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。这种散布在各处的无关的代码被称为横切（cross-cutting）代码，在OOP设计中，它导致了大量代码的重复，不利于各个模块的重用。
<!--more-->
AOP技术则相反，它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天工的妙手将这些剖开的切面复原，不留痕迹。

AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处都基本相似。比如权限认证、日志、事务处理。Aop 的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

<h2>AOP实现技术</h2>
实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。然而殊途同归，实现AOP的技术特性却是相同的，分别为：

1、join point（连接点）：是程序执行中的一个精确执行点，例如类中的一个方法。它是一个抽象的概念，在实现AOP时，并不需要去定义一个join point。
2、point cut（切入点）：本质上是一个捕获连接点的结构。在AOP中，可以定义一个point cut，来捕获相关方法的调用。
3、advice（通知）：是point cut的执行代码，是执行“方面”的具体逻辑。
Action taken at a particular join point. 即 Joinpoint 横且逻辑执行的时机
{% blockquote %}
3.1 before advice
在 Jointpoint 指定位置之前执行, 对于 Spring AOP 来说, 就是在某个方法执行之前执行
3.2 after advice
在 Jointpoint 指定位置之后执行, 对于 Spring AOP 来说, 就是在某个方法执行之后执行
又可分为三种, 分别是
	after returning advice: 比如方法正常返回之后执行, 是正常返回而没有抛出异常
	after advice: 比如方法返回之后执行, 无论是否抛出异常, 定义成 afer finally advice 更合适
	after throwing advice: 比如当方法执行出现异常时执行
3.3 around advice
顾名思义, 是围绕这 Joinpoint 前后, 比如在方法执行前后都可以执行, 所以同时兼有 before advice 和 after advice 的功能
Spring AOP 的 around advice 实现是利用 AOP Alliance 下的拦截器(Interceptor), 需要实现 MethodInterceptor 接口
3.4 introduction
{% endblockquote %}
4、aspect（方面）：point cut和advice结合起来就是aspect，它类似于OOP中定义的一个类，但它代表的更多是对象间横向的关系。
5、introduce（引入）：为对象引入附加的方法或属性，从而达到修改对象结构的目的。有的AOP工具又将其称为mixin。
上述的技术特性组成了基本的AOP技术，大多数AOP工具均实现了这些技术。它们也可以是研究AOP技术的基本术语。
<h2>Spring AOP与AspectJ的关系</h2>
Spring AOP是动态代理，实现只有JDK动态代理和CGLIB代理两种方式，那我们为什么要引入AspectJ呢，因为Spring使用AspectJ的配置方式。AspectJ是静态代理。Spring 允许使用 AspectJ Annotation 用于定义方面（Aspect）、切入点（Pointcut）和增强处理（Advice），Spring 框架则可识别并根据这些 Annotation 来生成AOP 代理,但是底层实现还是使用Spring AOP，并不依赖AspectJ的编译器或者织如器（Weaver）。
<h2>Spring AOP详解</h2>
Spring 的 AOP 代理由 Spring 的 IoC 容器负责生成、管理，其依赖关系也由 IoC 容器负责管理。
Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。

Spring IOC容器初始化的时候就会创建代理对象，并由IOC容器负责管理。
代理对象从DefaultAopProxyFactory类的方法生成。
{% codeblock lang:java %}
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        if(!config.isOptimize() && !config.isProxyTargetClass() && !this.hasNoUserSuppliedProxyInterfaces(config)) {
            return new JdkDynamicAopProxy(config);
        } else {
            Class targetClass = config.getTargetClass();
            if(targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: Either an interface or a target is required for proxy creation.");
            } else {
                return (AopProxy)(!targetClass.isInterface() && !Proxy.isProxyClass(targetClass)?
                new ObjenesisCglibAopProxy(config):new JdkDynamicAopProxy(config));
            }
        }
    }
{% endcodeblock %}
从代码可以看出Spring提供了两种代理对象生成方式，一种是JDK动态代理，一种是CGLIB动态代理。

<h2>JDK动态代理</h2>
{% codeblock lang:java %}
public Object getProxy(ClassLoader classLoader) {
        if(logger.isDebugEnabled()) {
            logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
        }

        Class[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised);
        this.findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
 //实现了InvocationHandler接口
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object oldProxy = null;
        boolean setProxyContext = false;
        TargetSource targetSource = this.advised.targetSource;
        Class targetClass = null;
        Object target = null;

        Object var13;
        try {
            if(!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
                Boolean retVal1 = Boolean.valueOf(this.equals(args[0]));
                return retVal1;
            }

            if(!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
                Integer retVal2 = Integer.valueOf(this.hashCode());
                return retVal2;
            }

            Object retVal;
            if(!this.advised.opaque && method.getDeclaringClass().isInterface() && method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                retVal = AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
                return retVal;
            }

            if(this.advised.exposeProxy) {
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
            }

            target = targetSource.getTarget();
            if(target != null) {
                targetClass = target.getClass();
            }
            //获取匹配targetClass与method的所有切面的通知
            List chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
            if(chain.isEmpty()) {
            	//直接执行目标对象方法
                retVal = AopUtils.invokeJoinpointUsingReflection(target, method, args);
            } else {

                ReflectiveMethodInvocation invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
            }

            Class returnType = method.getReturnType();
            if(retVal != null && retVal == target && returnType.isInstance(proxy) && !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                retVal = proxy;
            } else if(retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
                throw new AopInvocationException("Null return value from advice does not match primitive return type for: " + method);
            }

            var13 = retVal;
        } finally {
            if(target != null && !targetSource.isStatic()) {
                targetSource.releaseTarget(target);
            }

            if(setProxyContext) {
                AopContext.setCurrentProxy(oldProxy);
            }

        }

        return var13;
    }
{% endcodeblock %}

<h2>CGLIB动态字节码生成</h2>
CGLIB具有简单易用，它的运行速度要远远快于JDK的Proxy动态代理。
动态字节码生成技术是对目标对象进行继承扩展, 为其生成相应的子类, 而子类可以通过重写来扩展父类的行为。
{% codeblock lang:java %}
//定义拦截器，调用目标方法之前、调用目标方法之后织入增强处理
public class AroundAdvice implements MethodInterceptor {
	//参数：Object为由CGLib动态生成的代理类实例，Method为上文中实体类所调用的被代理的方法引用，Object[]为参数值列表，MethodProxy为生成的代理类对方法的代理引用。
    public Object intercept(Object target, Method method, Object[] args, MethodProxy proxy) throws java.lang.Throwable {
        System.out.println("执行目标方法之前...");
        // 执行目标方法，并保存目标方法执行后的返回值
        Object rvt = proxy.invokeSuper(target, new String[] { "被改变的参数,我" });
        System.out.println("执行目标方法之后...");
        return rvt + " 新增的内容";
    }
}

//生成代理对象方法
public class MonkeyProxyFactory
{
    public static Monkey getAuthInstance()
    {
        Enhancer en = new Enhancer();
        // 设置要代理的目标类
        en.setSuperclass(Monkey.class);
        // 设置要代理的拦截器
        en.setCallback(new AroundAdvice());
        // 生成代理类的实例
        return (Monkey)en.create();
    }
}

//使用方法
Monkey monkey = MonkeyProxyFactory.getAuthInstance();
monkey.eatPeaches("xxxx");

代码
protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
        Class proxyClass = enhancer.createClass();
        Object proxyInstance = null;
        if(objenesis.isWorthTrying()) {
            try {
                proxyInstance = objenesis.newInstance(proxyClass, enhancer.getUseCache());
            } catch (Throwable var7) {
                logger.debug("Unable to instantiate proxy using Objenesis, falling back to regular proxy construction", var7);
            }
        }

        if(proxyInstance == null) {
            try {
                proxyInstance = this.constructorArgs != null?proxyClass.getConstructor(this.constructorArgTypes).newInstance(this.constructorArgs):proxyClass.newInstance();
            } catch (Throwable var6) {
                throw new AopConfigException("Unable to instantiate proxy using Objenesis, and regular proxy instantiation via default constructor fails as well", var6);
            }
        }

        ((Factory)proxyInstance).setCallbacks(callbacks);
        return proxyInstance;
    }
{% endcodeblock %}

<h2>实例</h2>
spring aspectj aop 简单实例
pom引入的相关jar
{% codeblock lang:xml %}
<dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.6</version>
        </dependency>
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.1</version>
        </dependency>
{% endcodeblock %}

spring配置
{% codeblock lang:xml %}
<!-- 定义Aspect -->
    <bean id="guardian" class="com.crazyfish.kamal.test.testaop.Guardian" />
    <bean id="monkey" class="com.crazyfish.kamal.test.testaop.Monkey" />

    <!-- 启动Spring对AspectJ支持 -->
    <aop:aspectj-autoproxy />
{% endcodeblock %}

自定义注解类
{% codeblock lang:java %}
package com.crazyfish.kamal.test.testaop;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
//自定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface BusinessAnnotation {
    String moduleName();
    String option();
}
{% endcodeblock %}

被织入类
{% codeblock lang:java %}
package com.crazyfish.kamal.test.testaop;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Created by kamal on 15/8/27.
 */
public class Monkey {
    @BusinessAnnotation(moduleName="monkey",option="偷桃子")//自定义注解
    public void stealPeaches(String name){
        System.out.println(name+"正在偷桃...");
    }

    public int buyPeaches(String name){
        return 2;
    }

    public void eatPeaches(String name){
        System.out.println(name + "吃了个桃子");
        throw new RuntimeException("出现exception");
    }
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("conf/applicationContext.xml");
        Monkey monkey = (Monkey) context.getBean("monkey");
        try {
            monkey.stealPeaches("lee");
            monkey.buyPeaches("lpp");
            monkey.eatPeaches("kamal");
        }
        catch(Exception e) {}
    }
}
{% endcodeblock %}

aspect 类
{% codeblock lang:java %}
package com.crazyfish.kamal.test.testaop;

import org.aspectj.lang.annotation.*;

@Aspect
public class Guardian{

    @Pointcut("execution(* com.crazyfish.kamal.test.testaop.IPeople.*Peaches(..))")
    public String  foundMonkey(){
        System.out.println("大家好");
        return "hoooo";
    }

    @Before(value="foundMonkey()")//在 Jointpoint 指定位置之前执行
    public void foundBefore(){
        System.out.println("果园有所异动...");
    }

    @After("foundMonkey()")
    public void after(JoinPoint joinPoint) {
        System.out.println("after");
    }
    //返回之后执行
    @AfterReturning("foundMonkey() && @annotation(annotation) && args(name,..)")
    public void foundAfter(String name,BusinessAnnotation annotation){
        System.out.println("这里return了..." + name + " log:" + annotation.moduleName());
    }

    @Around("foundMonkey()")
    public void around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("around start..环绕开始了");
        try {
            pjp.proceed();
        } catch (Throwable ex) {
            System.out.println("error in around");
            throw ex;
        }
        System.out.println("around end");
    }

    @AfterThrowing(pointcut = "foundMonkey()", throwing = "error")
    public void afterThrowing(JoinPoint jp, Throwable error) {
        System.out.println("error:" + error);
    }

}
{% endcodeblock %}

使用
{% codeblock lang:java %}
public class Monkey implements IPeople {
    @BusinessAnnotation(moduleName="monkey",option="偷桃子")//自定义注解
    public void stealPeaches(String name){
        System.out.println(name+"正在偷桃...");
    }

    public String buyPeaches(String name){
        return name;
    }

    public void eatPeaches(String name){
        this.stealPeaches("不错，");
        System.out.println(name + "吃了个桃子");
        //throw new RuntimeException("出现exception");
    }
    public static void main(String[] args) {
        //cglib
        Monkey monkey = MonkeyProxyFactory.getAuthInstance();
        //monkey.eatPeaches("xxxx");

        //JDK代理
        System.out.println("max:" + Integer.MAX_VALUE + ":" + Integer.MIN_VALUE);
        ApplicationContext context = new ClassPathXmlApplicationContext("conf/applicationContext.xml");
        IPeople people = (IPeople) context.getBean("monkey");
        IPeople people1 = new Monkey();
        try {
            people.eatPeaches("YY");
            //people.stealPeaches("lee");
            //people.buyPeaches("lpp");
            //people.eatPeaches("kamal");
        }
        catch(Exception e) {}
    }
}
{% endcodeblock %}
从上面代码执行结果可以看出：
cglib代理执行到目标方法类内，调用this的方法会执行对该方法的增强方法。

jdk动态代理则不会执行。
一旦调用最终抵达了目标对象 （此处为Monkey类的引用），任何对自身的调用例如this.stealPeaches("不错，")将对this引用进行调用而非代理。这一点意义重大， 它意味着自我调用将不会导致和方法调用关联的通知得到执行的机会。
<h2>参考文献</h2>
1.[SpringAOP核心源码分析](http://xsh5324.iteye.com/blog/1846862)
2.[SpringAOP源码分析](http://www.uml.org.cn/j2ee/201301102.asp)