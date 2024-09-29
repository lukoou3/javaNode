
# Semaphore
https://www.cnblogs.com/JavaBuild/p/18133329

## 什么是Semaphore？


在前面我们讲过的synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，而`Semaphore(信号量)`可以用来控制同时访问特定资源的线程数量，多线程同时操作共享资源，仍然存在着线程不安全问题，要想多线程安全，理应结合锁进一步保障。


## Semaphore的主要结构


我们跟进信号量的源码中浏览一圈，发现其实它内部主要的方法就2个：

```java
// 初始共享资源数量
final Semaphore semaphore = new Semaphore(5);
// 获取1个许可
semaphore.acquire();
// 释放1个许可
semaphore.release();
```


**① acquire()**:获取许可


跟进这个方法后，我们会发现其内部调用了AQS的一个final 方法acquireSharedInterruptibly()，这个方法中又调用了tryAcquireShared(arg)方法，作为AQS中的钩子方法，这个方法的实现在Semaphore的两个静态内部类**FairSync（公平模式）**和**NonfairSync（非公平模式）**中。


**【源码解析1】**


```java
/**
 *  获取1个许可证
 */
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
/**
 * 共享模式下获取许可证，获取成功则返回，失败则加入阻塞队列，挂起线程
 */
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // 尝试获取许可证，arg为获取许可证个数，当可用许可证数减当前获取的许可证数结果小于0,则创建一个节点加入阻塞队列，挂起当前线程。
    if (tryAcquireShared(arg) < 0)
      doAcquireSharedInterruptibly(arg);
}
```

继续跟入tryAcquireShared(arg)方法，虽然它在AQS中，但它作为钩子方法，最终的实现则回到了Semaphore的内部类中。


**【扩展】**


* **FairSync（公平模式）：** 调用 acquire() 方法的顺序就是获取许可证的顺序，遵循 FIFO；    
* **NonfairSync（非公平模式）：** 抢占式的，在构造方法中通过一个布尔值fair来配置公平与非公平，默认为非公平模式。


**② release()**:释放许可


同样跟入这个方法，里面用了AQS的releaseShared()，而在这个方法内也毫无疑问的用了tryReleaseShared(int arg)这个钩子方法，原理同上，不再冗释，需要注意的是释放共享锁的同时也会唤醒同步队列中的一个线程。


**【源码解析2】**


```java
// 释放一个许可证
public void release() {
    sync.releaseShared(1);
}
 
// 释放共享锁，同时会唤醒同步队列中的一个线程。
public final boolean releaseShared(int arg) {
    //释放共享锁
    if (tryReleaseShared(arg)) {
      //唤醒同步队列中的一个线程
      doReleaseShared();
      return true;
    }
    return false;
}
 
```


## Semaphore的使用


OK，讲到这里，把信号量中主要的方法解释完了，我们来写一个小demo感受一下它的使用：


**【测试用例1】**


```java
public class Test {
    private final Semaphore semaphore;
 
    /*构造一个令牌*/
    public Test(int acq){
        this.semaphore= new Semaphore(acq);
    }
    public void useSemaphore(){
        try {
            semaphore.acquire();
            // 使用资源
            System.out.println("资源开始使用了 " + Thread.currentThread().getName());
            Thread.sleep(1000); // 模拟资源使用时间
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release();
            System.out.println("资源释放了 " + Thread.currentThread().getName());
        }
    }
 
    public static void main(String[] args) {
        Test test = new Test(3);
        for (int i = 0; i < 5; i++) {
            new Thread(test::useSemaphore).start();
        }
    }
}
```

**输出：**

```
资源开始使用了 Thread-0
资源开始使用了 Thread-1
资源开始使用了 Thread-3
资源释放了 Thread-0
资源开始使用了 Thread-2
资源开始使用了 Thread-4
资源释放了 Thread-1
资源释放了 Thread-3
资源释放了 Thread-4
资源释放了 Thread-2
```



