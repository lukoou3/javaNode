# Spring注解驱动开发（三）

# 1. 属性赋值@value赋值

使用@Value赋值

- 基本数值
- 可以写SPEL表达式 #{}
- 可以${}获取配置文件信息（在运行的环境变量中的值）



使用xml时候导入配置文件是

```XML
<context:property-placeholder location="classpath:person.properties"/>
```

**使用注解可以在配置类添加一个@PropertySource注解把配置文件中k/v保存到运行的环境中**

使用${key}来获取

```java
@PropertySource(value = {"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValue {

    @Bean
    public Person person() {
        return new Person();
    }
}
```


```java
@Data
public class Person {

    @Value("vhuj")
    private String name;

    @Value("#{20-2}")
    private Integer age;

    @Value("${person.nickName}")
    private String nickName;
}
```

测试

```java
    @Test
    public void test01() {
        printBean(applicationContext);
        System.out.println("---------------------------");

        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);

        System.out.println("---------------------------");

    }
```

输出

```
---------------------------
Person(name=vhuj, age=18, nickName=三三)
---------------------------
```

# 2. 自动装配@Autowired@Qualifier@Primary

自动转配：

​	Spring利用依赖注入（DI），完成对IOC容器中各个组件的依赖关系赋值

 @Autowired自动注入:

​	a. 默认优先按照类型去容器中寻找对应的组件，如果找到去赋值

​	b. 如果找到到相同类型的组件，再将属性名（`BookDao bookdao`）作为组件的id去容器中查找

​	c. 接下来还可以使用`@Qualifier("bookdao")`明确指定需要装配的id

​	d. 默认是必须的，我们可以指定    `@Autowired(required=false)`，指定非必须

@Primary让Spring自动装配时首先装配

# 3. 自动装配@Resource和@Inject

Spring还支持使用@Resource (JSR250) 和@Inject (JSR330) 注解，这两个是java规范



@Resource和@Autowired一样实现自动装配功能，默认是按组件名称进行装配的

没有支持@Primary和@Autowird(required=false)的功能



# 4. 自动装配其他地方的自动装配

@Autowired：构造器、参数、方法属性等

标注到方法位子上@Bean+方法参数，参数从容器中获取

```java
public class Boss {
    // 属性
    @Autowired
    private Car car;
	
    // 构造器 如果构造器只有一个有参构造器可以省略
    @Autowired
    public Boss(@Autowired ar car) {
    }

    public Car getCar() {
        return car;
    }
	
    // set方法
    @Autowired		 // 参数
    public void setCar(@Autowired Car car) {
        this.car = car;
    }
}
```

# 5. 自动装配Aware注入Spring底层注解

自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory 等等），自定义组件实现xxxAware，在创建对象的时候会调用接口规定的方法注入相关的组件

```java
/**
 * Marker superinterface indicating that a bean is eligible to be
 * notified by the Spring container of a particular framework object
 * through a callback-style method. Actual method signature is
 * determined by individual subinterfaces, but should typically
 * consist of just one void-returning method that accepts a single
 * argument.
 */
public interface Aware {

}
```

我们实现几个常见的Aware接口

```java
@Component
public class Red implements BeanNameAware ,BeanFactoryAware, ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setBeanName(String name) {
        System.out.println("当前Bean的名字: " + name);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("当前的BeanFactory: " + beanFactory);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println("传入的ioc: " + applicationContext);
    }
}
```

注入到配置中测试

```java
public class IOCTestAware {

    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAware.class);

    }
}
```

测试结果

```
当前Bean的名字: red
当前的BeanFactory: org.springframework.beans.factory.support.DefaultListableBeanFactory@159c4b8: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,mainConfigOfAware,red]; root of factory hierarchy
传入的ioc: org.springframework.context.annotation.AnnotationConfigApplicationContext@1e89d68: startup date [Tue Sep 25 10:29:17 CST 2018]; root of context hierarchy

```

把Spring自定义组件注入到容器中

**原理：**

```java
public interface ApplicationContextAware extends Aware {}
```

`xxxAware`都是通过`xxxProcessor`来处理的

比如：`ApplicationContextAware`  对应`ApplicationContextAwareProcessor`

# 6. 自动装配@Profile环境搭建

Profile是Spring为我们提供可以根据当前环境，动态的激活和切换一系组件的功能


@Profile: 指定组件在哪一个环境的情况下才能被注册到容器中，不指定任何环境都能被注册这个组件

1）加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境，如果指定了default，那么这个bean默认会被注册到容器中  
2）@Profile 写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效  
3）没有标注环境标识的bean，在任何环境都是加载的  

现在，我们来给数据源加上标识：

引入数据源和mysql驱动：
```xml
<!--数据源-->
<!-- https://mvnrepository.com/artifact/c3p0/c3p0 -->
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
<!--数据库驱动-->
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.44</version>
</dependency>
```

再写一个dbconfig.properties
```properties
db.user=root
db.password=12358
db.driverClass=com.mysql.jdbc.Driver
```

这个时候，我们就可以这样来进行配置：记得加上@PropertySource("classpath:/dbconfig.properties")告诉配置文件的位置

```java
/**
 * @Profile:
 *      Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
 * 开发环境，测试环境，生产环境
 * 我们以切换数据源为例：
 * 数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）
 * @Profile: 指定组件在哪一个环境的情况下才能被注册到容器中，不指定任何环境都能被注册这个组件
 * 1）加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境，如果指定了
 * default，那么这个bean默认会被注册到容器中
 * 2）@Profile 写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效
 * 3）没有标注环境标识的bean，在任何环境都是加载的
 */
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Profile("test")
    @Bean
    public Yellow yellow() {
        return new Yellow();
    }

    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}
```



**a.** 使用命令动态参数激活：虚拟机参数位子加载 `-Dspring.profiles.active=test

**b.** 使用代码激活环境

```java
public class IOCTestProfile {

    @Test
    public void test01() {
        // 1. 使用无参构造器创建一个applicationContext
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        // 2. 设置要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("test");
        // 3. 注册主配置类
        applicationContext.register(MainConfigOfProfile.class);
        // 4. 启动刷新容器
        applicationContext.refresh();
    }
}
```


