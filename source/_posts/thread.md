---
title: 多线程与高并发
date: 2020-02-16 22:05:29
toc: true
description: 线程作为一个进程里面最小的执行单元它就叫一个线程，用简单的话讲一个程序里不同的执行路径就叫做一个线程
tags: 
- Thread
---
线程: 作为一个进程里面最小的执行单元它就叫一个线程，用简单的话讲一个程序里不同的执行路径就叫做一个线程
#### 一、创建线程的几种方式：
###### 1. 继承Thread，并重写该类的run方法
<!--more-->
```java
public class MyThread extends Thread{

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName()+" 是继承写法。");
    }

    public static void main(String[] args){
        MyThread thread = new MyThread();
        thread.setName("extends thread");
        thread.start();
    }

}
```

###### 2. 通过Runnable接口创建线程类,创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。
```java
public class MyThread implements Runnable {

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName()+" 写法。");
    }

    public static void main(String[] args){
        Thread t = new Thread(new MyThread(),"implements Runnable");
        t.start();
    }

}
```

###### 3. 通过Callable和Future创建线程
    - 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
    - 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。
    - 使用FutureTask对象作为Thread对象的target创建并启动新线程。
    - 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
```java
public class MyCallableThread implements Callable<Integer> {

    @Override
    public Integer call(){
        System.out.println(Thread.currentThread().getName()+" 写法。");
        return 1;
    }

    public static void main(String[] args){
        FutureTask<Integer> task = new FutureTask<Integer>(new MyCallableThread());
        new Thread(task,"implements Callable").start();
        try {
            System.out.println("有返回值：" + task.get());
        } catch (InterruptedException e){
            e.printStackTrace();
        } catch (ExecutionException e){
            e.printStackTrace();
        }
    }

}

```


#### 二、 线程操作的几个方法  
* start: 启动一个新的线程,start方法必须子线程第一个调用的方法，start不能够重复调用，新线程会调用runnable接口提供的run方法
* run: run方法是子线程的执行体，子线程从进入run方法开始直至run方法执行接收意味着子线程的任务执行接收， 在主线程直接调用run方法是不能创建子线程，只是普通方法调用
* sleep: 睡眠，当前线程暂停一段时间让给别的线程去运行。Sleep是怎么复活由睡眠时间而定，等睡眠到规定的时间自动复活.
* Yield: 就是当前线程正在执行的时候停止下来进入等待队列，回到等待队列里在系统的调度算法里头，还是依然有可能把你刚回去的这个线程拿回来继续执行，当然，更大的可能性是把原来等待的那些拿出一个来执行，所以yield的意思是我让出一下CPU，后面你们能不能抢到那我不管
* join: 意思就是在自己当前线程加入你调用Join的线程()，本线程等待。等调用的线程运行 完了，自己再去执行。t1和t2两个线程，在t1的某个点上调用了t2.join,它会跑到t2去运行，t1等待t2运 行完毕继续t1运行(自己join自己没有意义)
  
wait() 和 notify() 方法说明几点：
1. 调用notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中**随机**选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。
2. 除了notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用 notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有获得锁的那一个线程才能进入可执行状态。
3. wait()法需要释放锁，所以必须在synchronized中使用，否则会抛出异常 IllegalMonitorStateException
4. notify()方法也必须在synchronized中使用，并且应该指定对象 
5. synchronized()、wait()、notify()对象必须一致，一个synchronized()代码块中只能有一个线程调

#### 三、线程六种状态  
1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
2. 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
3. 阻塞(BLOCKED)：表示线程阻塞于锁。
4. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
5. 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
6. 终止(TERMINATED)：表示该线程已经执行完毕。
![线程状态转换图](thread/status.jpg)

*线程什么状态时候会被挂起?挂起是否也是一个状态?  
Running的时候，在一个cpu上会跑很多个线程，cpu会隔一段时间执行这个线程一下，在隔一段时间执行那个线程一下，这个是cpu内部的一个调度，从running扔回去就叫线程被挂起。

#### 四、synchronized  
多个线程去访问同一个资源的时候对这个资源上锁  

* synchronized 使用方法和特性
    - 在同步代码外嵌套synchronized(Object/Class)
    - synchronized方法和synchronized(this)执行这段代码它是等值的
    - 每次都定义个一个锁的对象Object o 把它new出来，那加锁的时候太麻烦每次都要new一个新的对象出来，所以呢，有一个简单的方式就是 synchronized(this)锁定当前对象就行
    - 静态方法static是没有this对象的，你不需要new出一个对象来就能执行这个方法，但如果这个这个上面加一个synchronized的话就代表synchronized(T.class)。这里这个synchronized(T.class)锁的就是T类的对象
    - 类锁和对象锁互不干扰，只有当监视的是同一个class（Class对象）的锁或同一个对象实例的锁才发生互斥
    - 可重入，一个同步方法可以调用另外一个同步方法，一个线程已经拥有某个对象的锁，再次申请的时候仍然会得到该对象的锁
    - 程序在执行过程中，如果出现异常，默认情况锁会被释放所以，在并发处理的过程中，有异常要多加小心，不然可能会发生不一致的情况。比如，在一个web app处理过程中，多个servlet线程共同访问同一个资源，这时如果异常处理不合适， * 在第一个线程中抛出异常，其他线程就会进入同步代码区，有可能会访问到异常产生时的数据。因此要非常小心的处理同步业务逻辑中的异常
    - 如果锁的是new出来的对象，在某一种特定的不小心的情况下你把o变成了别的对象了，这个时候线程的并发就会出问题。锁是在对象的头上两位来作为代表的，你这线程本来大家都去访问这两位了，结果突然把 这把锁变成别的对象，去访问别的对象的两位了，这俩之间就没有任何关系了。因此，以对象作为锁的 时候不让它发生改变，加final。

#### 五、锁升级的概念  
原来要去找操作系统，要找内核去申请这把锁，到后期做了对 synchronized的一些改进，他的效率比原来要改变了不少，改进的地方。当我们使用synchronized的 时候HotSpot的实现是这样的:  
上来之后第一个去访问某把锁的线程 比如sync (Object) ，来了之后先在 这个Object的头上面markword记录这个线程。(如果只有第一个线程访问的时候实际上是没有给这个 Object加锁的，在内部实现的时候，只是记录这个线程的ID(偏向锁))。    
偏向锁如果有线程争用的话，就升级为自旋锁，概念就是(有一个哥们儿在蹲马桶 ，另外来了一个哥 们，他就在旁边儿等着，他不会跑到cpu的就绪队列里去，而就在这等着占用cpu，用一个while的循环 在这儿转圈玩儿， 很多圈之后不行的话就再一次进行升级)。    
自旋锁转圈十次之后，升级为重量级锁，重量级锁就是去操作系统那里去申请资源。这是一个锁升级的过程。  

#### 六、volatile   
使一个变量在多个线程中可见，保证线程的可见性，同时防止指令重排序。线程可见性在CPU的级别是用缓存一直性来保 证的;禁止指令重排序CPU级别是你禁止不了的，那是人家内部运行的过程，提高效率的。但是在 虚拟机级别你家volatile之后呢，这个指令重排序就可以禁止。严格来讲，还要去深究它的内部的 话，它是加了读屏障和写屏障，这个是CPU的一个原语。  
A B线程都用到一个变量，java默认是A线程中保留一份copy,这样如果B线程修改了该变量，则A线程未必知道，使用volatile关键字，会让所有线程都会读到变量的修改值  
并不能保证多个线程共同修改running变量时所带来的不一致问题，也就是说volatile不能替代synchronized  

#### 七、CAS 比较和交换（Conmpare And Swap)
它将内存位置的内容与给定值进行比较，只有在相同的情况下，将该内存位置的内容修改为新的给定值。 这是作为单个原子操作完成的。 原子性保证新值基于最新信息计算; 如果该值在同一时间被另一个线程更新，则写入将失败。  
凡是以Atomic开头的都是用CAS这种操作来保证线程安全的这么一些个类。AtomicInteger的意思就是里面包了一个Int类型，这个int类型的自增 count++ 是线程安全的，还有拿值等等是线程安全的，由于我们在工作开发中经常性的有那种需求，一个值所有的线程共同访问它往 上递增 ，所以jdk专门提供了这样的一些类。  
它的内部调用，就会跑到Unsafe类去(不安全的)，Unsafe中对CAS的实现是C++写的。也就是说AtomicInteger它的内部是调用了 Unsafe这个类里面的方法CompareAndSetI(CAS)。这个比较并且设定的意思是什么呢，我原来想改变某一个值0 ，我想把它变成1，但是其中我想做到线程安全，就只能加锁synchronized ，不然线程就不安全。我现在可以用另外一种操作来替代这把锁，就是cas操作，你可以把它想象成一个方法，这个方法有三个参数，cas(V，Expected，NewValue)。  
V第一个参数是要改的那个值;Expected第二个参数是期望当前的这个值会是几;NewValue要设定的新值。当前这个线程想改这个值的时候我期望你这值就是0，你不能是个1，如果是1就说明我这值不对，然后想把你变成1。  
当你判断的时候，发现是我期望的值，还没有进行新值设定的时候值发生了改变怎么办，cas是cpu的原语支持，也就是说cas操作是cpu指令级别上的支持，中间不能被打断。  
 
ABA问题：  
假如说你有一个值，我拿到这个值是1，想把它变成2，我拿到1用cas操作，期望值是1，准备变成2，这个对象Object，在这个过程中，没有一个线程改过我肯定是可以更改的，但是 如果有一个线程先把这个1变成了2后来又变回1，中间值更改过，它不会影响我这个cas下面操作，这就是ABA问题。 这种问题怎么解决。如果是int类型的，最终值是你期望的，也没有关系，这种没关系可以不去管这个问题。如果你确实想管这个问题可以加版本号，做任何一个值的修改，修改完之后加一，后面检查的时候连带版本号一起检查。  

#### 八、Atomic 类  
AtomXXX类本身方法都是原子性的，但不能保证多个方法连续调用是原子性的  
多线程对一个数进行递增方法:
1. 一个long类型的数，递增的时候我们加锁; 
2. 使用AtomicLong可以让它不断的往上递增;
3. LongAdder;
```java
public class Incrementer {
    static Long counter1 = new Long(0L);
    static AtomicLong counter2 = new AtomicLong(0L);
    static LongAdder counter3 = new LongAdder();
    final static Object o = new Object();

    public static void main(String[] args) {
        
        //synchronized
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    synchronized (o) {  //去掉锁之后结果就有问题
                    counter1= counter1+1;
                    }
                }
            }).start();
        }

        //AtomicLong
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    counter2.incrementAndGet();
                }
            }).start();
        }

        //LongAdder
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    counter3.increment();
                }
            }).start();
        }


        try {
            //简单处理，休眠主线程等上面的计算线程完成。
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println(counter1);
        System.out.println(counter2);
        System.out.println(counter3);

    }
}
```

#### 九、基于CAS的一些新类型的锁
###### 1. RenentrantLock 可重入锁
* ReentrantLock是可以替代synchronized的,
* 需要手动枷锁，手动解锁、可以出现多个不同的等待队列
* ReentrantLock有一些功能还是要比synchronized强大的，强大的地方，你可以使用tryLock进行尝试 锁定，不管锁定与否，方法都将继续执行，synchronized如果搞不定的话他肯定就阻塞了，但是用 ReentrantLock你自己就可以决定你到底要不要wait。
* 原来写synchronized的地方换 写lock.lock()，加完锁之后需要注意的是记得lock.unlock()解锁，由于synchronized是自动解锁的，大括号执行完就结束了。lock就不行，lock必须得手动解锁，手动解锁一定要写在try...finally里面保证最好一定要解锁，不然的话上锁之后中间执行的过程有问题了，死在那了，别人就永远也拿不到这把锁了。
* ReentrantLock还可以用lock.lockInterruptibly()这个类，对interrupt()方法做出相应，可以被打断的加锁，如果以这种方式加锁的话我们可以调用一个t2.interrupt(); 打断线程2的等待。
* ReentrantLock还可以指定为公平锁，公平锁的意思是当我们new一个ReentrantLock你可以传一个参数为true，这个true表示公平锁，公平锁的意思是谁等在前面就先让谁执行，而不是说谁后来了之后就马上让谁执行。如果说这个锁不公平，来了一个线程上来就抢，它是有可能先抢到的。（是否公平锁分别有NonfairSync & FairSync 两个不同的实现）
* 除了synchronized之外，多数内部都是用的cas。AQS的实际上它内部用的是 park和unpark，也不是全都用的cas,他还是做了一个锁升级的概念，只不过这个锁升级做的比较隐秘， 在等待这个队列的时候如果你拿不到还是进入一个阻塞的状态，前面至少有一个cas的状态，他不像原先就直接进入阻塞状态了。（参考后面的源码阅读部分）
```java
public class ReentrantLockTest {
    static Integer i = new Integer(0);
    static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        for (int j = 0; j < 10000; j++) {
            new Thread(() -> {
                for (int k = 0; k < 100 ; k++) {
                    lock.lock();
                    try {
                        i++;
                    } finally {
                        // 必须要必须要必须要手动释放锁,必须要必须要必须要手动释放锁,必须要必须要必须要手动释放锁(重要的事情说三遍)
                        // 使用syn锁定的话如果遇到异常，jvm会手动释放锁，但是lock必须手动释放锁
                        lock.unlock();
                    }
                }
            }).start();
        }

        try {
            //简单处理，休眠主线程等上面的计算线程完成。
            TimeUnit.SECONDS.sleep(5);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        System.out.println(i);
    }
}
```

* ReentrantLock 非公平锁原码阅读
```
     ReentrantLock lock = new ReentrantLock();
     lock.lock();  //从断点跟踪
     
     public void lock() {
         sync.acquire(1);
     }
     
     public final void acquire(int arg) {
         if (!tryAcquire(arg) &&
             acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
             selfInterrupt();
     }
     
     final boolean nonfairTryAcquire(int acquires) {
         final Thread current = Thread.currentThread();
         int c = getState(); //the current value of synchronization state.
         if (c == 0) {
            //如果当前对象没有锁，直接设置一个排它锁
             if (compareAndSetState(0, acquires)) {
                 setExclusiveOwnerThread(current);
                 return true;
             }
         }
         else if (current == getExclusiveOwnerThread()) {
            // 如果当前持有锁的是同一个线程，则设置为可重入
             int nextc = c + acquires;
             if (nextc < 0) // overflow
                 throw new Error("Maximum lock count exceeded");
             setState(nextc);
             return true;
         }
         return false; //没拿到锁，返回fallse
     }
     
     
    
     private Node addWaiter(Node mode) {
         //mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared, 
         //now 一个共享锁或排它锁节点
         Node node = new Node(mode);  
        
         //取出队列中最后一个节点，设置为该新节点的上一个节点。并返回新节点。
         //如果最后一个节点为空，初使化一个同步队列
         for (;;) {
             Node oldTail = tail;
             if (oldTail != null) {
                 node.setPrevRelaxed(oldTail);
                 if (compareAndSetTail(oldTail, node)) {
                     oldTail.next = node;
                     return node;
                 }
             } else {
                 initializeSyncQueue();
             }
         }
    }
    
    final boolean acquireQueued(final Node node, int arg) {
        boolean interrupted = false;
        try {
            for (;;) {
                final Node p = node.predecessor();
                //取出前面新new 节点，判断上一个节点，是不是头部节点，如果是的话，直接再次尝试拿锁。
                //如果拿锁成功，那新节点，就是头部节点
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    return interrupted;
                }
                //判断是否继续自旋拿锁，还是park
                if (shouldParkAfterFailedAcquire(p, node))
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            if (interrupted)
                selfInterrupt();
            throw t;
        }
    }
 
```

###### 2.CountDownLatch  
倒数，Latch是门栓的意思(倒数的一个门栓，5、4、3、2、1数到了，我这个门栓就开 了)
刚前面的递增方法，用的休眠主线程等计算线程完成后，再打印结果，这并不好，我们用CountDownLatch 改造下：
```java
public class Incrementer {
    
    static Long counter1 = Long.valueOf(0);
    static CountDownLatch latch = new CountDownLatch(10000);

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    synchronized (Incrementer.class) {  //去掉锁之后结果就有问题
                        counter1++;
                        latch.countDown();
                    }
                    try {
                        TimeUnit.MILLISECONDS.sleep(10);
                        //添加休眠，方便对比结果。
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }

        System.out.println("创建线程完成,等待结果。");
        try {
            latch.await(); //这里会阻塞住，等latch 倒数到0的时候，才会继续执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(counter1);
    }
}
```

###### 3. CyclicBarrier  
循环栅栏，这有一个栅栏，什么时候人满了就把栅栏推倒， 哗啦哗啦的都放出去，出去之后扎栅栏又重新起来，再来人，满了，推倒之后又继续。
```java
public class BusDispatcher {

    static CountDownLatch latch = new CountDownLatch(100);
    static volatile int num = 0;
    static CyclicBarrier barrier = new CyclicBarrier(10, ()->{
        System.out.println(Thread.currentThread().getName()+" - 人满了发车。。。。。");
    });

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                try {
                    System.out.println("第"+(++num)+"个人上车");
                    latch.countDown();
                    barrier.await();
                } catch (InterruptedException e){
                    e.printStackTrace();
                } catch (BrokenBarrierException e){
                    e.printStackTrace();
                }
            }).start();
        }
        try {
            latch.await();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        System.out.println("人车调度任务完成。");
    }
}
```

###### 4. Phaser  
Phaser它就更像是结合了CountDownLatch和CyclicBarrier，翻译一下叫阶段。  
Phaser是按照不同的阶段来对线程进行执行，就是它本身是维护着一个阶段这样的一个成员变量，当前 我是执行到那个阶段，是第0个，还是第1个阶段啊等等，每个阶段不同的时候这个线程都可以往前走， 有的线程走到某个阶段就停了，有的线程一直会走到结束。你的程序中如果说用到分好几个阶段执行 ， 而且有的人必须得几个人共同参与的一种情形的情况下可能会用到这个Phaser。  
```java
public class PhaserTest {

    static GamePhaser phaser = new GamePhaser(3);

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                sleepSeconds(1);
                System.out.println(Thread.currentThread().getName()+" 到");
                phaser.arriveAndAwaitAdvance();

                System.out.println(Thread.currentThread().getName()+"玩第一个游戏。。。");
                sleepSeconds(2);
                phaser.arriveAndAwaitAdvance();

                System.out.println(Thread.currentThread().getName()+"玩第二个游戏。。。");
                sleepSeconds(3);
                phaser.arriveAndAwaitAdvance();

                System.out.println(Thread.currentThread().getName()+"玩第三个游戏。。。");
                sleepSeconds(4);
                phaser.arriveAndAwaitAdvance();

                System.out.println(Thread.currentThread().getName()+"玩第四个游戏。。。");
                sleepSeconds(5);
                phaser.arriveAndAwaitAdvance();

                System.out.println(Thread.currentThread().getName()+"准备离开。。。");
                sleepSeconds(2);
                phaser.arriveAndAwaitAdvance();
            },"00"+(i+1)).start();
        }
    }

    private static void sleepSeconds(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    static class GamePhaser extends Phaser {

        public GamePhaser(int parties) {
            super(parties);
        }

        /**
         * @param phase             第几个阶段，
         * @param registeredParties 当前有多少线程参与
         * @return if this phaser should terminate, 是否要结束phaser
         */
        @Override
        protected boolean onAdvance(int phase, int registeredParties) {
            switch (phase) {
                case 0:
                    System.out.println("registeredParties:" + registeredParties);
                    System.out.println("完成第" + phase + "阶段， 当前时间(秒)" + System.currentTimeMillis() / 1000);
                    System.out.println("人到齐了开始玩第一个游戏^-^");
                    System.out.println("");
                    return false;
                case 1:
                    System.out.println("registeredParties:" + registeredParties);
                    System.out.println("完成第" + phase + "阶段， 当前时间(秒)" + System.currentTimeMillis() / 1000);
                    System.out.println("开始玩第二个游戏^-^");
                    System.out.println("");
                    return false;
                case 2:
                    System.out.println("registeredParties:" + registeredParties);
                    System.out.println("完成第" + phase + "阶段， 当前时间(秒)" + System.currentTimeMillis() / 1000);
                    System.out.println("开始玩第三个游戏^-^");
                    System.out.println("");
                    return false;
                case 3:
                    System.out.println("registeredParties:" + registeredParties);
                    System.out.println("完成第" + phase + "阶段， 当前时间(秒)" + System.currentTimeMillis() / 1000);
                    System.out.println("开始玩第四个游戏^-^");
                    System.out.println("");
                    return false;
                case 4:
                    System.out.println("registeredParties:" + registeredParties);
                    System.out.println("完成第" + phase + "阶段， 当前时间(秒)" + System.currentTimeMillis() / 1000);
                    System.out.println("游戏结束了，谢谢参与 ^-^");
                    System.out.println("");
                    return true;
                default:
                    return true;
            }
        }
    }
}
```

###### 5. ReadWriteLock
这个ReadWriteLock 是读写锁。读写锁的概念其实就是共享锁和排他锁，读锁就是共享锁，写锁就是排他锁。  
```java
public class ReadWriteLockTest {

    static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    static Lock readLock = readWriteLock.readLock();
    static Lock writeLock = readWriteLock.writeLock();

    static int value = 0;

    public static void main(String[] arges){
        for (int i = 0; i < 10; i++) {
            //读线程不用阻塞，可以并发完成工作
            new Thread(()->{
                readLock.lock();
                try {
                    ReadWriteLockTest.read();
                } finally {
                    //记得Reentrant
                    readLock.unlock();
                }
            }).start();
        }

        //写线程会阻塞，需要排序完成任务
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                writeLock.lock();
                try {
                    ReadWriteLockTest.write(1);
                } finally {
                    writeLock.unlock();
                }
            }).start();
        }
    }

    public static void read(){
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e){
          e.printStackTrace();
        }
        System.out.println("read value =" + value);
    }

    public static void write(int num){
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e){
            e.printStackTrace();
        }
        value += num;
        System.out.println("write value =" + value);
    }

}
```

###### 6. Semaphore
Semaphore 含义就是限流，比如说流水线人不能全去上洗手间吧，所以得限制，每个上洗手间的人必须要领到洗手卡才能去，没有卡的人得等着前面的人回来，并归还了卡。
默认Semaphore是非公平的，new Semaphore(2, true)第二个值传true才是设置公平  
```java
public class SemaphoreTest {

    //定义限制数量
    static Semaphore semaphore = new Semaphore(2, true);

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+"想上厕所,排队领洗手卡。。。");
                    semaphore.acquire(1);
                    System.out.println("时间（秒）:"+System.currentTimeMillis()/1000+"，"+Thread.currentThread().getName()+"等到洗手卡了，上厕所ing。。。");
                    TimeUnit.SECONDS.sleep(4);
                }catch (InterruptedException e){
                    e.printStackTrace();
                } finally {
                    //用完了一定要记得归还release, 不然后边的人就没得用了。
                    semaphore.release(1);
                }
            },"姓名"+(1+i)).start();

            try {
                TimeUnit.SECONDS.sleep(1);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}
```

###### 7. Exchanger
交换器，俩人之间互相交换个数据用的，这里收到的消息得是成对的，否则 Exchanger 一直在等待有人来交换。
```java
public class ExchangerTest {

    static Exchanger<String> exchanger = new Exchanger<String>();

    public static void main(String[] args) {

        new Thread(() -> {
            try {
                String contents = exchanger.exchange(Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName() + " 收到消息：" + contents);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread 001").start();

        new Thread(() -> {
            try {
                String contents = exchanger.exchange(Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName() + " 收到消息：" + contents);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread 002").start();

        new Thread(() -> {
            try {
                String contents = exchanger.exchange(Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName() + " 收到消息：" + contents);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread 003").start();

        new Thread(() -> {
            try {
                String contents = exchanger.exchange(Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName() + " 收到消息：" + contents);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread 004").start();
    }
}
```

#### 十、LockSuport
*在JDK1.6中的java.util.concurrent的子包locks中引了LockSupport这个API，LockSupport是一个比较底层的工具类，用来创建锁和其他同步工具类的基本线程阻塞原语。java锁和同步器框架的核心 AQS: AbstractQueuedSynchronizer，就是通过调用 LockSupport.park()和 LockSupport.unpark()的方法，来实现线程的阻塞和唤醒的。

LockSuport的几个特点
1. LockSupport不需要synchornized加锁就可以实现线程的阻塞和唤醒 
2. LockSupport.unpartk()可以先于LockSupport.park()执行，并且线程不会阻塞 
3. 如果一个线程处于等待状态，连续调用了两次park()方法，就会使该线程永远无法被唤醒
4. park()和unpark()方法的实现是由Unsefa类提供的，而Unsefa类是由C和C++语言完成的，它主要通过一个变量作为一个标识，变量值在0，1之间来回切换，当这个变量大于0的时候线程就获得了“令牌”，其实park()和unpark()方法就是在改变这个变量的值，来达到线程的阻塞和唤醒的
```java
// LockSuport 要求用线程顺序打印A1B2C3....Z26
public class PrintFactory {

    static char[] letters = {'a', 'b', 'c', 'd', 'e', 'f', 'g'};
    static int[] nums = {1, 2, 3, 4, 5, 6, 7};
    static Thread letterPrinter = new Thread(LetterThread::print);
    static Thread numPrinter = new Thread(NumThread::print);

    public static void main(String[] args) {
        letterPrinter.start();
        numPrinter.start();
    }


    static class LetterThread {
        public static void print() {
            for (int i = 0; i < letters.length; i++) {
                System.out.print(letters[i]);
                LockSupport.unpark(numPrinter);
                if(i < letters.length -1){
                    LockSupport.park();
                }

            }
        }
    }

    static class NumThread {
        public static void print() {
            LockSupport.park();
            for (int i = 0; i < nums.length; i++) {
                System.out.print(nums[i]);
                LockSupport.unpark(letterPrinter);
                if(i < letters.length -1){
                    LockSupport.park();
                }
            }
        }
    }

}
```

#### 十一、Java的四种引用 强、软、弱、虚
* 比如Object o = new Object()，这就是普通的引用，也就是强引用，只要有一个应用指向这个对象，那么垃圾回收器一定不会回收它。
* 软引用 SoftReference<byte[]> m = new SoftReference<>(new byte[1024102410])   
  当有一个对象(字节数组)被一个软引用所指向的时候，只有系统内存不够 用的时候，才会回收它(字节数组)
* 弱引用 WeakReference m = new WeakReference<>(new M())
  只要遭遇到gc就会回收
* 虚引用 PhantomReference<M> phantomReference = new PhantomReference<>(new M(), QUEUE);
  对于虚引用它就干一件事，它就是管理堆外内存的， 
  首先第一点，这个虚引用的构造方法至少都是两个参数的， 
  第二个参数还必须是一个队列，这个虚引用基本没用，就是说不是给你用的，那么它是给谁用的呢?是给写JVM(虚拟机)的人用的  
  

#### 十二、并发容器
容器 分两大类Collection、Map，Collection又分三大类List、Set、Queue队列
###### Set 
Set 与List,Queue 的主要区别是不会有重复元素

###### Queue
* **Queue 实现的实际上是一个队列，有进有出，它实现了很多对线程友好的API offer、peek、poll，他的一个子类型叫 BlockingQueue对线程友好的API又添加了put和take，这两个实现了阻塞操作，这个是在其他的List、 Set里面都是没有的。这里面最重要的就是是叫做阻塞队列，它的实现的初衷就是为了线程池、高并发做准备的。**
* Queue里面还有一个子接口叫Deque叫双端队列，一般的队列只是从一端往里扔从另一端往外取。 Deque就是说你可以从反方向装从另外一个方向取。

| Queue Method | Equivalent Deque Method | 说明 |
| --- | --- | --- |
| add(e) | addLast(e) | 向队尾插入元素，失败则抛出异常 |
| offer(e) | offerLast(e) | 向队尾插入元素，失败则返回false |
| remove(e) | removeFirst(e) | 获取并删除首元素，失败则抛出异常 |
| poll(e) | pollFirst(e) | 获取并删除首元素，失败则返回null |
| element(e) | getFirst(e) | 获取但不删除首元素，失败则抛出异常 |
| peek(e) | peekFirst(e) | 获取但不删除首元素，失败则返回null |

###### ArrayList & LinkedList
* 没有加锁，线程不安全。  
* ArrayList是基于数组实现的，LinkedList是基于双链表实现的。
* LinkedList还实现了Deque接口，Deque接口是Queue接口的子接口，它代表一个双向队列，因此LinkedList可以作为双向对列。
* 因为Array是基于索引(index)的数据结构，它使用索引在数组中搜索和读取数据是很快的，可以直接返回数组中index位置的元素，因此在随机访问集合元素上有较好的性能。Array获取数据的时间复杂度是O(1),但是要插入、删除数据却是开销很大的，因为这需要移动数组中插入位置之后的的所有元素。
* 相对于ArrayList，LinkedList的随机访问集合元素时性能较差，因为需要在双向列表中找到要index的位置，再返回；但在插入，删除操作是更快的。

###### Vector & HashTable
最开始java1.0容器里只有两个，第一个叫Vector可以单独的往里扔，还有一个是Hashtable是可以一对一对往里扔的。Vector相对于实现了List接口，Hashtable实现了Map接口，Vector 和 Hashtable 自带锁所以性能低。

###### HashMap
HashMap没有锁，线程不安全,他虽然速度比较快，但是多线程时数据会出问题。

###### Collections.synchronizedHashMap
Map<String,String> map = Collections.synchronizedMap(new HashMap<String,String>());  
用的是SynchronizedMap这个方法，给HashMap我们手动加锁，它的源码自己做了一个Object，然后每次都是SynchronizedObject，严格来讲他和那个Hashtable效率上区别不大。

##### ConcurrentMap 接口
###### ConcurrentHashMap & ConcurrentSkipListMap
ConcurrentHashMap是多线程里面真正用的，提高效率主要提高在读上面，由于它往里插的时候内部又做了各种各样的判断，本来是链表的，到8之后又变成了红黑树，然后里面又做了各种各样的cas的判断，所以他往里插的数据是要更低一些的。  
ConcurrentSkipListMap 通过**跳表**来实现的高并发容器并且这个Map是有排序的;  
这两个的区别一个是有序的一个是无序的，同时都支持并发的操作
  
##### CopyOnWrite 
###### CopyOnWriteArrayList & CopyOnWriteArraySet & CopyOnWriteMap
写时复制，当Write的时候我们要进行复制。这个原码非常简单，当我们需要往里面加元素的时候，把里面的元素得复制出来，再添加一个位置存放新元素。 而且在写的时个有加锁，但在读的时候没有锁。在写的时候特别少，读的时候很多的情况下，在这个时候就可以考虑CopyOnWrite这种方式来提高效率。
```
//CopyOnWriteMap.put方法
public V put(K key, V value) {
    synchronized(this) {
        Map<K, V> newMap = new HashMap(this.internalMap);
        V val = newMap.put(key, value);
        this.internalMap = newMap;
        return val;
    }
}

//CopyOnWriteArrayList.add
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}

//CopyOnWriteArraySet 中实际是由CopyOnWriteArrayList存放的，在add的时候直接调用的CopyOnWriteArrayList.addIfAbsent(...)
```

##### BlockingQueue 接口
BlockingQueue的概念重点是在Blocking上，Blocking阻塞，Queue队列，是阻塞队列。  
BlockingQueue在 Queue的基础上又添加了两个方法，这两个方法一个叫put，一个叫take。这两个方法是真真正正的实现了阻塞。put往里装如果满了的话我这个线程会阻塞住，take往外取如果空了的话线程会阻塞住。所 以这个BlockingQueue就实现了生产者消费者里面的那个容器。

###### LinkedBlockingQueue
用链表实现的BlockingQueue，是一个无界队列
###### ArrayBlockingqueue
ArrayBlockingQueue是有界的，可以指定它一个固定的值10，它容器就是10，那么当你往里面扔容器的时候，一旦他满了这个put方法就会阻塞住。然后你可以看看用add方法满了之后他会报异常。 offer用返回值来判断到底加没加成功，offer还有另外一个写法你可以指定一个时间尝试着往里面加1秒钟，1秒钟之后如果加不进去它就返回了.
###### DelayQueue
DelayQueue可以实现在时间上的排序，这个DelayQueue能实现按照在里面等待的时间来进行排序。
###### SynchronousQueue
SynchronousQueue容量为0，就是这个东西它不是用来装内容的，SynchronousQueue是专门用来两 个线程之间传内容的，给线程下达任务。 
这个Queue和其他的很重要的区别就是 你不能往里头装东西，只能用来阻塞式的put调用，要求是前面得有人等着拿这个东西的时候你才可以 往里装，但容量为0，其实说白了就是我要递到另外一个的手里才可以。
###### TransferQueue   
TransferQueue传递，实际上是前面这各种各样Queue的一个组合，它可以给线程来传递任务，以此同时不像是SynchronousQueue只能传递一个，TransferQueue做成列表可以传好多个。比较牛X的是它添加了一个方法叫transfer，如果我们用put就相当于一个线程来了往里一装它就走了。transfer就是装完在这等着，阻塞等有人把它取走我这个线程才回去干我自己的事情。  
一般使用场景:是我做了一件事情，我这个事情要求有一个结果，有了这个结果之后我可以继续进行我下面的这个事情的时候，比方说 我付了钱，这个订单我付账完成了，但是我一直要等这个付账的结果完成才可以给客户反馈。  

#### PriorityQueue
PriorityQueue特点是它内部你往里装的时候并不是按顺序往里装的，而是内部进行了一个排序。





#### 十三、线程池
###### Executor
执行者，是一个接口类，他有一个方法叫执行，那么执行的东西是 Runnable。

###### ExecutorService
是从Executor继承，除了去实现Executor可以去执行一个任务之外，还完善了整个任务执行器的一个生命周期，就拿线程池来举例子，一个线程池里面一堆的线程就是一堆的工人，执行完一个任务之后我这个线程怎么结束啊；  
线程池定义了这样一些个方法：
```
void shutdown();//结束
List<Runnable> shutdownNow();//马上结束
boolean isShutdown();//是否结束了
boolean isTerminated();//是不是整体都执行完了
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;//等着结束，等多长时间，时间到了还不结束的话他 就返回false
<T> Future<T> submit(Callable<T> task);
Future<?> submit(Runnable task);
List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
<T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
...
```
```java
public class ExecutroServiceTest {

    public static void main(String[] args) {
        ExecutorService es = Executors.newCachedThreadPool();
        FutureTask<Integer> task = new FutureTask(() -> {
            System.out.println("Executor Service Test");
            return 1;
        });
        es.execute(task);
        es.shutdownNow();
    }
}
```

他是实现了一些个线程的线程池的生命周期的东西，扩展了Executor的接口，真正的线程池的现实是在ExecutorService的这个基础上来实现的。  
ExecutorService的时候你会发现他除了Executor执行任务之外还有submit提交任务，执行任务是直接拿过来马上运行，而submit是扔给这个线程池，什么时候运行由这个线程池来决定，相当于 是异步的，我只要往里面一扔就不管了。那好，如果不管的话什么时候他有结果啊，这里面就涉及了比较新的类:比如说Future、RunnableFuture、FutureTask。

###### Callable
以前定义一个线程的任务只能去实现Runnable接口，那在1.5之后他就增加了Callable这个接口。  
Callable是什么，他类似于Runnable，不过 Callable可以有返回值。  
```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
    */
    V call() throws Exception;
}
```

###### Future
Future代表的是那个Callable被执行完了之后我怎么才能拿到那个结果啊，它会封装到一个Future里面。
Future将来，未来。未来你执行完之后可以把这个结果放到这个未来有可能执行完的结果里头，所以Future代表的是未来执行完的一个结果。
把Callable的任务扔给线程池，线程池异步的执行完了，就是把任务交给线程池之后，调用get方法直到有结果之后get会返回。Callable一般是配合线程池和Future来用的。

###### FutureTask
其实更灵活的一个用法是FutureTask，即是一个Future同时又是一个Task，原来这Callable只能一个Task只能是一个任务但是他不能作为一个Future来用。这个FutureTask相当于是我自己可以作为一个任务来用，同时这个任务完成之后的结果也存在于这个对象里，为什么他能做到这一点，因 为FutureTask他实现了RunnableFuture，而RunnableFuture即实现了Runnable又实现了Future，所以他即是一个任务又是一个Future
```java
public class FutrueTaskTest {

    public static void main(String[] args) {
        FutureTask task = new FutureTask(()->{
            System.out.println("future task test");
            return 1;
        });
        new Thread(task).start();
    }
}
```

###### CompletableFuture
CompletableFuture他的底层用的是ForkJoinPool，底层特别复杂，但是用法特别灵活。

##### 线程池
目前JDK提供的有两种类型
1. ThreadPoolExecutor 普通的线程池
2. ForkJoinPool

###### ThreadPoolExecutor
ThreadPoolExecutor他的父类是AbstractExecutorService，AbstractExecutorService 实现了 ExecutorService，再ExecutorService的父类是Executor，所以ThreadPoolExecutor就相当于线程池的执行器  

定义这一个线程池，这里面的七个参数:
1. corePoolSoze 核心线程数，最开始的时候是有这个线程池里面是有一定的核心线程数 的;
2. maximumPoolSize 最大线程数，线程数不够了，能扩展到最大线程是多少; 
3. keepAliveTime 生存时间，意思是这个线程有很长时间没干活了请你把它归还给操作系
4. TimeUnit.SECONDS 生存时间的单位到底是毫秒纳秒还是秒自己去定义;
5. 任务队列，就是我们前面讲的BlockingQueue，各种各样的BlockingQueue都可以;
6. 线程工厂, 要去实现ThreadFactory的接口，这个接口只有一个方法叫newThread，所以就是产生线程的，可以通过这种方式产生自定义的线程，默认产生的是defaultThreadFactory，而defaultThreadFactory 产生线程的时候有几个特点: new出来的时候指定了group制定了线程名字，然后指定的这个线程 绝对不是守护线程，设定好你线程的优先级。自己可以定义产生的到底是什么样的线程，指定线程名叫什么(为什么要指定线程名称，有什么意义，就是可以方便出错是回溯);
7. 拒绝策略，指的是线程池忙，而且任务队列满这种情况下我们就要执行各种各样的拒绝策略，
   jdk默认提供了四种拒绝策略，也是可以自定义的。
    * 1:Abort:抛异常 
    * 2:Discard:扔掉，不抛异常 
    * 3:DiscardOldest:扔掉排队时间最久的 
    * 4:CallerRuns:调用者处理服务  
   一般情况这四种我们会自定义策略，去实现这个拒绝策略的接口，处理的方式是一般我们的消息需要保存下来，并且记录日志。

##### JDK给我们提供了一些默认的线程池的实现
###### 1. SingleThreadPool
只有一个线程，这个一个线程的线程池可以保证我们扔进去的任务是顺序执行的。
```
ExecutorService service = Executors.newSingleThreadExecutor();

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

###### 2. CachedThreadPool 
CachedThreadPool的特点，就是你来一个任务我给你启动一个线程，当然前提是我的线程池里面有线程存在而且他还没有到达60秒钟的回收时间的时候，来一个任务，如果有线程存在我就用现有的线程，但是在有新的任务来的时候，如果其他线程忙就启动一个新的，CachedThreadPool用的任务队列是 synchronousQueue，它是一个手递手容量为空的Queue，就是你来一个东西必须得有一个线程把他拿走，不然我提交任务的线程从这阻塞住了。
```
ExecutorService executor = Executors.newCachedThreadPool();
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

###### 3. FixedThreadPool
FixedThreadPool指定一个参数，到底有多少个线程，你看他的核心线程和最大线程都是固定的，因为他的最大线程和核心线程都是固定的就没有回收之说，所以把keepAliveTime指定成0，这里用的是LinkedBlockingQueue
```
ExecutorService executor = Executors.newFixedThreadPool(10);
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
###### 4. ScheduledPool
定时任务线程池，隔一段时间之后这个任务会执行。这个就是我们专门用来执行定时任务的一个线程池。看源码，我们newScheduledThreadPool的时 候他返回的是ScheduledThreadPoolExecutor，然后在ScheduledThreadPoolExecutor里面他调用了 super，他的super又是ThreadPoolExecutor，它本质上还是ThreadPoolExecutor，所以并不是别的，参数还是ThreadPool的七个参数。这是专门给定时任务用的这样的一个线程池。
```
ExecutorService executor = Executors.newScheduledThreadPool(10);

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```

###### 5. WorkStealingPool (它实际上是new了一个ForkJoinPool)
WorkStealing指的是和原来线程池的区别每一个线程都有自己单独队列，所以任务不断往里扔的时候它会在每一个线程的队列上不断的累积，让某一个线程执行完自己的任务之后就回去另外一个线程上面偷，所以这个叫WorkStealing。
```
ExecutorService executor = Executors.newWorkStealingPool();

public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool
        (Runtime.getRuntime().availableProcessors(),
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
```
###### 6. ForkJoinPool
它适合把大任务切分成一个一个的小任务去运行，小任务还是觉得比较大，再切。切完这个任务执行完了要进行一个汇总。

###### Cache vs Fixed
什么时候用Cache什么时候用Fixed，你得精确的控制有多少个线程数，控制数量问题多数情况下得预估并发量。如果线程池中的数量过多，最终他们会竞争稀缺的处理器和内存资源，浪费大量的时间在上下文切换上，反之，如果线程的数目过少，正如你的应用所面临的情况，处理器的一些核可能就无法充分利用。
《Java并发编程实战》作者 Brian Goetz建议，线程池大小与处理器的利用率之比可以使用公式来进行计算估算:线程池=你有多少个cpu 乘以 cpu期望利用率 乘以 (1+ W/C)。W除以C是等待时间与计算时间的比率。


###### 线程池的一些5种状态
1. RUNNING:正常运行的;
2. SHUTDOWN:调用了shutdown方法了进入了shutdown状态; 
3. STOP:调用了shutdownnow马上让他停止; 
4. TIDYING:调用了shutdown然后这个线程也执行完了，现在正在整理的这个过程叫TIDYING; 
5. TERMINATED:整个线程全部结束;

#### 十四、JMH
JMH -java Microbenchmark Harness
微基准测试，它是测的某一个方法的性能。支持命令行或IDEA开发工具运行，idea运行需要添加插件。直接在Idea插件查找JMH安装。安装好后就可以像Junit一样写测试。
官网 http://openjdk.java.net/projects/code-tools/jmh/
Maven 引用：
```xml
<!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-core -->
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.21</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess -->
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.21</version>
    <scope>test</scope>
</dependency>
```
使用示例,代码写在test目录下，类似junit测试：
```
@Benchmark
@Warmup(iteration=1, time=3)  //在专业测试里面首先要进行预热，预热多少次，预热多少时间
@Fork(5)  //用多少个线程去执行我们的程序 
@BenchmarkMode(Mode.Throughput)  //是对基准测试的一个模式，这个模式用的最多的是 Throughput吞吐量
@Measurement(iteration=1, time=3) //是整个测试要测试多少遍，调用这个方法要调用多少次
public void test() {
    //.... 调用测试方法
}
```


#### 十四、Disrupter
参考网站：http://ifeve.com/disruptor/
Disruptor 开源的并发框架，并获得2011 Duke’s 程序框架创新奖，能够在无锁的情况下实现网络的Queue并发操作。
* 如果把它用作MQ的话，**单机**最快的MQ，性能非常的高，主要是它里面 用的全都是CAS, 另外把各种各样的性能开发到了极致。
* Disruptor就是在内存里，Disruptor简单理解就是内存里用于存放元素的一个高效率的队列。
* Disruptor叫无锁、高并发、环形Buffer，直接覆盖(不用清除)旧的数据，降低GC频率，用于生产者消费者模式
* RingBuffer是一个环形队列，和其他队列不一样的是他是一个环形队列，环形的Buffer。一般情况下我们的容器是一个队列，不管你是用链表实现还是用数组实现的，它会是一个队列，那么这个队列生产者这 边使劲往里塞，消费者这边使劲往外拿，但Disruptor的核心是一个环形的buffer。
* RingBuffer的序号，指向下一个可用的元素
* 采用数组实现，没有首尾指针对比ConcurrentLinkedQueue，用数组实现的速度更快  
  假如长度为8，当添加到第12个元素的时候在哪个序号上呢?用12%8  
  决定当Buffer被填满的时候到底是覆盖还是等待，由Produce决定 长度设为2的n次幂，利于二进制计算，例如:12%8=12&(8-1)
  
###### 等待策略
* (常用)BlockingWaitStrategy:通过线程堵塞的方式，等待生产者唤醒，被唤醒后，再循环检查 依赖的sequence是否已经消费。
* BusySpinWaitStrategy:线程一直自旋等待，可能比较耗cpu 
* LiteBlockingWaitStrategy:线程阻塞等待生产者唤醒，与BlockingWaitStrategy相比，区别在 signalNeeded.getAndSet，如果两个线程同时访问一个访问waitfor，一个访问signalAll时，可以 减少lock加锁次数 
* LiteTimeoutBlockingWaitStrategy:与LiteBlockingWaitStrategy相比，设置了阻塞时间，超过时间后抛出异常 
* PhasedBackoffWaitStrategy:根据时间参数和传入的等待策略来决定使用那种等待策略 
* TimeoutBlockingWaitStrategy:相对于BlockingWaitStrategy来说，设置了等待时间，超过后抛 出异常
* (常用)YieldingWaitStrategy:尝试100次，然后Thread.yield()让出cpu 
* (常用)SleepingWaitStrategy:sleep




   








 
























 




















