# 序列化机制-采用protostuff和kryo高性能序列化框架实现RestTemplate的序列化组件

摘抄自：https://www.jianshu.com/p/2836c6f4c124

## 序列化


* 序列化可以简单理解为对象-->字节的过程，同理，反序列化则是相反的过程。为什么需要序列化？因为网络传输只认字节。所以互信的过程依赖于序列化。    
* 网络传输的性能等诸多因素，通常会支持多种序列化方式以供使用者插拔使用，一些常用的序列化方案hessian，kryo，Protostuff、FST等，其中最快、效果最好的要数Kryo和Protostuff


### RedisConfiguration的配置


* 创建Redis连接工厂对象（RedisConnectionFactory）    
* 创建RestTemplate对象根据RedisConnectionFactory对象。    
* 配置相关的RedisSerializaer组件


```java
@Configuration
public class RedisConfiguration {

    @Bean("redisConnectionFactory")
    public RedisConnectionFactory redisConnectionFactory(RedisConfigMapper mapper) {
        List<RedisConfig> redisConfigs = mapper.getRedisConfig();
        List<String> clusterNodes = new ArrayList<>();
        for (RedisConfig rc : redisConfigs) {
            clusterNodes.add(rc.getUrl() + ":" + rc.getPort());
        }
        // 获取Redis集群配置信息
        RedisClusterConfiguration rcf = new RedisClusterConfiguration(clusterNodes);
        return new JedisConnectionFactory(rcf);
    }

    @Bean("redisTemplate")
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        // redis value使用的序列化器
        template.setValueSerializer(new XXXRedisSerializer<>());
        // redis key使用的序列化器
        template.setKeySerializer(new XXXRedisSerializer<>());
        template.setHashKeySerializer(new XXXRedisSerializer<>());
        template.setHashValueSerializer(new XXXRedisSerializer<>());
        template.afterPropertiesSet();
        return template;
    }
}

```


## Kryo序列化实现


### Maven配置文件


```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>com.esotericsoftware</groupId>
            <artifactId>kryo</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>de.javakaffee</groupId>
            <artifactId>kryo-serializers</artifactId>
            <version>0.41</version>
        </dependency>
        <dependency>
            <groupId>com.esotericsoftware</groupId>
            <artifactId>kryo-shaded</artifactId>
            <version>4.0.1</version>
        </dependency>
    </dependencies>

```


由于其底层依赖于ASM技术，与Spring等框架可能会发生ASM依赖的版本冲突（文档中表示这个冲突还挺容易出现）所以提供了另外一个依赖以供解决此问题：**kryo-shaded**


### Kryo三种读写方式


如果知道class字节码，并且对象不为空


```java
  kryo.writeObject(output, classObject);
  RestClass restClass = kryo.readObject(input, RestClass.class);

```


快速入门中的序列化/反序列化的方式便是这一种。而Kryo考虑到someObject可能为null，也会导致返回的结果为null，所以提供了第二套读写方式。


如果知道class字节码，并且对象可能为空


```java
kryo.writeObjectOrNull(output, classObject);
RestClass someObject = kryo.readObjectOrNull(input, RestClass.class);

```


但这两种方法似乎都不能满足我们的需求，在RPC调用中，序列化和反序列化分布在不同的端点，对象的类型确定，我们不想依赖于手动指定参数，最好将字节码的信息直接存放到序列化结果中，在反序列化时自行读取字节码信息。Kryo考虑到了这一点，于是提供了第三种方式。如果实现类的字节码未知，并且对象可能为null。


```java
  kryo.writeClassAndObject(output, object);
Object object = kryo.readClassAndObject(input);
if (object instanceof RestClass) {
}

```


我们牺牲了一些空间一些性能去存放字节码信息


### 支持的序列化类型







![](assets/ee4fa3be169cd24190124558f27d1a89.png)


image


上面表格中支持的类型一览无余，这都是其默认支持的。


```java
Kryo kryo = new Kryo();
kryo.addDefaultSerializer(RestClass.class, RestSerializer.class);

```


这样的方式，也可以为一个Kryo实例扩展序列化器


### Kryo支持类型：


* 枚举    
* 集合、数组    
* 子类/多态    
* 循环引用    
* 内部类    
* 泛型


### Kryo反序列化的异常问题


* Kryo不支持Bean中增删字段，如果使用Kryo序列化了一个类，存入了Redis，对类进行了修改，会导致反序列化的异常。    
* 另外需要注意的一点是使用反射创建的一些类序列化的支持。如使用Arrays.asList();创建的List对象，会引起序列化异常。    
* 不支持包含无参构造器类的反序列化，尝试反序列化一个不包含无参构造器的类将会得到以下的异常：    
* 保证每个类具有无参构造器是应当遵守的编程规范，但实际开发中一些第三库的相关类不包含无参构造，的确是有点麻烦。


### Kryo是线程不安全的


借助ThreadLocal来维护以保证其线程安全。


```java
private static final ThreadLocal<Kryo> kryos = new ThreadLocal<Kryo>() {
    protected Kryo initialValue() {
        Kryo kryo = new Kryo();
        // configure kryo instance, customize settings
        return kryo;
    };
};

// Somewhere else, use Kryo
Kryo k = kryos.get();
...

```


#### Kryo相关配置参数


每个Kryo实例都可以拥有两个配置参数。


* kryo.setRegistrationRequired(false);//关闭注册行为



**Kryo支持对注册行为，如kryo.register(SomeClazz.class)，这会赋予该Class一个从0开始的编号，但Kryo使用注册行为最大的问题在于，其不保证同一个Class每一次注册的号码相同，这与注册的顺序有关，也就意味着在不同的机器、同一个机器重启前后都有可能拥有不同的编号，这会导致序列化产生问题，所以在分布式项目中，一般关闭注册行为**。


* kryo.setReferences(true);//支持循环引用



循环引用，Kryo为了追求高性能，可以关闭循环引用的支持。不过我并不认为关闭它是一件好的选择，大多数情况下，请保持kryo.setReferences(true)。


### 常用Kryo工具类


```java
public class KryoSerializer {

    public byte[] serialize(Object obj) {
        Kryo kryo = kryoLocal.get();
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        Output output = new Output(byteArrayOutputStream);//<1>
        kryo.writeClassAndObject(output, obj);//<2>
        output.close();
        return byteArrayOutputStream.toByteArray();
    }

    public <T> T deserialize(byte[] bytes) {
        Kryo kryo = kryoLocal.get();
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        Input input = new Input(byteArrayInputStream);// <1>
        input.close();
        return (T) kryo.readClassAndObject(input);//<2>
    }

    private static final ThreadLocal<Kryo> kryoLocal = new ThreadLocal<Kryo>() {//<3>
        @Override
        protected Kryo initialValue() {
            Kryo kryo = new Kryo();
            kryo.setReferences(true);//默认值为true,强调作用
            kryo.setRegistrationRequired(false);//默认值为false,强调作用
            return kryo;
        }
    };
}

```


* Kryo的Input和Output接收一个InputStream和OutputStream，Kryo通常完成字节数组和对象的转换，所以常用的输入输出流实现为ByteArrayInputStream/ByteArrayOutputStream。    
* writeClassAndObject和readClassAndObject配对使用在分布式场景下是最常见的，序列化时将字节码存入序列化结果中，便可以在反序列化时不必要传入字节码信息。    
* 使用ThreadLocal维护Kryo实例，这样减少了每次使用都实例化一次Kryo的开销又可以保证其线程安全。


### KryoRedisSerializer


数据交换或数据持久化，比如使用kryo把对象序列化成字节数组发送给消息队列或者放到redis等等应用场景。


```java
public class KryoRedisSerializer<T> implements RedisSerializer<T> {

    private static final String DEFAULT_ENCODING = "UTF-8";

    //每个线程的 Kryo 实例
    private static final ThreadLocal<Kryo> kryoLocal = new ThreadLocal<Kryo>() {
        @Override
        protected Kryo initialValue() {
            Kryo kryo = new Kryo();

            /**
             * 不要轻易改变这里的配置！更改之后，序列化的格式就会发生变化，
             * 上线的同时就必须清除 Redis 里的所有缓存，
             * 否则那些缓存再回来反序列化的时候，就会报错
             */
            //支持对象循环引用（否则会栈溢出）
            kryo.setReferences(true); //默认值就是 true，添加此行的目的是为了提醒维护者，不要改变这个配置

            //不强制要求注册类（注册行为无法保证多个 JVM 内同一个类的注册编号相同；而且业务系统中大量的 Class 也难以一一注册）
            kryo.setRegistrationRequired(false); //默认值就是 false，添加此行的目的是为了提醒维护者，不要改变这个配置

            //Fix the NPE bug when deserializing Collections.
            ((Kryo.DefaultInstantiatorStrategy) kryo.getInstantiatorStrategy())
                    .setFallbackInstantiatorStrategy(new StdInstantiatorStrategy());

            return kryo;
        }
    };

    /**
     * 获得当前线程的 Kryo 实例
     *
     * @return 当前线程的 Kryo 实例
     */
    public static Kryo getInstance() {
        return kryoLocal.get();
    }

     @Override
    public byte[] serialize(T t) throws SerializationException {
        byte[] buffer = new byte[2048];
        Output output = new Output(buffer);
        getInstance().writeClassAndObject(output, t);
        return output.toBytes();
    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException {
        Input input = new Input(bytes);
        @SuppressWarnings("unchecked")
        T t = (T) getInstance().readClassAndObject(input);
        return t;
    }

}

```


## protostuff序列化实现


### Maven配置文件


```xml
<!-- 序列化 -->
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.1.3</version>
</dependency>
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.1.3</version>
</dependency>

```


## 定义一个ProtoStuffUtil工具类


```java
@Slf4j
public class ProtoStuffUtil {
    /**
     * 序列化对象
     *
     * @param obj
     * @return
     */
    public static <T> byte[] serialize(T obj) {
        if (obj == null) {
            log.error("Failed to serializer, obj is null");
            throw new RuntimeException("Failed to serializer");
        }

        @SuppressWarnings("unchecked") Schema<T> schema = (Schema<T>) RuntimeSchema.getSchema(obj.getClass());
        LinkedBuffer buffer = LinkedBuffer.allocate(1024 * 1024);
        byte[] protoStuff;
        try {
            protoStuff = ProtostuffIOUtil.toByteArray(obj, schema, buffer);
        } catch (Exception e) {
            log.error("Failed to serializer, obj:{}", obj, e);
            throw new RuntimeException("Failed to serializer");
        } finally {
            buffer.clear();
        }
        return protoStuff;
    }

    /**
     * 反序列化对象
     *
     * @param paramArrayOfByte
     * @param targetClass
     * @return
     */
    public static <T> T deserialize(byte[] paramArrayOfByte, Class<T> targetClass) {
        if (paramArrayOfByte == null || paramArrayOfByte.length == 0) {
            log.error("Failed to deserialize, byte is empty");
            throw new RuntimeException("Failed to deserialize");
        }

        T instance;
        try {
            instance = targetClass.newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            log.error("Failed to deserialize", e);
            throw new RuntimeException("Failed to deserialize");
        }

        Schema<T> schema = RuntimeSchema.getSchema(targetClass);
        ProtostuffIOUtil.mergeFrom(paramArrayOfByte, instance, schema);
        return instance;
    }

    /**
     * 序列化列表
     *
     * @param objList
     * @return
     */
    public static <T> byte[] serializeList(List<T> objList) {
        if (objList == null || objList.isEmpty()) {
            log.error("Failed to serializer, objList is empty");
            throw new RuntimeException("Failed to serializer");
        }

        @SuppressWarnings("unchecked") Schema<T> schema =
                (Schema<T>) RuntimeSchema.getSchema(objList.get(0).getClass());
        LinkedBuffer buffer = LinkedBuffer.allocate(1024 * 1024);
        byte[] protoStuff;
        ByteArrayOutputStream bos = null;
        try {
            bos = new ByteArrayOutputStream();
            ProtostuffIOUtil.writeListTo(bos, objList, schema, buffer);
            protoStuff = bos.toByteArray();
        } catch (Exception e) {
            log.error("Failed to serializer, obj list:{}", objList, e);
            throw new RuntimeException("Failed to serializer");
        } finally {
            buffer.clear();
            try {
                if (bos != null) {
                    bos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return protoStuff;
    }

    /**
     * 反序列化列表
     *
     * @param paramArrayOfByte
     * @param targetClass
     * @return
     */
    public static <T> List<T> deserializeList(byte[] paramArrayOfByte, Class<T> targetClass) {
        if (paramArrayOfByte == null || paramArrayOfByte.length == 0) {
            log.error("Failed to deserialize, byte is empty");
            throw new RuntimeException("Failed to deserialize");
        }

        Schema<T> schema = RuntimeSchema.getSchema(targetClass);
        List<T> result;
        try {
            result = ProtostuffIOUtil.parseListFrom(new ByteArrayInputStream(paramArrayOfByte), schema);
        } catch (IOException e) {
            log.error("Failed to deserialize", e);
            throw new RuntimeException("Failed to deserialize");
        }
        return result;
    }
}

```


### 不修改RedisSerializer组件和重写操作


直接自定义RedisClient工具类方式，代理RedisTemplate的工具类方法


```java
@Component
public class RedisClient {

    private final RedisTemplate<String, String> redisTemplate;

    @Autowired
    public RedisClient(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    /**
     * get cache
     *
     * @param field
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> T get(final String field, Class<T> targetClass) {
        byte[] result = redisTemplate.execute((RedisCallback<byte[]>) connection -> connection.get(field.getBytes()));
        if (result == null) {
            return null;
        return ProtoStuffUtil.deserialize(result, targetClass);
    }

    /**
     * put cache
     *
     * @param field
     * @param obj
     * @param <T>
     * @return
     */
    public <T> void set(String field, T obj) {
        final byte[] value = ProtoStuffUtil.serialize(obj);
        redisTemplate.execute((RedisCallback<Void>) connection -> {
            connection.set(field.getBytes(), value);
            return null;
        });
    }

    /**
     * put cache with expire time
     *
     * @param field
     * @param obj
     * @param expireTime 单位: s
     * @param <T>
     */
    public <T> void setWithExpire(String field, T obj, final long expireTime) {
        final byte[] value = ProtoStuffUtil.serialize(obj);
        redisTemplate.execute((RedisCallback<Void>) connection -> {
            connection.setEx(field.getBytes(), expireTime, value);
            return null;
        });
    }

    /**
     * get list cache
     *
     * @param field
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> List<T> getList(final String field, Class<T> targetClass) {
        byte[] result = redisTemplate.execute((RedisCallback<byte[]>) connection -> connection.get(field.getBytes()));
        if (result == null) {
            return null;
        }

        return ProtoStuffUtil.deserializeList(result, targetClass);
    }

    /**
     * put list cache
     *
     * @param field
     * @param objList
     * @param <T>
     * @return
     */
    public <T> void setList(String field, List<T> objList) {
        final byte[] value = ProtoStuffUtil.serializeList(objList);
        redisTemplate.execute((RedisCallback<Void>) connection -> {
            connection.set(field.getBytes(), value);
            return null;
        });
    }

    /**
     * put list cache with expire time
     *
     * @param field
     * @param objList
     * @param expireTime
     * @param <T>
     * @return
     */
    public <T> void setListWithExpire(String field, List<T> objList, final long expireTime) {
        final byte[] value = ProtoStuffUtil.serializeList(objList);
        redisTemplate.execute((RedisCallback<Void>) connection -> {
            connection.setEx(field.getBytes(), expireTime, value);
            return null;
        });
    }

    /**
     * get h cache
     *
     * @param key
     * @param field
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> T hGet(final String key, final String field, Class<T> targetClass) {
        byte[] result = redisTemplate
                .execute((RedisCallback<byte[]>) connection -> connection.hGet(key.getBytes(), field.getBytes()));
        if (result == null) {
            return null;
        }

        return ProtoStuffUtil.deserialize(result, targetClass);
    }

    /**
     * put hash cache
     *
     * @param key
     * @param field
     * @param obj
     * @param <T>
     * @return
     */
    public <T> boolean hSet(String key, String field, T obj) {
        final byte[] value = ProtoStuffUtil.serialize(obj);
        return redisTemplate.execute(
                (RedisCallback<Boolean>) connection -> connection.hSet(key.getBytes(), field.getBytes(), value));
    }

    /**
     * put hash cache
     *
     * @param key
     * @param field
     * @param obj
     * @param <T>
     */
    public <T> void hSetWithExpire(String key, String field, T obj, long expireTime) {
        final byte[] value = ProtoStuffUtil.serialize(obj);
        redisTemplate.execute((RedisCallback<Void>) connection -> {
            connection.hSet(key.getBytes(), field.getBytes(), value);
            connection.expire(key.getBytes(), expireTime);
            return null;
        });
    }

    /**
     * get list cache
     *
     * @param key
     * @param field
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> List<T> hGetList(final String key, final String field, Class<T> targetClass) {
        byte[] result = redisTemplate
                .execute((RedisCallback<byte[]>) connection -> connection.hGet(key.getBytes(), field.getBytes()));
        if (result == null) {
            return null;
        }

        return ProtoStuffUtil.deserializeList(result, targetClass);
    }

    /**
     * put list cache
     *
     * @param key
     * @param field
     * @param objList
     * @param <T>
     * @return
     */
    public <T> boolean hSetList(String key, String field, List<T> objList) {
        final byte[] value = ProtoStuffUtil.serializeList(objList);
        return redisTemplate.execute(
                (RedisCallback<Boolean>) connection -> connection.hSet(key.getBytes(), field.getBytes(), value));
    }

    /**
     * get cache by keys
     *
     * @param key
     * @param fields
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> Map<String, T> hMGet(String key, Collection<String> fields, Class<T> targetClass) {
        List<byte[]> byteFields = fields.stream().map(String::getBytes).collect(Collectors.toList());
        byte[][] queryFields = new byte[byteFields.size()][];
        byteFields.toArray(queryFields);
        List<byte[]> cache = redisTemplate
                .execute((RedisCallback<List<byte[]>>) connection -> connection.hMGet(key.getBytes(), queryFields));

        Map<String, T> results = new HashMap<>(16);
        Iterator<String> it = fields.iterator();
        int index = 0;
        while (it.hasNext()) {
            String k = it.next();
            if (cache.get(index) == null) {
                index++;
                continue;
            }

            results.put(k, ProtoStuffUtil.deserialize(cache.get(index), targetClass));
            index++;
        }

        return results;
    }

    /**
     * set cache by keys
     *
     * @param field
     * @param values
     * @param <T>
     */
    public <T> void hMSet(String field, Map<String, T> values) {
        Map<byte[], byte[]> byteValues = new HashMap<>(16);
        for (Map.Entry<String, T> value : values.entrySet()) {
            byteValues.put(value.getKey().getBytes(), ProtoStuffUtil.serialize(value.getValue()));
        }

        redisTemplate.execute((RedisCallback<Void>) connection -> {
            connection.hMSet(field.getBytes(), byteValues);
            return null;
        });
    }

    /**
     * get caches in hash
     *
     * @param key
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> Map<String, T> hGetAll(String key, Class<T> targetClass) {
        Map<byte[], byte[]> records = redisTemplate
                .execute((RedisCallback<Map<byte[], byte[]>>) connection -> connection.hGetAll(key.getBytes()));

        Map<String, T> ret = new HashMap<>(16);
        for (Map.Entry<byte[], byte[]> record : records.entrySet()) {
            T obj = ProtoStuffUtil.deserialize(record.getValue(), targetClass);
            ret.put(new String(record.getKey()), obj);
        }

        return ret;
    }

    /**
     * list index
     *
     * @param key
     * @param index
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> T lIndex(String key, int index, Class<T> targetClass) {
        byte[] value =
                redisTemplate.execute((RedisCallback<byte[]>) connection -> connection.lIndex(key.getBytes(), index));
        return ProtoStuffUtil.deserialize(value, targetClass);
    }

    /**
     * list range
     *
     * @param key
     * @param start
     * @param end
     * @param targetClass
     * @param <T>
     * @return
     */
    public <T> List<T> lRange(String key, int start, int end, Class<T> targetClass) {
        List<byte[]> value = redisTemplate
                .execute((RedisCallback<List<byte[]>>) connection -> connection.lRange(key.getBytes(), start, end));
        return value.stream().map(record -> ProtoStuffUtil.deserialize(record, targetClass))
                .collect(Collectors.toList());
    }

    /**
     * list left push
     *
     * @param key
     * @param obj
     * @param <T>
     */
    public <T> void lPush(String key, T obj) {
        final byte[] value = ProtoStuffUtil.serialize(obj);
        redisTemplate.execute((RedisCallback<Long>) connection -> connection.lPush(key.getBytes(), value));
    }

    /**
     * list left push
     *
     * @param key
     * @param objList
     * @param <T>
     */
    public <T> void lPush(String key, List<T> objList) {
        List<byte[]> byteFields = objList.stream().map(ProtoStuffUtil::serialize).collect(Collectors.toList());
        byte[][] values = new byte[byteFields.size()][];

        redisTemplate.execute((RedisCallback<Long>) connection -> connection.lPush(key.getBytes(), values));
    }

    /**
     * 精确删除key
     *
     * @param key
     */
    public void deleteCache(String key) {
        redisTemplate.delete(key);
    }


    /**
     * 排行榜的存入
     *
     * @param redisKey
     * @param immutablePair
     */
    public void zAdd(String redisKey, ImmutablePair<String, BigDecimal> immutablePair) {
        final byte[] key = redisKey.getBytes();
        final byte[] value = immutablePair.getLeft().getBytes();
        redisTemplate.execute((RedisCallback<Boolean>) connection -> connection
                .zAdd(key, immutablePair.getRight().doubleValue(), value));

    }

    /**
     * 获取排行榜低->高排序
     *
     * @param redisKey 要进行排序的类别
     * @param start
     * @param end
     * @return
     */
    public List<ImmutablePair<String, BigDecimal>> zRangeWithScores(String redisKey, int start, int end) {
        Set<RedisZSetCommands.Tuple> items = redisTemplate.execute(
                (RedisCallback<Set<RedisZSetCommands.Tuple>>) connection -> connection
                        .zRangeWithScores(redisKey.getBytes(), start, end));
        return items.stream()
                .map(record -> ImmutablePair.of(new String(record.getValue()), BigDecimal.valueOf(record.getScore())))
                .collect(Collectors.toList());
    }


    /**
     * 获取排行榜高->低排序
     *
     * @param redisKey 要进行排序的类别
     * @param start
     * @param end
     * @return
     */
    public List<ImmutablePair<String, BigDecimal>> zRevRangeWithScores(String redisKey, int start, int end) {
        Set<RedisZSetCommands.Tuple> items = redisTemplate.execute(
                (RedisCallback<Set<RedisZSetCommands.Tuple>>) connection -> connection
                        .zRevRangeWithScores(redisKey.getBytes(), start, end));
        return items.stream()
                .map(record -> ImmutablePair.of(new String(record.getValue()), BigDecimal.valueOf(record.getScore())))
                .collect(Collectors.toList());
    }
}

```


### 参考资料


[https://www.cnblogs.com/hntyzgn/p/7122709.html](https://links.jianshu.com/go?to=https://www.cnblogs.com/hntyzgn/p/7122709.html "https://www.cnblogs.com/hntyzgn/p/7122709.html")

