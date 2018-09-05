title: java并发之线程池
date: 2016-01-27 16:03:36
tags: java 线程池
categories: 线程池
---
线程池（Thread Pool）对于限制应用程序中同一时刻运行的线程数很有用。为什么要限制线程数目呢？因为每启动一个新线程都会有相应的性能和资源开销，每个线程都需要给栈分配一些内存资源等，线程切换时，上下文切换会消耗大量CPU时间。

我们可以把可以并发执行的任务传递给一个线程池，来替代为每个并发执行的任务都启动一个新的线程。只要池里有空闲的线程，任务就会分配给一个线程执行。在线程池的内部，任务被插入一个阻塞队列（Blocking Queue），线程池里的线程会去取这个队列里的任务。当一个新任务插入队列时，一个空闲线程就会成功的从队列中取出任务并且执行它。

线程池经常应用在多线程服务器上。每个通过网络到达服务器的连接都被包装成一个任务并且传递给线程池。线程池的线程会并发的处理连接上的请求。Java5之后，引入Executor框架，内部使用线程池机制，在java.util.concurrent包下，通过该框架控制线程的启动执行和关闭，大大简化并发编程的操作。
<!--more-->
通过使用Executor启动线程比使用Thread的start方法更易管理，效率更好（线程池，几乎可以不用创建线程），并且避免this指针逃逸问题。
<h2>线程池Executor框架</h2>
{% img /img/executor.png java线程池Executor框架 %}
ExecutorService 的生命周期包括三种状态：运行、关闭、终止。创建后便进入运行状态，当调用了 shutdown（）方法时，便进入关闭状态，此时意味着 ExecutorService 不再接受新的任务，但它还在执行已经提交了的任务，当把已经提交了的任务执行完后，便到达终止状态。如果不调用 shutdown（）方法，ExecutorService 会一直处在运行状态，不断接收新的任务，执行新的任务，服务器端一般不需要关闭它，保持一直运行即可。
<h2>ThreadPoolExecutor介绍及参数解释</h2>
从图中可以看出，ThreadPoolExecutor(corePoolSize,maximumPoolSize, keepAliveTime, milliseconds,runnableTaskQueue, handler)是最重要的实现类，Executors静态工厂类提供的静态方法都来自它，只不过参数不同而已。
<h3>ThreadPoolExecutor参数解释</h3>
1).corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。

2).runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。 可以选择以下几个阻塞队列。
	a.ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
	b.LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
	c.SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
	d.PriorityBlockingQueue：一个具有优先级的无限阻塞队列。
3).maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。

4).ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。

5).RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。
	a.AbortPolicy：直接抛出异常。
	b.CallerRunsPolicy：只用调用者所在线程来运行任务。
	c.DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
	d.DiscardPolicy：不处理，丢弃掉。
	e.当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。

6).keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。

7).TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

<h3>ThreadPoolExecutor通过execute方法执行任务时策略</h3>

1.如果线程池中的线程数量少于 corePoolSize，即使线程池中有空闲线程，也会创建一个新的线程来执行新添加的任务；

2.如果线程池中的线程数量大于等于 corePoolSize，但缓冲队列 workQueue 未满，则将新添加的任务放到 workQueue 中，按照 FIFO 的原则依次等待执行（线程池中有线程空闲出来后依次将缓冲队列中的任务交付给空闲的线程执行）；

3.如果线程池中的线程数量大于等于 corePoolSize，且缓冲队列 workQueue 已满，但线程池中的线程数量小于 maximumPoolSize，则会创建新的线程来处理被添加的任务；

4.如果线程池中的线程数量等于了 maximumPoolSize，有 4 种才处理方式（该构造方法调用了含有 5 个参数的构造方法，并将最后一个构造方法为 RejectedExecutionHandler 类型，它在处理线程溢出时有 4 种方式）。

另外，当线程池中的线程数量大于 corePoolSize 时，如果里面有线程的空闲时间超过了 keepAliveTime，就将其移除线程池，这样，可以动态地调整线程池中线程的数量。
<h2>Executors介绍</h2>
但是JDKDoc并不建议直接使用该方法，而是使用封装了的Executors中的静态方法，能够满足大部分场景的使用。Executors中主要的方法有：

1.newFixedThreadPool(...)
newFixedThreadPool 与 cacheThreadPool 差不多，也是能 reuse 就用，但不能随时建新的线程。
其独特之处:任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。它使用无界队列（采用预定义容量的 LinkedBlockingQueue，理论上是该缓冲队列可以对无限多的任务排队）。和 cacheThreadPool 不同，FixedThreadPool 没有 IDLE 机制（可能也有，但既然文档没提，肯定非常长，类似依赖上层的 TCP 或 UDP IDLE 机制之类的），所以 FixedThreadPool 多数针对一些很稳定很固定的正规并发线程，多用于服务器。从方法的源代码看，cache池和fixed 池调用的是同一个底层 池，只不过参数不同:fixed 池线程数固定，并且是0秒IDLE（无IDLE）。cache 池线程数支持 0-Integer.MAX_VALUE(显然完全没考虑主机的资源承受能力），60 秒 IDLE 。

2.newSingleThreadExecutor(...)
用的是和 cache 池和 fixed 池相同的底层池，但线程数目是 1-1,0 秒 IDLE（无 IDLE）SingleThreadExecutor(...)
返回单线程的Executor，将多个任务交给此Exector时，这个线程处理完一个任务后接着处理下一个任务，若该线程出现异常，将会有一个新的线程来替代。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
说明：LinkedBlockingQueue会无限的添加需要执行的Runnable。

3.newCachedThreadPool(...)
如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。核心线程数量为0。
SynchronousQueue是个特殊的队列。 SynchronousQueue队列的容量为0。当试图为SynchronousQueue添加任务时，执行会失败。只有当一边从SynchronousQueue取数据，一边向SynchronousQueue添加数据才可以成功。SynchronousQueue仅仅起到数据交换的作用，并不保存线程。但newCachedThreadPool()方法没有线程上限。Runable添加到SynchronousQueue会被立刻取出。
根据用户的任务数创建相应的线程来处理，该线程池不会对线程数目加以限制，完全依赖于JVM能创建线程的数量，可能引起内存不足。
他会先查看池中有没有以前建立的线程，如果有，就 reuse 如果没有，就建一个新的线程加入池中
缓存型池子通常用于执行一些生存期很短的异步型任务 因此在一些面向连接的 daemon 型 SERVER 中用得不多。但对于生存期短的异步任务，它是 Executor 的首选。
能 reuse 的线程，必须是 timeout IDLE 内的池中线程，缺省 timeout 是 60s,超过这个 IDLE 时长，线程实例将被终止及移出池。缓冲队列采用 SynchronousQueue。
注意，放入 CachedThreadPool 的线程不必担心其结束，超过 TIMEOUT 不活动，其会自动被终止。
一般来说，CachedTheadPool 在程序执行过程中通常会创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程，因此它是合理的 Executor 的首选，只有当这种方式会引发问题时（比如需要大量长时间面向连接的线程时），才需要考虑用 FixedThreadPool。（摘自《Thinking in Java》第四版）

4.newScheduledThreadPool(...)
调度型线程池
这个池子里的线程可以按 schedule 依次 delay 执行，或周期执行

5.newSingleThreadScheduledExecutor()
此线程池支持定时以及周期性执行任务的需求

{% codeblock lang:java Executor框架demo %}
package com.crazyfish.kamal.test.testthreadpool;

import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.*;

/**
 * @author lipengpeng
 * @desc 并发包线程池使用
 * @date 2016-01-27 上午11:06
 */
public class TestThreadPool {
    public static void main(String args[]) throws InterruptedException {
        //基本方法，不建议直接使用，建议优先使用Execurors中的静态方法
        ThreadPoolExecutor service = new ThreadPoolExecutor(4,4,0, TimeUnit.MILLISECONDS,new LinkedBlockingDeque<Runnable>(10)){
            @Override
            protected void beforeExecute(Thread t, Runnable r) {
                System.out.println("准备执行:" + ((Task)r).getName());
            }

            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.println("执行完成:" + ((Task)r).getName());
            }

            @Override
            protected void terminated() {
                System.out.println("执行退出");
            }
        };

        for(int i = 0;i < 10;i ++){
            //service.execute(new Task("task" + i));
            //System.out.println("线程池中线程数目：" + service.getPoolSize() + "，队列中等待执行的任务数目：" +
             //       service.getQueue().size() + "，已执行完的任务数目：" + service.getCompletedTaskCount());
        }
        //固定大小线程池
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for(int i = 0;i < 10;i ++){
            //executorService.execute(new Task());
        }
        //有返回值的异步任务
        List<Future<String>> list = new LinkedList<Future<String>>();
        for(int i = 0;i < 10;i ++){
            Future<String> rlt = executorService.submit(new MyCallable("task" + i));
            list.add(rlt);
            //这种使用列表保存返回结果稍微有点复杂，而且有不足的地方。如果第一个任务耗费非常长的时间来执行，然后其他的任务都早于它结束，
            // 那么当前线程就无法在第一个任务结束之前获得执行结果，别着急，Java 为你提供了解决方案——CompletionService。
        }
        for(Future<String> f : list){
            try {
                System.out.println("rlt:" + f.get(3,TimeUnit.SECONDS));
            }catch(Exception e){
                e.printStackTrace();
            }
        }

        //使用CompletionService
        CompletionService<String> pool = new ExecutorCompletionService<String>(executorService);
        for(int i = 0;i < 10;i ++){
            pool.submit(new MyCallable("task" + i));
        }
        try {
            for(int i = 0;i < 10;i ++){
                    String rlt = pool.take().get();
                    System.out.println("comp:" + rlt );
            }
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println(executorService.isShutdown() + "" + executorService.isTerminated());
        //平缓的关闭线程池。线程池停止接受新的任务，同时等待已经提交的任务执行完毕，包括那些进入队列还没有开始的任务。shutdown()方法执行过程中，线程池处于SHUTDOWN状态
        //executorService.shutdown();//如果你不这么做，JVM 并不会去关闭这些线程
        //立即关闭线程池。线程池停止接受新的任务，同时线程池取消所有执行的任务和已经进入队列但是还没有执行的任务
        executorService.shutdownNow();
        try {
            Thread.sleep(2000);//等待线程池中的任务执行完成
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(executorService.isShutdown() + "" + executorService.isTerminated());
    }

    static class Task implements Runnable{
        private String name;
        public Task(String name){
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + name + " run ...");
        }
    }

    static class MyCallable implements Callable<String>{
        private String name;
        public MyCallable(String name){
            this.name = name;
        }

        @Override
        public String call() throws Exception {
            return "task " + name + " return result";
        }
    }
}
{% endcodeblock %}