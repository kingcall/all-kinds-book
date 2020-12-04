### DatabaseMetaData Interface in Java-JDBC

DatabaseMetaData in Java, which resides in **java.sql** package, provides information about the database (DB meta data) you are connected to.

Using the methods provided by Java DatabaseMetaData interface you can get information about-

- Database like DB name and version
- JDBC driver like the driver’s name and version,
- names of DB schemas,
- name of tables in any DB schema,
- names of views,
- information about the procedures.



In this post we’ll see examples of some of the commonly used methods. You can get the list of full methods here- https://docs.oracle.com/javase/9/docs/api/java/sql/DatabaseMetaData.html

**Table of contents**

1. [How to get DatabaseMetaData object in JDBC](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html#DBMetaDataObject)
2. [DatabaseMetaData example-Getting DB product and version information](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html#DBMetaDataVersion)
3. [DatabaseMetaData example-Getting driver information](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html#DBMetaDataDriver)
4. [Example to get tables using DatabaseMetaData in JDBC](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html#DBMetaDataTable)
5. [Example to get Procedures using DatabaseMetaData in JDBC](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html#DBMetaDataProcedure)



### How to get DatabaseMetaData object in JDBC

You can get the DatabaseMetaData instance by calling the **getMetaData()** method of the Connection class.

```
DatabaseMetaData dbMetaData = connection.getMetaData();
```

### DatabaseMetaData example-Getting DB product and version information

This example code shows how you can get DB name and version information using DatabaseMetaData in JDBC.

```
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBMetaData {

 public static void main(String[] args) {
  Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                        "root", "admin");
    
    DatabaseMetaData dbMetaData = connection.getMetaData();
    
    System.out.println("Database Name - " + dbMetaData.getDatabaseProductName());
    System.out.println("Database Version - " + dbMetaData.getDatabaseProductVersion());
    System.out.println("Database Major Version - " + dbMetaData.getDatabaseMajorVersion());
    System.out.println("Database Minor Version - " + dbMetaData.getDatabaseMinorVersion());
    System.out.println("Database User - " + dbMetaData.getUserName());
    
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

### DatabaseMetaData example - Getting driver information

```
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBMetaData {
 public static void main(String[] args) {
  Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                     "root", "admin");
    
    DatabaseMetaData dbMetaData = connection.getMetaData();
    
    System.out.println("Driver Name - " + dbMetaData.getDriverName());
    System.out.println("Driver Version - " + dbMetaData.getDriverVersion());
    System.out.println("Driver Major Version - " + dbMetaData.getDriverMajorVersion());
    System.out.println("Driver Minor Version - " + dbMetaData.getDriverMinorVersion());
    System.out.println("JDBC Major Version - " + dbMetaData.getJDBCMajorVersion());
    System.out.println("JDBC Minor Version - " + dbMetaData.getJDBCMinorVersion());
    
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

### Example to get tables using DatabaseMetaData in JDBC

For getting tables you can use **getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types)** method. You can provide null as value for all the parameters, that way you don’t narrow the search and all the tables are returned. If you want to narrow your search to get specific tables then you can provide values for these parameters.

```
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

public class DBMetaData {

 public static void main(String[] args) {
  Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/world", 
                        "root", "admin");
    
    DatabaseMetaData dbMetaData = connection.getMetaData();
    
    ResultSet rs = dbMetaData.getTables(null, null, null, null);
    while (rs.next()){
     System.out.println("Table name " + rs.getString(3));
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
Table name city
Table name country
Table name countrylanguage
```

Here I am connecting to “**world**” schema in MySQL and getting all the tables.

Each table description in the returned [ResultSet](https://www.netjstech.com/2017/12/resultset-interface-in-java-jdbc.html) has the following columns:

| Column Name               | Type   | Description                                                  |
| ------------------------- | ------ | ------------------------------------------------------------ |
| TABLE_CAT                 | String | table catalog (may be null)                                  |
| TABLE_SCHEM               | String | table schema (may be null)                                   |
| TABLE_NAME                | String | table name                                                   |
| TABLE_TYPE                | String | table type. Typical types are "TABLE", "VIEW" etc.           |
| REMARKS                   | String | explanatory comment on the table (may be null)               |
| TYPE_CAT                  | String | the types catalog (may be null)                              |
| TYPE_SCHEM                | String | the types schema (may be null)                               |
| TYPE_NAME                 | String | type name (may be null)                                      |
| SELF_REFERENCING_COL_NAME | String | name of the designated "identifier" column of a typed table (may be null) |
| REF_GENERATION            | String | specifies how values in SELF_REFERENCING_COL_NAME are created. |

That’s why column index is 3 while getting result from ResultSet as **TABLE_NAME** is at number 3.

### Example to get Procedures using DatabaseMetaData in JDBC

For getting procedures you can use **getProcedures(String catalog, String schemaPattern, String procedureNamePattern)** method. Again you can pass null as value for all the parameters if you don’t want to narrow the search.

```
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

public class DBMetaData {

 public static void main(String[] args) {
  Connection connection = null;
   try {
    // Loading driver
    Class.forName("com.mysql.jdbc.Driver");
    
    // Creating connection
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs", 
                        "root", "admin");
    
    DatabaseMetaData dbMetaData = connection.getMetaData();
    
    ResultSet rs = dbMetaData.getProcedures(null, null, null);
    
    while (rs.next()){
     System.out.println("Procedure name " + rs.getString(3));
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

Each procedure description in the returned ResultSet has the following columns:

| Column Name             | Type   | Description                                                  |
| ----------------------- | ------ | ------------------------------------------------------------ |
| PROCEDURE_CAT           | String | procedure catalog (may be null)                              |
| PROCEDURE_SCHEM         | String | procedure schema (may be null)                               |
| PROCEDURE_NAME          | String | procedure name                                               |
| reserved for future use |        |                                                              |
| reserved for future use |        |                                                              |
| reserved for future use |        |                                                              |
| REMARKS                 | String | explanatory comment on the procedure                         |
| PROCEDURE_TYPE          | short  | type name (may be null)                                      |
| SPECIFIC_NAME           | String | The name which uniquely identifies this procedure within its schema. |

That’s why column index is 3 while getting result from ResultSet as **PROCEDURE_NAME** is at number 3.

That's all for this topic **DatabaseMetaData Interface in Java-JDBC**. If you have any doubt or any suggestions to make please drop a comment. Thanks!