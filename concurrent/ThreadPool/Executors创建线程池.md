## Executors创建线程池
Executors类是java.util.concurrent提供的一个创建线程池的工厂类，使用该类可以方便的创建线程池，此类提供的几种方法，支持创建四种类型的线程池，分别是：newCachedThreadPool、newFixedThreadPool、newScheduledThreadPool、newSingleThreadExecutor。

四种类型的线程池简介：

* **newFixedThreadPool**   
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

* **newCachedThreadPoo**   
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

* **newScheduledThreadPool**   
创建一个可定期或者延时执行任务的定长线程池，支持定时及周期性任务执行。 

* **newSingleThreadExecutor**   
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

### 四种线程池对比
| 线程池方法              | 初始化线程池数 | 最大线程池数      | 线程池中线程存活时间 | 时间单位 | 工作队列            |
| ----------------------- | -------------- | ----------------- | -------------------- | -------- | ------------------- |
| newCachedThreadPool     | 0              | Integer.MAX_VALUE | 60                   | 秒       | SynchronousQueue    |
| newFixedThreadPool      | 入参指定大小   | 入参指定大小      | 0                    | 毫秒     | LinkedBlockingQueue |
| newScheduledThreadPool  | 入参指定大小   | Integer.MAX_VALUE | 0                    | 微秒     | DelayedWorkQueue    |
| newSingleThreadExecutor | 1              | 1                 | 0                    | 毫秒     | LinkedBlockingQueue |

### 线程池新增线程流程
keepAliveTime和maximumPoolSize及BlockingQueue的类型均有关系。如果BlockingQueue是无界的，那么永远不会触发maximumPoolSize，自然keepAliveTime也就没有了意义。

反之，如果核心数较小，有界BlockingQueue数值又较小，同时keepAliveTime又设的很小，如果任务频繁，那么系统就会频繁的申请回收线程。
![64bee149f5f8878df8aaf265f30b577c44cd6479](/assets/64bee149f5f8878df8aaf265f30b577c44cd6479.png)

### newFixedThreadPool
创建一个可重用的固定线程数量的线程池，以共享的无界队列方式来运行这些线程。
```java
ExecutorService threadPool = Executors.newFixedThreadPool(3);    // 创建可以容纳3个线程的线程池
```

创建一个固定大小的线程池。每次提交一个任务就会创建一个线程，直到创建的线程数量达到线程池的最大大小nThreads。线程池的大小一旦达到最大值就会保持不变。如果在所有线程都处于活动状态时，这时再有其他任务提交，他们将等待队列中直到有空闲的线程可用。如果任何线程由于执行过程中的故障而终止，将会有一个新线程将取代这个线程执行后续任务。

* 底层：返回ThreadPoolExecutor实例，接收参数为所设定线程数量nThread，corePoolSize为nThread，maximumPoolSize为nThread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：new LinkedBlockingQueue<Runnable>() 无界阻塞队列  
* 通俗：创建可容纳固定数量线程的池子，每个线程的存活时间是无限的，当池子满了就不在添加线程了；如果池中的所有线程均在繁忙状态，对于新任务会进入阻塞队列中(无界的阻塞队列)  
* 适用：执行长期的任务，性能好很多。  
* 使用场景： 大多数情况下，使用的线程池，首选推荐FixedThreadPool。OS系统和硬件是有线程支持上限。不能随意的无限制提供线程池。   

#### 一个例子
```java
/**
 * 线程池
 * 固定容量线程池
 * FixedThreadPool - 固定容量线程池。创建线程池的时候，容量固定。
 *  构造的时候，提供线程池最大容量
 * new xxxxx -> 
 * ExecutorService - 线程池服务类型。所有的线程池类型都实现这个接口。
 *  实现这个接口，代表可以提供线程池能力。
 *  shutdown - 优雅关闭。 不是强行关闭线程池，回收线程池中的资源。而是不再处理新的任务，将已接收的任务处理完毕后
 *      再关闭。
 * Executors - Executor的工具类。类似Collection和Collections的关系。
 *  可以更简单的创建若干种线程池。
 */
package concurrent.t08;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class Test_02_FixedThreadPool {
	
	public static void main(String[] args) {
		ExecutorService service = Executors.newFixedThreadPool(5);
		for(int i = 0; i < 6; i++){
			service.execute(new Runnable() {
				@Override
				public void run() {
					try {
						TimeUnit.MILLISECONDS.sleep(500);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(Thread.currentThread().getName() + " - test executor");
				}
			});
		}
		
		System.out.println(service);
		
		service.shutdown();
		// 是否已经结束， 相当于回收了资源。
		System.out.println(service.isTerminated());
		// 是否已经关闭， 是否调用过shutdown方法
		System.out.println(service.isShutdown());
		System.out.println(service);
		
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		// service.shutdown();
		System.out.println(service.isTerminated());
		System.out.println(service.isShutdown());
		System.out.println(service);
	}

}
```
输出
```
java.util.concurrent.ThreadPoolExecutor@1c980484[Running, pool size = 5, active threads = 5, queued tasks = 1, completed tasks = 0]
false
true
java.util.concurrent.ThreadPoolExecutor@1c980484[Shutting down, pool size = 5, active threads = 5, queued tasks = 1, completed tasks = 0]
pool-1-thread-3 - test executor
pool-1-thread-1 - test executor
pool-1-thread-4 - test executor
pool-1-thread-2 - test executor
pool-1-thread-5 - test executor
pool-1-thread-3 - test executor
true
true
java.util.concurrent.ThreadPoolExecutor@1c980484[Terminated, pool size = 0, active threads = 0, queued tasks = 0, completed tasks = 6]
```

#### 一个使用Future的例子
```java
/**
 * 线程池
 * 固定容量线程池
 */
package concurrent.t08;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

public class Test_03_Future {
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		/*FutureTask<String> task = new FutureTask<>(new Callable<String>() {
			@Override
			public String call() throws Exception {
				return "first future task";
			}
		});
		
		new Thread(task).start();
		
		System.out.println(task.get());*/
		
		ExecutorService service = Executors.newFixedThreadPool(1);
		
		Future<String> future = service.submit(new Callable<String>() {
			@Override
			public String call() {
				try {
					TimeUnit.MILLISECONDS.sleep(500);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("aaa");
				return Thread.currentThread().getName() + " - test executor";
			}
		});
		System.out.println(future);
		System.out.println(future.isDone()); // 查看线程是否结束， 任务是否完成。 call方法是否执行结束
		
		System.out.println(future.get()); // 获取call方法的返回值。会阻塞
		System.out.println(future.isDone());
	}

}
```
输出
```
java.util.concurrent.FutureTask@5166b0df
false
aaa
pool-1-thread-1 - test executor
true
```

#### 使用固定容量线程池实现简单的并行计算
```java
/**
 * 线程池
 * 固定容量线程池, 简单应用
 */
package concurrent.t08;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class Test_04_ParallelComputingWithFixedThreadPool {
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		long start = System.currentTimeMillis();
		computing(1, 2000000);
		long end = System.currentTimeMillis();
		System.out.println("computing times : " + (end - start));
		
		ExecutorService service = Executors.newFixedThreadPool(5);
		
		ComputingTask t1 = new ComputingTask(1, 60000);
		ComputingTask t2 = new ComputingTask(60001, 110000);
		ComputingTask t3 = new ComputingTask(110001, 150000);
		ComputingTask t4 = new ComputingTask(150001, 180000);
		ComputingTask t5 = new ComputingTask(180001, 200000);
		
		Future<List<Integer>> f1 = service.submit(t1);
		Future<List<Integer>> f2 = service.submit(t2);
		Future<List<Integer>> f3 = service.submit(t3);
		Future<List<Integer>> f4 = service.submit(t4);
		Future<List<Integer>> f5 = service.submit(t5);
		
		start = System.currentTimeMillis();
		f1.get();
		f2.get();
		f3.get();
		f4.get();
		f5.get();
		end = System.currentTimeMillis();
		System.out.println("parallel computing times : " + (end - start));
		
	}
	
	static class ComputingTask implements Callable<List<Integer>>{
		int start, end;
		public ComputingTask(int start, int end){
			this.start = start;
			this.end = end;
		}
		public List<Integer> call() throws Exception{
			List<Integer> results = new ArrayList<Integer>();
			boolean isPrime = true;
			for(int i = start; i <= end; i++){
				for(int j = 1; j < Math.sqrt(i); j++){
					if(i % j == 0){
						isPrime = false;
						break;
					}
				}
				if(isPrime){
					results.add(i);
				}
				isPrime = true;
			}
			
			return results;
		}
	}
	
	private static List<Integer> computing(Integer start, Integer end){
		List<Integer> results = new ArrayList<Integer>();
		boolean isPrime = true;
		for(int i = start; i <= end; i++){
			for(int j = 1; j < Math.sqrt(i); j++){
				if(i % j == 0){
					isPrime = false;
					break;
				}
			}
			if(isPrime){
				results.add(i);
			}
			isPrime = true;
		}
		
		return results;
	}

}
```
输出(4和cpu)：
```
computing times : 31
parallel computing times : 16
```

### newCachedThreadPool
创建一个可缓存的无界线程池，该方法无参数。当线程池中的线程空闲时间超过60s则会自动回收该线程，当任务超过线程池的线程数则创建新线程。线程池的大小上限为Integer.MAX_VALUE，可看做是无限大。
```java
ExecutorService threadPool = Executors.newCachedThreadPool();    // 线程池的大小会根据执行的任务数量动态分配  
```

创建线程本身需要很多资源，包括内存，记录线程状态，以及控制阻塞等等。因此，相比另外两种线程池，在需要频繁创建短期异步线程的场景下，newCachedThreadPool能够复用已完成而未关闭的线程来提高程序性能。通俗讲就是：CachedThreadPool会创建一个缓存区，将初始化的线程缓存起来，如果线程有可用的，就使用之前创建好的线程，如果没有可用的，就会新创建线程。另外，终止并且从缓存中移除已有60秒未被使用的线程。

* 底层：返回ThreadPoolExecutor实例，corePoolSize为0；maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue(同步队列)  
* 通俗：当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，因此会在池中寻找可用线程来执行，若有可以线程则执行，若没有可用线程则创建一个线程来执行该任务；若池中线程空闲时间超过指定大小，则该线程会被销毁。  
* 适用：执行很多短期异步的小程序或者负载较轻的服务器  
* 应用场景： 内部应用或测试应用。 内部应用，有条件的内部数据瞬间处理时应用，如：电信平台夜间执行数据整理（有把握在短时间内处理完所有工作，且对硬件和软件有足够的信心）。 测试应用，在测试的时候，尝试得到硬件或软件的最高负载量，用于提供FixedThreadPool容量的指导。  

#### 简单的例子
```java
/**
 * 线程池
 * 无容量限制的线程池（最大容量默认为Integer.MAX_VALUE）
 */
package concurrent.t08;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class Test_05_CachedThreadPool {
	
	public static void main(String[] args) {
		ExecutorService service = Executors.newCachedThreadPool();
		System.out.println(service);
		
		for(int i = 0; i < 5; i++){
			service.execute(new Runnable() {
				@Override
				public void run() {
					try {
						TimeUnit.MILLISECONDS.sleep(500);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(Thread.currentThread().getName() + " - test executor");
				}
			});
		}
		
		System.out.println(service);
		
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		System.out.println(service);
	}

}
```
输出
```
java.util.concurrent.ThreadPoolExecutor@1955e0bc[Running, pool size = 0, active threads = 0, queued tasks = 0, completed tasks = 0]
java.util.concurrent.ThreadPoolExecutor@1955e0bc[Running, pool size = 5, active threads = 5, queued tasks = 0, completed tasks = 0]
pool-1-thread-4 - test executor
pool-1-thread-2 - test executor
pool-1-thread-5 - test executor
pool-1-thread-1 - test executor
pool-1-thread-3 - test executor
java.util.concurrent.ThreadPoolExecutor@1955e0bc[Running, pool size = 5, active threads = 0, queued tasks = 0, completed tasks = 5]
```

#### 简单的例子(使用lambda表达式)
```java
/**
     * 1.创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程<br>
     * 2.当任务数增加时，此线程池又可以智能的添加新线程来处理任务<br>
     * 3.此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小<br>
     * 
     */
    public static void cacheThreadPool() {
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        for (int i = 1; i <= 10; i++) {
            final int ii = i;
            try {
                Thread.sleep(ii * 1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            cachedThreadPool.execute(()->out.println("线程名称：" + Thread.currentThread().getName() + "，执行" + ii));
        }

    }
```
输出
```
线程名称：pool-1-thread-1，执行1
线程名称：pool-1-thread-1，执行2
线程名称：pool-1-thread-1，执行3
线程名称：pool-1-thread-1，执行4
线程名称：pool-1-thread-1，执行5
线程名称：pool-1-thread-1，执行6
线程名称：pool-1-thread-1，执行7
线程名称：pool-1-thread-1，执行8
线程名称：pool-1-thread-1，执行9
线程名称：pool-1-thread-1，执行10
```

### newScheduledThreadPool
创建一个可安排在给定延迟时间后 运行的或者可定期地执行的线程池。
```java
ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(3);    // 效果类似于Timer定时器   
```

* 底层：创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列  
* 通俗：创建一个有初始大小的线程池，线程池内线程存活时间无限制，线程池可以支持定时及周期性任务执行，如果所有线程均处于繁忙状态，对于新任务会进入DelayedWorkQueue队列中，这是一种按照超时时间排序的队列结构  
* 适用：周期性执行任务的场景  
* 使用场景： 计划任务时选用（DelaydQueue），如：电信行业中的数据整理，每分钟整理，每小时整理，每天整理等。  

参数：  
* command - 要执行的任务    
* initialDelay - 首次执行的延迟时间    
* period - 连续执行之间的周期    
* unit - initialDelay 和 period 参数的时间单位

该线程池提供了多个方法：

* schedule(Runnable command, long delay, TimeUnit unit)，延迟一定时间后执行Runnable任务；    
* schedule(Callable callable, long delay, TimeUnit unit)，延迟一定时间后执行Callable任务；    
* scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)，延迟一定时间后，以间隔period时间的频率周期性地执行任务；    
* scheduleWithFixedDelay(Runnable command, long initialDelay, long delay,TimeUnit unit)，与scheduleAtFixedRate()方法很类似，但是不同的是scheduleWithFixedDelay()方法的周期时间间隔是以上一个任务执行结束到下一个任务开始执行的间隔，而scheduleAtFixedRate()方法的周期时间间隔是以上一个任务开始执行到下一个任务开始执行的间隔，也就是这一些任务系列的触发时间都是可预知的。

#### 例子
```java
/**
     * 创建一个定长线程池，支持定时及周期性任务执行。延迟执行
     */
    public static void sceduleThreadPool() {
        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
        Runnable r1 = () -> out.println("线程名称：" + Thread.currentThread().getName() + "，执行:3秒后执行");
        scheduledThreadPool.schedule(r1, 3, TimeUnit.SECONDS);
        Runnable r2 = () -> out.println("线程名称：" + Thread.currentThread().getName() + "，执行:延迟2秒后每3秒执行一次");
        scheduledThreadPool.scheduleAtFixedRate(r2, 2, 3, TimeUnit.SECONDS);
        Runnable r3 = () -> out.println("线程名称：" + Thread.currentThread().getName() + "，执行:普通任务");
        for (int i = 0; i < 5; i++) {
            scheduledThreadPool.execute(r3);
        }
    }
```
output：
```
线程名称：pool-1-thread-1，执行:普通任务
线程名称：pool-1-thread-5，执行:普通任务
线程名称：pool-1-thread-4，执行:普通任务
线程名称：pool-1-thread-3，执行:普通任务
线程名称：pool-1-thread-2，执行:普通任务
线程名称：pool-1-thread-1，执行:延迟2秒后每3秒执行一次
线程名称：pool-1-thread-5，执行:3秒后执行
线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
```

### newSingleThreadExecutor
创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。
```java
ExecutorService threadPool = Executors.newSingleThreadExecutor();    // 创建只含有单个线程的线程池，如果当前线程在执行任务时突然中断，则会创建一个新的线程去替代它从而继续执行任务。
```
创建一个单线程的线程池。这个线程池中只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么就会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

* 底层：FinalizableDelegatedExecutorService包装的ThreadPoolExecutor实例，corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：new LinkedBlockingQueue<Runnable>() 无解阻塞队列    
* 通俗：创建只有一个线程的线程池，且线程的存活时间是无限的；当该线程正繁忙时，对于新任务会进入阻塞队列中(无界的阻塞队列)    
* 适用：一个任务一个任务执行的场景  
* 使用场景： 保证任务顺序时使用。如： 游戏大厅中的公共频道聊天。秒杀。

#### 例子
```java
/** 
     *创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行
     */
    public static void singleTheadPoolTest() {
        ExecutorService pool = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            final int ii = i;
            pool.execute(() -> out.println(Thread.currentThread().getName() + "=>" + ii));
        }
    }

```
-----output-------
```
 线程名称：pool-1-thread-1，执行0
 线程名称：pool-1-thread-1，执行1
 线程名称：pool-1-thread-1，执行2
 线程名称：pool-1-thread-1，执行3
 线程名称：pool-1-thread-1，执行4
```

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






