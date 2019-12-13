## 线程池基础

### 为什么要使用线程池？
* 1.减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。    
* 2.可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。  

在面向对象编程中，创建和销毁对象是很费时间的，因为创建一个对象要获取内存资源或者其它更多资源。在Java中更是如此，虚拟机将试图跟踪每一个对象，以便能够在对象销毁后进行垃圾回收。所以提高服务程序效率的一个手段就是尽可能减少创建和销毁对象的次数，特别是一些很耗资源的对象创建和销毁。如何利用已有对象来服务就是一个需要解决的关键问题，其实这就是一些"池化资源"技术产生的原因。

线程池的优势很明显，如下：

* 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；    
* 提高系统响应速度，当有任务到达时，无需等待新线程的创建便能立即执行；    
* 方便线程并发数的管控，线程若是无限制的创建，不仅会额外消耗大量系统资源，更是占用过多资源而阻塞系统或oom等状况，从而降低系统的稳定性。线程池能有效管控线程，统一分配、调优，提供资源使用率；    
* 更强大的功能，线程池提供了定时、定期以及可控线程数等功能的线程池，使用方便简单。

### 线程池创建方式
java.util.concurrent包中提供了多种线程池的创建方式，我们可以直接使用ThreadPoolExecutor类直接创建一个线程池，也可以使用Executors类创建，下面我们分别说一下这几种创建的方式。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在Executors类里面提供了一些静态工厂，生成一些常用的线程池。

在《阿里巴巴java开发手册》中指出了线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样一方面是线程的创建更加规范，可以合理控制开辟线程的数量；另一方面线程的细节管理交给线程池处理，优化了资源的开销。而线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool()、newSingleThreadExecutor()、newCachedThreadPool()等创建线程池的方法，但都有其局限性，不够灵活；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

1 阿里开发文档中，要求禁止使用Executors 直接创建线程池，是因为Executors 提供的方法在创建线程池时，workQueue ，corePoolSize ，maximumPoolSize 不可控，有可能导致OOM ,
2 因此建议使用ThreadPoolExecutor 创建线程池，根据业务场景选择合适的maximumPoolSize，
3 建议使用单例模式进行线程池的创建，保证一个服务中每一个线程池仅被初始化一次

```java
public class ThreadPoolManager {

    private static ThreadPoolExecutor instance;

    public static ThreadPoolExecutor getInstance() {
        if (instance == null) {
            synchronized (ThreadManager.class) {
                if (instance == null) {
                    int cpuNum = Runtime.getRuntime().availableProcessors();// 获取处理器数量
                    int threadNum = cpuNum * 2 + 1;// 根据cpu数量,计算出合理的线程并发数
                    int corePoolSize = cpuNum * 2;
                    int maximumPoolSize = corePoolSize + 1;
                    long keepAliveTime = 60l;
                    instance = new ThreadPoolExecutor(corePoolSize,maximumPoolSize,keepAliveTime,
                            TimeUnit.MILLISECONDS,new ArrayBlockingQueue<>(300),Executors.defaultThreadFactory(),
                            new ThreadPoolExecutor.AbortPolicy());
                }
            }
        }
        return instance;
    }
}
```

### 几个主要的类
Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。

比较重要的几个类：
| 接口/类                     | 说明                                                                                                           |
| --------------------------- | -------------------------------------------------------------------------------------------------------------- |
| ExecutorService             | 真正的线程池接口。                                                                                             |
| ScheduledExecutorService    | 继承自ExecutorService，和Timer/TimerTask类似，可安排在给定的延迟后运行或定期执行的命令。                                                       |
| Executors                   | 里面提供了一些静态工厂方法，生成一些常用的线程池，方法的返回类型是ExecutorService/ScheduledExecutorService接口 |
| ThreadPoolExecutor          | ExecutorService的默认实现。                                                                                    |
| ScheduledThreadPoolExecutor | 继承ThreadPoolExecutor的ScheduledExecutorService接口实现，可安排在给定的延迟后运行或定期执行的命令，此类优于Timer。                             |

#### Executor
```java
public interface Executor {
    void execute(Runnable command);
}
```
一个执行提交的runnable任务的对象。这个接口提供了一种解耦任务的提交和执行的方式。它替代了那种显式的创建线程执行的方式`new Thread(new(RunnableTask())).start()`。

* 线程池顶级接口。定义方法，void execute(Runnable)。方法是用于处理任务的一个服务方法。调用者提供Runnable接口的实现，线程池通过线程执行这个Runnable。服务方法无返回值的。是Runnable接口中的run方法无返回值。

* 常用方法 - void execute(Runnable)

* 作用是： 启动线程任务的。

#### ExecutorService
![ExecutorService](/assets/ExecutorService.png)

* Executor接口的子接口。提供了一个新的服务方法，submit。有返回值（Future类型）。submit方法提供了overload方法。其中有参数类型为Runnable的，不需要提供返回值的；有参数类型为Callable，可以提供线程执行后的返回值。

* Future，是submit方法的返回值。代表未来，也就是线程执行结束后的一种结果。如返回值。

* 常见方法 - void execute(Runnable)， Future submit(Callable)， Future submit(Runnable)

* 线程池状态： Running， ShuttingDown， Termitnaed  
Running - 线程池正在执行中。活动状态。  
ShuttingDown - 线程池正在关闭过程中。优雅关闭。一旦进入这个状态，线程池不再接收新的任务，处理所有已接收的任务，处理完毕后，关闭线程池。  
Terminated - 线程池已经关闭。  

#### Future
![concurrent_future](/assets/concurrent_future.png)
未来结果，代表线程任务执行结束后的结果。获取线程执行结果的方式是通过get方法获取的。get无参，阻塞等待线程执行结束，并得到结果。get有参，阻塞固定时长，等待线程执行结束后的结果，如果在阻塞时长范围内，线程未执行结束，抛出异常。

常用方法： T get() T get(long, TimeUnit)

#### Callable
```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```
Callable 和 Runnable 接口 的区别就是 Runnable的run方法无返回值，Callable的call方法有返回值。

#### Executors
Executor的工具类。类似Collection/Array和Collections/Arrays的关系。可以快速的提供若干种线程池。如：固定容量的，无限容量的，容量为1等各种线程池。

线程池是一个进程级的重量级资源。默认的生命周期和JVM一致。当开启线程池后，直到JVM关闭为止，是线程池的默认生命周期。如果手工调用shutdown方法，那么线程池执行所有的任务后，自动关闭。

* 开始 - 创建线程池。  
* 结束 - JVM关闭或调用shutdown并处理完所有的任务。  


**Executors提供的四种常见线程池**： 

* 1、newSingleThreadExecutor 

创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

* 2、newFixedThreadPool 

创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

* 3、newScheduledThreadPool 

创建一个可定期或者延时执行任务的定长线程池，支持定时及周期性任务执行。 

* 4、newCachedThreadPoo 

创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。 

#### ThreadPoolExecutor
线程池底层实现。除ForkJoinPool外，其他常用线程池底层都是使用ThreadPoolExecutor实现的。默认提供的线程池不满足条件时使用。如：初始线程数据4，最大线程数200，线程空闲周期30秒。

阿里开发文档中，要求禁止使用Executors 直接创建线程池，是因为Executors 提供的方法在创建线程池时不够灵活，workQueue ，corePoolSize ，maximumPoolSize 不可控，有可能导致OOM ,因此建议使用ThreadPoolExecutor 创建线程池，根据业务场景选择合适的maximumPoolSize， 建议使用单例模式进行线程池的创建，保证一个服务中每一个线程池仅被初始化一次。

由于其他常用线程池方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

### 线程池关闭
线程池关闭
调用线程池的shutdown()或shutdownNow()方法来关闭线程池。

* shutdown：按过去执行已提交任务的顺序发起一个有序的关闭，但是不接受新任务。如果已经关闭，则调用没有其他作用。  
* shutdownNow：尝试停止所有的活动执行任务、暂停等待任务的处理，并返回等待执行的任务列表。在从此方法返回的任务队列中排空（移除）这些任务。  

shutdownNow中断采用interrupt方法，所以无法响应中断的任务可能永远无法终止。但调用上述的两个关闭之一，isShutdown()方法返回值为true，当所有任务都已关闭，表示线程池关闭完成，则isTerminated()方法返回值为true。当需要立刻中断所有的线程，不一定需要执行完任务，可直接调用shutdownNow()方法。

### 线程池大小设置
如何合理地估算线程池大小，这个问题是比较复杂的，比较粗糙的估算方式：

* 如果是CPU密集型应用，则线程池大小设置为N+1  
* 如果是IO密集型应用，则线程池大小设置为2N+1  

但是根据我在实际应用场景的经验，这种估算有时并不准确，这里不展开讨论线程池大小的设置，可以看一下这一篇文章的分析：[如何合理地估算线程池大小](http://ifeve.com/how-to-calculate-threadpool-size/ "如何合理地估算线程池大小")

### 线程池的状态监控
利用线程池提供的参数进行监控，参数如下：

* getTaskCount：返回曾计划执行的近似任务总数。因为在计算期间任务和线程的状态可能动态改变，所以返回值只是一个近似值。    
* getCompletedTaskCount：返回已完成执行的近似任务总数。因为在计算期间任务和线程的状态可能动态改变，所以返回值只是一个近似值，但是该值在整个连续调用过程中不会减少。    
* getLargestPoolSize：线程池曾经创建过的最大线程数量，通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。    
* getPoolSize：线程池的线程数量。    
* getActiveCount：返回主动执行任务的近似线程数。

通过扩展线程池进行监控：继承线程池并重写线程池的beforeExecute()，afterExecute()和terminated()方法，可以在任务执行前、后和线程池关闭前自定义行为。如监控任务的平均执行时间，最大执行时间和最小执行时间等。

使用ThreadPoolExecutor直接创建线程池时，可以使用第三方的ThreadFactory，或者自己实现ThreadFactory接口，拓展更多的属性，例如设置线程名称、执行开始时间、优先级等等。





































