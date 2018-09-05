title: java设计模式之代理模式
date: 2015-08-06 14:50:57
tags: 设计模式
categories: 设计模式
---
代理模式的作用是：为其他对象提供一种代理以控制对这个对象的访问。在某些情况下，一个客户不想或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用。 
代理模式一般涉及到三个角色： 
抽象角色：声明真实对象和代理对象的共同接口； 
代理角色：代理对象角色内部含有对真实对象的引用，从而可以操作真实对象，同时代理对象提供与真实对象相同的接口以便在任何时刻都能代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。 
真实角色：代理角色所代表的真实对象，是我们最终要引用的对象。在某些情况下，一个对象不想或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用。代理模式的思想是为了提供额外的处理或者不同的操作而在实际对象与调用者之间插入一个代理对象。这些额外的操作通常需要与实际对象进行通信。代理可分为静态代理和动态代理。
<!--more-->
                                  代理模式结构类图
                   {% img /img/proxy.png 400 370 代理模式结构类图 %}
<h3>静态代理</h3>
首先建立抽象角色
{% codeblock lang:java %}
package com.crazyfish.kamal.test.TestSomething;

/**
 * Created by kamal on 15/8/24.
 */
public interface IUserLogin {
    boolean login();
    boolean register();
}
{% endcodeblock %}

建立实际类对象，即抽象对象的真实行为,真实角色行为
{% codeblock lang:java %}
package com.crazyfish.kamal.test.TestSomething;

/**
 * Created by kamal on 15/8/24.
 */
public class UserLoginImpl implements IUserLogin {
    public boolean login() {
        System.out.println("you can login");
        return false;
    }

    public boolean register() {
        System.out.println("you can register");
        return false;
    }
}
{% endcodeblock %}
建立代理类，也会实现抽象对象类，但是他并没有做实际的事情，只做了一些修饰性的行为，比如查找权限，如果合法则调用真实的类行为
{% codeblock lang:java %}
package com.crazyfish.kamal.test.TestSomething;

/**
 * Created by kamal on 15/8/24.
 */
public class UserLoginProxy implements IUserLogin{
    public IUserLogin login;
    public UserLoginProxy(IUserLogin login){
        this.login = login;
    }

    public boolean login() {
        System.out.println("查找权限，result-->i can't login ,but by proxy");//do something others;
        login.login();
        return false;
    }

    public boolean register() {
        System.out.println("i can't register ,but by proxy");
        login.register();
        return false;
    }
    public static void main(String args[]){
        IUserLogin lo = new UserLoginImpl();
        IUserLogin pr = new UserLoginProxy(lo);
        pr.login();
        pr.register();
    }
}
{% endcodeblock %}
按照上述的方法使用代理模式，那么真实角色必须是事先已经存在的（即具体的行为），并将其作为代理对象的内部属性。但实际中，一个真实角色必须对应一个代理角色，如果大量使用会导致类的急剧膨胀；此外，如果事先并不知道真实角色，静态代理是无法使用的，但是可以通过Java的动态代理类来解决。
<h3>动态代理</h3>
抽象对象和具体对象用上面的两个类就可以了。
现在只需要实现代理类
Java动态代理类位于Java.lang.reflect包下，一般主要涉及到以下两个类：

(1). Interface InvocationHandler：该接口中仅定义了一个方法Object：invoke(Object obj,Method method, Object[] args)。在实际使用时，第一个参数obj一般是指代理类，method是被代理的方法，如上例中的request()，args为该方法的参数数组。 这个抽象方法在代理类中动态实现。


(2).Proxy：该类即为动态代理类，作用类似于上例中的ProxySubject，其中主要包含以下内容：

Protected Proxy(InvocationHandler h)：构造函数，估计用于给内部的h赋值。

Static Class getProxyClass (ClassLoader loader, Class[] interfaces)：获得一个代理类，其中loader是类装载器，interfaces是真实类所拥有的全部接口的数组。

Static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)：返回代理类的一个实例，返回后的代理类可以当作被代理类使用(可使用被代理类的在Subject接口中声明过的方法)。

 

所谓Dynamic Proxy是这样一种class：它是在运行时生成的class，在生成它时你必须提供一组interface给它，然后该class就宣称它实现了这些 interface。你当然可以把该class的实例当作这些interface中的任何一个来用。当然啦，这个Dynamic Proxy其实就是一个Proxy，它不会替你作实质性的工作，在生成它的实例时你必须提供一个handler，由它接管实际的工作。

在使用动态代理类时，我们必须实现InvocationHandler接口，以第一节中的示例为例：
{% codeblock lang:java %} 
package com.crazyfish.kamal.test.TestSomething;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * Created by kamal on 15/8/24.
 */
public class DynamicProxy implements InvocationHandler {
    //代理对象
    public Object obj;
    public DynamicProxy(Object obj){
        this.obj = obj;
    }
    //代理方法
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("查找权限，result-->i can't " + method.getName() + ",but by proxy");
        Object o = method.invoke(obj,args);
        return o;
    }
    public static void main(String args[]){
        try {
            IUserLogin lo = new UserLoginImpl();
            DynamicProxy inv = new DynamicProxy(lo);
            IUserLogin l = (IUserLogin) Proxy.newProxyInstance(lo.getClass().getClassLoader(), lo.getClass().getInterfaces(), inv);
            l.login();
            l.register();
        }catch (Exception  e){
            e.printStackTrace();
            System.exit(2);
        }
    }
}
{% endcodeblock %}
<h3>静态代理与动态代理的比较</h3>
（1）静态代理由程序员创建或工具生成代理类的源码，再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。
（2）静态代理只服务于一个接口。而动态代理可以服务于多个接口。例如上述代码中的静态代理实现的是IUserLogin接口，该代理类只服务于该接口，而动态代理DynamicProxy实现的是InvocationHandler接口，不仅可以为 IUserLogin接口服务，也可为其他新增接口服务。
（3）静态代理中如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。
（4）动态代理类的源码是在程序运行期间由JVM根据反射等机制动态生成，不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。动态代理中的ClassLoader是类装载器类，负责将类的字节码装载到 Java 虚拟机（JVM）中并为其定义类对象，然后该类才能被使用。Proxy 静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是由 JVM 在运行时动态生成的而非预存在于任何一个 .class 文件中。 每次生成动态代理类对象时都需要指定一个类装载器对象.
（5）动态代理与普通的代理相比较，最大的好处是接口中声明的所有方法都被转移到一个集中的方法中处理（invoke），这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。
