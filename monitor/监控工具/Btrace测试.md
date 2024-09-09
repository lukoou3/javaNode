
## 下载配置环境变量
`http://github.com/btraceio/btrace`，在Release页面里下载最新版本，解压就可以使用。zip那个是Windows版本的。

/etc/profile.d/env.sh:
```sh
#BTRACE_HOME
export BTRACE_HOME=/opt/module/btrace
export PATH=$PATH:$BTRACE_HOME/bin

```

## 编写脚本引入依赖

maven中似乎没有, 直接引用本地的jar。

pom.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>BtraceNote</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-client</artifactId>
            <scope>system</scope>
            <type>jar</type>
            <systemPath>${project.basedir}/lib/btrace-client.jar</systemPath>
            <version>2.2.5</version>
        </dependency>
        <dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-agent</artifactId>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/btrace-agent.jar</systemPath>
            <version>2.2.5</version>
        </dependency>
        <dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-boot</artifactId>
            <scope>system</scope>
            <type>jar</type>
            <systemPath>${project.basedir}/lib/btrace-boot.jar</systemPath>
            <version>2.2.5</version>
        </dependency>
    </dependencies>
</project>
```

## 编写脚本测试

### 打印方法进入和退出时的信息
以druid构建Segments镜像信息为例：

SqlSegmentsMetadataManagerMethods.java:
```java
package com.btrace.druid;

import org.openjdk.btrace.core.annotations.*;

import static org.openjdk.btrace.core.BTraceUtils.*;

@BTrace
public class SqlSegmentsMetadataManagerMethods {

    @OnMethod(clazz = "org.apache.druid.metadata.SqlSegmentsMetadataManager", method = "doPoll")
    public static void doPollEntry(){
        long millis = timeMillis();
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + "[" + name(currentThread()) + "],doPoll start at millis:" + millis);
        println("#############################");
    }

    @OnMethod(clazz = "org.apache.druid.metadata.SqlSegmentsMetadataManager", method = "doPoll",
            location = @Location(value = Kind.RETURN))
    public static void doPollReturn(@Duration long duration) {
        long millis = timeMillis();
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + "[" + name(currentThread()) + "],doPoll end at millis:" + millis + ", duration ms:" + duration/1000000);
        println("#############################");
    }

    @OnMethod(clazz = "org.apache.druid.client.DataSourcesSnapshot", method = "fromUsedSegments")
    public static void fromUsedSegmentsEntry(){
        long millis = timeMillis();
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + "[" + name(currentThread()) + "],fromUsedSegments start at millis:" + millis);
        println("#############################");
    }

    @OnMethod(clazz = "org.apache.druid.client.DataSourcesSnapshot", method = "fromUsedSegments",
            location = @Location(value = Kind.RETURN))
    public static void fromUsedSegmentsReturn(@Duration long duration) {
        long millis = timeMillis();
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + "[" + name(currentThread())+ "],fromUsedSegments end at millis:" + millis + ", duration ms:" + duration/1000000);
        println("#############################");
    }
}

```

测试：
```
(base) [root@single btrace]# btrace 6253 SqlSegmentsMetadataManagerMethods.java
[main] INFO org.openjdk.btrace.client.Main - Attaching BTrace to PID: 6253
[main] INFO org.openjdk.btrace.client.Client - Successfully started BTrace probe: SqlSegmentsMetadataManagerMethods.java
2024-05-25 09:16:52.063[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],doPoll start at millis:1716628612063

#############################

2024-05-25 09:16:52.077[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],fromUsedSegments start at millis:1716628612077

#############################

2024-05-25 09:16:52.080[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],fromUsedSegments end at millis:1716628612080, duration ms:3

#############################

2024-05-25 09:16:52.080[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],doPoll end at millis:1716628612080, duration ms:17

#############################

2024-05-25 09:17:02.082[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],doPoll start at millis:1716628622082

#############################

2024-05-25 09:17:02.091[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],fromUsedSegments start at millis:1716628622090

#############################

2024-05-25 09:17:02.092[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],fromUsedSegments end at millis:1716628622092, duration ms:1

#############################

2024-05-25 09:17:02.092[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],doPoll end at millis:1716628622092, duration ms:10

#############################

```

### 打印方法调用栈

SqlSegmentsMetadataManagerMethodsJstack.java:
```java
package com.btrace.druid;

import org.openjdk.btrace.core.annotations.*;

import static org.openjdk.btrace.core.BTraceUtils.*;

@BTrace
public class SqlSegmentsMetadataManagerMethodsJstack {

    @OnMethod(clazz = "org.apache.druid.metadata.SqlSegmentsMetadataManager", method = "doPoll")
    public static void doPollEntry(){
        long millis = timeMillis();
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + "[" + name(currentThread()) + "],doPoll enter at millis:" + millis);
        jstack(); //打印堆栈信息
        println("#############################");
    }
}

```

测试：
```
(base) [root@single btrace]# btrace 6253 SqlSegmentsMetadataManagerMethodsJstack.java
[main] INFO org.openjdk.btrace.client.Main - Attaching BTrace to PID: 6253
[main] INFO org.openjdk.btrace.client.Client - Successfully started BTrace probe: SqlSegmentsMetadataManagerMethodsJstack.java
2024-05-25 09:18:52.184[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],doPoll enter at millis:1716628732184

org.apache.druid.metadata.SqlSegmentsMetadataManager.doPoll(SqlSegmentsMetadataManager.java)
org.apache.druid.metadata.SqlSegmentsMetadataManager.poll(SqlSegmentsMetadataManager.java:880)
org.apache.druid.metadata.SqlSegmentsMetadataManager.lambda$createPollTaskForStartOrder$0(SqlSegmentsMetadataManager.java:347)
java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
java.lang.Thread.run(Thread.java:750)

#############################

2024-05-25 09:19:02.196[org.apache.druid.metadata.SqlSegmentsMetadataManager-Exec--0],doPoll enter at millis:1716628742196

org.apache.druid.metadata.SqlSegmentsMetadataManager.doPoll(SqlSegmentsMetadataManager.java)
org.apache.druid.metadata.SqlSegmentsMetadataManager.poll(SqlSegmentsMetadataManager.java:880)
org.apache.druid.metadata.SqlSegmentsMetadataManager.lambda$createPollTaskForStartOrder$0(SqlSegmentsMetadataManager.java:347)
java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
java.lang.Thread.run(Thread.java:750)

#############################

```

### 打印构造函数参数
```
查看日志是昨天20:03机器关机，今天凌晨00:30重启后druid集群状态不正常，界面显示管理节点列表是192.168.44.14(leader)和192.168.44.11，但是摄入任务获取的管理节点列表是192.168.44.11，导致任务访问192.168.44.14节点认证不通过。

重启192.168.44.14节点，任务恢复正常。

可能是druid通过zookeeper动态发现管理节点的bug，偶发现象。
```

错误日志：
```
2024-06-13T02:39:05,894 INFO [ServiceClientFactory-0] org.apache.druid.rpc.ServiceClientImpl - Service [OVERLORD] issued redirect to unknown URL [http://192.168.44.14:8081/druid/indexer/v1/action] on attempt #55; retrying in 60,000 ms.
2024-06-13T02:40:05,898 INFO [ServiceClientFactory-1] org.apache.druid.rpc.ServiceClientImpl - Service [OVERLORD] issued redirect to unknown URL [http://192.168.44.14:8081/druid/indexer/v1/action] on attempt #56; retrying in 60,000 ms.
2024-06-13T02:41:05,902 INFO [ServiceClientFactory-0] org.apache.druid.rpc.ServiceClientImpl - Service [OVERLORD] issued redirect to unknown URL [http://192.168.44.14:8081/druid/indexer/v1/action] on attempt #57; retrying in 60,000 ms.
2024-06-13T02:42:05,906 INFO [ServiceClientFactory-1] org.apache.druid.rpc.ServiceClientImpl - Service [OVERLORD] issued redirect to unknown URL [http://192.168.44.14:8081/druid/indexer/v1/action] on attempt #58; retrying in 60,000 ms.
```

ServiceClientImpl类判断不满足以下条件会报上面的错：
```java
serviceLocations.getLocations().stream().anyMatch(loc -> serviceLocationMatches(loc, redirectLocationNoPath))
```

```java
public class ServiceLocations
{
  private static final Logger log = new Logger(ServiceLocations.class.getSimpleName()); // 新加的调试
  private final Set<ServiceLocation> locations;
  private final boolean closed;

  private ServiceLocations(final Set<ServiceLocation> locations, final boolean closed)
  {
    this.locations = Preconditions.checkNotNull(locations, "locations");
    this.closed = closed;

    log.info("locations:[%s],closed:[%s]", locations, closed); // 新加的调试

    if (closed && !locations.isEmpty()) {
      throw new IAE("Locations must be empty for closed services");
    }
  }
}
```

ServiceLocationsBtrace.java:
```java
package com.btrace.druid;

import org.openjdk.btrace.core.types.AnyType;
import org.openjdk.btrace.core.annotations.*;

import static org.openjdk.btrace.core.BTraceUtils.*;

@BTrace
public class ServiceLocationsBtrace {

    @OnMethod(clazz = "org.apache.druid.rpc.ServiceLocations", method = "<init>")
    public static void doPollEntry(AnyType locations, boolean closed){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":locations:" + locations + ",closed:" + closed + "");
        println("#############################");
    }
}
```



### 打印异常调用栈

异常日志信息只是`java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.Long (java.lang.Integer and java.lang.Long are in module java.base of loader 'bootstrap')`，想要打印调用栈，发现没法捕获ClassCastException和Exception的异常，自己定义的异常可以捕获：

```java
package com.btrace.druid;

import org.openjdk.btrace.core.types.AnyType;
import org.openjdk.btrace.core.annotations.*;

import static org.openjdk.btrace.core.BTraceUtils.*;

@BTrace
public class ClassCastExceptionBtrace {
    @OnMethod(clazz = "com.geedgenetworks.starrocks.udf.MyException", method = "<init>")
    public static void doPollEntry0(AnyType s){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":s:" + s);
        jstack();
        println("#############################");
    }
    
    @OnMethod(clazz = "java.lang.ClassCastException", method = "<init>")
    public static void doPollEntry1(AnyType s){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":s:" + s);
        jstack();
        println("#############################");
    }
    
    @OnMethod(clazz = "java.lang.Throwable", method = "<init>")
    public static void doPollEntry2(){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS"));
        jstack();
        println("#############################");
    }
    
    @OnMethod(clazz = "java.lang.Throwable", method = "<init>")
    public static void doPollEntry3(AnyType s){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":s:" + s);
        jstack();
        println("#############################");
    }
    
}
```

解决方式druid，构造了QueryInterruptedException来包装异常，捕获它，然后打印其包装异常的调用栈：

```java
package com.btrace.druid;

import org.openjdk.btrace.core.types.AnyType;
import org.openjdk.btrace.core.annotations.*;

import static org.openjdk.btrace.core.BTraceUtils.*;

@BTrace
public class QueryInterruptedExceptionBtrace {
    @OnMethod(clazz = "org.apache.druid.query.QueryInterruptedException", method = "<init>")
    public static void doPollEntry0(Throwable s){
        jstack(s); // s异常的调用栈
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":s:" + s);
        jstack(); // QueryInterruptedException构造函数的调用栈
        println("#############################");
    }
    

    @OnMethod(clazz = "org.apache.druid.query.QueryInterruptedException", method = "<init>")
    public static void doPollEntry2(AnyType s, AnyType s2){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":s:" + s + ",s2:" + s2);
        jstack();
        println("#############################");
    }
    
    @OnMethod(clazz = "org.apache.druid.query.QueryInterruptedException", method = "<init>")
    public static void doPollEntry3(AnyType s, AnyType s1, AnyType s2){
        println(timestamp("yyyy-MM-dd HH:mm:ss.SSS") + ":s:" + s + ":s1:" + s1 + ",s2:" + s2);
        jstack();
        println("#############################");
    }
    
}
```

输出：

```
java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.Long (java.lang.Integer and java.lang.Long are in module java.base of loader 'bootstrap')

        org.apache.druid.query.aggregation.sketch.HdrHistogram.HdrHistogramToQuantilePostAggregator$1.compare(HdrHistogramToQuantilePostAggregator.java:53)
        org.apache.druid.query.topn.TopNNumericResultBuilder.lambda$new$0(TopNNumericResultBuilder.java:96)
        java.base/java.util.PriorityQueue.siftUpUsingComparator(PriorityQueue.java:675)
        java.base/java.util.PriorityQueue.siftUp(PriorityQueue.java:652)
        java.base/java.util.PriorityQueue.offer(PriorityQueue.java:345)
        java.base/java.util.PriorityQueue.add(PriorityQueue.java:326)
        org.apache.druid.query.topn.TopNNumericResultBuilder.addEntry(TopNNumericResultBuilder.java:209)
        org.apache.druid.query.topn.TopNBinaryFn.apply(TopNBinaryFn.java:133)
        org.apache.druid.query.topn.TopNBinaryFn.apply(TopNBinaryFn.java:40)
        org.apache.druid.java.util.common.guava.ParallelMergeCombiningSequence$MergeCombineAction.compute(ParallelMergeCombiningSequence.java:584)
        java.base/java.util.concurrent.RecursiveAction.exec(RecursiveAction.java:189)
        java.base/java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:290)
        java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(ForkJoinPool.java:1020)
        java.base/java.util.concurrent.ForkJoinPool.scan(ForkJoinPool.java:1656)
        java.base/java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1594)
        java.base/java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:183)

2024-08-21 09:30:50.391:s:java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.Long (java.lang.Integer and java.lang.Long are in module java.base of loader 'bootstrap')

org.apache.druid.query.QueryInterruptedException.<init>(QueryInterruptedException.java:64)
org.apache.druid.server.QueryResultPusher.push(QueryResultPusher.java:166)
org.apache.druid.sql.http.SqlResource.doPost(SqlResource.java:125)
jdk.internal.reflect.GeneratedMethodAccessor99.invoke(Unknown Source)
java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
java.base/java.lang.reflect.Method.invoke(Method.java:566)
com.sun.jersey.spi.container.JavaMethodInvokerFactory$1.invoke(JavaMethodInvokerFactory.java:60)
com.sun.jersey.server.impl.model.method.dispatch.AbstractResourceMethodDispatchProvider$ResponseOutInvoker._dispatch(AbstractResourceMethodDispatchProvider.java:205)
com.sun.jersey.server.impl.model.method.dispatch.ResourceJavaMethodDispatcher.dispatch(ResourceJavaMethodDispatcher.java:75)
com.sun.jersey.server.impl.uri.rules.HttpMethodRule.accept(HttpMethodRule.java:302)
com.sun.jersey.server.impl.uri.rules.ResourceClassRule.accept(ResourceClassRule.java:108)
com.sun.jersey.server.impl.uri.rules.RightHandPathRule.accept(RightHandPathRule.java:147)
com.sun.jersey.server.impl.uri.rules.RootResourceClassesRule.accept(RootResourceClassesRule.java:84)
com.sun.jersey.server.impl.application.WebApplicationImpl._handleRequest(WebApplicationImpl.java:1542)
com.sun.jersey.server.impl.application.WebApplicationImpl._handleRequest(WebApplicationImpl.java:1473)
com.sun.jersey.server.impl.application.WebApplicationImpl.handleRequest(WebApplicationImpl.java:1419)
com.sun.jersey.server.impl.application.WebApplicationImpl.handleRequest(WebApplicationImpl.java:1409)
com.sun.jersey.spi.container.servlet.WebComponent.service(WebComponent.java:409)
com.sun.jersey.spi.container.servlet.ServletContainer.service(ServletContainer.java:558)
com.sun.jersey.spi.container.servlet.ServletContainer.service(ServletContainer.java:733)
javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
```








