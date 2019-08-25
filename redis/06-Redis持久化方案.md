# Redis持久化方案
## 一、RDB持久化
RDB方式的持久化是通过快照（snapshotting）完成的，当符合一定条件时Redis会自动将内存中的数据进行快照并持久化到硬盘。

RDB是Redis默认采用的持久化方式。
```
save 900 1
save 300 10
save 60 10000
```

### 1、持久化条件配置
save 开头的一行就是持久化配置，可以配置多个条件（每行配置一个条件），每个条件之间是“或”的关系。

“save 900 1”表示15分钟（900秒钟）内至少1个键被更改则进行快照。
“save 300 10”表示5分钟（300秒）内至少10个键被更改则进行快照。

### 2、配置快照文件目录
配置dir指定rdb快照文件的位置
```
# Note that you must specify a directory here, not a file name.
dir ./
```

### 3、配置快照文件的名称
设置dbfilename指定rdb快照文件的名称
```
# The filename where to dump the DB
dbfilename dump.rdb
```

Redis启动后会读取RDB快照文件，将数据从硬盘载入到内存。根据数据量大小与结构和服务器性能不同，这个时间也不同。通常将记录一千万个字符串类型键、大小为1GB的快照文件载入到内存中需要花费20～30秒钟。

### 4、问题总结
**通过RDB方式实现持久化，一旦Redis异常退出，就会丢失最后一次快照以后更改的所有数据。**这就需要开发者根据具体的应用场合，通过组合设置自动快照条件的方式来将可能发生的数据损失控制在能够接受的范围。

如果数据很重要以至于无法承受任何损失，则可以考虑使用**AOF**方式进行持久化。

## 二、AOF持久化
默认情况下Redis没有开启AOF（append only file）方式的持久化，【操作一次就写一次数据】

可以通过修改redis.conf配置文件中的appendonly参数开启
```
appendonly yes
```

开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬
盘中的AOF文件。

AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的。
```
dir ./
```

默认的文件名是appendonly.aof，可以通过appendfilename参数修改：
```
appendfilename appendonly.aof
```

重启服务，再添加数据，然后会有一个aof文件,然后分析appendonly.aof文件内容
```
[root@A01 bin]# ./redis-cli shutdown
[root@A01 bin]# ./redis-server redis.conf
[root@A01 bin]# ./redis-cli
127.0.0.1:6379> set ip 192.168.1.1
OK
```







