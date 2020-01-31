## JPA的使用(了解)

### JPA入门程序
#### 开发环境
开发环境使用jdk1.8
开发工具使用IntelliJ IDEA
数据库：mysql
Maven工程pom文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.itcast</groupId>
    <artifactId>jpa-day1</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.hibernate.version>5.0.7.Final</project.hibernate.version>
    </properties>

    <dependencies>
        <!-- junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <!-- hibernate对jpa的支持包 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>${project.hibernate.version}</version>
        </dependency>

        <!-- c3p0 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-c3p0</artifactId>
            <version>${project.hibernate.version}</version>
        </dependency>

        <!-- log日志 -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <!-- Mysql and MariaDB -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
    </dependencies>

</project>
```

#### 入门程序

##### 需求
向cst_customer表插入一条数据。

##### 配置JPA的核心配置文件
在工程的classpath下创建一个名为META-INF的文件夹，由于我们是maven工程所以可以把此文件放到src/main/resources目录下。在此文件夹下创建一个名为persistence.xml的配置文件。注意文件名必须是persistence.xml不可改动

persistence.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <!--需要配置persistence-unit节点
        持久化单元：
            name：持久化单元名称
            transaction-type：事务管理的方式
                    JTA：分布式事务管理
                    RESOURCE_LOCAL：本地事务管理
    -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--jpa的实现方式 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <!--可选配置：配置jpa实现方的配置信息-->
        <properties>
            <!-- 数据库信息
                用户名，javax.persistence.jdbc.user
                密码，  javax.persistence.jdbc.password
                驱动，  javax.persistence.jdbc.driver
                数据库地址   javax.persistence.jdbc.url
            -->
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="111111"/>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql:///jpa"/>

            <!--配置jpa实现方(hibernate)的配置信息
                显示sql           ：   false|true
                自动创建数据库表    ：  hibernate.hbm2ddl.auto
                        create      : 程序运行时创建数据库表（如果有表，先删除表再创建）
                        update      ：程序运行时创建表（如果有表，不会创建表）
                        none        ：不会创建表

            -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>
```

##### 创建表对象的实体类
```java
import javax.persistence.*;

/**
 * 客户的实体类
 *      配置映射关系
 *
 *
 *   1.实体类和表的映射关系
 *      @Entity:声明实体类
 *      @Table : 配置实体类和表的映射关系
 *          name : 配置数据库表的名称
 *   2.实体类中属性和表中字段的映射关系
 *
 *
 */
@Entity
@Table(name = "cst_customer")
public class Customer {

    /**
     * @Id：声明主键的配置
     * @GeneratedValue:配置主键的生成策略
     *      strategy
     *          GenerationType.IDENTITY ：自增，mysql
     *                 * 底层数据库必须支持自动增长（底层数据库支持的自动增长方式，对id自增）
     *          GenerationType.SEQUENCE : 序列，oracle
     *                  * 底层数据库必须支持序列
     *          GenerationType.TABLE : jpa提供的一种机制，通过一张数据库表的形式帮助我们完成主键自增
     *          GenerationType.AUTO ： 由程序自动的帮助我们选择主键生成策略
     * @Column:配置属性和字段的映射关系
     *      name：数据库表中字段的名称
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId; //客户的主键

    @Column(name = "cust_name")
    private String custName;//客户名称

    @Column(name="cust_source")
    private String custSource;//客户来源

    @Column(name="cust_level")
    private String custLevel;//客户级别

    @Column(name="cust_industry")
    private String custIndustry;//客户所属行业

    @Column(name="cust_phone")
    private String custPhone;//客户的联系方式

    @Column(name="cust_address")
    private String custAddress;//客户地址

    public Long getCustId() {
        return custId;
    }

    public void setCustId(Long custId) {
        this.custId = custId;
    }

    public String getCustName() {
        return custName;
    }

    public void setCustName(String custName) {
        this.custName = custName;
    }

    public String getCustSource() {
        return custSource;
    }

    public void setCustSource(String custSource) {
        this.custSource = custSource;
    }

    public String getCustLevel() {
        return custLevel;
    }

    public void setCustLevel(String custLevel) {
        this.custLevel = custLevel;
    }

    public String getCustIndustry() {
        return custIndustry;
    }

    public void setCustIndustry(String custIndustry) {
        this.custIndustry = custIndustry;
    }

    public String getCustPhone() {
        return custPhone;
    }

    public void setCustPhone(String custPhone) {
        this.custPhone = custPhone;
    }

    public String getCustAddress() {
        return custAddress;
    }

    public void setCustAddress(String custAddress) {
        this.custAddress = custAddress;
    }
}
```

##### 测试代码
```java
public class CustomerTest {

    @Test
    public void testAddCustomer() throws Exception {
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        EntityManager entityManager = factory.createEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();
        Customer customer = new Customer();
        customer.setCustName("张三");
        customer.setCustAddress("北京");
        customer.setCustIndustry("旅游");
        customer.setCustLevel("vip");
        customer.setCustPhone("13600132258");
        customer.setCustSource("网络");
        entityManager.persist(customer);
        transaction.commit();
        entityManager.close();
        factory.close();
    }
}
```

#### 细节说明
##### 常用注解的说明
```
@Entity
    作用：指定当前类是实体类。
@Table
    作用：指定实体类和表之间的对应关系。
    属性：
        name：指定数据库表的名称
@Id
    作用：指定当前字段是主键。
@GeneratedValue
    作用：指定主键的生成方式。。
    属性：
        strategy ：指定主键生成策略。
@Column
    作用：指定实体类属性和数据库表之间的对应关系
    属性：
        name：指定数据库表的列名称。
        unique：是否唯一  
        nullable：是否可以为空  
        inserttable：是否可以插入  
        updateable：是否可以更新  
        columnDefinition: 定义建表时创建此列的DDL  
        secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字
```

##### JPA中的主键生成策略
通过annotation（注解）来映射hibernate实体的,基于annotation的hibernate主键标识为@Id, 其生成规则由@GeneratedValue设定的.这里的@id和@GeneratedValue都是JPA的标准用法。

**JPA提供的四种标准用法为TABLE,SEQUENCE,IDENTITY,AUTO**。

```
* @GeneratedValue:配置主键的生成策略
*      strategy
*          GenerationType.IDENTITY ：自增，mysql
*                 * 底层数据库必须支持自动增长（底层数据库支持的自动增长方式，对id自增）
*          GenerationType.SEQUENCE : 序列，oracle
*                  * 底层数据库必须支持序列
*          GenerationType.TABLE : jpa提供的一种机制，通过一张数据库表的形式帮助我们完成主键自增
*          GenerationType.AUTO ： 由程序自动的帮助我们选择主键生成策略
```


具体说明如下：

**IDENTITY:主键由数据库自动生成（主要是自动增长型）、mysql使用**

用法：
```java
@Id  
@GeneratedValue(strategy = GenerationType.IDENTITY) 
private Long custId;
```

**SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。oracle使用**

用法：
```java
@Id  
@GeneratedValue(strategy = GenerationType.SEQUENCE,generator="payablemoney_seq")  
@SequenceGenerator(name="payablemoney_seq", sequenceName="seq_payment")  
private Long custId;

-----------------------------------------------------------
//@SequenceGenerator源码中的定义
@Target({TYPE, METHOD, FIELD})   
@Retention(RUNTIME)  
public @interface SequenceGenerator {  
   //表示该表主键生成策略的名称，它被引用在@GeneratedValue中设置的“generator”值中
   String name();  
   //属性表示生成策略用到的数据库序列名称。
   String sequenceName() default "";  
   //表示主键初识值，默认为0
   int initialValue() default 0;  
   //表示每次主键值增加的大小，例如设置1，则表示每次插入新记录后自动加1，默认为50
   int allocationSize() default 50;  
}
```

AUTO：由程序自动的帮助我们选择主键生成策略

用法：
```java
@Id  
@GeneratedValue(strategy = GenerationType.AUTO)  
private Long custId;
```

TABLE：使用一个特定的数据库表格来保存主键

用法：
```java
@Id  
@GeneratedValue(strategy = GenerationType.TABLE, generator="pk_gen")  
@TableGenerator(name = "pk_gen",  
    table="tb_generator",  
    pkColumnName="gen_name",  
    valueColumnName="gen_value",  
    pkColumnValue="PAYABLEMOENY_PK",  
    allocationSize=1  
) 
private Long custId;

--------------------------------------------------------
//@TableGenerator的定义：
@Target({TYPE, METHOD, FIELD})   
@Retention(RUNTIME)  
public @interface TableGenerator {  
  //表示该表主键生成策略的名称，它被引用在@GeneratedValue中设置的“generator”值中
  String name();  
  //表示表生成策略所持久化的表名，例如，这里表使用的是数据库中的“tb_generator”。
  String table() default "";  
  //catalog和schema具体指定表所在的目录名或是数据库名
  String catalog() default "";  
  String schema() default "";  
  //属性的值表示在持久化表中，该主键生成策略所对应键值的名称。例如在“tb_generator”中将“gen_name”作为主键的键值
  String pkColumnName() default "";  
  //属性的值表示在持久化表中，该主键当前所生成的值，它的值将会随着每次创建累加。例如，在“tb_generator”中将“gen_value”作为主键的值 
  String valueColumnName() default "";  
  //属性的值表示在持久化表中，该生成策略所对应的主键。例如在“tb_generator”表中，将“gen_name”的值为“CUSTOMER_PK”。 
  String pkColumnValue() default "";  
  //表示主键初识值，默认为0。 
  int initialValue() default 0;  
  //表示每次主键值增加的大小，例如设置成1，则表示每次创建新记录后自动加1，默认为50。
  int allocationSize() default 50;  
  UniqueConstraint[] uniqueConstraints() default {};  
} 

//这里应用表tb_generator，定义为 ：
CREATE TABLE  tb_generator (  
  id NUMBER NOT NULL,  
  gen_name VARCHAR2(255) NOT NULL,  
  gen_value NUMBER NOT NULL,  
  PRIMARY KEY(id)  
)
```

##### 对象使用方法介绍
###### Persistence工具类
Persistence工具类主要作用是用于获取EntityManagerFactory对象的 。通过调用该类的createEntityManagerFactory静态方法，根据配置文件中持久化单元名称创建EntityManagerFactory。
```java
//1. 创建 EntitymanagerFactory
String unitName = "myJpa";
EntityManagerFactory factory= Persistence.createEntityManagerFactory(unitName);

```

###### EntityManagerFactory
EntityManagerFactory 接口主要用来创建 EntityManager 实例
```java
//创建实体管理类
EntityManager em = factory.createEntityManager();
```

**由于EntityManagerFactory 是一个线程安全的对象（即多个线程访问同一个EntityManagerFactory 对象不会有线程安全问题），并且EntityManagerFactory 的创建极其浪费资源，所以在使用JPA编程时，我们可以对EntityManagerFactory 的创建进行优化，只需要做到一个工程只存在一个EntityManagerFactory 即可**

###### EntityManager
在 JPA 规范中, EntityManager是完成持久化操作的核心对象。实体类作为普通 java对象，只有在调用 EntityManager将其持久化后才会变成持久化对象。EntityManager对象在一组实体类与底层数据源之间进行 O/R 映射的管理。它可以用来管理和更新 Entity Bean, 根椐主键查找 Entity Bean, 还可以通过JPQL语句查询实体。

我们可以通过调用EntityManager的方法完成获取事务，以及持久化数据库的操作

方法说明：
```
getTransaction : 获取事务对象
persist ： 保存操作
merge ： 更新操作
remove ： 删除操作
find/getReference ： 根据id查询
```

###### EntityTransaction
在 JPA 规范中, EntityTransaction是完成事务操作的核心对象，对于EntityTransaction在我们的java代码中承接的功能比较简单
```java
begin：开启事务
commit：提交事务
rollback：回滚事务
```

### 使用JPA实现CRUD操作
#### 提取工具类
```java
/**
 * 解决实体管理器工厂的浪费资源和耗时问题
 *      通过静态代码块的形式，当程序第一次访问此工具类时，创建一个公共的实体管理器工厂对象
 *
 * 第一次访问getEntityManager方法：经过静态代码块创建一个factory对象，再调用方法创建一个EntityManager对象
 * 第二次方法getEntityManager方法：直接通过一个已经创建好的factory对象，创建EntityManager对象
 */
public class JpaUtils {

    private static EntityManagerFactory factory;

    static  {
        //1.加载配置文件，创建entityManagerFactory
        factory = Persistence.createEntityManagerFactory("myJpa");
    }

    /**
     * 获取EntityManager对象
     */
    public static EntityManager getEntityManager() {
       return factory.createEntityManager();
    }
}
```

#### 添加数据
```java
/**
 * 测试jpa的保存
 *      案例：保存一个客户到数据库中
 *  Jpa的操作步骤
 *     1.加载配置文件创建工厂（实体管理器工厂）对象
 *     2.通过实体管理器工厂获取实体管理器
 *     3.获取事务对象，开启事务
 *     4.完成增删改查操作
 *     5.提交事务（回滚事务）
 *     6.释放资源
 */
@Test
public void testSave() {
//        //1.加载配置文件创建工厂（实体管理器工厂）对象
//        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
//        //2.通过实体管理器工厂获取实体管理器
//        EntityManager em = factory.createEntityManager();
    EntityManager em = JpaUtils.getEntityManager();
    //3.获取事务对象，开启事务
    EntityTransaction tx = em.getTransaction(); //获取事务对象
    tx.begin();//开启事务
    //4.完成增删改查操作：保存一个客户到数据库中
    Customer customer = new Customer();
    customer.setCustName("传智播客");
    customer.setCustIndustry("教育");
    //保存，
    em.persist(customer); //保存操作
    //5.提交事务
    tx.commit();
    //6.释放资源
    em.close();
//       factory.close();

}
```

#### 删除数据
```java
/**
 * 删除客户的案例
 *
 */
@Test
public  void testRemove() {
    //1.通过工具类获取entityManager
    EntityManager entityManager = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.增删改查 -- 删除客户

    //i 根据id查询客户
    Customer customer = entityManager.find(Customer.class,1l);
    //ii 调用remove方法完成删除操作
    entityManager.remove(customer);

    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

#### 修改数据
```java
/**
 * 更新客户的操作
 *      merge(Object)
 */
@Test
public  void testUpdate() {
    //1.通过工具类获取entityManager
    EntityManager entityManager = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.增删改查 -- 更新操作

    //i 查询客户
    Customer customer = entityManager.find(Customer.class,1l);
    //ii 更新客户
    customer.setCustIndustry("it教育");
    entityManager.merge(customer);

    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

#### 根据id查询
在删除和修改的数据的代码中，我们已经使用了find方法来查询数据，此方法为即时加载方法。也就是说查询之后马上得到数据。

在jpa中还提供一个根据id查询数据的方法getReference，此方法和find方法可以起到相同的效果，不同的是此方法为延迟加载（懒加载）方法。

```java
/**
 * 根据id查询客户
 *  使用find方法查询：
 *      1.查询的对象就是当前客户对象本身
 *      2.在调用find方法的时候，就会发送sql语句查询数据库
 *
 *  立即加载
 *
 *
 */
@Test
public  void testFind() {
    //1.通过工具类获取entityManager
    EntityManager entityManager = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.增删改查 -- 根据id查询客户
    /**
     * find : 根据id查询数据
     *      class：查询数据的结果需要包装的实体类类型的字节码
     *      id：查询的主键的取值
     */
    Customer customer = entityManager.find(Customer.class, 1l);
   // System.out.print(customer);
    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

```java
/**
 * 根据id查询客户
 *      getReference方法
 *          1.获取的对象是一个动态代理对象
 *          2.调用getReference方法不会立即发送sql语句查询数据库
 *              * 当调用查询结果对象的时候，才会发送查询的sql语句：什么时候用，什么时候发送sql语句查询数据库
 *
 * 延迟加载（懒加载）
 *      * 得到的是一个动态代理对象
 *      * 什么时候用，什么使用才会查询
 */
@Test
public  void testReference() {
    //1.通过工具类获取entityManager
    EntityManager entityManager = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();
    //3.增删改查 -- 根据id查询客户
    /**
     * getReference : 根据id查询数据
     *      class：查询数据的结果需要包装的实体类类型的字节码
     *      id：查询的主键的取值
     */
    Customer customer = entityManager.getReference(Customer.class, 1l);
    System.out.print(customer);
    //4.提交事务
    tx.commit();
    //5.释放资源
    entityManager.close();
}
```

#### 使用JPQL查询
JPQL全称Java Persistence Query Language，基于首次在EJB2.0中引入的EJB查询语言(EJB QL),Java持久化查询语言(JPQL)是一种可移植的查询语言，旨在以面向对象表达式语言的表达式，将SQL语法和简单查询语义绑定在一起·使用这种语言编写的查询是可移植的，可以被编译成所有主流数据库服务器上的SQL。

其特征与原生SQL语句类似，并且完全面向对象，通过类名和属性访问，而不是表名和表的属性。

##### 查询全部
```java
/**
 * 查询全部
 *      jqpl：from cn.itcast.domain.Customer
 *      sql：SELECT * FROM cst_customer
 */
@Test
public void testFindAll() {
    //1.获取entityManager对象
    EntityManager em = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    //3.查询全部
    String jpql = "from Customer ";
    Query query = em.createQuery(jpql);//创建Query查询对象，query对象才是执行jqpl的对象

    //发送查询，并封装结果集
    List list = query.getResultList();

    for (Object obj : list) {
        System.out.print(obj);
    }

    //4.提交事务
    tx.commit();
    //5.释放资源
    em.close();
}
```

##### 排序查询
```java
/**
 * 排序查询： 倒序查询全部客户（根据id倒序）
 *      sql：SELECT * FROM cst_customer ORDER BY cust_id DESC
 *      jpql：from Customer order by custId desc
 *
 * 进行jpql查询
 *      1.创建query查询对象
 *      2.对参数进行赋值
 *      3.查询，并得到返回结果
 */
@Test
public void testOrders() {
    //1.获取entityManager对象
    EntityManager em = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    //3.查询全部
    String jpql = "from Customer order by custId desc";
    Query query = em.createQuery(jpql);//创建Query查询对象，query对象才是执行jqpl的对象

    //发送查询，并封装结果集
    List list = query.getResultList();

    for (Object obj : list) {
        System.out.println(obj);
    }

    //4.提交事务
    tx.commit();
    //5.释放资源
    em.close();
}
```

##### 统计总数
```java
/**
 * 使用jpql查询，统计客户的总数
 *      sql：SELECT COUNT(cust_id) FROM cst_customer
 *      jpql：select count(custId) from Customer
 */
@Test
public void testCount() {
    //1.获取entityManager对象
    EntityManager em = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    //3.查询全部
    //i.根据jpql语句创建Query查询对象
    String jpql = "select count(custId) from Customer";
    Query query = em.createQuery(jpql);
    //ii.对参数赋值
    //iii.发送查询，并封装结果

    /**
     * getResultList ： 直接将查询结果封装为list集合
     * getSingleResult : 得到唯一的结果集
     */
    Object result = query.getSingleResult();

    System.out.println(result);

    //4.提交事务
    tx.commit();
    //5.释放资源
    em.close();
}
```

##### 分页查询
```java
/**
 * 分页查询
 *      sql：select * from cst_customer limit 0,2
 *      jqpl : from Customer
 */
@Test
public void testPaged() {
    //1.获取entityManager对象
    EntityManager em = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    //3.查询全部
    //i.根据jpql语句创建Query查询对象
    String jpql = "from Customer";
    Query query = em.createQuery(jpql);
    //ii.对参数赋值 -- 分页参数
    //起始索引
    query.setFirstResult(0);
    //每页查询的条数
    query.setMaxResults(2);

    //iii.发送查询，并封装结果

    /**
     * getResultList ： 直接将查询结果封装为list集合
     * getSingleResult : 得到唯一的结果集
     */
    List list = query.getResultList();

    for(Object obj : list) {
        System.out.println(obj);
    }

    //4.提交事务
    tx.commit();
    //5.释放资源
    em.close();
}
```

##### 条件查询
```java
/**
 * 条件查询
 *     案例：查询客户名称以‘传智播客’开头的客户
 *          sql：SELECT * FROM cst_customer WHERE cust_name LIKE  ?
 *          jpql : from Customer where custName like ?
 */
@Test
public void testCondition() {
    //1.获取entityManager对象
    EntityManager em = JpaUtils.getEntityManager();
    //2.开启事务
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    //3.查询全部
    //i.根据jpql语句创建Query查询对象
    String jpql = "from Customer where custName like ? ";
    Query query = em.createQuery(jpql);
    //ii.对参数赋值 -- 占位符参数
    //第一个参数：占位符的索引位置（从1开始），第二个参数：取值
    query.setParameter(1,"传智播客%");

    //iii.发送查询，并封装结果

    /**
     * getResultList ： 直接将查询结果封装为list集合
     * getSingleResult : 得到唯一的结果集
     */
    List list = query.getResultList();

    for(Object obj : list) {
        System.out.println(obj);
    }

    //4.提交事务
    tx.commit();
    //5.释放资源
    em.close();
}
```
