```java
package com.czf.blob;

import com.czf.util.JDBCUtils;
import org.junit.Test;

import java.sql.Connection;
import java.sql.PreparedStatement;

/**
 * 使用PreparedStatement实现批量数据的操作
 *
 * update、delete本身具有批量操作的效果
 * 此时的批量操作主要指的是批量插入。使用PreparedStatement如何实现更高效的批量操作？
 *
 * 题目：向goods表中插入两万条数据
 * 方式一：使用Statement
 */
public class InsertTest {
    //批量插入的方式二：使用PreparedStatement
    @Test
    public void testInsert1() {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {

            long start = System.currentTimeMillis();

            connection = JDBCUtils.getConnection();
            String sql = "insert into goods(name) values(?)";
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 1; i <= 20000; i++) {
                preparedStatement.setObject(1, "name_" + i);
                preparedStatement.execute();
            }
            long end = System.currentTimeMillis();
            System.out.println("花费的时间为：" + (end - start));//20000:49918
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(connection, preparedStatement,null);
        }
    }

    /**
     * 批量插入的方式三：
     * 1.addBatch()、executeBatch()、clearBatch()
     * 2.mysql服务器默认是关闭批处理的，我们需要通过一个参数，让mysql开启批处理的支持
     *              ?rewriteBatchedStatements=true  写在配置文件url后面
     */
    @Test
    public void testInsert2() {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {

            long start = System.currentTimeMillis();

            connection = JDBCUtils.getConnection();
            String sql = "insert into goods(name) values(?)";
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 1; i <= 1000000; i++) {
                preparedStatement.setObject(1, "name_" + i);

                //1.“攒”sql
                preparedStatement.addBatch();
                if (i % 500 == 0){
                    //2.执行
                    preparedStatement.executeBatch();//注意是executeBatch()，不是execute()否则就只插入了i/500条数据

                    //3.清空
                    preparedStatement.clearBatch();
                }
            }
            long end = System.currentTimeMillis();
            System.out.println("花费的时间为：" + (end - start));//20000:49918 -- 1153   1000000：17204
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(connection, preparedStatement,null);
        }
    }

    /**
     * 批量插入的方式四：在方式三基础上设置连接不允许自动提交数据
     */
    @Test
    public void testInsert3() {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {

            long start = System.currentTimeMillis();

            connection = JDBCUtils.getConnection();

            //设置不允许自动提交数据
            connection.setAutoCommit(false);
            String sql = "insert into goods(name) values(?)";
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 1; i <= 1000000; i++) {
                preparedStatement.setObject(1, "name_" + i);

                //1.“攒”sql
                preparedStatement.addBatch();
                if (i % 500 == 0){
                    //2.执行
                    preparedStatement.executeBatch();

                    //3.清空
                    preparedStatement.clearBatch();
                }
            }
            //提交数据
            connection.commit();
            long end = System.currentTimeMillis();
            System.out.println("花费的时间为：" + (end - start));//20000:49918 -- 1153 1000000：17204 -- 8511
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(connection, preparedStatement,null);
        }
    }
}

```

