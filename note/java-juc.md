# java.util.concurrent

## 并发工具类 - 分类

1. 并发安全: 互斥同步, 非互斥同步, 无同步方案
2. 管理线程, 提高效率
3. 线程协作

## 线程池

### 线程池介绍

- 反复创建线程开销大
- 过多的线程会占用太多线程

---
优点:

- 加快响应速度
- 合理利用cpu和内存
- 方便管理

### 创建和停止线程池

#### 线程池构造函数的参数

- corePoolSIze 核心线程数
- maxPoolSize 最大线程数
- keepAliveTime 保持存活时间
- unit 时间单位
- workQueue 工作队列
- threadFactory 线程工厂
- handler 拒绝策略

是否新增线程的判断顺序:

1. corePoolSize
2. workQueue
3. maxPoolSize

---

- ArrayBlockingQueue 有界队列
- LinkedBlockingQueue 无界队列
- SynchronousQueue 交换队列(size=0)

---

```java
/**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

### 常见线程池的特点和用法

```java
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>(), threadFactory);
    }
   public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>()));
    }
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(), threadFactory));
    }
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
    }
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>(), threadFactory);
    }
    public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new DelegatedScheduledExecutorService (new ScheduledThreadPoolExecutor(1));
    }
    public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {
        return new DelegatedScheduledExecutorService   (new ScheduledThreadPoolExecutor(1, threadFactory));
    }
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

> 线程池的线程数量设定为多少比较合适?

- CPU密集型(加密, 计算hash): 最佳线程数为CPU核心数的1-2倍左右
- 耗时I/O型(读写数据库, 文件, 网络读写等): 最佳线程数一般会大于cpu核心数很多倍, 以JVM线程监控显示繁忙情况为依据,保证线程空闲可以衔接上 -> 线程数 = CPU核心数 * (1 + 平均等待时间 / 平均工作时间)

---

- fixedThreadPool: 固定线程数
- cachedThreadPool: 可缓存, 回收
- scheduledThreadPool: 定时执行
- singleThreadPool: 唯一线程
- workStealingPool: fork/join

> 停止线程池的正确方法

- `java.util.concurrent.ThreadPoolExecutor#isShutdown()`
- `java.util.concurrent.ThreadPoolExecutor#isTerminated()`
- `java.util.concurrent.ThreadPoolExecutor#awaitTermination()`
- `java.util.concurrent.ThreadPoolExecutor#shutdown()`
  - 通知线程池关闭(需要执行完提交的所有任务)
- `java.util.concurrent.ThreadPoolExecutor#shutdownNow()`

```java
    public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(STOP);
            interruptWorkers();
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
```

> 四种拒绝策略

- AbortPolicy: 直接拒绝(抛异常)
- DiscardPolicy: 直接丢掉
- DiscardOldestPolicy: 丢弃最老的线程
- CallerRunsPolicy: 调用者执行当前提交线程

### 组成部分

- 线程池管理器
- 工作线程
- 任务队列
- 任务接口(Task)

### 线程池状态

- RUNNING: 接受新任务并处理排队任务
- SHUTDOWN: 不接受新任务, 但处理排队任务
- STOP: 不接受新任务, 不处理排队任务, 并中断正在运行的任务
- TIDYING: 所有任务都已终止, workerCount为零时, 线程会切换到TIDYING状态, 并将运行terminate()方法
- TERMINATED: terminate()运行完成

### 注意事项

- 注意任务堆积
- 避免线程数过度增加
- 排查线程泄露

## ThreadLocal

### 两大使用场景

1. 每个线程需要一个独享的对象(工具类 -> `SimpleDataFormat` & `Random`)
2. 每个线程内需要保存全局变量(在拦截器中获取用户信息, 可以让不同方法直接使用, 避免参数传递)

### 两个作用

1. 让某个需要用到的对象在线程间隔离(每个线程都有自己独立的对象)
2. 在任何方法中都能轻易取得对象

---

1. 线程安全
2. 不需要加锁, 提高执行效率
3. 更加高效利用内存, 节省开销
4. 免去传参的繁琐

### API

- `T initialValue()`
- `void set(T t)`
- `T get()`
- `void remove()`

### 三个类

- Thread
- ThreadLocal
- ThreadLocalMap

### 注意点

某个对象不再有用, 但占用的内存却不能回收

#### value泄漏

ThreadLocalMap的每个entry都是一个对key的弱引用, 对value的强引用. 如果线程不终止,那么key对应的value就无法回收  
Thread -> ThreadLocalMap -> Entry(key==null) -> value  
调用remove方法, 就会删除对应的entry对象, 可以避免内存泄漏

#### ThreadLocal空指针

对于包装类型, get前需要set,装箱开箱的过程可能会造成空指针

#### 共享对象

如果ThreadLocal封装的是static对象, 无法保证线程安全

## 锁

lock -> read -> load -> use  -> assign -> store -> write -> unlock

### Lock接口

锁是一种工具， 用于控制对共享资源的访问

synchronized：

- 效率低： 锁的释放情况少， 试图获得锁时不能设定超时， 不能中断一个正在试图获得锁的线程。
- 不够灵活：加锁和释放的实际单一， 每个锁仅有单一的条件（某个对象）
- 无法知道是否成功获取锁

### APIs

- lock()
- tryLock()
- unlock()
- lockInterruptibly()

### 可见性

#### happens-before

- 程序次序规则: 一个线程内, 按照代码顺序, 书写在前面的操作先行发生于书写在后面的操作(单线程)
- 锁定规则: 一个unlock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则: 对一个变量的写操作先行发生于后面对这个变量的读操作
- 传递规则: 如果操作A先行发生于操作B, 而操作B又先行发生于操作C, 则可以推导出操作A先行发生于操作C
- 线程启动规则: Thread对象的start()方法先行发生于此线程的每一个动作
- 线程终端原则: 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则: 线程中所有的操作都先行发生于线程的终止检测, 可以通过Thread.join()方法结束, Thread.isAlive()的返回值检测到线程已经终止执行
- 对象终结规则: 一个对象的初始化完成先行发生于finalize()方法的开始

### 读写锁

- 同时可以有多个线程读
- 现在有读线程在执行， 写线程需要等待
- 现有写线程在执行， 读写线程都需等待

### 读锁插队策略

- 公平锁不允许插队
- 非公平锁
  - 写锁可以随时插队
  - 读锁仅在等待队列头结点不是写线程的时候可以插队

### 自旋锁

一般用于多核服务器， 在并发度不是特别高的情况下， 比阻塞锁的效率高  
也适用于临界区比较短小的情况

### JVM 优化

1. 自旋锁和自适应
2. 锁消除
3. 锁粗化

---

1. 缩小同步代码块
2. 尽量不要锁住方法
3. 减少请求锁的次数
4. 避免人为制造“热点”
5. 锁中不要再包含锁
6. 选择合适的锁和工具类

## 发布对象

- 发布对象: 使一个对象能够被当前范围之外的代码所使用
- 对象逸出: 一种错误的发布. 当一个对象还没有构造完成时, 就被其他线程所见.

### 安全发布对象

1. 在静态初始化函数中初始化一个对象引用
2. 将对象的引用保存到volatile类型域或者AtomicReference对象中
3. 将对象的引用保存到某个正确构造对象的final类型域中
4. 将对象的引用保存到一个由锁保护的域中

## 原子类

### 原子类, 作用

- 不可分割
- 一个操作是不可中断的
- java.util.concurrent.atomic.*

---

- 粒度更细
- 效率更高(除高度竞争情况下)

---

把普通变量升级成具有原子功能的变量

- AtomicIntegerFieldUpdater对普通变量进行升级
- 使用场景: 偶尔需要一个原子get/set操作

## CAS

### CAS概念

```java
    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        if (oldValue == expectedValue) {
            value = newValue;
        }
        return oldValue;
    }
```

### CAS应用场景

- 乐观锁
- 并发容器
- 原子类
  - AtomicInteger加载Unsafe工具, 用来直接操作内存数据
  - 用Unsafe来实现底层操作
  - 用volatile修饰value字段, 保证可见性

```java
    private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");

    private volatile int value;
```

## 线程安全策略

### 不可变对象

- 对象创建以后其状态不能修改
- 对象所有域都是final类型
- 对象是正确创建的

---

- final关键字: 类, 方法, 变量
  - 修饰类: 不能被继承
  - 修饰方法: 1.锁定方法不能被继承类修改; 2.效率
  - 修饰变量: 基本数据类型变量, 引用类型变量

---

- Collections.unmodifiable*
- Guava: Immutable*

### 线程封闭

- Ad-hoc线程封闭: 程序控制实现, 最糟糕
- 堆栈封闭: 局部变量, 无并发问题
- ThreadLocal线程封闭

### 线程不安全类与写法

- StringBuilder(不安全) <--> StringBuffer(安全)
- SimpleDateFormat
- ArrayList, HashSet, HashMap

### 线程安全 - 同步容器

- ArrayList -> Vector, Stack
- HashMap -> Hashtable (key, value not null)
- Collections.synchronized*

### JUC

- ArrayList -> CopyOnWriteArrayList
- HashSet, TreeSet -> CopyOnWriteArraySet, ConcurrentSkipListSet
- HashMap, TreeMap -> ConcurrentHashMap, ConcurrentSkipListMap

## AQS

- 使用Node实现FIFO队列, 可以用于构建锁或者其他同步装置的基础框架
- 使用了int类型表示状态(state)(暗示重入的最大次数)
- 使用方法: 继承
- 子类通过继承并通过实现它的方法, 管理其状态(acquire和release)
- 可以同时实现排他锁和共享锁(Elusive和Shared)

---

- CountDownLatch
- Semaphore
- CylicBarrier
- ReentrantLock
- Condition
- FutureTask
