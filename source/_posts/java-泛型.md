title: java 泛型
date: 2015-08-03 18:51:58
tags: java 泛型
categories: java
---
泛型的本质是将数据类型参数化，即数据类型被指定为一个参数。泛型优点：编译的时候检查类型安全，所有的强制类型转换都是自动和隐式的，提高代码重用率。Java的泛型基本上完全在编译器中实现，有编译器执行类型检查和类型推断，然后生成普通的非反省的字节码。该技术叫擦除（erasure）（编译器使用泛型类型信息保证类型安全，然后在生成字节码之前将其擦除）。
 <!--more-->
 <h4>泛型优点</h4>
 编译器执行类型检查，保证这些类型转换绝对无误。
{% codeblock lang:java 泛型优点 %}
        //不使用泛型类型 运行时可能抛出异常
        List list1 = new ArrayList();
        list1.add(38);                      //编译器不检查值类型
        String str1 = (String)list1.get(0); //需手动强制转换,如转换类型与原数据类型不一致将抛出ClassCastException异常
        
        //使用泛型类型 编译时通不过
        List<String> list2 = new ArrayList<String>();
        list2.add("kamal");         //[类型安全的写入数据] 编译器检查该值,该值必须是String类型才能通过编译
        String str2 = list2.get(0); //[类型安全的读取数据] 不需要手动转换
{% endcodeblock %}
<h4>泛型类型擦除</h4>
Java泛型只存在于编译期，编译好的Java字节码不包含泛型类型信息。编译后擦除泛型信息。JVM是没有泛型概念的。所以在java中使用泛型重载是不行的。
{% codeblock lang:java 类型擦除 %}
        List<String>    list1 = new ArrayList<String>();
        List<Integer> list2 = new ArrayList<Integer>();
          
        System.out.println(list1.getClass() == list2.getClass()); // 输出： true
        System.out.println(list1.getClass().getName()); // 输出： java.util.ArrayList
        System.out.println(list2.getClass().getName()); // 输出： java.util.ArrayList
{% endcodeblock %}
<h4>泛型不是协变的</h4>
什么叫做协变？
协变是什么意思？ 维基百科中是这样定义的：在一门程序设计语言的类型系统中，一个类型规则或者类型构造器是： * 协变（covariant），如果它保持了子类型序关系≦。该序关系是：子类型≦基类型。 * 逆变（contravariant），如果它逆转了子类型序关系。 * 不变（invariant），如果上述两种均不适用。 首先考虑数组类型构造器： 从Animal类型，可以得到Animal[]（“animal数组”）。 是否可以把它当作 * 协变：一个Cat[]也是一个Animal[] * 逆变：一个Animal[]也是一个Cat[] * 以上二者均不是（不变)。

Java中数组是协变的（covariant），例如：如果Integer扩展了Number，那么不仅Integer是Number，而且Integer[]也是Number[]，在需要Number[]的地方可以传递或者赋值Integer[]。但是这一原理不适用于泛型，即在需要List<Number>的地方，传递和赋值List<Integer>是不行的。
为什么不这么做呢？
因为这样做将破坏要提供的类型安全泛型。如果能够将List<Integer>赋给List<Number>。则如下代码就允许将非Integer得内容放入List<Integer>
{% codeblock lang:java java泛型非协变 %}
List<Integer> li = new ArrayList<Integer>(); 
List<Number> ln = li; // 非法的 
ln.add(new Float(3.1415));//Number赋值Float完全合法，但是实际上赋值给li列表，而Float肯定不能转换为Integer
{% endcodeblock %}
数组能协变而泛型不能协变将导致，不能实例化泛型类型的数组（new List<String>[10]是非法的)，除非类型参数是一个未绑定的通配符（new List<?>[10]是合法的)。如果允许泛型数组会造成什么后果呢？
{% codeblock lang:java 数组协变 %}
List<String>[] lsa = new List<String>[10]; // illegal 
 Object[] oa = lsa;  // OK because List<String> is a subtype of Object 
 List<Integer> li = new ArrayList<Integer>(); 
 li.add(new Integer(3)); 
 oa[0] = li; 
 String s = lsa[0].get(0);//ClassCastException
 {% endcodeblock %}
 数组协变会破坏泛型的类型安全。
 <h4>泛型通配符</h4>
 为了解决类型被限制死了不能动态根据实例来确定的缺点，引入了“通配符泛型”，针对上面的例子，使用通配泛型格式为<? extends Collection>，“？”代表未知类型，这个类型是实现Collection接口。
注意：
1、如果只指定了<?>，而没有extends，则默认是允许Object及其下的任何Java类了。也就是任意类。
2、通配符泛型不单可以向下限制，如<? extends Collection>，还可以向上限制，如<? super Double>，表示类型只能接受Double及其上层父类类型，如Number、Object类型的实例。
3、泛型类定义可以有多个泛型参数，中间用逗号隔开，还可以定义泛型接口，泛型方法。这些都与泛型类中泛型的使用规则类似。
<h4>泛型方法</h4>
可以在类中包含参数化方法，而这个方法所在的类可以是泛型类，也可以不是泛型类。是否拥有泛型方法，和所在的类是否泛型没有关系。泛型方法使得该方法能够独立于类而产生变化。以下是一个基本原则：如果泛型方法可以取代整个类的泛型化，就应该只使用泛型方法。另外，对于一个static方法而言，无法访问泛型类的参数类型，所以static方法需要使用泛型能力，就必须成为泛型方法。
{% codeblock lang:java 泛型方法 %}
public <T,M,N> void getTType(T t,M m,N n){  
        //传入int,long ,double等基本类型时，自动装箱机制 
        //会将基本类型包装成相应的对象  
        System.out.println(t.getClass().getName());  
        System.out.println(m.getClass().getName());  
        System.out.println(n.getClass().getName());  
}  
{% endcodeblock %}
<h4>泛型代码举例</h4>
{% codeblock lang:java 泛型代码举例 %}
package com.crazyfish.kamal.test.testgeneric;
/**
 * @author lipengpeng
 * @desc
 * @date 2016-01-28 下午1:12
 */
public interface Human {
    public void run();
}

package com.crazyfish.kamal.test.testgeneric;
/**
 * @author lipengpeng
 * @desc 男人
 * @date 2016-01-28 下午1:16
 */
public class Man implements Human {
    public String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Man(){}
    
    public Man(String name){
        this.name = name;
    }
    @Override
    public void run() {
        System.out.println("man run...");
    }
}

package com.crazyfish.kamal.test.testgeneric;
/**
 * @author lipengpeng
 * @desc 女人
 * @date 2016-01-28 下午1:15
 */
public class Woman implements Human {
    public String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    public Woman(String name){
        this.name = name;
    }
    @Override
    public void run() {
        System.out.println("woman run...");
    }
}

package com.crazyfish.kamal.test.testgeneric;
/**
 * @author lipengpeng
 * @desc
 * @date 2016-01-28 下午1:23
 */
public class Cook<T extends Woman> {
    public void cook(T woman){
        System.out.println("cook " + woman.getName());
    }
}

package com.crazyfish.kamal.test.testgeneric;
/**
 * @author lipengpeng
 * @desc
 * @date 2016-01-28 下午1:23
 */
public class Work<T extends Man> {
    public void working(T man){
        System.out.println("working " + man.getName());
    }
}

package com.crazyfish.kamal.test.testgeneric;
/**
 * @author lipengpeng
 * @desc 具体的某一个人
 * @date 2016-01-28 下午1:16
 */
public class Kamal extends Man {
    private String name;

    @Override
    public String getName() {
        return name;
    }

    @Override
    public void setName(String name) {
        this.name = name;
    }

    public Kamal(String name){
        this.name = name;
    }

    public void run(){
        System.out.println("kamal run ...");
    }
}

package com.crazyfish.kamal.test.testnetty;
/**
 * @author lipengpeng
 * @desc
 * @date 2016-01-29 下午5:48
 */
import com.crazyfish.kamal.test.testgeneric.Man;
import com.crazyfish.kamal.test.testgeneric.Work;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
//Work<Man>是ParameterizedType（参数化类型），因为它需要参数Man或其他参数，Man不是参数化类型
public class Base<T> extends Work<Man> {//去掉extends Work<Man>则无法实例化entityClass
    private Class<T> entityClass;

    public Base(){
        //entityClass =(Class<T>)((ParameterizedType)getClass().getGenericSuperclass()).getActualTypeArguments()[0];
        try {
            Class<?> clazz = getClass(); //获取实际运行的类的 Class
            Type type = clazz.getGenericSuperclass(); //获取实际运行的类的直接超类的泛型类型
            if(type instanceof ParameterizedType){ //如果该泛型类型是参数化类型
                Type[] parameterizedType = ((ParameterizedType)type).getActualTypeArguments();//获取泛型类型的实际类型参数集
                entityClass = (Class<T>) parameterizedType[0]; //取出第一个(下标为0)参数的值
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //泛型的实际类型参数的类全名
    public String getEntityName(){
        return entityClass.getName();
    }

    //泛型的实际     类型参数的类名
    public String getEntitySimpleName(){
        return entityClass.getSimpleName();
    }

    //泛型的实际类型参数的Class
    public Class<T> getEntityClass() {
        return entityClass;
    }

}

package com.crazyfish.kamal.test.testgeneric;
import com.crazyfish.kamal.test.testnetty.Base;
import java.util.ArrayList;
import java.util.List;

/**
 * @author lipengpeng
 * @desc
 * @date 2016-01-28 下午1:17
 */
public class Main {
    public static void main(String args[]){
        Woman woman = new Woman("Rose");
        Cook cook = new Cook<Woman>();
        //Cook cook1 = new Cook<Man>();//无法编译通过
        Man man = new Man("Tom");
        Kamal kamal = new Kamal("kamal");
        Work work = new Work<Kamal>();

        cook.cook(woman);
        work.working(man);
        work.working(kamal);
        List<Man> list = new ArrayList();
        testSuper(list);

        List<Kamal> list1 = new ArrayList();
        list1.add(kamal);
        testExtends(list1);
        //testSuper(list1);//无法编译
        //testExtends(list);//无法编译

        Base<String> base = new Base<String>();
        System.out.println(base.getEntityClass());

        //编译器进行了类型推导和自动打包，也就是说原来需要我们自己对类型进行的判断和处理，现在编译器帮我们做了
        getTType(1.3,new Man(),"123");
    }

    //super 下界，？至少是super后面的类或者其父类（？至少应该是Man或者Man的父类
    public static void testSuper(List<? super Man> work){
        Man lee = new Man("lee");
        Human human = new Man("xx");
        //work.add(human);//无法插入
        work.add(lee);//为什么可以add呢，因为？一定是Man或者Man的超类
        try {
            Kamal tmp = (Kamal) work.get(0);//向下类型转换失败，get获取的一定是Man或者Man的父类，因为用super限定类List容器能够存储的类型
        }catch(Exception e){
            //e.printStackTrace();
        }
        Kamal jack = new Kamal("Jack");
        work.add(jack);

        ArrayList<? super Integer> collection = null;//限定通配符下届
        collection = new ArrayList<Object>();
        collection = new ArrayList<Number>();
    }

    //extends 上界，？至少是extends后面的类或者其子类（？至少是Kamal或者其之类）
    public static void testExtends(List<? extends Kamal> work){
        Man lee = new Man("lee");
        //work.add(lee);//无法add,向下类型转换失败,因为add一定是Kamal或者Kamal的子类
        try {
            Kamal tmp = (Kamal) work.get(0);
        }catch(Exception e){
            e.printStackTrace();
        }
        Kamal jack = new Kamal("Jack");
        //work.add(jack);//与work.add(lee)进行对比，这个类型明明是合法的呀，为什么不能add呢，因为work集合中有可能之前存在的是Kamal的子类，则
        //kamal强制转换为kamal的子类失败
        Man k = work.get(0);//向上类型转换，没有问题
        System.out.println(k.getName());

        ArrayList<? extends Number> collection = null;//限定通配符上界
        collection = new ArrayList<Number>();
        collection = new ArrayList<Short>();
    }

    //泛型方法
    public static <T,M,N> void getTType(T t,M m,N n){
        //传入int,long ,double等基本类型时，自动装箱机制
        //会将基本类型包装成相应的对象
        System.out.println(t.getClass().getName());
        System.out.println(m.getClass().getName());
        System.out.println(n.getClass().getName());
    }
}
{% endcodeblock %}