---
layout: post
title: Java线程池源码分析
categories: JavaSE
description: Java线程池源码分析
keywords: leetcode
---

### 线程池
假如没有线程池，当存在较多的并发任务的时候，每执行一次任务，系统就要创建一个线程，任务完成后进行销毁，一旦并发任务过多，频繁的创建和销毁线程将会大大降低系统的效率。线程池能够对线程进行统一的分配，通过固定数量的线程来负责处理任务，避免了频繁的创建和销毁对象，使线程能够重复的利用，执行多个任务。

### Java线程池的结构
![Java线程池结构](https://upload-images.jianshu.io/upload_images/14607771-b2d17a1d103b8d3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***Executor*** 最顶层接口，仅有execute方法。真正的线程池接口应该是它的子接口ExecutorService
***ExecutorService***接口，主要对Executor接口补充了一些方法，例如shutdown()、submit()等方法
***ThreadPoolExecutor***  ExecutorService的默认实现，作为自定义线程池的主要类。
***ScheduledExecutorService*** 用来解决任务重复执行的问题
***ScheduledThreadPoolExecutor*** 继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现。

### ThreadPoolExecutor
ThreadPoolExecutor使ExecutorService的默认实现，其功能也最为基础，通过对该类的源码解析来了解整个线程池的工作机制。

ThreadPoolExecutor最完整的构造器
```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```
***corePoolSize*** 线程池中保存的线程个数，包含了空闲线程
***maximumPoolSize*** 为线程池中最多允许线程个数
***keepAliveTime*** 当线程产生的数量多于core时候，空闲线程在这个时间如果没有新任务将被终止
***unit*** keepAliveTime的时间单位
***workQueue*** 线程工作任务队列。任务被执行前保存至工作队列 常用的有ArrayBlockingQueue、LinkedBlockingQuene、priorityBlockingQuene等等
***threadFactory*** 执行程序创建新线程时使用的工厂，默认为DefaultThreadFactory
***handler*** 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序，主要有四种AbortPolicy、CallerRunsPolicy、DiscardOldestPolicy、DiscardPolicy
#### 工作流程图如图
![image.png](https://upload-images.jianshu.io/upload_images/14607771-2ed6ce29aedcbf16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 源码解析
首先，来看一下几个变量定义
```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

    // Packing and unpacking ctl
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    private static int ctlOf(int rs, int wc) { return rs | wc; }
```
线程池使用一个整型变量的高三位作为线程池的状态。
高位111为RUNNING      注：RUNNING为负数，最小
高位000为SHUTDOWN
高位001表示STOP
高位010表示TIDYING
高位100表示TERMINATED
workerCountOf方法用来取低29位的数值，返回线程池的线程数。
runStateOf方法则用来取高3位的数值，返回当前线程池的状态。
然后，来看用于提交任务的submit方法。
```
public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
```
该方法将Runnable进行了封装，我们这里先不考虑，先看看执行的关键方法execute();
```
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) { // workerCountOF()方法获取当前线程数量，若线程数量小于核心线程数的时候 直接进入addWorker 启动运行
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        // 上面是当线程数量小于core的时候，下面是针对线程数量大于core
        if (isRunning(c) && workQueue.offer(command)) { //若线程池处于运行状态，则添加到阻塞队列中
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command)) //recheck 若线程池已经关闭，就remove任务
                reject(command); //执行拒绝策略
            else if (workerCountOf(recheck) == 0) //线程池处于running状态，但是没有线程，则创建线程
                addWorker(null, false);
        }
        else if (!addWorker(command, false)) // 线程池关闭或者往workQueue提交任务失败，则reject任务
            reject(command);
    }
```
execute方法中可以看到，addWork作为一个创建线程并执行任务的重要方法
```
private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && // rs >= SHUTDOWN 说明线程池不是running; 当线程池关闭，任务为null，workQueue不为空的时候才可以往下进行 
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize)) // core为true 则超过core添加失败
                    return false;
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }
        //到这里说明能够创建线程执行任务
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);
            final Thread t = w.thread; //获取 worker 中的线程对象，Worker的构造方法会调用 ThreadFactory 来创建一个新的线程
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock(); //上锁
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) { // 线程池运行或者 线程池关闭 任务为null
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w); //添加进HashSet
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;  //添加成功
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {  //如果添加成功则启动线程
                    t.start();  
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)  //启动失败则把w从works中移除
                addWorkerFailed(w);  
        }
        return workerStarted;
    }
```
接下来看看Work类中的run方法，中的runWorker方法
```
public void run() {
    runWorker(this);
}
final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // 释放锁，允许中断
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) { //getTask方法不断从阻塞队列中获取任务
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run(); //执行任务的run方法
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }

```
### 三种常见线程池
Executors类中为我们提供了集中常见的线程池，分别有FixedThreadPool、SingleThreadExecutor、CachedThreadPool三种
```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
FixedThreadPool顾名思义  可以设置固定数量的线程池。但是LinkedBlockingQueue是无界阻塞队列，队列可存放的大小为Integer.MAX_VALUE，因此maximumPoolSize和keepAliveTime将会是个无用参数，拒绝策略也没用
```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
同上，不过线程池大小仅为1
```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
线程池的线程数可达到Integer.MAX_VALUE。使用SynchronousQueue作为阻塞队列。newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销；
