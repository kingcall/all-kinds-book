### Statement Interface in Java-JDBC

In the post [Java JDBC Steps to Connect to DB](https://www.netjstech.com/2017/12/java-jdbc-steps-to-connect-to-db.html) we have already seen a complete example using the interfaces Driver, [Connection](https://www.netjstech.com/2017/12/connection-interface-in-java-jdbc.html), Statement and [ResultSet](https://www.netjstech.com/2017/12/resultset-interface-in-java-jdbc.html) provided by the JDBC API. In this post we’ll see Java Statement interface in detail.

### Statement interface in JDBC

java.sql.Statement [interface](https://www.netjstech.com/2015/05/interface-in-java.html) in JDBC API is used to execute a static SQL statement and returning the result of the executed query.

Statement interface has two sub-interfaces **CallableStatement** and **PreparedStatement**.

**PreparedStatement**– PreparedStatement object stores the SQL statement in its pre-compiled state. That way it can efficiently execute the same SQL statement multiple times with different parameters.

- Refer [PreparedStatement Interface in Java-JDBC](https://www.netjstech.com/2017/12/preparedstatement-interface-in-java-jdbc.html) to get more details about PreparedStatement.

**CallableStatement**- This interface is used to execute SQL stored procedures.

- Refer [CallableStatement Interface in Java-JDBC](https://www.netjstech.com/2017/12/callablestatement-interface-in-java-jdbc.html) to get more details about CallableStatement.

You can get a Statement object by calling the **Connection.createStatement()** method on the Connection object.

### Frequently used methods of the Statement interface

Mostly you will use the execute methods of the Java Statement interface to execute queries.

1. **boolean execute(String sql)**- Executes the given SQL statement (it can be any kind of SQL query), which may return multiple results.
   Returns a boolean which is true if the first result is a ResultSet object; false if it is an update count or there are no results.
2. **ResultSet executeQuery(String sql)**- Executes the given SQL statement, which returns a single ResultSet object. If you want to execute a Select SQL query which returns results you should use this method.
3. **int executeUpdate(String sql)**- Executes the given SQL statement, which may be an INSERT, UPDATE, or DELETE statement or an SQL statement that returns nothing, such as an SQL DDL statement.
   Returns an int denoting either the row count for the rows that are inserted, deleted, updated or returns 0 if nothing is returned.
   **Note:**This method cannot be called on a PreparedStatement or CallableStatement.
4. **int[] executeBatch()**- Submits a batch of commands to the database for execution and if all commands execute successfully, returns an array of update counts.

### Java Statement example

Let’s see an example where SQL statements are executed using execute(), executeUpdate and executeQuery methods. In the example-

Using **execute()** method a SQL statement is executed and then the boolean value is checked.

Using **executeUpdate()** method insert, update and delete statements are executed and row count of the affected rows is displayed.

Using **executeQuery()** method select statement is executed and the returned ResultSet is processed.

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCStmt {
  public static void main(String[] args) {
    Connection connection = null;
    try {
      // Loading driver
      Class.forName("com.mysql.jdbc.Driver");
   
      // Creating connection
      connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                   "root", "admin");
  
      // creating Statement
      Statement stmt = connection.createStatement();  
            
      /** execute method **/
      boolean flag = stmt.execute("Update Employee set age = 40 where id in (5, 6)");
      if(flag == false){
        System.out.println("Updated rows " + stmt.getUpdateCount() );
      }
            
      /** executeUpdate method **/
      // Insert
      int count = stmt.executeUpdate("Insert into employee(name, age) values('Kim', 23)");
      System.out.println("Rows Inserted " + count);
            
      // update
      count = stmt.executeUpdate("Update Employee set age = 35 where id = 17");
      System.out.println("Rows Updated " + count);
            
      //delete
      count = stmt.executeUpdate("Delete from Employee where id = 5");
      System.out.println("Rows Deleted " + count);
            
      /** executeQuery method **/
      // Executing Query
      ResultSet rs = stmt.executeQuery("Select * from Employee");

      // Processing Resultset
      while(rs.next()){
        System.out.println("id : " + rs.getInt("id") + " Name : " 
          + rs.getString("name") + " Age : " + rs.getInt("age")); 
      }    
    } catch (ClassNotFoundException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (SQLException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }finally{
      if(connection != null){
        //closing connection 
        try {
          connection.close();
        } catch (SQLException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      } // if condition
    }// finally
  }
}
```

