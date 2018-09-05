title: java设计模式之适配器模式
date: 2017-03-22 15:52:11
tags: 设计模式 适配器
categories: 设计模式
---
<h2>什么是适配器</h2>
适配器模式中引入了一个被称为适配器(Adapter)的包装类，而它所包装的对象称为适配者(Adaptee)，即被适配的类。适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用。也就是说：当客户类调用适配器的方法时，在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的，客户类并不直接访问适配者类。因此，适配器让那些由于接口不兼容而不能交互的类可以一起工作。 

定义
适配器模式(Adapter Pattern)：将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

在适配器模式中，我们通过增加一个新的适配器类来解决接口不兼容的问题，使得原本没有任何关系的类可以协同工作。根据适配器类与适配者类的关系不同，适配器模式可分为对象适配器和类适配器两种，在对象适配器模式中，适配器与适配者之间是关联关系；在类适配器模式中，适配器与适配者之间是继承（或实现）关系。在实际开发中，对象适配器的使用频率更高。
<!--more-->

<h2>对象适配器</h2>
软件公司OA系统需要提供一个加密模块，将用户机密信息（如口令、邮箱等）加密之后再存储在数据库中，系统已经定义好了数据库操作类。为了提高开发效率，现需要重用已有的加密算法，这些算法封装在一些由第三方提供的类中，有些甚至没有源代码。试使用适配器模式设计该加密模块，实现在不修改现有类的基础上重用第三方加密方法。

{% codeblock lang:java %}
/** 
 * 系统已经定义好的加密接口
 */
public interface Encryption {
    /**
     * 加密方法
     */
    public String encode(String string);
    /**
     * 解密方法
     */
    public String decode(String string);
}
{% endcodeblock %}

{% codeblock lang:java %}
/** 
 * 第三方的加密类，这里只是模拟，在现实情况下我们可能不会拥有源码
 */
public class ThirdPartyEncryption {
    public String tEncode(String string) {
        return "加密之后的" + string;
    }
    public String tDecode(String string) {
        return "解密之后的" + string;
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
/** 
 * 适配器类实现系统定义好的接口
 */
public class EncryptionAdapter implements Encryption{
    // 组合一个第三方的加密类
    private ThirdPartyEncryption tpe;
    public EncryptionAdapter() {
        tpe = new ThirdPartyEncryption();
    }
    @Override
    public String encode(String string) {
        // 调用第三方的加密方法
        return tpe.tEncode(string);
    }
    @Override
    public String decode(String string) {
        // 调用第三方的解密方法
        return tpe.tDecode(string);
    }
}
{% endcodeblock %}

配置文件：
{% codeblock lang:xml%}
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <className>com.design.structure.adapterpattern.EncryptionAdapter</className>
</config>
{% endcodeblock %}

客户端：
{% codeblock lang:java %}
public class Main {
    public static void main(String[] args) {
        Encryption encryption = (Encryption) XMLUtil.getBean();
        String encodeString = encryption.encode("password");
        String decodeString = encryption.decode("password");
        //TODO 将encodeString插入数据库
        System.out.println(encodeString);
        System.out.println(decodeString);
    }
}
{% endcodeblock %}

为了让系统具备良好的灵活性和可扩展性，我们在配置文件config.xml中存储了适配器类的类名，这样当更换其他适配器的时候就可以不修改代码了。符合“开闭原则”。

<h2>类适配器</h2>
除了对象适配器模式之外，适配器模式还有一种形式，那就是类适配器模式，类适配器模式和对象适配器模式最大的区别在于适配器和适配者之间的关系不同，对象适配器模式中适配器和适配者之间是关联关系，而类适配器模式中适配器和适配者是继承关系

更具上面的例子我们可以用类适配器来修一下

{% codeblock lang:java %}
public class Adapter extends ThirdPartyEncryption implements Encryption{
    @Override
    public String encode(String string) {
        return tEncode(string);
    }
    @Override
    public String decode(String string) {
        return tDecode(string);
    }
}
{% endcodeblock %}

这里我们的适配器类继承了第三方的加密类，同时实现了系统定义好的加密接口。实现了适配

由于Java、C#等语言不支持多重类继承，因此类适配器的使用受到很多限制，例如如果目标抽象类Encryption不是接口，而是一个类，就无法使用类适配器；此外，如果适配者Adapter为最终(Final)类，也无法使用类适配器。在Java等面向对象编程语言中，大部分情况下我们使用的是对象适配器，类适配器较少使用。

<h2>适配器的优缺点</h2>
适配器模式将现有接口转化为客户类所期望的接口，实现了对现有类的复用，它是一种使用频率非常高的设计模式，在软件开发中得以广泛应用，在spring等开源框架、驱动程序设计（如JDBC中的数据库驱动程序）中也使用了适配器模式。

适配器模式的优点
无论是对象适配器模式还是类适配器模式都具有如下优点：

1.将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构。
2.增加了类的透明性和复用性，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。
3.灵活性和扩展性都非常好，通过使用配置文件，可以很方便地更换适配器，也可以在不修改原有代码的基础上增加新的适配器类，完全符合“开闭原则”。
4.由于适配器类是适配者类的子类，因此可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强。
5.一个对象适配器可以把多个不同的适配者适配到同一个目标。
6.可以适配一个适配者的子类，由于适配器和适配者之间是关联关系，根据“里氏代换原则”，适配者的子类也可通过该适配器进行适配。

适配器模式的缺点
1.对于Java、C#等不支持多重类继承的语言，一次最多只能适配一个适配者类，不能同时适配多个适配者；

2.适配者类不能为最终类，如在Java中不能为final类，C#中不能为sealed类；
3.在Java、C#等语言中，类适配器模式中的目标抽象类只能为接口，不能为类，其使用有一定的局限性；
4.与类适配器模式相比，要在适配器中置换适配者类的某些方法比较麻烦。如果一定要置换掉适配者类的一个或多个方法，可以先做一个适配者类的子类，将适配者类的方法置换掉，然后再把适配者类的子类当做真正的适配者进行适配，实现过程较为复杂。

适用场景
1.系统需要使用一些现有的类，而这些类的接口（如方法名）不符合系统的需要，甚至没有这些类的源代码。
2.想创建一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作。