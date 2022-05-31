# 序列化机制-针对Java对象压缩及序列化技术的探索之路
摘抄自：`https://www.jianshu.com/p/91dde7cbfc80`

## 序列化和反序列化


* 序列化就是指把对象转换为字节码；对象传递和保存时，保证对象的完整性和可传递性。把对象转换为有字节码，以便在网络上传输或保存在本地文件中；    
* 反序列化就是指把字节码恢复为对象；根据字节流中保存的对象状态及描述信息，通过反序列化重建对象；    
* 一般情况下要求实现Serializable接口，该接口中没有定义任何成员，只是起到标记对象是否可以被序列化的作用。对象在进行序列化和反序列化的时候，必须实现Serializable接口，但并不强制声明唯一的serialVersionUID，是否声明serialVersionUID对于对象序列化的向上向下的兼容性有很大的影响。


## 为何需要有序列化呢？


* **一方面是为了存储在磁盘中**，    
* **另一方面为了网络远程传输的内容**。


## Java实现序列化的方式


### 二进制格式 + 指定语言层级



JavaBuiltIn（java原生）、JavaManual（根据成员变量类型，手工写）、FstSerliazation、Kryo


### 二进制格式 + 跨语言层级



Protobuf(Google)、Thrift(Facebook)、 AvroGeneric、Hessian


### JSON 格式化



Jackson、Gson、FastJSON等


### 类JSON格式化：



**CKS （textual JSON-like format）、BSON（JSON-like format with extended datatypes）、JacksonBson、MongoDB**


### XML文件格式化



**XmlXStream等**


## 序列化的分类



序列化工具大致就可以分为以上几类，简单概括就分为二进制binary和文本格式（json、xml）两大类。


### 在速度的对比上一般有如下规律：


* binary > textual    
* language-specific > language-unspecific



而textual中，由json相比xml冗余度更低因此速度上更胜一筹，而json又比bson这类textual serialization技术上更成熟，框架的选择上更丰富和优秀。


下面重点介绍下Kryo、fast-serialiation、fastjson、protocol-buffer


### Java原生序列化(青铜级别)


* Java本身提供的序列化工具基本上能胜任大多数场景下的序列化任务，关于其序列化机制。    
* 需要类实现了Serializable或Externalizable接口，否则会抛出异常，然后使用ObjectOutputStream与ObjectInputStream将对象写入写出。    
* Java自带的序列化工具在序列化过程中需要不仅需要将对象的完整的class name记录下来，还需要把该类的定义也都记录下，包括所有其他引用的类，这会是一笔很大的开销，尤其是仅仅序列化单个对象的时候。    
* 正因为java序列化机制会把所有meta-data记录下来，因此当修改了类的所在的包名后，反序列化则会报错。


```java
 //对象转成字节码
 ByteArrayOutputStream byteArrayOutputStream = new  ByteArrayOutputStream();
 ObjectOutputStream outputStream = new
 ObjectOutputStream(byteArrayOutputStream);
 outputStream.writeObject(VoUtil.getUser());
 byte  []bytes = byteArrayOutputStream.toByteArray();
 outputStream.close();
 //字节码转换成对象
 ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
 ObjectInputStream inputStream = new ObjectInputStream(byteArrayInputStream);
 Model result = （Model) inputStream.readObject();
 inputStream.close();

```


### Kryo序列化框架（星耀级别）


* kryo根据上述Java原生序列化机制的一些问题，对了很多优化工作，而且提供了很多serializer，甚至封装了Unsafe类型的序列化方式，更多关于Unsafe类型的序列化方式。    
* kryo，是一个快速序列化/反序列化工具，效率比java高出一个级别，序列化出来的结果，是其自定义的、独有的一种格式，体积更小，一般只用来进行序列化和反序列化，而不用于在多个系统、甚至多种语言间进行数据交换（目前 kryo 也只有 java 实现），目前已经有多家大公司使用，相对比较稳定。


```xml
    <dependency>
       <groupId>com.esotericsoftware</groupId>
       <artifactId>kryo</artifactId>
       <version>4.0.0</version>
     </dependency>

```


### KryoUtils序列化和反序列化操作


#### Kryo有三组读写对象的方法


* 如果不知道对象的具体类，且对象可以为null：


```java
kryo.writeClassAndObject(output, object);
Object object = kryo.readClassAndObject(input);

```


* 如果类已知且对象可以为null：


```java
kryo.writeObjectOrNull(output, someObject);
SomeClass someObject = kryo.readObjectOrNull(input, SomeClass.class);

```


* 如果类已知且对象不能为null:


```java
kryo.writeObject(output, someObject);
SomeClass someObject = kryo.readObject(input, SomeClass.class);

```


### 序列化和反序列化操作工具类KryoUtils


#### Kryo 和 KryoRegister


Kryo的运行速度是java Serializable 的20倍左右

Kryo的文件大小是java Serializable的一半左右


#### Kryo有两种模式:


一种是先注册(regist)，再写对象，即writeObject函数，实际上如果不先注册，在写对象时也会注册，并为class分配一个id。



注意，跨进程，则必须两端都按同样的模式，否则会出错，因为必须要明确类对应的唯一id。


另一种是写类名及对象，即writeClassAndObject函数。


writeClassAndObject函数是先写入一个约定的数字，再写入类ID（第一次要先写-1，再写类ID + 类名），写入引用关系，最后才写真正的数据。


### Kryo的操作模式


```java

static Kryo kryo = new Kryo();

public static byte[] serialize(Object obj) {
    byte[] buffer = new byte[2048];
    Output output = new Output(buffer);
    kryo.writeClassAndObject(output, obj);
    byte[] bs = output.toBytes();
    output.close();
    return bs;
}

public static Object deserialize(byte[] src) {
    Input input = new Input(src);
    Object obj = kryo.readClassAndObject(input);
    input.close();
    return obj;
}


```


### Kryo的Register操作模式


```java

static Kryo kryo = null;
static{
    kryo = new Kryo();
    kryo.setReferences(false);
    kryo.setRegistrationRequired(false);
    kryo.setInstantiatorStrategy(new StdInstantiatorStrategy());
}

public static byte[] serialize(Object obj) {
    kryo.register(obj.getClass());
    byte[] buffer = new byte[2048];
    Output output = new Output(buffer);
    kryo.writeObject(output, obj);
    byte[] bs = output.toBytes();
    output.close();
    return bs;
}

public static Object deserialize(byte[] src, Class<?> clazz) {
    kryo.register(clazz);
    Input input = new Input(src);
    Object obj = kryo.readObject(input, clazz);
    input.close();
    return obj;
}

```



推荐：[https://blog.csdn.net/fanjunjaden/article/details/72823866](https://links.jianshu.com/go?to=https://blog.csdn.net/fanjunjaden/article/details/72823866 "https://blog.csdn.net/fanjunjaden/article/details/72823866")



借鉴网上的一个很不错的工具类！


```java
public class KryoUtils  {
    /**
     * （池化Kryo实例）使用ThreadLocal
     */
    private static final ThreadLocal<Kryo> kryos = new ThreadLocal<Kryo>() {
        @Override
        protected Kryo initialValue() {
            Kryo kryo = new Kryo();
            //支持对象循环引用（否则会栈溢出）
            kryo.setReferences(true);
            // 不强制要求注册类（注册行为无法保证多个 JVM 内同一个类的注册编号相同；
            // 而且业务系统中大量的 Class 也难以一一注册）
            kryo.setRegistrationRequired(false);
            //Fix the NPE bug when deserializing Collections.
            kryo.setInstantiatorStrategy(new StdInstantiatorStrategy());
            return kryo;
        }
    };
    /**
     * （池化Kryo实例）使用KryoPool
     */
    private static KryoFactory factory = new KryoFactory() {
        public Kryo create () {
            Kryo kryo = new Kryo();
            return kryo;
        }
    };
    private static KryoPool pool = new KryoPool.Builder(factory).softReferences().build();
    /**
     * 使用ThreadLocal创建Kryo
     * 把java对象序列化成byte[];
     * @param obj java对象
     * @return
     */
    public static <T>  byte[] serializeObject(T obj) {
        ByteArrayOutputStream os=null;
        Output output=null;
        if(null != obj){
            Kryo kryo = kryos.get();
            try {
                os = new ByteArrayOutputStream();
                output = new Output(os);
                kryo.writeObject(output, obj);
                close(output);
                return os.toByteArray();
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                close(os);
            }
        }
        return null;
    }

    /**
     * 使用ThreadLocal创建Kryo
     * 把byte[]反序列化成指定的java对象
     * @param bytes
     * @param t 指定的java对象
     * @param <T>
     * @return 指定的java对象
     */
    public static <T> T unSerializeObject(byte[] bytes,Class<T> t) {
        ByteArrayInputStream is=null;
        Input input=null;
        if(null != bytes && bytes.length>0 && null!=t){
            try {
                Kryo kryo = kryos.get();
                is = new ByteArrayInputStream(bytes);
                input = new Input(is);
                return kryo.readObject(input,t);
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                close(is);
                close(input);
            }
        }
        return null;
    }

    /**
     * 使用ThreadLocal创建Kryo
     * 把List序列化成byte[];
     * @param list java对象
     * @return
     */
    public static <T>  byte[]  serializeList(List<T> list ) {
        ByteArrayOutputStream os=null;
        Output output=null;
        byte[] bytes = null;
        if(null != list && list.size()>0){
            Kryo kryo = kryos.get();
            try {
                os = new ByteArrayOutputStream();
                output = new Output(os);
                kryo.writeObject(output,list);
                close(output);
                bytes = os.toByteArray();
                return bytes;
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                close(os);
            }
        }
        return null;
    }

    /**
     * 使用ThreadLocal创建Kryo
     * 把byte[]反序列化成指定的List<T>
     * @param bytes byte数组
     * @param <T>
     * @return 指定java对象的List
     */
    public static <T> List<T> unSerializeList(byte[] bytes) {
        ByteArrayInputStream is=null;
        Input input=null;
        if(null !=bytes && bytes.length>0){
            try {
                Kryo kryo = kryos.get();
                is = new ByteArrayInputStream(bytes);
                input = new Input(is);
                List<T> list = kryo.readObject(input,ArrayList.class);
                return list;
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                close(is);
                close(input);
            }
        }
        return null;
    }
    /**
     * 使用ThreadLocal创建Kryo
     * 把java对象转序列化存储在文件中;
     * @param obj java对象
     * @return
     */
    public static <T>  boolean serializeFile(T obj,String path) {
        if(null != obj){
            Output output=null;
            try {
                Kryo kryo = kryos.get();
                output = new Output(new FileOutputStream(path));
                kryo.writeObject(output, obj);
                return true;
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                close(output);
            }
        }
        return false;
    }

    /**
     * 使用ThreadLocal创建Kryo
     * 把序列化的文件反序列化成指定的java对象
     * @param path 文件路径
     * @param t 指定的java对象
     * @param <T>
     * @return 指定的java对象
     */
    public static <T> T unSerializeFile(String path,Class<T> t) {
        if(null != path && null !=t ){
            Input input=null;
            try {
                Kryo kryo = kryos.get();
                input = new Input(new FileInputStream(path));
                return kryo.readObject(input,t);
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                close(input);
            }
        }
        return null;
    }

    /**
     * 使用KryoPool SoftReferences创建Kryo
     * 把java对象序列化成byte[] ;
     * @param obj java对象
     * @return
     */
    public static <T>  byte[] serializePoolSoftReferences (T obj) {
        if(null!=obj){
            Kryo kryo =pool.borrow();
            ByteArrayOutputStream os=null;
            Output output=null;
            try {
                os = new ByteArrayOutputStream();
                output = new Output(os);
                kryo.writeObject(output, obj);
                close(output);
                byte [] bytes = os.toByteArray();
                return bytes;
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                pool.release(kryo);
                close(os);
            }
        }
        return null;
    }
    /**
     * 使用KryoPool SoftReferences创建Kryo
     * 把byte[]反序列化成指定的java对象
     * @param bytes
     * @return
     */
    public static <T> T unSerializePoolSoftReferences(byte[] bytes,Class<T> t) {
        if(null !=bytes && bytes.length>0 && null!=t){
            Kryo kryo =pool.borrow();
            ByteArrayInputStream is=null;
            Output output=null;
            try {
                is = new ByteArrayInputStream(bytes);
                Input input= new Input(is);
                return kryo.readObject(input, t);
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                pool.release(kryo);
                close(is);
                close(output);
            }
        }
        return null;
    }


    /**
     * 使用KryoPool SoftReferences创建Kryo
     * 把java对象序列化成byte[] ;
     * @param obj java对象
     * @return
     */
    public static <T>  byte[] serializePoolCallback (final T obj) {
        if(null != obj){
            try {
                return pool.run(new KryoCallback<byte[]>() {
                    public byte[] execute(Kryo kryo) {
                        ByteArrayOutputStream os = new ByteArrayOutputStream();
                        Output output = new Output(os);
                        kryo.writeObject(output,obj);
                        output.close();
                        try {
                            os.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        return os.toByteArray();
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    /**
     * 使用KryoPool SoftReferences创建Kryo
     * 把byte[]反序列化成指定的java对象
     * @param bytes
     * @return
     */
    public static <T> T unSerializePoolCallback(final byte[] bytes, final Class<T> t) {
        if(null != bytes && bytes.length>0 && null != t){
            try {
              return pool.run(new KryoCallback<T>() {
                    public T execute(Kryo kryo) {
                        ByteArrayInputStream is = new ByteArrayInputStream(bytes);
                        Input input = new Input(is);
                        T result =kryo.readObject(input,t);
                        input.close();
                        try {
                            is.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        return result;
                    }
              });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    /**
     * 关闭io流对象
     *
     * @param closeable
     */
    public static void close(Closeable closeable) {
        if (closeable != null) {
            try {
                closeable.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

```



参考官方文档：[https://github.com/EsotericSoftware/kryo](https://links.jianshu.com/go?to=https://github.com/EsotericSoftware/kryo "https://github.com/EsotericSoftware/kryo")


## FST序列化机制（钻石级别）


* FST（Fast-serialization-Tool），与kryo类似是apache组织的一个开源项目，完全兼容JDK序列化协议的系列化框架，序列化速度大概是JDK的4-10倍，体积更小，大小是JDK大小1/3左右，重新实现的 Java 快速对象序列化的开发包。    
* 相对来说是一个很新的序列化工具，速度于kryo有一些差距，在生产环境上的场景上测试，效果几乎于kryo一致，都能瞬间反序列化出内容并渲染。



**Java 快速序列化库 FST 已经发布了 2.0 版本，该版本的包名已经更改，无法平滑升级。另外官方建议为了稳定性考虑还是使用最新的 1.58 版本为好**


### Maven配置


```xml
<dependency>
       <groupId>de.ruedigermoeller</groupId>
       <artifactId>fst</artifactId>
       <version>1.58</version>
</dependency>

```


### 案例代码


```java
static FSTConfiguration configuration = FSTConfiguration
           .createDefaultConfiguration();

public static byte[] serialize(Object obj){
     return configuration.asByteArray((Serializable)obj);
}
public static Object deserialize(byte[] sec){
    return configuration.asObject(sec);
}

```



官方文档：[https://github.com/RuedigerMoeller/fast-serialization/wiki/Serialization](https://links.jianshu.com/go?to=https://github.com/RuedigerMoeller/fast-serialization/wiki/Serialization "https://github.com/RuedigerMoeller/fast-serialization/wiki/Serialization")


### protostuff（王者级别）


Protocol buffers是一个用来序列化结构化数据的技术，支持多种语言诸如C++、Java以及Python语言，可以使用该技术来持久化数据或者序列化成网络传输的数据。相比较一些其他的XML技术而言，该技术的一个明显特点就是更加节省空间（以二进制流存储）、速度更快以及更加灵活。


protostuff，是google在原来的protobuffer是的优化产品。使用起来也比较简单易用，目前效率也是最好的一种序列化工具。


```xml
 <dependency>
       <groupId>io.protostuff</groupId>
       <artifactId>protostuff-core</artifactId>
       <version>1.4.0</version>
     </dependency>
     <dependency>
       <groupId>io.protostuff</groupId>
       <artifactId>protostuff-runtime</artifactId>
       <version>1.4.0</version>
     </dependency>

```


#### protostuff工具类


```java
public class ProtostuffUtil {

    public static <T> byte[] serializer(T t){
        Schema schema = RuntimeSchema.getSchema(t.getClass());
        return ProtostuffIOUtil.toByteArray(t,schema,
                LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE));

    }

    public static <T> T deserializer(byte []bytes,Class<T> c) {
        T t = null;
        try {
            t = c.newInstance();
            Schema schema = RuntimeSchema.getSchema(t.getClass());
             ProtostuffIOUtil.mergeFrom(bytes,t,schema);
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return t;
    }
}

```


## Fastjson（钻石）


一个JSON库涉及的最基本功能就是序列化和反序列化。Fastjson支持java bean的直接序列化。 使用com.alibaba.fastjson.JSON这个类进行序列化和反序列化。


```java
public static String serialize(Object obj){
    String json = JSON.toJSONString(obj);
    return json;
}
public static Object deserialize(String json, Class<?> clazz){
    Object obj = JSON.parseObject(json, clazz);
    return obj;
}

```


#### Maven配置


```xml
 <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>fastjson</artifactId>
       <version>1.2.47</version>
  </dependency>

```


### Gson（钻石）


这里采用JSON格式同时使用采用Google的gson进行转义.


```java
static Gson gson = new Gson();

public static String serialize(Object obj){
    String json = gson.toJson(obj);
    return json;
}
public static Object deserialize(String json, Class<?> clazz){
    Object obj = gson.fromJson(json, clazz);
    return obj;
}

```


### Jackson（铂金）


Jackson库（[http://jackson.codehaus.org](https://links.jianshu.com/go?to=http://jackson.codehaus.org "http://jackson.codehaus.org")），是基于java语言的开源json格式解析工具，整个库（使用最新的2.2版本）包含3个jar包：


* jackson-core.jar——核心包（必须），提供基于“流模式”解析的API。    
* jackson-databind——数据绑定包（可选），提供基于“对象绑定”和“树模型”相关API。    
* jackson-annotations——注解包（可选），提供注解功能。


性能较高，“流模式”的解析效率超过绝大多数类似的json包。

核心包：JsonParser（json流读取），JsonGenerator（json流输出）。

数据绑定包：ObjectMapper（构建树模式和对象绑定模式），JsonNode（树节点）


```java
public static String serialize(Object obj){
    ObjectMapper mapper = new ObjectMapper();
    String json = null;
    try {
        json = mapper.writeValueAsString(obj);
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    return json;
}
public static Object deserialize(String json, Class<?> clazz){
    ObjectMapper mapper = new ObjectMapper();
    Object obj = null;
    try {
        obj = mapper.readValue(json, clazz);
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    return obj;
}

```


下表是几种方案的各项指标的一个对比







![](assets/8e52b417e737740583663f97bdd9b5a4.png)
