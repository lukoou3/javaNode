
# 5. 缓存处理

缓存处理的目标：主要是为了提高查询的性能，我们通常采用Redis缓存解决。

注意：

1）缓存不是来代替mysql的，只是减轻mysql访问压力

2）缓存的数据命中率越高越好，即被重复读取的数据放入缓存最佳。



## 5.1 Redis环境搭建

我们以docker的形式搭建Redis 服务

```
docker run -id --name=tensquare_redis -p 6379:6379 redis:4.0.11
```

测试：

使用redis-cli或RedisDesktopManager来测试是否才能连接成功！

## 5.2 SpringDataRedis

Spring-data-redis是spring大家族的一部分，提供了在Spring应用中通过简单的配置访问redis服务，对redis底层开发包(Jedis,  JRedis, and RJC)进行了高度封装，RedisTemplate提供了redis各种操作。



## 5.3 实现文章的缓存处理

需求分析：将文章数据对象缓存到Redis中，并设置过期时间。

注意：Redis在这里的作用是减轻MySQL数据库的访问压力。

### 5.3.1 根据ID查询文章时的缓存优化

（1）在tensquare_article 的pom.xml引入依赖

```xml
 		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```

（2）修改application.yml ,在spring节点下添加配置

```yaml
  redis:
    # Redis服务器地址
    host: 192.168.40.128
```

【参考】

更多配置项如下

```yaml
  redis:
    # Redis服务器地址（默认localhost）
    host: 192.168.40.128
    # Redis服务器连接端口（6379）
    port: 6379
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器连接密码（默认为空）
    password:
    # 连接超时时间（毫秒）
    timeout: 5000
    jedis:
      pool:
        # 连接池最大连接数（使用负值表示没有限制，默认是8）
        max-active: 8
        # 连接池中的最大空闲连接（使用负值表示没有限制，默认是8）
        max-idle: 8
        # 连接池最大阻塞等待时间（使用负值表示没有限制，默认是-1ms）
        max-wait: -1ms
        # 连接池中的最小空闲连接（默认是0）
        min-idle: 0
```



（3）修改ArticleService 引入RedisTemplate，并修改findById方法

```java
	//注入RedisTemplate
	@Autowired
	private RedisTemplate redisTemplate;

	/**
	 * 根据ID查询实体
	 * @param id
	 * @return
	 */
	public Article findArticleById(String id) {
		//先看缓存中有没有，这里我们人为定义key的规则
		Article article = (Article) redisTemplate.opsForValue().get("article_" + id);
		//判断
		if(null==article){
			//如果缓存没有，则到数据库查询并放入缓存
			article = articleRepository.findById(id).get();
			redisTemplate.opsForValue().set("article_" + id,article);
			System.out.println("从数据库中查询的数据，并放入Redis缓存");
		}else{
			System.out.println("从Redis缓存中查询的数据");
		}
		return article;
	}
```

这样在查询的时候，就会自动将文章放入缓存。

### 5.3.2 修改或删除后清除缓存-缓存同步

```java
/**
	 * 修改
	 * @param article
	 */
	public void updateArticle(Article article) {
		articleRepository.save(article);
		////数据库修改成功后，清除redis旧数据缓存
		redisTemplate.delete("article_" + article.getId() );
	}

	/**
	 * 删除
	 * @param id
	 */
	public void deleteArticleById(String id) {
		articleRepository.deleteById(id);
		//清除旧的缓存 
		redisTemplate.delete("article_" + id);
	}
```



### 5.3.3 Redis缓存过期设置处理

修改findById方法 ，设置1天的过期时间

```java
			//将数据放入缓存并设置过期时间
			redisTemplate.opsForValue().set("article_" + id,article,1, TimeUnit.DAYS);
```



为了方便测试，我们可以把过期时间改为10秒，然后观察控制台输出

```java
			redisTemplate.opsForValue().set("article_" + id,article,15, TimeUnit.SECONDS);
```

测试结果：数据会在指定的时间到达后过期：

```
Hibernate: select article0_.id as id1_0_0_, article0_.channelid as channeli2_0_0_, article0_.columnid as columnid3_0_0_, article0_.comment as comment4_0_0_, article0_.content as content5_0_0_, article0_.createtime as createti6_0_0_, article0_.image as image7_0_0_, article0_.ispublic as ispublic8_0_0_, article0_.istop as istop9_0_0_, article0_.state as state10_0_0_, article0_.thumbup as thumbup11_0_0_, article0_.title as title12_0_0_, article0_.type as type13_0_0_, article0_.updatetime as updatet14_0_0_, article0_.url as url15_0_0_, article0_.userid as userid16_0_0_, article0_.visits as visits17_0_0_ from tb_article article0_ where article0_.id=?
从数据库中查询的数据，并放入Redis缓存
从Redis缓存中查询的数据
从Redis缓存中查询的数据
从Redis缓存中查询的数据
从Redis缓存中查询的数据
Hibernate: select article0_.id as id1_0_0_, article0_.channelid as channeli2_0_0_, article0_.columnid as columnid3_0_0_, article0_.comment as comment4_0_0_, article0_.content as content5_0_0_, article0_.createtime as createti6_0_0_, article0_.image as image7_0_0_, article0_.ispublic as ispublic8_0_0_, article0_.istop as istop9_0_0_, article0_.state as state10_0_0_, article0_.thumbup as thumbup11_0_0_, article0_.title as title12_0_0_, article0_.type as type13_0_0_, article0_.updatetime as updatet14_0_0_, article0_.url as url15_0_0_, article0_.userid as userid16_0_0_, article0_.visits as visits17_0_0_ from tb_article article0_ where article0_.id=?
从数据库中查询的数据，并放入Redis缓存
```



### 5.3.4 键值的序列化方式（补充）

RedisTemplate序列化默认使用的JdkSerializeable, 存储二进制字节码，通用性可读性下降。

可以通过自定义序列化的方式。

新建config包，并在其中新建RedisConfig文件，参考内容如下

com.tensquare.article.config.RedisConfig

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

//Redis的配置类
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    /**
     * RedisTemplate配置
     * @param factory
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        //创建模板对象
        RedisTemplate<String, Object> redisTemplate= new RedisTemplate<String, Object>(factory);

        // 创建Jackson2JsonRedisSerialize，要用其替换替换默认序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置value的序列化规则和 key的序列化规则
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```

【提示】

如果只使用java语言，则默认的jdk序列化器就可以使用了。

但如果要跨平台，多种语言读取redis，则尽量使用String序列化器，值序列化为json。比较通用。



## 5.4 Spring Cache整合Redis

Spring Cache使用方法与Spring对事务管理的配置相似。Spring Cache的核心就是对某个方法进行缓存，其实质就是缓存该方法的返回结果，并把方法参数和结果用键值对的方式存放到缓存中，当再次调用该方法使用相应的参数时，就会直接从缓存里面取出指定的结果进行返回。

常用注解：

@Cacheable-------使用这个注解的方法在执行后会缓存其返回结果。

@CacheEvict--------使用这个注解的方法在其执行前或执行后移除Spring Cache中的某些元素。

【提示】

具体使用什么缓存技术的实现，根据实际情况来。比如可以是Ehcache、Redis等。

今天使用Redis作为缓存。

## 5.5 活动微服务的活动信息的缓存

### 5.5.1 活动微服务代码生成

（1）使用代码生成器生成活动微服务代码 tensquare_gathering

（2）拷贝到当前工程，并在父工程引入。

（3）修改Application类名称为GatheringApplication

（4）修改application.yml 中的端口为9005 ,url 为

```
jdbc:mysql://192.168.40.128:3306/tensquare_gathering?characterEncoding=UTF8
```

（5）进行浏览器测试

```
GET http://localhost:9005/gathering
```



### 5.5.2 根据id查询活动信息详情的缓存处理

步骤：
（1）我们在tensquare_gathering的pom.xml中引入SpringDataRedis

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```

（2）修改application.yml , 在spring节点下添加redis 配置

```yaml
  redis:
    # Redis服务器地址
    host: 192.168.40.128
```



（3）为GatheringApplication添加@EnableCaching开启缓存支持

```java
@SpringBootApplication
@EnableCaching
public class GatheringApplication {
}
```



（4）在GatheringService的findById方法添加缓存注解，这样当此方法第一次运行，在
缓存中没有找到对应的value和key，则将查询结果放入缓存。

```java
/**
	 * 根据ID查询实体
	 * @param id
	 * @return
	 */
	@Cacheable(value="gathering",key="#id")
	public Gathering findGatheringById(String id) {
		return gatheringRepository.findById(id).get();
	}
```



（5）当我们对数据进行删改的时候，需要更新缓存。其实更新缓存也就是清除缓存，因
为清除缓存后，用户再次调用查询方法无法提取缓存会重新查找数据库中的记录并放入
缓存。

在GatheringService的update、deleteById方法上添加清除缓存的注解

```java
	/**
	 * 修改
	 * @param gathering
	 */
	@CacheEvict(value="gathering",key="#gathering.id")
	public void updateGathering(Gathering gathering) {
		gatheringRepository.save(gathering);
	}

	/**
	 * 删除
	 * @param id
	 */
	@CacheEvict(value="gathering",key="#id")
	public void deleteGatheringById(String id) {
		gatheringRepository.deleteById(id);
	}
```

（6）测试



### 5.5.3 缓存过期设置处理

在spring的节点下配置

```yaml
  cache:
    redis:
      # 设置redis缓存对象存活时间（单位毫秒）
      time-to-live: 60000
```

【说明】：这里设置为60000，意味着1分钟后缓存就过期了。
























