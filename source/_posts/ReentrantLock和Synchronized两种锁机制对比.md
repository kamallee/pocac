title: ReentrantLock和Synchronized两种锁机制对比
date: 2016-02-24 10:50:42
tags: ReentrantLock Synchronized 锁
categories: 锁
---
并发和分布式编程中锁是必不可少的，没有锁就无法实现资源的贡献，Java为我们提供了RentrantLock和Synchronized两种同步机制。
<!--more-->
概念：
互斥锁：一次最多只能有一个线程持有锁。
公平性：如果在绝对时间上，先对锁进行获取的请求一定被先满足，那么这个锁是公平的，反之，是不公平的，也就是说等待时间最长的线程最有机会获取锁，也可以说锁的获取是有序的。
自旋锁（spinlock）：是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。自旋锁等待期间，线程的状态不会改变，线程一直是用户态并且是活动的(active)。
可重入锁：同一个线程再次进入同步代码的时候，可以使用自己已经获取到的锁,这就是可重入锁。java里面内置锁(synchronize)和Lock(ReentrantLock)都是可重入的。可重入锁又叫做递归锁。

多线程和并发性并不是什么新内容，但是 Java 语言设计中的创新之一就是，它是第一个直接把跨平台线程模型和正规的内存模型集成到语言中的主流语言。
<h2>synchronized</h2>
把代码块声明为synchronized，通常是指该代码具有原子性（atomicity）和可见性（visibility）。原子性意味着一个线程一次只能执行由一个指定监控对象（lock）保护的代码，从而防止多个线程在更新共享状态时相互冲突。可见性则更为微妙；它要对付内存缓存和编译器优化的各种反常行为。一般来说，线程以某种不必让其他线程立即可以看到的方式（不管这些线程在寄存器中、在处理器特定的缓存中，还是通过指令重排或者其他编译器优化），不受缓存变量值的约束，但是如果开发人员使用了同步，如下面的代码所示，那么运行库将确保某一线程对变量所做的更新先于对现有 synchronized 块所进行的更新，当进入由同一监控器（lock）保护的另一个 synchronized 块时，将立刻可以看到这些对变量所做的更新。类似的规则也存在于 volatile 变量上。

所以，实现同步操作需要考虑安全更新多个共享变量所需的一切，不能有争用条件，不能破坏数据（假设同步的边界位置正确），而且要保证正确同步的其他线程可以看到这些变量的最新值。通过定义一个清晰的、跨平台的内存模型，通过遵守下面这个简单规则，构建“一次编写，随处运行”的并发类是有可能的。

不论什么时候，只要您将编写的变量接下来可能被另一个线程读取，或者您将读取的变量最后是被另一个线程写入的，那么您必须进行同步。不过现在好了一点，在最近的JVM中，没有争用的同步（一个线程拥有锁的时候，没有其他线程企图获得锁）的性能成本还是很低的。（也不总是这样；早期JVM中的同步还没有优化，所以让很多人都这样认为，但是现在这变成了一种误解，人们认为不管是不是争用，同步都有很高的性能成本。）

<h2>synchronized的改进</h2>
如此看来同步相当好了，是么？那么为什么 JSR 166 小组花了这么多时间来开发 java.util.concurrent.lock 框架呢？答案很简单－同步是不错，但它并不完美。它有一些功能性的限制 —— 它无法中断一个正在等候获得锁的线程，也无法通过轮询得到锁，如果不想等下去，也就没法得到锁。同步还要求锁的释放只能在与获得锁所在的堆栈帧相同的堆栈帧中进行，多数情况下，这没问题（而且与异常处理交互得很好），但是，确实存在一些非块结构的锁定更合适的情况。

synchronized是托管给JVM执行的，而lock是java写的控制锁的代码。在Java1.5中，synchronize是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用Java提供的Lock对象，性能更高一些。但是到了Java1.6发生了变化synchronize在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在Java1.6上synchronize的性能并不比Lock差。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地。
<h2>ReentrantLock 类</h2>
java.util.concurrent.lock 中的 Lock 框架是锁定的一个抽象，它允许把锁定的实现作为 Java 类，而不是作为语言的特性来实现。这就为 Lock 的多种实现留下了空间，各种实现可能有不同的调度算法、性能特性或者锁定语义。 ReentrantLock 类实现了 Lock ，它拥有与 synchronized 相同的并发性和内存语义，但是添加了类似轮询锁、定时锁等候和可中断锁等候的一些特性。此外，它还提供了在激烈争用情况下更佳的性能。（换句话说，当许多线程都想访问共享资源时，JVM 可以花更少的时候来调度线程，把更多时间用在执行线程上。）

reentrant 锁意味着什么呢？简单来说，它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放。这模仿了 synchronized 的语义；如果线程进入由线程已经拥有的监控器保护的 synchronized 块，就允许线程继续进行，当线程退出第二个（或者后续） synchronized 块的时候，不释放锁，只有线程退出它进入的监控器保护的第一个 synchronized 块时，才释放锁。

代码示例中，可以看到 Lock 和 synchronized 有一点明显的区别 —— lock 必须在 finally 块中释放。否则，如果受保护的代码将抛出异常，锁就有可能永远得不到释放！这一点区别看起来可能没什么，但是实际上，它极为重要。忘记在 finally 块中释放锁，可能会在程序中留下一个定时炸弹，当有一天炸弹爆炸时，您要花费很大力气才有找到源头在哪。而使用同步，JVM 将确保锁会获得自动释放。
{% codeblock lang:java 用 ReentrantLock 保护代码块 %}
Lock lock = new ReentrantLock();
lock.lock();
try { 
  // update object state
}
finally {
  lock.unlock(); 
}
{% endcodeblock %}
除此之外，与目前的 synchronized 实现相比，争用下的 ReentrantLock 实现更具可伸缩性。（在未来的 JVM 版本中，synchronized 的争用性能很有可能会获得提高。）这意味着当许多线程都在争用同一个锁时，使用 ReentrantLock 的总体开支通常要比 synchronized 少得多。

ReentrantLock的lock机制有2种，忽略中断锁和响应中断锁，这给我们带来了很大的灵活性。比如：如果A、B 2个线程去竞争锁，A线程得到了锁，B线程等待，但是A线程这个时候实在有太多事情要处理，就是 一直不返回，B线程可能就会等不及了，想中断自己，不再等待这个锁了，转而处理其他事情。这个时候ReentrantLock就提供了2种机制，第一，B线程中断自己（或者别的线程中断它），但是ReentrantLock 不去响应，继续让B线程等待，你再怎么中断，我全当耳边风（synchronized原语就是如此）；第二，B线程中断自己（或者别的线程中断它），ReentrantLock 处理了这个中断，并且不再等待这个锁的到来，完全放弃。
<h4>方法介绍</h4>
tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

ReentrantReadWriteLock里面提供了很多丰富的方法，主要有两个方法：readLock()和writeLock()用来获取读锁和写锁。如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。
<h2>条件变量</h2>
根类 Object 包含某些特殊的方法，用来在线程的 wait() 、 notify() 和 notifyAll() 之间进行通信。这些是高级的并发性特性，许多开发人员从来没有用过它们 —— 这可能是件好事，因为它们相当微妙，很容易使用不当。幸运的是，随着 JDK 5.0 中引入 java.util.concurrent ，开发人员几乎更加没有什么地方需要使用这些方法了。

通知与锁定之间有一个交互 —— 为了在对象上 wait 或 notify ，您必须持有该对象的锁。就像 Lock 是同步的概括一样， Lock 框架包含了对 wait 和 notify 的概括，这个概括叫作 条件（Condition） 。 Lock 对象则充当绑定到这个锁的条件变量的工厂对象，与标准的 wait 和 notify 方法不同，对于指定的 Lock ，可以有不止一个条件变量与它关联。这样就简化了许多并发算法的开发。例如， 条件（Condition） 的 Javadoc 显示了一个有界缓冲区实现的示例，该示例使用了两个条件变量，“not full”和“not empty”，它比每个 lock 只用一个 wait 设置的实现方式可读性要好一些（而且更有效）。 Condition 的方法与 wait 、 notify 和 notifyAll 方法类似，分别命名为 await 、 signal 和 signalAll ，因为它们不能覆盖 Object 上的对应方法。
回页首
<h2>这不公平</h2>
如果查看 Javadoc，您会看到， ReentrantLock 构造器的一个参数是 boolean 值，它允许您选择想要一个 公平（fair）锁，还是一个 不公平（unfair）锁。公平锁使线程按照请求锁的顺序依次获得锁；而不公平锁则允许直接获取锁，在这种情况下，线程有时可以比先请求锁的其他线程先得到锁。

为什么我们不让所有的锁都公平呢？毕竟，公平是好事，不公平是不好的，不是吗？（当孩子们想要一个决定时，总会叫嚷“这不公平”。我们认为公平非常重要，孩子们也知道。）在现实中，公平保证了锁是非常健壮的锁，有很大的性能成本。要确保公平所需要的记帐（bookkeeping）和同步，就意味着被争夺的公平锁要比不公平锁的吞吐率更低。作为默认设置，应当把公平设置为 false ，除非公平对您的算法至关重要，需要严格按照线程排队的顺序对其进行服务。

那么同步又如何呢？内置的监控器锁是公平的吗？答案令许多人感到大吃一惊，它们是不公平的，而且永远都是不公平的。但是没有人抱怨过线程饥渴，因为 JVM 保证了所有线程最终都会得到它们所等候的锁。确保统计上的公平性，对多数情况来说，这就已经足够了，而这花费的成本则要比绝对的公平保证的低得多。所以，默认情况下 ReentrantLock 是“不公平”的，这一事实只是把同步中一直是事件的东西表面化而已。如果您在同步的时候并不介意这一点，那么在 ReentrantLock 时也不必为它担心。

synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。

而Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是CAS操作（Compare and Swap）。我们可以进一步研究ReentrantLock的源代码，会发现其中比较重要的获得锁的一个方法是compareAndSetState。这里其实就是调用的CPU提供的特殊指令。

现代的CPU提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。这个算法称作非阻塞算法，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。
<h2>还不要抛弃 synchronized</h2>
虽然 ReentrantLock 是个非常动人的实现，相对 synchronized 来说，它有一些重要的优势，但是我认为急于把 synchronized 视若敝屣，绝对是个严重的错误。 java.util.concurrent.lock 中的锁定类是用于高级用户和高级情况的工具 。一般来说，除非您对 Lock 的某个高级特性有明确的需要，或者有明确的证据（而不是仅仅是怀疑）表明在特定情况下，同步已经成为可伸缩性的瓶颈，否则还是应当继续使用 synchronized。

为什么我在一个显然“更好的”实现的使用上主张保守呢？因为对于 java.util.concurrent.lock 中的锁定类来说，synchronized 仍然有一些优势。比如，在使用 synchronized 的时候，不能忘记释放锁；在退出 synchronized 块时，JVM 会为您做这件事。您很容易忘记用 finally 块释放锁，这对程序非常有害。您的程序能够通过测试，但会在实际工作中出现死锁，那时会很难指出原因（这也是为什么根本不让初级开发人员使用 Lock 的一个好理由。）

另一个原因是因为，当JVM用synchronized管理锁定请求和释放时，JVM在生成线程转储时能够包括锁定信息。这些对调试非常有价值，因为它们能标识死锁或者其他异常行为的来源。 Lock 类只是普通的类，JVM 不知道具体哪个线程拥有 Lock 对象。而且，几乎每个开发人员都熟悉 synchronized，它可以在 JVM 的所有版本中工作。在 JDK 5.0 成为标准（从现在开始可能需要两年）之前，使用 Lock 类将意味着要利用的特性不是每个 JVM 都有的，而且不是每个开发人员都熟悉的。
<h2>什么时候选择用 ReentrantLock 代替 synchronized</h2>
既然如此，我们什么时候才应该使用 ReentrantLock 呢？答案非常简单 —— 在确实需要一些 synchronized 所没有的特性的时候，比如时间锁等候、可中断锁等候、无块结构锁、多个条件变量或者轮询锁。 ReentrantLock 还具有可伸缩性的好处，应当在高度争用的情况下使用它，但是请记住，大多数 synchronized 块几乎从来没有出现过争用，所以可以把高度争用放在一边。我建议用 synchronized 开发，直到确实证明 synchronized 不合适，而不要仅仅是假设如果使用 ReentrantLock “性能会更好”。请记住，这些是供高级用户使用的高级工具。（而且，真正的高级用户喜欢选择能够找到的最简单工具，直到他们认为简单的工具不适用为止。）。一如既往，首先要把事情做好，然后再考虑是不是有必要做得更快。

Lock 框架是同步的兼容替代品，它提供了 synchronized 没有提供的许多特性，它的实现在争用下提供了更好的性能。但是，这些明显存在的好处，还不足以成为用 ReentrantLock 代替 synchronized 的理由。相反，应当根据您是否 需要 ReentrantLock 的能力来作出选择。大多数情况下，您不应当选择它 —— synchronized 工作得很好，可以在所有 JVM 上工作，更多的开发人员了解它，而且不太容易出错。只有在真正需要 Lock 的时候才用它。在这些情况下，您会很高兴拥有这款工具。
{% codeblock lang:java 代码实例 %}
package com.crazyfish.kamal.test.TestSomething;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author lipengpeng
 * @desc
 * @date 2016-02-24 下午1:51
 */
public class Buffer {
    private Object lock = this;
    private ReentrantLock lock1 = new ReentrantLock();
    private Condition condition = lock1.newCondition();
    public void write(){
//        synchronized (lock){
//            long startTime = System.currentTimeMillis();
//            System.out.println("start write");
//            for(;;){
//                if( System.currentTimeMillis() - startTime > Integer.MAX_VALUE)
//                    break;
//            }
//            System.out.println("write end");
//        }
        lock1.lock();
        try{
            long startTime = System.currentTimeMillis();
            System.out.println("start write");
            for(;;){
                long start = System.currentTimeMillis();
                for (;;) {
                    if (System.currentTimeMillis() - start > 2000) {
                        System.out.println("太累了，休息一下，主动放弃锁");
                        condition.await();
                        System.out.println("重新获得锁继续执行");
                        break;
                    }
                }
                if( System.currentTimeMillis() - startTime > Integer.MAX_VALUE)
                    break;
            }
            System.out.println("write end");
        } catch (Exception e) {
            e.printStackTrace();
        } finally{
            lock1.unlock();
        }
    }
    public void read()throws InterruptedException{
//        synchronized (lock){
//            System.out.println("read");
//        }
        //lock1.lockInterruptibly();
        if(lock1.tryLock(2, TimeUnit.SECONDS)){//1改成>2就可以了
            System.out.println("尝试获得锁成功，获得锁");
        }else{
            System.out.println("尝试失败，没有获得锁");
        }
        try{
            System.out.println("read");
            //condition.signal();//发送信号让write继续执行
        }finally {
            lock1.unlock();//如果没有获得锁，放弃锁会抛异常
        }
    }
    public static void main(String args[]){
        Buffer buffer = new Buffer();
        final Writer writer = new Writer(buffer);
        final Reader reader = new Reader(buffer);
        writer.start();
        reader.start();
        //等待一段时间发出中断请求，Syn不能中断，Lock可以
        new Thread(new Runnable() {
            @Override
            public void run() {
                long start = System.currentTimeMillis();
                for (;;) {
                    if (System.currentTimeMillis() - start > 5000) {
                        //System.out.println("不等了，尝试中断");
                        //reader.interrupt();
                        break;
                    }
                }
            }
        }).start();
    }
}
class Writer extends Thread{
    private Buffer buff;
    public Writer(Buffer buf){
        this.buff = buf;
    }
    @Override
    public void run(){
        buff.write();//使用并发代码
    }
}
class Reader extends Thread{
    private Buffer buff;
    public Reader(Buffer buff){
        this.buff = buff;
    }
    @Override
    public void run(){
        try {
            buff.read();//收到异常
        } catch (InterruptedException e) {
            System.out.println("不等待锁了，退出");
        }
        System.out.println("read end");
    }
}
{% endcodeblock %}