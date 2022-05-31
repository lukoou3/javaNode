# 对于Redis中设置了过期时间的Key，你需要知道这些内容

摘抄自：https://blog.csdn.net/lxw1844912514/article/details/106110836


熟悉Redis的同学应该知道，Redis的每个Key都可以设置一个过期时间，当达到过期时间的时候，这个key就会被自动删除。


# 在为key设置过期时间需要注意的事项


## 1、DEL/SET/GETSET等命令会清除过期时间


在使用**DEL、SET、GETSET**等会覆盖key对应value的命令操作一个设置了过期时间的key的时候，会导致对应的key的过期时间被清除。

```java
//设置mykey的过期时间为300s127.0.0.1:6379> set mykey hello ex 300OK//查看过期时间127.0.0.1:6379> ttl mykey(integer) 294//使用set命令覆盖mykey的内容127.0.0.1:6379> set mykey ollehOK//过期时间被清除127.0.0.1:6379> ttl mykey(integer) -1
```

## 2、INCR/LPUSH/HSET等命令则不会清除过期时间


而在使用**INCR/LPUSH/HSET**这种只是修改一个key的value，而不是覆盖整个value的命令，则不会清除key的过期时间。


**INCR**：

```java
//设置incr_key的过期时间为300s127.0.0.1:6379> set incr_key 1 ex 300OK127.0.0.1:6379> ttl incr_key(integer) 291//进行自增操作127.0.0.1:6379> incr incr_key(integer) 2127.0.0.1:6379> get incr_key"2"//查询过期时间，发现过期时间没有被清除127.0.0.1:6379> ttl incr_key(integer) 277
```

**LPUSH**：

```java
//新增一个list类型的key，并添加一个为1的值127.0.0.1:6379> LPUSH list 1(integer) 1//为list设置300s的过期时间127.0.0.1:6379> expire list 300(integer) 1//查看过期时间127.0.0.1:6379> ttl list(integer) 292//往list里面添加值2127.0.0.1:6379> lpush list 2(integer) 2//查看list的所有值127.0.0.1:6379> lrange list 0 11) "2"2) "1"//能看到往list里面添加值并没有使过期时间清除127.0.0.1:6379> ttl list(integer) 252
```

## 3、PERSIST命令会清除过期时间


当使用**PERSIST**命令将一个设置了过期时间的key转变成一个持久化的key的时候，也会清除过期时间。

```java
127.0.0.1:6379> set persist_key haha ex 300OK127.0.0.1:6379> ttl persist_key(integer) 296//将key变为持久化的127.0.0.1:6379> persist persist_key(integer) 1//过期时间被清除127.0.0.1:6379> ttl persist_key(integer) -1
```

## 4、使用RENAME命令，老key的过期时间将会转到新key上


在使用例如：**RENAME KEY_A KEY_B**命令将KEY_A重命名为KEY_B，不管KEY_B有没有设置过期时间，新的key KEY_B将会继承KEY_A的所有特性。

```java
//设置key_a的过期时间为300s127.0.0.1:6379> set key_a value_a ex 300OK//设置key_b的过期时间为600s127.0.0.1:6379> set key_b value_b ex 600OK127.0.0.1:6379> ttl key_a(integer) 279127.0.0.1:6379> ttl key_b(integer) 591//将key_a重命名为key_b127.0.0.1:6379> rename key_a key_bOK//新的key_b继承了key_a的过期时间127.0.0.1:6379> ttl key_b(integer) 248
```

这里篇幅有限，我就不一一将key_a重命名到key_b的各个情况列出来，大家可以在自己电脑上试一下key_a设置了过期时间，key_b没设置过期时间这种情况。


## 5、使用EXPIRE/PEXPIRE设置的过期时间为负数或者使用EXPIREAT/PEXPIREAT设置过期时间戳为过去的时间会导致key被删除


**EXPIRE：**

```java
127.0.0.1:6379> set key_1 value_1OK127.0.0.1:6379> get key_1"value_1"//设置过期时间为-1127.0.0.1:6379> expire key_1 -1(integer) 1//发现key被删除127.0.0.1:6379> get key_1(nil)
```

**EXPIREAT：**

```java
127.0.0.1:6379> set key_2 value_2OK127.0.0.1:6379> get key_2"value_2"//设置的时间戳为过去的时间127.0.0.1:6379> expireat key_2 10000(integer) 1//key被删除127.0.0.1:6379> get key_2(nil)
```

## 6、EXPIRE命令可以更新过期时间


对一个已经设置了过期时间的key使用expire命令，可以更新其过期时间。

```java
//设置key_1的过期时间为100s127.0.0.1:6379> set key_1 value_1 ex 100OK127.0.0.1:6379> ttl key_1(integer) 95//更新key_1的过期时间为300s127.0.0.1:6379> expire key_1 300(integer) 1127.0.0.1:6379> ttl key_1(integer) 295
```

在Redis2.1.3以下的版本中，使用expire命令更新一个已经设置了过期时间的key的过期时间会失败。并且对一个设置了过期时间的key使用LPUSH/HSET等命令修改其value的时候，会导致Redis删除该key。


# Redis的过期策略


那你有没有想过一个问题，Redis里面如果有大量的key，怎样才能高效的找出过期的key并将其删除呢，难道是遍历每一个key吗？假如同一时期过期的key非常多，Redis会不会因为一直处理过期事件，而导致读写指令的卡顿。



这里说明一下，Redis是单线程的，所以一些耗时的操作会导致Redis卡顿，比如当Redis数据量特别大的时候，使用keys * 命令列出所有的key。


实际上Redis使用**懒惰删除****+****定期删除**相结合的方式处理过期的key。


## 懒惰删除


所谓**懒惰删除**就是在客户端访问该key的时候，redis会对key的过期时间进行检查，如果过期了就立即删除。


这种方式看似很完美，在访问的时候检查key的过期时间，不会占用太多的额外CPU资源。但是如果一个key已经过期了，如果长时间没有被访问，那么这个key就会一直存留在内存之中，严重消耗了内存资源。


## 定期删除


**定期删除**的原理是，Redis会将所有设置了过期时间的key放入一个字典中，然后每隔一段时间从字典中随机一些key检查过期时间并删除已过期的key。


Redis默认每秒进行10次过期扫描：


* **从过期字典中随机20个key**    
* **删除这20个key中已过期的**    
* **如果超过25%的key过期，则重复第一步**


同时，为了保证不出现循环过度的情况，Redis还设置了扫描的时间上限，默认不会超过25ms.


**往日精选文章**


[最中肯的Redis规范全在这了](http://mp.weixin.qq.com/s?__biz=MzU1NTEzMDAxNQ==&chksm=fbd84b2dccafc23b400c7d4ddc849ce46dc022e284e240d2ede6d74697dbdcaafd168663036d&idx=2&mid=2247485149&scene=21&sn=0a98aff33ff004107775b0b077f6d918#wechat_redirect "最中肯的Redis规范全在这了")


[Redis 高级面试题 学会这些还怕进不了大厂？](http://mp.weixin.qq.com/s?__biz=MzU1NTEzMDAxNQ==&chksm=fbd84871ccafc1677c7d89446c61a333a8f1d2ac50e4630be4df92c8c68ab0a7046beee9bf31&idx=2&mid=2247484801&scene=21&sn=a714c5dc9bb12cbd6f7dc8de9e22fd15#wechat_redirect "Redis 高级面试题 学会这些还怕进不了大厂？")


[Redis中缓存雪崩、缓存穿透等问题的解决方案](http://mp.weixin.qq.com/s?__biz=MzU1NTEzMDAxNQ==&chksm=fbd84b22ccafc234c2431be476f47bfbb75b2c7a59837536039daee3758af041296c37dd2f5a&idx=1&mid=2247485138&scene=21&sn=b94602afd80168c9159653cf310bf0c1#wechat_redirect "Redis中缓存雪崩、缓存穿透等问题的解决方案")


[掌握Redis主从复制、哨兵、Cluster三种集群模式](http://mp.weixin.qq.com/s?__biz=MzU1NTEzMDAxNQ==&chksm=fbd84822ccafc1346160d8828c79c16d1c3cfb7ee5e60dc2beb345e8181f6e429ff9fc50723e&idx=1&mid=2247484882&scene=21&sn=7dedfbf461c22e2ca0299516f39b0420#wechat_redirect "掌握Redis主从复制、哨兵、Cluster三种集群模式")


[2020 年最新版 68 道Redis面试题，收藏起来备用！](http://mp.weixin.qq.com/s?__biz=MzU1NTEzMDAxNQ==&chksm=fbd8483bccafc12d5b23d2806a4afd2f307dd242ee6d22a6ced54e5e3c65d16569a376c6baa9&idx=1&mid=2247484875&scene=21&sn=895b25ec6ecc7ac889d11b6e94359cbd#wechat_redirect "2020 年最新版 68 道Redis面试题，收藏起来备用！")













