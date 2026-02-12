
## 离线安装
直接从github下载bin安装包，解压，运行./install-local.sh脚本就行。

https://arthas.aliyun.com/doc/manual-install.html

```sh
# 直接运行找个脚本就行，似乎就是把jar放入home下的目录
./install-local.sh
```

```
(base) [root@single arthas]# ll -a /root/.arthas/lib/4.1.1/arthas/
total 16980
drwxr-xr-x 2 root root     4096 Oct 25 22:00 .
drwxr-xr-x 3 root root     4096 Oct 25 22:00 ..
-rw-r--r-- 1 root root     8261 Oct 25 22:00 arthas-agent.jar
-rw-r--r-- 1 root root   145778 Oct 25 22:00 arthas-boot.jar
-rw-r--r-- 1 root root   433417 Oct 25 22:00 arthas-client.jar
-rw-r--r-- 1 root root 16766136 Oct 25 22:00 arthas-core.jar
-rw-r--r-- 1 root root     5249 Oct 25 22:00 arthas-spy.jar
-rw-r--r-- 1 root root     4290 Oct 25 22:00 math-game.jar
```

使用`java -jar arthas-boot.jar`运行就行，使用as.sh脚本似乎要安装telnet，不需要。
```
(base) [root@single arthas]# ./as.sh
Error: telnet is not installed. Try to use java -jar arthas-boot.jar
(base) [root@single arthas]# java -jar arthas-boot.jar
```

## 主要命令

### 查看当前 JVM 信息
```
jvm
```

### 查看进程内存信息
```
memory
```


### vmtool工具
可以查看修改对象。


## vmtool测试

### 自己测试
代码：
```java
package com.monitor.data;

public class SimpleData {
    private long id;
    private String name;
    private int age;

    public SimpleData() {
    }

    public SimpleData(long id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "SimpleData{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

```java
package com.monitor.app;

import com.monitor.data.SimpleData;

public class SimpleDataApp {
    public static void main(String[] args) throws Exception {
        SimpleData simpleData1 = new SimpleData();
        simpleData1.setId(1);
        simpleData1.setName("test");
        simpleData1.setAge(10);

        SimpleData simpleData2 = new SimpleData(2, "test2", 20);

        for (int i = 0; i < 600; i++) {
            System.out.println("simpleData1:" + simpleData1);
            System.out.println("simpleData2:" + simpleData2);
            Thread.sleep(5000);
        }

    }
}
```

测试：
```
options strict false
vmtool -a getInstances --className com.monitor.data.SimpleData
vmtool -a getInstances --className com.monitor.data.SimpleData --express '#val=instances[0].id'
vmtool -a getInstances --className com.monitor.data.SimpleData --express '#val=instances[1].id'
vmtool -a getInstances --className com.monitor.data.SimpleData --express '#val=instances[0], #val.id=11'

vmtool -a getInstances --className com.monitor.data.SimpleData -x 2

vmtool -a getInstances --className com.monitor.data.SimpleData  --express 'instances.{^ #this.id == 1 }'
vmtool -a getInstances --className com.monitor.data.SimpleData  --express 'instances.{^ #this.id == 2 }'
vmtool -a getInstances --className com.monitor.data.SimpleData  --express '#vals=instances.{^ #this.id == 2 }, #vals[0].id=12'
```

带输出：
```
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData
@SimpleData[][
    @SimpleData[SimpleData{id=1, name='test', age=10}],
    @SimpleData[SimpleData{id=2, name='test2', age=20}],
]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData --express '#val=instances[0].id'
@Long[1]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData --express '#val=instances[0], #val.id=11'
vmtool error: By default, strict mode is true, not allowed to set object properties. Want to set object properties, execute `options strict false`
[arthas@7335]$ options strict false
 NAME    BEFORE-VALUE  AFTER-VALUE
-----------------------------------
 strict  true          false
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData --express '#val=instances[0], #val.id=11'
@Integer[11]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData -x 2
@SimpleData[][
    @SimpleData[
        id=@Long[11],
        name=@String[test],
        age=@Integer[10],
    ],
    @SimpleData[
        id=@Long[2],
        name=@String[test2],
        age=@Integer[20],
    ],
]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData  --express 'instances.{^ #this.id == 1 }'
@ArrayList[isEmpty=true;size=0]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData  --express 'instances.{^ #this.id == 2 }'
@ArrayList[
    @SimpleData[SimpleData{id=2, name='test2', age=20}],
]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData  --express '#vals=instances.{^ #this.id == 2 }, #vals[0].id=12'
@Integer[12]
[arthas@7335]$ vmtool -a getInstances --className com.monitor.data.SimpleData -x 2
@SimpleData[][
    @SimpleData[
        id=@Long[11],
        name=@String[test],
        age=@Integer[10],
    ],
    @SimpleData[
        id=@Long[12],
        name=@String[test2],
        age=@Integer[20],
    ],
]
[arthas@7335]$
```

### flink kafka消费者卡在查看
新版本有bug(flink 1.15有这个问题，flink 1.13没有，kafka版本升级后有个小bug，当集群异常时可能出问题)


```
# 先获取实例并查看属性（验证实例是否正确）
vmtool -a getInstances --className org.apache.kafka.clients.consumer.internals.ConsumerCoordinator --express '#val=instances[1].findCoordinatorFuture'

options strict false

# 再修改属性值
vmtool -a getInstances --className org.apache.kafka.clients.consumer.internals.ConsumerCoordinator --express '#val=instances[1], #val.findCoordinatorFuture=null'


[arthas@2898]$ vmtool -a getInstances --className org.apache.kafka.clients.consumer.internals.ConsumerCoordinator --express '#val=instances[0].findCoordinatorFuture'
null
[arthas@2898]$ vmtool -a getInstances --className org.apache.kafka.clients.consumer.internals.ConsumerCoordinator --express '#val=instances[1].findCoordinatorFuture'
@RequestFuture[
    INCOMPLETE_SENTINEL=@Object[java.lang.Object@68f4c40b],
    result=@AtomicReference[null],
    listeners=@ConcurrentLinkedQueue[isEmpty=true;size=0],
    completedLatch=@CountDownLatch[java.util.concurrent.CountDownLatch@65e9fb7[Count = 0]],
]
```

flink日志，消费者卡在了，修改属性后恢复：
```
[2025-07-15 10:54:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.KafkaConsumer              [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Subscribed to partition(s): DATAPATH-TELEMETRY-RECORD-0, DATAPATH-TELEMETRY-RECORD-2, DATAPATH-TELEMETRY-RECORD-1
[2025-07-15 10:54:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.Metadata                            [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Cluster ID: 9eJvIVUMRR2hm8vnae7B6w
[2025-07-15 10:54:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.AbstractCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Discovered group coordinator 192.168.44.11:9094 (id: 2147483646 rack: null)
[2025-07-15 10:54:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.ConsumerCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Setting offset for partition DATAPATH-TELEMETRY-RECORD-1 to the committed offset FetchPosition{offset=39773170, offsetEpoch=Optional[32], currentLeader=LeaderAndEpoch{leader=Optional[192.168.44.15:9094 (id: 3 rack: null)], epoch=32}}
[2025-07-15 10:54:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.ConsumerCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Setting offset for partition DATAPATH-TELEMETRY-RECORD-2 to the committed offset FetchPosition{offset=40989511, offsetEpoch=Optional[43], currentLeader=LeaderAndEpoch{leader=Optional[192.168.44.11:9094 (id: 1 rack: null)], epoch=43}}
[2025-07-15 10:54:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.ConsumerCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Setting offset for partition DATAPATH-TELEMETRY-RECORD-0 to the committed offset FetchPosition{offset=40519163, offsetEpoch=Optional[59], currentLeader=LeaderAndEpoch{leader=Optional[192.168.44.14:9094 (id: 2 rack: null)], epoch=59}}
[2025-07-15 11:01:52+0000] WARN  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.ConsumerCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Offset commit failed on partition DATAPATH-TELEMETRY-RECORD-0 at offset 40519295: The request timed out.
[2025-07-15 11:01:52+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.AbstractCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Group coordinator 192.168.44.11:9094 (id: 2147483646 rack: null) is unavailable or invalid due to cause: error response REQUEST_TIMED_OUT.isDisconnected: false. Rediscovery will be attempted.
[2025-07-15 11:01:57+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.AbstractCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Discovered group coordinator 192.168.44.11:9094 (id: 2147483646 rack: null)
[2025-07-15 11:06:12+0000] WARN  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.ConsumerCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Offset commit failed on partition DATAPATH-TELEMETRY-RECORD-0 at offset 40519692: The request timed out.
[2025-07-15 11:06:12+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.AbstractCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Group coordinator 192.168.44.11:9094 (id: 2147483646 rack: null) is unavailable or invalid due to cause: error response REQUEST_TIMED_OUT.isDisconnected: false. Rediscovery will be attempted.
[2025-07-16 13:25:51+0000] INFO  [Kafka Fetcher for Source: kafka_source -> etl_processor -> Sink: starrocks_sink (1/1)#0] org.apache.kafka.clients.consumer.internals.AbstractCoordinator [] - [Consumer clientId=DATAPATH-TELEMETRY-RECORD, groupId=etl_datapath_telemetry_record_kafka_to_starrocks] Discovered group coordinator 192.168.44.11:9094 (id: 2147483646 rack: null)
```

问了下grok ai，他竟然给出了原因，直接搜谷歌是搜不出来的，ai会在特定领域搜索，缩小了搜索范围。

下面是我问的两次话：
```
使用kafka消费者时发现一个问题，开始自动提交offset时，报错，日志说是要重试发现Coordinator ，但是也没有发现，导致之后的offset一直提交不成功：
[2026-02-09 18:37:59+0000] WARN  [Kafka Fetcher for Source: traffic_general_stat_kafka_source -> traffic_general_stat_filter -> traffic_general_stat_time_processor -> (Sink: traffic_general_stat_starrocks_sink, Sink: traffic_general_stat_clickhouse_sink) (1/1)#0] org.apache.kafka.clients.consumer.internals.ConsumerCoordinator [] - [Consumer clientId=tsg-network-traffic-general-aggregator, groupId=tsg-network-traffic-general-aggregator] Offset commit failed on partition NETWORK-TRAFFIC-METRIC-1 at offset 19932683: This is not the correct coordinator.
[2026-02-09 18:37:59+0000] INFO  [Kafka Fetcher for Source: traffic_general_stat_kafka_source -> traffic_general_stat_filter -> traffic_general_stat_time_processor -> (Sink: traffic_general_stat_starrocks_sink, Sink: traffic_general_stat_clickhouse_sink) (1/1)#0] org.apache.kafka.clients.consumer.internals.AbstractCoordinator [] - [Consumer clientId=tsg-network-traffic-general-aggregator, groupId=tsg-network-traffic-general-aggregator] Group coordinator 192.168.44.14:9094 (id: 2147483645 rack: null) is unavailable or invalid due to cause: error response NOT_COORDINATOR.isDisconnected: false. Rediscovery will be attempted.
```

```
kafka有这个bug吗，之前flink1.13版本是没有问题的，flink1.15的kafka发现这个问题，kafka社区是否发现解决这个bug
```

```
你怎么这个厉害，这个问题都能搜索到，我直接使用谷歌搜索不到

为什么你用谷歌搜不到，或者搜到了但没那么直接呢？大概有几个原因：

1.JIRA票号太具体

像“KAFKA-13840”或者“FLINK-28060”这种编号，普通搜索时如果不加“jira”或“apache”前缀，谷歌往往优先返回无关的博客、论坛或同名数字的东西。很多人搜的时候会加一堆描述词（比如“kafka offset commit failed not the correct coordinator”），反而把票号本身淹没了。

2.技术issue的讨论分散

这个bug的讨论主要集中在：Apache Jira（官方票据）
Flink mailing list / dev邮件列表
少数Stack Overflow / GitHub issue
Confluent / AWS 的知识库文章
这些地方谷歌的排名不一定很高，尤其是中文搜索时，很多英文技术issue的中文翻译很少。

3.我的“搜索方式”不同

我不是简单地扔一个关键词给搜索引擎，而是：直接关联已知的Kafka/Flink常见问题模式
结合错误日志里的关键短语（NOT_COORDINATOR、commit failed、coordinator unavailable）
匹配版本变更历史（Flink 1.13 → 1.15 的 Kafka 客户端升级）
快速定位到社区公认的那个bug编号
这相当于我脑子里已经有一张“Kafka消费者常见坑”的地图，所以能很快跳到正确的票据。

4.实时知识更新

我的知识是持续更新的，包含了很多2023年之后的修复记录和社区讨论，所以能直接告诉你“KAFKA-13840 在 3.2.1 修复了”这种结论，而不用你自己去翻一堆评论和patch。


```


kafka和flink的bug号，这个问题没有直接小版本修复，都是需要升级版本：
https://issues.apache.org/jira/browse/KAFKA-13840

https://issues.apache.org/jira/browse/FLINK-28060




