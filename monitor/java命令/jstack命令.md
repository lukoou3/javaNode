## Java命令：Jstack
```
http://www.tianshouzhi.com/api/tutorials/jvm/351
https://www.hollischuang.com/archives/110
https://blog.csdn.net/cockroach02/article/details/82701458
```

jstack命令主要用于调试java程序运行过程中的线程堆栈信息，可以用于检测死锁，进程耗用cpu过高报警问题的排查。

jstack 主要用于生成虚拟机当前时刻的「线程快照」。线程快照是当前 Java 虚拟机每一条线程正在执行的方法堆栈的集合。生成线程快照的主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待。

### Jstack功能
jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

**So,jstack命令主要用来查看Java线程的调用堆栈的，可以用来分析线程问题（如死锁、cpu过高）**。

### jstack语法格式
其实只需要`jstack pid`就够了。

jstack语法格式：
```sh
[root@www wangxiaoxiao]# jstack

Usage:

    jstack [-l] <pid>

    jstack -F [-m] [-l] <pid>

Options:

    -F  强制dump线程堆栈信息. 用于进程hung住， jstack <pid>命令没有响应的情况

    -m  同时打印java和本地(native)线程栈信息，m是mixed mode的简写

    -l  打印锁的额外信息
```

### 线程dump信息说明
jstack命令会打印出所有的线程，包括用户自己启动的线程和jvm后台线程，我们主要关注的是用户线程，如
```sh
[root@www wangxiaoxiao]# jstack 15525

2017-02-14 21:10:02

Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.65-b01 mixed mode):



"elasticsearch[Native][merge][T#1]" #98 daemon prio=5 os_prio=0 tid=0x00007f031c009000 nid=0x4129 waiting on condition [0x00007f02f61ee000]

   java.lang.Thread.State: WAITING (parking)

    at sun.misc.Unsafe.park(Native Method)

    - parking to wait for  <0x00000000eea589f0> (a org.elasticsearch.common.util.concurrent.EsExecutors$ExecutorScalingQueue)

    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)

    at java.util.concurrent.LinkedTransferQueue.awaitMatch(LinkedTransferQueue.java:737)

    at java.util.concurrent.LinkedTransferQueue.xfer(LinkedTransferQueue.java:647)

    at java.util.concurrent.LinkedTransferQueue.take(LinkedTransferQueue.java:1269)

    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)

    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)

    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)

    at java.lang.Thread.run(Thread.java:745)

....
```

#### 线程dump信息说明
线程dump信息说明：

* **elasticsearch[Native][merge][T#1]** 是我们为线程起的名字    
* **daemon** 表示线程是否是守护线程    
* **prio** 表示我们为线程设置的优先级    
* **os_prio** 表示的对应的操作系统线程的优先级，由于并不是所有的操作系统都支持线程优先级，所以可能会出现都置为0的情况    
* **tid** 是java中为这个线程的id    
* **nid** 是这个线程对应的操作系统本地线程id，每一个java线程都有一个对应的操作系统线程，**我们查找cpu占用过高的查找的就是这个nid**    
* **wait on condition**表示当前线程处于等待状态，但是并没列出具体原因    
* **java.lang.Thread.State: WAITING (parking)** 也是表示的处于等待状态，括号中的内容说明了导致等待的原因，例如这里的parking说明是因为调用了 LockSupport.park方法导致等待

#### java.lang.Thread.State说明
java.lang.Thread.State说明：

一个Thread对象可以有多个状态，在`java.lang.Thread.State`中，总共定义六种状态：

**1、NEW**

线程刚刚被创建，也就是已经new过了，但是还没有调用start()方法，jstack命令不会列出处于此状态的线程信息

**2、RUNNABLE** <span style="color:#E30000;font-weight:normal">#java.lang.Thread.State: RUNNABLE</span>

RUNNABLE这个名字很具有欺骗性，很容易让人误以为处于这个状态的线程正在运行。事实上，这个状态只是表示，线程是可运行的。我们已经无数次提到过，一个单核CPU在同一时刻，只能运行一个线程。

**3、BLOCKED** <span style="color:#E30000;font-weight:normal"># java.lang.Thread.State: BLOCKED (on object monitor)</span>

线程处于阻塞状态，正在等待一个monitor lock。通常情况下，是因为本线程与其他线程公用了一个锁。其他在线程正在使用这个锁进入某个synchronized同步方法块或者方法，而本线程进入这个同步代码块也需要这个锁，最终导致本线程处于阻塞状态。

**4、WAITING**

等待状态，调用以下方法可能会导致一个线程处于等待状态：

* `Object.wait` 不指定超时时间 <span style="color:#E30000;font-weight:normal"># java.lang.Thread.State: WAITING (on object monitor)</span>    
* `Thread.join` with no timeout    
* `LockSupport.park` <span style="color:#E30000;font-weight:normal">#java.lang.Thread.State: WAITING (parking)</span>  

例如：对于wait()方法，一个线程处于等待状态，通常是在等待其他线程完成某个操作。本线程调用某个对象的wait()方法，其他线程处于完成之后，调用同一个对象的notify或者notifyAll()方法。Object.wait()方法只能够在同步代码块中调用。调用了wait()方法后，会释放锁。

**5、TIMED_WAITING**

线程等待指定的时间，对于以下方法的调用，可能会导致线程处于这个状态：

* `Thread.sleep` <span style="color:#E30000;font-weight:normal">#java.lang.Thread.State: TIMED_WAITING (sleeping)</span>    
* `Object.wait` 指定超时时间 <span style="color:#E30000;font-weight:normal">#java.lang.Thread.State: TIMED_WAITING (on object monitor)</span>    
* `Thread.join` with timeout    
* `LockSupport.parkNanos` <span style="color:#E30000;font-weight:normal">#java.lang.Thread.State: TIMED_WAITING (parking)</span>    
* `LockSupport.parkUntil` <span style="color:#E30000;font-weight:normal">#java.lang.Thread.State: TIMED_WAITING (parking)</span>

**6、TERMINATED**

线程终止。

说明，对于 java.lang.Thread.State: WAITING (on object monitor)和java.lang.Thread.State: TIMED_WAITING (on object monitor)，对于这两个状态，是因为调用了Object的wait方法(前者没有指定超时，后者指定了超时)，由于wait方法肯定要在syncronized代码中编写，因此肯定是如类似以下代码导致：
```java
synchronized(obj) {
    .........
    obj.wait();
    .........
}
```
因此通常的堆栈信息中，必然后一个lock标记，如下(locked <0x00000000eca75aa8>)：
```
"CM1" #21 daemon prio=5 os_prio=0 tid=0x00007f02f0d6d800 nid=0x3d48 in Object.wait() [0x00007f02fefef000]

   java.lang.Thread.State: WAITING (on object monitor)

        at java.lang.Object.wait(Native Method)

        at java.lang.Object.wait(Object.java:502)

        at java.util.TimerThread.mainLoop(Timer.java:526)

        - locked <0x00000000eca75aa8> (a java.util.TaskQueue)

        at java.util.TimerThread.run(Timer.java:505)
```

#### 调用修饰
表示线程在方法调用时,额外的重要的操作。线程Dump分析的重要信息。修饰上方的方法调用。

```
locked <地址> 目标：使用synchronized申请对象锁成功,监视器的拥有者。

waiting to lock <地址> 目标：使用synchronized申请对象锁未成功,在迚入区等待。

waiting on <地址> 目标：使用synchronized申请对象锁成功后,释放锁幵在等待区等待。

parking to wait for <地址> 目标
```

**locked**
```
at oracle.jdbc.driver.PhysicalConnection.prepareStatement
- locked <0x00002aab63bf7f58> (a oracle.jdbc.driver.T4CConnection)
at oracle.jdbc.driver.PhysicalConnection.prepareStatement
- locked <0x00002aab63bf7f58> (a oracle.jdbc.driver.T4CConnection)
at com.jiuqi.dna.core.internal.db.datasource.PooledConnection.prepareStatement
```
通过synchronized关键字,成功获取到了对象的锁,成为监视器的拥有者,在临界区内操作。对象锁是可以线程重入的。

**waiting to lock**
```
at com.jiuqi.dna.core.impl.CacheHolder.isVisibleIn(CacheHolder.java:165)
- waiting to lock <0x0000000097ba9aa8> (a CacheHolder)
at com.jiuqi.dna.core.impl.CacheGroup$Index.findHolder
at com.jiuqi.dna.core.impl.ContextImpl.find
at com.jiuqi.dna.bap.basedata.common.util.BaseDataCenter.findInfo
```
通过synchronized关键字,没有获取到了对象的锁,线程在监视器的进入区等待。在调用栈顶出现,线程状态为Blocked。

**waiting on**
```
at java.lang.Object.wait(Native Method)
- waiting on <0x00000000da2defb0> (a WorkingThread)
at com.jiuqi.dna.core.impl.WorkingManager.getWorkToDo
- locked <0x00000000da2defb0> (a WorkingThread)
at com.jiuqi.dna.core.impl.WorkingThread.run
```
通过synchronized关键字,成功获取到了对象的锁后,调用了wait方法,进入对象的等待区等待。在调用栈顶出现,线程状态为WAITING或TIMED_WATING。

**parking to wait for**

park是基本的线程阻塞原语,不通过监视器在对象上阻塞。随concurrent包会出现的新的机制,不synchronized体系不同。

#### 线程动作
线程状态产生的原因
```
runnable:状态一般为RUNNABLE。

in Object.wait():等待区等待,状态为WAITING或TIMED_WAITING。

waiting for monitor entry:进入区等待,状态为BLOCKED。

waiting on condition:等待区等待、被park。

sleeping:休眠的线程,调用了Thread.sleep()。
```

Wait on condition 该状态出现在线程等待某个条件的发生。具体是什么原因，可以结合 stacktrace来分析。 最常见的情况就是线程处于sleep状态，等待被唤醒。 常见的情况还有等待网络IO：在java引入nio之前，对于每个网络连接，都有一个对应的线程来处理网络的读写操作，即使没有可读写的数据，线程仍然阻塞在读写操作上，这样有可能造成资源浪费，而且给操作系统的线程调度也带来压力。在 NewIO里采用了新的机制，编写的服务器程序的性能和可扩展性都得到提高。 正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几 乎消耗了所有的带宽，仍然有大量数据等待网络读 写；另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。所以要结合系统的一些性能观察工具来综合分析，比如 netstat统计单位时间的发送包的数目，如果很明显超过了所在网络带宽的限制 ; 观察 cpu的利用率，如果系统态的 CPU时间，相对于用户态的 CPU时间比例较高；如果程序运行在 Solaris 10平台上，可以用 dtrace工具看系统调用的情况，如果观察到 read/write的系统调用的次数或者运行时间遥遥领先；这些都指向由于网络带宽所限导致的网络瓶颈。

#### 关于死锁
在 JAVA 5中加强了对死锁的检测。线程 Dump中可以直接报告出 Java级别的死锁，如下所示：
```
Found one Java-level deadlock:

=============================

"Thread-1":

waiting to lock monitor 0x0003f334 (object 0x22c19f18, a java.lang.Object),

which is held by "Thread-0"

"Thread-0":

waiting to lock monitor 0x0003f314 (object 0x22c19f20, a java.lang.Object),

which is held by "Thread-1"
```

#### 关于Lock类
在 JDK 5.0中，引入了 Lock机制，从而使开发者能更灵活的开发高性能的并发多线程程序，可以替代以往 JDK中的 synchronized和 Monitor的 机制。但是，要注意的是，因为 Lock类只是一个普通类， JVM无从得知 Lock对象的占用情况，所以在线程 DUMP中，也不会包含关于 Lock的信息， 关于死锁等问题，就不如用 synchronized的编程方式容易识别。

#### 关于nid
每个线程都有一个tid 和nid，tid是java中这个线程的编号，而nid(native id)是对应操作系统线程id。有的时候，我们会收到报警，说服务器，某个进程占用CPU过高，肯定是因为某个java线程有耗CPU资源的方法。

我们在linux中通过`top -Hp`命令，在Windows中通过 Process Explorer 来找出某个进程占用cpu最高的线程的id，这个id就对应我们这里的nid，线程dump信息中的nid是16进制的，我们通过转换就能快速的查找到这个线程的堆栈信息。

### jstack例子:死锁分析
jstack命令导出的线程 Dump中可以直接报告出 Java级别的死锁。但是，要注意的是，因为 Lock类只是一个普通类， JVM无从得知 Lock对象的占用情况，所以在线程 DUMP中，也不会包含关于 Lock的信息， 关于死锁等问题，就不如用 synchronized的编程方式容易识别。

看一段死锁的程序：
```java
package javaCommand;
/**
 * @author hollis
 */
public class JStackDemo {
    public static void main(String[] args) {
        Thread t1 = new Thread(new DeadLockclass(true));//建立一个线程
        Thread t2 = new Thread(new DeadLockclass(false));//建立另一个线程
        t1.start();//启动一个线程
        t2.start();//启动另一个线程
    }
}
class DeadLockclass implements Runnable {
    public boolean falg;// 控制线程
    DeadLockclass(boolean falg) {
        this.falg = falg;
    }
    public void run() {
        /**
         * 如果falg的值为true则调用t1线程
         */
        if (falg) {
            while (true) {
                synchronized (Suo.o1) {
                    System.out.println("o1 " + Thread.currentThread().getName());
                    synchronized (Suo.o2) {
                        System.out.println("o2 " + Thread.currentThread().getName());
                    }
                }
            }
        }
        /**
         * 如果falg的值为false则调用t2线程
         */
        else {
            while (true) {
                synchronized (Suo.o2) {
                    System.out.println("o2 " + Thread.currentThread().getName());
                    synchronized (Suo.o1) {
                        System.out.println("o1 " + Thread.currentThread().getName());
                    }
                }
            }
        }
    }
}
 
class Suo {
    static Object o1 = new Object();
    static Object o2 = new Object();
}
```
当我启动该程序时，我们看一下控制台：
```
o1 Thread-0
o2 Thread-1
```
我们发现，程序只输出了两行内容，然后程序就不再打印其它的东西了，但是程序并没有停止。这样就产生了死锁。 当线程1使用synchronized锁住了o1的同时，线程2也是用synchronized锁住了o2。当两个线程都执行完第一个打印任务的时候，线程1想锁住o2，线程2想锁住o1。但是，线程1当前锁着o1，线程2锁着o2。所以两个想成都无法继续执行下去，就造成了死锁。

然后，我们使用jstack来看一下线程堆栈信息：
```r
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f0134003ae8 (object 0x00000007d6aa2c98, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f0134006168 (object 0x00000007d6aa2ca8, a java.lang.Object),
  which is held by "Thread-1"
 
Java stack information for the threads listed above:
===================================================
"Thread-1":
    at javaCommand.DeadLockclass.run(JStackDemo.java:40)
    - waiting to lock <0x00000007d6aa2c98> (a java.lang.Object)
    - locked <0x00000007d6aa2ca8> (a java.lang.Object)
    at java.lang.Thread.run(Thread.java:745)
"Thread-0":
    at javaCommand.DeadLockclass.run(JStackDemo.java:27)
    - waiting to lock <0x00000007d6aa2ca8> (a java.lang.Object)
    - locked <0x00000007d6aa2c98> (a java.lang.Object)
    at java.lang.Thread.run(Thread.java:745)
 
Found 1 deadlock.
```
堆栈写的很明显，它告诉我们 Found one Java-level deadlock，然后指出造成死锁的两个线程的内容。然后，又通过 Java stack information for the threads listed above来显示更详细的死锁的信息。 他说
```
Thread-1在想要执行第40行的时候，当前锁住了资源<0x00000007d6aa2ca8>,但是他在等待资源<0x00000007d6aa2c98> Thread-0在想要执行第27行的时候，当前锁住了资源<0x00000007d6aa2c98>,但是他在等待资源<0x00000007d6aa2ca8> 由于这两个线程都持有资源，并且都需要对方的资源，所以造成了死锁。 原因我们找到了，就可以具体问题具体分析，解决这个死锁了。
```

### jstack例子:找出cpu最高的线程
* 找到CPU使用最高的进程的pid

* 在linux中通过`top -Hp [pid]`命令，在Windows中通过 Process Explorer 来找出这个个进程占用cpu使用率最高的线程的id，这个id就对应线程dump中的nid

* 使用`jstack [pid] > pid.txt`命令将线程dump信息写入到文件中方便查询

* 将在操作系统中找到的线程id转为16进制，linux中用`printf "%x\n" [id]`，Windows中可以用计算器

* 然后查找线程dump文件中的nid，就找到了对应的线程，可以看到当时线程的堆栈信息

#### windows下找出java程序占用cpu很高的线程并找到问题代码 
`https://blog.csdn.net/hexin373/article/details/8846919/`

jvisualvm 和 jconsole貌似都只能看到总共占用的cpu 看不到每个线程分别占用的cpu呢

所以在windows平台上要找出到底是哪个线程占用的cpu还不那么容易，linux用top就简单多了

解决方法:

<span style="color:#CC6666;font-weight:bold">1.找到java进程对应的pid。</span>

找pid的方法是:打开任务管理器，然后点击 "查看" 菜单，然后点击 "选择列"，把pid勾上，然后就可以在任务管理器里面看到所有进程的pid值了。(也可以用第三步中提到的工具直接查看)

<span style="color:#CC6666;font-weight:bold">2.然后把java进程导出线程快照。直接运行命令。</span>

```
jstack 31372 > c:/31372.stack
```

<span style="color:#CC6666;font-weight:bold">3.在windows下只能查看进程的cpu占用率，要查看线程的cpu占用率要借助其他的工具，我这里用的是微软提供的 Process Explorer </span>

下载地址http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx

下载完后解压运行，找到对应的进程，直接双击或者右键点击需要查看的进程点击properties

<span style="color:#CC6666;font-weight:bold">4.然后选择 Threads 选项卡，找到占用cpu的线程的nid</span>

<span style="color:#CC6666;font-weight:bold">5.把nid转换成16进制，我这里直接用系统自带的计算器转换，置于为什么要转换，是因为先前用jstack导出的信息里面线程对应的tid是16进制的。</span>

最后得到的线程nid的16进制的值为 7C84

<span style="color:#CC6666;font-weight:bold">6.在 c盘的31372.stack文件中查找 7C84</span>


<span style="color:#CC6666;font-weight:bold">7.根据线程的堆栈信息找出问题代码</span>

#### linux下找出java程序占用cpu很高的线程
使用`top -Hp [pid]`命令找出进程中占用cpu最高的线程nid。

用`printf "%x\n" [id]`命令可以将10进制转成16进制。

在vi编辑器中使用`/string` 命令查找字符串。n 命令重复上一条检索命令。N 命令重复上一条检索命令，但检索方向改变。


