# SpringDataElasticsearch
SpringData项目中的一个子项目。把ElasticSearch的API进一步封装。使用更简单方便。

使用原生api可以实现的功能SpringDataElasticSearch都可以实现。

## 关键字方法
| 关键字              | 使用示例                           | 等同于的ES查询                                                                                                       |
| ------------------- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| And                 | findByNameAndPrice                 | {“bool” : {“must” : [ {“field” : {“name” : “?”}}, {“field” : {“price” : “?”}} ]}}                                    |
| Or                  | findByNameOrPrice                  | {“bool” : {“should” : [ {“field” : {“name” : “?”}}, {“field” : {“price” : “?”}} ]}}                                  |
| Is                  | findByName                         | {“bool” : {“must” : {“field” : {“name” : “?”}}}}                                                                     |
| Not                 | findByNameNot                      | {“bool” : {“must_not” : {“field” : {“name” : “?”}}}}                                                                 |
| Between             | findByPriceBetween                 | {“bool” : {“must” : {“range” : {“price” : {“from” : ?,”to” : ?,”include_lower” : true,”include_upper” : true}}}}}    |
| LessThanEqual       | findByPriceLessThan                | {“bool” : {“must” : {“range” : {“price” : {“from” : null,”to” : ?,”include_lower” : true,”include_upper” : true}}}}} |
| GreaterThanEqual    | findByPriceGreaterThan             | {“bool” : {“must” : {“range” : {“price” : {“from” : ?,”to” : null,”include_lower” : true,”include_upper” : true}}}}} |
| Before              | findByPriceBefore                  | {“bool” : {“must” : {“range” : {“price” : {“from” : null,”to” : ?,”include_lower” : true,”include_upper” : true}}}}} |
| After               | findByPriceAfter                   | {“bool” : {“must” : {“range” : {“price” : {“from” : ?,”to” : null,”include_lower” : true,”include_upper” : true}}}}} |
| Like                | findByNameLike                     | {“bool” : {“must” : {“field” : {“name” : {“query” : “? *”,”analyze_wildcard” : true}}}}}                             |
| StartingWith        | findByNameStartingWith             | {“bool” : {“must” : {“field” : {“name” : {“query” : “? *”,”analyze_wildcard” : true}}}}}                             |
| EndingWith          | findByNameEndingWith               | {“bool” : {“must” : {“field” : {“name” : {“query” : “*?”,”analyze_wildcard” : true}}}}}                              |
| Contains/Containing | findByNameContaining               | {“bool” : {“must” : {“field” : {“name” : {“query” : “?”,”analyze_wildcard” : true}}}}}                               |
| In                  | findByNameIn(Collectionnames)      | {“bool” : {“must” : {“bool” : {“should” : [ {“field” : {“name” : “?”}}, {“field” : {“name” : “?”}} ]}}}}             |
| NotIn               | findByNameNotIn(Collectionnames)   | {“bool” : {“must_not” : {“bool” : {“should” : {“field” : {“name” : “?”}}}}}}                                         |
| True                | findByAvailableTrue                | {“bool” : {“must” : {“field” : {“available” : true}}}}                                                               |
| False               | findByAvailableFalse               | {“bool” : {“must” : {“field” : {“available” : false}}}}                                                              |
| OrderBy             | findByAvailableTrueOrderByNameDesc | {“sort” : [{ “name” : {“order” : “desc”} }],”bool” : {“must” : {“field” : {“available” : true}}}}                    |

## QueryBuilders 方法介绍
* QueryBuilder boolQueryBuilder = QueryBuilders.boolQuery(); // bool语句的封装 组合语句 and not or    
* QueryBuilders.termQuery(null,null); //精确查询 完全匹配    
* QueryBuilders.termsQuery(null,1,2); // 精确查询 批量匹配    
* QueryBuilders.matchQuery(null,null); //单个匹配 field不支持通配符, 前缀具高级特性    
* QueryBuilders.matchAllQuery(); //查询所有    
* QueryBuilders.multiMatchQuery("text","",""); //匹配多个字段, field有通配符忒行    
* QueryBuilders.idsQuery(); //根据id查询    
* QueryBuilders.constantScoreQuery(boolQueryBuilder).boost(12.12f); //包裹查询, 高于设定分数, 不计算相关性    
* QueryBuilders.disMaxQuery(); // 对子查询的结果做union, score沿用子查询score的最大值,    
* QueryBuilders.fuzzyQuery("",""); //模糊查询 不能用通配符    
* QueryBuilders.moreLikeThisQuery(new String[2]); //基于内容的查询    
* QueryBuilders.boostingQuery();//它接受一个positive查询和一个negative查询。只有匹配了positive查询的文档才会被包含到结果集中，但是同时匹配了negative查询的文档会被降低其相关度，通过将文档原本的_score和negative_boost参数进行相乘来得到新的_score    
* QueryBuilders.functionScoreQuery(); //根据权重分查询    
* QueryBuilders.rangeQuery(); //范围查询    
* QueryBuilders.spanNearQuery() //跨度查询    
* QueryBuilders.wildcardQuery("user", "ki*hy") //通配符查询    
* QueryBuilders.nestedQuery() //嵌套查询

## 和Spring整合测试

### 依赖
```java
<properties>
    <es-version>6.5.3</es-version>
    <spring.version>5.1.12.RELEASE</spring.version>
</properties>


<dependencies>
    <!-- spring、My Elasticsearch是从springboot中查出来的，容易冲突 -->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-elasticsearch</artifactId>
        <version>3.1.14.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>compile</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.8</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

### 配置文件
applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xsi:schemaLocation="
      http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context.xsd
      http://www.springframework.org/schema/data/elasticsearch
      http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">
    <!--配置TransportClient对象-->
    <elasticsearch:transport-client id="client"
                                    cluster-name="My Elasticsearch"
                                    cluster-nodes="localhost:9300"/>

    <!--配置ElasticSearchTemplate对象-->
    <bean id="template" class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="client"/>
    </bean>

    <!--配置dao的包扫描器-->
    <elasticsearch:repositories base-package="com.dao"
                                elasticsearch-template-ref="template"/>
</beans>
```

### pojo
```java
package com.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName = "poetry",type = "poem")
public class Poem {
    @Id
    private String id;

    @Field(type = FieldType.Text, store = true, analyzer = "ik_max_word",searchAnalyzer = "ik_max_word")
    private String title;

    @Field(type = FieldType.Keyword, store = true)
    private String author;

    @Field(type = FieldType.Integer, store = true)
    private Integer year;

    @Field(type = FieldType.Text, store = true, analyzer = "ik_max_word",searchAnalyzer = "ik_max_word")
    private String content;

}
```

### Repository
```java
package com.dao;

import com.pojo.Poem;
import org.springframework.data.elasticsearch.annotations.Query;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

import java.util.List;

public interface PoemRepository extends ElasticsearchRepository<Poem, String>{
    List<Poem> findByContentContains(String content);

    /**
     * @Query里面写的是query里面的内容
     */
    //("{\"query\": {\"term\": {\"content\": \"?0\"}}}")
    @Query("{\"term\": {\"content\": \"?0\"}}")
    List<Poem> findByContentUseSql(String content);

    //@Query("{\"query\": {\"query_string\": { \"default_field\": \"content\",\"query\": \"?0\"}}}")
    @Query("{\"query_string\": { \"default_field\": \"content\",\"query\": \"?0\"}}")
    List<Poem> findByContentUseSql2(String content);
}
```

### Repository测试
index和mapping会自动创建
```java
package com.dao;

import com.pojo.Poem;
import org.elasticsearch.index.query.QueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;
import java.util.Optional;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class PoemRepositoryTest {
    @Autowired
    private PoemRepository poemRepository;

    @Test
    public void count(){
        long count = poemRepository.count();
        System.out.println(count);
    }

    @Test
    public void save(){
        Poem poem = new Poem("1", "过故洛阳城", "司马光", 1200,
                "烟愁雨啸黍华生，宫阙簪裳旧帝京。若问古今兴废事，请君只看洛阳城。");
        poemRepository.save(poem);
        poem = new Poem("2", "赠妓云英", "罗隐", 1300,
                "钟陵醉别十余春，重见云英掌上身。我未成名卿未嫁，可能俱是不如人。");
        poemRepository.save(poem);
        poem = new Poem("3", "芙蓉楼送辛渐", "王昌龄", 1500,
                "寒雨连江夜入吴，平明送客楚山孤。洛阳亲友如相问，一片冰心在玉壶。");
        poemRepository.save(poem);
    }

    @Test
    public void findById(){
        Optional<Poem> optional = poemRepository.findById("1");
        optional.ifPresent(System.out::print);
    }

    @Test
    public void findByContentContains(){
        //匹配的是分词，这个查询不到
        List<Poem> poems = poemRepository.findByContentContains("只看洛阳城");
        poems.forEach(System.out::println);
    }

    @Test
    public void findByContentUseSql(){
        List<Poem> poems = poemRepository.findByContentUseSql("洛阳");
        poems.forEach(System.out::println);
    }

    @Test
    public void findByContentUseSql2(){
        List<Poem> poems = poemRepository.findByContentUseSql2("只看洛阳城");
        poems.forEach(System.out::println);
    }

    @Test
    public void termQuery(){
        QueryBuilder queryBuilder = QueryBuilders.termQuery("content", "洛阳城");
        Iterable<Poem> poems = poemRepository.search(queryBuilder);
        poems.forEach(System.out::println);

        System.out.println("-------------------");

        queryBuilder = QueryBuilders.termQuery("content", "只看洛阳城");
        poems = poemRepository.search(queryBuilder);
        poems.forEach(System.out::print);

        QueryBuilders.queryStringQuery("");
    }

    @Test
    public void queryStringQuery() {
        QueryBuilder queryBuilder = QueryBuilders.queryStringQuery("只看洛阳城").defaultField("content");
        Iterable<Poem> poems = poemRepository.search(queryBuilder);
        poems.forEach(System.out::println);
    }

}
```

### 索引库操作 

#### 创建索引库
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class SprintDataElasticSearchTest {

    @Autowired
    private ElasticsearchTemplate template;

    @Test
    public void createIndex() throws Exception {
        template.createIndex("blog3");
    }

}
```

#### 设置mapping信息
创建一个Entity类，对应索引库中的type。配置Entity的属性和索引库中的字段的映射关系。

实体类
```java
@Document(indexName = "blog3", type = "article")
public class Article {
    @Id
    @Field(type = FieldType.Long, store = true)
    private long id;
    @Field(type = FieldType.text, store = true, analyzer = "ik_max_word")
    private String title;
    @Field(type = FieldType.text, store = true, analyzer = "ik_max_word")
    private String content;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Article{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", content='" + content + '\'' +
                '}';
    }
}
```
测试代码
```java
@Test
public void putMapping() throws Exception {
    template.putMapping(Article.class);
}
```


#### 创建索引库并设置mapping
```java
@Test
public void createIndexWithMapping() {
    template.createIndex(Article.class);
    template.putMapping(Article.class);
}
```

#### 删除索引库
```java
@Test
public void deleteIndex() {
    template.deleteIndex(Article.class);
    template.deleteIndex("blog2");
}
```

## 在springboot中使用
十分简单，只需引入依赖配置yml即可：

依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
application.yml
```yml
spring:
  data:
    mongodb:
      host: localhost
      database: poetry
      port: 27017
    #elasticsearch
    elasticsearch:
      cluster-name: My Elasticsearch
      cluster-nodes: hadoop01:9300,hadoop02:9300,hadoop03:9300
```
完整配置：
```yml
server:
  #服务端口
  port: 9001
spring:
  application:
    #指定服务名
    name: SpringbootPoetry
  #数据源配置
  datasource:
    #新版本的驱动,引入的是8.0.18
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/poetry?characterEncoding=utf-8&serverTimezone=GMT%2B8
    username: root
    password: 123456
  #JPA的整合配置
  jpa:
    database: mysql
    show-sql: true
    #方便测试
    generate-ddl: true
  data:
    mongodb:
      host: localhost
      database: poetry
      port: 27017
    #elasticsearch
    elasticsearch:
      cluster-name: My Elasticsearch
      cluster-nodes: hadoop01:9300,hadoop02:9300,hadoop03:9300
logging:
  level:
    root: warn
    org.springframework.data.mongodb.core: debug
    #org.mongodb.driver: debug
```















```java

```