# EhCache2
在互联网应用中，数据存储和访问通常有两个地方：DB和缓存。

1. 各自的优缺点：DB属于持久化存储，缓存属于非持久化存储（有过期时间）；缓存相对DB来说，插入和访问的速度要快很多。其中缓存又分为本地缓存（例如ehcache）和网络缓存（例如redis）。

 
2. 它们三者的访问速度比较：ehcache > redis > DB。ehcache的特点是缓存容量小，但是存取速度快（没有网络传输消耗）；redis缓存容量大，存取速度次之（因为redis缓存服务器通常单独部署在应用主机以外）；DB存取速度最慢。

## 简介
EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认CacheProvider。Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存,Java EE和轻量级容器。它具有内存和磁盘存储，缓存加载器,缓存扩展,缓存异常处理程序,一个gzip缓存servlet过滤器,支持REST和SOAP api等特点。

### 主要的特性有

* 快速    
* 简单    
* 多种缓存策略    
* 缓存数据有两级：内存和磁盘，因此无需担心容量问题    
* 缓存数据会在虚拟机重启的过程中写入磁盘    
* 可以通过RMI、可插入API等方式进行分布式缓存    
* 具有缓存和缓存管理器的侦听接口    
* 支持多缓存管理器实例，以及一个实例的多个缓存区域    
* 提供Hibernate的缓存实现  

### 集成
可以单独使用，一般在第三方库中被用到的比较多（如mybatis、shiro等）ehcache 对分布式支持不够好，多个节点不能同步，通常和redis一块使用

### 灵活性
ehcache具备**对象api接口**和**可序列化api接口**

不能序列化的对象可以使用出磁盘存储外ehcache的所有功能

支持基于Cache和基于Element的过期策略，每个Cache的存活时间都是可以设置和控制的。

提供了LRU、LFU和FIFO缓存淘汰算法，Ehcache 1.2引入了最少使用和先进先出缓存淘汰算法，构成了完整的缓存淘汰算法。

提供内存和磁盘存储，Ehcache和大多数缓存解决方案一样，提供高性能的内存和磁盘存储。

动态、运行时缓存配置，存活时间、空闲时间、内存和磁盘存放缓存的最大数目都是可以在运行时修改的。

### 应用持久化
在jvm重启后，持久化到磁盘的存储可以复原数据

Ehache是第一个引入缓存数据持久化存储的开源java缓存框架，缓存的数据可以在机器重启后从磁盘上重新获得

根据需要将缓存刷到磁盘。将缓存条目刷到磁盘的操作可以通过cache.fiush方法执行,这大大方便了ehcache的使用

### ehcache 和 redis 比较
* ehcache直接在jvm虚拟机中缓存，速度快，效率高；但是缓存共享麻烦，集群分布式应用不方便。    
* redis是通过socket访问到缓存服务，效率比ecache低，比数据库要快很多，处理集群和分布式缓存方便，有成熟的方案。如果是单个应用或者对缓存访问要求很高的应用，用ehcache。如果是大型系统，存在缓存共享、分布式部署、缓存内容很大的，建议用redis。  

ehcache也有缓存共享方案，不过是通过RMI或者Jgroup多播方式进行广播缓存通知更新，缓存共享复杂，维护不方便；简单的共享可以，但是涉及到缓存恢复，大数据缓存，则不合适。

## 引入依赖
```xml
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.6</version>
</dependency>
```

## Hello World

### 配置文件
ehcache.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
    <!--
          磁盘存储:将缓存中暂时不使用的对象,转移到硬盘,类似于Windows系统的虚拟内存
           path:指定在硬盘上存储对象的路径
    -->
    <diskStore path="F:\ehcache" />

  <!-- 默认缓存 -->
  <defaultCache
          maxEntriesLocalHeap="10000"
          eternal="false"
          timeToIdleSeconds="120"
          timeToLiveSeconds="120"
          maxEntriesLocalDisk="10000000"
          diskExpiryThreadIntervalSeconds="120"
          memoryStoreEvictionPolicy="LRU">
    <persistence strategy="localTempSwap"/>
  </defaultCache>

  <!-- helloworld缓存 -->
  <cache name="HelloWorldCache"
         maxElementsInMemory="1000"
         eternal="false"
         timeToIdleSeconds="5"
         timeToLiveSeconds="5"
         overflowToDisk="false"
         memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```

### 测试类
```java
package com.zyc;

import org.junit.Test;

import net.sf.ehcache.Cache;
import net.sf.ehcache.CacheManager;
import net.sf.ehcache.Element;

public class Test1 {
    
    @Test
    public void test1() {
        // 1. 创建缓存管理器
        String path = EhcacheTest.class.getClassLoader().getResource("ehcache.xml").getPath();
        CacheManager cacheManager = CacheManager.create(path);
        
        // 2. 获取缓存对象
        Cache cache = cacheManager.getCache("HelloWorldCache");
        
        // 3. 创建元素
        Element element = new Element("key1", "value1");
        
        // 4. 将元素添加到缓存
        cache.put(element);
        
        // 5. 获取缓存
        Element value = cache.get("key1");
        System.out.println(value);
        System.out.println(value.getObjectValue());
        
        // 6. 删除元素
        cache.remove("key1");
        
        Person p1 = new Person("小明",18,"杭州");
        Element pelement = new Element("xm", p1);
        cache.put(pelement);
        Element pelement2 = cache.get("xm");
        System.out.println(pelement2.getObjectValue());
        
        System.out.println(cache.getSize());
        
        // 7. 刷新缓存
        cache.flush();
        
        // 8. 关闭缓存管理器
        cacheManager.shutdown();

    }

}
```

## 缓存配置
### xml配置方式
#### diskStore 
path ：指定磁盘存储的位置

#### defaultCache
默认的缓存

* maxEntriesLocalHeap=“10000”    
* eternal=“false”    
* timeToIdleSeconds=“120”    
* timeToLiveSeconds=“120”    
* maxEntriesLocalDisk=“10000000”    
* diskExpiryThreadIntervalSeconds=“120”    
* memoryStoreEvictionPolicy=“LRU”   


#### cache
自定义的缓存，当自定的配置不满足实际情况时可以通过自定义（可以包含多个cache节点）

* name : 缓存的名称，可以通过指定名称获取指定的某个Cache对象    
* maxElementsInMemory ：内存中允许存储的最大的元素个数，0代表无限个    
* clearOnFlush：内存数量最大时是否清除。    
* eternal ：设置缓存中对象是否为永久的，如果是，超时设置将被忽略，对象从不过期。根据存储数据的不同，例如一些静态不变的数据如省市区等可以设置为永不过时    
* timeToIdleSeconds ： 设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。    
* timeToLiveSeconds ：缓存数据的生存时间（TTL），也就是一个元素从构建到消亡的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是0就意味着元素可以停顿无穷长的时间。    
* overflowToDisk ：内存不足时，是否启用磁盘缓存。    
* maxEntriesLocalDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。    
* maxElementsOnDisk：硬盘最大缓存个数。    
* diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。    
* diskPersistent：是否在VM重启时存储硬盘的缓存数据。默认值是false。    
* diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。  
* memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是**LRU**（最近最少使用）。你可以设置为**FIFO**（先进先出）或是**LFU**（较少使用）。  

### 编程方式配置
```java
Cache cache = manager.getCache("mycache"); 
CacheConfiguration config = cache.getCacheConfiguration(); 
config.setTimeToIdleSeconds(60); 
config.setTimeToLiveSeconds(120); 
config.setmaxEntriesLocalHeap(10000); 
config.setmaxEntriesLocalDisk(1000000);
```

### 持久化配置
Ehcache默认配置的话 为了提高效率，所以有一部分缓存是在内存中，然后达到配置的内存对象总量，则才根据策略持久化到硬盘中，这里是有一个问题的，假如系统突然中断运行 那内存中的那些缓存，直接被释放掉了，不能持久化到硬盘；这种数据丢失，对于一般项目是不会有影响的，但是对于我们的爬虫系统，我们是用来判断重复Url的，所以数据不能丢失；

这时候我们就需要通过Ehcache配置，来实现缓存的持久化，不存内存中。

类必须实现序列化接口，不需要的属性用transientx修饰。
```xml
<!-- 
    maxElementsInMemory设置成1，overflowToDisk设置成true，只要有一个缓存元素，就直接存到硬盘上去
    eternal设置成true，代表对象永久有效
    maxElementsOnDisk设置成0 表示硬盘中最大缓存对象数无限大
    diskPersistent设置成true表示缓存虚拟机重启期数据 
 -->
<cache 
  name="a"
  maxElementsInMemory="1" 
  eternal="true"
  overflowToDisk="true" 
  maxElementsOnDisk="0"
  diskPersistent="true"/>
```

**diskPersistent:只有设置为true时jvm关闭时才会刷新数据到磁盘，重启时会读取。不设置就算数据溢出存到磁盘上，jvm退出时会删除数据。**

#### 持久化代码
需要调用shutdown将内存中的数据放到磁盘上
```java
manager.shutdown(); // 关闭缓存管理器,diskPersistent="true"持久化所有数据到磁盘
```

#### 自动持久化
想利用spring 的注解，不想手动shutdown ，因此web.xml 配置listener 监听，在销毁的时候进行shutdown,这里利用ehcache 的监听.
```xml
<!-- ehcache 磁盘缓存 监控，持久化恢复 -->
<listener>
    <listener-class>net.sf.ehcache.constructs.web.ShutdownListener</listener-class>
</listener>
```

## 测试
### 基本功能测试
ehcache.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
    <!--
          磁盘存储:将缓存中暂时不使用的对象,转移到硬盘,类似于Windows系统的虚拟内存
           path:指定在硬盘上存储对象的路径
    -->
    <diskStore path="F:\ehcache" />

    <!--
         defaultCache:默认的缓存配置信息,如果不加特殊说明,则所有对象按照此配置项处理
         maxElementsInMemory:设置了缓存的上限,最多存储多少个记录对象
         eternal:代表对象是否永不过期
         overflowToDisk:当内存中Element数量达到maxElementsInMemory时，Ehcache将会Element写到磁盘中
    -->
    <defaultCache
            maxElementsInMemory="100"
            eternal="true"
            overflowToDisk="true"/>

    <!--
             diskPersistent:只有设置为true时jvm关闭时才会刷新数据到磁盘，重启时会读取。不设置就算数据溢出存到磁盘上，jvm退出时会删除数据。
    -->
    <cache
            name="cach01"
            maxElementsInMemory="100"
            eternal="true"
            overflowToDisk="true"
            diskPersistent="true"
    />

    <!--
        Ehcache默认配置的话 为了提高效率，所以有一部分缓存是在内存中，然后达到配置的内存对象总量，则才根据策略持久化到硬盘中，
        这里是有一个问题的，假如系统突然中断运行 那内存中的那些缓存，直接被释放掉了，不能持久化到硬盘；
        这种数据丢失，对于一般项目是不会有影响的，但是对于我们的爬虫系统，我们是用来判断重复Url的，所以数据不能丢失；
        这时候我们就需要通过Ehcache配置，来实现缓存的持久化，不存内存中。

        maxElementsInMemory设置成1，overflowToDisk设置成true，只要有一个缓存元素，就直接存到硬盘上去
        eternal设置成true，代表对象永久有效
        maxElementsOnDisk设置成0 表示硬盘中最大缓存对象数无限大
        diskPersistent设置成true表示缓存虚拟机重启期数据
     -->
    <cache
            name="cach02"
            maxElementsInMemory="1"
            eternal="true"
            overflowToDisk="true"
            maxElementsOnDisk="0"
            diskPersistent="true"/>

</ehcache>
```

测试代码：
```java
import net.sf.ehcache.Cache;
import net.sf.ehcache.CacheManager;
import net.sf.ehcache.Element;

public class EhcacheTest {

    public static void main(String[] args) {
        // 根据ehcache.xml配置文件创建Cache管理器
        String path = EhcacheTest.class.getClassLoader().getResource("ehcache.xml").getPath();
        CacheManager manager = CacheManager.create(path);
        Cache c = manager.getCache("cach01"); // 获取指定Cache
        Element e = new Element("key01", "value01"); // 实例化一个元素
        c.put(e); // 把一个元素添加到Cache中

        Element e2 = c.get("key01"); // 根据Key获取缓存元素
        System.out.println(e2);
        //System.out.println(e2.getObjectValue()); // 获取Element的value


        //c.flush(); // 刷新缓存
        manager.shutdown(); // 关闭缓存管理器,diskPersistent="true"持久化所有数据到磁盘
    }
}
```

### EhcacheUtil工具类
```java
public class EhcacheUtil {

	private static final String path = "/ehcache.xml";

	private URL url;

	private CacheManager manager;

	private static EhcacheUtil ehCache;

	private EhcacheUtil(String path) {
		url = getClass().getResource(path);
		manager = CacheManager.create(url);
	}

	public static EhcacheUtil getInstance() {
		if (ehCache== null) {
			ehCache= new EhcacheUtil(path);
		}
		return ehCache;
	}

	public void put(String cacheName, String key, Object value) {
		Cache cache = manager.getCache(cacheName);
		Element element = new Element(key, value);
		cache.put(element);
	}

	public Object get(String cacheName, String key) {
		Cache cache = manager.getCache(cacheName);
		Element element = cache.get(key);
		return element == null ? null : element.getObjectValue();
	}

	public Cache get(String cacheName) {
		return manager.getCache(cacheName);
	}

	public void remove(String cacheName, String key) {
		Cache cache = manager.getCache(cacheName);
		cache.remove(key);
	}

}
```




```

```



