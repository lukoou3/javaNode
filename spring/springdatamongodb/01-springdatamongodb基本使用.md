## springdatamongodb基本使用
springdata系列的api都差不多，学过了springdata jpa再来看springdatamongodb就十分容易了。不同的是springdatamongodb不需要配置事务，因为springdatamongodb本身就支持事务。

### 环境配置
#### springboot中环境配置
依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```
application.yml:
```yml
server:
  #服务端口
  port: 9001
spring:
  data:
    mongodb:
      host: localhost
      database: poetry
      port: 27017
```
或者：
```properties
# user - 用户名、secret - 密码、mongo1.example.com - 数据库地址、27017- 端口号、teat - 库名 
spring.data.mongodb.uri=mongodb://user:secret@mongo1.example.com:27017/test
```
或者application.properties：
```properties
#mongodb连接地址，集群用“;”隔开
spring.mongo.mongoDatabaseAddress=10.110.112.165:27092;10.110.112.166:27092
#mongo数据名
spring.mongo.dbname=mongodb
#mongo用户
spring.mongo.username=mongodbopr
#mongo密码
spring.mongo.password=123456
#mongo最大连接数
spring.mongo.connectionsPerHost=50
```

### 实体类映射
我们可以通过实体类来进行关系映射，从而能够方便的使用对象进行数据操作。

Spring Data MongoDB提供了一些方便的映射注释供我们使用，下面列出官方说明地址以供参考

官方说明文档：https://docs.spring.io/spring-data/mongodb/docs/2.1.3.RELEASE/reference/html/#mapping-usage

* @Document：用于标记此实体类是mongodb集合映射类。可以使用collection参数指定集合名称。特别需要注意的是如果实体类没有为任何字段创建索引将不会自动创建集合。    
* @Id：用于标记id字段，对应的MongoDB的_id字段，没有标记此字段的实体也会自动生成id字段，但是我们无法通过实体来获取id。id建议使用ObjectId类型来创建。    
* @Indexed：用于标记为某一字段创建索引。name定义索引名称、unique是否为唯一索引，默认false。direction参数可以指定排序方向，升或降序。建索引可以大大的提高查询效率。    
* @CompoundIndexs：用于创建复合索引。def参数可以定义复合索引的字段及排序方向。建复合索引可以有效地提高多字段的查询效率。    
* @Field：用于指定某一个字段映射到数据库中的名称。    
* @Transient：被该注解标注的，将不会被录入到数据库中。只作为普通的javaBean属性。    
* @PersistenceConstructor：用于声明构造函数，作用是把从数据库取出的数据实例化为对象。    
* @DBRef：用于指定与其他集合的级联关系，但是需要注意的是并不会自动创建级联集合。  

一个简单的例子：
```java
//注释此类将被映射到数据库的一个集合（collection为集合名称）
@Document(collection = "ex_entity_test")

//创建联合索引
@CompoundIndexes({
        //联合索引 name 索引名称 、def 索引字段、parameter1升序、parameter3降序
        @CompoundIndex(name = "compound_index", def = "{'parameter1': 1, 'parameter3': -1}")
})

public class EntityTest implements Serializable {
    //标记id字段
    @Id
    private ObjectId id;
    //创建单字段索引（默认ASCENDING 升序、DESCENDING 降序）
    @Indexed(unique = true, direction = DESCENDING)
    private Long parameter1;
    //修改映射到数据库中的名称
    @Field("parameter2_")
    private String parameter2;

    private Date parameter3;

    private Integer parameter4;

    //关联其他集合（不添加此注释时List将会保存具体的实体值，而添加了此注释List保存的是关联集合的id）
    @DBRef
    private List<EntityTest1> parameter5;
    //此字段不映射到数据库
    @Transient
    private Integer parameter6;

    public EntityTest(){

    }
    //声明构造函数，用于实例化查询结果数据
    @PersistenceConstructor
    public EntityTest(Long parameter1, String parameter2, Date parameter3, Integer parameter4, List<EntityTest1> parameter5) {
        this.parameter1 = parameter1;
        this.parameter2 = parameter2;
        this.parameter3 = parameter3;
        this.parameter4 = parameter4;
        this.parameter5 = parameter5;
    }

    public ObjectId getId() {
        return id;
    }

    public void setId(ObjectId id) {
        this.id = id;
    }

    ......

    public Integer getParameter6() {
        return parameter6;
    }

    public void setParameter6(Integer parameter6) {
        this.parameter6 = parameter6;
    }
}
```
上例关联的集合:
```java
//注释此类将被映射到数据库的一个集合（collection为集合名称）
@Document(collection = "ex_entity_test1")

public class EntityTest1 implements Serializable {
    //标记id字段
    @Id
    private ObjectId id;

    //如果实体类没有为任何字段创建索引将不会自动创建集合
    @Indexed
    private Long parameter1;

    public EntityTest1(){

    }

    public EntityTest1(Long parameter1) {
        this.parameter1 = parameter1;
    }

    public ObjectId getId() {
        return id;
    }

    public void setId(ObjectId id) {
        this.id = id;
    }

    public Long getParameter1() {
        return parameter1;
    }

    public void setParameter1(Long parameter1) {
        this.parameter1 = parameter1;
    }
}
```

几个例子：
```java
//把一个java类声明为mongodb的文档，可以通过collection参数指定这个类对应的文档。
//@Document(collection="mongodb 对应 collection 名")
// 若未加 @Document ，如果省略，则默认使用类名小写映射集合
@Document(collection="poem")
@Data
public class Poem {
    //主键标识，该属性的值会自动对应mongodb的主键字段"_id"，如果该属性名就叫“id”,则该注解可以省略，否则必须写
    @Id
    private String id;
    private String title;
    private String dynasty;
    private String author;
    private List<String> contents;
    private List<String> tags;
    //该属性对应mongodb的字段的名字，如果一致，则无需该注解
    @Field("yizhu_cont")
    private String cont;
    @Field("yizhu_yi")
    private String yi;
    @Field("yizhu_zhu")
    private String zhu;
    @Field("yizhu_yizhu")
    private String yizhu;
    @Field("yizhu_shang")
    private String shang;
    private List<String> backgrounds;
    @Field("play_url")
    private String playUrl;
    @Field("play_author")
    private String playAuthor;
}

@Document(collection = "author")
@CompoundIndexes({
        @CompoundIndex(name = "name_idx", def = "{'name': 1, 'dynasty': 1}", unique = true)
})
@Data
@NoArgsConstructor
public class Author {
    @Id
    private ObjectId id;
    //@Indexed(unique = true)
    private String name;
    private String dynasty;
    private String info;

}
```

### MongoRepository
![](assets/markdown-img-paste-20200202203121121.png)
和springdata jpa类似，我们只需要继承MongoRepository接口，就可以显示大部分的需求。MongoRepository和JpaRepository用法一样。
```java
public interface MongoRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    ....
}
```

#### 方法名称命名规则查询
和springdata jpa类似MongoRepository同样支持方法名称命名规则查询，规则和springdata jpa一样，用idea编写时还会有提示，十分方便。

#### 基于@Query 注解的查询
@Query的属性：

* value 查询语句    
* count 作为统计的查询 返回int值    
* delete 作为删除语句并返回删除后的文档集合    
* fields 限定需要返回哪些字段

占位符是从0开始的。

例子：
```java
/**
 * 使用@Query实现自定义查询
 * value 查询语句
 * count 作为统计的查询 返回int值
 * delete 作为删除语句并返回删除后的文档集合
 * fields 限定需要返回哪些字段，默认查询全部
 *
 */
@Query(value = "{'title': {'$regex': ?0}}",
        fields = "{'id' : 1, 'title' : 1, 'author' : 1, 'contents' : 1}")
List<Poem> findByTitleUseSql(String title);

@Query("{ 'firstname' : ?0 }")
List<Person> findByThePersonsFirstname(String firstname);

@Query(count = true, value = "{'$and':[{'tenant':?3},{'reportStatus':?0}," +
            " {'dealTime':{'$gte':?1}}, {'dealTime':{'$lte':?2}}]}")
int countByStatusAndDealTimeBetween(ReportStatus status, Date begin, Date end, long tenant);

@Query("{'$and':[{'userId':?0},{'deleteFlag':false}," +
        "{'$or':[{'content':{'$regex':?1}},{'location':{'$regex':?1}},{'createAtStr':{'$regex':?1}}]}]}")
List<Idea> findByKeyWord(String userId, String key, Pageable pageable);
```

#### 使用MongoTemplate类来实现对某列的更新
Repository.save方法可以实现更新，更新的是全部。可以使用MongoTemplate类更新部分字段。其实效率相差不了多少。

看下面一个例子：

我们看一下以下点赞的临时示例代码： SpitService 新增updateThumbup方法
```java
    /**
     * 点赞-效率低
     * @param id
     */
    public void updateSpitThumbupToIncrementingOld(String id){
        Spit spit = spitRepository.findById(id).get();
        spit.setThumbup(spit.getThumbup()+1);
        spitRepository.save(spit);
    }
```

以上方法虽然实现起来比较简单，但是执行效率并不高，因为我只需要将点赞数加1就可以了，没必要查询出所有字段修改后再更新所有字段。

我们可以使用MongoTemplate类来实现对某列的操作。
（1）修改SpitService

```java
    //注入MongoTemplate
    @Autowired
    private MongoTemplate mongoTemplate;

/**
     * 使用底层的MongoTemplate工具类来直接操作
     * 根据吐槽的id，增加点赞的数量
     * @param id
     */
   public void updateSpitThumbupToIncrementing(String id){
       System.out.println("------"+id);
       //构建查询条件对象
       Query query=new Query();
       //添加条件
       query.addCriteria(Criteria.where("_id").is(id));

       //构建要修改的对象
       Update update=new Update();
        //对点赞数递增1
        //参数1：递增的字段，参数2：递增的步进，这里是每次+1
        update.inc("thumbup",1);
       //        update.set(key,value)//更新部分字段

       //更新符合条件的第一条记录，
       //参数1：查询条件对象；参数2：要修改的对象；参数3：映射的实体类或集合名字
       //db.spit.update({_id:"213123123123"},{$inc:{thumbup:NumberInt(1)}})
       mongoTemplate.updateFirst(query,update,"spit");
       //更新多条文档-参考
//       mongoTemplate.updateMulti(Query.query(Criteria.where("content").is("吐槽测试01"))
//               ,Update.update("visits",1100)
//               ,Spit.class);
   }
```

（2）SpitController新增方法

```java
/**
     *  根据吐槽id，增加点赞的数量
     * @param id
     * @return
     */
    @PutMapping("/thumbup/{id}")
    public ResultDTO incrementThumbup(@PathVariable String id){
        spitService.updateSpitThumbupToIncrementing(id);
        return new ResultDTO(true,StatusCode.OK,"点赞成功");
    }
```

#### CrudRepository、PagingAndSortingRepository接口
和springdata jpa类似CrudRepository、PagingAndSortingRepository接口本来就是**org.springframework.data.repository**包下的。


#### MongoRepository的save和insert方法
我们都知道 mongodb 有两种添加数据的方式  一种 就是  save 方法   另外一种 insert  方法
这里两个方法的区别就是  
 
* insert:当主键"_id"在集合中存在时，不做任何处理。 存在时**抛异常**    
* save:当主键"_id"在集合中存在时，进行更新。 数据整体都会更新 ，新数据会替换掉原数据 ID 以外的所有数据。如ID 不存在就新增一条数据  

save 方法 需要遍历列表，一个个插入， 而 insert 方法 是直接批量插入

MongoRepository的save和insert方法和mongodb的一样

数据比较多 想使用 insert  批量插入 提高速度  怎么办？
第一种  使用 原生 MongoTemplate 类  进行处理  想怎么样就 怎么样。
第二种   看 MongoRepository  接口 的具体实现类   SimpleMongoRepository   save 方法到底怎么写的。

spring-boot2.1.11.RELEASE:
```java
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null!");

    //判断一下主键_id的值是否存在,存在返回false,不存在为true.通过处理类设置主键Id的,就会走save,而不是insert了
    if (entityInformation.isNew(entity)) {
        return mongoOperations.insert(entity, entityInformation.getCollectionName());
    }

    return mongoOperations.save(entity, entityInformation.getCollectionName());
}

@Override
public <S extends T> List<S> saveAll(Iterable<S> entities) {

    Assert.notNull(entities, "The given Iterable of entities not be null!");

    Streamable<S> source = Streamable.of(entities);
    //判断是否全部都是新增的
    boolean allNew = source.stream().allMatch(it -> entityInformation.isNew(it));

    //全部都是新增的就insert
    if (allNew) {
        List<S> result = source.stream().collect(Collectors.toList());
        return new ArrayList<>(mongoOperations.insert(result, entityInformation.getCollectionName()));
    }
    //非全部都是新增的就遍历调用this.save
    return source.stream().map(this::save).collect(Collectors.toList());
}
```

可以看到这个版本的saveAll方法，是可以执行批量insert的。

看一个其它版本的，也是类似，还是java8使用方便啊：
```java
public class SimpleMongoRepository<T, ID extends Serializable> implements MongoRepository<T, ID> {
    private final MongoOperations mongoOperations;
    private final MongoEntityInformation<T, ID> entityInformation;

    public SimpleMongoRepository(MongoEntityInformation<T, ID> metadata, MongoOperations mongoOperations) {
        Assert.notNull(mongoOperations);
        Assert.notNull(metadata);
        this.entityInformation = metadata;
        this.mongoOperations = mongoOperations;
    }

    public <S extends T> S save(S entity) {
        Assert.notNull(entity, "Entity must not be null!");
        // 关键在这里  isNow 这个方法的实现类非常不好找  
　　　　//  判断一下主键的值是否存在,存在返回false,反正为true.通过  处理类 设置主键Id的,就会走save,而不是insert了
        if(this.entityInformation.isNew(entity)) {
            this.mongoOperations.insert(entity, this.entityInformation.getCollectionName());
        } else {
            this.mongoOperations.save(entity, this.entityInformation.getCollectionName());
        }

        return entity;
    }

    public <S extends T> List<S> save(Iterable<S> entities) {
        Assert.notNull(entities, "The given Iterable of entities not be null!");
        List<S> result = convertIterableToList(entities);
        boolean allNew = true;
        Iterator var4 = entities.iterator();

        Object entity;
        while(var4.hasNext()) {
            entity = var4.next();
            // 关键还是在这里  
            if(allNew && !this.entityInformation.isNew(entity)) {
                allNew = false;
            }
        }
        // 如果集合中 并未有 设置ID 主键的值  那么就 调用 insertAll 做批量插入
        if(allNew) {
            this.mongoOperations.insertAll(result);
        } else {
            var4 = result.iterator();
        // 否则 就 遍历集合  逐个 插入 或更新 
            while(var4.hasNext()) {
                entity = var4.next();
                this.save(entity);
            }
        }

        return result;
    }
}
```

### MongoOperations/MongoTemplate
可以使用MongoTemplate来操作数据库了。

MongoTemplate为我们提供了全面的增删改查等操作数据库的方法，详情可查看官方说明文档和API文档

官方说明文档：https://docs.spring.io/spring-data/mongodb/docs/2.1.3.RELEASE/reference/html/#mongo-template

API文档：https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html

```java
public <T> T findOne(Query query, Class<T> entityClass) 
public boolean exists(Query query, Class<?> entityClass)
public <T> List<T> find(Query query, Class<T> entityClass)
public <T> T findById(Object id, Class<T> entityClass)

public <T> T findAndModify(Query query, Update update, Class<T> entityClass)
public <T> T findAndModify(Query query, Update update, FindAndModifyOptions options, Class<T> entityClass)
public class FindAndModifyOptions {
    boolean returnNew;   // 是否返回更新后的值
    boolean upsert;      // 没有找到是否插入 
    boolean remove;      // 找到是否删除
}

public <T> T findAndRemove(Query query, Class<T> entityClass)
public long count(Query query, Class<?> entityClass) 

public void insert(Object objectToSave) 
public void insert(Collection<? extends Object> batchToSave, Class<?> entityClass)
public void insertAll(Collection<? extends Object> objectsToSave)

public void save(Object objectToSave) 保存/修改

public WriteResult upsert(Query query, Update update, Class<?> entityClass) 
public WriteResult updateFirst(Query query, Update update, Class<?> entityClass)
public WriteResult updateMulti(Query query, Update update, Class<?> entityClass)

public WriteResult remove(Object object)
public WriteResult remove(Query query, String collectionName)


public <T> List<T> findAll(Class<T> entityClass)
public <T> List<T> findAllAndRemove(Query query, Class<T> entityClass)
public DB getDb()
DBCollection getCollection(String collectionName); DBCollection 中包含各种CRUD操作以及对集合本身的定义操作（索引、命名）
public String getCollectionName(Class<?> entityClass)
public <T> MapReduceResults<T> mapReduce(String inputCollectionName, String mapFunction, String reduceFunction, Class<T> entityClass)
public <T> MapReduceResults<T> mapReduce(String inputCollectionName, String mapFunction, String reduceFunction,MapReduceOptions mapReduceOptions, Class<T> entityClass) 

public <T> GroupByResults<T> group(String inputCollectionName, GroupBy groupBy, Class<T> entityClass)


public <O> AggregationResults<O> aggregate(TypedAggregation<?> aggregation, Class<O> outputType)
public <O> AggregationResults<O> aggregate(Aggregation aggregation, Class<?> inputType, Class<O> outputType)
```

官网基本的例子：
```java
package org.spring.example;

import static org.springframework.data.mongodb.core.query.Criteria.where;
import static org.springframework.data.mongodb.core.query.Update.update;
import static org.springframework.data.mongodb.core.query.Query.query;

import java.util.List;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoDbFactory;

import com.mongodb.MongoClient;

public class MongoApp {

  private static final Log log = LogFactory.getLog(MongoApp.class);

  public static void main(String[] args) {

    MongoOperations mongoOps = new MongoTemplate(new SimpleMongoDbFactory(new MongoClient(), "database"));

    Person p = new Person("Joe", 34);

    // Insert is used to initially store the object into the database.
    mongoOps.insert(p);
    log.info("Insert: " + p);

    // Find
    p = mongoOps.findById(p.getId(), Person.class);
    log.info("Found: " + p);

    // Update
    mongoOps.updateFirst(query(where("name").is("Joe")), update("age", 35), Person.class);
    p = mongoOps.findOne(query(where("name").is("Joe")), Person.class);
    log.info("Updated: " + p);

    // Delete
    mongoOps.remove(p);

    // Check that deletion worked
    List<Person> people =  mongoOps.findAll(Person.class);
    log.info("Number of people = : " + people.size());


    mongoOps.dropCollection(Person.class);
  }
}
```

例子：
```java
@Controller
public class MongoController {

    @Autowired
    private MongoTemplate mongoTemplate;

    @GetMapping(value = "/")
    public String index(){
        return "redirect:/find";
    }

    //新增文档
    @GetMapping(value = "/insert")
    public String insert(){
        //insert方法并不提供级联类的保存，所以级联类需要先自己先保存
        EntityTest1 entityTest1_1 = new EntityTest1(1000L);
        //执行完insert后对象entityTest1将会获得保存后的id
        mongoTemplate.insert(entityTest1_1);
        //再添加一条
        EntityTest1 entityTest1_2 = new EntityTest1(1001L);
        mongoTemplate.insert(entityTest1_2);
        //创建列表并将保存后的关联对象添加进去
        ArrayList<EntityTest1> entityTest1List = new ArrayList<EntityTest1>();
        entityTest1List.add(entityTest1_1);
        entityTest1List.add(entityTest1_2);

        //新增主体对象
        EntityTest entityTest = new EntityTest(100L,"test",new Date(),10,entityTest1List);
        //新增数据的主键已经存在，则会抛DuplicateKeyException异常
        mongoTemplate.insert(entityTest);
        return "redirect:/find";
    }

    //保存文档
    //保存与新增的主要区别在于，如果主键已经存在，新增抛出异常，保存修改数据
    @GetMapping(value = "/save")
    public String save(){
        //查询最后一条数据并更新
        Sort sort = new Sort(Sort.Direction.DESC,"parameter3");
        EntityTest entityTest = mongoTemplate.findOne(Query.query(Criteria.where("")).with(sort),EntityTest.class);
        entityTest.setParameter4(3000);
        //保存数据的主键已经存在，则会对已经存在的数据修改
        mongoTemplate.save(entityTest);
        return "redirect:/find";
    }

    //删除文档
    @GetMapping(value = "/delete")
    public String delete(){
        //查询第一条数据并删除
        EntityTest entityTest = mongoTemplate.findOne(Query.query(Criteria.where("")),EntityTest.class);
        //remove方法不支持级联删除所以要单独删除子数据
        List<EntityTest1> entityTest1List = entityTest.getParameter5();
        for(EntityTest1 entityTest1:entityTest1List){
            mongoTemplate.remove(entityTest1);
        }
        //删除主数据
        mongoTemplate.remove(entityTest);

        return "redirect:/find";
    }

    //更新文档
    @GetMapping(value = "/update")
    public String update(){
        //将查询条件符合的全部文档更新
        Query query = new Query();
        Update update = Update.update("parameter2_","update");
        mongoTemplate.updateMulti(query,update,EntityTest.class);
        return "redirect:/find";
    }

    //查询文档
    @GetMapping(value = "/find")
    public String find(Model model){
        //查询小于当前时间的数据，并按时间倒序排列
        Sort sort = new Sort(Sort.Direction.DESC,"parameter3");
        List<EntityTest> findTestList = mongoTemplate.find(Query.query(Criteria.where("parameter3").lt(new Date())).with(sort) ,EntityTest.class);
        model.addAttribute("findTestList",findTestList);

        //使用findOne查询如果结果极为多条，则返回排序在最上面的一条
        EntityTest findOneTest = mongoTemplate.findOne(Query.query(Criteria.where("parameter3").lt(new Date())).with(sort) ,EntityTest.class);
        model.addAttribute("findOneTest",findOneTest);

        //模糊查询
        List<EntityTest> findTestList1 = mongoTemplate.find(Query.query(Criteria.where("parameter3").lt(new Date()).and("parameter2").regex("es")) ,EntityTest.class);
        model.addAttribute("findTestList1",findTestList1);

        //分页查询（每页3行第2页）
        Pageable pageable = new PageRequest(1,3,sort);
        List<EntityTest> findTestList2 = mongoTemplate.find(Query.query(Criteria.where("parameter3").lt(new Date())).with(pageable) ,EntityTest.class);
        //共多少条
        Long count = mongoTemplate.count(Query.query(Criteria.where("parameter3").lt(new Date())),EntityTest.class);
        //返回分页对象
        Page<EntityTest> page = new PageImpl<EntityTest>(findTestList2,pageable,count);
        model.addAttribute("page",page);

        //分页查询（通过起始行和数量也可以自己实现分页逻辑）
        List<EntityTest> findTestList3 = mongoTemplate.find(Query.query(Criteria.where("parameter3").lt(new Date())).with(sort).skip(3).limit(3) ,EntityTest.class);
        model.addAttribute("findTestList3",findTestList3);

        return "/index";
    }
}
```

#### 结合Sort、Criteria、Query、Update等
使用MongoTemplate结合Sort、Criteria、Query、Update以及分页Pageable类灵活地进行对mongodb数据库进行增删改查。

query方法：
```java
//根据查询条件查询
public <T> List<T> find(Query query, Class<T> entityClass){}
//根据查询条件查询返回一条记录
public <T> <T>findOne(Query query, Class<T> entityClass){}
//查询该collection所有记录
public <T> List<T> findAll(Class<T> entityClass){}
```

insert方法：
```java
//新增一条记录
public <T> T insert(T objectToSave){}
//在collectionName中新增一条记录
public <T> T insert(T objectToSave, String collectionName) {}
//
public <T> T save(T objectToSave){}
```

remove方法：
```java
//根据Object删除
public DeleteResult remove(Object object) {}
//根据查询条件进行删除
public DeleteResult remove(Query query, Class<?> entityClass){}
```

update方法：
```java
//
public UpdateResult upsert(Query query, Update update, Class<?> entityClass) {}
//更新查询出来的第一条记录
public UpdateResult updateFirst(Query query, Update update, String collectionName) {}
```

##### Criteria
Criteria查询条件类，类似于SQL的where，常用方法：
```java
//声明定义查询条件，且为静态方法
where(String key)
//与操作
and(String key)
//正则表达式，即可为模糊查询
regex(String re)
//包含
in(Object... o)    
//大于
gt(Object o)
//大于等于
gte(Object o)
//等于
is(Object o)
//小于
lt(Object o)
//小于等于
lte(Object o) 
//非
not()
//创建与操作
andOperator(Criteria... criteria)
```

API文档：https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/query/Criteria.html

| 方法                              | 说明                                                                                                                                                  |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| where(Stringkey)                  | 静态方法，用它可以很方便的定义查询条件例如：mongoTemplate.find(Query.query(Criteria.where("parameter3").lt(new Date()).and("parameter2").regex("es")) |
| and(Stringkey)                    | 与操作                                                                                                                                                |
| gt(Objecto)                       | 大于                                                                                                                                                  |
| gte(Objecto)                      | 大于等于                                                                                                                                              |
| in(Object... o)                   | 包含                                                                                                                                                  |
| is(Objecto)                       | 等于                                                                                                                                                  |
| lt(Objecto)                       | 小于                                                                                                                                                  |
| lte(Objecto)                      | 小于等于                                                                                                                                              |
| not()                             | 非                                                                                                                                                    |
| regex(Stringre)                   | 正则表达式                                                                                                                                            |
| andOperator(Criteria... criteria) | 创建与操作                                                                                                                                            |
| orOperator(Criteria... criteria)  | 创建或操作                                                                                                                                            |


##### Query
Query查询对象，具有查询的全部信息，其中包括筛选条件、排序、返回数量等。常用的方法：
```java
//定义查询对象，静态方法
query(CriteriaDefinition criteriaDefinition)
//在本次查询添加一个CriteriaDefinition查询条件
addCriteria(CriteriaDefinition criteriaDefinition)
//添加一个Sort排序对象
with(Sort sort)
//添加一个Pageable分页对象、通常情况下，分页和排序一起使用。
with(Pageable pageable)
```
API文档：https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/query/Query.html

常用方法介绍：
| 方法                                               | 说明                                                                                                                                                                                  |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| query(CriteriaDefinition criteriaDefinition)       | 静态方法通过注入一个CriteriaDefinition条件对象获得Query查询对象。在简单查询时使用此方法将非常的方便。例如：mongoTemplate.find(Query.query(Criteria.where("parameter3").lt(new Date()) |
| addCriteria(CriteriaDefinition criteriaDefinition) | 添加一个CriteriaDefinition查询条件类到本次查询                                                                                                                                        |
| skip(long skip)                                    | 跳过文档的数量，可以与limit配合使用实现分页效果。                                                                                                                                     |
| limit(int limit)                                   | 查询返回的文档数量。                                                                                                                                                                  |
| with(Sort sort)                                    | 添加一个Sort排序对象                                                                                                                                                                  |
| with(Pageable pageable)                            | 添加一个Pageable分页对象。Pageable可以注入一个Sort，所以分页和排序可以一起添加。例如：Pageable pageable = new PageRequest(1,3,sort);                                                  |

##### Sort
查询排序类，使用此类可以创建查询排序。
```java
//构造方法创建一个排序。direction为排序方向的枚举类型，properties为排序字段数组
Sort(Sort.Direction direction, String... properties)
//多个排序条件链接
and(Sort sort)
//返回升序排列对象
ascending()	
//返回降序排列对象
descending()	
```


API文档：https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html?is-external=true 

常用方法介绍：
| 方法                                               | 说明                                                                          |
| -------------------------------------------------- | ----------------------------------------------------------------------------- |
| Sort(Sort.Directiondirection,String... properties) | 构造方法创建一个排序。direction为排序方向的枚举类型，properties为排序字段数组 |
| and(Sortsort)                                      | 多个排序条件链接                                                              |
| ascending()                                        | 返回升序排列对象                                                              |
| descending()                                       | 返回降序排列对象                                                              |

### MongoTemplate测试
摘抄自网络：https://blog.csdn.net/qq_41970025/article/details/90603925
注： 此处的spring-data-mongodb架包是 1.8.0版本 ，更高版本的操作方法有部分修改，具体可自行查看源码
（懂一个版本；再看源码修改部分，很容易理解的）

#### 注入MongoTemplate
在对应的 Dao层 注入 MongoTemplate

#### 获取集合
```java
mongoTemplate.getCollection(mongoTemplate.getCollectionName(entityClass)).distinct(key, query.getQueryObject())

// entityClass：实体类，实际上就是实体类.class；如：User.class
// mongoTemplate.getCollectionName(entityClass)：可获取到entityClass实体类所对应的集合名称
// mongoTemplate.getCollection(mongoTemplate.getCollectionName(entityClass))：可通过集合名称获取到对应集合
// mongoTemplate.getCollection(collectionName)：返回的是基本的Driver集合对象，即DBCollection类型

// 因此使用 getCollection() 方法获取到的集合类型，不是我们在开发过程中所使用的集合类型

// key：指定键值，实际上就是MongoDB数据库集合中文档的字段名
// query：查询对象
// query.getQueryObject()：获取对应查询对象的查询条件
// .distinct(key, query.getQueryObject())：在单个集合或视图，查询满足条件的所有文档中，指定字段的不同值
```

.distinct(字段 , 查询)：在 单个 集合或视图中查找指定字段的不同值，并返回 List 集合

如：有MongoDB中集合 inventory
```
{ "_id": 1, "dept": "A", "item": { "sku": "111", "color": "red" }, "sizes": [ "S", "M" ] }
{ "_id": 2, "dept": "A", "item": { "sku": "111", "color": "blue" }, "sizes": [ "M", "L" ] }
{ "_id": 3, "dept": "B", "item": { "sku": "222", "color": "blue" }, "sizes": [ "S" ] }
{ "_id": 4, "dept": "A", "item": { "sku": "333", "color": "black" }, "sizes": [ "S" ] }
```
mongoTemplate.getCollection("inventory").distinct("dept") ：从inventory集合中的所有文档返回dept字段的不同值；结果为：[ "A", "B" ]

mongoTemplate.getCollection("inventory").distinct("item.sku") ：从inventory集合中的所有文档返回sku嵌入字段的不同值；结果为：[ "111", "222", "333" ]

mongoTemplate.getCollection("inventory").distinct("sizes") ：从inventory集合中的所有文档返回数组字段的不同值；结果为：[ "M", "S", "L" ]

mongoTemplate.getCollection("inventory").distinct("item.sku", { dept: "A" }) ：从inventory集合中 dept字段等于A 的文档中返回sku嵌入字段的不同值；结果为：[ "111", "333" ] 


#### 增加操作（插入）
可以一次性插入一个 对象集合 ，也可以单次插入 一个对象 ；需要插入的数据量大的时候，建议使用批量插入
```java
List<SysUser> userList = new ArrayList<>();

userList.add(new SysUser(11,"1","白小飞","18888888888"));
userList.add(new SysUser(9,"1","白敬亭","13333333333"));
userList.add(new SysUser(5,"1","林宥嘉","14444444444"));
userList.add(new SysUser(6,"1","彭于晏","15555555555"));
userList.add(new SysUser(7,"1","周杰伦","16666666666"));

mongoTemplate.insert(userList, "userList");	// 第一个参数是要插入的数据（文档），第二个参数是集合名称；批量插入
```

```java
SysUser sysUser = new SysUser(12,"1","黄小飞","18888888888");
mongoTemplate.insert(sysUser, "userList");	// 第一个参数是要插入的数据（文档），第二个参数是要插入的mongo集合名

//或者

SysUser sysUser = new SysUser(12,"1","黄小飞","18888888888");
mongoTemplate.save(sysUser);	// 参数表示要保存的数据（文档）；save()方法只能用于单个对象保存，不可用于批量插入
```

#### 删除操作
```java
// 删除author为yinjihuan的数据
Query query = Query.query(Criteria.where("author").is("yinjihuan"));
mongoTemplate.remove(query, Article.class);
```

```java
// 也可直接指定MongoDB集合名称
query = Query.query(Criteria.where("author").is("yinjihuan"));
mongoTemplate.remove(query, "article_info");
```

```java
// 删除集合，可传实体类，也可以传名称
mongoTemplate.dropCollection(Article.class);
mongoTemplate.dropCollection("article_info");
```

```java
// 删除数据库；在开发中，开发所使用的数据库是在配置文件中配置的；使用这个方法即可直接删除配置对应的数据库
mongoTemplate.getDb().dropDatabase();
```

下面这2种 适合 知道要删除几条数据的场景
```java
//查询出符合条件的第一个结果，并将符合条件的数据删除,只会删除第一条
query = Query.query(Criteria.where("author").is("yinjihuan"));
Article article = mongoTemplate.findAndRemove(query, Article.class);
```
```java
//查询出符合条件的所有结果，并将符合条件的所有数据删除
query = Query.query(Criteria.where("author").is("yinjihuan"));
List<Article> articles = mongoTemplate.findAllAndRemove(query, Article.class);
```

#### 修改操作
准备要练习的数据2条，如下：
```js
{
    "_id": ObjectId("57c43caed4c63d7e39b5dc48"),
    "title": "MongoTemplate的基本使用",
    "url": "http://cxytiandi.com/blog/detail/0",
    "author": "yinjihuan",
    "tags": [
        "java",
        "mongodb",
        "spring"
    ],
    "visit_count": 0,
    "add_time": ISODate("2016-08-29T13:46:21.881Z")
}
{
    "_id": ObjectId("57c43caed4c63d7e39b5dc49"),
    "title": "MongoTemplate的基本使用",
    "url": "http://cxytiandi.com/blog/detail/1",
    "author": "yinjihuan",
    "tags": [
        "java",
        "mongodb",
        "spring"
    ],
    "visit_count": 0,
    "add_time": ISODate("2016-08-29T13:46:22.201Z")
}
```

修改第一条author为yinjihuan的数据中的title和visitCount
```java
Query query = Query.query(Criteria.where("author").is("yinjihuan"));
Update update = Update.update("title", "MongoTemplate").set("visitCount", 10);
mongoTemplate.updateFirst(query, update, Article.class);
```

修改后我们会发现第一条数据的title还有visit_count被修改了

修改全部符合条件的
```java
query = Query.query(Criteria.where("author").is("yinjihuan"));
update = Update.update("title", "MongoTemplate").set("visitCount", 10);
mongoTemplate.updateMulti(query, update, Article.class);
```

修改后我们会发现所有数据的title还有visit_count被修改了

特殊更新，更新author为jason的数据，如果没有author为jason的数据则以此条件创建一条新的数据
当没有符合条件的文档，就以这个条件和更新文档为基础创建一个新的文档，如果找到匹配的文档就正常的更新。
```java
query = Query.query(Criteria.where("author").is("jason"));
update = Update.update("title", "MongoTemplate").set("visitCount", 10);
mongoTemplate.upsert(query, update, Article.class);
```
```java
// 修改后结果如下,我们会发现新增了一条数据
{
    "_id": ObjectId("57c6e1601e4735b2c306cdb7"),
    "author": "jason",
    "title": "MongoTemplate",
    "visit_count": 10
}
```

更新条件不变，更新字段改成了一个我们集合中不存在的，用set方法如果更新的key不存在则创建一个新的key
```java
query = Query.query(Criteria.where("author").is("jason"));
update = Update.update("title", "MongoTemplate").set("money", 100);
mongoTemplate.updateMulti(query, update, Article.class);
```
```java
// 修改后结果如下,我们会发现新加了一个key
{
    "_id": ObjectId("57c6e1601e4735b2c306cdb7"),
    "author": "jason",
    "title": "MongoTemplate",
    "visit_count": 10,
    "money":100
}
```

**update的inc方法用于做累加操作**，将money在之前的基础上加上100
```java
query = Query.query(Criteria.where("author").is("jason"));
update = Update.update("title", "MongoTemplate").inc("money", 100);
mongoTemplate.updateMulti(query, update, Article.class);
```

update的rename方法用于修改key的名称
```java
query = Query.query(Criteria.where("author").is("jason"));
update = Update.update("title", "MongoTemplate").rename("visitCount", "vc");
mongoTemplate.updateMulti(query, update, Article.class);
// 修改后我们会发现字段visitCount被修改成为了vc
```

update的unset方法用于删除key
```java
query = Query.query(Criteria.where("author").is("jason"));
update = Update.update("title", "MongoTemplate").unset("vc");
mongoTemplate.updateMulti(query, update, Article.class);
// 修改后我们会发现vc这个key被删除了
```

update的pull方法用于删除tags数组中的java
```java
query = Query.query(Criteria.where("author").is("yinjihuan"));
update = Update.update("title", "MongoTemplate").pull("tags", "java");
mongoTemplate.updateMulti(query, update, Article.class);
//// 修改后我们会发现tags里的java被删除了
```

#### 查询操作
MongoDB的查询方式很多种，下面只列了一些常用的，比如：

* 1、=查询    
* 2、模糊查询    
* 3、大于小于范围查询    
* 4、in查询    
* 5、or查询    
* 6、查询一条，查询全部  

根据作者查询所有符合条件的数据，返回List
```java
Query query = Query.query(Criteria.where("author").is("yinjihuan"));
List<Article> articles = mongoTemplate.find(query, Article.class);
```

只查询符合条件的第一条数据，返回Article对象
```java
query = Query.query(Criteria.where("author").is("yinjihuan"));
Article article = mongoTemplate.findOne(query, Article.class);
```

查询集合中所有数据，不加条件
```java
articles = mongoTemplate.findAll(Article.class);
```

查询符合条件的数量
```java
query = Query.query(Criteria.where("author").is("yinjihuan"));
long count = mongoTemplate.count(query, Article.class);
```

根据主键ID查询
```java
article = mongoTemplate.findById(new ObjectId("57c6e1601e4735b2c306cdb7"), Article.class);
```

in查询
```java
List<String> authors = Arrays.asList("yinjihuan", "jason");
query = Query.query(Criteria.where("author").in(authors));
articles = mongoTemplate.find(query, Article.class);
```

ne（!=）查询
```java
query = Query.query(Criteria.where("author").ne("yinjihuan"));
articles = mongoTemplate.find(query, Article.class);

```

lt(<)查询访问量小于10的文章
```java
query = Query.query(Criteria.where("visitCount").lt(10));
articles = mongoTemplate.find(query, Article.class);
```

范围查询，大于5小于10
```java
query = Query.query(Criteria.where("visitCount").gt(5).lt(10));
articles = mongoTemplate.find(query, Article.class);
```

模糊查询，author中包含a的数据
```java
query = Query.query(Criteria.where("author").regex("a"));
articles = mongoTemplate.find(query, Article.class);
```

数组查询，查询tags里数量为3的数据
```java
query = Query.query(Criteria.where("tags").size(3));
articles = mongoTemplate.find(query, Article.class);
```

or查询，查询author=jason的或者visitCount=0的数据
```java
query = Query.query(Criteria.where("").orOperator(
    Criteria.where("author").is("jason"),
    Criteria.where("visitCount").is(0)));
articles = mongoTemplate.find(query, Article.class);
```

