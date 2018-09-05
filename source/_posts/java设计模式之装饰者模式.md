title: java设计模式之装饰者模式
date: 2015-10-09 14:23:58
tags: 设计模式
categories: 设计模式
---
在装饰模式中的各个角色有：
　　（1）抽象构件（Component）角色：给出一个抽象接口，以规范准备接收附加责任的对象。
　　（2）具体构件（Concrete Component）角色：定义一个将要接收附加责任的类。
　　（3）装饰（Decorator）角色：持有一个构件（Component）对象的实例，并实现一个与抽象构件接口一致的接口。
　　（4）具体装饰（Concrete Decorator）角色：负责给构件对象添加上附加的责任。
<!--more-->
装饰者模式介绍
通过使用修饰模式，可以在运行时扩充一个类的功能。原理是：增加一个修饰类包裹原来的类，包裹的方式一般是通过在将原来的对象作为修饰类的构造函数的参数。装饰类实现新的功能，但是，在不需要用到新功能的地方，它可以直接调用原来的类中的方法。修饰类必须和原来的类有相同的接口。
修饰模式是类继承的另外一种选择。类继承在编译时候增加行为，而装饰模式是在运行时增加行为。
当有几个相互独立的功能需要扩充时，这个区别就变得很重要。在有些面向对象的编程语言中，类不能在运行时被创建，通常在设计的时候也不能预测到有哪几种功能组合。这就意味着要为每一种组合创建一个新类。相反，修饰模式是面向运行时候的对象实例的,这样就可以在运行时根据需要进行组合。一个修饰模式的示例是JAVA里的Java I/O Streams的实现。
{% codeblock lang:java %}
package com.crazyfish.kamal.test.testdesignpattern;

/**
 * Created by kamal on 15/10/9.
 */
public class TestDecorator {
    public static void main(String args[]){
        Coffee cof = new Espresso();
        Coffee c1 = new AddMilk(cof);

        Coffee coff = new Decaf();
        Coffee c2 = new AddSoy(coff);

        Coffee c3 = new AddSoy(cof);
        System.out.println("c1:" + c1.cost() + "c2:" + c2.cost() + "c3:" + c3.cost());
    }
}
interface Coffee{//抽象构件
    double cost();//总花费
}
class Espresso implements Coffee{//具体构件
    public double cost(){
        return 3;
    }
}
class Decaf implements Coffee{//具体构件
    public double cost(){
        return 5;
    }
}

interface Condiment extends Coffee{//装饰
    double cost();
}

class AddMilk implements Condiment{//具体装饰
    Coffee coffee;
    public AddMilk(Coffee coffee){
        this.coffee = coffee;
    }
    public double cost(){
        return coffee.cost() + 2.5;
    }
}
class AddSoy implements Condiment{//具体装饰
    Coffee coffee;
    public AddSoy(Coffee coffee){
        this.coffee = coffee;
    }
    public double cost(){
        return coffee.cost() + 3.5;
    }
}
{% endcodeblock %}
装饰者模式典型应用场合就是java io中：
{% img /img/javaio.jpeg 700 350 装饰者模式应用 %} 
