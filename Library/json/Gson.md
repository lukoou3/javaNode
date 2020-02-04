## Gson
### Gson简介
Gson 是google解析Json的一个开源类库，它可以将Java对象转换为相应的JSON形式，也可以将JSON字符串转换为对应的Java对象。

#### 引入依赖
```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

#### Gson对象
在进行序列化与反序列操作前，需要先实例化一个 com .google.gson.Gson 对象，获取 Gson 对象的方法有两种
```java
//通过构造函数来获取
Gson gson = new Gson();
//通过 GsonBuilder 来获取，可以进行多项特殊配置
Gson gson = new GsonBuilder().create(); 
```

Gson提供了**fromJson()** 和**toJson()** 两个直接用于解析和生成的方法，前者实现反序列化，后者实现了序列化；同时每个方法都提供了重载方法

### Gson主要API
```java
//序列化
public String toJson(Object src);
//反序列化
public <T> T fromJson(String json, Class<T> classOfT);
//解决泛型类型擦除的问题：new TypeToken<List<Student>>(){}.getType()
public <T> T fromJson(String json, Type typeOfT);
```

### Gson注解
#### @SerializedName 属性重命名
value属性配置属性映射，当需要兼容多个属性名称时可以使用alternate属性。
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD})
public @interface SerializedName {
    String value();

    String[] alternate() default {};
}
```

#### @Expose注解
此功能提供了一种方法，您可以将要排除的对象的某些字段标记为序列化和反序列化为 JSON。要使用此批注，必须使用 `new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()`创建的 Gson 实例将排除类中未标注@Expose注释的所有字段。

Expose 注解包含两个属性值，且均声明了默认值。Expose 的含义即为“暴露”，即用于对外暴露字段，serialize 用于指定是否进行序列化，deserialize 用于指定是否进行反序列化。如果字段不声明 Expose 注解，则意味着不进行序列化和反序列化操作，相当于两个属性值均为 false 。此外，Expose 注解需要和 GsonBuilder 构建的 Gson 对象一起使用才能生效。
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
public @interface Expose {
    boolean serialize() default true;

    boolean deserialize() default true;
}
```

Expose 注解的注解值声明情况有四种
```java
@Expose(serialize = true, deserialize = true)   //序列化和反序列化都生效
@Expose(serialize = false, deserialize = true)  //序列化时不生效，反序列化时生效
@Expose(serialize = true, deserialize = false)  //序列化时生效，反序列化时不生效
@Expose(serialize = false, deserialize = false) //序列化和反序列化都不生效，和不写注解一样
```

#### 自定义排除策略
```java
@Test
public void toJsonPropertyFilter() {
    List<Student> stus = new ArrayList<>();
    stus.add(new Student("燕青丝", 18, new Date()));
    stus.add(new Student("沐玄音", 19, new Date()));
    stus.add(new Student("苏流沙", 20, new Date()));
    Teacher teacher = new Teacher("莫南", 18, new Date(), stus);

    //setExclusionStrategies 方法在序列化和反序列化时都会生效，如果只是想指定其中一种情况下的排除策略或分别指定排除策略，可以改为使用‍addSerializationExclusionStrategy、addDeserializationExclusionStrategy
    Gson gson = new GsonBuilder()
            .setPrettyPrinting()
            .setExclusionStrategies(new ExclusionStrategy() {
                @Override
                public boolean shouldSkipField(FieldAttributes f) {
                    //birthday不输出
                    return f.getName().equals("birthday");
                }

                @Override
                public boolean shouldSkipClass(Class<?> clazz) {
                    return false;
                }
            }).create();
    String json = gson.toJson(teacher);
    System.out.println(json);
}
```

### 导出null值
Gson在默认情况下是不动导出值null的键的。
```java
Gson gson = new GsonBuilder().serializeNulls() .create();
```

### 日期格式化
格式化输出、日期时间及其它：
```java
Gson gson = new GsonBuilder()
    //序列化null
    .serializeNulls()
    // 设置日期时间格式，另有2个重载方法
    // 在序列化和反序化时均生效
    .setDateFormat("yyyy-MM-dd")
     //禁止转义html标签
    .disableHtmlEscaping()
    //格式化输出
    .setPrettyPrinting()
    .create();
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
import com.entity.PageResult;
import com.entity.Student;
import com.entity.Teacher;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.*;
import com.google.gson.internal.LinkedTreeMap;
import com.google.gson.reflect.TypeToken;
import org.apache.commons.io.FileUtils;
import org.junit.Test;

import java.io.File;
import java.util.*;

public class GSONTest {

    @Test
    public void toJson(){
        Gson gson = new Gson();//通过构造函数来获取
        Student stu = new Student("燕青丝", 18, new Date());
        //时间的默认格式：Feb 4, 2020 10:37:04 AM
        String json = gson.toJson(stu);
        System.out.println(json);

        gson = new GsonBuilder()//通过 GsonBuilder 来获取，可以进行多项特殊配置
                .setDateFormat("yyyy-MM-dd HH:mm:ss")// 设置日期时间格式，在序列化和反序化时均生效
                .setPrettyPrinting()//格式化输出
                .create();
        json = gson.toJson(stu);
        System.out.println(json);
    }

    @Test
    public void toJsonNullFormat() {
        Gson gson = new Gson();
        Student stu = new Student(null, 18, null);
        //默认null不会输出
        String json = gson.toJson(stu);
        System.out.println(json);

        //序列化null
        gson = new GsonBuilder().serializeNulls().create();
        json = gson.toJson(stu);
        System.out.println(json);
    }

    @Test
    public void toJsonPretty() {
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        Student stu = new Student("燕青丝", 18, new Date());
        String json = gson.toJson(stu);
        System.out.println(json);
    }

    @Test
    public void toJsonList() {
        List<Student> stus = new ArrayList<>();
        stus.add(new Student("燕青丝", 18, new Date()));
        stus.add(new Student("沐璇音", 19, new Date()));
        stus.add(new Student("苏流沙", 20, new Date()));

        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        String json = gson.toJson(stus);
        System.out.println(json);
    }

    @Test
    public void toJsonPropertyFilter() {
        List<Student> stus = new ArrayList<>();
        stus.add(new Student("燕青丝", 18, new Date()));
        stus.add(new Student("沐璇音", 19, new Date()));
        stus.add(new Student("苏流沙", 20, new Date()));
        Teacher teacher = new Teacher("莫南", 18, new Date(), stus);

        //setExclusionStrategies 方法在序列化和反序列化时都会生效，如果只是想指定其中一种情况下的排除策略或分别指定排除策略，可以改为使用‍addSerializationExclusionStrategy、addDeserializationExclusionStrategy
        Gson gson = new GsonBuilder()
                .setPrettyPrinting()
                .setExclusionStrategies(new ExclusionStrategy() {
                    @Override
                    public boolean shouldSkipField(FieldAttributes f) {
                        //birthday不输出
                        return f.getName().equals("birthday");
                    }

                    @Override
                    public boolean shouldSkipClass(Class<?> clazz) {
                        return false;
                    }
                }).create();
        String json = gson.toJson(teacher);
        System.out.println(json);
    }

    @Test
    public void toJsonComplex() {
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

        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        PageResult<Teacher> pageResult = new PageResult<>(100L, teachers);
        String json = gson.toJson(pageResult);
        System.out.println(json);
    }

    @Test
    public void fromJson() {
        Gson gson = new Gson();
        String json = "{\"age\":18,\"birthday\":\"Feb 4, 2020 10:37:04 AM\",\"name\":\"燕青丝\"}";
        Student student = gson.fromJson(json, Student.class);
        System.out.println(student);

        //单引号也可以解析
        json = "{'age':18,'birthday':'Feb 4, 2020 10:37:04 AM','name':'燕青丝'}";
        student = gson.fromJson(json, Student.class);
        System.out.println(student);

        //反序列化时可以自动识别yyyy-MM-dd HH:mm:ss
        json = "{\"age\":18,\"birthday\":\"2020-02-03 18:26:43\",\"name\":\"燕青丝\"}";
        student = gson.fromJson(json, Student.class);
        System.out.println(student);
    }

    @Test
    public void fromJsonFieldNotExist() {
        Gson gson = new Gson();
        String json = "{\"age\":18,\"birthday\":\"Feb 4, 2020 10:37:04 AM\",\"name\":\"燕青丝\",\"info\":\"info\"}";
        //默认会忽略在 json 中存在但 Java 对象不存在的属性，不像Jackson那样抛出异常
        Student student = gson.fromJson(json, Student.class);
        System.out.println(student);
    }

    @Test
    public void fromJsonArray() {
        Gson gson = new Gson();
        String json = "[{\"age\":18,\"name\":\"燕青丝\"},{\"age\":19,\"name\":\"沐璇音\"},{\"age\":20,\"name\":\"苏流沙\"}]";
        List<Student> students = gson.fromJson(json, new TypeToken<List<Student>>(){}.getType());
        System.out.println(students);

        //List中元素为LinkedTreeMap
        List list = gson.fromJson(json, List.class);
        System.out.println(list);

        List<LinkedTreeMap<String,Object>> stus = gson.fromJson(json, new TypeToken<List<LinkedTreeMap<String,Object>>>(){}.getType());
        System.out.println(stus);

        List<LinkedHashMap<String,Object>> stus1 = gson.fromJson(json, new TypeToken<List<LinkedHashMap<String,Object>>>(){}.getType());
        System.out.println(stus1);
    }

    @Test
    public void readValueComplex() throws Exception {
        GsonBuilder builder = new GsonBuilder();
        //添加一个long转date的解析器
        builder.registerTypeAdapter(Date.class, (JsonDeserializer<Date>) (json, typeOfT, context) -> new Date(json.getAsJsonPrimitive().getAsLong()));
        Gson gson =builder.create();

        String path = getClass().getClassLoader().getResource("TeacherPageResult.json").getPath();
        String json = FileUtils.readFileToString(new File(path), "utf-8");
        ObjectMapper mapper = new ObjectMapper();

        PageResult<Teacher> pageResult = gson.fromJson(json, new TypeToken<PageResult<Teacher>>(){}.getType());
        List<Teacher> teachers = pageResult.getRows();
        for (Teacher teacher : teachers) {
            System.out.println(teacher);
        }
    }
}
```

