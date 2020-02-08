## Future接口的局限性
### Future接口测试
Java 1.5开始，提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。
Future接口可以构建异步应用，是多线程开发中常见的设计模式。

测试：
```java
import org.junit.Test;

import java.util.concurrent.*;

public class FutureTest {

    @Test
    public void test() {
        ExecutorService executor = Executors.newFixedThreadPool(4);

        Callable<String> callable = () -> {
            Thread.sleep(2000);
            //int i = 1 / 0;
            return Thread.currentThread().getName();
        };
        Future<String> future = executor.submit(callable);

        System.out.println(future.isDone());

        try {
            future.get(1, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            // 当前线程在等待过程中被中断
            System.out.println("当前线程被中断");
        } catch (ExecutionException e) {
            // 计算抛出一个异常
            System.out.println("callable执行过程中抛出异常");
        } catch (TimeoutException e) {
            // 在Future对象完成之前超时
            System.out.println("超时");
        }

        try {
            String result = future.get();//会一直阻塞
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            System.out.println("callable执行过程中抛出异常: " + e.getMessage());
        }

        //junit中主线程结束后，程序就会结束。这里不用关闭线程池。
        //executor.shutdown();
    }

}
```
输出
```
false
超时
pool-1-thread-1
```

### Future接口的局限性
JDK5 新增了 Future 接口，用于描述一个异步计算的结果。虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，**只能通过阻塞或者轮询的方式得到任务的结果**。 例如：
```java
public static void main(String[] args) throws Exception {
    ExecutorService es = Executors.newSingleThreadExecutor();

    // 在 Java8 中，推荐使用 Lambda 来替代匿名 Callable 实现类
    Future<Integer> f = es.submit(() -> {
        System.out.println(Thread.currentThread().getName());

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        return 123;
    });

    // 当前 main 线程阻塞，直至 future 得到值
    System.out.println(f.get());

    es.shutdown();
}
```

阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果，**为什么不能用观察者设计模式呢？即当计算结果完成及时通知监听者。（例如通过回调的方式）**

关于 Future 接口，还有如下一段描述：
```
The Future interface was added in Java 5 to serve as a result of an asynchronous computation, but it did not have any methods to combine these computations or handle possible errors.
不能很好地组合多个异步任务，也不能处理可能的异常。
```

Future很难直接表述多个Future 结果之间的依赖性，开发中，我们经常需要达成以下目的：

* 将两个异步计算合并为一个————这两个异步计算之间相互独立，同时第二个又依赖于第    
* 一个的结果。    
* 等待Future集合中的所有任务都完成。    
* 仅等待Future集合中最快结束的任务完成（有可能因为它们试图通过不同的方式计算同    
* 一个值），并返回它的结果。    
* 通过编程方式完成一个Future任务的执行（即以手工设定异步操作结果的方式）。    
* 应对Future的完成事件（即当Future的完成事件发生时会收到通知，并能使用Future    
* 计算的结果进行下一步的操作，不只是简单地阻塞等待操作的结果）。    



