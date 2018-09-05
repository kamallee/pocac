title: java设计模式之策略模式
date: 2017-06-8 15:12:33
tags: 设计模式 策略 
categories: 设计模式
---
<h2>什么事策略模式</h2>
　　策略模式属于对象的行为模式。其用意是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。策略模式使得算法可以在不影响到客户端的情况下发生变化。
　　策略模式把一个系列的算法封装到一个系列的具体策略类里面，作为一个抽象策略类的子类或策略接口的实现类。简单地说：准备一组算法，并将每一个算法封装起来，使它们可以互换。
disruptor的消费者等待策略就是用了策略模式，JDK中的ThreadPoolExecutor线程池对拒绝任务的处理策略。
　　这个模式涉及到3个角色：
　　环境(Context)角色：持有一个Strategy抽象策略类或策略接口的引用。
　　抽象策略(Strategy)角色：这是一个抽象角色，由一个接口或抽象类实现。此角色声明所有的具体策略类需要重写的方法。
　　具体策略(Concrete Strategy)角色：封装了相关的算法或行为。
<!--more-->
<h2>示意性实例</h2>
　　使用场景实例：
　　假设某个网站销售各种书籍，对初级会员没有提供折扣，对中级会员提供每本10%的促销折扣，对高级会员提供每本20%的促销折扣。

折扣接口
{% codeblock lang:java%}
public interface MemberStrategy {
     /**
      * 计算图书的价格
      * @param booksPrice    图书的原价
      * @return    计算出打折后的价格
      */
     public double calcPrice(double booksPrice);
}
{% endcodeblock %}
初级会员折扣实现类
{% codeblock lang:java%}
public class PrimaryMemberStrategy implements MemberStrategy {
      @Override
     public double calcPrice(double booksPrice) {
         System.out.println("对于初级会员的没有折扣");
         return booksPrice;
     }
}
{% endcodeblock %}
中级会员折扣实现类
{% codeblock lang:java%}
public class IntermediateMemberStrategy implements MemberStrategy {
     @Override
     public double calcPrice(double booksPrice) {
         System.out.println("对于中级会员的折扣为10%");
         return booksPrice * 0.9;
     }
}
 {% endcodeblock %}
高级会员折扣实现类
{% codeblock lang:java%}
public class AdvancedMemberStrategy implements MemberStrategy {
     @Override
    public double calcPrice(double booksPrice) {
         System.out.println("对于高级会员的折扣为20%");
         return booksPrice * 0.8;
     }
}
{% endcodeblock %}
价格类
{% codeblock lang:java%}
public class Price {
     // 持有一个具体的策略对象
     private MemberStrategy strategy;
     /**
      * 构造函数，传入一个具体的策略对象
      * @param strategy    具体的策略对象
      */
     public Price(MemberStrategy strategy){
         this.strategy = strategy;
     }
      /**
      * 计算图书的价格
      * @param booksPrice    图书的原价
      * @return    计算出打折后的价格
      */
     public double quote(double booksPrice){
         return this.strategy.calcPrice(booksPrice);
     }
}
{% endcodeblock %}
客户端

{% codeblock lang:java%}
public class Client {
    public static void main(String[] args) {
         // 选择并创建需要使用的策略对象
         MemberStrategy strategy = new AdvancedMemberStrategy();
         // 创建环境
         Price price = new Price(strategy);
         // 计算价格
         double quote = price.quote(300);
         System.out.println("图书的最终价格为：" + quote);
    }
}
{% endcodeblock %}

<h2>总结</h2>
策略模式的重心
策略模式的重心不是如何实现算法，而是如何组织、调用这些算法，从而让程序结构更灵活，具有更好的维护性和扩展性。策略算法是相同行为的不同实现。在运行期间，策略模式在每一个时刻只能使用一个具体的策略实现对象。把所有的具体策略实现类的共同公有方法封装到抽象类里面，将代码向继承等级结构的上方集中。

算法的平等性
　　策略模式一个很大的特点就是各个策略算法的平等性。对于一系列具体的策略算法，大家的地位是完全一样的，正因为这个平等性，才能实现算法之间可以相互替换。所有的策略算法在实现上也是相互独立的，相互之间是没有依赖的。
　　所以可以这样描述这一系列策略算法：策略算法是相同行为的不同实现。

　　运行时策略的唯一性
　　运行期间，策略模式在每一个时刻只能使用一个具体的策略实现对象，虽然可以动态地在不同的策略实现中切换，但是同时只能使用一个。

　　公有的行为
　　经常见到的是，所有的具体策略类都有一些公有的行为。这时候，就应当把这些公有的行为放到共同的抽象策略角色Strategy类里面。当然这时候抽象策略角色必须要用Java抽象类实现，而不能使用接口。
　　这其实也是典型的将代码向继承等级结构的上方集中的标准做法。

策略模式优点
　　1 通过策略类的等级结构来管理算法族。
　　2 避免使用将采用哪个算法的选择与算法本身的实现混合在一起的多重条件(if-else if-else)语句。

　　策略模式缺点：
　　1 客户端必须知道所有的策略类，并自行决定使用哪一个策略类。
　　2 由于策略模式把每个算法的具体实现都单独封装成类，针对不同的情况生成的对象就会变得很多。