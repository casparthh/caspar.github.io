---
title: 多线程 - 线程池
date: 2020-02-16 21:05:29
toc: true
categories:
- 技术笔记
tags: 
- Thread
---
#### 一、Executor
执行者，是一个接口类，他有一个方法叫执行，那么执行的东西是 Runnable。

#### 二、ExecutorService
是从Executor继承，除了去实现Executor可以去执行一个任务之外，还完善了整个任务执行器的一个生命周期，就拿线程池来举例子，一个线程池里面一堆的线程就是一堆的工人，执行完一个任务之后我这个线程怎么结束啊；  
线程池定义了这样一些个方法：
* void shutdown();//结束
* List<Runnable> shutdownNow();//马上结束
* boolean isShutdown();//是否结束了
* boolean isTerminated();//是不是整体都执行完了
* boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;//等着结束，等多长时间，时间到了还不结束的话他 就返回false
* <T> Future<T> submit(Callable<T> task);
* Future<?> submit(Runnable task);
* List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
* <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
<!--more-->

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

#### 三、Callable
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

#### 四、Future
Future代表的是那个Callable被执行完了之后我怎么才能拿到那个结果啊，它会封装到一个Future里面。
Future将来，未来。未来你执行完之后可以把这个结果放到这个未来有可能执行完的结果里头，所以Future代表的是未来执行完的一个结果。
把Callable的任务扔给线程池，线程池异步的执行完了，就是把任务交给线程池之后，调用get方法直到有结果之后get会返回。Callable一般是配合线程池和Future来用的。

#### 五、FutureTask
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

#### 六、CompletableFuture
CompletableFuture他的底层用的是ForkJoinPool，底层特别复杂，但是用法特别灵活。

#### 七、目前JDK提供的有两种类型
1. ThreadPoolExecutor 普通的线程池
2. ForkJoinPool

#### 八、ThreadPoolExecutor
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

#### 九、JDK给我们提供了一些默认的线程池的实现
##### 1. SingleThreadPool
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

##### 2. CachedThreadPool 
CachedThreadPool的特点，就是你来一个任务我给你启动一个线程，当然前提是我的线程池里面有线程存在而且他还没有到达60秒钟的回收时间的时候，来一个任务，如果有线程存在我就用现有的线程，但是在有新的任务来的时候，如果其他线程忙就启动一个新的，CachedThreadPool用的任务队列是 synchronousQueue，它是一个手递手容量为空的Queue，就是你来一个东西必须得有一个线程把他拿走，不然我提交任务的线程从这阻塞住了。
```
ExecutorService executor = Executors.newCachedThreadPool();
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

##### 3. FixedThreadPool
FixedThreadPool指定一个参数，到底有多少个线程，你看他的核心线程和最大线程都是固定的，因为他的最大线程和核心线程都是固定的就没有回收之说，所以把keepAliveTime指定成0，这里用的是LinkedBlockingQueue
```
ExecutorService executor = Executors.newFixedThreadPool(10);
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
##### 4. ScheduledPool
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

##### 5. WorkStealingPool (它实际上是new了一个ForkJoinPool)
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
##### 6. ForkJoinPool
它适合把大任务切分成一个一个的小任务去运行，小任务还是觉得比较大，再切。切完这个任务执行完了要进行一个汇总。

##### 7. Cache vs Fixed
什么时候用Cache什么时候用Fixed，你得精确的控制有多少个线程数，控制数量问题多数情况下得预估并发量。如果线程池中的数量过多，最终他们会竞争稀缺的处理器和内存资源，浪费大量的时间在上下文切换上，反之，如果线程的数目过少，正如你的应用所面临的情况，处理器的一些核可能就无法充分利用。
《Java并发编程实战》作者 Brian Goetz建议，线程池大小与处理器的利用率之比可以使用公式来进行计算估算:线程池=你有多少个cpu 乘以 cpu期望利用率 乘以 (1+ W/C)。W除以C是等待时间与计算时间的比率。


#### 十、线程池的一些5种状态
1. RUNNING:正常运行的;
2. SHUTDOWN:调用了shutdown方法了进入了shutdown状态; 
3. STOP:调用了shutdownnow马上让他停止; 
4. TIDYING:调用了shutdown然后这个线程也执行完了，现在正在整理的这个过程叫TIDYING; 
5. TERMINATED:整个线程全部结束;
