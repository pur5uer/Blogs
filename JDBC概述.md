# JDBC概述

> JDBC是sun公司提供一套用于数据库操作的接口，java程序员只需要面向这套接口编程即可。不同的数据库厂商，需要针对这套接口，提供不同的实现。不同的实现的集合，即为不同的数据库驱动。 ——《面向接口编程》

JDBC接口包括两个层次

- 面向应用的API：Java API，抽象接口，供应用开发人员使用。
- 面向数据库的API：Java Driver API，供开发商开发数据库驱动用。

## JDBC程序编写

1. 导入java.sql.*
2. 加载并注册驱动程序
3. 创建Connection对象
4. 创建Statement对象
5. 执行SQL语句
   - 查询
     - Statement对象执行SQL查询语句，返回ResultSet对象
     - 使用ResultSet对象
     - 关闭ResultSet对象
   - 更新
     - Statement对象执行SQL更新语句
6. 关闭Statement对象
7. 关闭Connection对象