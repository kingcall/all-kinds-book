### CallableStatement Interface in Java-JDBC

In the post [Statement Interface in Java-JDBC](https://www.netjstech.com/2017/12/statement-interface-in-java-jdbc.html) and [PreparedStatement Interface in Java-JDBC](https://www.netjstech.com/2017/12/preparedstatement-interface-in-java-jdbc.html) we have already seen how you can use **Statement** to execute static SQL statements and **PreparedStatement** to execute precompiled parameterized SQL statements. On the same lines the JDBC API provides **CallableStatement** interface that extends PreparedStatement and used to **execute SQL stored procedures**.

### Stored Procedure

Stored procedure is a subroutine which resides with in the database and may have DB specific way of writing it. If you have a huge SQL statement or a group of SQL statements involving more than one table, checking for conditions, looping it is better to write it as a stored procedure. That way you will need to make just one call to the DB server and your pre-compiled procedure would be executed in the same space as your DB server.

That brings you the advantages like efficiency as it is already compiled, reduced network traffic as its full execution happens in the DB server.

### Obtaining JDBC CallableStatement object

CallableStatement object can be created using the prepareCall() method of the [Connection interface](https://www.netjstech.com/2017/12/connection-interface-in-java-jdbc.html).

```
CallableStatement callableStatement = connection.prepareCall(“{call PROCEDURE_NAME(?, ?, ?)}”); 
```

Here **‘?’** is a place holder used to register **IN**, **OUT** and **INOUT** parameters.

You can also also call functions using Callable statement, in that case general form would be like -

```
CallableStatement callableStatement = connection.prepareCall(“? = {call PROCEDURE_NAME(?, ?, ?)}”);
```

### How to use CallableStatement in JDBC

In order to pass values to the IN and INOUT parameters of the stored procedure you need to use the appropriate setter method. CallableStatement inherits setter methods from PreparedStatement and there are different setter methods for different data types i.e. setInt(), setString(), setDate() etc.

You also need to register OUT parameters of the stored procedure. For that you need to use **registerOutParameter** method which takes column index or column name and type as parameters. It has other [overloaded methods](https://www.netjstech.com/2015/04/method-overloading-in-java.html) too.

There are also various getter methods (like getString(), getLong(), getTime()) for *getting the values from the OUT parameters*.

In order to execute the CallableStatement you can use execute() methods -

- **execute()**- Any SQL statement. Returns a boolean which is true if the first result is a ResultSet object; false if it is an update count or there are no results.
- **executeUpdate()**- For DML statements like Insert, Update or DDL statements like Create.
- **ExecuteQuery()**- For SQL statement that returns ResultSet.

### Java CallableStatement examples

Let’s see some examples using CallableStatement in JDBC. Database used is MySql, schema is netjs and table employee with columns id, age and name, where id is auto-generated.

**1. CallableStatement example-Executing stored procedure having IN params**

In this example let’s execute a stored procedure that has **only IN params** using CallableStatement. The stored procedure inserts a new row into the table.

**insert_employee_proc.sql**

```
CREATE PROCEDURE `insert_employee_proc`(IN param_name VARCHAR(35), IN param_age int)
BEGIN
  INSERT into EMPLOYEE (name, age) values 
  (param_name, param_age);
END
```

You can see in the stored procedure that there are two IN parameters in the stored procedure.

**Java Code**

```
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class JDBCCallableStmt {

 public static void main(String[] args) {
   Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                        "root", "admin");
    
    // Getting CallableStatement object
    CallableStatement cStatement = connection.prepareCall(
       "{call insert_employee_proc(?, ?)}");
    // Setting params
    cStatement.setString(1, "Jackie");
    cStatement.setInt(2, 45);
    
    int count = cStatement.executeUpdate();
    System.out.println("Count of rows inserted " + count);
   
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

**2. CallableStatement example-Executing stored procedure having IN and OUT params**

In this Java CallableStatement example let’s execute a stored procedure that has **both IN and OUT params** using CallableStatement. The stored procedure has a select query to which id is passed as an IN parameter and age and name for that id are send in OUT parameters.

**select_employee_proc.sql**

```
CREATE PROCEDURE `select_employee_proc`(IN param_id int, 
    OUT param_name varchar(35), OUT param_age int)
BEGIN
 SELECT name, age INTO param_name, param_age
 from EMPLOYEE where id = param_id;
END
```

**Java Code**

```
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.JDBCType;
import java.sql.SQLException;

public class JDBCCallableStmt {

 public static void main(String[] args) {
   Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                        "root", "admin");
    
    // Getting CallableStatement object
    CallableStatement cStatement = connection.prepareCall(
      "{call select_employee_proc(?, ?, ?)}");
    // Setting params
    cStatement.setInt(1, 26);
    // Registering OUT parameters Using 
    // JDBCType enum which is added in Java 8
    cStatement.registerOutParameter(2, JDBCType.VARCHAR);
    
    cStatement.registerOutParameter(3, JDBCType.INTEGER);
    
    cStatement.executeQuery();
    
    // Reading the OUT paramter here 
    System.out.println("Fetched Result " + "Name: " + cStatement.getString(2) + 
      " Age: " + cStatement.getInt(3));
   
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

**3. CallableStatement example-Executing stored procedure returning multiple rows**

Let’s see an example where stored procedure returns multiple rows as result. In that case you can use execute or executeQuery to execute the procedure using CallableStatement and that will return the resultset. In this example execute method is used in order to show how it uses other methods like getResultSet and getMoreResults.

**all_employee_proc.sql**

```
CREATE PROCEDURE `all_employee_proc`(IN param_age int)
BEGIN
  SELECT * from employee where age > param_age;
END
```

The stored procedure returns all employees whose age is greater than the passed age integer parameter.

**Java code**

```
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JDBCCallableStmt {

  public static void main(String[] args) {
    Connection connection = null;
    try {
      // Loading driver
      Class.forName("com.mysql.jdbc.Driver");
      
      // Creating connection
      connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                          "root", "admin");
      
      // Getting CallableStatement object
      CallableStatement cStatement = connection.prepareCall("{call all_employee_proc(?)}");
      // Setting params
      cStatement.setInt(1, 30);
    
      boolean hasResults = cStatement.execute();
      while(hasResults){
        ResultSet rs = cStatement.getResultSet();
        while(rs.next()){
          System.out.println("id : " + rs.getInt("id") + " Name : " 
            + rs.getString("name") + " Age : " + rs.getInt("age")); 
        }
        hasResults = cStatement.getMoreResults();
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

**Output**

```
id : 6 Name : Tim Age : 40
id : 8 Name : Johnny Age : 35
id : 17 Name : Johnny Age : 35
id : 18 Name : Bob Age : 45
id : 25 Name : Jacky Age : 50
id : 26 Name : Jackie Age : 45
```

That's all for this topic **CallableStatement Interface in Java-JDBC**. If you have any doubt or any suggestions to make please drop a comment. Thanks!