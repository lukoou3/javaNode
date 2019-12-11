## java 中断线程的几种方式 interrupt()
`https://www.cnblogs.com/myseries/p/10918819.html`
总结一下：

在 Java 中有以下 3 种方法可以终止正在运行的线程：    
* 使用退出标识(在循环结构中)，使线程正常退出，也就是当 run() 方法完成后线程终止。
* 使用 interrupt() 方法中断线程。
* 使用 stop() 方法强行终止线程，但是不推荐使用这个方法，因为 stop() 和 suspend() 及 resume() 一样，都是作废过期的方法，使用它们可能产生不可预料的结果。

interrupt、isInterrupted、interrupted：   
* 调用 interrupt() 方法仅仅是在当前线程中打了一个停止的标记，并不是真的停止线程。如果线程被Object.wait, Thread.join和Thread.sleep等方法之一阻塞，被标记中断的线程会抛出InterruptedException异常，并清除线程的中断状态。  
* interrupted() 方法是Thread的静态方法，测试当前线程是否已经中断。线程的中断状态 由该方法清除。换句话说，如果连续两次调用该方法，则第二次调用将返回 false。  
* isInterrupted() 方法是Thread的实例方法，测试线程是否已经中断。线程的中断状态 不受该方法的影响。  

java的线程中断，在有阻塞的情况下，可以通过interrupt方法实现。在线程没有阻塞的情况下，是通过标识位实现的，依赖于程序员自己检查。
下面代码中的t1线程将永远不会被中断。程序永远不会退出。
```java
public class ThreadInterruptTest {
    static long i = 0;
    public static void main(String[] args) {
        System.out.println("begin");
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                    i++;
                    System.out.println(String.valueOf(i));
                }
            }
        });
        t1.start();
        t1.interrupt();
    }
}
```

### 中断
中断（Interrupt）一个线程意味着在该线程完成任务之前停止其正在进行的一切，有效地中止其当前的操作。线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序。虽然初次看来它可能显得简单，但是，你必须进行一些预警以实现期望的结果。你最好还是牢记以下的几点告诫。

首先，忘掉Thread.stop方法。虽然它确实停止了一个正在运行的线程，然而，这种方法是不安全也是不受提倡的，这意味着，在未来的JAVA版本中，它将不复存在。


### 如何安全的结束一个正在运行的线程
java.lang.Thread类包含了一些常用的方法，如：start(), stop(), stop(Throwable) ,suspend(), destroy() ,resume()。通过这些方法，我们可以对线程进行方便的操作，但是这些方法中，只有start()方法得到了保留。

在JDK帮助文档以及Sun公司的一篇文章《Why are Thread.stop, Thread.suspend and Thread.resume Deprecated?》中都讲解了舍弃这些方法的原因。

简单来说是因为：使用stop方法虽然可以强行终止正在运行或挂起的线程，但使用stop方法是很危险的，就象突然关闭计算机电源，而不是按正常程序关机一样，可能会产生不可预料的结果，因此，并不推荐使用stop方法来终止线程。

那么，我们究竟**应该如何停止线程呢**？  
* 1、任务中一般都会有循环结构，只要用一个标记控制住循环，就可以结束任务。  
* 2、如果线程处于了冻结状态，无法读取标记，此时可以使用interrupt()方法将线程从冻结状态强制恢复到运行状态中来，让线程具备CPU的执行资格。  

### 使用退出标志停止线程
当run方法执行完后，线程就会退出。但有时run方法是永远不会结束的，如在服务端程序中使用线程进行监听客户端请求，或是其他的需要循环处理的任务。

在这种情况下，一般是将这些任务放在一个循环中，如while循环。如果想使while循环在某一特定条件下退出，最直接的方法就是设一个boolean类型的标志，并通过设置这个标志为true或false来控制while循环是否退出。

```java
public class test1 {
    public static volatile boolean exit =false;  //退出标志
    
    public static void main(String[] args) {
        new Thread() {
            public void run() {
                System.out.println("线程启动了");
                while (!exit) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("线程结束了");
            }
        }.start();
        
        try {
            Thread.sleep(1000 * 5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        exit = true;//5秒后更改退出标志的值,没有这段代码，线程就一直不能停止
    }
}
```

### 使用interrupt方法停止线程
Thread.interrupt()方法: 作用是中断线程。将会设置该线程的中断状态位，即设置为true，中断的结果线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序本身。线程会不时地检测这个中断标示位，以判断线程是否应该被中断（中断标示值是否为true）。它并不像stop方法那样会中断一个正在运行的线程。

**interrupt()方法只是改变中断状态，不会中断一个正在运行的线程**。需要用户自己去监视线程的状态为并做处理。**支持线程中断的方法（也就是线程中断后会抛出interruptedException的方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常**。这一方法实际完成的是，给受阻塞的线程发出一个中断信号，这样受阻线程检查到中断标识，就得以退出阻塞的状态（interrupt和sleep等方法不分前后，只要sleep等方法检测到中断状态，就会抛出中断异常）。 

更确切的说，如果线程被Object.wait, Thread.join和Thread.sleep三种方法之一阻塞，此时调用该线程的interrupt()方法，那么该线程将抛出一个 InterruptedException中断异常（该线程必须事先预备好处理此异常），从而提早地终结被阻塞状态。如果线程没有被阻塞，这时调用 interrupt()将不起作用，直到执行到wait(),sleep(),join()时,才马上会抛出 InterruptedException。

如果线程被interrupt，大概有这么几种情况。  
* 如果线程堵塞在object.wait、Thread.join和Thread.sleep，将会清除线程的中断状态，并抛出InterruptedException;  
* 如果线程堵塞在java.nio.channels.InterruptibleChannel的IO上，Channel将会被关闭，线程被置为中断状态，并抛出java.nio.channels.ClosedByInterruptException；  
* 如果线程堵塞在java.nio.channels.Selector上，线程被置为中断状态，select方法会马上返回，类似调用wakeup的效果；  
* 如果不是以上三种情况，thread.interrupt()方法仅仅是设置线程的中断状态为true。  

#### 使用 interrupt() + InterruptedException来中断线程
线程处于阻塞状态，如Thread.sleep、wait、IO阻塞等情况时，调用interrupt方法后，sleep等方法将会抛出一个InterruptedException：
```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        public void run() {
            System.out.println("线程启动了");
            try {
                Thread.sleep(1000 * 100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程结束了");
        }
    };
    thread.start();

    try {
        Thread.sleep(1000 * 5);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    thread.interrupt();//作用是：在线程阻塞时抛出一个中断信号，这样线程就得以退出阻塞的状态
}
```

#### 使用 interrupt() + isInterrupted()来中断线程
Thread.**interrupted()**:测试**当前线程**是否已经中断（**静态方法**）。如果连续调用该方法，则第二次调用将返回false。在api文档中说明interrupted()方法具有清除状态的功能。执行后具有将状态标识清除为false的功能。

Thread.this.**isInterrupted()**:测试线程是否已经中断，但是不能清除状态标识。

```java
public static void main(String[] args) {
    Thread thread = new Thread() {
        public void run() {
            System.out.println("线程启动了");
            while (!isInterrupted()) {
                System.out.println(isInterrupted());//调用 interrupt 之后为true
            }
            System.out.println("线程结束了");
        }
    };
    thread.start();

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    thread.interrupt();
    System.out.println("线程是否被中断：" + thread.isInterrupted());//true
}
```

#### 一个综合的例子
来一个综合的例子：
```java
public class test1 {

    static volatile boolean flag = true;

    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("开始休眠");
                try {
                    Thread.sleep(100 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("结束休眠，开始死循环");
                while (flag) {
                }
                System.out.println("------------------子线程结束------------------");
            }
        });
        thread.start();

        Scanner scanner = new Scanner(System.in);
        System.out.println("输入1抛出一个中断异常，输入2修改循环标志位，输入3判断线程是否阻塞，输入其他结束Scanner\n");
        while (scanner.hasNext()) {
            String text = scanner.next();
            System.out.println("你输入了：" + text + "\n");
            if ("1".equals(text)) {
                thread.interrupt();
            } else if ("2".equals(text)) {
                flag = false; //如果不设为false，主线程结束后子线程仍在运行
            } else if ("3".equals(text)) {
                System.out.println(thread.isInterrupted());
            } else {
                scanner.close();
                break;
            }
        }
        System.out.println("------------------主线程结束------------------");
    }
}
```
![885859-20190524155422570-592234637](/assets/885859-20190524155422570-592234637.png)

#### 不能结束的情况
注意下面这种是根本不能结束的情况！
```java
public class Test {
    public static void main(String[] args) {
        Thread thread = new Thread() {
            public void run() {
                System.out.println("线程启动了");
                while (true) {//对于这种情况，即使线程调用了intentrupt()方法并且isInterrupted()，但线程还是会继续运行，根本停不下来！
                    System.out.println(isInterrupted());//调用interrupt之后为true
                }
            }
        };
        thread.start();
        thread.interrupt();//注意，此方法不会中断一个正在运行的线程，它的作用是：在线程受到阻塞时抛出一个中断信号，这样线程就得以退出阻塞的状态
        while (true) {
            System.out.println("是否isInterrupted：" + thread.isInterrupted());//true
        }
    }
}
```

#### 关于interrupted()和isInterrupted()方法的注意事项
![885859-20190524155721463-347862106](/assets/885859-20190524155721463-347862106.png)
![885859-20190524155737797-364388846](/assets/885859-20190524155737797-364388846.png)

* interrupted()是静态方法：内部实现是调用的当前线程的isInterrupted()，并且会重置当前线程的中断状态。测试当前线程是否已经中断（静态方法）。返回的是上一次的中断状态，并且会清除该状态，所以连续调用两次，第一次返回true，第二次返回false。

* isInterrupted()是实例方法，是调用该方法的对象所表示的那个线程的isInterrupted()，不会重置当前线程的中断状态测试线程当前是否已经中断，但是不能清除状态标识。

测试方法验证：  
1：
```java
public static void main(String[] args){
    Thread thread = new Thread() {
        @Override
        public void run() {
            try
            {
                Thread.sleep(100 * 1000);
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();                
            }
        }
    };       
    thread.start();
    
    thread.interrupt();
    System.out.println("第一次调用返回值：" + Thread.interrupted());
    System.out.println("第二次调用返回值：" + Thread.interrupted());
}
```
```java
第一次调用返回值：false
第二次调用返回值：false
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.dp.dac.util.Test01$1.run(Test01.java:11)
```
中断的线程是我们自己创建的thread线程，我调用的interrupted()，由上面源码可知是判断当前线程的中断状态，当前线程是main线程，我根本没有中断过main线程，所以2次调用均返回“false”

2：
```java
public static void main(String[] args){
    Thread.currentThread().interrupt();
    System.out.println("第一次调用返回值：" + Thread.interrupted());
    System.out.println("第二次调用返回值：" + Thread.interrupted());
}
```
```java
第一次调用返回值：true
第二次调用返回值：false
```
中断的线程是当前线程（main线程），我调用的interrupted()，由上面源码可知是判断当前线程的中断状态，当前线程是main线程，所以第1次调用结果返回“true”，因为我确实中断了main线程，由源码可知interrupted()调用的是isInterrupted()，并会重置中断状态，所以第一次调用之后把中断状态给重置了，从中断状态重置为非中断状态，所以第2次调用的结果返回“false” ，而且知道interrupt只是标记中断状态，并不会立马中断线程。

 3：
```java
public static void main(String[] args){
    Thread.currentThread().interrupt();
    System.out.println("第一次调用返回值：" + Thread.currentThread().isInterrupted());
    System.out.println("第二次调用返回值：" + Thread.currentThread().isInterrupted());

    try
    {
    	Thread.sleep(100 * 1000);
    }
    catch (InterruptedException e)
    {
    	 System.out.println("第三次调用返回值：" + Thread.currentThread().isInterrupted());
    	e.printStackTrace();			
    }
}
```
```java
 第一次调用返回值：true
 第二次调用返回值：true
 第二次调用返回值：false
 java.lang.InterruptedException: sleep interrupted
 	at java.lang.Thread.sleep(Native Method)
 	at com.dp.dac.util.Test01.main(Test01.java:30)
```
可以知道isInterrupted()不会重置中断状态。sleep方法抛出异常会清除中断状态（如果任何线程中断了当前线程。当抛出该异常时，当前线程的中断状态被清除）。

#### wait 与 interrupt
obj.wait也能被interrupt抛出InterruptedException，不必等obj.notify()，不过要等到其他的线程释放了obj的锁。
```java
public static void main(String[] args){
    final Object obj = new Object();
    Thread thread = new Thread() {
        @Override
        public void run() {
            synchronized (obj) {
                System.out.println("sync start");
                try {
                    obj.wait();
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("sync end");
            }
            
        }
    };       
    thread.start();
    
    try {
        Thread.sleep(1 * 1000);
    }
    catch (InterruptedException e) {
        e.printStackTrace();
    }
   
    //thread.interrupt();//上面thread的obj.wait()处会立马跑出异常。
    synchronized (obj) {
        thread.interrupt();//必须等此处释放锁后，上面thread的obj.wait()处才会跑出异常。
        try {
            Thread.sleep(5 * 1000);
            System.out.println("main end");
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```java
sync start
main end
java.lang.InterruptedException
sync end
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:503)
	at com.dp.dac.util.Test02$1.run(Test02.java:13)
```

