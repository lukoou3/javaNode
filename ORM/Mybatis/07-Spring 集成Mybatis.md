# Spring 集成Mybatis

## Spring 集成Mybatis
`https://blog.csdn.net/hellozpc/article/details/80878563#commentBox`

### 引入spring和Mybatis相关依赖
```xml
<!--数据库连接池-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.8</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.1.3.RELEASE</version>
</dependency>
<!--spring集成Junit测试-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.1.3.RELEASE</version>
    <scope>test</scope>
</dependency>
<!--spring容器-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.1.3.RELEASE</version>
</dependency>
```

### 配置spring配置文件
applicationContext-dao.xml
```xml

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
   http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
   http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <!-- 加载配置文件 -->
    <context:property-placeholder location="classpath:properties/*.properties"/>
    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url"
                  value="jdbc:mysql://${jdbc.host}:3306/${jdbc.database}?useUnicode=true&amp;characterEncoding=utf-8&amp;zeroDateTimeBehavior=convertToNull"/>
        <property name="username" value="${jdbc.userName}"/>
        <property name="password" value="${jdbc.passWord}"/>
        <!-- 初始化连接大小 -->
        <property name="initialSize" value="${jdbc.initialSize}"></property>
        <!-- 连接池最大数据库连接数  0 为没有限制 -->
        <property name="maxActive" value="${jdbc.maxActive}"></property>
        <!-- 连接池最大的空闲连接数，这里取值为20，表示即使没有数据库连接时依然可以保持20空闲的连接，而不被清除，随时处于待命状态 0 为没有限制 -->
        <property name="maxIdle" value="${jdbc.maxIdle}"></property>
        <!-- 连接池最小空闲 -->
        <property name="minIdle" value="${jdbc.minIdle}"></property>
        <!--最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制-->
        <property name="maxWait" value="${jdbc.maxWait}"></property>
    </bean>

    <!-- spring和MyBatis完美整合 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 自动扫描mapping.xml文件 -->
        <property name="mapperLocations" value="classpath:mappers/*.xml"></property>
        <!--如果mybatis-config.xml没有特殊配置也可以不需要下面的配置-->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
    </bean>

    <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.zpc.mybatis.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>

    <!-- (事务管理)transaction manager -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

db.properties
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.host=localhost
jdbc.database=ssmdemo
jdbc.userName=root
jdbc.passWord=123456
jdbc.initialSize=0
jdbc.maxActive=20
jdbc.maxIdle=20
jdbc.minIdle=1
jdbc.maxWait=1000
```

由于applicationContext-dao.xml中配置了Mapper接口扫描，所以删除mybatis-config.xml中的配置，否则报已映射错误：
Caused by: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Mapped Statements collection already contains value for MyMapper.selectUser
删除mybatis-config.xml中的映射配置：
```xml
<!--<mappers>-->
    <!--<mapper resource="mappers/MyMapper.xml"/>-->
    <!--<mapper resource="mappers/UserDaoMapper.xml"/>-->
    <!--<mapper resource="mappers/UserMapper.xml"/>-->
    <!--<mapper resource="mappers/OrderMapper.xml"/>-->
<!--</mappers>-->
```

或者在构建sqlSessionFactory时不配置mybatis-config.xml也行：
```xml
<!-- spring和MyBatis完美整合 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <!-- 自动扫描mapping.xml文件 -->
    <property name="mapperLocations" value="classpath:mappers/*.xml"></property>
    <!--如果mybatis-config.xml没有特殊配置也可以不需要下面的配置-->
    <!--<property name="configLocation" value="classpath:mybatis-config.xml" />-->
</bean>
```

### 测试
UserMapperSpringTest.java
```java
import com.zpc.mybatis.dao.UserMapper;
import com.zpc.mybatis.pojo.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

//目标：测试一下spring的bean的某些功能
@RunWith(SpringJUnit4ClassRunner.class)//junit整合spring的测试//立马开启了spring的注解
@ContextConfiguration(locations="classpath:spring/applicationContext-*.xml")//加载核心配置文件，自动构建spring容器
public class UserMapperSpringTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testQueryUserByTableName() {
        List<User> userList = this.userMapper.queryUserByTableName("tb_user");
        for (User user : userList) {
            System.out.println(user);
        }
    }

    @Test
    public void testLogin() {
        System.out.println(this.userMapper.login("hj", "123456"));
    }

    @Test
    public void testQueryUserById() {
        System.out.println(this.userMapper.queryUserById("1"));
        User user = new User();
        user.setName("美女");
        user.setId("1");
        userMapper.updateUser(user);

        System.out.println(this.userMapper.queryUserById("1"));
    }

    @Test
    public void testQueryUserAll() {
        List<User> userList = this.userMapper.queryUserAll();
        for (User user : userList) {
            System.out.println(user);
        }
    }

    @Test
    public void testInsertUser() {
        User user = new User();
        user.setAge(20);
        user.setBirthday(new Date());
        user.setName("大神");
        user.setPassword("123456");
        user.setSex(2);
        user.setUserName("bigGod222");
        this.userMapper.insertUser(user);
        System.out.println(user.getId());
    }

    @Test
    public void testUpdateUser() {
        User user = new User();
        user.setBirthday(new Date());
        user.setName("静静");
        user.setPassword("123456");
        user.setSex(0);
        user.setUserName("Jinjin");
        user.setId("1");
        this.userMapper.updateUser(user);
    }

    @Test
    public void testDeleteUserById() {
        this.userMapper.deleteUserById("1");
    }

    @Test
    public void testqueryUserList() {
        List<User> users = this.userMapper.queryUserList(null);
        for (User user : users) {
            System.out.println(user);
        }
    }

    @Test
    public void queryUserListByNameAndAge() throws Exception {
        List<User> users = this.userMapper.queryUserListByNameAndAge("鹏程", 20);
        for (User user : users) {
            System.out.println(user);
        }
    }

    @Test
    public void queryUserListByNameOrAge() throws Exception {
        List<User> users = this.userMapper.queryUserListByNameOrAge(null, 16);
        for (User user : users) {
            System.out.println(user);
        }
    }

    @Test
    public void queryUserListByIds() throws Exception {
        List<User> users = this.userMapper.queryUserListByIds(new String[]{"5", "2"});
        for (User user : users) {
            System.out.println(user);
        }
    }

}
```













```xml

```

```java

```
