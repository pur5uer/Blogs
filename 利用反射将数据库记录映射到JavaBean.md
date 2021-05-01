# 利用反射将数据库记录映射到JavaBean

## 对象-关系映射ORM(Object Relational Mapping)

> 对象-关系映射系统的首要目标是通过向程序员提供对象模型来减轻他们建造应用的工作，同时保持了在底层使用一个健壮的关系数据库所带来的好处。另一个额外的好处是，在操纵缓存于内存中的对象时，相比于直接访问下层数据库，对象-关系系统可以带来巨大的性能提升。
>
> ​																											——《数据库系统概念》

### 概述

对象-关系映射系统建立于传统关系数据库之上。它允许程序员定义数据库关系的元组与程序设计语言中对象之间的映射。不同于持久化程序设计语言，对象是瞬态的，并且没有持久的对象标识。

- 一个数据表对应一个Java类
- 表中的一条记录对应Java类的一个对象
- 表中的一个字段对应Java类的一个属性

### 缺点

对象-关系映射系统可能会面临大规模数据更新所带来的巨大的开销，而且可能仅提供有限的查询能力。不过对于大多数应用，对象-关系映射系统利大于弊。对象-关系映射系统已经被广泛采用，如Hibernate框架。



## 一个例子

Bean：

```java
package com.czf.bean;

import java.sql.Date;

public class Order {
    private int orderId;
    private String orderName;
    private Date orderDate;

    public Order(int orderId, String orderName, Date orderDate) {
        this.orderId = orderId;
        this.orderName = orderName;
        this.orderDate = orderDate;
    }

    public Order() {
    }

    public int getOrderId() {
        return orderId;
    }

    public void setOrderId(int orderId) {
        this.orderId = orderId;
    }

    public String getOrderName() {
        return orderName;
    }

    public void setOrderName(String orderName) {
        this.orderName = orderName;
    }

    public Date getOrderDate() {
        return orderDate;
    }

    public void setOrderDate(Date orderDate) {
        this.orderDate = orderDate;
    }

    @Override
    public String toString() {
        return "Order{" +
                "orderId=" + orderId +
                ", orderName='" + orderName + '\'' +
                ", orderDate=" + orderDate +
                '}';
    }
}

```

针对Order的查询：

```java
package com.czf.preparedstatement.crud;

import com.czf.bean.Order;
import com.czf.util.JDBCUtils;
import org.junit.Test;

import java.lang.reflect.Field;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class OrderForQuery {
    @Test
    public void orderForQueryTest(){
        String sql = "select order_id orderId, order_name orderName, order_date orderDate from `order`";
        List<Order> orders = orderForQuery(sql, null);
        for (Order order : orders) {
            System.out.println(order);
        }
    }
    /**
     * order表的通用查询
     * 查询order表并返回一个Order对象列表（ORM）
     * 针对表的字段名与类的属性名不相同的情况：
     * 1.声明sql时，必须使用类的属性名来命名字段的别名
     * 2.使用ResultSetMetaData时，需要使用getColumnLabel()来替换getColumnName()，获取列的别名
     * 说明：如果sql中没有给字段别名，getColumnLabel()获取的就是列名，所以建议使用getColumnLabel()
     * @param sql
     * @param args
     * @return  Order对象列表
     */
    public List<Order> orderForQuery(String sql, Object ...args){
        List<Order> orders = new ArrayList<Order>();
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            //获得连接
            connection = JDBCUtils.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            //填充占位符
            if (args != null) {
                for (int i = 0; i < args.length; i++) {
                    preparedStatement.setObject(i + 1, args[i]);
                }
            }
            //执行查询返回结果集
            resultSet = preparedStatement.executeQuery();
            //得到元数据
            ResultSetMetaData metaData = resultSet.getMetaData();
            //得到结果集列数
            int columnCount = metaData.getColumnCount();
            while(resultSet.next()){
                Order order = new Order();
                for (int i = 0; i < columnCount; i++) {
                    //反射得到Field
                    Field field = order.getClass().getDeclaredField(metaData.getColumnLabel(i + 1));
                    Object columnValue = resultSet.getObject(i + 1);
                    field.setAccessible(true);
                    field.set(order, columnValue);
                }
                orders.add(order);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //关闭连接
            JDBCUtils.closeResource(connection, preparedStatement, resultSet);
            return orders;
        }
    }
}

```

控制台输出：

```
Order{orderId=1, orderName='AA', orderDate=2010-03-04}
Order{orderId=2, orderName='DD', orderDate=2000-02-01}
Order{orderId=4, orderName='GG', orderDate=1994-06-28}
```

此代码只适用于Order类，可编写所有JavaBean的通用映射。

## 查询数据库并映射到JavaBean（通用）

```java
package com.czf.preparedstatement.crud;

import com.czf.util.JDBCUtils;
import org.junit.Test;

import java.lang.reflect.Field;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ClassForQuery {

    @Test
    public void testClassForQuery() throws ClassNotFoundException {
        Class customerClass = Class.forName("com.czf.bean.Customer");
        Class orderClass = Class.forName("com.czf.bean.Order");
        String sql0 = "select name, email, birth from customers where email like ?";
        String sq11 = "select order_id orderId, order_name orderName, order_date orderDate from `order`";
        List<Object> objects = classForQuery(customerClass, sql0, "%126%");
        for (Object object : objects) {
            System.out.println(object);
        }
        System.out.println("-----------------------------------------------------------");
        objects = classForQuery(orderClass, sq11, null);
        for (Object object : objects) {
            System.out.println(object);
        }
    }
    /**
     * 查询表并映射到JavaBean（通用）
     * @param clazz 要映射的类
     * @param sql   查询语句
     * @param args  填充占位符的可变参数
     * @return 一个ArrayList<Object>对象
     */
    public List<Object> classForQuery(Class clazz, String sql, Object ...args){
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        List<Object> classList = new ArrayList<Object>();
        try {
            connection = JDBCUtils.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            if (args != null) {
                for (int i = 0; i < args.length; i++) {
                    preparedStatement.setObject(i + 1, args[i]);
                }
            }
            resultSet = preparedStatement.executeQuery();
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            while(resultSet.next()){
                //实例化clazz
                Object o = clazz.getConstructor().newInstance();
                //填充field
                for (int i = 0; i < columnCount; i++) {
                    Object value = resultSet.getObject(i + 1);
                    Field field = clazz.getDeclaredField(metaData.getColumnLabel(i + 1));
                    field.setAccessible(true);
                    field.set(o, value);
                }
                //添加进列表
                classList.add(o);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            JDBCUtils.closeResource(connection, preparedStatement, resultSet);
            return classList;
        }
    }
}

```

控制台输出：

```latex
Customer{name='汪峰', email='wf@126.com', birth=2010-02-02}
Customer{name='陈道明', email='bdf@126.com', birth=2014-01-17}
Customer{name='黎明', email='LiM@126.com', birth=1998-09-08}
Customer{name='张学友', email='zhangxy@126.com', birth=1998-12-21}
Customer{name='朱茵', email='zhuyin@126.com', birth=2014-01-16}
Customer{name='莫扎特', email='beidf@126.com', birth=2014-01-17}
-----------------------------------------------------------
Order{orderId=1, orderName='AA', orderDate=2010-03-04}
Order{orderId=2, orderName='DD', orderDate=2000-02-01}
Order{orderId=4, orderName='GG', orderDate=1994-06-28}

Process finished with exit code 0

```

