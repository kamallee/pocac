title: 'CyclicBarrier&Semaphore'
date: 2016-05-04 20:10:18
tags: 并发 CyclicBarrier Semaphore
categories: 并发
---
<h2>CyclicBarrier</h2>
字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。
从命名可以看出，CyclicBarrier就像一个栅栏，拦住一批线程（运动员），等到所有线程都执行完成才能让线程继续往下执行（所以运动员都准备好了才能都越过栅栏往前冲刺）。
CountDownLatch: 一个线程(或者多个)， 等待另外N个线程完成某个事情之后才能执行。   CyclicBarrier: N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。
<!--more-->
{% codeblock lang:java %}
package com.crazyfish.kamal.test.TestSomething;
import java.util.concurrent.CyclicBarrier;
/**
 * @author lipengpeng
 * @desc
 * @date 2016-05-05 下午7:25
 */
public class TestCylicBarrier {
    static class Runner extends Thread{
        private CyclicBarrier cylicBarrier;
        public Runner(CyclicBarrier cylicBarrier){
            this.cylicBarrier = cylicBarrier;
        }
        @Override
        public void run(){
            try{
                Thread.sleep(3000);
                System.out.println("线程" + Thread.currentThread().getName() + "执行完毕!");
                cylicBarrier.await();//线程挂起，等待其他线程执行完毕，线程没有结束
            }catch(Exception e){
                e.printStackTrace();
            }
            System.out.println("所有线程执行完毕..");
        }
    }

    public static void main(String args[]){
        CyclicBarrier cyclicBarrier = new CyclicBarrier(4);
        for(int i = 0 ;i < 4;i ++){
            new Runner(cyclicBarrier).start();
        }
        try{
            Thread.sleep(3000);
        }catch(Exception e){}
        for(int i = 0 ;i < 4;i ++){
            new Runner(cyclicBarrier).start();//可以重复使用CyclicBarrier对象，而CountDownLatch不行
        }
    }
}

{% endcodeblock %}

<h2>Semaphore</h2>
Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。
{% codeblock lang:java %}
package com.crazyfish.kamal.test.TestSomething;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

/**
 * @author lipengpeng
 * @desc
 * @date 2016-05-06 下午2:35
 */
public class TestSemaphore {
    public static void main(String args[]){
        ExecutorService executor = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(5);
        for(int i = 0;i < 10;i ++){
            final int who = i;
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    try{
                        semaphore.acquire();
                        System.out.println("谁获得了许可-》" + who);
                        Thread.sleep((long) (Math.random() * 100));
                        semaphore.release();
                        System.out.println("release 剩余可用：" + semaphore.availablePermits());
                    }catch(Exception e){}
                }
            };
            executor.execute(runnable);
        }
        executor.shutdown();
    }
}
{% endcodeblock %}