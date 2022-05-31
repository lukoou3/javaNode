# kryo序列化使用及踩坑过程
摘抄自：https://www.jianshu.com/p/be25792212ac

kryo序列化使用及采坑


1、kryo序列化使用过程


（1）、导入依赖


<dependency>


<groupId>com.esotericsoftware</groupId>


<artifactId>kryo</artifactId>


<version>5.0.0-RC4</version>


</dependency>


（2）、实现序列化和反序列的接口


接口：


```java
package com.simba.service;

public interface Serializer {

  /**
   * 序列化
   * @param object
   * @param bytes
   */
  void serializer(Object object,byte[] bytes);

  /**
   *
   * @param object
   * @param bytes
   * @param offset
   * @param count
   */
  void serializer(Object object,byte[] bytes,int offset,int count);

  /**
   * 反序列化
   * @param bytes
   * @param <T>
   * @return
   */
  <T> T deserializer(byte[] bytes);

  /**
   * 反序列化
   * @param bytes
   * @param offset
   * @param count
   * @param <T>
   * @return
   */
  <T> T deserializer(byte[] bytes,int offset,int count);
}


```


实现类：


```java
package com.simba.service.impl;

import com.esotericsoftware.kryo.Kryo;
import com.esotericsoftware.kryo.io.Input;
import com.esotericsoftware.kryo.io.Output;
import com.esotericsoftware.kryo.serializers.BeanSerializer;
import com.simba.service.Serializer;
import java.io.OutputStream;
import org.objenesis.strategy.StdInstantiatorStrategy;
import org.springframework.stereotype.Component;


/**
 * kryo实现序列化和反序列化接口
 * kryo不是线程安全的，需要注意，使用独立线程实现
 */
@Component
public class KryoSerializer implements Serializer {

  //将kryo对象存储在线程中，只有这个线程可以访问到，这样保证kryo的线程安全性，ThreadLocal(线程内部存储类)
  //通过get()&set()方法读取线程内的数据
  private final ThreadLocal<Kryo> kryoThreadLocal = new ThreadLocal<Kryo>(){
    @Override
    protected Kryo initialValue() {
      Kryo kryo = new Kryo();
      kryo.register(aClass, new BeanSerializer<>(kryo,aClass));
      //引用，对A对象序列化时，默认情况下kryo会在每个成员对象第一次序列化时写入一个数字，
      // 该数字逻辑上就代表了对该成员对象的引用，如果后续有引用指向该成员对象，
      // 则直接序列化之前存入的数字即可，而不需要再次序列化对象本身。
      // 这种默认策略对于成员存在互相引用的情况较有利，否则就会造成空间浪费
      // （因为没序列化一个成员对象，都多序列化一个数字），
      // 通常情况下可以将该策略关闭，kryo.setReferences(false);
      kryo.setReferences(false);
      //设置是否注册全限定名，
      kryo.setRegistrationRequired(false);
      //设置初始化策略，如果没有默认无参构造器，那么就需要设置此项,使用此策略构造一个无参构造器
      kryo.setInstantiatorStrategy(new StdInstantiatorStrategy());
      return kryo;
    }
  };

  private final  ThreadLocal<Output> outputThreadLocal = new ThreadLocal<>();

  private final  ThreadLocal<Input> inputThreadLocal = new ThreadLocal<>();

  private Class<?> aClass = null;


  public Class<?> getaClass() {
    return aClass;
  }

  public void setaClass(Class<?> aClass) {
    this.aClass = aClass;
  }

  @Override
  public void serializer(Object object, byte[] bytes) {
    Kryo kryo = kryoThreadLocal.get();
    Output output =getOutPut(bytes);
    kryo.writeObjectOrNull(output,object,object.getClass());
    output.flush();
  }

  @Override
  public void serializer(Object object, byte[] bytes, int offset, int count) {
    Kryo kryo = kryoThreadLocal.get();
    Output output =getOutPut(bytes,offset,count);
    kryo.writeObjectOrNull(output,object,object.getClass());
    output.flush();
  }

  @Override
  public <T> T deserializer(byte[] bytes) {
    Kryo kryo =  kryoThreadLocal.get();
    Input input = getInPut(bytes);
    return (T)kryo.readObjectOrNull(input,aClass);
  }

  @Override
  public <T> T deserializer(byte[] bytes, int offset, int count){
    Kryo kryo = kryoThreadLocal.get();
    Input input = getInPut(bytes,offset,count);
    return (T)kryo.readObjectOrNull(input,aClass);
  }

  private Output getOutPut(byte[] bytes){
    Output output = outputThreadLocal.get();
    if(output == null){
      output = new Output();
      outputThreadLocal.set(new Output());
    }
    if(bytes!=null){
      output.setBuffer(bytes);
    }
    return output;
  }
  private Output getOutPut(byte[] bytes,int offset,int count){
    Output output = outputThreadLocal.get();
    if(output == null){
      output = new Output();
      outputThreadLocal.set(new Output());
    }
    if(bytes!=null){
      output.writeBytes(bytes,offset,count);
    }
    return output;
  }
  private Input getInPut(byte[] bytes){
    Input input = inputThreadLocal.get();
    if(input == null){
      input= new Input();
      outputThreadLocal.set(new Output());
    }
    if(bytes!=null){
      input.setBuffer(bytes);
    }
    return input;
  }

  private Input getInPut(byte[] bytes,int offser,int count){
    Input input = inputThreadLocal.get();
    if(input == null){
      input= new Input();
      outputThreadLocal.set(new Output());
    }
    if(bytes!=null){
      input.setBuffer(bytes,offser,count);
    }
    return input;
  }
}


```


（3）、测试代码


测试代码：


```kotlin
package com.simba.controller;


import com.simba.model.SubTestSerialization;
import com.simba.model.TestSerialization;
import com.simba.service.impl.KryoSerializer;
import java.util.Arrays;
import org.junit.Test;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/kryo")
/**
 * kryo序列化类，是有
 */
public class KryoController {

  @RequestMapping("/test")
  public String test(){
    return "test";
  }

  @Test
  public void kayoSerializer(){
    byte[] bytes =new byte[200];
    String[] strings = {"s","s1"};
    System.out.println(Arrays.toString(bytes));
    KryoSerializer kryoSerializer = new KryoSerializer();
    kryoSerializer.setaClass(TestSerialization.class);
    TestSerialization testSerialization = new TestSerialization();
    testSerialization.setText("aaaaasdwe");
    testSerialization.setName("f");
    testSerialization.setId(999);
    testSerialization.setFlag(false);
    testSerialization.setList(Arrays.asList(strings));
    SubTestSerialization subTestSerialization = new SubTestSerialization();
    subTestSerialization.setName("test");
    testSerialization.setSubTestSerialization(subTestSerialization);

    kryoSerializer.serializer(testSerialization,bytes);
    System.out.println(testSerialization.toString());
    System.out.println(Arrays.toString(bytes));
    System.out.println("=====================================");
    TestSerialization testSerialization1 = kryoSerializer.deserializer(bytes);
    System.out.println(testSerialization1.toString());
    System.out.println(Arrays.toString(bytes));
  }

}


```


2、踩坑过程


（1）、com.esotericsoftware.kryo.KryoException: java.lang.NullPointerException Serialization trace:


list (com.simba.model.TestSerialization)


此问题出现的原因是类TestSerialization中有ArrayList的属性list，导致在解码的过程中程序无法解析


解决方案：


使用最新版的pom文件即可，现阶段官方文档（[https://github.com/EsotericSoftware/kryo/blob/master/README.md](https://links.jianshu.com/go?to=https://github.com/EsotericSoftware/kryo/blob/master/README.md "https://github.com/EsotericSoftware/kryo/blob/master/README.md")）


中描述的最新的pom文件是<version>5.0.0-RC4</version>。


（2）、在初始kryo的时候，一定要注意kryo的各种配置


最重要的单个配置为：（详细的说明在代码中有说明）


kryo.setReferences(false);


kryo.setRegistrationRequired(false);


kryo.setInstantiatorStrategy(new StdInstantiatorStrategy());


（3）、其他问题可以直接看官网文档


（[https://github.com/EsotericSoftware/kryo/blob/master/README.md](https://links.jianshu.com/go?to=https://github.com/EsotericSoftware/kryo/blob/master/README.md "https://github.com/EsotericSoftware/kryo/blob/master/README.md")）

