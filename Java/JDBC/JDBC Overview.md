### JDBC Tutorial - Java JDBC Overview

This JDBC tutorial gives an overview of JDBC which is the Java API for developing Java applications that access relational databases.

### Why use JDBC

JDBC provides developers with a uniform [interface](https://www.netjstech.com/2015/05/interface-in-java.html) to connect with different relational databases like Oracle, MySQL, DB2, Access etc.

JDBC provides a set of interfaces and classes that standardize the interaction with different databases and abstracts you as a developer with the inner working of the proprietary databases. You just need to know the standard JDBC steps to connect to database, query it in order to fetch results or update DB. **Note here** that the *SQL may differ* according to the DB used.

### JDBC Drivers

In Order to connect to database JDBC uses **JDBC drivers**. Since JDBC driver acts as a connector between JDBC and proprietary databases JDBC drivers are DB specific and generally provided by the DB vendor itself.

**As example**– In order to connect to MySql DB you will need a MySql JDBC connector driver which is bundled in the mysql-connector-javaXXX.jar.

- Refer [Types of JDBC Drivers](https://www.netjstech.com/2017/12/types-of-jdbc-drivers.html) to know more about the types of driver available.

The interaction of JDBC with the database using JDBC driver can be pictorially represented as follows -

[![JDBC Drivers](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:29:04-JDBC%252Bdriver.png)](https://3.bp.blogspot.com/-8dfZNGsyhsU/WipVLevZ2cI/AAAAAAAAAdE/CBN_nYHU4RMhowz3t_yqNsW2d1_lCPr3wCPcBGAYYCw/s1600/JDBC%2Bdriver.png)

### Packages in JDBC API

The JDBC API is comprised of two packages:

- **java.sql**- Referred to as the JDBC core API
- **javax.sql**- Referred to as the JDBC Optional Package API

You automatically get both packages when you download the Java Platform Standard Edition (Java SE).

### Changes in JDBC 4.x

The current version of JDBC which comes bundled with Java 8 is JDBC 4.2. There are some noticeable changes in the 4.x versions like-

1. Addition of the java.sql.SQLType Interface
2. Addition of the java.sql.JDBCType Enum– Using SQLType interface and JDBCType [Enum](https://www.netjstech.com/2017/03/enum-type-in-java.html) you can identify the generic SQL types like CLOB, REF_CURSOR, TINYINT, VARCHAR etc.
3. You can use try-with-resources statement to automatically close resources of type Connection, ResultSet, and Statement.
4. Automatic loading of JDBC drivers on the class path.

### Steps for connecting to DB using JDBC

JDBC API provides a set of interfaces and classes for connecting to DB, creating SQL statement, executing created SQL statement in database, returning the results and processing that ResultSet.

These steps can be summarized as follows-

- Loading driver
- Creating connection to DB
- Creating Statement
- Executing Query
- Processing ResultSet
- Closing connection

**Refer** [Java JDBC Steps to Connect to DB](https://www.netjstech.com/2017/12/java-jdbc-steps-to-connect-to-db.html) to see these steps in details and a JDBC example Java program.

A pictorial representation of these steps can be represented as follows.

[![JDBC DB connection steps](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:29:04-JDBC%252BSteps.png)](https://4.bp.blogspot.com/-gz21NyuRma4/WipVstfUbDI/AAAAAAAAAdI/Kq7VwI7CfJ035NypuIKh_MtlkO6lMd_WACPcBGAYYCw/s1600/JDBC%2BSteps.png)

That's all for this topic **JDBC Tutorial - Java JDBC Overview**. If you have any doubt or any suggestions to make please drop a comment. Thanks!