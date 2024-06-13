
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










