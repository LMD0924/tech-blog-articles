# JDBC

**JAVA数据库连接**

## JDBC的搭建步骤

1.准备数据库

2.官网下载数据连接驱动jar包

3.创建Java项目，在项目下创建lib文件夹，将下载的驱动jar包复制到文件夹里

4.选中lib文件夹右键->add as library，与项目集成

5.编写代码

```java
public class JDBCQuick{
    public static void main(String[] args) throws Exception{
        //1.注册驱动
        //jdk6之后不用显示的调用Class.forName()来加载JDBC驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        
        //2.获取连接对象
        String url="jdbc:mysql://localhost:3306/数据库名";
        String username="root";
        String password="数据库密码";
        Connection connection =DriverManager.getConnection(url,username,password);
        
        //3.获取执行sql语句对象
        Statement statement = connection.createStatement();
        
        //4.编写sql语句，执行,接收返回的结果集
        String sql="select * from 表名";
        ResultSet resultSet = statement.executeQuery(sql);
        
        //5.遍历resultSet结果集
        while(resultSet.next()){
            int id=resultSet.getInt("id");
            String name=resultSet.getString("name");
            System.out.println(id+name);
        }
        
        //6.释放资源(先开后关)
        resultSet.close();
        statement.close();
        connection.close();
    }
}
```

# JDBC 核心组件详解

## 1. **Class.forName() - 驱动程序加载**

### **作用**：
动态加载并注册 JDBC 驱动程序

```java
// 传统方式（JDBC 4.0 之前必须）
Class.forName("com.mysql.cj.jdbc.Driver");

// JDBC 4.0+（自动加载，通常不需要显式调用）
// Service Provider Mechanism 会自动检测
```

### **原理**：
```java
// 当调用 Class.forName() 时：
// 1. 加载类到JVM
// 2. 执行静态代码块
// 3. MySQL Driver类的静态代码块：
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    static {
        try {
            // 自动注册到DriverManager
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

### **现代用法**：
```java
// MySQL 8.0+ 推荐写法
Class.forName("com.mysql.cj.jdbc.Driver");

// 不同数据库的驱动类：
// MySQL: com.mysql.cj.jdbc.Driver
// Oracle: oracle.jdbc.OracleDriver
// PostgreSQL: org.postgresql.Driver
// SQL Server: com.microsoft.sqlserver.jdbc.SQLServerDriver
```

## 2. **Connection - 数据库连接**

### **作用**：
表示与数据库的会话，是执行SQL语句的基础

### **获取连接**：
```java
// 基础方式
String url = "jdbc:mysql://localhost:3306/mydb";
String user = "root";
String password = "123456";
Connection conn = DriverManager.getConnection(url, user, password);

// 完整URL格式
// jdbc:mysql://主机:端口/数据库名?参数=值

// 带参数的URL（推荐）
String url = "jdbc:mysql://localhost:3306/mydb?"
    + "useSSL=false&"
    + "serverTimezone=Asia/Shanghai&"
    + "characterEncoding=utf8&"
    + "allowPublicKeyRetrieval=true";

// 使用Properties
Properties props = new Properties();
props.setProperty("user", "root");
props.setProperty("password", "123456");
props.setProperty("useUnicode", "true");
props.setProperty("characterEncoding", "UTF-8");
Connection conn = DriverManager.getConnection(url, props);
```

### **重要方法**：
```java
// 创建Statement
Statement stmt = conn.createStatement();

// 创建PreparedStatement（推荐）
PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");

// 事务控制
conn.setAutoCommit(false);  // 开启事务
conn.commit();              // 提交事务
conn.rollback();            // 回滚事务

// 连接信息
DatabaseMetaData meta = conn.getMetaData();  // 数据库元数据
String dbName = meta.getDatabaseProductName();

// 关闭连接
conn.close();  // 必须关闭，推荐用try-with-resources
```

### **连接池管理**：
```java
// 使用HikariCP连接池（推荐）
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
config.setUsername("root");
config.setPassword("123456");
config.setMaximumPoolSize(10);
config.setMinimumIdle(5);

HikariDataSource ds = new HikariDataSource(config);
Connection conn = ds.getConnection();  // 从连接池获取
```

## 3. **Statement - 静态SQL执行器**

### **特点**：
- 用于执行静态SQL语句
- 容易导致SQL注入攻击
- 每次执行都要编译SQL

### **使用方法**：
```java
Statement stmt = conn.createStatement();

// 1. 执行查询（返回ResultSet）
ResultSet rs = stmt.executeQuery("SELECT * FROM users");

// 2. 执行更新（返回受影响行数）
int rows = stmt.executeUpdate("UPDATE users SET name='张三' WHERE id=1");

// 3. 执行任意SQL（返回boolean）
boolean hasResultSet = stmt.execute("CALL my_procedure()");

// 4. 批量执行
stmt.addBatch("INSERT INTO users(name) VALUES('Tom')");
stmt.addBatch("INSERT INTO users(name) VALUES('Jerry')");
int[] results = stmt.executeBatch();

// 5. 获取生成的主键
stmt.executeUpdate("INSERT ...", Statement.RETURN_GENERATED_KEYS);
ResultSet keys = stmt.getGeneratedKeys();
if (keys.next()) {
    int id = keys.getInt(1);
}
```

### **SQL注入风险**：
```java
// ❌ 危险！SQL注入漏洞
String userInput = "admin' OR '1'='1";
String sql = "SELECT * FROM users WHERE username = '" + userInput + "'";
// 实际SQL: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// 会返回所有用户数据！

// ✅ 使用PreparedStatement避免
```

## 4. **PreparedStatement - 预编译SQL执行器**

### **特点**：
- 预编译SQL，性能更好
- 防止SQL注入
- 可重用
- 支持参数化查询

### **使用方法**：
```java
// 1. 创建PreparedStatement
String sql = "INSERT INTO users(name, age, email) VALUES(?, ?, ?)";
PreparedStatement pstmt = conn.prepareStatement(sql);

// 2. 设置参数（索引从1开始）
pstmt.setString(1, "张三");      // 第一个参数
pstmt.setInt(2, 25);            // 第二个参数
pstmt.setString(3, "zhang@example.com");

// 3. 执行
int rows = pstmt.executeUpdate();

// 4. 重用（设置新参数再次执行）
pstmt.setString(1, "李四");
pstmt.setInt(2, 30);
pstmt.setString(3, "li@example.com");
rows = pstmt.executeUpdate();
```

### **参数类型设置**：
```java
// 各种数据类型的设置方法
pstmt.setString(1, "字符串");
pstmt.setInt(2, 100);
pstmt.setLong(3, 1000L);
pstmt.setDouble(4, 99.99);
pstmt.setFloat(5, 9.9f);
pstmt.setBoolean(6, true);
pstmt.setDate(7, java.sql.Date.valueOf("2024-01-19"));
pstmt.setTimestamp(8, new Timestamp(System.currentTimeMillis()));
pstmt.setNull(9, Types.VARCHAR);  // 设置NULL值
pstmt.setObject(10, anyObject);   // 通用设置
```

### **批量操作**：
```java
String sql = "INSERT INTO users(name, age) VALUES(?, ?)";
PreparedStatement pstmt = conn.prepareStatement(sql);

for (int i = 1; i <= 1000; i++) {
    pstmt.setString(1, "User" + i);
    pstmt.setInt(2, 20 + i % 30);
    pstmt.addBatch();  // 添加到批处理
    
    // 每100条执行一次
    if (i % 100 == 0) {
        pstmt.executeBatch();
        pstmt.clearBatch();
    }
}
pstmt.executeBatch();  // 执行剩余
```

### **高级功能**：
```java
// 1. 返回自增主键
String sql = "INSERT INTO users(name) VALUES(?)";
PreparedStatement pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
pstmt.setString(1, "王五");
pstmt.executeUpdate();

ResultSet rs = pstmt.getGeneratedKeys();
if (rs.next()) {
    int id = rs.getInt(1);
}

// 2. 指定返回的列
pstmt = conn.prepareStatement(sql, new String[]{"id", "create_time"});

// 3. 设置结果集类型
pstmt = conn.prepareStatement(sql, 
    ResultSet.TYPE_SCROLL_INSENSITIVE,  // 可滚动
    ResultSet.CONCUR_READ_ONLY          // 只读
);

// 4. 存储过程调用
String callSql = "{call get_user_by_id(?)}";
CallableStatement cstmt = conn.prepareCall(callSql);
cstmt.setInt(1, 100);
ResultSet rs = cstmt.executeQuery();
```

## 📊 **Statement vs PreparedStatement 对比**

| 特性         | Statement        | PreparedStatement    |
| ------------ | ---------------- | -------------------- |
| **SQL编译**  | 每次执行都编译   | 预编译一次，重复使用 |
| **性能**     | 较差             | 较好                 |
| **SQL注入**  | 容易发生         | 防止注入             |
| **参数化**   | 不支持           | 支持                 |
| **可读性**   | 差（拼接SQL）    | 好                   |
| **使用场景** | DDL、简单静态SQL | DML、带参数的SQL     |

## 💡 **完整使用示例**

```java
import java.sql.*;

public class JdbcExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb?"
                   + "useSSL=false&serverTimezone=Asia/Shanghai";
        String user = "root";
        String password = "123456";
        
        // 使用try-with-resources自动关闭资源
        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            
            // 1. 创建表（Statement）
            try (Statement stmt = conn.createStatement()) {
                String createTable = "CREATE TABLE IF NOT EXISTS users ("
                                  + "id INT PRIMARY KEY AUTO_INCREMENT, "
                                  + "name VARCHAR(50) NOT NULL, "
                                  + "age INT, "
                                  + "email VARCHAR(100))";
                stmt.execute(createTable);
            }
            
            // 2. 插入数据（PreparedStatement）
            String insertSql = "INSERT INTO users(name, age, email) VALUES(?, ?, ?)";
            try (PreparedStatement pstmt = conn.prepareStatement(insertSql, 
                    Statement.RETURN_GENERATED_KEYS)) {
                
                pstmt.setString(1, "张三");
                pstmt.setInt(2, 25);
                pstmt.setString(3, "zhang@example.com");
                pstmt.executeUpdate();
                
                // 获取生成的主键
                ResultSet keys = pstmt.getGeneratedKeys();
                if (keys.next()) {
                    System.out.println("插入成功，ID: " + keys.getInt(1));
                }
            }
            
            // 3. 查询数据
            String selectSql = "SELECT * FROM users WHERE age > ?";
            try (PreparedStatement pstmt = conn.prepareStatement(selectSql)) {
                pstmt.setInt(1, 20);
                
                try (ResultSet rs = pstmt.executeQuery()) {
                    while (rs.next()) {
                        int id = rs.getInt("id");
                        String name = rs.getString("name");
                        int age = rs.getInt("age");
                        String email = rs.getString("email");
                        
                        System.out.printf("ID: %d, 姓名: %s, 年龄: %d, 邮箱: %s%n", 
                                         id, name, age, email);
                    }
                }
            }
            
            // 4. 事务处理
            conn.setAutoCommit(false);  // 关闭自动提交
            try {
                // 执行多个更新操作
                String updateSql = "UPDATE users SET age = ? WHERE id = ?";
                try (PreparedStatement pstmt = conn.prepareStatement(updateSql)) {
                    pstmt.setInt(1, 30);
                    pstmt.setInt(2, 1);
                    pstmt.executeUpdate();
                    
                    // 模拟异常
                    // int x = 1 / 0;
                    
                    conn.commit();  // 提交事务
                    System.out.println("事务提交成功");
                }
            } catch (Exception e) {
                conn.rollback();  // 回滚事务
                System.out.println("事务回滚: " + e.getMessage());
            } finally {
                conn.setAutoCommit(true);  // 恢复自动提交
            }
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## 🚀 **最佳实践**

1. **资源管理**：始终使用 try-with-resources
2. **使用PreparedStatement**：避免SQL注入，提高性能
3. **连接池**：生产环境使用连接池（HikariCP、Druid）
4. **异常处理**：正确处理SQLException
5. **关闭资源**：按 ResultSet → Statement → Connection 顺序关闭
6. **SQL管理**：考虑使用MyBatis等ORM框架

```java
// 最佳实践模板
public class UserDao {
    private DataSource dataSource;  // 使用连接池
    
    public User findById(int userId) {
        String sql = "SELECT id, name, email FROM users WHERE id = ?";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, userId);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    User user = new User();
                    user.setId(rs.getInt("id"));
                    user.setName(rs.getString("name"));
                    user.setEmail(rs.getString("email"));
                    return user;
                }
            }
        } catch (SQLException e) {
            throw new RuntimeException("Database error", e);
        }
        return null;
    }
}
```

## 📚 **总结**

- **Class.forName()**：加载驱动（JDBC 4.0+ 可省略）
- **Connection**：数据库连接，事务管理的基础
- **Statement**：静态SQL执行，简单但危险
- **PreparedStatement**：预编译SQL，安全高效，推荐使用

理解这些组件的关系是掌握 JDBC 编程的关键！

# JDBC 完整教程：基础 → 进阶 → 高级

## 📚 **目录**
1. [基础篇](#基础篇) - JDBC 核心概念和基本操作
2. [进阶篇](#进阶篇) - PreparedStatement、事务、连接池
3. [高级篇](#高级篇) - 性能优化、监控、最佳实践

---

# 一、基础篇

## 1.1 JDBC 是什么？

**JDBC**（Java Database Connectivity）是 Java 连接数据库的标准 API

```java
// JDBC 核心组件关系
应用程序 → JDBC API → JDBC 驱动管理器 → 数据库驱动 → 数据库
```

## 1.2 环境准备

### 添加 MySQL 驱动依赖

**Maven:**
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

**Gradle:**
```groovy
implementation 'mysql:mysql-connector-java:8.0.33'
```

**手动添加:** 下载 mysql-connector-java.jar

## 1.3 第一个 JDBC 程序

```java
import java.sql.*;

public class JDBCBasicDemo {
    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        
        try {
            // 1. 加载驱动（JDBC 4.0+ 可省略）
            Class.forName("com.mysql.cj.jdbc.Driver");
            
            // 2. 建立连接
            String url = "jdbc:mysql://localhost:3306/testdb";
            String user = "root";
            String password = "123456";
            conn = DriverManager.getConnection(url, user, password);
            
            // 3. 创建 Statement
            stmt = conn.createStatement();
            
            // 4. 执行查询
            String sql = "SELECT id, name, age FROM users";
            rs = stmt.executeQuery(sql);
            
            // 5. 处理结果集
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                int age = rs.getInt("age");
                System.out.println("ID: " + id + ", Name: " + name + ", Age: " + age);
            }
            
        } catch (ClassNotFoundException e) {
            System.err.println("驱动加载失败: " + e.getMessage());
        } catch (SQLException e) {
            System.err.println("数据库错误: " + e.getMessage());
        } finally {
            // 6. 释放资源（重要！）
            try {
                if (rs != null) rs.close();
                if (stmt != null) stmt.close();
                if (conn != null) conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 1.4 数据库基本操作

### 创建表
```java
public void createTable() throws SQLException {
    String sql = "CREATE TABLE IF NOT EXISTS students (" +
                 "id INT PRIMARY KEY AUTO_INCREMENT," +
                 "name VARCHAR(50) NOT NULL," +
                 "gender CHAR(1)," +
                 "score DECIMAL(5,2)," +
                 "birthday DATE," +
                 "created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP" +
                 ")";
    
    try (Connection conn = getConnection();
         Statement stmt = conn.createStatement()) {
        stmt.execute(sql);
        System.out.println("表创建成功");
    }
}
```

### 插入数据
```java
public void insertStudent() throws SQLException {
    String sql = "INSERT INTO students(name, gender, score, birthday) " +
                 "VALUES('张三', 'M', 89.5, '2000-01-15')";
    
    try (Connection conn = getConnection();
         Statement stmt = conn.createStatement()) {
        int rows = stmt.executeUpdate(sql);
        System.out.println("插入成功，影响行数: " + rows);
    }
}
```

### 更新数据
```java
public void updateStudent() throws SQLException {
    String sql = "UPDATE students SET score = 95.0 WHERE name = '张三'";
    
    try (Connection conn = getConnection();
         Statement stmt = conn.createStatement()) {
        int rows = stmt.executeUpdate(sql);
        System.out.println("更新成功，影响行数: " + rows);
    }
}
```

### 删除数据
```java
public void deleteStudent() throws SQLException {
    String sql = "DELETE FROM students WHERE score < 60";
    
    try (Connection conn = getConnection();
         Statement stmt = conn.createStatement()) {
        int rows = stmt.executeUpdate(sql);
        System.out.println("删除成功，影响行数: " + rows);
    }
}
```

## 1.5 ResultSet 详解

```java
public void resultSetDemo() throws SQLException {
    // 1. 写SQL语句：查询students表的所有数据
    String sql = "SELECT * FROM students";
    
    // 2. 创建连接，执行查询（try-with-resources自动关闭）
    try (Connection conn = getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        
        // ==================== 第一部分：查看表结构 ====================
        
        // 获取结果的"元数据"（就是表的结构信息）
        ResultSetMetaData metaData = rs.getMetaData();
        // 一共有多少列？（比如：id, name, gender, score...）
        int columnCount = metaData.getColumnCount();
        
        System.out.println("表结构:");
        for (int i = 1; i <= columnCount; i++) {  // 注意：列索引从1开始！
            // 获取第i列的列名和数据类型
            System.out.println(metaData.getColumnName(i) + " - " +
                             metaData.getColumnTypeName(i));
        }
        // 输出示例：
        // id - INT
        // name - VARCHAR
        // gender - CHAR
        // score - DECIMAL
        
        // ==================== 第二部分：遍历数据行 ====================
        
        System.out.println("\n数据:");
        
        // 循环条件：rs.next() 移动到下一行，如果有数据返回true，没有返回false
        while (rs.next()) {
            // 现在rs指向当前行，我们可以获取这行的数据
            
            // ---------- 方法1：按列名获取（推荐，易读） ----------
            int id = rs.getInt("id");        // 获取"id"列的值，转成int
            String name = rs.getString("name"); // 获取"name"列的值
            
            // ---------- 方法2：按索引获取（更快，但要知道列顺序） ----------
            String gender = rs.getString(3);    // 第3列（gender）
            double score = rs.getDouble(4);     // 第4列（score）
            Date birthday = rs.getDate(5);      // 第5列（birthday）
            Timestamp createdAt = rs.getTimestamp(6); // 第6列（created_at）
            
            // ---------- 特殊处理：NULL值 ----------
            String email = rs.getString("email");  // 假设可能为NULL
            // 如果刚才获取的email是NULL，wasNull()返回true
            if (rs.wasNull()) {
                email = "暂无邮箱";  // 给一个默认值
            }
            
            // ---------- 输出这一行的数据 ----------
            System.out.printf("ID: %d, 姓名: %s, 性别: %s, 分数: %.2f%n",
                             id, name, gender, score);
        }
        // 循环结束：所有行都处理完了
    }
    // try块结束，自动关闭conn、stmt、rs
}
```

## 1.6 异常处理

```java
public class JDBCHelper {
    
    public static void executeQuery(String sql) {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        
        try {
            conn = getConnection();
            stmt = conn.createStatement();
            rs = stmt.executeQuery(sql);
            
            // 处理结果...
            
        } catch (SQLException e) {
            // 处理不同类型的SQL异常
            handleSQLException(e);
        } finally {
            closeResources(conn, stmt, rs);
        }
    }
    
    private static void handleSQLException(SQLException e) {
        System.err.println("SQL状态码: " + e.getSQLState());
        System.err.println("错误代码: " + e.getErrorCode());
        System.err.println("错误信息: " + e.getMessage());
        
        // 常见错误处理
        switch (e.getErrorCode()) {
            case 1062: // 重复键
                System.err.println("数据重复，请检查唯一约束");
                break;
            case 1451: // 外键约束
                System.err.println("存在关联数据，无法删除");
                break;
            case 1045: // 权限错误
                System.err.println("数据库访问权限不足");
                break;
            default:
                e.printStackTrace();
        }
    }
    
    private static void closeResources(Connection conn, Statement stmt, ResultSet rs) {
        // 关闭顺序：ResultSet → Statement → Connection
        try {
            if (rs != null && !rs.isClosed()) rs.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        try {
            if (stmt != null && !stmt.isClosed()) stmt.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        try {
            if (conn != null && !conn.isClosed()) conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## 1.7 工具类封装

```java
public class DBUtil {
    
    private static final String URL;
    private static final String USER;
    private static final String PASSWORD;
    
    static {
        // 从配置文件加载（推荐）
        // Properties props = new Properties();
        // props.load(new FileInputStream("db.properties"));
        
        URL = "jdbc:mysql://localhost:3306/testdb?useSSL=false&serverTimezone=Asia/Shanghai";
        USER = "root";
        PASSWORD = "123456";
    }
    
    // 获取连接
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
    
    // 执行查询（返回ResultSet）
    public static ResultSet executeQuery(String sql) throws SQLException {
        Connection conn = getConnection();
        Statement stmt = conn.createStatement();
        return stmt.executeQuery(sql);
    }
    
    // 执行更新（返回影响行数）
    public static int executeUpdate(String sql) throws SQLException {
        try (Connection conn = getConnection();
             Statement stmt = conn.createStatement()) {
            return stmt.executeUpdate(sql);
        }
    }
    
    // 关闭资源（使用try-with-resources更安全）
    public static void close(Connection conn, Statement stmt, ResultSet rs) {
        closeResultSet(rs);
        closeStatement(stmt);
        closeConnection(conn);
    }
    
    private static void closeConnection(Connection conn) {
        if (conn != null) {
            try {
                if (!conn.isClosed()) {
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    
    private static void closeStatement(Statement stmt) {
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    
    private static void closeResultSet(ResultSet rs) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

---

# 二、进阶篇

## 2.1 PreparedStatement（核心）

### 为什么使用 PreparedStatement？

```java
public class PreparedStatementDemo {
    
    // ❌ 不安全的方式（SQL注入风险）
    public void unsafeLogin(String username, String password) throws SQLException {
        // 如果 username 输入: admin' OR '1'='1
        String sql = "SELECT * FROM users WHERE username = '" + username 
                   + "' AND password = '" + password + "'";
        // 实际SQL: SELECT * FROM users WHERE username = 'admin' OR '1'='1' ...
        // 会返回所有用户！
    }
    
    // ✅ 安全的方式
    public boolean safeLogin(String username, String password) throws SQLException {
        String sql = "SELECT COUNT(*) FROM users WHERE username = ? AND password = ?";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, username);  // 第一个参数
            pstmt.setString(2, password);  // 第二个参数
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return rs.getInt(1) > 0;
                }
            }
        }
        return false;
    }
}
```

### 完整 CRUD 示例

```java
public class UserCRUD {
    
    // 插入用户
    public int insertUser(User user) throws SQLException {
        String sql = "INSERT INTO users(username, password, email, age) VALUES(?, ?, ?, ?)";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql, 
                 Statement.RETURN_GENERATED_KEYS)) {
            
            pstmt.setString(1, user.getUsername());
            pstmt.setString(2, user.getPassword());
            pstmt.setString(3, user.getEmail());
            
            if (user.getAge() != null) {
                pstmt.setInt(4, user.getAge());
            } else {
                pstmt.setNull(4, Types.INTEGER);
            }
            
            int rows = pstmt.executeUpdate();
            
            // 获取自增主键
            try (ResultSet keys = pstmt.getGeneratedKeys()) {
                if (keys.next()) {
                    user.setId(keys.getInt(1));
                }
            }
            
            return rows;
        }
    }
    
    // 查询用户
    public User findUserById(int id) throws SQLException {
        String sql = "SELECT * FROM users WHERE id = ?";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return mapResultSetToUser(rs);
                }
            }
        }
        return null;
    }
    
    // 更新用户
    public int updateUser(User user) throws SQLException {
        String sql = "UPDATE users SET username = ?, email = ?, age = ? WHERE id = ?";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, user.getUsername());
            pstmt.setString(2, user.getEmail());
            pstmt.setInt(3, user.getAge());
            pstmt.setInt(4, user.getId());
            
            return pstmt.executeUpdate();
        }
    }
    
    // 删除用户
    public int deleteUser(int id) throws SQLException {
        String sql = "DELETE FROM users WHERE id = ?";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            return pstmt.executeUpdate();
        }
    }
    
    private User mapResultSetToUser(ResultSet rs) throws SQLException {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setUsername(rs.getString("username"));
        user.setPassword(rs.getString("password"));
        user.setEmail(rs.getString("email"));
        user.setAge(rs.getInt("age"));
        
        // 处理可能的NULL值
        if (rs.wasNull()) {
            user.setAge(null);
        }
        
        return user;
    }
}
```

## 2.2 批处理操作

```java
import java.sql.*;
import java.util.*;

public class BatchOperationDemo {
    
    // 正确的数据库URL（必须加 rewriteBatchedStatements=true）
    private static final String BATCH_URL = 
        "jdbc:mysql://localhost:3306/testdb?" +
        "rewriteBatchedStatements=true&" +  // 关键参数！开启批处理重写
        "useSSL=false&" +
        "serverTimezone=Asia/Shanghai&" +
        "characterEncoding=utf8";
    
    // 获取批处理专用的连接
    private Connection getBatchConnection() throws SQLException {
        return DriverManager.getConnection(BATCH_URL, "root", "123456");
    }
    
    // 优化后的批量插入方法
    public void batchInsert(List<User> users) throws SQLException {
        String sql = "INSERT INTO users(username, email, age) VALUES(?, ?, ?)";
        
        // ❌ 错误：try-with-resources中不能在catch块里引用conn
        // Connection conn = null; // 需要这样声明
        
        Connection conn = null;
        PreparedStatement pstmt = null;
        
        try {
            conn = getBatchConnection();
            pstmt = conn.prepareStatement(sql);
            
            // 1. 关闭自动提交，提高性能
            conn.setAutoCommit(false);
            
            int batchSize = 1000;  // 每批1000条
            int count = 0;
            
            for (User user : users) {
                pstmt.setString(1, user.getUsername());
                pstmt.setString(2, user.getEmail());
                pstmt.setInt(3, user.getAge());
                pstmt.addBatch();  // 添加到批处理
                
                count++;
                
                // 2. 每1000条执行一次，避免内存溢出
                if (count % batchSize == 0) {
                    pstmt.executeBatch();
                    pstmt.clearBatch();  // 清空批处理
                    System.out.println("已批量插入 " + count + " 条记录");
                }
            }
            
            // 3. 执行剩余的
            if (count % batchSize != 0) {
                int[] results = pstmt.executeBatch();
                System.out.println("插入剩余 " + (count % batchSize) + " 条记录");
            }
            
            // 4. 提交事务
            conn.commit();
            System.out.println("批量插入完成，总计: " + count + " 条");
            
        } catch (SQLException e) {
            // 5. 发生异常时回滚
            if (conn != null) {
                conn.rollback();
            }
            throw new SQLException("批量插入失败，已回滚", e);
            
        } finally {
            // 6. 恢复自动提交并关闭资源
            if (conn != null) {
                try {
                    conn.setAutoCommit(true);
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (pstmt != null) {
                try {
                    pstmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    // 批量更新（优化版）
    public int[] batchUpdate(List<User> users) throws SQLException {
        String sql = "UPDATE users SET email = ?, age = ? WHERE id = ?";
        
        Connection conn = null;
        PreparedStatement pstmt = null;
        
        try {
            conn = getBatchConnection();
            pstmt = conn.prepareStatement(sql);
            
            conn.setAutoCommit(false);
            
            for (User user : users) {
                pstmt.setString(1, user.getEmail());
                pstmt.setInt(2, user.getAge());
                pstmt.setInt(3, user.getId());
                pstmt.addBatch();
            }
            
            // 执行批处理
            int[] updateCounts = pstmt.executeBatch();
            conn.commit();
            
            // 统计结果
            int success = 0, failed = 0;
            for (int i = 0; i < updateCounts.length; i++) {
                if (updateCounts[i] == Statement.EXECUTE_FAILED) {
                    failed++;
                    System.err.println("第 " + (i+1) + " 条更新失败: " + users.get(i));
                } else if (updateCounts[i] >= 0) {
                    success += updateCounts[i];
                }
            }
            
            System.out.println("批量更新完成: 成功更新 " + success + " 行，失败 " + failed + " 条");
            return updateCounts;
            
        } catch (SQLException e) {
            if (conn != null) {
                conn.rollback();
            }
            throw new SQLException("批量更新失败，已回滚", e);
            
        } finally {
            // 清理资源
            if (pstmt != null) pstmt.close();
            if (conn != null) {
                conn.setAutoCommit(true);
                conn.close();
            }
        }
    }
}
```

```
import java.sql.*;
import java.util.*;

/**
 * 高级批处理工具类
 */
public class AdvancedBatchProcessor {
    
    // 完整的批处理优化参数
    private static final String OPTIMIZED_URL = 
        "jdbc:mysql://localhost:3306/testdb?" +
        "rewriteBatchedStatements=true&" +      // 1. 开启批处理重写
        "useServerPrepStmts=true&" +            // 2. 使用服务器端预编译
        "cachePrepStmts=true&" +                // 3. 缓存预编译语句
        "prepStmtCacheSize=250&" +              // 4. 缓存大小
        "prepStmtCacheSqlLimit=2048&" +         // 5. SQL长度限制
        "useSSL=false&" +
        "serverTimezone=Asia/Shanghai&" +
        "characterEncoding=utf8";
    
    /**
     * 高性能批量插入
     * @param tableName 表名
     * @param dataList 数据列表
     * @param batchSize 批大小（推荐1000-5000）
     */
    public void highPerformanceBatchInsert(
            String tableName, 
            List<Map<String, Object>> dataList, 
            int batchSize) throws SQLException {
        
        if (dataList == null || dataList.isEmpty()) {
            return;
        }
        
        // 从第一条数据获取列名
        Map<String, Object> firstRow = dataList.get(0);
        Set<String> columns = firstRow.keySet();
        
        // 构建SQL：INSERT INTO table(col1, col2) VALUES(?, ?)
        StringBuilder sqlBuilder = new StringBuilder();
        sqlBuilder.append("INSERT INTO ").append(tableName).append(" (");
        
        StringBuilder placeholders = new StringBuilder(" VALUES (");
        for (String column : columns) {
            sqlBuilder.append(column).append(", ");
            placeholders.append("?, ");
        }
        
        // 移除最后的 ", "
        sqlBuilder.setLength(sqlBuilder.length() - 2);
        placeholders.setLength(placeholders.length() - 2);
        
        sqlBuilder.append(")").append(placeholders).append(")");
        String sql = sqlBuilder.toString();
        
        System.out.println("批处理SQL: " + sql);
        
        long startTime = System.currentTimeMillis();
        
        try (Connection conn = DriverManager.getConnection(OPTIMIZED_URL, "root", "123456");
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            int totalCount = dataList.size();
            int processed = 0;
            
            for (Map<String, Object> row : dataList) {
                int paramIndex = 1;
                for (String column : columns) {
                    Object value = row.get(column);
                    pstmt.setObject(paramIndex++, value);
                }
                pstmt.addBatch();
                processed++;
                
                // 分批执行
                if (processed % batchSize == 0 || processed == totalCount) {
                    int[] results = pstmt.executeBatch();
                    pstmt.clearBatch();
                    
                    // 计算进度
                    double progress = (double) processed / totalCount * 100;
                    System.out.printf("进度: %.1f%% (%d/%d)%n", 
                                     progress, processed, totalCount);
                }
            }
            
            conn.commit();
            
            long endTime = System.currentTimeMillis();
            System.out.printf("批量插入完成! 总计 %d 条，耗时 %.2f 秒，平均 %.0f 条/秒%n",
                             totalCount, 
                             (endTime - startTime) / 1000.0,
                             totalCount / ((endTime - startTime) / 1000.0));
            
        } catch (SQLException e) {
            throw new SQLException("批量插入失败: " + e.getMessage(), e);
        }
    }
    
    /**
     * 批量插入并返回生成的主键
     */
    public List<Integer> batchInsertWithReturnKeys(List<User> users) throws SQLException {
        String sql = "INSERT INTO users(username, email, age) VALUES(?, ?, ?)";
        List<Integer> generatedKeys = new ArrayList<>();
        
        try (Connection conn = DriverManager.getConnection(OPTIMIZED_URL, "root", "123456");
             PreparedStatement pstmt = conn.prepareStatement(
                 sql, Statement.RETURN_GENERATED_KEYS)) {
            
            conn.setAutoCommit(false);
            
            for (User user : users) {
                pstmt.setString(1, user.getUsername());
                pstmt.setString(2, user.getEmail());
                pstmt.setInt(3, user.getAge());
                pstmt.addBatch();
            }
            
            pstmt.executeBatch();
            
            // 获取所有生成的主键
            try (ResultSet rs = pstmt.getGeneratedKeys()) {
                while (rs.next()) {
                    generatedKeys.add(rs.getInt(1));
                }
            }
            
            conn.commit();
            
        } catch (SQLException e) {
            throw new SQLException("批量插入失败，无法获取主键", e);
        }
        
        return generatedKeys;
    }
}
```

## 🚀 **更高级的批处理工具类**

```java
import java.sql.*;
import java.util.*;

/**
 * 高级批处理工具类
 */
public class AdvancedBatchProcessor {
    
    // 完整的批处理优化参数
    private static final String OPTIMIZED_URL = 
        "jdbc:mysql://localhost:3306/testdb?" +
        "rewriteBatchedStatements=true&" +      // 1. 开启批处理重写
        "useServerPrepStmts=true&" +            // 2. 使用服务器端预编译
        "cachePrepStmts=true&" +                // 3. 缓存预编译语句
        "prepStmtCacheSize=250&" +              // 4. 缓存大小
        "prepStmtCacheSqlLimit=2048&" +         // 5. SQL长度限制
        "useSSL=false&" +
        "serverTimezone=Asia/Shanghai&" +
        "characterEncoding=utf8";
    
    /**
     * 高性能批量插入
     * @param tableName 表名
     * @param dataList 数据列表
     * @param batchSize 批大小（推荐1000-5000）
     */
    public void highPerformanceBatchInsert(
            String tableName, 
            List<Map<String, Object>> dataList, 
            int batchSize) throws SQLException {
        
        if (dataList == null || dataList.isEmpty()) {
            return;
        }
        
        // 从第一条数据获取列名
        Map<String, Object> firstRow = dataList.get(0);
        Set<String> columns = firstRow.keySet();
        
        // 构建SQL：INSERT INTO table(col1, col2) VALUES(?, ?)
        StringBuilder sqlBuilder = new StringBuilder();
        sqlBuilder.append("INSERT INTO ").append(tableName).append(" (");
        
        StringBuilder placeholders = new StringBuilder(" VALUES (");
        for (String column : columns) {
            sqlBuilder.append(column).append(", ");
            placeholders.append("?, ");
        }
        
        // 移除最后的 ", "
        sqlBuilder.setLength(sqlBuilder.length() - 2);
        placeholders.setLength(placeholders.length() - 2);
        
        sqlBuilder.append(")").append(placeholders).append(")");
        String sql = sqlBuilder.toString();
        
        System.out.println("批处理SQL: " + sql);
        
        long startTime = System.currentTimeMillis();
        
        try (Connection conn = DriverManager.getConnection(OPTIMIZED_URL, "root", "123456");
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            int totalCount = dataList.size();
            int processed = 0;
            
            for (Map<String, Object> row : dataList) {
                int paramIndex = 1;
                for (String column : columns) {
                    Object value = row.get(column);
                    pstmt.setObject(paramIndex++, value);
                }
                pstmt.addBatch();
                processed++;
                
                // 分批执行
                if (processed % batchSize == 0 || processed == totalCount) {
                    int[] results = pstmt.executeBatch();
                    pstmt.clearBatch();
                    
                    // 计算进度
                    double progress = (double) processed / totalCount * 100;
                    System.out.printf("进度: %.1f%% (%d/%d)%n", 
                                     progress, processed, totalCount);
                }
            }
            
            conn.commit();
            
            long endTime = System.currentTimeMillis();
            System.out.printf("批量插入完成! 总计 %d 条，耗时 %.2f 秒，平均 %.0f 条/秒%n",
                             totalCount, 
                             (endTime - startTime) / 1000.0,
                             totalCount / ((endTime - startTime) / 1000.0));
            
        } catch (SQLException e) {
            throw new SQLException("批量插入失败: " + e.getMessage(), e);
        }
    }
    
    /**
     * 批量插入并返回生成的主键
     */
    public List<Integer> batchInsertWithReturnKeys(List<User> users) throws SQLException {
        String sql = "INSERT INTO users(username, email, age) VALUES(?, ?, ?)";
        List<Integer> generatedKeys = new ArrayList<>();
        
        try (Connection conn = DriverManager.getConnection(OPTIMIZED_URL, "root", "123456");
             PreparedStatement pstmt = conn.prepareStatement(
                 sql, Statement.RETURN_GENERATED_KEYS)) {
            
            conn.setAutoCommit(false);
            
            for (User user : users) {
                pstmt.setString(1, user.getUsername());
                pstmt.setString(2, user.getEmail());
                pstmt.setInt(3, user.getAge());
                pstmt.addBatch();
            }
            
            pstmt.executeBatch();
            
            // 获取所有生成的主键
            try (ResultSet rs = pstmt.getGeneratedKeys()) {
                while (rs.next()) {
                    generatedKeys.add(rs.getInt(1));
                }
            }
            
            conn.commit();
            
        } catch (SQLException e) {
            throw new SQLException("批量插入失败，无法获取主键", e);
        }
        
        return generatedKeys;
    }
}
```



## 2.3 事务管理

```java
public class TransactionDemo {
    
    // 简单事务示例：转账
    public boolean transferMoney(int fromId, int toId, BigDecimal amount) {
        Connection conn = null;
        PreparedStatement pstmt1 = null;
        PreparedStatement pstmt2 = null;
        
        try {
            conn = DBUtil.getConnection();
            conn.setAutoCommit(false);  // 开启事务
            
            // 1. 扣款
            String sql1 = "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?";
            pstmt1 = conn.prepareStatement(sql1);
            pstmt1.setBigDecimal(1, amount);
            pstmt1.setInt(2, fromId);
            pstmt1.setBigDecimal(3, amount);
            int rows1 = pstmt1.executeUpdate();
            
            if (rows1 == 0) {
                throw new SQLException("余额不足或账户不存在");
            }
            
            // 2. 收款
            String sql2 = "UPDATE accounts SET balance = balance + ? WHERE id = ?";
            pstmt2 = conn.prepareStatement(sql2);
            pstmt2.setBigDecimal(1, amount);
            pstmt2.setInt(2, toId);
            pstmt2.executeUpdate();
            
            // 提交事务
            conn.commit();
            return true;
            
        } catch (SQLException e) {
            // 回滚事务
            if (conn != null) {
                try {
                    conn.rollback();
                    System.err.println("事务回滚: " + e.getMessage());
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
            return false;
            
        } finally {
            // 关闭资源
            DBUtil.closeStatement(pstmt1);
            DBUtil.closeStatement(pstmt2);
            DBUtil.closeConnection(conn);
        }
    }
    
    // 事务隔离级别
    public void transactionIsolation() throws SQLException {
        Connection conn = DBUtil.getConnection();
        
        // 设置事务隔离级别
        conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
        
        // 常用隔离级别：
        // TRANSACTION_READ_UNCOMMITTED - 读未提交（脏读）
        // TRANSACTION_READ_COMMITTED   - 读已提交（Oracle默认）
        // TRANSACTION_REPEATABLE_READ  - 可重复读（MySQL默认）
        // TRANSACTION_SERIALIZABLE     - 串行化（最严格）
        
        try {
            conn.setAutoCommit(false);
            
            // 执行事务操作...
            
            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        } finally {
            conn.close();
        }
    }
}
```

## 2.4 存储过程调用

```java
public class StoredProcedureDemo {
    
    // 调用无返回值的存储过程
    public void callSimpleProc(int userId) throws SQLException {
        String sql = "{CALL update_user_status(?)}";
        
        try (Connection conn = DBUtil.getConnection();
             CallableStatement cstmt = conn.prepareCall(sql)) {
            
            cstmt.setInt(1, userId);
            cstmt.execute();
            System.out.println("存储过程执行完成");
        }
    }
    
    // 调用有返回值的存储过程
    public User callProcWithResult(int userId) throws SQLException {
        String sql = "{CALL get_user_by_id(?, ?, ?)}";
        
        try (Connection conn = DBUtil.getConnection();
             CallableStatement cstmt = conn.prepareCall(sql)) {
            
            // 设置输入参数
            cstmt.setInt(1, userId);
            
            // 注册输出参数
            cstmt.registerOutParameter(2, Types.VARCHAR);  // 用户名
            cstmt.registerOutParameter(3, Types.VARCHAR);  // 邮箱
            
            cstmt.execute();
            
            // 获取输出参数
            User user = new User();
            user.setUsername(cstmt.getString(2));
            user.setEmail(cstmt.getString(3));
            
            return user;
        }
    }
    
    // 调用返回结果集的存储过程
    public List<User> callProcWithResultSet() throws SQLException {
        String sql = "{CALL get_active_users()}";
        List<User> users = new ArrayList<>();
        
        try (Connection conn = DBUtil.getConnection();
             CallableStatement cstmt = conn.prepareCall(sql)) {
            
            boolean hasResults = cstmt.execute();
            
            // 处理结果集
            while (hasResults) {
                try (ResultSet rs = cstmt.getResultSet()) {
                    while (rs.next()) {
                        users.add(mapResultSetToUser(rs));
                    }
                }
                hasResults = cstmt.getMoreResults();
            }
        }
        return users;
    }
}
```

## 2.5 连接池技术

### HikariCP 连接池（推荐）

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class ConnectionPoolDemo {
    private static HikariDataSource dataSource;
    
    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/testdb");
        config.setUsername("root");
        config.setPassword("123456");
        
        // 连接池配置
        config.setMaximumPoolSize(20);          // 最大连接数
        config.setMinimumIdle(5);               // 最小空闲连接
        config.setConnectionTimeout(30000);     // 连接超时30秒
        config.setIdleTimeout(600000);          // 空闲超时10分钟
        config.setMaxLifetime(1800000);         // 连接最大生存时间30分钟
        config.setConnectionTestQuery("SELECT 1"); // 测试查询
        
        // MySQL特定配置
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        
        dataSource = new HikariDataSource(config);
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
    
    public static void close() {
        if (dataSource != null && !dataSource.isClosed()) {
            dataSource.close();
        }
    }
    
    // 监控连接池状态
    public static void printPoolStats() {
        System.out.println("连接池状态:");
        System.out.println("活动连接: " + dataSource.getHikariPoolMXBean().getActiveConnections());
        System.out.println("空闲连接: " + dataSource.getHikariPoolMXBean().getIdleConnections());
        System.out.println("等待连接: " + dataSource.getHikariPoolMXBean().getThreadsAwaitingConnection());
        System.out.println("总连接: " + dataSource.getHikariPoolMXBean().getTotalConnections());
    }
}
```

### Druid 连接池

```java
import com.alibaba.druid.pool.DruidDataSource;

public class DruidPoolDemo {
    private static DruidDataSource dataSource;
    
    static {
        dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:mysql://localhost:3306/testdb");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        
        // 基本配置
        dataSource.setInitialSize(5);      // 初始连接数
        dataSource.setMinIdle(5);          // 最小空闲连接
        dataSource.setMaxActive(20);       // 最大连接数
        dataSource.setMaxWait(60000);      // 获取连接超时时间
        
        // 监控配置
        dataSource.setTimeBetweenEvictionRunsMillis(60000); // 检测间隔
        dataSource.setMinEvictableIdleTimeMillis(300000);   // 最小生存时间
        
        // 测试配置
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(false);
        dataSource.setTestOnReturn(false);
        dataSource.setValidationQuery("SELECT 1");
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```

---

# 三、高级篇

## 3.1 性能优化

### 查询优化技巧

```java
public class PerformanceOptimizer {
    
    // 1. 使用正确的fetchSize
    public void optimizeFetchSize() throws SQLException {
        String sql = "SELECT * FROM large_table";
        
        try (Connection conn = DBUtil.getConnection();
             Statement stmt = conn.createStatement()) {
            
            // 设置合适的fetchSize（默认是0，一次性获取所有结果）
            stmt.setFetchSize(1000);  // 每次从数据库获取1000行
            
            try (ResultSet rs = stmt.executeQuery(sql)) {
                while (rs.next()) {
                    // 处理大数据集时，分批次获取可以提高性能
                }
            }
        }
    }
    
    // 2. 只获取需要的列
    public void selectSpecificColumns() throws SQLException {
        // ❌ 不好：SELECT * FROM users
        // ✅ 好：只选择需要的列
        String sql = "SELECT id, username, email FROM users";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            // 设置查询超时
            pstmt.setQueryTimeout(30);  // 30秒超时
            
            try (ResultSet rs = pstmt.executeQuery()) {
                // 处理结果
            }
        }
    }
    
    // 3. 使用分页避免内存溢出
    public List<User> paginateUsers(int page, int size) throws SQLException {
        String sql = "SELECT * FROM users ORDER BY id LIMIT ? OFFSET ?";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, size);
            pstmt.setInt(2, (page - 1) * size);
            
            List<User> users = new ArrayList<>();
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    users.add(mapResultSetToUser(rs));
                }
            }
            return users;
        }
    }
    
    // 4. 使用游标处理大数据集
    public void processLargeDatasetWithCursor() throws SQLException {
        String sql = "SELECT * FROM huge_table";
        
        try (Connection conn = DBUtil.getConnection()) {
            // 创建可滚动、只读的结果集
            PreparedStatement pstmt = conn.prepareStatement(
                sql, 
                ResultSet.TYPE_FORWARD_ONLY,  // 只能向前移动
                ResultSet.CONCUR_READ_ONLY     // 只读
            );
            
            // 设置fetch direction（MySQL需要这个设置）
            pstmt.setFetchDirection(ResultSet.FETCH_FORWARD);
            pstmt.setFetchSize(Integer.MIN_VALUE);  // 使用流式获取
            
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    // 逐行处理，避免内存溢出
                    processRow(rs);
                }
            }
        }
    }
}
```

### 批量操作优化

```java
public class BatchOptimizer {
    
    // 1. 使用rewriteBatchedStatements参数
    private static final String OPTIMIZED_URL = 
        "jdbc:mysql://localhost:3306/testdb?" +
        "rewriteBatchedStatements=true&" +  // 重要！开启批量重写
        "useServerPrepStmts=true&" +        // 使用服务器端预编译
        "cachePrepStmts=true";              // 缓存预编译语句
    
    // 2. 批量插入优化
    public void optimizedBatchInsert(List<User> users) throws SQLException {
        String sql = "INSERT INTO users(username, email) VALUES(?, ?)";
        
        try (Connection conn = DriverManager.getConnection(OPTIMIZED_URL, "root", "123456");
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            for (User user : users) {
                pstmt.setString(1, user.getUsername());
                pstmt.setString(2, user.getEmail());
                pstmt.addBatch();
            }
            
            // 执行批处理
            int[] results = pstmt.executeBatch();
            conn.commit();
            
            // 清空批处理，避免内存泄漏
            pstmt.clearBatch();
        }
    }
    
    // 3. 分批提交（避免事务过大）
    public void batchInsertWithChunk(List<User> users, int chunkSize) throws SQLException {
        String sql = "INSERT INTO users(username, email) VALUES(?, ?)";
        
        try (Connection conn = getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            int count = 0;
            for (User user : users) {
                pstmt.setString(1, user.getUsername());
                pstmt.setString(2, user.getEmail());
                pstmt.addBatch();
                count++;
                
                // 每chunkSize条提交一次
                if (count % chunkSize == 0) {
                    pstmt.executeBatch();
                    conn.commit();
                    pstmt.clearBatch();
                    System.out.println("已提交 " + count + " 条记录");
                }
            }
            
            // 提交剩余的
            if (count % chunkSize != 0) {
                pstmt.executeBatch();
                conn.commit();
            }
        }
    }
}
```

## 3.2 监控和诊断

### SQL 执行监控

```java
public class SQLMonitor {
    
    // 监控SQL执行时间
    public List<User> executeWithMonitor(String sql, Object... params) throws SQLException {
        long startTime = System.currentTimeMillis();
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            // 设置参数
            for (int i = 0; i < params.length; i++) {
                pstmt.setObject(i + 1, params[i]);
            }
            
            List<User> result = new ArrayList<>();
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    result.add(mapResultSetToUser(rs));
                }
            }
            
            long endTime = System.currentTimeMillis();
            logSQLPerformance(sql, params, endTime - startTime, result.size());
            
            return result;
        }
    }
    
    private void logSQLPerformance(String sql, Object[] params, long duration, int rowCount) {
        System.out.println("SQL执行报告:");
        System.out.println("SQL: " + sql);
        System.out.println("参数: " + Arrays.toString(params));
        System.out.println("耗时: " + duration + "ms");
        System.out.println("返回行数: " + rowCount);
        System.out.println("平均每行耗时: " + (rowCount > 0 ? duration / rowCount : 0) + "ms");
        
        // 可以根据阈值发出警告
        if (duration > 1000) {  // 超过1秒
            System.err.println("警告: SQL执行时间过长！");
        }
    }
    
    // 获取数据库性能指标
    public void printDatabaseMetrics() throws SQLException {
        try (Connection conn = DBUtil.getConnection()) {
            DatabaseMetaData meta = conn.getMetaData();
            
            System.out.println("=== 数据库性能指标 ===");
            System.out.println("数据库: " + meta.getDatabaseProductName());
            System.out.println("版本: " + meta.getDatabaseProductVersion());
            System.out.println("最大连接数: " + meta.getMaxConnections());
            System.out.println("默认事务隔离级别: " + meta.getDefaultTransactionIsolation());
            
            // 执行SHOW STATUS获取MySQL状态
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SHOW STATUS LIKE 'Threads_connected'")) {
                if (rs.next()) {
                    System.out.println("当前连接数: " + rs.getString(2));
                }
            }
            
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SHOW STATUS LIKE 'Queries'")) {
                if (rs.next()) {
                    System.out.println("总查询数: " + rs.getString(2));
                }
            }
        }
    }
}
```

### 慢查询日志

```java
public class SlowQueryLogger {
    
    private static final long SLOW_QUERY_THRESHOLD = 1000;  // 1秒
    
    public <T> T executeWithSlowQueryLog(String sql, SQLExecutor<T> executor) throws SQLException {
        long startTime = System.currentTimeMillis();
        
        T result = executor.execute();
        
        long endTime = System.currentTimeMillis();
        long duration = endTime - startTime;
        
        if (duration > SLOW_QUERY_THRESHOLD) {
            logSlowQuery(sql, duration);
        }
        
        return result;
    }
    
    private void logSlowQuery(String sql, long duration) {
        String log = String.format(
            "[SLOW QUERY] 耗时: %dms, SQL: %s, 时间: %s",
            duration, 
            sql, 
            new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date())
        );
        
        // 写入日志文件
        try (FileWriter writer = new FileWriter("slow_query.log", true)) {
            writer.write(log + "\n");
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 也可以发送到监控系统
        sendToMonitoringSystem(log);
    }
    
    @FunctionalInterface
    public interface SQLExecutor<T> {
        T execute() throws SQLException;
    }
}
```

## 3.3 高级特性

### 可滚动结果集

```java
public class ScrollableResultSetDemo {
    
    public void scrollResultSet() throws SQLException {
        String sql = "SELECT * FROM users ORDER BY id";
        
        try (Connection conn = DBUtil.getConnection();
             Statement stmt = conn.createStatement(
                 ResultSet.TYPE_SCROLL_INSENSITIVE,  // 可滚动
                 ResultSet.CONCUR_READ_ONLY          // 只读
             );
             ResultSet rs = stmt.executeQuery(sql)) {
            
            // 1. 移动到第一行
            rs.first();
            System.out.println("第一行: " + rs.getInt("id"));
            
            // 2. 移动到最后一行
            rs.last();
            System.out.println("最后一行: " + rs.getInt("id"));
            
            // 3. 获取总行数
            int rowCount = rs.getRow();  // 当前是最后一行
            System.out.println("总行数: " + rowCount);
            
            // 4. 移动到第5行
            rs.absolute(5);
            System.out.println("第5行: " + rs.getString("username"));
            
            // 5. 相对移动
            rs.relative(-2);  // 向前移动2行
            System.out.println("第3行: " + rs.getString("username"));
            
            // 6. 判断是否在第一行/最后一行之前
            rs.beforeFirst();
            System.out.println("是否在第一行之前: " + rs.isBeforeFirst());
            
            // 7. 逆序遍历
            rs.afterLast();
            while (rs.previous()) {
                System.out.println("逆序: " + rs.getInt("id"));
            }
        }
    }
    
    // 分页查询（使用可滚动结果集）
    public List<User> paginateWithScroll(int page, int size) throws SQLException {
        String sql = "SELECT * FROM users ORDER BY id";
        
        try (Connection conn = DBUtil.getConnection();
             Statement stmt = conn.createStatement(
                 ResultSet.TYPE_SCROLL_INSENSITIVE,
                 ResultSet.CONCUR_READ_ONLY
             );
             ResultSet rs = stmt.executeQuery(sql)) {
            
            List<User> users = new ArrayList<>();
            
            // 计算起始位置
            int start = (page - 1) * size + 1;
            if (rs.absolute(start)) {
                int count = 0;
                do {
                    users.add(mapResultSetToUser(rs));
                    count++;
                } while (rs.next() && count < size);
            }
            
            return users;
        }
    }
}
```

### 可更新结果集

```java
public class UpdatableResultSetDemo {
    
    public void updateThroughResultSet() throws SQLException {
        String sql = "SELECT id, username, email FROM users WHERE id = ?";
        
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(
                 sql,
                 ResultSet.TYPE_SCROLL_SENSITIVE,  // 敏感（可感知更新）
                 ResultSet.CONCUR_UPDATABLE         // 可更新
             )) {
            
            pstmt.setInt(1, 1);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    // 1. 更新当前行
                    rs.updateString("email", "new_email@test.com");
                    rs.updateRow();  // 提交更新
                    
                    // 2. 插入新行
                    rs.moveToInsertRow();  // 移动到插入行
                    rs.updateString("username", "newuser");
                    rs.updateString("email", "newuser@test.com");
                    rs.insertRow();  // 插入新行
                    
                    // 3. 删除行
                    rs.last();  // 移动到要删除的行
                    rs.deleteRow();
                    
                    // 4. 取消更新
                    rs.updateString("email", "test@test.com");
                    rs.cancelRowUpdates();  // 取消未提交的更新
                }
            }
        }
    }
    
    // 批量更新结果集
    public void batchUpdateResultSet(List<User> updates) throws SQLException {
        String sql = "SELECT * FROM users WHERE is_active = TRUE FOR UPDATE";
        
        try (Connection conn = DBUtil.getConnection();
             Statement stmt = conn.createStatement(
                 ResultSet.TYPE_SCROLL_SENSITIVE,
                 ResultSet.CONCUR_UPDATABLE
             );
             ResultSet rs = stmt.executeQuery(sql)) {
            
            conn.setAutoCommit(false);
            
            for (User update : updates) {
                // 移动到对应的行（假设id唯一）
                while (rs.next()) {
                    if (rs.getInt("id") == update.getId()) {
                        rs.updateString("email", update.getEmail());
                        rs.updateInt("age", update.getAge());
                        rs.updateRow();
                        break;
                    }
                }
                // 回到结果集开始位置，用于下一个更新
                rs.beforeFirst();
            }
            
            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        }
    }
}
```

## 3.4 连接泄露检测

```java
public class ConnectionLeakDetector {
    
    private static final Map<Connection, StackTraceElement[]> connectionMap = 
        new WeakHashMap<>();
    private static final ScheduledExecutorService scheduler = 
        Executors.newScheduledThreadPool(1);
    
    static {
        // 每隔5分钟检查一次连接泄露
        scheduler.scheduleAtFixedRate(() -> {
            checkForLeaks();
        }, 5, 5, TimeUnit.MINUTES);
    }
    
    // 包装Connection，添加追踪信息
    public static Connection wrapConnection(Connection conn) {
        // 记录创建连接的堆栈信息
        StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
        connectionMap.put(conn, stackTrace);
        
        return new ConnectionWrapper(conn);
    }
    
    // 连接包装器
    private static class ConnectionWrapper implements Connection {
        private final Connection delegate;
        private volatile long lastUsedTime;
        
        public ConnectionWrapper(Connection delegate) {
            this.delegate = delegate;
            this.lastUsedTime = System.currentTimeMillis();
        }
        
        @Override
        public Statement createStatement() throws SQLException {
            updateLastUsedTime();
            return delegate.createStatement();
        }
        
        @Override
        public PreparedStatement prepareStatement(String sql) throws SQLException {
            updateLastUsedTime();
            return delegate.prepareStatement(sql);
        }
        
        @Override
        public void close() throws SQLException {
            connectionMap.remove(delegate);
            delegate.close();
        }
        
        private void updateLastUsedTime() {
            this.lastUsedTime = System.currentTimeMillis();
        }
        
        public long getLastUsedTime() {
            return lastUsedTime;
        }
        
        // 其他方法委托给原始的Connection...
        @Override
        public CallableStatement prepareCall(String sql) throws SQLException {
            return delegate.prepareCall(sql);
        }
        
        @Override
        public String nativeSQL(String sql) throws SQLException {
            return delegate.nativeSQL(sql);
        }
        
        @Override
        public void setAutoCommit(boolean autoCommit) throws SQLException {
            delegate.setAutoCommit(autoCommit);
        }
        
        @Override
        public boolean getAutoCommit() throws SQLException {
            return delegate.getAutoCommit();
        }
        
        @Override
        public void commit() throws SQLException {
            delegate.commit();
        }
        
        @Override
        public void rollback() throws SQLException {
            delegate.rollback();
        }
        
        // ... 实现所有Connection接口方法
    }
    
    // 检查连接泄露
    private static void checkForLeaks() {
        long currentTime = System.currentTimeMillis();
        long leakThreshold = 5 * 60 * 1000;  // 5分钟
        
        for (Map.Entry<Connection, StackTraceElement[]> entry : connectionMap.entrySet()) {
            ConnectionWrapper wrapper = (ConnectionWrapper) entry.getKey();
            long idleTime = currentTime - wrapper.getLastUsedTime();
            
            if (idleTime > leakThreshold) {
                System.err.println("发现可能的连接泄露:");
                System.err.println("连接空闲时间: " + (idleTime / 1000) + "秒");
                System.err.println("创建堆栈:");
                
                StackTraceElement[] stackTrace = entry.getValue();
                for (StackTraceElement element : stackTrace) {
                    System.err.println("  " + element);
                }
                
                // 尝试自动关闭泄露的连接
                try {
                    if (!wrapper.isClosed()) {
                        wrapper.close();
                        System.err.println("已自动关闭泄露的连接");
                    }
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## 3.5 最佳实践总结

### 配置文件示例

**db.properties:**
```properties
# 数据库配置
db.url=jdbc:mysql://localhost:3306/testdb?useSSL=false&serverTimezone=Asia/Shanghai&rewriteBatchedStatements=true
db.username=root
db.password=123456

# 连接池配置
db.pool.maxSize=20
db.pool.minIdle=5
db.pool.connectionTimeout=30000
db.pool.idleTimeout=600000

# 性能配置
db.fetchSize=1000
db.queryTimeout=30
```

### 终极工具类

```java
import java.io.InputStream;
import java.sql.*;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * JDBC终极工具类
 * 包含连接池、监控、性能优化等功能
 */
public final class UltimateDBUtil {
    
    private static volatile DataSource dataSource;
    private static final Properties config = new Properties();
    private static final Map<String, PreparedStatement> statementCache = 
        new ConcurrentHashMap<>();
    
    static {
        try {
            // 加载配置
            InputStream is = UltimateDBUtil.class.getClassLoader()
                .getResourceAsStream("db.properties");
            config.load(is);
            
            // 初始化连接池
            initDataSource();
            
            // 注册关闭钩子
            Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                closeDataSource();
                clearStatementCache();
            }));
            
        } catch (Exception e) {
            throw new RuntimeException("数据库工具类初始化失败", e);
        }
    }
    
    private static void initDataSource() {
        HikariConfig hikariConfig = new HikariConfig();
        hikariConfig.setJdbcUrl(config.getProperty("db.url"));
        hikariConfig.setUsername(config.getProperty("db.username"));
        hikariConfig.setPassword(config.getProperty("db.password"));
        
        hikariConfig.setMaximumPoolSize(
            Integer.parseInt(config.getProperty("db.pool.maxSize", "20")));
        hikariConfig.setMinimumIdle(
            Integer.parseInt(config.getProperty("db.pool.minIdle", "5")));
        hikariConfig.setConnectionTimeout(
            Long.parseLong(config.getProperty("db.pool.connectionTimeout", "30000")));
        
        // 优化配置
        hikariConfig.addDataSourceProperty("cachePrepStmts", "true");
        hikariConfig.addDataSourceProperty("prepStmtCacheSize", "250");
        hikariConfig.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        hikariConfig.addDataSourceProperty("useServerPrepStmts", "true");
        
        dataSource = new HikariDataSource(hikariConfig);
    }
    
    // 获取带监控的连接
    public static Connection getConnection() throws SQLException {
        long startTime = System.currentTimeMillis();
        Connection conn = dataSource.getConnection();
        long endTime = System.currentTimeMillis();
        
        // 记录获取连接的时间
        if (endTime - startTime > 100) {  // 超过100ms发出警告
            System.err.println("获取数据库连接耗时: " + (endTime - startTime) + "ms");
        }
        
        return ConnectionLeakDetector.wrapConnection(conn);
    }
    
    // 带缓存的PreparedStatement
    public static PreparedStatement getPreparedStatement(String sql) throws SQLException {
        return statementCache.computeIfAbsent(sql, k -> {
            try {
                Connection conn = getConnection();
                return conn.prepareStatement(sql);
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        });
    }
    
    // 安全的查询方法
    public static <T> List<T> query(String sql, ResultSetMapper<T> mapper, Object... params) 
            throws SQLException {
        long startTime = System.currentTimeMillis();
        
        try (Connection conn = getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            // 设置参数
            for (int i = 0; i < params.length; i++) {
                pstmt.setObject(i + 1, params[i]);
            }
            
            // 设置查询优化参数
            pstmt.setFetchSize(Integer.parseInt(config.getProperty("db.fetchSize", "1000")));
            pstmt.setQueryTimeout(Integer.parseInt(config.getProperty("db.queryTimeout", "30")));
            
            List<T> results = new ArrayList<>();
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    results.add(mapper.map(rs));
                }
            }
            
            long endTime = System.currentTimeMillis();
            monitorQuery(sql, params, endTime - startTime, results.size());
            
            return results;
        }
    }
    
    // 事务模板方法
    public static <T> T executeInTransaction(TransactionCallback<T> callback) throws SQLException {
        Connection conn = null;
        try {
            conn = getConnection();
            conn.setAutoCommit(false);
            
            T result = callback.doInTransaction(conn);
            conn.commit();
            
            return result;
        } catch (SQLException e) {
            if (conn != null) {
                conn.rollback();
            }
            throw e;
        } finally {
            if (conn != null) {
                conn.setAutoCommit(true);
                conn.close();
            }
        }
    }
    
    // 批量操作模板
    public static int[] batchUpdate(String sql, List<Object[]> batchParams) throws SQLException {
        try (Connection conn = getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            for (Object[] params : batchParams) {
                for (int i = 0; i < params.length; i++) {
                    pstmt.setObject(i + 1, params[i]);
                }
                pstmt.addBatch();
            }
            
            int[] results = pstmt.executeBatch();
            conn.commit();
            
            return results;
        }
    }
    
    // 监控方法
    private static void monitorQuery(String sql, Object[] params, long duration, int rowCount) {
        if (duration > 1000) {  // 慢查询
            String log = String.format(
                "慢查询: %dms, SQL: %s, 参数: %s, 行数: %d",
                duration, sql, Arrays.toString(params), rowCount
            );
            
            // 记录到日志文件
            try (FileWriter writer = new FileWriter("slow_queries.log", true)) {
                writer.write(new Date() + " - " + log + "\n");
            } catch (IOException e) {
                e.printStackTrace();
            }
            
            // 发送到监控系统（可选）
            // sendToMonitoringSystem(log);
        }
    }
    
    // 清理资源
    private static void closeDataSource() {
        if (dataSource != null && !dataSource.isClosed()) {
            dataSource.close();
        }
    }
    
    private static void clearStatementCache() {
        for (PreparedStatement stmt : statementCache.values()) {
            try {
                if (!stmt.isClosed()) {
                    stmt.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        statementCache.clear();
    }
    
    // 回调接口
    @FunctionalInterface
    public interface ResultSetMapper<T> {
        T map(ResultSet rs) throws SQLException;
    }
    
    @FunctionalInterface
    public interface TransactionCallback<T> {
        T doInTransaction(Connection conn) throws SQLException;
    }
    
    // 私有构造方法，防止实例化
    private UltimateDBUtil() {
        throw new AssertionError("不能实例化工具类");
    }
}
```

## 🎯 **学习路线总结**

### 基础阶段掌握：
1. JDBC 驱动加载和连接建立
2. Statement 基本 CRUD 操作
3. ResultSet 结果集处理
4. 基本的异常处理

### 进阶阶段掌握：
1. PreparedStatement 防注入和预编译
2. 事务管理和隔离级别
3. 批处理操作优化
4. 连接池技术（HikariCP/Druid）

### 高级阶段掌握：
1. 性能监控和调优
2. 大数据量处理技巧
3. 连接泄露检测和预防
4. 生产环境最佳实践

### 生产环境建议：
1. **始终使用连接池**
2. **始终使用 PreparedStatement**
3. **合理设置事务边界**
4. **监控慢查询和连接泄露**
5. **使用配置文件管理数据库参数**
6. **定期进行性能测试和优化**

通过这三个阶段的学习，你将能够编写出高效、安全、可维护的 JDBC 代码！

