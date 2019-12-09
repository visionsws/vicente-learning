## Java 并发编程

### 01、简介

百丈高楼平地起，要想学好多线程，首先还是的了解一下线程的基础，这边文章将带着大家来了解一下线程的基础知识。

### 02、线程的创建方式

1. 实现 `Runnable` 接口
2. 继承 `Thread` 类
3. 实现 `Callable` 接口通过 `FutureTask` 包装器来创建线程
4. 通过线程池创建线程

下面将用线程池和 `Callable` 的方式来创建线程

```
public class CallableDemo implements Callable<String> {

    @Override
    public String call() throws Exception {
        int a=1;
        int b=2;
        System. out .println(a+b);
        return "执行结果:"+(a+b);
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建一个可重用固定线程数为1的线程池
        ExecutorService executorService = Executors.newFixedThreadPool (1);
        CallableDemo callableDemo=new CallableDemo();
        //执行线程,用future来接收线程的返回值
        Future<String> future = executorService.submit(callableDemo);
        //打印线程的返回值
        System. out .println(future.get());
        executorService.shutdown();
    }
}
```

执行结果

```
3
执行结果:3
```

### 03、线程的生命周期

1. NEW：初始状态，线程被构建，但是还没有调用 start 方法。
2. RUNNABLED：运行状态，JAVA 线程把操作系统中的就绪和运行两种状态统一称为“运行中”。调用线程的 `start()` 方法使线程进入就绪状态。
3. BLOCKED：阻塞状态，表示线程进入等待状态,也就是线程因为某种原因放弃了 CPU 使用权。比如访问 `synchronized` 关键字修饰的方法，没有获得对象锁。
4. Waiting ：等待状态，比如调用 wait() 方法。
5. TIME_WAITING：超时等待状态，超时以后自动返回。比如调用 `sleep(long millis)` 方法
6. TERMINATED：终止状态，表示当前线程执行完毕。

看下源码：

```
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
}
```

### 04、线程的优先级

1. 线程的最小优先级：1
2. 线程的最大优先级：10
3. 线程的默认优先级：5
4. 通过调用 `getPriority()` 和 `setPriority(int newPriority)` 方法来获得和设置线程的优先级

看下源码：

```
	/**
     * The minimum priority that a thread can have.
     */
    public final static int MIN_PRIORITY = 1;

    /**
     * The default priority that is assigned to a thread.
     */
    public final static int NORM_PRIORITY = 5;

    /**
     * The maximum priority that a thread can have.
     */
    public final static int MAX_PRIORITY = 10;
```

看下代码：

```
public class ThreadA extends Thread {

    public static void main(String[] args) {
        ThreadA a = new ThreadA();
        System.out.println(a.getPriority());//5
        a.setPriority(8);
        System.out.println(a.getPriority());//8
    }
}
```

#### 线程优先级特性：

1. 继承性：比如 A 线程启动 B 线程，则B线程的优先级与 A 是一样的。
2. 规则性：高优先级的线程总是大部分先执行完，但不代表高优先级线程全部先执行完。
3. 随机性：优先级较高的线程不一定每一次都先执行完。

### 05、线程的停止

1. `stop()` 方法，这个方法已经标记为过时了，强制停止线程，相当于 kill -9。
2. `interrupt()` 方法，优雅的停止线程。告诉线程可以停止了，至于线程什么时候停止，取决于线程自身。

看下停止线程的代码：

```
public class InterruptDemo {
    private static int i ;
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            //默认情况下isInterrupted 返回 false、通过 thread.interrupt 变成了 true
            while (!Thread.currentThread().isInterrupted()) {
                i++;
            }
            System.out.println("Num:" + i);
        }, "interruptDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt(); //不加这句,thread线程不会停止
    }
}
```

看上面这段代码，主线程 main 方法调用 `thread`线程的 `interrupt()` 方法，就是告诉`thread` 线程，你可以停止了（其实是将 `thread` 线程的一个属性设置为了 true ），然后 `thread` 线程通过 `isInterrupted()` 方法获取这个属性来判断是否设置为了 true。这里我再举一个例子来说明一下，

看代码：

```
public class ThreadDemo {
    private volatile static Boolean interrupt = false ;
    private static int i ;

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (!interrupt) {
                i++;
            }
            System.out.println("Num:" + i);
        }, "ThreadDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        interrupt = true;
    }
}
```

是不是很相似，再简单总结一下：

当其他线程通过调用当前线程的 interrupt 方法，表示向当前线程打个招呼，告诉他可以中断线程的执行了，并不会立即中断线程，至于什么时候中断，取决于当前线程自己。

线程通过检查自身是否被中断来进行相应，可以通过 isInterrupted() 来判断是否被中断。

这种通过标识符来实现中断操作的方式能够使线程在终止时有机会去清理资源，而不是武断地将线程停止，因此这种终止线程的做法显得更加安全和优雅。

### 06、线程的复位

两种复位方式：

1. Thread.interrupted()
2. 通过抛出 InterruptedException 的方式

然后了解一下什么是复位：

线程运行状态时 Thread.isInterrupted() 返回的线程状态是 false，然后调用 thread.interrupt() 中断线程 Thread.isInterrupted() 返回的线程状态是 true，最后调用 Thread.interrupted() 复位线程Thread.isInterrupted() 返回的线程状态是 false 或者抛出 InterruptedException 异常之前，线程会将状态设为 false。

下面来看下两种方式复位线程的代码，首先是 Thread.interrupted() 的方式复位代码：

```
public class InterruptDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (true) {
                //Thread.currentThread().isInterrupted()默认是false,当main方式执行thread.interrupt()时,状态改为true
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("before:" + Thread.currentThread().isInterrupted());//before:true
                    Thread.interrupted(); // 对线程进行复位，由 true 变成 false
                    System.out.println("after:" + Thread.currentThread().isInterrupted());//after:false
                }
            }
        }, "interruptDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt();
    }
}
```

抛出 InterruptedException 复位线程代码：

```
public class InterruptedExceptionDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    // break;
                }
            }
        }, "interruptDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt();
        System.out.println(thread.isInterrupted());
    }
}
```

结果：

```
false
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at com.cl.concurrentprogram.InterruptedExceptionDemo.lambda$main$0(InterruptedExceptionDemo.java:16)
	at java.lang.Thread.run(Thread.java:748)
```

需要注意的是，InterruptedException 异常的抛出并不意味着线程必须终止，而是提醒当前线程有中断的操作发生，至于接下来怎么处理取决于线程本身，比如

1. 直接捕获异常不做任何处理
2. 将异常往外抛出
3. 停止当前线程，并打印异常信息

像我上面的例子,如果抛出 InterruptedException 异常,我就break跳出循环让 thread 线程终止。

 **为什么要复位**

Thread.interrupted() 是属于当前线程的，是当前线程对外界中断信号的一个响应，表示自己已经得到了中断信号，但不会立刻中断自己，具体什么时候中断由自己决定，让外界知道在自身中断前，他的中断状态仍然是 false，这就是复位的原因。



## 线程池

### 优点
  1. 重用线程池中的线程，避免因为线程的创建和销毁带来的性能开销。
  2. 有效控制线程的最大并发数，避免因大量的线程之间互相抢占系统资源而导致的阻塞 。
  3. 能够对线程进行简单的管理，并提供定时执行和执行循环间隔执行等功能

###  线程池原理

  谈到线程池就会想到池化技术，其中最核心的思想就是把宝贵的资源放到一个池子中；每次使用都从里面获取，用完之后又放回池子供其他人使用，有点吃大锅饭的意思。

  那在 Java 中又是如何实现的呢？

  

  所以我们重点来看下 `ThreadPoolExecutor` 是怎么玩的。

  首先是创建线程的 api：

```
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {  }复制代码
```

- 变量
  - corePoolSize: 核心线程数
  - maximumPoolSize: 最大线程数
  - workQueue:用于存放任务的阻塞队列。
  - keepAliveTime: 非核心线程存活时间
  - unit:keepAliveTime的时间单位。
  - threadFactory:为线程池提供新线程的工厂，可以给创建的线程设置有意义的名字，可方便排查问题。
  - handler：异常处理策略，主要有四种类型。
- 规则
  1. 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。 
  2. 如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。 
  3. 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。
  4. 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。

### 四种拒绝策略

- AbortPolicy(抛出一个异常，默认的)
- DiscardPolicy(直接丢弃任务)
- DiscardOldestPolicy（丢弃队列里最老的任务，将当前这个任务继续提交给线程池）
- CallerRunsPolicy（交给线程池调用所在的线程进行处理)

通常我们都是使用:

```
threadPool.execute(new Job());
```

这样的方式来提交一个任务到线程池中，所以核心的逻辑就是 `execute()` 函数了。

### 线程池状态

线程池有这几个状态：RUNNING,SHUTDOWN,STOP,TIDYING,TERMINATED。

```
   //线程池状态
   private static final int RUNNING    = -1 << COUNT_BITS;
   private static final int SHUTDOWN   =  0 << COUNT_BITS;
   private static final int STOP       =  1 << COUNT_BITS;
   private static final int TIDYING    =  2 << COUNT_BITS;
   private static final int TERMINATED =  3 << COUNT_BITS;

```

在具体分析之前先了解下线程池中所定义的状态，这些状态都和线程的执行密切相关：

- `RUNNING` 自然是运行状态，指可以接受任务执行队列里的任务
- `SHUTDOWN` 指调用了 `shutdown()` 方法，不再接受新任务了，但是队列里的任务得执行完毕。
- `STOP` 指调用了 `shutdownNow()` 方法，不再接受新任务，同时抛弃阻塞队列里的所有任务并中断所有正在执行任务。
- `TIDYING` 所有任务都执行完毕，在调用 `shutdown()/shutdownNow()` 中都会尝试更新为这个状态。
- `TERMINATED` 终止状态，当执行 `terminated()` 后会更新为这个状态。

用图表示为：



![img](https://user-gold-cdn.xitu.io/2018/7/30/164e8a5a634e71b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



然后看看 `execute()` 方法是如何处理的：

### 线程池的工作过程

1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。
2. 当调用 execute() 方法添加一个任务时，线程池会做如下判断：
   - 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；
   - 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；
   - 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
   - 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常RejectExecutionException。
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

### 线程池最常用的提交方法

线程池最常用的提交任务的方法有两种：

**execute**:

```
ExecutorService.execute(Runnable runable)；
```

**submit**:

```
FutureTask task = ExecutorService.submit(Runnable runnable);
	
FutureTask<T> task = ExecutorService.submit(Runnable runnable,T Result);
	
FutureTask<T> task = ExecutorService.submit(Callable<T> callable);
```

submit(Callable callable)的实现，submit(Runnable runnable)同理。

```
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    FutureTask<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

可以看出submit开启的是有返回结果的任务，会返回一个FutureTask对象，这样就能通过get()方法得到结果。submit最终调用的也是execute(Runnable runable)，submit只是将Callable对象或Runnable封装成一个FutureTask对象，因为FutureTask是个Runnable，所以可以在execute中执行。关于Callable对象和Runnable怎么封装成FutureTask对象，见[Callable和Future、FutureTask的使用](http://www.silencedut.com/2016/06/15/Callable%E5%92%8CFuture%E3%80%81FutureTask%E7%9A%84%E4%BD%BF%E7%94%A8)。

### 如何配置线程

流程聊完了再来看看上文提到了几个核心参数应该如何配置呢？

有一点是肯定的，线程池肯定是不是越大越好。

通常我们是需要根据这批任务执行的性质来确定的。

- IO 密集型任务：由于线程并不是一直在运行，所以可以尽可能的多配置线程，比如 CPU 个数 * 2
- CPU 密集型任务（大量复杂的运算）应当分配较少的线程，比如 CPU 个数相当的大小。

当然这些都是经验值，最好的方式还是根据实际情况测试得出最佳配置。

### 优雅的关闭线程池

有运行任务自然也有关闭任务，从上文提到的 5 个状态就能看出如何来关闭线程池。

其实无非就是两个方法 `shutdown()/shutdownNow()`。

但他们有着重要的区别：

- `shutdown()` 执行后停止接受新任务，会把队列的任务执行完毕。
- `shutdownNow()` 也是停止接受新任务，但会中断所有的任务，将线程池状态变为 stop。

> 两个方法都会中断线程，用户可自行判断是否需要响应中断。

`shutdownNow()` 要更简单粗暴，可以根据实际场景选择不同的方法。

我通常是按照以下方式关闭线程池的：

```
        long start = System.currentTimeMillis();
        for (int i = 0; i <= 5; i++) {
            pool.execute(new Job());
        }

        pool.shutdown();

        while (!pool.awaitTermination(1, TimeUnit.SECONDS)) {
            LOGGER.info("线程还在执行。。。");
        }
        long end = System.currentTimeMillis();
        LOGGER.info("一共处理了【{}】", (end - start));
复制代码
```

`pool.awaitTermination(1, TimeUnit.SECONDS)` 会每隔一秒钟检查一次是否执行完毕（状态为 `TERMINATED`），当从 while 循环退出时就表明线程池已经完全终止了。

## SpringBoot 使用线程池

既然用了 SpringBoot ，那自然得发挥 Spring 的特性，所以需要 Spring 来帮我们管理线程池：

```
@Configuration
public class TreadPoolConfig {


    /**
     * 消费队列线程
     * @return
     */
    @Bean(value = "consumerQueueThreadPool")
    public ExecutorService buildConsumerQueueThreadPool(){
        ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
                .setNameFormat("consumer-queue-thread-%d").build();

        ExecutorService pool = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.MILLISECONDS,
                new ArrayBlockingQueue<Runnable>(5),namedThreadFactory,new ThreadPoolExecutor.AbortPolicy());

        return pool ;
    }



}
复制代码
```

使用时：

```
    @Resource(name = "consumerQueueThreadPool")
    private ExecutorService consumerQueueThreadPool;


    @Override
    public void execute() {

        //消费队列
        for (int i = 0; i < 5; i++) {
            consumerQueueThreadPool.execute(new ConsumerQueueThread());
        }

    }
```

其实也挺简单，就是创建了一个线程池的 bean，在使用时直接从 Spring 中取出即可。

## 监控线程池

谈到了 SpringBoot，也可利用它 actuator 组件来做线程池的监控。

线程怎么说都是稀缺资源，对线程池的监控可以知道自己任务执行的状况、效率等。

关于 actuator 就不再细说了，感兴趣的可以看看[这篇](http://t.cn/ReimM0o)，有详细整理过如何暴露监控端点。

其实 ThreadPool 本身已经提供了不少 api 可以获取线程状态：



![img](https://user-gold-cdn.xitu.io/2018/7/30/164e8a5a8452855a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



很多方法看名字就知道其含义，只需要将这些信息暴露到 SpringBoot 的监控端点中，我们就可以在可视化页面查看当前的线程池状态了。

## 线程池异常处理

在使用线程池处理任务的时候，任务代码可能抛出RuntimeException，抛出异常后，线程池可能捕获它，也可能创建一个新的线程来代替异常的线程，我们可能无法感知任务出现了异常，因此我们需要考虑线程池异常情况。

### 当提交新任务时，异常如何处理?

我们先来看一段代码：

```
       ExecutorService threadPool = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            threadPool.submit(() -> {
                System.out.println("current thread name" + Thread.currentThread().getName());
                Object object = null;
                System.out.print("result## "+object.toString());
            });
        }

复制代码
```

显然，这段代码会有异常，我们再来看看执行结果

> 1.直接try...catch捕获异常

> 2.submit执行的任务，通过Future.get()接收抛出的异常

> 3.为工作者线程设置UncaughtExceptionHandler，在uncaughtException方法中处理异常

> 4.重写ThreadPoolExecutor的afterExecute方法，处理传递的异常引用

## 线程池的工作队列

**线程池都有哪几种工作队列？**

- ArrayBlockingQueue
- LinkedBlockingQueue
- DelayQueue
- PriorityBlockingQueue
- SynchronousQueue

### ArrayBlockingQueue

ArrayBlockingQueue（有界队列）是一个用数组实现的有界阻塞队列，按FIFO排序量。

### LinkedBlockingQueue

LinkedBlockingQueue（可设置容量队列）基于链表结构的阻塞队列，按FIFO排序任务，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为Integer.MAX_VALUE，吞吐量通常要高于ArrayBlockingQuene；newFixedThreadPool线程池使用了这个队列

### DelayQueue

DelayQueue（延迟队列）是一个任务定时周期的延迟执行的队列。根据指定的执行时间从小到大排序，否则根据插入到队列的先后排序。newScheduledThreadPool线程池使用了这个队列。

### PriorityBlockingQueue

PriorityBlockingQueue（优先级队列）是具有优先级的无界阻塞队列；

### SynchronousQueue

SynchronousQueue（同步队列）一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene，newCachedThreadPool线程池使用了这个队列。

针对面试题：**线程池都有哪几种工作队列？** 我觉得，**回答以上几种ArrayBlockingQueue，LinkedBlockingQueue，SynchronousQueue等，说出它们的特点，并结合使用到对应队列的常用线程池(如newFixedThreadPool线程池使用LinkedBlockingQueue)，进行展开阐述，** 就可以啦。

## 几种常用的线程池

在 JDK 1.5 之后推出了相关的 api，常见的创建线程池方式有以下几种：

- `Executors.newCachedThreadPool()`：无限线程池。
- `Executors.newFixedThreadPool(nThreads)`：创建固定大小的线程池。
- `Executors.newSingleThreadExecutor()`：创建单个线程的线程池。
- `Executors.newScheduledThreadPool()`  定时及周期执行的线程池) 

  其实看这三种方式创建的源码就会发现：



### newFixedThreadPool

```
  public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
复制代码
```

#### 线程池特点：

- 核心线程数和最大线程数大小一样
- 没有所谓的非空闲时间，即keepAliveTime为0
- 阻塞队列为无界队列LinkedBlockingQueue

#### 工作机制：

- 提交任务
- 如果线程数少于核心线程，创建核心线程执行任务
- 如果线程数等于核心线程，把任务添加到LinkedBlockingQueue阻塞队列
- 如果线程执行完任务，去阻塞队列取任务，继续执行。

#### 实例代码

```
   ExecutorService executor = Executors.newFixedThreadPool(10);
                    for (int i = 0; i < Integer.MAX_VALUE; i++) {
                        executor.execute(()->{
                            try {
                                Thread.sleep(10000);
                            } catch (InterruptedException e) {
                                //do nothing
                            }
            });
复制代码
```

**面试题：使用无界队列的线程池会导致内存飙升吗？**

答案 **：会的，newFixedThreadPool使用了无界的阻塞队列LinkedBlockingQueue，如果线程获取一个任务后，任务的执行时间比较长(比如，上面demo设置了10秒)，会导致队列的任务越积越多，导致机器内存使用不停飙升，** 最终导致OOM。

#### 使用场景

FixedThreadPool 适用于处理CPU密集型的任务，确保CPU在长期被工作线程使用的情况下，尽可能的少的分配线程，即适用执行长期的任务。

### newCachedThreadPool

```
   public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
复制代码
```

#### 线程池特点：

- 核心线程数为0
- 最大线程数为Integer.MAX_VALUE
- 阻塞队列是SynchronousQueue
- 非核心线程空闲存活时间为60秒

当提交任务的速度大于处理任务的速度时，每次提交一个任务，就必然会创建一个线程。极端情况下会创建过多的线程，耗尽 CPU 和内存资源。由于空闲 60 秒的线程会被终止，长时间保持空闲的 CachedThreadPool 不会占用任何资源。

#### 工作机制



![img](https://user-gold-cdn.xitu.io/2019/7/15/16bf2d1734c8f96d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 提交任务
- 因为没有核心线程，所以任务直接加到SynchronousQueue队列。
- 判断是否有空闲线程，如果有，就去取出任务执行。
- 如果没有空闲线程，就新建一个线程执行。
- 执行完任务的线程，还可以存活60秒，如果在这期间，接到任务，可以继续活下去；否则，被销毁。

#### 实例代码

```
  ExecutorService executor = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName()+"正在执行");
            });
        }

```

#### 使用场景

用于并发执行大量短期的小任务。

### newSingleThreadExecutor

```
  public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
复制代码
```

#### 线程池特点

- 核心线程数为1
- 最大线程数也为1
- 阻塞队列是LinkedBlockingQueue
- keepAliveTime为0

#### 工作机制



![img](https://user-gold-cdn.xitu.io/2019/7/15/16bf2e62d7dbaaea?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 提交任务
- 线程池是否有一条线程在，如果没有，新建线程执行任务
- 如果有，讲任务加到阻塞队列
- 当前的唯一线程，从队列取任务，执行完一个，再继续取，一个人（一条线程）夜以继日地干活。

#### 实例代码

```
  ExecutorService executor = Executors.newSingleThreadExecutor();
                for (int i = 0; i < 5; i++) {
                    executor.execute(() -> {
                        System.out.println(Thread.currentThread().getName()+"正在执行");
                    });
        }
```

#### 使用场景

适用于串行执行任务的场景，一个任务一个任务地执行。

### ewScheduledThreadPool

```
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
复制代码
```

#### 线程池特点

- 最大线程数为Integer.MAX_VALUE
- 阻塞队列是DelayedWorkQueue
- keepAliveTime为0
- scheduleAtFixedRate() ：按某种速率周期执行
- scheduleWithFixedDelay()：在某个延迟后执行

#### 工作机制

- 添加一个任务
- 线程池中的线程从 DelayQueue 中取任务
- 线程从 DelayQueue 中获取 time 大于等于当前时间的task
- 执行完后修改这个 task 的 time 为下次被执行的时间
- 这个 task 放回DelayQueue队列中

#### 实例代码

```
    /**
    创建一个给定初始延迟的间隔性的任务，之后的下次执行时间是上一次任务从执行到结束所需要的时间+* 给定的间隔时间
    */
    ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
        scheduledExecutorService.scheduleWithFixedDelay(()->{
            System.out.println("current Time" + System.currentTimeMillis());
            System.out.println(Thread.currentThread().getName()+"正在执行");
        }, 1, 3, TimeUnit.SECONDS);
```



```
    /**
    创建一个给定初始延迟的间隔性的任务，之后的每次任务执行时间为 初始延迟 + N * delay(间隔) 
    */
    ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
            scheduledExecutorService.scheduleAtFixedRate(()->{
            System.out.println("current Time" + System.currentTimeMillis());
            System.out.println(Thread.currentThread().getName()+"正在执行");
        }, 1, 3, TimeUnit.SECONDS);;
```

#### 使用场景

周期性执行任务的场景，需要限制线程数量的场景

回到面试题：**说说几种常见的线程池及使用场景？**

回答这四种经典线程池 **：newFixedThreadPool，newSingleThreadExecutor，newCachedThreadPool，newScheduledThreadPool，分线程池特点，工作机制，使用场景分开描述，再分析可能存在的问题，比如newFixedThreadPool内存飙升问题** 即可





### 文章来源

[带你聊聊 Java 并发编程之线程基础](https://mp.weixin.qq.com/s/zn7hD2XILnBnL8Jdcg4biA)

[如何优雅的使用和理解线程池](https://juejin.im/post/5b5e5fa96fb9a04fb900e1ce)

[Java线程池解析](https://juejin.im/post/5d1882b1f265da1ba84aa676#heading-37)