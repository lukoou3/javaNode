## java线程池ThreadPoolExecutor类使用详解
`https://www.cnblogs.com/dafanjoy/p/9729358.html`

在《阿里巴巴java开发手册》中指出了线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样一方面是线程的创建更加规范，可以合理控制开辟线程的数量；另一方面线程的细节管理交给线程池处理，优化了资源的开销。而线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool()、newSingleThreadExecutor()、newCachedThreadPool()等创建线程池的方法，但都有其局限性，不够灵活；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

下面我们就对ThreadPoolExecutor的使用方法进行一个详细的概述。

### ThreadPoolExecutor的构造函数
最常用构造函数各参数含义：
```java
public ThreadPoolExecutor
(int corePoolSize, // 核心容量，创建线程池的时候，默认有多少线程。也是线程池保持的最少线程数
int maximumPoolSize, // 最大容量，线程池最多有多少线程
long keepAliveTime, // 生命周期，0为永久。当线程空闲多久后，自动回收。
TimeUnit unit, // 生命周期单位，为生命周期提供单位，如：秒，毫秒
BlockingQueue<Runnable> workQueue // 任务队列，阻塞队列。注意，泛型必须是Runnable
);
```

ThreadPoolExecutor的又一个构造函数(多一个参数)：
```java
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

构造函数的参数含义如下：

* corePoolSize:核心线程数,线程池保留线程的数量,即使这些线程是空闲.除非设置了allowCoreThreadTimeOut ；    
* maximumPoolSize:指定了线程池中的最大线程数量，这个参数会根据你使用的workQueue任务队列的类型，决定线程池会开辟的最大线程数量；    
* keepAliveTime:当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；    
* unit:keepAliveTime的单位    
* workQueue:任务队列，执行前用于保持任务的队列，此队列仅保持由 execute 方法提交的 Runnable 任务，它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种；；    
* threadFactory:线程工厂，用于创建线程，一般用默认即可，默认为DefaultThreadFactory类；    
* handler:拒绝策略，由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序，默认策略为ThreadPoolExecutor.AbortPolicy；

### ThreadPoolExecutor构造函数各个参数详细解释
* **corePoolSize**（线程池基本大小）：当向线程池提交一个任务时，若线程池已创建的线程数小于corePoolSize，即便此时存在空闲线程，也会通过创建一个新线程来执行该任务，直到已创建的线程数大于或等于corePoolSize时，才会根据是否存在空闲线程，来决定是否需要创建新的线程。除了利用提交新任务来创建和启动线程（按需构造），也可以通过 prestartCoreThread() 或 prestartAllCoreThreads() 方法来提前启动线程池中的基本线程。    
* **maximumPoolSize**（线程池最大大小）：线程池所允许的最大线程个数。当队列满了，且已创建的线程数小于maximumPoolSize，则线程池会创建新的线程来执行任务。另外，对于无界队列，可忽略该参数。    
* **keepAliveTime**（线程存活保持时间）：默认情况下，当线程池的线程个数多于corePoolSize时，线程的空闲时间超过keepAliveTime则会终止。但只要keepAliveTime大于0，allowCoreThreadTimeOut(boolean) 方法也可将此超时策略应用于核心线程。另外，也可以使用setKeepAliveTime()动态地更改参数。    
* **unit**（存活时间的单位）：时间单位，分为7类，从细到粗顺序：NANOSECONDS（纳秒），MICROSECONDS（微妙），MILLISECONDS（毫秒），SECONDS（秒），MINUTES（分），HOURS（小时），DAYS（天）；    
* **workQueue**（任务队列）：用于传输和保存等待执行任务的阻塞队列。可以使用此队列与线程池进行交互：  
1、如果运行的线程数少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。  
2、如果运行的线程数等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。  
3、如果无法将请求加入队列，则创建新的线程，除非创建此线程超出maximumPoolSize，在这种情况下，任务将被拒绝。    
* **threadFactory**（线程工厂）：用于创建新线程。由同一个threadFactory创建的线程，属于同一个ThreadGroup，创建的线程优先级都为Thread.NORM_PRIORITY，以及是非守护进程状态。threadFactory创建的线程也是采用new Thread()方式，threadFactory创建的线程名都具有统一的风格：pool-m-thread-n（m为线程池的编号，n为线程池内的线程编号）；    
* **handler**（线程饱和策略）：当线程池和队列都满了，则表明该线程池已达饱和状态。  
1、ThreadPoolExecutor.AbortPolicy：处理程序遭到拒绝，则直接抛出运行时异常RejectedExecutionException。(默认策略)ThreadPoolExecutor.AbortPolicy：处理程序遭到拒绝，则直接抛出运行时异常RejectedExecutionException。(默认策略)    
2、ThreadPoolExecutor.CallerRunsPolicy：调用者所在线程来运行该任务，此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。ThreadPoolExecutor.CallerRunsPolicy：调用者所在线程来运行该任务，此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。    
3、ThreadPoolExecutor.DiscardPolicy：无法执行的任务将被删除。ThreadPoolExecutor.DiscardPolicy：无法执行的任务将被删除。    
4、ThreadPoolExecutor.DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重新尝试执行任务（如果再次失败，则重复此过程）。ThreadPoolExecutor.DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重新尝试执行任务（如果再次失败，则重复此过程）。   

### 线程池参数选择
1 **corePoolSize** 业务分为cpu密集型和io密集型，一般web应用都是io密集型，cpu密集型如数学计算，视频编解码等

java 可以使用int cpuNumber = Runtime.getRuntime().availableProcessors() 获取部署机器的cpu核心数(Intel cpu获取的是xianchengshu)线程数

cpu密集型业务corePoolSize = cpuNumber io密集型业务corePoolSize = cpuNumber*2 ,

2 **maximumPoolSize** 根据业务场景选择 一般 maximumPoolSize = corePoolSize +1

在业务存在偶尔短时间需要处理大量业务时，可以适当放大maximumPoolSize 的值，在短时间内增加并发数量处理业务

3 **keepAliveTime** 非核心线程空闲时间，根据自己业务，如果maximumPoolSize = corePoolSize 应该设置为0，

如果需要maximumPoolSize >corePoolSize 则需要根据实际业务场景，选择非核心线程的时间，根据newCachedThreadPool的构造函数看，可以设置为60s，在短时间处理洪峰业务处理时，减少线程创建带来的性能消耗，同时在高并发业务处理完成后能及时释放线程资源

4 **unit** 时间单位，配合keepAliveTime使用

5 **workQueue** 的选择 ArrayBlockingQueue和LinkedBlockingQueue 的区别请见另一篇博客[ArrayBlockingQueue 和LinkedBlockingQueue 代码解析(JDK8)](https://yq.aliyun.com/articles/712761?spm=a2c4e.11153940.0.0.4e814af39iLD24 "ArrayBlockingQueue 和LinkedBlockingQueue 代码解析(JDK8)")

6 **threadFactory** 创建线程池线程的工厂，也就是申请线程资源的工厂，执行线程(执行任务)的创建和这个无关，如果对线程资源申请没有特殊需求，可以使用默认的Executors.defaultThreadFactory()，如果想查看线程池什么时候申请了线程资源，比如非核心线程的创建，可以实现ThreadFactory 并重写newThread方法，可以记录线程资源申请时间和线程信息，在非核心线程被回收后，再次申请线程资源时，也是通过threadFactory进行重新创建线程
```java
public class TThreadFactory implements ThreadFactory {

    @Override
    public Thread newThread(Runnable r) {
        System.out.println(new Date());
        Thread t= new Thread(r,"new Thread ");
        return t;
    }
}
```

7 **handler** 饱和策略 也就是当线程池没有没有空余线程，线程等待队列满，并且线程池线程数量等于最大线程数，执行饱和策略
饱和策略分为4种

（1）AbortPolicy 直接抛出异常，并抛弃该线程任务，不影响主线程和正在执行和等待的线程

可以使用ThreadPoolExecutor中的AbortPolicy进行创建
```java
new ThreadPoolExecutor.AbortPolicy() {// 队列已满,而且当前线程数已经超过最大线程数时的异常处理策略
  @Override
  public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    super.rejectedExecution(r, e);
    }
  }
```
 
（2）CallerRunsPolicy 在线程池执行饱和策略时，由主线程执行子线程任务，也就是说，主线程execute(）或者submit()被调用后，主线程执行子线程任务，主线程任务挂起等待子线程任务结束后继续执行

ThreadPoolExecutor中的CallerRunsPolicy 进行创建

（3） DiscardPolicy 线程池饱和后，抛弃所有新增的线程

ThreadPoolExecutor中的DiscardPolicy 进行创建

（4）DiscardOldestPolicy 抛弃queue中最老的线程任务，将新任务添加到queue的底部，抛弃掉的任务将不会执行

ThreadPoolExecutor中的DiscardOldestPolicy 进行创建

### 线程池新增线程流程
keepAliveTime和maximumPoolSize及BlockingQueue的类型均有关系。如果BlockingQueue是无界的，那么永远不会触发maximumPoolSize，自然keepAliveTime也就没有了意义。

反之，如果核心数较小，有界BlockingQueue数值又较小，同时keepAliveTime又设的很小，如果任务频繁，那么系统就会频繁的申请回收线程。
![64bee149f5f8878df8aaf265f30b577c44cd6479](/assets/64bee149f5f8878df8aaf265f30b577c44cd6479.png)

### workQueue任务队列
用于传输和保存等待执行任务的阻塞队列。可以使用此队列与线程池进行交互：   
* 1、如果运行的线程数少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。    
* 2、如果运行的线程数等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。    
* 3、如果无法将请求加入队列，则创建新的线程，除非创建此线程超出maximumPoolSize，在这种情况下，任务将被拒绝。

上面我们已经介绍过了，它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列；

#### 直接提交队列
可以使用SynchronousQueue队列，SynchronousQueue是一个特殊的BlockingQueue，它没有容量，没执行一个插入操作就会阻塞，需要再执行一个删除操作才会被唤醒，反之每一个删除操作也都要等待对应的插入操作。

```java
//maximumPoolSize设置为2 ，拒绝策略为AbortPolic策略，直接抛出异常
new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```

从下面的例子可以看到，当任务队列为SynchronousQueue，创建的线程数大于maximumPoolSize时，直接执行了拒绝策略抛出异常。

使用SynchronousQueue队列，提交的任务不会被保存，总是会马上提交执行。如果用于执行任务的线程数量小于maximumPoolSize，则尝试创建新的进程，如果达到maximumPoolSize设置的最大值，则根据你设置的handler执行拒绝策略。因此这种方式你提交的任务不会被缓存起来，而是会被马上执行，在这种情况下，你需要对你程序的并发量有个准确的评估，才能设置合适的maximumPoolSize数量，否则很容易就会执行拒绝策略；

```java
public static void main( String[] args ) {
    //maximumPoolSize设置为2 ，拒绝策略为AbortPolic策略，直接抛出异常
    ExecutorService threadPool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
    for(int i=0;i<3;i++) {
        threadPool.execute(new Runnable() {				
            @Override
            public void run() {
                try{
                    Thread.sleep(1000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName());
            }
        });
    }   
}
```
输出
```java
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task concurrent.t08.Test_10$1@466bcf5d rejected from java.util.concurrent.ThreadPoolExecutor@697a9f24[Running, pool size = 2, active threads = 2, queued tasks = 0, completed tasks = 0]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2048)
	at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:821)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1372)
	at concurrent.t08.Test_10.main(Test_10.java:15)
pool-1-thread-1
pool-1-thread-2
```

#### 直接提交队列
有界的任务队列可以使用ArrayBlockingQueue实现，如下所示
```java
pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(10),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```

使用ArrayBlockingQueue有界任务队列，若有新的任务需要执行时，线程池会创建新的线程，直到创建的线程数量达到corePoolSize时，则会将新的任务加入到等待队列中。若等待队列已满，即超过ArrayBlockingQueue初始化的容量，则继续创建线程，直到线程数量达到maximumPoolSize设置的最大线程数量，若大于maximumPoolSize，则执行拒绝策略。在这种情况下，线程数量的上限与有界任务队列的状态有直接关系，如果有界队列初始容量较大或者没有达到超负荷的状态，线程数将一直维持在corePoolSize以下，反之当任务队列已满时，则会以maximumPoolSize为最大线程数上限。

#### 无界的任务队列
无界的任务队列可以使用LinkedBlockingQueue实现，如下所示
```java
pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```

使用无界任务队列，线程池的任务队列可以无限制的添加新的任务，而线程池创建的最大线程数量就是你corePoolSize设置的数量，也就是说在这种情况下maximumPoolSize这个参数是无效的，哪怕你的任务队列中缓存了很多未执行的任务，当线程池的线程数达到corePoolSize后，就不会再增加了；若后续有新的任务加入，则直接进入队列等待，当使用这种任务队列模式时，一定要注意你任务提交与处理之间的协调与控制，不然会出现队列中的任务由于无法及时处理导致一直增长，直到最后资源耗尽的问题。

#### 优先任务队列(属于无界的任务队列)
优先任务队列通过PriorityBlockingQueue实现，下面我们通过一个例子演示下
```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //优先任务队列
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new PriorityBlockingQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
          
        for(int i=0;i<20;i++) {
            pool.execute(new ThreadTask(i));
        }    
    }
}

public class ThreadTask implements Runnable,Comparable<ThreadTask>{
    
    private int priority;
    
    public int getPriority() {
        return priority;
    }

    public void setPriority(int priority) {
        this.priority = priority;
    }

    public ThreadTask() {
        
    }
    
    public ThreadTask(int priority) {
        this.priority = priority;
    }

    //当前对象和其他对象做比较，当前优先级大就返回-1，优先级小就返回1,值越小优先级越高
    public int compareTo(ThreadTask o) {
         return  this.priority>o.priority?-1:1;
    }
    
    public void run() {
        try {
            //让线程阻塞，使后续任务进入缓存队列
            Thread.sleep(1000);
            System.out.println("priority:"+this.priority+",ThreadName:"+Thread.currentThread().getName());
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    
    }
}
```
我们来看下执行的结果情况
```java
priority:0,ThreadName:pool-1-thread-1
priority:9,ThreadName:pool-1-thread-1
priority:8,ThreadName:pool-1-thread-1
priority:7,ThreadName:pool-1-thread-1
priority:6,ThreadName:pool-1-thread-1
priority:5,ThreadName:pool-1-thread-1
priority:4,ThreadName:pool-1-thread-1
priority:3,ThreadName:pool-1-thread-1
priority:2,ThreadName:pool-1-thread-1
priority:1,ThreadName:pool-1-thread-1
```
大家可以看到除了第一个任务直接创建线程执行外，其他的任务都被放入了优先任务队列，按优先级进行了重新排列执行，且线程池的线程数一直为corePoolSize，也就是只有一个。

通过运行的代码我们可以看出PriorityBlockingQueue它其实是一个特殊的无界队列，它其中无论添加了多少个任务，线程池创建的线程数也不会超过corePoolSize的数量，只不过其他队列一般是按照先进先出的规则处理任务，而PriorityBlockingQueue队列可以自定义规则根据任务的优先级顺序先后执行。

#### 工作队列对比
BlockingQueue的插入/移除/检查这些方法，对于不能立即满足但可能在将来某一时刻可以满足的操作，共有4种不同的处理方式：第一种是抛出一个异常，第二种是返回一个特殊值（null 或false，具体取决于操作），第三种是在操作可以成功前，无限期地阻塞当前线程，第四种是在放弃前只在给定的最大时间限制内阻塞。如下表格：

| 操作 | 抛出异常  | 特殊值   | 阻塞   | 超时                 |
| ---- | --------- | -------- | ------ | -------------------- |
| 插入 | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
| 移除 | remove()  | poll()   | take() | poll(time, unit)     |
| 检查 | element() | peek()   | 不可用 | 不可用               |

**实现BlockingQueue接口的常见类如下**：   
* ArrayBlockingQueue：基于数组的有界阻塞队列。队列按FIFO原则对元素进行排序，队列头部是在队列中存活时间最长的元素，队尾则是存在时间最短的元素。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。 这是一个典型的“有界缓存区”，固定大小的数组在其中保持生产者插入的元素和使用者提取的元素。一旦创建了这样的缓存区，就不能再增加其容量。试图向已满队列中放入元素会导致操作受阻塞；试图从空队列中提取元素将导致类似阻塞。ArrayBlockingQueue构造方法可通过设置fairness参数来选择是否采用公平策略，公平性通常会降低吞吐量，但也减少了可变性和避免了“不平衡性”，可根据情况来决策。    
* LinkedBlockingQueue：基于链表的无界阻塞队列。与ArrayBlockingQueue一样采用FIFO原则对元素进行排序。基于链表的队列吞吐量通常要高于基于数组的队列。    
* SynchronousQueue：同步的阻塞队列。其中每个插入操作必须等待另一个线程的对应移除操作，等待过程一直处于阻塞状态，同理，每一个移除操作必须等到另一个线程的对应插入操作。SynchronousQueue没有任何容量。不能在同步队列上进行 peek，因为仅在试图要移除元素时，该元素才存在；除非另一个线程试图移除某个元素，否则也不能（使用任何方法）插入元素；也不能迭代队列，因为其中没有元素可用于迭代。Executors.newCachedThreadPool使用了该队列。    
* PriorityBlockingQueue：基于优先级的无界阻塞队列。优先级队列的元素按照其自然顺序进行排序，或者根据构造队列时提供的 Comparator 进行排序，具体取决于所使用的构造方法。优先级队列不允许使用 null 元素。依靠自然顺序的优先级队列还不允许插入不可比较的对象（这样做可能导致 ClassCastException）。虽然此队列逻辑上是无界的，但是资源被耗尽时试图执行 add 操作也将失败（导致 OutOfMemoryError）。

### 拒绝策略
一般我们创建线程池时，为防止资源被耗尽，任务队列都会选择创建有界任务队列，但种模式下如果出现任务队列已满且线程池创建的线程数达到你设置的最大线程数时，这时就需要你指定ThreadPoolExecutor的RejectedExecutionHandler参数即合理的拒绝策略，来处理线程池"超载"的情况。ThreadPoolExecutor自带的拒绝策略如下：

* 1、AbortPolicy策略：该策略会直接抛出异常，阻止系统正常工作；    
* 2、CallerRunsPolicy策略：如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行；    
* 3、DiscardOledestPolicy策略：该策略会丢弃任务队列中最老的一个任务，也就是当前任务队列中最先被添加进去的，马上要被执行的那个任务，并尝试再次提交；    
* 4、DiscardPolicy策略：该策略会默默丢弃无法处理的任务，不予任何处理。当然使用此策略，业务场景中需允许任务的丢失；  

以上内置的策略均实现了RejectedExecutionHandler接口，当然你也可以自己扩展RejectedExecutionHandler接口，定义自己的拒绝策略，我们看下示例代码：可以看到由于任务加了休眠阻塞，执行需要花费一定时间，导致会有一定的任务被丢弃，从而执行自定义的拒绝策略；
```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //自定义拒绝策略
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5),
                Executors.defaultThreadFactory(), new RejectedExecutionHandler() {
            public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                System.out.println(r.toString()+"执行了拒绝策略");
                
            }
        });
          
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadTask());
        }    
    }
}

public class ThreadTask implements Runnable{    
    public void run() {
        try {
            //让线程阻塞，使后续任务进入缓存队列
            Thread.sleep(1000);
            System.out.println("ThreadName:"+Thread.currentThread().getName());
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    
    }
}
```
输出结果：
```
com.hhxx.test.ThreadTask@33909752执行了拒绝策略
com.hhxx.test.ThreadTask@55f96302执行了拒绝策略
com.hhxx.test.ThreadTask@3d4eac69执行了拒绝策略
ThreadName:pool-1-thread-2
ThreadName:pool-1-thread-1
ThreadName:pool-1-thread-1
ThreadName:pool-1-thread-2
ThreadName:pool-1-thread-1
ThreadName:pool-1-thread-2
ThreadName:pool-1-thread-1
```
可以看到由于任务加了休眠阻塞，执行需要花费一定时间，导致会有一定的任务被丢弃，从而执行自定义的拒绝策略；

### ThreadFactory自定义线程创建
线程池中线程就是通过ThreadPoolExecutor中的ThreadFactory，线程工厂创建的。那么通过自定义ThreadFactory，可以按需要对线程池中创建的线程进行一些特殊的设置，如命名、优先级等，下面代码我们通过ThreadFactory对线程池中创建的线程进行记录与命名：可以看到线程池中，每个线程的创建我们都进行了记录输出与命名。
```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //自定义线程工厂
        pool = new ThreadPoolExecutor(2, 4, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5),
                new ThreadFactory() {
            public Thread newThread(Runnable r) {
                System.out.println("线程"+r.hashCode()+"创建");
                //线程命名
                Thread th = new Thread(r,"threadPool"+r.hashCode());
                return th;
            }
        }, new ThreadPoolExecutor.CallerRunsPolicy());
          
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadTask());
        }    
    }
}

public class ThreadTask implements Runnable{    
    public void run() {
        //输出执行线程的名称
        System.out.println("ThreadName:"+Thread.currentThread().getName());
    }
}
```
输出结果：
```
线程118352462创建
线程1550089733创建
线程865113938创建
ThreadName:threadPool1550089733
ThreadName:threadPool118352462
线程1442407170创建
ThreadName:threadPool1550089733
ThreadName:threadPool1550089733
ThreadName:threadPool1550089733
ThreadName:threadPool865113938
ThreadName:threadPool865113938
ThreadName:threadPool118352462
ThreadName:threadPool1550089733
ThreadName:threadPool1442407170
```
可以看到线程池中，每个线程的创建我们都进行了记录输出与命名。

### ThreadPoolExecutor扩展

ThreadPoolExecutor扩展主要是围绕beforeExecute()、afterExecute()和terminated()三个接口实现的，

* 1、beforeExecute：线程池中任务运行前执行    
* 2、afterExecute：线程池中任务运行完毕后执行    
* 3、terminated：线程池退出后执行  

通过这三个接口我们可以监控每个任务的开始和结束时间，或者其他一些功能。下面我们可以通过代码实现一下:可以看到通过对beforeExecute()、afterExecute()和terminated()的实现，我们对线程池中线程的运行状态进行了监控，在其执行前后输出了相关打印信息。另外使用shutdown方法可以比较安全的关闭线程池， 当线程池调用该方法后，线程池中不再接受后续添加的任务。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。

```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args ) throws InterruptedException
    {
        //实现自定义接口
        pool = new ThreadPoolExecutor(2, 4, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5),
                new ThreadFactory() {
            public Thread newThread(Runnable r) {
                System.out.println("线程"+r.hashCode()+"创建");
                //线程命名
                Thread th = new Thread(r,"threadPool"+r.hashCode());
                return th;
            }
        }, new ThreadPoolExecutor.CallerRunsPolicy()) {
    
            protected void beforeExecute(Thread t,Runnable r) {
                System.out.println("准备执行："+ ((ThreadTask)r).getTaskName());
            }
            
            protected void afterExecute(Runnable r,Throwable t) {
                System.out.println("执行完毕："+((ThreadTask)r).getTaskName());
            }
            
            protected void terminated() {
                System.out.println("线程池退出");
            }
        };
          
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadTask("Task"+i));
        }    
        pool.shutdown();
    }
}

public class ThreadTask implements Runnable{    
    private String taskName;
    public String getTaskName() {
        return taskName;
    }
    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }
    public ThreadTask(String name) {
        this.setTaskName(name);
    }
    public void run() {
        //输出执行线程的名称
        System.out.println("TaskName"+this.getTaskName()+"---ThreadName:"+Thread.currentThread().getName());
    }
}
```
输出结果：
```
线程118352462创建
线程1550089733创建
准备执行：Task0
准备执行：Task1
TaskNameTask0---ThreadName:threadPool118352462
线程865113938创建
执行完毕：Task0
TaskNameTask1---ThreadName:threadPool1550089733
执行完毕：Task1
准备执行：Task3
TaskNameTask3---ThreadName:threadPool1550089733
执行完毕：Task3
准备执行：Task2
准备执行：Task4
TaskNameTask4---ThreadName:threadPool1550089733
执行完毕：Task4
准备执行：Task5
TaskNameTask5---ThreadName:threadPool1550089733
执行完毕：Task5
准备执行：Task6
TaskNameTask6---ThreadName:threadPool1550089733
执行完毕：Task6
准备执行：Task8
TaskNameTask8---ThreadName:threadPool1550089733
执行完毕：Task8
准备执行：Task9
TaskNameTask9---ThreadName:threadPool1550089733
准备执行：Task7
执行完毕：Task9
TaskNameTask2---ThreadName:threadPool118352462
TaskNameTask7---ThreadName:threadPool865113938
执行完毕：Task7
执行完毕：Task2
线程池退出
```

可以看到通过对beforeExecute()、afterExecute()和terminated()的实现，我们对线程池中线程的运行状态进行了监控，在其执行前后输出了相关打印信息。另外使用shutdown方法可以比较安全的关闭线程池， 当线程池调用该方法后，线程池中不再接受后续添加的任务。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。









