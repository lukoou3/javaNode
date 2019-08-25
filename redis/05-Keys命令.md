# Keys命令
## 一、设置key的生存时间
Redis在实际使用过程中更多的用作缓存，然而缓存的数据一般都是需要设置生存时间的，即：到期后数据销毁。
```
EXPIRE key seconds			 设置key的生存时间（单位：秒）key在多少秒后会自动删除
TTL key 					查看key生于的生存时间
PERSIST key				清除生存时间 
PEXPIRE key milliseconds	生存时间设置单位为：毫秒
```

例子：
```
192.168.101.3:7002> set test 1		设置test的值为1
OK
192.168.101.3:7002> get test			获取test的值
"1"
192.168.101.3:7002> EXPIRE test 5	设置test的生存时间为5秒
(integer) 1
192.168.101.3:7002> TTL test			查看test的生于生成时间还有1秒删除
(integer) 1
192.168.101.3:7002> TTL test
(integer) -2
192.168.101.3:7002> get test			获取test的值，已经删除
(nil)
```

## 二、其它命令（自学）
### 1、keys
返回满足给定pattern 的所有key
```
redis 127.0.0.1:6379> keys mylist*
1) "mylist"
2) "mylist5"
3) "mylist6"
4) "mylist7"
5) "mylist8"
```

### 2、exists
确认一个key 是否存在

示例：从结果来看，数据库中不存在HongWan 这个key，但是age 这个key 是存在的
```
redis 127.0.0.1:6379> exists HongWan
(integer) 0
redis 127.0.0.1:6379> exists age
(integer) 1
redis 127.0.0.1:6379>
```

### 3、del
删除一个key
```
redis 127.0.0.1:6379> del age
(integer) 1
redis 127.0.0.1:6379> exists age
(integer) 0
```

### 4、rename
重命名key

示例：age 成功的被我们改名为age_new 了
```
redis 127.0.0.1:6379[1]> keys *
1) "age"
redis 127.0.0.1:6379[1]> rename age age_new
OK
redis 127.0.0.1:6379[1]> keys *
1) "age_new"
redis 127.0.0.1:6379[1]>
```

### 5、type
返回值的类型

示例：这个方法可以非常简单的判断出值的类型
```
redis 127.0.0.1:6379> type addr
string
redis 127.0.0.1:6379> type myzset2
zset
redis 127.0.0.1:6379> type mylist
list
redis 127.0.0.1:6379>
```




