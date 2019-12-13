## BlockingQueue
`https://cloud.tencent.com/developer/article/1014686?from=10680`
`https://cloud.tencent.com/developer/article/1014694`
url 是 Java 阻塞队列源码分析 文章的地址

下面只选择总结性的内容

### 什么是阻塞队列
阻塞队列其实就是生产者-消费者模型中的容器。

当生产者往队列中添加元素时，如果队列已经满了，生产者所在的线程就会阻塞，直到消费者取元素时 notify 它； 
 消费者去队列中取元素时，如果队列中是空的，消费者所在的线程就会阻塞，直到生产者放入元素 notify 它。

具体到 Java 中，使用 BlockingQueue 接口表示阻塞队列：
```java
public interface BlockingQueue<E> extends Queue<E> {
    //添加失败时会抛出异常，比较粗暴
    boolean add(E e);

    //添加失败时会返回 false，比较温婉，比 add 强
    boolean offer(E e);

    //添加元素时，如果没有空间，会阻塞等待；可以响应中断
    void put(E e) throws InterruptedException;

    //添加元素到队列中，如果没有空间会等待参数中的时间，超时返回，会响应中断
    boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    //获取并移除队首元素，如果没有元素就会阻塞等待
    E take() throws InterruptedException;

    //获取并移除队首元素，如果没有就会阻塞等待参数的时间，超时返回
    E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

    //返回队列中剩余的空间
    int remainingCapacity();

    //移除队列中某个元素，如果存在的话返回 true，否则返回 false
    boolean remove(Object o);

    //检查队列中是否包含某个元素，至少包含一个就返回 true
    public boolean contains(Object o);

    //将当前队列所有元素移动到给定的集合中，这个方法比反复地获取元素更高效
    //返回移动的元素个数
    int drainTo(Collection<? super E> c);

    //移动队列中至多 maxElements 个元素到指定的集合中
    int drainTo(Collection<? super E> c, int maxElements);
}
```

BlockingQueue 中不允许有 null 元素，因此在 add(), offer(), put() 时如果参数是 null，会抛出空指针。null 是用来有异常情况时做返回值的。

可以看到，在队列操作（添加/获取）当前不可用时，BlockingQueue 的方法有四种处理方式：

* 抛出异常  
对应的是 add(), remove(), element()

* 返回某个值（null 或者 false）  
offer(), poll(), peek()

* 阻塞当前线程，直到操作可以进行 
put(), take()

* 阻塞一段时间，超时后退出  
offer, poll()

### 7 种阻塞队列的特点
这里简单总结下 Java 中 7 种阻塞队列的特点：

* ArrayBlockingQueue   
环形数组实现的、有界的队列，一旦创建后，容量不可变
基于数组，在添加删除上性能还是不如链表

* LinkedBlockingQueue：  
基于链表、有界阻塞队列
添加和获取是两个不同的锁，所以并发添加/获取效率更高些
Executors.newFixedThreadPool() 使用了这个队列

* PriorityBlockingQueue   
基于数组的、支持优先级的、无界阻塞队列
使用自然排序或者定制排序指定排序规则
添加元素时，当数组中元素大于等于容量时，会扩容（当前队列中元素个数小于 64 个，数组容量就乘 3；否则就乘 2 加 2），拷贝数组

* DelayQueue 
支持延时获取元素的、无界阻塞队列
添加元素时如果超出限制也会扩容
Leader-Follower 模型

* SynchronousQueue 
容量为 0
一个添加操作后必须等待一个获取操作才可以继续添加
吞吐量高于 LinkedBlockingQueue 和 ArrayBlockingQueue

* LinkedTransferQueue  
由链表组成的、无界阻塞队列
实现了 TransferQueue 接口
CPU 自旋等待消费者取走元素，自旋一定次数后结束

* LinkedBlockingDeque  
由双向链表组成的、双向阻塞队列
可以从队列两端插入和移除元素
多了一个操作队列的方向，在多线程同时入队时，可以减少一半的竞争




