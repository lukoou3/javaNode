## DBUtils

### DbUtils简介
commons-dbutils 是 Apache 组织提供的一个开源 JDBC工具类库，它是对JDBC的简单封装，学习成本极低，并且使用dbutils能极大简化jdbc编码的工作量，创建连接、结果集封装、释放资源，同时也不会影响程序的性能。创建连接、结果集封装、释放资源因此dbutils成为很多不喜欢hibernate的公司的首选。


核心类：

* org.apache.commons.dbutils.QueryRunner --- 核心    
* org.apache.commons.dbutils.ResultSetHandler --- 结果集封装器    
* org.apache.commons.dbutils.DbUtils --- 工具类   

### 核心类
#### DbUtils类
DbUtils ：提供如加载驱动、关闭连接、事务提交、回滚等常规工作的工具类，里面的所有方法都是静态的。主要方法如下：

- DbUtils类提供了三个重载的关闭方法。这些方法检查所提供的参数是不是NULL，如果不是的话，它们就关闭Connection、Statement和ResultSet。

        public static void close(…) throws java.sql.SQLException

- 这一类"quietly"方法不仅能在Connection、Statement和ResultSet为NULL情况下避免关闭，还能隐藏一些在程序中抛出的SQLException。

        public static void closeQuietly(…)

- 用来提交连接，然后关闭连接，并且在关闭连接时不抛出SQL异常。 

        public static void commitAndCloseQuietly(Connection conn)

- 装载并注册JDBC驱动程序，如果成功就返回true。使用该方法，你不需要捕捉这个异常ClassNotFoundException。
    
        public static boolean loadDriver(java.lang.String driverClassName)

#### QueryRunner类 
- 该类简单化了SQL查询，它与ResultSetHandler组合在一起使用可以完成大部分的数据库操作，能够大大减少编码量。

- QueryRunner类提供了两个构造方法：

    - 默认的构造方法：QueryRunner()

    - 需要一个 javax.sql.DataSource 来作参数的构造方法：QueryRunner(DataSource ds)
           
            // 返回数据库连接池
            public static DataSource getDataSource() {
                return dataSource;
            }

- 常用方法（分为两种情况）：
    
    - 批处理
    
            batch(Connection conn, String sql, Object[][] params)  // 传递连接批处理
            batch(String sql, Object[][] params)  // 不传递连接批处理
    
    - 查询操作
    
            public Object query(Connection conn, String sql, ResultSetHandler<T> rsh, Object... params)
            public Object query(String sql, ResultSetHandler<T> rsh, Object... params) 
    
    - 更新操作
    
            public int update(Connection conn, String sql, Object... params)
            public int update(String sql, Object... params)
            
    - 插入并返回主键     
            
            //可以返回主键
            insert(String sql, ResultSetHandler<T> rsh, Object... params)

#### ResultSetHandler接口 
- 该接口用于处理 java.sql.ResultSet，将数据按要求转换为另一种形式。

- ResultSetHandler 接口提供了一个单独的方法：

        Object handle(ResultSet rs){}

- ResultSetHandler 接口的实现类（构造方法不唯一，在这里只用最常见的构造方法）：

    - ArrayHandler()：把结果集中的第一行数据转成对象数组（存入Object[]）。

    - ArrayListHandler()：把结果集中的每一行数据都转成一个对象数组，再存放到List中。

    - **BeanHandler(Class`<T>` type)**：将结果集中的第一行数据封装到一个对应的JavaBean实例中。

    - **BeanListHandler(Class`<T>` type)**：将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里。

    - **ColumnListHandler(String columnName/int columnIndex)**：将结果集中某一列的数据存放到List中。

    - MapHandler()：将结果集中的第一行数据封装到一个Map里，key是列名，value就是对应的值。

    - MapListHandler()：将结果集中的每一行数据都封装到一个Map里，然后再将所有的Map存放到List中。

    - KeyedHandler(String columnName)：将结果集每一行数据保存到一个“小”map中,key为列名，value该列的值，再将所有“小”map对象保存到一个“大”map中 ， “大”map中的key为指定列，value为“小”map对象

    - **ScalarHandler(int columnIndex)**：通常用来保存只有一行一列的结果集。    

> 注意：DBUtils-1.4版本中的 ScalarHandler, ColumnHandler, and KeyedHandler没有泛型！要使用1.5以上的版本。
> 
> Release Notes Address : http://commons.apache.org/proper/commons-dbutils/changes-report.html

### 使用步骤
1. 将DBUtils的jar包加入到项目工程的build path中。

2. 对于CUD，有两种不同的情况：

    - 情况一：
    
        如果使用 QueryRunner(DataSource ds) 构造器创建QueryRunner对象，需要使用连接池，如DBCP、C3P0等等，数据库事务交给DBUtils框架进行管理 ---- **默认情况下每条SQL语句单独一个事务**。

        - 在这种情况下，使用如下方法：
            ```java
            batch(String sql, Object[][] params)

            query(String sql, ResultSetHandler<T> rsh, Object... params) 

            update(String sql, Object... params) 
            
            //可以返回主键
            insert(String sql, ResultSetHandler<T> rsh, Object... params)
            ```
        - demo：
            ```java
                @Test
                public void testDelete() throws SQLException {
                    QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                    String sql = "delete from users where id = ?";
                    queryRunner.update(sql, 3);
                }
            
                @Test
                public void testUpdate() throws SQLException {
                    QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                    String sql = "update users set password = ? where username = ?";
                    Object[] param = { "nihao", "小明" };
                    queryRunner.update(sql, param);
                }
            
                @Test
                public void testInsert() throws SQLException {
                    // 第一步 创建QueryRunner对象
                    QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
            
                    // 第二步 准备方法参数
                    String sql = "insert into users values(null,?,?,?)";
                    Object[] param = { "小丽", "qwe", "xiaoli@itcast.cn" };
            
                    // 第三步 调用 query / update
                    queryRunner.update(sql, param);
                }
                
                @Test
                public void testInsertReturnId() throws SQLException {
                    QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
            
                    People people = new People("莫南", 18, new Date());
            
                    //看源码知道是通过stmt.getGeneratedKeys()获取新增的主键的，mysql的int对应java的Long，这里泛型写int会报错：java.lang.Long cannot be cast to java.lang.Integer
                    Long id = queryRunner.insert("insert into people(name,age,birthday) values(?,?,?)",
                            new ScalarHandler<Long>(),
                            people.getName(), people.getAge(), people.getBirthday());
                    System.out.println(id);
                }
            ```
    - 情况二：
    
        如果使用 QueryRunner() 构造器创建QueryRunner对象 ，**需要自己管理事务**，因为框架没有连接池无法获得数据库连接。
    
        - 在这种情况下，要使用传入Connection对象参数的方法：
            ```java
                query(Connection conn, String sql, ResultSetHandler<T> rsh, Object... params)
        
                update(Connection conn, String sql, Object... params) 
            ```
        - demo:
            ```java
                // 事务控制
                @Test
                public void testTransfer() throws SQLException {
                    double money = 100;
                    String outAccount = "aaa";
                    String inAccount = "bbb";
                    String sql1 = "update account set money = money - ? where name= ?";
                    String sql2 = "update account set money = money + ? where name= ?";
            
                    // 传入DataSource的构造器，默认每条SQL语句一个单独事务，而在这里要自己管理业务，所以不合适！
                    // QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
        
                    QueryRunner queryRunner = new QueryRunner();// 不要传递连接池 --- 手动事务管理
                    Connection conn = JDBCUtils.getConnection();
                    conn.setAutoCommit(false);
                    try {
                        queryRunner.update(conn, sql1, money, outAccount); // 注意要传入Connection对象的方法
                        // int d = 1 / 0;
                        queryRunner.update(conn, sql2, money, inAccount);
            
                        System.out.println("事务提交！");
                        DbUtils.commitAndCloseQuietly(conn);
                    } catch (Exception e) {
                        System.out.println("事务回滚！");
                        DbUtils.rollbackAndCloseQuietly(conn);
                        e.printStackTrace();
                    }
                }
            ```
3. 对于R，需要用到ResultSetHandler接口，该接口有9大实现类，
    ```java
        public class ResultSetHandlerTest {
            // ScalarHandler 通常用于保存只有一行一列的结果集，例如分组函数
            @Test
            public void demo9() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select count(*) from account";
        
                long count = (Long) queryRunner.query(sql, new ScalarHandler(1)); // 得到结果集的第1列
                System.out.println(count);
            }
        
            // KeyedHandler 将结果集每一行数据保存到一个“小”map中,key为列名，value该列的值，再将所有“小”map对象保存到一个“大”map中 ， “大”map中的key为指定列，value为“小”map对象
            @Test
            public void demo8() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
        
                Map<Object, Map<String, Object>> map = queryRunner.query(sql,
                        new KeyedHandler("id"));
                System.out.println(map);
            }
        
            // MapListHandler 将结果集每一行数据保存到map中，key列名 value该列的值 ---- 再将所有map对象保存到List集合中
            @Test
            public void demo7() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
                List<Map<String, Object>> list = queryRunner.query(sql,
                        new MapListHandler());
                for (Map<String, Object> map : list) {
                    System.out.println(map);
                }
            }
        
            // MapHander 将结果集第一行数据封装到Map集合中，key是列名，value为该列的值
            @Test
            public void demo6() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
                Map<String, Object> map = queryRunner.query(sql, new MapHandler()); // 列名为String类型，该列的值为Object类型
                System.out.println(map);
            }
        
            // ColumnListHandler 获得结果集的某一列，将该列的所有值存入List<Object>中
            @Test
            public void demo5() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
        
                // 因为每列类型都不一样，所以用List<Object>存储
                // List<Object> list = queryRunner.query(sql,
                // new ColumnListHandler("name")); // 得到表列名为name的列
                List<Object> list = queryRunner.query(sql, new ColumnListHandler(2)); // 得到结果集的第2列
                System.out.println(list);
            }
        
            // BeanListHander 将结果集每一条数据，转为JavaBean对象，再保存到list集合中
            @Test
            public void demo4() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
                List<Account> accounts = queryRunner.query(sql,
                        new BeanListHandler<Account>(Account.class));
        
                for (Account account : accounts) {
                    System.out.println(account.getId());
                    System.out.println(account.getName());
                    System.out.println(account.getMoney());
                    System.out.println("----------------");
                }
            }
        
            // BeanHandler 将结果集第一行数据封装到JavaBean对象中
            @Test
            public void demo3() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
        
                // 传入 Account.class字节码文件：为了在方法中 通过反射构造Account对象
                // 使用BeanHandler注意事项 ：数据库中的表列名 与 Bean类中属性 名称一致！！！
                Account account = queryRunner.query(sql, new BeanHandler<Account>(
                        Account.class));
                System.out.println(account.getId());
                System.out.println(account.getName());
                System.out.println(account.getMoney());
            }
        
            // ArrayListHandler 将结果集每一行数据保存到List<Object[]>中
            @Test
            public void demo2() throws SQLException {
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
                List<Object[]> list = queryRunner.query(sql, new ArrayListHandler());
        
                for (Object[] objects : list) {
                    System.out.println(Arrays.toString(objects));
                }
            }
        
            // ArrayHandler 将结果集第一行数据保存到Object[]中
            @Test
            public void demo1() throws SQLException {
                // 使用DBUtils
                QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
                String sql = "select * from account";
        
                // 对象数组存储rs第一行数据的所有列
                Object[] values = queryRunner.query(sql, new ArrayHandler());
                System.out.println(Arrays.toString(values));
            }
        }
    ```

### 测试
#### entity
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class People {
    private int id;
    private String name;
    private int age;
    private Date birthday;

    public People(String name, int age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }
}
```
sql:
```java
CREATE TABLE `people` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL DEFAULT '',
  `age` tinyint(3) unsigned NOT NULL DEFAULT '0',
  `birthday` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 数据源
```java
import com.mchange.v2.c3p0.ComboPooledDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

public class JdbcUtils {
    //采用饿汉模式，默认 c3p0-copnfig.xml 文件，所以使用该工具需在 classpath 中添加 c3p0-copnfig.xml 配置文件
    private static DataSource ds = new ComboPooledDataSource();

    public static DataSource getDataSource() {
        return ds;
    }

    public static Connection getConnection() throws SQLException {
        //从数据源中获取数据库连接
        return ds.getConnection();
    }
}

```
c3p0-config.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>

  <!-- default-config 默认的配置，  -->
  <default-config>
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost/jdbc_test?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false</property>
    <property name="user">root</property>
    <property name="password">123456</property>
    
    
    <property name="initialPoolSize">10</property>
    <property name="maxIdleTime">30</property>
    <property name="maxPoolSize">100</property>
    <property name="minPoolSize">10</property>
    <property name="maxStatements">200</property>
  </default-config>

</c3p0-config>
```


#### 基本的CRUT与事务测试
```java
import com.entity.People;
import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.ColumnListHandler;
import org.apache.commons.dbutils.handlers.MapListHandler;
import org.apache.commons.dbutils.handlers.ScalarHandler;
import org.junit.Test;

import java.sql.Connection;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Map;

public class DBUtilsTest {

    /**
     * 在这里Date无法更新，一直报下面的异常，搞了半天发现是mysql-connector-java5.1.47的bug。妈的改成5.1.46就好了
     * at com.mysql.jdbc.PreparedStatement.setTimestamp(PreparedStatement.java:4241)
     * @throws SQLException
     */
    @Test
    public void testInsert() throws SQLException {
        //This class is thread safe.
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());

        People people = new People("莫南", 18, new Date());

        int rows = queryRunner.update("insert into people(name,age,birthday) values(?,?,?)",
                people.getName(), people.getAge(), people.getBirthday());
        System.out.println(rows);
    }

    @Test
    public void testInsertReturnId() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());

        People people = new People("莫南", 18, new Date());

        //看源码知道是通过stmt.getGeneratedKeys()获取新增的主键的，mysql的int对应java的Long，这里泛型写int会报错：java.lang.Long cannot be cast to java.lang.Integer
        Long id = queryRunner.insert("insert into people(name,age,birthday) values(?,?,?)",
                new ScalarHandler<Long>(),
                people.getName(), people.getAge(), people.getBirthday());
        System.out.println(id);
    }

    @Test
    public void testBatch() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());

        Object[][] params = new Object[10][3];
        for (int i = 0; i < params.length; i++) {
            params[i][0] = "莫南" + i;
            params[i][1] = 20 + i;
            params[i][2] = new Date();
        }

        //批量更新
        int[] rows = queryRunner.batch("insert into people(name,age,birthday) values(?,?,?)",
                params);
        System.out.println(Arrays.toString(rows));
    }

    @Test
    public void testInsertBatch() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());

        Object[][] params = new Object[10][3];
        for (int i = 0; i < params.length; i++) {
            params[i][0] = "莫南" + i;
            params[i][1] = 20 + i;
            params[i][2] = new Date();
        }

        //批量更新
        List<Long> ids = queryRunner.insertBatch("insert into people(name,age,birthday) values(?,?,?)",
                new ColumnListHandler<Long>(), params);
        System.out.println(ids);

        //只有一列列名为GENERATED_KEY
        List<Map<String, Object>> mapList = queryRunner.insertBatch("insert into people(name,age,birthday) values(?,?,?)",
                new MapListHandler(), params);
        System.out.println(mapList);
    }

    @Test
    public void testUpdate() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());

        People people = new People(2,"莫南", 18, new Date());

        int rows = queryRunner.update("update people set name=?,age=?,birthday=? where id = ?",
                people.getName(),people.getAge(),new Date(),people.getId());
        System.out.println(rows);
    }

    @Test
    public void testDelete() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        int rows = queryRunner.update("delete from people where id = ?", 3);
        System.out.println(rows);
    }

    @Test
    public void testQuery() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        People people = queryRunner.query("select * from people where id = ?",
                new BeanHandler<>(People.class), 2);
        System.out.println(people);
    }

    // 事务控制
    @Test
    public void testTransfer() throws SQLException {
        // 传入DataSource的构造器，默认每条SQL语句一个单独事务，而在这里要自己管理业务，所以不合适！
        // QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());

        QueryRunner queryRunner = new QueryRunner();// 不要传递连接池 --- 手动事务管理
        Connection conn = JdbcUtils.getConnection();
        conn.setAutoCommit(false);//默认是自动提交的

        try {
            queryRunner.update(conn, "update people set age = age - 5 where id = ?", 2); // 注意要传入Connection对象的方法
            //int d = 1 / 0;
            queryRunner.update(conn, "update people set age = age + 5 where id = ?", 2);

            System.out.println("事务提交！");
            DbUtils.commitAndCloseQuietly(conn);
        } catch (Exception e) {
            System.out.println("事务回滚！");
            DbUtils.rollbackAndCloseQuietly(conn);
            e.printStackTrace();
        }
    }

}
```

#### ResultSetHandler测试
```java
import com.entity.People;
import org.apache.commons.dbutils.BasicRowProcessor;
import org.apache.commons.dbutils.BeanProcessor;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.*;
import org.junit.Test;

import java.sql.SQLException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class DBUtilsResultSetHandlerTest {

    // BeanHandler 将结果集第一行数据封装到JavaBean对象中
    @Test
    public void testBeanHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";

        // 使用BeanHandler注意事项 ：数据库中的表列名 与 Bean类中属性 名称一致！！！
        People people = queryRunner.query(sql, new BeanHandler<>(
                People.class));
        System.out.println(people);

        /**
         * 通过查看源码发现是可以配置映射关系的，默认传的是BasicRowProcessor，
         * BasicRowProcessor的defaultConvert是BeanProcessor
         * 我们传入一个BasicRowProcessor(BeanProcessor convert)即可
         */
        sql = "select id p_id,name p_name,age p_age,birthday p_birthday from people";
        Map<String,String> columnToProperty = new HashMap<String, String>();
        columnToProperty.put("p_id", "id");
        columnToProperty.put("p_name", "name");
        columnToProperty.put("p_age", "age");
        columnToProperty.put("p_birthday", "birthday");
        people = queryRunner.query(sql, new BeanHandler<>(
                People.class, new BasicRowProcessor(new BeanProcessor(columnToProperty))));
        System.out.println(people);
    }

    // BeanListHander 将结果集每一条数据，转为JavaBean对象，再保存到list集合中
    @Test
    public void testBeanListHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";
        List<People> peopleList = queryRunner.query(sql,
                new BeanListHandler<>(People.class));
        System.out.println(peopleList);
    }

    // ScalarHandler 通常用于保存只有一行一列的结果集，例如分组函数
    @Test
    public void testScalarHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select count(*) count from people";

        long count = queryRunner.query(sql, new ScalarHandler<Long>(1)); // 得到结果集的第1列
        System.out.println(count);

        // 还是建议传入列名
        count = queryRunner.query(sql, new ScalarHandler<Long>("count"));
        System.out.println(count);

        // 不传参数查的是第一列
        queryRunner.query(sql, new ScalarHandler<Long>());
        System.out.println(count);
    }

    // ColumnListHandler 获得结果集的某一列，将该列的所有值存入List<Object>中
    @Test
    public void testColumnListHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";

        //我说怎么泛型的返回值是List<T> AbstractListHandler<T> implements ResultSetHandler<List<T>>
        List<String> list = queryRunner.query(sql, new ColumnListHandler<String>("name"));// 得到结果集的第2列
        System.out.println(list);
    }

    // MapHander 将结果集第一行数据封装到Map集合中，key是列名，value为该列的值
    @Test
    public void testMapHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";
        Map<String, Object> map = queryRunner.query(sql, new MapHandler()); // 列名为String类型，该列的值为Object类型
        System.out.println(map);
    }

    // MapListHandler 将结果集每一行数据保存到map中，key列名 value该列的值 ---- 再将所有map对象保存到List集合中
    @Test
    public void testMapListHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";
        List<Map<String, Object>> list = queryRunner.query(sql,
                new MapListHandler());
        for (Map<String, Object> map : list) {
            System.out.println(map);
        }
    }

    // KeyedHandler<K> 查询返回的类型为Map<String, Map<String, Object>>，K为查询列的类型
    // KeyedHandler 将结果集每一行数据保存到一个小map中,key为列名，value该列的值，再将所有小map对象保存到一个大map中 ， 大map中的key为指定列，value为小map对象
    // key相同的，后查询到的会覆盖前面查询到的
    @Test
    public void testKeyedHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";

        Map<String, Map<String, Object>> nameMap = queryRunner.query(sql,
                new KeyedHandler<String>("name"));
        System.out.println(nameMap);

        Map<Integer, Map<String, Object>> idMap = queryRunner.query(sql,
                new KeyedHandler<Integer>("id"));
        System.out.println(idMap);
    }

    // ArrayHandler 将结果集第一行数据保存到Object[]中
    @Test
    public void testArrayHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";

        // 对象数组存储rs第一行数据的所有列
        Object[] values = queryRunner.query(sql, new ArrayHandler());
        System.out.println(Arrays.toString(values));
    }

    // ArrayListHandler 将结果集每一行数据保存到List<Object[]>中
    @Test
    public void testArrayListHandler() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "select * from people";
        List<Object[]> list = queryRunner.query(sql, new ArrayListHandler());
        list.forEach(array -> System.out.println(Arrays.toString(array)));
    }

}
```





```java

```













