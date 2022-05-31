# Java Redis Pipeline 使用示例

摘抄自：https://www.cnblogs.com/panchanggui/p/9878912.html

## 1. 参考的优秀文章


* [Request/Response protocols and RTT](http://redis.io/topics/pipelining "Request/Response protocols and RTT")




## 2. 来源


原来，系统中一个树结构的数据来源是Redis，由于数据增多、业务复杂，查询速度并不快。究其原因，是单次查询的数量太多了，一个树结构，大概要几万次Redis的交互。于是，尝试用Redis的Pipelining特性。




## 3. 测试Pipelining使用与否的差别




### 3.1. 不使用pipelining


首先，不使用pipelining，插入10w条记录，再删除10w条记录，看看需要多久。


首先来个小程序，用于计算程序消耗的时间：


```java
import java.util.Date;
import java.util.concurrent.TimeUnit;
 
 
public class TimeLag {
     
    private Date start;
    private Date end;
     
    public TimeLag() {
        start = new Date();
    }
     
    public String cost() {
        end = new Date();
        long c = end.getTime() - start.getTime();
         
        String s = new StringBuffer().append("cost ").append(c).append(" milliseconds (").append(c / 1000).append(" seconds).").toString();
        return s;
    }
     
    public static void main(String[] args) throws InterruptedException {
        TimeLag t = new TimeLag();
        TimeUnit.SECONDS.sleep(2);
        System.out.println(t.cost());
    }
 
}
```


```java
package com.nicchagil.study.jedis;
 
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
 
 
public class HowToTest {
 
    public static void main(String[] args) {
        // 连接池
        JedisPool jedisPool = new JedisPool("192.168.1.9", 6379);
         
        /* 操作Redis */
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
             
            TimeLag t = new TimeLag();
            System.out.println("操作前，全部Key值：" + jedis.keys("*"));
            /* 插入多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                jedis.set(i.toString(), i.toString());
            }
             
            /* 删除多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                jedis.del(i.toString());
            }
            System.out.println("操作前，全部Key值：" + jedis.keys("*"));
            System.out.println(t.cost());
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }
 
}
```


日志，Key值“user_001”是我的Redis存量的值，忽略即可：

```java
操作前，全部Key值：[user_001]
操作前，全部Key值：[user_001]
cost 35997 milliseconds (35 seconds).
```


### 3.2. 使用pipelining


```java
package com.nicchagil.study.jedis;
 
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Pipeline;
 
 
public class HowToTest {
 
    public static void main(String[] args) {
        // 连接池
        JedisPool jedisPool = new JedisPool("192.168.1.9", 6379);
         
        /* 操作Redis */
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
             
            TimeLag t = new TimeLag();
             
            System.out.println("操作前，全部Key值：" + jedis.keys("*"));
            Pipeline p = jedis.pipelined();
            /* 插入多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                p.set(i.toString(), i.toString());
            }
             
            /* 删除多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                p.del(i.toString());
            }
            p.sync();
            System.out.println("操作前，全部Key值：" + jedis.keys("*"));
             
            System.out.println(t.cost());
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }
 
}
```




日志：

```java
操作前，全部Key值：[user_001]
操作前，全部Key值：[user_001]
cost 629 milliseconds (0 seconds).
```


## 4. 为什么Pipelining这么快？


先看看原来的多条命令，是如何执行的：

```java
sequenceDiagram
Redis Client->>Redis Server: 发送第1个命令
Redis Server->>Redis Client: 响应第1个命令
Redis Client->>Redis Server: 发送第2个命令
Redis Server->>Redis Client: 响应第2个命令
Redis Client->>Redis Server: 发送第n个命令
Redis Server->>Redis Client: 响应第n个命令
```




Pipeling机制是怎样的呢：


```java
sequenceDiagram
Redis Client->>Redis Server: 发送第1个命令（缓存在Redis Client，未即时发送）
Redis Client->>Redis Server: 发送第2个命令（缓存在Redis Client，未即时发送）
Redis Client->>Redis Server: 发送第n个命令（缓存在Redis Client，未即时发送）
Redis Client->>Redis Server: 发送累积的命令
Redis Server->>Redis Client: 响应第1、2、n个命令
```



## 5. Pipelining的局限性（重要！）


基于其特性，它有两个明显的局限性：


* 鉴于Pipepining发送命令的特性，Redis服务器是以队列来存储准备执行的命令，而队列是存放在有限的内存中的，所以不宜一次性发送过多的命令。如果需要大量的命令，可分批进行，效率不会相差太远滴，总好过内存溢出嘛~~    
* 由于pipeline的原理是收集需执行的命令，到最后才一次性执行。所以无法在中途立即查得数据的结果（需待pipelining完毕后才能查得结果），这样会使得无法立即查得数据进行条件判断（比如判断是非继续插入记录）。


比如，以下代码中，`response.get()`在`p.sync();`完毕前无法执行，否则，会报异常

```java
redis.clients.jedis.exceptions.JedisDataException: Please close pipeline or multi block before calling this method.

```

```java
package com.nicchagil.study.jedis;
 
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Response;
 
 
public class HowToTest {
 
    public static void main(String[] args) {
        // 连接池
        JedisPool jedisPool = new JedisPool("192.168.1.9", 6379);
         
        /* 操作Redis */
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
             
            TimeLag t = new TimeLag();
             
            System.out.println("操作前，全部Key值：" + jedis.keys("*"));
            Pipeline p = jedis.pipelined();
            /* 插入多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                p.set(i.toString(), i.toString());
            }
             
            Response<String> response = p.get("999");
            // System.out.println(response.get()); // 执行报异常：redis.clients.jedis.exceptions.JedisDataException: Please close pipeline or multi block before calling this method.
             
            /* 删除多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                p.del(i.toString());
            }
            p.sync();
             
            System.out.println(response.get());
            System.out.println("操作前，全部Key值：" + jedis.keys("*"));
             
            System.out.println(t.cost());
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
         
    }
}
```



## 6. 如何使用Pipelining查询大量数据


用`Map<String, Response<String>>`先将`Response`缓存起来再使用就OK了：


```java
package com.nicchagil.study.jedis;
 
import java.util.HashMap;
import java.util.Map;
 
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Response;
 
 
public class GetMultiRecordWithPipelining {
 
    public static void main(String[] args) {
        // 连接池
        JedisPool jedisPool = new JedisPool("192.168.1.9", 6379);
         
        /* 操作Redis */
        Jedis jedis = null;
        Map<String, Response<String>> map = new HashMap<String, Response<String>>();
        try {
            jedis = jedisPool.getResource();
             
            TimeLag t = new TimeLag(); // 开始计算时间
             
            Pipeline p = jedis.pipelined();
            /* 插入多条数据 */
            for(Integer i = 0; i < 100000; i++) {
                if (i % 2 == 1) {
                    map.put(i.toString(), p.get(i.toString()));
                }
            }
            p.sync();
             
            /* 由Response对象获取对应的值 */
            Map<String, String> resultMap = new HashMap<String, String>();
            String result = null;
            for (String key : map.keySet()) {
                result = map.get(key).get();
                if (result != null && result.length() > 0) {
                    resultMap.put(key, result);
                }
            }
            System.out.println("get record num : " + resultMap.size());
             
            System.out.println(t.cost()); // 计时结束
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
         
    }
}
```

