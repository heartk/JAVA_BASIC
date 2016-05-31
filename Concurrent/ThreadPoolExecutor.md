

## 线程池的使用

在前面的文章中，我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：
如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的
效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种方法使得线程可以服用，就是执行完一任务，并不被销毁，而是可以继续执行其他的任务？

在 Java 中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池，首先我们从 最核心的
ThreadPoolExecutor 类中的方法讲起，然后再讲述它的实现原理，接着给出了它的使用示例，最后讨论了一下如何合理配置线程池的大小。

以下是本文的目录大纲：
一、 Java中的ThreadPoolExecutor类
二、深入剖析线程池的实现原理
三、使用示例
四、如何合理配置线程池的大小

### Java 中的ThreadPoolExecutor 类

    java.util.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻的了解Java中的线程池，
    必须先了解这个类。下面我们来看一下ThreadPoolExecutor类的具体实现源码。
    
    在 ThreadPoolExecutor 类中提供了四个构造方法：
```    
    public class ThreadPoolExecutor extends AbstractExecutorService {
        
        ······
        public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit
            BlockingQueue<Runnable> workQueue);
        
        public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
            BlockingQueue<Runnable> workQueue, RejectedExecutionHandle handler);
        
        public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize, long keepAliveTime, TimeUnit unit,
            BlockingQueue<Runnable> workQueue, RejectedExecutionHandle handle);
        
        public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
            BolckingQueue<Runnable workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler);
        
        ······
    }
```

    从上面的代码可以得知， ThreadPoolExecutor 继承了 AbstractExecutorService 类，并提供了四个构造器，事实上，
通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用第四个构造器进行初始化工作。

    下面解释下一个构造器中的各个参数的含义

* corePoolSize: 核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，
                线程池中没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了 prestartAllCoreThread()或者prestartCoreThread()方法，从这个两个
                方法的名字可以看出，是预创建线程的意思，即在没有任务来之前就创建 corePoolSize 个线程或者一个线程。默认情况下，
                在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目
                达到corePoolSize之后，就会把到达的任务放到缓存队列当中；

* maximumPoolSize: 线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线城池中最多能创建多少个线程；

* keepAliveTime: 表示线程没有任务执行时最多保持多久时间会停止。默认情况下，只有当线程池中的线程数大于 corePoolSize时
                 keepAliveTime才会起作用，直到线程池中的线程数不大于 corePoolSize。即当线程池中的线程数大于corePoolSize时，
                 如果一个线程空闲的时间达到KeepAliveTime,则会终止，直到线程池中的线程书不超过 corePoolSize.但是如果调了
                 allowCoreThreadTimeOut(boolean)方法，在线程池中的线程书不大于corePoolSize时，keepAliveTime参数也会起作用，
                 直到线程池中的线程数为0；
* unit: 参数keepAliveTime 的时间单位，有7种取值，在Ti么Unit类中有7种静态属性：

```
TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒

```

* workQueue: 一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大的影响，
             一般来说，这里的阻塞队列有一下几种选择：
```
ArrayBlockingQueue;
LinkedBlockingQueue;
SynchronousQueue;
```
    ArrayBlockingQueue 和priorityBlockingQueue 使用较少，一般使用 LinkedBlockingQueue 和Synchronous。线程池的排队
    策略与 BlockingQueue有关
* threadFactory: 线程工厂，主要用来创建线程
* handler: 表示当拒绝处理任务时的策略，有以下四种取值：

```
ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException 异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy: 丢弃队列前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy: 由调用线程处理该任务

```
具体参数的配置与线程池的关系将在下一节讲述。
从上面给出的ThreadPoolExecutor 类的代码可以知道，ThreadPoolExecutor 继承了 AbstractExecutorService, 我们来看一下 AbstractExecutor 的实现：

```
public abstract class AbstractExecutorService implements ExecutorService {

    protected <T> RunnableFuture<T> newTastFor(Runnable runnable,T value){};
    protected <T> RunnableFuture<T> newTastFor(Callable<T> callable) {};
    public Future<?> submit(Runnable task, T reasult) {};
    public <T> Future<T> submit(Runnable task, T result) {};
    public <T> Future<T> submit(Callable<T> task) {};
    private <T>T doInvokeAny(Collection<? extends Callable<T> tasks, boolean timed, long nanos) {
        throws InterruptedException, ExcutionException, TimeoutException {
            、、、
        }
    }
       public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
    };
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
    };
}
```
AbstractExecutorService 是一个抽象类，它实现了ExecutorService接口。
我们接着看 ExecutorService 接口实现：
```
public interface ExecutorService extends Executor {
    void shutdown();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    <T>　Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(runnable task);
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T> tasks) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
 }
```
而ExecutorService又是继承了Executor接口，我们看一下Executor接口的实现：
```
public interface Executor {
    void execute(Runnable command);
}
```
到这里，大家应该明白了ThreadPoolExecutor、AbstractExecutorService、ExecutorService和Executor几个之间的关系了。

　　Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；

　　然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；

　　抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；

　　然后ThreadPoolExecutor继承了类AbstractExecutorService。

　　在ThreadPoolExecutor类中有几个非常重要的方法：
　　
```
execute()
submit()
shutdown()
shutdownNow()
```
　execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

　submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果（Future相关内容将在下一篇讲述）。

　shutdown()和shutdownNow()是用来关闭线程池的。

　还有很多其他的方法：

　比如：getQueue() 、getPoolSize() 、getActiveCount()、getCompletedTaskCount()等获取与线程池相关属性的方法，有兴趣的朋友可以自行查阅API。

### 二、深入剖析线程池实现原理
在上一届我们从宏观上介绍了 ThreadPoolExecutor, 下面我们来深入解析一下线程池的具体使用实现原理，将从下面几个方面讲解:
1. 线程池状态
2. 任务的执行
3. 线程池中的线程初始化
4. 任务缓存队列及排队策略
5. 任务拒绝策略
6. 线程池的关闭
7. 线程池容量的动态调整

###### 1. 线程池的状态
    在 ThreadPoolExecutor中定义了一个volatile变量，另外定义了几个static final 变量表示线程池的各个状态：
```    
    volatile int runState;
    static final int RUNNING = 0;
    static final int SHUTDOWN = 1;
    static final int STOP = 2;
    static final int TERMINATED = 3;
    
```
runState 表示当前线程池的状态，它是一个 volatile 变量用来保证线程之间的可见性；
下面几个static final 变量表示 runState 可能的几个取值。
当创建线程池后，初始时，线程池处于 RUNNING 状态；
如果调用了 shutdown() 方法， 则线程池处于 SHUTDOWN 状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；
如果调用了 shutdown() 方法， 则线程池处于 STOP 状态，此时线程池不能接受新的任务，并且回去尝试终止正在执行的任务；
当线程池 处于 SHUTDOWN 或 STOP 状态，并且所有工作线程已经撤销，任务缓存队列已经清空或执行结束后， 线程池设置为 TERMINATED 状态。

###### 2. 任务的执行

在了解将任务提交给线程池到任务执行完毕整个过程之前，我们来看一下ThreadPoolExecutor 类中其他的一些比较重要成员变量：
```
private final BlockingQueue<Runnable> workQueue; // 任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock(); // 线程池的主要状态锁，对线程池状态，比如：线程池大小、runState等 的改变都要使用这个锁

private final HashSet<Worker> worders = new HashSet<Worker>(); // 用来存放工作集

private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
 
private volatile int   poolSize;       //线程池中当前的线程数
 
private volatile RejectedExecutionHandler handler; //任务拒绝策略
 
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
 
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
 
private long completedTaskCount;   //用来记录已经执行完毕的任务个数

```
每个变量的作用都已经标明出来了，这里要重点解释一下corePoolSize、maximumPoolSize、largestPoolSize三个变量。

　　corePoolSize在很多地方被翻译成核心池大小，其实我的理解这个就是线程池的大小。举个简单的例子：

　　假如有一个工厂，工厂里面有10个工人，每个工人同时只能做一件任务。

　　因此只要当10个工人中有工人是空闲的，来了任务就分配给空闲的工人做；

　　当10个工人都有任务在做时，如果还来了任务，就把任务进行排队等待；

　　如果说新任务数目增长的速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招4个临时工人进来；

　　然后就将任务也分配给这4个临时工人做；

　　如果说着14个工人做任务的速度还是不够，此时工厂主管可能就要考虑不再接收新的任务或者抛弃前面的一些任务了。

　　当这14个工人当中有人空闲时，而新任务增长的速度又比较缓慢，工厂主管可能就考虑辞掉4个临时工了，只保持原来的10个工人，
　　毕竟请额外的工人是要花钱的。

 

　　这个例子中的corePoolSize就是10，而maximumPoolSize就是14（10+4）。

　　也就是说corePoolSize就是线程池大小，maximumPoolSize在我看来是线程池的一种补救措施，即任务量突然过大时的一种补救措施。

　　不过为了方便理解，在本文后面还是将corePoolSize翻译成核心池大小。

　　largestPoolSize只是一个用来起记录作用的变量，用来记录线程池中曾经有过的最大线程数目，跟线程池的容量没有任何关系。

　　下面我们进入正题，看一下任务从提交到最终执行完毕经历了哪些过程。

　　在ThreadPoolExecutor类中，最核心的任务提交方法是execute()方法，虽然通过submit也可以提交任务，
　　但是实际上submit方法里面最终调用的还是execute()方法，所以我们只需要研究execute()方法的实现原理即可：

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        if (runState == RUNNING && workQueue.offer(command)) {
            if (runState != RUNNING || poolSize == 0)
                ensureQueuedTaskHandled(command);
        }
        else if (!addIfUnderMaximumPoolSize(command))
            reject(command); // is shutdown or saturated
    }
}
```
上面的代码可能看起来不是那么容易理解，下面我们一句一句解释：

　　首先，判断提交的任务command是否为null，若是null，则抛出空指针异常；

　　接着是这句，这句要好好理解一下：
```
 if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) 
```
    由于是或条件运算符，所以先计算前半部分的值，如果线程池中当前线程数不小于核心池大小，那么就会直接进入下面的if语句块了。
如果线程池中当前线程数小于核心池大小，则接着执行后半部分，也就是执行

```
    addIfUnderCorePoolSize(command)
```
    如果执行完addIfUnderCorePoolSize这个方法返回false，则继续执行下面的if语句块，否则整个方法就直接执行完毕了。
　　如果执行完addIfUnderCorePoolSize这个方法返回false，然后接着判断：
　　
```
    if(runState == RUNNING && work!ueue.offer(command))
```
如果当前线程池处于RUNNING状态，则将任务放入任务缓存队列；如果当前线程池不处于RUNNING状态或者任务放入缓存队列失败，则执行：
```
    addIfUnderMaximumPoolSize 方法失败，则执行reject() 方法进行任务拒绝处理。
```    
    如果执行 addIfUnderMaximumPoolSize(command)
    回到前面：
```
    if (runState == RUNNING && workQueue.offer(command))
```
    这句的执行，如果说当前线程池处于RUNNING状态且将任务放入任务缓存队列成功，则继续进行判断：
    　
```
    if (runState != RUNNING || poolSize == 0)
```
    这句判断是为了防止在将此任务添加进任务缓存队列的同时 
    其他线程突然调用shutdown或者shutdownNow方法关闭了线程池的一种应急措施。如果是这样就执行：
```
    ensureQueuedTaskHandled(command)
```
    进行应急处理，从名字可以看出是保证，添加到任务缓存队列中的任务得到处理。
    我们接着看2个关键方法的实现： addIfUnderCorePoolSize 和 addIfUnderMaximumPoolSize:
    
```
private boolean addIfUnderCorePoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if ( poolSize < corePoolSize && runState = RUNNING) 
            t = addThread(firstTask); // 创建线程去执行 firstTask 任务 
    } finally {
        mainLock.unlock();
    }
    if ( t == null ) {
        if ( t = null )
            return false;
    }
    t.start();
    return true;
}
```
这个是addIfUnderCorePoolSize方法的具体实现，从名字可以看出它的意图就是当低于核心吃大小时执行的方法。
下面看其具体实现，首先获取到锁，因为这地方涉及到线程池状态的变化，先通过if语句判断当前线程池中的线程数目是否小于核心池大小，
有朋友也许会有疑问：前面在execute()方法中不是已经判断过了吗，只有线程池当前线程数目小于核心池大小才会执行addIfUnderCorePoolSize方法的，
为何这地方还要继续判断？原因很简单，前面的判断过程中并没有加锁，因此可能在execute方法判断的时候poolSize小于corePoolSize，
而判断完之后，在其他线程中又向线程池提交了任务，就可能导致poolSize不小于corePoolSize了，所以需要在这个地方继续判断。
然后接着判断线程池的状态是否为RUNNING，原因也很简单，因为有可能在其他线程中调用了shutdown或者shutdownNow方法。然后就是执行
```
 t = addThread(firstTask);
```
这个方法也非常关键，传进去的参数为提交的任务，返回值为 Thread 类型， 然后接着在下面判断 t 是否为空，为空则表明穿件线程失败
（ 即 poolSize >= corePoolSize 或者 runState 不等于 RUNNING ,否则调用 t.start() 方法启动线程。

我们来看一下 addThread 方法的实现：

```
private Thread addThread(Runnable firstTask) {
    Worker w = new Worker(firstTask);
    Thread t = threadFactory.newThread(w); // 创建一个线程，执行任务
    if( t != null ) {
        w.thread = t; // 创建一个线程，执行任务
        workers.add(w);
        int nt = ++ poolSize; // 当先线程数加1 
        if （ nt > largestPoolSize )
            largestPoolSize = nt;
    }
    return t;
}

```
在 addThread 方法中，首先用提交的任务创建了一个 Worker 对象，然后调用线程工厂 threadFactory 创建了一个新的线程t, 
然后将线程的引用赋值给了 Worker 对象的成员变量thread,接着通过workers.add(w) 将Worker 对象添加到工作集当中。
下面我们看一下 Worker 类的实现：
 ```
private final class Worker implements Runnable {
    private final ReentrantLock runLock = new ReentrantLock();
    private Runnable firstTask;
    volatile long completedTask;
    Thread thread;
    Worker ( Runnable firstTask ) {
        this.firstTask = firstTask;
    }
    boolean isActive() {
        return runLock.isLocked();
    }
    void interruptIfIdle() {
        final ReentrantLock runLock = this.runLock;
        if (runLock.tryLock()) {
            try {
        if (thread != Thread.currentThread())
        thread.interrupt();
            } finally {
                runLock.unlock();
            }
        }
    }
    void interruptNow() {
        thread.interrupt();
    }
 
    private void runTask(Runnable task) {
        final ReentrantLock runLock = this.runLock;
        runLock.lock();
        try {
            if (runState < STOP &&
                Thread.interrupted() &&
                runState >= STOP)
            boolean ran = false;
            beforeExecute(thread, task);   //beforeExecute方法是ThreadPoolExecutor类的一个方法，没有具体实现，用户可以根据
            //自己需要重载这个方法和后面的afterExecute方法来进行一些统计信息，比如某个任务的执行时间等           
            try {
                task.run();
                ran = true;
                afterExecute(task, null);
                ++completedTasks;
            } catch (RuntimeException ex) {
                if (!ran)
                    afterExecute(task, ex);
                throw ex;
            }
        } finally {
            runLock.unlock();
        }
    }
 
    public void run() {
        try {
            Runnable task = firstTask;
            firstTask = null;
            while (task != null || (task = getTask()) != null) {
                runTask(task);
                task = null;
            }
        } finally {
            workerDone(this);   //当任务队列中没有任务时，进行清理工作       
        }
    }
 }
 
 
 ```
 　它实际上实现了Runnable接口，因此上面的Thread t = threadFactory.newThread(w);效果跟下面这句的效果基本一样：
 　
 ```
    Thread t = new Thread(w);
 ```
相当于传进去了一个 Runnable 接口，在线程 t 中执行这个 Runnable。
既然Worker 实现了 Runnable 接口，那么自然最核心的方法便是 run() 方法了：

```
    public void run() {
        try {
            Runnable task = firstTask;
            firstTask = null;
            while ( task != null || (task = getTask()) != null ) {
                runTask(task);
                task = null;
            }
        } finally {
            workerDone(this);
        }
        
    }
```
　　从run方法的实现可以看出，它首先执行的是通过构造器传进来的任务firstTask，在调用runTask()执行完firstTask之后，
　　在while循环里面不断通过getTask()去取新的任务来执行，那么去哪里取呢？自然是从任务缓存队列里面去取，
　　getTask是ThreadPoolExecutor类中的方法，并不是Worker类中的方法，下面是getTask方法的实现：
　
```
Runnable getTask() {
    for (;;) {
        try {
            int state = runState;
            if (state > SHUTDOWN)
                return null;
            Runnable r;
            if (state == SHUTDOWN)  // Help drain queue
                r = workQueue.poll();
            else if (poolSize > corePoolSize || allowCoreThreadTimeOut) //如果线程数大于核心池大小或者允许为核心池线程设置空闲时间，
                //则通过poll取任务，若等待一定的时间取不到任务，则返回null
                r = workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS);
            else
                r = workQueue.take();
            if (r != null)
                return r;
            if (workerCanExit()) {    //如果没取到任务，即r为null，则判断当前的worker是否可以退出
                if (runState >= SHUTDOWN) // Wake up others
                    interruptIdleWorkers();   //中断处于空闲状态的worker
                return null;
            }
            // Else retry
        } catch (InterruptedException ie) {
            // On interruption, re-check runState
        }
    }
}
```

　在getTask中，先判断当前线程池状态，如果runState大于SHUTDOWN（即为STOP或者TERMINATED），则直接返回null。

　　如果runState为SHUTDOWN或者RUNNING，则从任务缓存队列取任务。

　　如果当前线程池的线程数大于核心池大小corePoolSize或者允许为核心池中的线程设置空闲存活时间，则调用poll(time,timeUnit)来取任务，这个方法会等待一定的时间，如果取不到任务就返回null。

　　然后判断取到的任务r是否为null，为null则通过调用workerCanExit()方法来判断当前worker是否可以退出，我们看一下workerCanExit()的实现：

```

private boolean workerCanExit() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    boolean canExit;
    //如果runState大于等于STOP，或者任务缓存队列为空了
    //或者  允许为核心池线程设置空闲存活时间并且线程池中的线程数目大于1
    try {
        canExit = runState >= STOP ||
            workQueue.isEmpty() ||
            (allowCoreThreadTimeOut &&
             poolSize > Math.max(1, corePoolSize));
    } finally {
        mainLock.unlock();
    }
    return canExit;
}
```
也就是说如果线程池处于STOP状态、或者任务队列已为空或者允许为核心池线程设置空闲存活时间并且线程数大于1时，允许worker退出。
如果允许worker退出，则调用interruptIdleWorkers()中断处于空闲状态的worker，我们看一下interruptIdleWorkers()的实现：
 ```
void interruptIdleWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)  //实际上调用的是worker的interruptIfIdle()方法
            w.interruptIfIdle();
    } finally {
        mainLock.unlock();
    }
}
 ```
从实现可以看出，它实际上调用的是worker的interruptIfIdle()方法，在worker的interruptIfIdle()方法中：
```
void interruptIfIdle() {
    final ReentrantLock runLock = this.runLock;
    if (runLock.tryLock()) {    //注意这里，是调用tryLock()来获取锁的，因为如果当前worker正在执行任务，锁已经被获取了，是无法获取到锁的
                                //如果成功获取了锁，说明当前worker处于空闲状态
        try {
    if (thread != Thread.currentThread())  
    thread.interrupt();
        } finally {
            runLock.unlock();
        }
    }
}
```
 这里有一个非常巧妙的设计方式，假如我们来设计线程池，可能会有一个任务分派线程，当发现有线程空闲时，就从任务缓存队列中取一个任务交给空闲线程执行。
 但是在这里，并没有采用这样的方式，因为这样会要额外地对任务分派线程进行管理，无形地会增加难度和复杂度，
 这里直接让执行完任务的线程去任务缓存队列里面取任务来执行。

我们再看addIfUnderMaximumPoolSize方法的实现，这个方法的实现思想和addIfUnderCorePoolSize方法的实现思想非常相似，
唯一的区别在于addIfUnderMaximumPoolSize方法是在线程池中的线程数达到了核心池大小并且往任务队列中添加任务失败的情况下执行的：
```
private boolean addIfUnderMaximumPoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (poolSize < maximumPoolSize && runState == RUNNING)
            t = addThread(firstTask);
    } finally {
        mainLock.unlock();
    }
    if (t == null)
        return false;
    t.start();
    return true;
}

```

看到没有，其实它和addIfUnderCorePoolSize方法的实现基本一模一样，只是if语句判断条件中的poolSize < maximumPoolSize不同而已。

　　到这里，大部分朋友应该对任务提交给线程池之后到被执行的整个过程有了一个基本的了解，下面总结一下：

　　1）首先，要清楚corePoolSize和maximumPoolSize的含义；

　　2）其次，要知道Worker是用来起到什么作用的；

　　3）要知道任务提交给线程池之后的处理策略，这里总结一下主要有4点：

如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；
如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，
若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；
如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；
如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，
直至线程池中的线程数目不大于corePoolSize；如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被终止。
    





































## 线程池的使用

在前面的文章中，我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：
如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的
效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种方法使得线程可以服用，就是执行完一任务，并不被销毁，而是可以继续执行其他的任务？

在 Java 中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池，首先我们从 最核心的
ThreadPoolExecutor 类中的方法讲起，然后再讲述它的实现原理，接着给出了它的使用示例，最后讨论了一下如何合理配置线程池的大小。

以下是本文的目录大纲：
一、 Java中的ThreadPoolExecutor类
二、深入剖析线程池的实现原理
三、使用示例
四、如何合理配置线程池的大小

### Java 中的ThreadPoolExecutor 类

    java.util.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻的了解Java中的线程池，
    必须先了解这个类。下面我们来看一下ThreadPoolExecutor类的具体实现源码。
    
    在 ThreadPoolExecutor 类中提供了四个构造方法：
```    
    public class ThreadPoolExecutor extends AbstractExecutorService {
        
        ······
        public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit
            BlockingQueue<Runnable> workQueue);
        
        public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
            BlockingQueue<Runnable> workQueue, RejectedExecutionHandle handler);
        
        public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize, long keepAliveTime, TimeUnit unit,
            BlockingQueue<Runnable> workQueue, RejectedExecutionHandle handle);
        
        public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
            BolckingQueue<Runnable workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler);
        
        ······
    }
```

    从上面的代码可以得知， ThreadPoolExecutor 继承了 AbstractExecutorService 类，并提供了四个构造器，事实上，
通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用第四个构造器进行初始化工作。

    下面解释下一个构造器中的各个参数的含义

* corePoolSize: 核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，
                线程池中没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了 prestartAllCoreThread()或者prestartCoreThread()方法，从这个两个
                方法的名字可以看出，是预创建线程的意思，即在没有任务来之前就创建 corePoolSize 个线程或者一个线程。默认情况下，
                在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目
                达到corePoolSize之后，就会把到达的任务放到缓存队列当中；

* maximumPoolSize: 线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线城池中最多能创建多少个线程；

* keepAliveTime: 表示线程没有任务执行时最多保持多久时间会停止。默认情况下，只有当线程池中的线程数大于 corePoolSize时
                 keepAliveTime才会起作用，直到线程池中的线程数不大于 corePoolSize。即当线程池中的线程数大于corePoolSize时，
                 如果一个线程空闲的时间达到KeepAliveTime,则会终止，直到线程池中的线程书不超过 corePoolSize.但是如果调了
                 allowCoreThreadTimeOut(boolean)方法，在线程池中的线程书不大于corePoolSize时，keepAliveTime参数也会起作用，
                 直到线程池中的线程数为0；
* unit: 参数keepAliveTime 的时间单位，有7种取值，在Ti么Unit类中有7种静态属性：

```
TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒

```

* workQueue: 一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大的影响，
             一般来说，这里的阻塞队列有一下几种选择：
```
ArrayBlockingQueue;
LinkedBlockingQueue;
SynchronousQueue;
```
    ArrayBlockingQueue 和priorityBlockingQueue 使用较少，一般使用 LinkedBlockingQueue 和Synchronous。线程池的排队
    策略与 BlockingQueue有关
* threadFactory: 线程工厂，主要用来创建线程
* handler: 表示当拒绝处理任务时的策略，有以下四种取值：

```
ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException 异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy: 丢弃队列前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy: 由调用线程处理该任务

```
具体参数的配置与线程池的关系将在下一节讲述。
从上面给出的ThreadPoolExecutor 类的代码可以知道，ThreadPoolExecutor 继承了 AbstractExecutorService, 我们来看一下 AbstractExecutor 的实现：

```
public abstract class AbstractExecutorService implements ExecutorService {

    protected <T> RunnableFuture<T> newTastFor(Runnable runnable,T value){};
    protected <T> RunnableFuture<T> newTastFor(Callable<T> callable) {};
    public Future<?> submit(Runnable task, T reasult) {};
    public <T> Future<T> submit(Runnable task, T result) {};
    public <T> Future<T> submit(Callable<T> task) {};
    private <T>T doInvokeAny(Collection<? extends Callable<T> tasks, boolean timed, long nanos) {
        throws InterruptedException, ExcutionException, TimeoutException {
            、、、
        }
    }
       public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
    };
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
    };
}
```
AbstractExecutorService 是一个抽象类，它实现了ExecutorService接口。
我们接着看 ExecutorService 接口实现：
```
public interface ExecutorService extends Executor {
    void shutdown();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    <T>　Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(runnable task);
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T> tasks) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
 }
```
而ExecutorService又是继承了Executor接口，我们看一下Executor接口的实现：
```
public interface Executor {
    void execute(Runnable command);
}
```
到这里，大家应该明白了ThreadPoolExecutor、AbstractExecutorService、ExecutorService和Executor几个之间的关系了。

　　Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；

　　然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；

　　抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；

　　然后ThreadPoolExecutor继承了类AbstractExecutorService。

　　在ThreadPoolExecutor类中有几个非常重要的方法：
　　
```
execute()
submit()
shutdown()
shutdownNow()
```
　execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

　submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果（Future相关内容将在下一篇讲述）。

　shutdown()和shutdownNow()是用来关闭线程池的。

　还有很多其他的方法：

　比如：getQueue() 、getPoolSize() 、getActiveCount()、getCompletedTaskCount()等获取与线程池相关属性的方法，有兴趣的朋友可以自行查阅API。

### 二、深入剖析线程池实现原理
在上一届我们从宏观上介绍了 ThreadPoolExecutor, 下面我们来深入解析一下线程池的具体使用实现原理，将从下面几个方面讲解:
1. 线程池状态
2. 任务的执行
3. 线程池中的线程初始化
4. 任务缓存队列及排队策略
5. 任务拒绝策略
6. 线程池的关闭
7. 线程池容量的动态调整

###### 1. 线程池的状态
    在 ThreadPoolExecutor中定义了一个volatile变量，另外定义了几个static final 变量表示线程池的各个状态：
```    
    volatile int runState;
    static final int RUNNING = 0;
    static final int SHUTDOWN = 1;
    static final int STOP = 2;
    static final int TERMINATED = 3;
    
```
runState 表示当前线程池的状态，它是一个 volatile 变量用来保证线程之间的可见性；
下面几个static final 变量表示 runState 可能的几个取值。
当创建线程池后，初始时，线程池处于 RUNNING 状态；
如果调用了 shutdown() 方法， 则线程池处于 SHUTDOWN 状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；
如果调用了 shutdown() 方法， 则线程池处于 STOP 状态，此时线程池不能接受新的任务，并且回去尝试终止正在执行的任务；
当线程池 处于 SHUTDOWN 或 STOP 状态，并且所有工作线程已经撤销，任务缓存队列已经清空或执行结束后， 线程池设置为 TERMINATED 状态。

###### 2. 任务的执行

在了解将任务提交给线程池到任务执行完毕整个过程之前，我们来看一下ThreadPoolExecutor 类中其他的一些比较重要成员变量：
```
private final BlockingQueue<Runnable> workQueue; // 任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock(); // 线程池的主要状态锁，对线程池状态，比如：线程池大小、runState等 的改变都要使用这个锁

private final HashSet<Worker> worders = new HashSet<Worker>(); // 用来存放工作集

private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
 
private volatile int   poolSize;       //线程池中当前的线程数
 
private volatile RejectedExecutionHandler handler; //任务拒绝策略
 
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
 
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
 
private long completedTaskCount;   //用来记录已经执行完毕的任务个数

```
每个变量的作用都已经标明出来了，这里要重点解释一下corePoolSize、maximumPoolSize、largestPoolSize三个变量。

　　corePoolSize在很多地方被翻译成核心池大小，其实我的理解这个就是线程池的大小。举个简单的例子：

　　假如有一个工厂，工厂里面有10个工人，每个工人同时只能做一件任务。

　　因此只要当10个工人中有工人是空闲的，来了任务就分配给空闲的工人做；

　　当10个工人都有任务在做时，如果还来了任务，就把任务进行排队等待；

　　如果说新任务数目增长的速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招4个临时工人进来；

　　然后就将任务也分配给这4个临时工人做；

　　如果说着14个工人做任务的速度还是不够，此时工厂主管可能就要考虑不再接收新的任务或者抛弃前面的一些任务了。

　　当这14个工人当中有人空闲时，而新任务增长的速度又比较缓慢，工厂主管可能就考虑辞掉4个临时工了，只保持原来的10个工人，
　　毕竟请额外的工人是要花钱的。

 

　　这个例子中的corePoolSize就是10，而maximumPoolSize就是14（10+4）。

　　也就是说corePoolSize就是线程池大小，maximumPoolSize在我看来是线程池的一种补救措施，即任务量突然过大时的一种补救措施。

　　不过为了方便理解，在本文后面还是将corePoolSize翻译成核心池大小。

　　largestPoolSize只是一个用来起记录作用的变量，用来记录线程池中曾经有过的最大线程数目，跟线程池的容量没有任何关系。

　　下面我们进入正题，看一下任务从提交到最终执行完毕经历了哪些过程。

　　在ThreadPoolExecutor类中，最核心的任务提交方法是execute()方法，虽然通过submit也可以提交任务，
　　但是实际上submit方法里面最终调用的还是execute()方法，所以我们只需要研究execute()方法的实现原理即可：

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        if (runState == RUNNING && workQueue.offer(command)) {
            if (runState != RUNNING || poolSize == 0)
                ensureQueuedTaskHandled(command);
        }
        else if (!addIfUnderMaximumPoolSize(command))
            reject(command); // is shutdown or saturated
    }
}
```
上面的代码可能看起来不是那么容易理解，下面我们一句一句解释：

　　首先，判断提交的任务command是否为null，若是null，则抛出空指针异常；

　　接着是这句，这句要好好理解一下：
```
 if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) 
```
    由于是或条件运算符，所以先计算前半部分的值，如果线程池中当前线程数不小于核心池大小，那么就会直接进入下面的if语句块了。
如果线程池中当前线程数小于核心池大小，则接着执行后半部分，也就是执行

```
    addIfUnderCorePoolSize(command)
```
    如果执行完addIfUnderCorePoolSize这个方法返回false，则继续执行下面的if语句块，否则整个方法就直接执行完毕了。
　　如果执行完addIfUnderCorePoolSize这个方法返回false，然后接着判断：
　　
```
    if(runState == RUNNING && work!ueue.offer(command))
```
如果当前线程池处于RUNNING状态，则将任务放入任务缓存队列；如果当前线程池不处于RUNNING状态或者任务放入缓存队列失败，则执行：
```
    addIfUnderMaximumPoolSize 方法失败，则执行reject() 方法进行任务拒绝处理。
```    
    如果执行 addIfUnderMaximumPoolSize(command)
    回到前面：
```
    if (runState == RUNNING && workQueue.offer(command))
```
    这句的执行，如果说当前线程池处于RUNNING状态且将任务放入任务缓存队列成功，则继续进行判断：
    　
```
    if (runState != RUNNING || poolSize == 0)
```
    这句判断是为了防止在将此任务添加进任务缓存队列的同时 
    其他线程突然调用shutdown或者shutdownNow方法关闭了线程池的一种应急措施。如果是这样就执行：
```
    ensureQueuedTaskHandled(command)
```
    进行应急处理，从名字可以看出是保证，添加到任务缓存队列中的任务得到处理。
    我们接着看2个关键方法的实现： addIfUnderCorePoolSize 和 addIfUnderMaximumPoolSize:
    
```
private boolean addIfUnderCorePoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if ( poolSize < corePoolSize && runState = RUNNING) 
            t = addThread(firstTask); // 创建线程去执行 firstTask 任务 
    } finally {
        mainLock.unlock();
    }
    if ( t == null ) {
        if ( t = null )
            return false;
    }
    t.start();
    return true;
}
```
这个是addIfUnderCorePoolSize方法的具体实现，从名字可以看出它的意图就是当低于核心吃大小时执行的方法。
下面看其具体实现，首先获取到锁，因为这地方涉及到线程池状态的变化，先通过if语句判断当前线程池中的线程数目是否小于核心池大小，
有朋友也许会有疑问：前面在execute()方法中不是已经判断过了吗，只有线程池当前线程数目小于核心池大小才会执行addIfUnderCorePoolSize方法的，
为何这地方还要继续判断？原因很简单，前面的判断过程中并没有加锁，因此可能在execute方法判断的时候poolSize小于corePoolSize，
而判断完之后，在其他线程中又向线程池提交了任务，就可能导致poolSize不小于corePoolSize了，所以需要在这个地方继续判断。
然后接着判断线程池的状态是否为RUNNING，原因也很简单，因为有可能在其他线程中调用了shutdown或者shutdownNow方法。然后就是执行
```
 t = addThread(firstTask);
```
这个方法也非常关键，传进去的参数为提交的任务，返回值为 Thread 类型， 然后接着在下面判断 t 是否为空，为空则表明穿件线程失败
（ 即 poolSize >= corePoolSize 或者 runState 不等于 RUNNING ,否则调用 t.start() 方法启动线程。

我们来看一下 addThread 方法的实现：

```
private Thread addThread(Runnable firstTask) {
    Worker w = new Worker(firstTask);
    Thread t = threadFactory.newThread(w); // 创建一个线程，执行任务
    if( t != null ) {
        w.thread = t; // 创建一个线程，执行任务
        workers.add(w);
        int nt = ++ poolSize; // 当先线程数加1 
        if （ nt > largestPoolSize )
            largestPoolSize = nt;
    }
    return t;
}

```
在 addThread 方法中，首先用提交的任务创建了一个 Worker 对象，然后调用线程工厂 threadFactory 创建了一个新的线程t, 
然后将线程的引用赋值给了 Worker 对象的成员变量thread,接着通过workers.add(w) 将Worker 对象添加到工作集当中。
下面我们看一下 Worker 类的实现：
 ```
private final class Worker implements Runnable {
    private final ReentrantLock runLock = new ReentrantLock();
    private Runnable firstTask;
    volatile long completedTask;
    Thread thread;
    Worker ( Runnable firstTask ) {
        this.firstTask = firstTask;
    }
    boolean isActive() {
        return runLock.isLocked();
    }
    void interruptIfIdle() {
        final ReentrantLock runLock = this.runLock;
        if (runLock.tryLock()) {
            try {
        if (thread != Thread.currentThread())
        thread.interrupt();
            } finally {
                runLock.unlock();
            }
        }
    }
    void interruptNow() {
        thread.interrupt();
    }
 
    private void runTask(Runnable task) {
        final ReentrantLock runLock = this.runLock;
        runLock.lock();
        try {
            if (runState < STOP &&
                Thread.interrupted() &&
                runState >= STOP)
            boolean ran = false;
            beforeExecute(thread, task);   //beforeExecute方法是ThreadPoolExecutor类的一个方法，没有具体实现，用户可以根据
            //自己需要重载这个方法和后面的afterExecute方法来进行一些统计信息，比如某个任务的执行时间等           
            try {
                task.run();
                ran = true;
                afterExecute(task, null);
                ++completedTasks;
            } catch (RuntimeException ex) {
                if (!ran)
                    afterExecute(task, ex);
                throw ex;
            }
        } finally {
            runLock.unlock();
        }
    }
 
    public void run() {
        try {
            Runnable task = firstTask;
            firstTask = null;
            while (task != null || (task = getTask()) != null) {
                runTask(task);
                task = null;
            }
        } finally {
            workerDone(this);   //当任务队列中没有任务时，进行清理工作       
        }
    }
 }
 
 
 ```
 　它实际上实现了Runnable接口，因此上面的Thread t = threadFactory.newThread(w);效果跟下面这句的效果基本一样：
 　
 ```
    Thread t = new Thread(w);
 ```
相当于传进去了一个 Runnable 接口，在线程 t 中执行这个 Runnable。
既然Worker 实现了 Runnable 接口，那么自然最核心的方法便是 run() 方法了：

```
    public void run() {
        try {
            Runnable task = firstTask;
            firstTask = null;
            while ( task != null || (task = getTask()) != null ) {
                runTask(task);
                task = null;
            }
        } finally {
            workerDone(this);
        }
        
    }
```
　　从run方法的实现可以看出，它首先执行的是通过构造器传进来的任务firstTask，在调用runTask()执行完firstTask之后，
　　在while循环里面不断通过getTask()去取新的任务来执行，那么去哪里取呢？自然是从任务缓存队列里面去取，
　　getTask是ThreadPoolExecutor类中的方法，并不是Worker类中的方法，下面是getTask方法的实现：
　
```
Runnable getTask() {
    for (;;) {
        try {
            int state = runState;
            if (state > SHUTDOWN)
                return null;
            Runnable r;
            if (state == SHUTDOWN)  // Help drain queue
                r = workQueue.poll();
            else if (poolSize > corePoolSize || allowCoreThreadTimeOut) //如果线程数大于核心池大小或者允许为核心池线程设置空闲时间，
                //则通过poll取任务，若等待一定的时间取不到任务，则返回null
                r = workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS);
            else
                r = workQueue.take();
            if (r != null)
                return r;
            if (workerCanExit()) {    //如果没取到任务，即r为null，则判断当前的worker是否可以退出
                if (runState >= SHUTDOWN) // Wake up others
                    interruptIdleWorkers();   //中断处于空闲状态的worker
                return null;
            }
            // Else retry
        } catch (InterruptedException ie) {
            // On interruption, re-check runState
        }
    }
}
```

　在getTask中，先判断当前线程池状态，如果runState大于SHUTDOWN（即为STOP或者TERMINATED），则直接返回null。

　　如果runState为SHUTDOWN或者RUNNING，则从任务缓存队列取任务。

　　如果当前线程池的线程数大于核心池大小corePoolSize或者允许为核心池中的线程设置空闲存活时间，则调用poll(time,timeUnit)来取任务，这个方法会等待一定的时间，如果取不到任务就返回null。

　　然后判断取到的任务r是否为null，为null则通过调用workerCanExit()方法来判断当前worker是否可以退出，我们看一下workerCanExit()的实现：

```

private boolean workerCanExit() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    boolean canExit;
    //如果runState大于等于STOP，或者任务缓存队列为空了
    //或者  允许为核心池线程设置空闲存活时间并且线程池中的线程数目大于1
    try {
        canExit = runState >= STOP ||
            workQueue.isEmpty() ||
            (allowCoreThreadTimeOut &&
             poolSize > Math.max(1, corePoolSize));
    } finally {
        mainLock.unlock();
    }
    return canExit;
}
```
也就是说如果线程池处于STOP状态、或者任务队列已为空或者允许为核心池线程设置空闲存活时间并且线程数大于1时，允许worker退出。
如果允许worker退出，则调用interruptIdleWorkers()中断处于空闲状态的worker，我们看一下interruptIdleWorkers()的实现：
 ```
void interruptIdleWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)  //实际上调用的是worker的interruptIfIdle()方法
            w.interruptIfIdle();
    } finally {
        mainLock.unlock();
    }
}
 ```
从实现可以看出，它实际上调用的是worker的interruptIfIdle()方法，在worker的interruptIfIdle()方法中：
```
void interruptIfIdle() {
    final ReentrantLock runLock = this.runLock;
    if (runLock.tryLock()) {    //注意这里，是调用tryLock()来获取锁的，因为如果当前worker正在执行任务，锁已经被获取了，是无法获取到锁的
                                //如果成功获取了锁，说明当前worker处于空闲状态
        try {
    if (thread != Thread.currentThread())  
    thread.interrupt();
        } finally {
            runLock.unlock();
        }
    }
}
```
 这里有一个非常巧妙的设计方式，假如我们来设计线程池，可能会有一个任务分派线程，当发现有线程空闲时，就从任务缓存队列中取一个任务交给空闲线程执行。
 但是在这里，并没有采用这样的方式，因为这样会要额外地对任务分派线程进行管理，无形地会增加难度和复杂度，
 这里直接让执行完任务的线程去任务缓存队列里面取任务来执行。

我们再看addIfUnderMaximumPoolSize方法的实现，这个方法的实现思想和addIfUnderCorePoolSize方法的实现思想非常相似，
唯一的区别在于addIfUnderMaximumPoolSize方法是在线程池中的线程数达到了核心池大小并且往任务队列中添加任务失败的情况下执行的：
```
private boolean addIfUnderMaximumPoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (poolSize < maximumPoolSize && runState == RUNNING)
            t = addThread(firstTask);
    } finally {
        mainLock.unlock();
    }
    if (t == null)
        return false;
    t.start();
    return true;
}

```

看到没有，其实它和addIfUnderCorePoolSize方法的实现基本一模一样，只是if语句判断条件中的poolSize < maximumPoolSize不同而已。

　　到这里，大部分朋友应该对任务提交给线程池之后到被执行的整个过程有了一个基本的了解，下面总结一下：

　　1）首先，要清楚corePoolSize和maximumPoolSize的含义；

　　2）其次，要知道Worker是用来起到什么作用的；

　　3）要知道任务提交给线程池之后的处理策略，这里总结一下主要有4点：

如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；
如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，
若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；
如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；
如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，
直至线程池中的线程数目不大于corePoolSize；如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被终止。
    
































