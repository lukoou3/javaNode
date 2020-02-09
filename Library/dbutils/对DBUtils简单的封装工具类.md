## 对DBUtils简单的封装工具类

### 工具类
DataBaseUtils：
```java
import com.test.JdbcUtils;
import org.apache.commons.dbutils.BasicRowProcessor;
import org.apache.commons.dbutils.BeanProcessor;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.ResultSetHandler;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.apache.commons.dbutils.handlers.ColumnListHandler;
import org.apache.commons.dbutils.handlers.ScalarHandler;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;
import java.util.Map;

/**
 * 对Apache DBUtils进行简单的封装
 */
public class DataBaseUtils {
    private static QueryRunner queryRunner;//This class is thread safe.

    public static QueryRunner getQueryRunner() {
        if (queryRunner == null) {
            synchronized (DataBaseUtils.class) {
                if (queryRunner == null) {
                    queryRunner = new QueryRunner(JdbcUtils.getDataSource());
                }
            }
        }
        return queryRunner;
    }

    //update操作：insert、update、delete都可以
    public static int update(String sql, Object... params) throws SQLException {
        return getQueryRunner().update(sql, params);
    }

    //传入的Connection方法不会关闭
    public static int update(Connection conn, String sql, Object... params) throws SQLException {
        return getQueryRunner().update(conn, sql, params);
    }

    public static int[] updateBatch(String sql, Object[][] params) throws SQLException {
        return getQueryRunner().batch(sql, params);
    }

    public static int[] updateBatch(Connection conn, String sql, Object[][] params) throws SQLException {
        return getQueryRunner().batch(conn, sql, params);
    }

    //执行插入返回主键
    public static <T> T insertReturnId(String sql, Object... params) throws SQLException {
        return getQueryRunner().insert(sql, new ScalarHandler<T>(), params);
    }

    public static <T> T insertReturnId(Connection conn, String sql, Object... params) throws SQLException {
        return getQueryRunner().insert(conn, sql, new ScalarHandler<T>(), params);
    }

    public static <T> List<T> insertBatchReturnId(String sql, Object[][] params) throws SQLException {
        return getQueryRunner().insertBatch(sql, new ColumnListHandler<T>(), params);
    }

    public static <T> List<T> insertBatchReturnId(Connection conn, String sql, Object[][] params) throws SQLException {
        return getQueryRunner().insertBatch(conn, sql, new ColumnListHandler<T>(), params);
    }

    //返回查询查到的第一条数据转为bean
    public static <T> T findBean(String sql, Class<T> type, Object... params) throws SQLException {
        return getQueryRunner().query(sql, new BeanHandler<T>(type), params);
    }

    public static <T> T findBean(Connection conn, String sql, Class<T> type, Object... params) throws SQLException {
        return getQueryRunner().query(conn, sql, new BeanHandler<T>(type), params);
    }

    public static <T> T findBean(String sql, Class<T> type, Map<String,String> columnToProperty, Object... params) throws SQLException {
        return getQueryRunner().query(sql, new BeanHandler<T>(type, new BasicRowProcessor(new BeanProcessor(columnToProperty))), params);
    }

    public static <T> T findBean(Connection conn, String sql, Class<T> type, Map<String,String> columnToProperty, Object... params) throws SQLException {
        return getQueryRunner().query(conn, sql, new BeanHandler<T>(type, new BasicRowProcessor(new BeanProcessor(columnToProperty))), params);
    }

    //返回查询查到的所有数据
    public static <T> List<T> findAllBean(String sql, Class<T> type, Object... params) throws SQLException {
        return getQueryRunner().query(sql, new BeanListHandler<T>(type), params);
    }

    public static <T> List<T> findAllBean(Connection conn, String sql, Class<T> type, Object... params) throws SQLException {
        return getQueryRunner().query(conn, sql, new BeanListHandler<T>(type), params);
    }

    public static <T> List<T> findAllBean(String sql, Class<T> type, Map<String,String> columnToProperty, Object... params) throws SQLException {
        return getQueryRunner().query(sql, new BeanListHandler<T>(type, new BasicRowProcessor(new BeanProcessor(columnToProperty))), params);
    }

    public static <T> List<T> findAllBean(Connection conn, String sql, Class<T> type, Map<String,String> columnToProperty, Object... params) throws SQLException {
        return getQueryRunner().query(conn, sql, new BeanListHandler<T>(type, new BasicRowProcessor(new BeanProcessor(columnToProperty))), params);
    }

    public static long count(String sql, Object... params) throws SQLException {
        return getQueryRunner().query(sql, new ScalarHandler<Long>(), params);
    }

    public static long count(Connection conn, String sql, Object... params) throws SQLException {
        return getQueryRunner().query(conn, sql, new ScalarHandler<Long>(), params);
    }

    public static <T> T query(String sql, ResultSetHandler<T> rsh, Object... params) throws SQLException {
        return getQueryRunner().query(sql, rsh, params);
    }

    public static <T> T query(Connection conn, String sql, ResultSetHandler<T> rsh, Object... params) throws SQLException {
        return getQueryRunner().query(conn, sql, rsh, params);
    }

    /**
     * Close a <code>Connection</code>, avoid closing if null.
     *
     * @param conn Connection to close.
     * @throws SQLException if a database access error occurs
     */
    public static void close(Connection conn) throws SQLException {
        if (conn != null) {
            conn.close();
        }
    }

    /**
     * Close a <code>ResultSet</code>, avoid closing if null.
     *
     * @param rs ResultSet to close.
     * @throws SQLException if a database access error occurs
     */
    public static void close(ResultSet rs) throws SQLException {
        if (rs != null) {
            rs.close();
        }
    }

    /**
     * Close a <code>Statement</code>, avoid closing if null.
     *
     * @param stmt Statement to close.
     * @throws SQLException if a database access error occurs
     */
    public static void close(Statement stmt) throws SQLException {
        if (stmt != null) {
            stmt.close();
        }
    }

    /**
     * Close a <code>Connection</code>, avoid closing if null and hide
     * any SQLExceptions that occur.
     *
     * @param conn Connection to close.
     */
    public static void closeQuietly(Connection conn) {
        try {
            close(conn);
        } catch (SQLException e) { // NOPMD
            // quiet
        }
    }

    /**
     * Close a <code>Connection</code>, <code>Statement</code> and
     * <code>ResultSet</code>.  Avoid closing if null and hide any
     * SQLExceptions that occur.
     *
     * @param conn Connection to close.
     * @param stmt Statement to close.
     * @param rs ResultSet to close.
     */
    public static void closeQuietly(Connection conn, Statement stmt,
                                    ResultSet rs) {

        try {
            closeQuietly(rs);
        } finally {
            try {
                closeQuietly(stmt);
            } finally {
                closeQuietly(conn);
            }
        }

    }

    /**
     * Close a <code>ResultSet</code>, avoid closing if null and hide any
     * SQLExceptions that occur.
     *
     * @param rs ResultSet to close.
     */
    public static void closeQuietly(ResultSet rs) {
        try {
            close(rs);
        } catch (SQLException e) { // NOPMD
            // quiet
        }
    }

    /**
     * Close a <code>Statement</code>, avoid closing if null and hide
     * any SQLExceptions that occur.
     *
     * @param stmt Statement to close.
     */
    public static void closeQuietly(Statement stmt) {
        try {
            close(stmt);
        } catch (SQLException e) { // NOPMD
            // quiet
        }
    }

    // 开启事务
    public static void beginTransaction(Connection conn) throws SQLException {
        if (conn != null) {
            conn.setAutoCommit(false);
        }
    }

    /**
     * Commits a <code>Connection</code> then closes it, avoid closing if null.
     *
     * @param conn Connection to close.
     * @throws SQLException if a database access error occurs
     */
    public static void commitAndClose(Connection conn) throws SQLException {
        if (conn != null) {
            try {
                conn.commit();
            } finally {
                conn.close();
            }
        }
    }

    /**
     * Commits a <code>Connection</code> then closes it, avoid closing if null
     * and hide any SQLExceptions that occur.
     *
     * @param conn Connection to close.
     */
    public static void commitAndCloseQuietly(Connection conn) {
        try {
            commitAndClose(conn);
        } catch (SQLException e) { // NOPMD
            // quiet
        }
    }

    /**
     * Rollback any changes made on the given connection.
     * @param conn Connection to rollback.  A null value is legal.
     * @throws SQLException if a database access error occurs
     */
    public static void rollback(Connection conn) throws SQLException {
        if (conn != null) {
            conn.rollback();
        }
    }

    /**
     * Performs a rollback on the <code>Connection</code> then closes it,
     * avoid closing if null.
     *
     * @param conn Connection to rollback.  A null value is legal.
     * @throws SQLException if a database access error occurs
     * @since DbUtils 1.1
     */
    public static void rollbackAndClose(Connection conn) throws SQLException {
        if (conn != null) {
            try {
                conn.rollback();
            } finally {
                conn.close();
            }
        }
    }

    /**
     * Performs a rollback on the <code>Connection</code> then closes it,
     * avoid closing if null and hide any SQLExceptions that occur.
     *
     * @param conn Connection to rollback.  A null value is legal.
     * @since DbUtils 1.1
     */
    public static void rollbackAndCloseQuietly(Connection conn) {
        try {
            rollbackAndClose(conn);
        } catch (SQLException e) { // NOPMD
            // quiet
        }
    }
}
```

JdbcUtils：
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

### 测试
entity
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

测试类
```java
import com.dbutils.DataBaseUtils;
import com.entity.People;
import org.apache.commons.dbutils.handlers.ColumnListHandler;
import org.apache.commons.dbutils.handlers.MapListHandler;
import org.junit.Test;

import java.sql.SQLException;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Map;

public class DataBaseUtilsTest {

    @Test
    public void update() throws SQLException {
        int rows = DataBaseUtils.update("update people set name=?,age=? where id = ?", "燕青丝", 18, 10);
        System.out.println(rows);
    }

    @Test
    public void updateBatch() throws SQLException {
        Object[][] params = new Object[10][3];
        for (int i = 0; i < params.length; i++) {
            params[i][0] = "燕青丝" + i;
            params[i][1] = 17;
            params[i][2] = 11 + i;
        }
        int[] rowsArray = DataBaseUtils.updateBatch("update people set name=?,age=? where id = ?", params);
        System.out.println(Arrays.toString(rowsArray));
    }

    @Test
    public void insertReturnId() throws SQLException {
        People people = new People("沐璇音", 18, new Date());
        Long id = DataBaseUtils.<Long>insertReturnId("insert into people(name,age,birthday) values(?,?,?)", people.getName(), people.getAge(), people.getBirthday());
        System.out.println(id);
    }

    @Test
    public void insertBatchReturnId() throws SQLException {
        Object[][] params = new Object[10][3];
        for (int i = 0; i < params.length; i++) {
            params[i][0] = "沐璇音" + i;
            params[i][1] = 18;
            params[i][2] = new Date();
        }

        List<Long> ids = DataBaseUtils.<Long>insertBatchReturnId("insert into people(name,age,birthday) values(?,?,?)", params);
        System.out.println(ids);
    }

    @Test
    public void findBean() throws SQLException {
        People people = DataBaseUtils.findBean("select * from people", People.class);
        System.out.println(people);
    }


    @Test
    public void findAllBean() throws SQLException {
        List<People> peopleList = DataBaseUtils.findAllBean("select * from people", People.class);
        System.out.println(peopleList);
    }

    @Test
    public void count() throws SQLException {
        long count = DataBaseUtils.count("select count(*) from people");
        System.out.println(count);
    }

    @Test
    public void query() throws SQLException {
        List<String> names = DataBaseUtils.query("select name from people", new ColumnListHandler<String>());
        System.out.println(names);
    }

    @Test
    public void queryListMap() throws SQLException {
        /**
         * 从这里可以看出来数据源的初始化需要耗时间，数据库连接的创建也需要耗时间，从连接池获取连接时间忽略不计。
         */
        System.out.println(System.currentTimeMillis());
        DataBaseUtils.getQueryRunner();
        System.out.println(System.currentTimeMillis());

        DataBaseUtils.update("update people set name=?,age=? where id = ?", "燕青丝", 17, 10);
        System.out.println(System.currentTimeMillis());
        List<Map<String, Object>> mapList = DataBaseUtils.query("select id,name,age from people", new MapListHandler());
        System.out.println(System.currentTimeMillis());
        System.out.println(mapList);
    }
}
```




