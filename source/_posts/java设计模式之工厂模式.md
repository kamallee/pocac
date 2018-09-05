title: java设计模式之工厂模式
date: 2015-08-06 11:22:03
tags: 设计模式
---
工厂方法模式是类的创建模式，又叫做虚拟构造子(Virtual Constructor)模式或者多态性工厂（Polymorphic Factory）模式。
工厂方法模式的用意是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类中。
<!--more-->
<h2>简单工厂模式</h2>
{% img /img/simplefactory.png 简单工厂模式 %}
<h4>简单工厂模式demo</h4>
简单工厂模式又称静态工厂方法模式
建立一个工厂（一个类方法buyComputer）来制造新的对象。客户只需要向工厂中传递电脑品牌就可以获得相应的电脑，而无需知道具体的电脑创建细节。
{% codeblock lang:java 简单工长 %}
package com.crazyfish.kamal.test.testdesignpattern.testfactorymodel;

/**
 * @author lipengpeng
 * @desc 简单工厂模式
 * @date 2016-02-03 下午3:58
 */
public class SimpleFactory {
    public interface Computer{
        public void run();
    }

    public static class Apple implements Computer{
        @Override
        public void run(){
            System.out.println("我是苹果电脑");
        }
    }

    public static class Ibm implements Computer{
        @Override
        public void run(){
            System.out.println("我是ibm电脑");
        }
    }

    public static class SimpleFactorys{
        public Computer buyComputer(String type){
            if( type.equals("apple")){
                return new Apple();
            }else{
                return new Ibm();
            }
        }
    }

    public static void main(String args[]){
        SimpleFactorys factorys = new SimpleFactorys();
        Computer com = factorys.buyComputer("apple");
        com.run();
    }
}
{% endcodeblock %}
从代码可以看出，简单工厂（确实够简单）只有一个工厂，用来处理所有的创建逻辑，如果产品增多，则如代码中的if-else将会很多，日后增加将会平凡对该方法（buyComputer）进行修改，违背开闭原则（对扩张开放，对修改封闭）。
<h2>工厂方法模式</h2>
工厂方法模式去掉了简单工厂模式中工厂方法的静态属性，使得它可以被子类继承。这样在简单工厂模式里集中在工厂方法上的压力可以由工厂方法模式里不同的工厂子类来分担。
{% img /img/factorymethod.png 工厂方法模式 %}
<h4>工厂方法模式demo</h4>
{% codeblock lang:java 工厂方法模式 %}
package com.crazyfish.kamal.test.testdesignpattern.testfactorymodel;

/**
 * @author lipengpeng
 * @desc 工厂方法模式
 * @date 2016-02-03 下午4:10
 */
public class FactoryMethod {

    //产品类
    public interface Computer{
        public void run();
    }

    public static class Apple implements Computer{
        @Override
        public void run(){
            System.out.println("我是苹果电脑");
        }
    }

    public static class Ibm implements Computer{
        @Override
        public void run(){
            System.out.println("我是ibm电脑");
        }
    }

    //工厂类
    public interface FactoryMethods{
        public Computer buyComputer();
    }

    public static class BuyApple implements FactoryMethods{

        @Override
        public Computer buyComputer() {
            return new Apple();
        }
    }

    public static class BuyIbm implements FactoryMethods{
        @Override
        public Computer buyComputer(){
            return new Ibm();
        }
    }

    public static void main(String args[]){
        FactoryMethods factoryMethods = new BuyApple();//需要什么我就创建什么
        factoryMethods.buyComputer().run();

        FactoryMethods factoryMethods1 = new BuyIbm();
        factoryMethods1.buyComputer().run();
    }
}
{% endcodeblock %}
从代码可以看出，当有新的产品产生时，只要对抽象产品、抽象工厂进行扩展，那么就可以被客户使用，而不必去修改任何已有的代码。可以看出工厂角色的结构也是符合开闭原则的。但是缺点也很明显，产品的增多导致类不断膨胀，因为每个产品都需要一个类扩展自抽象产品类，工厂类也相应增加扩展自抽象工厂类的子类。
<h2>抽象工厂模式</h2>
抽象工厂模式是工厂方法模式的升级版本，他用来创建一组相关或者相互依赖的对象。抽象工厂针对的是有多个产品（如apple,cpu,主板等)的等级结构。而工厂方法模式针对的是一个产品(只有apple)的等级结构。
产品族：
所谓产品族，是指位于不同产品等级结构中，功能相关联的产品组成的家族。比如AMD的主板、芯片组、CPU组成一个家族，Intel的主板、芯片组、CPU组成一个家族。而这两个家族都来自于三个产品等级：主板、芯片组、CPU。一个等级结构是由相同的结构的产品组成
{% img /img/abstractfactory.png 抽象工厂模式 %}
<h4>抽象工厂模式demo</h4>
{% codeblock lang:java 抽象工厂模式 %}
package com.crazyfish.kamal.test.testdesignpattern.testfactorymodel;

/**
 * @author lipengpeng
 * @desc 抽象工厂
 * @date 2016-02-03 下午4:18
 */
public class AbstractFactory {
    public interface Computer {
        public void run();
    }

    public interface CPU {
        public void run();
    }

    public static class Apple implements Computer {
        @Override
        public void run() {
            System.out.println("我是苹果电脑");
        }
    }

    public static class Ibm implements Computer {
        @Override
        public void run() {
            System.out.println("我是ibm电脑");
        }
    }

    public static class IntelCPU implements CPU{
        @Override
        public void run(){
            System.out.println("使用IntelCPU");
        }
    }

    public static class AMDCPU implements CPU{
        @Override
        public void run(){
            System.out.println("使用AMDCPU");
        }
    }

    public interface AbstractFactorys{
        public Computer buyComputer();
        public CPU withCPU();
    }

    public static class AppleWithIntelCPU implements AbstractFactorys{

        @Override
        public Computer buyComputer() {
            return new Apple();
        }

        @Override
        public CPU withCPU() {
            return new IntelCPU();
        }
    }

    public static class IbmWithAMDCPU implements AbstractFactorys{

        @Override
        public Computer buyComputer() {
            return new Ibm();
        }

        @Override
        public CPU withCPU() {
            return new AMDCPU();
        }
    }

    public static void main(String args[]){
        AbstractFactorys abstractFactorys = new AppleWithIntelCPU();
        abstractFactorys.buyComputer().run();
        abstractFactorys.withCPU().run();

        AbstractFactorys abstractFactorys1 = new IbmWithAMDCPU();
        abstractFactorys1.buyComputer().run();
        abstractFactorys1.withCPU().run();
    }
}
{% endcodeblock %}
从代码可以看出，当产品只有一种的时候，而不是产品族，将会退化成工厂方法模式。