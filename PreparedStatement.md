# 使用PreparedStatement



## Statement与SQL注入

Statement对象可以执行一个字符串，字符串的内容是SQL语句。遗憾的是，当需要根据用户输入构建SQL语句时，这个包含SQL语句的字符串需要通过字符串拼接得到。这个漏洞可以被SQL注入利用。**SQL注入技术可以被恶意黑客用来窃取数据或损坏数据库。**

假设一个Java程序输入一个字符串name，并构建下面的查询：

```java
String sql = "select * from customers where name = '" + name + "'";
statement.excuteQuery(sql);
```

如果用户没有输入一个名字，而是输入：

```
X' or 'Y' = 'Y
```

这样，产生的SQL语句就变成

```sql
select * from customers where name = 'X' or 'Y' = 'Y'
```

这个查询中，where子句总为真，所以查询结果返回整个customers关系。

## 利用PreparedStatement来执行SQL语句

我们可以通过以?来代表以后再给出的实际值，而创建一个PreparedStatement（预编译语句）的实例化对象。我们称?为占位符。

```java
            String sql = "insert into customers(name, email, birth) values(?,?,?)";
            preparedStatement = connection.prepareStatement(sql);
```

在准备通过PreparedStatement对象执行SQL语句之前，我们必须为?参数设定具体的值。例如：

```java
 			preparedStatement.setString(1, "Naruto");
            preparedStatement.setString(2, "Naruto@gmail.com");
            //注意格式大小写
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
            java.util.Date parse = simpleDateFormat.parse("1997-09-01");
            //注意区别java.util.Date和java.sql.Date
            preparedStatement.setDate(3, new java.sql.Date(parse.getTime()));
```

最后调用excute()来执行SQL语句

```java
			preparedStatement.execute();
```

Tips：可以通过PreparedStatement.setObject(int parameterIndex, Object x)方法来批量为?参数设置具体的值，以进一步复用代码，例如

```java
    //通用的增删改操作
    public void update(String sql, Object ...args){
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {
            //1.获取数据库连接
            connection = JDBCUtils.getConnection();
            //2.预编译sql语句，返回PreparedStatement实例
            preparedStatement = connection.prepareStatement(sql);
            //3.填充占位符
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            //4.执行
            preparedStatement.execute();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            //5.关闭资源
            JDBCUtils.closeResource(connection, preparedStatement);
        }
    }
```

