# 插入和查询Blob类型数据

> SQL提供字符数据的大对象数据类型(clob)和二进制数据的大对象类型(blob)。在这些数据类型中字符“lob”代表“Large Object”。	——《数据库系统概念》



```java
package com.czf.blob;

import com.czf.bean.Customer;
import com.czf.util.JDBCUtils;
import org.junit.Test;

import java.io.*;
import java.sql.*;
import java.text.ParseException;
import java.text.SimpleDateFormat;

/**
 * 测试使用PreparedStatement操作Blob类型的数据
 */
public class BlobTest {
    //向数据表customers中插入Blob类型的字段
    @Test
    public void testInsert() throws SQLException, IOException, ClassNotFoundException, ParseException {
        Connection connection = JDBCUtils.getConnection();
        String sql = "insert into customers(name, birth, photo) values(?, ?, ?)";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        FileInputStream inputStream = new FileInputStream(new File("C:\\Users\\18751\\Pictures\\Saved Pictures\\志摩凛.jpg"));
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date date = new Date(simpleDateFormat.parse("2000-10-01").getTime());
        preparedStatement.setObject(3, inputStream);
        preparedStatement.setObject(2, date);
        preparedStatement.setObject(1, "Shima Rin");
        preparedStatement.execute();
        JDBCUtils.closeResource(connection, preparedStatement, null);
    }

    //查询数据表customers中Blob类型的字段
    @Test
    public void testQuery(){
        InputStream binaryStream = null;
        FileOutputStream fileOutputStream = null;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JDBCUtils.getConnection();
            String sql = "select id, name, email, birth, photo from customers where id = ?";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setObject(1, 20);
            resultSet = preparedStatement.executeQuery();
            while(resultSet.next()){
    //            方式一：
    //            int id = resultSet.getInt(1);
    //            String name = resultSet.getString(2);
    //            String email = resultSet.getString(3);
    //            Date birth = resultSet.getDate(4);
                //方式二：
                int id = resultSet.getInt("id");
                String name = resultSet.getString("name");
                String email = resultSet.getString("email");
                Date birth = resultSet.getDate("birth");

                Customer customer = new Customer(name, email,birth);
                System.out.println(customer);

                if (customer.getName().equals("Shima Rin")) {
                    //将Blob类型字段下载下来，以文件方式保存在本地
                    Blob photo = resultSet.getBlob("photo");
                    binaryStream = photo.getBinaryStream();
                    fileOutputStream = new FileOutputStream("C:\\Users\\18751\\Desktop\\Shima Rin.jpg");
                    byte[] buffer = new byte[1024];
                    int len;
                    while((len = binaryStream.read(buffer)) != -1){
                        fileOutputStream.write(buffer, 0, len);
                    }
                }
            }
            binaryStream.close();
            fileOutputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(connection, preparedStatement, resultSet);
        }
    }
}

```

testQuery控制台输出：

```latex
Customer{name='Shima Rin', email='null', birth=2000-10-01}
```

输出流成功输出到了桌面。

