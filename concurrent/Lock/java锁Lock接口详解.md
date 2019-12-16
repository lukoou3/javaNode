## java锁Lock接口详解
```
https://www.cnblogs.com/dolphin0520/p/3923167.html
https://www.cnblogs.com/Java-Script/p/11090619.html
https://www.cnblogs.com/myseries/p/10784076.html
```

### synchronized的缺陷
如果一个代码块被synchronized关键字修饰，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待直至占有锁的线程释放锁。事实上，占有锁的线程释放锁一般会是以下三种情况之一：

* 1：占有锁的线程执行完了该代码块，然后释放对锁的占有；    
* 2：占有锁线程执行发生异常，此时JVM会让线程自动释放锁；    
* 3：占有锁线程进入 WAITING 状态从而释放锁，例如在该线程中调用wait()方法等。

试考虑以下几种情况：

* synchronized 方法或语句的使用提供了对与每个对象相关的隐式监视器锁的访问，但却强制所有锁获取和释放均要出现在一个块结构中：当获取了多个锁时，它们必须以相反的顺序释放，且必须在与所有锁被获取时相同的词法范围内释放所有锁。但有时也需要以更为灵活的方式使用锁。Lock 接口的实现允许锁在不同的作用范围内获取和释放，并允许以任何顺序获取和释放多个锁。    
* 在使用synchronized关键字的情形下，假如占有锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，那么其他线程就只能一直等待，别无他法。这会极大影响程序执行效率。因此，就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间 (解决方案：tryLock(long time, TimeUnit unit)) 或者 能够响应中断 (解决方案：lockInterruptibly())），这种情况可以通过 Lock 解决。    
* 我们知道，当多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作也会发生冲突现象，但是读操作和读操作不会发生冲突现象。但是如果采用synchronized关键字实现同步的话，就会导致一个问题，即当多个线程都只是进行读操作时，也只有一个线程在可以进行读操作，其他线程只能等待锁的释放而无法进行读操作。因此，需要一种机制来使得当多个线程都只是进行读操作时，线程之间不会发生冲突。同样地，Lock也可以解决这种情况 (解决方案：ReentrantReadWriteLock) 。    
* 我们可以通过Lock得知线程有没有成功获取到锁 (解决方案：ReentrantLock) ，但这个是synchronized无法办到的。

上面提到的几种情形，我们都可以通过Lock来解决，但 synchronized 关键字却无能为力。事实上，Lock 是 java.util.concurrent.locks包 下的接口，Lock 实现提供了比 synchronized 关键字 更广泛的锁操作，它能以更优雅的方式处理线程同步问题。

总结一下，也就是说Lock提供了比synchronized更多的功能。但是要注意以下几点：

* 1）Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；    
* 2）Lock和synchronized有一点非常大的不同，采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而**Lock则必须要用户去手动释放锁**，如果没有主动释放锁，就有可能导致出现死锁现象。

### java.util.concurrent.locks包下常用的类与接口（lock是jdk 1.5后新增的）
![885859-20190428143036791-1293316018](/assets/885859-20190428143036791-1293316018.png)
（1）Lock和ReadWriteLock是两大锁的根接口，Lock代表实现类是ReentrantLock（可重入锁），ReadWriteLock（读写锁）的代表实现类是ReentrantReadWriteLock。

Lock 接口支持那些语义不同（重入、公平等）的锁规则，可以在非阻塞式结构的上下文（包括 hand-over-hand 和锁重排算法）中使用这些规则。主要的实现是 ReentrantLock。

ReadWriteLock 接口以类似方式定义了一些读取者可以共享而写入者独占的锁。此包只提供了一个实现，即 ReentrantReadWriteLock，因为它适用于大部分的标准用法上下文。但程序员可以创建自己的、适用于非标准要求的实现。

（2）Condition 接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用 Object.wait 访问的隐式监视器类似，但提供了更强大的功能。Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用。 需要特别指出的是，单个 Lock 可能与多个 Condition 对象关联。Condition 方法的名称与对应的 Object 版本中的不同。

### Lock接口
Lock接口有6个方法：
```java
// 获取锁  
void lock()   

// 如果当前线程未被中断，则获取锁，可以响应中断  
void lockInterruptibly()   

// 返回绑定到此 Lock 实例的新 Condition 实例  
Condition newCondition()   

// 仅在调用时锁为空闲状态才获取该锁，可以响应中断  
boolean tryLock()   

// 如果锁在给定的等待时间内空闲，并且当前线程未被中断，则获取锁  
boolean tryLock(long time, TimeUnit unit)   

// 释放锁  
void unlock()
```
下面来逐个讲述Lock接口中每个方法的使用，lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()是用来获取锁的。unLock()方法是用来释放锁的。newCondition()这个方法暂且不在此讲述，介绍Condition的时候讲述。

在Lock中声明了四个方法来获取锁，那么这四个方法有何区别呢？

#### lock()
lock()方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。

在前面已经讲到，如果采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此，一般来说，使用Lock必须在try…catch…块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：
```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```

#### tryLock() & tryLock(long time, TimeUnit unit)
tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true；如果获取失败（即锁已被其他线程获取），则返回false，也就是说，这个方法无论如何都会立即返回（在拿不到锁时不会一直在那等待）。

tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false，同时可以响应中断。如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

一般情况下，通过tryLock来获取锁时是这样使用的：
```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){

     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

#### lockInterruptibly()　
lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程 正在等待获取锁，则这个线程能够 响应中断，即中断线程的等待状态。例如，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

由于lockInterruptibly()的声明中抛出了异常，所以lock.lockInterruptibly()必须放在try块中或者在调用lockInterruptibly()的方法外声明抛出 InterruptedException，但推荐使用后者。因此，lockInterruptibly()一般的使用形式如下：
```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```

注意，当一个线程获取了锁之后，是不会被interrupt()方法中断的。因为本身在前面的文章中讲过单独调用interrupt()方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。因此当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

### ReentrantLock
ReentrantLock，即 可重入锁。ReentrantLock是唯一实现了Lock接口的类，并且ReentrantLock提供了更多的方法。下面通过一些实例学习如何使用 ReentrantLock。

构造方法（不带参数 和带参数  true： 公平锁； false: 非公平锁）：
```java
public ReentrantLock() {
    //在ReentrantLock中定义了2个静态内部类，一个是NotFairSync，一个是FairSync，分别用来实现非公平锁和公平锁。
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

除了继承自Lock的几个方法，在ReentrantLock类中定义了很多方法，比如：
* isFair()        //判断锁是否是公平锁    
* isLocked()    //判断锁是否被任何线程获取了    
* isHeldByCurrentThread()   //判断锁是否被当前线程获取了    
* hasQueuedThreads()   //判断是否有线程在等待该锁


#### lock()的简单例子
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockThread {
	Lock lock = new ReentrantLock();//确保使用的是同一把锁

	public void lock() {
		// 获取锁
		lock.lock();
		try {
			System.out.println(Thread.currentThread() + " get the lock");
			Thread.sleep(500);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			// 释放锁
			lock.unlock();
			System.out.println(Thread.currentThread() + " release the lock");
		}
	}

	public static void main(String[] args) {
		LockThread lt = new LockThread();
		new Thread(() -> lt.lock()).start();
		new Thread(() -> lt.lock()).start();
	}
}
```
输出：
```
Thread[Thread-1,5,main] get the lock
Thread[Thread-1,5,main] release the lock
Thread[Thread-0,5,main] get the lock
Thread[Thread-0,5,main] release the lock
```

#### tryLock()的简单例子
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockThread01 {
	Lock lock = new ReentrantLock();//确保使用的是同一把锁

	public void lock() {
		if (lock.tryLock()) {
			try {
				System.out.println(Thread.currentThread() + "获取到锁");
				Thread.sleep(500);
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
				System.out.println(Thread.currentThread() + "释放了锁");
			}
		} else {
			System.out.println(Thread.currentThread() + "未获取到锁");
		}
	}

	public static void main(String[] args) {
		LockThread01 lt = new LockThread01();
		new Thread(() -> lt.lock()).start();
		new Thread(() -> lt.lock()).start();
	}
}
```
输出：
```
Thread[Thread-0,5,main]获取到锁
Thread[Thread-1,5,main]未获取到锁
Thread[Thread-0,5,main]释放了锁
```

#### tryLock(long, TimeUnit)的简单例子
```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockThread01 {
	Lock lock = new ReentrantLock();//确保使用的是同一把锁

	public void lock() throws InterruptedException {
		//如果锁在给定的等待时间内空闲，并且当前线程未被中断，则获取锁。在这里可以被打断。
		if (lock.tryLock(1, TimeUnit.SECONDS)) {
			try {
				System.out.println(Thread.currentThread() + "获取到锁");
				Thread.sleep(500);
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
				System.out.println(Thread.currentThread() + "释放了锁");
			}
		} else {
			System.out.println(Thread.currentThread() + "未获取到锁");
		}
	}

	public static void main(String[] args) {
		LockThread01 lt = new LockThread01();
		new Thread(() -> {
			try {
				lt.lock();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
		new Thread(() -> {
			try {
				lt.lock();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
	}
}
```
输出(这里阻塞等待了1秒，所以两个线程都获取到了锁)：
```
Thread[Thread-0,5,main]获取到锁
Thread[Thread-0,5,main]释放了锁
Thread[Thread-1,5,main]获取到锁
Thread[Thread-1,5,main]释放了锁
```


#### lockInterruptibly()的简单例子
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockThread02 {
	Lock lock = new ReentrantLock();//确保使用的是同一把锁

	public void lock() throws InterruptedException {
		System.out.println(Thread.currentThread() + "开始尝试获取锁");
		lock.lockInterruptibly();
		try {
			System.out.println(Thread.currentThread() + "获取到锁");
			Thread.sleep(500);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
			System.out.println(Thread.currentThread() + "释放了锁");
		}
	}

	public static void main(String[] args) throws InterruptedException {
		LockThread02 lt = new LockThread02();
		new Thread(() -> {
			try {
				lt.lock();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
		Thread.sleep(100);
		Thread thread = new Thread(() -> {
			try {
				lt.lock();
			} catch (InterruptedException e) {
				System.out.println(Thread.currentThread() + "抛出InterruptedException");
			}
		});
		thread.start();
		Thread.sleep(100);
		thread.interrupt();
	}
}
```
输出：
```
Thread[Thread-0,5,main]开始尝试获取锁
Thread[Thread-0,5,main]获取到锁
Thread[Thread-1,5,main]开始尝试获取锁
Thread[Thread-1,5,main]抛出InterruptedException
Thread[Thread-0,5,main]释放了锁
```

### ReadWriteLock 接口
ReadWriteLock 接口只有两个方法：
```java
//返回用于读取操作的锁  
Lock readLock()   
//返回用于写入操作的锁  
Lock writeLock() 
```
ReadWriteLock 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持，而写入锁是独占的。

读写锁有两个锁：一个是读操作相关的锁，也称为共享锁；一个是写操作相关的锁，也称为排他锁。多个读锁之间不互斥;读锁与写锁互斥;写锁与写锁互斥。

### ReentrantReadWriteLock
ReentrantReadWriteLock实现了ReadWriteLock接口。ReentrantReadWriteLock里面提供了很多丰富的方法，不过最主要的有两个方法：readLock()和writeLock()用来获取读锁和写锁。

读写锁有两个锁：一个是读操作相关的锁，也称为共享锁；一个是写操作相关的锁，也称为排他锁。多个读锁之间不互斥;读锁与写锁互斥;写锁与写锁互斥。

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockTest {
	private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

	public void read(){
		try {
			readWriteLock.readLock().lock();
			System.out.println("获得读锁"+Thread.currentThread().getName()+" timestamp:"+System.currentTimeMillis());
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}finally {
			readWriteLock.readLock().unlock();
		}
	}

	public void write(){
		try{
			readWriteLock.writeLock().lock();
			System.out.println("获得写锁"+Thread.currentThread().getName()+" timestamp:"+System.currentTimeMillis());
			Thread.sleep(1000);
		}catch (Exception e){
			e.printStackTrace();
		}finally {
			readWriteLock.writeLock().unlock();
		}
	}

	public static void main(String[] args){
		ReadWriteLockTest reentrantReadWriteLockTest = new ReadWriteLockTest();
		Runnable readRunnable = () -> reentrantReadWriteLockTest.read();
		Thread threadA = new Thread(readRunnable);
		threadA.setName("A");
		threadA.start();
		Thread threadB = new Thread(readRunnable);
		threadB.setName("B");
		threadB.start();
		Runnable writeRunnable = () -> reentrantReadWriteLockTest.write();
		Thread writeA = new Thread(writeRunnable);
		writeA.setName("writeA");
		writeA.start();
		Thread writeB = new Thread(writeRunnable);
		writeB.setName("writeB");
		writeB.start();
	}
}
```
输出：
```
获得读锁B timestamp:1576480758058
获得读锁A timestamp:1576480758058
获得写锁writeB timestamp:1576480759072
获得写锁writeA timestamp:1576480760086
```

### Condition
关键字 synchronized 与 wait() 和 notify() / notifyAll() 方法结合可以实现等待/通知模型， ReentrantLock 类也可以实现同样的功能，需要借助 Condition对象。

Condition 对象是JDK1.5中出现的技术，它有更好的灵活性，比如可实现多路通知功能，也就是在一个 Lock 对象里面可以创建多个 Condition (即对象监视器)实例，线程对象可以注册在指定的 Condition 中，从而可以有选择性地进行线程通知，在调度线程上更加灵活。因此通常来说比较推荐使用Condition，java的阻塞队列实际上是使用了Condition来模拟线程间协作。

使用 notify() / notifyAll() 方法进行通知时，被通知的线程由JVM随机选择。通过 ReentrantLock 结合 Condition 可以实现选择性通知。

synchronized 相当于整个 Lock 对象中只有一个单一的 Condition 对象，所有的线程都注册在它一个对象的身上。线程 notifyAll() 时，需要通知所有的 WAITING 线程，没有选择权。

* Condition是个接口，基本的方法就是await()和signal()方法；    
* Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition()    
* 调用Condition的await()和signal()方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用   

Condition与Object类相关方法类比：   
* Object 类中的 wait() 方法相当于 Condition 类中的 await() 方法。    
* Object 类中的 wait(long timeout) 方法相当于 Condition 类中的 awaitNanos(long nanosTimeout) 、await(long time,TimeUnit unit) 方法（awaitNanos的时间单位为毫微秒)。    
* Object 类中的 notify() 方法相当于 Condition 类中的 signal() 方法。    
* Object 类中的 notifyAll() 方法相当于 Condition 类中的 signalAll() 方法。

#### 用Condition实现的生产者消费者模式
练习（生产者消费者模式）：
自定义同步容器，容器容量上限为10。可以在多线程中应用，并保证数据线程安全。

##### Lock、Condition实现
```java
import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class TestContainer02<E> {

	private final LinkedList<E> list = new LinkedList<E>();
	private final int MAX = 10;
	private int count = 0;
	
	private Lock lock = new ReentrantLock();
	private Condition producer = lock.newCondition();
	private Condition consumer = lock.newCondition();
	
	public int getCount(){
		return count;
	}
	
	public void put(E e){
		lock.lock();
		try {
			while(list.size() == MAX){
				System.out.println(Thread.currentThread().getName() + " 等待。。。");
				// 进入等待队列。释放锁标记。
				// 借助条件，进入的等待队列。
				producer.await();
			}
			System.out.println(Thread.currentThread().getName() + " put 。。。");
			list.add(e);
			count++;
			// 借助条件，唤醒所有的消费者。
			consumer.signalAll();
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public E get(){
		E e = null;

		lock.lock();
		try {
			while(list.size() == 0){
				System.out.println(Thread.currentThread().getName() + " 等待。。。");
				// 借助条件，消费者进入等待队列
				consumer.await();
			}
			System.out.println(Thread.currentThread().getName() + " get 。。。");
			e = list.removeFirst();
			count--;
			// 借助条件，唤醒所有的生产者
			producer.signalAll();
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		} finally {
			lock.unlock();
		}
		
		return e;
	}
	
	public static void main(String[] args) {
		final TestContainer02<String> c = new TestContainer02<String>();
		for(int i = 0; i < 10; i++){
			new Thread(new Runnable() {
				@Override
				public void run() {
					for(int j = 0; j < 5; j++){
						System.out.println(c.get());
					}
				}
			}, "consumer"+i).start();
		}
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}
		for(int i = 0; i < 2; i++){
			new Thread(new Runnable() {
				@Override
				public void run() {
					for(int j = 0; j < 25; j++){
						c.put("container value " + j); 
					}
				}
			}, "producer"+i).start();
		}
	}
	
}
```

##### synchronized、wait/notify实现
```java
import java.util.LinkedList;
import java.util.concurrent.TimeUnit;

public class TestContainer01<E> {

	private final LinkedList<E> list = new LinkedList<E>();
	private final int MAX = 10;
	private int count = 0;
	
	public synchronized int getCount(){
		return count;
	}
	
	public synchronized void put(E e){
		while(list.size() == MAX){
			try {
				this.wait();
			} catch (InterruptedException e1) {
				e1.printStackTrace();
			}
		}
		
		list.add(e);
		count++;
		this.notifyAll();
	}
	
	public synchronized E get(){
		E e = null;
		while(list.size() == 0){
			try{
				this.wait();
			} catch (InterruptedException e1) {
				e1.printStackTrace();
			}
		}
		e = list.removeFirst();
		count--;
		this.notifyAll();
		return e;
	}
	
	public static void main(String[] args) {
		final TestContainer01<String> c = new TestContainer01<String>();
		for(int i = 0; i < 10; i++){
			new Thread(new Runnable() {
				@Override
				public void run() {
					for(int j = 0; j < 5; j++){
						System.out.println(c.get());
					}
				}
			}, "consumer"+i).start();
		}
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		for(int i = 0; i < 2; i++){
			new Thread(new Runnable() {
				@Override
				public void run() {
					for(int j = 0; j < 25; j++){
						c.put("container value " + j); 
					}
				}
			}, "producer"+i).start();
		}
	}
	
}
```










