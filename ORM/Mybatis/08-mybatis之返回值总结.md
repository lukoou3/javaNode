# mybatis之返回值总结
`https://www.cnblogs.com/ye-feng-yu/p/10987587.html`

我们思考下，返回值类型一般分为

* 数字类型，比如查询记录的个数    
* 单个对象    
* 多个对象，使用List封装    
* 单个对象，使用map封装    
* 多个对象，使用map封装   

## 1、返回值类型为数字类型
1、mapper接口，我们简单查询所有记录，返回Long类型的值。
```java
public interface PersonMapper
{
    Long getTotalNumberOfPerson();
}
```

2、mapper映射文件
```xml
<select id="getTotalNumberOfPerson" resultType="long">
    select count(*) from person
</select>
```

注意在mapper文件中，只需要 resultType 为 long 类型即可。

## 2、查询单个对象
这种查询直接返回和数据库表对应的相关实体，并且只有一条记录，一般都是按照id去查询。

1、mapper接口
```java
public interface PersonMapper
{
    Person getPerson(Integer id);
}
```

2、mapper映射文件
```xml
<select id="getPerson" resultType="com.yefengyu.mybatis.entity.Person">
     select id, first_name firstName, last_name lastName, age, email, address  from person where id = #{id}
</select>
```

总结：在查询单个对象，返回的是实体类型的时候，只需要将resultType设置为全类名即可。


## 3、查询多个对象，使用list封装
这种查询一般是根据某种条件，查询出很多个结果，然后使用List封装起来。

1、mapper接口，根据address查询多个Person对象
```java
public interface PersonMapper
{
    List<Person> getPersons(String address);
}
```

2、mapper映射文件，注意对于mapper接口中返回List类型的，mapper映射文件的resultType只需要设置List所包含的对象的类型即可。
```xml
<select id="getPersons" resultType="com.yefengyu.mybatis.entity.Person">
    select id, first_name firstName, last_name lastName, age, email, address from person where address = #{address}
</select>
```

## 4、查询单个对象，使用map封装
这种情况，类似第二小节，也是单个对象，只不过我们需要返回的是一个map，将单个对象封装成map，数据库的列名对应的属性名称作为key，返回的结果作为value.

1、mapper接口
```java
public interface PersonMapper
{
    Map<String, Object> getPersonMap(Integer id);
}
```

2、mapper映射文件,此时需要把resultType设置为map
```xml
<select id="getPersonMap" resultType="map">
    select id, first_name firstName, last_name lastName, age, email, address from person where id = #{id}
</select>
```

结果如下，注意在使用map接收返回值的时候，如果某个字段为null，那么就不会封装到map中，比如下面的结果中就没有Email字段的结果。
```js
{firstName=Schmitt, lastName=Carine, address=beijing, id=1, age=25}
```

## 5、查询多个对象，使用map封装
这个情况，一般是查询多条记录，使用主键作为key，使用对象作为value，见下面mapper接口。

1、mapper接口
```java
public interface PersonMapper
{
    @MapKey("id")
    Map<Integer, Person> getPersonsMap(String address);
}
```

2、mapper映射文件
```xml
<select id="getPersonsMap" resultType="map">
    select id, first_name firstName, last_name lastName, age, email, address from person where address = #{address}
</select>
```

3、测试
```java
public class Main
{
    public static void main(String[] args)
        throws IOException
    {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        PersonMapper mapper = sqlSession.getMapper(PersonMapper.class);
        Map<Integer, Person> personMap = mapper.getPersonsMap("beijing");
        for (Map.Entry<Integer, Person> personEntry : personMap.entrySet())
        {
            System.out.println(personEntry.getKey() + " : " + personEntry.getValue());
        }
        sqlSession.close();
    }
}
```

4、结果
```
1 : {firstName=Schmitt, lastName=Carine, address=beijing, id=1, age=25}
2 : {firstName=King, lastName=Jean, address=beijing, id=2, age=36, email=Jean@163.com}
8 : {firstName=Gao, lastName=Diego, address=beijing, id=8, age=45, email=66666@qq.com}
9 : {firstName=Piestrzeniewicz, lastName=Schmitt, address=beijing, id=9, age=36, email=44444@qq.com}
10 : {firstName=Frdrique, lastName=Juri, address=beijing, id=10, age=25, email=99999@qq.com}
```

注意：mapper接口需要使用MapKey注解指定将某个属性的值作为map的key。本例中使用person表的主键id对应的实体的属性id作为key。除此之外，mapper映射文件中resultType设置为map。


## 6、总结
* 数字类型，比如查询记录的个数。resultType为 int 或者 long等。    
* 单个对象。resultType为对象全类名    
* 多个对象，使用List封装。resultType为对象全类名。    
* 单个对象，使用map封装。resultType为map。    
* 多个对象，使用map封装。resultType为map。注意mapper接口的方法需要使用MapKey注解指定key为哪个属性。  















```xml

```

```java

```



