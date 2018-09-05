title: java回调
date: 2015-08-20 11:42:30
tags: java 
categories: java
---
 回调是一种设计模式。所谓回调，就是客户程序C调用服务程序S中的某个函数A，然后S又在某个时候反过来调用C中的某个函数B，对于C来说，这个B便叫做回调函数。
回调流程：首先要定义一个接口及其要实现的方法；
其次要被回调的地方必须实现上面的接口，或者采用匿名内部类实现;
最后在另一个处理耗时或者等待操作的类中，接收传来的IDataListener对象，然后调用回调方法。
<!--more-->
{% codeblock lang:java %}
package com.crazyfish.kamal.test.testjavacollback;

/**
 * Created by kamal on 15/8/20.
 */
public interface IDataListener {
    void dataChangeListener();
}
{% endcodeblock %}

{% codeblock lang:java %}
package com.crazyfish.kamal.test.testjavacollback;

import java.util.concurrent.CountDownLatch;

/**
 * Created by kamal on 15/8/20.
 */
public class Publisher {
    IDataListener listener;
    public Publisher(IDataListener listener){
        this.listener = listener;//获取监听接口，谁实现他就会调用哪个实现函数
    }
    //同步调用
    public void publish(){//发布信息，会通知sucriber，怎么通知呢？通过回调
        try{
            Thread.sleep(3000);//模拟等待一段时间才发布消息，或者处理时间
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        System.out.println("sync:我发布信息啦，你已经告诉我要通知你，所以我来通知你了");
        listener.dataChangeListener();
    }
    //异步调用
    public void publish(CountDownLatch lat){//发布信息，会通知sucriber，怎么通知呢？通过回调
        try{
            Thread.sleep(3000);//模拟等待一段时间才发布消息，或者处理时间
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        System.out.println("async:我发布信息啦，你已经告诉我要通知你，所以我来通知你了");
        listener.dataChangeListener();
        lat.countDown();//已经通知到位
    }
}
{% endcodeblock %}

{% codeblock lang:java  %}
package com.crazyfish.kamal.test.testjavacollback;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * Created by kamal on 15/8/20.
 */
public class Subcriber{

    public void subcribe(){//向Publisher订阅，等到数据变更通知我
        //同步订阅
        Publisher publisher = new Publisher(new IDataListener() {//IDataListener实现，用于被回调
            public void dataChangeListener() {
                System.out.println("sync:我被调用了");
            }
        });
        publisher.publish();
        //异步订阅
        final CountDownLatch lat = new CountDownLatch(1);
        new Thread(new Runnable() {
            public void run() {
                Publisher publisher = new Publisher(new IDataListener() {//IDataListener实现，用于被回调
                    public void dataChangeListener() {
                        System.out.println("async:我被调用了");
                    }
                });
                publisher.publish(lat);
            }
        }).start();
        System.out.println("async:我干其他事情去了，等你发布了通知我哦");
        try{
            boolean rl = lat.await(30L, TimeUnit.SECONDS);
            if( rl ){
                System.out.println("async:我收到通知啦，谢谢通知");
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }

    }
    public static void main(String args[]){
        Subcriber subcriber = new Subcriber();
        subcriber.subcribe();
    }
}
{% endcodeblock %}
