### DataSource in Java-JDBC

In the examples given in the previous post [Java JDBC Steps to Connect to DB](https://www.netjstech.com/2017/12/java-jdbc-steps-to-connect-to-db.html) we have seen how to get a connection using **DriverManager** class. That’s ok for sample code where you just need to test using a connection and close it. But in a real life application creating connection object every time DB interaction is needed will be very time consuming. What you need is a connection pool where a given number of connection objects are created in the beginning itself and are reused.

In this post we’ll see another way of connecting to DB from your Java application using a DataSource object which provides the connection pooling. There are other advantages of using DataSource in JDBC too.

### Advantages of using DataSource

1. Layer of abstraction

   – In an enterprise application you can configure JDBC DataSource in the application server and register it with JNDI. That way user just need to know the bound logical name for the DataSource and the DB specific details are hidden.

   

2. Connection pooling

   \- If the pool has a connection that can satisfy the request, it returns the connection to the application. If all connections with in the pool are in use, a new connection is created and returned to the application. The application uses the connection to perform some work on the database and then returns the object back to the pool. The connection is then available for the next connection request.

   By pooling the connection you can reuse the connection objects rather then creating it every time that improves performance for database-intensive applications because creating connection objects is costly both in terms of time and resources.

3. **Distributed transaction**– DataSource implementations can also provide a connection object that can be used in distributed transactions. A distributed transaction is a transaction that accesses two or more DBMS servers.

### DataSource interface in Java

JDBC API provides an [interface](https://www.netjstech.com/2015/05/interface-in-java.html) DataSource that has to be implemented by vendor specific DataSource implementations. DataSource interface is part of **javax.sql** package and it has two overridden methods.

- getConnection()
- getConnection(String username, String password)

Both of these methods return a [Connection](https://www.netjstech.com/2017/12/connection-interface-in-java-jdbc.html) object.

### DataSource Implementations

A JDBC driver should include at least a basic DataSource implementation. For example MySQL DB JDBC driver includes the implementation **com.mysql.jdbc.jdbc2.optional.MysqlDataSource** and Oracle DB’s implementation is **oracle.jdbc.pool.OracleDataSource**.

### JDBC DataSource Examples

Let’s see some examples of DataSource in use. First let us see one example with MySQL DataSource. Though people prefer to use one of the connection pooling library **Apache DBCP** or **mchange c3p0** atleast with stand alone Java programs, we’ll see example of these too.

**Table of contents**

1. [MySQL DataSource Example](https://www.netjstech.com/2017/12/datasource-in-java-jdbc.html#MySQLDSExp)
2. [Apache DBCP DataSource Example](https://www.netjstech.com/2017/12/datasource-in-java-jdbc.html#DBCPDSExp)
3. [c3p0 DBCP DataSource Example](https://www.netjstech.com/2017/12/datasource-in-java-jdbc.html#c3p0DSExp)



### MySQL DataSource Example

Here we have a DataSource class that is a singleton class giving the instance of MysqlDataSource. Also the schema in MySQL is **netjs** and table is **Employee**. You should have the **mysql-connector** jar in your class path.

There is another class DSConnection where we get the instance of MysqlDataSource and use it to get the Connection object.

Then a [PreparedStatement](https://www.netjstech.com/2017/12/preparedstatement-interface-in-java-jdbc.html) object is created with a Select SQL statement to get Employee data having the passed ID.

**DataSource.java**

```
import com.mysql.jdbc.jdbc2.optional.MysqlDataSource;

public class DataSource {
  private static final String DRIVER_CLASS = "com.mysql.jdbc.Driver";
  private static final String DB_CONNECTION_URL = "jdbc:mysql://localhost:3306/netjs";
  private static final String DB_USER = "root";
  private static final String DB_PWD = "admin";
  private static DataSource ds;
  private MysqlDataSource mySqlDS = new MysqlDataSource();

  //private constructor
  private DataSource(){
    //mySqlDS.setDriverClassName(DRIVER_CLASS);
    mySqlDS.setUrl(DB_CONNECTION_URL);
    mySqlDS.setUser(DB_USER);
    mySqlDS.setPassword(DB_PWD);
  }
 
 /**
   *static method for getting instance.
 */
 public static DataSource getInstance(){
     if(ds == null){
        ds = new DataSource();
     }
     return ds;
 }

 public MysqlDataSource getMySqlDS() {
    return mySqlDS;
 }

 public void setMySqlDS(MysqlDataSource mySqlDS) {
    this.mySqlDS = mySqlDS;
 }
}
```

**DSConnection.java**

```
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import com.mysql.jdbc.jdbc2.optional.MysqlDataSource;

public class DSConnection {

 public static void main(String[] args) {
  DSConnection dsCon = new DSConnection();
  try {
   dsCon.displayEmployee(6);
  } catch (SQLException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
 
 /** 
  * @param connection
  * @param id
  * @throws SQLException
  */
 private void displayEmployee(int id) throws SQLException{
  
  Connection connection = null; 
  String selectSQL = "Select * from employee where id = ?";
  PreparedStatement prepStmt = null;
  try {
   MysqlDataSource basicDS = DataSource.getInstance().getMySqlDS();
   connection = basicDS.getConnection();
   prepStmt = connection.prepareStatement(selectSQL);
   prepStmt.setInt(1, id);
   ResultSet rs = prepStmt.executeQuery();
   while(rs.next()){
     System.out.println("id : " + rs.getInt("id") + " Name : " 
                   + rs.getString("name") + " Age : " + rs.getInt("age")); 
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
    if(connection != null){
     connection.close();
    }
  }
 }
}
```

### DataSource example Using Apache DBCP

With stand alone Java programs where data source is needed it is more convenient to use connection pooling library like DBCP.

You need the following jars in your project’s classpath (Download path- https://commons.apache.org/proper/commons-dbcp/), check the versions as per your Java and DB versions.

```
commons-dbcp2-2.1.1.jar
commons-pool2-2.5.0.jar
commons-logging-1.2.jar
```

and the JDBC driver for the DB used. In this example MySQL is used so mysql-connector-java-5.1.39 jar is used.

Here we have a DataSource class that is a singleton class creating and returning the instance of dbcp2 BasicDataSource.

There is another class DSConnection where we get the instance of dbcp2 BasicDataSource and use it to get the Connection object.

Then a PreparedStatement object is created with a Select SQL statement to get Employee data having the passed ID.

DB schema in MySQL is netjs and table is Employee.

**DataSource.java**

```
import org.apache.commons.dbcp2.BasicDataSource;

public class DataSource {
 private static final String DRIVER_CLASS = "com.mysql.jdbc.Driver";
 private static final String DB_CONNECTION_URL = "jdbc:mysql://localhost:3306/netjs";
 private static final String DB_USER = "root";
 private static final String DB_PWD = "admin";
 private static DataSource ds;
 private BasicDataSource basicDS = new BasicDataSource();

 //private constructor
 private DataSource(){
  //BasicDataSource basicDS = new BasicDataSource();
  basicDS.setDriverClassName(DRIVER_CLASS);
  basicDS.setUrl(DB_CONNECTION_URL);
  basicDS.setUsername(DB_USER);
  basicDS.setPassword(DB_PWD);
  
  // Parameters for connection pooling
  basicDS.setInitialSize(10);
  basicDS.setMaxTotal(10); 
 }
 
 /**
   *static method for getting instance.
 */
 public static DataSource getInstance(){
    if(ds == null){
        ds = new DataSource();
    }
    return ds;
 }

 public BasicDataSource getBasicDS() {
  return basicDS;
 }

 public void setBasicDS(BasicDataSource basicDS) {
  this.basicDS = basicDS;
 }
}
```

**DSConnection.java**

```
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import org.apache.commons.dbcp2.BasicDataSource;

public class DSConnection {
 public static void main(String[] args) {
  DSConnection dsCon = new DSConnection();
  try {
   dsCon.displayEmployee(6);
  } catch (SQLException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
 
 /**
  * @param connection
  * @param id
  * @throws SQLException
  */
 private void displayEmployee(int id) throws SQLException{
  
  Connection connection = null; 
  String selectSQL = "Select * from employee where id = ?";
  PreparedStatement prepStmt = null;
  try {
   BasicDataSource basicDS = DataSource.getInstance().getBasicDS();
   connection = basicDS.getConnection();
   prepStmt = connection.prepareStatement(selectSQL);
   prepStmt.setInt(1, id);
   ResultSet rs = prepStmt.executeQuery();
   while(rs.next()){
     System.out.println("id : " + rs.getInt("id") + " Name : " 
                   + rs.getString("name") + " Age : " + rs.getInt("age")); 
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
    if(connection != null){
     connection.close();
    }
  }
 }
}
```

### DataSource example Using c3p0

Another way of getting DataSource object is using c3p0 library. With stand alone Java program you can create an instance of **ComboPooledDataSource**.

**Jars used**

```
c3p0-0.9.5.2.jar
mchange-commons-java-0.2.11.jar
```

If we retain the same class structure as explained above. Now the classes would look like -

**DataSource.java**

```
import java.beans.PropertyVetoException;
import com.mchange.v2.c3p0.ComboPooledDataSource;

public class DataSource {
 private static final String DRIVER_CLASS = "com.mysql.jdbc.Driver";
 private static final String DB_CONNECTION_URL = "jdbc:mysql://localhost:3306/netjs";
 private static final String DB_USER = "root";
 private static final String DB_PWD = "admin";
 private static DataSource ds;
 private ComboPooledDataSource cpds = new ComboPooledDataSource();

 //private constructor
 private DataSource() throws PropertyVetoException{
     cpds.setDriverClass(DRIVER_CLASS); //loads the jdbc driver            
     cpds.setJdbcUrl(DB_CONNECTION_URL);
     cpds.setUser(DB_USER);                                  
     cpds.setPassword(DB_PWD);                                  
      
     // the settings below are optional 
     // c3p0 can work with defaults
     cpds.setMinPoolSize(5);                                     
     cpds.setAcquireIncrement(5);
     cpds.setMaxPoolSize(20);
 }
 
 /**
   *Static method for getting instance.
   * @throws PropertyVetoException 
 */
 public static DataSource getInstance() throws PropertyVetoException{
    if(ds == null){
        ds = new DataSource();
    }
    return ds;
 }

 public ComboPooledDataSource getCpds() {
  return cpds;
 }

 public void setCpds(ComboPooledDataSource cpds) {
  this.cpds = cpds;
 }
}
```

**DSConnection.java**

```
import java.beans.PropertyVetoException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import com.mchange.v2.c3p0.ComboPooledDataSource;

public class DSConnection {
 public static void main(String[] args) throws PropertyVetoException {
  DSConnection dsCon = new DSConnection();
  try {
   dsCon.displayEmployee(6);
  } catch (SQLException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
 
 /**
  * @param connection
  * @param id
  * @throws SQLException
  * @throws PropertyVetoException 
  */
 private void displayEmployee(int id) throws SQLException, PropertyVetoException{
  
  Connection connection = null; 
  String selectSQL = "Select * from employee where id = ?";
  PreparedStatement prepStmt = null;
  try {
   ComboPooledDataSource basicDS = DataSource.getInstance().getCpds();
   connection = basicDS.getConnection();
   prepStmt = connection.prepareStatement(selectSQL);
   prepStmt.setInt(1, id);
   ResultSet rs = prepStmt.executeQuery();
   while(rs.next()){
     System.out.println("id : " + rs.getInt("id") + " Name : " 
                   + rs.getString("name") + " Age : " + rs.getInt("age")); 
   }
  }finally{
    if(prepStmt != null){
     prepStmt.close();
    }
    if(connection != null){
     connection.close();
    }
  }
 }
}
```

That's all for this topic **DataSource in Java-JDBC**. If you have any doubt or any suggestions to make please drop a comment. Thanks!