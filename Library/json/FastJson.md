## FastJson
`https://www.cnblogs.com/jajian/p/10051901.html#autoid-9-3-0`

### FastJson简介
#### 什么是fastjson?
fastjson是阿里的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。

#### fastjson的优点
##### 1 速度快
fastjson相对其他JSON库的特点是快，从2011年fastjson发布1.1.x版本之后，其性能从未被其他Java实现的JSON库超越。

##### 2 使用广泛
fastjson在阿里巴巴大规模使用，在数万台服务器上部署，fastjson在业界被广泛接受。在2012年被开源中国评选为最受欢迎的国产开源软件之一。

##### 3 测试完备
fastjson有非常多的testcase，在1.2.11版本中，testcase超过3321个。每次发布都会进行回归测试，保证质量稳定。

##### 4 使用简单
fastjson的API十分简洁。
```
String text = JSON.toJSONString(obj); //序列化
VO vo = JSON.parseObject("{...}", VO.class); //反序列化
```
##### 5 功能完备
支持泛型，支持流处理超大文本，支持枚举，支持序列化和反序列化扩展。

### 下载和使用
可以在maven中央仓库中直接下载jar：
```
https://repo1.maven.org/maven2/com/alibaba/fastjson/
```

或者配置maven依赖
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.58</version>
</dependency>
```

### FastJson核心类和主要的API
三个核心类：  
* JSON：fastjson的解析器，用于json字符串和javaBean、Json对象的转换    
* JSONObject：fastJson提供的json对象，实现`Map<String, Object>`接口    
* JSONArray：fastJson提供json数组对象，实现`List<Object>`接口  

最重要的API：
```java
// Java对象转换为JSON字符串
public static final String toJSONString(Object object);
//JSON字符串转换为Java对象
public static final <T> T parseObject(String text, Class<T> clazz, Feature... features);
```

序列化：
```java
String jsonString = JSON.toJSONString(obj);
```
反序列化：
```java
VO vo = JSON.parseObject("...", VO.class);
```
泛型反序列化：
```java
import com.alibaba.fastjson.TypeReference;

List<VO> list = JSON.parseObject("...", new TypeReference<List<VO>>() {});

//这种方式还是挺有用的
PageResult<Teacher> pageResult = JSON.parseObject(json, new TypeReference<PageResult<Teacher>>() {});

public class PageResult<T> {
   //总记录数
   private Long total;
   private List<T> rows;
}
```

主要的API：
```java
// 把JSON文本parse为JSONObject或者JSONArray 
public static final Object parse(String text); 

// 把JSON文本parse成JSONObject 
public static final JSONObject parseObject(String text)；  
// 把JSON文本parse为JavaBean 
public static final <T> T parseObject(String text, Class<T> clazz);

// 把JSON文本parse成JSONArray 
public static final JSONArray parseArray(String text); 
//把JSON文本parse成JavaBean集合 
public static final <T> List<T> parseArray(String text, Class<T> clazz); 

// 将JavaBean序列化为JSON文本 
public static final String toJSONString(Object object); 
// 将JavaBean序列化为带格式的JSON文本(漂亮地打印)
public static final String toJSONString(Object object, boolean prettyFormat); 
//将JavaBean转换为JSONObject或者JSONArray。
public static final Object toJSON(Object javaObject); 

// 日期格式化
public static String toJSONStringWithDateFormat(Object object, String dateFormat, SerializerFeature... features)

// 更多的定制格式化
public static String toJSONString(Object object, SerializeFilter filter, SerializerFeature... features)
public static String toJSONString(Object object, SerializeFilter[] filters, SerializerFeature... features)
```

### 将对象中的空值输出
在fastjson中，缺省是不输出空值的。无论Map中的null和对象属性中的null，序列化的时候都会被忽略不输出，这样会减少产生文本的大小。但如果需要输出空值怎么做呢？

如果你需要输出空值，需要使用 `SerializerFeature.WriteMapNullValue`
```java
Model obj = ...;
JSON.toJSONString(obj, SerializerFeature.WriteMapNullValue);
```

几种空值特别处理方式：  
| SerializerFeature       | 描述                                    |
| ----------------------- | --------------------------------------- |
| WriteNullListAsEmpty    | 将Collection类型字段的字段空值输出为[]  |
| WriteNullStringAsEmpty  | 将字符串类型字段的空值输出为空字符串 "" |
| WriteNullNumberAsZero   | 将数值类型字段的空值输出为0             |
| WriteNullBooleanAsFalse | 将Boolean类型字段的空值输出为false      |

具体的示例参考如下，可以同时选择多个：
```java
class Model {
      public List<Objec> items;
}

Model obj = ....;

String text = JSON.toJSONString(obj, SerializerFeature.WriteMapNullValue, SerializerFeature.WriteNullListAsEmpty);
```

### Fastjson 处理日期
Fastjson 处理日期的API很简单，例如：
```java
JSON.toJSONStringWithDateFormat(date, "yyyy-MM-dd HH:mm:ss.SSS")
```

使用ISO-8601日期格式
```java
JSON.toJSONString(obj, SerializerFeature.UseISO8601DateFormat);
```

全局修改日期格式
```java
JSON.DEFFAULT_DATE_FORMAT = "yyyy-MM-dd";
JSON.toJSONString(obj, SerializerFeature.WriteDateUseDateFormat);
```

反序列化能够自动识别如下日期格式：

* ISO-8601日期格式    
* yyyy-MM-dd    
* yyyy-MM-dd HH:mm:ss    
* yyyy-MM-dd HH:mm:ss.SSS    
* 毫秒数字    
* 毫秒数字字符串    
* .NET JSON日期格式    
* new Date(198293238)  

虽然上面处理了单个的日期类型和全局的日期类型格式的配置，但是有时候我们需要的是对象中个别的日期类型差异化，并不一定是同一种格式的。那如何处理呢？接下来介绍 Fastjson 的定制序列化。

### Fastjson 定制序列化
#### 简介
fastjson支持多种方式定制序列化。

* 通过@JSONField定制序列化    
* 通过@JSONType定制序列化    
* 通过SerializeFilter定制序列化    
* 通过ParseProcess定制反序列化  

#### 使用@JSONField配置
##### JSONField 注解介绍
```java
package com.alibaba.fastjson.annotation;

public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.42版本之后才支持
    int ordinal() default 0;

     // 指定字段的名称
    String name() default "";

    // 指定字段的格式，对日期格式有用
    String format() default "";

    // 是否序列化
    boolean serialize() default true;

    // 是否反序列化
    boolean deserialize() default true;
}
```

##### JSONField配置方式
可以把@JSONField配置在字段或者getter/setter方法上，例如：
配置在字段上
```java
public class VO {
     @JSONField(name="ID")
     private int id;

     @JSONField(name="birthday",format="yyyy-MM-dd")
     public Date date;
}
```

##### 使用format配置日期格式化
可以定制化配置各个日期字段的格式化
```java
public class A {
     // 配置date序列化和反序列使用yyyyMMdd日期格式
     @JSONField(format="yyyyMMdd")
     public Date date;
}
```

##### 使用serialize/deserialize指定字段不序列化
```java
public class A {
    @JSONField(serialize=false)
    public Date date;
}

public class A {
    @JSONField(deserialize=false)
    public Date date;
}
```

##### 使用ordinal指定字段的顺序
缺省Fastjson序列化一个java bean，是根据fieldName的字母序进行序列化的，你可以通过ordinal指定字段的顺序。这个特性需要1.1.42以上版本。
```java
public static class VO {
    @JSONField(ordinal = 3)
    private int f0;

    @JSONField(ordinal = 2)
    private int f1;

    @JSONField(ordinal = 1)
    private int f2;
}
```

##### 使用serializeUsing制定属性的序列化类
在fastjson 1.2.16版本之后，JSONField支持新的定制化配置serializeUsing，可以单独对某一个类的某个属性定制序列化，比如：
```java
public static class Model {
    @JSONField(serializeUsing = ModelValueSerializer.class)
    public int value;
}

public static class ModelValueSerializer implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType,
                      int features) throws IOException {
        Integer value = (Integer) object;
        String text = value + "元";
        serializer.write(text);
    }
}
```
测试代码
```java
Model model = new Model();
model.value = 100;
String json = JSON.toJSONString(model);
Assert.assertEquals("{\"value\":\"100元\"}", json);
```

#### 使用@JSONType配置
和JSONField类似，但JSONType配置在类上，而不是field或者getter/setter方法上。

#### 通过SerializeFilter定制序列化
##### 简介
SerializeFilter是通过编程扩展的方式定制序列化。fastjson支持6种SerializeFilter，用于不同场景的定制序列化。

* PropertyPreFilter 根据PropertyName判断是否序列化    
* PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化    
* NameFilter 修改Key，如果需要修改Key,process返回值则可    
* ValueFilter 修改Value    
* BeforeFilter 序列化时在最前添加内容    
* AfterFilter 序列化时在最后添加内容  

##### PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化
```java
public interface PropertyFilter extends SerializeFilter {
   boolean apply(Object object, String propertyName, Object propertyValue);
}
```
可以通过扩展实现根据object或者属性名称或者属性值进行判断是否需要序列化。例如：
```java
PropertyFilter filter = new PropertyFilter() {

    public boolean apply(Object source, String name, Object value) {
        if ("id".equals(name)) {
            int id = ((Integer) value).intValue();
            return id >= 100;
        }
        return false;
    }
};

JSON.toJSONString(obj, filter); // 序列化的时候传入filter
```

##### PropertyPreFilter 根据PropertyName判断是否序列化
和PropertyFilter不同只根据object和name进行判断，在调用getter之前，这样避免了getter调用可能存在的异常。
```java
public interface PropertyPreFilter extends SerializeFilter {
     boolean apply(JSONSerializer serializer, Object object, String name);
}
```

##### NameFilter 序列化时修改Key
如果需要修改Key,process返回值则可
```java
public interface NameFilter extends SerializeFilter {
    String process(Object object, String propertyName, Object propertyValue);
}
```
fastjson内置一个PascalNameFilter，用于输出将首字符大写的Pascal风格。 例如：
```java
import com.alibaba.fastjson.serializer.PascalNameFilter;

Object obj = ...;
String jsonStr = JSON.toJSONString(obj, new PascalNameFilter());
```

##### ValueFilter 序列化时修改Value
```java
public interface ValueFilter extends SerializeFilter {
  Object process(Object object, String propertyName, Object propertyValue);
}
```

##### BeforeFilter 序列化时在最前添加内容
在序列化对象的所有属性之前执行某些操作,例如调用 writeKeyValue 添加内容
```java
public abstract class BeforeFilter implements SerializeFilter {
   protected final void writeKeyValue(String key, Object value) { ... }
    // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
    public abstract void writeBefore(Object object);
}
```

##### AfterFilter 序列化时在最后添加内容
在序列化对象的所有属性之后执行某些操作,例如调用 writeKeyValue 添加内容
```java
public abstract class AfterFilter implements SerializeFilter {
 protected final void writeKeyValue(String key, Object value) { ... }
   // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
   public abstract void writeAfter(Object object);
}
```

#### 通过ParseProcess定制反序列化
##### 简介
ParseProcess是编程扩展定制反序列化的接口。fastjson支持如下ParseProcess：

* ExtraProcessor 用于处理多余的字段    
* ExtraTypeProvider 用于处理多余字段时提供类型信息  

##### 使用ExtraProcessor 处理多余字段
```java
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}
    
ExtraProcessor processor = new ExtraProcessor() {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }
};
    
VO vo = JSON.parseObject("{\"id\":123,\"name\":\"abc\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals("abc", vo.getAttributes().get("name"));
```

##### 使用ExtraTypeProvider 为多余的字段提供类型
```java
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}
    
class MyExtraProcessor implements ExtraProcessor, ExtraTypeProvider {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }
    
    public Type getExtraType(Object object, String key) {
        if ("value".equals(key)) {
            return int.class;
        }
        return null;
    }
};
ExtraProcessor processor = new MyExtraProcessor();
    
VO vo = JSON.parseObject("{\"id\":123,\"value\":\"123456\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals(123456, vo.getAttributes().get("value")); // value本应该是字符串类型的，通过getExtraType的处理变成Integer类型了。
```

### 在 Spring MVC 中集成 Fastjson
如果你使用 Spring MVC 来构建 Web 应用并对性能有较高的要求的话，可以使用 Fastjson 提供的FastJsonHttpMessageConverter 来替换 Spring MVC 默认的 HttpMessageConverter 以提高 @RestController @ResponseBody @RequestBody 注解的 JSON序列化速度。下面是配置方式，非常简单。

#### XML式
如果是使用 XML 的方式配置 Spring MVC 的话，只需在 Spring MVC 的 XML 配置文件中加入下面配置即可
```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter"/>      
    </mvc:message-converters>
</mvc:annotation-driven>
```

通常默认配置已经可以满足大部分使用场景，如果你想对它进行自定义配置的话，你可以添加 FastJsonConfig Bean。
```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
            <property name="fastJsonConfig" ref="fastJsonConfig"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="fastJsonConfig" class="com.alibaba.fastjson.support.config.FastJsonConfig">
    <!--   自定义配置...   -->
</bean>
```

#### 编程式
如果是使用编程的方式（通常是基于 Spring Boot 项目）配置 Spring MVC 的话只需继承 WebMvcConfigurerAdapter覆写configureMessageConverters方法即可，就像下面这样。
```java
@Configuration
public class WebMvcConfigurer extends WebMvcConfigurerAdapter {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        //自定义配置...
        //FastJsonConfig config = new FastJsonConfig();
        //config.set ...
        //converter.setFastJsonConfig(config);
        converters.add(0, converter);
    }
}
```

```
注意：
1、如果你使用的 Fastjson 版本小于1.2.36的话(强烈建议使用最新版本)，在与Spring MVC 4.X 版本集成时需使用 FastJsonHttpMessageConverter4。

2、SpringBoot 2.0.1版本中加载WebMvcConfigurer的顺序发生了变动，故需使用converters.add(0, converter);指定FastJsonHttpMessageConverter在converters内的顺序，否则在SpringBoot 2.0.1及之后的版本中将优先使用Jackson处理。
```

### 在 Spring Data Redis 中集成 Fastjson
通常我们在 Spring 中使用 Redis 是通过 Spring Data Redis 提供的 RedisTemplate 来进行的，如果你准备使用 JSON 作为对象序列/反序列化的方式并对序列化速度有较高的要求的话，建议使用 Fastjson 提供的 GenericFastJsonRedisSerializer 或 FastJsonRedisSerializer 作为 RedisTemplate 的 RedisSerializer。下面是配置方式，非常简单。

#### XML式
如果是使用 XML 的方式配置 Spring Data Redis 的话，只需将 RedisTemplate 中的 Serializer 替换为 GenericFastJsonRedisSerializer 即可。
```xml
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jedisConnectionFactory"/>
    <property name="defaultSerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
</bean>
```
下面是完整的 Spring 集成 Redis 配置供参考。
```xml
<!-- Redis 连接池配置(可选) -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxTotal" value="${redis.pool.maxActive}"/>
    <property name="maxIdle" value="${redis.pool.maxIdle}"/>
    <property name="maxWaitMillis" value="${redis.pool.maxWait}"/>
    <property name="testOnBorrow" value="${redis.pool.testOnBorrow}"/>
     <!-- 更多连接池配置...-->
</bean>
<!-- Redis 连接工厂配置 -->
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <!--设置连接池配置，不设置的话会使用默认的连接池配置，若想禁用连接池可设置 usePool = false -->   
    <property name="poolConfig" ref="jedisPoolConfig" />  
    <property name="hostName" value="${host}"/>
    <property name="port" value="${port}"/>
    <property name="password" value="${password}"/>
    <property name="database" value="${database}"/>
    <!-- 更多连接工厂配置...-->
</bean>
<!-- RedisTemplate 配置 -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <!-- 设置 Redis 连接工厂-->
    <property name="connectionFactory" ref="jedisConnectionFactory"/>
    <!-- 设置默认 Serializer ，包含 keySerializer & valueSerializer -->
    <property name="defaultSerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
    <!-- 单独设置 keySerializer -->
    <property name="keySerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
    <!-- 单独设置 valueSerializer -->
    <property name="valueSerializer">
        <bean class="com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer"/>
    </property>
</bean>
```

#### 编程式
如果是使用编程的方式（通常是基于 Spring Boot 项目）配置 RedisTemplate 的话只需在你的配置类(被@Configuration注解修饰的类)中显式创建 RedisTemplate Bean，设置 Serializer 即可。
```java
@Bean
public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate redisTemplate = new RedisTemplate();
    redisTemplate.setConnectionFactory(redisConnectionFactory);

    GenericFastJsonRedisSerializer fastJsonRedisSerializer = new GenericFastJsonRedisSerializer();
    redisTemplate.setDefaultSerializer(fastJsonRedisSerializer);//设置默认的Serialize，包含 keySerializer & valueSerializer

    //redisTemplate.setKeySerializer(fastJsonRedisSerializer);//单独设置keySerializer
    //redisTemplate.setValueSerializer(fastJsonRedisSerializer);//单独设置valueSerializer
    return redisTemplate;
}
```
通常使用 GenericFastJsonRedisSerializer 即可满足大部分场景，如果你想定义特定类型专用的 RedisTemplate 可以使用 FastJsonRedisSerializer来代替 GenericFastJsonRedisSerializer，配置是类似的。


### 测试
#### entity
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    private String name;
    private int age;
    private Date birthday;
}
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Teacher {
    private String name;
    private int age;
    private Date birthday;
    private List<Student> students;
}
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class PageResult<T> {
   //总记录数
   private Long total;
   private List<T> rows;
}
```
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {
   //结果状态码
   private Integer code;
   //成功标记
   private Boolean flag;
   //请求结果信息
   private String message;
   //如果是查询=>在该属性中表达
   private Object data;

   public Result(Integer code, Boolean flag, String message) {
      this.code = code;
      this.flag = flag;
      this.message = message;
   }
}
```

#### 测试代码
```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import com.alibaba.fastjson.serializer.*;
import com.entity.*;
import org.apache.commons.io.FileUtils;
import org.junit.Assert;
import org.junit.Test;

import java.io.File;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.List;

public class FastJsonTest {
    private String dateFormat = "yyyy-MM-dd HH:mm:ss";

    @Test
    public void toJSONString() {
        Student stu = new Student();
        stu.setName("燕青丝");
        stu.setAge(18);
        stu.setBirthday(new Date());

        String stuJson = JSON.toJSONString(stu);
        System.out.println(stuJson);
        System.out.println(stu.getBirthday().getTime());

        //Date格式化输出
        stuJson = JSON.toJSONStringWithDateFormat(stu, dateFormat);
        System.out.println(stuJson);
    }

    @Test
    public void toJSONStringNullFormat() {
        Student stu = new Student(null, 18, null);
        //默认null不会输出
        String stuJson = JSON.toJSONString(stu);
        System.out.println(stuJson);

        //{"age":18,"birthday":null,"name":null}
        stuJson = JSON.toJSONString(stu, SerializerFeature.WriteMapNullValue);
        System.out.println(stuJson);

        //{"age":18,"birthday":null,"name":""}
        stuJson = JSON.toJSONString(stu, SerializerFeature.WriteNullStringAsEmpty);
        System.out.println(stuJson);
    }

    @Test
    public void toJSONStringPretty() {
        Student stu = new Student("燕青丝", 18, new Date());
        //prettyFormat：漂亮地格式化json
        //调用的是toJSONString(object, SerializerFeature.PrettyFormat)
        String stuJson = JSON.toJSONString(stu, true);
        System.out.println(stuJson);
    }

    @Test
    public void toJSONStringList() {
        List<Student> stus = new ArrayList<>();
        stus.add(new Student("燕青丝", 18, new Date()));
        stus.add(new Student("沐璇音", 19, new Date()));
        stus.add(new Student("苏流沙", 20, new Date()));

        String stuJson = JSON.toJSONString(stus);
        System.out.println(stuJson);
        stuJson = JSON.toJSONString(stus, true);
        System.out.println(stuJson);
    }

    @Test
    public void toJSONStringPropertyFilter() {
        List<Student> stus = new ArrayList<>();
        stus.add(new Student("燕青丝", 18, new Date()));
        stus.add(new Student("沐璇音", 19, new Date()));
        stus.add(new Student("苏流沙", 20, new Date()));
        Teacher teacher = new Teacher("莫南", 18, new Date(), stus);

        String teacherJson = JSON.toJSONString(teacher);
        System.out.println(teacherJson);
        teacherJson = JSON.toJSONString(teacher, true);
        System.out.println(teacherJson);

        System.out.println("------------------------------------------------------------");
        /**
         * PropertyFilter：属性过滤器
         * Object object：属性所属的对象
         * String name：属性的name
         * Object value：属性的值
         */
        PropertyFilter propertyFilter = (Object object, String name, Object value) -> {
            //System.out.println(object.getClass() + " ," + name + " ," + value);
            //Teacher的birthday不输出
            if (object instanceof Teacher && name.equals("birthday")) {
                return false;
            }
            //Student的age不输出
            if (object instanceof Student && name.equals("age")) {
                return false;
            }
            return true;
        };
        teacherJson = JSON.toJSONString(teacher, propertyFilter, SerializerFeature.PrettyFormat);
        System.out.println(teacherJson);
    }

    @Test
    public void toJSONStringNameFilterValueFilter() {
        Student stu = new Student("燕青丝", 18, new Date());
        //NameFilter 修改Key，如果需要修改Key,process返回值则可
        //String process(Object object, String name, Object value);
        NameFilter nameFilter = (obj, name, value) -> name + "1";
        //ValueFilter 修改Value
        //Object process(Object object, String name, Object value);
        ValueFilter valueFilter = (obj, name, value) -> "2" + value;

        String json = JSON.toJSONString(stu, new SerializeFilter[]{nameFilter, valueFilter}, SerializerFeature.PrettyFormat);
        System.out.println(json);
    }


    @Test
    public void toJSONStringComplex() {
        List<Teacher> teachers = new ArrayList<>();
        teachers.add(new Teacher("莫南", 18, new Date(), Arrays.asList(
                new Student("燕青丝", 18, new Date()),
                new Student("沐璇音", 18, new Date()),
                new Student("苏流沙", 20, new Date())
        )));
        teachers.add(new Teacher("莫扶苏", 18, new Date(), Arrays.asList(
                new Student("洛夕也", 18, new Date()),
                new Student("倾天达", 18, new Date()),
                new Student("轻轻寒", 20, new Date())
        )));

        PageResult<Teacher> pageResult = new PageResult<>(100L, teachers);
        String json = JSON.toJSONString(pageResult, true);
        System.out.println(json);

        System.out.println("------------------------------------------------------------");

        Result result = new Result(StatusCode.OK, true, "查询成功", pageResult);
        json = JSON.toJSONString(result, true);
        System.out.println(json);

        System.out.println("------------------------------------------------------------");

        result = new Result(StatusCode.OK, true, "查询成功", teachers);
        json = JSON.toJSONString(result, true);
        System.out.println(json);
    }

    @Test
    public void parseObject() {
        String json = "{\"age\":18,\"birthday\":1580725603492,\"name\":\"燕青丝\"}";
        Student student = JSON.parseObject(json, Student.class);
        System.out.println(student);

        //单引号也可以解析
        json = "{'age':18,'birthday':1580725603492,'name':'燕青丝'}";
        student = JSON.parseObject(json, Student.class);
        System.out.println(student);

        /**
         * 反序列化能够自动识别如下日期格式：
         * ISO-8601日期格式
         * yyyy-MM-dd
         * yyyy-MM-dd HH:mm:ss
         * yyyy-MM-dd HH:mm:ss.SSS
         * 毫秒数字
         * 毫秒数字字符串
         * .NET JSON日期格式
         * new Date(198293238)
         */
        json = "{\"age\":18,\"birthday\":\"2020-02-03 18:26:43\",\"name\":\"燕青丝\"}";
        student = JSON.parseObject(json, Student.class);
        System.out.println(student);
    }

    @Test
    public void parseObjectFieldNotExist() {
        String json = "{\"age\":18,\"birthday\":1580725603492,\"name\":\"燕青丝\",\"info\":\"info\"}";
        //默认会忽略在 json 中存在但 Java 对象不存在的属性，不像Jackson那样抛出异常
        Student student = JSON.parseObject(json, Student.class);
        System.out.println(student);
    }

    @Test
    public void parseArray() {
        String json = "[{\"age\":18,\"birthday\":1580726222830,\"name\":\"燕青丝\"},{\"age\":19,\"birthday\":1580726222830,\"name\":\"沐璇音\"},{\"age\":20,\"birthday\":1580726222830,\"name\":\"苏流沙\"}]";
        //JSONArray类型
        Object obj = JSON.parse(json);
        System.out.println(obj);

        //解析数组用parseArray
        List<Student> students = JSON.parseArray(json, Student.class);
        System.out.println(students);
    }

    @Test
    public void parseObjectComplex() throws Exception {
        String path = getClass().getClassLoader().getResource("TeacherPageResult.json").getPath();
        String json = FileUtils.readFileToString(new File(path), "utf-8");
        PageResult pageResult = JSON.parseObject(json, PageResult.class);
        //System.out.println(pageResult);

        //因为pageResult中List<T> rows用的泛型，无法判断类型，只能用JSONObject
        List<JSONObject> teachers = pageResult.getRows();
        for (JSONObject jsonObject : teachers) {
            Teacher teacher = jsonObject.toJavaObject(Teacher.class);
            System.out.println(teacher);
        }
    }

    @Test
    public void parseObjectComplex2() throws Exception {
        String path = getClass().getClassLoader().getResource("TeacherPageResult.json").getPath();
        String json = FileUtils.readFileToString(new File(path), "utf-8");
        TeacherPageResult pageResult = JSON.parseObject(json, TeacherPageResult.class);

        //再定义一个类明确类型就好了List<Teacher> rows;
        List<Teacher> teachers = pageResult.getRows();
        for (Teacher teacher : teachers) {
            System.out.println(teacher);
        }
    }

    @Test
    public void parseObjectComplex3() throws Exception {
        String path = getClass().getClassLoader().getResource("TeacherPageResult.json").getPath();
        String json = FileUtils.readFileToString(new File(path), "utf-8");
        //也可以不额外定义类
        PageResult<Teacher> pageResult = JSON.parseObject(json, new TypeReference<PageResult<Teacher>>() {});

        List<Teacher> teachers = pageResult.getRows();
        for (Teacher teacher : teachers) {
            System.out.println(teacher);
        }
    }

    @Test
    public void jsonObject() {
        String json = "{\"age\":18,\"birthday\":1580725603492,\"name\":\"燕青丝\"}";
        //JSONObject implements Map<String, Object>
        JSONObject jsonObject = JSON.parseObject(json);

        System.out.println(jsonObject.keySet());
        Assert.assertEquals(JSON.toJSONString(jsonObject), jsonObject.toJSONString());


        //JSONObject转Student
        Student student = jsonObject.toJavaObject(Student.class);
        System.out.println(student);
    }

    @Test
    public void jsonArray() {
        String json = "[{\"age\":18,\"birthday\":1580726222830,\"name\":\"燕青丝\"},{\"age\":19,\"birthday\":1580726222830,\"name\":\"沐璇音\"},{\"age\":20,\"birthday\":1580726222830,\"name\":\"苏流沙\"}]";
        //JSONArray implements List<Object>
        JSONArray jsonArray = JSON.parseArray(json);

        System.out.println(jsonArray.get(0));

        System.out.println(jsonArray.toJSONString());
    }

}
```


