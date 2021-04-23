# JDBC获取数据库连接

## 概述

1. 提供数据库连接的基本信息
   - url
   - user
   - password
2. 加载并注册驱动
3. 获取连接



## 具体步骤

1. 在src目录下创建properties文件
2. 编辑properties文件，存储数据库连接的配置信息，例如：

```properties
#数据库连接的配置信息
user=root
password=123123
url=jdbc:mysql://localhost:3306/jdbc_learning
driverClass=com.mysql.cj.jdbc.Driver
```

3. 读取配置文件中的基本信息，保存到字符串中，例如：

```java
		//读取配置文件中的基本信息
        InputStream resourceAsStream = ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
        Properties properties = new Properties();
        properties.load(resourceAsStream);
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        String url = properties.getProperty("url");
        String driverClass = properties.getProperty("driverClass");
```

4. 加载并注册驱动

```java
		//加载并注册驱动
        Class.forName(driverClass);
/**
         * 省略注册操作的原因
         * 在Driver.class中有如下静态代码块
         * static {
         *         try {
         *             DriverManager.registerDriver(new Driver());
         *         } catch (SQLException var1) {
         *             throw new RuntimeException("Can't register driver!");
         *         }
         *     }
         */
```

5. 获取连接

```java
        //获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println(connection);
```

### 使用properties配置文件的好处

1. 实现了数据和代码的分离。实现了解耦。
2. 修改配置文件信息，避免了程序重新打包。

## 完整代码

代码中使用了Junit进行测试

```java
package com.czf.connection;
import org.junit.Test;
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;
@Test
    public void testConnection5() throws IOException, ClassNotFoundException, SQLException {
        //1.读取配置文件中的基本信息
        InputStream resourceAsStream = ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
        Properties properties = new Properties();
        properties.load(resourceAsStream);
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        String url = properties.getProperty("url");
        String driverClass = properties.getProperty("driverClass");
        //2.加载驱动
        Class.forName(driverClass);
        //3.获取连接
        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println(connection);
    }
```



## 注意

connection需要关闭，否则会引起资源的极大浪费或者内存泄漏，这里只是获得连接，所以没有关闭。