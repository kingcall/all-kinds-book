### Batch Processing in Java JDBC - Insert, Update Queries as a Batch

If you have a large number of similar queries it is better to process them in a batch rather than as individual queries.

Processing them as a batch provides better performance as you send a group of queries in a single network communication rather than sending individual queries one by one.

### Batch support in JDBC

JDBC provides batch support in the form of **addBatch()** and **executeBatch()** methods.

If you are using [Statement interface](https://www.netjstech.com/2017/12/statement-interface-in-java-jdbc.html) then use **addBatch(String Sql)** method to add queries to the batch.

For [PreparedStatement](https://www.netjstech.com/2017/12/preparedstatement-interface-in-java-jdbc.html) and [CallableStatement](https://www.netjstech.com/2017/12/callablestatement-interface-in-java-jdbc.html) use **addBatch()** method as parameters for the query are provided later.

Once you have added queries to a batch you can call **executeBatch()** method to execute the whole batch of queries.

DB you are using may not support batch updates. You should use the **supportsBatchUpdates()** method of the [DatabaseMetaData interface](https://www.netjstech.com/2017/12/databasemetadata-interface-in-java-jdbc.html) to check whether the target database supports batch updates or not.

The method returns true if this database supports batch updates; false otherwise.



### JDBC batch processing example

As already stated if you have bulk processing to be done where you insert, update or delete a large number of records its better to run those queries as a batch. **As example** - In many applications you get a CSV file or a list of records for insertion, deletion or update in bulk.

In the example we’ll insert a list of ten employees into the table in MySQL DB. We’ll add insert records in a batch and then execute the batch. Here **PreparedStatement object** is used to execute the batch.

Here the batch is executed when batch size is 5.

In MySQL DB you will have to add **rewriteBatchedStatements=true** parameter in the DB URL then only JDBC will pack as many queries as possible into a single network packet otherwise, even with batch queries will be send individually.

So the **getConnection()** method which creates a connection using MySQL DB properties will look like-

```
/**
 * Method for getting the connection
 * @return
 */
public static Connection getConnection(){
  Connection connection = null;
  try {
   // Loading driver
   Class.forName("com.mysql.jdbc.Driver");
   
   // Creating connection
   connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/netjs?
                rewriteBatchedStatements=true","root","admin");
   
  } catch (ClassNotFoundException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
  } catch (SQLException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
  }
  return connection;
}
```

**insertEmployee()** method which takes the list as argument as process queries in a batch size of 5 will look like-

```
/**
 * 
 * @param connection
 * @throws SQLException
 */
private void insertEmployee(List<EmployeeBean> empList){
    Connection connection = null;
    String insertSQL = "Insert into employee (name, age) values (?, ?)";
    PreparedStatement prepStmt = null;
    int[] count;
    try {
        connection = getConnection();
        prepStmt = connection.prepareStatement(insertSQL);
        for(int i = 0; i < empList.size(); i++){
            EmployeeBean emp = empList.get(i);
            prepStmt.setString(1, emp.getName());
            prepStmt.setInt(2, emp.getAge());
            prepStmt.addBatch();
            // Process batch of 5 records
            if(i%5 == 0){
                count = prepStmt.executeBatch();
            }
            
        }
        count = prepStmt.executeBatch();

    }catch (SQLException e) {
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
    }
} 
```

### JDBC Batch processing with transaction

Queries in a batch are *not part of a transaction automatically*. Which means if any statement in the batch fails, the statements already executed in the batch are committed to the DB. If you have to roll back the whole batch in case of failure then you need to run your batch inside the transaction so that all the queries in the batch are executed or none.

- Refer [Transaction in Java-JDBC](https://www.netjstech.com/2018/01/transaction-in-java-jdbc.html) to know more about transaction management in JDBC.

With the transaction related changes the insertEmployee() method will look like -

```
private void insertEmployee(List<EmployeeBean> empList){
    Connection connection = null;
    String insertSQL = "Insert into employee (name, age) values (?, ?)";
    PreparedStatement prepStmt = null;
    int[] count;
    try {
        connection = getConnection();
        // Setting auto commit false
        connection.setAutoCommit(false);
        prepStmt = connection.prepareStatement(insertSQL);
        for(int i = 0; i < empList.size(); i++){
            EmployeeBean emp = empList.get(i);
            prepStmt.setString(1, emp.getName());
            prepStmt.setInt(2, emp.getAge());
            prepStmt.addBatch();
            // Process batch of 5 records
            if(i%5 == 0){
                count = prepStmt.executeBatch();
                
            }
            
        }
        count = prepStmt.executeBatch();
        // Committing the transaction
        connection.commit();
    }catch (SQLException e) {
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
    }
}
```

That's all for this topic **Batch Processing in Java JDBC - Insert, Update Queries as a Batch**. If you have any doubt or any suggestions to make please drop a comment. Thanks!