# spring注解,看这一篇就可以了
`https://www.jianshu.com/p/26add6e58bfa`

spring提供了一系列注解，有很多作用，此处对spring核心包注解和部分常用注解做归纳和总结。

## @Configuration
@Configuration作用在类上，声明一个class需要被spring解析以扩充beanDefinition。

```
@Configration注解同时被
@Component注解修饰，因此具有被自动加载的特点，被@Configuration修饰的类本身也会作为definition注册。
value属性是Configuration bean名称。
```

## @ComponentScan
作用在类上，声明需要对包路径下资源进行扫描，效果等同于`<context:component-scan>`
```
value(basePackages) 属性指定要扫描的包路径，多个用逗号分割。
basePackageClasses 属性指定class做扫描。
nameGenerator 属性指定beanName命名策略。
scopeResolver 属性指定从definition中解析出scope元信息的方式。
scopedProxy 指定缺省的scopedProxy，决定不同scope之间的连接方式。
resourcePattern 模糊匹配scan的文件。
useDefaultFilters 是否使用默认的探测注解（默认探测注解含@Component和被@Component修饰的注解）
includeFilters指定includeFilter，内容是Filter对象，Filter有多种类型，且能细化多种行为，
例如：
@ComponentScan(basePackages = "com.concretepage",
includeFilters = @Filter(type = FilterType.REGEX, pattern="com.concretepage..Util"),
excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = IUserService.class))
excludeFilters排除的Filter。
lazyInit延迟加载。
还有ComponentScan中定义的子注解@Filter，这个注解很有意思，@Target({})，@Target里没有任何内容，意
味着@Filter不用于修饰任何资源，而只是单纯的作为元信息载体。主要作为@ComponentScan的includeFilters
和excludeFilters的内容，用于决定scan时过滤搜索资源的策略。
```

```
@Filter有以下属性：
type(value)决定过滤器类型：
ANNOTATION：注解类型，配合AnnotationTypeFilter使用，筛选出被指定注解修饰的资源。
ASSIGNABLE_TYPE：枚举出扫描类型，配合AssignableTypeFilter使用，加载指定类型的资源（含子类)
ASPECTJ：用AspectJ正则方式搜索名称符合要求的资源。
REGEX：Java正则匹配资源。REGEX：Java正则匹配资源。
CUSTOM：自定义的FIlter.
classes属性配合type使用，当type是：
ANNOTATION：待扫描注解
ASSIGNABLE_TYPE：用户枚举的类
CUSTOM：自定义Filter
```


## @PropertySource
作用在类上，加载指定路径资源到Environment，从而实现注入。

```
name属性指定propertySource资源名称，缺省是空，会自动生成。
value属性指定资源路径，必填，可以是"classpath:/com/myco/app.properties" or "file:/path/to/file"，也可
以是${...}，作为变量时可以用当前所有加载到Environment里的属性做渲染。value必须指向唯一资源，不能模糊
匹配到多个。
ignoreResourceNotFound，声明是否必须要找到value指定的资源，缺省false表示没有该资源时会抛错。
encoding编码。
factory属性指定封装propertySource的PropertySourceFactory，PropertySourceFactory用于将一个
EncodedResource资源封装成PropertySource资源，在factory这一步可以干预PropertySource的生成。缺省用
spring默认的factory。
```


## @Conditional
作用在类和方法上，决定@Component（含子注解）和@Bean是否需要被加载。@Confitional的value属性是一组实现Condition接口的类，例如：

```
@Conditional({ ConditionTrueTest.class, ConditionTrueTest.class })
Condition接口唯一的方法是matches：
boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
入参ConditionContext是回调matcher方法时的上下文，其中包含beanFactory，environment，
resourceLoader等，几乎包含了容器各方面的资源。
入参AnnotatedTypeMetadata是被该@Conditional注解修饰资源的元信息（元信息有类和方法两种，其中
AnnotatedTypeMetadata是类元信息，MethodMetadata（AnnotatedTypeMetadata的子类）是方法元信
息）。
当@Conditional注解有多个Condition判断条件时，多个条件之间是与的关系，也即只有所有Condition均满足条
件时，才会被加载。
```


## @Bean

```
作用在方法和注解上，作用在方法上标记此处会生成一个beanDefinition。配合Profile, Scope, Lazy,
DependsOn, Primary, Order使用，描述definition行为。@Bean注解在实现上基于factory-method，因此修饰
在方法上。@Bean注解本身带有name，autowire，destroyMethod，initMethod属性，充实definition。
value和name属性均指定bean名称。
autowire指定bean自动加载模式，缺省不自动加载。
initMethod指定初始化方法，等同于xml里的init-method属性，存储在definition的initMethodName变量里，
缺省空。
destroyMethod指定销毁方法，同initMethod类似，等同于destroy-method。缺省值"(inferred)"，意味着
bean里无参的close和shutdown方法会被作为destoryMethod回调（spring称之为推断）。源码见
org.springframework.beans.factory.support.DisposableBeanAdapter.inferDestroyMethodIfNecessary(...)
多说一句，被@Bean修饰的方法会作为definition的factory method，用静态或动态工厂创建bean对象。一个
@Bean生成一个definition，但definition的很多属性不通过@Bean的属性来标记，而通过其它注解同样修饰在该
方法上来定义。
```


## @Profile
修饰类或方法，用于分类偏好（如区分环境），应用启动时决定启用哪些profile。

```
xml也可以指定profile，此时profile作为beans标签的一个属性，效果同@Profile。
value属性的值是字符串数组，用于存储多个profile，容器指定任意一个均可。
@Profile注解被@Conditional(ProfileCondition.class)注解修饰，因此在使用方面走的是@Conditional的判断
逻辑，判断实现就是org.springframework.context.annotation.ProfileCondition，该实现在match方法里获取
注解@Profile的value值，并通过Environment对象判断该profile是否被当前容器接纳，如果不被接纳，解析就终
止了。
@Conditional解析在很多地方都有，例如：
org.springframework.context.annotation.ConfigurationClassParser.processConfigurationClass()
指定应用启动偏好有多种方式：
a. @ActiveProfiles注解（spring.test）
b. JVM启动参数，-Dspring.profiles.active=xxx
c. ConfigurableEnvironment.setActiveProfiles(xx)，显式指定profile
d. web启动参数
<context-param>
<param-name>spring.profiles.active</param-name>
<param-value>xxx</param-value>
</context-param>
e. System.setProperty(AbstractEnvironment.ACTIVE_PROFILES_PROPERTY_NAME, xx);
```

## @Scope
修饰类和方法，修饰类默认对类里所有bean生效，bean scope。


## @Lazy
Lazy注解作用于很广，能和很多注解配合使用，用于标记是否延迟加载。

```
value属性默认是true，也即带上@Lazy注解后默认就是延迟加载。
```

## @DependsOn
标记是否依赖加载。

```
属性value的内容是字符串数据，因此一个bean能依赖多个其它bean做实例化，效果等同于xml里的dependson。在spring做bean实例化时，如果definition中有dependsOn，则会先实例化依赖的bean。
```

## @Primary
依赖注入有多个匹配项时，优先注入。

```
从BeanFactory中获取bean或做自动依赖注入时，可能会获得多个符合条件的对象，此时被primary修饰的对象会
优先注入，但如果有两个bean都被primary修饰，就会抛错了。
```

## @Order
定义顺序，值越小优先级越高，支持负数。

```
属性value值决定优先级，缺省是最低优先级。
```

## @Import
加载一个被@Configuration修饰的class。

```
属性value是Class数组，因此可以注入多个值。Import注解的处理逻辑会依照Import注入class类型不同而执行不
同行为：
如果被import的类是ImportSelector接口的实现类，则会回调该类实现的selectImports方法，通过返回值决定实
际要import的class（实现ImportSelector接口的类未必会被import，取决于返回值）。
如果import的类是DeferredImportSelector接口的实现类，则会延迟直到所有Configuration都被解析完成后，
再判断具体加载哪些class，这在使用某些Conditional条件判断里是需要的。
如果import的类是ImportBeanDefinitionRegistrar接口的实现类，则该类会被声明为一个registrar，最终在从
configClass中加载definition时，向BeanFactory中注入更多
definition（ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsFromRegistrars(...)）。
如果以上都不满足，才会作为被@Component修饰的类来解析。
源码见org.springframework.context.annotation.ConfigurationClassParser.processImports(...)
```

## @ImportResource
加载一个xml资源，和spring xml配置里import语义相同。

```
value和locations属性指定需要import的多个资源。
reader属性决定资源文件解析器，默认会依照文件后缀自动适配xml或groovy。
```

## @Component、@Controller、@Service、@Repository、@Configuration

```
修饰class，自动探测时加载成bean。
value属性是组件名称。
```

## @Autowired
修饰字段，方法，构造函数，入参，实现自动注入，byType。

```
required属性决定依赖注入是否必选，缺省true，意味着没有合适类型bean注入时，会抛出异常。
```

## @Qualifier
修饰字段，方法，构造函数，入参，@Primary的精细化管理版本，通常配合@Autowired使用。

```
quafile相当于是给bean取了别名，缺省以beanName作为qualify。quafily定义通过xml的方式，在bean标签里
写quafily标签。@Quafily注解里指定quafily名称，从而控制依赖注入的具体对象，通常配合@Autowire注解使
用，使得注入方式由byType变成byName。
```

## @value
修饰字段，方法，入参，实现属性注入，类型xml里的property。

```
同value注解相似的还有@Inject和@Autowired，这三个注解都表明属性需要被自动注入。而三个注入的统一探
测逻辑在AutowiredAnnotationBeanPostProcessor.postProcessMergedBeanDefinition()方法中被探测登记，
在CommonAnnotationBeanPostProcessor.postProcessPropertyValues()方法中被依赖注入。
```

## @Required
修饰方法，决定是否强制注入，注解通常修饰set方法，要求set方法必须被依赖注入。


## @Lookup
修饰方法，被修饰的方法会被动态代理重构，效果同xml中的lookup-method。

```
value属性的值会作为beanName从beanFactory中获取实际的bean，缺省value为空，意味着会以方法的返回类
型去获取factory中的bean。
```

## @Resource
javax的注解，实现自动注入，优先byName（缺省以字段名称或入参名称做搜索），其次byType。（需要
CommonAnnotationBeanPostProcessor）


## @PostConstruct
javax的注解，修饰方法，此方法会在依赖注入后回调。（需要CommonAnnotationBeanPostProcessor）


## @PreDestroy
javax的注解，修饰方法，在对象从容器移除之前回调。（需要CommonAnnotationBeanPostProcessor）


