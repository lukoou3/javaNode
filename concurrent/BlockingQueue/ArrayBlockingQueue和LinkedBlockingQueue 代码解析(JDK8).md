## ArrayBlockingQueue 和 LinkedBlockingQueue 代码解析(JDK8)
`https://yq.aliyun.com/articles/712761?spm=a2c4e.11153940.0.0.4e814af39iLD24`

在使用线程池的时候，需要指定BlockingQueue 常用的一般有ArrayBlockingQueue和LinkedBlockingQueue

有一天被问到有什么区别没回答上来，因此从代码的层面解析一下

### BlcokingQueue相关方法
BlockingQueue的插入/移除/检查这些方法，对于不能立即满足但可能在将来某一时刻可以满足的操作，共有4种不同的处理方式：第一种是抛出一个异常，第二种是返回一个特殊值（null 或false，具体取决于操作），第三种是在操作可以成功前，无限期地阻塞当前线程，第四种是在放弃前只在给定的最大时间限制内阻塞。如下表格：

| 操作 | 抛出异常  | 特殊值   | 阻塞   | 超时                 |
| ---- | --------- | -------- | ------ | -------------------- |
| 插入 | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
| 移除 | remove()  | poll()   | take() | poll(time, unit)     |
| 检查 | element() | peek()   | 不可用 | 不可用               |

注意**element、peek和poll、remove、take的区别**，element、peek不会删除元素。

**插入方法**：

* offer(e)：将指定元素添加到Queue中，如果可以容纳，返回true，否则返回false    
* add(e)：将指定元素添加到Queue中，如果可以容纳，返回true，否则抛出异常    
* put(e)：put操作 和offer操作基本一致，只不过在Queue满时进行等待，直到Queue有可用的空间    
* offer(e, time, unit)：将指定元素添加到Queue中，在到达指定的等待时间前等待可用的空间（如果有必要），超时还没空间返回false   

**移除方法**：

* poll()：获取并移除Queue的头，如果此队列为空，则返回 null。    
* remove()：取出Queue中首位元素，若为空则抛出异常    
* take()：取出Queue中首位元素，若为空则阻塞，直到有新对象被加入    
* poll(time, unit)：获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。  

**检查方法**：

* element()：返回队列头部元素，如果队列为空，则抛出异常    
* peek()：返回队列头部元素，如果队列为空，则返回null  

### ArrayBlockingQueue
顾名思义，就是用Array来实现的queue，Blocking 则说明是线程安全的

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable {
    private static final long serialVersionUID = -817911632652898426L;
    final Object[] items;
    int takeIndex;
    int putIndex;
    int count;
    final ReentrantLock lock;
    private final Condition notEmpty;
    private final Condition notFull;
}
```

* items 存储数据的数组    
* takeIndex 取数据时数组的下标    
* putIndex 放数据时的下标    
* count 数据的数量    
* lock 使用ReentrantLock 来保证线程安全    
* notEmpty 非空信号量，用来进行取数据时的信号量    
* notFull 非满信号量，在写数据时数据满时的等待信号量

#### 构造函数
```java
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];//指定数组大小
    lock = new ReentrantLock(fair); //根据参数确定lock是否为公平锁，默认为false
    notEmpty = lock.newCondition(); //新建两个lock的信号量
    notFull =  lock.newCondition();
}
```

#### 写数据 
研究代码发现 put add offer三个方法都调用了enqueue方法，ArrayBlockingQueue 将对数组的实际操作在jdk8抽象了出来,相对于jdk7进行了一定优化

```java
/**
 * Inserts element at current put position, advances, and signals.
 * Call only when holding lock.
 */
//该方法只有在对象获取到锁之后才能调用
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items; //获取数组对象
    items[putIndex] = x; //putIndex 默认值为0
    if (++putIndex == items.length) //在putIndex到达数组尾部时，重新指向数组第一个位置
        putIndex = 0;
    count++; //数组元素+1
    notEmpty.signal(); //非空信号发送
}
```

##### offer 
offer方法 尝试插入数据，在数组满时返回false，正常插入 返回true
```java
public boolean offer(E e) {
    checkNotNull(e); //校验数据是否为null
    final ReentrantLock lock = this.lock; //获取对象锁
    lock.lock(); //对当前对象加锁
    try {
        if (count == items.length) //如果数组满，返回false
            return false;
        else {
            enqueue(e); //数组没满，插入数据，返回true
            return true;
        }
    } finally {
        lock.unlock(); //释放锁
    }
}
```

##### add
ArrayBlockingQueue 调用了父类AbstractQueue的add方法，  
在插入成功时返回true，在插入失败(数组满)时，抛出异常，  
AbstractQueue 的add方法调用了offer()方法，所以add是offer的功能升级版

```java
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
```

##### put 
put方法 在进行数据插入时，会尝试获取锁并相应异常，同时，在数组满时，会一致等待，直到数组有了空闲空间

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();//尝试获取锁并相应异常
    try {
        while (count == items.length) //数组满，
            notFull.await(); //等待非满信号
        enqueue(e);
    } finally {
        lock.unlock(); //在数据正常插入或者其他线程抛出异常后，解锁
    }
}
```

##### offer(E e, long timeout, TimeUnit unit) 
ArrayBlockingQueue 还提供了一种超时配置的方法，在数组数据满超过timeout后返回fasle

```java
public boolean offer(E e, long timeout, TimeUnit unit)
      throws InterruptedException {

      checkNotNull(e);
      long nanos = unit.toNanos(timeout);
      final ReentrantLock lock = this.lock;
      lock.lockInterruptibly();
      try {
          while (count == items.length) {
              if (nanos <= 0)
                  return false;
              nanos = notFull.awaitNanos(nanos); //condition超时后 返回-1
          }
          enqueue(e);
          return true;
      } finally {
          lock.unlock();
      }
  }
```

#### 取数据 
和写数据一样，取数据jdk8也进行了一定优化 统一调用dequeue方法

```java
private E dequeue() {
     // assert lock.getHoldCount() == 1;
     // assert items[takeIndex] != null;
     final Object[] items = this.items; 
     @SuppressWarnings("unchecked")
     E x = (E) items[takeIndex]; //获取最老数据
     items[takeIndex] = null; //最老数据位置置空
     if (++takeIndex == items.length) //下标到达最后 置零
         takeIndex = 0;
     count--; 
     if (itrs != null) //itrl目前没看到初始化的位置 ，暂时不清楚有什么用
         itrs.elementDequeued();
     notFull.signal();
     return x;
 }
```

##### poll
很简单 列表为空返回null否则放回对应数据

```java
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock(); //加锁
    try {
        return (count == 0) ? null : dequeue(); //列表为空返回null否则放回对应数据
    } finally {
        lock.unlock(); //解锁
    }
}
```

##### remove
获取并移除此队列的头。  
ArrayBlockingQueue 调用了父类AbstractQueue的remove方法，  
此方法与 poll 唯一的不同在于：此队列为空时将抛出一个异常。 除非队列为空，否则此实现返回 poll 的结果。   
AbstractQueue 的remove方法调用了poll()方法，所以add是offer的功能升级版
```java
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}
```

##### take
尝试加锁，在数组为空时一直等待，直到有新数据或者被外部中断
```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

##### poll(long timeout, TimeUnit unit)(E e, long timeout, TimeUnit unit) 
也提供了等待超过timeout 返回null的poll方法
```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos); //如果超时awaitNanos 返回-1 ，最后返回null
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

#### 检查数据
##### peek
获取但不移除此队列的头；如果此队列为空，则返回 null。   
因此一条数据可以重复peek多次
```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return itemAt(takeIndex); // null when queue is empty
    } finally {
        lock.unlock();
    }
}
final E itemAt(int i) {
    return (E) items[i];
}
```

##### element
获取，但是不移除此队列的头。此方法与 peek 唯一的不同在于：此队列为空时将抛出一个异常。 
ArrayBlockingQueue 调用了父类AbstractQueue的element方法。  
AbstractQueue 的element方法调用了peek()方法
```java
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}
```

### LinkedBlockingQueue
顾名思义，就是使用链表来存储的线程安全的队列，LinkedBlockingQueue 采用了读写锁分离，因此在短时间内产生大量读写操作时，比arrayBlockingQueue性能更加优秀
```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable {
    private static final long serialVersionUID = -6903933977591709194L;
    private final int capacity;
    private final AtomicInteger count;
    transient LinkedBlockingQueue.Node<E> head;
    private transient LinkedBlockingQueue.Node<E> last;
    private final ReentrantLock takeLock;
    private final Condition notEmpty;
    private final ReentrantLock putLock;
    private final Condition notFull;
}
```
* capacity 链表的最大长度，默认为Integer.MAX_VALUE    
* count 元素数量    
* head 头节点    
* last 尾节点    
* takeLock 取数据锁    
* notEmpty 非空信号量    
* putLock 写数据锁    
* notFull 非满信号量

#### 构造函数
在构造LinkedBlockingQueue对象时，如果没有指定容量大小，默认采用Integer.MAX_VALUE（当生产者速度大于消费者速度时，可能会造成内存消耗殆尽） 
```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}
public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity; //设置最大长度
    last = head = new Node<E>(null); //
}
```

#### 写数据 
LinkedBlockingQueue同样提供了三个函数 put offer add  
同样提供了enqueue方法，该方法仅在获取到putLock 后执行
```java
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node; //在尾节点添加数据
}
```

##### offer(E e)
将指定元素插入到此队列的尾部（如果立即可行且不会超出此队列的容量），在成功时返回 true，如果此队列已满，则返回 false。当使用有容量限制的队列时，此方法通常要优于 add 方法，后者可能无法插入元素，而只是抛出一个异常。 
```java
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    final AtomicInteger count = this.count; //获取元素数量
    if (count.get() == capacity) //如果链表长度到达上限，返回null
        return false;
    int c = -1;
    Node<E> node = new Node<E>(e); //创建新节点
    final ReentrantLock putLock = this.putLock; 
    putLock.lock();  //写锁加锁
    try {
        if (count.get() < capacity) { //如果没有到达链表上限
            enqueue(node);   //新增节点
            c = count.getAndIncrement(); //获取元素数量并将count+1(c=count，count++)，
                                         //读写锁分离，链表数量可能有减少
            if (c + 1 < capacity)  //如果链表数量没有达到上限，非满信号量通知
                notFull.signal();
        }
    } finally {
        putLock.unlock(); //解锁
    }
    if (c == 0)  //如果链表原来的数量为0
        signalNotEmpty(); //非空信号量通知
    return c >= 0;  //返回插入结果 成功返回true，失败返回fasle
}
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock; //获取读锁
    takeLock.lock(); //读锁加锁，防止数据被读取
    try {
        notEmpty.signal(); //非空信号量通知
    } finally {
        takeLock.unlock(); //读锁解锁
    }
}
```

##### add(E e) 
和ArrayBlockingQueue一样，LinkedBlockingQueue 调用了父类AbstractQueue的add方法，在新增成功后返回true，在新增失败后直接抛出异常
```java
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
```

##### put(E e)
put操作 和offer操作基本一致，只不过在链表满时进行等待，知道链表节点减少
```java
public void put(E e) throws InterruptedException {
   if (e == null) throw new NullPointerException();
   int c = -1;
   Node<E> node = new Node<E>(e);
   final ReentrantLock putLock = this.putLock; //获取写锁
   final AtomicInteger count = this.count;
   putLock.lockInterruptibly(); //对写锁加锁并相应异常
   try {
  
       while (count.get() == capacity) { //如果节点数量到达上限
           notFull.await(); //等待非满信号量的通知
       }
       enqueue(node); //在数据被弹出后，插入新节点
       c = count.getAndIncrement();  
       if (c + 1 < capacity)
           notFull.signal();
   } finally {
       putLock.unlock(); //释放写锁
   }
   if (c == 0) 
       signalNotEmpty();
}
```

##### offer(E e, long timeout, TimeUnit unit)
和ArrayBlockingQueue一样，如果链表长度到达上限，就等待timeout ，超时后直接返回fasle
```java
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (e == null) throw new NullPointerException();
    long nanos = unit.toNanos(timeout);
    int c = -1;
    final ReentrantLock putLock = this.putLock;//获取写锁
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();//对写锁加锁并相应异常
    try {
        while (count.get() == capacity) {//如果节点数量到达上限
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(new Node<E>(e));//插入新节点
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();//释放写锁
    }
    if (c == 0)
        signalNotEmpty();
    return true;
}
```

#### 读取数据 
同样提供了dequeue方法
```java
private E dequeue() {
    // assert takeLock.isHeldByCurrentThread();
    // assert head.item == null;
    Node<E> h = head; // 获取头节点
    Node<E> first = h.next; //first设置为新的头节点
    h.next = h; // help GC //需要移除的节点next指向自己帮助gc
    head = first;  //head 置为新的头节点
    E x = first.item; //获取返回值得item
    first.item = null; //first item设置为null 
    return x; //返回item
}
```

##### poll()
获取并移除此队列的头，如果此队列为空，则返回 null。
```java
public E poll() {
    final AtomicInteger count = this.count; //获取数量
    if (count.get() == 0) //如果链表节点数量为空 返回null
        return null;
    E x = null;
    int c = -1;
    final ReentrantLock takeLock = this.takeLock; //获取读锁并加锁
    takeLock.lock();
    try {
        if (count.get() > 0) {
            x = dequeue(); //获取数据
            c = count.getAndDecrement(); //数量-1
            if (c > 1)  //剩余节点>1
                notEmpty.signal(); //非空信号通知
        }
    } finally {
        takeLock.unlock(); //读锁解锁
    }
    if (c == capacity) //可能有线程在等在非满信号，-1前数量=限定长度
        signalNotFull();
    return x;
}
private void signalNotFull() {
    final ReentrantLock putLock = this.putLock; //获取写锁并加锁
    putLock.lock();
    try {
        notFull.signal(); //非满信号通知
    } finally {
        putLock.unlock(); //写锁解锁
    } 
}
```

##### remove
获取并移除此队列的头。  
和ArrayBlockingQueue一样 调用了父类AbstractQueue的remove方法，  
此方法与 poll 唯一的不同在于：此队列为空时将抛出一个异常。 除非队列为空，否则此实现返回 poll 的结果。   
```java
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}
```

##### take(E e)
链表到达最大长度。等待，可以被异常中断
```java
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

##### poll(long timeout, TimeUnit unit)
等待超过timeout 返回null
```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E x = null;
    int c = -1;
    long nanos = unit.toNanos(timeout);
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

### ArrayBlockingQueue和LinkedBlockingQueue的区别
一下内容摘抄自不同的博客。

#### 版本1
两者的区别：

* 1 Linked读写锁分离，在短时间内发生大量读写交替操作时性能高    
* 2 Array在读写操作时不需要维护额外节点，空间较少    
* 3 Array使用int count Linked使用AtomicInteger ，    

因此：Array使用唯一Lock来保证count强一致性，Linked使用Atomic来保证count的准确性

#### 版本2
ArrayBlockingQueue和LinkedBlockingQueue的区别：

<p style="padding:0px;" data-spm-anchor-id="a2c4e.11153940.0.i6.b31a2f0azGlymp">ArrayBlockingQueue和LinkedBlockingQueue的区别：<br style="margin:0px;padding:0px;"><br style="margin:0px;padding:0px;">
1. 队列中的锁的实现不同<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ArrayBlockingQueue中的锁是没有分离的，即生产和消费用的是同一个锁；<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LinkedBlockingQueue中的锁是分离的，即生产用的是putLock，消费是takeLock<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;<br style="margin:0px;padding:0px;">
2. 在生产或消费时操作不同<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue基于数组，在生产和消费的时候，是直接将枚举对象插入或移除的，不会产生或销毁任何额外的对象实例；<br style="margin:0px;padding:0px;">
&nbsp;&nbsp; &nbsp; LinkedBlockingQueue基于链表，在生产和消费的时候，需要把枚举对象转换为Node&lt;E&gt;进行插入或移除，会生成一个额外的Node对象，这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。<br style="margin:0px;padding:0px;"><br style="margin:0px;padding:0px;">
3. 队列大小初始化方式不同<br style="margin:0px;padding:0px;">
&nbsp;&nbsp; &nbsp; ArrayBlockingQueue是有界的，必须指定队列的大小；<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LinkedBlockingQueue是无界的，可以不指定队列的大小，但是默认是Integer.MAX_VALUE。当然也可以指定队列大小，从而成为有界的。<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;<br style="margin:0px;padding:0px;">
注意:<br style="margin:0px;padding:0px;">
在使用LinkedBlockingQueue时，若用默认大小且当生产速度大于消费速度时候，有可能会内存溢出<br style="margin:0px;padding:0px;">
在使用ArrayBlockingQueue和LinkedBlockingQueue分别对1000000个简单字符做入队操作时，<br style="margin:0px;padding:0px;">
&nbsp;&nbsp; &nbsp; &nbsp; LinkedBlockingQueue的消耗是ArrayBlockingQueue消耗的10倍左右，<br style="margin:0px;padding:0px;">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 即LinkedBlockingQueue消耗在1500毫秒左右，而ArrayBlockingQueue只需150毫秒左右。<br style="margin:0px;padding:0px;">
按照实现原理来分析，ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea之所以没这样去做，也许是因为ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。</p>

#### 版本3
ArrayBlcokingQueue，LinkedBlockingQueue与Disruptor三种队列对比与分析

1、ArrayBlcokingQueue与LinkedBlockingQueue，一般认为前者基于数组实现，初始化后不需要再创建新的对象，但没有进行锁分离，所以内存GC压力较小，但性能会相对较低；后者基于链表实现，每次都需要创建  一个node对象，会存在频繁的创建销毁操作，GC压力较大，但插入和删除数据是不同的锁，进行了锁分离，性能会相对较好；从测试结果上看，其实两者性能和GC上差别都不大，在实际运用过程中，我认为一般场景下ArrayBlcokingQueue的性能已经足够应对，处于对GC压力的考虑，及潜在的OOM的风险我建议普通情况下使用ArrayBlcokingQueue即可。当然你也可以使用LinkedBlockingQueue，从测试结果上看，它相比ArrayBlcokingQueue性能上有有所提升但并不明显，结合gc的压力和潜在OOM的风险，所以结合应用的场景需要综合考虑。

2、Disruptor做为一款高性能队列框架，确实足够优秀，在测试中我们可以看到无论是性能和GC压力都远远好过ArrayBlcokingQueue与LinkedBlockingQueue;如果你追求更高的性能，那么Disruptor是一个很好的选择。
但需要注意的是，你需要结合自己的硬件配置和业务场景，正确配置Disruptor，选择合适的消费策略，这样不仅可以获取较高的性能，同时可以保证硬件资源的合理分配。

3、对这三种阻塞队列的测试，并不是为了比较孰优孰劣，主要是为了加强理解，实际的业务应用需要根据情况合理进行选择。这里只是结合自己的使用，对它们进行一个简单的总结，并没有进行较深入的探究，如有错误的的地方还请指正与海涵。

#### 版本4
BlockingQueue成员介绍：

<ul>
<li><strong>ArrayBlockingQueue</strong> <br>
1、基于数据实现的阻塞队列，内部维护一个定长数组，一旦创建，容量无法改变 <br>
2、支持可选的公平性策略用于订购所选的生产者和消费者线程（公平性通常会降低吞吐），默认非公平锁 <br>
3、内部维护两个整型变量用来标识队列的头部和尾部在数组中的位置 <br>
4、对于生产者和消费者采用的是用一个锁控制数据同步</li>
<li><strong>LinkedBlockingQueue</strong> <br>
1、对于生产者和消费者分别采用独立的锁来控制数据同步 <br>
2、在构造LinkedBlockingQueue对象时，如果没有指定容量大小，默认采用Integer.MAX_VALUE（当生产者速度大于消费者速度时，可能会造成内存消耗殆尽） <br>
3、在插入或删除元素时会生成一个额外Node对象</li>
<li><strong>DelayQueue</strong> <br>
DelayQueue是一个没有大小限制的队列，只有当其中的元素指定的延迟时间到了，才能够从队列中获取到该元素。因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。 <br>
使用场景：DelayQueue使用场景较少，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。</li>
<li><strong>PriorityBlockingQueue</strong> <br>
基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。</li>
<li><strong>SynchronousQueue</strong> <br>
1、声明一个SynchronousQueue有两种不同的方式，它们之间有着不太一样的行为。公平模式和非公平模式的区别: <br>
公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略； <br>
非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。</li>
</ul>

