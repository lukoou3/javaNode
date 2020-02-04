## Jackson
### Jackson 简介
Jackson 是当前用的比较广泛的，用来序列化和反序列化 json 的 Java 的开源库。Spring MVC 的默认 json 解析器便是 Jackson。 Jackson 优点很多。 Jackson 所依赖的 jar 包较少 ，简单易用。

Jackson 的核心模块由三部分组成

jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。

jackson-annotations，注解包，提供标准注解功能；

jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

jackson-databind 依赖 jackson-core 和 jackson-annotations，当添加 jackson-databind 之后， jackson-core 和 jackson-annotations 也随之添加到 Java 项目工程中。在添加相关依赖包之后，就可以使用 Jackson。

### 引入依赖
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.0</version>
</dependency>
```
jackson-databind 依赖 jackson-core 和 jackson-annotations，当添加 jackson-databind 之后， jackson-core 和 jackson-annotations 也随之添加到 Java 项目工程中。在添加相关依赖包之后，就可以使用 Jackson。

### ObjectMapper 的基本使用
ObjectMapper只需创建一次即可，可以多次使用。writeValueAsString方法序列化，readValue方法反序列化。

```java
ObjectMapper mapper = new ObjectMapper(); 
Person person = new Person(); 
person.setName("Tom"); 
person.setAge(40); 
String jsonString = mapper.writeValueAsString(person); 
Person deserializedPerson = mapper.readValue(jsonString, Person.class);
```

### ObjectMapper 主要api
```java
public String writeValueAsString(Object value) throws JsonProcessingException
public <T> T readValue(String content, Class<T> valueType) throws JsonProcessingException, JsonMappingException
public <T> T readValue(String content, TypeReference<T> valueTypeRef) throws JsonProcessingException, JsonMappingException
```

### ObjectMapper 配置
在调用 writeValue 或调用 readValue 方法之前，往往需要设置 ObjectMapper 的相关配置信息。这些配置信息应用 java 对象的所有属性上。示例如下
```java
//在反序列化时忽略在 json 中存在但 Java 对象不存在的属性 
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,
   false); 
//在序列化时日期格式默认为 yyyy-MM-dd'T'HH:mm:ss.SSSZ 
mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,false) 
//在序列化时忽略值为 null 的属性 
mapper.setSerializationInclusion(Include.NON_NULL); 
//忽略值为默认值的属性 
mapper.setDefaultPropertyInclusion(Include.NON_DEFAULT);

// SerializationFeature for changing how JSON is written

// to enable standard indentation ("pretty-printing"):
mapper.enable(SerializationFeature.INDENT_OUTPUT);
// to allow serialization of "empty" POJOs (no properties to serialize)
// (without this setting, an exception is thrown in those cases)
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
// to write java.util.Date, Calendar as number (timestamp):
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

// DeserializationFeature for changing how JSON is read as POJOs:

// to prevent exception when encountering unknown property:
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
// to allow coercion of JSON empty String ("") to null Object value:
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

### 常用注解
Jackson 根据它的默认方式序列化和反序列化 java 对象，若根据实际需要，灵活的调整它的默认方式，可以使用 Jackson 的注解。常用的注解及用法如下。

| 注解               | 用法                                                                                                                                                  |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| @JsonProperty      | 用于属性，把属性的名称序列化时转换为另外一个名称。示例：@JsonProperty("birth_ d ate")private Date birthDate;                                          |
| @JsonFormat        | 用于属性或者方法，把属性的格式序列化时转换成指定的格式。示例：@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm")public Date getBirthDate() |
| @JsonIgnore        | 忽略该属性                                                                                                                                            |
| @JsonPropertyOrder | 用于类， 指定属性在序列化时 json 中的顺序 ， 示例：@JsonPropertyOrder({ "birth_Date", "name" })public class Person                                    |
| @JsonCreator       | 用于构造方法，和 @JsonProperty 配合使用，适用有参数的构造方法。 示例：@JsonCreatorpublic Person(@JsonProperty("name")String name) {…}                 |
| @JsonAnySetter     | 用于属性或者方法，设置未反序列化的属性名和值作为键值存储到 map 中@JsonAnySetterpublic void set(String key, Object value) {map.put(key, value);}       |
| @JsonAnyGetter     | 用于方法 ，获取所有未序列化的属性public Map<String, Object> any() { return map; }                                                                     |

### 日期格式化
springboot和springMVC中可以直接配置
```java
//时期格式化，springboot中配置spring.jackson.date-format即可
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
json = mapper.writeValueAsString(stu);
System.out.println(json);
```

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
import com.entity.*;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.FileUtils;
import org.junit.Test;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.*;

public class JacksonTest {

    @Test
    public void writeValueAsString() throws Exception {
        //创建一次，重复使用
        ObjectMapper mapper = new ObjectMapper(); // create once, reuse
        Student stu = new Student("燕青丝", 18, new Date());
        String json = mapper.writeValueAsString(stu);
        System.out.println(json);
        System.out.println(stu.getBirthday().getTime());

        //时期格式化，springboot中配置spring.jackson.date-format即可
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        json = mapper.writeValueAsString(stu);
        System.out.println(json);
    }

    @Test
    public void writeValueAsStringPretty() throws Exception {
        ObjectMapper mapper = new ObjectMapper(); // create once, reuse
        Student stu = new Student("燕青丝", 18, new Date());
        String json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(stu);
        System.out.println(json);

        //writerWithDefaultPrettyPrinter不会影响全局
        System.out.println(mapper.writeValueAsString(stu));
    }

    @Test
    public void writeNull() throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        //在序列化时忽略值为 null 的属性
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        Student stu = new Student(null, 18, null);
        String json = mapper.writeValueAsString(stu);
        System.out.println(json);
    }

    @Test
    public void writeValueAsStringList() throws Exception {
        List<Student> stus = new ArrayList<>();
        stus.add(new Student("燕青丝", 18, new Date()));
        stus.add(new Student("沐璇音", 19, new Date()));
        stus.add(new Student("苏流沙", 20, new Date()));

        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(stus);
        System.out.println(json);

        json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(stus);
        System.out.println(json);
    }

    @Test
    public void writeValueAsStringComplex() throws Exception {
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

        ObjectMapper mapper = new ObjectMapper();
        PageResult<Teacher> pageResult = new PageResult<>(100L, teachers);
        String json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(pageResult);
        System.out.println(json);

        System.out.println("------------------------------------------------------------");

        Result result = new Result(StatusCode.OK, true, "查询成功", pageResult);
        json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(result);
        System.out.println(json);

        System.out.println("------------------------------------------------------------");

        result = new Result(StatusCode.OK, true, "查询成功", teachers);
        json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(result);
        System.out.println(json);
    }

    @Test
    public void readValue() throws Exception {
        String json = "{\"age\":18,\"birthday\":1580725603492,\"name\":\"燕青丝\"}";
        ObjectMapper mapper = new ObjectMapper();
        Student student = mapper.readValue(json, Student.class);
        System.out.println(student);

        //单引号无法解析
        //json = "{'age':18,'birthday':1580725603492,'name':'燕青丝'}";

        json = "{\"age\":18,\"birthday\":\"2020-02-03 18:26:43\",\"name\":\"燕青丝\"}";
        //必须配置DateFormat才能解析，无法自动识别
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        student = mapper.readValue(json, Student.class);
        System.out.println(student);

        json = "{\"age\":18,\"birthday\":1580725603492,\"name\":\"燕青丝\"}";
        //配置DateFormat后照样可以解析毫秒时间戳
        student = mapper.readValue(json, Student.class);
        System.out.println(student);
    }

    @Test
    public void readValueArray() throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        String json = "[{\"age\":18,\"birthday\":1580726222830,\"name\":\"燕青丝\"},{\"age\":19,\"birthday\":1580726222830,\"name\":\"沐璇音\"},{\"age\":20,\"birthday\":1580726222830,\"name\":\"苏流沙\"}]";
        //解析json数组需要用泛型
        List<Student> students = mapper.readValue(json, new TypeReference<List<Student>>() {});
        System.out.println(students);

        //List中元素为LinkedHashMap
        List list = mapper.readValue(json, List.class);
        System.out.println(list);

        List<LinkedHashMap<String,Object>> stus = mapper.readValue(json, new TypeReference<List<LinkedHashMap<String,Object>>>() {});
        System.out.println(stus);
    }

    @Test
    public void readValueComplex() throws Exception {
        String path = getClass().getClassLoader().getResource("TeacherPageResult.json").getPath();
        String json = FileUtils.readFileToString(new File(path), "utf-8");
        ObjectMapper mapper = new ObjectMapper();

        PageResult<Teacher> pageResult = mapper.readValue(json, new TypeReference<PageResult<Teacher>>() {});
        List<Teacher> teachers = pageResult.getRows();
        for (Teacher teacher : teachers) {
            System.out.println(teacher);
        }
    }

    @Test
    public void readValueFieldNotExist() throws Exception {
        String json = "{\"age\":18,\"name\":\"燕青丝\"}";
        ObjectMapper mapper = new ObjectMapper();
        Student student = mapper.readValue(json, Student.class);
        System.out.println(student);

        json = "{\"age\":18,\"birthday\":1580725603492,\"name\":\"燕青丝\",\"info\":\"info\"}";
        //在反序列化时忽略在 json 中存在但 Java 对象不存在的属性，默认会抛出异常
        //mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // to prevent exception when encountering unknown property:
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        //configure(feature,state) 和 enable(feature)、disable(feature) 等价
        student = mapper.readValue(json, Student.class);
        System.out.println(student);
    }

}
```






