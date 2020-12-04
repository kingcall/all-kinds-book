### ResultSet Interface in Java-JDBC

java.sql.ResultSet interface in JDBC API represents the storage for the data you get by executing a SQL statement that queries the database.

A ResultSet object maintains a **cursor** pointing at the result data. Initially the cursor is positioned before the first row. The **next method** moves the cursor to the next row, it returns false when there are no more rows in the ResultSet object.

### Creating and iterating a ResultSet Example

```
ResultSet rs = stmt.executeQuery("Select * from Employee");
   
// Processing Resultset
while(rs.next()){
 System.out.println("id : " + rs.getInt("id") + " Name : " +  rs.getString("name") + " Age : " + rs.getInt("age")); 
}
```

By default, ResultSet [object](https://www.netjstech.com/2015/04/object-in-java.html) is not updatable and has a forward moving cursor only. Thus, you can iterate through it only once and only from the first row to the last row. But ResultSet interface provides parameters that can produce ResultSet objects that are scrollable and/or updatable.

### Fields for scrollable ResultSet

ResultSet interface in Java has fields that determine whether ResultSet object will be scrollable or not and will it be sensitive to the changes to the data that is represented by ResultSet or not.

- **TYPE_FORWARD_ONLY**- The constant indicating the type for a ResultSet object whose cursor may move only forward.
- **TYPE_SCROLL_INSENSITIVE**- The constant indicating the type for a ResultSet object that is scrollable but generally not sensitive to changes to the data that underlies the ResultSet. Which means you can move the cursor to an absolute position or relative to the current cursor position. If the data in the DB is changed by another thread/process that change won’t be reflected in the data stored in the ResultSet.
- **TYPE_SCROLL_SENSITIVE**- The constant indicating the type for a ResultSet object that is scrollable and generally sensitive to changes to the data that underlies the ResultSet. Which means you can move the cursor to an absolute position or relative to the current cursor position. If the data in the DB is changed by another thread/process that change is reflected in the data stored in the ResultSet.

### Fields for updatable ResultSet

- **CONCUR_READ_ONLY**- The constant indicating the concurrency mode for a ResultSet object that may NOT be updated.
- **CONCUR_UPDATABLE**- The constant indicating the concurrency mode for a ResultSet object that may be updated.

### Java ResultSet interface example

Let’s see an example with scrollable resultset, DB used here is MySql, schema is netjs and table is Employee.

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCResultSet {

 public static void main(String[] args) {
  try(Connection connection = DriverManager.getConnection(
      "jdbc:mysql://localhost:3306/netjs", "root", "admin")){
   // creating Statement
   Statement stmt = connection.createStatement(
                           ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);  
   
   // Executing Query
   ResultSet rs = stmt.executeQuery("Select * from Employee");
   System.out.println("Displaying all rows");
   // Processing Resultset
   while(rs.next()){
       System.out.println("id : " + rs.getInt("id") + " Name : " 
        + rs.getString("name") + " Age : " + rs.getInt("age")); 
   }
   // moving to 3rd row
   rs.absolute(3);
   System.out.println("Displaying 3rd row");
   System.out.println("id : " + rs.getInt("id") + " Name : " 
                          + rs.getString("name") + " Age : " + rs.getInt("age")); 
  }catch (SQLException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
}
```

**Output**

```
Displaying all rows
id : 5 Name : Tom Age : 35
id : 6 Name : Tim Age : 20
id : 7 Name : John Age : 25
id : 8 Name : Johnny Age : 35
id : 17 Name : Johnny Age : 65
Displaying 3rd row
id : 7 Name : John Age : 25
```

### Getter Methods in ResultSet

You would have noticed in the examples how appropriate data type getter method is used (i.e. getInt, getString) for retrieving column values from the current row. You can retrieve value using either the index number of the column or the name of the column.

In general, using the column index will be more efficient. Columns are numbered from 1. Drawback is, any alteration in the table structure will mean change in the indexes in the Java code.

### Updater methods in ResultSet

There are also updater methods corresponding to the data types which are used when your ResultSet is updatable. Using updater methods you can update the column values then update the row in the DB. Updater methods are used in conjunction with **updateRow** and **insertRow** methods.

Let’s see an example to update a row and insert a row using ResultSet methods.

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCResultSetUpdate {

 public static void main(String[] args) {
  try(Connection connection = DriverManager.getConnection(
                   "jdbc:mysql://localhost:3306/netjs", "root", "admin")){
   // creating Statement
   Statement stmt = connection.createStatement(
                          ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);  
   
   // Executing Query
   ResultSet rs = stmt.executeQuery("Select * from Employee");
   System.out.println("Displaying all rows");
   // Processing Resultset
   while(rs.next()){
       System.out.println("id : " + rs.getInt("id") + " Name : " 
                               + rs.getString("name") + " Age : " + rs.getInt("age")); 
   }
   // moving to 3rd row
   rs.absolute(3);
   // updating age column for 3rd row
   rs.updateInt("age", 28);
   rs.updateRow();
   System.out.println("Displaying 3rd row");
   System.out.println("id : " + rs.getInt("id") + " Name : " 
                            + rs.getString("name") + " Age : " + rs.getInt("age"));
   
   /*** Inserting row  ***/
   // moves cursor to the insert row
   rs.moveToInsertRow(); 
     
   //rs.updateInt("id",18); //updates the first column using column name
   rs.updateString(2, "Bob"); //updates the second column using column index
   rs.updateInt("age",45);
   rs.insertRow();
   rs.moveToCurrentRow();
   
  }catch (SQLException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
}
```

### Other fields of ResultSet interface

Some of the fields are already mentioned with examples, ResultSet in Java has some other fields which are as follows-

- **CLOSE_CURSORS_AT_COMMIT**- This constant indicates that open ResultSet will be closed when the current transaction is commited.
- **HOLD_CURSORS_OVER_COMMIT**- This constant indicates that open ResultSet will remain open when the current transaction is commited.
- **FETCH_FORWARD**- The constant indicating that the rows in a result set will be processed in a forward direction; first-to-last.
- **FETCH_REVERSE**- The constant indicating that the rows in a result set will be processed in a reverse direction; last-to-first.
- **FETCH_UNKNOWN**- The constant indicating that the order in which rows in a result set will be processed is unknown.

### Methods of the ResultSet

Most of the often used methods of the ResultSet are already covered with the examples. Some of the other methods which are used for moving the cursor are as follows-

- **afterLast()**- Moves the cursor to the end of this ResultSet object, just after the last row.
- **beforeFirst()**- Moves the cursor to the front of this ResultSet object, just before the first row.
- **first()**- Moves the cursor to the first row in this ResultSet object.
- **last()**- Moves the cursor to the last row in this ResultSet object.
- **moveToCurrentRow()**- Moves the cursor to the remembered cursor position, usually the current row.
- **moveToInsertRow()**- Moves the cursor to the insert row.
- **next()**- Moves the cursor froward one row from its current position.
- **previous()**- Moves the cursor to the previous row in this ResultSet object.
- **relative(int rows)**- Moves the cursor a relative number of rows, either positive or negative.

**Reference**: https://docs.oracle.com/en/java/javase/12/docs/api/java.sql/java/sql/ResultSet.html

That's all for this topic **ResultSet Interface in Java-JDBC**. If you have any doubt or any suggestions to make please drop a comment. Thanks!